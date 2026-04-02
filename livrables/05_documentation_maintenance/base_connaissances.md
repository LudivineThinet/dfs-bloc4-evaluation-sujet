# Base de connaissances — Note de passation
Ce document est destiné à un pair chargé de reprendre la maintenance de l'application.

## 1. Presentation de l'application

OpsTrack Field Service est une application web de supervision des interventions techniques. Elle permet aux équipes support et exploitation de suivre les tickets d'incident, les techniciens planifiés et les incidents prioritaires via un tableau de bord centralisé.

## 2. Architecture technique

### 2.1 Composants principaux

| Composant | Technologie | Role |
| --- | --- | --- |
| Application principale | Laravel 12 / PHP 8.4 | Interface web, API REST, traitement du webhook |
| Serveur web | Apache2 2.4 | Sert Laravel et proxyfie vers le microservice Next.js |
| Base relationnelle | MySQL 8.0 | Données métier : tickets, techniciens, sites, interventions |
| Base NoSQL | MongoDB 8.0 | Journaux techniques et événements applicatifs |
| Cache | Redis 7.0 | Cache applicatif et stockage temporaire |
| Microservice dashboard | Next.js 15 / React 19 | Tableau de bord secondaire (`/dispatch-dashboard`) |
| API externe | Open-Meteo | Données météo enrichissant les interventions |

### 2.2 Schema d'architecture
```
Internet → Apache2 → Laravel 12
                  ↓           ↓           ↓
               MySQL       MongoDB      Redis
                  ↓
           Next.js (port 3000) via proxy Apache
                  ↓
           API Open-Meteo (externe)
```

## 3. Points d'attention connus

- Le microservice Next.js doit être démarré manuellement sur la production
- La base de production est vide — pas de données initiales
- 75 mises à jour système en attente sur la production dont 30 de sécurité
- Le disque est utilisé à 77% sur la qualification — à surveiller

## 4. Procedures operationnelles

### 4.1 Deploiement

Exécuter le script depuis la machine locale :
```bash
bash ~/deploy.sh
```
Le script transfère le code de la qualification vers la production, exécute les migrations et vérifie la disponibilité.

### 4.2 Sauvegarde et restauration

Sauvegarde MySQL :
```bash
mysqldump -u root -p opstrack > backup_$(date +%Y%m%d).sql
```
Restauration :
```bash
mysql -u root -p opstrack < backup.sql
```

### 4.3 Supervision et alertes

- Logs Apache : `/var/log/apache2/opstrack_error.log` et `opstrack_access.log`
- Endpoint de santé API : `GET /api/health` → doit retourner `{"status":"ok"}`
- Vérifier l'espace disque régulièrement : `df -h`

### 4.4 Acces et secrets

- Accès SSH : `ssh -i ubuntu.pem ubuntu@13.39.47.35` (production)
- Secrets dans `/var/www/opstrack/.env` — ne jamais versionner ce fichier
- phpMyAdmin qualification : `http://eval-dfs-q-tpl-20262-09.it-students.fr/phpmyadmin`

## 5. Bugs et failles corriges pendant l'epreuve

**Bugs corrigés :**
- Cache Redis mal configuré (`CACHE_STORE=database` → `redis`) — causait des compteurs périmés
- `TicketController` : `orWhereRaw` mal groupé — filtres de priorité désormais corrects
- `WebhookController` : déduplication via `firstOrCreate()` + statut mis à jour depuis le payload
- `Next.js dispatch-dashboard` : `payload.items` → `payload.data` — tableau de bord fonctionnel

**Failles corrigées :**
- `Options Indexes` désactivé dans Apache
- IP malveillante `206.189.35.70` bloquée via `ufw`
- `OPSTRACK_API_TOKEN` remplacé par un token sécurisé via `openssl rand -base64 32`
- `.env.example` : identifiants réels remplacés par des placeholders
- Injection SQL via `orWhereRaw` corrigée dans `TicketController`
- `WebhookController` : statut forcé à `scheduled` remplacé par le statut du payload
- `Next.js dispatch-dashboard` : `payload.items` → `payload.data`
- Token Next.js `.env.local` : `change-me` remplacé par le token sécurisé

**Failles identifiées non corrigées :**
- `hooks.php` sans signature HMAC complémentaire — correction recommandée pour une prochaine itération

## 6. Ameliorations recommandees

- Mettre à jour les 75 paquets système en attente
- Démarrer et superviser le microservice Next.js en production
- Implémenter une stratégie de sauvegarde automatique
- Ajouter une signature HMAC sur le webhook pour renforcer la sécurité


## 7. Contacts et ressources

- Dépôt de code : `https://github.com/itakademy/dfs-bloc4-evaluation-sujet`
- Documentation Laravel : `https://laravel.com/docs/12.x`
- Environnement de qualification : `http://eval-dfs-q-tpl-20262-09.it-students.fr`
- Environnement de production : `https://eval-dfs-p-tpl-20262-09.it-students.fr`
