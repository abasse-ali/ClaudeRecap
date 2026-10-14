# 07 — Mémoire, contexte et sessions

[← Sommaire](README.md)

## Deux systèmes de mémoire

| | Fichiers `CLAUDE.md` | Mémoire automatique |
|---|---|---|
| **Qui l'écrit** | Vous | Claude |
| **Contenu** | Instructions et règles | Apprentissages et préférences |
| **Portée** | Organisation, utilisateur, projet, local | Par dépôt (partagée entre worktrees) |
| **Chargement** | À chaque session | À chaque session (200 premières lignes ou 25 Ko) |
| **Usage** | Commandes de build, conventions, architecture | Vos préférences, vos corrections, contexte non déductible du code |

Claude les traite comme du **contexte**, pas comme une contrainte : pour bloquer une action quoi qu'il arrive, utilisez une règle de permission ou un hook.

## Les fichiers `CLAUDE.md`

| Portée | Emplacement | Partagé avec |
|--------|-------------|--------------|
| **Politique d'organisation** | macOS `/Library/Application Support/ClaudeCode/CLAUDE.md` ; Linux/WSL `/etc/claude-code/CLAUDE.md` ; Windows `C:\Program Files\ClaudeCode\CLAUDE.md` | Toute l'organisation |
| **Utilisateur** | `~/.claude/CLAUDE.md` | Vous, dans tous les projets |
| **Projet** | `./CLAUDE.md` ou `./.claude/CLAUDE.md` (ou `AGENTS.md`) | L'équipe (via git) |
| **Local** | `./CLAUDE.local.md` (à mettre dans `.gitignore`) | Vous, dans ce projet |

- Claude charge les `CLAUDE.md` du dossier courant **et de tous les dossiers parents** ; ceux des sous-dossiers se chargent quand Claude travaille dedans.
- **Importer** d'autres fichiers : `@chemin/vers/fichier.md` dans un `CLAUDE.md`.
- **Créer** : `/init` (analyse le projet et rédige le fichier). **Éditer** : `/memory`.
- **Auditer** (instructions obsolètes ou contradictoires) : `/doctor prompt-audit`.
- **Conseils** : rester sous ~200 lignes par fichier ; instructions vérifiables (« indentation 2 espaces », « lance `npm test` avant de committer », « les handlers d'API sont dans `src/api/handlers/` ») ; titres et puces ; pas de contradictions.
- **Quand ajouter une règle** : Claude refait deux fois la même erreur, une revue de code relève quelque chose qu'il aurait dû savoir, vous retapez la même correction qu'à la session précédente.

### `AGENTS.md`

Claude Code lit aussi les fichiers `AGENTS.md` (le format partagé avec d'autres outils de codage) comme instructions de projet, via le mod intégré `cc-plugin-agents-md`. On peut choisir quels fichiers d'instructions se chargent.

### Règles modulaires : `.claude/rules/`

Pour les gros projets, découpez les instructions en fichiers thématiques :

```text
mon-projet/
├── .claude/
│   ├── CLAUDE.md
│   └── rules/
│       ├── code-style.md
│       ├── testing.md
│       └── security.md
```

Une règle peut être **limitée à certains fichiers** avec un frontmatter `paths` ; elle ne se charge que quand Claude travaille sur des fichiers correspondants :

```markdown
---
paths:
  - "src/api/**/*.ts"
---
# Règles pour l'API
- Chaque endpoint valide ses entrées
- Utiliser le format d'erreur standard
```

Règles personnelles pour tous vos projets : `~/.claude/rules/`.

## La mémoire automatique

Claude prend des notes pour lui-même au fil du travail, avec quatre types : `user` (votre rôle, vos préférences), `feedback` (vos corrections), `project` (travail en cours, échéances, décisions), `reference` (où trouver l'info : tracker, tableau de bord…). Il n'enregistre pas ce qui se déduit du code.

- Stockage : `~/.claude/projects/<projet>/memory/` (fichier d'index `MEMORY.md`).
- Messages visibles : « Saved 2 memories », « Recalled 2 memories ».
- Activer/désactiver et consulter : `/memory` (ou réglage `autoMemoryEnabled`).
- Les sous-agents peuvent avoir leur propre mémoire persistante (`memory: user|project|local`).

## La fenêtre de contexte

Le contexte contient : prompt système, `CLAUDE.md`, descriptions des skills et outils, conversation, résultats d'outils, fichiers lus. Il a une taille limitée (jusqu'à 1 million de jetons selon le modèle).

| Action | Comment |
|--------|---------|
| Voir l'occupation | `/context` (grille colorée + suggestions), indicateur dans VS Code/Desktop, barre d'état |
| Résumer pour libérer de la place | `/compact [consignes]` |
| Compaction automatique | Se déclenche quand le contexte se remplit ; seuil réglable avec `/autocompact` ou `autoCompactWindow` |
| Compaction en veille | Une conversation de plus de 200 000 jetons inactive ~54 min est compactée avant l'expiration du cache (désactivable : `"idleCompaction": false`) |
| Repartir de zéro | `/clear` (la conversation précédente reste reprenable) |
| Résumer une partie | `/rewind` → « Summarize from here » / « Summarize up to here » |
| Isoler une tâche verbeuse | La confier à un **sous-agent** (son contexte est séparé) |
| Poser une question sans polluer | `/btw` |

Ce qui survit à la compaction : les `CLAUDE.md` sont rechargés, les tâches de la checklist persistent, les skills invoqués restent.

**Cache de prompt** : Claude Code met en cache automatiquement le début de la conversation (5 min ou 1 h). Changer de modèle ou d'effort, ou compacter, coûte un tour non mis en cache ; Claude Code peut demander confirmation. Les modifications de `CLAUDE.md` en cours de session ne s'appliquent qu'à la session suivante (ou après compaction).

## Les sessions

Chaque conversation est enregistrée en continu (transcription `.jsonl` dans `~/.claude/projects/`).

| Pour… | Faire |
|-------|-------|
| Reprendre la dernière conversation du dossier | `claude -c` / `claude --continue` |
| Choisir une conversation | `claude -r` / `/resume` (sélecteur avec recherche ; sessions d'arrière-plan marquées `bg`) |
| Reprendre par nom | `claude -r "auth-refactor"` |
| Reprendre une session liée à une PR | `claude --from-pr 123` |
| Nommer | `claude -n "nom"` ou `/rename` |
| Copier en reprenant (nouvel ID) | `claude -r <id> --fork-session` |
| Brancher la conversation pour essayer autre chose | `/branch [nom]` |
| Copier dans une nouvelle session d'arrière-plan | `/fork [prompt]` |
| Exporter en texte | `/export [fichier]` |
| Passer sur l'app Desktop | `/desktop` ou `claude --desktop --continue` |
| Ramener une session cloud | `/teleport` ou `claude --teleport` |
| Effacer les données locales | `claude purge [chemin]` |

Une session reprise restaure la conversation, l'agent choisi, le mode plan, l'objectif (`/goal`) actif, le worktree, les tâches planifiées non expirées…

## Points de contrôle et retour en arrière (`/rewind`)

Avant chaque prompt qui démarre un tour, Claude Code **capture l'état des fichiers** modifiés par ses outils (les 100 derniers points sont conservés, et ils survivent à la reprise de session).

Ouvrir le menu : `/rewind` (alias `/checkpoint`, `/undo`) ou **`Esc` `Esc`** sur un prompt vide. Choisir un message, puis :

| Option | Effet |
|--------|-------|
| **Restore code and conversation** | Revenir en arrière pour le code et la conversation |
| **Restore conversation** | Revenir dans la conversation, garder le code actuel |
| **Restore code** | Annuler les modifications de fichiers, garder la conversation |
| **Summarize from here** | Résumer la conversation à partir de ce point (libère du contexte) |
| **Summarize up to here** | Résumer ce qui précède ce point |
| **Never mind** | Annuler |

Limites : les fichiers modifiés **par des commandes Bash** ne sont pas suivis (ex. `rm`, `mv`), ni les modifications externes ou celles de la plupart des sous-agents, ni les liens symboliques. **Ce n'est pas un remplacement de git** : continuez à committer.

Dans VS Code, survolez un message pour le bouton de retour en arrière : « Fork conversation from here », « Rewind code to here », « Fork conversation and rewind code ».
