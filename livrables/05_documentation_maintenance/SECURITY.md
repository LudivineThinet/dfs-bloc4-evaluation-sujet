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
| Mesure corrective appliquee | Faille identifiée, correction recommandée : générer un token aléatoire via `openssl rand -base64 32` et le renseigner dans `.env` |
| Statut | `Identifie, non corrige` |
| Preuve de correction | N/A |
