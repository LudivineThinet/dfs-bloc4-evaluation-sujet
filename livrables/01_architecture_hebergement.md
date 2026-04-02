# Architecture cible et choix de l'hebergement

> Competence evaluee : `C29` — Selectionner une plateforme d'hebergement adaptee aux exigences techniques, economiques, qualitatives et reglementaires.

## 1. Analyse des besoins techniques

L'application OpsTrack Field Service repose sur une stack technique multi-composants identifiée sur l'environnement de qualification :

| Composant | Technologie | Version |
| --- | --- | --- |
| Langage backend | PHP | 8.4.18 |
| Framework backend | Laravel | 12 |
| Serveur web | Apache2 | 2.4 |
| Base de données relationnelle | MySQL | 8.0.45 |
| Base de données NoSQL | MongoDB | 8.0.19 |
| Cache et stockage temporaire | Redis | 7.0.15 |
| Microservice tableau de bord | Next.js | - |
| Système d'exploitation | Ubuntu | 24.04 LTS |

L'application expose une interface web principale, une API REST, un webhook (`hooks.php`) et un microservice Next.js (`dispatch-dashboard`). Elle consomme également une API publique tierce (Open Meteo).

Les besoins techniques principaux sont :
- un serveur web capable de servir PHP et de proxyfier vers Node.js
- deux bases de données (relationnelle et NoSQL) avec persistence des données
- un système de cache performant (Redis)
- une exposition publique sécurisée (HTTPS)
- une journalisation exploitable pour la maintenance

## 2. Architecture cible proposee

### 2.1 Diagramme de deploiement

<!-- Inserer ou decrire le diagramme d'architecture cible (composants, reseaux, services managés ou auto-heberges). -->

### 2.2 Description des composants

<!-- Pour chaque composant de l'architecture cible, decrire le role, le dimensionnement retenu et le service ou la technologie choisis. -->

| Composant | Service ou technologie | Dimensionnement | Justification |
| --- | --- | --- | --- |
|  |  |  |  |

## 3. Choix du fournisseur et des services

<!-- Justifier le choix du fournisseur d'hebergement et des services retenus. -->

### 3.1 Fournisseur retenu

### 3.2 Justification du choix

## 4. Estimation des couts

<!-- Fournir une estimation annuelle coherente du cout d'hebergement. -->

| Poste de depense | Cout mensuel estime | Cout annuel estime |
| --- | --- | --- |
|  |  |  |
| **Total** |  |  |

## 5. Elasticite et evolutivite

<!-- Expliquer la strategie retenue pour absorber la croissance de l'application : scalabilite horizontale, verticale, auto-scaling, etc. -->

## 6. Disponibilite et continuite de service

<!-- Decrire les mesures prevues pour garantir la disponibilite : redondance, basculement, SLA cible, tolerance de panne. -->

## 7. Securite et sauvegarde

<!-- Decrire les mesures de securite integrees a l'architecture cible : isolation reseau, chiffrement, gestion des secrets, strategie de sauvegarde. -->

## 8. Conformite et contraintes reglementaires

<!-- Expliciter les contraintes reglementaires prises en compte : protection des donnees, localisation, tracabilite, RGPD, etc. -->
