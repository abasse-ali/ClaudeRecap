# Diffuser les réponses en temps réel

> Source : https://code.claude.com/docs/en/agent-sdk/streaming-output · [← Index](../31-index-documentation-officielle.md)

*Recevoir les réponses de l'Agent SDK en temps réel, au fil de la génération du texte et des appels d'outils.*

Par défaut, l'Agent SDK produit (yield) un `AssistantMessage` complet pour chaque bloc de contenu non vide, comme un bloc de texte ou un appel d'outil, une fois que Claude a fini de générer ce bloc. Pour recevoir des mises à jour incrémentales pendant que le texte et les appels d'outils sont générés, activez le streaming des messages partiels.

> **Astuce** : cette page traite du streaming de **sortie** (recevoir les tokens en temps réel). Pour les modes d'**entrée** (la façon dont vous envoyez les messages), voir [Envoyer des messages aux agents](https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode). Vous pouvez aussi [diffuser les réponses de l'Agent SDK via la CLI](https://code.claude.com/docs/en/headless).

## Activer le streaming de sortie

Pour activer le streaming, réglez `include_partial_messages` (Python) ou `includePartialMessages` (TypeScript) à `true` dans vos options. Le SDK produit alors des messages `StreamEvent` contenant les événements bruts de l'API au fur et à mesure qu'ils arrivent, en plus des habituels `AssistantMessage` et `ResultMessage`.

Votre code doit ensuite :

1. Vérifier le type de chaque message pour distinguer `StreamEvent` des autres types de messages.
2. Pour un `StreamEvent`, extraire le champ `event` et vérifier son `type`.
3. Repérer les événements `content_block_delta` dont `delta.type` vaut `text_delta` : ce sont eux qui contiennent les fragments de texte proprement dits.

L'exemple ci-dessous active le streaming et affiche les fragments de texte dès leur arrivée. Notez les vérifications de type imbriquées : d'abord `StreamEvent`, puis `content_block_delta`, puis `text_delta`.

### Python

```python
from claude_agent_sdk import query, ClaudeAgentOptions
from claude_agent_sdk.types import StreamEvent
import asyncio


async def stream_response():
    options = ClaudeAgentOptions(
        include_partial_messages=True,
        allowed_tools=["Bash", "Read"],
    )

    async for message in query(prompt="List the files in my project", options=options):
        if isinstance(message, StreamEvent):
            event = message.event
            if event.get("type") == "content_block_delta":
                delta = event.get("delta", {})
                if delta.get("type") == "text_delta":
                    print(delta.get("text", ""), end="", flush=True)


asyncio.run(stream_response())
```

### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "List the files in my project",
  options: {
    includePartialMessages: true,
    allowedTools: ["Bash", "Read"]
  }
})) {
  if (message.type === "stream_event") {
    const event = message.event;
    if (event.type === "content_block_delta") {
      if (event.delta.type === "text_delta") {
        process.stdout.write(event.delta.text);
      }
    }
  }
}
```

## Référence de StreamEvent

Quand les messages partiels sont activés, vous recevez les événements de streaming bruts de l'API Claude, enveloppés dans un objet. Ce type porte un nom différent selon le SDK :

* **Python** : [`StreamEvent`](https://code.claude.com/docs/en/agent-sdk/python#streamevent) (à importer depuis `claude_agent_sdk.types`)
* **TypeScript** : [`SDKPartialAssistantMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkpartialassistantmessage) avec `type: 'stream_event'`

Les deux contiennent des événements bruts de l'API Claude, **pas du texte cumulé**. C'est à vous d'extraire et d'accumuler les deltas de texte.

Le champ `parent_tool_use_id` vaut toujours `None` en Python et `null` en TypeScript. Les événements de stream ne sont émis que pour la session principale ; les deltas au niveau des tokens provenant des sous-agents ne sont pas relayés. Pour attribuer une sortie à un sous-agent, utilisez les messages complets, qui portent `parent_tool_use_id`. Voir [Détecter l'invocation d'un sous-agent](https://code.claude.com/docs/en/agent-sdk/subagents#detect-subagent-invocation).

Claude Code renseigne `user_message_uuid` sur le premier événement de stream non-ping du tour, puis à nouveau lorsque le message auquel le tour répond change, dans les conditions décrites à [`user_message_uuid`](https://code.claude.com/docs/en/agent-sdk/typescript#user_message_uuid). Le `StreamEvent` Python n'expose pas ce champ.

Le champ `event` contient l'événement de streaming brut de l'[API Claude](https://platform.claude.com/docs/en/build-with-claude/streaming#event-types). Types d'événements courants :

| Type d'événement | Description |
| :- | :- |
| `message_start` | Début d'un nouveau message |
| `content_block_start` | Début d'un nouveau bloc de contenu (texte ou utilisation d'outil) |
| `content_block_delta` | Mise à jour incrémentale du contenu |
| `content_block_stop` | Fin d'un bloc de contenu |
| `message_delta` | Mises à jour au niveau du message (raison d'arrêt, utilisation) |
| `message_stop` | Fin du message |

## Déroulement des messages

Claude Code émet un `AssistantMessage` à mesure que chaque bloc de contenu non vide se termine : une réponse comportant un bloc de texte et un appel d'outil produit donc **deux** objets `AssistantMessage`. Chacun ne porte que son propre bloc de contenu, et tous deux partagent le même ID de message, que vous lisez via `message.message.id` en TypeScript et `message.message_id` en Python. Avec les messages partiels activés, chaque `AssistantMessage` arrive **avant** l'événement `content_block_stop` de son bloc, et vous recevez les messages dans cet ordre :

```text
StreamEvent (message_start)
StreamEvent (content_block_start) - text block
StreamEvent (content_block_delta) - text chunks...
AssistantMessage - complete text block
StreamEvent (content_block_stop)
StreamEvent (content_block_start) - tool_use block
StreamEvent (content_block_delta) - tool input chunks...
AssistantMessage - complete tool_use block
StreamEvent (content_block_stop)
StreamEvent (message_delta)
StreamEvent (message_stop)
... tool executes ...
... more streaming events for next turn ...
ResultMessage - final result
```

(Lecture : début du message ; début du bloc texte ; fragments de texte ; `AssistantMessage` du bloc texte complet ; fin du bloc ; début du bloc `tool_use` ; fragments de l'entrée de l'outil ; `AssistantMessage` du bloc `tool_use` complet ; fin du bloc ; delta du message ; fin du message ; exécution de l'outil ; événements de streaming du tour suivant ; `ResultMessage` avec le résultat final.)

Sans messages partiels, vous recevez tous les types de messages **sauf** `StreamEvent`. Types courants :

* `SystemMessage` (initialisation de la session) ;
* `AssistantMessage` (blocs de contenu complets) ;
* `ResultMessage` (résultat final) ;
* un message de frontière de compaction indiquant quand l'historique de la conversation a été compacté (`SDKCompactBoundaryMessage` en TypeScript ; `SystemMessage` avec le sous-type `"compact_boundary"` en Python).

### Gérer un stream interrompu

Si un stream est coupé au milieu d'un message, par exemple quand vous interrompez le tour ou que la connexion tombe, vous recevez quand même le `message_stop` de ce message avant la fin du tour. Un bloc de texte ou de réflexion (thinking) coupé reçoit aussi son `content_block_stop`. Un appel d'outil coupé, lui, **ne le reçoit pas** : si `message_stop` arrive alors que le bloc d'un appel d'outil est encore ouvert, considérez l'entrée de cet appel comme incomplète.

Avant Claude Code v2.1.290, un stream coupé pouvait terminer le tour sans `message_stop`, si bien qu'une réponse affichée à partir des événements de stream pouvait rester indiquée comme « en cours ». Le SDK Agent TypeScript embarque Claude Code v2.1.290 ou ultérieur à partir de sa v0.3.290, et le SDK Agent Python à partir de sa v0.2.164. Si une réponse reste affichée comme en cours après la fin du tour, mettez à jour le SDK.

## Diffuser les appels d'outils

Les appels d'outils sont eux aussi diffusés de façon incrémentale. Vous pouvez suivre le moment où les outils démarrent, recevoir leur entrée au fil de sa génération et voir quand ils se terminent. L'exemple ci-dessous suit l'outil en cours d'appel et accumule l'entrée JSON au fur et à mesure qu'elle arrive. Il utilise trois types d'événements :

* `content_block_start` : l'outil commence ;
* `content_block_delta` avec `input_json_delta` : des fragments de l'entrée arrivent ;
* `content_block_stop` : l'appel d'outil est terminé.

### Python

```python
from claude_agent_sdk import query, ClaudeAgentOptions
from claude_agent_sdk.types import StreamEvent
import asyncio


async def stream_tool_calls():
    options = ClaudeAgentOptions(
        include_partial_messages=True,
        allowed_tools=["Read", "Bash"],
    )

    # Suivre l'outil en cours et accumuler son entrée JSON
    current_tool = None
    tool_input = ""

    async for message in query(prompt="Read the README.md file", options=options):
        if isinstance(message, StreamEvent):
            event = message.event
            event_type = event.get("type")

            if event_type == "content_block_start":
                # Un nouvel appel d'outil commence
                content_block = event.get("content_block", {})
                if content_block.get("type") == "tool_use":
                    current_tool = content_block.get("name")
                    tool_input = ""
                    print(f"Starting tool: {current_tool}")

            elif event_type == "content_block_delta":
                delta = event.get("delta", {})
                if delta.get("type") == "input_json_delta":
                    # Accumuler l'entrée JSON au fil du stream
                    chunk = delta.get("partial_json", "")
                    tool_input += chunk
                    print(f"  Input chunk: {chunk}")

            elif event_type == "content_block_stop":
                # Appel d'outil terminé : afficher l'entrée finale
                if current_tool:
                    print(f"Tool {current_tool} called with: {tool_input}")
                    current_tool = None


asyncio.run(stream_tool_calls())
```

### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

// Suivre l'outil en cours et accumuler son entrée JSON
let currentTool: string | null = null;
let toolInput = "";

for await (const message of query({
  prompt: "Read the README.md file",
  options: {
    includePartialMessages: true,
    allowedTools: ["Read", "Bash"]
  }
})) {
  if (message.type === "stream_event") {
    const event = message.event;

    if (event.type === "content_block_start") {
      // Un nouvel appel d'outil commence
      if (event.content_block.type === "tool_use") {
        currentTool = event.content_block.name;
        toolInput = "";
        console.log(`Starting tool: ${currentTool}`);
      }
    } else if (event.type === "content_block_delta") {
      if (event.delta.type === "input_json_delta") {
        // Accumuler l'entrée JSON au fil du stream
        const chunk = event.delta.partial_json;
        toolInput += chunk;
        console.log(`  Input chunk: ${chunk}`);
      }
    } else if (event.type === "content_block_stop") {
      // Appel d'outil terminé : afficher l'entrée finale
      if (currentTool) {
        console.log(`Tool ${currentTool} called with: ${toolInput}`);
        currentTool = null;
      }
    }
  }
}
```

## Construire une interface en streaming

Cet exemple combine le streaming du texte et celui des outils dans une interface cohérente. Il mémorise si l'agent est en train d'exécuter un outil (drapeau `in_tool`) afin d'afficher des indicateurs d'état comme `[Using Read...]` pendant l'exécution des outils. Le texte est diffusé normalement hors appel d'outil, et la fin d'un outil déclenche un message « done ». Ce modèle est utile pour les interfaces de chat qui doivent montrer la progression pendant des tâches d'agent en plusieurs étapes.

### Python

```python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage
from claude_agent_sdk.types import StreamEvent
import asyncio
import sys


async def streaming_ui():
    options = ClaudeAgentOptions(
        include_partial_messages=True,
        allowed_tools=["Read", "Bash", "Grep"],
    )

    # Mémoriser si l'on est actuellement dans un appel d'outil
    in_tool = False

    async for message in query(
        prompt="Find all TODO comments in the codebase", options=options
    ):
        if isinstance(message, StreamEvent):
            event = message.event
            event_type = event.get("type")

            if event_type == "content_block_start":
                content_block = event.get("content_block", {})
                if content_block.get("type") == "tool_use":
                    # Un appel d'outil commence : afficher l'indicateur d'état
                    tool_name = content_block.get("name")
                    print(f"\n[Using {tool_name}...]", end="", flush=True)
                    in_tool = True

            elif event_type == "content_block_delta":
                delta = event.get("delta", {})
                # Ne diffuser le texte que hors exécution d'outil
                if delta.get("type") == "text_delta" and not in_tool:
                    sys.stdout.write(delta.get("text", ""))
                    sys.stdout.flush()

            elif event_type == "content_block_stop":
                if in_tool:
                    # Appel d'outil terminé
                    print(" done", flush=True)
                    in_tool = False

        elif isinstance(message, ResultMessage):
            # L'agent a terminé tout son travail
            print(f"\n\n--- Complete ---")


asyncio.run(streaming_ui())
```

### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

// Mémoriser si l'on est actuellement dans un appel d'outil
let inTool = false;

for await (const message of query({
  prompt: "Find all TODO comments in the codebase",
  options: {
    includePartialMessages: true,
    allowedTools: ["Read", "Bash", "Grep"]
  }
})) {
  if (message.type === "stream_event") {
    const event = message.event;

    if (event.type === "content_block_start") {
      if (event.content_block.type === "tool_use") {
        // Un appel d'outil commence : afficher l'indicateur d'état
        process.stdout.write(`\n[Using ${event.content_block.name}...]`);
        inTool = true;
      }
    } else if (event.type === "content_block_delta") {
      // Ne diffuser le texte que hors exécution d'outil
      if (event.delta.type === "text_delta" && !inTool) {
        process.stdout.write(event.delta.text);
      }
    } else if (event.type === "content_block_stop") {
      if (inTool) {
        // Appel d'outil terminé
        console.log(" done");
        inTool = false;
      }
    }
  } else if (message.type === "result") {
    // L'agent a terminé tout son travail
    console.log("\n\n--- Complete ---");
  }
}
```

## Limitations connues

* **Sortie structurée** : avec les messages partiels activés, le JSON est diffusé sous forme de fragments `input_json_delta` **non validés** d'un appel d'outil, et seul le résultat validé parvient au `ResultMessage.structured_output` final. Voir [sorties structurées](https://code.claude.com/docs/en/agent-sdk/structured-outputs) pour les détails.

## Étapes suivantes

Maintenant que vous savez diffuser le texte et les appels d'outils en temps réel, explorez ces sujets connexes :

* [Requêtes interactives ou ponctuelles](https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode) : choisir entre les modes d'entrée selon votre cas d'usage ;
* [Sorties structurées](https://code.claude.com/docs/en/agent-sdk/structured-outputs) : obtenir de l'agent des réponses JSON typées ;
* [Permissions](https://code.claude.com/docs/en/agent-sdk/permissions) : contrôler les outils que l'agent peut utiliser.
