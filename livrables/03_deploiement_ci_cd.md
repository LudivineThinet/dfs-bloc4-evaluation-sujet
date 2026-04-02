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
      │ 1. Déclenchement manuel : bash deploy.sh
      ▼
[Vérification qualification]
      │
      │ 2. HTTP 200 sur l'environnement de qualification
      ▼
[Transfert SSH + tar]
      │
      │ 3. Code compressé depuis qualification → production
      ▼
[Post-déploiement sur production]
      │
      │ 4. Permissions, migrations, cache, reload Apache
      ▼
[Smoke test]
      │
      │ 5. HTTP 200 sur https://eval-dfs-p-tpl-20262-09.it-students.fr
      ▼
[Déploiement réussi ]
```

## 2. Outillage retenu

| Outil | Role dans le pipeline | Justification |
| --- | --- | --- |
| Bash | Script de déploiement `deploy.sh` | Disponible nativement sur tous les systèmes Unix |
| SSH | Transfert sécurisé et exécution distante | Accès sécurisé aux serveurs via clé privée |
| tar | Compression et transfert du code | Outil standard Linux, rapide et fiable |
| php artisan | Commandes Laravel post-déploiement | Migrations, cache, configuration |
| curl | Vérification pre et post-déploiement | Smoke test HTTP/HTTPS |

## 3. Declenchement du deploiement

### 3.1 Mode de declenchement

Déclenchement manuel depuis la machine locale :
```bash
bash ~/deploy.sh
```

### 3.2 Reproductibilite

Le script est idempotent — il peut être exécuté plusieurs fois avec le même résultat. Chaque exécution transfère la version courante du code de qualification vers la production.

## 4. Controles prealables au deploiement

| Controle | Description | Critere de passage |
| --- | --- | --- |
| Disponibilité qualification | Requête HTTP sur l'environnement de qualification | HTTP 200 |
| Accès SSH production | Connexion SSH au serveur de production | Connexion établie |
| Espace disque | Vérification implicite lors du transfert | Transfert sans erreur |

## 5. Mise a jour de la production

1. Transfert du code via `tar` et `SSH` depuis la qualification
2. Attribution des permissions : `chown -R www-data:www-data` et `chmod -R 775 storage`
3. Exécution des migrations : `php artisan migrate --force`
4. Vidage du cache : `php artisan cache:clear && php artisan config:clear`
5. Rechargement Apache : `systemctl reload apache2`

## 6. Verification post-deploiement

### 6.1 Smoke tests

| Test | Commande ou methode | Resultat attendu |
| --- | --- | --- |
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
Déploiement réussi 
```

## 7. Conduite a tenir en cas d'echec

1. Identifier l'étape en échec dans la sortie du script
2. Consulter les logs Apache : `sudo tail -50 /var/log/apache2/opstrack_error.log`
3. Vérifier les permissions : `ls -la /var/www/opstrack/storage`
4. Vérifier les migrations : `php artisan migrate:status`
5. En cas d'indisponibilité prolongée, re-transférer depuis la qualification

## 8. Scripts et fichiers de configuration

| Fichier | Role |
| --- | --- |
| `deploy.sh` | Script principal de déploiement qualification → production |
| `/var/www/opstrack/.env` | Configuration de l'environnement de production |
| `/etc/apache2/sites-available/opstrack.conf` | Configuration du VirtualHost Apache |
| `/etc/apache2/sites-available/opstrack-le-ssl.conf` | Configuration HTTPS générée par Certbot |
