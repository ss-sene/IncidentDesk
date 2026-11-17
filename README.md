# IncidentDesk

**IncidentDesk** est une plateforme backend de gestion d’incidents conçue pour les petites équipes techniques. Elle centralise les incidents, applique des règles de priorisation, assigne les responsabilités, conserve un historique d’audit et escalade automatiquement les incidents critiques non traités.

Le projet met l’accent sur une architecture backend propre, des règles métier testables, une API REST sécurisée, des tâches asynchrones et un déploiement reproductible avec Docker.

## Problème

Dans une petite équipe technique, les incidents arrivent souvent par email, messagerie ou téléphone. Sans outil centralisé, plusieurs problèmes apparaissent :

- Les demandes se perdent ou ne sont pas suivies.
- Personne ne sait clairement qui est responsable d’un incident.
- Les incidents critiques ne sont pas escaladés à temps.
- L’historique des décisions et des changements de statut est difficile à retrouver.
- Les responsables n’ont pas de visibilité sur les incidents ouverts ou en retard.

## Solution

IncidentDesk fournit une API REST qui permet de :

- Créer et suivre des incidents.
- Attribuer un incident à un responsable.
- Définir une priorité et un statut.
- Ajouter des commentaires et conserver un historique des changements.
- Rechercher, filtrer et paginer les incidents.
- Appliquer des règles d’escalade automatique.
- Notifier les responsables lorsqu’un incident critique reste non traité.
- Contrôler l’accès selon les rôles utilisateurs.

La valeur principale du projet ne réside pas dans le CRUD, mais dans son **moteur de règles d’escalade** et dans la traçabilité complète du cycle de vie d’un incident.

## Fonctionnalités principales

- Création, consultation, mise à jour et clôture d’incidents.
- Attribution d’un incident à un utilisateur.
- Priorités : `low`, `medium`, `high`, `critical`.
- Statuts : `open`, `in_progress`, `resolved`, `closed`.
- Commentaires liés à un incident.
- Historique d’audit des changements de statut, de priorité et d’assignation.
- Recherche et filtrage par statut, priorité, responsable et période.
- Pagination des résultats.
- Authentification par JWT.
- Gestion des rôles : `admin`, `agent`, `viewer`.
- Règles d’escalade configurables.
- Notifications asynchrones via Celery et Redis.
- Documentation API automatique avec OpenAPI/Swagger.

## Exemple de règle d’escalade

```text
Si un incident est critique
et reste au statut open
pendant plus de 15 minutes,
alors :
1. L’incident est réassigné au rôle on-call.
2. Un événement d’escalade est créé.
3. Une notification est envoyée.
```

Les règles sont stockées en base de données afin de pouvoir être modifiées sans modifier le code applicatif.

## Architecture

```text
+---------------------+
| Client / API client |
+---------------------+
           |
           v
+---------------------+
|      FastAPI        |
|  REST API + JWT     |
+---------------------+
           |
           v
+---------------------+
|     SQLAlchemy      |
+---------------------+
           |
           v
+---------------------+
|     PostgreSQL      |
+---------------------+

+---------------------+
|     Celery Worker   |
+---------------------+
           |
           v
+---------------------+
|     Redis Broker    |
+---------------------+
           |
           v
+---------------------+
| Notification service|
+---------------------+
```

### Composants

| Composant | Rôle |
|---|---|
| FastAPI | Expose l’API REST, valide les requêtes et gère l’authentification |
| PostgreSQL | Stocke les utilisateurs, incidents, commentaires, événements et règles |
| SQLAlchemy | Fournit l’ORM et la couche d’accès aux données |
| Alembic | Gère les migrations de base de données |
| Pydantic | Valide les données entrantes et sortantes |
| Celery | Exécute les tâches asynchrones, notamment les escalades et notifications |
| Redis | Sert de broker pour Celery |
| Docker Compose | Orchestre les services en local |
| GitHub Actions | Exécute les tests et vérifie la qualité du code |

## Stack technique

- **Langage** : Python 3.12+
- **Framework API** : FastAPI
- **Base de données** : PostgreSQL
- **ORM** : SQLAlchemy
- **Migrations** : Alembic
- **Validation** : Pydantic
- **Authentification** : JWT
- **Tests** : pytest
- **Tâches asynchrones** : Celery
- **Broker** : Redis
- **Conteneurisation** : Docker, Docker Compose
- **CI/CD** : GitHub Actions
- **Documentation API** : OpenAPI / Swagger UI

## Décisions techniques

### FastAPI

FastAPI a été choisi pour son support natif des types Python, de Pydantic et de la documentation OpenAPI. Il permet de créer une API typée, validée et documentée avec peu de configuration supplémentaire.

### PostgreSQL

PostgreSQL est adapté à ce cas d’usage car les données sont fortement relationnelles : un incident possède un reporter, un responsable, des commentaires, des événements et des règles d’escalade.

### SQLAlchemy et Alembic

SQLAlchemy fournit une couche d’accès aux données claire et testable. Alembic permet de versionner les changements de schéma, ce qui est essentiel pour faire évoluer le projet de manière contrôlée.

### Celery et Redis

Les notifications et les vérifications d’escalade ne doivent pas bloquer les requêtes API. Elles sont donc exécutées de manière asynchrone par un worker Celery, avec Redis comme broker.

### JWT

L’authentification repose sur des tokens JWT afin de protéger les endpoints et d’identifier l’utilisateur qui effectue chaque action.

## Structure du projet

```text
incidentdesk/
├── app/
│   ├── main.py
│   ├── core/
│   │   ├── config.py
│   │   ├── security.py
│   │   └── logging.py
│   ├── models/
│   │   ├── user.py
│   │   ├── incident.py
│   │   ├── comment.py
│   │   ├── event.py
│   │   └── escalation_rule.py
│   ├── schemas/
│   ├── api/
│   │   ├── routes/
│   │   └── dependencies.py
│   ├── services/
│   │   ├── incident_service.py
│   │   ├── escalation_service.py
│   │   └── notification_service.py
│   ├── repositories/
│   └── workers/
│       ├── celery_app.py
│       └── tasks.py
├── tests/
│   ├── unit/
│   └── integration/
├── alembic/
├── docker-compose.yml
├── Dockerfile
├── requirements.txt
├── pyproject.toml
├── .env.example
└── README.md
```

## Modèle de données

### users

| Champ | Type | Description |
|---|---|---|
| id | UUID | Identifiant unique |
| email | String | Email unique de l’utilisateur |
| full_name | String | Nom complet |
| role | Enum | `admin`, `agent`, `viewer` |
| created_at | DateTime | Date de création |

### incidents

| Champ | Type | Description |
|---|---|---|
| id | UUID | Identifiant unique |
| title | String | Titre de l’incident |
| description | Text | Description détaillée |
| status | Enum | `open`, `in_progress`, `resolved`, `closed` |
| priority | Enum | `low`, `medium`, `high`, `critical` |
| reporter_id | UUID | Utilisateur ayant créé l’incident |
| assignee_id | UUID | Utilisateur responsable |
| due_at | DateTime | Échéance de traitement |
| created_at | DateTime | Date de création |
| updated_at | DateTime | Date de dernière mise à jour |

### incident_comments

| Champ | Type | Description |
|---|---|---|
| id | UUID | Identifiant unique |
| incident_id | UUID | Incident associé |
| author_id | UUID | Auteur du commentaire |
| content | Text | Contenu du commentaire |
| created_at | DateTime | Date de création |

### incident_events

| Champ | Type | Description |
|---|---|---|
| id | UUID | Identifiant unique |
| incident_id | UUID | Incident concerné |
| actor_id | UUID | Utilisateur ou système ayant déclenché l’événement |
| event_type | String | Type d’événement |
| from_value | String | Valeur précédente |
| to_value | String | Nouvelle valeur |
| created_at | DateTime | Date de l’événement |

### escalation_rules

| Champ | Type | Description |
|---|---|---|
| id | UUID | Identifiant unique |
| priority | Enum | Priorité concernée |
| max_wait_minutes | Integer | Délai avant escalade |
| escalate_to_role | String | Rôle de destination |
| is_active | Boolean | Règle active ou non |

## API principale

| Méthode | Endpoint | Description |
|---|---|---|
| POST | `/api/v1/auth/register` | Créer un compte utilisateur |
| POST | `/api/v1/auth/login` | Obtenir un token JWT |
| GET | `/api/v1/incidents` | Lister les incidents |
| POST | `/api/v1/incidents` | Créer un incident |
| GET | `/api/v1/incidents/{id}` | Obtenir un incident |
| PATCH | `/api/v1/incidents/{id}` | Mettre à jour un incident |
| DELETE | `/api/v1/incidents/{id}` | Supprimer un incident |
| POST | `/api/v1/incidents/{id}/comments` | Ajouter un commentaire |
| GET | `/api/v1/incidents/{id}/events` | Consulter l’historique |
| POST | `/api/v1/incidents/{id}/assign` | Assigner un responsable |
| GET | `/api/v1/escalation-rules` | Lister les règles d’escalade |
| POST | `/api/v1/escalation-rules` | Créer une règle d’escalade |

La documentation interactive est disponible sur :

```text
http://localhost:8000/docs
```

## Exemple de requête

### Créer un incident

```http
POST /api/v1/incidents
Content-Type: application/json
Authorization: Bearer <token>
```

```json
{
  "title": "API de paiement indisponible",
  "description": "Les paiements échouent depuis 10 minutes.",
  "priority": "critical",
  "due_at": "2026-11-17T15:00:00Z"
}
```

### Réponse attendue

```json
{
  "id": "8f2c...",
  "title": "API de paiement indisponible",
  "description": "Les paiements échouent depuis 10 minutes.",
  "status": "open",
  "priority": "critical",
  "reporter_id": "1a2b...",
  "assignee_id": null,
  "created_at": "2026-11-17T12:00:00Z"
}
```

## Tests

Le projet utilise `pytest` pour les tests unitaires et d’intégration.

### Tests couverts

- Création et validation des incidents.
- Transitions de statut valides et invalides.
- Attribution d’un responsable.
- Permissions selon les rôles.
- Authentification JWT.
- Règles d’escalade.
- Notifications asynchrones.
- Pagination et filtrage.
- Historique des événements.

### Lancer les tests

```bash
pytest
```

### Lancer les tests avec couverture

```bash
pytest --cov=app --cov-report=term-missing
```

## Démarrage local

### Prérequis

- Python 3.12+
- Docker et Docker Compose
- Git

### Cloner le projet

```bash
git clone [https://github.com/](https://github.com/)<ton-utilisateur>/incidentdesk.git
cd incidentdesk
```

### Configurer les variables d’environnement

```bash
cp .env.example .env
```

Puis renseigne les variables nécessaires dans `.env`.

### Lancer les services

```bash
docker compose up --build
```

L’API est disponible sur :

```text
http://localhost:8000
```

La documentation Swagger est disponible sur :

```text
http://localhost:8000/docs
```

## CI/CD

À chaque push sur la branche principale, GitHub Actions :

1. Installe les dépendances.
2. Vérifie le format du code.
3. Exécute les tests.
4. Mesure la couverture.
5. Construit l’image Docker.
6. Valide le build.

Le pipeline garantit que le code ne peut pas être fusionné s’il ne passe pas les tests.

## Sécurité

- Mots de passe hachés avec un algorithme adapté.
- Authentification par JWT.
- Contrôle d’accès basé sur les rôles.
- Validation stricte des entrées avec Pydantic.
- Secrets gérés via variables d’environnement.
- Aucune clé ou mot de passe stocké dans le dépôt.
- Journalisation des actions sensibles.

## Limites actuelles

- L’interface utilisateur n’est pas encore incluse dans le MVP.
- Les notifications peuvent être simulées en environnement de test.
- Le système n’est pas encore conçu pour un très fort volume concurrent.
- Les règles d’escalade sont actuellement simples et linéaires.
- La surveillance avancée et les alertes Prometheus/Grafana ne sont pas encore intégrées.

## Améliorations prévues

- Interface frontend en React/TypeScript.
- Intégration Slack et email réels.
- Tableau de bord avec indicateurs de résolution.
- Notifications basées sur les préférences utilisateur.
- Support multi-tenant.
- Observabilité avec Prometheus, Grafana et OpenTelemetry.
- Tests de charge et benchmarks.
- Déploiement Kubernetes avec Helm.

## Auteur

Développé par **[Ton nom]**.

- GitHub : https://github.com/<ton-utilisateur>
- LinkedIn : https://www.linkedin.com/in/<ton-profil>
- Portfolio : https://<ton-portfolio>

## Licence

Ce projet est distribué sous licence MIT.