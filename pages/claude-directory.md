# Explorer le répertoire .claude

> Source : https://code.claude.com/docs/en/claude-directory · [← Index](../31-index-documentation-officielle.md)

*Où Claude Code lit CLAUDE.md, settings.json, les hooks, skills, commandes, subagents, workflows, règles et l'auto memory. Explorez le répertoire `.claude` de votre projet et `~/.claude` dans votre répertoire personnel.*

Claude Code lit des instructions, des réglages, des skills, des subagents et de la mémoire depuis le répertoire de votre projet et depuis `~/.claude` dans votre répertoire personnel. Commitez les fichiers du projet dans git pour les partager avec votre équipe ; les fichiers de `~/.claude` sont une configuration personnelle qui s'applique à tous vos projets.

Sous Windows, `~/.claude` correspond à `%USERPROFILE%\.claude`. Si vous définissez [`CLAUDE_CONFIG_DIR`](https://code.claude.com/docs/en/env-vars), tous les chemins `~/.claude` de cette page se trouvent sous ce répertoire à la place.

La plupart des utilisateurs ne modifient que `CLAUDE.md` et `settings.json`. Si votre dépôt contient déjà un `AGENTS.md` destiné à d'autres agents de code, Claude Code [peut le lire](https://code.claude.com/docs/en/memory#agents-md) à la place d'un `CLAUDE.md`. Le reste du répertoire est facultatif : ajoutez des skills, des règles ou des subagents au fur et à mesure de vos besoins.

---

## Explorer le répertoire

Sur la page officielle, un explorateur interactif permet de cliquer sur les fichiers de l'arborescence pour voir ce que fait chacun, quand il est chargé, et un exemple. Il comporte deux onglets, **Project** (`your-project/`) et **Global (~/)**, un bouton « tout déplier / tout replier », un mode plein écran, la navigation au clavier (flèches haut/bas/gauche/droite) et un bouton **Copy** sur chaque exemple. Chaque nœud a une ancre `#ce-<id>` (ex. `#ce-claude-md`). Sur petit écran (moins de 700 px), la page affiche à la place : « L'explorateur interactif fonctionne mieux sur un écran plus grand. Consultez le tableau de référence des fichiers ci-dessous, ou affichez quand même l'explorateur. »

Tout le contenu de l'explorateur est retranscrit ci-dessous.

**Légende des badges :**

| Badge | Signification |
| - | - |
| `committed` | Fichier à commiter dans git (partagé avec l'équipe) |
| `gitignored` | Fichier ignoré par git (personnel) |
| `local only` | Fichier uniquement local (jamais dans un dépôt) |
| `Claude writes` | Fichier/dossier écrit et maintenu automatiquement par Claude |

### Onglet « Project » (`your-project/`)

Arborescence :

```text
your-project/
├── CLAUDE.md                 (committed)
├── .mcp.json                 (committed)
├── .worktreeinclude          (committed)
└── .claude/
    ├── settings.json         (committed)
    ├── settings.local.json   (gitignored)
    ├── rules/
    │   ├── testing.md        (committed)
    │   └── api-design.md     (committed)
    ├── skills/
    │   └── security-review/
    │       ├── SKILL.md      (committed)
    │       └── checklist.md  (committed)
    ├── commands/
    │   └── fix-issue.md      (committed)
    ├── output-styles/
    ├── agents/
    │   └── code-reviewer.md  (committed)
    ├── workflows/
    └── agent-memory/         (committed, Claude writes)
        └── <agent-name>/     (Claude writes)
            └── MEMORY.md     (committed, Claude writes)
```

#### `CLAUDE.md` (projet)

- **Badge** : committed
- **En bref** : instructions de projet que Claude lit à chaque session.
- **Quand il est chargé** : chargé dans le contexte au début de chaque session.
- **Description** : instructions propres au projet qui façonnent la manière dont Claude travaille dans ce dépôt. Mettez-y vos conventions, vos commandes courantes et le contexte architectural, pour que Claude opère avec les mêmes hypothèses que votre équipe.
- **Astuces** :
  - Visez moins de 200 lignes. Les fichiers plus longs se chargent quand même en entier, mais peuvent réduire le respect des consignes.
  - CLAUDE.md se charge dans chaque session. Si quelque chose ne compte que pour certaines tâches, déplacez-le dans un [skill](https://code.claude.com/docs/en/skills) ou une [règle limitée à des chemins](https://code.claude.com/docs/en/memory#organize-rules-with-claude/rules/) pour qu'il ne se charge qu'en cas de besoin.
  - Listez les commandes que vous lancez le plus (build, test, format…) pour que Claude les connaisse sans que vous ayez à les préciser à chaque fois.
  - Lancez `/memory` pour ouvrir et modifier CLAUDE.md depuis une session.
  - Fonctionne aussi à l'emplacement `.claude/CLAUDE.md` si vous préférez garder la racine du projet propre.
  - Si votre dépôt contient déjà un `AGENTS.md` pour d'autres agents de code, Claude Code [peut le lire](https://code.claude.com/docs/en/memory#agents-md) à la place d'un `CLAUDE.md`.
- **Exemple** : cet exemple concerne un projet TypeScript et React. Il liste les commandes de build et de test, les conventions du framework que Claude doit suivre, et des règles propres au projet comme le style d'export et l'organisation des fichiers.

```markdown
# Project conventions

## Commands
- Build: `npm run build`
- Test: `npm test`
- Lint: `npm run lint`

## Stack
- TypeScript with strict mode
- React 19, functional components only

## Rules
- Named exports, never default exports
- Tests live next to source: `foo.ts` -> `foo.test.ts`
- All API routes return `{ data, error }` shape
```

- **Documentation complète** : https://code.claude.com/docs/en/memory

#### `.mcp.json`

- **Badge** : committed
- **En bref** : serveurs MCP à portée projet, partagés avec votre équipe.
- **Quand il est chargé** : les serveurs se connectent au début de la session. Les schémas d'outils sont différés par défaut et chargés à la demande via la [recherche d'outils](https://code.claude.com/docs/en/mcp#scale-with-mcp-tool-search).
- **Description** : configure les serveurs MCP (Model Context Protocol) qui donnent à Claude accès à des outils externes : bases de données, API, navigateurs, etc. Ce fichier contient les serveurs à portée projet que toute l'équipe utilise. Les serveurs personnels que vous voulez garder pour vous vont plutôt dans `~/.claude.json`.
- **Astuces** :
  - Utilisez des références à des variables d'environnement pour les secrets : `${NOTION_TOKEN}`.
  - Se trouve à la racine du projet, pas dans `.claude/`.
  - Pour les serveurs dont vous seul avez besoin, lancez `claude mcp add --scope user`. Cela écrit dans `~/.claude.json` au lieu de `.mcp.json`.
- **Exemple** : cet exemple configure le serveur MCP Notion pour que Claude puisse lire et mettre à jour des pages de votre espace de travail. La référence `${NOTION_TOKEN}` est lue depuis l'environnement de votre shell quand Claude Code démarre le serveur ; le jeton n'atterrit donc jamais dans le fichier.

```json
{
  "mcpServers": {
    "notion": {
      "command": "npx",
      "args": ["-y", "@notionhq/notion-mcp-server"],
      "env": {
        "NOTION_TOKEN": "${NOTION_TOKEN}"
      }
    }
  }
}
```

- **Documentation complète** : https://code.claude.com/docs/en/mcp

#### `.worktreeinclude`

- **Badge** : committed
- **En bref** : fichiers gitignorés à copier dans les nouveaux worktrees.
- **Quand il est chargé** : lu quand Claude crée un worktree git via `--worktree`, l'outil `EnterWorktree`, ou `isolation: worktree` d'un subagent.
- **Description** : liste les fichiers gitignorés à copier depuis votre dépôt principal dans chaque nouveau worktree. Les worktrees sont des checkouts neufs, donc les fichiers non suivis comme `.env` y manquent par défaut. Les motifs utilisent la syntaxe `.gitignore`. Seuls les fichiers qui correspondent à un motif **et** qui sont gitignorés sont copiés : les fichiers suivis ne sont donc jamais dupliqués.
- **Astuces** :
  - Se trouve à la racine du projet, pas dans `.claude/`.
  - Git uniquement : si vous configurez un [hook WorktreeCreate](https://code.claude.com/docs/en/hooks#worktreecreate) pour un autre gestionnaire de versions, ce fichier n'est pas lu. Copiez plutôt les fichiers dans votre script de hook.
  - S'applique aussi aux sessions parallèles de l'[application de bureau](https://code.claude.com/docs/en/desktop#work-in-parallel-with-sessions).
- **Exemple** : cet exemple copie vos fichiers d'environnement locaux et une configuration de secrets dans chaque worktree créé par Claude. Les commentaires commencent par `#` et les lignes vides sont ignorées, comme dans `.gitignore`.

```text
# Environnement local
.env
.env.local

# Identifiants d'API
config/secrets.json
```

- **Documentation complète** : https://code.claude.com/docs/en/worktrees#copy-gitignored-files-into-worktrees

#### `.claude/` (dossier du projet)

- **En bref** : configuration, règles et extensions au niveau du projet.
- **Description** : tout ce que Claude Code lit et qui est propre à ce projet. Si vous utilisez git, commitez la plupart des fichiers de ce dossier pour que votre équipe les partage ; quelques-uns, comme `settings.local.json`, sont gitignorés quand Claude Code y enregistre des réglages. Le badge de chaque fichier indique lesquels.

#### `.claude/settings.json`

- **Badge** : committed
- **En bref** : permissions, hooks et configuration.
- **Quand il est chargé** : remplace le `~/.claude/settings.json` global. Les réglages locaux, les flags CLI et les managed settings le remplacent à leur tour.
- **Description** : réglages que Claude Code applique directement. Les permissions contrôlent quelles commandes et quels outils Claude peut utiliser ; les hooks exécutent vos scripts à des moments précis d'une session. Contrairement à CLAUDE.md, que Claude lit comme des indications, ces réglages sont appliqués de force, que Claude les suive ou non.
- **Clés courantes** :
  - [`permissions`](https://code.claude.com/docs/en/permissions) : autoriser, refuser ou demander confirmation avant que Claude n'utilise certains outils ou commandes.
  - [`hooks`](https://code.claude.com/docs/en/hooks) : exécuter vos propres scripts lors d'événements, comme avant un appel d'outil ou après la modification d'un fichier.
  - [`statusLine`](https://code.claude.com/docs/en/statusline) : personnaliser la ligne affichée en bas pendant que Claude travaille.
  - [`model`](https://code.claude.com/docs/en/settings-reference#available-settings) : choisir un modèle par défaut pour ce projet.
  - [`env`](https://code.claude.com/docs/en/settings-reference#environment-variables) : variables d'environnement définies dans chaque session.
  - [`outputStyle`](https://code.claude.com/docs/en/output-styles) : sélectionner un style de sortie personnalisé depuis `output-styles/`.
- **Astuces** :
  - Les motifs de permission Bash acceptent les jokers : `Bash(npm test *)` correspond à toute commande commençant par `npm test`.
  - Les réglages de type tableau comme `permissions.allow` se combinent entre toutes les portées ; les réglages scalaires comme `model` utilisent la valeur la plus spécifique.
- **Exemple** : cet exemple autorise les commandes `npm test` et `npm run` sans demande de confirmation, bloque `rm -rf`, et exécute Prettier sur les fichiers après que Claude les a modifiés ou écrits.

```json
{
  "permissions": {
    "allow": [
      "Bash(npm test *)",
      "Bash(npm run *)"
    ],
    "deny": [
      "Bash(rm -rf *)"
    ]
  },
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
      }]
    }]
  }
}
```

- **Documentation complète** : https://code.claude.com/docs/en/settings

#### `.claude/settings.local.json`

- **Badge** : gitignored
- **En bref** : vos remplacements personnels de réglages pour ce projet.
- **Quand il est chargé** : c'est le plus prioritaire des fichiers de réglages modifiables par l'utilisateur ; les flags CLI et les managed settings restent néanmoins prioritaires.
- **Description** : réglages personnels qui priment sur les valeurs par défaut du projet. Même format JSON que `settings.json`, gitignoré quand Claude Code y enregistre un réglage. Utilisez-le quand vous avez besoin de permissions ou de valeurs par défaut différentes de la configuration d'équipe.
- **Astuces** :
  - Même schéma que `settings.json`. Les réglages de type tableau comme `permissions.allow` se combinent entre les portées ; les réglages scalaires comme `model` utilisent la valeur locale.
  - Quand Claude Code enregistre un réglage dans ce fichier dans un dépôt qui ne l'ignore pas déjà, il ajoute `**/.claude/settings.local.json` à votre fichier d'exclusions git global : `core.excludesFile` de votre configuration git globale lorsqu'il est défini avec un chemin absolu ou préfixé par `~`, sinon `$XDG_CONFIG_HOME/git/ignore`, ou `~/.config/git/ignore`. Pour partager la règle d'exclusion avec votre équipe, ajoutez-la aussi au `.gitignore` du projet.
- **Exemple** : cet exemple ajoute des permissions Docker en plus de ce qu'autorise le `settings.json` de l'équipe.

```json
{
  "permissions": {
    "allow": [
      "Bash(docker *)"
    ]
  }
}
```

- **Documentation complète** : https://code.claude.com/docs/en/settings

#### `.claude/rules/`

- **En bref** : instructions par thème, éventuellement conditionnées par des chemins de fichiers.
- **Quand il est chargé** : les règles sans `paths:` se chargent au démarrage de la session. Les règles avec `paths:` se chargent quand Claude lit, écrit ou modifie un fichier correspondant.
- **Description** :
  - Instructions de projet découpées en fichiers thématiques qui peuvent se charger de façon conditionnelle selon les chemins de fichiers. Une règle sans frontmatter `paths:` se charge au démarrage de la session comme CLAUDE.md ; une règle avec `paths:` ne se charge que lorsque Claude lit, écrit ou modifie un fichier correspondant.
  - Comme CLAUDE.md, les règles sont des indications que Claude lit, pas une configuration que Claude Code impose. Pour un comportement garanti, utilisez les [hooks](https://code.claude.com/docs/en/hooks) ou les [permissions](https://code.claude.com/docs/en/permissions).
- **Astuces** :
  - Utilisez un frontmatter `paths:` avec des globs pour limiter les règles à des répertoires ou des types de fichiers.
  - Les sous-répertoires fonctionnent : `.claude/rules/frontend/react.md` est découvert automatiquement.
  - Quand CLAUDE.md approche les 200 lignes, commencez à le découper en règles.
- **Documentation complète** : https://code.claude.com/docs/en/memory#organize-rules-with-claude/rules/

##### `.claude/rules/testing.md`

- **Badge** : committed
- **En bref** : conventions de test limitées aux fichiers de test.
- **Quand il est chargé** : quand Claude lit, écrit ou modifie un fichier correspondant aux globs `paths:` ci-dessous.
- **Description** : exemple de règle qui ne se charge que lorsque Claude travaille sur des fichiers de test. Les globs `paths:` du frontmatter définissent quels fichiers la déclenchent ; ici, tout ce qui se termine par `.test.ts` ou `.test.tsx`. Pour les autres fichiers, cette règle n'est pas chargée dans le contexte.

```markdown
---
paths:
  - "**/*.test.ts"
  - "**/*.test.tsx"
---

# Testing Rules

- Use descriptive test names: "should [expected] when [condition]"
- Mock external dependencies, not internal modules
- Clean up side effects in afterEach
```

##### `.claude/rules/api-design.md`

- **Badge** : committed
- **En bref** : conventions d'API limitées au code backend.
- **Quand il est chargé** : quand Claude lit, écrit ou modifie un fichier correspondant au glob `paths:` ci-dessous.
- **Description** : second exemple montrant une règle limitée au code backend. Le glob `paths:` correspond aux fichiers sous `src/api/`, donc ces conventions ne se chargent que lorsque Claude travaille sur les routes d'API.

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Design Rules

- All endpoints must validate input with Zod schemas
- Return shape: { data: T } | { error: string }
- Rate limit all public endpoints
```

#### `.claude/skills/`

- **En bref** : prompts réutilisables que vous ou Claude invoquez par leur nom.
- **Quand il est chargé** : invoqué avec `/skill-name` ou quand Claude associe la tâche à un skill.
- **Description** : chaque skill est un dossier contenant un fichier `SKILL.md` et les fichiers annexes dont il a besoin. Par défaut, vous comme Claude pouvez invoquer un skill. Le frontmatter permet de le contrôler : `disable-model-invocation: true` pour les workflows réservés à l'utilisateur comme `/deploy`, ou `user-invocable: false` pour le masquer du menu `/` tout en laissant Claude l'invoquer.
- **Astuces** :
  - Les skills acceptent des arguments : `/deploy staging` passe « staging » dans `$ARGUMENTS`. Utilisez `$0`, `$1`, etc. pour un accès positionnel.
  - Le champ de frontmatter `description` détermine quand Claude invoque automatiquement le skill.
  - Joignez des documents de référence à côté de `SKILL.md`. Claude connaît le chemin du répertoire du skill et peut lire les fichiers annexes quand vous les mentionnez.
- **Documentation complète** : https://code.claude.com/docs/en/skills

##### `.claude/skills/security-review/`

- **En bref** : un skill regroupant `SKILL.md` et des fichiers annexes.

###### `security-review/SKILL.md`

- **Badge** : committed
- **En bref** : point d'entrée : déclencheur, possibilité d'invocation, instructions.
- **Quand il est chargé** : l'utilisateur tape `/security-review <target>` ; Claude ne peut pas invoquer ce skill automatiquement.
- **Description** :
  - Ce skill utilise `disable-model-invocation: true`, donc vous seul pouvez le déclencher ; Claude ne l'invoque jamais de lui-même.
  - La ligne ``!`...` `` exécute une commande shell et injecte sa sortie dans le prompt. `$ARGUMENTS` est remplacé par ce que vous avez tapé après le nom du skill. Claude voit le chemin du répertoire du skill : mentionner un fichier joint comme `checklist.md` lui permet de le lire.

```markdown
---
description: Reviews code changes for security vulnerabilities, authentication gaps, and injection risks
disable-model-invocation: true
argument-hint: <branch-or-path>
---

## Diff to review

!`git diff $ARGUMENTS`

Audit the changes above for:

1. Injection vulnerabilities (SQL, XSS, command)
2. Authentication and authorization gaps
3. Hardcoded secrets or credentials

Use checklist.md in this skill directory for the full review checklist.

Report findings with severity ratings and remediation steps.
```

###### `security-review/checklist.md`

- **Badge** : committed
- **En bref** : fichier annexe joint au skill.
- **Quand il est chargé** : Claude le lit à la demande pendant l'exécution du skill.
- **Description** : les skills peuvent embarquer n'importe quels fichiers annexes : documents de référence, modèles, scripts. Le chemin du répertoire du skill est ajouté en tête de `SKILL.md`, donc Claude peut lire les fichiers joints par leur nom. Pour les scripts appelés dans des commandes d'injection bash, utilisez l'espace réservé `${CLAUDE_SKILL_DIR}`.

```markdown
# Security Review Checklist

## Input Validation
- [ ] All user input sanitized before DB queries
- [ ] File upload MIME types validated
- [ ] Path traversal prevented on file operations

## Authentication
- [ ] JWT tokens expire after 24 hours
- [ ] API keys stored in environment variables
- [ ] Passwords hashed with bcrypt or argon2
```

#### `.claude/commands/`

- **En bref** : prompts mono-fichier invoqués avec `/name`.

> **Note** : les commandes et les skills sont désormais le même mécanisme. Pour les nouveaux workflows, utilisez plutôt [`skills/`](https://code.claude.com/docs/en/skills) : même invocation `/name`, et vous pouvez en plus y joindre des fichiers annexes.

- **Quand il est chargé** : l'utilisateur tape `/command-name`.
- **Description** : un fichier `commands/deploy.md` crée `/deploy` de la même façon qu'un skill `skills/deploy/SKILL.md`, et les deux peuvent être invoqués automatiquement par Claude. Les skills utilisent un répertoire contenant `SKILL.md`, ce qui permet de joindre des documents de référence, des modèles ou des scripts à côté du prompt.
- **Astuces** :
  - Utilisez `$ARGUMENTS` dans le fichier pour accepter des paramètres : `/fix-issue 123`.
  - Si un skill et une commande partagent le même nom, le skill l'emporte.
  - Les nouvelles commandes devraient généralement être des skills ; les commandes restent prises en charge.
- **Documentation complète** : https://code.claude.com/docs/en/skills

##### `.claude/commands/fix-issue.md`

- **Badge** : committed
- **En bref** : invoqué sous la forme `/fix-issue <number>`.

> **Note** : les commandes et les skills sont désormais le même mécanisme. Pour les nouveaux workflows, utilisez plutôt [`skills/`](https://code.claude.com/docs/en/skills) : même invocation `/name`, et vous pouvez en plus y joindre des fichiers annexes.

- **Description** :
  - Exemple de commande pour corriger une issue GitHub. Tapez `/fix-issue 123` et la ligne ``!`...` `` exécute `gh issue view 123` dans votre shell, en injectant la sortie dans le prompt avant que Claude ne le voie.
  - `$ARGUMENTS` est remplacé par ce que vous avez tapé après le nom de la commande. Pour un accès positionnel, utilisez `$0`, `$1`, etc.

```markdown
---
argument-hint: <issue-number>
---

!`gh issue view $ARGUMENTS`

Investigate and fix the issue above.

1. Trace the bug to its root cause
2. Implement the fix
3. Write or update tests
4. Summarize what you changed and why
```

#### `.claude/output-styles/`

- **En bref** : styles de sortie à portée projet, si votre équipe en partage.
- **Quand il est chargé** : fichiers lus au démarrage ; le style sélectionné avec `outputStyle` s'applique à chaque réponse.
- **Description** : les styles de sortie sont généralement personnels, donc la plupart se trouvent dans `~/.claude/output-styles/`. Placez-en un ici si votre équipe partage un style, comme un mode de revue utilisé par tous. Voir [`~/.claude/output-styles/`](#claudeoutput-styles-global) dans l'onglet Global pour l'explication complète et un exemple.
- **Documentation complète** : https://code.claude.com/docs/en/output-styles

#### `.claude/agents/`

- **En bref** : subagents spécialisés disposant de leur propre fenêtre de contexte.
- **Quand il est chargé** : s'exécute dans sa propre fenêtre de contexte quand vous ou Claude l'invoquez.
- **Description** : chaque fichier Markdown définit un subagent avec son propre prompt système, son accès aux outils et, éventuellement, son propre modèle. Les subagents s'exécutent dans une fenêtre de contexte neuve, ce qui garde la conversation principale propre. Utile pour le travail en parallèle ou les tâches isolées.
- **Astuces** :
  - Chaque agent reçoit une fenêtre de contexte neuve, distincte de votre session principale.
  - Restreignez l'accès aux outils par agent avec le champ de frontmatter `tools:`.
  - Tapez `@` et choisissez un agent dans l'autocomplétion pour lui déléguer directement.
- **Documentation complète** : https://code.claude.com/docs/en/sub-agents

##### `.claude/agents/code-reviewer.md`

- **Badge** : committed
- **En bref** : subagent de revue de code isolée.
- **Quand il est chargé** : Claude le lance pour les tâches de revue, ou vous le @-mentionnez depuis l'autocomplétion.
- **Description** : exemple de subagent limité aux outils en lecture seule. Le champ de frontmatter `description` indique à Claude quand lui déléguer automatiquement ; `tools:` le limite à Read, Grep et Glob pour qu'il puisse inspecter le code sans jamais le modifier. Le corps du fichier devient le prompt système du subagent.

```markdown
---
name: code-reviewer
description: Reviews code for correctness, security, and maintainability
tools: Read, Grep, Glob
---

You are a senior code reviewer. Review for:

1. Correctness: logic errors, edge cases, null handling
2. Security: injection, auth bypass, data exposure
3. Maintainability: naming, complexity, duplication

Every finding must include a concrete fix.
```

#### `.claude/workflows/`

- **En bref** : scripts de workflow dynamiques qui orchestrent de nombreux subagents.
- **Quand il est chargé** : chargés au démarrage ; chaque fichier devient une commande `/<name>`.
- **Description** : chaque fichier `.js` est un [workflow dynamique](https://code.claude.com/docs/en/workflows) : un script que le runtime exécute pour lancer et coordonner de nombreux subagents. Les workflows sont écrits par Claude et enregistrés ici depuis `/workflows`, plutôt que rédigés à partir de zéro.
- **Astuces** :
  - Enregistrez une exécution depuis `/workflows` avec la touche `s` pour créer l'un de ces fichiers.
  - Un workflow de projet l'emporte sur un workflow personnel du même nom situé dans `~/.claude/workflows/`.
- **Documentation complète** : https://code.claude.com/docs/en/workflows

#### `.claude/agent-memory/`

- **Badges** : committed, Claude writes
- **En bref** : mémoire persistante des subagents, distincte de l'auto memory de votre session principale.
- **Quand il est chargé** : les 200 premières lignes (plafonnées à 25 Ko) de `MEMORY.md` sont chargées dans le prompt système du subagent quand il s'exécute.
- **Description** : les subagents qui ont `memory: project` dans leur frontmatter reçoivent ici un répertoire de mémoire dédié. C'est distinct de l'[auto memory de votre session principale](https://code.claude.com/docs/en/memory#auto-memory) située dans `~/.claude/projects/` : chaque subagent lit et écrit son propre `MEMORY.md`, pas le vôtre.
- **Astuces** :
  - Créé uniquement pour les subagents qui définissent le champ de frontmatter `memory:`.
  - Ce répertoire contient la mémoire de subagent à portée projet, destinée à être partagée avec votre équipe. Pour garder la mémoire hors du contrôle de version, utilisez `memory: local`, qui écrit dans `.claude/agent-memory-local/`. Pour une mémoire inter-projets, utilisez `memory: user`, qui écrit dans `~/.claude/agent-memory/`.
  - L'auto memory de la session principale est une fonctionnalité différente ; voir `~/.claude/projects/` dans l'onglet Global.
- **Documentation complète** : https://code.claude.com/docs/en/sub-agents#enable-persistent-memory

##### `.claude/agent-memory/<agent-name>/`

- **Badge** : Claude writes (dossier par subagent).

###### `<agent-name>/MEMORY.md`

- **Badges** : committed, Claude writes
- **En bref** : le subagent écrit et maintient ce fichier automatiquement.
- **Quand il est chargé** : chargé dans le prompt système du subagent quand celui-ci démarre.
- **Description** : fonctionne comme votre [auto memory principale](https://code.claude.com/docs/en/memory#auto-memory) : le subagent crée et met à jour ce fichier lui-même. Vous ne l'écrivez pas. Le subagent le lit au début de chaque tâche et y consigne ce qu'il apprend.

```markdown
# code-reviewer memory

## Patterns seen
- Project uses custom Result<T, E> type, not exceptions
- Auth middleware expects Bearer token in Authorization header
- Tests use factory functions in test/factories/

## Recurring issues
- Missing null checks on API responses (src/api/*)
- Unhandled promise rejections in background jobs
```

### Onglet « Global (~/) »

Arborescence :

```text
~/
├── .claude.json                (local only)
└── .claude/
    ├── CLAUDE.md               (local only)
    ├── settings.json           (local only)
    ├── keybindings.json        (local only)
    ├── themes/
    ├── projects/               (Claude writes)
    │   └── <project>/memory/   (Claude writes)
    │       ├── MEMORY.md       (local only, Claude writes)
    │       └── debugging.md    (local only, Claude writes)
    ├── rules/
    ├── skills/
    ├── commands/
    ├── output-styles/
    │   └── teaching.md         (local only)
    ├── agents/
    ├── workflows/
    └── agent-memory/           (Claude writes)
```

#### `~/.claude.json`

- **Badge** : local only
- **En bref** : état de l'application et préférences d'interface.
- **Quand il est chargé** : lu au démarrage de la session pour vos préférences et vos serveurs MCP. Claude Code y réécrit quand vous modifiez des réglages dans `/config` ou approuvez des invites de confiance.
- **Description** : contient l'état qui n'a pas sa place dans `settings.json` : session OAuth, décisions de confiance par projet, vos serveurs MCP personnels et des bascules d'interface. Se gère surtout via `/config` plutôt qu'en le modifiant directement.
- **Astuces** :
  - Les bascules liées à l'IDE comme `autoConnectIde` et `externalEditorContext` se trouvent ici, pas dans `settings.json`.
  - La clé `projects` suit l'état par projet, comme l'acceptation de la boîte de dialogue de confiance et les métriques de la dernière session. Les règles de permission que vous approuvez en cours de session vont plutôt dans `.claude/settings.local.json`.
  - Les serveurs MCP définis ici sont uniquement les vôtres : la portée user s'applique à tous les projets, la portée local est par projet mais non commitée. Les serveurs partagés en équipe vont plutôt dans `.mcp.json` à la racine du projet.

```json
{
  "autoConnectIde": true,
  "externalEditorContext": true,
  "mcpServers": {
    "my-tools": {
      "command": "npx",
      "args": ["-y", "@example/mcp-server"]
    }
  }
}
```

- **Documentation complète** : https://code.claude.com/docs/en/settings-reference#global-config-settings

#### `~/.claude/` (dossier global)

- **En bref** : votre configuration personnelle pour tous les projets.
- **Description** : le pendant global du répertoire `.claude/` de votre projet. Les fichiers de ce dossier s'appliquent à tous les projets sur lesquels vous travaillez et ne sont jamais commités dans un dépôt.

#### `~/.claude/CLAUDE.md`

- **Badge** : local only
- **En bref** : préférences personnelles pour tous les projets.
- **Quand il est chargé** : au début de chaque session, dans chaque projet.
- **Description** : votre fichier d'instructions global. Chargé en même temps que le CLAUDE.md du projet au démarrage de la session, les deux sont donc ensemble dans le contexte. En cas de conflit, les instructions du projet priment. Limitez-le aux préférences valables partout : style de réponse, format des commits, conventions personnelles.
- **Astuces** :
  - Gardez-le court, car il se charge dans le contexte de chaque projet, en plus du CLAUDE.md propre au projet.
  - Bien adapté au style de réponse, au format des commits et aux conventions personnelles.

```markdown
# Global preferences

- Keep explanations concise
- Use conventional commit format
- Show the terminal command to verify changes
- Prefer composition over inheritance
```

- **Documentation complète** : https://code.claude.com/docs/en/memory

#### `~/.claude/settings.json`

- **Badge** : local only
- **En bref** : réglages par défaut pour tous les projets.
- **Quand il est chargé** : ce sont vos valeurs par défaut. Les `settings.json` de projet et les réglages locaux remplacent toute clé que vous y définissez aussi.
- **Description** :
  - Mêmes clés que le `settings.json` de projet : permissions, hooks, modèle, variables d'environnement, etc. Mettez ici les réglages que vous voulez dans tous les projets, comme des permissions que vous accordez toujours, un modèle préféré, ou un hook de notification qui s'exécute quel que soit le projet.
  - Les réglages suivent un ordre de priorité : le `settings.json` du projet remplace toute clé correspondante définie ici. C'est différent de CLAUDE.md, où les fichiers global et projet sont tous deux chargés dans le contexte au lieu d'être fusionnés clé par clé.

```json
{
  "permissions": {
    "allow": [
      "Bash(git log *)",
      "Bash(git diff *)"
    ]
  }
}
```

- **Documentation complète** : https://code.claude.com/docs/en/settings

#### `~/.claude/keybindings.json`

- **Badge** : local only
- **En bref** : raccourcis clavier personnalisés.
- **Quand il est chargé** : lu au démarrage de la session et rechargé à chaud quand vous modifiez le fichier.
- **Description** : remappe les raccourcis clavier de la CLI interactive. Lancez `/keybindings` pour créer ou ouvrir ce fichier avec une référence de schéma. Ctrl+C, Ctrl+D, Ctrl+M et Verr. Maj (Caps Lock) sont réservés et ne peuvent pas être remappés.
- **Exemple** : cet exemple associe `Ctrl+E` à l'ouverture de votre éditeur externe et désactive `Ctrl+S` en le mettant à `null`. Le champ `context` limite les associations à une partie précise de la CLI, ici la zone de saisie principale du chat.

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "$docs": "https://code.claude.com/docs/en/keybindings",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+s": null
      }
    }
  ]
}
```

- **Documentation complète** : https://code.claude.com/docs/en/keybindings

#### `~/.claude/themes/`

- **En bref** : thèmes de couleurs personnalisés.
- **Quand il est chargé** : lu au démarrage de la session et rechargé à chaud quand les fichiers changent. Listés dans `/theme`.
- **Description** : chaque fichier `.json` définit un thème de couleurs personnalisé : un préréglage intégré `base` plus une table `overrides` de jetons de couleur. Créez-en un de façon interactive avec `/theme` ou écrivez le JSON à la main. Sélectionner un thème personnalisé enregistre `custom:<slug>` comme préférence de thème.

```json
{
  "name": "Dracula",
  "base": "dark",
  "overrides": {
    "claude": "#bd93f9",
    "error": "#ff5555",
    "success": "#50fa7b"
  }
}
```

- **Documentation complète** : https://code.claude.com/docs/en/terminal-config#create-a-custom-theme

#### `~/.claude/projects/`

- **Badge** : Claude writes
- **En bref** : auto memory, les notes de Claude pour lui-même, par projet.
- **Quand il est chargé** : `MEMORY.md` est chargé au démarrage de la session ; les fichiers thématiques sont lus à la demande.
- **Description** : l'auto memory permet à Claude d'accumuler des connaissances d'une session à l'autre sans que vous n'écriviez quoi que ce soit. Claude enregistre des notes au fil du travail : commandes de build, enseignements de débogage, notes d'architecture. Chaque projet a son propre répertoire de mémoire, indexé par le chemin du dépôt.
- **Astuces** :
  - Activée par défaut. Basculez-la avec `/memory` ou le réglage `autoMemoryEnabled`.
  - `MEMORY.md` est l'index chargé à chaque session. Les 200 premières lignes ou les 25 premiers Ko, selon la limite atteinte en premier, sont lus.
  - Les fichiers thématiques comme `debugging.md` sont lus à la demande, pas au démarrage.
  - Ce sont de simples fichiers Markdown. Modifiez-les ou supprimez-les à tout moment.
- **Documentation complète** : https://code.claude.com/docs/en/memory#auto-memory

##### `~/.claude/projects/<project>/memory/`

- **Badge** : Claude writes
- **En bref** : les connaissances accumulées par Claude pour un projet.

###### `memory/MEMORY.md`

- **Badges** : local only, Claude writes
- **En bref** : Claude écrit et maintient ce fichier automatiquement.
- **Quand il est chargé** : les 200 premières lignes (plafonnées à 25 Ko) sont chargées au démarrage de la session.
- **Description** : Claude crée et met à jour ce fichier en travaillant ; vous ne l'écrivez pas vous-même. Il sert d'index que Claude lit au début de chaque session, et renvoie vers des fichiers thématiques pour le détail. Vous pouvez le modifier ou le supprimer, mais Claude continuera à le mettre à jour.

```markdown
# Memory Index

## Project
- [build-and-test.md](build-and-test.md): npm run build (~45s), Vitest, dev server on 3001
- [architecture.md](architecture.md): API client singleton, refresh-token auth

## Reference
- [debugging.md](debugging.md): auth token rotation and DB connection troubleshooting
```

- **Documentation complète** : https://code.claude.com/docs/en/memory

###### `memory/debugging.md`

- **Badges** : local only, Claude writes
- **En bref** : notes thématiques que Claude écrit quand `MEMORY.md` devient long.
- **Quand il est chargé** : Claude le lit quand une tâche liée se présente.
- **Description** : exemple de fichier thématique que Claude crée quand `MEMORY.md` devient trop long. Claude choisit le nom de fichier selon ce qu'il en extrait : `debugging.md`, `architecture.md`, `build-commands.md` ou similaire. Vous ne créez jamais ces fichiers vous-même. Claude ne relit un fichier thématique que lorsque la tâche en cours s'y rapporte.

```markdown
---
name: Debugging patterns
description: Auth token rotation and database connection troubleshooting for this project
type: reference
---

## Auth Token Issues
- Refresh token rotation: old token invalidated immediately
- If 401 after refresh: check clock skew between client and server

## Database Connection Drops
- Connection pool: max 10 in dev, 50 in prod
- Always check `docker compose ps` first
```

#### `~/.claude/rules/`

- **En bref** : règles au niveau utilisateur qui s'appliquent à tous les projets.
- **Quand il est chargé** : les règles sans `paths:` se chargent au démarrage de la session. Les règles avec `paths:` se chargent quand Claude lit, écrit ou modifie un fichier correspondant.
- **Description** : identique au `.claude/rules/` de projet, mais s'applique partout. À utiliser pour les conventions que vous voulez dans tout votre travail, comme votre style de code personnel ou le format des messages de commit.
- **Documentation complète** : https://code.claude.com/docs/en/memory#organize-rules-with-claude/rules/

#### `~/.claude/skills/`

- **En bref** : skills personnels disponibles dans tous les projets.
- **Quand il est chargé** : invoqué avec `/skill-name` dans n'importe quel projet.
- **Description** : les skills que vous avez créés pour vous-même et qui fonctionnent partout. Même structure que les skills de projet : chacun est un dossier avec `SKILL.md`, rattaché à votre compte utilisateur plutôt qu'à un seul projet.
- **Documentation complète** : https://code.claude.com/docs/en/skills

#### `~/.claude/commands/`

- **En bref** : commandes personnelles mono-fichier disponibles dans tous les projets.

> **Note** : les commandes et les skills sont désormais le même mécanisme. Pour les nouveaux workflows, utilisez plutôt [`skills/`](https://code.claude.com/docs/en/skills) : même invocation `/name`, et vous pouvez en plus y joindre des fichiers annexes.

- **Quand il est chargé** : l'utilisateur tape `/command-name` dans n'importe quel projet.
- **Description** : identique au `commands/` de projet, mais rattaché à votre compte utilisateur. Chaque fichier Markdown devient une commande disponible partout.
- **Documentation complète** : https://code.claude.com/docs/en/skills

#### `~/.claude/output-styles/` (global)

- **En bref** : jeux d'instructions personnalisés qui ajustent la façon dont Claude travaille.
- **Quand il est chargé** : fichiers lus au démarrage ; le style sélectionné avec `outputStyle` s'applique à chaque réponse.
- **Description** :
  - Chaque fichier Markdown définit un style de sortie : un ensemble d'instructions pour Claude qui, par défaut, remplace aussi les instructions intégrées propres aux tâches de génie logiciel. Utilisez-le pour adapter Claude Code à des usages au-delà du code, ou pour ajouter des modes d'enseignement ou de revue.
  - Sélectionnez un style intégré ou personnalisé avec `/output-style`, `/config` ou la clé `outputStyle` des réglages. Les styles définis ici sont disponibles dans tous les projets ; un style de projet portant le même nom l'emporte.
- **Astuces** :
  - Les styles intégrés Default, Proactive, Concise, Explanatory et Learning sont fournis avec Claude Code ; les styles personnalisés vont ici.
  - Définissez `keep-coding-instructions: true` dans le frontmatter pour conserver les instructions de tâche par défaut en plus de vos ajouts.
  - Changer de style en cours de session s'applique à partir de votre message suivant ; dans le terminal, un fichier de style créé ou modifié en cours de session est pris en compte après un redémarrage.
- **Documentation complète** : https://code.claude.com/docs/en/output-styles

##### `~/.claude/output-styles/teaching.md`

- **Badge** : local only
- **En bref** : exemple de style qui ajoute des explications et vous laisse les petites modifications.
- **Quand il est chargé** : actif quand `outputStyle` est réglé sur `teaching` dans les réglages.
- **Description** : avec ce style, Claude ajoute une note « Why this approach » (pourquoi cette approche) après chaque tâche et laisse des marqueurs `TODO(human)` pour les modifications de moins de 10 lignes au lieu de les écrire lui-même. Sélectionnez-le en réglant `outputStyle` sur le nom du fichier sans `.md`, ou sur le champ `name` si vous en définissez un dans le frontmatter.

```markdown
---
description: Explains reasoning and asks you to implement small pieces
keep-coding-instructions: true
---

After completing each task, add a brief "Why this approach" note
explaining the key design decision.

When a change is under 10 lines, ask the user to implement it
themselves by leaving a TODO(human) marker instead of writing it.
```

#### `~/.claude/agents/`

- **En bref** : subagents personnels disponibles dans tous les projets.
- **Quand il est chargé** : Claude délègue ou vous @-mentionnez l'agent, dans n'importe quel projet.
- **Description** : les subagents définis ici sont disponibles dans tous vos projets. Même format que les agents de projet.
- **Documentation complète** : https://code.claude.com/docs/en/sub-agents

#### `~/.claude/workflows/`

- **En bref** : workflows dynamiques personnels disponibles dans tous les projets.
- **Quand il est chargé** : chargés au démarrage ; chaque fichier devient une commande `/<name>`.
- **Description** : les scripts de workflow enregistrés ici sont disponibles dans tous vos projets. Un workflow de projet du même nom dans `.claude/workflows/` l'emporte.
- **Documentation complète** : https://code.claude.com/docs/en/workflows

#### `~/.claude/agent-memory/`

- **Badge** : Claude writes
- **En bref** : mémoire persistante pour les subagents ayant `memory: user`.
- **Quand il est chargé** : chargée dans le prompt système du subagent quand celui-ci démarre.
- **Description** : les subagents qui ont `memory: user` dans leur frontmatter stockent ici des connaissances qui persistent dans tous les projets. Pour une mémoire de subagent à portée projet, voir plutôt `.claude/agent-memory/`.
- **Documentation complète** : https://code.claude.com/docs/en/sub-agents#enable-persistent-memory

---

## Ce qui n'est pas montré

L'explorateur couvre les fichiers que vous rédigez et modifiez. Quelques fichiers liés se trouvent ailleurs :

| Fichier | Emplacement | Rôle |
| - | - | - |
| `managed-settings.json` | Au niveau système, variable selon l'OS | Réglages imposés par l'entreprise, que vos propres fichiers de réglages et les valeurs `--settings` ne peuvent pas remplacer, à quelques [exceptions près](https://code.claude.com/docs/en/settings#exceptions-to-managed-settings-precedence). Voir [où enregistrer le fichier](https://code.claude.com/docs/en/managed-settings#deploy-a-managed-settings-file) et [quelle source managée Claude Code utilise](https://code.claude.com/docs/en/managed-settings#precedence-within-the-managed-tier). |
| `CLAUDE.local.md` | Racine du projet | Vos préférences privées pour ce projet, chargées en même temps que CLAUDE.md. Créez-le manuellement et ajoutez-le à `.gitignore`. |
| `AGENTS.md` | Racine du projet, `.claude/` ou n'importe quel répertoire | Instructions de projet que vous rédigez pour les agents de code IA. Claude Code peut [le charger](https://code.claude.com/docs/en/memory#agents-md) à la place d'un `CLAUDE.md`. |
| Plugins installés | `~/.claude/plugins` | Marketplaces clonées, versions de plugins installées, le registre d'installation `installed_plugins.json` et les données par plugin, gérés par les commandes `claude plugin`. Les plugins [synchronisés depuis votre compte claude.ai](https://code.claude.com/docs/en/plugins/loading#synced-plugins) sont téléchargés dans `~/.claude/plugins/synced/`. Pour un plugin installé depuis une [source `command`](https://code.claude.com/docs/en/plugins/marketplace-reference#command-plugin-source) de marketplace en mode lien, Claude Code stocke ici des liens au lieu d'une copie, et les fichiers du plugin restent dans le répertoire affiché par la commande. Une source `command` nécessite Claude Code v2.1.229+. Un plugin référencé par chemin relatif dans une marketplace ajoutée depuis un chemin local se [charge aussi sur place](https://code.claude.com/docs/en/plugins/loading#find-plugins-on-disk) depuis son répertoire source plutôt que depuis une copie en cache. Voir [la mise en cache des plugins](https://code.claude.com/docs/en/plugins/loading#find-plugins-on-disk) pour le nettoyage des versions orphelines. |

`~/.claude` contient aussi des données que Claude Code écrit pendant que vous travaillez : transcriptions, historique des prompts, instantanés de fichiers, caches et journaux. Voir [Données de l'application](#données-de-lapplication) ci-dessous.

## Choisir le bon fichier

Les différents types de personnalisation se trouvent dans des fichiers différents. Utilisez ce tableau pour savoir où placer une modification.

| Vous voulez | Fichier à modifier | Portée | Référence |
| :- | :- | :- | :- |
| Donner à Claude le contexte et les conventions du projet | `CLAUDE.md` | projet ou global | [Memory](https://code.claude.com/docs/en/memory) |
| Autoriser ou bloquer des appels d'outils précis | `settings.json` `permissions` ou `hooks` | projet ou global | [Permissions](https://code.claude.com/docs/en/permissions), [Hooks](https://code.claude.com/docs/en/hooks) |
| Exécuter un script avant ou après les appels d'outils | `settings.json` `hooks` | projet ou global | [Hooks](https://code.claude.com/docs/en/hooks) |
| Définir des variables d'environnement pour la session | `settings.json` `env` | projet ou global | [Settings](https://code.claude.com/docs/en/settings-reference#all-settings) |
| Garder des remplacements personnels hors de git | `settings.local.json` | projet uniquement | [Portées des réglages](https://code.claude.com/docs/en/settings#where-settings-live) |
| Ajouter un prompt ou une capacité invoqué(e) avec `/name` | `skills/<name>/SKILL.md` | projet ou global | [Skills](https://code.claude.com/docs/en/skills) |
| Définir un subagent spécialisé avec ses propres outils | `agents/*.md` | projet ou global | [Subagents](https://code.claude.com/docs/en/sub-agents) |
| Orchestrer de nombreux subagents depuis un script | `workflows/*.js` | projet ou global | [Workflows dynamiques](https://code.claude.com/docs/en/workflows) |
| Connecter des outils externes via MCP | `.mcp.json` | projet uniquement | [MCP](https://code.claude.com/docs/en/mcp) |
| Changer la façon dont Claude formate ses réponses | `output-styles/*.md` | projet ou global | [Output styles](https://code.claude.com/docs/en/output-styles) |

## Référence des fichiers

Ce tableau liste tous les fichiers couverts par l'explorateur. Les fichiers de portée projet se trouvent dans votre dépôt sous `.claude/` (ou à la racine pour `CLAUDE.md`, `.mcp.json` et `.worktreeinclude`). Les fichiers de portée globale se trouvent dans `~/.claude/` et s'appliquent à tous les projets.

> **Note** : plusieurs éléments peuvent remplacer ce que vous mettez dans ces fichiers :
>
> - Les [managed settings](https://code.claude.com/docs/en/server-managed-settings) déployés par votre organisation priment sur tous les fichiers de réglages et toutes les valeurs `--settings`, sauf les [exceptions décrites dans Settings precedence](https://code.claude.com/docs/en/settings#exceptions-to-managed-settings-precedence).
> - Les flags CLI comme `--permission-mode` ou `--settings` remplacent `settings.json` pour cette session.
> - Certaines variables d'environnement priment sur le réglage équivalent, mais cela varie : consultez la [référence des variables d'environnement](https://code.claude.com/docs/en/env-vars) pour chacune.
>
> Voir [l'ordre de priorité des réglages](https://code.claude.com/docs/en/settings#settings-precedence) pour l'ordre complet.

Sur la page officielle, cliquer sur un nom de fichier ouvre le nœud correspondant dans l'explorateur (liens ci-dessous vers ces ancres).

| Fichier | Portée | Commit | Rôle | Référence |
| - | - | - | - | - |
| [`CLAUDE.md`](https://code.claude.com/docs/en/claude-directory#ce-claude-md) | Projet et global | ✓ | Instructions chargées à chaque session | [Memory](https://code.claude.com/docs/en/memory) |
| [`rules/*.md`](https://code.claude.com/docs/en/claude-directory#ce-rules) | Projet et global | ✓ | Instructions par thème, éventuellement conditionnées par chemin | [Rules](https://code.claude.com/docs/en/memory#organize-rules-with-claude/rules/) |
| [`settings.json`](https://code.claude.com/docs/en/claude-directory#ce-settings-json) | Projet et global | ✓ | Permissions, hooks, variables d'environnement, modèle par défaut | [Settings](https://code.claude.com/docs/en/settings) |
| [`settings.local.json`](https://code.claude.com/docs/en/claude-directory#ce-settings-local-json) | Projet uniquement | | Vos remplacements personnels, gitignorés quand Claude Code y enregistre un réglage | [Portées des réglages](https://code.claude.com/docs/en/settings#where-settings-live) |
| [`.mcp.json`](https://code.claude.com/docs/en/claude-directory#ce-mcp-json) | Projet uniquement | ✓ | Serveurs MCP partagés en équipe | [Portées MCP](https://code.claude.com/docs/en/mcp#mcp-installation-scopes) |
| [`.worktreeinclude`](https://code.claude.com/docs/en/claude-directory#ce-worktreeinclude) | Projet uniquement | ✓ | Fichiers gitignorés à copier dans les nouveaux worktrees | [Worktrees](https://code.claude.com/docs/en/worktrees#copy-gitignored-files-into-worktrees) |
| [`skills/<name>/SKILL.md`](https://code.claude.com/docs/en/claude-directory#ce-skills) | Projet et global | ✓ | Prompts réutilisables invoqués avec `/name` ou automatiquement | [Skills](https://code.claude.com/docs/en/skills) |
| [`commands/*.md`](https://code.claude.com/docs/en/claude-directory#ce-commands) | Projet et global | ✓ | Prompts mono-fichier ; même mécanisme que les skills | [Skills](https://code.claude.com/docs/en/skills) |
| [`output-styles/*.md`](https://code.claude.com/docs/en/claude-directory#ce-output-styles) | Projet et global | ✓ | Jeux d'instructions personnalisés qui ajustent la façon dont Claude travaille | [Output styles](https://code.claude.com/docs/en/output-styles) |
| [`agents/*.md`](https://code.claude.com/docs/en/claude-directory#ce-agents) | Projet et global | ✓ | Définitions de subagents avec leur propre prompt et leurs outils | [Subagents](https://code.claude.com/docs/en/sub-agents) |
| [`workflows/*.js`](https://code.claude.com/docs/en/claude-directory#ce-workflows) | Projet et global | ✓ | Scripts de workflow dynamiques écrits par Claude et enregistrés depuis `/workflows` ; chaque fichier devient une commande `/<name>` | [Workflows dynamiques](https://code.claude.com/docs/en/workflows) |
| [`agent-memory/<name>/`](https://code.claude.com/docs/en/claude-directory#ce-agent-memory) | Projet et global | ✓ | Mémoire persistante des subagents | [Mémoire persistante](https://code.claude.com/docs/en/sub-agents#enable-persistent-memory) |
| [`~/.claude.json`](https://code.claude.com/docs/en/claude-directory#ce-claude-json) | Global uniquement | | État de l'application, OAuth, bascules d'interface, serveurs MCP personnels | [Configuration globale](https://code.claude.com/docs/en/settings-reference#global-config-settings) |
| [`projects/<project>/memory/`](https://code.claude.com/docs/en/claude-directory#ce-global-projects) | Global uniquement | | Auto memory : les notes de Claude pour lui-même d'une session à l'autre | [Auto memory](https://code.claude.com/docs/en/memory#auto-memory) |
| [`keybindings.json`](https://code.claude.com/docs/en/claude-directory#ce-keybindings) | Global uniquement | | Raccourcis clavier personnalisés | [Keybindings](https://code.claude.com/docs/en/keybindings) |
| [`themes/*.json`](https://code.claude.com/docs/en/claude-directory#ce-themes) | Global uniquement | | Thèmes de couleurs personnalisés | [Thèmes personnalisés](https://code.claude.com/docs/en/terminal-config#create-a-custom-theme) |

## Champs de frontmatter par fichier

Les skills, fichiers de commande, subagents, styles de sortie et règles lisent leur configuration depuis un [frontmatter](https://code.claude.com/docs/en/glossary#frontmatter) YAML en tête du fichier, et chacun accepte son propre jeu de champs. Ce tableau liste les noms de champs pour chaque fichier, avec un lien vers la référence qui les décrit.

| Fichier | Champs de frontmatter | Référence |
| - | - | - |
| `skills/<name>/SKILL.md` | `name`, `description`, `when_to_use`, `argument-hint`, `arguments`, `disable-model-invocation`, `user-invocable`, `allowed-tools`, `disallowed-tools`, `model`, `effort`, `context`, `agent`, `background`, `hooks`, `paths`, `shell`, `metadata`, `license`, `compatibility` | [Frontmatter des skills](https://code.claude.com/docs/en/skills#frontmatter-reference) |
| `commands/*.md` | Les champs des skills sauf `name` et `paths` | [Frontmatter des skills](https://code.claude.com/docs/en/skills#frontmatter-reference) |
| `agents/*.md` | `name`, `description`, `tools`, `disallowedTools`, `model`, `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory`, `background`, `effort`, `isolation`, `color`, `initialPrompt`, `omitClaudeMd`, `experimental` | [Frontmatter des subagents](https://code.claude.com/docs/en/sub-agents#supported-frontmatter-fields) |
| `output-styles/*.md` | `name`, `description`, `keep-coding-instructions`, `force-for-plugin` | [Frontmatter des styles de sortie](https://code.claude.com/docs/en/output-styles#frontmatter) |
| `rules/*.md` | `paths` | [Frontmatter des règles](https://code.claude.com/docs/en/memory#rules-frontmatter-reference) |

Les agents fournis dans un [plugin](https://code.claude.com/docs/en/plugins/components#agents) ne prennent en compte qu'un sous-ensemble des champs des subagents.

## Dépanner la configuration

Si un réglage, un hook ou un fichier ne prend pas effet, consultez [Debug your configuration](https://code.claude.com/docs/en/debug-your-config) pour les commandes d'inspection et un tableau de diagnostic classé par symptôme.

## Données de l'application

Au-delà de la configuration que vous rédigez, `~/.claude` contient des données que Claude Code écrit pendant les sessions. Ces fichiers sont en texte clair. Tout ce qui passe par un outil est écrit dans une transcription sur disque : contenu de fichiers, sortie de commandes, texte collé.

### Nettoyées automatiquement

Claude Code supprime les fichiers des chemins ci-dessous une fois qu'ils sont plus anciens que [`cleanupPeriodDays`](https://code.claude.com/docs/en/settings-reference#cleanupperioddays), à condition de pouvoir déterminer la durée de rétention en toute sécurité. La valeur par défaut est de 30 jours et le minimum de 1 ; la valeur `0` échoue avec une erreur de validation. La même limite d'âge s'applique à la suppression automatique des [worktrees orphelins](https://code.claude.com/docs/en/worktrees#clean-up-subagent-and-background-session-worktrees).

| Chemin sous `~/.claude/` | Contenu |
| - | - |
| `projects/<project>/<session>.jsonl` | Transcription complète de la conversation : chaque message, appel d'outil et résultat d'outil |
| `projects/<project>/<session>.orphaned-<timestamp>-<suffix>.jsonl`, `projects/<project>/<session>.jsonl.superseded-<timestamp>` | Une transcription précédente de la session que Claude Code a mise de côté au lieu de l'écraser ou de la supprimer. Elle n'apparaît pas dans le sélecteur de sessions |
| `projects/<project>/<session>/subagents/` | Transcriptions des conversations de [subagents](https://code.claude.com/docs/en/sub-agents), supprimées avec la transcription de la session parente quand celle-ci expire |
| `projects/<project>/<session>/tool-results/` | Sorties d'outils volumineuses déversées dans des fichiers séparés, et copies en taille réelle des [images renvoyées par les outils MCP](https://code.claude.com/docs/en/mcp#images-in-tool-results) |
| `file-history/<session>/` | Instantanés, avant modification, des fichiers changés par Claude, utilisés pour la [restauration de checkpoints](https://code.claude.com/docs/en/checkpointing). Conserve les instantanés des 100 checkpoints les plus récents ; les fichiers d'instantané qu'aucun checkpoint conservé ne référence sont supprimés, sauf le premier instantané de chaque fichier |
| `plans/` | Fichiers de plan écrits pendant le [plan mode](https://code.claude.com/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode) |
| `debug/` | Journaux de débogage par session, écrits quand la journalisation de débogage est active, par exemple quand vous démarrez avec [`--debug`](https://code.claude.com/docs/en/cli-reference#cli-flags) ou lancez `/debug` |
| `paste-cache/` | Contenu des collages volumineux |
| `image-cache/<session>/` | Images jointes enregistrées par Claude Code v2.1.274 et antérieures. Les versions ultérieures enregistrent les images collées et jointes hors de `~/.claude`, dans un répertoire `images/` par session, sous le répertoire temporaire contrôlé par [`CLAUDE_CODE_TMPDIR`](https://code.claude.com/docs/en/env-vars). Le balayage supprime ici les répertoires restants des autres sessions, quel que soit leur âge. |
| `uploads/<session>/` | Fichiers que vous joignez depuis l'application web ou mobile, et photos jointes depuis l'application mobile, quand vous envoyez un message à une session [Remote Control](https://code.claude.com/docs/en/remote-control). Une pièce jointe envoyée à une [session cloud](https://code.claude.com/docs/en/claude-code-on-the-web) est enregistrée dans l'environnement cloud de cette session, pas sur votre machine. |
| `dev-mods/<session>/` | [Mods écrits par Claude](https://code.claude.com/docs/en/plugins/mods/create#ask-claude-for-a-mod) pendant la session |
| `session-env/` | Métadonnées d'environnement par session |
| `tasks/` | Listes de tâches écrites par les outils de tâches, un répertoire par liste |
| `shell-snapshots/` | Alias, fonctions et options du shell capturés au démarrage et appliqués par l'[outil Bash](https://code.claude.com/docs/en/tools-reference#bash-tool-behavior) à chaque commande. Supprimés à la sortie propre. Le balayage efface ceux qui restent après un plantage. |
| `backups/` | Versions antérieures de `~/.claude.json`, copiées quand Claude Code réécrit le fichier. Claude Code conserve les cinq plus récentes, plus une copie de toute version qu'il n'a pas pu analyser. |
| `feedback-bundles/` | Archives de transcriptions expurgées écrites par `/feedback` sur des fournisseurs tiers ou quand aucun identifiant Anthropic n'est configuré, à envoyer à l'équipe de votre compte Anthropic |
| `feedback/drafts/` | [Retours rédigés par Claude](https://code.claude.com/docs/en/tools-reference#sendfeedback-tool-behavior) en file d'attente, en attente de votre relecture dans `/feedback`. Balayés après `cleanupPeriodDays` ou 30 jours, selon la durée la plus courte. Quand la file atteint sa limite de 10 brouillons, Claude Code supprime le plus ancien pour faire de la place. |
| `usage-data/` | `report.html` et copies horodatées des rapports écrits par [`/insights`](https://code.claude.com/docs/en/costs#analyze-your-usage-patterns), plus les données d'analyse par session mises en cache pour les construire |
| `skills/.trash/`, `plugins/.trash/` | [Skills](https://code.claude.com/docs/en/skills#how-synced-skills-behave) et [plugins](https://code.claude.com/docs/en/plugins/loading#synced-plugins) supprimés par la synchronisation claude.ai, par exemple après que vous en avez désactivé un sur claude.ai ou arrêté la synchronisation. Les fichiers restent ici pour que vous puissiez les récupérer jusqu'à ce que le balayage les supprime |
| `plugins/installed_plugins.set-aside.<date>.<hash>.json`, `plugins/installed_plugins.unreadable.<date>.<hash>.kept` | Copies datées que Claude Code crée avant de réécrire [`installed_plugins.json`](https://code.claude.com/docs/en/plugins/loading#find-plugins-on-disk) : les enregistrements d'installation qu'il a écartés, et le contenu d'un fichier qu'il n'a pas pu lire. |
| `todos/`, `statsig/`, `logs/` | Répertoires hérités d'anciennes versions. Plus écrits. Le balayage supprime leur contenu puis le répertoire vide. |

Les fichiers de session de `sessions/`, l'auto memory et les transcriptions de Claude Desktop et Cowork suivent chacun leur propre règle de rétention :

- **`sessions/`** : contient un petit fichier par session en cours, utilisé pour détecter les sessions concurrentes et les plantages. Il ne fait pas partie du balayage par âge : Claude Code supprime chaque fichier à la sortie de sa session et efface les restes de plantage au lancement suivant.
- **Auto memory** : le balayage ne supprime pas les fichiers de mémoire du répertoire d'[auto memory](https://code.claude.com/docs/en/memory#auto-memory) d'un projet, `projects/<project>/memory/`. Claude Code ne supprime ce répertoire que s'il est resté vide pendant toute la période de rétention. Avant la v2.1.228, le balayage traitait les dossiers du répertoire de mémoire comme des données de session et pouvait supprimer d'anciens fichiers qu'ils contenaient.
- **Transcriptions de Claude Desktop et Cowork** : Claude Code conserve, quel que soit son âge, la transcription d'une session que vous avez démarrée ou poursuivie en dernier dans Claude Desktop ou Cowork. Pour donner une limite d'âge à ces transcriptions, définissez [`desktopSessionCleanupPeriodDays`](https://code.claude.com/docs/en/settings-reference#desktopsessioncleanupperioddays) (v2.1.248+ ; les versions antérieures les suppriment après `cleanupPeriodDays`).

  Claude Code supprime au contraire ces transcriptions après `cleanupPeriodDays` dans l'un ou l'autre de ces cas :

  - Des [managed settings](https://code.claude.com/docs/en/managed-settings) définissent `cleanupPeriodDays`.
  - La [configuration HIPAA s'applique à vos sessions](https://code.claude.com/docs/en/hipaa-setup#check-how-developers-sign-in-and-connect).

Claude Code saute le balayage par âge dans ces cas :

- **Mode bare** : quand vous lancez `claude -p` avec [`--bare`](https://code.claude.com/docs/en/headless#start-faster-with-bare-mode), Claude Code n'exécute pas le balayage dans cette session.
- **Balayage suspendu** : si Claude Code ne peut pas déterminer la durée de rétention en toute sécurité, il suspend le balayage de nettoyage ; l'[événement `retention_sweep`](https://code.claude.com/docs/en/monitoring-usage#retention-sweep-event) liste chaque configuration qui le suspend. Quand la cause est un fichier de réglages illisible ou impossible à analyser, ou des erreurs de réglages alors que `cleanupPeriodDays` ou `desktopSessionCleanupPeriodDays` est explicitement défini, Claude Code affiche aussi un avertissement dans `/status` jusqu'à ce que vous corrigiez les erreurs. Quand des [managed settings](https://code.claude.com/docs/en/server-managed-settings) fournissent `cleanupPeriodDays`, Claude Code exécute le balayage avec la valeur managée dans les deux cas.

Pour vérifier que les machines de votre organisation exécutent le balayage avec la période de rétention que vous avez définie, voir [Check the retention sweep](https://code.claude.com/docs/en/monitoring-usage#check-the-retention-sweep).

### Répertoire scratchpad de session

Le scratchpad est un répertoire par session que Claude Code fournit à Claude pour les fichiers temporaires : résultats intermédiaires, scripts d'aide et brouillons qui n'ont pas leur place dans votre projet. Quand Claude dit avoir enregistré quelque chose « dans le scratchpad », le fichier s'y trouve. Claude l'utilise au lieu de `/tmp`, et peut y créer, modifier et lire des fichiers sans demande de permission.

Le scratchpad se trouve sous le répertoire temporaire de Claude Code plutôt que sous `~/.claude`. Chemin de la session en cours selon la plateforme :

- **macOS** : `/private/tmp/claude-<uid>/<project>/<session-id>/scratchpad/`
- **Linux** : `/tmp/claude-<uid>/<project>/<session-id>/scratchpad/`, ou la même structure sous `$TMPDIR` quand votre système en définit un
- **Windows** : `%TEMP%\claude\<project>\<session-id>\scratchpad\`

`<project>` est le chemin de votre répertoire de travail dans lequel chaque caractère autre qu'une lettre ou un chiffre est remplacé par `-`, par exemple `-Users-you-my-project`. Si vous définissez [`CLAUDE_CODE_TMPDIR`](https://code.claude.com/docs/en/env-vars), l'arborescence est déplacée sous ce répertoire. Les hooks reçoivent le chemin de la session en cours dans [`scratchpad_dir`](https://code.claude.com/docs/en/hooks#common-input-fields).

Les fichiers du scratchpad durent aussi longtemps que la transcription de la session : le [balayage de rétention](#nettoyées-automatiquement) supprime le répertoire en même temps que la transcription, et [`claude purge`](#effacer-les-données-locales) ne touche pas au répertoire temporaire. Comme le répertoire se trouve dans l'emplacement temporaire du système, votre système d'exploitation peut aussi le vider, par exemple au redémarrage. Pour conserver quelque chose que Claude y a écrit, demandez à Claude de le déplacer dans votre projet.

Une session ne dispose d'un scratchpad que si toutes ces conditions sont réunies :

- Vous êtes connecté avec un compte claude.ai plutôt qu'avec une clé d'API.
- La session utilise l'API Anthropic, et non Amazon Bedrock, Agent Platform de Google Cloud ou Microsoft Foundry.
- [`enableArtifact`](https://code.claude.com/docs/en/settings-reference#enableartifact) n'est pas réglé sur `false`.

### Conservés jusqu'à ce que vous les supprimiez

Hormis les lignes qui indiquent le contraire, le balayage de nettoyage ne supprime pas les chemins ci-dessous, et Claude Code les conserve jusqu'à ce que vous les supprimiez.

| Chemin sous `~/.claude/` | Contenu |
| - | - |
| `history.jsonl` | Chaque prompt que vous avez tapé, avec horodatage et chemin du projet. Sert au rappel par flèche haut, à la recherche dans l'historique avec `Ctrl+R` et à la complétion des commandes shell `!`. Chaque balayage supprime les entrées plus anciennes que `cleanupPeriodDays` quand la [configuration HIPAA s'applique à vos sessions](https://code.claude.com/docs/en/hipaa-setup#check-how-developers-sign-in-and-connect). |
| `stats-cache.json` | Totaux agrégés de jetons et de coûts affichés par `/usage` |
| `remote-settings.json` | Copie en cache des [server-managed settings](https://code.claude.com/docs/en/server-managed-settings) de votre organisation, ou `{}` si votre organisation n'en a configuré aucun. Présent uniquement quand la session [les récupère](https://code.claude.com/docs/en/server-managed-settings#platform-availability). Claude Code vérifie les mises à jour au démarrage et toutes les heures pendant une session. Claude Code le supprime quand vous vous déconnectez. |
| `cache/changelog.md` | Copie en cache du changelog de Claude Code, affichée par `/release-notes`. Actualisée en arrière-plan. |
| `policy-limits.json` | Réglages de politique de fonctionnalités de votre organisation, en cache. Présent uniquement pour certains types de comptes. Actualisé automatiquement. Un fichier compagnon `policy-limits.json.stamp.json` enregistre à quel compte ou clé d'API appartient le cache. Claude Code supprime les deux fichiers quand vous vous déconnectez. |

#### Fichiers d'état à conserver

Selon les fonctionnalités que vous utilisez, `~/.claude/` contient aussi des fichiers que les tableaux de [Données de l'application](#données-de-lapplication) ne listent pas. Parmi eux, les caches et les fichiers de verrou peuvent être supprimés sans risque. Conservez en revanche ces fichiers d'état :

- `.credentials.json` : vos [identifiants de connexion](https://code.claude.com/docs/en/authentication#credential-management)
- `agent-memory/` : la [mémoire des subagents](https://code.claude.com/docs/en/sub-agents#enable-persistent-memory)
- `jobs/` et `daemon/` : l'état des [sessions en arrière-plan](https://code.claude.com/docs/en/agent-view#where-state-is-stored)

### Stockage en clair

> **Avertissement de sécurité** : les transcriptions et l'historique ne sont pas chiffrés au repos. Les permissions de fichiers de l'OS sont la seule protection. Si un outil lit un fichier `.env` ou si une commande affiche un identifiant, cette valeur est écrite dans `projects/<project>/<session>.jsonl`.

Pour réduire l'exposition :

- Baissez `cleanupPeriodDays` pour raccourcir la durée de conservation des transcriptions.
- Définissez [`desktopSessionCleanupPeriodDays`](https://code.claude.com/docs/en/settings-reference#desktopsessioncleanupperioddays) pour donner aussi une limite d'âge aux transcriptions de Claude Desktop et Cowork.
- Définissez la variable d'environnement [`CLAUDE_CODE_SKIP_PROMPT_HISTORY`](https://code.claude.com/docs/en/env-vars) pour ne pas écrire les transcriptions ni l'historique des prompts, quel que soit le mode. En mode non interactif, vous pouvez à la place passer `--no-session-persistence` avec `-p`, ou définir `persistSession: false` dans l'Agent SDK TypeScript ; le SDK Python n'a pas d'option équivalente.
- Utilisez des [règles de permission](https://code.claude.com/docs/en/permissions) pour interdire la lecture des fichiers d'identifiants.

### Effacer les données locales

Lancez `claude purge` pour supprimer l'état que Claude Code conserve pour un projet. La commande supprime :

- Les transcriptions et l'auto memory sous `projects/`
- Les entrées par session de `tasks/`, `debug/` et `file-history/`
- Les lignes de prompt correspondantes dans `history.jsonl`
- L'entrée du projet dans `~/.claude.json`

Les images que vous avez collées ou jointes dans les sessions du projet et le [scratchpad](#répertoire-scratchpad-de-session) de chaque session sont stockés sous le répertoire temporaire de Claude Code plutôt que sous `~/.claude`, donc la purge ne les supprime pas. Le [balayage de rétention](#nettoyées-automatiquement) supprime quand même les images une fois qu'elles sont plus anciennes que `cleanupPeriodDays` ; le scratchpad d'une session purgée reste en place jusqu'à ce que vous le supprimiez ou que votre système d'exploitation vide le répertoire temporaire.

La commande affiche le plan de suppression complet et demande confirmation avant de supprimer quoi que ce soit.

Avant la v2.1.288, la commande s'appelait `claude project purge`.

Les exemples ci-dessous utilisent `~/work/my-repo` comme exemple de chemin. Remplacez-le par le chemin de votre projet. Si aucun état ne correspond au chemin, la commande affiche une erreur et se termine avec le code de sortie 1.

Prévisualiser le plan sans rien supprimer :

```bash
claude purge ~/work/my-repo --dry-run
```

Le plan liste chaque élément correspondant et la raison de son inclusion :

```text
Purge plan for /home/user/work/my-repo:

  dir:    /home/user/.claude/projects/-home-user-work-my-repo
           project transcripts (.jsonl) and memory/
  config: projects["/home/user/work/my-repo"]
           project entry in ~/.claude.json (trust, history, MCP servers)
  filter: /home/user/.claude/history.jsonl
           12 prompt(s) typed in this project

shell-snapshots/ are not project-scoped and will not be touched
backups/ may still contain this project entry in old .claude.json snapshots (/home/user/.claude/backups); at most 5 are kept and they rotate out automatically
Dry run: 3 item(s) would be deleted.
```

Supprimer avec une seule demande de confirmation :

```bash
claude purge ~/work/my-repo
```

La commande affiche le même plan, puis demande `Delete 3 item(s) for /home/user/work/my-repo? This cannot be undone. [y/N]` et ne supprime que si vous répondez `y`.

Omettez le chemin pour choisir un projet dans une liste interactive.

Sauter la demande de confirmation, pour une utilisation dans des scripts :

```bash
claude purge ~/work/my-repo --yes
```

Options récapitulées :

| Option | Effet |
| - | - |
| `<path>` | Projet à purger (omis : choix dans une liste interactive) |
| `--dry-run` | Affiche le plan sans rien supprimer |
| `--yes` | Saute la demande de confirmation (scripts) |
| `--all` | À passer à la place d'un chemin : purge l'état de tous les projets à la fois, et supprime `history.jsonl` entièrement au lieu de le filtrer |
| `-i` | Parcourt le plan de suppression élément par élément |

La commande laisse de côté `shell-snapshots/` et `backups/` car ils ne sont pas propres à un projet, et le signale dans la sortie du plan.

> **Avertissement de sécurité** : si quelqu'un a lancé [`/heapdump`](https://code.claude.com/docs/en/troubleshooting#high-cpu-or-memory-usage) sur la machine, supprimez aussi les fichiers `.heapsnapshot` qu'il a écrits. Un instantané de tas contient la conversation complète et tous les identifiants que le processus détenait, et ni le balayage de rétention ni la purge n'y touchent.

Vous pouvez aussi supprimer à la main n'importe lequel des chemins de données d'application ci-dessus, à l'exception des [fichiers d'état à conserver](#fichiers-détat-à-conserver). Les nouvelles sessions ne sont pas affectées. Le tableau ci-dessous indique ce que vous perdez pour les sessions passées.

| Supprimer | Vous perdez |
| - | - |
| `~/.claude/projects/` | La reprise (resume), la continuation et le retour arrière (rewind) des sessions passées, et l'auto memory de tous les projets |
| `~/.claude/history.jsonl` | Le rappel des prompts par flèche haut, la recherche dans l'historique avec `Ctrl+R` et la complétion des commandes shell `!` |
| `~/.claude/paste-cache/` | Le texte collé dans les prompts rappelés ; voir [coller du contenu volumineux](https://code.claude.com/docs/en/terminal-config#paste-large-content) |
| `~/.claude/uploads/` | Les pièces jointes auxquelles les sessions [Remote Control](https://code.claude.com/docs/en/remote-control) passées font référence par chemin |
| `~/.claude/file-history/` | La restauration de checkpoints pour les sessions passées |
| `~/.claude/stats-cache.json` | Les totaux historiques affichés par `/usage` |
| `~/.claude/usage-data/` | Les rapports [`/insights`](https://code.claude.com/docs/en/costs#analyze-your-usage-patterns) passés et les données d'analyse en cache utilisées pour les construire |
| `~/.claude/feedback-bundles/` | Les archives de retours et de rapports de bug pas encore envoyées à l'équipe de votre compte Anthropic |
| `~/.claude/feedback/drafts/` | Les [retours rédigés par Claude](https://code.claude.com/docs/en/tools-reference#sendfeedback-tool-behavior) que vous n'avez pas envoyés |
| `~/.claude/remote-settings.json` | Rien. Récupéré à nouveau au prochain lancement. |
| `~/.claude/cache/changelog.md` | Rien. Actualisé en arrière-plan. |
| `~/.claude/policy-limits.json` | Rien. Actualisé automatiquement. |
| `~/.claude/tasks/` | Les listes de tâches qu'une session reprise récupérerait |
| `~/.claude/skills/.trash/`, `~/.claude/plugins/.trash/` | La possibilité de récupérer les [skills synchronisés](https://code.claude.com/docs/en/skills#how-synced-skills-behave) et [plugins synchronisés](https://code.claude.com/docs/en/plugins/loading#synced-plugins) que Claude Code a supprimés |
| `~/.claude/plugins/installed_plugins.set-aside.<date>.<hash>.json`, `~/.claude/plugins/installed_plugins.unreadable.<date>.<hash>.kept` | Les copies des enregistrements d'installation de plugins que Claude Code a écartés ou n'a pas pu lire. Rien ne les relit |
| `~/.claude/debug/`, `~/.claude/plans/`, `~/.claude/session-env/`, `~/.claude/shell-snapshots/`, `~/.claude/backups/` | Rien de visible pour l'utilisateur |
| `~/.claude/todos/`, `~/.claude/statsig/`, `~/.claude/logs/`, `~/.claude/image-cache/` | Rien. Répertoires hérités que les versions actuelles n'écrivent plus. |

> **Avertissement** : ne supprimez pas `~/.claude.json`, `~/.claude/settings.json` ni `~/.claude/plugins/` : ils contiennent votre authentification, vos préférences et vos plugins installés.

## Ressources associées

- [Manage Claude's memory](https://code.claude.com/docs/en/memory) : rédiger et organiser CLAUDE.md, les règles et l'auto memory
- [Configure settings](https://code.claude.com/docs/en/settings) : définir permissions, hooks, variables d'environnement et modèle par défaut
- [Create skills](https://code.claude.com/docs/en/skills) : créer des prompts et workflows réutilisables
- [Configure subagents](https://code.claude.com/docs/en/sub-agents) : définir des agents spécialisés disposant de leur propre contexte
