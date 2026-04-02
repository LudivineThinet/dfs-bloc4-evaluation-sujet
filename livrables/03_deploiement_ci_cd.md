# Deploiement automatise

> Competence evaluee : `C31` — Mettre en oeuvre un systeme de deploiement automatise respectant les bonnes pratiques DevOps.

## 1. Strategie de deploiement

### 1.1 Vue d'ensemble

La stratégie retenue est un déploiement par script bash déclenché manuellement depuis la machine locale. Le script transfère le code source depuis l'environnement de qualification vers l'environnement de production via SSH, puis exécute les commandes de mise à jour Laravel à distance.

Cette approche a été choisie en raison de l'indisponibilité du dépôt GitHub source (`itakademy/dfs-bloc4-evaluation-app`) depuis les serveurs, rendant un pipeline CI/CD classique (GitHub Actions) non applicable dans le contexte de l'épreuve.

### 1.2 Diagramme du pipeline
```
[Machine locale]
      │
      │ 1. Déclenchement manuel du script deploy.sh
      ▼
[Serveur qualification 13.39.47.65]
      │
      │ 2. Compression du code source (/var/www/opstrack)
      ▼
[Transfert SSH vers production]
      │
      │ 3. Décompression sur le serveur de production
      ▼
[Serveur production 13.39.47.35]
      │
      │ 4. Migrations, cache, permissions
      ▼
[Smoke test — vérification HTTP/HTTPS]
```

## 2. Outillage retenu

| Outil | Role dans le pipeline | Justification |
| --- | --- | --- |
| Bash | Script de déploiement | Disponible nativement sur tous les serveurs Linux |
| SSH | Transfert sécurisé et exécution distante | Accès sécurisé aux serveurs via clé privée |
| tar | Compression et transfert du code | Outil standard Linux, rapide et fiable |
| php artisan | Commandes Laravel post-déploiement | Migrations, cache, configuration |
| curl | Smoke test post-déploiement | Vérification de la disponibilité HTTP/HTTPS |

## 3. Declenchement du deploiement

### 3.1 Mode de declenchement

Déclenchement **manuel** depuis la machine locale en exécutant le script `deploy.sh` :
```bash
bash deploy.sh
```

### 3.2 Reproductibilite

Le script est idempotent — il peut être exécuté plusieurs fois avec le même résultat. Chaque exécution transfère la version courante du code de qualification vers la production.

## 4. Controles prealables au deploiement

| Controle | Description | Critere de passage |
| --- | --- | --- |
| Disponibilité qualification | Vérification que le serveur de qualification répond | HTTP 200 sur `http://eval-dfs-q-tpl-20262-09.it-students.fr` |
| Disponibilité production | Vérification que le serveur de production est accessible | Connexion SSH établie |
| Espace disque production | Vérification de l'espace disponible | Moins de 85% d'utilisation |

## 5. Mise a jour de la production

1. Transfert du code depuis la qualification vers la production via SSH et tar
2. Attribution des permissions : `chown -R www-data:www-data /var/www/opstrack`
3. Exécution des migrations : `php artisan migrate --force`
4. Vidage du cache : `php artisan cache:clear && php artisan config:clear`
5. Rechargement Apache : `systemctl reload apache2`

## 6. Verification post-deploiement

### 6.1 Smoke tests

| Test | Commande ou methode | Resultat attendu |
| --- | --- | --- |
| Disponibilité HTTP | `curl -o /dev/null -s -w "%{http_code}" http://eval-dfs-p-tpl-20262-09.it-students.fr` | 200 ou 301 |
| Disponibilité HTTPS | `curl -o /dev/null -s -w "%{http_code}" https://eval-dfs-p-tpl-20262-09.it-students.fr` | 200 |
| Endpoint santé API | `curl https://eval-dfs-p-tpl-20262-09.it-students.fr/api/health` | `{"status":"ok"}` |

### 6.2 Preuve de deploiement reussi

Le script `deploy.sh` a été exécuté avec succès et a retourné :
```
=== Vérification qualification ===
OK
=== Transfert du code ===
tar: Removing leading `/' from member names
=== Permissions ===
=== Migrations et cache ===
   INFO  Nothing to migrate.
   INFO  Application cache cleared successfully.
   INFO  Configuration cache cleared successfully.
=== Rechargement Apache ===
=== Smoke test ===
Déploiement réussi ✅

## 7. Conduite a tenir en cas d'echec

1. **Identifier l'étape en échec** via les logs Apache : `sudo tail -50 /var/log/apache2/opstrack_error.log`
2. **Rollback** : restaurer la version précédente depuis une sauvegarde ou re-transférer depuis la qualification
3. **Vérifier les permissions** : `ls -la /var/www/opstrack/storage`
4. **Vérifier les migrations** : `php artisan migrate:status`
5. **Notifier** l'équipe d'exploitation en cas d'indisponibilité prolongée

## 8. Scripts et fichiers de configuration

| Fichier | Role |
| --- | --- |
| `deploy.sh` | Script principal de déploiement qualification → production |
| `/var/www/opstrack/.env` | Configuration de l'environnement de production |
| `/etc/apache2/sites-available/opstrack.conf` | Configuration du VirtualHost Apache |
| `/etc/apache2/sites-available/opstrack-le-ssl.conf` | Configuration HTTPS générée par Certbot |
