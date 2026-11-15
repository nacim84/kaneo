# Documentation fonctionnelle de Kaneo

> Document établi à partir du dépôt Kaneo, version `2.27.0`. Il décrit le fonctionnement observable de l'application, ses règles métier principales et ses interfaces utilisateurs. Il ne constitue pas une spécification technique exhaustive.

## 1. Finalité du produit

Kaneo est une application de gestion de projets et de tâches, orientée équipes, Kanban et auto-hébergement. Elle permet de :

- organiser le travail par espaces de travail et projets ;
- planifier et exécuter les tâches dans plusieurs vues ;
- collaborer avec des membres disposant de rôles et permissions ;
- synchroniser le travail avec des outils de développement et de messagerie ;
- recevoir des notifications en temps réel et via des canaux externes ;
- exposer les données par API et au moyen d'un serveur MCP pour des assistants IA.

Le principe produit est de conserver un modèle volontairement simple : un espace contient des projets, un projet contient des colonnes et des tâches, et les changements sont propagés aux collaborateurs et aux intégrations.

## 2. Modèle fonctionnel

```text
Instance Kaneo
└── Utilisateurs
    └── Espaces de travail
        ├── Membres, invitations et rôles
        ├── Étiquettes communes
        └── Projets
            ├── Colonnes et règles de workflow
            ├── Champs personnalisés
            ├── Intégrations
            └── Tâches
                ├── Commentaires et activité
                ├── Étiquettes, relations et pièces jointes
                ├── Temps passé
                └── Liens vers des ressources externes
```

### Vocabulaire métier

| Objet | Rôle fonctionnel |
|---|---|
| **Instance** | Déploiement Kaneo autonome, avec sa propre base et sa propre configuration. |
| **Utilisateur** | Personne authentifiée dans l'instance. |
| **Espace de travail** | Frontière de collaboration, de visibilité et d'autorisation. |
| **Projet** | Conteneur de travail appartenant à un espace. |
| **Colonne** | Étape du workflow d'un projet. |
| **Tâche** | Unité de travail, identifiée par un numéro propre au projet, par exemple `KAN-12`. |
| **Étiquette** | Catégorie réutilisable à l'échelle d'un espace. |
| **Relation** | Lien entre deux tâches : sous-tâche, bloque ou liée. |
| **Activité** | Historique des changements et des commentaires d'une tâche. |

## 3. Acteurs et accès

### Authentification

Selon la configuration de l'instance, l'utilisateur peut se connecter par :

- email et mot de passe ;
- code OTP ou lien magique envoyé par email ;
- fournisseur social configuré, notamment GitHub, Google ou Discord ;
- fournisseur OAuth/OIDC personnalisé ;
- accès invité/anonyme si cette option est activée.

La première inscription peut devenir administrateur de l'instance lorsqu'aucun utilisateur n'existe encore. L'instance peut également désactiver les inscriptions, les invités ou certains modes de connexion.

### Rôles d'espace

Les permissions sont contrôlées par l'API et non uniquement par l'interface. Les rôles intégrés sont :

- **Viewer** : lecture seule ;
- **Member** : création et mise à jour courante des projets et tâches ;
- **Admin** : gestion avancée des projets, tâches, membres et paramètres ;
- **Owner** : contrôle complet de l'espace, y compris sa suppression.

Les administrateurs peuvent créer ou modifier des rôles personnalisés. Les permissions couvrent notamment les membres, invitations, projets, tâches, étiquettes, espace de travail et paramètres.

Un projet peut aussi être rendu public. Dans ce cas, son tableau et ses tâches sont consultables sans authentification, en lecture seule, via une URL publique.

## 4. Parcours de démarrage

1. L'utilisateur ouvre l'instance et s'inscrit ou se connecte.
2. Il complète son profil si nécessaire.
3. Il crée ou rejoint un espace de travail.
4. Il crée un premier projet et lui attribue un nom, une icône, une description et éventuellement un préfixe de tâches.
5. Il configure les colonnes et les règles de workflow.
6. Il invite les membres et leur attribue un rôle.
7. Il crée les premières tâches et choisit la vue adaptée à son activité.

## 5. Espaces de travail

Un espace de travail regroupe les personnes, projets et conventions d'une équipe.

Fonctions disponibles :

- création, renommage et description de l'espace ;
- gestion des membres actifs ;
- invitation par email et suivi des invitations en attente ;
- acceptation ou refus d'une invitation ;
- modification et transfert de rôles ;
- création de rôles personnalisés ;
- gestion des étiquettes communes ;
- consultation de la facturation lorsque celle-ci est activée ;
- suppression de l'espace si les autorisations et règles d'abonnement le permettent.

Les membres et les données restent isolés entre espaces. Une tâche ne peut être affectée qu'à un membre de l'espace auquel appartient son projet.

## 6. Projets

Un projet représente un flux de travail. Il possède :

- un nom, un slug et une icône ;
- une description ;
- un ordre dans la liste des projets ;
- un état actif ou archivé ;
- un niveau de visibilité privé ou public ;
- des colonnes, champs personnalisés, règles et intégrations.

Un projet peut être réordonné, modifié, archivé, désarchivé ou supprimé. La suppression est définitive et entraîne la suppression de ses éléments dépendants.

### Vues de projet

| Vue | Usage |
|---|---|
| **Board** | Exécution visuelle par colonnes avec déplacement des cartes. |
| **List** | Lecture dense, tri, contrôle et mise à jour rapide des tâches. |
| **Backlog** | Préparation des tâches avant leur entrée dans le flux actif. |
| **Calendar** | Positionnement des tâches planifiées dans un calendrier mensuel. |
| **Gantt** | Visualisation des tâches possédant une date de début et/ou de fin sur une ligne de temps. |

La vue Backlog est distincte d'une colonne nommée « Backlog ». Les tâches du backlog utilisent le statut système `planned`.

## 7. Workflow et configuration d'un projet

### Colonnes

Chaque projet possède des colonnes ordonnées. Une colonne définit notamment :

- son nom et son slug ;
- sa position ;
- son icône et sa couleur ;
- si elle représente un état final.

Le statut d'une tâche correspond au slug de sa colonne. Déplacer une tâche entre colonnes modifie donc son statut et sa position.

Les colonnes finales peuvent être utilisées pour considérer le travail comme terminé et permettre l'archivage groupé des tâches terminées.

### Champs personnalisés

Un projet peut définir des champs personnalisés de type :

- texte ;
- nombre ;
- date ;
- liste déroulante ;
- booléen.

Un champ peut être obligatoire, avoir une valeur par défaut et, pour une liste, posséder des options. Les valeurs sont portées par les tâches et peuvent être utilisées dans les filtres.

### Règles d'automatisation

Une règle associe :

1. un type d'intégration ;
2. un événement externe ;
3. une colonne cible.

Lorsqu'un événement correspondant est reçu, Kaneo peut déplacer automatiquement la tâche vers la colonne configurée. Cela permet, par exemple, d'aligner le statut Kaneo sur les étapes d'une issue ou d'une pull request.

## 8. Tâches

### Création et identification

Une tâche comporte au minimum un titre, une description, une priorité et un statut. Elle reçoit un numéro séquentiel dans son projet et est affichée sous la forme `<slug-projet>-<numéro>`.

Exemple : `WEB-42`.

### Informations gérées

Une tâche peut contenir :

- titre et description Markdown ;
- statut/colonne ;
- priorité : sans priorité, basse, moyenne, haute ou urgente ;
- responsable, ou aucun responsable ;
- date de début et date d'échéance ;
- étiquettes ;
- champs personnalisés ;
- commentaires ;
- historique d'activité ;
- images dans la description ou les commentaires ;
- temps passé ;
- relations avec d'autres tâches ;
- liens vers GitHub, Gitea ou d'autres systèmes.

### Actions principales

Selon les permissions de l'utilisateur, une tâche peut être :

- créée, modifiée ou supprimée ;
- déplacée dans une autre colonne ;
- déplacée vers un autre projet du même espace ;
- affectée ou désaffectée ;
- marquée avec une priorité, une échéance ou des étiquettes ;
- commentée ;
- liée à une autre tâche ;
- importée ou exportée ;
- mise à jour en lot avec d'autres tâches.

La suppression d'une tâche supprime également ses commentaires, étiquettes et entrées de temps associées.

### Relations entre tâches

Les relations disponibles sont :

- **sous-tâche** : rattachement hiérarchique ;
- **bloque** : une tâche empêche l'autre d'avancer ;
- **liée** : relation informative sans dépendance particulière.

### Recherche et filtrage

Dans un projet, les tâches peuvent être recherchées par texte, titre, description, numéro ou identifiant court. Les vues permettent également de filtrer par :

- statut ;
- priorité ;
- responsable ;
- échéance ;
- étiquettes ;
- valeurs de champs personnalisés.

Une recherche globale est disponible au niveau de l'espace/instance.

## 9. Planification et suivi du temps

La planification s'effectue avec les dates de début et d'échéance :

- la vue calendrier montre les tâches planifiées par mois ;
- la vue Gantt positionne les tâches sur une ligne de temps ;
- les tâches sans date n'apparaissent pas dans les vues temporelles.

Les utilisateurs peuvent enregistrer des entrées de temps sur une tâche, avec une heure de début, une heure de fin ou une durée, ainsi qu'une description. Les rappels d'échéance sont générés pour les tâches affectées lorsqu'ils sont activés.

## 10. Collaboration et notifications

### Collaboration dans une tâche

L'activité d'une tâche conserve les changements importants et les commentaires. Les commentaires sont attribués à leur auteur et peuvent provenir d'une intégration externe.

Les mises à jour importantes peuvent produire une notification personnelle : création de tâche, changement de statut, changement de responsable, commentaire ou rappel d'échéance, selon les préférences configurées.

### Notifications

La boîte de réception interne est la source de vérité. Un utilisateur peut :

- consulter ses notifications ;
- marquer une notification comme lue ;
- tout marquer comme lu ;
- vider sa boîte ;
- régler ses préférences de type de notification.

Les notifications externes peuvent être envoyées par :

- email ;
- ntfy ;
- Gotify ;
- webhook HTTP personnalisé.

Ces canaux sont personnels à l'utilisateur et peuvent être activés par espace, puis pour tous les projets ou une sélection de projets. Une défaillance d'envoi externe ne bloque pas la création de la notification interne.

## 11. Compte et préférences

Depuis les paramètres du compte, l'utilisateur peut :

- modifier son nom et sa langue ;
- ajouter, remplacer ou supprimer son avatar ;
- choisir le thème clair ou sombre ;
- définir la vue de tâche par défaut, le mode compact et l'état initial de la barre latérale ;
- choisir les informations visibles dans les listes et tableaux, notamment numéros, responsables, dates, étiquettes et priorités ;
- gérer ses clés API depuis la section développeur ;
- modifier son mot de passe ou demander sa réinitialisation ;
- supprimer son compte après confirmation.

La suppression d'un compte est irréversible. Dans un espace dont l'utilisateur est le seul membre, l'espace et ses données sont supprimés. Dans un espace partagé, l'utilisateur est retiré, ses tâches restent mais sont désassignées, son temps enregistré est conservé, ses commentaires sont supprimés et ses sessions et clés API sont révoquées.

La suppression est refusée si l'utilisateur est le seul propriétaire d'un espace partagé ou si un abonnement actif doit d'abord être résilié. Une reconnexion récente peut également être exigée.

## 12. Étiquettes

Les étiquettes sont définies au niveau de l'espace afin d'être réutilisables dans tous les projets.

Un utilisateur autorisé peut :

- créer une étiquette avec un nom et une couleur ;
- renommer ou recolorer une étiquette ;
- l'attacher ou la retirer d'une tâche ;
- supprimer l'étiquette.

Une modification est répercutée sur toutes les tâches qui l'utilisent. Une suppression importante peut être traitée progressivement : l'étiquette apparaît alors comme « suppression en attente » jusqu'à la fin du traitement.

## 13. Intégrations

### GitHub

Un projet peut être relié à un dépôt GitHub. L'intégration permet notamment de :

- importer les issues ouvertes comme tâches ;
- éviter les doublons en conservant un lien externe ;
- importer labels et commentaires ;
- afficher les pull requests liées ;
- synchroniser certains événements GitHub vers les tâches ;
- créer ou mettre à jour une issue GitHub depuis les événements Kaneo selon la configuration ;
- déplacer automatiquement une tâche via les règles de workflow.

Les changements GitHub sont reçus par webhook. Les importations volumineuses sont paginées et reprenables.

### Gitea

L'intégration Gitea suit le même principe général : liaison d'un dépôt, import des issues, conservation des liens externes, import des commentaires et possibilité de publier un lien vers la tâche Kaneo.

### Slack, Discord, Mattermost et Telegram

Ces intégrations sont principalement sortantes et configurées par projet. Elles peuvent publier les événements suivants, avec activation indépendante :

- tâche créée ;
- changement de statut ;
- changement de priorité ;
- changement de titre ;
- changement de description ;
- commentaire créé.

Les paramètres de connexion dépendent du service : webhook entrant, bot et identifiant de conversation ou configuration équivalente.

### Webhook HTTP générique

Kaneo peut envoyer un POST JSON vers une URL configurée par projet. Le payload contient l'événement, le projet, la tâche, l'acteur et les données spécifiques au changement.

Un secret facultatif permet de signer le corps avec HMAC-SHA256 dans l'en-tête `X-Kaneo-Signature`.

### API et MCP

L'API HTTP est documentée via OpenAPI et utilisée par l'application web. Les clés API peuvent être gérées depuis les paramètres du compte.

Kaneo propose aussi un serveur MCP :

- endpoint HTTP intégré `/api/mcp`, avec autorisation OAuth/PKCE ;
- package stdio `@kaneo/mcp`, avec authentification par appareil.

Les outils MCP donnent accès notamment aux espaces, projets, colonnes, tâches, commentaires, étiquettes, relations, notifications et entrées de temps, selon les permissions de l'utilisateur.

## 14. Partage public

Un projet peut être partagé publiquement par un utilisateur disposant de la permission correspondante. Le visiteur non authentifié peut alors :

- consulter le tableau ou la liste publique ;
- lire la description du projet ;
- ouvrir le détail d'une tâche ;
- consulter les informations rendues publiques.

Le partage public est en lecture seule. Les opérations d'écriture restent soumises à l'authentification et aux permissions de l'espace.

## 15. Facturation et entitlements

Lorsqu'elle est activée, la facturation est rattachée à l'espace de travail. Le système gère notamment :

- période d'essai ;
- abonnement et statut d'abonnement ;
- nombre de sièges ;
- création d'un checkout ;
- portail de gestion ;
- réception de webhooks de paiement ;
- contrôle d'accès à certaines fonctions selon l'entitlement.

La suppression d'un compte peut être refusée tant qu'un utilisateur est le seul propriétaire d'un espace partagé ou qu'un abonnement actif doit être résilié.

## 16. Fonctionnement transversal

### API comme autorité métier

Le navigateur ne fait qu'afficher ou masquer les actions disponibles. L'API revalide systématiquement :

- l'identité et la session ;
- l'appartenance à l'espace ;
- la permission requise ;
- la cohérence des relations entre espace, projet et tâche ;
- les règles de sécurité des intégrations et des fichiers.

### Persistance

PostgreSQL conserve les utilisateurs, espaces, projets, tâches, commentaires, activités, notifications, étiquettes, intégrations, rôles et entrées de temps. Les migrations permettent de faire évoluer le schéma sans repartir d'une base vide.

Les images de tâches peuvent être stockées via un stockage objet compatible S3 lorsque cette option est configurée. Les avatars utilisateurs sont conservés dans la base.

### Temps réel

Les changements d'un projet sont diffusés par WebSocket aux clients connectés. Les notifications personnelles utilisent un canal WebSocket associé à l'utilisateur.

En déploiement mono-instance, la diffusion reste en mémoire. Redis est optionnel et sert de relais lorsque plusieurs instances API doivent partager les événements temps réel.

### Événements

Les mutations publient des événements internes. Ils peuvent alimenter simultanément :

- l'activité de la tâche ;
- les notifications ;
- les WebSockets ;
- les intégrations sortantes ;
- les règles d'automatisation ;
- les rappels planifiés.

## 17. Scénarios fonctionnels de référence

### Préparer puis exécuter une fonctionnalité

1. Créer une tâche dans le Backlog.
2. Renseigner titre, description, priorité, responsable et étiquette.
3. Ajouter une date si nécessaire.
4. Déplacer la tâche dans la première colonne active lorsqu'elle est prête.
5. La faire progresser par glisser-déposer dans le Board.
6. Ajouter commentaires, temps passé et liens externes pendant l'exécution.
7. La placer dans une colonne finale puis archiver les tâches terminées si besoin.

### Connecter un dépôt GitHub

1. Ouvrir les intégrations du projet.
2. Choisir GitHub et sélectionner un dépôt autorisé.
3. Importer les issues ouvertes.
4. Vérifier les liens entre tâches et issues.
5. Configurer les colonnes cibles et les règles d'automatisation.
6. Laisser les webhooks maintenir les statuts et événements à jour.

### Donner un accès limité à un collaborateur

1. Inviter la personne dans l'espace.
2. Lui attribuer le rôle `viewer` ou un rôle personnalisé.
3. Vérifier les permissions de lecture, création, mise à jour et assignation.
4. Le serveur bloque les opérations qui dépassent ce rôle, même si un appel API manuel est tenté.

## 18. Règles et limites importantes

- Un espace de travail est la frontière principale d'autorisation et de confidentialité.
- Une tâche ne peut être assignée qu'à un membre de son espace.
- Un projet public ne permet pas d'écriture anonyme.
- Le statut d'une tâche est lié au slug d'une colonne du projet.
- Le backlog système (`planned`) n'est pas équivalent à une colonne utilisateur nommée « Backlog ».
- Les imports et suppressions volumineux peuvent être progressifs et reprendre après interruption.
- Les descriptions volumineuses peuvent être chargées par pages plutôt que dans les listes.
- Les intégrations sortantes ne remplacent pas l'historique interne de Kaneo.
- Les erreurs de livraison d'une notification externe ne suppriment pas la notification interne.
- La suppression d'une tâche, d'un projet ou d'un espace peut entraîner des suppressions en cascade.

## 19. Cartographie du dépôt

| Fonction | Emplacement principal |
|---|---|
| API et composition des routes | `apps/api/src/index.ts` |
| Authentification | `apps/api/src/auth.ts` |
| Modèle de données | `apps/api/src/database/schema.ts` |
| Permissions | `packages/permissions/src/index.ts` |
| Projets | `apps/api/src/project`, `apps/web/src/routes/.../project` |
| Tâches | `apps/api/src/task`, `apps/web/src/components/task` |
| Vues Board/List/Backlog/Calendar/Gantt | `apps/web/src/routes/.../project` et `apps/web/src/components` |
| Événements et temps réel | `apps/api/src/events`, `apps/api/src/ws` |
| Intégrations | `apps/api/src/*-integration`, `apps/api/src/plugins` |
| MCP | `apps/api/src/mcp`, `packages/mcp` |
| Guides utilisateurs existants | `apps/docs/core/functional` |

## 20. Conclusion

Kaneo fonctionne comme un système de gestion de travail centré sur l'espace de travail et le projet. Le chemin nominal est :

```text
Utilisateur → Espace → Projet → Workflow → Tâche → Activité/Notifications/Intégrations
```

L'API conserve la cohérence et les autorisations, PostgreSQL conserve l'état durable, l'interface React fournit les différentes vues, et le système d'événements relie les modifications aux notifications, aux intégrations et au temps réel.
