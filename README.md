# Relational Architecture

Skill pour Claude Code qui genere une architecture relationnelle complete pour n'importe quel projet de developpement.

## Ce que ca fait

A partir du code source d'un projet, ce skill :

1. **Liste** toutes les fonctionnalites, pages, API routes, hooks, composants et utilitaires
2. **Classifie** chaque element selon 5 niveaux (Core / Feature / Sub / Hook / Util)
3. **Relie** les elements avec 4 types de relations (`utilise`, `declenche`, `depend de`, `persiste dans`)
4. **Documente** le tout dans des fichiers modulaires (`docs/`)

## Pourquoi

- Les LLM (Claude Code, Cursor, etc.) comprennent mieux un projet quand les dependances sont explicites
- Evite les bugs en cascade en identifiant la zone d'impact avant chaque modification
- Fonctionne avec tout framework (Next.js, React, Vue, Python, etc.) et tout type de DB

## Installation

```bash
npx skills add AdChaine/relational-architecture
```

## Utilisation

Dans Claude Code :

```
/relational-architecture
```

## Fichiers generes

| Fichier | Contenu |
|---------|---------|
| `docs/architecture.md` | Arbre des fonctionnalites + table des relations |
| `docs/database.md` | Schema DB, tables, RLS, migrations |
| `docs/api-routes.md` | Endpoints API, params, reponses |
| `docs/decisions.md` | Journal des choix techniques (ADR) |
| `docs/data-flow.md` | Flux de donnees entre services |
| `docs/components.md` | Index des composants UI |
| `docs/hooks.md` | Index des hooks custom |
| `docs/README.md` | Index routeur |

## Niveaux de classification

| Niveau | Role | Risque si modifie |
|--------|------|-------------------|
| **Core** | Infrastructure fondamentale | Critique |
| **Feature** | Fonctionnalite utilisateur visible | Modere |
| **Sub** | Sous-composant UI | Faible |
| **Hook** | Logique / gestion d'etat | Modere |
| **Util** | Fonction pure | Tres faible |

## Licence

MIT
