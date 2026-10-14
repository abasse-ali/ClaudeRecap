# Se connecter à des outils externes avec MCP (Agent SDK)

> Source : https://code.claude.com/docs/en/agent-sdk/mcp · [← Index](../31-index-documentation-officielle.md)

*Configurer des serveurs MCP pour étendre votre agent avec des outils externes. Couvre les types de transport, la recherche d'outils (tool search) pour les grands ensembles d'outils, l'authentification et la gestion des erreurs.*

Le [Model Context Protocol (MCP)](https://modelcontextprotocol.io/docs/getting-started/intro) est un standard ouvert pour connecter des agents IA à des outils et sources de données externes. Avec MCP, votre agent peut interroger des bases de données, s'intégrer à des API comme Slack et GitHub, et se connecter à d'autres services sans écrire d'implémentations d'outils personnalisées.

Les serveurs MCP peuvent tourner comme processus locaux, se connecter via HTTP, ou s'exécuter directement au sein de votre application SDK.

> **Note** : cette page couvre la configuration MCP pour l'Agent SDK. Pour ajouter des serveurs MCP au CLI Claude Code afin qu'ils se chargent dans tous les projets, voir [MCP installation scopes](https://code.claude.com/docs/en/mcp#mcp-installation-scopes).

---

## Démarrage rapide

Cet exemple se connecte au serveur MCP de la [documentation Claude Code](https://code.claude.com/docs) en utilisant le [transport HTTP](#serveurs-httpsse) et utilise [`allowedTools`](#autoriser-les-outils-mcp) avec un joker pour autoriser tous les outils du serveur.

### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Use the docs MCP server to explain what hooks are in Claude Code",
  options: {
    mcpServers: {
      "claude-code-docs": {
        type: "http",
        url: "https://code.claude.com/docs/mcp"
      }
    },
    allowedTools: ["mcp__claude-code-docs__*"]
  }
})) {
  if (message.type === "result" && message.subtype === "success") {
    console.log(message.result);
  }
}
```

### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


async def main():
    options = ClaudeAgentOptions(
        mcp_servers={
            "claude-code-docs": {
                "type": "http",
                "url": "https://code.claude.com/docs/mcp",
            }
        },
        allowed_tools=["mcp__claude-code-docs__*"],
    )

    async for message in query(
        prompt="Use the docs MCP server to explain what hooks are in Claude Code",
        options=options,
    ):
        if isinstance(message, ResultMessage) and message.subtype == "success":
            print(message.result)


asyncio.run(main())
```

L'agent se connecte au serveur de documentation, cherche des informations sur les hooks et renvoie les résultats.

---

## Ajouter un serveur MCP

Vous pouvez configurer des serveurs MCP dans le code lors de l'appel à `query()`, ou dans un fichier `.mcp.json` chargé via [`settingSources`](#ajouter-un-serveur-depuis-un-fichier-de-configuration).

### Ajouter un serveur dans le code

Passez les serveurs MCP directement dans l'option `mcpServers`. Cet exemple démarre un serveur MCP local de système de fichiers pour `/Users/me/projects`. Remplacez ce chemin par un répertoire de votre machine.

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "List files in my project",
  options: {
    mcpServers: {
      filesystem: {
        command: "npx",
        args: ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"]
      }
    },
    allowedTools: ["mcp__filesystem__*"]
  }
})) {
  if (message.type === "result" && message.subtype === "success") {
    console.log(message.result);
  }
}
```

#### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


async def main():
    options = ClaudeAgentOptions(
        mcp_servers={
            "filesystem": {
                "command": "npx",
                "args": [
                    "-y",
                    "@modelcontextprotocol/server-filesystem",
                    "/Users/me/projects",
                ],
            }
        },
        allowed_tools=["mcp__filesystem__*"],
    )

    async for message in query(prompt="List files in my project", options=options):
        if isinstance(message, ResultMessage) and message.subtype == "success":
            print(message.result)


asyncio.run(main())
```

### Ajouter un serveur depuis un fichier de configuration

Créez un fichier `.mcp.json` à la racine de votre projet. Le fichier est pris en compte lorsque la source de réglages `project` est activée, ce qui est le cas avec les options par défaut de `query()`. Si vous définissez `settingSources` explicitement, incluez `"project"` pour que ce fichier se charge. Remplacez `/Users/me/projects` par un répertoire de votre machine :

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"]
    }
  }
}
```

---

## Moment de la connexion (connection timing)

Claude Code enregistre au démarrage les serveurs passés dans `options.mcpServers` et émet le [message init](#gestion-des-erreurs) une fois l'éventuelle attente du premier tour résolue. Le fait qu'un serveur de `options.mcpServers` retarde ou non le premier tour, et le moment où il se connecte, dépendent de son type :

| Type de serveur | Retarde le premier tour ? | Délai d'attente du premier tour |
| :- | :- | :- |
| Serveur stdio, ou serveur HTTP/SSE sans liste d'outils en cache | Oui, jusqu'à ce qu'il se connecte | [`MCP_TIMEOUT`](https://code.claude.com/docs/en/env-vars), 30 secondes par défaut ; la connexion échoue à cette échéance |
| Serveur distant avec une liste d'outils en cache, enregistrée par Claude Code lors d'une connexion précédente | Non ; les outils en cache sont disponibles dès le premier tour | Aucun ; il se connecte à son premier appel d'outil, et cette connexion différée a son propre délai d'expiration |
| [Serveur SDK](#serveurs-mcp-sdk) en processus (in-process) | Oui, jusqu'à ce qu'il se connecte et liste ses outils | [`MCP_TIMEOUT`](https://code.claude.com/docs/en/env-vars), 30 secondes par défaut, par tentative de connexion ; la connexion échoue à cette échéance |

Les serveurs chargés depuis des [fichiers de réglages](#ajouter-un-serveur-depuis-un-fichier-de-configuration) comme `.mcp.json` ou depuis des plugins affichent souvent `pending` dans le message init. Quand `options.mcpServers` contient un serveur stdio, HTTP ou SSE, le premier tour attend aussi ces serveurs en attente, jusqu'à `MCP_TIMEOUT`. Quand `options.mcpServers` est vide ou ne contient que des serveurs SDK, le premier tour attend au maximum 2 secondes :

- **Avec la [recherche d'outils (tool search)](https://code.claude.com/docs/en/agent-sdk/tool-search), comportement par défaut** : l'attente couvre les serveurs encore en attente configurés avec [`alwaysLoad: true`](https://code.claude.com/docs/en/mcp#exempt-a-server-from-deferral), et pas les autres. Les autres continuent de se connecter en arrière-plan. [Tool availability](https://code.claude.com/docs/en/mcp#tool-availability) décrit comment Claude accède à leurs outils une fois connectés.
- **Sans recherche d'outils** : l'attente couvre tous les serveurs en attente. [Configure tool search](https://code.claude.com/docs/en/agent-sdk/tool-search#configure-tool-search) explique ce qui désactive la recherche d'outils. Si vous excluez l'outil `ToolSearch` de la session, par exemple via `disallowedTools`, la session tourne aussi sans recherche d'outils.

Si vous définissez `permissionPromptToolName`, le premier tour attend aussi, dans tous les cas, le serveur de cet outil, jusqu'à `MCP_TIMEOUT`.

Pour régler vous-même l'attente du premier tour, ajoutez `CLAUDE_CODE_MCP_STARTUP_WAIT_MS` à l'[option `env`](https://code.claude.com/docs/en/agent-sdk/configuration#set-environment-variables), par exemple `CLAUDE_CODE_MCP_STARTUP_WAIT_MS: "5000"`. Le premier tour attend alors jusqu'à ce nombre de millisecondes pour chaque serveur en attente, que la recherche d'outils soit disponible ou non. Cette échéance remplace aussi l'attente `MCP_TIMEOUT` du premier tour pour les serveurs stdio, HTTP et SSE de `options.mcpServers`. `CLAUDE_CODE_MCP_STARTUP_WAIT_MS` nécessite Claude Code (v2.1.274+).

Les serveurs encore en attente à la fin de l'attente continuent de se connecter en arrière-plan. Réglez la variable à `0` pour sauter l'attente. Un serveur `permissionPromptToolName` conserve sa propre attente `MCP_TIMEOUT` quelle que soit la valeur.

Pour bloquer le démarrage lui-même, dans une phase distincte et antérieure à l'attente du premier tour (avant l'envoi du message init) :

- Réglez [`MCP_CONNECTION_NONBLOCKING`](https://code.claude.com/docs/en/env-vars) à `0` pour bloquer sur l'ensemble du lot de connexions. Claude Code plafonne cette attente à 5 secondes par défaut. Ajustez le plafond avec la variable d'environnement [`MCP_CONNECT_TIMEOUT_MS`](https://code.claude.com/docs/en/env-vars), en millisecondes. Les serveurs encore en attente à cette échéance continuent de se connecter en arrière-plan.
- Réglez `alwaysLoad: true` dans la configuration d'un serveur pour que ses outils soient disponibles avec leurs schémas complets dès le premier tour, [exemptés du report de la recherche d'outils](https://code.claude.com/docs/en/mcp#exempt-a-server-from-deferral). Claude Code attend au démarrage les outils de ce serveur, plafonné à la même échéance, tandis que les autres serveurs continuent de se connecter en arrière-plan ; un serveur distant avec une liste d'outils en cache les fournit sans se connecter, conformément au tableau ci-dessus.

Le message `system` de sous-type `init` rapporte le statut de chaque serveur au moment où il est émis ; voir [Gestion des erreurs](#gestion-des-erreurs) pour lire ces statuts.

---

## Autoriser les outils MCP

Les outils MCP nécessitent une permission explicite avant que Claude puisse les utiliser. Sans permission, Claude verra que les outils sont disponibles mais ne pourra pas les appeler.

### Convention de nommage des outils

Les outils MCP suivent le motif de nommage `mcp__<server-name>__<tool-name>`. Par exemple, un serveur GitHub nommé `"github"` avec un outil `list_issues` devient `mcp__github__list_issues`.

### Approbation automatique avec allowedTools

Utilisez `allowedTools` pour approuver automatiquement des outils MCP précis, afin que Claude puisse les utiliser sans demande de permission.

#### TypeScript

```typescript
const _ = {
  options: {
    mcpServers: {
      // vos serveurs
    },
    allowedTools: [
      "mcp__github__*", // Tous les outils du serveur github
      "mcp__db__query", // Uniquement l'outil query du serveur db
      "mcp__slack__send_message" // Uniquement send_message du serveur slack
    ]
  }
};
```

#### Python

```python
options = ClaudeAgentOptions(
    mcp_servers={
        # vos serveurs
    },
    allowed_tools=[
        "mcp__github__*",  # Tous les outils du serveur github
        "mcp__db__query",  # Uniquement l'outil query du serveur db
        "mcp__slack__send_message",  # Uniquement send_message du serveur slack
    ],
)
```

Les jokers (`*`) permettent d'autoriser tous les outils d'un serveur sans les lister un par un.

> **Note** : **Préférez `allowedTools` aux modes de permission pour l'accès MCP.** `permissionMode: "acceptEdits"` n'approuve pas automatiquement les outils MCP (seulement les modifications de fichiers et les commandes Bash de système de fichiers). `permissionMode: "bypassPermissions"` approuve bien automatiquement les outils MCP, mais désactive aussi la plupart des autres demandes de sécurité, ce qui est plus large que nécessaire ; voir [How permissions are evaluated](https://code.claude.com/docs/en/agent-sdk/permissions#how-permissions-are-evaluated) pour les demandes qui subsistent. Un joker dans `allowedTools` accorde exactement le serveur MCP voulu et rien de plus. Voir [Permission modes](https://code.claude.com/docs/en/agent-sdk/permissions#permission-modes) pour une comparaison complète.

### Découvrir les outils disponibles

Pour voir quels outils fournit un serveur MCP, consultez la documentation du serveur ou inspectez le tableau `tools` du message `system` init. Les noms d'outils MCP commencent par `mcp__`.

Claude Code émet le message init après l'[attente de connexion du premier tour](#moment-de-la-connexion-connection-timing) pour les serveurs passés dans `options.mcpServers` ; le tableau `tools` liste donc les outils `mcp__` de chaque serveur connecté à ce moment-là, plus ceux des serveurs ayant une [liste d'outils en cache](#moment-de-la-connexion-connection-timing), qui se connectent à la première utilisation. Les outils de tout autre serveur non encore connecté sont absents ; voir [Gestion des erreurs](#gestion-des-erreurs) pour lire le statut de chaque serveur.

Ce filtre affiche les noms des outils MCP :

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

const options = {
  mcpServers: {
    // vos serveurs
  },
};

for await (const message of query({ prompt: "...", options })) {
  if (message.type === "system" && message.subtype === "init") {
    const mcpTools = message.tools.filter((name) => name.startsWith("mcp__"));
    console.log("Available MCP tools:", mcpTools);
  }
}
```

#### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, SystemMessage


async def main():
    options = ClaudeAgentOptions(
        mcp_servers={
            # vos serveurs
        },
    )
    async for message in query(prompt="...", options=options):
        if isinstance(message, SystemMessage) and message.subtype == "init":
            mcp_tools = [t for t in message.data.get("tools", []) if t.startswith("mcp__")]
            print("Available MCP tools:", mcp_tools)


asyncio.run(main())
```

Vous pouvez aussi demander à Claude de lister les outils disponibles d'un serveur.

---

## Types de transport

Les serveurs MCP communiquent avec votre agent via différents protocoles de transport. Consultez la documentation du serveur pour savoir lequel il prend en charge :

- Si la doc vous donne une **commande à exécuter** (comme `npx @modelcontextprotocol/server-filesystem`), utilisez stdio.
- Si la doc vous donne une **URL**, utilisez HTTP ou SSE.
- Si vous construisez vos propres outils dans le code, utilisez un serveur MCP SDK.

### Serveurs stdio

Processus locaux qui communiquent via stdin/stdout. À utiliser pour les serveurs MCP exécutés sur la même machine. Pour la forme `.mcp.json`, utilisez les mêmes champs que dans [Ajouter un serveur depuis un fichier de configuration](#ajouter-un-serveur-depuis-un-fichier-de-configuration). Dans le code, passez la commande et ses arguments. Remplacez `/Users/me/projects` par un répertoire de votre machine.

#### TypeScript

```typescript
const _ = {
  options: {
    mcpServers: {
      filesystem: {
        command: "npx",
        args: ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"]
      }
    },
    allowedTools: ["mcp__filesystem__read_file", "mcp__filesystem__list_directory"]
  }
};
```

#### Python

```python
options = ClaudeAgentOptions(
    mcp_servers={
        "filesystem": {
            "command": "npx",
            "args": [
                "-y",
                "@modelcontextprotocol/server-filesystem",
                "/Users/me/projects",
            ],
        }
    },
    allowed_tools=["mcp__filesystem__read_file", "mcp__filesystem__list_directory"],
)
```

### Serveurs HTTP/SSE

Utilisez HTTP ou SSE pour les serveurs MCP hébergés dans le cloud et les API distantes. Pour la forme `.mcp.json`, utilisez les mêmes champs que l'exemple de [En-têtes HTTP pour les serveurs distants](#en-têtes-http-pour-les-serveurs-distants), avec `"type": "sse"` pour un serveur SSE. Dans le code, passez l'URL du serveur.

#### TypeScript

```typescript
const _ = {
  options: {
    mcpServers: {
      "remote-api": {
        type: "sse",
        url: "https://api.example.com/mcp/sse",
        headers: {
          Authorization: `Bearer ${process.env.API_TOKEN}`
        }
      }
    },
    allowedTools: ["mcp__remote-api__*"]
  }
};
```

#### Python

```python
options = ClaudeAgentOptions(
    mcp_servers={
        "remote-api": {
            "type": "sse",
            "url": "https://api.example.com/mcp/sse",
            "headers": {"Authorization": f"Bearer {os.environ['API_TOKEN']}"},
        }
    },
    allowed_tools=["mcp__remote-api__*"],
)
```

Pour le transport HTTP « streamable », utilisez `"type": "http"` à la place. Dans `.mcp.json` et les autres fichiers de configuration JSON, `"streamable-http"` est accepté comme alias de `"http"`. Le type `McpHttpServerConfig` des SDK ne déclare que `"http"` : utilisez donc `"http"` pour les serveurs passés dans le code.

### Serveurs MCP SDK

Définissez des outils personnalisés directement dans le code de votre application au lieu d'exécuter un processus serveur séparé. Voir le [guide des outils personnalisés](https://code.claude.com/docs/en/agent-sdk/custom-tools) pour les détails d'implémentation.

Un serveur MCP SDK enregistré par une [requête de contrôle `initialize`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkcontrolinitializeresponse) commence à se connecter dès que Claude Code traite la requête.

---

## Recherche d'outils MCP (tool search)

Quand vous avez de nombreux outils MCP configurés, leurs définitions peuvent consommer une part importante de votre fenêtre de contexte. La recherche d'outils résout ce problème en retenant les définitions d'outils hors du contexte et en ne chargeant que celles dont Claude a besoin à chaque tour.

La recherche d'outils est activée par défaut. Voir [Tool search](https://code.claude.com/docs/en/agent-sdk/tool-search) pour les options de configuration, les bonnes pratiques et l'utilisation de la recherche d'outils avec des outils SDK personnalisés.

---

## Authentification

La plupart des serveurs MCP exigent une authentification pour accéder aux services externes. Passez les identifiants via des variables d'environnement dans la configuration du serveur.

### Passer les identifiants via des variables d'environnement

Utilisez le champ `env` pour passer clés d'API, jetons et autres identifiants au serveur MCP.

#### Onglet « Dans le code »

TypeScript :

```typescript
const _ = {
  options: {
    mcpServers: {
      "api-server": {
        command: "npx",
        args: ["-y", "@your-org/api-mcp-server"],
        env: {
          API_KEY: process.env.API_KEY
        }
      }
    },
    allowedTools: ["mcp__api-server__*"]
  }
};
```

Python :

```python
options = ClaudeAgentOptions(
    mcp_servers={
        "api-server": {
            "command": "npx",
            "args": ["-y", "@your-org/api-mcp-server"],
            "env": {"API_KEY": os.environ["API_KEY"]},
        }
    },
    allowed_tools=["mcp__api-server__*"],
)
```

#### Onglet « .mcp.json »

```json
{
  "mcpServers": {
    "api-server": {
      "command": "npx",
      "args": ["-y", "@your-org/api-mcp-server"],
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

La syntaxe `${API_KEY}` développe les variables d'environnement à l'exécution.

### En-têtes HTTP pour les serveurs distants

Pour les serveurs HTTP et SSE, passez les en-têtes d'authentification directement dans la configuration du serveur.

#### Onglet « Dans le code »

TypeScript :

```typescript
const _ = {
  options: {
    mcpServers: {
      "secure-api": {
        type: "http",
        url: "https://api.example.com/mcp",
        headers: {
          Authorization: `Bearer ${process.env.API_TOKEN}`
        }
      }
    },
    allowedTools: ["mcp__secure-api__*"]
  }
};
```

Python :

```python
options = ClaudeAgentOptions(
    mcp_servers={
        "secure-api": {
            "type": "http",
            "url": "https://api.example.com/mcp",
            "headers": {"Authorization": f"Bearer {os.environ['API_TOKEN']}"},
        }
    },
    allowed_tools=["mcp__secure-api__*"],
)
```

#### Onglet « .mcp.json »

```json
{
  "mcpServers": {
    "secure-api": {
      "type": "http",
      "url": "https://api.example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${API_TOKEN}"
      }
    }
  }
}
```

La syntaxe `${API_TOKEN}` développe les variables d'environnement à l'exécution.

Pour un exemple complet et fonctionnel d'un serveur distant authentifié par en-têtes, voir [Lister les issues d'un dépôt](#lister-les-issues-dun-dépôt).

### Authentification OAuth2

La [spécification MCP prend en charge OAuth 2.1](https://modelcontextprotocol.io/specification/2025-03-26/basic/authorization) pour l'autorisation. Le SDK n'ouvre pas de navigateur et n'exécute pas de flux OAuth interactif. Quand un serveur configuré renvoie un défi d'autorisation et qu'aucun jeton stocké n'est disponible, l'exécution de l'agent continue sans les outils de ce serveur, et le serveur signale le statut `needs-auth`. Le tableau `mcp_servers` du [message system init](https://code.claude.com/docs/en/agent-sdk/typescript#sdksystemmessage) peut encore afficher `pending` pour ce serveur au moment de son émission. Pour vérifier si un serveur a besoin d'identifiants, interrogez périodiquement `mcpServerStatus()` dans le SDK TypeScript ou [`get_mcp_status()`](https://code.claude.com/docs/en/agent-sdk/python#methods) en Python.

Pour fournir des identifiants, réalisez le flux OAuth dans votre propre application et passez le jeton d'accès obtenu dans les `headers` du serveur.

#### TypeScript

```typescript
// Après avoir terminé le flux OAuth dans votre application.
// Implémentez getAccessTokenFromOAuthFlow pour votre fournisseur OAuth.
const accessToken = await getAccessTokenFromOAuthFlow();

const options = {
  mcpServers: {
    "oauth-api": {
      type: "http",
      url: "https://api.example.com/mcp",
      headers: {
        Authorization: `Bearer ${accessToken}`
      }
    }
  },
  allowedTools: ["mcp__oauth-api__*"]
};
```

#### Python

```python
# Après avoir terminé le flux OAuth dans votre application.
# Implémentez get_access_token_from_oauth_flow pour votre fournisseur OAuth.
access_token = await get_access_token_from_oauth_flow()

options = ClaudeAgentOptions(
    mcp_servers={
        "oauth-api": {
            "type": "http",
            "url": "https://api.example.com/mcp",
            "headers": {"Authorization": f"Bearer {access_token}"},
        }
    },
    allowed_tools=["mcp__oauth-api__*"],
)
```

---

## Exemples

### Lister les issues d'un dépôt

Cet exemple se connecte au [serveur MCP GitHub](https://github.com/github/github-mcp-server) distant pour lister les issues récentes. Il inclut des journaux de débogage pour vérifier la connexion MCP et les appels d'outils.

Avant l'exécution, créez un [jeton d'accès personnel GitHub](https://github.com/settings/personal-access-tokens) avec un accès en lecture aux dépôts que vous voulez interroger, et définissez-le comme variable d'environnement :

```bash
export GITHUB_TOKEN=YOUR_GITHUB_PAT
```

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "List the 3 most recent issues in anthropics/claude-code",
  options: {
    mcpServers: {
      github: {
        type: "http",
        url: "https://api.githubcopilot.com/mcp/",
        headers: {
          Authorization: `Bearer ${process.env.GITHUB_TOKEN}`
        }
      }
    },
    allowedTools: ["mcp__github__list_issues"]
  }
})) {
  // Vérifier que le serveur MCP s'est bien connecté
  if (message.type === "system" && message.subtype === "init") {
    console.log("MCP servers:", message.mcp_servers);
  }

  // Journaliser quand Claude appelle un outil MCP
  if (message.type === "assistant") {
    for (const block of message.message.content) {
      if (block.type === "tool_use" && block.name.startsWith("mcp__")) {
        console.log("MCP tool called:", block.name);
      }
    }
  }

  // Afficher le résultat final
  if (message.type === "result" && message.subtype === "success") {
    console.log(message.result);
  }
}
```

#### Python

```python
import asyncio
import os
from claude_agent_sdk import (
    query,
    ClaudeAgentOptions,
    ResultMessage,
    SystemMessage,
    AssistantMessage,
)


async def main():
    options = ClaudeAgentOptions(
        mcp_servers={
            "github": {
                "type": "http",
                "url": "https://api.githubcopilot.com/mcp/",
                "headers": {"Authorization": f"Bearer {os.environ['GITHUB_TOKEN']}"},
            }
        },
        allowed_tools=["mcp__github__list_issues"],
    )

    async for message in query(
        prompt="List the 3 most recent issues in anthropics/claude-code",
        options=options,
    ):
        # Vérifier que le serveur MCP s'est bien connecté
        if isinstance(message, SystemMessage) and message.subtype == "init":
            print("MCP servers:", message.data.get("mcp_servers"))

        # Journaliser quand Claude appelle un outil MCP
        if isinstance(message, AssistantMessage):
            for block in message.content:
                if hasattr(block, "name") and block.name.startswith("mcp__"):
                    print("MCP tool called:", block.name)

        # Afficher le résultat final
        if isinstance(message, ResultMessage) and message.subtype == "success":
            print(message.result)


asyncio.run(main())
```

Dans la ligne `MCP servers:`, un `status` `connected` pour `github` confirme que le jeton fonctionne. Si Claude Code dispose d'une [liste d'outils en cache](#moment-de-la-connexion-connection-timing) pour le serveur, le statut peut indiquer `pending` à la place, et le serveur se connecte à son premier appel d'outil. Si le statut est `failed` ou `needs-auth`, voir [Gestion des erreurs](#gestion-des-erreurs) avant de vous fier au résultat, car Claude peut se rabattre sur les outils intégrés quand le serveur est indisponible.

### Interroger une base de données

Cet exemple utilise [DBHub](https://github.com/bytebase/dbhub) pour interroger une base Postgres. L'agent découvre automatiquement le schéma de la base, écrit la requête SQL et renvoie les résultats.

**Sécurité** : l'outil `execute_sql` de DBHub exécute tout SQL émis par l'agent, y compris les écritures, sauf si vous le restreignez. Définir `readonly = true` dans le [fichier de configuration DBHub](https://dbhub.ai/config/toml) fait rejeter par DBHub les instructions `INSERT`, `UPDATE`, `DELETE` et DDL, si bien que l'exemple ne peut pas modifier vos données même si l'agent émet une écriture. DBHub résout `${DATABASE_URL}` depuis l'environnement du processus lors du chargement de la configuration, de sorte que la chaîne de connexion reste hors du fichier. Créez ce `dbhub.toml` à côté de votre script :

```toml
[[sources]]
id = "production"
dsn = "${DATABASE_URL}"

[[tools]]
name = "execute_sql"
source = "production"
readonly = true
```

Le script fait ensuite pointer DBHub vers le fichier de configuration au lieu de lui passer directement une chaîne de connexion. Avant l'exécution, définissez la variable d'environnement `DATABASE_URL` avec votre chaîne de connexion. Remplacez les valeurs d'exemple par celles de votre base :

```bash
export DATABASE_URL=postgresql://user:password@localhost:5432/mydb
```

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  // Requête en langage naturel - Claude écrit le SQL
  prompt: "How many users signed up last week? Break it down by day.",
  options: {
    mcpServers: {
      postgres: {
        command: "npx",
        // dbhub.toml définit readonly = true, donc execute_sql rejette les écritures
        args: ["-y", "@bytebase/dbhub", "--config", "dbhub.toml"]
      }
    },
    allowedTools: ["mcp__postgres__execute_sql"]
  }
})) {
  if (message.type === "result" && message.subtype === "success") {
    console.log(message.result);
  }
}
```

#### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


async def main():
    options = ClaudeAgentOptions(
        mcp_servers={
            "postgres": {
                "command": "npx",
                # dbhub.toml définit readonly = true, donc execute_sql rejette les écritures
                "args": [
                    "-y",
                    "@bytebase/dbhub",
                    "--config",
                    "dbhub.toml",
                ],
            }
        },
        allowed_tools=["mcp__postgres__execute_sql"],
    )

    # Requête en langage naturel - Claude écrit le SQL
    async for message in query(
        prompt="How many users signed up last week? Break it down by day.",
        options=options,
    ):
        if isinstance(message, ResultMessage) and message.subtype == "success":
            print(message.result)


asyncio.run(main())
```

---

## Gestion des erreurs

Les serveurs MCP peuvent échouer à se connecter pour diverses raisons : le processus serveur n'est peut-être pas installé, les identifiants peuvent être invalides, ou un serveur distant peut être injoignable.

Claude Code émet un message `system` de sous-type `init` au début de chaque requête. Ce message inclut le statut de connexion de chaque serveur MCP. Le champ `status` peut valoir `"pending"`, `"connected"`, `"failed"`, `"needs-auth"` ou `"disabled"`. Claude Code émet le message init après l'[attente de connexion du premier tour](#moment-de-la-connexion-connection-timing) pour les serveurs passés dans `options.mcpServers`, donc un tel serveur qui s'est connecté pendant l'attente affiche `"connected"`.

Dans le message init, ne traitez pas `"pending"` comme un échec en soi. Ce statut peut signifier :

- Le serveur ne s'est pas encore connecté. Voir [combien de temps Claude Code l'attend avant le premier tour](#moment-de-la-connexion-connection-timing).
- La liste d'outils du serveur a été [servie depuis le cache](#moment-de-la-connexion-connection-timing), avec une connexion établie à la première utilisation.
- L'échéance de connexion a expiré. Un tel serveur signale `"pending"` ou `"failed"` selon le moment.

Vérifiez `"failed"` ou `"needs-auth"` pour détecter les serveurs qui ne seront pas utilisables.

### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

try {
  for await (const message of query({
    prompt: "Process data",
    options: {
      mcpServers: {
        // Remplacez dataServer par la configuration de votre serveur
        "data-processor": dataServer
      }
    }
  })) {
    if (message.type === "system" && message.subtype === "init") {
      const unavailableServers = message.mcp_servers.filter(
        (s) => s.status === "failed" || s.status === "needs-auth"
      );

      if (unavailableServers.length > 0) {
        console.warn("Unavailable MCP servers:", unavailableServers);
      }
    }

    if (message.type === "result" && message.subtype === "error_during_execution") {
      console.error("Execution failed");
    }
  }
} catch (error) {
  // Un query() à usage unique lève une exception après avoir produit un résultat
  // d'erreur. Si l'échec était un résultat d'erreur, la branche du sous-type
  // d'erreur ci-dessus s'est déjà exécutée ; un échec de démarrage ou d'accès au
  // processus Claude Code ne produit aucun message de résultat. Les serveurs MCP
  // qui échouent à se connecter ne lèvent pas d'exception : utilisez la
  // vérification de statut ci-dessus, et notez que les serveurs encore
  // "pending" à l'init nécessitent une vérification de statut ultérieure.
  console.log(`Session ended with an error: ${error}`);
}
```

### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, SystemMessage, ResultMessage


async def main():
    # Remplacez data_server par la configuration de votre serveur
    options = ClaudeAgentOptions(mcp_servers={"data-processor": data_server})

    try:
        async for message in query(prompt="Process data", options=options):
            if isinstance(message, SystemMessage) and message.subtype == "init":
                unavailable_servers = [
                    s
                    for s in message.data.get("mcp_servers", [])
                    if s.get("status") in ("failed", "needs-auth")
                ]

                if unavailable_servers:
                    print(f"Unavailable MCP servers: {unavailable_servers}")

            if (
                isinstance(message, ResultMessage)
                and message.subtype == "error_during_execution"
            ):
                print("Execution failed")
    except Exception as error:
        # Un query() à usage unique lève une exception après avoir produit un
        # résultat d'erreur. Si l'échec était un résultat d'erreur, la branche du
        # sous-type d'erreur ci-dessus s'est déjà exécutée ; un échec de démarrage
        # ou d'accès au processus Claude Code ne produit aucun message de résultat.
        # Les serveurs MCP qui échouent à se connecter ne lèvent pas d'exception :
        # utilisez la vérification de statut ci-dessus, et notez que les serveurs
        # encore "pending" à l'init nécessitent une vérification ultérieure.
        print(f"Session ended with an error: {error}")


asyncio.run(main())
```

Le statut d'un serveur distant peut aussi changer après qu'il a signalé `"connected"`. Quand la connexion tombe en cours de session, Claude Code repasse le serveur à `"pending"` pendant la [reconnexion](https://code.claude.com/docs/en/mcp#automatic-reconnection). Un appel ultérieur à `mcpServerStatus()` en TypeScript, ou à [`ClaudeSDKClient.get_mcp_status()`](https://code.claude.com/docs/en/agent-sdk/python#methods) en Python, peut alors signaler `"pending"` pour un serveur que vous aviez vu connecté, sans aucun changement de configuration de votre côté.

Après cinq tentatives de reconnexion échouées, le serveur signale `"failed"`, ou `"needs-auth"` s'il doit être réautorisé. Un serveur HTTP ou SSE en `"failed"` peut encore revenir à `"connected"` sans appel de votre part, car Claude Code continue de [réessayer en arrière-plan](https://code.claude.com/docs/en/mcp#mid-session-drops-of-a-remote-server) après une erreur transitoire. Pour réessayer manuellement, appelez [`reconnectMcpServer()`](https://code.claude.com/docs/en/agent-sdk/typescript#methods) en TypeScript ou [`ClaudeSDKClient.reconnect_mcp_server()`](https://code.claude.com/docs/en/agent-sdk/python#methods) en Python.

### Récapitulatif des statuts de serveur MCP

| Statut | Signification |
| :- | :- |
| `pending` | Pas encore connecté, liste d'outils servie depuis le cache (connexion à la première utilisation), échéance expirée selon le timing, ou reconnexion en cours après une coupure. Pas un échec en soi |
| `connected` | Connecté et utilisable |
| `failed` | Échec de connexion (ou après cinq reconnexions échouées) ; un serveur HTTP/SSE peut encore se rétablir seul |
| `needs-auth` | Défi d'autorisation sans jeton stocké, ou réautorisation nécessaire |
| `disabled` | Serveur désactivé |

---

## Dépannage

### Le serveur affiche le statut « failed »

Consultez le message `init` pour voir quels serveurs n'ont pas réussi à se connecter.

#### TypeScript

```typescript
if (message.type === "system" && message.subtype === "init") {
  for (const server of message.mcp_servers) {
    if (server.status === "failed") {
      console.error(`Server ${server.name} failed to connect`);
    }
  }
}
```

#### Python

```python
if isinstance(message, SystemMessage) and message.subtype == "init":
    for server in message.data.get("mcp_servers", []):
        if server.get("status") == "failed":
            print(f"Server {server['name']} failed to connect")
```

Un statut `"pending"` ne signifie pas que le serveur a échoué. Voir [Gestion des erreurs](#gestion-des-erreurs) pour les cas qu'il recouvre à l'init. Pour obtenir des statuts à jour plus tard dans la session, appelez la méthode `mcpServerStatus()` de la requête dans le SDK TypeScript, ou [`ClaudeSDKClient.get_mcp_status()`](https://code.claude.com/docs/en/agent-sdk/python#methods) en Python.

Causes fréquentes :

- **Variables d'environnement manquantes** : assurez-vous que les jetons et identifiants requis sont définis. Pour les serveurs stdio, vérifiez que le champ `env` correspond à ce que le serveur attend.
- **Serveur non installé** : pour les commandes `npx`, vérifiez que le paquet existe et que Node.js est dans votre PATH.
- **Chaîne de connexion invalide** : pour les serveurs de base de données, vérifiez le format de la chaîne de connexion et que la base est accessible.
- **Problèmes réseau** : pour les serveurs HTTP/SSE distants, vérifiez que l'URL est joignable et que les pare-feux autorisent la connexion.

### Les outils ne sont pas appelés

Si Claude voit les outils mais ne les utilise pas, vérifiez que vous avez accordé la permission avec `allowedTools`.

#### TypeScript

```typescript
const _ = {
  options: {
    mcpServers: {
      // vos serveurs
    },
    allowedTools: ["mcp__servername__*"] // Approuve automatiquement les appels de ce serveur
  }
};
```

#### Python

```python
options = ClaudeAgentOptions(
    mcp_servers={
        # vos serveurs
    },
    allowed_tools=["mcp__servername__*"],  # Approuve automatiquement les appels de ce serveur
)
```

### Un outil manque dans un serveur MCP SDK

Dans le SDK TypeScript, quand le schéma d'entrée d'un outil ne peut pas être converti en JSON Schema, le serveur créé avec [`createSdkMcpServer()`](https://code.claude.com/docs/en/agent-sdk/typescript#createsdkmcpserver) omet cet outil lorsqu'il liste ses outils. Le SDK émet alors un avertissement. Sous Node.js, l'avertissement est un « process warning » de code `CLAUDE_SDK_MCP_TOOL_SCHEMA_UNCONVERTIBLE`, et il commence par ce texte :

```text
Tool "<name>" on SDK MCP server "<server>" was left out of the server's tool list, because its input schema cannot be converted to JSON Schema
```

La suite de l'avertissement donne le message de l'erreur de conversion lorsqu'il y en a un, puis indique quoi vérifier et modifier.

Avant le SDK Agent TypeScript v0.3.286, un seul schéma non convertible faisait échouer toute la liste d'outils du serveur sans cet avertissement, si bien qu'aucun outil de ce serveur n'atteignait Claude.

### Délais de connexion (timeouts)

Les connexions aux serveurs MCP expirent après 30 secondes par défaut. Pour changer la durée qu'un appel d'outil en cours peut prendre, définissez [`MCP_TOOL_TIMEOUT`](https://code.claude.com/docs/en/env-vars). Si votre serveur met plus longtemps à démarrer, la connexion échoue. Augmentez la limite de connexion avec la variable d'environnement [`MCP_TIMEOUT`](https://code.claude.com/docs/en/env-vars), en millisecondes. Pour les serveurs qui ont besoin de plus de temps de démarrage, envisagez aussi :

- D'utiliser un serveur plus léger si disponible
- De préchauffer le serveur avant de démarrer votre agent
- De consulter les journaux du serveur pour trouver la cause d'une initialisation lente

En TypeScript, vous pouvez définir la limite d'appel d'outil pour un seul [serveur MCP SDK](#serveurs-mcp-sdk) en passant [`timeout` à `createSdkMcpServer()`](https://code.claude.com/docs/en/agent-sdk/typescript#createsdkmcpserver).

### La sortie d'un outil dépasse le nombre maximal de tokens autorisé

Le SDK applique les mêmes limites de sortie MCP que Claude Code. Quand un résultat d'outil réussi, sans contenu image, dépasse 25 000 tokens, Claude Code enregistre la sortie dans un fichier et remplace le résultat de l'outil par un message d'erreur qui indique le chemin du fichier, afin que l'agent puisse relire la sortie par portions.

Pour changer la limite de tokens, définissez la variable d'environnement [`MAX_MCP_OUTPUT_TOKENS`](https://code.claude.com/docs/en/env-vars). Sauf si l'outil déclare `anthropic/maxResultSizeChars`, un résultat texte réussi de plus de 50 000 caractères est enregistré dans un fichier indépendamment de la limite de tokens. Voir [MCP output limits and warnings](https://code.claude.com/docs/en/mcp#mcp-output-limits-and-warnings) pour le comportement complet, y compris la façon dont un serveur déclare cette annotation.

---

## Ressources associées

- **[Guide des outils personnalisés](https://code.claude.com/docs/en/agent-sdk/custom-tools)** : construire votre propre serveur MCP qui tourne en processus avec votre application SDK
- **[Permissions](https://code.claude.com/docs/en/agent-sdk/permissions)** : contrôler quels outils MCP votre agent peut utiliser avec `allowedTools` et `disallowedTools`
- **[Référence du SDK TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript)** : référence complète de l'API, y compris les options de configuration MCP
- **[Référence du SDK Python](https://code.claude.com/docs/en/agent-sdk/python)** : référence complète de l'API, y compris les options de configuration MCP
- **[Annuaire des serveurs MCP](https://github.com/modelcontextprotocol/servers)** : parcourir les serveurs MCP disponibles pour bases de données, API, etc.
