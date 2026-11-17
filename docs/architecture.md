# Architecture — IncidentDesk

## 1. Vue d’ensemble

IncidentDesk est une application backend basée sur une architecture modulaire en couches. Le MVP comprend une API REST, une base de données relationnelle, des tâches asynchrones et un système de notifications.

## 2. Architecture cible

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

## 3. Composants

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

## 4. Structure du projet

```text
incidentdesk/
├── app/
│   ├── main.py
│   ├── core/
│   │   ├── config.py
│   │   ├── security.py
│   │   └── logging.py
│   ├── models/
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
├── docs/
├── docker-compose.yml
├── Dockerfile
├── requirements.txt
├── pyproject.toml
├── .env.example
└── README.md
```

## 5. Flux principal

### Création d’un incident

```text
1. Le client envoie une requête POST /incidents.
2. FastAPI valide les données avec Pydantic.
3. FastAPI vérifie le token JWT et les permissions.
4. Le service IncidentService crée l’incident.
5. SQLAlchemy enregistre l’incident dans PostgreSQL.
6. Un événement incident_created est créé.
7. L’API retourne 201 avec l’incident créé.
```

### Escalade automatique

```text
1. Celery Beat déclenche une tâche périodique.
2. Le worker Celery interroge les incidents critiques non traités.
3. Le service EscalationService applique les règles actives.
4. Si le délai est dépassé, l’incident est réassigné au rôle on-call.
5. Un événement escalated est créé.
6. Le service NotificationService envoie une notification.
7. L’incident est marqué comme escaladé avec escalated_at.
```

## 6. Décisions techniques

### ADR-001 : Utiliser FastAPI

- **Statut** : Accepted
- **Date** : 2026-11-17
- **Contexte** : IncidentDesk nécessite une API REST typée, validée et documentée automatiquement.
- **Décision** : Utiliser FastAPI comme framework backend.
- **Conséquences** : Documentation OpenAPI automatique, validation Pydantic native, développement rapide. En contrepartie, il faut respecter une structure claire pour éviter un projet monolithique difficile à maintenir.

### ADR-002 : Utiliser PostgreSQL

- **Statut** : Accepted
- **Date** : 2026-11-17
- **Contexte** : Les données sont fortement relationnelles : utilisateurs, incidents, commentaires, événements et règles.
- **Décision** : Utiliser PostgreSQL comme base de données principale.
- **Conséquences** : Intégrité relationnelle, requêtes SQL puissantes, index adaptés. En contrepartie, il faut gérer les migrations avec Alembic.

### ADR-003 : Utiliser Celery et Redis

- **Statut** : Accepted
- **Date** : 2026-11-17
- **Contexte** : Les notifications et vérifications d’escalade ne doivent pas bloquer les requêtes API.
- **Décision** : Utiliser Celery pour les tâches asynchrones et Redis comme broker.
- **Conséquences** : Traitement asynchrone, meilleure réactivité de l’API. En contrepartie, il faut surveiller les workers, les files d’attente et les erreurs de traitement.

### ADR-004 : Utiliser JWT

- **Statut** : Accepted
- **Date** : 2026-11-17
- **Contexte** : L’API doit protéger les routes et identifier les utilisateurs.
- **Décision** : Utiliser des tokens JWT pour l’authentification.
- **Conséquences** : Authentification stateless et intégration simple avec un frontend. En contrepartie, il faut gérer l’expiration, le renouvellement et la révocation éventuelle des tokens.

## 7. Sécurité

- Mots de passe hachés.
- Authentification JWT.
- Contrôle d’accès basé sur les rôles.
- Validation stricte des entrées avec Pydantic.
- Secrets gérés via variables d’environnement.
- Aucune clé ou mot de passe stocké dans le dépôt.
- Journalisation des actions sensibles.
- Requêtes SQL paramétrées via SQLAlchemy.

## 8. Observabilité

Le MVP doit inclure :

- Logs structurés.
- Endpoint `/health`.
- Logs des erreurs API.
- Logs des tâches Celery.
- Suivi des escalades.
- Métriques de base : nombre d’incidents, incidents critiques, escalades, erreurs.

## 9. Limites du MVP

- Pas de frontend complet dans la première itération.
- Notifications potentiellement simulées en environnement de test.
- Pas de multi-tenant.
- Pas de haute disponibilité avancée.
- Pas de scaling automatique.
- Pas de monitoring complet en production.
- Règles d’escalade simples et linéaires.

## 10. Évolutions futures

- Frontend React + TypeScript.
- Intégration Slack et email réels.
- Tableau de bord avec indicateurs de résolution.
- Support multi-tenant.
- Observabilité avec Prometheus, Grafana et OpenTelemetry.
- Déploiement Kubernetes avec Helm.
- Tests de charge et benchmarks.