# Entrée en streaming

> Source : https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode · [← Index](../31-index-documentation-officielle.md)

*Comprendre les deux modes d'entrée du Claude Agent SDK et quand utiliser chacun.*

## Vue d'ensemble

Le Claude Agent SDK prend en charge deux modes d'entrée distincts pour interagir avec les agents :

* **Mode d'entrée en streaming** (*Streaming Input Mode*) : une session persistante et interactive ;
* **Entrée par message unique** (*Single Message Input*) : des requêtes ponctuelles qui s'appuient sur l'état de session et la reprise (resume).

## Mode d'entrée en streaming (recommandé)

Le mode d'entrée en streaming est la façon **privilégiée** d'utiliser le Claude Agent SDK. Il donne un accès complet aux capacités de l'agent et permet des expériences riches et interactives.

Il permet à l'agent de fonctionner comme un processus de longue durée qui reçoit les entrées de l'utilisateur, gère les interruptions, fait remonter les demandes de permission et gère la session.

### Avantages

En mode d'entrée en streaming, vous travaillez dans une session persistante offrant ces capacités :

* **Envoi d'images** : joindre des images directement aux messages pour une analyse et une compréhension visuelles ;
* **Messages en file d'attente** : envoyer plusieurs messages traités séquentiellement, avec possibilité d'interrompre ;
* **Intégration des outils** : accès complet à tous les outils et aux serveurs MCP personnalisés pendant la session ;
* **Retour en temps réel** : voir les réponses au fur et à mesure de leur génération, pas seulement les résultats finaux ;
* **Persistance du contexte** : conserver naturellement le contexte de la conversation sur plusieurs tours.

### Exemple d'implémentation

Ces exemples lisent une image nommée `diagram.png` dans le répertoire de travail. Créez-en une à cet endroit au préalable, ou changez le nom du fichier pour pointer vers votre propre image.

#### TypeScript

```typescript
import { query, type SDKUserMessage } from "@anthropic-ai/claude-agent-sdk";
import { readFile } from "fs/promises";

async function* generateMessages(): AsyncGenerator<SDKUserMessage> {
  // Premier message
  yield {
    type: "user",
    message: {
      role: "user",
      content: "Analyze this codebase for security issues"
    },
    parent_tool_use_id: null
  };

  // Attendre des conditions ou une entrée utilisateur
  await new Promise((resolve) => setTimeout(resolve, 2000));

  // Relance avec une image
  yield {
    type: "user",
    message: {
      role: "user",
      content: [
        {
          type: "text",
          text: "Review this architecture diagram"
        },
        {
          type: "image",
          source: {
            type: "base64",
            media_type: "image/png",
            data: await readFile("diagram.png", "base64")
          }
        }
      ]
    },
    parent_tool_use_id: null
  };
}

// Traiter les réponses diffusées
for await (const message of query({
  prompt: generateMessages(),
  options: {
    maxTurns: 10,
    allowedTools: ["Read", "Grep"]
  }
})) {
  if (message.type === "result" && message.subtype === "success") {
    console.log(message.result);
  }
}
```

#### Python

```python
from claude_agent_sdk import (
    ClaudeSDKClient,
    ClaudeAgentOptions,
    AssistantMessage,
    TextBlock,
)
import asyncio
import base64


async def streaming_analysis():
    async def message_generator():
        # Premier message
        yield {
            "type": "user",
            "message": {
                "role": "user",
                "content": "Analyze this codebase for security issues",
            },
        }

        # Attendre des conditions
        await asyncio.sleep(2)

        # Relance avec une image
        with open("diagram.png", "rb") as f:
            image_data = base64.b64encode(f.read()).decode()

        yield {
            "type": "user",
            "message": {
                "role": "user",
                "content": [
                    {"type": "text", "text": "Review this architecture diagram"},
                    {
                        "type": "image",
                        "source": {
                            "type": "base64",
                            "media_type": "image/png",
                            "data": image_data,
                        },
                    },
                ],
            },
        }

    # Utiliser ClaudeSDKClient pour l'entrée en streaming
    options = ClaudeAgentOptions(max_turns=10, allowed_tools=["Read", "Grep"])

    async with ClaudeSDKClient(options) as client:
        # Envoyer l'entrée en streaming
        await client.query(message_generator())

        # Traiter les réponses
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(block.text)


asyncio.run(streaming_analysis())
```

À l'exécution, la version TypeScript affiche chaque réponse dès qu'elle est terminée. Dans la version Python, la boucle `receive_response()` s'arrête au premier message de résultat : elle affiche donc l'analyse de sécurité seulement. Pour lire les deux réponses, utilisez une paire `query()` / `receive_response()` par message, comme dans l'[exemple de poursuite d'une conversation de la référence Python](https://code.claude.com/docs/en/agent-sdk/python#example-continuing-a-conversation).

Si la `source` d'un bloc image est absente ou n'est pas un objet, le SDK ne signale **pas** d'erreur. Claude Code envoie à Claude une note textuelle à la place de l'image, par exemple `[Image could not be processed: image block has no source object]`, et la session continue.

> **Note** : dans le SDK TypeScript, si votre générateur de messages lève une exception (par exemple quand un fichier qu'il lit est absent), le stream se termine par une erreur indiquant `Claude Code process aborted by user` au lieu de l'erreur d'origine : quand vous voyez ce message, vérifiez d'abord le code de votre générateur. L'erreur peut aussi être précédée d'une longue ligne minifiée de source du SDK embarqué ; lisez donc la sortie jusqu'au bout pour trouver le texte de l'erreur.
>
> Dans le SDK Python, une exception du générateur est journalisée au niveau debug et la session se bloque sans lever d'erreur : si une session en streaming reste figée sans aucune sortie, activez la journalisation de debug et vérifiez votre générateur.

## Entrée par message unique

L'entrée par message unique est plus simple, mais plus limitée.

### Quand utiliser l'entrée par message unique

Utilisez l'entrée par message unique quand :

* vous avez besoin d'une réponse ponctuelle (one-shot) ;
* vous n'avez pas besoin de pièces jointes image ni de méthodes de contrôle en cours de session ;
* vous devez fonctionner dans un environnement sans état (stateless), comme une fonction lambda.

### Limitations

> **Avertissement** : le mode d'entrée par message unique ne prend **pas** en charge :
>
> * les pièces jointes image directement dans les messages ;
> * la mise en file d'attente dynamique de messages ;
> * l'interruption en temps réel ;
> * les conversations multi-tours naturelles.

Si une requête se termine par un résultat d'erreur, comme `error_max_turns`, un appel `query()` en message unique lève une erreur contenant le texte de l'échec **après** avoir produit le message de résultat final : entourez donc la boucle d'un bloc try si votre code doit continuer. Voir [Gérer le résultat](https://code.claude.com/docs/en/agent-sdk/agent-loop#handle-the-result) pour les sous-types de résultat.

### Exemple d'implémentation

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

// Requête ponctuelle simple
// query() lève une erreur après un résultat d'erreur, comme error_max_turns
try {
  for await (const message of query({
    prompt: "Explain the authentication flow",
    options: {
      maxTurns: 5,
      allowedTools: ["Read", "Grep"]
    }
  })) {
    if (message.type === "result" && message.subtype === "success") {
      console.log(message.result);
    }
  }
} catch (error) {
  console.error(`Query failed: ${error}`);
}

// Poursuivre la conversation grâce à la gestion de session
try {
  for await (const message of query({
    prompt: "Now explain the authorization process",
    options: {
      continue: true,
      maxTurns: 5
    }
  })) {
    if (message.type === "result" && message.subtype === "success") {
      console.log(message.result);
    }
  }
} catch (error) {
  console.error(`Query failed: ${error}`);
}
```

#### Python

```python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage
import asyncio


async def single_message_example():
    # Requête ponctuelle simple avec la fonction query()
    # query() lève ResultError après un résultat d'erreur, comme error_max_turns
    try:
        async for message in query(
            prompt="Explain the authentication flow",
            options=ClaudeAgentOptions(max_turns=5, allowed_tools=["Read", "Grep"]),
        ):
            if isinstance(message, ResultMessage) and message.subtype == "success":
                print(message.result)
    except Exception as e:
        print(f"Query failed: {e}")

    # Poursuivre la conversation grâce à la gestion de session
    try:
        async for message in query(
            prompt="Now explain the authorization process",
            options=ClaudeAgentOptions(continue_conversation=True, max_turns=5),
        ):
            if isinstance(message, ResultMessage) and message.subtype == "success":
                print(message.result)
    except Exception as e:
        print(f"Query failed: {e}")


asyncio.run(single_message_example())
```

À l'exécution, chaque requête affiche le texte de son résultat final : d'abord l'explication du flux d'authentification, puis celle du processus d'autorisation.
