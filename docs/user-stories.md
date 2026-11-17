# User Stories — IncidentDesk

## 1. Authentification

### US-001 — Créer un compte

```text
En tant qu’utilisateur,
je veux créer un compte avec mon email et mon mot de passe,
afin de pouvoir accéder à IncidentDesk.
```

#### Critères d’acceptation

- Given un email et un mot de passe valides, when l’utilisateur s’inscrit, then un compte est créé.
- Given un email déjà utilisé, when l’utilisateur tente de s’inscrire, then l’API retourne `409`.
- Given un mot de passe trop faible, when l’utilisateur s’inscrit, then l’API retourne `422`.

### US-002 — Se connecter

```text
En tant qu’utilisateur,
je veux me connecter avec mon email et mon mot de passe,
afin d’obtenir un token d’accès sécurisé.
```

#### Critères d’acceptation

- Given un compte existant, when l’utilisateur fournit des identifiants valides, then un JWT est retourné.
- Given un mot de passe incorrect, when l’utilisateur se connecte, then l’API retourne `401`.
- Given un compte inexistant, when l’utilisateur se connecte, then l’API retourne `401`.

### US-003 — Accéder à son profil

```text
En tant qu’utilisateur authentifié,
je veux consulter mon profil,
afin de vérifier mes informations et mon rôle.
```

#### Critères d’acceptation

- Given un token JWT valide, when l’utilisateur appelle `/auth/me`, then ses informations sont retournées.
- Given un token invalide ou expiré, when l’utilisateur appelle une route protégée, then l’API retourne `401`.

## 2. Gestion des incidents

### US-010 — Créer un incident

```text
En tant qu’agent,
je veux créer un incident avec un titre, une description et une priorité,
afin de pouvoir suivre un problème technique.
```

#### Critères d’acceptation

- Given un utilisateur authentifié avec le rôle `agent` ou `admin`, when il crée un incident avec des données valides, then l’incident est créé avec le statut `open`.
- Given un titre vide, when l’utilisateur soumet la requête, then l’API retourne `422`.
- Given une priorité invalide, when l’utilisateur soumet la requête, then l’API retourne `422`.
- Given un incident créé, when l’utilisateur consulte la liste, then l’incident apparaît avec sa priorité et son statut.

### US-011 — Consulter la liste des incidents

```text
En tant qu’utilisateur,
je veux consulter la liste des incidents,
afin de voir les problèmes ouverts et leur avancement.
```

#### Critères d’acceptation

- Given un utilisateur authentifié, when il appelle la liste des incidents, then les incidents sont retournés avec pagination.
- Given des filtres de statut ou de priorité, when l’utilisateur les applique, then seuls les incidents correspondants sont retournés.
- Given une recherche par titre ou description, when l’utilisateur la lance, then les incidents correspondants sont retournés.

### US-012 — Consulter le détail d’un incident

```text
En tant qu’utilisateur,
je veux consulter le détail d’un incident,
afin de comprendre le problème, son responsable et son historique.
```

#### Critères d’acceptation

- Given un incident existant, when l’utilisateur consulte son détail, then toutes ses informations sont retournées.
- Given un incident inexistant, when l’utilisateur consulte son détail, then l’API retourne `404`.
- Given un incident existant, when l’utilisateur consulte son détail, then les commentaires et événements sont accessibles.

### US-013 — Assigner un responsable

```text
En tant qu’agent,
je veux assigner un incident à un responsable,
afin que quelqu’un soit clairement responsable de sa résolution.
```

#### Critères d’acceptation

- Given un incident ouvert, when un agent l’assigne à un utilisateur, then l’incident contient le nouvel assignee.
- Given un changement d’assignation, when il est effectué, then un événement d’audit est créé.
- Given un viewer, when il tente d’assigner un incident, then l’API retourne `403`.

### US-014 — Changer le statut

```text
En tant qu’agent,
je veux faire évoluer le statut d’un incident,
afin de refléter son avancement réel.
```

#### Critères d’acceptation

- Given un incident `open`, when l’agent le passe en `in_progress`, then le statut est mis à jour.
- Given un incident `in_progress`, when l’agent le passe en `resolved`, then le statut est mis à jour.
- Given un incident `closed`, when l’agent tente de le repasser en `open`, then l’API retourne `409`.
- Given chaque changement de statut, then un événement d’audit est créé.

### US-015 — Ajouter un commentaire

```text
En tant qu’agent,
je veux ajouter des commentaires à un incident,
afin de documenter les actions et échanges liés au problème.
```

#### Critères d’acceptation

- Given un incident existant, when un agent ajoute un commentaire, then le commentaire est enregistré.
- Given un commentaire vide, when il est soumis, then l’API retourne `422`.
- Given un viewer, when il consulte un incident, then il peut lire les commentaires mais ne peut pas en ajouter.

## 3. Règles d’escalade

### US-020 — Créer une règle d’escalade

```text
En tant qu’admin,
je veux créer une règle d’escalade,
afin de définir comment les incidents doivent être escaladés.
```

#### Critères d’acceptation

- Given un admin authentifié, when il crée une règle valide, then la règle est enregistrée.
- Given un agent ou viewer, when il tente de créer une règle, then l’API retourne `403`.
- Given une priorité invalide, when l’admin crée une règle, then l’API retourne `422`.

### US-021 — Escalader automatiquement un incident

```text
En tant qu’admin,
je veux qu’un incident critique non pris en charge soit escaladé automatiquement,
afin qu’il ne reste pas sans responsable.
```

#### Critères d’acceptation

- Given un incident critique au statut `open`, when il reste non traité pendant 15 minutes, then il est escaladé.
- Given une escalade, then l’incident est réassigné au rôle `on_call`.
- Given une escalade, then un événement de type `escalated` est créé.
- Given une escalade, then une notification est envoyée.
- Given un incident déjà escaladé, when la tâche s’exécute à nouveau, then aucune double escalade n’est créée.

## 4. Notifications

### US-030 — Recevoir une notification d’escalade

```text
En tant que responsable on-call,
je veux recevoir une notification lorsqu’un incident critique m’est assigné,
afin de pouvoir intervenir rapidement.
```

#### Critères d’acceptation

- Given un incident escaladé, when la notification est déclenchée, then une notification est envoyée au rôle on-call.
- Given l’environnement de test, when la notification est déclenchée, then elle est simulée sans appel externe.
- Given un échec de notification, when il se produit, then l’incident reste escaladé et l’erreur est journalisée.