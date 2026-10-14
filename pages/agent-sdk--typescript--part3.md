# Référence de l'Agent SDK — TypeScript — partie 3/4

> Source : https://code.claude.com/docs/en/agent-sdk/typescript · [← Index](../31-index-documentation-officielle.md)

Cette partie couvre les types d'entrée des outils intégrés, les types de sortie des outils, les types de permissions et les autres types exportés (messages système, événements, statut MCP, usage, etc.).

## Types d'entrée des outils (Tool Input Types)

Documentation des schémas d'entrée des outils intégrés de Claude Code. Ces types sont exportés depuis `@anthropic-ai/claude-agent-sdk/sdk-tools` et peuvent servir à des interactions typées avec les outils.

### `ToolInputSchemas`

Union des types d'entrée d'outils exportés depuis `@anthropic-ai/claude-agent-sdk/sdk-tools` ; ses membres incluent :

```typescript theme={null}
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

**Nom de l'outil :** `Agent`. L'ancien nom `Task` reste accepté comme alias, et le tableau `tools` du message d'initialisation [`SDKSystemMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdksystemmessage) liste actuellement cet outil sous le nom `Task` pour la rétrocompatibilité.

> **Note** : le champ `mode` est déprécié et ignoré à partir de Claude Code v2.1.212. Un sous-agent s'exécute soit dans le mode de permission de la session parente, soit dans le [`permissionMode`](https://code.claude.com/docs/en/agent-sdk/typescript#agentdefinition) de sa définition ; ce sont les [règles d'héritage des sous-agents](https://code.claude.com/docs/en/agent-sdk/permissions#available-modes) qui tranchent.

```typescript theme={null}
type AgentInput = {
  description: string;
  prompt: string;
  subagent_type?: string;
  model?: "sonnet" | "opus" | "haiku" | "fable";
  effort?: "low" | "medium" | "high" | "xhigh" | "max";
  run_in_background?: boolean;
  name?: string;
  team_name?: string; // Déprécié ; ignoré
  mode?: "acceptEdits" | "auto" | "bypassPermissions" | "default" | "dontAsk" | "plan"; // Déprécié ; ignoré. Les règles d'héritage des sous-agents décident du mode de permission d'un sous-agent
  isolation?: "worktree" | "remote";
};
```

Lance un nouvel agent chargé de traiter de façon autonome des tâches complexes en plusieurs étapes.

### AskUserQuestion

**Nom de l'outil :** `AskUserQuestion`

```typescript theme={null}
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

Pose à l'utilisateur des questions de clarification pendant l'exécution. Voir [Gérer les approbations et les saisies utilisateur](https://code.claude.com/docs/en/agent-sdk/user-input#handle-clarifying-questions) pour les détails d'utilisation.

### Bash

**Nom de l'outil :** `Bash`

```typescript theme={null}
type BashInput = {
  command: string;
  timeout?: number; // millisecondes. Premier plan : plafonné à 600000 par défaut, les valeurs supérieures sont ramenées au plafond. Avec run_in_background (Claude Code v2.1.285+) : la limite de temps en arrière-plan, 1800000 si omis, plafonnée à 7200000 sauf relèvement
  description?: string;
  run_in_background?: boolean;
  dangerouslyDisableSandbox?: boolean;
};
```

Exécute des commandes Bash avec un délai d'expiration optionnel et une exécution en arrière-plan possible. Le répertoire de travail persiste entre les commandes, y compris pour les commandes lancées lors des tours suivants d'une session multi-tours ; l'état du shell, comme les variables d'environnement exportées, ne persiste pas. Pour les limites concernant les changements de répertoire conservés, voir [Ce qui persiste entre les commandes](https://code.claude.com/docs/en/tools-reference#what-persists-between-commands). Pour ce qui fixe le plafond en premier plan, voir [Délais d'expiration et limites de sortie](https://code.claude.com/docs/en/tools-reference#timeout-and-output-limits). Pour la limite de temps en arrière-plan, voir [Limite de temps des commandes en arrière-plan](https://code.claude.com/docs/en/tools-reference#time-limit-for-background-commands).

### Monitor

**Nom de l'outil :** `Monitor`

```typescript theme={null}
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

Exécute une source en arrière-plan et transmet chaque événement à Claude pour qu'il puisse réagir sans interrogation périodique (polling) : `command` exécute un script et émet un événement par ligne de stdout, et `ws` ouvre un WebSocket et émet un événement par trame texte. Fournissez exactement l'un des deux, `command` ou `ws`. La source `ws` nécessite Claude Code v2.1.195+.

`timeout_ms` est l'échéance de la surveillance en millisecondes. Valeur par défaut : 300000 ; valeurs acceptées jusqu'à 3600000. L'échéance effective est au plus de 1800000, soit 30 minutes : une valeur acceptée plus grande est raccourcie à ce maximum. À l'échéance, la surveillance se termine et Claude reçoit un avis unique pour pouvoir en relancer une s'il en a encore besoin.

Le type exporté marque `timeout_ms` comme obligatoire parce que le schéma remplit la valeur par défaut ; un appel qui l'omet est quand même validé.

Quand Monitor exécute une commande, il suit les mêmes règles de permission que Bash ; une surveillance WebSocket demande une approbation séparée. Voir la [référence de l'outil Monitor](https://code.claude.com/docs/en/tools-reference#monitor-tool) pour le comportement et la disponibilité selon le fournisseur.

### TaskOutput

Supprimé dans Claude Code v2.1.277, avec son type `TaskOutputInput`. Il récupérait auparavant la sortie d'une tâche d'arrière-plan en cours ou terminée ; Claude lit désormais le fichier de sortie d'une tâche d'arrière-plan avec `Read`.

Une entrée `disallowedTools` ou une règle de refus (deny) qui nomme encore `TaskOutput` est ignorée sans avertissement.

### Edit

**Nom de l'outil :** `Edit`

```typescript theme={null}
type FileEditInput = {
  file_path: string;
  old_string: string;
  new_string: string;
  replace_all?: boolean;
};
```

Effectue des remplacements exacts de chaînes dans des fichiers.

### Read

**Nom de l'outil :** `Read`

```typescript theme={null}
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
* `allow_large` permet à Claude de lire un fichier texte au-delà des [limites de taille habituelles](https://code.claude.com/docs/en/tools-reference#large-files) quand la fenêtre de contexte de la session a de la place. Nécessite l'Agent SDK v0.3.296+, qui embarque Claude Code v2.1.296.

Pour un PDF, Claude reçoit le contenu du fichier dans le contenu `tool_result` de l'appel Read. Une lecture qui renvoie la [sortie](https://code.claude.com/docs/en/agent-sdk/typescript#tool-output-types) `pdf` contient un bloc `text` de résumé suivi d'un bloc `document`. Une lecture qui renvoie la sortie `parts` contient le bloc `text` de résumé suivi d'un bloc par page extraite : un bloc `image`, ou un bloc `text` nommant la page quand Claude Code n'a pas pu la rendre en image. Avant l'Agent SDK v0.3.242, Claude Code livrait le contenu du fichier dans un message `user` distinct après le résultat de l'outil.

### Write

**Nom de l'outil :** `Write`

```typescript theme={null}
type FileWriteInput = {
  file_path: string;
  content: string;
};
```

Écrit un fichier sur le système de fichiers local, en l'écrasant s'il existe.

### Glob

**Nom de l'outil :** `Glob`

```typescript theme={null}
type GlobInput = {
  pattern: string;
  path?: string;
};
```

Recherche rapide de fichiers par motif, qui fonctionne quelle que soit la taille de la base de code.

### Grep

**Nom de l'outil :** `Grep`

```typescript theme={null}
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

Outil de recherche basé sur ripgrep, avec prise en charge des expressions régulières.

### TaskStop

**Nom de l'outil :** `TaskStop`

```typescript theme={null}
type TaskStopInput = {
  task_id?: string;
  shell_id?: string; // Déprécié : utilisez task_id
};
```

Arrête une tâche ou un shell d'arrière-plan en cours d'exécution, par ID. Depuis la v2.1.198, `task_id` accepte aussi un coéquipier d'une équipe d'agents (agent team) ou un agent d'arrière-plan nommé, par ID d'agent ou par nom.

### NotebookEdit

**Nom de l'outil :** `NotebookEdit`

```typescript theme={null}
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

```typescript theme={null}
type WebFetchInput = {
  url: string;
  prompt: string;
  offset?: number;
};
```

Récupère le contenu d'une URL et le traite avec un modèle d'IA.

`offset` est le nombre de caractères à sauter depuis le début de la page. Claude le définit pour continuer la lecture d'une page longue. Ce champ nécessite l'Agent SDK v0.3.290+.

### WebSearch

**Nom de l'outil :** `WebSearch`

```typescript theme={null}
type WebSearchInput = {
  query: string;
  allowed_domains?: string[];
  blocked_domains?: string[];
};
```

Recherche sur le web et renvoie des résultats formatés.

### Workflow

**Nom de l'outil :** `Workflow`

```typescript theme={null}
type WorkflowInput = {
  script?: string;
  name?: string;
  scriptPath?: string;
  args?: unknown; // n'importe quelle valeur JSON ; les typages publiés la présentent comme une table d'objets
  resumeFromRunId?: string;
  title?: string; // ignoré ; le bloc meta du script définit le titre
  description?: string; // ignoré ; le bloc meta du script définit la description
};
```

Exécute un [workflow dynamique](https://code.claude.com/docs/en/workflows) : un script qui orchestre de nombreux sous-agents en arrière-plan et renvoie un résultat consolidé unique. L'outil `Workflow` est disponible dans l'Agent SDK v0.3.149+. Au moins l'un de `script`, `name` ou `scriptPath` est requis.

| Champ | Type | Description |
| - | - | - |
| `script` | `string` | Script de workflow en ligne. Doit commencer par `export const meta = { name, description }` sous forme littérale, suivi du corps du script utilisant `agent()`, `parallel()`, `pipeline()` et `phase()`. Un tableau `phases` optionnel dans `meta` regroupe les agents sous des étapes nommées dans la vue de progression |
| `name` | `string` | Nom d'un workflow intégré ou d'un workflow enregistré dans `.claude/workflows/`. Résolu en script |
| `scriptPath` | `string` | Chemin vers un fichier de script de workflow sur disque, comme le `scriptPath` renvoyé par une exécution précédente. Prioritaire sur `script` et `name`. Claude Code rejette `scriptPath` avec une erreur quand les outils de la session n'incluent pas `Read` |
| `args` | `unknown` | Valeur d'entrée exposée au script comme variable globale `args`, pour les workflows nommés paramétrés, comme une question de recherche ou une liste de chemins de fichiers. Passez tableaux et objets comme de vraies valeurs JSON, pas comme une chaîne encodée en JSON |
| `resumeFromRunId` | `string` | ID d'exécution d'une invocation `Workflow` antérieure à reprendre. Les appels `agent()` terminés dont les entrées n'ont pas changé renvoient généralement des résultats en cache ; les autres s'exécutent réellement. [Reprendre après une pause](https://code.claude.com/docs/en/workflows#resume-after-a-pause) précise quels appels terminés sont réexécutés. Même session uniquement |
| `title` | `string` | Ignoré ; le bloc `meta` du script définit le titre |
| `description` | `string` | Ignoré ; le bloc `meta` du script définit la description |

### TodoWrite

**Nom de l'outil :** `TodoWrite`

```typescript theme={null}
type TodoWriteInput = {
  todos: Array<{
    content: string;
    status: "pending" | "in_progress" | "completed";
    activeForm: string;
  }>;
};
```

Crée et gère une liste de tâches structurée pour suivre la progression.

> **Note** : les outils suivants ne sont disponibles par défaut que sur Claude Opus 4.7, Claude Sonnet 4.6, Claude Haiku 4.5 et les modèles antérieurs. Sur tout autre modèle, y compris les ID de modèle que Claude Code ne reconnaît pas, ils ne sont pas disponibles sauf activation explicite (opt-in) :
>
> * `TodoWrite`
> * `TaskCreate`
> * `TaskGet`
> * `TaskUpdate`
> * `TaskList`
>
> Là où ces outils sont disponibles, Claude Code fournit les quatre outils Task, ou `TodoWrite` à leur place quand vous définissez `CLAUDE_CODE_ENABLE_TASKS=0`.
>
> Cet ensemble par défaut s'applique à Claude Code v2.1.268+, que l'Agent SDK TypeScript embarque à partir de la v0.3.268.
>
> Voir [Disponibilité selon le modèle](https://code.claude.com/docs/en/agent-sdk/todo-tracking#model-availability) pour activer ces outils.

### TaskCreate

**Nom de l'outil :** `TaskCreate`

```typescript theme={null}
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

```typescript theme={null}
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

Modifie (patch) une tâche par son ID. Définissez `status` à `"deleted"` pour la supprimer.

### TaskGet

**Nom de l'outil :** `TaskGet`

```typescript theme={null}
type TaskGetInput = {
  taskId: string;
};
```

Renvoie tous les détails d'une tâche, ou `null` quand l'ID est introuvable.

### TaskList

**Nom de l'outil :** `TaskList`

```typescript theme={null}
type TaskListInput = {};
```

Renvoie un instantané de toutes les tâches de la liste courante.

### ExitPlanMode

**Nom de l'outil :** `ExitPlanMode`

```typescript theme={null}
type ExitPlanModeInput = {
  /** Déprécié : n'est plus utilisé. */
  allowedPrompts?: Array<{
    tool: "Bash";
    prompt: string;
  }>;
  [k: string]: unknown;
};
```

Quitte le mode plan. Le champ `allowedPrompts` est déprécié et ignoré ; Claude Code l'accepte encore pour que les appelants et transcriptions existants restent valides. Avant la v2.1.205, il demandait des permissions Bash basées sur des prompts pour implémenter le plan.

### ListMcpResources

**Nom de l'outil :** `ListMcpResourcesTool`

```typescript theme={null}
type ListMcpResourcesInput = {
  server?: string;
};
```

Liste les ressources MCP disponibles sur les serveurs connectés.

### ReadMcpResource

**Nom de l'outil :** `ReadMcpResourceTool`

```typescript theme={null}
type ReadMcpResourceInput = {
  server: string;
  uri: string;
};
```

Lit une ressource MCP précise sur un serveur.

### EnterWorktree

**Nom de l'outil :** `EnterWorktree`

```typescript theme={null}
type EnterWorktreeInput = {
  name?: string;
  path?: string;
};
```

Crée un worktree git temporaire et y entre pour travailler de manière isolée. Passez `path` pour basculer dans un worktree existant au lieu d'en créer un nouveau. Lors de la première entrée, la cible doit être un worktree enregistré du dépôt courant ou, dans un espace de travail multi-dépôts, d'un dépôt imbriqué dedans ; depuis une session de worktree, elle doit se trouver sous `.claude/worktrees/` du dépôt de la session. `name` et `path` s'excluent mutuellement.

### ExitWorktree

**Nom de l'outil :** `ExitWorktree`

```typescript theme={null}
type ExitWorktreeInput = {
  action: "keep" | "remove";
  discard_changes?: boolean;
};
```

Quitte le worktree git courant et revient au répertoire de travail d'origine. L'action `keep` laisse le worktree et la branche sur disque, tandis que `remove` supprime les deux. `discard_changes` doit valoir `true` pour supprimer un worktree qui contient des fichiers non commités ou des commits non fusionnés.

### EnterPlanMode

**Nom de l'outil :** `EnterPlanMode`

```typescript theme={null}
type EnterPlanModeInput = {};
```

Entre en mode plan : Claude fait ses recherches et présente un plan avant d'effectuer des modifications.

### CronCreate

**Nom de l'outil :** `CronCreate`

```typescript theme={null}
type CronCreateInput = {
  cron: string;
  prompt: string;
  recurring?: boolean;
  durable?: boolean;
};
```

Planifie l'exécution d'un prompt selon une expression cron à 5 champs, en heure locale. Définissez `recurring` à `false` pour un déclenchement unique à la prochaine correspondance. Les jobs sont limités à la session par défaut, et une reprise avec `--resume` ou `--continue` restaure les jobs qui n'ont pas expiré. Voir [Tâches planifiées](https://code.claude.com/docs/en/scheduled-tasks).

Définir `durable` à `true` demande la persistance dans `.claude/scheduled_tasks.json` pour que le job survive aux redémarrages. La planification durable n'est pas disponible dans toutes les sessions : dans ce cas, Claude Code accepte `durable: true` mais crée le job pour la session uniquement. Lisez le champ `durable` de la sortie pour savoir si le job a été persisté.

### CronDelete

**Nom de l'outil :** `CronDelete`

```typescript theme={null}
type CronDeleteInput = {
  id: string;
};
```

Supprime un job cron planifié par l'ID renvoyé par `CronCreate`.

### CronList

**Nom de l'outil :** `CronList`

```typescript theme={null}
type CronListInput = {};
```

Liste les jobs cron planifiés : jobs durables de `.claude/scheduled_tasks.json` et jobs propres à la session courante.

### ScheduleWakeup

**Nom de l'outil :** `ScheduleWakeup`

```typescript theme={null}
type ScheduleWakeupInput = {
  delaySeconds?: number;
  reason?: string;
  prompt?: string;
  noop?: boolean;
  stop?: boolean;
};
```

Planifie un réveil unique qui déclenche le prompt donné après un délai. Cet outil sous-tend la commande `/loop` auto-cadencée (self-paced). Le runtime borne `delaySeconds` entre 60 et 3600 secondes. Les champs `delaySeconds`, `reason`, `prompt` et `noop` sont requis sauf si `stop` vaut true. `noop: true` signale un réveil où rien n'a changé. Définir `stop: true` annule le réveil en attente et termine la `/loop` auto-cadencée. Le champ `stop` nécessite Claude Code v2.1.202+. Voir la [ligne ScheduleWakeup de la référence des outils](https://code.claude.com/docs/en/tools-reference).

### RemoteTrigger

**Nom de l'outil :** `RemoteTrigger`

```typescript theme={null}
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

Gère les [Routines](https://code.claude.com/docs/en/routines), les exécutions Claude Code planifiées et déclenchées hébergées dans le cloud. Cet outil sous-tend la commande `/schedule`. `trigger_id` est requis pour les actions `get`, `update`, `run` et `list_runs`. `body` est requis pour `create`, `update` et `create_webhook_trigger`, et optionnel pour `run`.

`create_webhook_trigger` attache une source d'événements à une routine existante, comme un [événement GitHub](https://code.claude.com/docs/en/routines#add-a-github-trigger) qui la déclenche. Le `body` nomme la source, les événements et la routine à déclencher. Nécessite Claude Code v2.1.225+.

`list_runs` liste les exécutions récentes d'une routine, et `get_run_log` lit le journal d'une exécution. `session_id` désigne l'exécution à lire, issue d'un résultat `list_runs`, et `cursor` pagine les résultats de l'une ou l'autre action. Ces deux actions nécessitent Claude Code v2.1.227+.

Cet outil n'est disponible que lorsque la session est authentifiée avec un compte claude.ai sur un forfait où les Routines sont activées, et il est absent quand la politique de votre organisation désactive les [sessions cloud](https://code.claude.com/docs/en/claude-code-on-the-web). À partir de Claude Code v2.1.227, l'outil est aussi absent quand un Owner a [désactivé les routines pour l'organisation](https://code.claude.com/docs/en/routines#routines-are-disabled-by-your-organizations-policy). Avant la v2.1.227, une session où seul le commutateur des routines était désactivé affichait encore l'outil, et le serveur refusait ses appels.

### PushNotification

**Nom de l'outil :** `PushNotification`

```typescript theme={null}
type PushNotificationInput = {
  message: string;
  status: "proactive";
};
```

Envoie une notification push proactive à l'utilisateur. Gardez `message` sous 200 caractères, car les systèmes d'exploitation mobiles tronquent les textes plus longs. Voir la [ligne PushNotification de la référence des outils](https://code.claude.com/docs/en/tools-reference) pour la disponibilité selon le fournisseur ; la livraison des notifications push passe par une infrastructure hébergée par Anthropic, inaccessible depuis Amazon Bedrock, Claude Platform on AWS, Google Cloud's Agent Platform ou Microsoft Foundry.

### REPL

Supprimé dans la v2.1.275. Jusqu'à la v2.1.274, un outil expérimental `REPL` pouvait être activé avec `CLAUDE_CODE_REPL=1` dans l'[option `env`](https://code.claude.com/docs/en/agent-sdk/typescript#options).

### ReportFindings

**Nom de l'outil :** `ReportFindings`

```typescript theme={null}
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

Signale les constats d'une revue de code sous forme de liste structurée, pour que Claude Code puisse les afficher au lieu de les imprimer en texte. Les constats sont classés du plus grave au moins grave, avec au plus 32 par appel, et le tableau est vide quand aucun n'a survécu. Nécessite Claude Code v2.1.196+.

`level` est optionnel et contient le niveau d'effort que Claude déclare pour la revue. Claude Code ne le compare pas au niveau auquel la revue a réellement tourné ; les deux peuvent donc différer.

Chaque constat comporte ces champs :

* `file` : chemin relatif au dépôt du fichier concerné. Le `line` optionnel est la ligne (indexée à partir de 1) à laquelle il se rattache.
* `summary` : énoncé du défaut en une phrase. `failure_scenario` décrit les entrées et l'état concrets qui mènent à la sortie erronée ou au plantage.
* `short_summary` : libellé compressé optionnel d'au plus 60 caractères pour un affichage compact. Nécessite Claude Code v2.1.212+.
* `category` : slug court optionnel en kebab-case du type de constat, comme `correctness` ou `test-coverage`. Nécessite Claude Code v2.1.199+.
* `verdict` : défini quand une passe de vérification a eu lieu ; absent pour les revues uniquement en ligne (inline).
* `outcome` : défini uniquement lors d'un nouveau signalement après application des correctifs.

### Artifact

**Nom de l'outil :** `Artifact`

```typescript theme={null}
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

Publie un fichier local `.html` ou `.md` sous forme de page d'artifact hébergée, ou liste les artifacts publiés par l'utilisateur. Omettez `action` ou passez `"publish"` pour publier `file_path`, qui est obligatoire pour l'action de publication. Chacun des champs ci-dessous s'applique à une publication :

* `icon` : un mot générique court pour l'icône d'onglet du navigateur de l'artifact, comme `chart` ou `map`. Claude l'inclut lors d'une première publication et l'omet lors d'une mise à jour, ce qui conserve l'icône enregistrée de l'artifact.
* `favicon` : déprécié ; Claude l'omet.
* `title` : nomme la page publiée dans l'onglet du navigateur et dans la galerie quand le fichier HTML n'a pas de balise `<title>`.
* `url` : cible un artifact existant à mettre à jour sur place au lieu d'en créer un nouveau.

`force` est un écrasement de dernier recours qui supprime une version plus récente publiée par une autre session. En cas de conflit, la publication échouée renvoie le contenu plus récent ; Claude fusionne ses modifications sur ce contenu, ou relit l'artifact, puis publie à nouveau. Ne passez `force` que lorsque l'utilisateur demande explicitement d'abandonner cette version.

Passez `"list"` pour énumérer les artifacts publiés par l'utilisateur ; seuls `limit` et `scope` peuvent l'accompagner. `scope` vaut `"mine"` par défaut, ce qui liste les artifacts dont l'utilisateur est propriétaire ; `"shared"` liste les artifacts que d'autres personnes ont partagés avec l'utilisateur, et `"all"` liste les deux.

`limit` fixe le nombre maximal d'artifacts qu'une liste renvoie, de 1 à 200. Un `limit` supérieur à 50 nécessite l'Agent SDK v0.3.292+. Sans `limit`, une liste renvoie jusqu'à 25 éléments.

* `capabilities` : les capacités runtime qu'utilise la page publiée, indexées par nom de capacité, comme les [connecteurs que la page peut appeler](https://code.claude.com/docs/en/artifacts#pull-live-data-with-mcp-connectors). Le service d'artifacts valide la déclaration et rejette une publication qui nomme une capacité que le compte ne peut pas utiliser ou qui lui donne une configuration invalide. Passez `{}` pour effacer une déclaration enregistrée, et omettez le champ lors d'un redéploiement pour la conserver. Nécessite l'Agent SDK v0.3.235+.
* `contract` : la version runtime contre laquelle s'exécute la page publiée. Omettez-le pour conserver la version actuelle de l'artifact, passez `"latest"` pour mettre à niveau, ou passez une version précise pour figer (pin) ou revenir en arrière (rollback). Nécessite l'Agent SDK v0.3.235+.

Les types sont exportés, mais l'outil est désactivé par défaut dans les sessions Agent SDK. La publication exige aussi toutes les conditions du [tableau de disponibilité des artifacts](https://code.claude.com/docs/en/artifacts#availability), que les sessions authentifiées par clé API ne remplissent pas.

### Projects

**Nom de l'outil :** `Projects`

```typescript theme={null}
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

Lit et écrit le Project claude.ai attaché à la session. Le comportement dépend de `method` :

* `project_info` : renvoie les métadonnées du projet et la liste des documents.
* `project_read` : lit un document par son `path`.
* `project_search` : interroge la base de connaissances du projet avec `query`. `n` plafonne le nombre de résultats ; valeur par défaut `5`.
* `project_write` : crée ou remplace un document à `path` à partir d'exactement l'un de `content`, qui porte du texte en ligne, ou `local_path`, qui désigne un fichier du répertoire de travail. `present_to_user: true` marque le document écrit comme le livrable que l'utilisateur doit voir.
* `project_delete` : supprime un document par son `path`.

### ReadMcpResourceDir

**Nom de l'outil :** `ReadMcpResourceDirTool`

```typescript theme={null}
type ReadMcpResourceDirInput = {
  server: string;
  uri: string;
};
```

Liste les enfants directs d'une ressource de type répertoire sur un serveur MCP. Utilisable uniquement avec un serveur qui a déclaré prendre en charge le listage de répertoires ; le listage n'est pas récursif. Le listage de répertoires n'est pas activé dans toutes les sessions : quand il est désactivé, l'appel renvoie une liste `resources` vide et le champ `error` indique que le listage de répertoires n'est pas activé.

### RefreshMcpTools

**Nom de l'outil :** `RefreshMcpTools`

```typescript theme={null}
type RefreshMcpToolsInput = {
  server?: string; // ne rafraîchit que ce serveur ; omettez-le pour rafraîchir tous les serveurs connectés
};
```

Réinterroge la liste d'outils des serveurs MCP connectés et applique les éventuels changements. Les types sont exportés, mais Claude Code n'enregistre l'outil que si vous définissez `CLAUDE_CODE_ENABLE_REFRESH_MCP_TOOLS=1` dans l'[option `env`](https://code.claude.com/docs/en/agent-sdk/typescript#options), et seulement dans les sessions ayant au moins un serveur MCP. Nécessite Claude Code v2.1.211+.

### ShowOnboardingRolePicker

**Nom de l'outil :** `ShowOnboardingRolePicker`

```typescript theme={null}
type ShowOnboardingRolePickerInput = {};
```

Affiche une rangée de puces cliquables de sélection de rôle pendant l'onboarding de Cowork, afin que l'utilisateur choisisse son rôle et obtienne l'installation d'un plugin correspondant. Ne prend aucun argument ; la liste des rôles est définie par le client. L'appel bloque jusqu'à la réponse de l'utilisateur.

### McpInput

**Nom de l'outil :** noms d'outils MCP dynamiques de la forme `mcp__<server>__<tool>`

```typescript theme={null}
type McpInput = {
  [k: string]: unknown;
};
```

Les arguments des outils MCP forment un objet ouvert : chaque serveur définit ses propres paramètres, donc le type n'impose aucune contrainte sur les noms ou valeurs des champs. Consultez le schéma d'outil du serveur pour connaître les champs qu'accepte un outil donné.

## Types de sortie des outils (Tool Output Types)

Documentation des schémas de sortie des outils intégrés de Claude Code. Ces types sont exportés depuis `@anthropic-ai/claude-agent-sdk/sdk-tools` et représentent les données de réponse effectivement renvoyées par chaque outil.

### `ToolOutputSchemas`

Union des types de sortie d'outils exportés depuis `@anthropic-ai/claude-agent-sdk/sdk-tools` ; ses membres incluent :

```typescript theme={null}
type ToolOutputSchemas =
  | AgentOutput
  | ArtifactOutput
  | AskUserQuestionOutput
  | BashOutput
  | CronCreateOutput
  | CronDeleteOutput
  | CronListOutput
  | EnterPlanModeOutput
  | EnterWorktreeOutput
  | ExitPlanModeOutput
  | ExitWorktreeOutput
  | FileEditOutput
  | FileReadOutput
  | FileWriteOutput
  | GlobOutput
  | GrepOutput
  | ListMcpResourcesOutput
  | McpOutput
  | MonitorOutput
  | NotebookEditOutput
  | ProjectsOutput
  | PushNotificationOutput
  | ReadMcpResourceDirOutput
  | ReadMcpResourceOutput
  | RefreshMcpToolsOutput
  | RemoteTriggerOutput
  | ReportFindingsOutput
  | ScheduleWakeupOutput
  | ShowOnboardingRolePickerOutput
  | TaskCreateOutput
  | TaskGetOutput
  | TaskListOutput
  | TaskStopOutput
  | TaskUpdateOutput
  | TodoWriteOutput
  | WebFetchOutput
  | WebSearchOutput
  | WorkflowOutput;
```

### Agent

**Nom de l'outil :** `Agent`. L'ancien nom `Task` reste accepté comme alias, et le tableau `tools` du message d'initialisation [`SDKSystemMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdksystemmessage) liste actuellement cet outil sous le nom `Task` pour la rétrocompatibilité.

```typescript theme={null}
type AgentOutput =
  | {
      status: "completed";
      agentId: string;
      agentType?: string;
      content: Array<{ type: "text"; text: string; citations?: unknown[] | null }>;
      resolvedModel?: string;
      modelsUsed?: string[];
      totalToolUseCount: number;
      totalDurationMs: number;
      totalTokens: number;
      usage: {
        input_tokens: number;
        output_tokens: number;
        cache_creation_input_tokens: number | null;
        cache_read_input_tokens: number | null;
        server_tool_use: {
          web_search_requests: number;
          web_fetch_requests: number;
        } | null;
        service_tier: string | null;
        cache_creation: {
          ephemeral_1h_input_tokens: number;
          ephemeral_5m_input_tokens: number;
        } | null;
        inference_geo?: string | null;
        speed?: string | null;
        iterations?: unknown;
        output_tokens_details?: {
          thinking_tokens?: number | null;
        } | null;
        fallback_credit?: unknown;
      };
      toolStats?: {
        readCount: number;
        searchCount: number;
        bashCount: number;
        editFileCount: number;
        linesAdded: number;
        linesRemoved: number;
        otherToolCount: number;
        frameCount?: number;
      };
      prompt: string;
      worktreePath?: string;
      worktreeBranch?: string;
    }
  | {
      status: "async_launched";
      isAsync?: true;
      agentId: string;
      description: string;
      resolvedModel?: string;
      modelsUsed?: string[];
      prompt: string;
      outputFile: string;
      canReadOutputFile?: boolean;
    }
  | {
      status: "remote_launched";
      taskId: string;
      sessionUrl: string;
      description: string;
      prompt: string;
      outputFile: string;
    };
```

Renvoie le résultat du sous-agent. Union discriminée par le champ `status` : `"completed"` pour les tâches terminées, `"async_launched"` pour les tâches d'arrière-plan, et `"remote_launched"` pour les tâches que Claude Code a confiées à une session cloud, où `sessionUrl` pointe vers cette session et `taskId` l'identifie.

Sur la variante `completed`, `resolvedModel` nomme le modèle sur lequel le sous-agent a démarré, qui peut différer de l'entrée `model` demandée quand [`availableModels`](https://code.claude.com/docs/en/model-config#restrict-model-selection) ou une autre surcharge s'applique. Ce champ nécessite Claude Code v2.1.174+. Sur `async_launched`, il nomme le modèle utilisé au moment où la tâche est passée en arrière-plan.

`modelsUsed` liste, dans l'ordre, les modèles utilisés par le sous-agent. Le champ n'est présent que si un changement de modèle a eu lieu en cours d'exécution, et un modèle réapparaît quand l'exécution est revenue à lui. Sur `async_launched`, la liste couvre les modèles utilisés avant le passage en arrière-plan. `modelsUsed` et le comportement de `resolvedModel` lors du passage en arrière-plan nécessitent tous deux Claude Code v2.1.212+.

Si Claude Code a [conservé le worktree isolé du sous-agent](https://code.claude.com/docs/en/worktrees#isolate-subagents-with-worktrees), `worktreePath` dans le résultat `completed` indique où le trouver. `worktreeBranch` est sa branche, présente quand Claude Code a créé le worktree avec git.

Claude Code remplit `usage` et `totalTokens` à partir de la dernière requête API du sous-agent, et non de l'exécution entière : `usage.service_tier` est donc la chaîne de niveau de service que l'API a signalée pour cette requête. Quand il est présent, `usage.output_tokens_details.thinking_tokens` est le nombre de tokens de sortie de cette requête qui étaient des tokens de réflexion (thinking). Le champ `output_tokens_details` nécessite le SDK TypeScript v0.3.228+, qui embarque Claude Code v2.1.228. Le champ `fallback_credit` nécessite le SDK TypeScript v0.3.285+, qui embarque Claude Code v2.1.285.

`usage.output_tokens_details` a le même sens que [`Usage.output_tokens_details`](https://code.claude.com/docs/en/agent-sdk/typescript#usage), limité à cette dernière requête, mais chacun de ses niveaux est optionnel ici. Protégez à la fois l'objet et le champ, par exemple `usage.output_tokens_details?.thinking_tokens ?? 0`, au lieu de le lire directement.

Avant la v2.1.207, le type publié était plus restreint. Il omettait `worktreePath`, `worktreeBranch`, `citations`, `toolStats.frameCount` et les champs d'usage `inference_geo`, `speed` et `iterations`, et typait `service_tier` comme `"standard" | "priority" | "batch"`. Les champs que le type marque comme optionnels peuvent être absents des résultats enregistrés par des versions antérieures.

### AskUserQuestion

**Nom de l'outil :** `AskUserQuestion`

```typescript theme={null}
type AskUserQuestionOutput = {
  questions: Array<{
    question: string;
    header: string;
    options: Array<{ label: string; description: string; preview?: string }>;
    multiSelect: boolean;
  }>;
  answers: Record<string, string>;
  response?: string;
  annotations?: Record<string, { preview?: string; notes?: string }>;
  afkTimeoutMs?: number;
};
```

Renvoie les questions posées et les réponses de l'utilisateur. `response` est défini quand l'utilisateur a tapé une réponse libre au lieu de répondre aux questions structurées ; dans ce cas, Claude reçoit « The user responded: … » au lieu de la liste des réponses question par question.

### Bash

**Nom de l'outil :** `Bash`

```typescript theme={null}
type BashOutput = {
  stdout: string;
  stderr: string;
  rawOutputPath?: string;
  interrupted: boolean;
  isImage?: boolean;
  backgroundTaskId?: string;
  backgroundedByUser?: boolean;
  timedOutAfterMs?: number;
  backgroundCwdHint?: string;
  backgroundEndsWithFinalResponse?: true;
  dangerouslyDisableSandbox?: boolean;
  returnCodeInterpretation?: string;
  noOutputExpected?: boolean;
  structuredContent?: unknown[];
  persistedOutputPath?: string;
  persistedOutputSize?: number;
  staleReadFileStateHint?: string;
  ghRateLimitHint?: string;
  gitOperation?: {
    commit?: { sha: string; kind: "committed" | "amended" | "cherry-picked"; branch?: string };
    push?: { branch: string };
    branch?: { ref: string; action: "merged" | "rebased" };
    pr?: {
      number: number;
      url?: string;
      action: "created" | "edited" | "merged" | "commented" | "closed" | "reopened" | "ready" | "draft" | "auto-merge-enabled" | "auto-merge-disabled";
    };
  };
};
```

Les champs `stdout`, `stderr` et `backgroundTaskId` contiennent :

| Champ | Contenu |
| - | - |
| `stdout` | La sortie standard et la sortie d'erreur de la commande, fusionnées en un seul flux entrelacé |
| `stderr` | Les avis ajoutés par l'outil lui-même, comme une réinitialisation du répertoire de travail du shell, et non le stderr de la commande |
| `backgroundTaskId` | Présent pour les commandes en arrière-plan |

`timedOutAfterMs` est le délai d'expiration en millisecondes, défini quand la commande a atteint son délai et est passée en arrière-plan au lieu d'y avoir été lancée explicitement. `backgroundCwdHint` est défini quand la commande passée en arrière-plan contenait une commande interne de changement de répertoire comme `cd`, `pushd`, `popd` ou `chdir`, et signale que le répertoire de travail de la session n'a pas changé. Ces deux champs nécessitent Claude Code v2.1.210+.

Quand un sous-agent qui s'exécute au premier plan possède une commande passée en arrière-plan, la commande [se termine à la fin de l'exécution de ce sous-agent](https://code.claude.com/docs/en/tools-reference#when-a-background-command-stops). Claude Code définit `backgroundEndsWithFinalResponse` à `true` pour ces commandes, et omet le champ quand la commande survit au tour, comme c'est le cas des commandes lancées par la conversation principale ou par des sous-agents d'arrière-plan. Ce champ nécessite Claude Code v2.1.227+.

Claude Code définit `gitOperation.commit.branch` sur la branche nommée dans la ligne de résumé du commit de git, et l'omet pour un commit fait sur une HEAD détachée. Ce champ nécessite l'Agent SDK v0.3.227+. Claude Code signale une commande `gh pr reopen` comme l'action de PR `reopened`, ce qui nécessite l'Agent SDK v0.3.234+.

### Monitor

**Nom de l'outil :** `Monitor`

```typescript theme={null}
type MonitorOutput = {
  taskId: string;
  timeoutMs: number;
  persistent?: boolean;
};
```

Renvoie l'ID de la tâche d'arrière-plan du moniteur en cours. Utilisez cet ID avec `TaskStop` pour annuler la surveillance plus tôt.

### Edit

**Nom de l'outil :** `Edit`

```typescript theme={null}
type FileEditOutput = {
  filePath: string;
  oldString: string;
  newString: string;
  originalFile: string | null;
  structuredPatch: Array<{
    oldStart: number;
    oldLines: number;
    newStart: number;
    newLines: number;
    lines: string[];
  }>;
  userModified: boolean;
  replaceAll: boolean;
  gitDiff?: {
    filename: string;
    status: "modified" | "added";
    additions: number;
    deletions: number;
    changes: number;
    patch: string;
    repository?: string | null;
  };
};
```

Renvoie le diff structuré de l'opération de modification.

### Read

**Nom de l'outil :** `Read`

```typescript theme={null}
type FileReadOutput =
  | {
      type: "text";
      file: {
        filePath: string;
        content: string;
        numLines: number;
        startLine: number;
        totalLines: number;
        /** Vrai quand une lecture du fichier entier a été paginée automatiquement parce qu'elle dépassait le plafond de tokens (le contenu est une première page partielle). */
        truncatedByTokenCap?: boolean;
      };
    }
  | {
      type: "image";
      file: {
        base64: string;
        type: "image/jpeg" | "image/png" | "image/gif" | "image/webp";
        originalSize: number;
        dimensions?: {
          originalWidth?: number;
          originalHeight?: number;
          displayWidth?: number;
          displayHeight?: number;
        };
      };
    }
  | {
      type: "notebook";
      file: {
        filePath: string;
        cells: unknown[];
      };
    }
  | {
      type: "pdf";
      file: {
        filePath: string;
        base64: string;
        originalSize: number;
      };
    }
  | {
      type: "parts";
      file: {
        filePath: string;
        originalSize: number;
        count: number;
        outputDir: string;
      };
      /** Numéro de page du document de la première page extraite ; sert à étiqueter les images de pages dans le contenu du tool_result. */
      firstPage?: number;
      /** En processus uniquement : les octets des images de pages sont livrés sous forme de blocs image dans le contenu du tool_result et ne sont pas conservés dans le tool_use_result émis, donc cette clé y est absente. */
      pages?: {
        base64: string;
        mediaType: "image/jpeg" | "image/png" | "image/gif" | "image/webp";
        error?: string;
      }[];
    }
  | {
      type: "file_unchanged";
      file: {
        filePath: string;
      };
      /** Défini quand la déduplication a correspondu à une entrée injectée au démarrage (CLAUDE.md / mémoire imbriquée) plutôt qu'à un tool_result de Read antérieur. */
      source?: "seeded";
    };
```

Renvoie le contenu du fichier dans un format adapté à son type. Union discriminée par le champ `type`.

### Write

**Nom de l'outil :** `Write`

```typescript theme={null}
type FileWriteOutput = {
  type: "create" | "update";
  filePath: string;
  content: string;
  structuredPatch: Array<{
    oldStart: number;
    oldLines: number;
    newStart: number;
    newLines: number;
    lines: string[];
  }>;
  originalFile: string | null;
  gitDiff?: {
    filename: string;
    status: "modified" | "added";
    additions: number;
    deletions: number;
    changes: number;
    patch: string;
    repository?: string | null;
  };
  userModified?: boolean;
};
```

Renvoie le résultat de l'écriture avec des informations de diff structuré. Le contenu de `originalFile` et `structuredPatch` dépend de l'écriture :

* Pour un fichier nouvellement créé, `originalFile` vaut null et `structuredPatch` est vide
* Lors d'un écrasement, `originalFile` contient le contenu précédent, sauf si ce contenu dépasse environ 10 Mo : Claude Code saute alors le diff et renvoie `originalFile` à null et `structuredPatch` vide
* `structuredPatch` est aussi vide quand l'écriture n'a rien changé ou que le calcul du diff a expiré

### Glob

**Nom de l'outil :** `Glob`

```typescript theme={null}
type GlobOutput = {
  durationMs: number;
  numFiles: number;
  filenames: string[];
  truncated: boolean;
  totalMatches?: number;
  countIsComplete?: boolean;
};
```

Renvoie les chemins de fichiers correspondant au motif glob, triés par date de modification.

`totalMatches` et `countIsComplete` nécessitent Claude Code v2.1.191+. `totalMatches` indique le nombre de fichiers correspondants avant troncature. Quand `countIsComplete` vaut false, `totalMatches` est une borne inférieure, car la recherche sous-jacente a tronqué sa propre sortie.

### Grep

**Nom de l'outil :** `Grep`

```typescript theme={null}
type GrepOutput = {
  mode?: "content" | "files_with_matches" | "count";
  numFiles: number;
  filenames: string[];
  content?: string;
  numLines?: number;
  numMatches?: number;
  totalFiles?: number;
  totalLines?: number;
  appliedLimit?: number;
  appliedOffset?: number;
};
```

Renvoie les résultats de recherche. La forme varie selon `mode` : liste de fichiers, contenu avec correspondances, ou nombres de correspondances. En mode `count`, `numFiles` et `numMatches` sont des totaux sur l'ensemble complet des résultats, pas sur la tranche paginée. Avant la v2.1.208, un `head_limit` ou un `offset` qui tronquait les entrées listées tronquait aussi ces totaux.

`totalFiles` nécessite Claude Code v2.1.208+ et indique, en mode `files_with_matches`, le nombre total de résultats avant la pagination par `head_limit` et `offset`. `totalLines` nécessite Claude Code v2.1.210+ et indique, en mode `content`, le nombre total de lignes avant pagination.

### TaskStop

**Nom de l'outil :** `TaskStop`

```typescript theme={null}
type TaskStopOutput = {
  message: string;
  task_id: string;
  task_type: string;
  command?: string;
};
```

Renvoie une confirmation après l'arrêt de la tâche d'arrière-plan.

### NotebookEdit

**Nom de l'outil :** `NotebookEdit`

```typescript theme={null}
type NotebookEditOutput = {
  new_source: string;
  old_source?: string;
  cell_id?: string;
  cell_type: "code" | "markdown";
  language: string;
  edit_mode: string;
  error?: string;
  notebook_path: string;
  original_file: string;
  updated_file: string;
};
```

Renvoie le résultat de la modification du notebook, avec le contenu du fichier d'origine et du fichier mis à jour.

### WebFetch

**Nom de l'outil :** `WebFetch`

```typescript theme={null}
type WebFetchOutput = {
  bytes: number;
  code: number;
  codeText: string;
  result: string;
  durationMs: number;
  url: string;
  artifactRead?: {
    slug: string;
    ver?: string;
    seeded?: false;
  };
};
```

Renvoie le contenu récupéré avec le statut HTTP et des métadonnées.

`artifactRead` est l'enregistrement propre à Claude Code d'une lecture d'artifact, présent uniquement quand Claude a récupéré un artifact sur lequel la session peut publier. Claude Code le relit à la reprise d'une session afin qu'une publication ultérieure s'appuie sur la bonne version ; votre code n'a pas à agir dessus. `slug` nomme l'artifact, `ver` est la version que la lecture a enregistrée (absent si aucune n'a été enregistrée), et `seeded: false` marque une lecture dont la source complète n'a pas atteint Claude. Le champ `seeded` nécessite l'Agent SDK v0.3.239+.

### WebSearch

**Nom de l'outil :** `WebSearch`

```typescript theme={null}
type WebSearchOutput = {
  query: string;
  results: Array<
    | {
        tool_use_id: string;
        content: Array<{ title: string; url: string }>;
      }
    | string
  >;
  durationSeconds: number;
  searchCount?: number;
};
```

Renvoie les résultats de recherche web.

### Workflow

**Nom de l'outil :** `Workflow`

```typescript theme={null}
type WorkflowOutput = {
  status: "async_launched" | "remote_launched";
  taskId: string;
  taskType?: "local_workflow" | "remote_agent";
  workflowName?: string;
  runId?: string;
  summary?: string;
  transcriptDir?: string;
  scriptPath?: string;
  sessionUrl?: string; // défini quand le workflow a été lancé comme session cloud
  warning?: string;
  error?: string;
};
```

Renvoie immédiatement après que l'outil a accepté l'invocation. Le résultat final arrive plus tard sous forme d'achèvement de tâche. Vérifiez `error` avant de considérer l'exécution comme démarrée : un script qui échoue à la vérification syntaxique renvoie `status: "async_launched"` avec `error` défini, et ne s'exécute jamais.

| Champ | Type | Description |
| - | - | - |
| `status` | `"async_launched" \| "remote_launched"` | L'outil a accepté l'invocation. `"async_launched"` pour les exécutions en processus, `"remote_launched"` pour les exécutions confiées à une session cloud au lieu de s'exécuter en processus |
| `taskId` | `string` | Identifiant de la tâche d'arrière-plan de l'exécution |
| `taskType` | `"local_workflow" \| "remote_agent"` | Type de la tâche d'arrière-plan enregistrée, correspondant à la branche de `status` |
| `workflowName` | `string` | Le `meta.name` du script de workflow |
| `runId` | `string` | Identifiant d'exécution du workflow à passer comme `resumeFromRunId` lors d'une invocation ultérieure. Absent pour les exécutions `remote_launched`, où l'URL de la session cloud sert de poignée de reprise |
| `summary` | `string` | Description en une ligne de ce que fait le workflow |
| `transcriptDir` | `string` | Répertoire où les transcriptions des sous-agents sont écrites pendant l'exécution |
| `scriptPath` | `string` | Chemin du script de workflow persisté pour cette exécution. Modifiez-le et repassez-le comme `scriptPath` pour relancer sans renvoyer le script |
| `sessionUrl` | `string` | URL de la session cloud, définie quand `status` vaut `"remote_launched"` |
| `warning` | `string` | Avertissement non bloquant, par exemple un état git local qui diverge de la branche poussée qu'une session cloud va cloner |
| `error` | `string` | Défini quand le script échoue à la vérification syntaxique. Quand il est présent, l'exécution n'a pas démarré malgré le statut « lancé » |

### TodoWrite

**Nom de l'outil :** `TodoWrite`

```typescript theme={null}
type TodoWriteOutput = {
  oldTodos: Array<{
    content: string;
    status: "pending" | "in_progress" | "completed";
    activeForm: string;
  }>;
  newTodos: Array<{
    content: string;
    status: "pending" | "in_progress" | "completed";
    activeForm: string;
  }>;
};
```

Renvoie la liste de tâches précédente et la liste mise à jour.

> **Note** : les outils suivants ne sont disponibles par défaut que sur Claude Opus 4.7, Claude Sonnet 4.6, Claude Haiku 4.5 et les modèles antérieurs. Sur tout autre modèle, y compris les ID de modèle que Claude Code ne reconnaît pas, ils ne sont pas disponibles sauf activation explicite (opt-in) :
>
> * `TodoWrite`
> * `TaskCreate`
> * `TaskGet`
> * `TaskUpdate`
> * `TaskList`
>
> Là où ces outils sont disponibles, Claude Code fournit les quatre outils Task, ou `TodoWrite` à leur place quand vous définissez `CLAUDE_CODE_ENABLE_TASKS=0`.
>
> Cet ensemble par défaut s'applique à Claude Code v2.1.268+, que l'Agent SDK TypeScript embarque à partir de la v0.3.268.
>
> Voir [Disponibilité selon le modèle](https://code.claude.com/docs/en/agent-sdk/todo-tracking#model-availability) pour activer ces outils.

### TaskCreate

**Nom de l'outil :** `TaskCreate`

```typescript theme={null}
type TaskCreateOutput = {
  task: {
    id: string;
    subject: string;
  };
};
```

Renvoie la tâche créée avec l'ID qui lui a été attribué.

### TaskUpdate

**Nom de l'outil :** `TaskUpdate`

```typescript theme={null}
type TaskUpdateOutput = {
  success: boolean;
  taskId: string;
  updatedFields: string[];
  error?: string;
  statusChange?: {
    from: string;
    to: string;
  };
};
```

Renvoie le résultat de la mise à jour, y compris les champs modifiés.

### TaskGet

**Nom de l'outil :** `TaskGet`

```typescript theme={null}
type TaskGetOutput = {
  task: {
    id: string;
    subject: string;
    description: string;
    status: "pending" | "in_progress" | "completed";
    blocks: string[];
    blockedBy: string[];
  } | null;
};
```

Renvoie l'enregistrement complet de la tâche, ou `null` quand l'ID est introuvable.

### TaskList

**Nom de l'outil :** `TaskList`

```typescript theme={null}
type TaskListOutput = {
  tasks: Array<{
    id: string;
    subject: string;
    status: "pending" | "in_progress" | "completed";
    owner?: string;
    blockedBy: string[];
  }>;
};
```

Renvoie un instantané de toutes les tâches de la liste courante.

### ExitPlanMode

**Nom de l'outil :** `ExitPlanMode`

```typescript theme={null}
type ExitPlanModeOutput = {
  plan: string | null;
  isAgent: boolean;
  filePath?: string;
  hasTaskTool?: boolean;
  planWasEdited?: boolean;
  awaitingLeaderApproval?: boolean;
  requestId?: string;
};
```

Renvoie l'état du plan après la sortie du mode plan.

### ListMcpResources

**Nom de l'outil :** `ListMcpResourcesTool`

```typescript theme={null}
type ListMcpResourcesOutput = Array<{
  uri: string;
  name: string;
  mimeType?: string;
  description?: string;
  server: string;
}>;
```

Renvoie un tableau des ressources MCP disponibles.

### ReadMcpResource

**Nom de l'outil :** `ReadMcpResourceTool`

```typescript theme={null}
type ReadMcpResourceOutput = {
  contents: Array<{
    uri: string;
    mimeType?: string;
    text?: string;
    blobSavedTo?: string;
  }>;
  error?: string;
};
```

Renvoie le contenu de la ressource MCP demandée.

### EnterWorktree

**Nom de l'outil :** `EnterWorktree`

```typescript theme={null}
type EnterWorktreeOutput = {
  worktreePath: string;
  worktreeBranch?: string;
  message: string;
};
```

Renvoie des informations sur le worktree git.

### ExitWorktree

**Nom de l'outil :** `ExitWorktree`

```typescript theme={null}
type ExitWorktreeOutput = {
  action: "keep" | "remove";
  originalCwd: string;
  worktreePath: string;
  worktreeBranch?: string;
  tmuxSessionName?: string;
  discardedFiles?: number;
  discardedCommits?: number;
  message: string;
};
```

Renvoie l'action effectuée et des détails sur le worktree quitté.

### EnterPlanMode

**Nom de l'outil :** `EnterPlanMode`

```typescript theme={null}
type EnterPlanModeOutput = {
  message: string;
};
```

Renvoie une confirmation de l'entrée en mode plan.

### CronCreate

**Nom de l'outil :** `CronCreate`

```typescript theme={null}
type CronCreateOutput = {
  id: string;
  humanSchedule: string;
  recurring: boolean;
  durable?: boolean; // true quand persisté dans .claude/scheduled_tasks.json ; false quand limité à la session
};
```

Renvoie l'ID du job et une description lisible de la planification.

### CronDelete

**Nom de l'outil :** `CronDelete`

```typescript theme={null}
type CronDeleteOutput = {
  id: string;
};
```

Renvoie l'ID du job supprimé.

### CronList

**Nom de l'outil :** `CronList`

```typescript theme={null}
type CronListOutput = {
  jobs: {
    id: string;
    cron: string;
    humanSchedule: string;
    prompt: string;
    recurring?: boolean;
    durable?: boolean;
  }[];
};
```

Renvoie les jobs cron planifiés : jobs durables de `.claude/scheduled_tasks.json` et jobs propres à la session courante. Un job limité à la session porte `durable: false` ; les jobs lus depuis le disque omettent ce champ.

### ScheduleWakeup

**Nom de l'outil :** `ScheduleWakeup`

```typescript theme={null}
type ScheduleWakeupOutput = {
  scheduledFor: number;
  clampedDelaySeconds: number;
  wasClamped: boolean;
  stopped?: boolean;
  cancelledWakeups?: number;
};
```

Renvoie le moment où le réveil se déclenchera (horodatage epoch en millisecondes), le délai réellement utilisé, et si le délai demandé a été borné. Le champ `stopped` vaut `true` quand l'appel a terminé la boucle avec `stop: true` ; il nécessite Claude Code v2.1.202+. Le champ `cancelledWakeups` compte le nombre de réveils en attente annulés par un appel `stop: true`. Une valeur de 0 signifie qu'aucun n'était en attente, et un cron `/loop` récurrent n'est pas annulé par `stop: true`. Il nécessite Claude Code v2.1.206+.

### RemoteTrigger

**Nom de l'outil :** `RemoteTrigger`

```typescript theme={null}
type RemoteTriggerOutput = {
  status: number;
  json: string;
  summary?: string;
};
```

Renvoie le statut et le corps de la réponse de l'API pour l'opération sur le déclencheur.

### PushNotification

**Nom de l'outil :** `PushNotification`

```typescript theme={null}
type PushNotificationOutput = {
  message: string;
  pushSent?: boolean;
  localSent?: boolean;
  disabledReason?: "config_off" | "user_present" | "no_transport";
  sentAt?: string;
};
```

Renvoie les détails de livraison, notamment si une notification push ou locale a été envoyée et pourquoi la livraison a été ignorée.

### ReportFindings

**Nom de l'outil :** `ReportFindings`

```typescript theme={null}
type ReportFindingsOutput = {
  count: number;
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

Renvoie le nombre de constats signalés, la valeur `level` passée par Claude, et les constats renvoyés en écho pour le corps du résultat. Nécessite Claude Code v2.1.196+. Le champ `short_summary` renvoyé en écho nécessite Claude Code v2.1.212+.

### Artifact

**Nom de l'outil :** `Artifact`

```typescript theme={null}
type ArtifactOutput =
  | {
      url: string;
      path: string;
      title?: string;
      version?: string;
      capabilities?: unknown;
      stored?: {
        contract: string;
        capabilities?: Record<string, unknown>;
      };
      warnings?: string[];
      contract?: string;
      updated?: boolean;
      liveSubscription?: string;
    }
  | {
      artifacts: Array<{
        title: string;
        url: string;
        updatedAt?: string;
        rel?: "mine" | "shared";
      }>;
      truncated?: boolean;
      total?: number;
      total_at_least?: true;
      scope?: "shared" | "all";
    };
```

Pour l'action de publication, renvoie l'`url` de la page publiée et le `path` local qui a été publié, avec `updated` à true quand la publication a redéployé un artifact existant, et `warnings` contenant les éventuels avertissements émis à la publication. L'action de liste renvoie à la place les lignes `artifacts`, avec `truncated` défini quand il existe plus d'artifacts que la limite demandée. Sur les listes dont le scope n'est pas `"mine"`, chaque ligne porte `rel`, qui indique si l'utilisateur possède l'artifact ou s'il a été partagé avec lui, et le `scope` de la sortie enregistre quel scope non par défaut a produit la liste ; les deux sont absents des listes par défaut.

Un résultat de liste indique aussi `total`, le nombre d'artifacts correspondant au scope listé, y compris ceux au-delà de `limit`. Quand `total_at_least` est défini, ce nombre est une borne inférieure et d'autres artifacts peuvent exister. Ces deux champs nécessitent l'Agent SDK v0.3.292+.

### Projects

**Nom de l'outil :** `Projects`

```typescript theme={null}
type ProjectsOutput =
  | {
      method: "project_info";
      notice?: string;
      name: string;
      description: string;
      instructions: string;
      docs: Array<{ path: string; created_at: string | null }>;
      files?: Array<{
        path: string;
        file_kind: string;
        created_at: string | null;
      }>;
      sync_sources?: Array<{
        type: string | null;
        config: Record<string, unknown>;
      }>;
      knowledge: {
        knowledge_size: number;
        max_knowledge_size: number;
      };
    }
  | {
      method: "project_read";
      notice?: string;
      path: string;
      file_kind?: string;
      content?: string;
      local_file?: string;
      created_at: string | null;
    }
  | {
      method: "project_search";
      notice?: string;
      rag: boolean;
      hits?: Array<{ name?: string; doc_uuid?: string; text?: string }>;
      docs?: string[];
    }
  | {
      method: "project_write";
      notice?: string;
      path: string;
      doc_uuid: string;
      replaced: boolean;
      present_to_user?: boolean;
      local_path?: string;
    }
  | {
      method: "project_delete";
      notice?: string;
      path: string;
      deleted: boolean;
    };
```

Union discriminée par le champ `method`, en miroir de l'entrée. `project_read` renvoie les petits documents texte en ligne dans `content` et écrit les documents plus volumineux dans un chemin `local_file` à la place ; `project_search` renvoie des `hits` RAG avec `rag: true` quand l'index du projet est disponible, et se rabat sinon sur une liste de chemins `docs`.

### ReadMcpResourceDir

**Nom de l'outil :** `ReadMcpResourceDirTool`

```typescript theme={null}
type ReadMcpResourceDirOutput = {
  resources: Array<{
    uri: string;
    name: string;
    mimeType?: string;
  }>;
  error?: string;
};
```

Renvoie les enfants directs de la ressource répertoire. Les sous-répertoires apparaissent avec le mimeType `"inode/directory"` ; `error` contient un message lisible quand le serveur n'a pas pu lister le répertoire.

### RefreshMcpTools

**Nom de l'outil :** `RefreshMcpTools`

```typescript theme={null}
type RefreshMcpToolsOutput = Array<{
  server: string;
  status: "refreshed" | "error" | "not_connected";
  toolCount?: number; // outils désormais disponibles depuis ce serveur
  added?: string[]; // noms des outils ajoutés par ce rafraîchissement
  removed?: string[]; // noms des outils retirés par ce rafraîchissement
  error?: string; // raison de l'échec du rafraîchissement ou de l'indisponibilité du serveur
}>;
```

Renvoie une entrée par serveur : `refreshed` signifie que la liste d'outils réinterrogée a été appliquée, `error` que la réinterrogation a échoué et que l'ensemble d'outils précédent a été conservé, et `not_connected` que le serveur n'a pas de connexion active à interroger.

### ShowOnboardingRolePicker

**Nom de l'outil :** `ShowOnboardingRolePicker`

```typescript theme={null}
type ShowOnboardingRolePickerOutput = {
  role?: string;
  dismissed?: boolean;
};
```

Renvoie la sélection de l'utilisateur : `role` quand il a choisi une puce de rôle ou en a tapé un, et `dismissed: true` quand il a fermé le sélecteur. Un objet vide signifie que l'utilisateur a approuvé l'appel sans choisir de rôle.

### McpOutput

**Nom de l'outil :** noms d'outils MCP dynamiques de la forme `mcp__<server>__<tool>`

```typescript theme={null}
type McpOutput =
  | string
  | {
      type: string;
      [k: string]: unknown;
    }[]
  | {
      [k: string]: unknown;
    };
```

Les résultats des outils MCP sont renvoyés sous forme de chaîne ou de tableau de blocs de contenu, selon le serveur. La dernière branche (objet simple) du type exporté est un artefact de la génération du schéma. Pour un résultat qui porte aussi `structuredContent` ou des liens de ressources, voir [`tool_use_result`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkusermessage), qui contient cette valeur dans son membre `content`. À l'exécution, la valeur peut aussi être `undefined`, bien que le type exporté ne le modélise pas.

## Types de permissions (Permission Types)

### `PermissionUpdate`

Opérations de mise à jour des permissions.

```typescript theme={null}
type PermissionUpdate =
  | {
      type: "addRules";
      rules: PermissionRuleValue[];
      behavior: PermissionBehavior;
      destination: PermissionUpdateDestination;
    }
  | {
      type: "replaceRules";
      rules: PermissionRuleValue[];
      behavior: PermissionBehavior;
      destination: PermissionUpdateDestination;
    }
  | {
      type: "removeRules";
      rules: PermissionRuleValue[];
      behavior: PermissionBehavior;
      destination: PermissionUpdateDestination;
    }
  | {
      type: "setMode";
      mode: PermissionMode;
      destination: PermissionUpdateDestination;
    }
  | {
      type: "addDirectories";
      directories: string[];
      destination: PermissionUpdateDestination;
    }
  | {
      type: "removeDirectories";
      directories: string[];
      destination: PermissionUpdateDestination;
    };
```

### `PermissionBehavior`

```typescript theme={null}
type PermissionBehavior = "allow" | "deny" | "ask";
```

### `PermissionUpdateDestination`

```typescript theme={null}
type PermissionUpdateDestination =
  | "userSettings" // Réglages utilisateur globaux
  | "projectSettings" // Réglages de projet par répertoire
  | "localSettings" // Réglages de projet locaux
  | "session" // Session courante uniquement
  | "cliArg"; // Argument CLI
```

### `PermissionRuleValue`

```typescript theme={null}
type PermissionRuleValue = {
  toolName: string;
  ruleContent?: string;
};
```

## Autres types (Other Types)

### `ApiKeySource`

Origine de la clé API utilisée pour les requêtes de la session, indiquée comme `apiKeySource` dans le message d'initialisation [`SDKSystemMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdksystemmessage).

```typescript theme={null}
type ApiKeySource =
  | "ANTHROPIC_API_KEY"
  | "apiKeyHelper"
  | "/login managed key"
  | "none"
  | "user"
  | "project"
  | "org"
  | "temporary"
  | "oauth";
```

Claude Code signale l'une de ces quatre valeurs :

| Valeur | Clé utilisée |
| - | - |
| `ANTHROPIC_API_KEY` | La clé de la variable d'environnement `ANTHROPIC_API_KEY` |
| `apiKeyHelper` | La clé renvoyée par votre commande [`apiKeyHelper`](https://code.claude.com/docs/en/settings-reference#apikeyhelper) |
| `/login managed key` | La clé que Claude Code a enregistrée lors de votre connexion avec un [compte Claude Console](https://code.claude.com/docs/en/authentication#claude-console-authentication) |
| `none` | Aucune clé API. La session s'authentifie autrement, par exemple via une connexion claude.ai, un bearer token ou un fournisseur cloud |

L'Agent SDK v0.3.234+ liste ces quatre valeurs dans le type. Le type conserve aussi `user`, `project`, `org`, `temporary` et `oauth` pour que l'ancien code compile toujours, mais Claude Code ne les signale pas.

### `SdkBeta`

Fonctionnalités bêta disponibles, activables via l'option `betas`. Voir [Beta headers](https://platform.claude.com/docs/en/api/beta-headers) pour plus d'informations.

```typescript theme={null}
type SdkBeta = "context-1m-2025-08-07";
```

> **Avertissement** : sur l'API Claude, la bêta `context-1m-2025-08-07` est retirée pour Claude Sonnet 4.5 et Claude Sonnet 4. Si vous la passez encore avec l'un de ces modèles, les requêtes qui dépassent la fenêtre de contexte standard de 200K tokens renvoient une erreur ; retirez-la donc de `betas`. Pour exécuter une session avec une fenêtre de contexte de 1M tokens, définissez `model` sur un modèle qui [fonctionne avec la fenêtre 1M par défaut](https://code.claude.com/docs/en/model-config#extended-context), comme `claude-sonnet-5-5` ou `claude-opus-5-5`. Pour un modèle qui n'atteint 1M que via sa variante `[1m]`, ajoutez ce suffixe à l'ID du modèle, comme dans `claude-opus-4-6[1m]`.

### `SlashCommand`

Informations sur une commande disponible.

```typescript theme={null}
type SlashCommand = {
  name: string;
  description: string;
  argumentHint: string;
  aliases?: string[];
  builtin?: boolean;
};
```

`builtin` vaut `true` sur une ligne quand la commande appartient à Claude Code lui-même et que taper `/name` l'exécute. Il est absent pour une commande définie par un utilisateur, un projet, un plugin ou un serveur MCP, ainsi que pour une commande intégrée que l'un d'eux [remplace par son nom](https://code.claude.com/docs/en/skills#resolve-skills-that-share-a-name). Nécessite l'Agent SDK v0.3.277+.

### `ModelInfo`

Informations sur un modèle disponible.

```typescript theme={null}
type ModelInfo = {
  value: string;
  resolvedModel?: string;
  displayName: string;
  description: string;
  supportsEffort?: boolean;
  supportedEffortLevels?: ("low" | "medium" | "high" | "xhigh" | "max")[];
  supportsAdaptiveThinking?: boolean;
  supportsFastMode?: boolean;
  supportsAutoMode?: boolean;
};
```

| Champ | Type | Description |
| :- | :- | :- |
| `value` | `string` | Identifiant du modèle à passer dans les appels API |
| `resolvedModel` | `string \| undefined` | L'ID de modèle vers lequel se résout la `value` de cette entrée, par exemple `claude-sonnet-5-5` pour l'entrée de l'alias `sonnet`. Nécessite Claude Code v2.1.197+. |
| `displayName` | `string` | Nom d'affichage lisible |
| `description` | `string` | Description des capacités du modèle |
| `supportsEffort` | `boolean \| undefined` | Indique si ce modèle prend en charge les niveaux d'effort |
| `supportedEffortLevels` | `("low" \| "medium" \| "high" \| "xhigh" \| "max")[] \| undefined` | Niveaux d'effort acceptés par ce modèle |
| `supportsAdaptiveThinking` | `boolean \| undefined` | Indique si ce modèle prend en charge la réflexion adaptative, où Claude décide quand et combien réfléchir |
| `supportsFastMode` | `boolean \| undefined` | Indique si ce modèle prend en charge le mode rapide (fast mode) |
| `supportsAutoMode` | `boolean \| undefined` | Indique si ce modèle prend en charge le mode auto |

### `AgentInfo`

Informations sur un sous-agent disponible, invocable via l'outil Agent.

```typescript theme={null}
type AgentInfo = {
  name: string;
  description: string;
  model?: string;
};
```

| Champ | Type | Description |
| :- | :- | :- |
| `name` | `string` | Identifiant du type d'agent (par exemple `"Explore"`, `"general-purpose"`) |
| `description` | `string` | Description de quand utiliser cet agent |
| `model` | `string \| undefined` | Modèle utilisé par cet agent : un alias ou un ID de modèle, ou `'inherit'` pour le modèle du parent. Quand il vaut `undefined`, Claude Code choisit le modèle selon l'[ordre de sélection du modèle des sous-agents](https://code.claude.com/docs/en/sub-agents#choose-a-model) |

### `McpServerProvenance`

Le serveur MCP qui fournit un outil `mcp__*`, et l'origine de la définition de ce serveur. Les entrées des hooks [`PreToolUse`](https://code.claude.com/docs/en/agent-sdk/typescript#pretoolusehookinput), `PostToolUse`, `PostToolUseFailure`, `PermissionRequest` et `PermissionDenied` le portent sous le nom `mcp_server`, et les options de [`CanUseTool`](https://code.claude.com/docs/en/agent-sdk/typescript#canusetool) sous le nom `mcpServer`. Les deux l'omettent pour les outils qui ne proviennent pas d'un serveur MCP.

```typescript theme={null}
type McpServerProvenance = {
  name: string;
  source: string;
};
```

| Champ | Type | Description |
| :- | :- | :- |
| `name` | `string` | Le nom sous lequel le serveur est enregistré, la même valeur que celle que [`mcpServerStatus()`](https://code.claude.com/docs/en/agent-sdk/typescript#query-object) indique pour lui |
| `source` | `string` | Origine de la définition du serveur : `sdk`, `plugin` ou une portée de configuration |

`source` prend l'une des valeurs suivantes. L'ensemble est ouvert : traitez une valeur que vous ne reconnaissez pas comme une source configurée, jamais comme `sdk` :

* **`sdk`** : un serveur en processus que votre application a enregistré. Seule l'application hôte du SDK peut en enregistrer un ; un serveur configuré ne signale donc jamais `sdk`, quel que soit son nom.
* **`plugin`** : un serveur fourni par un [plugin](https://code.claude.com/docs/en/agent-sdk/plugins). Son `name` a la forme à portée `plugin:<plugin-name>:<server-name>` décrite dans [serveurs MCP fournis par des plugins](https://code.claude.com/docs/en/mcp#plugin-provided-mcp-servers).
* **Une portée de configuration** : `user`, `project`, `local`, `dynamic`, `managed`, `enterprise`, `claudeai` ou `agent`. Un serveur de `.mcp.json` signale `project`, et les [portées d'installation MCP](https://code.claude.com/docs/en/mcp#mcp-installation-scopes) définissent `local`, `project` et `user`. Les serveurs que votre application passe dans l'[option `mcpServers`](https://code.claude.com/docs/en/agent-sdk/typescript#options), hormis les serveurs SDK en processus, signalent `dynamic`.

Fondez vos décisions de confiance sur `source`, pas sur `name` ni sur le préfixe de nom d'outil `mcp__<server>__`. Pour toute source autre que `sdk`, `name` est un texte non fiable : échappez-le avant de l'afficher.

`McpServerProvenance` et les champs qui le portent nécessitent l'Agent SDK v0.3.274+.

### `McpServerStatus`

Statut d'un serveur MCP connecté.

```typescript theme={null}
type McpServerStatus = {
  name: string;
  status: "connected" | "failed" | "needs-auth" | "pending" | "disabled";
  serverInfo?: {
    name: string;
    version: string;
  };
  error?: string;
  config?: McpServerStatusConfig;
  scope?: string;
  source?: string;
  tools?: {
    name: string;
    description?: string;
    annotations?: {
      readOnly?: boolean;
      destructive?: boolean;
      openWorld?: boolean;
    };
    _meta?: Record<string, unknown>;
  }[];
};
```

`source` indique l'origine de la définition du serveur, avec les mêmes valeurs et la même règle de confiance que le `source` de [`McpServerProvenance`](https://code.claude.com/docs/en/agent-sdk/typescript#mcpserverprovenance). Ce champ nécessite l'Agent SDK v0.3.274+ et est absent des versions antérieures.

`_meta` sur une entrée de `tools` porte les membres MCP Apps du `_meta` de cet outil, afin que votre application puisse trouver la ressource `ui://` à afficher avec [`readMcpResource()`](https://code.claude.com/docs/en/agent-sdk/typescript#query-object). Claude Code transmet l'objet `ui` et la chaîne plate dépréciée `ui/resourceUri`, et retient toutes les autres clés. Dans `ui`, `resourceUri` est une chaîne `ui://` et `visibility` un tableau de `"model"` et `"app"` quand le serveur les définit ; tout autre membre est transmis tel quel. Claude Code supprime l'une ou l'autre clé quand sa valeur est mal formée, et omet `_meta` pour un outil qui ne déclare ni l'une ni l'autre. Le champ n'est présent que lorsque les [`capabilities`](https://code.claude.com/docs/en/agent-sdk/typescript#sdksystemmessage) du message d'initialisation incluent `mcp_tool_ui_meta_v1`, et nécessite l'Agent SDK TypeScript v0.3.280+.

### `McpServerStatusConfig`

La configuration d'un serveur MCP telle que la renvoie `mcpServerStatus()`. C'est l'union de tous les types de transport de serveur MCP.

```typescript theme={null}
type McpServerStatusConfig =
  | McpStdioServerConfig
  | McpSSEServerConfig
  | McpHttpServerConfig
  | McpSdkServerConfig
  | McpClaudeAIProxyServerConfig;
```

Voir [`McpServerConfig`](https://code.claude.com/docs/en/agent-sdk/typescript#mcpserverconfig) pour le détail de chaque type de transport.

### `AccountInfo`

Informations de compte de l'utilisateur authentifié.

```typescript theme={null}
type AccountInfo = {
  email?: string;
  organization?: string;
  subscriptionType?: string;
  tokenSource?: string;
  apiKeySource?: string;
};
```

### `ModelUsage`

Statistiques d'usage par modèle renvoyées dans les messages de résultat. La valeur `costUSD` est une estimation côté client. Voir [Suivre le coût et l'usage](https://code.claude.com/docs/en/agent-sdk/cost-tracking) pour les réserves concernant la facturation.

```typescript theme={null}
type ModelUsage = {
  inputTokens: number;
  outputTokens: number;
  thinkingTokens?: number;
  cacheReadInputTokens: number;
  cacheCreationInputTokens: number;
  webSearchRequests: number;
  costUSD: number;
  contextWindow: number;
  maxOutputTokens: number;
  canonicalModel?: string;
  provider?: string;
  costBasis?: 'list' | 'managed' | 'unknown';
};
```

`thinkingTokens` compte les tokens de réflexion générés par ce modèle. `outputTokens` les inclut déjà : n'additionnez donc pas les deux. Le champ est absent tant qu'aucun tour ne s'est exécuté sur une version de Claude Code qui l'enregistre ; une session reprise qui a commencé sur une version antérieure indique donc un compte partiel. `thinkingTokens` nécessite l'Agent SDK v0.3.257+.

Les champs `canonicalModel` et `provider` nécessitent Claude Code v2.1.218+. `canonicalModel` est l'ID de modèle canonique utilisé pour la recherche de tarif ; il peut différer de la chaîne de modèle brute qui sert de clé à l'entrée, par exemple quand cette chaîne est un ID propre à un fournisseur ou un alias.

`provider` nomme le backend d'API qui a servi le modèle, comme `firstParty`, `bedrock`, `vertex`, `foundry`, `anthropicAws`, `mantle` ou `gateway`.

`costBasis` nomme la table de prix qui a tarifé la dernière requête du modèle : `list` pour le prix catalogue, `managed` pour une table [`modelPricing`](https://code.claude.com/docs/en/settings-reference#modelpricing), ou `unknown` quand aucune ne correspondait à l'ID du modèle. Ce champ nécessite Claude Code v2.1.246+.

### `ConfigScope`

```typescript theme={null}
type ConfigScope = "local" | "user" | "project";
```

### `NonNullableUsage`

Une version de [`Usage`](https://code.claude.com/docs/en/agent-sdk/typescript#usage) où tous les champs nullables deviennent non nullables, sauf `fallback_credit`, qui peut encore valoir `null`.

```typescript theme={null}
type NonNullableUsage = {
  [K in keyof Usage]: K extends "fallback_credit"
    ? Usage[K]
    : NonNullable<Usage[K]>;
};
```

### `Usage`

Statistiques d'utilisation des tokens. C'est le type `BetaUsage` de `@anthropic-ai/sdk`.

```typescript theme={null}
type Usage = {
  input_tokens: number;
  output_tokens: number;
  cache_creation_input_tokens: number | null;
  cache_read_input_tokens: number | null;
  cache_creation: {
    ephemeral_5m_input_tokens: number;
    ephemeral_1h_input_tokens: number;
  } | null;
  server_tool_use: BetaServerToolUsage | null;
  service_tier: "standard" | "priority" | "batch" | null;
  speed: "standard" | "fast" | null;
  inference_geo: string | null;
  iterations: BetaIterationsUsage | null;
  output_tokens_details: BetaOutputTokensDetails | null;
  fallback_credit: BetaFallbackCreditUsage | null;
};
```

`BetaServerToolUsage`, `BetaIterationsUsage`, `BetaOutputTokensDetails` et `BetaFallbackCreditUsage` sont définis dans `@anthropic-ai/sdk`.

`output_tokens_details` ventile la sortie facturée par catégorie. Il contient actuellement un seul champ, `thinking_tokens: number`, qui compte les tokens de sortie générés par le modèle comme raisonnement interne, délimiteurs des blocs de réflexion compris. Le champ `output_tokens_details` nécessite le SDK TypeScript v0.3.228+, qui embarque Claude Code v2.1.228.

* **Facturation** : lisez cette ventilation pour l'observabilité, pas pour la facturation. `output_tokens` reste le total qui fait foi, et `output_tokens - thinking_tokens` donne une approximation de la sortie hors raisonnement.
* **Ce que couvre le compte** : le raisonnement brut produit par le modèle, qui peut être plus long que le texte de réflexion renvoyé dans le corps de la réponse. L'API le calcule en re-tokenisant ce texte brut ; il peut donc différer de quelques tokens du nombre exact généré par le modèle.
* **Streaming** : sur les messages assistant diffusés en streaming, cette ventilation, comme `output_tokens`, est un espace réservé de `message_start` et ne porte aucun compte réel ; lisez-la donc dans le `usage` du message de résultat, comme le décrit [Lire les tokens de sortie depuis le message de résultat](https://code.claude.com/docs/en/agent-sdk/cost-tracking#read-output-tokens-from-the-result-message). Sur le message de résultat, `thinking_tokens` vaut `0` quand le modèle ou le fournisseur ne fournit aucune ventilation.
* **Cas `null`** : `output_tokens_details` lui-même vaut `null` sur les messages assistant que Claude Code synthétise, comme les messages d'erreur d'API.

La présence de `fallback_credit` dans `Usage` dépend de votre version installée de `@anthropic-ai/sdk`, qui l'a ajouté en 0.115.0.

### `CallToolResult`

Type de résultat d'outil MCP (issu de `@modelcontextprotocol/sdk/types.js`). `structuredContent` est un objet JSON qui peut être renvoyé en plus de `content`, y compris avec des blocs image. Voir [Renvoyer des données structurées](https://code.claude.com/docs/en/agent-sdk/custom-tools#return-structured-data).

```typescript theme={null}
type CallToolResult = {
  content: Array<{
    type: "text" | "image" | "audio" | "resource" | "resource_link";
    // Les champs supplémentaires varient selon le type
  }>;
  structuredContent?: Record<string, unknown>;
  isError?: boolean;
};
```

### `SDKMcpResourceLink`

Un fichier qu'un outil MCP a renvoyé par référence. Claude Code construit chaque entrée à partir d'un bloc `resource_link` du résultat de l'outil et livre la liste comme `resourceLinks` dans [`SDKUserMessage.tool_use_result`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkusermessage), ou comme `resource_links` dans [`SDKTaskNotificationMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdktasknotificationmessage) quand l'appel s'est terminé en arrière-plan. Nécessite l'Agent SDK v0.3.257+.

```typescript theme={null}
type SDKMcpResourceLink = {
  uri: string;
  name: string;
  title?: string;
  description?: string;
  mimeType?: string;
  size?: number;
  annotations?: Record<string, unknown>;
};
```

Claude Code abandonne un bloc dont `uri` ou `name` n'est pas une chaîne, et omet un champ optionnel dont la valeur n'est pas du type indiqué.

| Champ | Type | Description |
| :- | :- | :- |
| `uri` | `string` | URI de la ressource, telle que renvoyée par le serveur |
| `name` | `string` | Nom donné à la ressource par le serveur |
| `title` | `string \| undefined` | Titre d'affichage, quand le serveur en a défini un |
| `description` | `string \| undefined` | Description, quand le serveur en a défini une |
| `mimeType` | `string \| undefined` | Type MIME, quand le serveur en a défini un |
| `size` | `number \| undefined` | Taille en octets, quand le serveur l'a définie |
| `annotations` | `Record<string, unknown> \| undefined` | L'objet d'annotations MCP du bloc, quand le serveur en a défini un |

### `ThinkingConfig`

Contrôle le comportement de réflexion/raisonnement de Claude. Prioritaire sur l'option dépréciée `maxThinkingTokens`.

```typescript theme={null}
type ThinkingDisplay = "summarized" | "omitted";

type ThinkingConfig =
  | { type: "adaptive"; display?: ThinkingDisplay } // Le modèle décide quand et combien raisonner (Opus 4.6+)
  | { type: "enabled"; budgetTokens?: number; display?: ThinkingDisplay } // Budget fixe de tokens de réflexion
  | { type: "disabled" }; // Pas de réflexion étendue
```

Le champ optionnel `display` contrôle si le texte de réflexion est renvoyé `"summarized"` (résumé) ou `"omitted"` (omis). Sur Claude Opus 4.7 et ultérieurs, la valeur par défaut de l'API est `"omitted"` : définissez donc `"summarized"` pour recevoir le contenu de réflexion dans des blocs `thinking`. Claude Code ne transmet pas votre valeur `display` à certains fournisseurs, comme Amazon Bedrock et Google Cloud's Agent Platform. Sur ces fournisseurs, Opus 4.7 et ultérieurs renvoient des blocs `thinking` vides même si vous définissez `display` à `"summarized"`.

### `SpawnedProcess`

Interface pour le lancement de processus personnalisé (utilisée avec l'option `spawnClaudeCodeProcess`). `ChildProcess` satisfait déjà cette interface.

```typescript theme={null}
interface SpawnedProcess {
  stdin: Writable;
  stdout: Readable;
  readonly killed: boolean;
  readonly exitCode: number | null;
  kill(signal: NodeJS.Signals): boolean;
  on(
    event: "exit",
    listener: (code: number | null, signal: NodeJS.Signals | null) => void
  ): void;
  on(event: "error", listener: (error: Error) => void): void;
  once(
    event: "exit",
    listener: (code: number | null, signal: NodeJS.Signals | null) => void
  ): void;
  once(event: "error", listener: (error: Error) => void): void;
  off(
    event: "exit",
    listener: (code: number | null, signal: NodeJS.Signals | null) => void
  ): void;
  off(event: "error", listener: (error: Error) => void): void;
}
```

### `SpawnOptions`

Options passées à la fonction de lancement (spawn) personnalisée.

```typescript theme={null}
interface SpawnOptions {
  command: string;
  args: string[];
  cwd?: string;
  env: Record<string, string | undefined>;
  signal: AbortSignal;
}
```

> **Note** : le champ `signal` indique à votre fonction de lancement quand détruire le processus. Passez-le comme option `signal` au `spawn()` de Node, ou transmettez-le au gestionnaire de démontage de votre VM ou conteneur.
>
> Ce signal ne se déclenche pas à l'instant où [`Options.abortController`](https://code.claude.com/docs/en/agent-sdk/typescript#options) avorte. Le SDK ferme d'abord le stdin du processus et attend environ deux secondes pour que la CLI s'arrête proprement, puis avorte ce signal. Pour réagir dès que l'appelant avorte, écoutez plutôt votre propre `Options.abortController.signal`, que votre fonction de lancement peut référencer depuis sa portée englobante.

### `McpSetServersResult`

Résultat d'une opération `setMcpServers()`.

```typescript theme={null}
type McpSetServersResult = {
  added: string[];
  removed: string[];
  errors: Record<string, string>;
};
```

Quand vous appelez `setMcpServers()`, Claude Code applique ces règles :

* **Serveurs que l'appel ne nomme pas** : en dehors d'une [session cloud](https://code.claude.com/docs/en/claude-code-on-the-web), Claude Code déconnecte les serveurs ajoutés par un appel `setMcpServers()` antérieur ainsi que les serveurs SDK en processus, et les liste dans `removed`. Les autres serveurs continuent de tourner et ne sont pas listés dans `removed`, notamment les serveurs stdio, HTTP et SSE de l'option [`mcpServers`](https://code.claude.com/docs/en/agent-sdk/typescript#options), les serveurs des fichiers de réglages et les serveurs fournis par des plugins.
* **Serveurs que l'appel nomme** : Claude Code remplace un serveur stdio, HTTP ou SSE ajouté par un appel `setMcpServers()` antérieur uniquement si sa configuration diffère de celle que vous avez passée. Un serveur SDK en processus déjà enregistré sous ce nom reste tel quel ; pour en échanger un, omettez-le dans un appel puis ajoutez-le dans le suivant.
* **Serveurs intégrés démarrés par la CLI au lancement** : si l'appel en nomme un, Claude Code abandonne cette entrée et la signale dans `errors`.

La promesse se résout une fois que les serveurs stdio, HTTP et SSE nouvellement ajoutés se sont connectés ou ont échoué ; les outils des serveurs connectés sont donc disponibles au tour suivant.

`added` liste les serveurs que Claude Code a ajoutés ou remplacés, qu'ils se soient connectés ou non. Un serveur qui n'a pas réussi à se connecter apparaît à la fois dans `added` et dans `errors`, avec le texte de l'échec sous `errors` et une ligne `failed` dans [`mcpServerStatus()`](https://code.claude.com/docs/en/agent-sdk/typescript#methods). Avant Claude Code v2.1.257, un serveur dont la tentative de connexion levait une exception n'était signalé que sous `errors`.

### `RewindFilesResult`

Résultat d'une opération `rewindFiles()`.

```typescript theme={null}
type RewindFilesResult = {
  canRewind: boolean;
  error?: string;
  filesChanged?: string[];
  insertions?: number;
  deletions?: number;
  skippedLinks?: number;
};
```

`skippedLinks` compte les chemins suivis que le retour arrière (rewind) a refusé de restaurer ou de supprimer par sécurité vis-à-vis des liens : un lien symbolique, un lien physique ou un autre fichier non régulier au chemin suivi, un répertoire parent qui ne se résout plus vers l'endroit où il pointait lors de la prise du point de contrôle, ou une sauvegarde qui n'a pas pu être lue en toute sécurité. Ce champ nécessite Claude Code v2.1.216+. Un appel d'aperçu avec `rewindFiles(userMessageId, { dryRun: true })` ne le définit jamais.

### `SDKStatusMessage`

Message de mise à jour de statut (par exemple, compactage en cours).

```typescript theme={null}
type SDKStatusMessage = {
  type: "system";
  subtype: "status";
  status: "compacting" | null;
  permissionMode?: PermissionMode;
  uuid: UUID;
  session_id: string;
};
```

### `SDKTaskNotificationMessage`

Notification émise quand une tâche d'arrière-plan se termine, échoue ou est arrêtée. Les tâches d'arrière-plan incluent les commandes Bash `run_in_background`, les surveillances [Monitor](https://code.claude.com/docs/en/agent-sdk/typescript#monitor) et les sous-agents d'arrière-plan. Pour les champs `run_id` et `ambient`, voir [`SDKTaskStartedMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdktaskstartedmessage), qui les définit avec leurs exigences de version.

```typescript theme={null}
type SDKTaskNotificationMessage = {
  type: "system";
  subtype: "task_notification";
  task_id: string;
  run_id?: string;
  tool_use_id?: string;
  status: "completed" | "failed" | "stopped";
  reason?: "worker_restart";
  output_file: string;
  summary: string;
  ambient?: boolean;
  usage?: {
    total_tokens: number;
    tool_uses: number;
    duration_ms: number;
  };
  resource_links?: SDKMcpResourceLink[];
  uuid: UUID;
  session_id: string;
};
```

`reason` est défini quand une tâche se termine pour une cause autre que son propre achèvement, échec ou arrêt ; il nécessite l'Agent SDK v0.3.273+. Claude Code ne le définit que dans les sessions qui se connectent via claude.ai : sessions cloud, y compris sur des runners auto-hébergés, et sessions Remote Control. Un appel local `query()` ne le définit jamais. Son unique valeur, `worker_restart`, signifie que le processus Claude Code qui exécutait la tâche a redémarré. La notification porte le statut `"stopped"` : traitez donc la tâche comme ni terminée ni échouée.

Quand Claude Code [passe un long appel d'outil MCP en arrière-plan](https://code.claude.com/docs/en/mcp#automatic-backgrounding-of-long-tool-calls), le bloc `tool_result` de cet appel ne contient qu'un espace réservé et le vrai résultat de l'appel arrive dans cette notification. Associez la notification à l'appel grâce à `tool_use_id`. Sur une notification `completed`, `resource_links` liste les fichiers que l'outil a renvoyés par référence sous forme d'entrées [`SDKMcpResourceLink`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkmcpresourcelink), avec les mêmes limites de 50 liens et 64 Kio que [`tool_use_result.resourceLinks`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkusermessage). Claude Code omet `resource_links` quand le résultat n'avait pas de liens et sur les notifications de tâches qui ne sont pas des appels d'outils MCP. `resource_links` nécessite l'Agent SDK v0.3.257+.

Claude Code ajoute en tête un avis à chaque notification de tâche qu'il envoie au modèle, sauf pour les livraisons marquées du [sous-type `scheduled-trigger`](https://code.claude.com/docs/en/agent-sdk/typescript#task-notification-subkinds), qui portent à la place un cadrage de tâche assignée. L'avis indique qu'aucune saisie humaine n'a eu lieu, afin que le modèle ne traite pas la notification comme une instruction ou une approbation de l'utilisateur.

Pour détecter un tour de notification de tâche, vérifiez `origin.kind === "task-notification"` sur le [`SDKUserMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkusermessage) ou le [`SDKResultMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkresultmessage), plutôt que de vous fier au texte de l'avis. Lisez `subkind` dans ce même champ si vous devez savoir ce qui l'a déclenchée. Avant la v2.1.205, Claude Code n'ajoutait pas l'avis aux notifications arrivées pendant que la session était inactive.

### `SDKToolUseSummaryMessage`

Résumé de l'utilisation des outils dans une conversation.

```typescript theme={null}
type SDKToolUseSummaryMessage = {
  type: "tool_use_summary";
  summary: string;
  preceding_tool_use_ids: string[];
  uuid: UUID;
  session_id: string;
};
```

### `SDKHookStartedMessage`

Émis quand un hook commence à s'exécuter.

Claude Code livre ce message, [`SDKHookProgressMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkhookprogressmessage) et [`SDKHookResponseMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkhookresponsemessage) dans le flux de messages immédiatement, y compris pendant qu'un hook `SessionStart` ou `Setup` est encore en cours au démarrage de la session. Les versions v2.1.169 à v2.1.203 de Claude Code livraient ces messages en un seul lot après la fin d'un hook `SessionStart` ou `Setup` ; la v2.1.204 a rétabli la livraison en direct.

```typescript theme={null}
type SDKHookStartedMessage = {
  type: "system";
  subtype: "hook_started";
  hook_id: string;
  hook_name: string;
  hook_event: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKHookProgressMessage`

Émis pendant l'exécution d'un hook, avec sa sortie stdout/stderr.

```typescript theme={null}
type SDKHookProgressMessage = {
  type: "system";
  subtype: "hook_progress";
  hook_id: string;
  hook_name: string;
  hook_event: string;
  stdout: string;
  stderr: string;
  output: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKHookResponseMessage`

Émis quand un hook termine son exécution.

```typescript theme={null}
type SDKHookResponseMessage = {
  type: "system";
  subtype: "hook_response";
  hook_id: string;
  hook_name: string;
  hook_event: string;
  output: string;
  stdout: string;
  stderr: string;
  exit_code?: number;
  outcome: "success" | "error" | "cancelled";
  uuid: UUID;
  session_id: string;
};
```

### `SDKToolProgressMessage`

Émis périodiquement pendant l'exécution d'un outil pour indiquer sa progression.

```typescript theme={null}
type SDKToolProgressMessage = {
  type: "tool_progress";
  tool_use_id: string;
  tool_name: string;
  parent_tool_use_id: string | null;
  elapsed_time_seconds: number;
  task_id?: string;
  heartbeat?: boolean;
  subagent_type?: string;
  subagent_retry?: {
    agent_id: string;
    attempt: number;
    max_retries: number;
    retry_delay_ms: number;
    error_status: number | null;
    error_category: string;
  };
  uuid: UUID;
  session_id: string;
};
```

Pendant qu'un appel d'outil s'exécute dans la conversation principale, Claude Code émet un message `tool_progress` toutes les 30 secondes avec `heartbeat: true`. Chaque battement (heartbeat) porte le nom de l'outil et le nombre de secondes écoulées, ce qui permet de distinguer un appel long d'une session bloquée. Claude Code n'émet pas de battements pour les appels d'outils à l'intérieur d'un sous-agent. Le champ `heartbeat` nécessite l'Agent SDK v0.3.214+. Avant la v2.1.257, Claude Code n'émettait pas non plus de battements pour un appel d'outil Agent au premier plan.

Sur les messages `tool_progress` de l'outil Agent autres que les battements, `subagent_type` nomme le type de sous-agent en cours, comme `general-purpose`. `subagent_retry` est présent pendant que ce sous-agent attend la fin d'un délai de réessai (backoff) après une erreur d'API, comme une limite de débit ou une surcharge, avec un message par tentative. Ces deux champs nécessitent l'Agent SDK v0.3.214+.

Pour afficher un indicateur de réessai à partir de `subagent_retry` :

* Suivez l'indicateur par `parent_tool_use_id`, unique pour chaque sous-agent. `tool_use_id` est partagé par les sous-agents parallèles issus d'un même tour assistant ; suivre par ce champ laisserait la mise à jour d'un sous-agent effacer l'indicateur d'un autre.
* Effacez l'indicateur quand un `tool_progress` ultérieur pour le même `parent_tool_use_id` arrive sans `subagent_retry` ni `heartbeat: true`, ou quand arrive le message de résultat de l'outil. Les trames avec `heartbeat: true` ne signalent que la vivacité : conservez l'indicateur quand l'une d'elles arrive. `attempt` peut dépasser `max_retries` en cas de réessai persistant : ne déduisez donc pas l'effacement des compteurs.
* Traitez `error_category` comme un jeton servant à choisir votre propre texte de message, pas comme un texte à afficher. Les valeurs sont `rate_limit`, `overloaded`, `authentication_failed`, `server_error`, `cloud_credential_error` et `unknown`. Traitez une valeur inconnue comme `unknown`, car des versions ultérieures peuvent ajouter des valeurs.

### `SDKAuthStatusMessage`

Émis pendant les flux d'authentification.

```typescript theme={null}
type SDKAuthStatusMessage = {
  type: "auth_status";
  isAuthenticating: boolean;
  output: string[];
  error?: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKTaskStartedMessage`

Émis quand une tâche démarre. Le champ `task_type` vaut `"local_bash"` pour les commandes Bash et les surveillances [Monitor](https://code.claude.com/docs/en/agent-sdk/typescript#monitor), `"local_agent"` pour les sous-agents, ou `"remote_agent"`.

```typescript theme={null}
type SDKTaskStartedMessage = {
  type: "system";
  subtype: "task_started";
  task_id: string;
  run_id?: string;
  tool_use_id?: string;
  description: string;
  task_type?: string;
  is_backgrounded?: boolean;
  spawn_depth?: number;
  parent_task_id?: string;
  ambient?: boolean;
  uuid: UUID;
  session_id: string;
};
```

`run_id` identifie une exécution de la tâche. Quand une tâche s'exécute à nouveau sous le même `task_id`, comme un [sous-agent que Claude reprend](https://code.claude.com/docs/en/agent-sdk/subagents#resume-subagents), la nouvelle exécution reçoit un nouveau `run_id`. Ce champ nécessite l'Agent SDK v0.3.292+. Utilisez-le pour distinguer les exécutions d'une même tâche :

* **Associer les messages à une exécution** : les messages `task_started`, [`task_progress`](https://code.claude.com/docs/en/agent-sdk/typescript#sdktaskprogressmessage), [`task_updated`](https://code.claude.com/docs/en/agent-sdk/typescript#sdktaskupdatedmessage) et [`task_notification`](https://code.claude.com/docs/en/agent-sdk/typescript#sdktasknotificationmessage) d'une exécution portent le même `run_id`. L'entrée de l'exécution dans chaque [`SDKBackgroundTasksChangedMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkbackgroundtaskschangedmessage) le porte aussi, de même que le [`runId`](https://code.claude.com/docs/en/agent-sdk/typescript#task-notification-subkinds) dans l'origine du message utilisateur qui livre la notification de l'exécution.
* **Détecter un message tardif** : comparés comme de simples chaînes, les ID d'exécution d'une tâche se trient dans l'ordre de démarrage des exécutions ; un message dont le `run_id` se trie avant l'exécution que vous affichez provient donc d'une exécution antérieure. Hormis cette comparaison, traitez la valeur comme opaque.
* **Se rabattre sur `task_id`** : quand un message ne porte pas de `run_id`, comme le `task_notification` qu'une session reprise émet pour clore une tâche lancée par le processus précédent, attribuez-le par `task_id`.

Par exemple, ces trois messages, réduits à leurs champs d'identification, proviennent d'un sous-agent d'arrière-plan que Claude a repris une fois. Les deux messages `task_started` partagent un `task_id`, et la notification qui a mis fin à la première exécution porte l'ID de la première exécution :

```json theme={null}
{"subtype": "task_started", "task_id": "a5c2e8f41b09d7a36", "run_id": "0mubbs7i8-6f0c2d41"}
{"subtype": "task_notification", "task_id": "a5c2e8f41b09d7a36", "run_id": "0mubbs7i8-6f0c2d41", "status": "completed"}
{"subtype": "task_started", "task_id": "a5c2e8f41b09d7a36", "run_id": "0mubc18m1-b7719e05"}
```

`ambient` vaut `true` pour les tâches qui ne font pas partie du travail de la session, comme les tâches que Claude Code exécute pour son propre fonctionnement. Les observateurs de mise à jour en direct sont aussi ambiants, y compris ceux que l'utilisateur a demandés. Excluez les tâches ambiantes des indicateurs d'activité. Ce champ nécessite l'Agent SDK v0.3.247+.

`ambient` apparaît aussi sur [`SDKTaskNotificationMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdktasknotificationmessage) et sur les entrées de [`SDKBackgroundTasksChangedMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkbackgroundtaskschangedmessage).

`is_backgrounded` et `spawn_depth` décrivent la façon dont Claude Code a démarré la tâche. Ces deux champs nécessitent l'Agent SDK v0.3.238+.

* `is_backgrounded` : Claude Code le définit sur les tâches `"local_agent"` et `"local_bash"`. `true` signifie que la tâche s'exécute en arrière-plan. `false` signifie qu'elle s'exécute au premier plan, et l'appel d'outil qui l'a lancée reste bloqué jusqu'à ce que la tâche se termine ou passe en arrière-plan.
* `spawn_depth` : Claude Code le définit uniquement sur les tâches `"local_agent"`. Un sous-agent lancé par le fil principal a une profondeur `1`. Un sous-agent lancé par un sous-agent de profondeur `1` a une profondeur `2`, et ainsi de suite.

Un [sous-agent repris](https://code.claude.com/docs/en/agent-sdk/subagents#resume-subagents) signale toujours `is_backgrounded: true`, car Claude Code exécute tout sous-agent repris en arrière-plan. Quand une tâche au premier plan passe plus tard en arrière-plan, Claude Code signale la nouvelle valeur de `is_backgrounded` dans un message [`task_updated`](https://code.claude.com/docs/en/agent-sdk/typescript#sdktaskupdatedmessage) au lieu d'envoyer un second `task_started`.

`parent_task_id` contient le `task_id` du sous-agent qui a lancé cette tâche. Utilisez-le pour regrouper chaque tâche sous le sous-agent qui l'a démarrée. Claude Code le définit sur les tâches de sous-agent, Bash et [Monitor](https://code.claude.com/docs/en/agent-sdk/typescript#monitor). Ce champ nécessite l'Agent SDK v0.3.292+. Il est absent quand :

* Le fil principal a lancé la tâche
* Claude Code ne suit plus la tâche parente
* Un [coéquipier](https://code.claude.com/docs/en/agent-teams) ou un agent à l'intérieur d'un workflow a lancé la tâche

Le parent peut être une tâche au premier plan ou une tâche déjà terminée : traitez donc un ID que vous ne reconnaissez pas comme une absence de parent.

### `SDKTaskProgressMessage`

Émis périodiquement pendant l'exécution d'un sous-agent ou d'une tâche d'arrière-plan. Pour le champ `run_id`, voir [`SDKTaskStartedMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdktaskstartedmessage), qui le définit avec son exigence de version.

Pour une tâche de sous-agent, le champ `summary` contient un résumé de progression généré par le modèle et n'est rempli que lorsque [`agentProgressSummaries`](https://code.claude.com/docs/en/agent-sdk/typescript#options) est activé. Pour un [appel d'outil MCP passé en arrière-plan](https://code.claude.com/docs/en/mcp#automatic-backgrounding-of-long-tool-calls), `summary` contient la dernière progression signalée par le serveur MCP et ne dépend pas de cette option.

```typescript theme={null}
type SDKTaskProgressMessage = {
  type: "system";
  subtype: "task_progress";
  task_id: string;
  run_id?: string;
  tool_use_id?: string;
  description: string;
  subagent_type?: string;
  usage: {
    total_tokens: number;
    tool_uses: number;
    duration_ms: number;
  };
  last_tool_name?: string;
  summary?: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKTaskUpdatedMessage`

Émis quand l'état d'une tâche d'arrière-plan change, par exemple lors de la transition de `running` à `completed`. Fusionnez `patch` dans votre table locale de tâches indexée par `task_id`. Le champ `end_time` est un horodatage epoch Unix en millisecondes, comparable avec `Date.now()`. Pour le champ `run_id`, voir [`SDKTaskStartedMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdktaskstartedmessage), qui le définit avec son exigence de version.

```typescript theme={null}
type SDKTaskUpdatedMessage = {
  type: "system";
  subtype: "task_updated";
  task_id: string;
  run_id?: string;
  patch: {
    status?: "pending" | "running" | "completed" | "failed" | "killed";
    description?: string;
    end_time?: number;
    total_paused_ms?: number;
    error?: string;
    is_backgrounded?: boolean;
  };
  uuid: UUID;
  session_id: string;
};
```

### `SDKBackgroundTasksChangedMessage`

Émis chaque fois que l'ensemble des tâches d'arrière-plan actives change : une tâche démarre, se termine ou est tuée ; un agent au premier plan passe en arrière-plan ; ou le champ `description`, `ambient` ou `parent_task_id` d'une tâche change. Pour les champs `run_id` et `parent_task_id` de chaque entrée, voir [`SDKTaskStartedMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdktaskstartedmessage), qui les définit avec leurs exigences de version.

Le tableau `tasks` est l'ensemble actif complet. Remplacez tout ensemble en cache par chaque charge utile au lieu d'apparier les événements `task_started` et `task_notification`, afin que le prochain changement d'appartenance corrige tout événement manqué.

Quand une tâche se termine, ses messages [`task_updated`](https://code.claude.com/docs/en/agent-sdk/typescript#sdktaskupdatedmessage) et [`task_notification`](https://code.claude.com/docs/en/agent-sdk/typescript#sdktasknotificationmessage) arrivent avant le `background_tasks_changed` qui la retire de la liste. L'ordre par rapport aux événements par tâche n'est sinon pas spécifié.

Rien n'est émis au démarrage. Réinitialisez à un ensemble vide chaque fois que le processus CLI de la session démarre ou redémarre, et laissez le prochain changement d'appartenance le repeupler.

Quand vous envoyez une requête de contrôle `initialize` répétée à une session en cours, par exemple avec [`reinitialize()`](https://code.claude.com/docs/en/agent-sdk/typescript#query-object) après une coupure du transport, Claude Code fait suivre la réponse d'un instantané de l'ensemble actif courant, même s'il est vide. Un hôte qui se reconnecte apprend donc ce qui tourne sans attendre le prochain changement d'appartenance. Avant l'Agent SDK v0.3.239, Claude Code n'envoyait aucun instantané après un `initialize` répété.

Nécessite Claude Code v2.1.203+.

```typescript theme={null}
type SDKBackgroundTasksChangedMessage = {
  type: "system";
  subtype: "background_tasks_changed";
  tasks: {
    task_id: string;
    run_id?: string;
    task_type: string;
    subagent_type?: string;
    description: string;
    parent_task_id?: string;
    ambient?: boolean;
  }[];
  uuid: UUID;
  session_id: string;
};
```

`subagent_type` nomme le type de sous-agent sur les entrées dont le [`task_type`](https://code.claude.com/docs/en/agent-sdk/typescript#sdktaskstartedmessage) vaut `"local_agent"`, comme `general-purpose` ou le nom d'un sous-agent personnalisé. Ce champ nécessite l'Agent SDK v0.3.293+.

### `SDKThinkingTokensMessage`

Émis pendant que Claude produit un bloc de réflexion, y compris un bloc caviardé (redacted). `estimated_tokens` est une estimation cumulée des tokens de réflexion générés jusqu'ici dans le bloc courant, et `estimated_tokens_delta` est l'incrément porté par cette trame. Utilisez ces estimations pour afficher la progression.

Quand le modèle ou le fournisseur fournit une ventilation, le compte final pour la boucle d'agent de niveau supérieur est le [`usage.output_tokens_details.thinking_tokens`](https://code.claude.com/docs/en/agent-sdk/typescript#usage) du message de résultat, qui [n'inclut pas les tokens des sous-agents](https://code.claude.com/docs/en/agent-sdk/cost-tracking#get-the-total-cost-of-a-query).

Nécessite Claude Code v2.1.153+.

```typescript theme={null}
type SDKThinkingTokensMessage = {
  type: "system";
  subtype: "thinking_tokens";
  estimated_tokens: number;
  estimated_tokens_delta: number;
  user_message_uuid?: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKSessionStateChangedMessage`

Émis quand Claude Code signale l'état de la session. Pour recevoir ces messages, définissez [`CLAUDE_CODE_EMIT_SESSION_STATE_EVENTS=1`](https://code.claude.com/docs/en/env-vars#variables). Claude Code peut signaler le même état plusieurs fois : lisez donc un message comme l'état courant de la session plutôt que comme une transition.

Le champ `state` prend l'une de ces valeurs :

* `running` : la session travaille.
* `idle` : Claude Code attend votre prochain prompt.
* `requires_action` : la session est bloquée en attente d'une réponse à une requête envoyée à votre hôte, comme une demande de permission.

Le message `idle` d'un tour et son message `result` peuvent arriver dans n'importe quel ordre. Pour changer si `idle` attend le travail d'arrière-plan, comme un sous-agent d'arrière-plan ou une exécution de [workflow](https://code.claude.com/docs/en/workflows), voir [`CLAUDE_CODE_BG_TASKS_REPORT_RUNNING`](https://code.claude.com/docs/en/env-vars#variables).

```typescript theme={null}
type SDKSessionStateChangedMessage = {
  type: "system";
  subtype: "session_state_changed";
  state: "idle" | "running" | "requires_action";
  uuid: UUID;
  session_id: string;
};
```

### `SDKFilesPersistedEvent`

Émis quand les points de contrôle de fichiers sont persistés sur disque.

```typescript theme={null}
type SDKFilesPersistedEvent = {
  type: "system";
  subtype: "files_persisted";
  files: { filename: string; file_id: string }[];
  failed: { filename: string; error: string }[];
  processed_at: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKRateLimitEvent`

Émis quand la session rencontre une limite de débit.

```typescript theme={null}
type SDKRateLimitEvent = {
  type: "rate_limit_event";
  rate_limit_info: {
    status: "allowed" | "allowed_warning" | "rejected";
    resetsAt?: number;
    utilization?: number;
    errorCode?: "credits_required";
    canUserPurchaseCredits?: boolean;
    hasChargeableSavedPaymentMethod?: boolean;
  };
  uuid: UUID;
  session_id: string;
};
```

Quand `errorCode` vaut `"credits_required"`, le rejet provient d'un abonnement claude.ai dont l'usage inclus est épuisé, et la session ne peut pas continuer tant que l'utilisateur n'a pas acheté de crédits d'usage. `canUserPurchaseCredits` indique si l'utilisateur authentifié peut acheter des crédits pour le compte, et `hasChargeableSavedPaymentMethod` indique si un moyen de paiement enregistré est disponible. Ces trois champs sont absents des événements de limite de débit qui ne sont pas des rejets de type credits-required. Nécessite Claude Code v2.1.181+.

### `SDKLocalCommandOutputMessage`

Claude Code n'émet pas ce type de message. Quand vous envoyez une commande comme `/context` ou `/usage` en tant que prompt, sa sortie arrive sous forme de [`SDKAssistantMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkassistantmessage).

```typescript theme={null}
type SDKLocalCommandOutputMessage = {
  type: "system";
  subtype: "local_command_output";
  content: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKCommandsChangedMessage`

Émis quand l'ensemble des commandes disponibles change en cours de session, par exemple quand Claude Code découvre des skills lorsque l'agent entre dans un sous-répertoire. Le tableau `commands` est la liste complète mise à jour : remplacez donc toute liste de commandes en cache par cette charge utile. Appeler [`supportedCommands()`](https://code.claude.com/docs/en/agent-sdk/typescript#query-object) après ce message renvoie la même liste mise à jour, car la méthode suit la dernière mise à jour poussée ; cela nécessite l'Agent SDK v0.3.216+. Dans les versions antérieures du SDK, `supportedCommands()` renvoie l'instantané capturé à l'initialisation et ne reflète jamais les changements en cours de session.

Claude Code émet aussi ce message quand les [prompts](https://code.claude.com/docs/en/mcp#use-mcp-prompts-as-commands) d'un serveur MCP rejoignent ou quittent la liste, par exemple quand un serveur finit de se connecter après le démarrage de la session. Cela nécessite Claude Code v2.1.281+.

```typescript theme={null}
type SDKCommandsChangedMessage = {
  type: "system";
  subtype: "commands_changed";
  commands: SlashCommand[];
  uuid: UUID;
  session_id: string;
};
```

### `SDKPromptSuggestionMessage`

Émis après un tour quand [`promptSuggestions`](https://code.claude.com/docs/en/agent-sdk/typescript#options) est activé et que Claude Code a généré une suggestion pour ce tour. Contient le prochain prompt utilisateur prédit. Pour les tours qui n'en reçoivent pas, voir [Quand Claude Code saute les suggestions](https://code.claude.com/docs/en/interactive-mode#when-claude-code-skips-suggestions).

```typescript theme={null}
type SDKPromptSuggestionMessage = {
  type: "prompt_suggestion";
  suggestion: string;
  uuid: UUID;
  session_id: string;
};
```

### `SDKConversationResetMessage`

Émis quand la conversation de la session est remplacée sans que la session se termine. Dans un appel `query()`, seuls `/clear` et ses alias produisent ce message. Montez une transcription vide sous `new_conversation_id` et supprimez tout titre de session en cache.

```typescript theme={null}
type SDKConversationResetMessage = {
  type: "conversation_reset";
  new_conversation_id: UUID;
  uuid: UUID;
  session_id: string;
  trigger?: "clear" | "plan_mode_exit" | "fresh_session" | "onboarding";
  user_message_uuid?: string;
  timestamp?: string;
};
```

Les champs optionnels décrivent la réinitialisation :

* `trigger` : ce qui a abandonné la conversation. Réinitialisez votre transcription à chaque message `conversation_reset`, y compris quand ce champ est absent ou porte une valeur que vous ne reconnaissez pas.
* `user_message_uuid` : l'`uuid` du message utilisateur qui portait le `/clear`. Utilisez-le pour associer la réinitialisation à ce message.
* `timestamp` : moment de la réinitialisation, sous forme de chaîne ISO 8601 en UTC. Utilisez-le pour l'affichage, pas pour ordonner les messages.

Les champs `trigger`, `user_message_uuid` et `timestamp` nécessitent Claude Code v2.1.281+.

Les typages publiés du SDK déclarent `SDKConversationResetMessage` à partir de Claude Code v2.1.203. Avant la v2.1.203, `SDKMessage` référençait le type sans le déclarer, si bien que le rétrécissement de type (narrowing) sur `type === "conversation_reset"` échouait à la vérification de types quand `skipLibCheck` était désactivé.

### `AbortError`

Classe d'erreur personnalisée pour les opérations d'abandon (abort).

```typescript theme={null}
class AbortError extends Error {}
```

`AbortError` est la seule classe d'erreur de l'API typée du SDK. Les autres échecs, comme la sortie du processus Claude Code ou son échec au lancement, rejettent l'itération des messages avec des erreurs qui ne portent aucune classe SDK sur laquelle faire une correspondance. La page [Dépannage](https://code.claude.com/docs/en/agent-sdk/troubleshooting) répertorie ces erreurs par message, avec la cause et le correctif de chacune.
