# Documentation d'API

## 1. Vue d'ensemble de l'API

| Champ | Valeur |
| --- | --- |
| URL de base | `https://eval-dfs-p-tpl-20262-09.it-students.fr/api` |
| Format | `JSON` |
| Authentification | Token via header `X-Api-Token` (valeur définie dans `OPSTRACK_API_TOKEN`) |

## 2. Endpoints disponibles

| Methode | Endpoint | Description | Authentification requise |
| --- | --- | --- | --- |
| GET | `/api/health` | Vérification de l'état de l'API | Non |
| GET | `/api/v1/tickets` | Liste des tickets | Oui |
| POST | `/api/v1/tickets` | Créer un ticket | Oui |
| GET | `/api/v1/tickets/{id}` | Détail d'un ticket | Oui |
| PUT | `/api/v1/tickets/{id}` | Modifier un ticket | Oui |
| GET | `/api/v1/technicians` | Liste des techniciens | Oui |
| GET | `/api/v1/external/weather` | Données météo du site | Oui |

## 3. Exemples de requetes et reponses

**GET /api/health** (sans authentification)
```bash
curl https://eval-dfs-p-tpl-20262-09.it-students.fr/api/health
```
Réponse :
```json
{
  "status": "ok",
  "service": "OpsTrack",
  "timestamp": "2026-04-02T10:00:00+02:00"
}
```

**GET /api/v1/tickets** (avec authentification)
```bash
curl -H "X-Api-Token: <token>" https://eval-dfs-p-tpl-20262-09.it-students.fr/api/v1/tickets
```

## 4. Codes d'erreur

| Code | Signification |
| --- | --- |
| 200 | Succès |
| 401 | Token manquant ou invalide |
| 404 | Ressource introuvable |
| 405 | Méthode HTTP non autorisée |
| 500 | Erreur serveur interne |
