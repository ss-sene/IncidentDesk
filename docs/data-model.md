# Data Model — IncidentDesk

## 1. Vue d’ensemble

Le modèle de données d’IncidentDesk repose sur cinq entités principales :

- `users`
- `incidents`
- `incident_comments`
- `incident_events`
- `escalation_rules`

## 2. Relations

```text
users 1 --- N incidents (reporter_id)
users 1 --- N incidents (assignee_id)
users 1 --- N incident_comments (author_id)
users 1 --- N incident_events (actor_id)

incidents 1 --- N incident_comments
incidents 1 --- N incident_events

escalation_rules est une table de configuration indépendante
```

## 3. Table : users

| Colonne | Type | Contrainte | Description |
|---|---|---|---|
| id | UUID | Primary key | Identifiant unique |
| email | VARCHAR(255) | Unique, not null | Email de l’utilisateur |
| hashed_password | VARCHAR(255) | Not null | Mot de passe haché |
| full_name | VARCHAR(255) | Not null | Nom complet |
| role | ENUM | Not null | `admin`, `agent`, `viewer` |
| created_at | TIMESTAMPTZ | Not null | Date de création |
| updated_at | TIMESTAMPTZ | Not null | Date de mise à jour |

### Index

```sql
CREATE UNIQUE INDEX idx_users_email ON users(email);
```

## 4. Table : incidents

| Colonne | Type | Contrainte | Description |
|---|---|---|---|
| id | UUID | Primary key | Identifiant unique |
| title | VARCHAR(255) | Not null | Titre de l’incident |
| description | TEXT | Not null | Description détaillée |
| status | ENUM | Not null | `open`, `in_progress`, `resolved`, `closed` |
| priority | ENUM | Not null | `low`, `medium`, `high`, `critical` |
| reporter_id | UUID | Foreign key vers users | Utilisateur ayant créé l’incident |
| assignee_id | UUID | Foreign key vers users, nullable | Responsable actuel |
| due_at | TIMESTAMPTZ | Nullable | Échéance de traitement |
| escalated_at | TIMESTAMPTZ | Nullable | Date de la dernière escalade |
| created_at | TIMESTAMPTZ | Not null | Date de création |
| updated_at | TIMESTAMPTZ | Not null | Date de mise à jour |

### Index

```sql
CREATE INDEX idx_incidents_status ON incidents(status);
CREATE INDEX idx_incidents_priority ON incidents(priority);
CREATE INDEX idx_incidents_assignee_id ON incidents(assignee_id);
CREATE INDEX idx_incidents_created_at ON incidents(created_at);
```

## 5. Table : incident_comments

| Colonne | Type | Contrainte | Description |
|---|---|---|---|
| id | UUID | Primary key | Identifiant unique |
| incident_id | UUID | Foreign key vers incidents, not null | Incident concerné |
| author_id | UUID | Foreign key vers users, not null | Auteur du commentaire |
| content | TEXT | Not null | Contenu du commentaire |
| created_at | TIMESTAMPTZ | Not null | Date de création |

### Index

```sql
CREATE INDEX idx_incident_comments_incident_id ON incident_comments(incident_id);
```

## 6. Table : incident_events

| Colonne | Type | Contrainte | Description |
|---|---|---|---|
| id | UUID | Primary key | Identifiant unique |
| incident_id | UUID | Foreign key vers incidents, not null | Incident concerné |
| actor_id | UUID | Foreign key vers users, nullable | Utilisateur ou système à l’origine de l’événement |
| event_type | VARCHAR(50) | Not null | Type d’événement |
| from_value | VARCHAR(255) | Nullable | Valeur précédente |
| to_value | VARCHAR(255) | Nullable | Nouvelle valeur |
| created_at | TIMESTAMPTZ | Not null | Date de l’événement |

### Types d’événements

```text
incident_created
status_changed
priority_changed
assignee_changed
comment_added
escalated
incident_resolved
incident_closed
```

### Index

```sql
CREATE INDEX idx_incident_events_incident_id ON incident_events(incident_id);
CREATE INDEX idx_incident_events_event_type ON incident_events(event_type);
```

## 7. Table : escalation_rules

| Colonne | Type | Contrainte | Description |
|---|---|---|---|
| id | UUID | Primary key | Identifiant unique |
| priority | ENUM | Not null | Priorité concernée |
| max_wait_minutes | INTEGER | Not null | Délai avant escalade |
| escalate_to_role | VARCHAR(50) | Not null | Rôle de destination |
| is_active | BOOLEAN | Not null, default true | Règle active ou non |
| created_at | TIMESTAMPTZ | Not null | Date de création |
| updated_at | TIMESTAMPTZ | Not null | Date de mise à jour |

### Règle par défaut

```json
{
  "priority": "critical",
  "max_wait_minutes": 15,
  "escalate_to_role": "on_call",
  "is_active": true
}
```

## 8. Règles de contrainte

- Un incident doit toujours avoir un `reporter_id`.
- Un incident peut ne pas avoir de `assignee_id` au moment de sa création.
- Un incident ne peut pas être escaladé deux fois sans nouvelle condition remplie.
- Un commentaire doit toujours être lié à un incident existant.
- Un événement doit toujours être lié à un incident existant.
- Les suppressions d’utilisateurs ne doivent pas supprimer les incidents ; utiliser une suppression logique ou une contrainte appropriée.

## 9. Décisions de modélisation

### Pourquoi une table `incident_events` ?

Elle permet de conserver un historique complet et auditables des changements : création, changement de statut, changement de priorité, assignation et escalade.

### Pourquoi `escalated_at` dans `incidents` ?

Elle permet de vérifier rapidement si un incident a déjà été escaladé et d’éviter les escalades en double.

### Pourquoi une table `escalation_rules` ?

Elle permet de modifier les règles sans changer le code applicatif. Les règles deviennent configurables par les administrateurs.