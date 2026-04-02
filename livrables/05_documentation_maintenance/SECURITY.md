# Journal de securite
Ce document recense les failles de securite identifiees pendant l'epreuve, leur evaluation et les mesures correctives appliquees.

## Faille 1

| Champ | Description |
| --- | --- |
| Date de detection | 2 avril 2026 |
| Composant concerne | Apache2 — serveur de qualification |
| Description de la faille | Tentatives de path traversal via `/cgi-bin/../../../../bin/sh` encodé en URL, et injection via le paramètre `lang` de `index.php` ayant retourné HTTP 200 |
| Severite estimee | `Haute` |
| Impact potentiel | Exécution de code arbitraire sur le serveur, prise de contrôle possible |
| Mesure corrective appliquee | Blocage de l'IP source `206.189.35.70` via `sudo ufw deny from 206.189.35.70` |
| Statut | `Mitigation en place` |
| Preuve de correction | `sudo ufw status` confirme la règle active |

## Faille 2

| Champ | Description |
| --- | --- |
| Date de detection | 2 avril 2026 |
| Composant concerne | Apache2 — configuration du VirtualHost |
| Description de la faille | `Options Indexes` activé — permettait la navigation dans les dossiers du serveur web |
| Severite estimee | `Moyenne` |
| Impact potentiel | Exposition de la structure de fichiers de l'application à tout visiteur |
| Mesure corrective appliquee | Remplacement de `Options Indexes FollowSymLinks` par `Options -Indexes +FollowSymLinks` dans le VirtualHost, suivi d'un reload Apache |
| Statut | `Corrige` |
| Preuve de correction | `sudo apache2ctl configtest` retourne `Syntax OK`, Apache rechargé avec succès |

## Faille 3

| Champ | Description |
| --- | --- |
| Date de detection | 2 avril 2026 |
| Composant concerne | Fichier `.env` — configuration applicative |
| Description de la faille | `OPSTRACK_API_TOKEN=change-me` — token d'authentification API jamais remplacé par une valeur sécurisée |
| Severite estimee | `Haute` |
| Impact potentiel | Accès non autorisé à l'ensemble des endpoints API protégés |
| Mesure corrective appliquee | Token remplacé par une valeur sécurisée générée via `openssl rand -base64 32` et renseignée dans `.env` |
| Statut | `Corrige` |
| Preuve de correction | `php artisan config:clear` exécuté avec succès sur le serveur de qualification |

## Faille 4

| Champ | Description |
| --- | --- |
| Date de detection | 2 avril 2026 |
| Composant concerne | `TicketController` — endpoint `GET /api/v1/tickets` |
| Description de la faille | `orWhereRaw` avec interpolation directe de la variable `$search` — risque d'injection SQL sur le paramètre de recherche |
| Severite estimee | `Haute` |
| Impact potentiel | Manipulation de la requête SQL, extraction de données non autorisées |
| Mesure corrective appliquee | Remplacement de `orWhereRaw` par `orWhere` — Laravel gère l'échappement automatiquement |
| Statut | `Corrige` |
| Preuve de correction | `php -l TicketController.php` → `No syntax errors detected` |

## Faille 5

| Champ | Description |
| --- | --- |
| Date de detection | 2 avril 2026 |
| Composant concerne | Fichier `.env.example` — versionné dans le dépôt Git |
| Description de la faille | Le fichier `.env.example` contient les identifiants réels de démonstration : `DB_PASSWORD=0000`, `OPSTRACK_API_TOKEN=change-me`, `WEBHOOK_BASIC_USER=user`, `WEBHOOK_BASIC_PASSWORD=password` |
| Severite estimee | `Haute` |
| Impact potentiel | Exposition publique des identifiants de l'environnement de qualification à quiconque ayant accès au dépôt |
| Mesure corrective appliquee | Remplacement de toutes les valeurs sensibles par des placeholders génériques (`your_db_password`, `your_secure_token_here`, etc.) |
| Statut | `Corrige` |
| Preuve de correction | Fichier `.env.example` modifié sur le serveur de qualification |

## Faille 6

| Champ | Description |
| --- | --- |
| Date de detection | 2 avril 2026 |
| Composant concerne | `WebhookController` — endpoint `hooks.php` |
| Description de la faille | Le webhook crée une nouvelle intervention à chaque appel sans vérifier si `external_event_id` existe déjà (pas de déduplication). De plus, le statut du ticket est systématiquement forcé à `scheduled` quelle que soit la valeur transmise par le webhook |
| Severite estimee | `Haute` |
| Impact potentiel | Création de doublons d'interventions en base de données, écart entre l'état réel du ticket et ce qu'affiche l'application |
| Mesure corrective appliquee | Remplacement de `create()` par `firstOrCreate()` sur `external_event_id`, et remplacement de `status = scheduled` par `status = $payload['status']` |
| Statut | `Corrige` |
| Preuve de correction | `php -l WebhookController.php` → `No syntax errors detected` |

## Faille 7

| Champ | Description |
| --- | --- |
| Date de detection | 2 avril 2026 |
| Composant concerne | Microservice Next.js — `dispatch-dashboard/lib/api.ts` |
| Description de la faille | Le microservice lit `payload.items` au lieu de `payload.data` — la réponse Laravel retourne toujours `data`, donc le tableau de bord affiche toujours une liste vide malgré une API fonctionnelle |
| Severite estimee | `Moyenne` |
| Impact potentiel | Tableau de bord secondaire toujours vide, supervision incomplète |
| Mesure corrective appliquee | Remplacement de `payload.items` par `payload.data` dans `lib/api.ts` |
| Statut | `Corrige` |
| Preuve de correction | Vérification via `curl` que l'API retourne bien `{"data":[...]}` |

## Faille 8

| Champ | Description |
| --- | --- |
| Date de detection | 2 avril 2026 |
| Composant concerne | Microservice Next.js — `dispatch-dashboard/.env.local` |
| Description de la faille | `LARAVEL_API_TOKEN=change-me` — le microservice utilisait un token invalide pour appeler l'API Laravel |
| Severite estimee | `Haute` |
| Impact potentiel | Le microservice ne peut pas s'authentifier correctement auprès de l'API |
| Mesure corrective appliquee | Token mis à jour dans `.env.local` avec la valeur sécurisée générée via `openssl rand -base64 32` |
| Statut | `Corrige` |
| Preuve de correction | Fichier `.env.local` mis à jour sur le serveur de qualification |

## Faille 9

| Champ | Description |
| --- | --- |
| Date de detection | 2 avril 2026 |
| Composant concerne | `hooks.php` — endpoint webhook |
| Description de la faille | Le webhook vérifie uniquement l'authentification HTTP Basic, sans signature complémentaire (HMAC) ni restriction sur l'origine des appels |
| Severite estimee | `Moyenne` |
| Impact potentiel | Un attaquant connaissant les identifiants Basic peut envoyer de faux événements webhook |
| Mesure corrective appliquee | Non corrigé pendant l'épreuve — correction recommandée : ajouter une vérification de signature HMAC sur le payload entrant |
| Statut | `Identifie, non corrige à temps` |
| Preuve de correction | N/A |
