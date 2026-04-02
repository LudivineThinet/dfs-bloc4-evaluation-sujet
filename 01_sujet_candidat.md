# Sujet candidat

## Certification Developpeur Full Stack - RNCP38606

### Bloc RNCP38606BC04

`Deployer et assurer le maintien en production d'une application`

---

| Champ | Valeur |
| --- | --- |
| Session | `2 avril 2026` |
| Modalite | Mise en situation professionnelle simulee individuelle de maintenance d'une application |
| Duree totale | `5 heures` |
| Organisation | `Partie 1 : 9h - 12h` - `Partie 2 : 13h - 15h` |
| Format de diffusion | Document candidat, sans donnees confidentielles |
| Donnees sensibles | Remises separement dans la fiche confidentielle individuelle |

## 1. Objet de l'epreuve

Le candidat intervient sur une application existante presentant des bugs techniques, des failles de securite, des composants a fiabiliser et des besoins de mise en production.

L'epreuve est construite pour evaluer les competences du bloc 4 dans une logique de maintenance et d'exploitation reelles, sur un environnement technique fourni par le centre.

## 2. Competences evaluees

| Code | Competence evaluee | Preuves principales attendues |
| --- | --- | --- |
| `C27` | Produire la documentation technique d'une application et alimenter une base de connaissances pour la maintenance | Documentation generee, changelog, journal de securite, base de connaissances |
| `C28` | Administrer l'enregistrement et la configuration de noms de domaines et de certificats de securite | DNS, domaine, HTTPS, certificat fonctionnel |
| `C29` | Selectionner une plateforme d'hebergement adaptee aux exigences techniques, economiques, qualitatives et reglementaires | Architecture cible, cout, elasticite, contraintes de conformite |
| `C30` | Administrer des services d'hebergement en appliquant les bonnes pratiques de securite pour maintenir la continuite de service | Production fonctionnelle, securisation, configuration reproductible |
| `C31` | Mettre en oeuvre un systeme de deploiement automatise respectant les bonnes pratiques DevOps | Pipeline ou script de deploiement, controles, smoke test |
| `C32` | Mettre en oeuvre un systeme de supervision pour detecter, diagnostiquer et corriger bugs, incidents et failles | Journalisation, alertes, sauvegarde, diagnostic, correction |

## 3. Consignes generales

- L'acces a Internet et l'usage d'outils d'assistance, y compris d'IA generative, sont autorises sauf instruction contraire du centre.
- Les telephones portables et terminaux personnels non autorises par le centre sont interdits pendant l'epreuve.
- Les messageries instantanees et toute communication entre candidats sont interdites pendant l'epreuve.
- Les informations d'acces, de session et les secrets techniques figurent dans la fiche confidentielle remise individuellement.
- Le candidat peut proposer des solutions techniques equivalentes a celles citees en exemple, a condition qu'elles soient justifiees, fonctionnelles et documentees.
- **Le candidat doit forker le present depot pour y rendre ses livrables. Chaque livrable doit faire l'objet d'une Pull Request distincte vers le depot d'origine** (cf. section 7).

## 4. Environnement fourni

| Ressource | Description minimale attendue |
| --- | --- |
| Code source | Depot contenant l'application support |
| Qualification / preproduction | Environnement de travail et de verification |
| Production | Environnement cible a administrer et mettre en service |
| Domaine public | Nom de domaine affecte a la production |
| Acces techniques | Acces SSH et, si necessaire, acces applicatifs utiles a l'epreuve |

L'application support comporte au minimum :

- un front-end ;
- un back-end d'API ;
- une base de donnees relationnelle ;
- une base NoSQL ;
- au moins un micro-service serverless ;
- des integrations externes de type API et webhook.

## 4 bis. Vue d'architecture de l'application support

L'application support `OpsTrack Field Service` repose sur les composants suivants :

| Composant | Role principal | Point d'attention pour l'epreuve |
| --- | --- | --- |
| `Laravel 12` | Application coeur metier, interface web principale, API REST et traitement du webhook `hooks.php` | Porte la logique metier, les integrations et une partie des anomalies a diagnostiquer |
| `MySQL` | Donnees transactionnelles de l'application | Contient les entites metier principales : utilisateurs, tickets, interventions, commentaires |
| `MongoDB` | Journaux techniques et evenements applicatifs | Utile pour le diagnostic, la tracabilite et l'analyse d'incidents |
| `Redis` | Cache applicatif et mecanismes de performance | Peut influencer le comportement observe sur les indicateurs et la coherence des donnees affichees |
| `Next.js` | Microservice `dispatch-dashboard` dedie a l'affichage d'un tableau de bord secondaire | Consomme l'API Laravel et peut constituer un point d'entree de diagnostic inter-services |
| API publique tierce | Enrichissement externe de certaines donnees de l'application | Sa disponibilite ou son integration peuvent influencer certains traitements |

### Flux applicatifs principaux

- le front principal et l'API metier sont servis par l'application Laravel ;
- Laravel lit et ecrit les donnees metier dans `MySQL` ;
- Laravel journalise certains evenements et traces dans `MongoDB` ;
- `Redis` est utilise pour accelerer certains traitements et stockages temporaires ;
- le microservice `Next.js` consomme l'API Laravel pour afficher un tableau de bord dedie ;
- un webhook appelle `hooks.php` a la racine du domaine pour injecter des evenements externes ;
- l'application peut consommer une API publique pour enrichir certains traitements.

### Perimetre d'analyse recommande au candidat

- verifier d'abord le coeur Laravel et les dependances de donnees (`MySQL`, `MongoDB`, `Redis`) ;
- controler ensuite les integrations entrantes et sortantes : API, webhook et API publique ;
- considerer le microservice `Next.js` comme un composant distinct, dependant de l'API Laravel ;
- distinguer ce qui releve d'un probleme de code, de configuration, de donnees ou d'integration entre services.

## 5. Travail demande

Les activites sont organisees par preuves de competence. Une meme action peut contribuer a plusieurs competences si elle est correctement tracee.

### 5.1 Synthese des attendus par partie

| Partie | Activite | Competences | Livrable principal |
| --- | --- | --- | --- |
| Partie 1 | Architecture cible et choix de l'hebergement | `C29` | `01_architecture_hebergement.md` |
| Partie 1 | Exploitation securisee de la production, domaine et TLS | `C28`, `C30` | `02_exploitation_securisee.md` |
| Partie 1 | Deploiement automatise qualification -> production | `C31` | `03_deploiement_ci_cd.md` |
| Partie 2 | Supervision, journalisation, sauvegarde et maintenance corrective | `C32` | `04_supervision_maintien.md` |
| Partie 2 | Documentation technique et transfert de connaissances | `C27` | `05_documentation_maintenance/` |

### 5.2 Partie 1 - Preparation du deploiement et mise en service

#### A. Architecture cible et choix de l'hebergement - `C29`

Depuis l'environnement de qualification et a partir de l'application fournie, le candidat doit :

- analyser les besoins techniques de l'application ;
- proposer une architecture cible d'hebergement pour une application en croissance ;
- justifier le choix d'un fournisseur et des services retenus ;
- estimer un cout annuel coherent ;
- expliciter les exigences qualitatives et reglementaires prises en compte.

L'architecture cible doit traiter au minimum :

- l'exposition publique des services ;
- l'isolation entre composants ;
- l'elasticite ou, a defaut, la capacite d'evolution ;
- la gestion des donnees relationnelles et NoSQL ;
- la securite, la sauvegarde et la supervision ;
- les contraintes de conformite utiles au contexte, notamment protection des donnees et tracabilite.

Il n'est pas demande d'implementer integralement cette architecture cible pendant l'epreuve. Elle doit etre formalisee et argumentee.

#### B. Exploitation securisee de l'environnement de production - `C28` et `C30`

En tenant compte de la contrainte economique suivante, `dans un premier temps, la production repose sur une seule machine`, le candidat doit :

- mettre en service l'application sur l'environnement de production ;
- administrer les services necessaires a son fonctionnement ;
- appliquer des mesures de securisation adaptees ;
- documenter une configuration reproductible ;
- administrer le domaine attribue et mettre en service le HTTPS.

Les attendus minimaux portent sur :

- le service web et l'execution de l'application ;
- les droits d'acces et l'isolation minimale des services ;
- la configuration reseau et systeme utile au fonctionnement ;
- la gestion des secrets et des parametres sensibles ;
- l'exposition controlee des services ;
- la resolution DNS et la mise en service du certificat.

#### C. Deploiement automatise entre qualification et production - `C31`

Le candidat doit mettre en oeuvre un systeme de deploiement automatise permettant de promouvoir l'application depuis l'environnement de qualification vers l'environnement de production.

Le dispositif retenu doit permettre :

- un declenchement explicite et reproductible ;
- la verification d'un niveau minimal de qualite avant deploiement ;
- une mise a jour effective de la production ;
- une verification post-deploiement de type `smoke test`.

### 5.3 Partie 2 - Maintien en production, supervision et documentation

#### D. Supervision, journalisation, sauvegarde et maintenance corrective - `C32`

Le candidat doit mettre en place ou configurer les moyens permettant de detecter, diagnostiquer et traiter un incident applicatif ou de securite.

Les attendus minimaux sont les suivants :

- une journalisation exploitable des services utiles ;
- des outils ou configurations d'audit adaptes au contexte ;
- des sondes et alertes pertinentes pour l'etat des services et la securite ;
- une strategie de sauvegarde et de restauration, ou a defaut de tolerance de panne, adaptee au contexte ;
- l'identification d'au moins un bug technique et la mise en oeuvre d'un correctif ;
- l'identification d'au moins une faille de securite et la mise en oeuvre d'une mesure corrective ;
- si le scenario fourni l'exige, l'identification de la source des appels suspects ou vulnerables.

Si une restauration complete n'est pas raisonnablement realisable sur la production pendant l'epreuve, le candidat peut valider sa procedure sur l'environnement de qualification, a condition de l'indiquer explicitement.

#### E. Documentation technique et transfert de connaissances - `C27`

Le candidat doit produire un ensemble documentaire exploitable par un pair charge de reprendre la maintenance de l'application.

Cet ensemble doit comporter :

- une documentation technique generee a partir du code source, hors dependances tierces ;
- une documentation d'API ou, si elle n'est pas pertinente, une note de justification ;
- un `CHANGELOG` a jour ;
- un document de securite ou journal des corrections de securite a jour ;
- une base de connaissances ou note de passation expliquant le fonctionnement, les points d'attention, les procedures de deploiement, de supervision et de reprise.

## 6. Livrables attendus

| Livrable | Contenu attendu |
| --- | --- |
| `01_architecture_hebergement.md` | Architecture cible, diagramme de deploiement, justification, chiffrage annuel, elasticite, disponibilite et contraintes reglementaires |
| `02_exploitation_securisee.md` | Configuration production, DNS utiles, HTTPS, hardening, configuration reproductible |
| `03_deploiement_ci_cd.md` | Chaine de deploiement, outillage retenu, controles prealables, declenchement, verification post-deploiement, conduite a tenir en cas d'echec |
| `04_supervision_maintien.md` | Journalisation, audit, supervision, alertes, sauvegarde, diagnostic du bug, diagnostic de la faille, mesures correctives |
| `05_documentation_maintenance/` | Documentation technique generee, documentation d'API ou note de non-applicabilite, `CHANGELOG.md`, `SECURITY.md` ou `journal_securite.md`, `base_connaissances.md` ou `note_passation.md` |

## 7. Modalite de rendu

Les livrables sont rendus via GitHub selon la procedure suivante :

1. **Forker** ce depot (`dfs-bloc4-evaluation-sujet`) sur votre compte GitHub personnel.
2. Completer les templates de livrables dans le dossier `livrables/` de votre fork.
3. Pour chaque livrable, creer une **Pull Request** distincte depuis votre fork vers le depot d'origine.
4. Nommer chaque Pull Request de maniere explicite, par exemple : `[Candidat 04] Livrable 01 — Architecture et hebergement`.
5. Les Pull Requests doivent etre ouvertes **avant la fin de l'epreuve**.

> Les Pull Requests constituent la trace formelle de rendu. Tout livrable non soumis par Pull Request avant la fin de l'epreuve sera considere comme non rendu.
