# Suivre les todos

> Source : https://code.claude.com/docs/en/agent-sdk/todo-tracking · [← Index](../31-index-documentation-officielle.md)

*Suivre les todos dans les sessions Agent SDK et afficher la progression de Claude dans votre application à partir d'appels d'outils structurés.*

Claude Code fournit les [outils de suivi de tâches](https://code.claude.com/docs/en/tools-reference#task-tool-availability) par défaut uniquement sur les modèles listés dans la section « Disponibilité selon le modèle ». Les modèles plus récents suivent un travail en plusieurs étapes sans liste de todos écrite : sur ces modèles, vous n'avez besoin de rien sur cette page pour que Claude mène à bien des tâches en plusieurs étapes.

Dans une session qui dispose des outils de suivi de tâches, Claude tient une liste de todos écrite et met à jour le statut de chaque élément au fil du travail. Chaque changement apparaît dans le flux de messages sous forme d'appel d'outil structuré. N'activez cette fonction (opt-in) pour une session que si votre application lit ces appels d'outils, que ce soit pour journaliser l'activité des tâches ou pour afficher sa propre vue de progression.

## Disponibilité selon le modèle

> **Note** : les outils suivants ne sont disponibles par défaut que sur Claude Opus 4.7, Claude Sonnet 4.6, Claude Haiku 4.5 et les modèles antérieurs. Sur tous les autres modèles, y compris les ID de modèle que Claude Code ne reconnaît pas, ils ne sont pas disponibles sauf activation explicite (opt-in) :
>
> - `TodoWrite`
> - `TaskCreate`
> - `TaskGet`
> - `TaskUpdate`
> - `TaskList`
>
> Partout où les outils sont disponibles, Claude Code fournit les quatre outils Task, ou `TodoWrite` à leur place si vous définissez `CLAUDE_CODE_ENABLE_TASKS=0`.
>
> Ce jeu par défaut s'applique à partir de Claude Code v2.1.268, que le TypeScript Agent SDK embarque à partir de v0.3.268.

Sur un modèle qui n'a pas ces outils par défaut, et à moins d'activer la session, vous ne voyez aucun bloc `tool_use` pour eux dans le flux de messages. L'Agent SDK applique ces valeurs par défaut via le binaire Claude Code qu'il embarque. Si vous pointez `pathToClaudeCodeExecutable` (TypeScript) ou `cli_path` (Python) vers votre propre installation de Claude Code, vous obtenez les outils que fournit cette installation, selon ses propres valeurs par défaut. Pour connaître le jeu exact dans une session en cours, [vérifiez quels outils sont disponibles](https://code.claude.com/docs/en/tools-reference#check-which-tools-are-available). Pour activer une session, faites l'une des choses suivantes :

- Nommez l'un des outils dans l'option [`allowedTools`](https://code.claude.com/docs/en/agent-sdk/permissions#allow-and-deny-rules) (TypeScript) ou `allowed_tools` (Python).
- Listez les outils dans l'option `tools`, qui restreint les outils intégrés de la session à ceux qu'elle nomme. Incluez les outils voulus en plus des autres outils intégrés que vous utilisez.
- Définissez `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` dans l'option `env`, comme le font les exemples de cette page. En TypeScript, `env` remplace l'environnement du sous-processus : étalez `...process.env` pour conserver les variables héritées. En Python, `env` est fusionné par-dessus l'environnement hérité.

## Cycle de vie d'un todo

Claude fait passer chaque todo par un cycle de vie prévisible :

1. **Créé** : Claude ajoute le todo avec le statut `pending` lorsqu'il identifie une tâche.
2. **Activé** : Claude passe le todo à `in_progress` lorsqu'il commence le travail.
3. **Terminé** : Claude le marque comme terminé (completed) lorsque la tâche aboutit.
4. **Supprimé** : Claude supprime un todo dont il n'a plus besoin en définissant `status: "deleted"` dans un appel `TaskUpdate`.

## Quand Claude crée des todos

Dans une session qui dispose des outils de suivi de tâches (voir « Disponibilité selon le modèle »), Claude crée des todos pour la plupart des travaux en plusieurs étapes, par exemple :

- **Tâches complexes en plusieurs étapes** nécessitant trois actions distinctes ou plus.
- **Listes de tâches fournies par l'utilisateur** lorsque plusieurs éléments sont mentionnés.
- **Opérations longues** qui bénéficient d'un suivi de progression.
- **Demandes explicites** lorsque l'utilisateur demande une organisation en todos.

Claude peut se passer de todos pour les demandes très courtes ou en une seule étape.

## Exemples

Avant d'exécuter ces exemples, installez le Claude Agent SDK en suivant le [quickstart](https://code.claude.com/docs/en/agent-sdk/quickstart). Tous les exemples de cette page partagent la même configuration de permissions et le même comportement de sortie :

- **Mode de permission** : les prompts d'exemple demandent à Claude d'effectuer un vrai travail sur un projet ; chaque exemple définit donc `permissionMode: "acceptEdits"` (TypeScript) ou `permission_mode="acceptEdits"` (Python) pour approuver automatiquement les modifications de fichiers produites. Voir [Modes de permission](https://code.claude.com/docs/en/agent-sdk/permissions#permission-modes) pour les alternatives.
- **Limite de tours** : chaque exemple s'exécute jusqu'à ce que l'agent termine et produise son message de résultat final. Si une session atteint d'abord sa limite de tours, ce message de résultat a le sous-type `error_max_turns`. Vérifiez `subtype` pour détecter cette fin.
- **Gestion des erreurs** : ces exemples utilisent des appels `query()` en un seul coup (single-shot). Après avoir produit un résultat `error_max_turns`, `query()` lève une erreur contenant `Reached maximum number of turns`. Chaque exemple enveloppe sa boucle dans un bloc try pour sortir proprement dans ce cas. Voir [Traiter le résultat](https://code.claude.com/docs/en/agent-sdk/agent-loop#handle-the-result) pour les sous-types de résultat.

> **Note** : les messages système de tâche, dont [`SDKTaskNotificationMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdktasknotificationmessage) (TypeScript) ou [`TaskNotificationMessage`](https://code.claude.com/docs/en/agent-sdk/python#tasknotificationmessage) (Python), concernent les tâches d'arrière-plan comme les commandes passées en arrière-plan et les sous-agents. Dans le flux de messages, l'activité des todos apparaît sous forme de blocs `tool_use` dans les messages assistant.

### Surveiller les changements de todos

L'exemple suivant surveille le flux assistant à la recherche de blocs `tool_use` `TaskCreate` et `TaskUpdate`, et affiche une ligne `+` avec le sujet (subject) de chaque nouvelle tâche, ainsi qu'une ligne de mise à jour avec l'ID de la tâche et son nouveau statut à chaque changement de statut. Utilisez cette forme si vous voulez un journal de l'activité des tâches plutôt qu'un affichage rendu. Les lignes `+` n'incluent pas les ID attribués : ce journal ne peut donc pas rattacher les mises à jour à leurs créations. Pour conserver cette correspondance, capturez les ID comme le fait l'exemple « Afficher la progression en temps réel ».

L'entrée `tool_use` diffusée est la forme brute émise par le modèle. Claude Code corrige certains noms de clés presque corrects avant l'exécution, en convertissant `id` ou `task_id` en `taskId` et `active_form` en `activeForm`, mais cette correction n'apparaît pas dans le flux. Lisez les champs d'entrée de `TaskUpdate` de manière défensive, comme le font les deux exemples de cette page, au lieu de supposer que le nom canonique est toujours présent.

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

try {
  for await (const message of query({
    prompt: "Create a static website with a home page, an about page, and a shared stylesheet, and track progress with todos",
    // Conserve les outils Task sur les modèles où Claude Code ne les fournit pas sinon.
    options: { maxTurns: 15, permissionMode: "acceptEdits", env: { ...process.env, CLAUDE_CODE_ENABLE_TODO_TOOLS: "1" } },
  })) {
    if (message.type !== "assistant") continue;
    for (const block of message.message.content) {
      if (block.type !== "tool_use") continue;
      if (block.name === "TaskCreate") {
        const input = block.input as { subject: string };
        console.log(`+ ${input.subject}`);
      } else if (block.name === "TaskUpdate") {
        const input = block.input as {
          taskId?: string;
          id?: string;
          task_id?: string;
          status?: string;
        };
        const taskId = input.taskId ?? input.id ?? input.task_id;
        if (taskId && input.status) console.log(`  ${taskId} -> ${input.status}`);
      }
    }
  }
} catch (error) {
  // Un query() en un seul coup lève une exception après avoir produit un résultat d'erreur.
  console.log(`Session ended with an error: ${error}`);
}
```

#### Python

```python
import asyncio

from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage, ToolUseBlock

async def main():
    try:
        async for message in query(
            prompt="Create a static website with a home page, an about page, and a shared stylesheet, and track progress with todos",
            # Conserve les outils Task sur les modèles où Claude Code ne les fournit pas sinon.
            options=ClaudeAgentOptions(max_turns=15, permission_mode="acceptEdits", env={"CLAUDE_CODE_ENABLE_TODO_TOOLS": "1"}),
        ):
            if not isinstance(message, AssistantMessage):
                continue
            for block in message.content:
                if not isinstance(block, ToolUseBlock):
                    continue
                if block.name == "TaskCreate":
                    print(f"+ {block.input.get('subject', '')}")
                elif block.name == "TaskUpdate" and block.input.get("status"):
                    task_id = (
                        block.input.get("taskId")
                        or block.input.get("id")
                        or block.input.get("task_id")
                    )
                    if task_id:
                        print(f"  {task_id} -> {block.input['status']}")
    except Exception as error:
        # Un query() en un seul coup lève une exception après avoir produit un résultat d'erreur.
        print(f"Session ended with an error: {error}")


asyncio.run(main())
```

### Afficher la progression en temps réel

L'exemple suivant surveille le flux assistant à la recherche de blocs `tool_use` `TaskCreate` et `TaskUpdate`, et tient dans une classe `TaskTracker` une table des tâches indexée par ID de tâche, en réaffichant un résumé de progression à chaque changement. Le résumé compte les tâches terminées et en cours, et affiche pour chaque élément actif son libellé `activeForm` à la place de son `subject`. Utilisez cette forme si votre application maintient un affichage de progression au lieu de journaliser chaque événement.

L'ID attribué à la tâche ne figure pas dans l'entrée de `TaskCreate`. Claude Code fournit la sortie structurée de chaque outil sur le message utilisateur qui porte son bloc `tool_result`, dans le champ `tool_use_result`. Pour `TaskCreate`, cet objet est documenté côté TypeScript sous le nom `TaskCreateOutput` dans [Tool Output Types](https://code.claude.com/docs/en/agent-sdk/typescript#tool-output-types) ; en Python, le champ est un simple dict de même forme. Le tracker associe chaque bloc `tool_result` à son appel `tool_use` via `tool_use_id` et lit `task.id` dans le `tool_use_result` du message associé. Claude peut relire la liste avec `TaskList` et les détails complets d'une tâche avec `TaskGet`.

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

type Task = { subject: string; activeForm?: string; status: string };

class TaskTracker {
  private tasks = new Map<string, Task>();
  private pendingCreates = new Map<string, { subject: string; activeForm?: string }>();

  displayProgress() {
    if (this.tasks.size === 0) {
      console.log("\nProgress: no open tasks\n");
      return;
    }

    const items = [...this.tasks.values()];
    const completed = items.filter((t) => t.status === "completed").length;
    const inProgress = items.filter((t) => t.status === "in_progress").length;

    console.log(`\nProgress: ${completed}/${this.tasks.size} completed`);
    console.log(`Currently working on: ${inProgress} task(s)\n`);

    for (const [id, task] of this.tasks) {
      const icon =
        task.status === "completed" ? "✅" : task.status === "in_progress" ? "🔧" : "❌";
      const text = task.status === "in_progress" && task.activeForm ? task.activeForm : task.subject;
      console.log(`${id}. ${icon} ${text}`);
    }
  }

  handleToolUse(block: { id: string; name: string; input: unknown }) {
    if (block.name === "TaskCreate") {
      const input = block.input as { subject: string; activeForm?: string; active_form?: string };
      this.pendingCreates.set(block.id, {
        subject: input.subject,
        activeForm: input.activeForm ?? input.active_form,
      });
    } else if (block.name === "TaskUpdate") {
      const input = block.input as {
        taskId?: string;
        id?: string;
        task_id?: string;
        status?: string;
        activeForm?: string;
        active_form?: string;
      };
      const taskId = input.taskId ?? input.id ?? input.task_id;
      if (!taskId) return;
      if (input.status === "deleted") {
        this.tasks.delete(taskId);
        this.displayProgress();
        return;
      }
      const task = this.tasks.get(taskId);
      if (!task) return;
      if (input.status) task.status = input.status;
      const active = input.activeForm ?? input.active_form;
      if (active) task.activeForm = active;
      this.displayProgress();
    }
  }

  handleToolResult(block: { tool_use_id: string; is_error?: boolean }, result: unknown) {
    const create = this.pendingCreates.get(block.tool_use_id);
    if (!create) return;
    this.pendingCreates.delete(block.tool_use_id);
    if (block.is_error) return;
    // Le message utilisateur du résultat porte la sortie structurée de l'outil dans
    // tool_use_result ; pour TaskCreate, c'est TaskCreateOutput,
    // { task: { id, subject } }.
    const out = result as { task?: { id: string } };
    if (!out?.task?.id) return;
    this.tasks.set(out.task.id, { ...create, status: "pending" });
    this.displayProgress();
  }

  async trackQuery(prompt: string) {
    try {
      for await (const message of query({
        prompt,
        options: { maxTurns: 20, permissionMode: "acceptEdits", env: { ...process.env, CLAUDE_CODE_ENABLE_TODO_TOOLS: "1" } },
      })) {
        if (message.type === "assistant") {
          for (const block of message.message.content) {
            if (block.type === "tool_use") this.handleToolUse(block);
          }
        }
        if (message.type === "user" && Array.isArray(message.message.content)) {
          for (const block of message.message.content) {
            if (block.type === "tool_result") this.handleToolResult(block, message.tool_use_result);
          }
        }
      }
    } catch (error) {
      // Un query() en un seul coup lève une exception après avoir produit un résultat d'erreur,
      // par exemple quand la limite maxTurns est atteinte.
      console.log(`Session ended with an error: ${error}`);
    }
  }
}

// Utilisation
const tracker = new TaskTracker();
await tracker.trackQuery("Build a complete authentication system with todos");
```

#### Python

```python
import asyncio

from claude_agent_sdk import (
    query,
    ClaudeAgentOptions,
    AssistantMessage,
    UserMessage,
    ToolUseBlock,
    ToolResultBlock,
)


class TaskTracker:
    def __init__(self):
        self.tasks: dict[str, dict] = {}
        self.pending_creates: dict[str, dict] = {}

    def display_progress(self):
        if not self.tasks:
            print("\nProgress: no open tasks\n")
            return

        completed = len([t for t in self.tasks.values() if t["status"] == "completed"])
        in_progress = len([t for t in self.tasks.values() if t["status"] == "in_progress"])

        print(f"\nProgress: {completed}/{len(self.tasks)} completed")
        print(f"Currently working on: {in_progress} task(s)\n")

        for task_id, task in self.tasks.items():
            icon = (
                "✅"
                if task["status"] == "completed"
                else "🔧"
                if task["status"] == "in_progress"
                else "❌"
            )
            text = (
                task["activeForm"]
                if task["status"] == "in_progress" and task.get("activeForm")
                else task["subject"]
            )
            print(f"{task_id}. {icon} {text}")

    def handle_tool_use(self, block: ToolUseBlock):
        if block.name == "TaskCreate":
            self.pending_creates[block.id] = {
                "subject": block.input.get("subject", ""),
                "activeForm": block.input.get("activeForm") or block.input.get("active_form"),
            }
        elif block.name == "TaskUpdate":
            task_id = (
                block.input.get("taskId")
                or block.input.get("id")
                or block.input.get("task_id")
            )
            if not task_id:
                return
            if block.input.get("status") == "deleted":
                self.tasks.pop(task_id, None)
                self.display_progress()
                return
            task = self.tasks.get(task_id)
            if not task:
                return
            if block.input.get("status"):
                task["status"] = block.input["status"]
            active = block.input.get("activeForm") or block.input.get("active_form")
            if active:
                task["activeForm"] = active
            self.display_progress()

    def handle_tool_result(self, block: ToolResultBlock, tool_use_result):
        create = self.pending_creates.pop(block.tool_use_id, None)
        if create is None or block.is_error:
            return
        # Le message utilisateur du résultat porte la sortie structurée de l'outil dans
        # tool_use_result ; pour TaskCreate, c'est {"task": {"id": ..., "subject": ...}}.
        task = (tool_use_result or {}).get("task") or {}
        if not task.get("id"):
            return
        self.tasks[task["id"]] = {**create, "status": "pending"}
        self.display_progress()

    async def track_query(self, prompt: str):
        try:
            async for message in query(
                prompt=prompt,
                options=ClaudeAgentOptions(
                    max_turns=20,
                    permission_mode="acceptEdits",
                    env={"CLAUDE_CODE_ENABLE_TODO_TOOLS": "1"},
                ),
            ):
                if isinstance(message, AssistantMessage):
                    for block in message.content:
                        if isinstance(block, ToolUseBlock):
                            self.handle_tool_use(block)
                if isinstance(message, UserMessage) and isinstance(message.content, list):
                    for block in message.content:
                        if isinstance(block, ToolResultBlock):
                            self.handle_tool_result(block, message.tool_use_result)
        except Exception as error:
            # Un query() en un seul coup lève une exception après avoir produit un résultat d'erreur,
            # par exemple quand la limite max_turns est atteinte.
            print(f"Session ended with an error: {error}")


# Utilisation
async def main():
    tracker = TaskTracker()
    await tracker.track_query("Build a complete authentication system with todos")


asyncio.run(main())
```

## Documentation associée

- [Référence de l'Agent SDK - TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript) : options, types et schémas d'outils du SDK TypeScript, y compris les types d'entrée et de sortie de l'outil Task.
- [Référence de l'Agent SDK - Python](https://code.claude.com/docs/en/agent-sdk/python) : options, types et documentation des outils du SDK Python.
- [Streaming Input](https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode) : les deux modes d'entrée, et quand utiliser l'entrée en streaming plutôt que les appels en un seul coup utilisés dans ces exemples.
- [Donner à Claude des outils personnalisés](https://code.claude.com/docs/en/agent-sdk/custom-tools) : définir vos propres outils avec le serveur MCP intégré au processus (in-process) du SDK.
