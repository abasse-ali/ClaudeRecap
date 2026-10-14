# Ajouter des composants à un plugin

> Source : https://code.claude.com/docs/en/plugins/components · [← Index](../31-index-documentation-officielle.md)

*Ajouter des skills, des hooks, des serveurs MCP et tous les autres types de composants à un plugin Claude Code, avec pour chacun un exemple qui passe la validation.*

Un plugin Claude Code est construit à partir de composants : skills, agents, hooks, serveurs MCP, etc. Chaque composant possède :

- un **dossier par défaut** dans le plugin ;
- une **clé de manifeste facultative** dans `.claude-plugin/plugin.json` qui remplace ce dossier ou s'y ajoute ;
- un **nom visible par l'utilisateur**.

Pour le tableau complet des champs de chaque clé, voir la [référence du manifeste](https://code.claude.com/docs/en/plugins/manifest-reference#fields).

Cette page sert à ajouter un composant à un plugin **qui se charge déjà**.

Après avoir ajouté un composant, lancez `/reload-plugins` dans une session en cours, ou démarrez-en une nouvelle, pour que Claude Code le charge. Pour vérifier le fichier du composant avant de le charger, exécutez [`claude plugin validate .`](https://code.claude.com/docs/en/plugins/cli-reference#plugin-validate) dans votre shell depuis le répertoire du plugin.

> **Note** : ces cas sont traités sur d'autres pages :
>
> - **Créer votre premier plugin** : commencez par [Créer un plugin](https://code.claude.com/docs/en/plugins/create).
> - **Installer le plugin de quelqu'un d'autre** : voir [Installer des plugins](https://code.claude.com/docs/en/plugins/install).
> - **Les utilisateurs de votre plugin sont sur claude.ai ou dans Cowork** : un autre ensemble de composants s'y charge. Voir [Plugin structure and testing](https://claude.com/docs/plugins/build) et le [tableau de prise en charge des composants](https://claude.com/docs/plugins/platform-support#compare-component-support-by-app).

## Explorer le répertoire du plugin

L'explorateur interactif de la page (« What goes in a plugin ») montre un plugin d'exemple, `my-plugin`, qui contient un composant de chaque type, chacun à son emplacement par défaut :

- un skill de revue (`review`) et une commande `about` ;
- un subagent de revue de sécurité ;
- un hook qui formate les fichiers après que Claude les a modifiés, et le dossier `scripts/` qu'il appelle ;
- un moniteur de journal (log) ;
- un style de sortie et un thème de couleurs ;
- un workflow d'audit des routes ;
- un exécutable `hello-plugin` ;
- des réglages par défaut ;
- un serveur MCP local et un serveur de langage Go.

Chaque fichier est le **plus petit exemple valide** de son format : il montre la forme, pas un usage réel. Un vrai skill ou agent contient des instructions complètes et souvent des fichiers annexes ; un vrai hook ou moniteur fait un vrai travail. Les sections suivantes reprennent ces mêmes fichiers comme exemples et renvoient à des versions plus complètes.

### Arborescence de l'exemple `my-plugin/`

```text
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # Manifeste
├── skills/
│   └── review/
│       └── SKILL.md         # Skills
├── commands/
│   └── about.md             # Commandes
├── agents/
│   └── security-reviewer.md # Agents
├── hooks/
│   └── hooks.json           # Hooks
├── monitors/
│   └── monitors.json        # Moniteurs
├── output-styles/
│   └── terse.md             # Styles de sortie
├── themes/
│   └── dracula.json         # Thèmes
├── workflows/
│   └── audit-routes.js      # Workflows
├── bin/
│   └── hello-plugin         # Exécutables
├── scripts/
│   └── format.sh            # Scripts
├── settings.json            # Réglages par défaut
├── .mcp.json                # Serveurs MCP
└── .lsp.json                # Serveurs LSP
```

| Élément | Chemin | Section de référence |
| :- | :- | :- |
| Manifeste | `.claude-plugin/plugin.json` | [Référence du manifeste](https://code.claude.com/docs/en/plugins/manifest-reference#manifest-file) |
| Skills | `skills/review/SKILL.md` | [Skills](https://code.claude.com/docs/en/plugins/components#skills) |
| Commandes | `commands/about.md` | [Commandes](https://code.claude.com/docs/en/plugins/components#commands) |
| Agents | `agents/security-reviewer.md` | [Agents](https://code.claude.com/docs/en/plugins/components#agents) |
| Hooks | `hooks/hooks.json` | [Hooks](https://code.claude.com/docs/en/plugins/components#hooks) |
| Moniteurs | `monitors/monitors.json` | [Moniteurs](https://code.claude.com/docs/en/plugins/components#monitors) |
| Styles de sortie | `output-styles/terse.md` | [Thèmes et styles de sortie](https://code.claude.com/docs/en/plugins/components#themes-and-output-styles) |
| Thèmes | `themes/dracula.json` | [Thèmes et styles de sortie](https://code.claude.com/docs/en/plugins/components#themes-and-output-styles) |
| Workflows | `workflows/audit-routes.js` | [Distribuer un workflow dans un plugin](https://code.claude.com/docs/en/workflows#distribute-a-workflow-in-a-plugin) |
| Exécutables | `bin/hello-plugin` | [Exécutables](https://code.claude.com/docs/en/plugins/components#executables) |
| Scripts | `scripts/format.sh` | [Hooks](https://code.claude.com/docs/en/plugins/components#hooks) |
| Réglages par défaut | `settings.json` | [Réglages par défaut](https://code.claude.com/docs/en/plugins/components#default-settings) |
| Serveurs MCP | `.mcp.json` | [Serveurs MCP](https://code.claude.com/docs/en/plugins/components#mcp-servers) |
| Serveurs LSP | `.lsp.json` | [Serveurs LSP](https://code.claude.com/docs/en/plugins/components#lsp-servers) |

### Contenu de chaque élément de l'explorateur

#### Manifeste — `.claude-plugin/plugin.json`

Le [manifeste](https://code.claude.com/docs/en/plugins/manifest-reference) est le fichier `plugin.json` du répertoire `.claude-plugin/` du plugin. Il contient les métadonnées du plugin et les valeurs `userConfig` que Claude Code demande à l'utilisateur. Claude Code charge un plugin **même sans manifeste**. Dans le fichier, seul `name` est obligatoire. Ici, `description` est le texte que les utilisateurs voient pour le plugin dans `/plugin`, et `version` maintient les utilisateurs sur cette version jusqu'à ce que vous la changiez :

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Review, formatting, and database tools for this team"
}
```

#### Skills — `skills/review/SKILL.md`

Un [skill](https://code.claude.com/docs/en/skills) est un fichier `SKILL.md`. Enregistrez chaque skill dans son propre répertoire sous `skills/`. Claude lit la `description` de chaque skill et, lorsque la demande de l'utilisateur y correspond (ici, demander à Claude de relire une pull request), il charge les instructions du skill et les suit. L'utilisateur peut aussi le lancer directement avec `/my-plugin:review` :

```markdown
---
description: Reviews a pull request for style and test coverage. Use when asked to review code.
---

Review the changed files. Report style problems first, then missing tests.
```

#### Commandes — `commands/about.md`

Une commande est un fichier Markdown unique que l'utilisateur lance par son nom. Les commandes sont l'**ancien format** : un skill se lance par son nom de la même façon et peut en plus embarquer des fichiers annexes dans son propre répertoire ; écrivez donc les nouvelles sous forme de skills et gardez `commands/` pour les fichiers existants. Ce fichier devient `/my-plugin:about` et accepte le même frontmatter qu'un skill :

```markdown
---
description: Summarize the repository
---

Summarize what this repository does in three sentences.
```

#### Agents — `agents/security-reviewer.md`

Un [subagent](https://code.claude.com/docs/en/sub-agents) est un assistant distinct, avec ses propres instructions et sa propre fenêtre de contexte, auquel Claude peut déléguer une tâche et dont il récupère le résultat. Chaque fichier Markdown sous `agents/` en définit un : le frontmatter le nomme et indique quand l'utiliser, le corps est son prompt système. Celui-ci s'appelle `my-plugin:security-reviewer`, et l'utilisateur peut l'invoquer avec `@agent-my-plugin:security-reviewer` :

```markdown
---
name: security-reviewer
description: Reviews code changes for security issues. Use after edits to authentication or input handling.
model: sonnet
---

You are a security reviewer. Read the changed files and report injection, authentication, and secrets-handling risks.
```

#### Hooks — `hooks/hooks.json`

Un [hook](https://code.claude.com/docs/en/hooks-guide) exécute automatiquement quelque chose à un moment du cycle de vie de Claude Code (par exemple après chaque modification de fichier) : une commande shell, une requête HTTP, un appel d'outil MCP, un prompt envoyé à un modèle ou un subagent. Enregistrez les hooks du plugin dans `hooks/hooks.json` à la racine du plugin. Celui-ci lance le script `scripts/format.sh` du plugin après que Claude a écrit ou modifié un fichier :

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/scripts/format.sh\""
          }
        ]
      }
    ]
  }
}
```

#### Moniteurs — `monitors/monitors.json`

Un moniteur est une commande shell que Claude Code démarre en arrière-plan au début de la session et laisse tourner jusqu'à la fin. Ce qu'il affiche parvient à Claude sous forme de notifications. Un champ `when` permet à la place de le démarrer la première fois qu'un skill donné s'exécute. Celui-ci suit un journal d'erreurs :

```json
[
  {
    "name": "error-log",
    "command": "tail -F ./logs/error.log",
    "description": "Application error log"
  }
]
```

#### Styles de sortie — `output-styles/terse.md`

Un plugin peut inclure des [styles de sortie](https://code.claude.com/docs/en/output-styles), qui changent la façon dont Claude met en forme et formule ses réponses. Enregistrez chaque style sous `output-styles/<name>.md`. Celui-ci apparaît dans `/output-style` sous le nom `my-plugin:terse` :

```markdown
---
name: terse
description: Answer in as few words as possible
keep-coding-instructions: true
---

Keep every reply short. Skip preambles and summaries.
```

#### Thèmes — `themes/dracula.json`

Un plugin peut inclure des [thèmes de couleurs](https://code.claude.com/docs/en/terminal-config#create-a-custom-theme) pour l'interface de Claude Code. Enregistrez chaque thème sous `themes/<slug>.json`. Celui-ci apparaît dans `/theme` sous le nom `Dracula`, marqué comme provenant de `my-plugin` :

```json
{
  "name": "Dracula",
  "base": "dark",
  "overrides": {
    "claude": "#bd93f9",
    "error": "#ff5555"
  }
}
```

#### Workflows — `workflows/audit-routes.js`

Le dossier `workflows/` contient des fichiers `.js` de [workflow](https://code.claude.com/docs/en/workflows) : un bloc `meta`, puis un corps de script qui orchestre plusieurs subagents. Celui-ci se lance avec `/my-plugin:audit-routes` (voir [Distribuer un workflow dans un plugin](https://code.claude.com/docs/en/workflows#distribute-a-workflow-in-a-plugin)) :

```javascript
export const meta = {
  name: 'audit-routes',
  description: 'Audit every route handler for missing auth checks',
}

const found = await agent('List every .ts file under src/routes/.', {
  schema: { type: 'object', required: ['files'], properties: { files: { type: 'array', items: { type: 'string' } } } },
})

const audits = await pipeline(found.files, file =>
  agent(`Audit ${file} for missing authentication checks.`, { label: file }),
)

return audits.filter(Boolean)
```

#### Exécutables — `bin/hello-plugin`

`bin/` est le moyen pour un plugin de fournir un outil en ligne de commande. Tant que le plugin est activé, Claude Code ajoute ce dossier au `PATH` du shell dans lequel il exécute les commandes ; Claude, ou les instructions d'un skill, peuvent donc lancer l'outil par son nom sans que l'utilisateur installe quoi que ce soit. Avec cet [exécutable](https://code.claude.com/docs/en/plugins/components#executables) en place, `hello-plugin` est une commande que Claude peut exécuter :

```bash
#!/bin/bash
echo "hello from my-plugin"
```

#### Scripts — `scripts/format.sh`

Le hook de `hooks/hooks.json` exécute un script, et l'exemple le range dans ce dossier. Le nom `scripts/` est une **convention**, pas un dossier que Claude Code recherche : le hook désigne le fichier par son chemin, `${CLAUDE_PLUGIN_ROOT}/scripts/format.sh`. Un script de formatage pourrait ressembler à ceci :

```bash
#!/bin/bash
npx prettier --write .
```

#### Réglages par défaut — `settings.json`

Un `settings.json` à la racine du plugin contient des [réglages](https://code.claude.com/docs/en/settings-reference) qui s'appliquent tant que le plugin est activé ; un plugin peut ainsi changer le comportement de la session, et pas seulement ajouter des composants. Seules deux clés prennent effet depuis un plugin, [`agent`](https://code.claude.com/docs/en/settings-reference#agent) et [`subagentStatusLine`](https://code.claude.com/docs/en/settings-reference#subagentstatusline) ; toutes les autres sont ignorées. Voir [Réglages par défaut](#réglages-par-défaut).

Celui-ci définit `agent`, qui fait tourner le fil principal de la session avec l'agent `security-reviewer` du plugin : le prompt système, les restrictions d'outils et le modèle de cet agent s'appliquent alors à toute la session :

```json
{
  "agent": "security-reviewer"
}
```

#### Serveurs MCP — `.mcp.json`

Un [serveur MCP](https://code.claude.com/docs/en/mcp) donne à Claude des outils provenant d'un système externe. Déclarez-le dans `.mcp.json` à la racine du plugin. Celui-ci démarre un serveur local à partir d'un script du plugin et apparaît dans `/mcp` sous le nom `plugin:my-plugin:db` :

```json
{
  "mcpServers": {
    "db": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/server.js"]
    }
  }
}
```

#### Serveurs LSP — `.lsp.json`

Un serveur LSP donne à Claude des [diagnostics et une navigation dans le code](https://code.claude.com/docs/en/plugins/code-intelligence) pour un langage. Déclarez le serveur dans `.lsp.json` à la racine du plugin. Celui-ci connecte le serveur de langage Go pour les fichiers `.go` :

```json
{
  "gopls": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": {
      ".go": "go"
    }
  }
}
```

## Ajouter chaque type de composant

Chaque section ci-dessous couvre un type de composant : où placer ses fichiers dans le plugin, un exemple qui passe la validation, ce que voit l'utilisateur une fois le plugin chargé, et la clé de manifeste qui modifie l'emplacement par défaut. Ajoutez ceux dont votre plugin a besoin ; **aucun n'est obligatoire**.

### Skills

Un [skill](https://code.claude.com/docs/en/skills) est un fichier `SKILL.md` que Claude peut charger lorsque sa description correspond à la tâche. L'utilisateur peut aussi le lancer comme une commande. Enregistrez chaque skill dans son propre répertoire sous `skills/` :

```text
my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── review/
        └── SKILL.md
```

Donnez au `SKILL.md` une `description` pour que Claude sache quand l'utiliser :

```markdown skills/review/SKILL.md
---
description: Reviews a pull request for style and test coverage. Use when asked to review code.
---

Review the changed files. Report style problems first, then missing tests.
```

Une fois le plugin chargé, `/my-plugin:review` lance le skill. Le nom de commande et les personnes autorisées à l'invoquer suivent ces règles :

- **Nom de commande** : `/<plugin>:<répertoire>`, donc `skills/review/SKILL.md` dans `my-plugin` donne `/my-plugin:review`. Si vous définissez `name` dans le frontmatter, il remplace le dernier segment et le préfixe du plugin est conservé. Voir [comment un skill obtient son nom de commande](https://code.claude.com/docs/en/skills#how-a-skill-gets-its-command-name).
- **Qui l'invoque** : Claude, l'utilisateur, ou les deux, selon le frontmatter. Voir [Contrôler qui invoque un skill](https://code.claude.com/docs/en/skills#control-who-invokes-a-skill).

Vous pouvez aussi placer des skills en dehors du répertoire `skills/` par défaut :

- **Répertoires supplémentaires** : listez-les dans la clé de manifeste `skills`. Ils **s'ajoutent** à l'analyse du dossier `skills/` par défaut au lieu de la remplacer, contrairement à `commands` et `agents`.
- **Un skill unique à la racine du plugin** : sans répertoire `skills/` ni clé de manifeste `skills`, un `SKILL.md` à la racine du plugin est chargé comme un skill unique. Définissez `name` dans son frontmatter, sinon une installation depuis une marketplace nomme le skill d'après son [répertoire de cache](https://code.claude.com/docs/en/plugins/loading#find-plugins-on-disk) plutôt que d'après votre plugin.

Pour inclure des instructions dans un plugin, écrivez-les sous forme de skill. Claude Code **ne charge pas** de `CLAUDE.md` à la racine du plugin, et `claude plugin validate` affiche l'avertissement `CLAUDE.md at the plugin root is not loaded as project context`.

Si une règle doit s'appliquer à chaque fois, comme [bloquer les modifications de fichiers protégés](https://code.claude.com/docs/en/hooks-guide#block-edits-to-protected-files), ajoutez-la au plugin sous forme de [hook](#hooks) plutôt que de skill. Pour choisir entre les deux, voir l'onglet Hook vs Skill dans [Comparer des fonctionnalités similaires](https://code.claude.com/docs/en/features-overview#compare-similar-features).

Pour les champs de frontmatter et les fichiers annexes, voir [Skills](https://code.claude.com/docs/en/skills).

### Commandes

Une commande est un fichier Markdown unique que l'utilisateur lance par son nom, par exemple `/my-plugin:about`.

> **Note** : les commandes sont l'ancien format, et les [skills](#skills) les remplacent pour tout nouveau travail. Un skill se lance par son nom de la même façon, et peut en plus embarquer des fichiers annexes dans son répertoire. Gardez `commands/` pour les fichiers que vous migrez depuis `.claude/commands/`.

Enregistrez une commande sous `commands/<fichier>.md` et elle devient `/<plugin>:<fichier>`. Un sous-répertoire ajoute un segment : `commands/db/migrate.md` devient `/my-plugin:db:migrate`.

Les fichiers de commande acceptent le même frontmatter que les skills.

#### Définir des commandes dans le manifeste

Ce n'est utile que si vous voulez garder les fichiers de commande ailleurs que dans `commands/`, ou définir une courte commande directement dans `plugin.json` sans fichier Markdown séparé. Définissez la clé de manifeste `commands` : Claude Code la lit **au lieu** d'analyser `commands/`. La clé accepte :

- un chemin ;
- un tableau de chemins ;
- un objet qui associe chaque nom de commande soit à un fichier `source`, soit à un contenu `content` en ligne.

Ce manifeste définit `/my-plugin:about` en ligne, sans fichier Markdown :

```json .claude-plugin/plugin.json
{
  "name": "my-plugin",
  "commands": {
    "about": {
      "content": "Summarize what this repository does in three sentences.",
      "description": "Summarize the repository"
    }
  }
}
```

Chargez le plugin et lancez `/my-plugin:about` dans la session pour confirmer qu'il est chargé.

Pour la syntaxe complète de la clé, voir [`commands`](https://code.claude.com/docs/en/plugins/manifest-reference#commands).

### Agents

Un [subagent](https://code.claude.com/docs/en/sub-agents) est un assistant distinct, avec ses propres instructions et sa propre fenêtre de contexte, auquel Claude peut déléguer une tâche. Chaque fichier Markdown sous `agents/` en définit un :

```markdown agents/security-reviewer.md
---
name: security-reviewer
description: Reviews code changes for security issues. Use after edits to authentication or input handling.
model: sonnet
---

You are a security reviewer. Read the changed files and report injection, authentication, and secrets-handling risks.
```

Cet agent s'appelle `my-plugin:security-reviewer`, et l'utilisateur peut [l'invoquer explicitement](https://code.claude.com/docs/en/sub-agents#invoke-subagents-explicitly) avec `@agent-my-plugin:security-reviewer`. La forme du nom est `<plugin>:<name>`, où `<name>` provient du champ `name` du frontmatter, ou du nom du fichier si ce champ est absent.

La clé de manifeste `agents` **remplace** l'analyse du dossier `agents/`.

#### Organiser les agents en sous-dossiers

Vous pouvez placer les fichiers d'agent du plugin dans des sous-dossiers de `agents/`. Claude Code [les charge récursivement](https://code.claude.com/docs/en/sub-agents#choose-the-subagent-scope) et joint le nom du plugin, chaque nom de sous-dossier et le nom du fichier par des deux-points pour former le nom qualifié de l'agent. Par exemple, `agents/review/security.md` dans un plugin nommé `my-plugin` se charge sous le nom `my-plugin:review:security`. Deux réglages modifient ce nom :

- **`name` dans le frontmatter** : il remplace uniquement le nom du fichier ; `name: audit` dans `agents/review/security.md` se charge donc sous `my-plugin:review:audit`.
- **Champ de manifeste [`agents`](https://code.claude.com/docs/en/plugins/manifest-reference#fields)** : un fichier listé là se charge sans les noms de sous-dossiers ; `"agents": "./custom/review/security.md"` se charge donc sous `my-plugin:security`.

#### Champs de frontmatter des agents de plugin

Le frontmatter d'un agent de plugin suit ces règles :

- **Champs pris en charge** : `name`, `description`, `model`, `effort`, `maxTurns`, `tools`, `disallowedTools`, `skills`, `memory`, `background`, `omitClaudeMd`, `isolation`, `color`, et la clé `cacheTtl` de `experimental`. La seule valeur valide pour `isolation` est `"worktree"`. Voir les [champs de frontmatter pris en charge](https://code.claude.com/docs/en/sub-agents#supported-frontmatter-fields) pour le rôle de chacun.
- **Champs ignorés** : `permissionMode`, `hooks`, `mcpServers` et `initialPrompt`. Un fichier d'agent ne peut pas ajouter de hooks ni de serveurs MCP à lui seul ; ajoutez-les plutôt comme [hooks](#hooks) et [serveurs MCP](#serveurs-mcp) du plugin.
- **Frontmatter qui ne s'analyse pas** : l'agent se charge quand même, avec tous ses champs ignorés. Il porte le nom du fichier, et sa description vaut `Agent from my-plugin plugin`. Exécutez [`claude plugin validate`](https://code.claude.com/docs/en/plugins/cli-reference#plugin-validate) dans votre shell pour repérer ces fichiers.

Pour le rôle de chaque champ et les règles de priorité, voir [Subagents](https://code.claude.com/docs/en/sub-agents#supported-frontmatter-fields).

### Hooks

Un [hook](https://code.claude.com/docs/en/hooks-guide) exécute automatiquement quelque chose à un moment du cycle de vie de Claude Code, comme après chaque modification de fichier : une commande shell, une requête HTTP, un appel d'outil MCP, un prompt envoyé à un modèle ou un subagent. Enregistrez les hooks du plugin dans `hooks/hooks.json` à la racine du plugin, sous une clé `"hooks"` de premier niveau, avec la même forme que l'objet `hooks` de `settings.json`. Vous pouvez ainsi recopier tel quel un hook existant de vos réglages.

Ce hook lance un script fourni après chaque `Write` ou `Edit` :

```json hooks/hooks.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/scripts/format.sh\""
          }
        ]
      }
    ]
  }
}
```

Enregistrez le script sous `scripts/format.sh` et rendez-le exécutable.

Chargez le plugin et demandez à Claude de modifier un fichier. Un hook `PostToolUse` qui se termine avec le code 0 n'affiche rien dans la transcription ; vérifiez qu'il s'est exécuté via la [journalisation de débogage](https://code.claude.com/docs/en/hooks#debug-hooks) ou d'après ce que le script lui-même a modifié.

Les hooks de `hooks/hooks.json` **et** ceux de la clé de manifeste `hooks` se chargent tous. Pour chaque événement et sa charge utile, voir [Événements de hook](https://code.claude.com/docs/en/hooks#hook-events).

Lorsqu'un autre plugin activé porte le même nom, l'un des deux enregistre ses hooks de `hooks/hooks.json` et ceux de l'autre sont écartés. Voir [Hooks quand deux plugins activés partagent un nom](https://code.claude.com/docs/en/plugins/loading#hooks-when-two-enabled-plugins-share-a-name) pour savoir lequel, et pour la note dans `/plugin` qui vous le signale.

Pour écrire des hooks sous forme de fonctions JavaScript qui s'exécutent dans Claude Code et peuvent dessiner dans son interface, listez un fichier de module sous une clé `modules` dans le même `hooks/hooks.json`. Un plugin qui en contient un est un **mod**. Voir [Créer un mod](https://code.claude.com/docs/en/plugins/mods/create).

#### Quand les hooks de plugin se déclenchent

Les hooks d'un plugin n'attendent pas qu'un des skills ou commandes du plugin soit utilisé. Claude Code les enregistre dès qu'une session charge le plugin, et ils se déclenchent sur leurs événements à partir de ce moment. Pour limiter les déclenchements d'un hook, restreignez son `matcher`.

Si un hook ne se déclenche jamais, voir [les hooks qui ne se déclenchent pas](https://code.claude.com/docs/en/plugins/troubleshooting#failed-to-load-hooks-from-and-hooks-that-dont-fire).

#### Environnement, guillemets et correspondance des outils MCP

L'environnement du hook, la mise entre guillemets de `${CLAUDE_PLUGIN_ROOT}` et les matchers ciblant les outils MCP du plugin fonctionnent ainsi :

- **Environnement** : chaque processus de hook reçoit `CLAUDE_PLUGIN_ROOT` et `CLAUDE_PLUGIN_DATA` dans son environnement, plus `CLAUDE_PLUGIN_OPTION_<KEY>` pour chaque valeur de [configuration utilisateur](#demander-des-valeurs-de-configuration-à-lutilisateur) ; votre script peut donc les y lire.
- **Guillemets** : lorsque `command` n'a pas d'`args`, elle s'exécute via un shell ; entourez donc le chemin `${CLAUDE_PLUGIN_ROOT}` de guillemets doubles, comme dans l'exemple `hooks/hooks.json` de la section [Hooks](#hooks), pour que le chemin développé reste un seul mot shell. Si vous passez `args` à la place, chaque élément est transmis comme un argument unique, sans shell, et ne nécessite aucun guillemet. Voir [forme exec et forme shell](https://code.claude.com/docs/en/hooks#exec-form-and-shell-form).
- **Cibler les propres outils MCP du plugin** : un outil d'un [serveur MCP déclaré par ce plugin](#serveurs-mcp) s'appelle `mcp__plugin_<plugin>_<server>__<tool>` ; écrivez ce nom complet dans le matcher. Un matcher portant uniquement sur le nom du serveur ne se déclenche jamais. Voir [Cibler les outils MCP](https://code.claude.com/docs/en/hooks#match-mcp-tools).

### Serveurs MCP

Un serveur MCP donne à Claude des outils provenant d'un système externe. Déclarez-le dans `.mcp.json` à la racine du plugin, avec la même forme qu'un [`.mcp.json` de projet](https://code.claude.com/docs/en/mcp#project-scope). Ce `.mcp.json` déclare un serveur nommé `db` :

```json .mcp.json
{
  "mcpServers": {
    "db": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/server.js"]
    }
  }
}
```

Vous pouvez aussi omettre l'enveloppe `mcpServers` et placer `db` au premier niveau du fichier.

Chargez le plugin et lancez `/mcp` pour confirmer que le serveur apparaît sous `plugin:my-plugin:db`.

`claude plugin validate` vérifie `.mcp.json` et signale comme **erreur** toute entrée de serveur que Claude Code abandonnerait au chargement (v2.1.281+).

Pour savoir où une entrée invalide apparaît au chargement, voir [Serveurs MCP qui ne démarrent pas](https://code.claude.com/docs/en/plugins/troubleshooting#invalid-mcp-server-config-for-and-mcp-servers-that-dont-start).

La clé de manifeste `mcpServers` accepte une table de serveurs en ligne, un chemin vers un fichier JSON, ou un tableau de ceux-ci. Lorsqu'un serveur du manifeste porte le même nom qu'un serveur de `.mcp.json`, celui du manifeste le remplace.

#### Toucher les utilisateurs de claude.ai et Cowork

Un serveur stdio local, comme le serveur `db` de la section [Serveurs MCP](#serveurs-mcp), fonctionne dans Claude Code et dans une session Cowork qui tourne sur votre machine dans l'application Claude Desktop, **mais pas sur claude.ai**. Pour atteindre aussi ces utilisateurs, référencez un serveur distant par son URL `https://`, que claude.ai et Cowork proposent à l'utilisateur comme connecteur, comme le montre [Bundle an MCP connector with its skill](https://claude.com/docs/plugins/build#bundle-an-mcp-connector-with-its-skill).

#### Noms de serveur, noms d'outils et rechargements

Les noms du serveur, la substitution de variables et le comportement au rechargement suivent ces règles :

- **Nom du serveur** : `plugin:<plugin>:<server>` ; le serveur `db` de `my-plugin` s'appelle donc `plugin:my-plugin:db` dans `/mcp`. Utilisez la même forme pour nommer le serveur dans un [hook `mcp_tool`](https://code.claude.com/docs/en/hooks#mcp-tool-hook-fields).
- **Noms d'outils** : `mcp__plugin_<plugin>_<server>__<tool>` ; un outil `query` sur ce serveur `db` s'appelle donc `mcp__plugin_my-plugin_db__query`. C'est le nom à utiliser dans les [règles de permission](https://code.claude.com/docs/en/permissions) et les [matchers de hook](#hooks).
- **Substitution** : `${CLAUDE_PLUGIN_ROOT}` et les autres [variables de chemin](#référencer-les-chemins-du-plugin-et-stocker-des-données) sont substituées dans `command`, `args` et `env`. Aucun guillemet n'est nécessaire dans `args`, car chaque élément est transmis comme un argument unique.
- **Rechargement** : lorsque l'utilisateur lance `/reload-plugins` et que [le rechargement s'applique](https://code.claude.com/docs/en/plugins/cli-reference#reloads-that-change-mcp-tools), un serveur dont la configuration n'a pas changé conserve sa connexion. Un serveur dont la configuration a changé se reconnecte, et un serveur retiré se déconnecte.

#### Inclure un serveur MCPB empaqueté

La clé `mcpServers` accepte aussi un serveur empaqueté sous forme de [fichier MCPB](https://github.com/modelcontextprotocol/mcpb), dont l'extension est `.mcpb` ou l'ancienne `.dxt`. Faites pointer la clé vers le fichier, sous forme de chemin dans le plugin ou d'URL `https://` :

```json .claude-plugin/plugin.json
{
  "name": "my-plugin",
  "mcpServers": "./servers/db.mcpb"
}
```

Le serveur prend son nom du `name` figurant dans le manifeste du bundle.

Le manifeste propre d'un bundle peut déclarer, dans un bloc `user_config`, les réglages que le serveur attend de l'utilisateur. Un serveur empaqueté dont un réglage obligatoire n'a pas de valeur enregistrée **ne démarre pas**. L'onglet **Errors** de `/plugin` affiche alors `Bundled MCP server "<name>" was not started: it needs configuration`.

Les utilisateurs fournissent les valeurs de l'une de deux façons :

- **Dans `/plugin`** : sélectionner le plugin dans l'onglet **Installed** et choisir **Configure**.
- **À l'installation, depuis le shell** : passer [`--config <server>.<key>=<value>`](https://code.claude.com/docs/en/plugins/cli-reference#plugin-install) à `claude plugin install` (v2.1.285+). Ne fonctionne que pour un bundle empaqueté à l'intérieur du plugin.

Pour les transports et l'authentification, voir [MCP](https://code.claude.com/docs/en/mcp#plugin-provided-mcp-servers).

### Serveurs LSP

Un serveur LSP donne à Claude des diagnostics et une navigation dans le code pour un langage. Si un [plugin officiel d'intelligence de code](https://code.claude.com/docs/en/plugins/code-intelligence) couvre déjà votre langage, installez-le plutôt que d'en écrire un. Sinon, déclarez le serveur dans `.lsp.json` à la racine du plugin :

```json .lsp.json
{
  "gopls": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": {
      ".go": "go"
    }
  }
}
```

Le fichier associe directement chaque nom de serveur à sa configuration, **sans objet enveloppe** autour de la table. `command` est le nom du binaire, avec ses arguments dans `args`. `extensionToLanguage` nécessite au moins une extension, chacune commençant par `.`.

`claude plugin validate` **ne lit pas** ce fichier. Lorsqu'une entrée est invalide, le fichier entier est ignoré au chargement et `Invalid LSP server config for ".lsp.json"` apparaît dans l'onglet **Errors** de `/plugin`.

Votre plugin configure la connexion mais **n'installe pas** le binaire du serveur, et chaque extension de fichier n'a qu'un seul serveur :

- **Binaire manquant** : Claude Code lance `command` par son nom depuis le `PATH` de l'utilisateur. Si le binaire n'y est pas, le serveur ne démarre pas et `claude --debug` journalise `LSP server <name> failed to start`.
- **Conflits d'extension** : lorsque deux serveurs activés revendiquent la même extension, le premier enregistré gère ces fichiers et l'autre n'est pas utilisé pour eux, que les serveurs viennent d'un même plugin ou de deux. L'onglet **Errors** de `/plugin` affiche l'avertissement `LSP server "<name>" is not used for <ext> files`.

La clé de manifeste `lspServers` accepte la même table en ligne, un chemin vers un fichier JSON, ou un tableau de ceux-ci, et ses serveurs **s'ajoutent** à ceux de `.lsp.json`. Lorsqu'un serveur du manifeste porte le même nom qu'un serveur de `.lsp.json`, celui du manifeste le remplace.

Pour `transport`, les délais d'expiration, les redémarrages et les autres champs, voir [`lspServers`](https://code.claude.com/docs/en/plugins/manifest-reference#lspservers).

Envoyez la sortie de journalisation sur **stderr, pas stdout**. Claude Code lit la sortie stdout d'un serveur uniquement comme des messages de protocole, et accepte des en-têtes de message jusqu'à **64 Kio** et un corps de message jusqu'à **32 Mio**.

Claude Code déconnecte un serveur qui dépasse l'une de ces limites ou qui écrit une sortie hors protocole sur stdout, et compte cette déconnexion comme un plantage pour `restartOnCrash` et `maxRestarts`. Lorsque vous lancez avec `--debug`, Claude Code écrit dans le journal de débogage une erreur indiquant la cause.

### Exécutables

Les fichiers de `bin/` à la racine du plugin sont dans le `PATH` du shell de l'outil Bash tant que le plugin est activé ; Claude peut donc les lancer comme de simples commandes. Ajoutez un script exécutable :

```bash bin/hello-plugin
#!/bin/bash
echo "hello from my-plugin"
```

Rendez-le exécutable avec `chmod +x bin/hello-plugin` et chargez le plugin. Quand vous demandez à Claude de lancer `hello-plugin`, le résultat de l'outil Bash affiche la sortie du script.

Les répertoires `bin/` des plugins viennent **après** les entrées du `PATH` de l'utilisateur ; un plugin ne peut donc pas masquer `git`, `ls` ou une autre commande système.

> **Attention (disponibilité)** : claude.ai et Cowork n'installent pas un plugin qui possède un répertoire `bin/` de premier niveau, y compris un plugin que vous [distribuez via les paramètres d'organisation de claude.ai](https://claude.com/docs/plugins/org-sync#keep-executables-out-of-the-top-level-bin-directory).

### Réglages par défaut

Pour définir des valeurs par défaut qui s'appliquent tant que le plugin est activé, ajoutez un `settings.json` à la racine du plugin, ou placez le même objet en ligne dans la clé de manifeste `settings`. Deux clés prennent effet, `agent` et `subagentStatusLine` ; toutes les autres sont ignorées.

Définissez `agent` pour faire tourner l'un des agents du plugin comme fil principal :

```json settings.json
{
  "agent": "security-reviewer"
}
```

Chargez le plugin et démarrez une session. Claude répond alors dans la conversation principale avec le prompt système et le modèle de l'agent `security-reviewer`.

Pour tout ce que contrôle cette clé, voir le [réglage `agent`](https://code.claude.com/docs/en/settings-reference#agent).

Lorsque la même clé est définie à plusieurs endroits, ces règles déterminent la valeur appliquée :

- **Le fichier l'emporte sur le manifeste** : quand les deux existent et que `settings.json` définit au moins une clé prise en charge, `settings.json` s'applique et le `settings` du manifeste est ignoré.
- **Les réglages utilisateur l'emportent sur les valeurs par défaut du plugin** : parmi les sources de réglages, les valeurs par défaut des plugins forment la couche la plus basse ; un `agent` défini par l'utilisateur dans `~/.claude/settings.json` remplace donc le vôtre.
- **Deux plugins définissent la même clé** : la valeur du plugin chargé en dernier s'applique, et `claude --debug` journalise `overrides setting`.

Pour la forme de `subagentStatusLine`, voir [lignes de statut des subagents](https://code.claude.com/docs/en/statusline#subagent-status-lines).

### Thèmes et styles de sortie

Un plugin peut inclure des thèmes de couleurs et des styles de sortie. Les deux apparaissent dans les mêmes sélecteurs que ceux de l'utilisateur. Pour l'un comme pour l'autre, définir la clé de manifeste **remplace** l'analyse du dossier.

| Composant | Enregistrer sous | Format | Apparaît dans | Clé de manifeste |
| :- | :- | :- | :- | :- |
| Thème | `themes/<slug>.json` | Le format de [fichier de thème personnalisé](https://code.claude.com/docs/en/terminal-config#create-a-custom-theme) que les utilisateurs écrivent dans `~/.claude/themes/` | `/theme`, sous le `name` du fichier | `experimental.themes` |
| Style de sortie | `output-styles/<name>.md` | Le format de [style de sortie personnalisé](https://code.claude.com/docs/en/output-styles#create-a-custom-output-style), avec frontmatter `name` et `description` | `/output-style`, sous `<plugin>:<name>` | `outputStyles` |

Les thèmes de plugin sont en **lecture seule** : quand un utilisateur en modifie un dans `/theme`, la modification est enregistrée comme une copie dans son propre répertoire de thèmes.

Ce thème recolore l'accent du prompt et le texte d'erreur sur le preset sombre :

```json themes/dracula.json
{
  "name": "Dracula",
  "base": "dark",
  "overrides": {
    "claude": "#bd93f9",
    "error": "#ff5555"
  }
}
```

### Canaux (channels)

Un [canal](https://code.claude.com/docs/en/channels) permet à un système extérieur, comme une application de messagerie, d'envoyer des messages dans une session. Dans un plugin, un canal est l'un des serveurs MCP, plus une entrée `channels` qui s'y lie et peut demander sa propre configuration. Ce manifeste lie un canal à un serveur `telegram` et demande un jeton de bot :

```json .claude-plugin/plugin.json
{
  "name": "my-plugin",
  "mcpServers": {
    "telegram": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/server.js"],
      "env": { "BOT_TOKEN": "${user_config.bot_token}" }
    }
  },
  "channels": [
    {
      "server": "telegram",
      "userConfig": {
        "bot_token": {
          "type": "string",
          "title": "Bot token",
          "description": "Telegram bot token",
          "sensitive": true
        }
      }
    }
  ]
}
```

`server` doit correspondre à une clé de `mcpServers`. Le `userConfig` propre à chaque canal a la même forme que la [clé `userConfig` de premier niveau](#demander-des-valeurs-de-configuration-à-lutilisateur).

Pour ce que le serveur doit implémenter et la façon dont les utilisateurs activent un plugin de canal, voir [Package as a plugin](https://code.claude.com/docs/en/channels-reference#package-as-a-plugin) dans la référence des canaux. Pour le tableau des champs, voir [`channels`](https://code.claude.com/docs/en/plugins/manifest-reference#channels).

### Moniteurs

Un moniteur est une commande shell qui tourne en arrière-plan pendant toute la session. Ce qu'elle affiche parvient à Claude sous forme de notifications ; Claude peut ainsi réagir à un journal ou à un changement d'état sans qu'on lui demande de le surveiller. Enregistrez les entrées dans `monitors/monitors.json` :

```json monitors/monitors.json
[
  {
    "name": "error-log",
    "command": "tail -F ./logs/error.log",
    "description": "Application error log"
  }
]
```

La commande s'exécute dans un shell, dans le répertoire de travail courant de la session.

> **Avertissement (sécurité)** : elle s'exécute avec **toutes vos permissions utilisateur** et **en dehors du [sandbox](https://code.claude.com/docs/en/sandboxing)**.

La commande d'un moniteur est limitée quant au moment où elle démarre et à ce qu'elle peut référencer :

- **Sessions interactives uniquement** : les moniteurs de plugin démarrent dans une session interactive et **jamais** en mode non interactif avec le flag `-p`. Ils ne démarrent pas non plus dans les sessions où le fournisseur d'API ou les réglages de télémétrie rendent l'[outil Monitor](https://code.claude.com/docs/en/tools-reference#monitor-tool) indisponible.
- **Pas de configuration utilisateur** : `command` reçoit les [variables de chemin](#référencer-les-chemins-du-plugin-et-stocker-des-données) et les `${ENV_VAR}` de l'environnement, mais jamais `${user_config.*}`. Un moniteur qui en référence un ne démarre pas, et les processus de moniteur ne reçoivent pas non plus `CLAUDE_PLUGIN_OPTION_<KEY>`.
- **Désactivation en cours de session** : si vous désactivez un plugin en cours de session, Claude Code n'arrête pas les moniteurs déjà lancés. Ils s'arrêtent à la fin de la session.

La clé de manifeste `experimental.monitors` accepte le même tableau en ligne ou un chemin vers un fichier JSON, et elle est lue **au lieu** de `monitors/monitors.json`.

Pour le déclencheur `when` et les autres champs, voir [`monitors`](https://code.claude.com/docs/en/plugins/manifest-reference#monitors).

## Demander des valeurs de configuration à l'utilisateur

*(ancre d'origine : `#user-configuration`)*

Déclarez les valeurs dont votre plugin a besoin dans la clé de manifeste `userConfig`, pour que les utilisateurs n'aient pas à modifier `settings.json` eux-mêmes. Chaque option apparaît dans une boîte de dialogue, avec son `title` comme libellé et sa `description` en dessous.

Définissez `"sensitive": true` pour un jeton ou un mot de passe. La boîte de dialogue masque alors la saisie, et la valeur est stockée dans le **stockage sécurisé** plutôt que dans `settings.json`.

Ce manifeste demande un point de terminaison (endpoint) et un jeton :

```json .claude-plugin/plugin.json
{
  "name": "my-plugin",
  "userConfig": {
    "api_url": {
      "type": "string",
      "title": "API URL",
      "description": "Base URL of your team's API"
    },
    "api_token": {
      "type": "string",
      "title": "API token",
      "description": "Token for your team's API",
      "sensitive": true
    }
  }
}
```

### Quand la boîte de dialogue de configuration apparaît

La boîte de dialogue fait partie de l'interface interactive `/plugin`. Elle s'ouvre, pour toute option pas encore définie, quand l'utilisateur :

- installe le plugin dans `/plugin` ;
- lance `/plugin install <plugin>@<marketplace>` dans une session ;
- active le plugin depuis l'onglet **Installed** de `/plugin`.

Pour ouvrir cette même boîte de dialogue à tout moment, l'utilisateur lance `/plugin configure <plugin>@<marketplace>`.

La [boîte de dialogue Manage plugins](https://code.claude.com/docs/en/vs-code#install-plugins) de l'extension VS Code demande les options non définies sous forme de formulaire après une installation, et une icône d'engrenage sur la ligne du plugin rouvre le formulaire avec toutes les options.

La commande shell `claude plugin install` **ne demande jamais** les valeurs `userConfig`. Pour définir des valeurs depuis le shell, passez chacune avec `--config KEY=VALUE` lors de l'installation, ou envoyez ensuite un objet JSON via un pipe à [`claude plugin configure --values-stdin`](https://code.claude.com/docs/en/plugins/cli-reference#plugin-configure).

Lorsque des options restent non définies, `claude plugin install` affiche une ligne `userConfig options not yet set`. Pour le texte exact de cette ligne, voir [La boîte de dialogue `userConfig` n'apparaît jamais](https://code.claude.com/docs/en/plugins/troubleshooting#the-userconfig-dialog-never-appears).

Pour les champs d'option, l'emplacement de stockage de chaque valeur, la façon dont un composant référence une valeur enregistrée, et les champs qui refusent `${user_config.*}`, voir [Configuration utilisateur](https://code.claude.com/docs/en/plugins/manifest-reference#user-configuration).

## Référencer les chemins du plugin et stocker des données

*(ancre d'origine : `#path-variables-and-persistent-data`)*

Vous ne savez pas où votre plugin sera installé ; référencez donc ses fichiers et ses données via ces variables plutôt que par des chemins fixes. Elles sont substituées dans le contenu des skills, commandes et agents, dans les commandes de hooks et de moniteurs, et dans les configurations de serveurs MCP et LSP. Elles sont aussi exportées vers les processus de hooks, MCP et LSP :

- **`${CLAUDE_PLUGIN_ROOT}`** : le répertoire d'installation du plugin. Chaque version a son propre [répertoire de cache](https://code.claude.com/docs/en/plugins/loading#find-plugins-on-disk), donc le chemin change quand le plugin est mis à jour. **N'y écrivez pas d'état.**
- **`${CLAUDE_PLUGIN_DATA}`** : un répertoire qui survit aux mises à jour, pour `node_modules`, les environnements virtuels et les caches. Il se résout en `~/.claude/plugins/data/<id>/` et est créé lors de sa première référence.
- **`${CLAUDE_PROJECT_DIR}`** : la racine du projet, la même valeur que reçoivent les hooks.

Dans le chemin du répertoire de données, `<id>` est l'identifiant du plugin dans lequel tout caractère autre que lettres, chiffres, `_` et `-` est remplacé par `-` ; `my-plugin@my-marketplace` devient donc `my-plugin-my-marketplace`.

Sous Windows, les chemins substitués utilisent des barres obliques (`/`) pour qu'un shell n'interprète pas les barres obliques inverses comme des échappements.

### Installer les dépendances dans le répertoire de données

Pour un plugin installé depuis une marketplace, Claude Code installe automatiquement les [dépendances de paquets Node.js](https://code.claude.com/docs/en/plugins/loading#node-js-package-dependencies) éligibles lorsqu'il met le plugin en cache ; vous n'avez donc peut-être pas besoin de les installer vous-même. Si c'est nécessaire, ce hook `SessionStart` installe `node_modules` dans `${CLAUDE_PLUGIN_DATA}` à la première exécution, puis à nouveau après une mise à jour qui modifie `package.json` :

```json hooks/hooks.json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "diff -q \"${CLAUDE_PLUGIN_ROOT}/package.json\" \"${CLAUDE_PLUGIN_DATA}/package.json\" >/dev/null 2>&1 || (cd \"${CLAUDE_PLUGIN_DATA}\" && cp \"${CLAUDE_PLUGIN_ROOT}/package.json\" . && npm install) || rm -f \"${CLAUDE_PLUGIN_DATA}/package.json\""
          }
        ]
      }
    ]
  }
}
```

Après la première session, `~/.claude/plugins/data/<id>/node_modules` existe. Un serveur MCP peut alors définir `NODE_PATH` à `${CLAUDE_PLUGIN_DATA}/node_modules` dans son `env`. Pour savoir quels champs substituent quelle variable, voir [Variables d'environnement](https://code.claude.com/docs/en/plugins/manifest-reference#environment-variables).

## Étapes suivantes

- [Référence du manifeste de plugin](https://code.claude.com/docs/en/plugins/manifest-reference) : champs de `plugin.json`, règles de chemins et disposition standard.
- [Tester les plugins avec des evals](https://code.claude.com/docs/en/plugin-evals) : vérifier que les composants ajoutés modifient le comportement de Claude comme prévu.
- [Publier et distribuer un plugin](https://code.claude.com/docs/en/plugins/publish) : versionner le plugin et le placer dans une marketplace.
- [Dépanner les plugins](https://code.claude.com/docs/en/plugins/troubleshooting) : que faire quand un composant ne se charge pas ou qu'un hook ne se déclenche pas.
