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
| `input_schema` | `type[T] \| dict[str, Any]` | Schéma de validation de l'entrée |
| `handler` | `Callable[[T], Awaitable[dict[str, Any]]]` | Fonction asynchrone qui gère l'exécution de l'outil |
| `annotations` | [`ToolAnnotations`](#toolannotations)` \| None` | Annotations d'outil facultatives (par exemple `readOnlyHint`, `destructiveHint`, `openWorldHint`, `maxResultSizeChars`) |

### `Transport`

Classe de base abstraite pour les implémentations de transport personnalisées. Utilisez-la pour communiquer avec le processus Claude via un canal personnalisé (par exemple, une connexion distante au lieu d'un sous-processus local).

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
| `allowed_tools` | `list[str]` | `[]` | Outils à approuver automatiquement sans demande, hormis les lectures sur des [chemins réseau](https://code.claude.com/docs/en/permissions#network-paths). Cela ne restreint **pas** Claude à ces seuls outils. Si vous y nommez l'un des [outils de suivi de tâches](https://code.claude.com/docs/en/agent-sdk/todo-tracking#model-availability), Claude Code y inscrit aussi la session. Les autres outils non listés retombent sur `permission_mode` et `can_use_tool`. Utilisez `disallowed_tools` pour bloquer des outils. Voir [Permissions](https://code.claude.com/docs/en/agent-sdk/permissions#allow-and-deny-rules) |
| `system_prompt` | `str \| SystemPromptPreset \| SystemPromptCustom \| SystemPromptFile \| None` | `None` | Configuration du prompt système. Passez une chaîne pour un prompt personnalisé, `{"type": "preset", "preset": "claude_code"}` pour le prompt système de Claude Code avec `"append"` facultatif, `{"type": "custom", "prompt": "..."}` pour un prompt personnalisé qui peut aussi définir `"snapshot"`, ou `{"type": "file", "path": "..."}` pour charger un gros prompt depuis le disque. Voir [`SystemPromptPreset`](#systempromptpreset), [`SystemPromptCustom`](#systempromptcustom) et [`SystemPromptFile`](#systempromptfile) |
| `mcp_servers` | `dict[str, McpServerConfig] \| str \| Path` | `{}` | Configurations de serveurs MCP ou chemin vers un fichier de configuration |
| `strict_mcp_config` | `bool` | `False` | Si `True`, utilise uniquement les serveurs passés dans `mcp_servers` et ignore le `.mcp.json` du projet, les réglages utilisateur, les serveurs MCP fournis par des plugins et les [connecteurs claude.ai](https://code.claude.com/docs/en/mcp#use-mcp-servers-from-claude-ai). Correspond au flag CLI `--strict-mcp-config` |
| `permission_mode` | `PermissionMode \| None` | `None` | Mode de permission pour l'utilisation des outils |
| `continue_conversation` | `bool` | `False` | Poursuit la conversation la plus récente |
| `resume` | `str \| None` | `None` | ID de la session à reprendre |
| `session_id` | `str \| None` | `None` | Utilise un ID de session précis au lieu d'un ID généré automatiquement. Doit être un UUID valide. Ne peut pas être combiné avec `continue_conversation` ou `resume`, sauf si `fork_session` est aussi défini |
| `max_turns` | `int \| None` | `None` | Nombre maximal de tours agentiques (allers-retours d'utilisation d'outils) |
| `max_budget_usd` | `float \| None` | `None` | Arrête la requête quand l'estimation de coût côté client atteint cette valeur en USD. L'estimation peut dépasser cette valeur, donc [gardez une marge](https://code.claude.com/docs/en/agent-sdk/agent-loop#budget-headroom). Ne compte que les dépenses de l'appel lui-même ; les totaux restaurés depuis une session reprise ne comptent pas. Pour les réserves sur la précision et le comportement de réinitialisation, voir [Suivre les coûts et l'usage](https://code.claude.com/docs/en/agent-sdk/cost-tracking) |
| `disallowed_tools` | `list[str]` | `[]` | Outils à refuser. Un nom nu comme `"Bash"` retire l'outil du contexte de Claude. Une règle ciblée comme `"Bash(rm *)"` laisse l'outil disponible et refuse les appels correspondants dans tous les modes de permission, y compris `bypassPermissions`, pour la commande [telle qu'écrite](https://code.claude.com/docs/en/permissions#bash-rule-limits). Voir [Permissions](https://code.claude.com/docs/en/agent-sdk/permissions#allow-and-deny-rules) |
| `enable_file_checkpointing` | `bool` | `False` | Active le suivi des modifications de fichiers pour pouvoir rembobiner. Voir [Points de contrôle de fichiers](https://code.claude.com/docs/en/agent-sdk/file-checkpointing) |
| `model` | `str \| None` | `None` | Alias de modèle Claude ou nom complet du modèle. Voir [valeurs acceptées et ID propres aux fournisseurs](https://code.claude.com/docs/en/model-config#available-models) |
| `fallback_model` | `str \| None` | `None` | Modèle de repli à utiliser si le modèle principal échoue. Accepte une liste séparée par des virgules. Pour des conseils, voir [Choisir un modèle](https://code.claude.com/docs/en/agent-sdk/configuration#choose-a-model) |
| `betas` | `list[SdkBeta]` | `[]` | Fonctionnalités bêta à activer. Voir [`SdkBeta`](#sdkbeta) pour les options disponibles |
| `output_format` | `dict[str, Any] \| None` | `None` | Format de sortie pour les réponses structurées (ex. `{"type": "json_schema", "schema": {...}}`). Voir [Sorties structurées](https://code.claude.com/docs/en/agent-sdk/structured-outputs) pour les détails |
| `permission_prompt_tool_name` | `str \| None` | `None` | Nom de l'outil MCP pour les demandes de permission |
| `cwd` | `str \| Path \| None` | `None` | Répertoire de travail courant |
| `cli_path` | `str \| Path \| None` | `None` | Chemin personnalisé vers l'exécutable CLI de Claude Code |
| `settings` | `str \| None` | `None` | Chemin vers un fichier de réglages ou une chaîne JSON en ligne |
| `add_dirs` | `list[str \| Path]` | `[]` | Répertoires supplémentaires auxquels Claude peut accéder. Le SDK passe chaque entrée à Claude Code sous forme de `--add-dir` ; avec la source de réglages `project`, Claude Code [charge aussi les skills, commandes et sous-agents du répertoire](https://code.claude.com/docs/en/permissions#additional-directories-grant-file-access-not-configuration) |
| `env` | `dict[str, str]` | `{}` | Variables d'environnement fusionnées par-dessus l'environnement hérité du processus. Voir [Variables d'environnement](https://code.claude.com/docs/en/env-vars) pour les variables lues par la CLI sous-jacente, et [Gérer les réponses API lentes ou bloquées](#gérer-les-réponses-api-lentes-ou-bloquées) pour les variables liées aux délais. Définissez `CLAUDE_AGENT_SDK_CLIENT_APP` pour identifier votre application dans l'en-tête User-Agent |
| `extra_args` | `dict[str, str \| None]` | `{}` | Arguments CLI supplémentaires à passer directement à la CLI |
| `max_buffer_size` | `int \| None` | `None` | Nombre maximal d'octets lors de la mise en tampon du stdout de la CLI |
| `debug_stderr` | `Any` | `sys.stderr` | *Obsolète* — le SDK ignore cette valeur. Utilisez le callback `stderr` pour la sortie stderr de la CLI |
| `stderr` | `Callable[[str], None] \| None` | `None` | Fonction de rappel pour la sortie stderr de la CLI |
| `can_use_tool` | [`CanUseTool`](#canusetool) ` \| None` | `None` | Callback de permission d'outil, invoqué uniquement quand le [flux de permissions](https://code.claude.com/docs/en/agent-sdk/permissions#how-permissions-are-evaluated) aboutit à une demande. Non invoqué pour les appels approuvés automatiquement par `allowed_tools`, des règles allow ou `permission_mode`. Une règle allow ne pré-approuve pas les [actions qu'aucun mode n'approuve automatiquement](https://code.claude.com/docs/en/permission-modes#actions-no-mode-auto-approves). Voir [`CanUseTool`](#canusetool) pour les détails |
| `hooks` | `dict[HookEvent, list[HookMatcher]] \| None` | `None` | Configurations de hooks pour intercepter des événements |
| `user` | `str \| None` | `None` | Sur les plateformes POSIX, le compte utilisateur du système sous lequel s'exécute le sous-processus Claude Code. Claude Code conserve l'environnement du processus parent, y compris `HOME`, et s'exécute dans `cwd` |
| `include_partial_messages` | `bool` | `False` | Inclut les événements de streaming de messages partiels. Une fois activé, des messages [`StreamEvent`](#streamevent) sont produits |
| `include_hook_events` | `bool` | `False` | Inclut les événements du cycle de vie des hooks dans le flux de messages sous forme d'objets `HookEventMessage` |
| `forward_subagent_text` | `bool` | `False` | Transmet les blocs de texte et de réflexion (thinking) des sous-agents dans le flux de messages. Sans cette option, Claude Code omet les blocs de texte et de réflexion d'un sous-agent exécuté au [premier plan](https://code.claude.com/docs/en/sub-agents#run-subagents-in-foreground-or-background). Pour les sous-agents imbriqués, les skills avec `context: fork` et la version de Claude Code nécessaire pour chacun, voir [Suivre les messages des sous-agents](https://code.claude.com/docs/en/headless#follow-subagent-messages). Nécessite le SDK Agent Python 0.2.140 ou ultérieur |
| `verbatim_prompts` | `bool` | `False` | Transmet chaque prompt tel qu'écrit. Le SDK envoie chaque message utilisateur avec `client_composed` à `True`. Voir [`client_composed`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkusermessage) pour ce que Claude Code ignore sur ces messages. Utilisez cette option quand le texte de votre prompt contient du contenu que l'utilisateur final n'a pas tapé. Pour un contrôle par tour, laissez-la désactivée et définissez plutôt `"client_composed": True` sur les messages streamés individuels. Tant que l'option est active, le SDK écrase toute valeur `client_composed` que vous définissez. Nécessite le SDK Agent Python 0.2.158 ou ultérieur et Claude Code v2.1.248 ou ultérieur ; la CLI fournie avec ces versions du SDK satisfait l'exigence Claude Code |
| `fork_session` | `bool` | `False` | Lors d'une reprise avec `resume`, bifurque vers un nouvel ID de session au lieu de poursuivre la session d'origine |
| `resume_session_at` | `str \| None` | `None` | Lors d'une reprise, ne charge la conversation que jusqu'au message portant cet UUID inclus. À utiliser avec `resume`, et généralement `fork_session`, pour bifurquer depuis un point antérieur. Nécessite le SDK Agent Python 0.2.137 ou ultérieur |
| `resume_drops_turn` | `str \| None` | `None` | UUID du prompt utilisateur dont le tour est supprimé par une troncature `resume_session_at`. S'il est défini, la CLI refuse la reprise si la plage supprimée contient des entrées non attribuables à ce tour. Nécessite le SDK Agent Python 0.2.137 ou ultérieur et Claude Code v2.1.223 ou ultérieur ; la CLI fournie avec ces versions du SDK satisfait l'exigence Claude Code |
| `agents` | `dict[str, AgentDefinition] \| None` | `None` | Sous-agents définis par programmation |
| `plugins` | `list[SdkPluginConfig]` | `[]` | Charge des plugins personnalisés depuis des chemins locaux. Voir [Plugins](https://code.claude.com/docs/en/agent-sdk/plugins) pour les détails |
| `sandbox` | [`SandboxSettings`](#sandboxsettings) ` \| None` | `None` | Configure le comportement du sandbox par programmation. Voir [Réglages du sandbox](#sandboxsettings) pour les détails |
| `setting_sources` | `list[SettingSource] \| None` | `None` (défaut CLI : toutes les sources) | Contrôle les réglages du système de fichiers à charger. Passez `[]` pour désactiver les réglages user, project et local. Avec `skills` défini et ce champ non défini, seules les sources user et project sont chargées. Définissez `setting_sources` explicitement pour conserver les réglages locaux. La politique gérée par terminal (endpoint-managed) est chargée dans tous les cas ; les réglages gérés par serveur sont récupérés quand la session s'authentifie avec un identifiant d'organisation sur une [configuration éligible](https://code.claude.com/docs/en/server-managed-settings#platform-availability). Pour les entrées lues indépendamment de cette option, voir [Ce que settingSources ne contrôle pas](https://code.claude.com/docs/en/agent-sdk/claude-code-features#what-settingsources-does-not-control) |
| `skills` | `list[str] \| Literal["all"] \| None` | `None` | Skills disponibles pour la session. Passez `"all"` pour activer tous les skills découverts, ou une liste de noms de skills. Ne passez que des noms exacts. Le SDK rejette les noms mal formés et les noms avec jokers par une `ValueError` avant de démarrer le processus Claude Code ; cette vérification nécessite le SDK Agent Python 0.2.129 ou ultérieur. Une fois défini, le SDK ajoute automatiquement l'outil Skill à `allowed_tools`. Si vous passez aussi `tools`, incluez `"Skill"` dans cette liste. Voir [Skills](https://code.claude.com/docs/en/agent-sdk/skills) |
| `max_thinking_tokens` | `int \| None` | `None` | *Obsolète* — nombre maximal de tokens pour les blocs de réflexion. Utilisez `thinking` à la place |
| `thinking` | [`ThinkingConfig`](#thinkingconfig) ` \| None` | `None` | Contrôle le comportement de réflexion étendue (extended thinking). Prioritaire sur `max_thinking_tokens` |
| `effort` | [`EffortLevel`](#effortlevel) ` \| None` | `None` | Niveau d'effort pour la profondeur de réflexion. Voir [ajuster le niveau d'effort](https://code.claude.com/docs/en/model-config#adjust-effort-level) |
| `session_store` | [`SessionStore`](https://code.claude.com/docs/en/agent-sdk/session-storage#the-sessionstore-interface) ` \| None` | `None` | Réplique les transcriptions de session vers un backend externe pour qu'un autre hôte puisse les reprendre. Voir [Persister les sessions dans un stockage externe](https://code.claude.com/docs/en/agent-sdk/session-storage) |
| `session_store_flush` | `Literal["batched", "eager"]` | `"batched"` | Quand vider les entrées de transcription répliquées vers `session_store`. `"batched"` vide une fois par tour ou quand le tampon est plein ; `"eager"` déclenche un vidage en arrière-plan après chaque trame. Ignoré quand `session_store` vaut `None` |
| `load_timeout_ms` | `int` | `60000` | Délai par appel pour `session_store.load()` et `list_subkeys()` lors de la matérialisation d'une reprise, en millisecondes |
| `task_budget` | `TaskBudget \| None` | `None` | Budget de tokens côté API. Envoyé sous la forme `output_config.task_budget` avec l'en-tête bêta `task-budgets-2026-03-13`. Passez `{"total": <int>}`. |

#### Gérer les réponses API lentes ou bloquées

Le sous-processus CLI lit plusieurs variables d'environnement qui contrôlent les délais d'API et la détection de blocages. Passez-les via `ClaudeAgentOptions.env` :

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
* `CLAUDE_CODE_MAX_RETRIES` : nombre maximal de nouvelles tentatives API. Défaut `10`, plafonné à `15`. Chaque nouvelle tentative dispose de sa propre fenêtre `API_TIMEOUT_MS`.

  Pour les exécutions sans surveillance qui doivent tenir pendant des pannes plus longues, définissez [`CLAUDE_CODE_RETRY_WATCHDOG=1`](https://code.claude.com/docs/en/errors#tune-retry-behavior) : il retente indéfiniment les erreurs transitoires de capacité et, sur Claude Code v2.1.199 ou ultérieur, porte à `300` le défaut pour les autres erreurs transitoires et supprime le plafond de cette variable.
* `CLAUDE_ASYNC_AGENT_STALL_TIMEOUT_MS` : chien de garde (watchdog) de blocage pour les sous-agents. Tant que le watchdog de flux est actif, le défaut est `CLAUDE_STREAM_IDLE_TIMEOUT_MS` plus 5 minutes, soit `600000` sauf si vous augmentez cette variable. Avec le watchdog de flux désactivé, le défaut est `600000`. Avant la v2.1.257, le défaut était toujours `600000`.

  Le minuteur se réinitialise à chaque événement de flux. En cas de blocage, Claude Code interrompt le sous-agent et signale le blocage au parent. Pour un sous-agent d'arrière-plan, il marque aussi la tâche comme échouée et joint tout résultat partiel.
* `CLAUDE_ENABLE_STREAM_WATCHDOG` avec `CLAUDE_STREAM_IDLE_TIMEOUT_MS` : watchdog de flux qui interrompt la requête lorsque les en-têtes sont arrivés mais que le corps de la réponse cesse d'être streamé. Le watchdog est actif par défaut pour tous les fournisseurs ; définissez `CLAUDE_ENABLE_STREAM_WATCHDOG=0` pour le désactiver. `CLAUDE_STREAM_IDLE_TIMEOUT_MS` vaut `300000` par défaut et est ramené à ce minimum. Après l'interruption, la section [Nouvelles tentatives automatiques](https://code.claude.com/docs/en/errors#automatic-retries) décrit ce que fait Claude Code, selon l'avancement de la réponse.

  Pendant que le watchdog attend une réponse qu'une passerelle derrière `ANTHROPIC_BASE_URL` maintient ouverte avec des pings keep-alive, un hôte qui définit `include_partial_messages` continue de recevoir des messages [`StreamEvent`](#streamevent) de type `ping`. Interprétez ces trames comme un signe de vie plutôt que de faire expirer la session sur un silence. Avant la v2.1.257, ces trames s'arrêtaient 5 minutes après le dernier véritable événement de flux.

### `OutputFormat`

Configuration de la validation de sortie structurée. Passez-la sous forme de `dict` au champ `output_format` de `ClaudeAgentOptions` :

```python theme={null}
# Forme de dict attendue pour output_format
{
    "type": "json_schema",
    "schema": {...},  # Votre définition JSON Schema
}
```

| Champ | Obligatoire | Description |
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

| Champ | Obligatoire | Description |
| :- | :- | :- |
| `type` | Oui | Doit valoir `"preset"` pour utiliser un prompt système prédéfini |
| `preset` | Oui | Doit valoir `"claude_code"` pour utiliser le prompt système de Claude Code |
| `append` | Non | Instructions supplémentaires à ajouter à la fin du prompt système prédéfini |
| `exclude_dynamic_sections` | Non | Déplace le contexte propre à chaque utilisateur, comme l'emplacement de la mémoire automatique, du prompt système vers le premier message utilisateur. Améliore la réutilisation du cache de prompt entre utilisateurs et machines. Voir [Modifier les prompts système](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts#improve-prompt-caching-across-users-and-machines) |
| `snapshot` | Non | Mettez `False` pour reconstruire le prompt système à chaque requête au lieu de [réutiliser le prompt enregistré par la session lors de sa première requête](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts#change-the-prompt-of-an-existing-session). Nécessite `claude-agent-sdk` v0.2.153 ou ultérieur |

### `SystemPromptCustom`

Un prompt système personnalisé sous forme d'objet, équivalent à passer une chaîne comme `system_prompt`, qui peut aussi définir `snapshot`. Nécessite `claude-agent-sdk` v0.2.153 ou ultérieur.

```python theme={null}
class SystemPromptCustom(TypedDict):
    type: Literal["custom"]
    prompt: str
    snapshot: NotRequired[bool]
```

| Champ | Obligatoire | Description |
| :- | :- | :- |
| `type` | Oui | Doit valoir `"custom"` |
| `prompt` | Oui | Le texte du prompt système. Passé à la CLI comme argument de ligne de commande, donc les [limites de longueur de ligne de commande](#systempromptfile) s'appliquent |
| `snapshot` | Non | Identique à [`SystemPromptPreset.snapshot`](#systempromptpreset), appliqué à `prompt` |

### `SystemPromptFile`

Configuration pour charger un prompt système personnalisé depuis un fichier au lieu de le passer comme chaîne. Le SDK la traduit par le flag CLI [`--system-prompt-file`](https://code.claude.com/docs/en/cli-reference#system-prompt-flags). Utilisez la forme fichier quand le prompt est volumineux : le SDK passe un `system_prompt` de type chaîne dans l'argv du sous-processus CLI, qui est soumis aux limites de longueur de ligne de commande du système d'exploitation avant même que le SDK n'envoie une requête API. Sous Linux, un argument unique de plus d'environ 128 Ko échoue au lancement du processus avec `Argument list too long`. Sous Windows, la ligne de commande entière est plafonnée à environ 32 Ko, donc la forme chaîne échoue à un seuil plus bas.

```python theme={null}
class SystemPromptFile(TypedDict):
    type: Literal["file"]
    path: str
```

| Champ | Obligatoire | Description |
| :- | :- | :- |
| `type` | Oui | Doit valoir `"file"` pour charger le prompt depuis le disque |
| `path` | Oui | Chemin d'un fichier contenant le prompt système |

### `SettingSource`

Contrôle les sources de configuration basées sur le système de fichiers depuis lesquelles le SDK charge les réglages.

```python theme={null}
SettingSource = Literal["user", "project", "local"]
```

| Valeur | Description | Emplacement |
| :- | :- | :- |
| `"user"` | Réglages utilisateur globaux | `~/.claude/settings.json` |
| `"project"` | Réglages de projet partagés (sous contrôle de version) | `.claude/settings.json` |
| `"local"` | Réglages de projet locaux, ajoutés au gitignore quand Claude Code y enregistre un réglage | `.claude/settings.local.json` |

#### Comportement par défaut

Quand `setting_sources` est omis ou vaut `None` et que `skills` n'est pas défini, `query()` charge les mêmes réglages du système de fichiers que la CLI Claude Code : user, project et local. Avec `skills` défini, la ligne [`setting_sources`](#claudeagentoptions) décrit le défaut en vigueur. La politique gérée par terminal (endpoint-managed) est chargée dans tous les cas ; les réglages gérés par serveur sont récupérés quand la session s'authentifie avec un identifiant d'organisation sur une [configuration éligible](https://code.claude.com/docs/en/server-managed-settings#platform-availability). Pour plus d'informations, voir [Ce que settingSources ne contrôle pas](https://code.claude.com/docs/en/agent-sdk/claude-code-features#what-settingsources-does-not-control).

#### Pourquoi utiliser setting\_sources

**Désactiver les réglages du système de fichiers :**

```python theme={null}
# Ne pas charger les réglages user, project ou local depuis le disque
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

> **Note** : dans le SDK Python 0.1.59 et antérieur, une liste vide était traitée comme l'omission de l'option, donc `setting_sources=[]` ne désactivait pas les réglages du système de fichiers. Passez à une version plus récente si vous avez besoin qu'une liste vide prenne effet. Le SDK TypeScript n'est pas concerné.

