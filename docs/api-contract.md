# API Contract — IncidentDesk

## 1. Généralités

Toutes les routes sont préfixées par :

```text
/api/v1
```

Format des réponses :

```text
application/json
```

Authentification :

```text
Authorization: Bearer <jwt_token>
```

## 2. Codes HTTP

| Code | Usage |
|---|---|
| 200 | Requête réussie |
| 201 | Ressource créée |
| 204 | Suppression réussie |
| 400 | Requête mal formée |
| 401 | Non authentifié |
| 403 | Permissions insuffisantes |
| 404 | Ressource introuvable |
| 409 | Conflit d’état |
| 422 | Erreur de validation |
| 500 | Erreur serveur |

## 3. Format d’erreur

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "La requête contient des données invalides.",
    "details": [
      {
        "field": "title",
        "message": "Le titre est obligatoire."
      }
    ]
  }
}
```

## 4. Authentification

### POST /api/v1/auth/register

Crée un compte utilisateur.

#### Requête

```json
{
  "email": "agent@example.com",
  "password": "StrongPassword123!",
  "full_name": "Alice Martin",
  "role": "agent"
}
```

#### Réponse 201

```json
{
  "id": "uuid",
  "email": "agent@example.com",
  "full_name": "Alice Martin",
  "role": "agent",
  "created_at": "2026-11-17T12:00:00Z"
}
```

#### Erreurs

- `409` : email déjà utilisé.
- `422` : données invalides.

### POST /api/v1/auth/login

Authentifie un utilisateur.

#### Requête

```json
{
  "email": "agent@example.com",
  "password": "StrongPassword123!"
}
```

#### Réponse 200

```json
{
  "access_token": "jwt_token",
  "token_type": "bearer",
  "expires_in": 3600
}
```

#### Erreurs

- `401` : identifiants invalides.

### GET /api/v1/auth/me

Retourne les informations de l’utilisateur connecté.

#### Réponse 200

```json
{
  "id": "uuid",
  "email": "agent@example.com",
  "full_name": "Alice Martin",
  "role": "agent"
}
```

## 5. Incidents

### GET /api/v1/incidents

Liste les incidents.

#### Query parameters

| Paramètre | Type | Description |
|---|---|---|
| status | string | Filtre par statut |
| priority | string | Filtre par priorité |
| assignee_id | UUID | Filtre par responsable |
| search | string | Recherche dans le titre et la description |
| page | integer | Numéro de page |
| size | integer | Nombre d’éléments par page |

#### Réponse 200

```json
{
  "items": [
    {
      "id": "uuid",
      "title": "API de paiement indisponible",
      "status": "open",
      "priority": "critical",
      "assignee_id": null,
      "created_at": "2026-11-17T12:00:00Z"
    }
  ],
  "page": 1,
  "size": 20,
  "total": 1
}
```

### POST /api/v1/incidents

Crée un incident.

#### Requête

```json
{
  "title": "API de paiement indisponible",
  "description": "Les paiements échouent depuis 10 minutes.",
  "priority": "critical",
  "due_at": "2026-11-17T15:00:00Z"
}
```

#### Réponse 201

```json
{
  "id": "uuid",
  "title": "API de paiement indisponible",
  "description": "Les paiements échouent depuis 10 minutes.",
  "status": "open",
  "priority": "critical",
  "reporter_id": "uuid",
  "assignee_id": null,
  "due_at": "2026-11-17T15:00:00Z",
  "created_at": "2026-11-17T12:00:00Z"
}
```

#### Erreurs

- `401` : non authentifié.
- `403` : rôle non autorisé.
- `422` : validation échouée.

### GET /api/v1/incidents/{incident_id}

Retourne le détail d’un incident.

#### Réponse 200

```json
{
  "id": "uuid",
  "title": "API de paiement indisponible",
  "description": "Les paiements échouent depuis 10 minutes.",
  "status": "open",
  "priority": "critical",
  "reporter_id": "uuid",
  "assignee_id": null,
  "due_at": "2026-11-17T15:00:00Z",
  "escalated_at": null,
  "created_at": "2026-11-17T12:00:00Z",
  "updated_at": "2026-11-17T12:00:00Z"
}
```

#### Erreurs

- `404` : incident introuvable.

### PATCH /api/v1/incidents/{incident_id}

Met à jour un incident.

#### Requête

```json
{
  "title": "API de paiement indisponible",
  "priority": "high"
}
```

#### Réponse 200

```json
{
  "id": "uuid",
  "title": "API de paiement indisponible",
  "status": "open",
  "priority": "high"
}
```

#### Erreurs

- `403` : permissions insuffisantes.
- `404` : incident introuvable.
- `422` : validation échouée.

### DELETE /api/v1/incidents/{incident_id}

Supprime un incident.

#### Autorisation

Réservé aux administrateurs.

#### Réponse 204

```text
No Content
```

### POST /api/v1/incidents/{incident_id}/assign

Assigne un responsable.

#### Requête

```json
{
  "assignee_id": "uuid"
}
```

#### Réponse 200

```json
{
  "id": "uuid",
  "assignee_id": "uuid",
  "status": "open"
}
```

### POST /api/v1/incidents/{incident_id}/status

Change le statut d’un incident.

#### Requête

```json
{
  "status": "in_progress"
}
```

#### Réponse 200

```json
{
  "id": "uuid",
  "status": "in_progress"
}
```

#### Erreurs

- `409` : transition de statut interdite.

### POST /api/v1/incidents/{incident_id}/comments

Ajoute un commentaire.

#### Requête

```json
{
  "content": "Le problème a été identifié, investigation en cours."
}
```

#### Réponse 201

```json
{
  "id": "uuid",
  "incident_id": "uuid",
  "author_id": "uuid",
  "content": "Le problème a été identifié, investigation en cours.",
  "created_at": "2026-11-17T12:05:00Z"
}
```

### GET /api/v1/incidents/{incident_id}/events

Retourne l’historique d’un incident.

#### Réponse 200

```json
{
  "items": [
    {
      "id": "uuid",
      "event_type": "incident_created",
      "actor_id": "uuid",
      "from_value": null,
      "to_value": "open",
      "created_at": "2026-11-17T12:00:00Z"
    }
  ]
}
```

## 6. Règles d’escalade

### GET /api/v1/escalation-rules

Liste les règles d’escalade.

#### Réponse 200

```json
{
  "items": [
    {
      "id": "uuid",
      "priority": "critical",
      "max_wait_minutes": 15,
      "escalate_to_role": "on_call",
      "is_active": true
    }
  ]
}
```

### POST /api/v1/escalation-rules

Crée une règle d’escalade.

#### Autorisation

Réservé aux administrateurs.

#### Requête

```json
{
  "priority": "critical",
  "max_wait_minutes": 15,
  "escalate_to_role": "on_call",
  "is_active": true
}
```

#### Réponse 201

```json
{
  "id": "uuid",
  "priority": "critical",
  "max_wait_minutes": 15,
  "escalate_to_role": "on_call",
  "is_active": true
}
```

### PATCH /api/v1/escalation-rules/{rule_id}

Met à jour une règle d’escalade.

#### Réponse 200

```json
{
  "id": "uuid",
  "priority": "critical",
  "max_wait_minutes": 10,
  "escalate_to_role": "on_call",
  "is_active": true
}
```

### DELETE /api/v1/escalation-rules/{rule_id}

Supprime une règle d’escalade.

#### Réponse 204

```text
No Content
```