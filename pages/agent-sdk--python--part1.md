# Référence de l'Agent SDK — Python — partie 1/2

> Source : https://code.claude.com/docs/en/agent-sdk/python · [← Index](../31-index-documentation-officielle.md)

Référence complète de l'API du SDK Agent pour Python : toutes les fonctions, types et classes. Cette partie 1/2 couvre l'installation, le choix entre `query()` et `ClaudeSDKClient`, les fonctions, les classes et le début des types (jusqu'à la fin des lignes 1 à 2085 de la page source).

## Installation

Installez le paquet dans un environnement virtuel. Sur les installations Python récentes de Debian, Ubuntu et Homebrew, lancer `pip install` contre le Python système échoue avec `error: externally-managed-environment`.

```bash theme={null}
python3 -m venv .venv
source .venv/bin/activate
pip install claude-agent-sdk
```

Pour uv, Windows PowerShell et la configuration de la clé API, voir [Setup dans le quickstart de l'Agent SDK](https://code.claude.com/docs/en/agent-sdk/quickstart#setup).

## Choisir entre `query()` et `ClaudeSDKClient`

Le SDK Python offre deux façons d'interagir avec Claude Code :

| Fonctionnalité | `query()` | `ClaudeSDKClient` |
| :- | :- | :- |
| **Session** | Crée une nouvelle session par défaut | Réutilise la même session |
| **Conversation** | Échange unique | Plusieurs échanges dans le même contexte |
| **Connexion** | Gérée automatiquement | Contrôle manuel |
| **Entrée en streaming** | ✅ Prise en charge | ✅ Prise en charge |
| **Interruptions** | ❌ Non prise en charge | ✅ Prise en charge |
| **Hooks** | ✅ Pris en charge | ✅ Pris en charge |
| **Outils personnalisés** | ✅ Pris en charge | ✅ Pris en charge |
| **Poursuivre la discussion** | Manuel via `continue_conversation` ou `resume` | ✅ Automatique |
| **Cas d'usage** | Tâches ponctuelles | Conversations continues |

Utilisez `ClaudeSDKClient` pour les applications interactives comme les interfaces de chat, ou lorsque l'action suivante dépend de la réponse de Claude.

## Fonctions

> **Note** : les blocs de signature et les fragments `async for` / `async with` isolés de cette page sont illustratifs. Pour les exécuter, enveloppez le corps dans `async def main(): ...` et appelez `asyncio.run(main())`.

### `query()`

Crée par défaut une nouvelle session pour chaque interaction avec Claude Code. Renvoie un itérateur asynchrone qui produit les messages au fur et à mesure qu'ils arrivent. Chaque appel à `query()` repart de zéro, sans mémoire des interactions précédentes, sauf si vous passez `continue_conversation=True` ou `resume` dans [`ClaudeAgentOptions`](#claudeagentoptions). Voir [Sessions](https://code.claude.com/docs/en/agent-sdk/sessions).

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
| `options` | `ClaudeAgentOptions \| None` | Objet de configuration facultatif (vaut `ClaudeAgentOptions()` par défaut si `None`) |
| `transport` | `Transport \| None` | Transport personnalisé facultatif pour communiquer avec le processus CLI |

#### Valeur renvoyée

Renvoie un `AsyncIterator[Message]` qui produit les messages de la conversation.

#### Exemple — Avec options

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
| `input_schema` | `type \| dict[str, Any]` | Schéma définissant les paramètres d'entrée de l'outil. Voir [Options de schéma d'entrée](#options-de-schéma-dentrée) |
| `annotations` | [`ToolAnnotations`](#toolannotations)` \| None` | Annotations d'outil MCP facultatives fournissant des indications comportementales aux clients |

#### Options de schéma d'entrée

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
   * **Python 3.10** : `typing` n'a pas de `NotRequired`. Importez `TypedDict` et `NotRequired` depuis `typing_extensions`, que le SDK installe sur Python 3.10.

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

#### Valeur renvoyée

Une fonction décorateur qui enveloppe l'implémentation de l'outil et renvoie une instance de `SdkMcpTool`.

#### Exemple

```python theme={null}
from claude_agent_sdk import tool
from typing import Any


@tool("greet", "Greet a user", {"name": str})
async def greet(args: dict[str, Any]) -> dict[str, Any]:
    return {"content": [{"type": "text", "text": f"Hello, {args['name']}!"}]}
```

#### `ToolAnnotations`

Indications comportementales pour un outil, passées comme argument `annotations` de [`tool()`](#tool). `ToolAnnotations` étend `mcp.types.ToolAnnotations` du SDK MCP avec un champ `maxResultSizeChars`, et chaque indication peut s'écrire en camelCase ou en snake\_case : `ToolAnnotations(readOnlyHint=True)` et `ToolAnnotations(read_only_hint=True)` sont équivalents. Pour relire une indication sur l'objet, utilisez l'orthographe déclarée par votre paquet `mcp` installé : `.readOnlyHint` avec `mcp` 1.x et `.read_only_hint` avec 2.x, tandis que `.maxResultSizeChars` fonctionne avec les deux. Vous pouvez aussi passer un simple `mcp.types.ToolAnnotations` partout où le SDK accepte des annotations.

Les noms snake\_case et le champ typé `maxResultSizeChars` nécessitent le SDK Agent Python 0.2.140 ou ultérieur. Les versions 0.1.31 à 0.2.139 réexportent `mcp.types.ToolAnnotations` sans modification. Sur les versions 0.1.55 à 0.2.139, vous pouvez quand même passer `maxResultSizeChars` comme argument nommé : la classe MCP accepte les champs supplémentaires et le SDK transmet la valeur à Claude Code.

Tous les champs sont facultatifs. Les clients ne doivent pas s'appuyer sur ces indications pour des décisions de sécurité.

| Champ | Type | Défaut | Description |
| :- | :- | :- | :- |
| `title` | `str \| None` | `None` | Titre lisible de l'outil |
| `readOnlyHint` | `bool \| None` | `False` | Si `True`, l'outil ne modifie pas son environnement |
| `destructiveHint` | `bool \| None` | `True` | Si `True`, l'outil peut effectuer des modifications destructrices (n'a de sens que si `readOnlyHint` vaut `False`) |
| `idempotentHint` | `bool \| None` | `False` | Si `True`, des appels répétés avec les mêmes arguments n'ont pas d'effet supplémentaire (n'a de sens que si `readOnlyHint` vaut `False`) |
| `openWorldHint` | `bool \| None` | `True` | Si `True`, l'outil interagit avec des entités externes (par exemple, la recherche web). Si `False`, le domaine de l'outil est fermé (par exemple, un outil de mémoire) |
| `maxResultSizeChars` | `int \| None` | `None` | Nombre de caractères jusqu'auquel Claude Code garde le résultat texte de cet outil en ligne dans la conversation au lieu de l'enregistrer dans un fichier, jusqu'à 500 000. Les résultats contenant des images ne sont pas concernés. C'est un réglage Claude Code plutôt qu'une indication MCP : le SDK l'envoie dans le `_meta` de l'outil sous `anthropic/maxResultSizeChars`. Voir [Augmenter la limite pour un outil précis](https://code.claude.com/docs/en/mcp#raise-the-limit-for-a-specific-tool) |

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

Crée un serveur MCP en processus (in-process) qui s'exécute au sein de votre application Python.

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

#### Valeur renvoyée

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
    tools=[add, multiply],  # Passer les fonctions décorées
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
| `offset` | `int` | `0` | Nombre de sessions à sauter depuis le début des résultats triés. À utiliser avec `limit` pour la pagination |
| `include_worktrees` | `bool` | `True` | Quand `directory` est dans un dépôt git, inclut les sessions de tous les chemins de worktree |

#### Type renvoyé : `SDKSessionInfo`

| Propriété | Type | Description |
| :- | :- | :- |
| `session_id` | `str` | Identifiant unique de la session |
| `summary` | `str` | Titre affiché : titre personnalisé, prompt le plus récent, résumé généré automatiquement ou premier prompt |
| `last_modified` | `int` | Date de dernière modification en millisecondes depuis l'epoch |
| `file_size` | `int \| None` | Taille du fichier de session en octets (`None` pour les backends de stockage distants) |
| `custom_title` | `str \| None` | Titre de la session : le titre défini par l'utilisateur, ou le titre généré automatiquement s'il n'y en a pas |
| `first_prompt` | `str \| None` | Premier prompt utilisateur significatif de la session |
| `git_branch` | `str \| None` | Branche git à la fin de la session |
| `cwd` | `str \| None` | Répertoire de travail de la session |
| `tag` | `str \| None` | Étiquette (tag) de session définie par l'utilisateur (voir [`tag_session()`](#tag_session)) |
| `created_at` | `int \| None` | Date de création de la session en millisecondes depuis l'epoch |

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
| `session_id` | `str` | obligatoire | L'ID de la session dont récupérer les messages |
| `directory` | `str \| None` | `None` | Répertoire de projet où chercher. S'il est omis, cherche dans tous les projets |
| `limit` | `int \| None` | `None` | Nombre maximal de messages à renvoyer |
| `offset` | `int` | `0` | Nombre de messages à sauter depuis le début |

#### Type renvoyé : `SessionMessage`

| Propriété | Type | Description |
| :- | :- | :- |
| `type` | `Literal["user", "assistant"]` | Rôle du message |
| `uuid` | `str` | Identifiant unique du message |
| `session_id` | `str` | Identifiant de la session |
| `message` | `Any` | Contenu brut du message |
| `parent_tool_use_id` | `str \| None` | Pour les messages de sous-agent, l'id du bloc tool-use `Agent` qui l'a lancé. `None` pour les messages de la session principale et les sessions plus anciennes |
| `parent_agent_id` | `str \| None` | Pour les messages d'un [sous-agent imbriqué](https://code.claude.com/docs/en/sub-agents#let-subagents-spawn-their-own-subagents), l'id d'agent du sous-agent parent. `None` pour les messages de la session principale, les messages de sous-agents de premier niveau et les sessions plus anciennes. Nécessite le SDK Agent Python 0.2.140 ou ultérieur |

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

Lit les métadonnées d'une seule session par son ID, sans parcourir tout le répertoire du projet. Synchrone ; renvoie immédiatement.

```python theme={null}
def get_session_info(
    session_id: str,
    directory: str | None = None,
) -> SDKSessionInfo | None
```

#### Paramètres

| Paramètre | Type | Défaut | Description |
| :- | :- | :- | :- |
| `session_id` | `str` | obligatoire | UUID de la session à consulter |
| `directory` | `str \| None` | `None` | Chemin du répertoire de projet. S'il est omis, cherche dans tous les répertoires de projet |

Renvoie [`SDKSessionInfo`](#type-renvoyé--sdksessioninfo), ou `None` si la session est introuvable.

#### Exemple

Consulte les métadonnées d'une seule session sans parcourir le répertoire du projet. Utile quand vous avez déjà un ID de session issu d'une exécution précédente.

```python theme={null}
from claude_agent_sdk import get_session_info

info = get_session_info("550e8400-e29b-41d4-a716-446655440000")
if info:
    print(f"{info.summary} (branch: {info.git_branch}, tag: {info.tag})")
```

### `rename_session()`

Renomme une session en ajoutant une entrée de titre personnalisé. Les appels répétés sont sans danger ; le titre le plus récent l'emporte. Synchrone.

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
| `session_id` | `str` | obligatoire | UUID de la session à renommer |
| `title` | `str` | obligatoire | Nouveau titre. Doit être non vide après suppression des espaces |
| `directory` | `str \| None` | `None` | Chemin du répertoire de projet. S'il est omis, cherche dans tous les répertoires de projet |

Lève `ValueError` si `session_id` n'est pas un UUID valide ou si `title` est vide ; `FileNotFoundError` si la session est introuvable.

#### Exemple

Renomme la session la plus récente pour la retrouver plus facilement. Le nouveau titre apparaît dans [`SDKSessionInfo.custom_title`](#type-renvoyé--sdksessioninfo) lors des lectures suivantes.

```python theme={null}
from claude_agent_sdk import list_sessions, rename_session

sessions = list_sessions(directory="/path/to/project", limit=1)
if sessions:
    rename_session(sessions[0].session_id, "Refactor auth module")
```

### `tag_session()`

Étiquette (tag) une session. Passez `None` pour effacer l'étiquette. Les appels répétés sont sans danger ; l'étiquette la plus récente l'emporte. Synchrone.

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
| `session_id` | `str` | obligatoire | UUID de la session à étiqueter |
| `tag` | `str \| None` | obligatoire | Chaîne d'étiquette, ou `None` pour l'effacer. Assainie (Unicode) avant stockage |
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

# Plus tard : trouver toutes les sessions portant cette étiquette
for session in list_sessions(directory="/path/to/project"):
    if session.tag == "needs-review":
        print(session.summary)
```

### `fork_session()`

Copie la transcription d'une session dans une nouvelle session afin d'emmener la conversation dans une autre direction, tandis que l'originale reste inchangée. Pour bifurquer depuis un point antérieur de la conversation, passez `up_to_message_id`. Synchrone.

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
| `session_id` | `str` | obligatoire | UUID de la session à forker |
| `directory` | `str \| None` | `None` | Chemin du répertoire de projet. S'il est omis, cherche dans tous les répertoires de projet |
| `up_to_message_id` | `str \| None` | `None` | Copie la transcription jusqu'au message portant cet UUID inclus, par exemple un `uuid` issu de [`get_session_messages()`](#get_session_messages). S'il est omis, copie toute la transcription |
| `title` | `str \| None` | `None` | Titre du fork. S'il est omis, le SDK en dérive un de la session d'origine, suivi de `(fork)` |

Renvoie un `ForkSessionResult` dont le `session_id` est l'UUID de la nouvelle session. Passez-le comme [`resume`](#claudeagentoptions) pour poursuivre le fork. Le fork n'inclut pas les [points de contrôle de fichiers](https://code.claude.com/docs/en/agent-sdk/file-checkpointing) de la session d'origine : vous ne pouvez donc pas le rembobiner vers un point de contrôle capturé avant le fork.

`fork_session()` lève :

* `ValueError` : `session_id` ou `up_to_message_id` n'est pas un UUID valide
* `ValueError` : la session n'a aucun message, ou `up_to_message_id` ne correspond à aucun message de la transcription
* `FileNotFoundError` : la session est introuvable

#### Exemple

Forke la session la plus récente sous un nouveau titre, puis reprend le fork. La session d'origine conserve son propre historique.

```python theme={null}
from claude_agent_sdk import fork_session, list_sessions

sessions = list_sessions(directory="/path/to/project", limit=1)
if sessions:
    forked = fork_session(sessions[0].session_id, title="Try the OAuth approach")
    print(forked.session_id)  # à passer comme ClaudeAgentOptions(resume=...) pour poursuivre le fork
```

## Classes

### `ClaudeSDKClient`

**Maintient une session de conversation sur plusieurs échanges.** C'est l'équivalent Python de la façon dont la fonction `query()` du SDK TypeScript fonctionne en interne : elle crée un objet client capable de poursuivre des conversations. Voir la [comparaison avec `query()`](#choisir-entre-query-et-claudesdkclient).

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
| `receive_response()` | Reçoit les messages jusqu'à un `ResultMessage` inclus |
| `interrupt()` | Envoie un signal d'interruption (ne fonctionne qu'en mode streaming) |
| `set_permission_mode(mode)` | Change le mode de permission de la session en cours |
| `set_model(model)` | Change le modèle de la session en cours. Passez `None` pour revenir au [modèle par défaut de Claude Code](https://code.claude.com/docs/en/model-config) |
| `rewind_files(user_message_id)` | Restaure les fichiers dans leur état au message utilisateur indiqué. Nécessite `enable_file_checkpointing=True`. Voir [Points de contrôle de fichiers](https://code.claude.com/docs/en/agent-sdk/file-checkpointing) |
| `get_mcp_status()` | Obtient l'état de tous les serveurs MCP configurés. Renvoie [`McpStatusResponse`](#mcpstatusresponse) |
| `get_context_usage()` | Obtient une ventilation de l'utilisation de la fenêtre de contexte par catégorie, skill et outil — les mêmes données que `/context` affiche en session interactive. Renvoie [`ContextUsageResponse`](#contextusageresponse). Pour calculer la ventilation, Claude Code effectue plusieurs requêtes API de comptage de tokens qui n'apparaissent pas dans le flux de messages ; voir [comment ces requêtes sont traitées](#contextusageresponse) |
| `reconnect_mcp_server(server_name)` | Retente la connexion à un serveur MCP qui a échoué ou a été déconnecté |
| `toggle_mcp_server(server_name, enabled)` | Active ou désactive un serveur MCP en cours de session. Désactiver un serveur stdio, SSE ou HTTP retire ses outils |
| `stop_task(task_id)` | Arrête une tâche d'arrière-plan en cours. Un [`TaskNotificationMessage`](#tasknotificationmessage) avec le statut `"stopped"` suit dans le flux de messages |
| `get_server_info()` | Obtient les informations d'initialisation du serveur, dont les commandes et styles de sortie disponibles |
| `disconnect()` | Se déconnecte de Claude |

#### Prise en charge du gestionnaire de contexte

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

> **Important** : lors de l'itération sur les messages, évitez d'utiliser `break` pour sortir prématurément, car cela peut provoquer des problèmes de nettoyage asyncio. Laissez plutôt l'itération se terminer naturellement ou utilisez des drapeaux pour noter que vous avez trouvé ce qu'il vous fallait.

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

        # Question de suivi - la session conserve le contexte précédent
        await client.query("What's the population of that city?")

        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(f"Claude: {block.text}")

        # Autre question de suivi - toujours dans la même conversation
        await client.query("What are some famous landmarks there?")

        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(f"Claude: {block.text}")


asyncio.run(main())
```

#### Exemple — Entrée en streaming avec ClaudeSDKClient

`query()` accepte aussi un itérable asynchrone de dicts de messages utilisateur : vous pouvez ainsi assembler le prompt au moment de l'envoi ou inclure des blocs de contenu comme des images. Claude Code commence à répondre au premier message produit dès son arrivée, sans attendre la fin de l'itérable, et `receive_response()` s'arrête au `ResultMessage` qui termine cette réponse. Mettez tout ce que Claude doit lire avant de répondre dans un seul message, comme le fait ce générateur, et associez chaque appel à `query()` à sa propre boucle `receive_response()`.

```python theme={null}
import asyncio
from claude_agent_sdk import ClaudeSDKClient


async def message_stream():
    """Assembler le prompt au moment de l'envoi et le produire comme un seul message utilisateur."""
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

> **Note** : **comportement du tampon après une interruption** — `interrupt()` envoie un signal d'arrêt mais ne vide pas le tampon de messages. Les messages déjà produits par la tâche interrompue, y compris son `ResultMessage`, restent dans le flux. Vous devez les vider avec `receive_response()` avant de lire la réponse à une nouvelle requête. Si vous envoyez une nouvelle requête juste après `interrupt()` et n'appelez `receive_response()` qu'une fois, vous recevrez les messages de la tâche interrompue, et non la réponse à la nouvelle requête.

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
    """Logique personnalisée pour les permissions d'outils."""

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
    # Ne listez pas aussi les outils filtrés dans allowed_tools : les règles allow approuvent les appels avant que can_use_tool ne s'exécute
    options = ClaudeAgentOptions(can_use_tool=custom_permission_handler)

    async with ClaudeSDKClient(options=options) as client:
        await client.query("Update the system config file")

        async for message in client.receive_response():
            # Utilisera le chemin sandbox à la place
            print(message)


asyncio.run(main())
```

## Types

> **Note** : **`@dataclass` vs `TypedDict`** — ce SDK utilise deux sortes de types. Les classes décorées par `@dataclass` (comme `ResultMessage`, `AgentDefinition`, `TextBlock`) sont des instances d'objets à l'exécution et prennent en charge l'accès par attribut : `msg.result`. Les classes définies avec `TypedDict` (comme `ThinkingConfigEnabled`, `McpStdioServerConfig`, `SyncHookJSONOutput`) sont **de simples dicts à l'exécution** et exigent l'accès par clé : `config["budget_tokens"]`, et non `config.budget_tokens`. La syntaxe d'appel `ClassName(field=value)` fonctionne pour les deux, mais seules les dataclasses produisent des objets dotés d'attributs.

