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
| Microservice tableau de bord | Next.js | 15.3.1 |
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
```
Internet
    │
    ▼
[Load Balancer / CDN]
    │
    ▼
[Serveur web — Apache2 + PHP]
    ├── Laravel 12 (API REST + interface web + webhook)
    └── Proxy → [Microservice Next.js 15 — dispatch-dashboard]
         │
    ┌────┴─────────────┐
    ▼                  ▼
[MySQL 8]          [MongoDB 8]
(données métier)   (journaux techniques)
    │
    ▼
[Redis 7]
(cache + stockage temporaire)
```

### 2.2 Description des composants

| Composant | Service ou technologie | Dimensionnement | Justification |
| --- | --- | --- | --- |
| Serveur web | Apache2 + PHP 8.4 | 2 vCPU, 4 Go RAM | Sert Laravel et proxyfie vers Next.js |
| Application backend | Laravel 12 | Inclus serveur web | Framework PHP principal, API REST et webhook |
| Microservice dashboard | Next.js 15 | 1 vCPU, 1 Go RAM | Tableau de bord secondaire indépendant |
| Base relationnelle | MySQL 8.0 | 2 vCPU, 4 Go RAM | Données métier transactionnelles |
| Base NoSQL | MongoDB 8.0 | 1 vCPU, 2 Go RAM | Journaux techniques et événements |
| Cache | Redis 7.0 | 1 vCPU, 1 Go RAM | Cache applicatif et stockage temporaire |
| API externe | Open-Meteo | Externe | Enrichissement météo des interventions |

## 3. Choix du fournisseur et des services

<!-- Justifier le choix du fournisseur d'hebergement et des services retenus. -->

## 3. Choix du fournisseur et des services

### 3.1 Fournisseur retenu

**Amazon Web Services (AWS)** — région `eu-west-3` (Paris)

### 3.2 Justification du choix

| Critère | Justification |
| --- | --- |
| Proximité géographique | Région Paris (eu-west-3) — données hébergées en France, conformité RGPD facilitée |
| Maturité | AWS est le leader mondial du cloud, documentation et support étendus |
| Services managés | RDS pour MySQL, ElastiCache pour Redis, DocumentDB pour MongoDB disponibles |
| Élasticité | Auto-scaling natif, possibilité de monter en charge rapidement |
| Sécurité | VPC, groupes de sécurité, IAM, chiffrement natif |
| Coût | Modèle pay-as-you-go adapté à une application en croissance |

Le choix d'AWS est cohérent avec l'environnement fourni pour l'épreuve, qui repose déjà sur des instances EC2 en région eu-west-3.

## 4. Estimation des couts

| Poste de depense | Cout mensuel estime | Cout annuel estime |
| --- | --- | --- |
| EC2 t3.medium (serveur web + Laravel) | ~30 € | ~360 € |
| RDS MySQL t3.micro | ~15 € | ~180 € |
| ElastiCache Redis t3.micro | ~15 € | ~180 € |
| DocumentDB t3.medium (MongoDB) | ~45 € | ~540 € |
| EC2 t3.small (microservice Next.js) | ~15 € | ~180 € |
| Route 53 (DNS) | ~1 € | ~12 € |
| Certificate Manager (TLS) | 0 € | 0 € |
| S3 (sauvegardes) | ~5 € | ~60 € |
| **Total estimé** | **~126 €** | **~1 512 €** |

> Estimations indicatives basées sur les tarifs publics AWS eu-west-3. Le calculateur officiel AWS (calculator.aws) permet d'affiner ces chiffres selon l'usage réel.

## 5. Elasticite et evolutivite

L'architecture cible prévoit deux niveaux d'évolution :

**Court terme — scalabilité verticale**
Augmentation des ressources de l'instance EC2 existante (passage de t3.medium à t3.large par exemple) sans modifier l'architecture. Solution simple et rapide à mettre en œuvre.

**Moyen terme — scalabilité horizontale**
Mise en place d'un load balancer (AWS ALB) devant plusieurs instances EC2 identiques. Le cache Redis centralisé (ElastiCache) et la base de données managée (RDS) permettent à plusieurs instances de partager le même état.

**Composants déjà prêts pour la scalabilité**
- Redis : externalisable sur ElastiCache sans modification du code
- MySQL : externalisable sur RDS sans modification du code
- Laravel : stateless par conception, compatible multi-instances
- Next.js : microservice indépendant, scalable séparément

## 6. Disponibilite et continuite de service

<!-- Decrire les mesures prevues pour garantir la disponibilite : redondance, basculement, SLA cible, tolerance de panne. -->

## 7. Securite et sauvegarde

<!-- Decrire les mesures de securite integrees a l'architecture cible : isolation reseau, chiffrement, gestion des secrets, strategie de sauvegarde. -->

## 8. Conformite et contraintes reglementaires

<!-- Expliciter les contraintes reglementaires prises en compte : protection des donnees, localisation, tracabilite, RGPD, etc. -->
