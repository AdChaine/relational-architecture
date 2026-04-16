# Format des livrables — Architecture Relationnelle

Ce document définit le format exact de chaque fichier à générer.

---

## 1. `docs/architecture.md` — Fichier principal

```markdown
# Architecture Relationnelle — [Nom du projet]

> Généré le [DATE] | Stack : [stack technique]
> Backend : [framework] | DB : [database] | LLM : [si applicable] | Deploy : [platform]

## Légende des niveaux

- 🔴 **Core** — Infrastructure critique (si ça tombe, tout meurt)
- 🟠 **Feature** — Fonctionnalité utilisateur visible
- 🟡 **Sub** — Sous-composant UI (affichage uniquement)
- 🔵 **Hook** — Logique / gestion d'état
- ⚪ **Util** — Fonction pure (entrée → sortie)

## Légende des relations

- `utilise →` — Importe ou consomme
- `déclenche →` — Provoque l'exécution de
- `dépend de →` — Ne fonctionne pas sans
- `persiste dans →` — Lit ou écrit des données dans

---

## Arbre des fonctionnalités

🔴 Core: Authentification
├── 🟠 Feature: Login Flow
│   ├── 🟡 Sub: Formulaire Email/Password
│   │   └── ⚪ Util: validateEmail()
│   └── 🟡 Sub: Bouton OAuth Google
├── 🔵 Hook: useAuthSession
│   utilise → Core: Authentification
│   déclenche → Feature: Dashboard, Feature: Notifications
└── ⚪ Util: tokenRefresh()
    utilise → Core: Authentification

🔴 Core: Base de données
├── 🟠 Feature: Dashboard
│   ├── dépend de → Core: Authentification
│   ├── 🟡 Sub: StatsCard
│   ├── 🟡 Sub: ChartRevenu
│   └── 🔵 Hook: useDashboardData
│       utilise → Core: Base de données
│       persiste dans → DB: analytics_cache
...
```

Règles de l'arbre :
- Indentation de 4 espaces ou caractères de branchement (`├──`, `└──`, `│`)
- Chaque élément préfixé par son emoji de niveau
- Les relations sont insérées sous l'élément concerné avec indentation supplémentaire
- Numérotation implicite par la hiérarchie (pas de numéros explicites requis)

---

## Table des relations

Sous l'arbre, inclure une table exhaustive :

```markdown
## Table des relations

| # | Source | Relation | Destination | Fichiers impactés |
|---|--------|----------|-------------|-------------------|
| 1 | Hook: useAuthSession | utilise → | Core: Auth Provider | `hooks/useAuth.ts`, `lib/auth.ts` |
| 2 | Hook: useAuthSession | déclenche → | Feature: Dashboard | `hooks/useAuth.ts`, `pages/dashboard.tsx` |
| 3 | Feature: Dashboard | dépend de → | Core: Authentification | `pages/dashboard.tsx`, `middleware.ts` |
| 4 | Feature: NoteEditor | persiste dans → | DB: notes | `components/NoteEditor.tsx`, `api/notes.ts` |
| ... | ... | ... | ... | ... |
```

La colonne "Fichiers impactés" est critique — elle permet à l'IA de savoir exactement quels fichiers auditer en cas de modification.

---

## 2. `docs/database.md`

```markdown
# Schéma Base de Données — [Nom du projet]

> Généré le [DATE] | DB : [PostgreSQL/Supabase/MySQL/etc.]

## Tables

### `users`
| Colonne | Type | Contraintes | Description |
|---------|------|-------------|-------------|
| id | uuid | PK, default gen_random_uuid() | Identifiant unique |
| email | text | UNIQUE, NOT NULL | Email de connexion |
| created_at | timestamptz | default now() | Date de création |

### `notes`
...

## Relations entre tables

| Table source | Relation | Table cible | Clé |
|-------------|----------|-------------|-----|
| notes | N:1 | users | notes.user_id → users.id |
| ... | ... | ... | ... |

## Politiques RLS (si Supabase)

| Table | Policy | Règle |
|-------|--------|-------|
| notes | select_own | auth.uid() = user_id |
| ... | ... | ... |

## Migrations récentes

| Date | Migration | Description |
|------|-----------|-------------|
| 2026-03-20 | add_notes_table | Création table notes avec RLS |
| ... | ... | ... |
```

---

## 3. `docs/api-routes.md`

```markdown
# Routes API — [Nom du projet]

> Généré le [DATE]

## Index

| Méthode | Route | Description | Auth requise |
|---------|-------|-------------|--------------|
| POST | /api/analyze | Analyse un batch via LLM | ✅ |
| GET | /api/transcripts | Liste les transcripts | ✅ |
| ... | ... | ... | ... |

## Détail

### POST `/api/analyze`
- **Auth** : Bearer token requis
- **Body** : `{ chunks: string[], model?: string }`
- **Réponse** : `{ results: AnalysisResult[] }`
- **Dépend de** : Core: OpenRouter API, Core: Auth
- **Persiste dans** : DB: analysis_results
- **Fichier** : `app/api/analyze/route.ts`

### GET `/api/transcripts`
...
```

---

## 4. `docs/decisions.md` — Journal ADR

```markdown
# Journal des Décisions Architecturales — [Nom du projet]

> Dernière mise à jour : [DATE]

## ADR-001 : [Titre de la décision]

- **Date** : YYYY-MM-DD
- **Statut** : Accepté | Rejeté | Remplacé par ADR-XXX
- **Contexte** : [Quel problème fallait-il résoudre ?]
- **Décision** : [Qu'avons-nous choisi ?]
- **Alternatives rejetées** :
  - [Option A] — Rejeté car [raison factuelle]
  - [Option B] — Rejeté car [raison factuelle]
- **Conséquences** : [Impacts sur le projet]

---

## ADR-002 : ...
```

Si un fichier `decisions.md` existe déjà, le conserver et l'enrichir (ne pas écraser les entrées existantes).

---

## 5. `docs/data-flow.md`

```markdown
# Flux de Données — [Nom du projet]

> Généré le [DATE]

## Pipeline principal

```
[Utilisateur]
  → Frontend (Next.js)
    → API Route /api/analyze
      → Service: OpenRouter
        → LLM (Claude/Gemini)
      ← Réponse structurée
    → Supabase (INSERT analysis_results)
  ← Affichage Dashboard
```

## Flux d'authentification

```
[Utilisateur]
  → Login Page
    → NextAuth / Clerk
      → OAuth Provider (Google/GitHub)
    ← Session JWT
  → Middleware (vérifie session)
    → Pages protégées
```

## Flux de [autre fonctionnalité]
...
```

---

## 6. `docs/components.md` et `docs/hooks.md`

Format identique — un index rapide :

```markdown
# Composants principaux — [Nom du projet]

> Généré le [DATE]

| Composant | Fichier | Type | Dépend de | Utilisé par |
|-----------|---------|------|-----------|-------------|
| StatsCard | components/StatsCard.tsx | Sub | Hook: useDashboardData | Feature: Dashboard |
| NoteEditor | components/NoteEditor.tsx | Sub | Hook: useNotes | Feature: Notes |
| ... | ... | ... | ... | ... |
```

---

## 7. `docs/README.md` — Index routeur

```markdown
# Documentation — [Nom du projet]

> Dernière mise à jour : [DATE]

## Index

| Fichier | Contenu | Quand le consulter |
|---------|---------|-------------------|
| `architecture.md` | Arbre des fonctionnalités + table des relations | Avant TOUTE modification. Point d'entrée principal. |
| `database.md` | Schéma DB, tables, RLS, migrations | Modification de données, ajout de table, query |
| `api-routes.md` | Endpoints API, params, réponses | Modification ou ajout de route API |
| `decisions.md` | Journal des choix techniques (ADR) | Avant de proposer un changement d'architecture |
| `data-flow.md` | Flux de données entre services | Debug, ajout d'intégration externe |
| `components.md` | Index des composants UI | Modification UI, ajout de composant |
| `hooks.md` | Index des hooks custom | Modification logique métier, état |
```

---

## 8. Mise à jour du CLAUDE.md (routeur)

Ajouter ou remplacer la section documentation dans le CLAUDE.md existant :

```markdown
## Documentation du projet

La documentation complète est dans `docs/`. Consulter l'index : `docs/README.md`

### Règles obligatoires avant toute modification

1. **Lire `docs/architecture.md`** avant de modifier une fonctionnalité — vérifier les dépendances dans la table des relations
2. **Lire `docs/decisions.md`** avant de proposer un changement d'architecture — vérifier qu'on ne reproduit pas une erreur passée
3. **Lire le fichier docs/ pertinent** selon le type de modification (DB → database.md, API → api-routes.md, etc.)
4. **Mettre à jour la documentation** après chaque modification significative (nouvelle feature, changement de dépendance, nouvelle décision)

### Zone d'impact

Avant de modifier un fichier, consulter la colonne "Fichiers impactés" dans `docs/architecture.md` pour identifier la zone d'impact (blast radius) et auditer les fichiers connexes après modification.
```

Ne pas écraser le reste du CLAUDE.md — uniquement ajouter/remplacer la section documentation.
