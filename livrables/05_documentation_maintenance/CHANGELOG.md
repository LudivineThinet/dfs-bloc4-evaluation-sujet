# Changelog
Toutes les modifications notables apportees pendant l'epreuve sont documentees dans ce fichier.
Le format s'inspire de [Keep a Changelog](https://keepachangelog.com/).

## [Session du 2 avril 2026]

### Ajoute
- Déploiement de l'application OpsTrack sur l'environnement de production (`eval-dfs-p-tpl-20262-09.it-students.fr`)
- Configuration du VirtualHost Apache en production avec proxy vers le microservice Next.js
- Certificat TLS Let's Encrypt via Certbot — HTTPS activé sur la production
- Création de la base de données MySQL `opstrack` en production et exécution des migrations
- Script de déploiement automatisé `deploy.sh` (qualification → production via SSH)

### Modifie
- Fichier `.env` qualification : `CACHE_STORE` passé de `database` à `redis`
- Fichier `.env` production : `APP_ENV=production`, `APP_DEBUG=false`, `APP_URL` mis à jour
- Configuration Apache qualification : `Options -Indexes` activé

### Corrige
- Bug : les compteurs du tableau de bord affichaient des valeurs périmées en raison d'une mauvaise configuration du cache (Redis non utilisé)
- Bug SQL : recherche de tickets avec `orWhereRaw` mal groupée corrigée dans `TicketController`, les filtres de priorité s'appliquent désormais correctement
- Bug WebhookController : déduplication via firstOrCreate() + statut mis à jour depuis le payload
- Bug Next.js : payload.items remplacé par payload.data

### Securite
- Blocage de l'IP malveillante `206.189.35.70` via `ufw` (tentatives de path traversal et injection)
- Désactivation de `Options Indexes` dans Apache (empêche la navigation dans les dossiers)
- Identification du token `OPSTRACK_API_TOKEN=change-me` comme faille à corriger
- Remplacement de `orWhereRaw` par `orWhere` dans `TicketController` , suppression du risque d'injection SQL via le paramètre de recherche
- .env.example : identifiants réels remplacés par des placeholders génériques
- Token `OPSTRACK_API_TOKEN` remplacé par une valeur sécurisée générée via `openssl rand -base64 32`
