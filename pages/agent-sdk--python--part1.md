# Référence de l'Agent SDK — Python — partie 1/2

> Source : https://code.claude.com/docs/en/agent-sdk/python · [← Index](../31-index-documentation-officielle.md)

Référence complète de l'API du SDK Agent pour Python, avec toutes les fonctions, tous les types et toutes les classes. Les ancres `#…` renvoient aux autres sections de la même page officielle.

## Installation

Installez le paquet dans un environnement virtuel. Sur les installations récentes de Python sous Debian, Ubuntu et Homebrew, lancer `pip install` contre le Python système échoue avec `error: externally-managed-environment`.

```bash theme={null}
python3 -m venv .venv
source .venv/bin/activate
pip install claude-agent-sdk
```

Pour uv, Windows PowerShell et la configuration de la clé API, voir [Setup dans le quickstart de l'Agent SDK](https://code.claude.com/docs/en/agent-sdk/quickstart#setup).

## Choisir entre `query()` et `ClaudeSDKClient`

Le SDK Python offre deux manières d'interagir avec Claude Code :

| Fonctionnalité | `query()` | `ClaudeSDKClient` |
| :- | :- | :- |
| **Session** | Crée une nouvelle session par défaut | Réutilise la même session |
| **Conversation** | Échange unique | Plusieurs échanges dans le même contexte |
| **Connexion** | Gérée automatiquement | Contrôle manuel |
| **Entrée en streaming** | ✅ Prise en charge | ✅ Prise en charge |
| **Interruptions** | ❌ Non prise en charge | ✅ Prise en charge |
| **Hooks** | ✅ Pris en charge | ✅ Pris en charge |
| **Outils personnalisés** | ✅ Pris en charge | ✅ Pris en charge |
| **Poursuivre la conversation** | Manuel via `continue_conversation` ou `resume` | ✅ Automatique |
| **Cas d'usage** | Tâches ponctuelles | Conversations continues |

Utilisez `ClaudeSDKClient` pour les applications interactives comme les interfaces de chat, ou lorsque l'action suivante dépend de la réponse de Claude.

## Fonctions (Functions)

> **Note** : les blocs de signature et les fragments `async for` / `async with` isolés de cette page sont illustratifs. Pour les exécuter, enveloppez le corps dans `async def main(): ...` et appelez `asyncio.run(main())`.

### `query()`

Crée par défaut une nouvelle session pour chaque interaction avec Claude Code. Renvoie un itérateur asynchrone qui produit les messages au fur et à mesure de leur arrivée. Chaque appel à `query()` repart de zéro, sans mémoire des interactions précédentes, sauf si vous passez `continue_conversation=True` ou `resume` dans [`ClaudeAgentOptions`](#claudeagentoptions). Voir [Sessions](https://code.claude.com/docs/en/agent-sdk/sessions).

```python theme={null}
async def query(
    *,
    prompt: str | AsyncIterable[dict[str, Any]],
    options: ClaudeAgentOptions | None = None,
    transport: Transport | None = None
) -> AsyncIterator[Message]
```

#### Paramètres

| Paramètre | Type | Description |
| :- | :- | :- |
| `prompt` | `str \| AsyncIterable[dict]` | Le prompt d'entrée, sous forme de chaîne ou d'itérable asynchrone pour le mode streaming |
| `options` | `ClaudeAgentOptions \| None` | Objet de configuration facultatif (vaut `ClaudeAgentOptions()` par défaut si None) |
| `transport` | `Transport \| None` | Transport personnalisé facultatif pour communiquer avec le processus CLI |

#### Valeur de retour

Renvoie un `AsyncIterator[Message]` qui produit les messages de la conversation.

#### Exemple — Avec des options

```python theme={null}
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions


async def main():
    options = ClaudeAgentOptions(
        system_prompt="You are an expert Python developer",
        permission_mode="acceptEdits",
    )

    async for message in query(prompt="Create a Python web server", options=options):
        print(message)


asyncio.run(main())
```

### `tool()`

Décorateur pour définir des outils MCP avec sûreté de typage.

```python theme={null}
def tool(
    name: str,
    description: str,
    input_schema: type | dict[str, Any],
    annotations: ToolAnnotations | None = None
) -> Callable[[Callable[[Any], Awaitable[dict[str, Any]]]], SdkMcpTool[Any]]
```

#### Paramètres

| Paramètre | Type | Description |
| :- | :- | :- |
| `name` | `str` | Identifiant unique de l'outil |
| `description` | `str` | Description lisible de ce que fait l'outil |
| `input_schema` | `type \| dict[str, Any]` | Schéma définissant les paramètres d'entrée de l'outil. Voir [Options de schéma d'entrée](#input-schema-options) |
| `annotations` | [`ToolAnnotations`](#toolannotations)` \| None` | Annotations d'outil MCP facultatives, fournissant des indications de comportement aux clients |

#### Options de schéma d'entrée (Input schema options)

1. **Correspondance de types simple** (recommandée) :

   ```python theme={null}
   {"text": str, "count": int, "enabled": bool}
   ```

2. **Format JSON Schema** (pour une validation complexe) :
   ```python theme={null}
   {
       "type": "object",
       "properties": {
           "text": {"type": "string"},
           "count": {"type": "integer", "minimum": 0},
       },
       "required": ["text"],
   }
   ```

3. **Classe TypedDict** : un schéma typé dont les clés `NotRequired` sont exclues de `required`.

   * **Python 3.11 et ultérieur** : importez `TypedDict` et `NotRequired` depuis `typing`.
   * **Python 3.10** : `typing` n'a pas de `NotRequired`. Importez `TypedDict` et `NotRequired` depuis `typing_extensions`, que le SDK installe sous Python 3.10.

   ```python theme={null}
   from typing import Annotated, Any, NotRequired, TypedDict
   from claude_agent_sdk import tool


   class ForecastArgs(TypedDict):
       latitude: Annotated[float, "Latitude coordinate"]
       hours: NotRequired[Annotated[int, "How many hours of forecast to return"]]


   @tool("get_forecast", "Get the hourly forecast for a location", ForecastArgs)
   async def get_forecast(args: dict[str, Any]) -> dict[str, Any]:
       hours = args.get("hours", 12)
       return {"content": [{"type": "text", "text": f"{hours}-hour forecast for {args['latitude']}"}]}
   ```

Dans les formes « correspondance simple » et TypedDict, enveloppez un type dans `Annotated[type, "description"]` pour définir la description du champ.

#### Valeur de retour

Une fonction décoratrice qui enveloppe l'implémentation de l'outil et renvoie une instance de `SdkMcpTool`.

#### Exemple

```python theme={null}
from claude_agent_sdk import tool
from typing import Any


@tool("greet", "Greet a user", {"name": str})
async def greet(args: dict[str, Any]) -> dict[str, Any]:
    return {"content": [{"type": "text", "text": f"Hello, {args['name']}!"}]}
```

#### `ToolAnnotations`

Indications de comportement d'un outil, passées comme argument `annotations` de [`tool()`](#tool). `ToolAnnotations` étend le `mcp.types.ToolAnnotations` du SDK MCP avec un champ `maxResultSizeChars`, et chaque indication peut s'écrire en camelCase ou en snake\_case : `ToolAnnotations(readOnlyHint=True)` et `ToolAnnotations(read_only_hint=True)` sont équivalents. Pour relire une indication sur l'objet, utilisez l'orthographe déclarée par votre paquet `mcp` installé : `.readOnlyHint` sur `mcp` 1.x et `.read_only_hint` sur 2.x, tandis que `.maxResultSizeChars` fonctionne sur les deux. Vous pouvez aussi passer un simple `mcp.types.ToolAnnotations` partout où le SDK accepte des annotations.

Les noms snake\_case et le champ typé `maxResultSizeChars` exigent le SDK Agent Python 0.2.140 ou ultérieur. Les versions 0.1.31 à 0.2.139 réexportent `mcp.types.ToolAnnotations` tel quel. Sur les versions 0.1.55 à 0.2.139, vous pouvez quand même passer `maxResultSizeChars` comme argument nommé : la classe MCP accepte les champs supplémentaires et le SDK transmet la valeur à Claude Code.

Tous les champs sont facultatifs. Les clients ne doivent pas se fier à ces indications pour des décisions de sécurité.

| Champ | Type | Défaut | Description |
| :- | :- | :- | :- |
| `title` | `str \| None` | `None` | Titre lisible de l'outil |
| `readOnlyHint` | `bool \| None` | `False` | Si `True`, l'outil ne modifie pas son environnement |
| `destructiveHint` | `bool \| None` | `True` | Si `True`, l'outil peut effectuer des modifications destructrices (n'a de sens que si `readOnlyHint` vaut `False`) |
| `idempotentHint` | `bool \| None` | `False` | Si `True`, des appels répétés avec les mêmes arguments n'ont pas d'effet supplémentaire (n'a de sens que si `readOnlyHint` vaut `False`) |
| `openWorldHint` | `bool \| None` | `True` | Si `True`, l'outil interagit avec des entités externes (par exemple une recherche web). Si `False`, le domaine de l'outil est fermé (par exemple un outil de mémoire) |
| `maxResultSizeChars` | `int \| None` | `None` | Nombre de caractères jusqu'auquel Claude Code garde le résultat texte de cet outil en ligne dans la conversation au lieu de l'enregistrer dans un fichier, jusqu'à 500 000. Les résultats contenant des images ne sont pas concernés. C'est un réglage Claude Code plutôt qu'une indication MCP : le SDK l'envoie dans le `_meta` de l'outil sous `anthropic/maxResultSizeChars`. Voir [Relever la limite pour un outil donné](https://code.claude.com/docs/en/mcp#raise-the-limit-for-a-specific-tool) |

```python theme={null}
from claude_agent_sdk import tool, ToolAnnotations
from typing import Any


@tool(
    "search",
    "Search the web",
    {"query": str},
    annotations=ToolAnnotations(readOnlyHint=True, openWorldHint=True),
)
async def search(args: dict[str, Any]) -> dict[str, Any]:
    return {"content": [{"type": "text", "text": f"Results for: {args['query']}"}]}
```

### `create_sdk_mcp_server()`

Crée un serveur MCP en processus (in-process) qui s'exécute dans votre application Python.

```python theme={null}
def create_sdk_mcp_server(
    name: str,
    version: str = "1.0.0",
    tools: list[SdkMcpTool[Any]] | None = None
) -> McpSdkServerConfig
```

#### Paramètres

| Paramètre | Type | Défaut | Description |
| :- | :- | :- | :- |
| `name` | `str` | - | Identifiant unique du serveur |
| `version` | `str` | `"1.0.0"` | Chaîne de version du serveur |
| `tools` | `list[SdkMcpTool[Any]] \| None` | `None` | Liste des fonctions-outils créées avec le décorateur `@tool` |

#### Valeur de retour

Renvoie un objet `McpSdkServerConfig` qui peut être passé à `ClaudeAgentOptions.mcp_servers`.

#### Exemple

```python theme={null}
from claude_agent_sdk import tool, create_sdk_mcp_server, ClaudeAgentOptions


@tool("add", "Add two numbers", {"a": float, "b": float})
async def add(args):
    return {"content": [{"type": "text", "text": f"Sum: {args['a'] + args['b']}"}]}


@tool("multiply", "Multiply two numbers", {"a": float, "b": float})
async def multiply(args):
    return {"content": [{"type": "text", "text": f"Product: {args['a'] * args['b']}"}]}


calculator = create_sdk_mcp_server(
    name="calculator",
    version="2.0.0",
    tools=[add, multiply],  # Passez les fonctions décorées
)

# Utilisation avec Claude
options = ClaudeAgentOptions(
    mcp_servers={"calc": calculator},
    allowed_tools=["mcp__calc__add", "mcp__calc__multiply"],
)
```

### `list_sessions()`

Liste les sessions passées avec leurs métadonnées. Filtre par répertoire de projet ou liste les sessions de tous les projets. Synchrone ; renvoie immédiatement.

```python theme={null}
def list_sessions(
    directory: str | None = None,
    limit: int | None = None,
    offset: int = 0,
    include_worktrees: bool = True
) -> list[SDKSessionInfo]
```

#### Paramètres

| Paramètre | Type | Défaut | Description |
| :- | :- | :- | :- |
| `directory` | `str \| None` | `None` | Répertoire dont lister les sessions. S'il est omis, renvoie les sessions de tous les projets |
| `limit` | `int \| None` | `None` | Nombre maximal de sessions à renvoyer |
| `offset` | `int` | `0` | Nombre de sessions à sauter depuis le début des résultats triés. À utiliser avec `limit` pour paginer |
| `include_worktrees` | `bool` | `True` | Quand `directory` est dans un dépôt git, inclut les sessions de tous les chemins de worktree |

#### Type de retour : `SDKSessionInfo`

| Propriété | Type | Description |
| :- | :- | :- |
| `session_id` | `str` | Identifiant unique de la session |
| `summary` | `str` | Titre affiché : titre personnalisé, prompt le plus récent, résumé généré automatiquement ou premier prompt |
| `last_modified` | `int` | Date de dernière modification, en millisecondes depuis l'epoch |
| `file_size` | `int \| None` | Taille du fichier de session en octets (`None` pour les backends de stockage distants) |
| `custom_title` | `str \| None` | Titre de la session : le titre défini par l'utilisateur, ou le titre généré automatiquement s'il n'y en a pas |
| `first_prompt` | `str \| None` | Premier prompt utilisateur significatif de la session |
| `git_branch` | `str \| None` | Branche git à la fin de la session |
| `cwd` | `str \| None` | Répertoire de travail de la session |
| `tag` | `str \| None` | Étiquette de session définie par l'utilisateur (voir [`tag_session()`](#tag_session)) |
| `created_at` | `int \| None` | Date de création de la session, en millisecondes depuis l'epoch |

#### Exemple

Affiche les 10 sessions les plus récentes d'un projet. Les résultats sont triés par `last_modified` décroissant, donc le premier élément est le plus récent. Omettez `directory` pour chercher dans tous les projets.

```python theme={null}
from claude_agent_sdk import list_sessions

for session in list_sessions(directory="/path/to/project", limit=10):
    print(f"{session.summary} ({session.session_id})")
```

### `get_session_messages()`

Récupère les messages d'une session passée. Synchrone ; renvoie immédiatement.

```python theme={null}
def get_session_messages(
    session_id: str,
    directory: str | None = None,
    limit: int | None = None,
    offset: int = 0
) -> list[SessionMessage]
```

#### Paramètres

| Paramètre | Type | Défaut | Description |
| :- | :- | :- | :- |
| `session_id` | `str` | requis | L'ID de la session dont récupérer les messages |
| `directory` | `str \| None` | `None` | Répertoire de projet où chercher. S'il est omis, cherche dans tous les projets |
| `limit` | `int \| None` | `None` | Nombre maximal de messages à renvoyer |
| `offset` | `int` | `0` | Nombre de messages à sauter depuis le début |

#### Type de retour : `SessionMessage`

| Propriété | Type | Description |
| :- | :- | :- |
| `type` | `Literal["user", "assistant"]` | Rôle du message |
| `uuid` | `str` | Identifiant unique du message |
| `session_id` | `str` | Identifiant de la session |
| `message` | `Any` | Contenu brut du message |
| `parent_tool_use_id` | `str \| None` | Pour les messages de sous-agent, l'id du bloc tool-use `Agent` qui l'a lancé. `None` pour les messages de la session principale et les sessions plus anciennes |
| `parent_agent_id` | `str \| None` | Pour les messages d'un [sous-agent imbriqué](https://code.claude.com/docs/en/sub-agents#let-subagents-spawn-their-own-subagents), l'id d'agent du sous-agent parent. `None` pour les messages de la session principale, ceux des sous-agents de premier niveau et les sessions plus anciennes. Exige le SDK Agent Python 0.2.140 ou ultérieur |

#### Exemple

```python theme={null}
from claude_agent_sdk import list_sessions, get_session_messages

sessions = list_sessions(limit=1)
if sessions:
    messages = get_session_messages(sessions[0].session_id)
    for msg in messages:
        print(f"[{msg.type}] {msg.uuid}")
```

### `get_session_info()`

Lit les métadonnées d'une seule session par son ID sans parcourir tout le répertoire du projet. Synchrone ; renvoie immédiatement.

```python theme={null}
def get_session_info(
    session_id: str,
    directory: str | None = None,
) -> SDKSessionInfo | None
```

#### Paramètres

| Paramètre | Type | Défaut | Description |
| :- | :- | :- | :- |
| `session_id` | `str` | requis | UUID de la session à rechercher |
| `directory` | `str \| None` | `None` | Chemin du répertoire de projet. S'il est omis, cherche dans tous les répertoires de projet |

Renvoie [`SDKSessionInfo`](#return-type-sdksessioninfo), ou `None` si la session est introuvable.

#### Exemple

Recherche les métadonnées d'une seule session sans parcourir le répertoire du projet. Utile quand vous disposez déjà d'un ID de session issu d'une exécution précédente.

```python theme={null}
from claude_agent_sdk import get_session_info

info = get_session_info("550e8400-e29b-41d4-a716-446655440000")
if info:
    print(f"{info.summary} (branch: {info.git_branch}, tag: {info.tag})")
```

### `rename_session()`

Renomme une session en ajoutant une entrée de titre personnalisé. Les appels répétés sont sans risque ; le titre le plus récent l'emporte. Synchrone.

```python theme={null}
def rename_session(
    session_id: str,
    title: str,
    directory: str | None = None,
) -> None
```

#### Paramètres

| Paramètre | Type | Défaut | Description |
| :- | :- | :- | :- |
| `session_id` | `str` | requis | UUID de la session à renommer |
| `title` | `str` | requis | Nouveau titre. Doit être non vide une fois les espaces retirés |
| `directory` | `str \| None` | `None` | Chemin du répertoire de projet. S'il est omis, cherche dans tous les répertoires de projet |

Lève `ValueError` si `session_id` n'est pas un UUID valide ou si `title` est vide ; `FileNotFoundError` si la session est introuvable.

#### Exemple

Renomme la session la plus récente pour la retrouver plus facilement plus tard. Le nouveau titre apparaît dans [`SDKSessionInfo.custom_title`](#return-type-sdksessioninfo) lors des lectures suivantes.

```python theme={null}
from claude_agent_sdk import list_sessions, rename_session

sessions = list_sessions(directory="/path/to/project", limit=1)
if sessions:
    rename_session(sessions[0].session_id, "Refactor auth module")
```

### `tag_session()`

Étiquette une session. Passez `None` pour effacer l'étiquette. Les appels répétés sont sans risque ; l'étiquette la plus récente l'emporte. Synchrone.

```python theme={null}
def tag_session(
    session_id: str,
    tag: str | None,
    directory: str | None = None,
) -> None
```

#### Paramètres

| Paramètre | Type | Défaut | Description |
| :- | :- | :- | :- |
| `session_id` | `str` | requis | UUID de la session à étiqueter |
| `tag` | `str \| None` | requis | Chaîne d'étiquette, ou `None` pour effacer. Assainie (Unicode) avant stockage |
| `directory` | `str \| None` | `None` | Chemin du répertoire de projet. S'il est omis, cherche dans tous les répertoires de projet |

Lève `ValueError` si `session_id` n'est pas un UUID valide ou si `tag` est vide après assainissement ; `FileNotFoundError` si la session est introuvable.

#### Exemple

Étiquette une session, puis filtre sur cette étiquette lors d'une lecture ultérieure. Passez `None` pour effacer une étiquette existante.

```python theme={null}
from claude_agent_sdk import list_sessions, tag_session

# Étiqueter la session la plus récente
sessions = list_sessions(directory="/path/to/project", limit=1)
if sessions:
    tag_session(sessions[0].session_id, "needs-review")

# Plus tard : retrouver toutes les sessions portant cette étiquette
for session in list_sessions(directory="/path/to/project"):
    if session.tag == "needs-review":
        print(session.summary)
```

### `fork_session()`

Copie la transcription d'une session dans une nouvelle session, pour emmener la conversation dans une autre direction tandis que l'originale reste inchangée. Pour bifurquer depuis un point antérieur de la conversation, passez `up_to_message_id`. Synchrone.

```python theme={null}
def fork_session(
    session_id: str,
    directory: str | None = None,
    up_to_message_id: str | None = None,
    title: str | None = None,
) -> ForkSessionResult
```

#### Paramètres

| Paramètre | Type | Défaut | Description |
| :- | :- | :- | :- |
| `session_id` | `str` | requis | UUID de la session à dupliquer (fork) |
| `directory` | `str \| None` | `None` | Chemin du répertoire de projet. S'il est omis, cherche dans tous les répertoires de projet |
| `up_to_message_id` | `str \| None` | `None` | Copie la transcription jusqu'au message portant cet UUID inclus, par exemple un `uuid` issu de [`get_session_messages()`](#get_session_messages). S'il est omis, copie toute la transcription |
| `title` | `str \| None` | `None` | Titre du fork. S'il est omis, le SDK en dérive un de la session originale, suivi de `(fork)` |

Renvoie un `ForkSessionResult` dont le `session_id` est l'UUID de la nouvelle session. Passez-le comme [`resume`](#claudeagentoptions) pour poursuivre le fork. Le fork n'inclut pas les [points de contrôle de fichiers](https://code.claude.com/docs/en/agent-sdk/file-checkpointing) de la session originale : vous ne pouvez donc pas le rembobiner vers un point de contrôle capturé avant le fork.

`fork_session()` lève :

* `ValueError` : `session_id` ou `up_to_message_id` n'est pas un UUID valide
* `ValueError` : la session ne contient aucun message, ou `up_to_message_id` ne correspond à aucun message de la transcription
* `FileNotFoundError` : la session est introuvable

#### Exemple

Duplique la session la plus récente sous un nouveau titre, puis reprend le fork. La session originale conserve son propre historique.

```python theme={null}
from claude_agent_sdk import fork_session, list_sessions

sessions = list_sessions(directory="/path/to/project", limit=1)
if sessions:
    forked = fork_session(sessions[0].session_id, title="Try the OAuth approach")
    print(forked.session_id)  # à passer à ClaudeAgentOptions(resume=...) pour poursuivre le fork
```

## Classes

### `ClaudeSDKClient`

**Maintient une session de conversation sur plusieurs échanges.** C'est l'équivalent Python du fonctionnement interne de la fonction `query()` du SDK TypeScript : il crée un objet client capable de poursuivre des conversations. Voir la [comparaison avec `query()`](#choosing-between-query-and-claudesdkclient).

```python theme={null}
class ClaudeSDKClient:
    def __init__(self, options: ClaudeAgentOptions | None = None, transport: Transport | None = None)
    async def connect(self, prompt: str | AsyncIterable[dict] | None = None) -> None
    async def query(self, prompt: str | AsyncIterable[dict], session_id: str = "default") -> None
    async def receive_messages(self) -> AsyncIterator[Message]
    async def receive_response(self) -> AsyncIterator[Message]
    async def interrupt(self) -> None
    async def set_permission_mode(self, mode: PermissionMode) -> None
    async def set_model(self, model: str | None = None) -> None
    async def rewind_files(self, user_message_id: str) -> None
    async def get_mcp_status(self) -> McpStatusResponse
    async def get_context_usage(self) -> ContextUsageResponse
    async def reconnect_mcp_server(self, server_name: str) -> None
    async def toggle_mcp_server(self, server_name: str, enabled: bool) -> None
    async def stop_task(self, task_id: str) -> None
    async def get_server_info(self) -> dict[str, Any] | None
    async def disconnect(self) -> None
```

#### Méthodes

| Méthode | Description |
| :- | :- |
| `__init__(options)` | Initialise le client avec une configuration facultative |
| `connect(prompt)` | Se connecte à Claude avec un prompt initial ou un flux de messages facultatif |
| `query(prompt, session_id)` | Envoie une nouvelle requête en mode streaming |
| `receive_messages()` | Reçoit tous les messages de Claude sous forme d'itérateur asynchrone |
| `receive_response()` | Reçoit les messages jusqu'à un ResultMessage inclus |
| `interrupt()` | Envoie un signal d'interruption (ne fonctionne qu'en mode streaming) |
| `set_permission_mode(mode)` | Change le mode de permission de la session en cours |
| `set_model(model)` | Change le modèle de la session en cours. Passez `None` pour revenir au [modèle par défaut de Claude Code](https://code.claude.com/docs/en/model-config) |
| `rewind_files(user_message_id)` | Restaure les fichiers dans leur état au message utilisateur indiqué. Exige `enable_file_checkpointing=True`. Voir [Points de contrôle de fichiers](https://code.claude.com/docs/en/agent-sdk/file-checkpointing) |
| `get_mcp_status()` | Obtient l'état de tous les serveurs MCP configurés. Renvoie [`McpStatusResponse`](#mcpstatusresponse) |
| `get_context_usage()` | Obtient la répartition de l'utilisation de la fenêtre de contexte par catégorie, skill et outil — les mêmes données que `/context` affiche en session interactive. Renvoie [`ContextUsageResponse`](#contextusageresponse). Pour calculer cette répartition, Claude Code fait plusieurs requêtes API de comptage de tokens qui n'apparaissent pas dans le flux de messages ; voir [comment ces requêtes sont traitées](#contextusageresponse) |
| `reconnect_mcp_server(server_name)` | Retente la connexion à un serveur MCP qui a échoué ou a été déconnecté |
| `toggle_mcp_server(server_name, enabled)` | Active ou désactive un serveur MCP en cours de session. Désactiver un serveur stdio, SSE ou HTTP retire ses outils |
| `stop_task(task_id)` | Arrête une tâche d'arrière-plan en cours. Un [`TaskNotificationMessage`](#tasknotificationmessage) avec le statut `"stopped"` suit dans le flux de messages |
| `get_server_info()` | Obtient les informations d'initialisation du serveur, y compris les commandes et styles de sortie disponibles |
| `disconnect()` | Se déconnecte de Claude |

#### Prise en charge du gestionnaire de contexte (Context Manager Support)

Le client peut être utilisé comme gestionnaire de contexte asynchrone pour une gestion automatique de la connexion :

```python theme={null}
import asyncio
from claude_agent_sdk import ClaudeSDKClient


async def main():
    async with ClaudeSDKClient() as client:
        await client.query("Hello Claude")
        async for message in client.receive_response():
            print(message)


asyncio.run(main())
```

> **Important** : lorsque vous itérez sur les messages, évitez d'utiliser `break` pour sortir prématurément, car cela peut causer des problèmes de nettoyage asyncio. Laissez plutôt l'itération se terminer naturellement, ou utilisez des drapeaux pour noter quand vous avez trouvé ce qu'il vous faut.

#### Exemple — Poursuivre une conversation

```python theme={null}
import asyncio
from claude_agent_sdk import ClaudeSDKClient, AssistantMessage, TextBlock, ResultMessage


async def main():
    async with ClaudeSDKClient() as client:
        # Première question
        await client.query("What's the capital of France?")

        # Traiter la réponse
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(f"Claude: {block.text}")

        # Question de suivi : la session conserve le contexte précédent
        await client.query("What's the population of that city?")

        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(f"Claude: {block.text}")

        # Autre question de suivi : toujours dans la même conversation
        await client.query("What are some famous landmarks there?")

        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(f"Claude: {block.text}")


asyncio.run(main())
```

#### Exemple — Entrée en streaming avec ClaudeSDKClient

`query()` accepte aussi un itérable asynchrone de dicts de messages utilisateur : vous pouvez ainsi assembler le prompt au moment de l'envoi ou inclure des blocs de contenu comme des images. Claude Code commence à répondre au premier message produit dès son arrivée, sans attendre la fin de l'itérable, et `receive_response()` s'arrête au `ResultMessage` qui termine cette réponse. Mettez tout ce que Claude doit lire avant de répondre dans un seul message, comme le fait ce générateur, et associez chaque appel `query()` à sa propre boucle `receive_response()`.

```python theme={null}
import asyncio
from claude_agent_sdk import ClaudeSDKClient


async def message_stream():
    """Assemble the prompt at send time and yield it as one user message."""
    readings = {"Temperature": "25°C", "Humidity": "60%"}
    data = ", ".join(f"{name}: {value}" for name, value in readings.items())
    yield {
        "type": "user",
        "message": {
            "role": "user",
            "content": f"Analyze the following sensor data and describe any patterns you see: {data}",
        },
    }


async def main():
    async with ClaudeSDKClient() as client:
        # Envoyer l'entrée en streaming à Claude
        await client.query(message_stream())

        # Traiter la réponse
        async for message in client.receive_response():
            print(message)

        # Suivi dans la même session
        await client.query("Should we be concerned about these readings?")

        async for message in client.receive_response():
            print(message)


asyncio.run(main())
```

#### Exemple — Utiliser les interruptions

```python theme={null}
import asyncio
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions, ResultMessage


async def interruptible_task():
    options = ClaudeAgentOptions(allowed_tools=["Bash"], permission_mode="acceptEdits")

    async with ClaudeSDKClient(options=options) as client:
        # Lancer une tâche longue
        await client.query("Count from 1 to 100 slowly, using the bash sleep command")

        # La laisser tourner un moment
        await asyncio.sleep(2)

        # Interrompre la tâche
        await client.interrupt()
        print("Task interrupted!")

        # Vider les messages de la tâche interrompue (y compris son ResultMessage)
        async for message in client.receive_response():
            if isinstance(message, ResultMessage):
                print(f"Interrupted task: terminal_reason={message.terminal_reason!r}")
                # terminal_reason vaut "aborted_streaming" ou "aborted_tools"
                # pour les tours interrompus

        # Envoyer une nouvelle commande
        await client.query("Just say hello instead")

        # Recevoir maintenant la nouvelle réponse
        async for message in client.receive_response():
            if isinstance(message, ResultMessage) and message.subtype == "success":
                print(f"New result: {message.result}")


asyncio.run(interruptible_task())
```

> **Note** : **comportement du tampon après une interruption** — `interrupt()` envoie un signal d'arrêt mais ne vide pas le tampon de messages. Les messages déjà produits par la tâche interrompue, y compris son `ResultMessage`, restent dans le flux. Vous devez les vider avec `receive_response()` avant de lire la réponse à une nouvelle requête. Si vous envoyez une nouvelle requête juste après `interrupt()` et n'appelez `receive_response()` qu'une fois, vous recevrez les messages de la tâche interrompue, pas la réponse à la nouvelle requête.

#### Exemple — Contrôle avancé des permissions

```python theme={null}
import asyncio
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions
from claude_agent_sdk.types import (
    PermissionResultAllow,
    PermissionResultDeny,
    ToolPermissionContext,
)


async def custom_permission_handler(
    tool_name: str, input_data: dict, context: ToolPermissionContext
) -> PermissionResultAllow | PermissionResultDeny:
    """Custom logic for tool permissions."""

    # Bloquer les écritures dans les répertoires système
    if tool_name == "Write" and input_data.get("file_path", "").startswith("/system/"):
        return PermissionResultDeny(
            message="System directory write not allowed", interrupt=True
        )

    # Rediriger les opérations sur fichiers sensibles
    if tool_name in ["Write", "Edit"] and "config" in input_data.get("file_path", ""):
        safe_path = f"./sandbox/{input_data['file_path']}"
        return PermissionResultAllow(
            updated_input={**input_data, "file_path": safe_path}
        )

    # Autoriser tout le reste
    return PermissionResultAllow(updated_input=input_data)


async def main():
    # Ne listez pas aussi les outils contrôlés dans allowed_tools : les règles allow approuvent les appels avant que can_use_tool ne s'exécute
    options = ClaudeAgentOptions(can_use_tool=custom_permission_handler)

    async with ClaudeSDKClient(options=options) as client:
        await client.query("Update the system config file")

        async for message in client.receive_response():
            # Utilisera le chemin sandbox à la place
            print(message)


asyncio.run(main())
```

## Types

> **Note** : **`@dataclass` ou `TypedDict`** — ce SDK utilise deux sortes de types. Les classes décorées avec `@dataclass` (comme `ResultMessage`, `AgentDefinition`, `TextBlock`) sont des instances d'objets à l'exécution et prennent en charge l'accès par attribut : `msg.result`. Les classes définies avec `TypedDict` (comme `ThinkingConfigEnabled`, `McpStdioServerConfig`, `SyncHookJSONOutput`) sont **de simples dicts à l'exécution** et exigent l'accès par clé : `config["budget_tokens"]`, et non `config.budget_tokens`. La syntaxe d'appel `ClassName(field=value)` fonctionne pour les deux, mais seules les dataclasses produisent des objets avec attributs.

### `SdkMcpTool`

Définition d'un outil MCP du SDK créé avec le décorateur `@tool`.

```python theme={null}
@dataclass
class SdkMcpTool(Generic[T]):
    name: str
    description: str
    input_schema: type[T] | dict[str, Any]
    handler: Callable[[T], Awaitable[dict[str, Any]]]
    annotations: ToolAnnotations | None = None
```

| Propriété | Type | Description |
| :- | :- | :- |
| `name` | `str` | Identifiant unique de l'outil |
| `description` | `str` | Description lisible |
| `input_schema` | `type[T] \| dict[str, Any]` | Schéma de validation des entrées |
| `handler` | `Callable[[T], Awaitable[dict[str, Any]]]` | Fonction asynchrone qui gère l'exécution de l'outil |
| `annotations` | [`ToolAnnotations`](#toolannotations)` \| None` | Annotations d'outil facultatives (par exemple `readOnlyHint`, `destructiveHint`, `openWorldHint`, `maxResultSizeChars`) |

### `Transport`

Classe de base abstraite pour les implémentations de transport personnalisées. Utilisez-la pour communiquer avec le processus Claude via un canal personnalisé (par exemple une connexion distante au lieu d'un sous-processus local).

> **Avertissement** : il s'agit d'une API interne de bas niveau. L'interface peut changer dans de futures versions. Les implémentations personnalisées doivent être mises à jour pour suivre tout changement d'interface.

```python theme={null}
from abc import ABC, abstractmethod
from collections.abc import AsyncIterator
from typing import Any


class Transport(ABC):
    @abstractmethod
    async def connect(self) -> None: ...

    @abstractmethod
    async def write(self, data: str) -> None: ...

    @abstractmethod
    def read_messages(self) -> AsyncIterator[dict[str, Any]]: ...

    @abstractmethod
    async def close(self) -> None: ...

    @abstractmethod
    def is_ready(self) -> bool: ...

    @abstractmethod
    async def end_input(self) -> None: ...
```

| Méthode | Description |
| :- | :- |
| `connect()` | Connecte le transport et le prépare à la communication |
| `write(data)` | Écrit des données brutes (JSON + saut de ligne) dans le transport |
| `read_messages()` | Itérateur asynchrone qui produit les messages JSON analysés |
| `close()` | Ferme la connexion et libère les ressources |
| `is_ready()` | Renvoie `True` si le transport peut envoyer et recevoir |
| `end_input()` | Ferme le flux d'entrée (par exemple, ferme stdin pour les transports à sous-processus) |

Import : `from claude_agent_sdk import Transport`

### `ClaudeAgentOptions`

Dataclass de configuration des requêtes Claude Code.

```python theme={null}
@dataclass
class ClaudeAgentOptions:
    tools: list[str] | ToolsPreset | None = None
    allowed_tools: list[str] = field(default_factory=list)
    system_prompt: str | SystemPromptPreset | SystemPromptCustom | SystemPromptFile | None = None
    mcp_servers: dict[str, McpServerConfig] | str | Path = field(default_factory=dict)
    strict_mcp_config: bool = False
    permission_mode: PermissionMode | None = None
    continue_conversation: bool = False
    resume: str | None = None
    session_id: str | None = None
    max_turns: int | None = None
    max_budget_usd: float | None = None
    disallowed_tools: list[str] = field(default_factory=list)
    model: str | None = None
    fallback_model: str | None = None
    betas: list[SdkBeta] = field(default_factory=list)
    output_format: dict[str, Any] | None = None
    permission_prompt_tool_name: str | None = None
    cwd: str | Path | None = None
    cli_path: str | Path | None = None
    settings: str | None = None
    add_dirs: list[str | Path] = field(default_factory=list)
    env: dict[str, str] = field(default_factory=dict)
    extra_args: dict[str, str | None] = field(default_factory=dict)
    max_buffer_size: int | None = None
    debug_stderr: Any = sys.stderr  # Obsolète
    stderr: Callable[[str], None] | None = None
    can_use_tool: CanUseTool | None = None
    hooks: dict[HookEvent, list[HookMatcher]] | None = None
    user: str | None = None
    include_partial_messages: bool = False
    include_hook_events: bool = False
    forward_subagent_text: bool = False
    verbatim_prompts: bool = False
    fork_session: bool = False
    resume_session_at: str | None = None
    resume_drops_turn: str | None = None
    agents: dict[str, AgentDefinition] | None = None
    setting_sources: list[SettingSource] | None = None
    skills: list[str] | Literal["all"] | None = None
    sandbox: SandboxSettings | None = None
    plugins: list[SdkPluginConfig] = field(default_factory=list)
    max_thinking_tokens: int | None = None  # Obsolète : utilisez thinking à la place
    thinking: ThinkingConfig | None = None
    effort: EffortLevel | None = None
    enable_file_checkpointing: bool = False
    session_store: SessionStore | None = None
    session_store_flush: SessionStoreFlushMode = "batched"
    load_timeout_ms: int = 60_000
    task_budget: TaskBudget | None = None
```

| Propriété | Type | Défaut | Description |
| :- | :- | :- | :- |
| `tools` | `list[str] \| ToolsPreset \| None` | `None` | Configuration des outils. Utilisez `{"type": "preset", "preset": "claude_code"}` pour les outils par défaut de Claude Code |
| `allowed_tools` | `list[str]` | `[]` | Outils à approuver automatiquement sans demande, hormis les lectures depuis des [chemins réseau](https://code.claude.com/docs/en/permissions#network-paths). Cela ne restreint pas Claude à ces seuls outils. Si vous nommez ici l'un des [outils de suivi de tâches](https://code.claude.com/docs/en/agent-sdk/todo-tracking#model-availability), Claude Code y inscrit aussi la session. Les autres outils non listés passent par `permission_mode` et `can_use_tool`. Utilisez `disallowed_tools` pour bloquer des outils. Voir [Permissions](https://code.claude.com/docs/en/agent-sdk/permissions#allow-and-deny-rules) |
| `system_prompt` | `str \| SystemPromptPreset \| SystemPromptCustom \| SystemPromptFile \| None` | `None` | Configuration du prompt système. Passez une chaîne pour un prompt personnalisé, `{"type": "preset", "preset": "claude_code"}` pour le prompt système de Claude Code avec `"append"` facultatif, `{"type": "custom", "prompt": "..."}` pour un prompt personnalisé pouvant aussi définir `"snapshot"`, ou `{"type": "file", "path": "..."}` pour charger un gros prompt depuis le disque. Voir [`SystemPromptPreset`](#systempromptpreset), [`SystemPromptCustom`](#systempromptcustom) et [`SystemPromptFile`](#systempromptfile) |
| `mcp_servers` | `dict[str, McpServerConfig] \| str \| Path` | `{}` | Configurations de serveurs MCP ou chemin vers un fichier de configuration |
| `strict_mcp_config` | `bool` | `False` | Si `True`, n'utilise que les serveurs passés dans `mcp_servers` et ignore le `.mcp.json` du projet, les réglages utilisateur, les serveurs MCP fournis par des plugins et les [connecteurs claude.ai](https://code.claude.com/docs/en/mcp#use-mcp-servers-from-claude-ai). Correspond au flag CLI `--strict-mcp-config` |
| `permission_mode` | `PermissionMode \| None` | `None` | Mode de permission pour l'usage des outils |
| `continue_conversation` | `bool` | `False` | Poursuit la conversation la plus récente |
| `resume` | `str \| None` | `None` | ID de la session à reprendre |
| `session_id` | `str \| None` | `None` | Utilise un ID de session précis au lieu d'un ID généré automatiquement. Doit être un UUID valide. Ne peut pas être combiné avec `continue_conversation` ou `resume`, sauf si `fork_session` est aussi défini |
| `max_turns` | `int \| None` | `None` | Nombre maximal de tours agentiques (allers-retours d'utilisation d'outils) |
| `max_budget_usd` | `float \| None` | `None` | Arrête la requête quand l'estimation de coût côté client atteint cette valeur en USD. L'estimation peut dépasser cette valeur, donc [gardez de la marge](https://code.claude.com/docs/en/agent-sdk/agent-loop#budget-headroom). Ne compte que la dépense propre de l'appel ; les totaux restaurés depuis une session reprise ne comptent pas. Pour les réserves de précision et le comportement de réinitialisation, voir [Suivre le coût et l'utilisation](https://code.claude.com/docs/en/agent-sdk/cost-tracking) |
| `disallowed_tools` | `list[str]` | `[]` | Outils à refuser. Un nom nu comme `"Bash"` retire l'outil du contexte de Claude. Une règle ciblée comme `"Bash(rm *)"` laisse l'outil disponible et refuse les appels correspondants dans tous les modes de permission, y compris `bypassPermissions`, pour la commande [telle qu'écrite](https://code.claude.com/docs/en/permissions#bash-rule-limits). Voir [Permissions](https://code.claude.com/docs/en/agent-sdk/permissions#allow-and-deny-rules) |
| `enable_file_checkpointing` | `bool` | `False` | Active le suivi des modifications de fichiers pour pouvoir rembobiner. Voir [Points de contrôle de fichiers](https://code.claude.com/docs/en/agent-sdk/file-checkpointing) |
| `model` | `str \| None` | `None` | Alias de modèle Claude ou nom complet du modèle. Voir les [valeurs acceptées et ID propres à chaque fournisseur](https://code.claude.com/docs/en/model-config#available-models) |
| `fallback_model` | `str \| None` | `None` | Modèle de repli à utiliser si le modèle principal échoue. Accepte une liste séparée par des virgules. Pour des conseils, voir [Choisir un modèle](https://code.claude.com/docs/en/agent-sdk/configuration#choose-a-model) |
| `betas` | `list[SdkBeta]` | `[]` | Fonctionnalités bêta à activer. Voir [`SdkBeta`](#sdkbeta) pour les options disponibles |
| `output_format` | `dict[str, Any] \| None` | `None` | Format de sortie pour les réponses structurées (ex. `{"type": "json_schema", "schema": {...}}`). Voir [Sorties structurées](https://code.claude.com/docs/en/agent-sdk/structured-outputs) pour les détails |
| `permission_prompt_tool_name` | `str \| None` | `None` | Nom de l'outil MCP pour les demandes de permission |
| `cwd` | `str \| Path \| None` | `None` | Répertoire de travail courant |
| `cli_path` | `str \| Path \| None` | `None` | Chemin personnalisé vers l'exécutable CLI de Claude Code |
| `settings` | `str \| None` | `None` | Chemin vers un fichier de réglages ou chaîne JSON en ligne |
| `add_dirs` | `list[str \| Path]` | `[]` | Répertoires supplémentaires accessibles à Claude. Le SDK passe chaque entrée à Claude Code sous forme de `--add-dir` ; avec la source de réglages `project`, Claude Code [charge aussi les skills, commandes et sous-agents du répertoire](https://code.claude.com/docs/en/permissions#additional-directories-grant-file-access-not-configuration) |
| `env` | `dict[str, str]` | `{}` | Variables d'environnement fusionnées par-dessus l'environnement hérité du processus. Voir [Variables d'environnement](https://code.claude.com/docs/en/env-vars) pour les variables que lit la CLI sous-jacente, et [Gérer les réponses API lentes ou bloquées](#handle-slow-or-stalled-api-responses) pour les variables liées aux délais. Définissez `CLAUDE_AGENT_SDK_CLIENT_APP` pour identifier votre application dans l'en-tête User-Agent |
| `extra_args` | `dict[str, str \| None]` | `{}` | Arguments CLI supplémentaires passés directement à la CLI |
| `max_buffer_size` | `int \| None` | `None` | Nombre maximal d'octets lors de la mise en tampon de la sortie stdout de la CLI |
| `debug_stderr` | `Any` | `sys.stderr` | *Obsolète* — le SDK ignore cette valeur. Utilisez le callback `stderr` pour la sortie stderr de la CLI |
| `stderr` | `Callable[[str], None] \| None` | `None` | Fonction de rappel pour la sortie stderr de la CLI |
| `can_use_tool` | [`CanUseTool`](#canusetool) ` \| None` | `None` | Callback de permission d'outil, invoqué uniquement quand le [flux de permissions](https://code.claude.com/docs/en/agent-sdk/permissions#how-permissions-are-evaluated) aboutit à une demande. Non invoqué pour les appels approuvés automatiquement par `allowed_tools`, les règles allow ou `permission_mode`. Une règle allow ne pré-approuve pas les [actions qu'aucun mode n'approuve automatiquement](https://code.claude.com/docs/en/permission-modes#actions-no-mode-auto-approves). Voir [`CanUseTool`](#canusetool) pour les détails |
| `hooks` | `dict[HookEvent, list[HookMatcher]] \| None` | `None` | Configurations de hooks pour intercepter des événements |
| `user` | `str \| None` | `None` | Sur les plateformes POSIX, le compte utilisateur du système sous lequel s'exécute le sous-processus Claude Code. Claude Code conserve l'environnement du processus parent, y compris `HOME`, et s'exécute dans `cwd` |
| `include_partial_messages` | `bool` | `False` | Inclut les événements de streaming de messages partiels. Une fois activé, des messages [`StreamEvent`](#streamevent) sont produits |
| `include_hook_events` | `bool` | `False` | Inclut les événements de cycle de vie des hooks dans le flux de messages sous forme d'objets `HookEventMessage` |
| `forward_subagent_text` | `bool` | `False` | Transmet les blocs de texte et de réflexion (thinking) des sous-agents dans le flux de messages. Sans cette option, Claude Code omet les blocs de texte et de réflexion d'un sous-agent qui s'exécute au [premier plan](https://code.claude.com/docs/en/sub-agents#run-subagents-in-foreground-or-background). Pour les sous-agents imbriqués, les skills avec `context: fork` et la version de Claude Code requise pour chacun, voir [Suivre les messages des sous-agents](https://code.claude.com/docs/en/headless#follow-subagent-messages). Exige le SDK Agent Python 0.2.140 ou ultérieur |
| `verbatim_prompts` | `bool` | `False` | Transmet chaque prompt tel qu'écrit. Le SDK envoie chaque message utilisateur avec `client_composed` à `True`. Voir [`client_composed`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkusermessage) pour ce que Claude Code saute sur ces messages. Utilisez cette option quand le texte de votre prompt contient du contenu que l'utilisateur final n'a pas tapé. Pour un contrôle tour par tour, laissez-la désactivée et définissez `"client_composed": True` sur des messages streamés individuels. Tant que l'option est active, le SDK écrase toute valeur `client_composed` que vous définissez. Exige le SDK Agent Python 0.2.158+ et Claude Code v2.1.248+ ; la CLI fournie avec ces versions du SDK satisfait l'exigence Claude Code |
| `fork_session` | `bool` | `False` | Lors d'une reprise avec `resume`, bifurque vers un nouvel ID de session au lieu de poursuivre la session originale |
| `resume_session_at` | `str \| None` | `None` | Lors d'une reprise, ne charge la conversation que jusqu'au message portant cet UUID inclus. À utiliser avec `resume`, et généralement `fork_session`, pour bifurquer depuis un point antérieur. Exige le SDK Agent Python 0.2.137 ou ultérieur |
| `resume_drops_turn` | `str \| None` | `None` | UUID du prompt utilisateur dont le tour est écarté par une troncature `resume_session_at`. Si défini, la CLI refuse la reprise si la plage écartée contient des entrées non attribuables à ce tour. Exige le SDK Agent Python 0.2.137+ et Claude Code v2.1.223+ ; la CLI fournie avec ces versions du SDK satisfait l'exigence Claude Code |
| `agents` | `dict[str, AgentDefinition] \| None` | `None` | Sous-agents définis par programmation |
| `plugins` | `list[SdkPluginConfig]` | `[]` | Charge des plugins personnalisés depuis des chemins locaux. Voir [Plugins](https://code.claude.com/docs/en/agent-sdk/plugins) pour les détails |
| `sandbox` | [`SandboxSettings`](#sandboxsettings) ` \| None` | `None` | Configure le comportement du bac à sable par programmation. Voir [Réglages du bac à sable](#sandboxsettings) pour les détails |
| `setting_sources` | `list[SettingSource] \| None` | `None` (défaut CLI : toutes les sources) | Contrôle quels réglages du système de fichiers charger. Passez `[]` pour désactiver les réglages user, project et local. Avec `skills` défini et ce champ non défini, seules les sources user et project sont chargées. Définissez `setting_sources` explicitement pour conserver les réglages locaux. La politique gérée par point de terminaison (endpoint-managed) est chargée dans tous les cas ; les réglages gérés par serveur sont récupérés quand la session s'authentifie avec un identifiant d'organisation sur une [configuration éligible](https://code.claude.com/docs/en/server-managed-settings#platform-availability). Pour les entrées lues indépendamment de cette option, voir [Ce que settingSources ne contrôle pas](https://code.claude.com/docs/en/agent-sdk/claude-code-features#what-settingsources-does-not-control) |
| `skills` | `list[str] \| Literal["all"] \| None` | `None` | Skills disponibles pour la session. Passez `"all"` pour activer toutes les skills découvertes, ou une liste de noms de skills. Ne passez que des noms exacts. Le SDK rejette les noms mal formés et les noms sous forme de joker avec une `ValueError` avant de démarrer le processus Claude Code ; cette vérification exige le SDK Agent Python 0.2.129 ou ultérieur. Une fois défini, le SDK ajoute automatiquement l'outil Skill à `allowed_tools`. Si vous passez aussi `tools`, incluez `"Skill"` dans cette liste. Voir [Skills](https://code.claude.com/docs/en/agent-sdk/skills) |
| `max_thinking_tokens` | `int \| None` | `None` | *Obsolète* — nombre maximal de tokens pour les blocs de réflexion. Utilisez `thinking` à la place |
| `thinking` | [`ThinkingConfig`](#thinkingconfig) ` \| None` | `None` | Contrôle le comportement de la réflexion étendue (extended thinking). Prime sur `max_thinking_tokens` |
| `effort` | [`EffortLevel`](#effortlevel) ` \| None` | `None` | Niveau d'effort pour la profondeur de réflexion. Voir [ajuster le niveau d'effort](https://code.claude.com/docs/en/model-config#adjust-effort-level) |
| `session_store` | [`SessionStore`](https://code.claude.com/docs/en/agent-sdk/session-storage#the-sessionstore-interface) ` \| None` | `None` | Recopie les transcriptions de session vers un backend externe pour qu'un autre hôte puisse les reprendre. Voir [Persister les sessions dans un stockage externe](https://code.claude.com/docs/en/agent-sdk/session-storage) |
| `session_store_flush` | `Literal["batched", "eager"]` | `"batched"` | Moment où vider les entrées de transcription recopiées vers `session_store`. `"batched"` vide une fois par tour ou quand le tampon est plein ; `"eager"` déclenche un vidage en arrière-plan après chaque trame. Ignoré quand `session_store` vaut `None` |
| `load_timeout_ms` | `int` | `60000` | Délai par appel pour `session_store.load()` et `list_subkeys()` pendant la matérialisation de la reprise, en millisecondes |
| `task_budget` | `TaskBudget \| None` | `None` | Budget de tokens côté API. Envoyé sous `output_config.task_budget` avec l'en-tête bêta `task-budgets-2026-03-13`. Passez `{"total": <int>}`. |

#### Gérer les réponses API lentes ou bloquées (Handle slow or stalled API responses)

Le sous-processus CLI lit plusieurs variables d'environnement qui contrôlent les délais d'API et la détection de blocage. Passez-les via `ClaudeAgentOptions.env` :

```python theme={null}
from claude_agent_sdk import ClaudeAgentOptions

options = ClaudeAgentOptions(
    env={
        "API_TIMEOUT_MS": "120000",
        "CLAUDE_CODE_MAX_RETRIES": "2",
        "CLAUDE_ASYNC_AGENT_STALL_TIMEOUT_MS": "120000",
    },
)
```

* `API_TIMEOUT_MS` : délai par requête sur le client Anthropic, en millisecondes. Défaut `600000`. S'applique à la boucle principale et à tous les sous-agents.
* `CLAUDE_CODE_MAX_RETRIES` : nombre maximal de nouvelles tentatives d'API. Défaut `10`, plafonné à `15`. Chaque tentative dispose de sa propre fenêtre `API_TIMEOUT_MS`.

  Pour les exécutions sans surveillance qui doivent patienter pendant des pannes plus longues, définissez [`CLAUDE_CODE_RETRY_WATCHDOG=1`](https://code.claude.com/docs/en/errors#tune-retry-behavior) : il retente indéfiniment les erreurs de capacité transitoires et, sur Claude Code v2.1.199 ou ultérieur, porte le défaut pour les autres erreurs transitoires à `300` et supprime le plafond de cette variable.
* `CLAUDE_ASYNC_AGENT_STALL_TIMEOUT_MS` : chien de garde de blocage pour les sous-agents. Tant que le chien de garde de flux est actif, le défaut est `CLAUDE_STREAM_IDLE_TIMEOUT_MS` plus 5 minutes, soit `600000` à moins d'augmenter cette variable. Avec le chien de garde de flux désactivé, le défaut est `600000`. Avant la v2.1.257, le défaut était toujours `600000`.

  Le minuteur se réinitialise à chaque événement de flux. En cas de blocage, Claude Code abandonne le sous-agent et signale le blocage au parent. Pour un sous-agent d'arrière-plan, il marque aussi la tâche comme échouée et joint tout résultat partiel.
* `CLAUDE_ENABLE_STREAM_WATCHDOG` avec `CLAUDE_STREAM_IDLE_TIMEOUT_MS` : chien de garde de flux qui abandonne la requête quand les en-têtes sont arrivés mais que le corps de la réponse cesse d'être transmis. Il est actif par défaut pour tous les fournisseurs ; définissez `CLAUDE_ENABLE_STREAM_WATCHDOG=0` pour le désactiver. `CLAUDE_STREAM_IDLE_TIMEOUT_MS` vaut `300000` par défaut et est ramené à ce minimum. Après l'abandon, [Nouvelles tentatives automatiques](https://code.claude.com/docs/en/errors#automatic-retries) décrit ce que fait Claude Code, selon l'avancement de la réponse.

  Pendant que le chien de garde attend une réponse qu'une passerelle derrière `ANTHROPIC_BASE_URL` maintient ouverte avec des pings keep-alive, un hôte qui définit `include_partial_messages` continue de recevoir des messages [`StreamEvent`](#streamevent) `ping`. Interprétez ces trames comme un signe de vie plutôt que de faire expirer la session sur un silence. Avant la v2.1.257, ces trames s'arrêtaient 5 minutes après le dernier véritable événement de flux.

### `OutputFormat`

Configuration de la validation des sorties structurées. Passez-la sous forme de `dict` au champ `output_format` de `ClaudeAgentOptions` :

```python theme={null}
# Forme de dict attendue pour output_format
{
    "type": "json_schema",
    "schema": {...},  # Votre définition JSON Schema
}
```

| Champ | Requis | Description |
| :- | :- | :- |
| `type` | Oui | Doit valoir `"json_schema"` pour la validation JSON Schema |
| `schema` | Oui | Définition JSON Schema pour la validation de la sortie |

### `SystemPromptPreset`

Configuration pour utiliser le prompt système prédéfini de Claude Code, avec ajouts facultatifs.

```python theme={null}
class SystemPromptPreset(TypedDict):
    type: Literal["preset"]
    preset: Literal["claude_code"]
    append: NotRequired[str]
    exclude_dynamic_sections: NotRequired[bool]
    snapshot: NotRequired[bool]
```

| Champ | Requis | Description |
| :- | :- | :- |
| `type` | Oui | Doit valoir `"preset"` pour utiliser un prompt système prédéfini |
| `preset` | Oui | Doit valoir `"claude_code"` pour utiliser le prompt système de Claude Code |
| `append` | Non | Instructions supplémentaires à ajouter à la fin du prompt système prédéfini |
| `exclude_dynamic_sections` | Non | Déplace le contexte propre à l'utilisateur, comme l'emplacement de la mémoire automatique, du prompt système vers le premier message utilisateur. Améliore la réutilisation du cache de prompt entre utilisateurs et machines. Voir [Modifier les prompts système](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts#improve-prompt-caching-across-users-and-machines) |
| `snapshot` | Non | Mettez `False` pour reconstruire le prompt système à chaque requête au lieu de [réutiliser le prompt enregistré par la session lors de sa première requête](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts#change-the-prompt-of-an-existing-session). Exige `claude-agent-sdk` v0.2.153 ou ultérieur |

### `SystemPromptCustom`

Un prompt système personnalisé sous forme d'objet, équivalent à passer une chaîne comme `system_prompt`, qui peut aussi définir `snapshot`. Exige `claude-agent-sdk` v0.2.153 ou ultérieur.

```python theme={null}
class SystemPromptCustom(TypedDict):
    type: Literal["custom"]
    prompt: str
    snapshot: NotRequired[bool]
```

| Champ | Requis | Description |
| :- | :- | :- |
| `type` | Oui | Doit valoir `"custom"` |
| `prompt` | Oui | Le texte du prompt système. Passé à la CLI comme argument de ligne de commande, donc les [limites de longueur de la ligne de commande](#systempromptfile) s'appliquent |
| `snapshot` | Non | Identique à [`SystemPromptPreset.snapshot`](#systempromptpreset), appliqué à `prompt` |

### `SystemPromptFile`

Configuration pour charger un prompt système personnalisé depuis un fichier au lieu de le passer en chaîne. Le SDK la traduit en flag CLI [`--system-prompt-file`](https://code.claude.com/docs/en/cli-reference#system-prompt-flags). Utilisez la forme fichier quand le prompt est volumineux : le SDK passe un `system_prompt` de type chaîne dans l'argv du sous-processus CLI, ce qui est soumis aux limites de longueur de ligne de commande du système avant même que le SDK n'envoie la moindre requête API. Sous Linux, un argument unique de plus d'environ 128 Ko échoue au lancement du processus avec `Argument list too long`. Sous Windows, toute la ligne de commande est plafonnée à environ 32 Ko, donc la forme chaîne échoue à un seuil plus bas.

```python theme={null}
class SystemPromptFile(TypedDict):
    type: Literal["file"]
    path: str
```

| Champ | Requis | Description |
| :- | :- | :- |
| `type` | Oui | Doit valoir `"file"` pour charger le prompt depuis le disque |
| `path` | Oui | Chemin vers un fichier contenant le prompt système |

### `SettingSource`

Contrôle à partir de quelles sources de configuration du système de fichiers le SDK charge les réglages.

```python theme={null}
SettingSource = Literal["user", "project", "local"]
```

| Valeur | Description | Emplacement |
| :- | :- | :- |
| `"user"` | Réglages utilisateur globaux | `~/.claude/settings.json` |
| `"project"` | Réglages de projet partagés (sous contrôle de version) | `.claude/settings.json` |
| `"local"` | Réglages de projet locaux, ignorés par git (gitignored) quand Claude Code y enregistre un réglage | `.claude/settings.local.json` |

#### Comportement par défaut

Quand `setting_sources` est omis ou vaut `None` et que `skills` n'est pas défini, `query()` charge les mêmes réglages du système de fichiers que la CLI Claude Code : user, project et local. Avec `skills` défini, la ligne [`setting_sources`](#claudeagentoptions) décrit le défaut actuel. La politique gérée par point de terminaison est chargée dans tous les cas ; les réglages gérés par serveur sont récupérés quand la session s'authentifie avec un identifiant d'organisation sur une [configuration éligible](https://code.claude.com/docs/en/server-managed-settings#platform-availability). Pour plus d'informations, voir [Ce que settingSources ne contrôle pas](https://code.claude.com/docs/en/agent-sdk/claude-code-features#what-settingsources-does-not-control).

#### Pourquoi utiliser setting\_sources

**Désactiver les réglages du système de fichiers :**

```python theme={null}
# Ne charger aucun réglage user, project ou local depuis le disque
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions


async def main():
    async for message in query(
        prompt="Analyze this code",
        options=ClaudeAgentOptions(
            setting_sources=[]
        ),
    ):
        print(message)


asyncio.run(main())
```

> **Note** : dans le SDK Python 0.1.59 et antérieur, une liste vide était traitée comme l'omission de l'option, donc `setting_sources=[]` ne désactivait pas les réglages du système de fichiers. Passez à une version plus récente si vous avez besoin qu'une liste vide soit prise en compte. Le SDK TypeScript n'est pas concerné.

**Charger uniquement certaines sources de réglages :**

```python theme={null}
# Charger uniquement les réglages de projet, ignorer user et local
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions


async def main():
    async for message in query(
        prompt="Run CI checks",
        options=ClaudeAgentOptions(
            setting_sources=["project"]  # Uniquement .claude/settings.json
        ),
    ):
        print(message)


asyncio.run(main())
```

**Applications 100 % SDK :**

```python theme={null}
# Tout définir par programmation.
# Passez [] pour désactiver les sources de réglages du système de fichiers.
import asyncio
from claude_agent_sdk import AgentDefinition, ClaudeAgentOptions, query


async def main():
    async for message in query(
        prompt="Review this PR",
        options=ClaudeAgentOptions(
            setting_sources=[],
            agents={
                "code-reviewer": AgentDefinition(
                    description="Reviews code changes",
                    prompt="You are a code reviewer. Report issues in the diff.",
                ),
            },
            allowed_tools=["Read", "Grep", "Glob"],
        ),
    ):
        print(message)


asyncio.run(main())
```

Pour charger les instructions de projet CLAUDE.md, incluez `"project"` dans `setting_sources`. Voir [Modifier les prompts système](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts#claude-md-files-for-project-level-instructions) pour savoir comment le chargement de CLAUDE.md interagit avec les options de prompt système.

#### Priorité des réglages (Settings precedence)

Quand plusieurs sources sont chargées, les réglages sont fusionnés selon cette priorité (de la plus haute à la plus basse) :

1. Réglages locaux (`.claude/settings.local.json`)
2. Réglages de projet (`.claude/settings.json`)
3. Réglages utilisateur (`~/.claude/settings.json`)

Les options programmatiques comme `agents`, `allowed_tools` et `settings` prennent le pas sur les réglages user, project et local du système de fichiers. Les réglages de politique gérée (managed policy) priment sur les options programmatiques.

### `AgentDefinition`

Configuration d'un sous-agent défini par programmation.

```python theme={null}
@dataclass
class AgentDefinition:
    description: str
    prompt: str
    tools: list[str] | None = None
    disallowedTools: list[str] | None = None
    model: str | None = None
    skills: list[str] | None = None
    memory: Literal["user", "project", "local"] | None = None
    mcpServers: list[str | dict[str, Any]] | None = None
    initialPrompt: str | None = None
    maxTurns: int | None = None
    background: bool | None = None
    effort: EffortLevel | int | None = None
    permissionMode: PermissionMode | None = None
```

| Champ | Requis | Description |
| :- | :- | :- |
| `description` | Oui | Description en langage naturel du moment où utiliser cet agent |
| `prompt` | Oui | Le prompt système de l'agent |
| `tools` | Non | Tableau des noms d'outils autorisés. S'il est omis, hérite de tous les [outils disponibles pour les sous-agents](https://code.claude.com/docs/en/sub-agents#available-tools) |
| `disallowedTools` | Non | Tableau de noms d'outils à retirer de l'ensemble d'outils de l'agent. Les motifs au niveau d'un serveur MCP sont aussi acceptés : `mcp__server` ou `mcp__server__*` retire tous les outils de ce serveur, et `mcp__*` retire tous les outils MCP de tous les serveurs |
| `model` | Non | Modèle spécifique à cet agent. Accepte un alias comme `"sonnet"`, `"opus"`, `"haiku"` ou `"inherit"`, ou un ID de modèle complet. Si vous l'omettez, Claude Code choisit le modèle selon l'[ordre de sélection du modèle des sous-agents](https://code.claude.com/docs/en/sub-agents#choose-a-model) |
| `skills` | Non | Liste de noms de skills à précharger dans le contexte de l'agent au démarrage. Les skills non listées restent invocables via l'outil Skill |
| `memory` | Non | Source de mémoire de cet agent : `"user"`, `"project"` ou `"local"` |
| `mcpServers` | Non | Serveurs MCP disponibles pour cet agent. Chaque entrée est un nom de serveur ou un dict en ligne `{name: config}` |
| `initialPrompt` | Non | Soumis automatiquement comme premier tour utilisateur quand cet agent s'exécute comme agent du thread principal |
| `maxTurns` | Non | Nombre maximal de tours agentiques avant l'arrêt de l'agent |
| `background` | Non | Exécute cet agent comme tâche d'arrière-plan non bloquante lorsqu'il est invoqué |
| `effort` | Non | Niveau d'effort de raisonnement de cet agent. Accepte un niveau nommé ou un entier. Voir [`EffortLevel`](#effortlevel) |
| `permissionMode` | Non | Mode de permission pour l'exécution des outils au sein de cet agent. Les [règles d'héritage des sous-agents](https://code.claude.com/docs/en/agent-sdk/permissions#available-modes) déterminent quand il s'applique. Voir [`PermissionMode`](#permissionmode) |

> **Note** : les noms de champs d'`AgentDefinition` sont en camelCase, comme `disallowedTools`, `permissionMode` et `maxTurns`. Ces noms correspondent directement au format d'échange partagé avec le SDK TypeScript. Cela diffère de `ClaudeAgentOptions`, qui utilise le snake\_case Python pour les champs de premier niveau équivalents comme `disallowed_tools` et `permission_mode`. Comme `AgentDefinition` est une dataclass, passer un argument nommé en snake\_case lève une `TypeError` à la construction.

### `PermissionMode`

Modes de permission pour contrôler l'exécution des outils.

```python theme={null}
PermissionMode = Literal[
    "default",  # Comportement de permission standard
    "acceptEdits",  # Accepte automatiquement les modifications de fichiers
    "plan",  # Mode planification : explorer sans modifier
    "dontAsk",  # Refuse tout ce qui n'est pas pré-approuvé au lieu de demander
    "bypassPermissions",  # Contourne les vérifications de permission ; les règles ask explicites demandent quand même (à utiliser avec prudence)
    "auto",  # Un classifieur (modèle) examine les actions comme les commandes shell et les requêtes réseau
]
```

### `EffortLevel`

Niveaux d'effort pour orienter la profondeur de réflexion.

```python theme={null}
EffortLevel = Literal[
    "low",  # Réflexion minimale, réponses les plus rapides
    "medium",  # Réflexion modérée
    "high",  # Raisonnement approfondi
    "xhigh",  # Raisonnement étendu ; se replie sur "high" pour les modèles qui ne le prennent pas en charge
    "max",  # Effort maximal
]
```

### `CanUseTool`

Alias de type pour les fonctions de rappel de permission d'outil.

```python theme={null}
CanUseTool = Callable[
    [str, dict[str, Any], ToolPermissionContext], Awaitable[PermissionResult]
]
```

Le callback reçoit :

* `tool_name` : nom de l'outil appelé
* `input_data` : les paramètres d'entrée de l'outil
* `context` : un `ToolPermissionContext` avec des informations supplémentaires

Il renvoie un `PermissionResult` (soit `PermissionResultAllow`, soit `PermissionResultDeny`).

Le callback remplace, côté SDK, la demande de permission interactive : il n'est invoqué que quand le [flux d'évaluation des permissions](https://code.claude.com/docs/en/agent-sdk/permissions#how-permissions-are-evaluated) aboutit à une demande. Les appels d'outils déjà approuvés par une entrée `allowed_tools`, une règle allow des réglages ou le mode de permission (comme `acceptEdits` ou `bypassPermissions`) ne l'invoquent jamais. Pour contrôler chaque appel d'outil, utilisez plutôt un [hook `PreToolUse`](https://code.claude.com/docs/en/agent-sdk/hooks).

Une règle allow ne pré-approuve pas les [actions qu'aucun mode n'approuve automatiquement](https://code.claude.com/docs/en/permission-modes#actions-no-mode-auto-approves) ; voir [Comment les permissions sont évaluées](https://code.claude.com/docs/en/agent-sdk/permissions#how-permissions-are-evaluated) pour savoir lesquelles atteignent le callback et ce qui se passe en modes `dontAsk` et `auto`.

### `ToolPermissionContext`

Informations de contexte passées aux callbacks de permission d'outil.

```python theme={null}
@dataclass
class ToolPermissionContext:
    signal: Any | None = None  # À venir : prise en charge du signal d'abandon
    suggestions: list[PermissionUpdate] = field(default_factory=list)
    tool_use_id: str | None = None
    agent_id: str | None = None
    blocked_path: str | None = None
    decision_reason: str | None = None
    title: str | None = None
    display_name: str | None = None
    description: str | None = None
```

| Champ | Type | Description |
| :- | :- | :- |
| `signal` | `Any \| None` | Réservé à une future prise en charge du signal d'abandon |
| `suggestions` | `list[PermissionUpdate]` | Suggestions de mise à jour des permissions venant de la CLI. Les demandes Bash incluent une suggestion avec la destination `localSettings` ; la renvoyer dans `updated_permissions` écrit donc la règle dans `.claude/settings.local.json`, qui persiste d'une session à l'autre. |
| `tool_use_id` | `str \| None` | Identifiant de l'appel d'outil précis concerné par cette demande. Toujours renseigné lorsqu'il est transmis à `can_use_tool` |
| `agent_id` | `str \| None` | ID du sous-agent quand l'appel provient d'un sous-agent ; `None` pour l'agent principal |
| `blocked_path` | `str \| None` | Chemin de fichier qui a déclenché la demande de permission, le cas échéant. Par exemple quand une commande Bash tente d'accéder à un chemin hors des répertoires autorisés |
| `decision_reason` | `str \| None` | Raison du déclenchement de cette demande de permission. Transmise depuis le `permissionDecisionReason` d'un hook PreToolUse quand le hook a renvoyé `"ask"` |
| `title` | `str \| None` | Phrase complète de la demande de permission, comme `Claude wants to read foo.txt`. À utiliser comme texte principal de la demande lorsqu'elle est présente |
| `display_name` | `str \| None` | Groupe nominal court décrivant l'action de l'outil, comme `Read file`, adapté aux libellés de boutons |
| `description` | `str \| None` | Sous-titre lisible pour l'interface de permission |

### `PermissionResult`

Type union des résultats de callback de permission.

```python theme={null}
PermissionResult = PermissionResultAllow | PermissionResultDeny
```

### `PermissionResultAllow`

Résultat indiquant que l'appel d'outil doit être autorisé.

```python theme={null}
@dataclass
class PermissionResultAllow:
    behavior: Literal["allow"] = "allow"
    updated_input: dict[str, Any] | None = None
    updated_permissions: list[PermissionUpdate] | None = None
```

| Champ | Type | Défaut | Description |
| :- | :- | :- | :- |
| `behavior` | `Literal["allow"]` | `"allow"` | Doit valoir "allow" |
| `updated_input` | `dict[str, Any] \| None` | `None` | Entrée modifiée à utiliser à la place de l'originale |
| `updated_permissions` | `list[PermissionUpdate] \| None` | `None` | Mises à jour de permissions à appliquer |

### `PermissionResultDeny`

Résultat indiquant que l'appel d'outil doit être refusé.

```python theme={null}
@dataclass
class PermissionResultDeny:
    behavior: Literal["deny"] = "deny"
    message: str = ""
    interrupt: bool = False
```

| Champ | Type | Défaut | Description |
| :- | :- | :- | :- |
| `behavior` | `Literal["deny"]` | `"deny"` | Doit valoir "deny" |
| `message` | `str` | `""` | Message expliquant pourquoi l'outil a été refusé |
| `interrupt` | `bool` | `False` | Indique s'il faut interrompre l'exécution en cours |

### `PermissionUpdate`

Configuration pour mettre à jour les permissions par programmation.

```python theme={null}
@dataclass
class PermissionUpdate:
    type: Literal[
        "addRules",
        "replaceRules",
        "removeRules",
        "setMode",
        "addDirectories",
        "removeDirectories",
    ]
    rules: list[PermissionRuleValue] | None = None
    behavior: Literal["allow", "deny", "ask"] | None = None
    mode: PermissionMode | None = None
    directories: list[str] | None = None
    destination: (
        Literal["userSettings", "projectSettings", "localSettings", "session"] | None
    ) = None
```

| Champ | Type | Description |
| :- | :- | :- |
| `type` | `Literal[...]` | Le type d'opération de mise à jour des permissions |
| `rules` | `list[PermissionRuleValue] \| None` | Règles pour les opérations d'ajout/remplacement/suppression |
| `behavior` | `Literal["allow", "deny", "ask"] \| None` | Comportement pour les opérations basées sur des règles |
| `mode` | `PermissionMode \| None` | Mode pour l'opération setMode |
| `directories` | `list[str] \| None` | Répertoires pour les opérations d'ajout/suppression de répertoires |
| `destination` | `Literal[...] \| None` | Où appliquer la mise à jour des permissions |

### `PermissionRuleValue`

Une règle à ajouter, remplacer ou supprimer dans une mise à jour de permissions.

```python theme={null}
@dataclass
class PermissionRuleValue:
    tool_name: str
    rule_content: str | None = None
```

### `ToolsPreset`

Configuration d'outils prédéfinie pour utiliser l'ensemble d'outils par défaut de Claude Code.

```python theme={null}
class ToolsPreset(TypedDict):
    type: Literal["preset"]
    preset: Literal["claude_code"]
```

### `ThinkingConfig`

Contrôle le comportement de la réflexion étendue. Union de trois configurations :

```python theme={null}
ThinkingDisplay = Literal["summarized", "omitted"]


class ThinkingConfigAdaptive(TypedDict):
    type: Literal["adaptive"]
    display: NotRequired[ThinkingDisplay]


class ThinkingConfigEnabled(TypedDict):
    type: Literal["enabled"]
    budget_tokens: int
    display: NotRequired[ThinkingDisplay]


class ThinkingConfigDisabled(TypedDict):
    type: Literal["disabled"]


ThinkingConfig = ThinkingConfigAdaptive | ThinkingConfigEnabled | ThinkingConfigDisabled
```

| Variante | Champs | Description |
| :- | :- | :- |
| `adaptive` | `type`, `display` | Claude décide de manière adaptative quand réfléchir |
| `enabled` | `type`, `budget_tokens`, `display` | Active la réflexion avec un budget de tokens précis |
| `disabled` | `type` | Désactive la réflexion |

Le champ facultatif `display` détermine si le texte de réflexion est renvoyé `"summarized"` (résumé) ou `"omitted"` (omis). Sur Claude Opus 4.7 et ultérieur, le défaut de l'API est `"omitted"` : définissez `"summarized"` pour recevoir le contenu de réflexion dans les sorties [`ThinkingBlock`](#thinkingblock). Claude Code ne transmet pas votre valeur `display` à certains fournisseurs, comme Amazon Bedrock et Agent Platform de Google Cloud. Chez ces fournisseurs, Opus 4.7 et ultérieur renvoient des `ThinkingBlock` vides même si vous définissez `display` à `"summarized"`.

Comme ce sont des classes `TypedDict`, ce sont de simples dicts à l'exécution. Construisez-les soit comme des littéraux de dict, soit en appelant la classe comme un constructeur ; les deux produisent un `dict`. Accédez aux champs avec `config["budget_tokens"]`, et non `config.budget_tokens` :

```python theme={null}
from claude_agent_sdk import ClaudeAgentOptions, ThinkingConfigEnabled

# Option 1 : littéral de dict (recommandé, aucun import nécessaire)
options = ClaudeAgentOptions(thinking={"type": "enabled", "budget_tokens": 20000})

# Option 2 : style constructeur (renvoie un simple dict)
config = ThinkingConfigEnabled(type="enabled", budget_tokens=20000)
print(config["budget_tokens"])  # 20000
# config.budget_tokens lèverait AttributeError
```

### `TaskBudget`

Budget de tâche côté API, en tokens, utilisé avec le champ `task_budget` de `ClaudeAgentOptions`.

```python theme={null}
class TaskBudget(TypedDict):
    total: int
```

| Champ | Type | Description |
| :- | :- | :- |
| `total` | `int` | Budget total de tokens pour la tâche |

Comme c'est un `TypedDict`, passez-le sous forme de simple dict, par exemple `ClaudeAgentOptions(task_budget={"total": 50000})`.

### `SdkBeta`

Type littéral des fonctionnalités bêta du SDK.

```python theme={null}
SdkBeta = Literal["context-1m-2025-08-07"]
```

À utiliser avec le champ `betas` de `ClaudeAgentOptions` pour activer des fonctionnalités bêta.

> **Avertissement** : sur l'API Claude, la bêta `context-1m-2025-08-07` est retirée pour Claude Sonnet 4.5 et Claude Sonnet 4. Si vous la passez encore avec l'un de ces modèles, les requêtes qui dépassent la fenêtre de contexte standard de 200K tokens renvoient une erreur : retirez-la donc de `betas`. Pour exécuter une session avec une fenêtre de contexte de 1M tokens, définissez `model` sur un modèle qui [fonctionne avec la fenêtre 1M par défaut](https://code.claude.com/docs/en/model-config#extended-context), comme `claude-sonnet-5-5` ou `claude-opus-5-5`. Pour un modèle qui n'atteint 1M que via sa variante `[1m]`, ajoutez le suffixe à l'ID du modèle, comme dans `claude-opus-4-6[1m]`.

### `McpSdkServerConfig`

Configuration des serveurs MCP du SDK créés avec `create_sdk_mcp_server()`.

```python theme={null}
class McpSdkServerConfig(TypedDict):
    type: Literal["sdk"]
    name: str
    instance: Any  # Instance de serveur MCP
```

### `McpServerConfig`

Type union des configurations de serveurs MCP.

```python theme={null}
McpServerConfig = (
    McpStdioServerConfig | McpSSEServerConfig | McpHttpServerConfig | McpSdkServerConfig
)
```

#### `McpStdioServerConfig`

```python theme={null}
class McpStdioServerConfig(TypedDict):
    type: NotRequired[Literal["stdio"]]  # Facultatif pour la rétrocompatibilité
    command: str
    args: NotRequired[list[str]]
    env: NotRequired[dict[str, str]]
```

#### `McpSSEServerConfig`

```python theme={null}
class McpSSEServerConfig(TypedDict):
    type: Literal["sse"]
    url: str
    headers: NotRequired[dict[str, str]]
```

#### `McpHttpServerConfig`

```python theme={null}
class McpHttpServerConfig(TypedDict):
    type: Literal["http"]
    url: str
    headers: NotRequired[dict[str, str]]
```

### `McpServerStatusConfig`

La configuration d'un serveur MCP telle que rapportée par [`get_mcp_status()`](#methods). C'est l'union de toutes les variantes de transport de [`McpServerConfig`](#mcpserverconfig), plus une variante `claudeai-proxy` en sortie seulement, pour les serveurs relayés par claude.ai.

```python theme={null}
McpServerStatusConfig = (
    McpStdioServerConfig
    | McpSSEServerConfig
    | McpHttpServerConfig
    | McpSdkServerConfigStatus
    | McpClaudeAIProxyServerConfig
)
```

`McpSdkServerConfigStatus` est la forme sérialisable de [`McpSdkServerConfig`](#mcpsdkserverconfig), avec uniquement les champs `type` (`"sdk"`) et `name` (`str`) ; l'`instance` en processus est omise. `McpClaudeAIProxyServerConfig` a les champs `type` (`"claudeai-proxy"`), `url` (`str`) et `id` (`str`).

### `McpStatusResponse`

Réponse de [`ClaudeSDKClient.get_mcp_status()`](#methods). Enveloppe la liste des états de serveurs sous la clé `mcpServers`.

```python theme={null}
class McpStatusResponse(TypedDict):
    mcpServers: list[McpServerStatus]
```

### `McpServerStatus`

État d'un serveur MCP connecté, contenu dans [`McpStatusResponse`](#mcpstatusresponse).

```python theme={null}
class McpServerStatus(TypedDict):
    name: str
    status: McpServerConnectionStatus  # "connected" | "failed" | "needs-auth" | "pending" | "disabled"
    serverInfo: NotRequired[McpServerInfo]
    error: NotRequired[str]
    config: NotRequired[McpServerStatusConfig]
    scope: NotRequired[str]
    tools: NotRequired[list[McpToolInfo]]
```

| Champ | Type | Description |
| :- | :- | :- |
| `name` | `str` | Nom du serveur |
| `status` | `str` | L'une des valeurs `"connected"`, `"failed"`, `"needs-auth"`, `"pending"` ou `"disabled"` |
| `serverInfo` | `dict` (facultatif) | Nom et version du serveur (`{"name": str, "version": str}`) |
| `error` | `str` (facultatif) | Message d'erreur si la connexion au serveur a échoué |
| `config` | [`McpServerStatusConfig`](#mcpserverstatusconfig) (facultatif) | Configuration du serveur. Même forme que [`McpServerConfig`](#mcpserverconfig) (stdio, SSE, HTTP ou SDK), plus une variante `claudeai-proxy` pour les serveurs connectés via claude.ai |
| `scope` | `str` (facultatif) | Portée de la configuration |
| `tools` | `list` (facultatif) | Outils fournis par ce serveur, chacun avec les champs `name`, `description` et `annotations` |

### `ContextUsageResponse`

Réponse de [`ClaudeSDKClient.get_context_usage()`](#methods). C'est la même charge utile que celle que Claude Code affiche pour la commande `/context` en session interactive : en plus des comptes de tokens, elle porte donc des champs d'affichage comme `color` et `gridRows` que Claude Code utilise pour dessiner la grille d'utilisation de `/context`.

Claude Code construit cette charge utile en envoyant plusieurs requêtes à l'API de [comptage de tokens](https://platform.claude.com/docs/en/build-with-claude/token-counting). Ces requêtes n'apparaissent pas dans le flux de messages, donc un suivi des coûts qui lit le flux ne les verra pas. Sur l'API Anthropic, le comptage de tokens n'est pas facturé.

```python theme={null}
class ContextUsageResponse(TypedDict):
    categories: list[ContextUsageCategory]
    totalTokens: int
    maxTokens: int
    rawMaxTokens: int
    percentage: float
    model: str
    isAutoCompactEnabled: bool
    memoryFiles: list[dict[str, Any]]
    mcpTools: list[dict[str, Any]]
    agents: list[dict[str, Any]]
    gridRows: list[list[dict[str, Any]]]
    autoCompactThreshold: NotRequired[int]
    deferredBuiltinTools: NotRequired[list[dict[str, Any]]]
    systemTools: NotRequired[list[dict[str, Any]]]
    systemPromptSections: NotRequired[list[dict[str, Any]]]
    slashCommands: NotRequired[dict[str, Any]]
    skills: NotRequired[dict[str, Any]]  # utilisation des skills avec répartition du frontmatter
    messageBreakdown: NotRequired[dict[str, Any]]  # tokens des messages par type
    apiUsage: NotRequired[dict[str, Any] | None]
```

Chaque entrée `ContextUsageCategory` porte `name`, `tokens`, `color` et un drapeau facultatif `isDeferred`. `totalTokens` est l'utilisation actuelle du contexte par la session, et `maxTokens` est la fenêtre par rapport à laquelle cette utilisation est mesurée. Cette fenêtre est la fenêtre de contexte du modèle, ou la fenêtre d'auto-compaction plus basse quand elle s'applique ; `rawMaxTokens` porte la même valeur que `maxTokens`. `apiUsage` contient l'utilisation de la dernière réponse de l'API, pas un total cumulé de la session. Claude Code laisse non définies les clés facultatives `deferredBuiltinTools`, `systemTools` et `systemPromptSections` : attendez-vous donc à leur absence même si le type les déclare.

### `SdkPluginConfig`

Configuration du chargement de plugins dans le SDK.

```python theme={null}
class SdkPluginConfig(TypedDict):
    type: Literal["local"]
    path: str
```

| Champ | Type | Description |
| :- | :- | :- |
| `type` | `Literal["local"]` | Doit valoir `"local"` (seuls les plugins locaux sont actuellement pris en charge) |
| `path` | `str` | Chemin absolu ou relatif vers le répertoire du plugin |

**Exemple :**

```python theme={null}
plugins = [
    {"type": "local", "path": "./my-plugin"},
    {"type": "local", "path": "/absolute/path/to/plugin"},
]
```

Pour des informations complètes sur la création et l'utilisation de plugins, voir [Plugins](https://code.claude.com/docs/en/agent-sdk/plugins).

## Types de messages (Message Types)

### `Message`

Type union de tous les messages possibles.

```python theme={null}
Message = (
    UserMessage
    | AssistantMessage
    | SystemMessage
    | ResultMessage
    | StreamEvent
    | RateLimitEvent
    | ConversationResetMessage
)
```

### `UserMessage`

Message d'entrée utilisateur.

```python theme={null}
@dataclass
class UserMessage:
    content: str | list[ContentBlock]
    uuid: str | None = None
    parent_tool_use_id: str | None = None
    tool_use_result: dict[str, Any] | None = None
    origin: MessageOrigin | None = None
```

| Champ | Type | Description |
| :- | :- | :- |
| `content` | `str \| list[ContentBlock]` | Contenu du message, sous forme de texte ou de blocs de contenu |
| `uuid` | `str \| None` | Identifiant unique du message |
| `parent_tool_use_id` | `str \| None` | ID de l'utilisation d'outil si ce message est une réponse de résultat d'outil |
| `tool_use_result` | `dict[str, Any] \| None` | Données de résultat d'outil, le cas échéant |
| `origin` | `MessageOrigin \| None` | Provenance de ce message, renseignée sur les tours injectés comme les notifications de tâche et les messages de pairs. `None` quand la CLI ne l'a pas attribué. Exige le SDK Agent Python 0.2.137 ou ultérieur |

Le SDK transmet `tool_use_result` depuis la CLI sans modification. Pour un outil d'un serveur MCP externe dont le résultat contient des blocs `resource_link`, le dict a une clé `resourceLinks` contenant une liste de dicts ayant les clés du type TypeScript [`SDKMcpResourceLink`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkmcpresourcelink). Claude reçoit chaque lien sous forme d'une ligne de texte dans le résultat de l'outil. Pour afficher les fichiers renvoyés par le serveur, lisez `resourceLinks` au lieu d'analyser ce texte. La clé `resourceLinks` exige le SDK Agent Python 0.2.150+ et Claude Code v2.1.257+ ; la CLI fournie avec cette version du SDK satisfait l'exigence Claude Code.

La CLI omet la clé quand le résultat ne contient aucun lien, ainsi que sur les résultats des sous-agents. La CLI garde au maximum 50 liens par résultat et cesse d'en ajouter dès que la liste atteint 64 Kio de JSON sérialisé. Un outil que vous définissez en processus avec [`tool()`](#tool) ne produit jamais cette clé, car le SDK aplatit ses blocs `resource_link` en texte avant que la CLI ne voie le résultat.

### `AssistantMessage`

Message de réponse de l'assistant, avec des blocs de contenu.

```python theme={null}
@dataclass
class AssistantMessage:
    content: list[ContentBlock]
    model: str
    parent_tool_use_id: str | None = None
    error: AssistantMessageError | None = None
    usage: dict[str, Any] | None = None
    message_id: str | None = None
    stop_reason: str | None = None
    session_id: str | None = None
    uuid: str | None = None
```

| Champ | Type | Description |
| :- | :- | :- |
| `content` | `list[ContentBlock]` | Liste des blocs de contenu de la réponse |
| `model` | `str` | Modèle qui a généré la réponse |
| `parent_tool_use_id` | `str \| None` | ID de l'utilisation d'outil s'il s'agit d'une réponse imbriquée |
| `error` | [`AssistantMessageError`](#assistantmessageerror) ` \| None` | Type d'erreur si la réponse a rencontré une erreur |
| `usage` | `dict[str, Any] \| None` | Utilisation de tokens par message (mêmes clés que [`ResultMessage.usage`](#resultmessage)) |
| `message_id` | `str \| None` | ID de message de l'API. Plusieurs messages d'un même tour partagent le même ID |
| `stop_reason` | `str \| None` | Raison d'arrêt renvoyée par l'API (par exemple `end_turn`, `tool_use`) |
| `session_id` | `str \| None` | ID de la session à laquelle appartient ce message |
| `uuid` | `str \| None` | Identifiant unique du message dans la transcription de la session |

### `AssistantMessageError`

Types d'erreur possibles pour les messages de l'assistant.

```python theme={null}
AssistantMessageError = Literal[
    "authentication_failed",
    "billing_error",
    "rate_limit",
    "invalid_request",
    "server_error",
    "unknown",
]
```

Le processus CLI sous-jacent peut émettre des types d'erreur que ce Literal ne liste pas, comme `max_output_tokens`. Le SDK transmet la valeur sans modification : traitez donc les chaînes hors de cette liste comme `unknown`. Le type TypeScript [`SDKAssistantMessageError`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkassistantmessage) liste l'ensemble complet des valeurs que la CLI peut émettre.

### `SystemMessage`

Message système avec métadonnées.

```python theme={null}
@dataclass
class SystemMessage:
    subtype: str
    data: dict[str, Any]
```

Les sous-types qui n'ont pas leur propre dataclass arrivent sous forme de `SystemMessage`. Pour suivre la session entre les tours, définissez [`CLAUDE_CODE_EMIT_SESSION_STATE_EVENTS=1`](https://code.claude.com/docs/en/env-vars#variables) et lisez `message.data["state"]` sur chaque message dont le `subtype` est `session_state_changed`. [`SDKSessionStateChangedMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdksessionstatechangedmessage) liste les états qu'il peut porter. Itérez avec `receive_messages()` pour les lire : `receive_response()` s'arrête au `ResultMessage`, et un message `session_state_changed` peut suivre ce résultat.

### `ResultMessage`

Message de résultat final, avec les informations de coût et d'utilisation.

```python theme={null}
@dataclass
class ResultMessage:
    subtype: str
    duration_ms: int
    duration_api_ms: int
    is_error: bool
    num_turns: int
    session_id: str
    stop_reason: str | None = None
    total_cost_usd: float | None = None
    usage: dict[str, Any] | None = None
    result: str | None = None
    structured_output: Any = None
    model_usage: dict[str, ModelUsage] | None = None
    permission_denials: list[Any] | None = None
    deferred_tool_use: DeferredToolUse | None = None
    errors: list[str] | None = None
    api_error_status: int | None = None
    uuid: str | None = None
    terminal_reason: str | None = None
    origin: MessageOrigin | None = None
```

Le champ `subtype` détermine quels autres champs sont renseignés. Il vaut l'un de `"success"`, `"error_during_execution"`, `"error_max_turns"`, `"error_max_budget_usd"` ou `"error_max_structured_output_retries"`. La dataclass Python aplatit toutes les variantes en une seule forme, donc les champs qui ne s'appliquent pas au sous-type renvoyé valent `None`.

Plusieurs champs portent des détails de diagnostic sur la façon dont la conversation s'est terminée :

* `is_error` : `True` quand la conversation s'est terminée dans un état d'erreur. Toujours `True` pour les sous-types `error_*`. Avec `subtype="success"`, il vaut `True` quand la dernière requête au modèle a échoué, c'est-à-dire que la boucle d'agent s'est achevée mais que le dernier appel API a renvoyé une erreur.
* `api_error_status` : le code d'état HTTP de l'erreur API qui a mis fin au tour. `None` quand le tour s'est terminé sans erreur de ce type. Renseigné uniquement avec `subtype="success"`.
* `result` : texte du dernier message de l'assistant avec `subtype="success"`, ou `None` pour les sous-types `error_*`. Quand `subtype="success"` et `is_error=True`, il contient la chaîne d'erreur de l'API si elle est disponible, mais peut être vide : consultez donc `api_error_status` et le contenu de l'`AssistantMessage` précédent pour les détails.
* `errors` : chaînes d'erreur au niveau de la boucle, comme le message de nombre maximal de tours. Renseigné uniquement pour les sous-types `error_*`.
* `terminal_reason` : pourquoi la boucle de requête s'est terminée, par exemple `"completed"`, `"max_turns"`, `"api_error"`, `"aborted_streaming"` ou `"aborted_tools"`. Une valeur `"aborted_streaming"` ou `"aborted_tools"` signifie que le tour a été abandonné avant de se terminer. Causes fréquentes : [`interrupt()`](#claudesdkclient) et un callback de permission renvoyant [`PermissionResultDeny`](#permissionresultdeny) avec `interrupt=True`. `None` sur les versions de la CLI antérieures à ce champ, sur les résultats des commandes locales comme `/voice` ou `/usage` (qui contournent la boucle de requête), ou sur les résultats d'erreur synthétisés émis quand la session échoue de façon fatale. Reflète [`SDKResultMessage.terminal_reason`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkresultmessage) du SDK TypeScript, qui liste l'ensemble complet des valeurs.
* `origin` : origine du message utilisateur qui a déclenché ce tour. En [mode d'entrée en streaming](https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode), vérifiez-le pour distinguer le résultat de votre propre prompt (où `origin` vaut `None` ou `{"kind": "human"}`) du résultat d'un tour injecté, comme une notification de tâche d'arrière-plan. Exige le SDK Agent Python 0.2.137 ou ultérieur.

Quand plusieurs tâches d'arrière-plan se terminent à peu d'intervalle, Claude Code peut répondre à leurs notifications en un seul tour plutôt qu'en un tour chacune. Vous recevez quand même un `ResultMessage` par notification, dans l'ordre, chacun avec un `origin` dont le `kind` vaut `"task-notification"`. Tous sauf le dernier ont `num_turns` à `0` et un `result` vide, et le dernier porte le tour qui répond à toutes.

Le dict `usage` ne couvre que la boucle d'agent principale et exclut les sous-agents et les autres appels de modèle imbriqués ou auxiliaires. En [mode d'entrée en streaming](https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode), les valeurs sont par tour. Préférez `model_usage` pour la comptabilité des tokens et des coûts. Le dict `usage` contient les clés suivantes lorsqu'elles sont présentes :

| Clé | Type | Description |
| - | - | - |
| `input_tokens` | `int` | Tokens d'entrée consommés par la boucle d'agent de premier niveau. [Les tokens des sous-agents ne sont pas inclus](https://code.claude.com/docs/en/agent-sdk/cost-tracking#get-the-total-cost-of-a-query) ; utilisez `model_usage` pour une comptabilité de tout l'arbre. |
| `output_tokens` | `int` | Tokens de sortie générés par la boucle d'agent de premier niveau. Les tokens des sous-agents ne sont pas inclus. |
| `cache_creation_input_tokens` | `int` | Tokens utilisés pour créer de nouvelles entrées de cache. |
| `cache_read_input_tokens` | `int` | Tokens lus depuis des entrées de cache existantes. |

Le dict `model_usage` associe les noms de modèles à leur utilisation par modèle. Il couvre tous les appels de modèle faits via le pipeline de requête : la boucle principale, les sous-agents et les appels internes comme la compaction et les agents Workflow. Les appels auxiliaires hors de ce pipeline, comme le classifieur de permissions et les requêtes de comptage de tokens, sont exclus de `model_usage`. Considérez `model_usage` comme une estimation, pas comme un relevé de facturation.

En [mode d'entrée en streaming](https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode), `model_usage` et `total_cost_usd` sont cumulatifs d'un tour à l'autre : lisez donc le dernier résultat plutôt que d'additionner les résultats. Un appel qui reprend une session compte aussi les [totaux restaurés depuis les appels précédents de la session](https://code.claude.com/docs/en/agent-sdk/cost-tracking#accumulate-costs-across-multiple-calls). Voir [Suivre les coûts en mode d'entrée en streaming](https://code.claude.com/docs/en/agent-sdk/cost-tracking#track-costs-in-streaming-input-mode) pour les réinitialisations et [Récupérer les totaux après un plantage de session](https://code.claude.com/docs/en/agent-sdk/cost-tracking#recover-totals-after-a-session-crash) pour les résultats remis à zéro.

Chaque valeur de `model_usage` est un TypedDict `ModelUsage`, importé via `from claude_agent_sdk.types import ModelUsage`. Ses clés sont en camelCase car le SDK transmet la valeur sans modification depuis le processus CLI sous-jacent, conformément au type TypeScript [`ModelUsage`](https://code.claude.com/docs/en/agent-sdk/typescript#modelusage) :

| Clé | Type | Description |
| - | - | - |
| `inputTokens` | `int` | Tokens d'entrée pour ce modèle. |
| `outputTokens` | `int` | Tokens de sortie pour ce modèle. |
| `cacheReadInputTokens` | `int` | Tokens lus depuis le cache pour ce modèle. |
| `cacheCreationInputTokens` | `int` | Tokens de création de cache pour ce modèle. |
| `webSearchRequests` | `int` | Requêtes de recherche web faites par ce modèle. |
| `thinkingTokens` | `int` | Tokens de réflexion générés par ce modèle, déjà comptés dans `outputTokens`. Absent tant qu'un tour n'a pas tourné sur une version de Claude Code qui l'enregistre, et non déclaré dans le TypedDict : lisez-le donc avec `.get()`. Exige le SDK Agent Python 0.2.150 ou ultérieur, dont la CLI fournie l'enregistre. |
| `costUSD` | `float` | Coût estimé en USD pour ce modèle, calculé côté client. Voir [Suivre le coût et l'utilisation](https://code.claude.com/docs/en/agent-sdk/cost-tracking) pour les réserves sur la facturation. |
| `contextWindow` | `int` | Taille de la fenêtre de contexte de ce modèle. |
| `maxOutputTokens` | `int` | Limite maximale de tokens de sortie pour ce modèle. |
| `canonicalModel` | `str` | ID de modèle canonique utilisé pour la recherche de prix. Peut différer de la chaîne de modèle brute qui sert de clé à l'entrée, comme un ID propre à un fournisseur ou un alias. Pas toujours présent. |
| `provider` | `str` | Fournisseur d'API qui a servi ce modèle, comme `firstParty`, `bedrock`, `vertex`, `foundry`, `anthropicAws`, `mantle` ou `gateway`. Pas toujours présent. |
| `costBasis` | `str` | Grille tarifaire ayant servi à chiffrer la dernière requête de ce modèle : `list` pour le prix catalogue, `managed` pour une table [`modelPricing`](https://code.claude.com/docs/en/settings-reference#modelpricing), ou `unknown` quand aucune ne correspondait à l'ID du modèle. Pas toujours présent, et non déclaré dans le TypedDict : lisez-le donc avec `.get()`. Exige Claude Code v2.1.246 ou ultérieur. |

### `StreamEvent`

Événement de flux pour les mises à jour partielles de messages pendant le streaming. Reçu uniquement quand `include_partial_messages=True` dans `ClaudeAgentOptions`. Import via `from claude_agent_sdk.types import StreamEvent`.

```python theme={null}
@dataclass
class StreamEvent:
    uuid: str
    session_id: str
    event: dict[str, Any]  # L'événement de flux brut de l'API Claude
    parent_tool_use_id: str | None = None
```

| Champ | Type | Description |
| :- | :- | :- |
| `uuid` | `str` | Identifiant unique de cet événement |
| `session_id` | `str` | Identifiant de la session |
| `event` | `dict[str, Any]` | Les données brutes de l'événement de flux de l'API Claude |
| `parent_tool_use_id` | `str \| None` | Toujours `None`. Les événements de flux ne sont émis que pour la session principale. Pour attribuer un message à un sous-agent, utilisez les messages complets comme [`AssistantMessage`](#assistantmessage) |

### `RateLimitEvent`

Émis quand l'état de la limite de débit change (par exemple de `"allowed"` à `"allowed_warning"`). Utilisez-le pour avertir les utilisateurs avant qu'ils n'atteignent une limite stricte, ou pour ralentir quand l'état est `"rejected"`.

```python theme={null}
@dataclass
class RateLimitEvent:
    rate_limit_info: RateLimitInfo
    uuid: str
    session_id: str
```

| Champ | Type | Description |
| :- | :- | :- |
| `rate_limit_info` | [`RateLimitInfo`](#ratelimitinfo) | État actuel de la limite de débit |
| `uuid` | `str` | Identifiant unique de l'événement |
| `session_id` | `str` | Identifiant de la session |

### `RateLimitInfo`

État de limite de débit porté par [`RateLimitEvent`](#ratelimitevent).

```python theme={null}
RateLimitStatus = Literal["allowed", "allowed_warning", "rejected"]
RateLimitType = Literal[
    "five_hour", "seven_day", "seven_day_opus", "seven_day_sonnet", "overage"
]


@dataclass
class RateLimitInfo:
    status: RateLimitStatus
    resets_at: int | None = None
    rate_limit_type: RateLimitType | None = None
    utilization: float | None = None
    overage_status: RateLimitStatus | None = None
    overage_resets_at: int | None = None
    overage_disabled_reason: str | None = None
    raw: dict[str, Any] = field(default_factory=dict)
```

| Champ | Type | Description |
| :- | :- | :- |
| `status` | `RateLimitStatus` | État actuel, l'un de `"allowed"`, `"allowed_warning"` ou `"rejected"`. `"allowed_warning"` signifie que la limite approche ; `"rejected"` signifie que la limite a été atteinte |
| `resets_at` | `int \| None` | Horodatage Unix de réinitialisation de la fenêtre de limite de débit |
| `rate_limit_type` | `RateLimitType \| None` | Quelle fenêtre de limite de débit s'applique |
| `utilization` | `float \| None` | Fraction de la limite de débit consommée (0.0 à 1.0) |
| `overage_status` | `RateLimitStatus \| None` | État de l'utilisation en dépassement payée à l'usage (overage), le cas échéant |
| `overage_resets_at` | `int \| None` | Horodatage Unix de réinitialisation de la fenêtre de dépassement |
| `overage_disabled_reason` | `str \| None` | Pourquoi le dépassement n'est pas disponible, si l'état est `"rejected"` |
| `raw` | `dict[str, Any]` | Dict brut complet venant de la CLI, y compris les champs non modélisés ci-dessus |

### `ConversationResetMessage`

Émis quand la conversation est remplacée sans mettre fin à la connexion, par exemple après `/clear`. Voir [Suivre les coûts en mode d'entrée en streaming](https://code.claude.com/docs/en/agent-sdk/cost-tracking#track-costs-in-streaming-input-mode) pour l'effet d'une réinitialisation sur les totaux cumulés des `ResultMessage` suivants. Exige le SDK Agent Python 0.2.137 ou ultérieur.

```python theme={null}
@dataclass
class ConversationResetMessage:
    new_conversation_id: str
    uuid: str
    session_id: str
```

| Champ | Type | Description |
| :- | :- | :- |
| `new_conversation_id` | `str` | Identifiant opaque de la nouvelle conversation. Ce n'est pas le `session_id` des messages suivants ; lisez celui-ci sur le message suivant |
| `uuid` | `str` | Identifiant unique du message |
| `session_id` | `str` | ID de la session qui a été réinitialisée. Les messages après la réinitialisation portent un nouveau `session_id` |

### `TaskStartedMessage`

Émis quand une tâche d'arrière-plan démarre. Une tâche d'arrière-plan est tout ce qui est suivi en dehors du tour principal : une commande Bash passée en arrière-plan, une surveillance [Monitor](#monitor), un sous-agent lancé via l'outil Agent, ou un agent distant. Le champ `task_type` indique lequel. Ce nommage n'a aucun rapport avec le renommage de l'outil `Task` en `Agent`.

```python theme={null}
@dataclass
class TaskStartedMessage(SystemMessage):
    task_id: str
    description: str
    uuid: str
    session_id: str
    tool_use_id: str | None = None
    task_type: str | None = None
```

| Champ | Type | Description |
| :- | :- | :- |
| `task_id` | `str` | Identifiant unique de la tâche |
| `description` | `str` | Description de la tâche |
| `uuid` | `str` | Identifiant unique du message |
| `session_id` | `str` | Identifiant de la session |
| `tool_use_id` | `str \| None` | ID de l'utilisation d'outil associée |
| `task_type` | `str \| None` | Type de tâche d'arrière-plan : `"local_bash"` pour les commandes Bash d'arrière-plan et les surveillances Monitor, `"local_agent"` ou `"remote_agent"` |

### `TaskUsage`

Données de tokens et de durée d'une tâche d'arrière-plan.

```python theme={null}
class TaskUsage(TypedDict):
    total_tokens: int
    tool_uses: int
    duration_ms: int
```

### `TaskProgressMessage`

Émis périodiquement avec des mises à jour de progression pour une tâche d'arrière-plan en cours.

```python theme={null}
@dataclass
class TaskProgressMessage(SystemMessage):
    task_id: str
    description: str
    usage: TaskUsage
    uuid: str
    session_id: str
    tool_use_id: str | None = None
    last_tool_name: str | None = None
```

| Champ | Type | Description |
| :- | :- | :- |
| `task_id` | `str` | Identifiant unique de la tâche |
| `description` | `str` | Description de l'état actuel |
| `usage` | `TaskUsage` | Utilisation de tokens de cette tâche jusqu'ici |
| `uuid` | `str` | Identifiant unique du message |
| `session_id` | `str` | Identifiant de la session |
| `tool_use_id` | `str \| None` | ID de l'utilisation d'outil associée |
| `last_tool_name` | `str \| None` | Nom du dernier outil utilisé par la tâche |

### `TaskNotificationMessage`

Émis quand une tâche d'arrière-plan se termine, échoue ou est arrêtée. Les tâches d'arrière-plan incluent les commandes Bash `run_in_background`, les surveillances Monitor et les sous-agents d'arrière-plan.

```python theme={null}
@dataclass
class TaskNotificationMessage(SystemMessage):
    task_id: str
    status: TaskNotificationStatus  # "completed" | "failed" | "stopped"
    output_file: str
    summary: str
    uuid: str
    session_id: str
    tool_use_id: str | None = None
    usage: TaskUsage | None = None
```

| Champ | Type | Description |
| :- | :- | :- |
| `task_id` | `str` | Identifiant unique de la tâche |
| `status` | `TaskNotificationStatus` | L'un de `"completed"`, `"failed"` ou `"stopped"` |
| `output_file` | `str` | Chemin vers le fichier de sortie de la tâche |
| `summary` | `str` | Résumé du résultat de la tâche |
| `uuid` | `str` | Identifiant unique du message |
| `session_id` | `str` | Identifiant de la session |
| `tool_use_id` | `str \| None` | ID de l'utilisation d'outil associée |
| `usage` | `TaskUsage \| None` | Utilisation de tokens finale de la tâche |

Quand la CLI [passe un long appel d'outil MCP en arrière-plan](https://code.claude.com/docs/en/mcp#automatic-backgrounding-of-long-tool-calls), le résultat d'outil de cet appel ne contient qu'un espace réservé et le vrai résultat arrive dans ce message. Sur une notification `"completed"` pour un tel appel, la CLI ajoute une clé `resource_links` listant les fichiers renvoyés par référence par l'outil, avec les mêmes entrées et limites que la clé `resourceLinks` de [`UserMessage.tool_use_result`](#usermessage). La clé `resource_links` exige le SDK Agent Python 0.2.150+ et Claude Code v2.1.257+ ; la CLI fournie avec cette version du SDK satisfait l'exigence Claude Code.

La dataclass n'a pas de champ pour `resource_links`. Lisez-le depuis le dict `data` que le message hérite de [`SystemMessage`](#systemmessage) : `message.data.get("resource_links")`. Associez la notification à l'appel grâce à `tool_use_id`. La CLI omet la clé quand le résultat ne contenait aucun lien, ainsi que sur les notifications de tâches qui ne sont pas des appels d'outils MCP.

## Types de blocs de contenu (Content Block Types)

### `ContentBlock`

Type union de tous les blocs de contenu.

```python theme={null}
ContentBlock = (
    TextBlock
    | ThinkingBlock
    | ToolUseBlock
    | ToolResultBlock
    | ServerToolUseBlock
    | ServerToolResultBlock
)
```

### `TextBlock`

Bloc de contenu texte.

```python theme={null}
@dataclass
class TextBlock:
    text: str
```

### `ThinkingBlock`

Bloc de contenu de réflexion (pour les modèles dotés de la capacité de réflexion).

```python theme={null}
@dataclass
class ThinkingBlock:
    thinking: str
    signature: str
```

### `ToolUseBlock`

Bloc de demande d'utilisation d'outil.

```python theme={null}
@dataclass
class ToolUseBlock:
    id: str
    name: str
    input: dict[str, Any]
```

### `ToolResultBlock`

Bloc de résultat d'exécution d'outil.

```python theme={null}
@dataclass
class ToolResultBlock:
    tool_use_id: str
    content: str | list[dict[str, Any]] | None = None
    is_error: bool | None = None
```

## Types d'erreurs (Error Types)

Les types ci-dessous définissent ce que votre code intercepte. Pour des entrées classées selon les messages d'erreur levés par ces types, avec la cause et la correction de chacun, voir [Dépannage](https://code.claude.com/docs/en/agent-sdk/troubleshooting).

### `ClaudeSDKError`

Classe d'exception de base pour toutes les erreurs du SDK.

```python theme={null}
class ClaudeSDKError(Exception):
    """Base error for Claude SDK."""
```

Quand un `query()` ponctuel se termine par un résultat d'erreur, par exemple une erreur de limite de tours, le SDK lève une [`ResultError`](#resulterror).

### `CLINotFoundError`

Levée quand la CLI Claude Code n'est pas installée ou introuvable.

```python theme={null}
class CLINotFoundError(CLIConnectionError):
    def __init__(
        self, message: str = "Claude Code not found", cli_path: str | None = None
    ):
        """
        Args:
            message: Error message (default: "Claude Code not found")
            cli_path: Optional path to the CLI that was not found
        """
```

### `CLIConnectionError`

Levée quand la connexion à Claude Code échoue.

```python theme={null}
class CLIConnectionError(ClaudeSDKError):
    """Failed to connect to Claude Code."""
```

### `ProcessError`

Levée quand le processus Claude Code échoue.

```python theme={null}
class ProcessError(ClaudeSDKError):
    def __init__(
        self, message: str, exit_code: int | None = None, stderr: str | None = None
    ):
        self.exit_code = exit_code
        self.stderr = stderr
```

### `ResultError`

Levée quand le processus Claude Code se termine parce que l'exécution s'est achevée sur un [message de résultat](#resultmessage) d'erreur, comme une erreur de limite de tours ou une erreur d'API. `ResultError` est une sous-classe de `ProcessError`, donc un gestionnaire `except ProcessError` existant l'intercepte aussi. Ses attributs portent les champs de ce message de résultat : vous pouvez ainsi agir selon la raison de l'échec sans analyser le texte du message. Exige le SDK Agent Python 0.2.140 ou ultérieur.

```python theme={null}
class ResultError(ProcessError):
    subtype: str | None  # "error_max_turns", "error_during_execution", ... ; "success" quand l'exécution s'est terminée sur une requête échouée
    errors: list[str]  # liste vide quand le message de résultat n'en signalait aucune
    result: str | None
    api_error_status: int | None
    terminal_reason: str | None  # "max_turns", "api_error", ... ; à vérifier avant subtype
    session_id: str | None
    data: dict[str, Any]  # la charge utile brute du message de résultat
```

Pour distinguer les échecs, vérifiez `terminal_reason` avant `subtype`. Quand la dernière requête échoue, par exemple sur une erreur d'API, Claude Code signale le `subtype` `"success"` avec la cause dans `terminal_reason`, par exemple `"api_error"` ; quand une limite que vous avez fixée met fin à l'exécution, comme `max_turns` ou `max_budget_usd`, il signale un sous-type `error_*`.

### `CLIJSONDecodeError`

Levée quand l'analyse JSON échoue.

```python theme={null}
class CLIJSONDecodeError(ClaudeSDKError):
    def __init__(self, line: str, original_error: Exception):
        """
        Args:
            line: The line that failed to parse
            original_error: The original JSON decode exception
        """
        self.line = line
        self.original_error = original_error
```

---

*Suite de la page (types de hooks et au-delà) : voir la partie 2/2 de cette référence.*
