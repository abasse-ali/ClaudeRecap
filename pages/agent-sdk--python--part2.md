# Référence de l'Agent SDK — Python — partie 2/2

> Source : https://code.claude.com/docs/en/agent-sdk/python · [← Index](../31-index-documentation-officielle.md)

Cette partie 2/2 couvre les types de hooks, les schémas d'entrée/sortie des outils intégrés, l'exemple d'interface de conversation continue, la gestion des erreurs, la configuration du sandbox et les liens « Voir aussi » (lignes 2086 à 3738 de la page source).

## Types de hooks

Pour un guide complet sur l'utilisation des hooks, avec exemples et schémas courants, voir le [guide des hooks](https://code.claude.com/docs/en/agent-sdk/hooks).

### `HookEvent`

Types d'événements de hook pris en charge.

```python theme={null}
HookEvent = Literal[
    "PreToolUse",  # Appelé avant l'exécution d'un outil
    "PostToolUse",  # Appelé après l'exécution d'un outil
    "PostToolUseFailure",  # Appelé quand l'exécution d'un outil échoue
    "UserPromptSubmit",  # Appelé quand un prompt est soumis
    "Stop",  # Appelé à l'arrêt de l'exécution
    "SubagentStop",  # Appelé quand un sous-agent s'arrête
    "PreCompact",  # Appelé avant la compaction des messages
    "Notification",  # Appelé pour les événements de notification
    "SubagentStart",  # Appelé quand un sous-agent démarre
    "PermissionRequest",  # Appelé quand une décision de permission est nécessaire
]
```

> **Note** : le SDK TypeScript prend en charge des événements de hook supplémentaires, pas encore disponibles en Python. Voir le [tableau de disponibilité des hooks](https://code.claude.com/docs/en/agent-sdk/hooks#available-hooks) pour la prise en charge par SDK.

### `HookCallback`

Définition de type des fonctions de rappel (callbacks) de hook.

```python theme={null}
HookCallback = Callable[[HookInput, str | None, HookContext], Awaitable[HookJSONOutput]]
```

Paramètres :

* `input` : entrée de hook fortement typée, avec unions discriminées selon `hook_event_name` (voir [`HookInput`](#hookinput))
* `tool_use_id` : identifiant facultatif d'utilisation d'outil (pour les hooks liés aux outils)
* `context` : contexte du hook avec des informations supplémentaires

Renvoie un [`HookJSONOutput`](#hookjsonoutput).

### `HookContext`

Informations de contexte transmises aux callbacks de hook.

```python theme={null}
class HookContext(TypedDict):
    signal: Any | None  # Futur : prise en charge d'un signal d'annulation (abort)
```

### `HookMatcher`

Configuration pour associer des hooks à des événements ou outils précis.

```python theme={null}
@dataclass
class HookMatcher:
    matcher: str | None = (
        None  # Nom d'outil ou motif à faire correspondre (ex. "Bash", "Write|Edit")
    )
    hooks: list[HookCallback] = field(
        default_factory=list
    )  # Liste des callbacks à exécuter
    timeout: float | None = (
        None  # Délai en secondes. S'il est omis, le défaut propre à l'événement s'applique :
        # 600 pour la plupart des événements, 30 pour UserPromptSubmit
    )
```

### `HookInput`

Type union de tous les types d'entrée de hook. Le type réel dépend du champ `hook_event_name`.

```python theme={null}
HookInput = (
    PreToolUseHookInput
    | PostToolUseHookInput
    | PostToolUseFailureHookInput
    | UserPromptSubmitHookInput
    | StopHookInput
    | SubagentStopHookInput
    | PreCompactHookInput
    | NotificationHookInput
    | SubagentStartHookInput
    | PermissionRequestHookInput
)
```

### `BaseHookInput`

Champs de base présents dans tous les types d'entrée de hook.

```python theme={null}
class BaseHookInput(TypedDict):
    session_id: str
    transcript_path: str
    cwd: str
    permission_mode: NotRequired[str]
```

| Champ | Type | Description |
| :- | :- | :- |
| `session_id` | `str` | Identifiant de la session en cours |
| `transcript_path` | `str` | Chemin du fichier de transcription de la session |
| `cwd` | `str` | Répertoire de travail courant |
| `permission_mode` | `str` (facultatif) | Mode de permission en cours |

### `PreToolUseHookInput`

Données d'entrée des événements de hook `PreToolUse`.

```python theme={null}
class PreToolUseHookInput(BaseHookInput):
    hook_event_name: Literal["PreToolUse"]
    tool_name: str
    tool_input: dict[str, Any]
    tool_use_id: str
    agent_id: NotRequired[str]
    agent_type: NotRequired[str]
```

| Champ | Type | Description |
| :- | :- | :- |
| `hook_event_name` | `Literal["PreToolUse"]` | Toujours "PreToolUse" |
| `tool_name` | `str` | Nom de l'outil sur le point d'être exécuté |
| `tool_input` | `dict[str, Any]` | Paramètres d'entrée de l'outil |
| `tool_use_id` | `str` | Identifiant unique de cette utilisation d'outil |
| `agent_id` | `str` (facultatif) | Identifiant du sous-agent, présent quand le hook se déclenche dans un sous-agent |
| `agent_type` | `str` (facultatif) | Type du sous-agent, présent quand le hook se déclenche dans un sous-agent |

### `PostToolUseHookInput`

Données d'entrée des événements de hook `PostToolUse`.

```python theme={null}
class PostToolUseHookInput(BaseHookInput):
    hook_event_name: Literal["PostToolUse"]
    tool_name: str
    tool_input: dict[str, Any]
    tool_response: Any
    tool_use_id: str
    agent_id: NotRequired[str]
    agent_type: NotRequired[str]
```

| Champ | Type | Description |
| :- | :- | :- |
| `hook_event_name` | `Literal["PostToolUse"]` | Toujours "PostToolUse" |
| `tool_name` | `str` | Nom de l'outil qui a été exécuté |
| `tool_input` | `dict[str, Any]` | Paramètres d'entrée utilisés |
| `tool_response` | `Any` | Réponse de l'exécution de l'outil |
| `tool_use_id` | `str` | Identifiant unique de cette utilisation d'outil |
| `agent_id` | `str` (facultatif) | Identifiant du sous-agent, présent quand le hook se déclenche dans un sous-agent |
| `agent_type` | `str` (facultatif) | Type du sous-agent, présent quand le hook se déclenche dans un sous-agent |

### `PostToolUseFailureHookInput`

Données d'entrée des événements de hook `PostToolUseFailure`. Appelé quand l'exécution d'un outil échoue.

```python theme={null}
class PostToolUseFailureHookInput(BaseHookInput):
    hook_event_name: Literal["PostToolUseFailure"]
    tool_name: str
    tool_input: dict[str, Any]
    tool_use_id: str
    error: str
    is_interrupt: NotRequired[bool]
    agent_id: NotRequired[str]
    agent_type: NotRequired[str]
```

| Champ | Type | Description |
| :- | :- | :- |
| `hook_event_name` | `Literal["PostToolUseFailure"]` | Toujours "PostToolUseFailure" |
| `tool_name` | `str` | Nom de l'outil qui a échoué |
| `tool_input` | `dict[str, Any]` | Paramètres d'entrée utilisés |
| `tool_use_id` | `str` | Identifiant unique de cette utilisation d'outil |
| `error` | `str` | Message d'erreur de l'exécution échouée |
| `is_interrupt` | `bool` (facultatif) | True quand l'échec est parvenu à Claude Code sous forme d'abandon (abort) plutôt que d'erreur signalée par l'outil. Annuler un outil en cours avec `interrupt()` ne déclenche pas ce hook ; le résultat de l'outil porte alors le message d'interruption |
| `agent_id` | `str` (facultatif) | Identifiant du sous-agent, présent quand le hook se déclenche dans un sous-agent |
| `agent_type` | `str` (facultatif) | Type du sous-agent, présent quand le hook se déclenche dans un sous-agent |

### `UserPromptSubmitHookInput`

Données d'entrée des événements de hook `UserPromptSubmit`.

```python theme={null}
class UserPromptSubmitHookInput(BaseHookInput):
    hook_event_name: Literal["UserPromptSubmit"]
    prompt: str
```

| Champ | Type | Description |
| :- | :- | :- |
| `hook_event_name` | `Literal["UserPromptSubmit"]` | Toujours "UserPromptSubmit" |
| `prompt` | `str` | Le prompt soumis |

### `StopHookInput`

Données d'entrée des événements de hook `Stop`.

```python theme={null}
class StopHookInput(BaseHookInput):
    hook_event_name: Literal["Stop"]
    stop_hook_active: bool
```

| Champ | Type | Description |
| :- | :- | :- |
| `hook_event_name` | `Literal["Stop"]` | Toujours "Stop" |
| `stop_hook_active` | `bool` | Indique si le hook d'arrêt est actif |

### `SubagentStopHookInput`

Données d'entrée des événements de hook `SubagentStop`.

```python theme={null}
class SubagentStopHookInput(BaseHookInput):
    hook_event_name: Literal["SubagentStop"]
    stop_hook_active: bool
    agent_id: str
    agent_transcript_path: str
    agent_type: str
```

| Champ | Type | Description |
| :- | :- | :- |
| `hook_event_name` | `Literal["SubagentStop"]` | Toujours "SubagentStop" |
| `stop_hook_active` | `bool` | Indique si le hook d'arrêt est actif |
| `agent_id` | `str` | Identifiant unique du sous-agent |
| `agent_transcript_path` | `str` | Chemin du fichier de transcription du sous-agent |
| `agent_type` | `str` | Type du sous-agent |

### `PreCompactHookInput`

Données d'entrée des événements de hook `PreCompact`.

```python theme={null}
class PreCompactHookInput(BaseHookInput):
    hook_event_name: Literal["PreCompact"]
    trigger: Literal["manual", "auto"]
    custom_instructions: str | None
```

| Champ | Type | Description |
| :- | :- | :- |
| `hook_event_name` | `Literal["PreCompact"]` | Toujours "PreCompact" |
| `trigger` | `Literal["manual", "auto"]` | Ce qui a déclenché la compaction |
| `custom_instructions` | `str \| None` | Instructions personnalisées pour la compaction |

### `NotificationHookInput`

Données d'entrée des événements de hook `Notification`.

```python theme={null}
class NotificationHookInput(BaseHookInput):
    hook_event_name: Literal["Notification"]
    message: str
    title: NotRequired[str]
    notification_type: str
```

| Champ | Type | Description |
| :- | :- | :- |
| `hook_event_name` | `Literal["Notification"]` | Toujours "Notification" |
| `message` | `str` | Contenu du message de notification |
| `title` | `str` (facultatif) | Titre de la notification |
| `notification_type` | `str` | Type de notification |

### `SubagentStartHookInput`

Données d'entrée des événements de hook `SubagentStart`.

```python theme={null}
class SubagentStartHookInput(BaseHookInput):
    hook_event_name: Literal["SubagentStart"]
    agent_id: str
    agent_type: str
```

| Champ | Type | Description |
| :- | :- | :- |
| `hook_event_name` | `Literal["SubagentStart"]` | Toujours "SubagentStart" |
| `agent_id` | `str` | Identifiant unique du sous-agent |
| `agent_type` | `str` | Type du sous-agent |

### `PermissionRequestHookInput`

Données d'entrée des événements de hook `PermissionRequest`. Permet aux hooks de gérer les décisions de permission par programme.

```python theme={null}
class PermissionRequestHookInput(BaseHookInput):
    hook_event_name: Literal["PermissionRequest"]
    tool_name: str
    tool_input: dict[str, Any]
    permission_suggestions: NotRequired[list[Any]]
    agent_id: NotRequired[str]
    agent_type: NotRequired[str]
```

| Champ | Type | Description |
| :- | :- | :- |
| `hook_event_name` | `Literal["PermissionRequest"]` | Toujours "PermissionRequest" |
| `tool_name` | `str` | Nom de l'outil qui demande la permission |
| `tool_input` | `dict[str, Any]` | Paramètres d'entrée de l'outil |
| `permission_suggestions` | `list[Any]` (facultatif) | Mises à jour de permission suggérées par la CLI |
| `agent_id` | `str` (facultatif) | Identifiant du sous-agent, présent quand le hook se déclenche dans un sous-agent |
| `agent_type` | `str` (facultatif) | Type du sous-agent, présent quand le hook se déclenche dans un sous-agent |

### `HookJSONOutput`

Type union des valeurs de retour des callbacks de hook.

```python theme={null}
HookJSONOutput = AsyncHookJSONOutput | SyncHookJSONOutput
```

#### `SyncHookJSONOutput`

Sortie de hook synchrone, avec champs de contrôle et de décision.

```python theme={null}
class SyncHookJSONOutput(TypedDict):
    # Champs de contrôle
    continue_: NotRequired[bool]  # Indique s'il faut poursuivre (défaut : True)
    suppressOutput: NotRequired[bool]  # Masquer stdout dans la transcription
    stopReason: NotRequired[str]  # Message quand continue vaut False

    # Champs de décision
    decision: NotRequired[Literal["block"]]
    systemMessage: NotRequired[str]  # Message d'avertissement pour l'utilisateur
    reason: NotRequired[str]  # Retour (feedback) destiné à Claude

    # Sortie propre au hook
    hookSpecificOutput: NotRequired[HookSpecificOutput]
```

> **Note** : utilisez `continue_` (avec un tiret bas) dans le code Python. Il est automatiquement converti en `continue` lors de l'envoi à la CLI.

#### `HookSpecificOutput`

Union discriminée de types de sortie `TypedDict` propres à chaque événement. Le champ `hookEventName` détermine quels champs sont valides. Pour le détail complet des champs disponibles par événement de hook, voir [Contrôler l'exécution avec des hooks](https://code.claude.com/docs/en/agent-sdk/hooks#outputs).

```python theme={null}
class PreToolUseHookSpecificOutput(TypedDict):
    hookEventName: Literal["PreToolUse"]
    permissionDecision: NotRequired[Literal["allow", "deny", "ask", "defer"]]
    permissionDecisionReason: NotRequired[str]
    updatedInput: NotRequired[dict[str, Any]]
    additionalContext: NotRequired[str]


class PostToolUseHookSpecificOutput(TypedDict):
    hookEventName: Literal["PostToolUse"]
    additionalContext: NotRequired[str]
    updatedToolOutput: NotRequired[Any]
    updatedMCPToolOutput: NotRequired[Any]  # Outils MCP uniquement. Préférez updatedToolOutput, qui fonctionne pour tous les outils


class PostToolUseFailureHookSpecificOutput(TypedDict):
    hookEventName: Literal["PostToolUseFailure"]
    additionalContext: NotRequired[str]


class UserPromptSubmitHookSpecificOutput(TypedDict):
    hookEventName: Literal["UserPromptSubmit"]
    additionalContext: NotRequired[str]


class NotificationHookSpecificOutput(TypedDict):
    hookEventName: Literal["Notification"]
    additionalContext: NotRequired[str]


class SubagentStartHookSpecificOutput(TypedDict):
    hookEventName: Literal["SubagentStart"]
    additionalContext: NotRequired[str]


class PermissionRequestHookSpecificOutput(TypedDict):
    hookEventName: Literal["PermissionRequest"]
    decision: dict[str, Any]


HookSpecificOutput = (
    PreToolUseHookSpecificOutput
    | PostToolUseHookSpecificOutput
    | PostToolUseFailureHookSpecificOutput
    | UserPromptSubmitHookSpecificOutput
    | NotificationHookSpecificOutput
    | SubagentStartHookSpecificOutput
    | PermissionRequestHookSpecificOutput
)
```

#### `AsyncHookJSONOutput`

Sortie de hook asynchrone, qui diffère l'exécution du hook.

```python theme={null}
class AsyncHookJSONOutput(TypedDict):
    async_: Literal[True]  # Mettre à True pour différer l'exécution
    asyncTimeout: NotRequired[int]  # Délai en millisecondes
```

> **Note** : utilisez `async_` (avec un tiret bas) dans le code Python. Il est automatiquement converti en `async` lors de l'envoi à la CLI.

### Exemple d'utilisation des hooks

Cet exemple enregistre deux hooks : l'un bloque les commandes Bash dangereuses comme `rm -rf /`, l'autre journalise toutes les utilisations d'outils à des fins d'audit. Le hook de sécurité ne s'exécute que sur les commandes Bash (grâce au `matcher`), tandis que le hook de journalisation s'exécute sur tous les outils.

```python theme={null}
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, HookMatcher, HookContext
from typing import Any


async def validate_bash_command(
    input_data: dict[str, Any], tool_use_id: str | None, context: HookContext
) -> dict[str, Any]:
    """Valide et bloque éventuellement les commandes bash dangereuses."""
    if input_data["tool_name"] == "Bash":
        command = input_data["tool_input"].get("command", "")
        if "rm -rf /" in command:
            return {
                "hookSpecificOutput": {
                    "hookEventName": "PreToolUse",
                    "permissionDecision": "deny",
                    "permissionDecisionReason": "Dangerous command blocked",
                }
            }
    return {}


async def log_tool_use(
    input_data: dict[str, Any], tool_use_id: str | None, context: HookContext
) -> dict[str, Any]:
    """Journalise toutes les utilisations d'outils pour audit."""
    print(f"Tool used: {input_data.get('tool_name')}")
    return {}


options = ClaudeAgentOptions(
    hooks={
        "PreToolUse": [
            HookMatcher(
                matcher="Bash", hooks=[validate_bash_command], timeout=120
            ),  # 2 min pour la validation
            HookMatcher(
                hooks=[log_tool_use]
            ),  # S'applique à tous les outils (délai par défaut de l'événement)
        ],
        "PostToolUse": [HookMatcher(hooks=[log_tool_use])],
    }
)

async def main():
    async for message in query(prompt="Analyze this codebase", options=options):
        print(message)


asyncio.run(main())
```

## Types d'entrée/sortie des outils

Documentation des schémas d'entrée/sortie des outils intégrés de Claude Code. Bien que le SDK Python ne les exporte pas sous forme de types, ils représentent la structure des entrées et sorties d'outils dans les messages.

Chaque sortie présentée est la valeur que vous lisez dans [`UserMessage.tool_use_result`](https://code.claude.com/docs/en/agent-sdk/python#usermessage) pour l'outil concerné. Les noms de clés apparaissent exactement tels que Claude Code les émet. Une clé annotée `| None` accompagnée d'un commentaire « présent quand » ou « facultatif » est omise lorsqu'elle ne s'applique pas.

### Agent

**Nom de l'outil :** `Agent`. L'ancien nom `Task` est toujours accepté comme alias, et la liste `tools` du [`SystemMessage`](https://code.claude.com/docs/en/agent-sdk/python#systemmessage) d'initialisation présente cet outil sous le nom `Task` pour la rétrocompatibilité.

**Entrée :**

```python theme={null}
{
    "description": str,  # Courte description de la tâche (3 à 5 mots)
    "prompt": str,  # La tâche que l'agent doit accomplir
    "subagent_type": str | None,  # Le type d'agent spécialisé à utiliser
    "model": "sonnet" | "opus" | "haiku" | "fable" | None,  # Modèle imposé pour cet agent
    "effort": "low" | "medium" | "high" | "xhigh" | "max" | None,  # Effort de raisonnement pour cet agent
    "run_in_background": bool | None,  # Les agents tournent en arrière-plan par défaut ; mettre False pour une exécution synchrone
    "name": str | None,  # Nom de l'agent lancé
    "team_name": str | None,  # Obsolète ; ignoré
    "mode": "acceptEdits" | "auto" | "bypassPermissions" | "default" | "dontAsk" | "plan" | None,  # Obsolète ; ignoré. Les règles d'héritage des sous-agents décident de leur mode de permission
    "isolation": "worktree" | "remote" | None,  # Mode d'isolation des modifications de l'agent
}
```

Lance un nouvel agent pour traiter de façon autonome des tâches complexes en plusieurs étapes.

**Sortie (status : `"completed"`) :**

```python theme={null}
{
    "status": "completed",
    "agentId": str,  # ID de l'agent qui s'est exécuté
    "agentType": str | None,  # Le type de sous-agent qui a traité la tâche
    "content": [  # Blocs de contenu du résultat
        {
            "type": "text",
            "text": str,
            "citations": list | None,
        }
    ],
    "resolvedModel": str | None,  # Modèle sur lequel le sous-agent a démarré
    "modelsUsed": list[str] | None,  # Modèles utilisés dans l'ordre, répétitions consécutives fusionnées
    "totalToolUseCount": int,  # Nombre d'appels d'outils effectués par l'agent
    "totalDurationMs": int,  # Durée d'exécution en millisecondes
    "totalTokens": int,  # Nombre de tokens de la dernière requête API, pas de toute l'exécution
    "usage": {  # Statistiques d'utilisation des tokens
        "input_tokens": int,
        "output_tokens": int,
        "cache_creation_input_tokens": int | None,
        "cache_read_input_tokens": int | None,
        "server_tool_use": {"web_search_requests": int, "web_fetch_requests": int} | None,
        "service_tier": str | None,
        "cache_creation": {"ephemeral_1h_input_tokens": int, "ephemeral_5m_input_tokens": int} | None,
        "inference_geo": str | None,
        "speed": str | None,
        "iterations": Any | None,
        "output_tokens_details": {"thinking_tokens": int | None} | None,
        "fallback_credit": Any | None,
    },
    "toolStats": {  # Activité agrégée des outils pendant l'exécution
        "readCount": int,
        "searchCount": int,
        "bashCount": int,
        "editFileCount": int,
        "linesAdded": int,
        "linesRemoved": int,
        "otherToolCount": int,
        "frameCount": int | None,
    } | None,
    "prompt": str,  # Le prompt exécuté par l'agent
    "worktreePath": str | None,  # Présent quand Claude Code a conservé le worktree du sous-agent
    "worktreeBranch": str | None,  # Présent quand Claude Code a créé ce worktree avec git
}
```

**Sortie (status : `"async_launched"`) :**

```python theme={null}
{
    "status": "async_launched",
    "isAsync": bool | None,  # True pour les lancements en arrière-plan
    "agentId": str,  # ID de l'agent lancé
    "description": str,  # La description de la tâche
    "resolvedModel": str | None,  # Modèle utilisé au moment du passage en arrière-plan
    "modelsUsed": list[str] | None,  # Modèles utilisés avant le passage en arrière-plan, dans l'ordre, répétitions consécutives fusionnées
    "prompt": str,  # Le prompt exécuté par l'agent
    "outputFile": str,  # Chemin du fichier où la sortie de l'agent est écrite
    "canReadOutputFile": bool | None,  # Indique si le fichier de sortie peut être lu directement
}
```

**Sortie (status : `"remote_launched"`) :**

```python theme={null}
{
    "status": "remote_launched",
    "taskId": str,  # ID de la tâche envoyée
    "sessionUrl": str,  # Lien vers la session cloud
    "description": str,  # La description de la tâche
    "prompt": str,  # Le prompt exécuté par l'agent
    "outputFile": str,  # Chemin du fichier où la sortie de l'agent est écrite
}
```

Renvoie le résultat du sous-agent. La sortie est discriminée par le champ `status` : `"completed"` pour les tâches terminées, `"async_launched"` pour les tâches en arrière-plan, et `"remote_launched"` pour les tâches que Claude Code a envoyées vers une session cloud, où `sessionUrl` pointe vers cette session et `taskId` l'identifie. Si Claude Code a [conservé le worktree isolé du sous-agent](https://code.claude.com/docs/en/worktrees#isolate-subagents-with-worktrees), `worktreePath` (variante `completed`) indique où le trouver, et `worktreeBranch` est sa branche lorsque Claude Code a créé le worktree avec git.

Dans la variante `completed`, `resolvedModel` nomme le modèle sur lequel le sous-agent a démarré, qui peut différer du `model` demandé en entrée quand [`availableModels`](https://code.claude.com/docs/en/model-config#restrict-model-selection) ou une autre surcharge s'applique. Ce champ nécessite Claude Code v2.1.174+. Dans la variante `async_launched`, `resolvedModel` nomme le modèle utilisé au moment où l'agent est passé en arrière-plan : un changement de modèle survenu avant ce passage y est donc reflété. Le champ `modelsUsed` des deux variantes liste les modèles utilisés dans l'ordre, répétitions consécutives fusionnées ; il n'est défini que si le modèle a changé en cours d'exécution. `modelsUsed` et le comportement de `resolvedModel` au passage en arrière-plan nécessitent Claude Code v2.1.212+.

Claude Code remplit `usage` et `totalTokens` à partir de la dernière requête API du sous-agent, et non de toute l'exécution. Lorsqu'il est présent, `thinking_tokens` sous `output_tokens_details` dans `usage` est le nombre de tokens de sortie de cette requête qui étaient des tokens de réflexion (thinking). La clé `output_tokens_details` nécessite le SDK Python v0.2.136+ (qui embarque Claude Code v2.1.228). La clé `fallback_credit` nécessite le SDK Python v0.2.162+ (qui embarque Claude Code v2.1.285).

### AskUserQuestion

**Nom de l'outil :** `AskUserQuestion`

Pose des questions de clarification à l'utilisateur pendant l'exécution. Voir [Gérer les approbations et la saisie utilisateur](https://code.claude.com/docs/en/agent-sdk/user-input#handle-clarifying-questions) pour les détails d'utilisation.

**Entrée :**

```python theme={null}
{
    "questions": [  # Questions à poser à l'utilisateur (1 à 4 questions)
        {
            "question": str,  # La question complète à poser
            "header": str,  # Libellé très court affiché comme pastille/étiquette (12 caractères max.)
            "options": [  # Les choix disponibles (2 à 4 options)
                {
                    "label": str,  # Texte affiché pour cette option (1 à 5 mots)
                    "description": str,  # Explication de ce que signifie cette option
                    "preview": str | None,  # Contenu d'aperçu affiché quand l'option a le focus
                }
            ],
            "multiSelect": bool,  # Mettre à true pour autoriser plusieurs sélections
        }
    ],
    "answers": dict[str, str] | None,
    # Réponses de l'utilisateur, remplies par le système de permissions. Les
    # réponses à choix multiples sont une chaîne des libellés sélectionnés
    # joints par des virgules ; une liste de libellés est acceptée en entrée
    # et convertie sous cette forme
    "annotations": dict[str, dict] | None,
    # Annotations de l'utilisateur par question, indexées par le texte de la
    # question. Chaque valeur peut porter "preview" (le contenu d'aperçu de
    # l'option sélectionnée) et "notes" (notes libres sur la sélection)
    "metadata": dict | None,  # Métadonnées d'analytique, comme {"source": "remember"} ; non affichées à l'utilisateur
}
```

**Sortie :**

```python theme={null}
{
    "questions": [  # Les questions qui ont été posées
        {
            "question": str,
            "header": str,
            "options": [{"label": str, "description": str, "preview": str | None}],
            "multiSelect": bool,
        }
    ],
    "answers": dict[str, str],  # Associe le texte de la question à la chaîne de réponse
    # Les réponses à choix multiples sont séparées par des virgules
    "response": str | None,
    # Réponse libre saisie au lieu de répondre aux questions ; quand elle est
    # définie, Claude reçoit "The user responded: ..." à la place de la liste des réponses
    "annotations": dict[str, dict] | None,  # "preview" et "notes" par question, issus des sélections de l'utilisateur
    "afkTimeoutMs": int | None,  # Défini quand la boîte de dialogue s'est résolue automatiquement après ce nombre de millisecondes d'inactivité de l'utilisateur ; absent quand l'utilisateur a répondu
}
```

### Bash

**Nom de l'outil :** `Bash`

Pour ce qui fixe le plafond au premier plan, voir [Délais et limites de sortie](https://code.claude.com/docs/en/tools-reference#timeout-and-output-limits). Pour la limite de durée en arrière-plan, voir [Limite de durée des commandes en arrière-plan](https://code.claude.com/docs/en/tools-reference#time-limit-for-background-commands).

**Entrée :**

```python theme={null}
{
    "command": str,  # La commande à exécuter
    "timeout": int | None,  # Millisecondes. Premier plan : plafonné à 600000 par défaut, les valeurs supérieures sont ramenées au plafond. Avec run_in_background (Claude Code v2.1.285 ou ultérieur) : limite de durée en arrière-plan, 1800000 si omis, plafonnée à 7200000 sauf relèvement
    "description": str | None,  # Description claire et concise (5 à 10 mots)
    "run_in_background": bool | None,  # Mettre à true pour exécuter en arrière-plan
}
```

**Sortie :**

```python theme={null}
{
    "stdout": str,  # Sortie de la commande ; stdout et stderr arrivent fusionnés dans ce flux unique entrelacé
    "stderr": str,  # Avis ajoutés par l'outil lui-même, pas le stderr de la commande
    "interrupted": bool,  # Indique si la commande a été interrompue
    "isImage": bool | None,  # Indique si stdout contient des données d'image
    "backgroundTaskId": str | None,  # ID de la tâche d'arrière-plan si la commande tourne en arrière-plan
}
```

### Monitor

**Nom de l'outil :** `Monitor`

Exécute une source en arrière-plan et transmet chaque événement à Claude pour qu'il puisse réagir sans interrogation périodique (polling) : `command` exécute un script et émet un événement par ligne de stdout, et `ws` ouvre un WebSocket et émet un événement par trame texte. Fournissez exactement l'un des deux, `command` ou `ws`.

Quand Monitor exécute une commande, il suit les mêmes règles de permission que Bash ; une surveillance WebSocket demande une approbation séparée. La source `ws` nécessite Claude Code v2.1.195+. Voir la [référence de l'outil Monitor](https://code.claude.com/docs/en/tools-reference#monitor-tool) pour le comportement et la disponibilité selon le fournisseur.

**Entrée :**

```python theme={null}
{
    "command": str | None,  # Script shell ; chaque ligne de stdout est un événement, la fin du script termine la surveillance
    "ws": dict | None,  # Source WebSocket : {"url": str, "protocols": list[str] | None} ; chaque trame texte est un événement
    "description": str,  # Courte description affichée dans les notifications
    "timeout_ms": int | None,  # Échéance en millisecondes (défaut 300000, max. 3600000 ; l'échéance effective est d'au plus 1800000)
}
```

**Sortie :**

```python theme={null}
{
    "taskId": str,  # ID de la tâche de surveillance en arrière-plan
    "timeoutMs": int,  # L'échéance effective de la surveillance en millisecondes
    "persistent": bool | None,  # False : toute surveillance a une échéance
}
```

### Edit

**Nom de l'outil :** `Edit`

**Entrée :**

```python theme={null}
{
    "file_path": str,  # Chemin absolu du fichier à modifier
    "old_string": str,  # Le texte à remplacer
    "new_string": str,  # Le texte de remplacement
    "replace_all": bool | None,  # Remplacer toutes les occurrences (défaut False)
}
```

**Sortie :**

```python theme={null}
{
    "filePath": str,  # Le fichier modifié
    "oldString": str,  # Le texte qui a été remplacé
    "newString": str,  # Le texte qui l'a remplacé
    "originalFile": str | None,  # Contenu du fichier avant la modification
    "structuredPatch": [  # Blocs de diff (hunks) de la modification
        {
            "oldStart": int,
            "oldLines": int,
            "newStart": int,
            "newLines": int,
            "lines": list[str],
        }
    ],
    "userModified": bool,  # Indique si l'utilisateur a changé la modification proposée avant de l'accepter
    "replaceAll": bool,  # Indique si toutes les occurrences ont été remplacées
    "gitDiff": {  # Résumé facultatif du diff git du fichier
        "filename": str,
        "status": "modified" | "added",
        "additions": int,
        "deletions": int,
        "changes": int,
        "patch": str,
        "repository": str | None,  # owner/repo GitHub lorsque disponible
    } | None,
}
```

### Read

**Nom de l'outil :** `Read`

**Entrée :**

```python theme={null}
{
    "file_path": str,  # Chemin absolu du fichier à lire
    "offset": int | None,  # Numéro de ligne à partir duquel lire
    "limit": int | None,  # Nombre de lignes à lire
}
```

La sortie prend l'une des formes suivantes selon ce que Claude a lu. Vérifiez la clé `type` pour les distinguer.

**Sortie (type : `"text"`) :**

```python theme={null}
{
    "type": "text",
    "file": {
        "filePath": str,  # Le fichier lu
        "content": str,  # Le contenu renvoyé
        "numLines": int,  # Nombre de lignes du contenu renvoyé
        "startLine": int,  # Numéro de la ligne où commence le contenu
        "totalLines": int,  # Nombre total de lignes du fichier
        "truncatedByTokenCap": bool | None,  # Présent et True quand la lecture du fichier entier a dépassé le plafond de tokens et que content est la première page
    },
}
```

**Sortie (type : `"image"`) :**

```python theme={null}
{
    "type": "image",
    "file": {
        "base64": str,  # Données de l'image encodées en base64
        "type": "image/jpeg" | "image/png" | "image/gif" | "image/webp",  # Type MIME de l'image
        "originalSize": int,  # Taille d'origine du fichier en octets
        "dimensions": {  # Informations facultatives de dimensions pour la correspondance des coordonnées
            "originalWidth": int | None,  # Facultatif ; largeur d'origine en pixels
            "originalHeight": int | None,  # Facultatif ; hauteur d'origine en pixels
            "displayWidth": int | None,  # Facultatif ; largeur après redimensionnement
            "displayHeight": int | None,  # Facultatif ; hauteur après redimensionnement
        } | None,
    },
}
```

**Sortie (type : `"notebook"`) :**

```python theme={null}
{
    "type": "notebook",
    "file": {
        "filePath": str,  # Le notebook lu
        "cells": list,  # Cellules du notebook
    },
}
```

**Sortie (type : `"pdf"`) :**

```python theme={null}
{
    "type": "pdf",
    "file": {
        "filePath": str,  # Le PDF lu
        "base64": str,  # Données du PDF encodées en base64
        "originalSize": int,  # Taille du fichier en octets
    },
}
```

**Sortie (type : `"parts"`) :**

```python theme={null}
{
    "type": "parts",
    "file": {
        "filePath": str,  # Le PDF lu
        "originalSize": int,  # Taille du fichier en octets
        "count": int,  # Nombre de pages extraites sous forme d'images
        "outputDir": str,  # Répertoire contenant les images des pages extraites
    },
    "firstPage": int | None,  # Numéro de page facultatif, dans le document, de la première page extraite
}
```

**Sortie (type : `"file_unchanged"`) :**

```python theme={null}
{
    "type": "file_unchanged",  # Le fichier n'a pas changé depuis la dernière lecture par Claude dans cette session, le contenu n'est donc pas répété
    "file": {
        "filePath": str,
    },
    "source": "seeded" | None,  # Présent quand la copie précédente provient d'un fichier CLAUDE.md ou mémoire chargé au démarrage plutôt que d'un appel Read
}
```

### Write

**Nom de l'outil :** `Write`

**Entrée :**

```python theme={null}
{
    "file_path": str,  # Chemin absolu du fichier à écrire
    "content": str,  # Le contenu à écrire dans le fichier
}
```

**Sortie :**

```python theme={null}
{
    "type": "create" | "update",  # Indique si l'écriture a créé un nouveau fichier ou écrasé un fichier existant
    "filePath": str,  # Le fichier écrit
    "content": str,  # Le contenu écrit
    "structuredPatch": [  # Blocs de diff ; vide pour un nouveau fichier, quand rien n'a changé ou quand Claude Code a sauté le diff
        {
            "oldStart": int,
            "oldLines": int,
            "newStart": int,
            "newLines": int,
            "lines": list[str],
        }
    ],
    "originalFile": str | None,  # Contenu précédent ; None pour un nouveau fichier ou quand le contenu précédent était trop volumineux pour être inclus
    "gitDiff": {  # Résumé facultatif du diff git du fichier
        "filename": str,
        "status": "modified" | "added",
        "additions": int,
        "deletions": int,
        "changes": int,
        "patch": str,
        "repository": str | None,  # owner/repo GitHub lorsque disponible
    } | None,
    "userModified": bool | None,  # Facultatif ; indique si l'utilisateur a modifié le contenu proposé avant de l'accepter
}
```

### Glob

**Nom de l'outil :** `Glob`

**Entrée :**

```python theme={null}
{
    "pattern": str,  # Le motif glob auquel faire correspondre les fichiers
    "path": str | None,  # Le répertoire où chercher (cwd par défaut)
}
```

**Sortie :**

```python theme={null}
{
    "durationMs": int,  # Durée de la recherche, en millisecondes
    "numFiles": int,  # Nombre de chemins renvoyés, après une éventuelle troncature
    "filenames": list[str],  # Chemins des fichiers correspondants
    "truncated": bool,  # Indique si les résultats ont été tronqués à la limite de 100 fichiers
    "totalMatches": int | None,  # Nombre total facultatif de fichiers correspondants avant troncature ; borne inférieure quand countIsComplete vaut False
    "countIsComplete": bool | None,  # Facultatif ; indique si totalMatches est exact
}
```

`totalMatches` et `countIsComplete` nécessitent Claude Code v2.1.191+.

### Grep

**Nom de l'outil :** `Grep`

**Entrée :**

```python theme={null}
{
    "pattern": str,  # L'expression régulière
    "path": str | None,  # Fichier ou répertoire où chercher
    "glob": str | None,  # Motif glob pour filtrer les fichiers
    "type": str | None,  # Type de fichier à rechercher
    "output_mode": str | None,  # "content", "files_with_matches" ou "count"
    "-i": bool | None,  # Recherche insensible à la casse
    "-n": bool | None,  # Afficher les numéros de ligne
    "-B": int | None,  # Lignes à afficher avant chaque correspondance
    "-A": int | None,  # Lignes à afficher après chaque correspondance
    "-C": int | None,  # Lignes à afficher avant et après
    "context": int | None,  # Lignes à afficher avant et après ; -C est un alias
    "-o": bool | None,  # N'afficher que les parties correspondantes de chaque ligne
    "head_limit": int | None,  # Limiter la sortie aux N premières lignes/entrées
    "offset": int | None,  # Sauter les N premières lignes/entrées avant d'appliquer head_limit
    "multiline": bool | None,  # Activer le mode multiligne
}
```

**Sortie :**

```python theme={null}
{
    "mode": "content" | "files_with_matches" | "count" | None,  # Le mode de sortie utilisé
    "numFiles": int,  # Nombre de fichiers dans le résultat ; toujours 0 en mode content
    "filenames": list[str],  # Fichiers correspondants en mode files_with_matches ; vide dans les autres modes
    "content": str | None,  # Lignes correspondantes en mode content, ou comptes par fichier en mode count
    "numLines": int | None,  # Nombre de lignes de content, présent en mode content
    "numMatches": int | None,  # Nombre total de correspondances, présent en mode count
    "totalFiles": int | None,  # Total facultatif avant head_limit et offset, en mode files_with_matches
    "totalLines": int | None,  # Total facultatif avant head_limit et offset, en mode content
    "appliedLimit": int | None,  # Présent quand head_limit a tronqué le résultat
    "appliedOffset": int | None,  # Présent quand un offset a été appliqué
}
```

Grep renvoie cette forme de dict dans chaque mode de sortie. Les clés facultatives présentes dépendent de `output_mode`.

`totalFiles` nécessite Claude Code v2.1.208+. `totalLines` nécessite Claude Code v2.1.210+.

### NotebookEdit

**Nom de l'outil :** `NotebookEdit`

**Entrée :**

```python theme={null}
{
    "notebook_path": str,  # Chemin absolu du notebook Jupyter
    "cell_id": str | None,  # L'ID de la cellule à modifier
    "new_source": str,  # Le nouveau source de la cellule
    "cell_type": "code" | "markdown" | None,  # Le type de la cellule
    "edit_mode": "replace" | "insert" | "delete" | None,  # Type d'opération de modification
}
```

**Sortie :**

```python theme={null}
{
    "new_source": str,  # Le source écrit dans la cellule
    "old_source": str | None,  # Source précédent de la cellule, présent pour replace et delete
    "cell_id": str | None,  # ID de la cellule modifiée, lorsque disponible
    "cell_type": "code" | "markdown",  # Le type de cellule
    "language": str,  # Le langage de programmation du notebook
    "edit_mode": str,  # Le mode de modification utilisé
    "error": str | None,  # Message d'erreur quand l'opération a échoué
    "notebook_path": str,  # Le fichier notebook
    "original_file": str,  # Contenu du notebook avant la modification
    "updated_file": str,  # Contenu du notebook après la modification
}
```

### WebFetch

**Nom de l'outil :** `WebFetch`

**Entrée :**

```python theme={null}
{
    "url": str,  # L'URL dont récupérer le contenu
    "prompt": str,  # Le prompt à exécuter sur le contenu récupéré
    "offset": int | None,  # Nombre de caractères à sauter depuis le début de la page. Nécessite le SDK Agent Python 0.2.164 ou ultérieur
}
```

**Sortie :**

```python theme={null}
{
    "bytes": int,  # Taille du contenu récupéré en octets
    "code": int,  # Code de réponse HTTP
    "codeText": str,  # Texte du code de réponse HTTP
    "result": str,  # Résultat obtenu en appliquant le prompt au contenu
    "durationMs": int,  # Durée de récupération et de traitement du contenu, en millisecondes
    "url": str,  # URL récupérée
}
```

### WebSearch

**Nom de l'outil :** `WebSearch`

**Entrée :**

```python theme={null}
{
    "query": str,  # La requête de recherche à utiliser
    "allowed_domains": list[str] | None,  # N'inclure que les résultats de ces domaines
    "blocked_domains": list[str] | None,  # Ne jamais inclure de résultats de ces domaines
}
```

**Sortie :**

```python theme={null}
{
    "query": str,  # La requête de recherche
    "results": list[str | {"tool_use_id": str, "content": list[{"title": str, "url": str}]}],
    "durationSeconds": float,  # Durée de la recherche en secondes
}
```

### TodoWrite

**Nom de l'outil :** `TodoWrite`

> **Note** : les outils suivants ne sont disponibles par défaut que sur Claude Opus 4.7, Claude Sonnet 4.6, Claude Haiku 4.5 et les modèles antérieurs. Sur tout autre modèle, y compris les ID de modèle que Claude Code ne reconnaît pas, ils ne sont pas disponibles sauf activation explicite (opt-in) :
>
> * `TodoWrite`
> * `TaskCreate`
> * `TaskGet`
> * `TaskUpdate`
> * `TaskList`
>
> Là où ces outils sont disponibles, Claude Code fournit les quatre outils Task, ou `TodoWrite` à la place quand vous définissez `CLAUDE_CODE_ENABLE_TASKS=0`.
>
> Cet ensemble par défaut s'applique à partir de Claude Code v2.1.268, que le SDK Agent TypeScript embarque à partir de v0.3.268.
>
> Voir [Disponibilité selon le modèle](https://code.claude.com/docs/en/agent-sdk/todo-tracking#model-availability) pour les activer.

**Entrée :**

```python theme={null}
{
    "todos": [
        {
            "content": str,  # La description de la tâche
            "status": "pending" | "in_progress" | "completed",  # Statut de la tâche
            "activeForm": str,  # Forme active (en cours) de la description
        }
    ]
}
```

**Sortie :**

```python theme={null}
{
    "oldTodos": [  # La liste de tâches avant la mise à jour
        {
            "content": str,
            "status": "pending" | "in_progress" | "completed",
            "activeForm": str,
        }
    ],
    "newTodos": [  # La liste de tâches après la mise à jour
        {
            "content": str,
            "status": "pending" | "in_progress" | "completed",
            "activeForm": str,
        }
    ],
}
```

### TaskCreate

**Nom de l'outil :** `TaskCreate`

**Entrée :**

```python theme={null}
{
    "subject": str,  # Titre court de la tâche
    "description": str,  # Corps détaillé de la tâche
    "activeForm": str | None,  # Libellé au présent affiché pendant que la tâche est en cours
    "metadata": dict | None,  # Métadonnées arbitraires de l'appelant
}
```

**Sortie :**

```python theme={null}
{
    "task": {"id": str, "subject": str},  # Tâche créée, avec l'ID attribué
}
```

### TaskUpdate

**Nom de l'outil :** `TaskUpdate`

**Entrée :**

```python theme={null}
{
    "taskId": str,  # ID de la tâche à modifier
    "status": Literal["pending", "in_progress", "completed", "deleted"] | None,
    "subject": str | None,
    "description": str | None,
    "activeForm": str | None,
    "addBlocks": list[str] | None,  # ID des tâches que cette tâche bloque désormais
    "addBlockedBy": list[str] | None,  # ID des tâches qui bloquent désormais cette tâche
    "owner": str | None,
    "metadata": dict | None,
}
```

**Sortie :**

```python theme={null}
{
    "success": bool,
    "taskId": str,
    "updatedFields": list[str],  # Noms des champs modifiés
    "error": str | None,
    "statusChange": {"from": str, "to": str} | None,
}
```

### TaskGet

**Nom de l'outil :** `TaskGet`

**Entrée :**

```python theme={null}
{
    "taskId": str,  # ID de la tâche à lire
}
```

**Sortie :**

```python theme={null}
{
    "task": {
        "id": str,
        "subject": str,
        "description": str,
        "status": Literal["pending", "in_progress", "completed"],
        "blocks": list[str],
        "blockedBy": list[str],
    } | None,  # None quand l'ID est introuvable
}
```

### TaskList

**Nom de l'outil :** `TaskList`

**Entrée :**

```python theme={null}
{}
```

**Sortie :**

```python theme={null}
{
    "tasks": [
        {
            "id": str,
            "subject": str,
            "status": Literal["pending", "in_progress", "completed"],
            "owner": str | None,
            "blockedBy": list[str],
        }
    ],
}
```

### TaskOutput

Supprimé dans Claude Code v2.1.277. Récupérait auparavant la sortie d'une tâche d'arrière-plan en cours ou terminée, avec `BashOutput` accepté comme alias ; Claude lit désormais le fichier de sortie d'une tâche d'arrière-plan avec `Read`.

Une entrée `disallowed_tools` ou une règle de refus (deny) qui nomme encore l'un de ces deux noms est ignorée sans avertissement.

### TaskStop

**Nom de l'outil :** `TaskStop`. Les anciens noms `KillShell` et `KillBash` sont toujours acceptés comme alias.

**Entrée :**

```python theme={null}
{
    "task_id": str | None,  # L'ID de la tâche d'arrière-plan à arrêter
    "shell_id": str | None,  # Obsolète : utilisez task_id à la place
}
```

**Sortie :**

```python theme={null}
{
    "message": str,  # Message d'état sur l'opération
    "task_id": str,  # L'ID de la tâche arrêtée
    "task_type": str,  # Le type de la tâche arrêtée
    "command": str | None,  # La commande ou la description de la tâche arrêtée
}
```

### ExitPlanMode

**Nom de l'outil :** `ExitPlanMode`

**Entrée :**

```python theme={null}
{
    "plan": str  # Le plan à soumettre à l'utilisateur pour approbation
}
```

**Sortie :**

```python theme={null}
{
    "plan": str | None,  # Le plan présenté à l'utilisateur
    "isAgent": bool,  # True quand c'est un sous-agent qui a appelé l'outil
    "filePath": str | None,  # Présent quand le plan a été enregistré dans un fichier
    "hasTaskTool": bool | None,  # Facultatif ; indique si l'outil Agent est disponible dans le contexte courant
    "planWasEdited": bool | None,  # Présent et True quand l'utilisateur a modifié le plan avant de l'approuver
    "awaitingLeaderApproval": bool | None,  # Présent et True quand un coéquipier a envoyé le plan au chef d'équipe (team lead) pour approbation
    "requestId": str | None,  # ID facultatif de cette demande d'approbation
}
```

### ListMcpResources

**Nom de l'outil :** `ListMcpResourcesTool`

**Entrée :**

```python theme={null}
{
    "server": str | None  # Nom de serveur facultatif pour filtrer les ressources
}
```

Le résultat est une liste et non un dict : `tool_use_result` contient donc une `list` pour cet outil.

**Sortie :**

```python theme={null}
[  # Une entrée par ressource
    {
        "uri": str,  # URI de la ressource
        "name": str,  # Nom de la ressource
        "mimeType": str | None,  # Type MIME facultatif
        "description": str | None,  # Description facultative
        "server": str,  # Serveur qui fournit cette ressource
    }
]
```

### ReadMcpResource

**Nom de l'outil :** `ReadMcpResourceTool`

**Entrée :**

```python theme={null}
{
    "server": str,  # Le nom du serveur MCP
    "uri": str,  # L'URI de la ressource à lire
}
```

**Sortie :**

```python theme={null}
{
    "contents": [
        {
            "uri": str,  # URI de la ressource
            "mimeType": str | None,  # Type MIME facultatif
            "text": str | None,  # Contenu texte, ou une note sur le contenu binaire
            "blobSavedTo": str | None,  # Présent quand Claude Code a enregistré le contenu binaire sur disque ; chemin du fichier enregistré
        }
    ],
    "error": str | None,  # Présent quand le serveur n'a pas pu lire la ressource
}
```

## Construire une interface de conversation continue

L'exemple suivant garde un même `ClaudeSDKClient` connecté d'un tour à l'autre, de sorte que Claude se souvient des messages précédents. Tapez `new` pour se déconnecter puis se reconnecter afin d'obtenir une session neuve, ou `exit` pour terminer la conversation.

```python theme={null}
from claude_agent_sdk import (
    ClaudeSDKClient,
    ClaudeAgentOptions,
    AssistantMessage,
    TextBlock,
)
import asyncio


class ConversationSession:
    """Maintient une seule session de conversation avec Claude."""

    def __init__(self, options: ClaudeAgentOptions | None = None):
        self.client = ClaudeSDKClient(options)
        self.turn_count = 0

    async def start(self):
        await self.client.connect()
        print("Starting conversation session. Claude will remember context.")
        print(
            "Commands: 'exit' to quit, 'interrupt' to stop current task, 'new' for new session"
        )

        while True:
            user_input = input(f"\n[Turn {self.turn_count + 1}] You: ")

            if user_input.lower() == "exit":
                break
            elif user_input.lower() == "interrupt":
                await self.client.interrupt()
                print("Task interrupted!")
                continue
            elif user_input.lower() == "new":
                # Se déconnecter puis se reconnecter pour une session neuve
                await self.client.disconnect()
                await self.client.connect()
                self.turn_count = 0
                print("Started new conversation session (previous context cleared)")
                continue

            # Envoyer le message - la session conserve tous les messages précédents
            await self.client.query(user_input)
            self.turn_count += 1

            # Traiter la réponse
            print(f"[Turn {self.turn_count}] Claude: ", end="")
            async for message in self.client.receive_response():
                if isinstance(message, AssistantMessage):
                    for block in message.content:
                        if isinstance(block, TextBlock):
                            print(block.text, end="")
            print()  # Saut de ligne après la réponse

        await self.client.disconnect()
        print(f"Conversation ended after {self.turn_count} turns.")


async def main():
    options = ClaudeAgentOptions(
        allowed_tools=["Read", "Write", "Bash"], permission_mode="acceptEdits"
    )
    session = ConversationSession(options)
    await session.start()


# Exemple de conversation :
# Tour 1 - Vous : "Create a file called hello.py"
# Tour 1 - Claude : "I'll create a hello.py file for you..."
# Tour 2 - Vous : "What's in that file?"
# Tour 2 - Claude : "The hello.py file I just created contains..." (il se souvient !)
# Tour 3 - Vous : "Add a main function to it"
# Tour 3 - Claude : "I'll add a main function to hello.py..." (il sait de quel fichier il s'agit !)

asyncio.run(main())
```

## Gestion des erreurs

L'exemple suivant entoure un appel à `query()` de gestionnaires pour quatre des [types d'erreur](https://code.claude.com/docs/en/agent-sdk/python#error-types) que le SDK lève.

Cet exemple intercepte [`ResultError`](https://code.claude.com/docs/en/agent-sdk/python#resulterror), qui nécessite le SDK Agent Python 0.2.140+.

```python theme={null}
import asyncio

from claude_agent_sdk import (
    query,
    CLINotFoundError,
    ProcessError,
    ResultError,
    CLIJSONDecodeError,
)


async def main():
    try:
        async for message in query(prompt="Hello"):
            print(message)
    except CLINotFoundError:
        print(
            "Claude Code CLI not found. Try reinstalling: pip install --force-reinstall claude-agent-sdk"
        )
    # Interceptez ResultError avant ProcessError, dont elle est une sous-classe.
    # Son message porte le texte de l'erreur. Une requête finale en échec, comme
    # une erreur d'API, arrive avec le subtype "success" : testez donc d'abord terminal_reason.
    except ResultError as e:
        if e.terminal_reason == "api_error":
            print(f"API request failed: {e}")
        else:
            print(f"Query ended with an error result ({e.terminal_reason or e.subtype}): {e}")
    except ProcessError as e:
        print(f"Process failed with exit code: {e.exit_code}")
    except CLIJSONDecodeError as e:
        print(f"Failed to parse response: {e}")


asyncio.run(main())
```

Points clés :

* `CLINotFoundError` : la CLI Claude Code est introuvable ; le message suggère `pip install --force-reinstall claude-agent-sdk`.
* `ResultError` : à intercepter **avant** `ProcessError` (dont elle hérite) ; son message contient le texte de l'erreur. Une requête finale en échec (par exemple une erreur d'API) arrive avec `subtype` `"success"`, d'où l'intérêt de tester d'abord `terminal_reason` (ex. `"api_error"`), puis `subtype` à défaut.
* `ProcessError` : expose `exit_code`, le code de sortie du processus.
* `CLIJSONDecodeError` : échec d'analyse de la réponse.

## Configuration du sandbox

### `SandboxSettings`

Configuration du comportement du sandbox. Utilisez-la pour activer le confinement (sandboxing) des commandes et configurer les restrictions réseau par programme.

```python theme={null}
class SandboxSettings(TypedDict, total=False):
    enabled: bool
    autoAllowBashIfSandboxed: bool
    excludedCommands: list[str]
    allowUnsandboxedCommands: bool
    network: SandboxNetworkConfig
    ignoreViolations: SandboxIgnoreViolations
    enableWeakerNestedSandbox: bool
```

| Propriété | Type | Défaut | Description |
| :- | :- | :- | :- |
| `enabled` | `bool` | `False` | Active le mode sandbox pour l'exécution des commandes |
| `autoAllowBashIfSandboxed` | `bool` | `True` | Approuve automatiquement les commandes bash quand le sandbox est activé |
| `excludedCommands` | `list[str]` | `[]` | Commandes qui contournent les restrictions du sandbox, comme `["docker *"]`. Elles s'exécutent automatiquement hors sandbox sans intervention du modèle ; [`sandbox.excludedCommands`](https://code.claude.com/docs/en/settings-reference#sandbox-excludedcommands) précise quand une entrée s'applique |
| `allowUnsandboxedCommands` | `bool` | `True` | Autorise le modèle à demander l'exécution de commandes hors sandbox. Avec `True`, le modèle peut définir `dangerouslyDisableSandbox` dans l'entrée de l'outil, ce qui renvoie au [système de permissions](#repli-sur-les-permissions-pour-les-commandes-hors-sandbox) |
| `network` | [`SandboxNetworkConfig`](#sandboxnetworkconfig) | `None` | Configuration du sandbox propre au réseau |
| `ignoreViolations` | [`SandboxIgnoreViolations`](#sandboxignoreviolations) | `None` | Configure les violations du sandbox à ignorer |
| `enableWeakerNestedSandbox` | `bool` | `False` | Active un sandbox imbriqué plus faible, pour la compatibilité |

> **Note** : le sandbox dépend de la prise en charge par la plateforme et, sous Linux, d'outils comme `bubblewrap` et `socat`. Par défaut, quand `enabled` vaut `True` mais que le sandbox ne peut pas démarrer, les commandes s'exécutent hors sandbox avec un avertissement sur stderr. Ce défaut diffère du SDK TypeScript, où `failIfUnavailable` vaut `true` par défaut.
>
> Définissez `"failIfUnavailable": True` dans vos réglages de sandbox pour arrêter à la place. La clé n'est pas encore déclarée dans `SandboxSettings`, mais le SDK la transmet à Claude Code, qui la respecte. `query()` renvoie alors un `ResultMessage` avec `subtype="error_during_execution"` et la raison dans `errors`. Comme il s'agit d'un appel `query()` ponctuel (single-shot), le SDK lève une exception après avoir produit ce résultat d'erreur : entourez donc la boucle d'un bloc try pour continuer au-delà. Voir [Traiter le résultat](https://code.claude.com/docs/en/agent-sdk/agent-loop#handle-the-result) pour le contrat d'erreur.

#### Exemple d'utilisation

```python theme={null}
import asyncio

from claude_agent_sdk import query, ClaudeAgentOptions

sandbox_settings = {
    "enabled": True,
    "autoAllowBashIfSandboxed": True,
    "failIfUnavailable": True,
    "network": {"allowLocalBinding": True},
}


async def main():
    try:
        async for message in query(
            prompt="Build and test my project",
            options=ClaudeAgentOptions(sandbox=sandbox_settings),
        ):
            print(message)
    except Exception as error:
        # Un query() ponctuel lève une exception après avoir produit un résultat d'erreur,
        # par exemple quand failIfUnavailable est défini et que le sandbox ne peut pas démarrer.
        print(f"Session ended with an error: {error}")


asyncio.run(main())
```

> **Avertissement** : **Sécurité des sockets Unix** — l'option `allowUnixSockets` peut donner accès à des services système qui sortent du sandbox. Par exemple, autoriser `/var/run/docker.sock` revient en pratique à accorder un accès complet au système hôte via l'API Docker, en contournant l'isolation du sandbox. N'autorisez que les sockets Unix strictement nécessaires et comprenez les implications de sécurité de chacun.

### `SandboxNetworkConfig`

Configuration réseau du mode sandbox. Ces réglages s'appliquent aux commandes Bash confinées quand `enabled` vaut `True` dans le [`SandboxSettings`](#sandboxsettings) parent. Ils ne restreignent pas l'outil WebFetch, qui utilise à la place des [règles de permission](https://code.claude.com/docs/en/permissions#webfetch).

```python theme={null}
class SandboxNetworkConfig(TypedDict, total=False):
    allowedDomains: list[str]
    deniedDomains: list[str]
    allowManagedDomainsOnly: bool
    allowUnixSockets: list[str]
    allowAllUnixSockets: bool
    allowLocalBinding: bool
    allowMachLookup: list[str]
    httpProxyPort: int
    socksProxyPort: int
```

| Propriété | Type | Défaut | Description |
| :- | :- | :- | :- |
| `allowedDomains` | `list[str]` | `[]` | Noms de domaine auxquels les processus confinés peuvent accéder |
| `deniedDomains` | `list[str]` | `[]` | Noms de domaine auxquels les processus confinés ne peuvent pas accéder. Prioritaire sur `allowedDomains` |
| `allowManagedDomainsOnly` | `bool` | `False` | Réglages gérés uniquement : défini dans les réglages gérés (managed settings), ignore les `allowedDomains` et les règles d'autorisation `WebFetch(domain:...)` provenant de sources de réglages non gérées. Sans effet quand il est défini via les options du SDK |
| `allowUnixSockets` | `list[str]` | `[]` | macOS uniquement : chemins de sockets Unix accessibles aux processus, comme le socket Docker. Ignoré sous Linux |
| `allowAllUnixSockets` | `bool` | `False` | Autorise l'accès à tous les sockets Unix |
| `allowLocalBinding` | `bool` | `False` | Autorise les processus à écouter sur des ports locaux (par exemple pour des serveurs de développement) |
| `allowMachLookup` | `list[str]` | `[]` | macOS uniquement : noms de services XPC/Mach à autoriser. Accepte un joker (wildcard) final |
| `httpProxyPort` | `int` | `None` | Port du proxy HTTP pour les requêtes réseau |
| `socksProxyPort` | `int` | `None` | Port du proxy SOCKS pour les requêtes réseau |

> **Note** : le proxy intégré du sandbox applique la liste d'autorisation réseau d'après le nom d'hôte demandé et ne termine ni n'inspecte le trafic TLS ; des techniques comme le [domain fronting](https://en.wikipedia.org/wiki/Domain_fronting) peuvent donc potentiellement le contourner. Voir [Limites de sécurité du sandboxing](https://code.claude.com/docs/en/sandboxing#security-limitations) pour les détails et [Déploiement sécurisé](https://code.claude.com/docs/en/agent-sdk/secure-deployment#traffic-forwarding) pour configurer un proxy qui termine le TLS.

### `SandboxIgnoreViolations`

Configuration pour ignorer certaines violations du sandbox.

```python theme={null}
class SandboxIgnoreViolations(TypedDict, total=False):
    file: list[str]
    network: list[str]
```

| Propriété | Type | Défaut | Description |
| :- | :- | :- | :- |
| `file` | `list[str]` | `[]` | Motifs de chemins de fichiers dont les violations sont ignorées |
| `network` | `list[str]` | `[]` | Motifs réseau dont les violations sont ignorées |

### Repli sur les permissions pour les commandes hors sandbox

Quand `allowUnsandboxedCommands` est activé, le modèle peut demander à exécuter des commandes hors du sandbox en définissant `dangerouslyDisableSandbox: True` dans l'entrée de l'outil. Ces demandes retombent sur le système de permissions existant : votre gestionnaire `can_use_tool` est invoqué, ce qui vous permet d'implémenter votre propre logique d'autorisation.

Vos entrées `excludedCommands`, elles, sortent un appel du sandbox sans aucune intervention du modèle ; [`sandbox.excludedCommands`](https://code.claude.com/docs/en/settings-reference#sandbox-excludedcommands) précise quand une entrée s'applique.

L'exemple suivant journalise chaque demande hors sandbox et la refuse, sauf si votre propre logique d'autorisation l'accepte :

```python theme={null}
import asyncio
from claude_agent_sdk import (
    query,
    ClaudeAgentOptions,
    HookMatcher,
    PermissionResultAllow,
    PermissionResultDeny,
    ToolPermissionContext,
)


def is_command_authorized(command: str | None) -> bool:
    # Remplacez par votre propre logique d'autorisation
    return False



async def can_use_tool(
    tool: str, input: dict, context: ToolPermissionContext
) -> PermissionResultAllow | PermissionResultDeny:
    # Vérifier si le modèle demande à contourner le sandbox
    if tool == "Bash" and input.get("dangerouslyDisableSandbox"):
        # Le modèle demande à exécuter cette commande hors du sandbox
        print(f"Unsandboxed command requested: {input.get('command')}")

        if is_command_authorized(input.get("command")):
            return PermissionResultAllow()
        return PermissionResultDeny(
            message="Command not authorized for unsandboxed execution"
        )
    return PermissionResultAllow()


# Requis : un hook factice garde le flux ouvert pour can_use_tool
async def dummy_hook(input_data, tool_use_id, context):
    return {"continue_": True}


async def prompt_stream():
    yield {
        "type": "user",
        "message": {"role": "user", "content": "Deploy my application"},
    }


async def main():
    async for message in query(
        prompt=prompt_stream(),
        options=ClaudeAgentOptions(
            sandbox={
                "enabled": True,
                "allowUnsandboxedCommands": True,  # Le modèle peut demander une exécution hors sandbox
            },
            permission_mode="default",
            can_use_tool=can_use_tool,
            hooks={"PreToolUse": [HookMatcher(matcher=None, hooks=[dummy_hook])]},
        ),
    ):
        print(message)


asyncio.run(main())
```

> **Avertissement** : les commandes exécutées avec `dangerouslyDisableSandbox: True` ont un accès complet au système. Assurez-vous que votre gestionnaire `can_use_tool` valide soigneusement ces demandes.
>
> Si `permission_mode` vaut `bypassPermissions` et que `allowUnsandboxedCommands` est activé, le modèle peut exécuter de façon autonome des commandes hors du sandbox sans demande d'approbation, à l'exception des [actions qu'aucun mode n'approuve automatiquement](https://code.claude.com/docs/en/permission-modes#actions-no-mode-auto-approves). Cette combinaison permet en pratique au modèle de s'échapper silencieusement de l'isolation du sandbox.

## Voir aussi

* [Vue d'ensemble du SDK](https://code.claude.com/docs/en/agent-sdk/overview) — concepts généraux du SDK
* [Référence du SDK TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript) — documentation du SDK TypeScript
* [Outils personnalisés](https://code.claude.com/docs/en/agent-sdk/custom-tools) — définir des outils MCP en processus (in-process) que Claude peut appeler
* [Référence de la CLI](https://code.claude.com/docs/en/cli-reference) — interface en ligne de commande
* [Workflows courants](https://code.claude.com/docs/en/common-workflows) — guides pas à pas
