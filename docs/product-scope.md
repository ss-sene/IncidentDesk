# Product Scope — IncidentDesk

## 1. Nom du produit

**IncidentDesk**

## 2. Vision

IncidentDesk est une plateforme de gestion d’incidents conçue pour les petites équipes techniques. Elle permet de centraliser les incidents, d’assigner des responsables, de suivre leur évolution et d’escalader automatiquement les incidents critiques non traités.

## 3. Problème à résoudre

Les petites équipes techniques reçoivent souvent des incidents par email, messagerie ou téléphone. Sans outil centralisé, plusieurs problèmes apparaissent :

- Les demandes se perdent.
- Personne ne sait clairement qui est responsable d’un incident.
- Les incidents critiques ne sont pas escaladés à temps.
- L’historique des décisions et des changements de statut est difficile à retrouver.
- Les responsables n’ont pas de visibilité sur les incidents ouverts ou en retard.

## 4. Objectif du MVP

Construire une version fonctionnelle d’IncidentDesk permettant à une équipe technique de :

- Créer et suivre des incidents.
- Attribuer un responsable à chaque incident.
- Suivre le cycle de vie d’un incident.
- Conserver un historique des changements.
- Escalader automatiquement les incidents critiques non traités.
- Notifier les responsables lors d’une escalade.

## 5. Utilisateurs cibles

| Rôle | Description | Cas d’usage principal |
|---|---|---|
| Admin | Responsable de l’équipe ou de la plateforme | Configure les règles d’escalade, gère les utilisateurs et supervise les incidents |
| Agent | Membre de l’équipe technique | Crée, consulte, met à jour, s’assigne et résout les incidents |
| Viewer | Observateur ou personne non technique | Consulte les incidents et commentaires sans pouvoir les modifier |

## 6. Périmètre inclus dans le MVP

### Authentification

- Inscription d’un utilisateur.
- Connexion avec email et mot de passe.
- Génération d’un token JWT.
- Déconnexion.
- Gestion des rôles : `admin`, `agent`, `viewer`.

### Gestion des incidents

- Créer un incident.
- Consulter la liste des incidents.
- Consulter le détail d’un incident.
- Modifier un incident.
- Supprimer un incident, réservé aux administrateurs.
- Assigner un responsable.
- Changer le statut d’un incident.
- Ajouter un commentaire.
- Consulter l’historique des changements.

### Recherche et filtres

- Rechercher par titre ou description.
- Filtrer par statut.
- Filtrer par priorité.
- Filtrer par responsable.
- Filtrer par période de création.
- Paginer les résultats.

### Règles d’escalade

- Créer une règle d’escalade.
- Modifier une règle d’escalade.
- Activer ou désactiver une règle.
- Détecter automatiquement les incidents en retard.
- Réassigner un incident au rôle on-call.
- Créer un événement d’escalade.
- Envoyer une notification.

### Notifications

- Notification email ou Slack.
- Mode mock pour les tests.
- Notification lors d’une escalade.
- Notification lors de l’assignation d’un incident critique.

## 7. Périmètre exclu du MVP

Les fonctionnalités suivantes ne seront pas développées dans la première version :

- Interface frontend complète.
- Multi-tenant.
- Paiements.
- Intégration Jira.
- Intégration Slack avancée.
- Rapports avancés.
- Machine learning.
- Chat en temps réel.
- Application mobile.
- SSO.
- Kubernetes.
- Tests de charge avancés.
- Support multilingue.

## 8. Parcours principal

```text
Un agent crée un incident critique
        |
L’incident est créé avec le statut open
        |
Aucun agent ne le prend en charge
        |
Après 15 minutes, la règle d’escalade se déclenche
        |
L’incident est réassigné au rôle on-call
        |
Un événement d’escalade est créé
        |
Une notification est envoyée
        |
Un agent prend en charge l’incident
        |
L’incident est résolu puis clôturé
```

## 9. Critères de succès du MVP

Le MVP est considéré comme terminé lorsque :

- Un utilisateur peut s’inscrire et se connecter.
- Un agent peut créer un incident.
- Un agent peut assigner un incident.
- Un agent peut changer le statut d’un incident.
- Un agent peut ajouter un commentaire.
- L’historique des changements est visible.
- Un incident critique non traité est escaladé après 15 minutes.
- Une notification est envoyée lors de l’escalade.
- Les tests couvrent les règles critiques.
- Le projet démarre avec Docker Compose.
- Le README explique le projet, l’architecture et le démarrage.

## 10. Indicateurs de succès

| Indicateur | Objectif |
|---|---|
| Incidents critiques escaladés automatiquement | 100 % des incidents critiques non traités après le délai configuré |
| Traçabilité des changements | 100 % des changements de statut, priorité et assignation enregistrés |
| Tests critiques | Couverture des règles métier et de l’authentification |
| Déploiement local | Démarrage complet avec `docker compose up --build` |
| Documentation | README, architecture, modèle de données et contrat API disponibles |