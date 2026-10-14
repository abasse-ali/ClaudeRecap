# Référence de l'Agent SDK — TypeScript — partie 3/4

> Source : https://code.claude.com/docs/en/agent-sdk/typescript · [← Index](../31-index-documentation-officielle.md)

Cette partie couvre les types d'entrée et de sortie des outils intégrés, les types de permissions et les « autres types » (messages système, événements, résultats de méthodes, erreurs). Les ancres `#…` renvoient aux autres sections de la même page officielle.

---

## Types d'entrée des outils (Tool Input Types)

Documentation des schémas d'entrée des outils intégrés de Claude Code. Ces types sont exportés depuis `@anthropic-ai/claude-agent-sdk/sdk-tools` et peuvent servir à des interactions typées avec les outils.

### `ToolInputSchemas`

Union des types d'entrée d'outils exportée depuis `@anthropic-ai/claude-agent-sdk/sdk-tools` ; ses membres incluent :

```typescript
type ToolInputSchemas =
  | AgentInput
  | ArtifactInput
  | AskUserQuestionInput
  | BashInput
  | CronCreateInput
  | CronDeleteInput
  | CronListInput
  | EnterPlanModeInput
  | EnterWorktreeInput
  | ExitPlanModeInput
  | ExitWorktreeInput
  | FileEditInput
  | FileReadInput
  | FileWriteInput
  | GlobInput
  | GrepInput
  | ListMcpResourcesInput
  | McpInput
  | MonitorInput
  | NotebookEditInput
  | ProjectsInput
  | PushNotificationInput
  | ReadMcpResourceDirInput
  | ReadMcpResourceInput
  | RefreshMcpToolsInput
  | RemoteTriggerInput
  | ReportFindingsInput
  | ScheduleWakeupInput
  | ShowOnboardingRolePickerInput
  | TaskCreateInput
  | TaskGetInput
  | TaskListInput
  | TaskStopInput
  | TaskUpdateInput
  | TodoWriteInput
  | WebFetchInput
  | WebSearchInput
  | WorkflowInput;
```

### Agent

**Nom de l'outil :** `Agent`. L'ancien nom `Task` est toujours accepté comme alias, et le tableau `tools` du message d'initialisation [`SDKSystemMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdksystemmessage) liste actuellement cet outil sous le nom `Task` pour la rétrocompatibilité.

> **Note** : le champ `mode` est obsolète et ignoré à partir de Claude Code v2.1.212. Un sous-agent s'exécute soit dans le mode de permission de la session parente, soit dans le [`permissionMode`](https://code.claude.com/docs/en/agent-sdk/typescript#agentdefinition) de sa définition ; ce sont les [règles d'héritage des sous-agents](https://code.claude.com/docs/en/agent-sdk/permissions#available-modes) qui tranchent.

```typescript
type AgentInput = {
  description: string;
  prompt: string;
  subagent_type?: string;
  model?: "sonnet" | "opus" | "haiku" | "fable";
  effort?: "low" | "medium" | "high" | "xhigh" | "max";
  run_in_background?: boolean;
  name?: string;
  team_name?: string; // Obsolète ; ignoré
  mode?: "acceptEdits" | "auto" | "bypassPermissions" | "default" | "dontAsk" | "plan"; // Obsolète ; ignoré. Les règles d'héritage des sous-agents décident du mode de permission d'un sous-agent
  isolation?: "worktree" | "remote";
};
```

Lance un nouvel agent chargé de traiter de façon autonome des tâches complexes en plusieurs étapes.

### AskUserQuestion

**Nom de l'outil :** `AskUserQuestion`

```typescript
type AskUserQuestionInput = {
  questions: Array<{
    question: string;
    header: string;
    options: Array<{ label: string; description: string; preview?: string }>;
    multiSelect: boolean;
  }>;
  answers?: Record<string, string>;
  annotations?: Record<string, { preview?: string; notes?: string }>;
  metadata?: { source?: string };
};
```

Pose des questions de clarification à l'utilisateur pendant l'exécution. Voir [Gérer les approbations et les saisies utilisateur](https://code.claude.com/docs/en/agent-sdk/user-input#handle-clarifying-questions) pour l'utilisation.

### Bash

**Nom de l'outil :** `Bash`

```typescript
type BashInput = {
  command: string;
  timeout?: number; // millisecondes. Premier plan : plafonné à 600000 par défaut, les valeurs supérieures sont ramenées au plafond. Avec run_in_background (Claude Code v2.1.285+) : limite de durée en arrière-plan, 1800000 si omis, plafonnée à 7200000 sauf relèvement
  description?: string;
  run_in_background?: boolean;
  dangerouslyDisableSandbox?: boolean;
};
```

Exécute des commandes Bash avec délai d'expiration optionnel et exécution en arrière-plan. Le répertoire de travail persiste entre les commandes, y compris pour les commandes lancées lors de tours ultérieurs d'une session multi-tours ; l'état du shell, comme les variables d'environnement exportées, ne persiste pas.

- Pour les limites sur les changements de répertoire qui sont conservés, voir [What persists between commands](https://code.claude.com/docs/en/tools-reference#what-persists-between-commands).
- Pour ce qui fixe le plafond au premier plan, voir [Timeout and output limits](https://code.claude.com/docs/en/tools-reference#timeout-and-output-limits).
- Pour la limite de durée en arrière-plan, voir [Time limit for background commands](https://code.claude.com/docs/en/tools-reference#time-limit-for-background-commands).

### Monitor

**Nom de l'outil :** `Monitor`

```typescript
type MonitorInput = {
  description: string;
  timeout_ms: number;
  command?: string;
  ws?: {
    url: string;
    protocols?: string[];
  };
};
```

Exécute une source en arrière-plan et transmet chaque événement à Claude pour qu'il réagisse sans interroger en boucle :

- `command` exécute un script et émet un événement par ligne de stdout ;
- `ws` ouvre un WebSocket et émet un événement par trame texte.

Fournissez exactement l'un des deux, `command` ou `ws`. La source `ws` nécessite Claude Code v2.1.195+.

`timeout_ms` est l'échéance de la surveillance, en millisecondes. Valeur par défaut : 300000 ; valeurs acceptées jusqu'à 3600000. L'échéance effective est d'au plus 1800000 (30 minutes) : une valeur acceptée plus grande est raccourcie à ce maximum. À l'échéance, la surveillance se termine et Claude reçoit un unique avis lui permettant d'en lancer une nouvelle si besoin.

Le type exporté marque `timeout_ms` comme obligatoire parce que le schéma remplit la valeur par défaut ; un appel qui l'omet reste valide.

Quand Monitor exécute une commande, il suit les mêmes règles de permission que Bash ; une surveillance WebSocket demande une approbation séparée. Voir la [référence de l'outil Monitor](https://code.claude.com/docs/en/tools-reference#monitor-tool) pour le comportement et la disponibilité selon le fournisseur.

### TaskOutput

Supprimé dans Claude Code v2.1.277, avec son type `TaskOutputInput`. Il servait auparavant à récupérer la sortie d'une tâche d'arrière-plan en cours ou terminée ; Claude lit désormais le fichier de sortie d'une tâche d'arrière-plan avec `Read`.

Une entrée `disallowedTools` ou une règle de refus (deny) qui nomme encore `TaskOutput` est ignorée sans avertissement.

### Edit

**Nom de l'outil :** `Edit`

```typescript
type FileEditInput = {
  file_path: string;
  old_string: string;
  new_string: string;
  replace_all?: boolean;
};
```

Effectue des remplacements de chaînes exacts dans des fichiers.

### Read

**Nom de l'outil :** `Read`

```typescript
type FileReadInput = {
  file_path: string;
  offset?: number;
  limit?: number;
  pages?: string;
  allow_large?: boolean;
};
```

Lit des fichiers du système de fichiers local, y compris texte, images, PDF et notebooks Jupyter.

* Utilisez `pages` pour des plages de pages PDF (par exemple `"1-5"`).
* `allow_large` permet à Claude de lire un fichier texte au-delà des [limites de taille habituelles](https://code.claude.com/docs/en/tools-reference#large-files) lorsque la fenêtre de contexte de la session a de la place. Nécessite Agent SDK v0.3.296+ (qui embarque Claude Code v2.1.296).

Pour un PDF, Claude reçoit le contenu du fichier à l'intérieur du contenu `tool_result` de l'appel Read :

- une lecture qui renvoie la [sortie](https://code.claude.com/docs/en/agent-sdk/typescript#tool-output-types) `pdf` contient un bloc `text` de résumé suivi d'un bloc `document` ;
- une lecture qui renvoie la sortie `parts` contient le bloc `text` de résumé suivi d'un bloc par page extraite : un bloc `image`, ou un bloc `text` nommant la page lorsque Claude Code n'a pas pu la rendre en image.

Avant Agent SDK v0.3.242, Claude Code transmettait le contenu du fichier dans un message `user` séparé, après le résultat de l'outil.

### Write

**Nom de l'outil :** `Write`

```typescript
type FileWriteInput = {
  file_path: string;
  content: string;
};
```

Écrit un fichier sur le système de fichiers local, en l'écrasant s'il existe.

### Glob

**Nom de l'outil :** `Glob`

```typescript
type GlobInput = {
  pattern: string;
  path?: string;
};
```

Correspondance rapide de motifs de fichiers, quelle que soit la taille de la base de code.

### Grep

**Nom de l'outil :** `Grep`

```typescript
type GrepInput = {
  pattern: string;
  path?: string;
  glob?: string;
  type?: string;
  output_mode?: "content" | "files_with_matches" | "count";
  "-i"?: boolean;
  "-o"?: boolean; // n'affiche que les parties correspondantes de chaque ligne ; nécessite output_mode: "content"
  "-n"?: boolean;
  "-B"?: number;
  "-A"?: number;
  "-C"?: number;
  context?: number;
  head_limit?: number;
  offset?: number;
  multiline?: boolean;
};
```

Outil de recherche fondé sur ripgrep, avec prise en charge des expressions régulières.

### TaskStop

**Nom de l'outil :** `TaskStop`

```typescript
type TaskStopInput = {
  task_id?: string;
  shell_id?: string; // Obsolète : utiliser task_id
};
```

Arrête une tâche ou un shell d'arrière-plan en cours, par ID. Depuis v2.1.198, `task_id` accepte aussi un coéquipier d'agent team ou un agent d'arrière-plan nommé, désigné par son ID d'agent ou son nom.

### NotebookEdit

**Nom de l'outil :** `NotebookEdit`

```typescript
type NotebookEditInput = {
  notebook_path: string;
  cell_id?: string;
  new_source: string;
  cell_type?: "code" | "markdown";
  edit_mode?: "replace" | "insert" | "delete";
};
```

Modifie des cellules de notebooks Jupyter.

### WebFetch

**Nom de l'outil :** `WebFetch`

```typescript
type WebFetchInput = {
  url: string;
  prompt: string;
  offset?: number;
};
```

Récupère le contenu d'une URL et le traite avec un modèle d'IA.

`offset` est le nombre de caractères à sauter depuis le début de la page. Claude le définit pour poursuivre la lecture d'une longue page. Ce champ nécessite Agent SDK v0.3.290+.

### WebSearch

**Nom de l'outil :** `WebSearch`

```typescript
type WebSearchInput = {
  query: string;
  allowed_domains?: string[];
  blocked_domains?: string[];
};
```

Effectue une recherche sur le web et renvoie des résultats formatés.

### Workflow

**Nom de l'outil :** `Workflow`

```typescript
type WorkflowInput = {
  script?: string;
  name?: string;
  scriptPath?: string;
  args?: unknown; // n'importe quelle valeur JSON ; les typings publiés l'affichent comme une table d'objets
  resumeFromRunId?: string;
  title?: string; // ignoré ; c'est le bloc meta du script qui fixe le titre
  description?: string; // ignoré ; c'est le bloc meta du script qui fixe la description
};
```

Exécute un [workflow dynamique](https://code.claude.com/docs/en/workflows) : un script qui orchestre de nombreux sous-agents en arrière-plan et renvoie un résultat consolidé unique. L'outil `Workflow` est disponible à partir d'Agent SDK v0.3.149. Au moins l'un des champs `script`, `name` ou `scriptPath` est obligatoire.

| Champ | Type | Description |
| - | - | - |
| `script` | `string` | Script de workflow en ligne. Doit commencer par `export const meta = { name, description }` sous forme littérale, suivi du corps du script utilisant `agent()`, `parallel()`, `pipeline()` et `phase()`. Un tableau `phases` optionnel dans `meta` regroupe les agents sous des étapes nommées dans la vue de progression |
| `name` | `string` | Nom d'un workflow intégré ou enregistré dans `.claude/workflows/`. Résolu en script |
| `scriptPath` | `string` | Chemin d'un fichier de script de workflow sur disque, par exemple le `scriptPath` renvoyé par une exécution précédente. Prioritaire sur `script` et `name`. Claude Code rejette `scriptPath` avec une erreur si les outils de la session n'incluent pas `Read` |
| `args` | `unknown` | Valeur d'entrée exposée au script via la variable globale `args`, pour des workflows nommés paramétrés (une question de recherche, une liste de chemins de fichiers…). Passez tableaux et objets comme vraies valeurs JSON, pas comme chaîne encodée en JSON |
| `resumeFromRunId` | `string` | ID d'exécution d'une invocation `Workflow` antérieure à reprendre. Les appels `agent()` terminés dont les entrées n'ont pas changé renvoient généralement des résultats en cache ; les autres s'exécutent réellement. [Resume after a pause](https://code.claude.com/docs/en/workflows#resume-after-a-pause) précise quels appels terminés sont relancés. Même session uniquement |
| `title` | `string` | Ignoré ; c'est le bloc `meta` du script qui fixe le titre |
| `description` | `string` | Ignoré ; c'est le bloc `meta` du script qui fixe la description |

### TodoWrite

**Nom de l'outil :** `TodoWrite`

```typescript
type TodoWriteInput = {
  todos: Array<{
    content: string;
    status: "pending" | "in_progress" | "completed";
    activeForm: string;
  }>;
};
```

Crée et gère une liste de tâches structurée pour suivre l'avancement.

> **Note** : les outils suivants ne sont disponibles par défaut que sur Claude Opus 4.7, Claude Sonnet 4.6, Claude Haiku 4.5 et les modèles antérieurs. Sur tout autre modèle, y compris les ID de modèle que Claude Code ne reconnaît pas, ils ne sont pas disponibles sauf activation explicite (opt-in) :
>
> * `TodoWrite`
> * `TaskCreate`
> * `TaskGet`
> * `TaskUpdate`
> * `TaskList`
>
> Partout où ces outils sont disponibles, Claude Code fournit les quatre outils Task, ou `TodoWrite` à la place si vous définissez `CLAUDE_CODE_ENABLE_TASKS=0`.
>
> Cet ensemble par défaut s'applique à partir de Claude Code v2.1.268, que le SDK Agent TypeScript embarque à partir de v0.3.268.
>
> Voir [Model availability](https://code.claude.com/docs/en/agent-sdk/todo-tracking#model-availability) pour activer ces outils.

### TaskCreate

**Nom de l'outil :** `TaskCreate`

```typescript
type TaskCreateInput = {
  subject: string;
  description: string;
  activeForm?: string;
  metadata?: Record<string, unknown>;
};
```

Crée une tâche unique et renvoie l'ID qui lui est attribué.

### TaskUpdate

**Nom de l'outil :** `TaskUpdate`

```typescript
type TaskUpdateInput = {
  taskId: string;
  status?: "pending" | "in_progress" | "completed" | "deleted";
  subject?: string;
  description?: string;
  activeForm?: string;
  addBlocks?: string[];
  addBlockedBy?: string[];
  owner?: string;
  metadata?: Record<string, unknown>;
};
```

Modifie partiellement une tâche par son ID. Définissez `status` à `"deleted"` pour la supprimer.

### TaskGet

**Nom de l'outil :** `TaskGet`

```typescript
type TaskGetInput = {
  taskId: string;
};
```

Renvoie le détail complet d'une tâche, ou `null` si l'ID est introuvable.

### TaskList

**Nom de l'outil :** `TaskList`

```typescript
type TaskListInput = {};
```

Renvoie un instantané de toutes les tâches de la liste courante.

### ExitPlanMode

**Nom de l'outil :** `ExitPlanMode`

```typescript
type ExitPlanModeInput = {
  /** Obsolète : n'est plus utilisé. */
  allowedPrompts?: Array<{
    tool: "Bash";
    prompt: string;
  }>;
  [k: string]: unknown;
};
```

Quitte le mode plan. Le champ `allowedPrompts` est obsolète et ignoré ; Claude Code l'accepte encore pour que les appelants et transcriptions existants restent valides. Avant v2.1.205, il servait à demander des permissions Bash basées sur des prompts pour mettre en œuvre le plan.

### ListMcpResources

**Nom de l'outil :** `ListMcpResourcesTool`

```typescript
type ListMcpResourcesInput = {
  server?: string;
};
```

Liste les ressources MCP disponibles sur les serveurs connectés.

### ReadMcpResource

**Nom de l'outil :** `ReadMcpResourceTool`

```typescript
type ReadMcpResourceInput = {
  server: string;
  uri: string;
};
```

Lit une ressource MCP précise sur un serveur.

### EnterWorktree

**Nom de l'outil :** `EnterWorktree`

```typescript
type EnterWorktreeInput = {
  name?: string;
  path?: string;
};
```

Crée un worktree git temporaire et y entre pour travailler de façon isolée. Passez `path` pour basculer dans un worktree existant au lieu d'en créer un nouveau. Conditions sur la cible :

- à la première entrée, la cible doit être un worktree enregistré du dépôt courant ou, dans un espace de travail multi-dépôts, d'un dépôt imbriqué dedans ;
- depuis une session déjà dans un worktree, elle doit se trouver sous `.claude/worktrees/` du dépôt de la session.

`name` et `path` sont mutuellement exclusifs.

### ExitWorktree

**Nom de l'outil :** `ExitWorktree`

```typescript
type ExitWorktreeInput = {
  action: "keep" | "remove";
  discard_changes?: boolean;
};
```

Quitte le worktree git courant et revient au répertoire de travail d'origine. L'action `keep` laisse le worktree et la branche sur disque, tandis que `remove` supprime les deux. `discard_changes` doit valoir `true` pour supprimer un worktree contenant des fichiers non commités ou des commits non fusionnés.

### EnterPlanMode

**Nom de l'outil :** `EnterPlanMode`

```typescript
type EnterPlanModeInput = {};
```

Entre en mode plan, où Claude étudie la situation et présente un plan avant d'effectuer des modifications.

### CronCreate

**Nom de l'outil :** `CronCreate`

```typescript
type CronCreateInput = {
  cron: string;
  prompt: string;
  recurring?: boolean;
  durable?: boolean;
};
```

Planifie l'exécution d'un prompt selon une expression cron à 5 champs, en heure locale. Définissez `recurring` à `false` pour un déclenchement unique à la prochaine correspondance. Les tâches sont par défaut limitées à la session ; une reprise avec `--resume` ou `--continue` restaure les tâches non expirées. Voir [Scheduled tasks](https://code.claude.com/docs/en/scheduled-tasks).

Définir `durable` à `true` demande une persistance dans `.claude/scheduled_tasks.json` pour que la tâche survive aux redémarrages. La planification durable n'est pas disponible dans toutes les sessions : dans ce cas, Claude Code accepte `durable: true` mais crée la tâche pour la session seulement. Lisez le champ `durable` de la sortie pour savoir si la tâche a été persistée.

### CronDelete

**Nom de l'outil :** `CronDelete`

```typescript
type CronDeleteInput = {
  id: string;
};
```

Supprime une tâche cron planifiée à partir de l'ID renvoyé par `CronCreate`.

### CronList

**Nom de l'outil :** `CronList`

```typescript
type CronListInput = {};
```

Liste les tâches cron planifiées : les tâches durables issues de `.claude/scheduled_tasks.json` et les tâches limitées à la session courante.

### ScheduleWakeup

**Nom de l'outil :** `ScheduleWakeup`

```typescript
type ScheduleWakeupInput = {
  delaySeconds?: number;
  reason?: string;
  prompt?: string;
  noop?: boolean;
  stop?: boolean;
};
```

Planifie un réveil unique qui déclenche le prompt donné après un délai. Cet outil sous-tend la commande `/loop` auto-cadencée.

- Le runtime borne `delaySeconds` entre 60 et 3600 secondes.
- Les champs `delaySeconds`, `reason`, `prompt` et `noop` sont obligatoires sauf si `stop` vaut true.
- `noop: true` signale un réveil où rien n'a changé.
- `stop: true` annule le réveil en attente et met fin au `/loop` auto-cadencé. Le champ `stop` nécessite Claude Code v2.1.202+.

Voir la [ligne ScheduleWakeup de la référence des outils](https://code.claude.com/docs/en/tools-reference).

### RemoteTrigger

**Nom de l'outil :** `RemoteTrigger`

```typescript
type RemoteTriggerInput = {
  action:
    | "list"
    | "get"
    | "create"
    | "update"
    | "run"
    | "create_webhook_trigger"
    | "list_runs"
    | "get_run_log";
  trigger_id?: string;
  session_id?: string;
  cursor?: string;
  body?: {
    [k: string]: unknown;
  };
};
```

Gère les [Routines](https://code.claude.com/docs/en/routines), c'est-à-dire les exécutions Claude Code planifiées et déclenchées, hébergées dans le cloud. Cet outil sous-tend la commande `/schedule`.

- `trigger_id` est obligatoire pour les actions `get`, `update`, `run` et `list_runs`.
- `body` est obligatoire pour `create`, `update` et `create_webhook_trigger`, et optionnel pour `run`.
- `create_webhook_trigger` attache une source d'événements à une routine existante, par exemple un [événement GitHub](https://code.claude.com/docs/en/routines#add-a-github-trigger) qui la déclenche. Le `body` nomme la source, les événements et la routine à déclencher. Nécessite Claude Code v2.1.225+.
- `list_runs` liste les exécutions récentes d'une routine, et `get_run_log` lit le journal d'une exécution. `session_id` désigne l'exécution à lire (issue d'un résultat `list_runs`), et `cursor` pagine les résultats des deux actions. Les deux actions nécessitent Claude Code v2.1.227+.

Disponibilité :

- l'outil n'est disponible que si la session est authentifiée avec un compte claude.ai sur un forfait où les Routines sont activées ;
- il est absent quand la politique de votre organisation désactive les [sessions cloud](https://code.claude.com/docs/en/claude-code-on-the-web) ;
- à partir de Claude Code v2.1.227, il est aussi absent quand un Owner a [désactivé les routines pour l'organisation](https://code.claude.com/docs/en/routines#routines-are-disabled-by-your-organizations-policy). Avant v2.1.227, une session dont seul l'interrupteur des routines était désactivé affichait encore l'outil, et le serveur refusait ses appels.

### PushNotification

**Nom de l'outil :** `PushNotification`

```typescript
type PushNotificationInput = {
  message: string;
  status: "proactive";
};
```

Envoie une notification push proactive à l'utilisateur. Gardez `message` sous 200 caractères, car les systèmes d'exploitation mobiles tronquent les textes plus longs. Voir la [ligne PushNotification de la référence des outils](https://code.claude.com/docs/en/tools-reference) pour la disponibilité selon le fournisseur : l'envoi des push passe par une infrastructure hébergée par Anthropic, inaccessible depuis Amazon Bedrock, Claude Platform on AWS, Google Cloud's Agent Platform ou Microsoft Foundry.

### REPL

Supprimé dans v2.1.275. Jusqu'à v2.1.274, un outil expérimental `REPL` pouvait être activé avec `CLAUDE_CODE_REPL=1` dans l'[option `env`](https://code.claude.com/docs/en/agent-sdk/typescript#options).

### ReportFindings

**Nom de l'outil :** `ReportFindings`

```typescript
type ReportFindingsInput = {
  level?: "low" | "medium" | "high" | "xhigh" | "max";
  findings: Array<{
    file: string;
    line?: number;
    summary: string;
    failure_scenario: string;
    short_summary?: string;
    category?: string;
    verdict?: "CONFIRMED" | "PLAUSIBLE";
    outcome?: "fixed" | "skipped" | "no_change_needed";
  }>;
};
```

Signale les constats d'une revue de code sous forme de liste structurée, pour que Claude Code puisse les afficher au lieu de les imprimer en texte. Les constats sont triés du plus grave au moins grave, au plus 32 par appel, et le tableau est vide si aucun n'a survécu. Nécessite Claude Code v2.1.196+.

`level` est optionnel et contient le niveau d'effort que Claude indique pour la revue. Claude Code ne le compare pas au niveau auquel la revue a réellement tourné ; les deux peuvent donc différer.

Chaque constat porte les champs suivants :

* `file` : chemin relatif au dépôt du fichier concerné. Le champ optionnel `line` est la ligne (numérotée à partir de 1) à laquelle il s'ancre.
* `summary` : énoncé du défaut en une phrase. `failure_scenario` décrit les entrées et l'état concrets qui mènent à une sortie erronée ou à un plantage.
* `short_summary` : libellé compressé optionnel d'au plus 60 caractères pour un affichage compact. Nécessite Claude Code v2.1.212+.
* `category` : identifiant court optionnel en kebab-case du type de constat, par exemple `correctness` ou `test-coverage`. Nécessite Claude Code v2.1.199+.
* `verdict` : défini lorsqu'une passe de vérification a eu lieu ; absent pour les revues uniquement en ligne.
* `outcome` : défini uniquement lors d'un nouveau signalement après application des corrections.

### Artifact

**Nom de l'outil :** `Artifact`

```typescript
type ArtifactInput = {
  action?: "publish" | "list";
  file_path?: string;
  favicon?: string;
  icon?: string;
  limit?: number;
  scope?: "mine" | "shared" | "all";
  title?: string;
  description?: string;
  label?: string;
  url?: string;
  force?: boolean;
  capabilities?: Record<string, unknown>;
  contract?: "latest" | string;
};
```

Publie un fichier local `.html` ou `.md` comme page d'artefact hébergée, ou liste les artefacts publiés de l'utilisateur. Omettez `action` ou passez `"publish"` pour publier `file_path`, obligatoire pour l'action de publication. Chacun des champs ci-dessous s'applique à une publication :

* `icon` : un mot court et générique pour l'icône d'onglet du navigateur, comme `chart` ou `map`. Claude l'inclut lors d'une première publication et l'omet lors d'une mise à jour, ce qui conserve l'icône enregistrée.
* `favicon` : obsolète ; Claude l'omet.
* `title` : nomme la page publiée dans l'onglet du navigateur et la galerie lorsque le fichier HTML n'a pas de balise `<title>`.
* `url` : cible un artefact existant à mettre à jour sur place au lieu d'en créer un nouveau.

`force` est un écrasement de dernier recours qui supprime une version plus récente publiée par une autre session. En cas de conflit, la publication échouée renvoie le contenu plus récent ; Claude fusionne ses modifications sur ce contenu, ou relit l'artefact, puis republie. Ne passez `force` que si l'utilisateur demande explicitement d'abandonner cette version.

Passez `"list"` pour énumérer les artefacts publiés de l'utilisateur ; seuls `limit` et `scope` peuvent l'accompagner. `scope` vaut `"mine"` par défaut (artefacts dont l'utilisateur est propriétaire) ; `"shared"` liste les artefacts que d'autres personnes ont partagés avec l'utilisateur, et `"all"` liste les deux.

`limit` fixe le nombre maximal d'artefacts renvoyés par une liste, de 1 à 200. Un `limit` supérieur à 50 nécessite Agent SDK v0.3.292+. Sans `limit`, une liste renvoie jusqu'à 25 artefacts.

* `capabilities` : les capacités d'exécution utilisées par la page publiée, indexées par nom de capacité, comme les [connecteurs que la page peut appeler](https://code.claude.com/docs/en/artifacts#pull-live-data-with-mcp-connectors). Le service d'artefacts valide la déclaration et rejette une publication qui nomme une capacité inutilisable par le compte ou qui lui donne une configuration invalide. Passez `{}` pour effacer une déclaration enregistrée, et omettez le champ lors d'un redéploiement pour la conserver. Nécessite Agent SDK v0.3.235+.
* `contract` : la version d'exécution contre laquelle tourne la page publiée. Omettez-le pour garder la version actuelle de l'artefact, passez `"latest"` pour mettre à niveau, ou une version précise pour épingler ou revenir en arrière. Nécessite Agent SDK v0.3.235+.

Les types sont exportés, mais l'outil est désactivé par défaut dans les sessions Agent SDK. La publication exige aussi toutes les conditions du [tableau de disponibilité des artefacts](https://code.claude.com/docs/en/artifacts#availability), que les sessions authentifiées par clé API ne remplissent pas.

### Projects

**Nom de l'outil :** `Projects`

```typescript
type ProjectsInput = {
  method:
    | "project_info"
    | "project_read"
    | "project_search"
    | "project_write"
    | "project_delete";
  path?: string;
  content?: string;
  local_path?: string;
  present_to_user?: boolean;
  query?: string;
  n?: number;
};
```

Lit et écrit le Projet claude.ai rattaché à la session. Le comportement dépend de `method` :

* `project_info` : renvoie les métadonnées du projet et la liste des documents.
* `project_read` : lit un document par son `path`.
* `project_search` : interroge la base de connaissances du projet avec `query`. `n` plafonne le nombre de résultats et vaut `5` par défaut.
* `project_write` : crée ou remplace un document à `path` à partir d'exactement l'un de : `content` (texte en ligne) ou `local_path` (fichier situé dans le répertoire de travail). `present_to_user: true` marque le document écrit comme le livrable que l'utilisateur doit voir.
* `project_delete` : supprime un document par son `path`.

### ReadMcpResourceDir

**Nom de l'outil :** `ReadMcpResourceDirTool`

```typescript
type ReadMcpResourceDirInput = {
  server: string;
  uri: string;
};
```

Liste les enfants directs d'une ressource de type répertoire sur un serveur MCP. Utilisable uniquement avec un serveur ayant déclaré prendre en charge le listage de répertoires ; le listage n'est pas récursif. Le listage de répertoires n'est pas activé dans toutes les sessions : lorsqu'il est désactivé, l'appel renvoie une liste `resources` vide et le champ `error` indique que le listage de répertoires n'est pas activé.

### RefreshMcpTools

**Nom de l'outil :** `RefreshMcpTools`

```typescript
type RefreshMcpToolsInput = {
  server?: string; // ne rafraîchit que ce serveur ; omettre pour rafraîchir tous les serveurs connectés
};
```

Réinterroge la liste d'outils des serveurs MCP connectés et applique les changements éventuels. Les types sont exportés, mais Claude Code n'enregistre l'outil que si vous définissez `CLAUDE_CODE_ENABLE_REFRESH_MCP_TOOLS=1` dans l'[option `env`](https://code.claude.com/docs/en/agent-sdk/typescript#options), et seulement dans les sessions comptant au moins un serveur MCP. Nécessite Claude Code v2.1.211+.

### ShowOnboardingRolePicker

**Nom de l'outil :** `ShowOnboardingRolePicker`

```typescript
type ShowOnboardingRolePickerInput = {};
```

Affiche une rangée de puces cliquables de choix de rôle pendant l'onboarding Cowork, pour que l'utilisateur choisisse son rôle et obtienne l'installation d'un plugin adapté. Ne prend aucun argument ; la liste des rôles est définie par le client. L'appel bloque jusqu'à la réponse de l'utilisateur.

### McpInput

**Nom de l'outil :** noms dynamiques d'outils MCP de la forme `mcp__<server>__<tool>`

```typescript
type McpInput = {
  [k: string]: unknown;
};
```

Les arguments d'un outil MCP forment un objet ouvert : chaque serveur définit ses propres paramètres, le type n'impose donc aucune contrainte sur les noms ou valeurs des champs. Consultez le schéma d'outil du serveur pour connaître les champs qu'accepte un outil donné.

