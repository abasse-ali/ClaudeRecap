# Donner des outils personnalisés à Claude

> Source : https://code.claude.com/docs/en/agent-sdk/custom-tools · [← Index](../31-index-documentation-officielle.md)

*Définir des outils personnalisés avec le serveur MCP in-process de l'Agent SDK de Claude, pour que Claude puisse appeler vos fonctions, interroger vos API et effectuer des opérations propres à votre domaine.*

Les outils personnalisés étendent l'Agent SDK en vous permettant de définir vos propres fonctions, que Claude peut appeler au cours d'une conversation. Grâce au serveur MCP in-process (exécuté dans le même processus) du SDK, vous pouvez donner à Claude l'accès à des bases de données, des API externes, de la logique métier, ou toute autre capacité dont votre application a besoin.

---

## Référence rapide

| Ce que vous voulez faire | Comment faire |
| :- | :- |
| Définir un outil | Utilisez [`@tool`](https://code.claude.com/docs/en/agent-sdk/python#tool) (Python) ou [`tool()`](https://code.claude.com/docs/en/agent-sdk/typescript#tool) (TypeScript) avec un nom, une description, un schéma et un handler. Voir [Créer un outil personnalisé](#créer-un-outil-personnalisé). |
| Rendre un paramètre facultatif | Déclarez-le facultatif dans le schéma et appliquez la valeur par défaut dans le handler. Voir [Rendre un paramètre facultatif](#rendre-un-paramètre-facultatif). |
| Enregistrer un outil auprès de Claude | Enveloppez-le dans `create_sdk_mcp_server` / `createSdkMcpServer` et passez-le à `mcpServers` dans `query()`. Voir [Appeler un outil personnalisé](#appeler-un-outil-personnalisé). |
| Pré-approuver un outil | Ajoutez-le à vos outils autorisés (allowed tools). Voir [Configurer les outils autorisés](#configurer-les-outils-autorisés). |
| Retirer un outil intégré du contexte de Claude | Passez un tableau `tools` ne listant que les outils intégrés que vous voulez garder. Voir [Configurer les outils autorisés](#configurer-les-outils-autorisés). |
| Laisser Claude appeler des outils en parallèle | Définissez `readOnlyHint: true` sur les outils sans effet de bord. Voir [Ajouter des annotations d'outil](#ajouter-des-annotations-doutil). |
| Maîtriser le message d'erreur que lit Claude | Renvoyez `isError: true` pour composer vous-même le message au lieu de faire remonter l'exception brute. Voir [Gérer les erreurs](#gérer-les-erreurs). |
| Renvoyer des images ou des fichiers | Utilisez des blocs `image` ou `resource` dans le tableau `content`. Voir [Renvoyer des images et des ressources](#renvoyer-des-images-et-des-ressources). |
| Renvoyer un résultat JSON lisible par une machine | Définissez `structuredContent` sur le résultat. Voir [Renvoyer des données structurées](#renvoyer-des-données-structurées). |
| Passer à l'échelle avec de nombreux outils | Utilisez la [recherche d'outils (tool search)](https://code.claude.com/docs/en/agent-sdk/tool-search) pour charger les outils à la demande. |

---

## Créer un outil personnalisé

Un outil est défini par quatre parties, passées en arguments au helper [`tool()`](https://code.claude.com/docs/en/agent-sdk/typescript#tool) en TypeScript ou au décorateur [`@tool`](https://code.claude.com/docs/en/agent-sdk/python#tool) en Python :

- **Nom** : identifiant unique que Claude utilise pour appeler l'outil.
- **Description** : ce que fait l'outil. Claude la lit pour décider quand l'appeler.
- **Schéma d'entrée** : les arguments acceptés par l'outil, déclarés selon le langage :
  - **TypeScript** : un [schéma Zod](https://zod.dev/). Les `args` du handler tirent leurs types de ce schéma. Appelez `.describe()` sur un champ pour lui donner une description visible par Claude.
  - **Python** : un dict associant des noms à des types, comme `{"latitude": float}`, que le SDK convertit pour vous en JSON Schema. Enveloppez un type dans `Annotated`, comme `{"latitude": Annotated[float, "Latitude coordinate"]}`, pour donner au champ une description visible par Claude. Le décorateur accepte aussi directement un dict [JSON Schema](https://json-schema.org/understanding-json-schema/about) complet lorsque vous avez besoin d'enums, de plages de valeurs, de champs facultatifs ou d'objets imbriqués.
- **Handler** : la fonction asynchrone exécutée quand Claude appelle l'outil. Elle reçoit les arguments validés et doit renvoyer un objet contenant :
  - `content` (obligatoire) : un tableau de blocs de résultat, chacun avec un `type` parmi `"text"`, `"image"`, `"audio"`, `"resource"` ou `"resource_link"`. Voir [Renvoyer des images et des ressources](#renvoyer-des-images-et-des-ressources) pour les blocs non textuels.
  - `structuredContent` (facultatif) : un objet JSON contenant le résultat sous forme de données lisibles par une machine, renvoyé en plus de `content`. Voir [Renvoyer des données structurées](#renvoyer-des-données-structurées).
  - `isError` (facultatif) : mettez-le à `true` pour signaler un échec de l'outil afin que Claude puisse y réagir. Voir [Gérer les erreurs](#gérer-les-erreurs).

Une fois l'outil défini, enveloppez-le dans un serveur avec [`createSdkMcpServer`](https://code.claude.com/docs/en/agent-sdk/typescript#createsdkmcpserver) (TypeScript) ou [`create_sdk_mcp_server`](https://code.claude.com/docs/en/agent-sdk/python#create_sdk_mcp_server) (Python). Le serveur s'exécute in-process, à l'intérieur de votre application, et non comme un processus séparé.

Les exemples Python de cette page qui font des requêtes HTTP utilisent [httpx](https://www.python-httpx.org/). Ajoutez-le avec le gestionnaire de paquets de votre projet :

#### Python (uv)

```bash
uv add httpx
```

#### Python (pip)

```bash
pip install httpx
```

### Exemple : outil météo

Cet exemple définit un outil `get_temperature` et l'enveloppe dans un serveur MCP, sans passer ce serveur à `query`. Pour exécuter l'outil, voir [Appeler un outil personnalisé](#appeler-un-outil-personnalisé) ci-dessous.

#### Python

```python
from typing import Annotated, Any
import httpx
from claude_agent_sdk import tool, create_sdk_mcp_server


# Définir un outil : nom, description, schéma d'entrée, handler
@tool(
    "get_temperature",
    "Get the current temperature at a location",
    {
        "latitude": Annotated[float, "Latitude coordinate"],
        "longitude": Annotated[float, "Longitude coordinate"],
    },
)
async def get_temperature(args: dict[str, Any]) -> dict[str, Any]:
    async with httpx.AsyncClient() as client:
        response = await client.get(
            "https://api.open-meteo.com/v1/forecast",
            params={
                "latitude": args["latitude"],
                "longitude": args["longitude"],
                "current": "temperature_2m",
                "temperature_unit": "fahrenheit",
            },
        )
        data = response.json()

    # Renvoyer un tableau content - Claude le voit comme le résultat de l'outil
    return {
        "content": [
            {
                "type": "text",
                "text": f"Temperature: {data['current']['temperature_2m']}°F",
            }
        ]
    }


# Envelopper l'outil dans un serveur MCP in-process
weather_server = create_sdk_mcp_server(
    name="weather",
    version="1.0.0",
    tools=[get_temperature],
)
```

#### TypeScript

```typescript
import { tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

// Définir un outil : nom, description, schéma d'entrée, handler
const getTemperature = tool(
  "get_temperature",
  "Get the current temperature at a location",
  {
    latitude: z.number().describe("Latitude coordinate"), // .describe() ajoute une description de champ visible par Claude
    longitude: z.number().describe("Longitude coordinate")
  },
  async (args) => {
    // args est typé d'après le schéma : { latitude: number; longitude: number }
    const response = await fetch(
      `https://api.open-meteo.com/v1/forecast?latitude=${args.latitude}&longitude=${args.longitude}&current=temperature_2m&temperature_unit=fahrenheit`
    );
    const data: any = await response.json();

    // Renvoyer un tableau content - Claude le voit comme le résultat de l'outil
    return {
      content: [{ type: "text", text: `Temperature: ${data.current.temperature_2m}°F` }]
    };
  }
);

// Envelopper l'outil dans un serveur MCP in-process
const weatherServer = createSdkMcpServer({
  name: "weather",
  version: "1.0.0",
  tools: [getTemperature]
});
```

Voir la référence TypeScript de [`tool()`](https://code.claude.com/docs/en/agent-sdk/typescript#tool) ou la référence Python de [`@tool`](https://code.claude.com/docs/en/agent-sdk/python#tool) pour le détail complet des paramètres, y compris les formats d'entrée JSON Schema et la structure de la valeur de retour.

### Rendre un paramètre facultatif

Pour rendre un paramètre facultatif, déclarez-le facultatif dans le schéma et appliquez la valeur par défaut dans le handler :

- **TypeScript** : ajoutez `.optional()` au champ Zod.
- **Python** : le schéma sous forme de dict exige toutes les clés. Utilisez la forme JSON Schema, laissez le paramètre hors de `required`, et lisez-le avec `args.get()`. Pour un schéma typé avec des clés facultatives, voir [classe TypedDict](https://code.claude.com/docs/en/agent-sdk/python#input-schema-options).

L'outil [`get_precipitation_chance` ci-dessous](#ajouter-dautres-outils) illustre les deux approches.

### Appeler un outil personnalisé

Passez le serveur MCP que vous avez créé à `query` via l'option `mcpServers`. La clé dans `mcpServers` devient le segment `{server_name}` du nom pleinement qualifié de chaque outil : `mcp__{server_name}__{tool_name}`. Listez ce nom dans `allowedTools` pour que l'outil s'exécute sans demande d'autorisation.

Ces extraits réutilisent le `weatherServer` de l'[exemple d'outil météo](#exemple--outil-météo) pour demander à Claude le temps qu'il fait à un endroit précis.

#### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


async def main():
    options = ClaudeAgentOptions(
        mcp_servers={"weather": weather_server},
        allowed_tools=["mcp__weather__get_temperature"],
    )

    async for message in query(
        prompt="What's the temperature in San Francisco?",
        options=options,
    ):
        # ResultMessage est le message final, une fois tous les appels d'outils terminés
        if isinstance(message, ResultMessage) and message.subtype == "success":
            print(message.result)


asyncio.run(main())
```

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "What's the temperature in San Francisco?",
  options: {
    mcpServers: { weather: weatherServer },
    allowedTools: ["mcp__weather__get_temperature"]
  }
})) {
  // "result" est le message final, une fois tous les appels d'outils terminés
  if (message.type === "result" && message.subtype === "success") {
    console.log(message.result);
  }
}
```

Combinez cet extrait avec les définitions de l'outil et du serveur de l'[exemple d'outil météo](#exemple--outil-météo) dans un seul fichier, `weather.py` ou `weather.ts`, puis exécutez-le depuis votre terminal :

#### TypeScript (exécution)

```bash
npx tsx weather.ts
```

#### Python (uv) (exécution)

```bash
uv run weather.py
```

#### Python (pip) (exécution)

Activez l'environnement virtuel dans lequel vous avez installé le SDK, puis lancez :

```bash
python weather.py
```

Claude appelle `get_temperature` et le script affiche une réponse d'une ligne avec la température actuelle à San Francisco.

### Ajouter d'autres outils

Un serveur contient autant d'outils que vous en listez dans son tableau `tools`. Avec plusieurs outils sur un même serveur, vous pouvez lister chacun d'eux individuellement dans `allowedTools`, ou utiliser le joker `mcp__weather__*` pour couvrir tous les outils exposés par le serveur.

L'exemple ci-dessous définit un second outil, `get_precipitation_chance`, et remplace la définition de `weatherServer` de l'[exemple d'outil météo](#exemple--outil-météo) par une définition qui liste les deux outils dans le tableau.

#### Python

```python
# Définir un second outil pour le même serveur
@tool(
    "get_precipitation_chance",
    "Get the hourly precipitation probability for a location",
    {
        "type": "object",
        "properties": {
            "latitude": {"type": "number"},
            "longitude": {"type": "number"},
            "hours": {
                "type": "integer",
                "minimum": 1,
                "maximum": 24,
                "description": "How many hours of forecast to return",
            },
        },
        # 'hours' est laissé hors de required, donc Claude peut l'omettre
        "required": ["latitude", "longitude"],
    },
)
async def get_precipitation_chance(args: dict[str, Any]) -> dict[str, Any]:
    # 'hours' n'est pas dans required - le lire avec .get() pour retomber sur une valeur par défaut
    hours = args.get("hours", 12)
    async with httpx.AsyncClient() as client:
        response = await client.get(
            "https://api.open-meteo.com/v1/forecast",
            params={
                "latitude": args["latitude"],
                "longitude": args["longitude"],
                "hourly": "precipitation_probability",
                "forecast_days": 1,
            },
        )
        data = response.json()
    chances = data["hourly"]["precipitation_probability"][:hours]

    return {
        "content": [
            {
                "type": "text",
                "text": f"Next {hours} hours: {'%, '.join(map(str, chances))}%",
            }
        ]
    }


# Reconstruire le serveur avec les deux outils dans le tableau
weather_server = create_sdk_mcp_server(
    name="weather",
    version="1.0.0",
    tools=[get_temperature, get_precipitation_chance],
)
```

#### TypeScript

```typescript
// Définir un second outil pour le même serveur
const getPrecipitationChance = tool(
  "get_precipitation_chance",
  "Get the hourly precipitation probability for a location",
  {
    latitude: z.number(),
    longitude: z.number(),
    hours: z
      .number()
      .int()
      .min(1)
      .max(24)
      .optional() // .optional() permet à Claude d'omettre le paramètre
      .describe("How many hours of forecast to return")
  },
  async (args) => {
    const hours = args.hours ?? 12; // Appliquer la valeur par défaut dans le handler
    const response = await fetch(
      `https://api.open-meteo.com/v1/forecast?latitude=${args.latitude}&longitude=${args.longitude}&hourly=precipitation_probability&forecast_days=1`
    );
    const data: any = await response.json();
    const chances = data.hourly.precipitation_probability.slice(0, hours);

    return {
      content: [{ type: "text", text: `Next ${hours} hours: ${chances.join("%, ")}%` }]
    };
  }
);

// Reconstruire le serveur avec les deux outils dans le tableau
const weatherServer = createSdkMcpServer({
  name: "weather",
  version: "1.0.0",
  tools: [getTemperature, getPrecipitationChance]
});
```

La [recherche d'outils (tool search)](https://code.claude.com/docs/en/agent-sdk/tool-search) est activée par défaut et diffère le chargement des outils MCP du SDK : Claude voit le nom de chaque outil dans une liste compacte et charge son schéma complet à la demande. Si la recherche d'outils est désactivée, chaque outil de ce tableau consomme de l'espace dans la fenêtre de contexte à chaque tour. En TypeScript, passez `alwaysLoad: true` dans l'argument `extras` de [`tool()`](https://code.claude.com/docs/en/agent-sdk/typescript#tool) ou dans les options de [`createSdkMcpServer()`](https://code.claude.com/docs/en/agent-sdk/typescript#createsdkmcpserver) pour garder le schéma complet d'un outil dans le prompt initial.

### Ajouter des annotations d'outil

Les [annotations d'outil](https://modelcontextprotocol.io/docs/concepts/tools#tool-annotations) sont des métadonnées facultatives décrivant le comportement d'un outil. Passez-les comme cinquième argument du helper `tool()` en TypeScript, ou via l'argument nommé `annotations` du décorateur `@tool` en Python. Tous les champs d'indication (hint) sont des booléens.

| Champ | Valeur par défaut | Signification |
| :- | :- | :- |
| `readOnlyHint` | `false` | L'outil ne modifie pas son environnement. Détermine si l'outil peut être appelé en parallèle avec d'autres outils en lecture seule. |
| `destructiveHint` | `true` | L'outil peut effectuer des modifications destructrices. Purement informatif. |
| `idempotentHint` | `false` | Des appels répétés avec les mêmes arguments n'ont pas d'effet supplémentaire. Purement informatif. |
| `openWorldHint` | `true` | L'outil atteint des systèmes extérieurs à votre processus. Purement informatif. |

Les annotations sont des métadonnées, pas un mécanisme de contrôle. Un outil marqué `readOnlyHint: true` peut tout de même écrire sur le disque si c'est ce que fait son handler. Gardez l'annotation fidèle au comportement réel du handler.

Cet exemple ajoute `readOnlyHint` à l'outil `get_temperature` de l'[exemple d'outil météo](#exemple--outil-météo).

#### Python

```python
from claude_agent_sdk import tool, ToolAnnotations


@tool(
    "get_temperature",
    "Get the current temperature at a location",
    {"latitude": float, "longitude": float},
    annotations=ToolAnnotations(
        readOnlyHint=True
    ),  # Permet à Claude de regrouper cet appel avec d'autres appels en lecture seule
)
async def get_temperature(args):
    return {"content": [{"type": "text", "text": "..."}]}
```

#### TypeScript

```typescript
import { tool } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

tool(
  "get_temperature",
  "Get the current temperature at a location",
  { latitude: z.number(), longitude: z.number() },
  async (args) => ({ content: [{ type: "text", text: `...` }] }),
  { annotations: { readOnlyHint: true } } // Permet à Claude de regrouper cet appel avec d'autres appels en lecture seule
);
```

Voir `ToolAnnotations` dans la référence [TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript#toolannotations) ou [Python](https://code.claude.com/docs/en/agent-sdk/python#toolannotations).

---

## Contrôler l'accès aux outils

L'[exemple d'outil météo](#exemple--outil-météo) a enregistré un serveur et listé des outils dans `allowedTools`. Cette section explique comment délimiter l'accès lorsque vous avez plusieurs outils ou que vous voulez restreindre les outils intégrés. Pour la construction des noms d'outils, voir [Appeler un outil personnalisé](#appeler-un-outil-personnalisé).

### Configurer les outils autorisés

L'option `tools` et les listes d'outils autorisés/interdits agissent sur deux couches : la **disponibilité**, qui détermine si un outil apparaît dans le contexte de Claude, et la **permission**, qui détermine si un appel est approuvé une fois que Claude le tente. `tools` et les entrées `disallowedTools` sous forme de nom nu modifient la disponibilité. `allowedTools` et les règles `disallowedTools` avec portée (scoped) modifient la permission. Si vous nommez l'un des [outils de suivi des tâches](https://code.claude.com/docs/en/agent-sdk/todo-tracking#model-availability) dans `allowedTools`, Claude Code active aussi ces outils pour la session.

| Option | Couche | Effet |
| :- | :- | :- |
| `tools: ["Read", "Grep"]` | Disponibilité | Seuls les outils intégrés listés sont dans le contexte de Claude. Les outils intégrés non listés sont retirés. Les outils MCP ne sont pas affectés. |
| `tools: []` | Disponibilité | Tous les outils intégrés sont retirés. Claude ne peut utiliser que vos outils MCP. |
| outils autorisés (allowed tools) | Permission | Les outils listés s'exécutent sans demande d'autorisation. Les autres outils non listés restent disponibles ; leurs appels passent par le [flux de permissions](https://code.claude.com/docs/en/agent-sdk/permissions). |
| outils interdits (disallowed tools) | Les deux | Un nom d'outil nu comme `"Bash"` retire l'outil du contexte de Claude, comme si on l'omettait de `tools`. Une règle avec portée comme `"Bash(rm *)"` laisse l'outil dans le contexte et ne refuse que les appels qui correspondent [tels qu'écrits](https://code.claude.com/docs/en/permissions#bash-rule-limits). |

Pour retirer complètement un outil intégré, omettez-le de `tools` ou listez son nom nu dans `disallowedTools` (Python : `disallowed_tools`) ; dans les deux cas, l'outil reste hors du contexte et Claude ne tente jamais de l'appeler. Une règle `disallowedTools` avec portée bloque les appels correspondants mais laisse l'outil visible, si bien que Claude peut gaspiller un tour à essayer de l'utiliser. Voir [Configurer les permissions](https://code.claude.com/docs/en/agent-sdk/permissions) pour l'ordre d'évaluation complet.

---

## Gérer les erreurs

Une erreur dans un handler n'arrête pas la boucle de l'agent. Le serveur MCP in-process du SDK intercepte les exceptions non gérées et les renvoie comme résultats d'erreur ; la façon dont vous signalez une erreur détermine donc ce que lit Claude, et non si la requête échoue :

| Ce qui se passe | Résultat |
| :- | :- |
| Le handler lève une exception non interceptée | Le serveur MCP la convertit en résultat d'erreur portant le message brut de l'exception. Claude voit ce message, et la boucle de l'agent continue. |
| Le handler intercepte l'erreur et renvoie `isError: true` (TS) / `"is_error": True` (Python) | Claude voit le message que vous composez. Vous pouvez ajouter le contexte qui manque à l'exception brute, par exemple quelle requête a échoué ou quoi essayer à la place. |

Dans les deux cas, Claude peut réessayer, essayer un autre outil ou expliquer l'échec. Interceptez vous-même les erreurs quand le message brut de l'exception ne suffit pas à Claude pour agir.

L'exemple ci-dessous intercepte deux types d'échecs dans le handler et compose le message d'erreur que lit Claude. Un statut HTTP différent de 200 est détecté à partir de la réponse et renvoyé comme résultat d'erreur. Une erreur réseau ou un JSON invalide est intercepté par le `try/except` (Python) ou `try/catch` (TypeScript) englobant et renvoyé lui aussi comme résultat d'erreur. Dans les deux cas, Claude reçoit un message qui décrit l'échec au lieu d'une simple chaîne d'exception.

#### Python

```python
import json
import httpx
from typing import Any
from claude_agent_sdk import tool


@tool(
    "fetch_data",
    "Fetch data from an API",
    {"endpoint": str},  # Schéma simple
)
async def fetch_data(args: dict[str, Any]) -> dict[str, Any]:
    try:
        async with httpx.AsyncClient() as client:
            response = await client.get(args["endpoint"])
            if response.status_code != 200:
                # Renvoyer l'échec comme résultat d'outil pour que Claude puisse y réagir.
                # is_error marque l'appel comme échoué plutôt que comme des données étranges.
                return {
                    "content": [
                        {
                            "type": "text",
                            "text": f"API error: {response.status_code} {response.reason_phrase}",
                        }
                    ],
                    "is_error": True,
                }

            data = response.json()
            return {"content": [{"type": "text", "text": json.dumps(data, indent=2)}]}
    except Exception as e:
        # Compose le message que lit Claude. Une exception non interceptée
        # parviendrait à Claude sous forme de str(e) brut, sans contexte.
        return {
            "content": [{"type": "text", "text": f"Failed to fetch data: {str(e)}"}],
            "is_error": True,
        }
```

#### TypeScript

```typescript
import { tool } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

tool(
  "fetch_data",
  "Fetch data from an API",
  {
    endpoint: z.string().url().describe("API endpoint URL")
  },
  async (args) => {
    try {
      const response = await fetch(args.endpoint);

      if (!response.ok) {
        // Renvoyer l'échec comme résultat d'outil pour que Claude puisse y réagir.
        // isError marque l'appel comme échoué plutôt que comme des données étranges.
        return {
          content: [
            {
              type: "text",
              text: `API error: ${response.status} ${response.statusText}`
            }
          ],
          isError: true
        };
      }

      const data = await response.json();
      return {
        content: [
          {
            type: "text",
            text: JSON.stringify(data, null, 2)
          }
        ]
      };
    } catch (error) {
      // Compose le message que lit Claude. Une exception non interceptée
      // parviendrait à Claude sous forme de message d'erreur brut, sans contexte.
      return {
        content: [
          {
            type: "text",
            text: `Failed to fetch data: ${error instanceof Error ? error.message : String(error)}`
          }
        ],
        isError: true
      };
    }
  }
);
```

---

## Renvoyer des images et des ressources

Le tableau `content` d'un résultat d'outil accepte des blocs `text`, `image`, `audio`, `resource` et `resource_link`. Vous pouvez les mélanger dans une même réponse. En TypeScript, le SDK enregistre les blocs audio sur le disque et Claude reçoit un bloc texte contenant le chemin du fichier enregistré ; en Python, le SDK supprime les blocs audio du résultat de l'outil et journalise un avertissement.

Claude reçoit chaque bloc de lien de ressource (resource link) sous la forme d'un bloc texte contenant le nom, l'URI et la description du lien. En TypeScript, votre application reçoit aussi les liens eux-mêmes sous forme de [`resourceLinks`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkmcpresourcelink) dans le `tool_use_result` du message utilisateur ; en Python, le SDK les aplatit en texte avant que le CLI ne voie le résultat, si bien que la [clé `resourceLinks`](https://code.claude.com/docs/en/agent-sdk/python#usermessage) Python n'est jamais produite pour les outils in-process.

### Images

Un bloc image transporte les octets de l'image directement (inline), encodés en base64. Il n'existe pas de champ URL. Pour renvoyer une image accessible à une URL, récupérez-la dans le handler, lisez les octets de la réponse et encodez-les en base64 avant de les renvoyer. Une image PNG, JPEG, GIF ou WebP parvient à Claude comme entrée visuelle ; une image de tout autre type est enregistrée sur le disque et Claude reçoit à la place son chemin de fichier sous forme de texte.

| Champ | Type | Remarques |
| :- | :- | :- |
| `type` | `"image"` | |
| `data` | `string` | Octets encodés en base64. Base64 brut uniquement, sans préfixe `data:image/...;base64,` |
| `mimeType` | `string` | Obligatoire. Par exemple `image/png`, `image/jpeg`, `image/webp`, `image/gif` |

#### Python

```python
import base64
import httpx
from claude_agent_sdk import tool


# Définir un outil qui récupère une image depuis une URL et la renvoie à Claude
@tool("fetch_image", "Fetch an image from a URL and return it to Claude", {"url": str})
async def fetch_image(args):
    async with httpx.AsyncClient() as client:  # Récupérer les octets de l'image
        response = await client.get(args["url"])

    return {
        "content": [
            {
                "type": "image",
                "data": base64.b64encode(response.content).decode(
                    "ascii"
                ),  # Encoder les octets bruts en base64
                "mimeType": response.headers.get(
                    "content-type", "image/png"
                ),  # Lire le type MIME depuis la réponse
            }
        ]
    }
```

#### TypeScript

```typescript
import { tool } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

tool(
  "fetch_image",
  "Fetch an image from a URL and return it to Claude",
  {
    url: z.string().url()
  },
  async (args) => {
    const response = await fetch(args.url); // Récupérer les octets de l'image
    const buffer = Buffer.from(await response.arrayBuffer()); // Lire dans un Buffer pour l'encodage base64
    const mimeType = response.headers.get("content-type") ?? "image/png";

    return {
      content: [
        {
          type: "image",
          data: buffer.toString("base64"), // Encoder les octets bruts en base64
          mimeType
        }
      ]
    };
  }
);
```

### Ressources

Un bloc ressource intègre un contenu identifié par une URI. Le contenu réel est transporté dans le champ `text` ou `blob` du bloc. Utilisez-le quand votre outil produit un fichier généré ou un enregistrement provenant d'un système externe.

| Champ | Type | Remarques |
| :- | :- | :- |
| `type` | `"resource"` | |
| `resource.uri` | `string` | Identifiant du contenu. N'importe quel schéma d'URI |
| `resource.text` | `string` | Le contenu, s'il est textuel. Fournissez ce champ ou `blob`, pas les deux |
| `resource.blob` | `string` | Le contenu encodé en base64, s'il est binaire. TypeScript uniquement : le SDK Python supprime les ressources binaires du résultat de l'outil et journalise un avertissement |
| `resource.mimeType` | `string` | Facultatif |

Cet exemple montre un bloc ressource renvoyé depuis un handler d'outil. Le SDK ne lit pas l'URI de l'exemple, `file:///tmp/report.md`.

#### TypeScript

```typescript
return {
  content: [
    {
      type: "resource",
      resource: {
        uri: "file:///tmp/report.md", // Étiquette que Claude peut référencer, pas un chemin lu par le SDK
        mimeType: "text/markdown",
        text: "# Report\n..." // Le contenu réel, inline
      }
    }
  ]
};
```

#### Python

```python
return {
    "content": [
        {
            "type": "resource",
            "resource": {
                "uri": "file:///tmp/report.md",  # Pas un chemin lu par le SDK
                "mimeType": "text/markdown",
                "text": "# Report\n...",  # Le contenu réel, inline
            },
        }
    ]
}
```

Ces formes de blocs proviennent du type MCP `CallToolResult`. Voir la [spécification MCP](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#tool-result) pour la définition complète.

---

## Renvoyer des données structurées

`structuredContent` est un objet JSON facultatif sur le résultat, distinct du tableau `content`. Utilisez-le pour renvoyer des valeurs brutes que Claude peut lire comme des champs exacts, au lieu de devoir les extraire d'une chaîne de texte ou d'une image.

Lorsque `structuredContent` est défini, Claude reçoit le JSON ainsi que les éventuels blocs image ou ressource de `content`. Les blocs texte de `content` ne sont **pas** transmis, car ils sont supposés faire doublon avec les données structurées. L'exemple ci-dessous génère un graphique sous forme de bloc image et renvoie, depuis le même handler, les points de données sous-jacents dans `structuredContent`. Dans l'extrait, `chartPngBuffer` est un `Buffer` contenant les octets PNG du graphique rendu.

#### TypeScript

```typescript
return {
  content: [
    {
      type: "image",
      data: chartPngBuffer.toString("base64"),
      mimeType: "image/png"
    }
  ],
  structuredContent: {
    series: "temperature_2m",
    unit: "fahrenheit",
    points: [62.1, 63.4, 65.0, 64.2]
  }
};
```

> **Note** : le décorateur Python `@tool` ne transmet que `content` et `is_error` du dict renvoyé par le handler. Pour renvoyer `structuredContent` depuis Python, exécutez un [serveur MCP autonome](https://code.claude.com/docs/en/agent-sdk/mcp) au lieu d'un serveur SDK in-process.

---

## Exemple : convertisseur d'unités

Cet outil convertit des valeurs entre unités de longueur, de température et de poids. Un utilisateur peut demander « convert 100 kilometers to miles » ou « what is 72°F in Celsius », et Claude choisit le bon type d'unité et les bonnes unités à partir de la demande.

Il illustre deux approches :

- **Schémas à enum** : `unit_type` est limité à un ensemble fixe de valeurs. En TypeScript, utilisez `z.enum()`. En Python, le schéma sous forme de dict ne prend pas en charge les enums ; le dict JSON Schema complet est donc nécessaire.
- **Gestion des entrées non prises en charge** : lorsqu'une paire de conversion est introuvable, le handler renvoie `isError: true` pour que Claude puisse expliquer à l'utilisateur ce qui n'a pas fonctionné, au lieu de traiter l'échec comme un résultat normal.

#### Python

```python
from typing import Any
from claude_agent_sdk import tool, create_sdk_mcp_server


# z.enum() en TypeScript devient une contrainte "enum" en JSON Schema.
# Le schéma sous forme de dict n'a pas d'équivalent, donc le JSON Schema complet est nécessaire.
@tool(
    "convert_units",
    "Convert a value from one unit to another",
    {
        "type": "object",
        "properties": {
            "unit_type": {
                "type": "string",
                "enum": ["length", "temperature", "weight"],
                "description": "Category of unit",
            },
            "from_unit": {
                "type": "string",
                "description": "Unit to convert from, e.g. kilometers, fahrenheit, pounds",
            },
            "to_unit": {"type": "string", "description": "Unit to convert to"},
            "value": {"type": "number", "description": "Value to convert"},
        },
        "required": ["unit_type", "from_unit", "to_unit", "value"],
    },
)
async def convert_units(args: dict[str, Any]) -> dict[str, Any]:
    conversions = {
        "length": {
            "kilometers_to_miles": lambda v: v * 0.621371,
            "miles_to_kilometers": lambda v: v * 1.60934,
            "meters_to_feet": lambda v: v * 3.28084,
            "feet_to_meters": lambda v: v * 0.3048,
        },
        "temperature": {
            "celsius_to_fahrenheit": lambda v: (v * 9) / 5 + 32,
            "fahrenheit_to_celsius": lambda v: (v - 32) * 5 / 9,
            "celsius_to_kelvin": lambda v: v + 273.15,
            "kelvin_to_celsius": lambda v: v - 273.15,
        },
        "weight": {
            "kilograms_to_pounds": lambda v: v * 2.20462,
            "pounds_to_kilograms": lambda v: v * 0.453592,
            "grams_to_ounces": lambda v: v * 0.035274,
            "ounces_to_grams": lambda v: v * 28.3495,
        },
    }

    key = f"{args['from_unit']}_to_{args['to_unit']}"
    fn = conversions.get(args["unit_type"], {}).get(key)

    if not fn:
        return {
            "content": [
                {
                    "type": "text",
                    "text": f"Unsupported conversion: {args['from_unit']} to {args['to_unit']}",
                }
            ],
            "is_error": True,
        }

    result = fn(args["value"])
    return {
        "content": [
            {
                "type": "text",
                "text": f"{args['value']} {args['from_unit']} = {result:.4f} {args['to_unit']}",
            }
        ]
    }


converter_server = create_sdk_mcp_server(
    name="converter",
    version="1.0.0",
    tools=[convert_units],
)
```

#### TypeScript

```typescript
import { tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

const convert = tool(
  "convert_units",
  "Convert a value from one unit to another",
  {
    unit_type: z.enum(["length", "temperature", "weight"]).describe("Category of unit"),
    from_unit: z
      .string()
      .describe("Unit to convert from, e.g. kilometers, fahrenheit, pounds"),
    to_unit: z.string().describe("Unit to convert to"),
    value: z.number().describe("Value to convert")
  },
  async (args) => {
    type Conversions = Record<string, Record<string, (v: number) => number>>;

    const conversions: Conversions = {
      length: {
        kilometers_to_miles: (v) => v * 0.621371,
        miles_to_kilometers: (v) => v * 1.60934,
        meters_to_feet: (v) => v * 3.28084,
        feet_to_meters: (v) => v * 0.3048
      },
      temperature: {
        celsius_to_fahrenheit: (v) => (v * 9) / 5 + 32,
        fahrenheit_to_celsius: (v) => ((v - 32) * 5) / 9,
        celsius_to_kelvin: (v) => v + 273.15,
        kelvin_to_celsius: (v) => v - 273.15
      },
      weight: {
        kilograms_to_pounds: (v) => v * 2.20462,
        pounds_to_kilograms: (v) => v * 0.453592,
        grams_to_ounces: (v) => v * 0.035274,
        ounces_to_grams: (v) => v * 28.3495
      }
    };

    const key = `${args.from_unit}_to_${args.to_unit}`;
    const fn = conversions[args.unit_type]?.[key];

    if (!fn) {
      return {
        content: [
          {
            type: "text",
            text: `Unsupported conversion: ${args.from_unit} to ${args.to_unit}`
          }
        ],
        isError: true
      };
    }

    const result = fn(args.value);
    return {
      content: [
        {
          type: "text",
          text: `${args.value} ${args.from_unit} = ${result.toFixed(4)} ${args.to_unit}`
        }
      ]
    };
  }
);

const converterServer = createSdkMcpServer({
  name: "converter",
  version: "1.0.0",
  tools: [convert]
});
```

Une fois le serveur défini, passez-le à `query` de la même manière que dans l'exemple météo. Cet exemple envoie trois prompts différents dans une boucle pour montrer le même outil traitant différents types d'unités. Pour chaque réponse, il inspecte les objets `AssistantMessage` (qui contiennent les appels d'outils effectués par Claude pendant ce tour) et affiche chaque `ToolUseBlock` avant d'afficher le texte final du `ResultMessage`. Cela permet de voir quand Claude utilise l'outil et quand il répond à partir de ses propres connaissances.

Comme la [recherche d'outils](https://code.claude.com/docs/en/agent-sdk/tool-search) est activée par défaut, la sortie peut aussi inclure un appel `ToolSearch` lorsque Claude charge le schéma différé de l'outil.

#### Python

```python
import asyncio
from claude_agent_sdk import (
    query,
    ClaudeAgentOptions,
    ResultMessage,
    AssistantMessage,
    ToolUseBlock,
)


async def main():
    options = ClaudeAgentOptions(
        mcp_servers={"converter": converter_server},
        allowed_tools=["mcp__converter__convert_units"],
    )

    prompts = [
        "Convert 100 kilometers to miles.",
        "What is 72°F in Celsius?",
        "How many pounds is 5 kilograms?",
    ]

    for prompt in prompts:
        try:
            async for message in query(prompt=prompt, options=options):
                if isinstance(message, AssistantMessage):
                    for block in message.content:
                        if isinstance(block, ToolUseBlock):
                            print(f"[tool call] {block.name}({block.input})")
                elif isinstance(message, ResultMessage) and message.subtype == "success":
                    print(f"Q: {prompt}\nA: {message.result}\n")
        except Exception as error:
            # Un query() à usage unique lève une exception après avoir émis un résultat d'erreur.
            # Seuls les résultats de succès sont affichés ci-dessus : on gère donc l'échec ici
            # et on continue avec le prompt suivant.
            print(f"Call failed: {error}")


asyncio.run(main())
```

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

const prompts = [
  "Convert 100 kilometers to miles.",
  "What is 72°F in Celsius?",
  "How many pounds is 5 kilograms?"
];

for (const prompt of prompts) {
  try {
    for await (const message of query({
      prompt,
      options: {
        mcpServers: { converter: converterServer },
        allowedTools: ["mcp__converter__convert_units"]
      }
    })) {
      if (message.type === "assistant") {
        for (const block of message.message.content) {
          if (block.type === "tool_use") {
            console.log(`[tool call] ${block.name}`, block.input);
          }
        }
      } else if (message.type === "result" && message.subtype === "success") {
        console.log(`Q: ${prompt}\nA: ${message.result}\n`);
      }
    }
  } catch (error) {
    // Un query() à usage unique lève une exception après avoir émis un résultat d'erreur.
    // Seuls les résultats de succès sont journalisés ci-dessus : on gère donc l'échec ici
    // et on continue avec le prompt suivant.
    console.error(`Call failed: ${error}`);
  }
}
```

---

## Étapes suivantes

Vous pouvez combiner les approches de cette page dans un même serveur : un seul serveur peut contenir côte à côte un outil de base de données, un outil passerelle d'API et un outil de rendu d'images.

Pour aller plus loin :

- Si votre serveur grossit jusqu'à des dizaines d'outils, voir la [recherche d'outils (tool search)](https://code.claude.com/docs/en/agent-sdk/tool-search) pour différer leur chargement jusqu'à ce que Claude en ait besoin.
- Pour vous connecter à des serveurs MCP externes (système de fichiers, GitHub, Slack) au lieu de construire le vôtre, voir [Se connecter à des serveurs MCP](https://code.claude.com/docs/en/agent-sdk/mcp).
- Pour contrôler quels outils s'exécutent automatiquement et lesquels nécessitent une approbation, voir [Configurer les permissions](https://code.claude.com/docs/en/agent-sdk/permissions).
