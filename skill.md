---
name: relational-architecture
description: >
  Génère une architecture relationnelle complète pour un projet de développement (SaaS, app client, outil interne).
  Produit un arbre indenté des fonctionnalités classifiées (Core/Feature/Sub/Hook/Util), une table des dépendances
  explicites (utilise/déclenche/dépend de/persiste dans), un schéma DB, un flux de données, et un CLAUDE.md routeur.
  Déclencher ce skill dès que l'utilisateur mentionne : architecture de projet, documentation de projet, dépendances
  entre features, bugs en cascade, "mon projet casse quand je modifie X", cartographie de fonctionnalités, arbre de
  dépendances, refactoring de documentation, structurer un SaaS, documenter pour Claude Code, éviter les régressions,
  split de documentation, améliorer le CLAUDE.md, ou tout projet avec 3+ fonctionnalités qui nécessite une documentation
  structurée. Utiliser aussi quand l'utilisateur démarre un nouveau projet et veut poser des fondations solides, ou
  quand un projet existant souffre de documentation insuffisante. Ce skill s'applique à tout framework (Next.js, React,
  Vue, Node, Python, etc.) et tout type de base de données.
user_invocable: true
---

# Architecture Relationnelle

Méthode de documentation structurée qui cartographie chaque fonctionnalité d'un projet, ses dépendances et ses relations, afin que tout LLM (Claude Code, Cursor, etc.) comprenne l'environnement global et évite les bugs en cascade.

## Concepts fondamentaux

Cette méthode repose sur 4 concepts d'ingénierie logicielle éprouvés, adaptés au travail avec des agents IA :

1. **Dependency Mapping** — Cartographier "qui utilise quoi", "qui déclenche quoi", "qui dépend de quoi"
2. **Separation of Concerns** — Classifier chaque élément selon son rôle (Core/Feature/Sub/Hook/Util)
3. **Architecture Decision Records (ADR)** — Documenter le "pourquoi" des choix techniques
4. **Documentation Modulaire** — Fichiers ciblés avec un index/routeur central plutôt qu'un monolithe

## Workflow en 4 étapes

### Étape 1 : LISTER

Analyser l'intégralité du projet (code source, pas seulement la doc existante). Identifier TOUTES les fonctionnalités, pages, API routes, hooks, composants et utilitaires. Forcer une analyse nouvelle même si un CLAUDE.md ou une documentation existe déjà — ne jamais se baser uniquement sur la doc existante qui peut être obsolète ou incomplète.

Livrable : Liste exhaustive de tous les éléments fonctionnels du projet.

### Étape 2 : CLASSIFIER

Attribuer à chaque élément un des 5 niveaux :

| Niveau | Rôle | Risque si modifié | Exemples |
|--------|------|-------------------|----------|
| **Core** | Infrastructure fondamentale. Si ça tombe, tout le projet est mort. | Critique | Config DB (Supabase/Postgres), Auth provider (NextAuth/Clerk), Stripe SDK, routeur global, middleware auth |
| **Feature** | Fonctionnalité visible par l'utilisateur. Assemble des Sub, Hooks et Utils. | Modéré | Dashboard analytique, Générateur de résumés IA, Page facturation, Import de transcripts |
| **Sub** | Sous-composant UI. Reçoit des données, les affiche. Pas de logique métier. | Faible | BoutonTelechargerPDF, CarteUtilisateur, ChampRecherche, ModalConfirmation |
| **Hook** | Logique / état. Gère la mémoire applicative sans interface graphique. | Modéré | useFetchTranscripts, useTheme, usePaymentStatus, useAuthSession |
| **Util** | Fonction pure. Entrée → Sortie, sans état ni effet de bord. | Très faible | formatDateToFrench(), calculateTVA(), cleanHtmlTags(), slugify() |

### Étape 3 : RELIER

Mapper les dépendances avec exactement 4 types de relations :

| Relation | Signification | Exemple |
|----------|--------------|---------|
| `utilise →` | Fait appel à, importe, consomme | Login Flow **utilise →** Core: Auth |
| `déclenche →` | Provoque l'exécution de, émet un événement vers | onAuthChange **déclenche →** Dashboard + Notifications |
| `dépend de →` | Ne peut pas fonctionner sans | Dashboard **dépend de →** Auth |
| `persiste dans →` | Écrit ou lit des données dans | NoteEditor **persiste dans →** DB: notes_table |

Chaque relation est explicite, factuelle, sans ambiguïté. Vocabulaire froid et précis — pas de formulations vagues comme "le dashboard permet de faire cela".

### Étape 4 : DOCUMENTER

Produire la documentation modulaire avec les fichiers suivants dans le dossier `docs/` du projet :

1. **`docs/architecture.md`** — Arbre indenté + table des relations (fichier principal)
2. **`docs/database.md`** — Schéma des tables, relations, RLS policies
3. **`docs/api-routes.md`** — Endpoints, méthodes, paramètres, réponses
4. **`docs/decisions.md`** — Journal ADR des choix techniques (créé ou mis à jour)
5. **`docs/data-flow.md`** — Diagramme textuel du flux de données
6. **`docs/components.md`** — Index des composants UI principaux
7. **`docs/hooks.md`** — Index des hooks custom
8. **`docs/README.md`** — Index/sommaire de la documentation

Et mise à jour du **CLAUDE.md** (ou `.cursorrules`, selon l'outil) pour qu'il serve de routeur vers ces fichiers.

## Règles critiques

1. **Analyse forcée** — Toujours analyser le code source réel. Ne jamais se fier uniquement à la documentation existante.
2. **Zéro secrets** — Ne jamais inclure de clés API, URLs de webhook, tokens ou secrets dans la documentation. Utiliser des noms descriptifs : `API_OPENROUTER_KEY` et non la valeur.
3. **Température basse** — Textes factuels et froids. Pas de prose explicative dans l'arbre ou la table. Le format est conçu pour être lu par un LLM, pas pour être joli.
4. **Mise à jour du routeur** — Le CLAUDE.md doit pointer vers `docs/architecture.md` et les autres fichiers avec des instructions claires sur quand les consulter.
5. **Datation** — Chaque fichier généré inclut la date de génération en en-tête.

## Adaptation au projet

Avant de commencer, évaluer la taille du projet :

- **Petit** (< 3 features, < 20 fichiers) : Un seul `architecture.md` peut suffire, pas besoin de split
- **Moyen** (3-10 features, 20-80 fichiers) : Split complet recommandé
- **Gros** (10+ features, 80+ fichiers) : Split complet obligatoire + envisager un sous-dossier par module

Adapter aussi au framework :
- **Next.js/React** : Les 5 niveaux s'appliquent directement (Hook = React hooks custom)
- **Vue** : Hook → Composable, Sub → Component enfant
- **Python/Backend pur** : Hook → Service/Handler, Sub → Helper class
- **N8N/No-code** : Core → Workflow principal, Feature → Sous-workflow, Hook → Trigger/Webhook
