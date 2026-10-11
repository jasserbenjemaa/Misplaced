# 🎒 LOST & FOUND — Campus

> Application web de déclaration et de restitution d'objets perdus ou trouvés, avec détection automatique de correspondances, notifications et modération.

**Stack :** Node.js / Express · MongoDB (Mongoose) · React
**Équipe :** 8 personnes (4 binômes) — voir [Équipe](#-équipe)

> **📝 À adapter :** ce README propose un contexte (campus universitaire) et des choix par défaut cohérents avec le cahier des charges. Remplacez-les par vos propres décisions et gardez les justifications : c'est ce qui est évalué.

---

## Sommaire

1. [Contexte et justification](#1-contexte-et-justification)
2. [Fonctionnalités](#2-fonctionnalités)
3. [Architecture](#3-architecture)
4. [Installation et lancement](#4-installation-et-lancement)
5. [Modèle de données MongoDB](#5-modèle-de-données-mongodb)
6. [Rôles et permissions](#6-rôles-et-permissions)
7. [Cycle de vie des déclarations](#7-cycle-de-vie-des-déclarations)
8. [Binôme 1 — Utilisateurs, déclarations, sécurité](#8-binôme-1--utilisateurs-déclarations-sécurité)
9. [Binôme 2 — Recherche et exploration](#9-binôme-2--recherche-et-exploration)
10. [Binôme 3 — Matching et résolution](#10-binôme-3--matching-et-résolution)
11. [Binôme 4 — Administration, modération, statistiques](#11-binôme-4--administration-modération-statistiques)
12. [Notifications](#12-notifications)
13. [Validation et gestion des erreurs](#13-validation-et-gestion-des-erreurs)
14. [API, documentation et Postman](#14-api-documentation-et-postman)
15. [Scénario de démonstration](#15-scénario-de-démonstration)
16. [Cas non prévus et décisions](#16-cas-non-prévus-et-décisions)
17. [Fonctionnalités supplémentaires](#17-fonctionnalités-supplémentaires)
18. [Équipe](#-équipe)

---

## 1. Contexte et justification

**Contexte choisi : un campus universitaire** (étudiants, enseignants, personnel administratif, agents d'accueil).

**Pourquoi ce contexte ?**

- Le volume d'objets perdus est élevé et récurrent (bibliothèque, amphithéâtres, cafétéria, salles de TP).
- Les lieux sont bien délimités et nommables (bâtiments, salles), ce qui rend la **comparaison géographique fiable** pour le matching.
- Les utilisateurs sont identifiables (adresse e-mail universitaire), ce qui limite les abus.
- Il existe déjà un acteur naturel pour la modération : l'administration / l'accueil du campus.

**Impact du contexte sur la conception**

| Aspect | Choix |
|---|---|
| Utilisateurs | Étudiants, personnel, administrateurs (accueil) |
| Catégories | Carte étudiante, clés, ordinateur / tablette, téléphone, sac, vêtement, bouteille / gourde, livre / cours, accessoire (écouteurs, chargeur…), autre |
| Lieux | Liste de bâtiments et zones du campus (bibliothèque, amphi A, cafétéria…) avec notion de **proximité** entre zones |
| Règles métier | Une carte étudiante ou un document d'identité n'affiche jamais le nom en public ; un objet trouvé peut être déposé à l'accueil |
| Modération | Signalement par les utilisateurs, traitement par l'administration |
| Vocabulaire UI | « J'ai perdu », « J'ai trouvé », « Déposé à l'accueil », « Récupéré » |

---

## 2. Fonctionnalités

- Création de compte, connexion, profil
- Déclaration d'un objet **perdu** ou **trouvé** (avec photo optionnelle)
- Recherche multicritères avec tri et pagination
- **Détection automatique de correspondances** avec score et niveau de confiance
- Réaction à une correspondance : accepter, refuser, demander à vérifier, feedback
- **Historique** complet de chaque correspondance
- Notifications internes (lues / non lues)
- Signalement de contenus problématiques
- Tableau de bord administrateur : modération, gestion des utilisateurs, statistiques

---

## 3. Architecture

```
┌────────────┐   HTTP / JSON    ┌──────────────────┐     ┌───────────┐
│  React SPA │ ───────────────▶ │  API Express     │ ──▶ │  MongoDB  │
│  (Vite)    │ ◀─────────────── │  (REST, JWT)     │ ◀── │           │
└────────────┘                  └──────────────────┘     └───────────┘
                                   │  modules :
                                   │  auth · declarations · search
                                   │  matching · notifications
                                   │  admin · reports · stats
```

### Structure du dépôt

```
lost-and-found/
├── backend/
│   ├── src/
│   │   ├── config/          # env, connexion DB
│   │   ├── models/          # schémas Mongoose
│   │   ├── routes/
│   │   ├── controllers/
│   │   ├── services/        # logique métier (matching, notifications…)
│   │   ├── middlewares/     # auth, rôles, validation, erreurs
│   │   ├── validators/      # schémas de validation
│   │   ├── utils/
│   │   └── app.js / server.js
│   └── tests/
├── frontend/
│   └── src/
│       ├── api/             # appels HTTP
│       ├── components/
│       ├── pages/
│       ├── context/         # auth, notifications
│       └── hooks/
├── postman/                 # collections + environnements
├── docs/                    # documentation API (Swagger/OpenAPI), schémas
└── README.md
```

### Choix techniques

| Sujet | Choix | Justification |
|---|---|---|
| Authentification | JWT (access token) + hash `bcrypt` | Simple, sans état, adapté à une SPA |
| ODM | Mongoose | Schémas et validation côté modèle |
| Validation | Joi / Zod (au choix de l'équipe) | Toute entrée est considérée non fiable |
| Documentation | Swagger / OpenAPI | Testable directement, versionnable |
| Front | React + React Router | Navigation par pages, état d'auth via contexte |

---

## 4. Installation et lancement

### Prérequis

- Node.js ≥ 18
- MongoDB ≥ 6 (local ou Atlas)
- npm

### Backend

```bash
cd backend
npm install
cp .env.example .env     # puis renseigner les valeurs
npm run seed             # (optionnel) données de démonstration
npm run dev              # http://localhost:5000
```

**Variables d'environnement (`backend/.env`)**

```env
PORT=5000
MONGODB_URI=mongodb://localhost:27017/lost_and_found
JWT_SECRET=changez-moi
JWT_EXPIRES_IN=1h
BCRYPT_ROUNDS=12
CLIENT_URL=http://localhost:5173
MATCH_THRESHOLD=50
```

### Frontend

```bash
cd frontend
npm install
cp .env.example .env     # VITE_API_URL=http://localhost:5000/api
npm run dev              # http://localhost:5173
```

### Comptes de démonstration (après `npm run seed`)

| Rôle | E-mail | Mot de passe |
|---|---|---|
| Administrateur | admin@campus.test | *(voir seed)* |
| Utilisateur | alice@campus.test | *(voir seed)* |
| Utilisateur | bob@campus.test | *(voir seed)* |

---

## 5. Modèle de données MongoDB

Cinq collections principales. Les choix de référence vs. embarqué sont justifiés ci-dessous.

### `users`

| Champ | Type | Remarques |
|---|---|---|
| `email` | String | unique, normalisé en minuscules |
| `passwordHash` | String | jamais renvoyé par l'API (`select: false`) |
| `firstName`, `lastName` | String | |
| `role` | `user` \| `admin` | défaut `user` |
| `status` | `active` \| `suspended` \| `deleted` | suppression logique |
| `createdAt`, `updatedAt` | Date | |

### `declarations`

| Champ | Type | Remarques |
|---|---|---|
| `type` | `lost` \| `found` | |
| `title` | String | obligatoire |
| `description` | String | |
| `category` | String (enum) | obligatoire, filtre dur du matching |
| `color`, `brand` | String | optionnels |
| `location` | `{ zone, details }` | `zone` issue de la liste des lieux du campus |
| `eventDate` | Date | date de perte / de découverte, obligatoire, non future |
| `photos` | [String] | URLs |
| `privateDetails` | String | signe distinctif connu seulement du propriétaire (**jamais exposé publiquement**) |
| `heldAt` | String | où se trouve l'objet trouvé (ex. « accueil bibliothèque ») |
| `status` | enum | voir [cycle de vie](#7-cycle-de-vie-des-déclarations) |
| `author` | ObjectId → `users` | |
| `createdAt`, `updatedAt` | Date | |

### `matches`

| Champ | Type | Remarques |
|---|---|---|
| `lost` | ObjectId → `declarations` | |
| `found` | ObjectId → `declarations` | |
| `score` | Number (0–100) | |
| `confidence` | `high` \| `medium` \| `low` | |
| `scoreDetails` | Object | contribution de chaque critère (explicabilité) |
| `status` | `proposed` \| `accepted` \| `rejected` \| `to_verify` \| `resolved` \| `expired` | |
| `reactions` | `{ lostOwner, finder }` | dernière réaction de chaque partie |
| `createdAt`, `updatedAt` | Date | |

Index unique sur `{ lost, found }` : une même paire n'est jamais proposée deux fois.

### `matchevents` (historique)

Collection séparée, en ajout seul (*append-only*), pour ne pas faire grossir indéfiniment le document `match`.

| Champ | Type | Remarques |
|---|---|---|
| `match` | ObjectId | |
| `type` | enum | `created`, `proposed`, `accepted`, `rejected`, `to_verify`, `feedback`, `status_changed`, `recomputed`, `returned` |
| `actor` | ObjectId \| null | `null` = système |
| `data` | Object | score avant/après, message, ancien/nouveau statut… |
| `createdAt` | Date | |

### `notifications`

| Champ | Type |
|---|---|
| `user` | ObjectId |
| `type` | enum (`new_match`, `match_accepted`, `match_rejected`, `status_changed`, `report_handled`, `item_returned`…) |
| `payload` | Object (ids liés, message) |
| `readAt` | Date \| null |
| `createdAt` | Date |

### `reports`

| Champ | Type |
|---|---|
| `declaration` | ObjectId |
| `reporter` | ObjectId |
| `reason` | `fake` \| `inappropriate` \| `duplicate` \| `incorrect_info` \| `personal_data` \| `other` |
| `comment` | String |
| `status` | `open` \| `dismissed` \| `actioned` |
| `handledBy`, `handledAt`, `resolutionNote` | |

### Index

| Collection | Index | Raison |
|---|---|---|
| `declarations` | `{ type: 1, status: 1, category: 1, eventDate: -1 }` | requête de recherche la plus fréquente |
| `declarations` | `{ "location.zone": 1, eventDate: -1 }` | filtre par lieu et période |
| `declarations` | index texte sur `title`, `description`, `brand` | recherche plein texte |
| `declarations` | `{ author: 1, createdAt: -1 }` | « mes déclarations » |
| `matches` | `{ lost: 1, found: 1 }` (unique) | anti-doublon |
| `matches` | `{ status: 1, createdAt: -1 }` | suivi et stats |
| `matchevents` | `{ match: 1, createdAt: 1 }` | lecture chronologique de l'historique |
| `notifications` | `{ user: 1, readAt: 1, createdAt: -1 }` | liste et compteur de non lues |
| `reports` | `{ status: 1, createdAt: -1 }` | file de modération |
| `users` | `{ email: 1 }` (unique) | connexion |

---

## 6. Rôles et permissions

| Action | Visiteur | Utilisateur | Auteur | Admin |
|---|:-:|:-:|:-:|:-:|
| Consulter les déclarations publiques (sans champs privés) | ✅ | ✅ | ✅ | ✅ |
| Créer une déclaration | ❌ | ✅ | — | ✅ |
| Modifier une déclaration | ❌ | ❌ | ✅ (si non résolue) | ✅ |
| Supprimer une déclaration | ❌ | ❌ | ✅ (si aucun match accepté) | ✅ |
| Marquer comme récupérée | ❌ | ❌ | ✅ | ✅ |
| Réagir à une correspondance | ❌ | ❌ | ✅ (parties concernées) | ✅ |
| Signaler une déclaration | ❌ | ✅ | — | ✅ |
| Masquer / restaurer / traiter un signalement | ❌ | ❌ | ❌ | ✅ |
| Suspendre un utilisateur | ❌ | ❌ | ❌ | ✅ |
| Voir les statistiques | ❌ | ❌ | ❌ | ✅ |

---

## 7. Cycle de vie des déclarations

```
        ┌──────── open ────────┐
        │          │           │
     (modération)  │ match     │ auteur clôt / expiration
        ▼          ▼ accepté   ▼
     hidden     in_progress   closed
        │          │
   (restauration)  │ remise confirmée
        ▼          ▼
       open     returned
```

| Statut | Signification | Qui peut le changer |
|---|---|---|
| `open` | Visible et éligible au matching | Auteur, système |
| `in_progress` | Une correspondance a été acceptée, remise en cours | Système |
| `returned` | Objet restitué à son propriétaire | Auteur, admin |
| `closed` | Clôturée sans restitution (abandon, expiration) | Auteur, admin, système |
| `hidden` | Masquée par la modération | Admin uniquement |

---

## 8. Binôme 1 — Utilisateurs, déclarations, sécurité

### Endpoints

| Méthode | Route | Description | Accès |
|---|---|---|---|
| POST | `/api/auth/register` | Création de compte | Public |
| POST | `/api/auth/login` | Connexion, retourne un JWT | Public |
| GET | `/api/auth/me` | Utilisateur connecté | Connecté |
| PATCH | `/api/users/me` | Modifier son profil | Connecté |
| DELETE | `/api/users/me` | Supprimer son compte | Connecté |
| POST | `/api/declarations` | Créer une déclaration | Connecté |
| GET | `/api/declarations/:id` | Consulter une déclaration | Public (champs filtrés) |
| PATCH | `/api/declarations/:id` | Modifier | Auteur / admin |
| DELETE | `/api/declarations/:id` | Supprimer | Auteur / admin |
| GET | `/api/declarations/mine` | Mes déclarations | Connecté |
| PATCH | `/api/declarations/:id/status` | Changer le statut | Selon règles |

### Décisions

- **Champs obligatoires :** `type`, `title`, `category`, `location.zone`, `eventDate`.
- **Modification :** autorisée tant que la déclaration n'est pas `returned` ou `closed`. Une modification significative (catégorie, lieu, date, couleur) **déclenche un recalcul** des correspondances (voir cas 3 en section 10).
- **Suppression d'une déclaration :** suppression logique. Si elle a une correspondance acceptée, la suppression est refusée (il faut d'abord annuler ou clôturer la correspondance).
- **Suppression d'un compte :** le compte passe à `deleted`, les données personnelles (nom, e-mail) sont **anonymisées**. Les déclarations `open` sont `closed`, les correspondances en cours passent à `expired` et l'autre partie est notifiée. Les déclarations `returned` sont conservées de façon anonyme pour les statistiques.
- **Données privées :** `privateDetails`, e-mail et identité de l'auteur ne sont jamais renvoyés dans les réponses publiques. L'identité n'est révélée à l'autre partie qu'**après acceptation mutuelle** d'une correspondance.

### Sécurité

- Mots de passe hachés avec `bcrypt` (coût 12), politique minimale de mot de passe.
- JWT signé, expiration courte, vérifié par un middleware `authenticate`.
- Middleware `authorize(...roles)` et vérifications de propriété (`req.user.id === declaration.author`).
- `helmet` (en-têtes HTTP), `cors` restreint à `CLIENT_URL`.
- `express-rate-limit` sur `/auth/*` (anti-force brute) et sur les créations.
- `express-mongo-sanitize` contre l'injection d'opérateurs NoSQL (`$ne`, `$gt`…).
- Limitation de la taille des corps de requête, liste blanche des champs modifiables (pas de *mass assignment* sur `role` ou `status`).
- Messages d'erreur d'authentification génériques (pas d'énumération de comptes).
- Compte `suspended` : connexion refusée.

---

## 9. Binôme 2 — Recherche et exploration

### Endpoint

`GET /api/declarations`

### Filtres supportés

| Paramètre | Exemple | Remarque |
|---|---|---|
| `type` | `found` | `lost` / `found` |
| `category` | `bag` | |
| `color` | `black` | insensible à la casse |
| `zone` | `library` | lieu du campus |
| `from`, `to` | `2026-10-01` | période sur `eventDate` |
| `period` | `week` | raccourci : `today`, `week`, `month` |
| `q` | `sac à dos` | recherche plein texte |
| `status` | `open` | défaut : `open` uniquement |

Exemple correspondant à l'énoncé :
`/api/declarations?type=found&category=bag&color=black&zone=library&period=week`

### Tri

`sort=recent` (défaut) · `sort=oldest` · `sort=relevance` (si `q` fourni, score texte MongoDB)

### Pagination

- **Par défaut : `page` + `limit`** (limit plafonné à 50), avec `total`, `totalPages`, `hasNext` dans la réponse.
- Pour les gros volumes, **pagination par curseur** (`cursor` = `{eventDate, _id}` du dernier élément) : performance constante, là où `skip(n)` devient coûteux.
- Projection : la liste ne renvoie que les champs nécessaires à l'affichage (pas de description complète).

### Montée en charge

| Volume | Stratégie |
|---|---|
| 20 – 500 | Index simples, `page/limit` suffisent |
| 10 000 | Index composé, projection, `limit` plafonné, filtre `status: open` par défaut |
| 100 000 + | Pagination par curseur, `count` approximatif ou omis, archivage des déclarations anciennes, envisager Atlas Search |

Les requêtes critiques sont vérifiées avec `explain("executionStats")` pour s'assurer qu'elles utilisent bien un index (`IXSCAN`, pas `COLLSCAN`).

### Recherche sans résultat

Réponse `200` avec une liste vide et un champ `suggestions` (ex. « élargir la période », « retirer le filtre couleur »). Côté interface : message explicite et bouton « Créer une déclaration pour être alerté ».

---

## 10. Binôme 3 — Matching et résolution

### Déclenchement

Le matching est exécuté :

- à la **création** d'une déclaration (comparée aux déclarations du type opposé, `open`) ;
- à la **modification significative** d'une déclaration ;
- via une **tâche planifiée** quotidienne (rattrapage et expiration des vieilles correspondances).

### Algorithme : filtre dur + score pondéré

**1. Filtres éliminatoires (candidats)**

- types opposés (`lost` ↔ `found`), statut `open` ;
- **même catégorie** ;
- date de découverte **≥ date de perte** (on ne trouve pas un objet avant de le perdre) ;
- auteurs différents.

**2. Score sur 100**

| Critère | Poids | Calcul |
|---|---:|---|
| Lieu | 25 | même zone = 25 · zone voisine = 12 · sinon 0 |
| Date | 20 | décroissance avec l'écart de jours entre perte et découverte |
| Couleur | 15 | identique = 15 · proche = 7 · inconnue = neutre |
| Texte (titre + description) | 20 | similarité sur mots-clés normalisés (minuscules, sans accents, sans mots vides) |
| Marque | 10 | identique = 10 · inconnue = neutre |
| Détails privés | 10 | cohérence entre l'indice du propriétaire et la description du trouveur |

**3. Niveaux de confiance**

| Score | Confiance | Action |
|---|---|---|
| ≥ 75 | `high` | Correspondance créée, les deux parties notifiées |
| 50 – 74 | `medium` | Correspondance créée, proposée avec mention « à vérifier » |
| < 50 | — | Ignorée |

Chaque match stocke `scoreDetails` : on peut toujours expliquer *pourquoi* le système a proposé cette paire.

> **Justification :** un filtre dur évite des calculs inutiles et des faux positifs évidents ; le score pondéré donne de la souplesse (un lieu un peu différent ne disqualifie pas) ; les poids sont des constantes configurables, ajustables avec les retours utilisateurs.

### Réactions et états d'une correspondance

```
proposed ──▶ accepted (par les deux parties) ──▶ resolved
   │  │
   │  └──▶ to_verify (infos complémentaires demandées) ──▶ proposed / rejected
   └─────▶ rejected
               │
   (nouvelle information significative) ──▶ recalcul
   expired (aucune réponse, objet clôturé, compte supprimé)
```

| Endpoint | Description |
|---|---|
| GET `/api/matches` | Mes correspondances |
| GET `/api/matches/:id` | Détail + score détaillé |
| POST `/api/matches/:id/accept` | Accepter |
| POST `/api/matches/:id/reject` | Refuser (motif optionnel) |
| POST `/api/matches/:id/verify` | Demander des vérifications |
| POST `/api/matches/:id/feedback` | Ajouter un commentaire |
| POST `/api/matches/:id/resolve` | Confirmer la restitution |
| GET `/api/matches/:id/history` | Historique chronologique |

### Historique

Chaque événement important est écrit dans `matchevents` (création, proposition, acceptation, refus, feedback, changement de statut, recalcul, restitution), avec l'acteur, la date et les données utiles. Un administrateur ou un utilisateur qui revient plusieurs jours après peut relire la **chronologie complète** et comprendre ce qui s'est passé sans deviner.

### Cas particuliers

| # | Cas | Règle retenue |
|---|---|---|
| 1 | Deux objets similaires (ex. deux sacs noirs) | Le score seul ne suffit pas : le **champ `privateDetails`** et un échange de vérification (`to_verify`) sont nécessaires avant que les identités soient révélées. Une correspondance `high` n'est jamais automatiquement acceptée. |
| 2 | Un objet perdu, plusieurs objets trouvés | Toutes les correspondances ≥ seuil sont conservées et affichées **triées par score**, avec le détail des critères. L'utilisateur en traite une à la fois ; refuser n'empêche pas les autres. |
| 3 | Refus puis nouvelle information | Si la modification porte sur un champ significatif (catégorie, lieu, date, couleur, marque, détails privés), le match refusé est **réévalué** ; s'il dépasse le seuil avec les nouvelles données, il repasse en `proposed` et l'événement `recomputed` est tracé. |
| 4 | Deux personnes revendiquent le même objet | Plusieurs matchs peuvent exister, mais **un seul peut passer en `accepted`** côté trouveur. Les autres sont mis `to_verify` en attente. Le trouveur ou un admin tranche à partir des détails privés. |
| 5 | Correspondance tardive | Le matching est toujours exécuté, mais la composante date est plafonnée : au-delà de **60 jours**, le score ne peut plus atteindre `high`. Une déclaration `open` de plus de **90 jours** est proposée à la clôture. |
| 6 | Objet récupéré | La déclaration passe en `returned`, le match en `resolved`, les **autres matchs possibles passent en `expired`**, leurs auteurs sont notifiés, et la déclaration de l'autre partie est clôturée. |

---

## 11. Binôme 4 — Administration, modération, statistiques

### Signalements

| Endpoint | Description | Accès |
|---|---|---|
| POST `/api/declarations/:id/reports` | Signaler une déclaration | Connecté |
| GET `/api/admin/reports` | Liste filtrable par statut | Admin |
| PATCH `/api/admin/reports/:id` | Traiter (rejeter ou donner suite) | Admin |

**Règles :**

- Un utilisateur ne peut signaler **qu'une fois** une même déclaration.
- Un signalement ne masque rien automatiquement (évite les abus de signalement) ; au-delà d'un seuil de signalements distincts, la déclaration est **marquée prioritaire** dans la file.
- Un signalement jugé non fondé est `dismissed` : la déclaration reste visible, l'auteur n'est pas pénalisé.

### Modération

| Endpoint | Description |
|---|---|
| PATCH `/api/admin/declarations/:id/hide` | Masquer (motif obligatoire) |
| PATCH `/api/admin/declarations/:id/restore` | Restaurer |
| DELETE `/api/admin/declarations/:id` | Supprimer |
| PATCH `/api/admin/users/:id/suspend` | Suspendre (durée / motif) |
| PATCH `/api/admin/users/:id/reactivate` | Réactiver |
| GET `/api/admin/users` | Liste des utilisateurs |

Toute action de modération est journalisée (qui, quoi, quand, motif) et notifie la personne concernée.

### Statistiques

`GET /api/admin/stats` — calculées avec des **pipelines d'agrégation MongoDB** (`$group`, `$match`, `$facet`, `$dateTrunc`).

| Indicateur | Source |
|---|---|
| Nombre d'utilisateurs (actifs / suspendus) | `users` |
| Déclarations perdues / trouvées | `declarations` groupées par `type` |
| Correspondances proposées / acceptées / refusées | `matches` groupées par `status` |
| Taux de restitution | `returned` / total |
| Délai moyen avant restitution | écart `createdAt` → date de résolution |
| Signalements ouverts / traités | `reports` |
| Évolution par jour / semaine | `$dateTrunc` sur `createdAt` |
| Top catégories et zones | `$group` + `$sort` + `$limit` |

---

## 12. Notifications

| Événement | Destinataire(s) |
|---|---|
| Nouvelle correspondance | Les deux auteurs |
| Correspondance acceptée | L'autre partie |
| Correspondance refusée | L'autre partie (sans exposer le motif détaillé) |
| Vérification demandée / feedback | L'autre partie |
| Changement de statut d'une déclaration | Auteur |
| Objet récupéré | Les parties des autres matchs concernés |
| Signalement traité | Le rapporteur et l'auteur de la déclaration |
| Déclaration masquée / compte suspendu | Utilisateur concerné |

**Stockage :** collection `notifications`, un document par destinataire.
**Lecture :** `readAt: null` = non lue ; `PATCH /api/notifications/:id/read` et `PATCH /api/notifications/read-all`.
**Endpoints :** `GET /api/notifications` (paginé, filtre `unread`), `GET /api/notifications/unread-count`.
**Côté front :** pastille de compteur dans l'en-tête, rafraîchie à intervalle régulier (polling).

---

## 13. Validation et gestion des erreurs

### Validation

Toutes les entrées (corps, paramètres, query string) sont validées par un middleware **avant** d'atteindre les contrôleurs :

- types, longueurs, formats (e-mail, ObjectId, dates) ;
- énumérations (type, catégorie, zone, statut) ;
- champs inconnus rejetés ou retirés ;
- dates cohérentes (pas dans le futur) ;
- bornes sur la pagination (`limit` ≤ 50) ;
- nettoyage des chaînes (trim, échappement) pour limiter les injections et le XSS stocké.

La validation est doublée côté Mongoose, et la validation côté React n'est qu'un confort : **le backend ne s'y fie jamais**.

### Format d'erreur unique

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Données invalides",
    "details": [
      { "field": "eventDate", "message": "La date ne peut pas être dans le futur" }
    ]
  }
}
```

| HTTP | Code | Cas |
|---|---|---|
| 400 | `VALIDATION_ERROR` / `BAD_REQUEST` | Données invalides |
| 401 | `UNAUTHENTICATED` | Jeton absent ou invalide |
| 403 | `FORBIDDEN` | Droits insuffisants, compte suspendu |
| 404 | `NOT_FOUND` | Ressource inexistante |
| 409 | `CONFLICT` | Doublon, transition d'état interdite |
| 429 | `TOO_MANY_REQUESTS` | Limite de requêtes dépassée |
| 500 | `INTERNAL_ERROR` | Erreur inattendue (détails jamais exposés au client) |

Un middleware d'erreurs centralisé et une classe `AppError` garantissent des réponses cohérentes ; les erreurs 500 sont journalisées côté serveur uniquement.

---

## 14. API, documentation et Postman

- **Documentation :** Swagger / OpenAPI disponible sur `http://localhost:5000/api/docs` (fichier source dans `docs/`).
- **Collections Postman :** dossier [`postman/`](./postman) — une collection par binôme, plus une collection de **scénario de soutenance** exécutable de bout en bout.
- **Environnements :** `local.postman_environment.json` (variables `baseUrl`, `tokenUserA`, `tokenUserB`, `tokenAdmin`, `lostId`, `foundId`, `matchId`).
- Les requêtes enregistrent automatiquement les identifiants et jetons dans les variables (scripts *Tests*), et contiennent des assertions (code HTTP, structure de réponse).

---

## 15. Scénario de démonstration

Exécutable via la collection Postman `Scenario — Soutenance` ou depuis l'interface.

| Étape | Action | Résultat attendu |
|---|---|---|
| 1 | Alice se connecte et déclare un **sac à dos noir perdu** lundi à la bibliothèque | Déclaration `open` |
| 2 | Bob se connecte et déclare un **sac noir trouvé** mardi près de la bibliothèque | Déclaration `open` |
| 3 | Le système calcule le score | Match créé (score ≈ 80, confiance `high`) |
| 4 | Alice et Bob reçoivent une notification | `unread-count` = 1 |
| 5 | Alice demande une vérification (« il y a un porte-clés rouge ») | Match `to_verify`, événement tracé |
| 6 | Bob confirme et accepte | Match `accepted`, identités révélées |
| 7 | Alice et Bob acceptent, la remise a lieu, Alice confirme | Match `resolved`, déclarations `returned` |
| 8 | Les autres matchs éventuels | Passés en `expired`, auteurs notifiés |
| 9 | Un utilisateur signale une déclaration, l'admin la traite | Signalement `dismissed` ou `actioned` |
| 10 | L'admin consulte le tableau de bord | Statistiques à jour |

---

## 16. Cas non prévus et décisions

| Situation | Décision |
|---|---|
| Utilisateur qui supprime son compte | Anonymisation, clôture des déclarations ouvertes, expiration des matchs, notification des contreparties |
| Déclaration supprimée avec une correspondance | Refusée si le match est `accepted` ; sinon match `expired` et notification |
| Plusieurs utilisateurs revendiquent le même objet | Voir cas 4 : un seul match accepté, vérification par détails privés |
| Déclaration signalée alors qu'elle est légitime | Signalement `dismissed`, aucune sanction automatique |
| Modification importante d'une déclaration | Recalcul des matchs, événement `recomputed` |
| Objet retrouvé très longtemps après la perte | Matching autorisé mais score plafonné (cas 5) |
| Énorme volume de données | Index, projections, pagination par curseur, archivage |
| Information personnelle publiée par erreur | Signalement `personal_data` traité en priorité ; l'admin masque immédiatement, l'auteur peut modifier |

---

## 17. Fonctionnalités supplémentaires

> **📝 À compléter** uniquement avec ce que vous implémentez réellement, en expliquant **le problème que chaque fonctionnalité résout**.

| Fonctionnalité | Problème résolu |
|---|---|
| Détails privés de vérification | Empêche une personne de réclamer un objet qui n'est pas le sien |
| Score détaillé et explicable | Aide l'utilisateur à juger si la proposition est crédible |
| Photos | Réduit l'ambiguïté entre objets similaires |
| *(autre)* | |

---

## 👥 Équipe

| Binôme | Domaine | Membres |
|---|---|---|
| 1 | Utilisateurs, déclarations, sécurité | *Prénom NOM*, *Prénom NOM* |
| 2 | Recherche et exploration | *Prénom NOM*, *Prénom NOM* |
| 3 | Matching et résolution | *Prénom NOM*, *Prénom NOM* |
| 4 | Administration, modération, statistiques | *Prénom NOM*, *Prénom NOM* |

---

## Licence

Projet pédagogique — *à préciser*.
