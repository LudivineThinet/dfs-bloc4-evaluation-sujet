# Supervision, journalisation, sauvegarde et maintenance corrective

> Competence evaluee : `C32` — Mettre en oeuvre un systeme de supervision pour detecter, diagnostiquer et corriger bugs, incidents et failles.

## 1. Journalisation

### 1.1 Services journalises

| Service | Emplacement des journaux | Niveau de detail |
| --- | --- | --- |
| Apache2 | `/var/log/apache2/opstrack_error.log` | Erreurs HTTP, tentatives d'attaque |
| Apache2 | `/var/log/apache2/opstrack_access.log` | Toutes les requêtes entrantes |
| Laravel | `/var/www/opstrack/storage/logs/laravel.log` | Erreurs applicatives, debug |
| MySQL | `/var/log/mysql/error.log` | Erreurs base de données |
| MongoDB | `/var/log/mongodb/mongod.log` | Erreurs et événements NoSQL |

### 1.2 Configuration de la journalisation

La journalisation Laravel est configurée via le fichier `.env` :
- `LOG_CHANNEL=stack`
- `LOG_LEVEL=debug`

Les logs Apache sont configurés dans le virtualhost : erreurs dans `opstrack_error.log`, accès dans `opstrack_access.log`.

## 2. Outils et configurations d'audit

Les outils suivants ont été utilisés pour auditer l'état du système et de l'application :

| Outil | Usage |
| --- | --- |
| `systemctl status` | Vérification de l'état des services (Apache, MySQL, Redis, MongoDB) |
| `sudo tail -50 /var/log/apache2/opstrack_error.log` | Lecture des erreurs Apache |
| `sudo tail -50 /var/log/apache2/opstrack_access.log` | Analyse des accès et détection d'attaques |
| `cat /var/www/opstrack/.env` | Audit de la configuration applicative |
| `sudo ufw status` | Vérification des règles de pare-feu |
| `ls /tmp/*.php` | Recherche de fichiers PHP suspects |

L'audit a permis d'identifier deux problèmes majeurs : une mauvaise configuration du cache (Redis non utilisé) et des tentatives d'intrusion actives depuis l'IP `206.189.35.70`.

## 3. Supervision et alertes

### 3.1 Sondes mises en place

| Sonde | Cible | Seuil ou condition | Action en cas d'alerte |
| --- | --- | --- | --- |
| Disponibilité HTTP | `http://eval-dfs-q-tpl-20262-09.it-students.fr` | Réponse non 200 | Redémarrage Apache + notification |
| État MySQL | Service `mysql` | Service arrêté | Redémarrage + notification |
| État Redis | Service `redis` | Service arrêté | Redémarrage + notification |
| État MongoDB | Service `mongod` | Service arrêté | Redémarrage + notification |
| Espace disque | Partition `/` | Utilisation > 85% | Notification (actuellement à 77.5%) |
| Logs d'erreur Apache | `/var/log/apache2/opstrack_error.log` | Nouvelles erreurs critiques | Analyse manuelle |

### 3.2 Mecanisme d'alerte

Dans le cadre de cet examen, la supervision est assurée manuellement via la lecture des logs et la vérification des services. En production, il serait recommandé de mettre en place un outil de supervision comme **UptimeRobot** (gratuit, surveille la disponibilité HTTP) ou **Netdata** (supervision système en temps réel) avec des alertes par email.

## 4. Strategie de sauvegarde et restauration

### 4.1 Elements sauvegardes

| Element | Methode | Frequence | Retention |
| --- | --- | --- | --- |
| Base MySQL | `mysqldump -u root -p opstrack > backup.sql` | Quotidienne | 7 jours |
| Base MongoDB | `mongodump --db opstrack_logs` | Quotidienne | 7 jours |
| Fichier `.env` | Copie manuelle sécurisée | A chaque modification | Illimitée |
| Code source | Dépôt GitHub | A chaque commit | Illimitée |

### 4.2 Procedure de restauration

Pour restaurer la base MySQL :
```bash
mysql -u root -p opstrack < backup.sql
```

Pour restaurer MongoDB :
```bash
mongorestore --db opstrack_logs dump/opstrack_logs
```

La procédure de sauvegarde a été validée sur l'environnement de qualification. Une restauration complète en production nécessiterait l'arrêt temporaire de l'application.

## 5. Diagnostic et correction du bug technique

### 5.1 Symptome observe

**a) Cache Redis mal configuré**
Le tableau de bord principal affiche des compteurs (tickets ouverts, critiques, techniciens) qui ne correspondent pas à l'état réel des données. Les valeurs semblent figées et ne se mettent pas à jour correctement.

**b) Filtres de recherche incohérents**
Une recherche de tickets combinée à un filtre de priorité retourne des résultats incorrects — des tickets de toutes priorités apparaissent malgré le filtre appliqué.

### 5.2 Demarche de diagnostic

**a) Cache Redis mal configuré**
- Lecture du fichier `.env` : `cat /var/www/opstrack/.env`
- Vérification de la configuration du cache : `CACHE_STORE=database`
- Or l'architecture de l'application prévoit Redis comme système de cache
- Vérification que Redis tourne bien : `systemctl status redis` → actif

**b) Filtres de recherche incohérents**
- Lecture du fichier `app/Http/Controllers/Api/TicketController.php`
- Identification de l'utilisation de `orWhereRaw` avec interpolation directe de `$search`
- La requête SQL générée ne groupe pas correctement le `OR` avec le filtre de priorité

### 5.3 Cause racine identifiee

**a) Cache Redis mal configuré**
Le cache est configuré sur la base de données MySQL (`CACHE_STORE=database`) au lieu de Redis (`CACHE_STORE=redis`). Les données mises en cache ne se rafraîchissent pas avec la même rapidité, ce qui provoque l'affichage de compteurs périmés sur le tableau de bord.

**b) Filtres de recherche incohérents**
```php
// Code problématique
$query->where('title', 'like', "%{$search}%")
    ->orWhereRaw("reference like '%{$search}%'");
```
Le `OR` sans parenthèses fait que le filtre priorité ne s'applique pas correctement sur les résultats.

### 5.4 Correctif applique

**a) Cache Redis mal configuré**

Modification du fichier `.env` :
```
CACHE_STORE=redis
```
Puis vidage du cache existant :
```bash
cd /var/www/opstrack
php artisan cache:clear
php artisan config:clear
```

**b) Filtres de recherche incohérents**
```php
// Code corrigé
$query->where(function ($q) use ($search) {
    $q->where('title', 'like', "%{$search}%")
      ->orWhere('reference', 'like', "%{$search}%");
});
```
Le groupement via `where(function...)` isole le `OR` entre parenthèses. `orWhere` remplace `orWhereRaw` — Laravel gère l'échappement automatiquement.

### 5.5 Verification apres correction

**a) Cache Redis mal configuré**
Rechargement de la page du tableau de bord et vérification que les compteurs correspondent aux données réelles en base MySQL.

**b) Filtres de recherche incohérents**
```bash
php -l /var/www/opstrack/app/Http/Controllers/Api/TicketController.php
# Résultat : No syntax errors detected
```

## 6. Diagnostic et correction de la faille de securite

### 6.1 Faille identifiee

Deux types d'attaques ont été détectés dans les logs Apache en provenance de l'IP `206.189.35.70` :

1. **Path traversal** : tentatives d'accès à `/cgi-bin/../../../../bin/sh` via des chemins encodés en URL, visant à exécuter un shell système.
2. **Injection via paramètre `lang`** : requêtes ciblant `index.php?lang=../../../../usr/local/lib/php/pearcmd` pour tenter de créer un fichier PHP malveillant dans `/tmp/`. Ces requêtes ont reçu une réponse HTTP 200.
3. **Tentative de vol du fichier `.env`** : requête `GET /.env` interceptée (réponse 404).

### 6.2 Demarche de diagnostic

- Connexion SSH au serveur de qualification
- Lecture des logs Apache : `sudo tail -50 /var/log/apache2/opstrack_error.log` et `opstrack_access.log`
- Vérification de la présence de fichiers PHP suspects dans `/tmp/` : `ls /tmp/*.php` → aucun fichier trouvé

### 6.3 Evaluation du risque

| Attaque | Sévérité | Résultat |
| --- | --- | --- |
| Path traversal | Haute | Bloquée par Apache (AH10244) |
| Injection via `lang` | Critique | Réponse 200 — serveur a traité la requête |
| Vol du `.env` | Haute | Bloquée (404) |

L'injection via le paramètre `lang` est la plus préoccupante : le serveur a répondu 200, ce qui indique que la requête a été traitée par Laravel sans validation suffisante de l'entrée.

### 6.4 Mesure corrective appliquee

**1. Désactivation de `Options Indexes` dans la configuration Apache** pour empêcher la navigation dans les dossiers :
```apache
<Directory /var/www/opstrack/public>
    AllowOverride All
    Require all granted
    Options -Indexes
</Directory>
```

**2. Validation des paramètres d'entrée** : le paramètre `lang` ne devrait accepter que des valeurs alphanumériques connues. À corriger dans le code Laravel.

**3. Blocage de l'IP malveillante** :
```bash
sudo ufw deny from 206.189.35.70
```

### 6.5 Verification apres correction
```bash
sudo apache2ctl configtest
sudo systemctl reload apache2
sudo ufw status
```

## 7. Autres observations

- **Mises à jour système en attente** : 42 paquets sont à mettre à jour dont 12 correctifs de sécurité. Commande : `sudo apt upgrade`
- **Redémarrage système requis** : le serveur signale un redémarrage nécessaire suite à des mises à jour noyau.
- APP_URL corrigée : le fichier .env pointait vers une ancienne machine (ec2-13-39-149-207.eu-west-3.compute.amazonaws.com). L'URL a été mise à jour avec l'URL de production.
- OPSTRACK_API_TOKEN sécurisé : le token a été remplacé par une valeur générée via openssl rand -base64 32.
- **Disque à 77.5%** : l'espace disque est déjà significativement utilisé, à surveiller.
- **Tentative de vol du `.env`** : une requête `GET /.env` a été interceptée (réponse 404). Le fichier n'est pas accessible publiquement mais la tentative confirme l'intérêt des attaquants pour ce fichier.
