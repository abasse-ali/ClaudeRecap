# Suivre le coût et l'utilisation

> Source : https://code.claude.com/docs/en/agent-sdk/cost-tracking · [← Index](../31-index-documentation-officielle.md)

*Apprendre à suivre l'utilisation des tokens, estimer les coûts et configurer le cache de prompt avec le Claude Agent SDK.*

Le Claude Agent SDK fournit des informations détaillées sur l'utilisation des tokens pour chaque interaction avec Claude. Ce guide explique comment suivre correctement l'utilisation et interpréter les coûts rapportés, en particulier avec des utilisations d'outils en parallèle et des conversations en plusieurs étapes.

Pour la documentation complète de l'API, voir la [référence du SDK TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript) et la [référence du SDK Python](https://code.claude.com/docs/en/agent-sdk/python).

### Des estimations, pas de la facturation

> **Avertissement** : les champs `total_cost_usd` et `costUSD` sont des **estimations côté client**, pas des données de facturation faisant foi. Le SDK les calcule localement à partir d'une table de prix intégrée au moment du build, sauf si une table [`modelPricing`](https://code.claude.com/docs/en/settings-reference#modelpricing) est en vigueur. Ils peuvent s'écarter de ce qui vous est réellement facturé lorsque :
>
> * les tarifs changent
> * la version installée du SDK ne reconnaît pas un modèle
> * des règles de facturation s'appliquent que le client ne peut pas modéliser
>
> Une règle de facturation que le SDK modélise bien est la [tarification liée à la résidence des données](https://platform.claude.com/docs/en/about-claude/pricing#data-residency-pricing). Quand l'`usage` d'une réponse indique `inference_geo: "us"`, le SDK multiplie par 1,1 le prix catalogue des tokens de cette réponse. Les frais par requête, comme la recherche web, ne sont pas multipliés. Nécessite le SDK Agent TypeScript v0.3.239+ ou le SDK Agent Python v0.2.144+.
>
> Utilisez ces champs pour vous éclairer en développement et pour une budgétisation approximative. Pour une facturation faisant foi, utilisez l'[API Usage and Cost](https://platform.claude.com/docs/en/build-with-claude/usage-cost-api) ou la page Usage de la [Claude Console](https://platform.claude.com/usage). Ne facturez pas d'utilisateurs finaux et ne déclenchez pas de décisions financières à partir de ces champs.

## Comprendre l'utilisation des tokens

Les SDK TypeScript et Python exposent les mêmes données d'utilisation sous des noms de champs différents :

* **TypeScript** fournit le détail des tokens par étape sur chaque message assistant (`message.message.id`, `message.message.usage`), le coût par modèle via `modelUsage` sur le message de résultat, et un total cumulé sur le message de résultat.
* **Python** fournit le détail des tokens par étape sur chaque message assistant via `message.usage` et `message.message_id`, le coût par modèle via `model_usage` sur le message de résultat, et le total cumulé sur le message de résultat via `total_cost_usd`.

Les deux SDK utilisent le même modèle de coût sous-jacent et exposent la même granularité. La différence porte sur le nommage des champs et l'endroit où l'utilisation par étape est imbriquée.

Le suivi des coûts suppose de comprendre comment le SDK délimite les données d'utilisation :

* **Appel `query()`** : une invocation de la fonction `query()` du SDK. Un seul appel peut comporter plusieurs étapes : Claude répond, utilise des outils, obtient des résultats et répond à nouveau. Chaque appel produit un message [`result`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkresultmessage) à la fin, sauf en [mode d'entrée en streaming](https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode), où un appel `query()` porte plusieurs tours utilisateur et où chaque tour émet son propre message `result`.
* **Étape** : un cycle requête/réponse unique au sein d'un appel `query()`. Chaque étape produit des messages assistant avec leur utilisation de tokens.
* **Session** : une série d'appels `query()` reliés par un ID de session via l'option `resume`. Les résultats d'un appel repris rapportent la dépense de toute la session, pas seulement celle de cet appel. Voir [Cumuler les coûts sur plusieurs appels](#cumuler-les-coûts-sur-plusieurs-appels) pour la façon dont les totaux se reportent.

Le diagramme de la source montre le flux de messages d'un seul appel `query()` : l'étape 1 comporte quatre messages assistant partageant le même ID et la même utilisation (à compter une seule fois), l'étape 2 un message assistant avec un nouvel ID, et le message de résultat final affiche le `total_cost_usd` estimé.

### Étapes

1. **Chaque étape produit des messages assistant.** Quand Claude répond, il envoie un ou plusieurs messages assistant. En TypeScript, chaque message assistant contient un `BetaMessage` imbriqué (accessible via `message.message`) avec un `id` et un objet [`usage`](https://platform.claude.com/docs/en/api/messages) contenant les comptes de tokens (`input_tokens`, `output_tokens`). En Python, la dataclass `AssistantMessage` expose les mêmes données directement via `message.usage` et `message.message_id`. Quand Claude utilise plusieurs outils dans un même tour, tous les messages de ce tour partagent le même ID : dédupliquez par ID pour éviter de compter deux fois.
2. **Le message de résultat fournit l'estimation cumulée.** Quand l'appel `query()` se termine, le SDK émet un message de résultat avec `total_cost_usd` et l'`usage` cumulé, typé [`SDKResultMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkresultmessage) en TypeScript et [`ResultMessage`](https://code.claude.com/docs/en/agent-sdk/python#resultmessage) en Python. Si vous n'avez besoin que du total estimé, vous pouvez ignorer l'utilisation par étape et lire cette seule valeur.
   * Si vous faites plusieurs appels `query()` indépendants, chaque résultat ne reflète que le coût de cet appel. Un appel qui reprend une session compte aussi la dépense antérieure de la session.
   * En mode d'entrée en streaming, chaque tour émet son propre message de résultat. Voir [Suivre les coûts en mode d'entrée en streaming](#suivre-les-coûts-en-mode-dentrée-en-streaming) pour lire les totaux d'appel dans ce mode.

## Suivre les coûts en mode d'entrée en streaming

En [mode d'entrée en streaming](https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode), un appel `query()` porte plusieurs tours utilisateur et chaque tour émet son propre message de résultat. Les champs du résultat n'ont pas la même portée :

* **`usage`** : couvre uniquement ce tour, et dans ce tour uniquement la boucle de l'agent principal, pas les sous-agents qu'il a exécutés.
* **`total_cost_usd` et `modelUsage` (ou `model_usage` en Python)** : portent le total courant de tout l'appel jusqu'ici, plus toute dépense restaurée quand l'appel a repris une session.

Dans un appel où votre application n'envoie jamais `/clear`, `/reset` ou `/new`, lisez le dernier résultat pour obtenir les totaux de l'appel au lieu d'additionner les résultats.

Les totaux courants repartent de zéro chaque fois que l'application envoie l'une de ces trois commandes ; à l'intérieur d'un appel `query()`, rien d'autre ne les réinitialise. Trois résultats comptent pour votre comptabilité :

* **Le résultat propre au tour `/clear`** : ne couvre que ce qui s'est exécuté depuis la réinitialisation, et porte un nouveau `session_id`.
* **Chaque résultat ultérieur** : continue de compter à partir de cette réinitialisation.
* **Le dernier résultat avant chaque `/clear`** : contient le total des tours depuis la réinitialisation précédente.

Pour totaliser l'appel entier, ajoutez le dernier résultat précédant chaque `/clear` au résultat final de l'appel. Tous les autres résultats, y compris celui du tour `/clear`, sont remplacés par un résultat ultérieur.

En TypeScript, le SDK émet aussi un [`SDKConversationResetMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkconversationresetmessage) à chaque réinitialisation, ce qui permet de détecter les réinitialisations dans le flux. En Python, le SDK émet de même un `ConversationResetMessage`. Avant le SDK Python v0.2.137, l'itérateur Python abandonnait ce message : sur ces versions, comptez vous-même les réinitialisations à partir des tours `/clear` envoyés par votre application.

`maxBudgetUsd` (TypeScript) ou `max_budget_usd` (Python) ne compte que la dépense propre à l'appel : les totaux restaurés depuis une session reprise ne sont pas imputés, et un `/clear` remet le budget à zéro.

## Obtenir le coût total d'une requête

Le message de résultat, typé [`SDKResultMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkresultmessage) en TypeScript et [`ResultMessage`](https://code.claude.com/docs/en/agent-sdk/python#resultmessage) en Python, marque la fin de la boucle de l'agent pour un appel `query()`. Il inclut `total_cost_usd`, le coût estimé cumulé de toutes les étapes de l'appel. Un appel qui reprend une session compte aussi la dépense antérieure de la session. Deux précautions à la lecture de cette valeur :

* En Python, le champ est typé optionnel : vérifiez qu'il n'est pas `None` avant de le lire.
* Les résultats de succès comme d'erreur le portent, mais le résultat final d'un [plantage de session](#récupérer-les-totaux-après-un-plantage-de-session) peut le porter à zéro.

En mode d'entrée en streaming, lisez les totaux d'appel comme décrit dans [Suivre les coûts en mode d'entrée en streaming](#suivre-les-coûts-en-mode-dentrée-en-streaming).

Les trois champs de niveau résultat diffèrent dans ce qu'ils comptent quand l'agent lance des [sous-agents](https://code.claude.com/docs/en/agent-sdk/subagents). Utilisez `modelUsage` (ou `model_usage` en Python) pour la comptabilité des tokens de tout l'arbre ; le champ `usage` sous-estime dès qu'il y a imbrication.

| Champ | Activité des sous-agents |
| - | - |
| `usage` | Exclue. Ne compte que la boucle de l'agent de premier niveau ; les tokens consommés dans les sous-agents ne sont pas ajoutés |
| `total_cost_usd` | Incluse. Compte les requêtes des sous-agents en plus de la boucle de premier niveau |
| `modelUsage` / `model_usage` | Incluse. Compte les requêtes des sous-agents en plus de la boucle de premier niveau, ventilées par modèle |

En [mode d'entrée à message unique](https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode#single-message-input), si des sous-agents en arrière-plan tournent encore à la fin du dernier tour, Claude Code les attend, jusqu'au plafond décrit dans [tâches en arrière-plan à la sortie](https://code.claude.com/docs/en/headless#background-tasks-at-exit), avant d'émettre le résultat. Les champs `total_cost_usd`, `duration_api_ms` et `modelUsage` (ou `model_usage` en Python) du résultat incluent le travail effectué pendant cette attente.

Les exemples suivants parcourent le flux de messages d'un appel `query()` et affichent le coût total à l'arrivée du message `result` :

### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

try {
  for await (const message of query({ prompt: "Summarize this project" })) {
    if (message.type === "result") {
      console.log(`Total cost: $${message.total_cost_usd}`);
    }
  }
} catch (error) {
  // Un query() single-shot lève une exception après avoir émis un résultat d'erreur. Si
  // l'échec était un résultat d'erreur, il portait quand même total_cost_usd et la
  // branche ci-dessus s'est déjà exécutée ; les échecs de connexion ou de processus
  // n'émettent aucun message de résultat.
  console.error(`Session ended with an error: ${error}`);
}
```

### Python

```python
from claude_agent_sdk import query, ResultMessage
import asyncio


async def main():
    try:
        async for message in query(prompt="Summarize this project"):
            if isinstance(message, ResultMessage):
                print(f"Total cost: ${message.total_cost_usd or 0}")
    except Exception as error:
        # Un query() single-shot lève une exception après avoir émis un résultat d'erreur. Si
        # l'échec était un résultat d'erreur, la branche ci-dessus s'est déjà exécutée ;
        # les échecs de connexion ou de processus n'émettent aucun message de résultat.
        print(f"Session ended with an error: {error}")


asyncio.run(main())
```

Pour borner ce que les sous-agents peuvent ajouter à `total_cost_usd`, définissez les [limites de profondeur, de concurrence et de dépense](https://code.claude.com/docs/en/agent-sdk/subagents#cap-subagent-depth-concurrency-and-spend) sur la requête.

## Suivre l'utilisation par étape et par modèle

Les exemples de cette section utilisent les noms de champs TypeScript. En Python, les champs équivalents sont [`AssistantMessage.usage`](https://code.claude.com/docs/en/agent-sdk/python#assistantmessage) et `AssistantMessage.message_id` pour l'utilisation par étape, et [`ResultMessage.model_usage`](https://code.claude.com/docs/en/agent-sdk/python#resultmessage) pour la ventilation par modèle.

### Suivre l'utilisation par étape

Chaque message assistant contient un `BetaMessage` imbriqué (accessible via `message.message`) avec un `id` et un objet `usage` contenant les comptes de tokens. Quand Claude utilise des outils en parallèle, plusieurs messages partagent le même `id` avec des données d'utilisation identiques. Gardez la trace des ID déjà comptés et ignorez les doublons pour éviter des totaux gonflés.

> **Avertissement** : les valeurs par étape dédupliquées sont exactes pour les tokens d'entrée et de cache. Le `output_tokens` par étape est une **valeur provisoire (placeholder)** : [lisez les tokens de sortie depuis le message de résultat](#lire-les-tokens-de-sortie-depuis-le-message-de-résultat).

L'exemple suivant cumule les tokens d'entrée sur toutes les étapes, en ne comptant qu'une fois chaque ID de message unique de la boucle principale et en ignorant les messages des sous-agents, puis lit le total de sortie depuis le message de résultat, qui couvre la boucle principale :

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

const seenIds = new Set<string>();
let totalInputTokens = 0;
let resultOutputTokens = 0;

try {
  for await (const message of query({ prompt: "Summarize this project" })) {
    if (message.type === "assistant" && !message.parent_tool_use_id) {
      const msgId = message.message.id;

      // Les appels d'outils parallèles partagent le même ID : ne compter qu'une fois
      if (!seenIds.has(msgId)) {
        seenIds.add(msgId);
        totalInputTokens += message.message.usage.input_tokens;
      }
    }
    if (message.type === "result") {
      // Le output_tokens par étape est provisoire ; le message de résultat
      // porte le total de sortie cumulé.
      resultOutputTokens = message.usage.output_tokens;
    }
  }
} catch (error) {
  // Un query() single-shot lève une exception après avoir émis un résultat d'erreur ; le
  // total d'entrée ci-dessous reflète donc encore les étapes exécutées avant l'échec.
  console.error(`Session ended with an error: ${error}`);
}

console.log(`Steps: ${seenIds.size}`);
console.log(`Input tokens: ${totalInputTokens}`);
console.log(`Output tokens: ${resultOutputTokens}`);
```

### Ventiler l'utilisation par modèle

Le message de résultat inclut [`modelUsage`](https://code.claude.com/docs/en/agent-sdk/typescript#modelusage), une table associant chaque nom de modèle à ses comptes de tokens et à son coût. C'est utile quand vous utilisez plusieurs modèles (par exemple Haiku pour les sous-agents et Opus pour l'agent principal) et voulez voir où vont les tokens.

Le champ `costBasis` de chaque entrée indique quelle table de prix a valorisé la dernière requête de ce modèle : `list` pour le prix catalogue, `managed` pour une table [`modelPricing`](https://code.claude.com/docs/en/settings-reference#modelpricing), ou `unknown` quand aucune ne correspondait à l'ID du modèle. Ce champ nécessite Claude Code v2.1.246+.

L'exemple suivant exécute une requête et affiche le coût et la ventilation des tokens pour chaque modèle utilisé :

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

try {
  for await (const message of query({ prompt: "Summarize this project" })) {
    if (message.type !== "result") continue;

    for (const [modelName, usage] of Object.entries(message.modelUsage)) {
      console.log(`${modelName}: $${usage.costUSD.toFixed(4)}`);
      console.log(`  Input tokens: ${usage.inputTokens}`);
      console.log(`  Output tokens: ${usage.outputTokens}`);
      console.log(`  Cache read: ${usage.cacheReadInputTokens}`);
      console.log(`  Cache creation: ${usage.cacheCreationInputTokens}`);
    }
  }
} catch (error) {
  // Un query() single-shot lève une exception après avoir émis un résultat d'erreur. Si
  // l'échec était un résultat d'erreur, la ventilation par modèle ci-dessus a déjà été
  // affichée ; les échecs de connexion ou de processus n'émettent aucun message de résultat.
  console.error(`Session ended with an error: ${error}`);
}
```

## Cumuler les coûts sur plusieurs appels

Chaque appel `query()` renvoie `total_cost_usd` dans ses résultats. La façon de combiner les valeurs dépend du fait que les appels partagent ou non une session :

* **Appels indépendants, sans option `resume` ni `continue`** : chaque résultat ne couvre que son propre appel ; additionnez vous-même les totaux, comme le font les exemples ci-dessous.
* **Appels qui reprennent la même session** : Claude Code enregistre les totaux de la session dans son [transcript](https://code.claude.com/docs/en/sessions#where-transcripts-are-stored) quand le processus se termine normalement, et les restaure quand un appel ultérieur reprend ou bifurque (fork) la session. Chaque résultat inclut déjà la dépense antérieure de la session. Lisez le dernier résultat pour le total de la session ; additionner les résultats compterait deux fois la dépense restaurée. Avant la v2.1.277, une session reprise via le SDK ou `claude -p` démarrait ses totaux à zéro, si bien que les résultats de chaque appel ne couvraient que cet appel.

Dans les deux cas, le chiffre combiné reste une [estimation côté client](#des-estimations-pas-de-la-facturation).

En mode d'entrée en streaming, lisez le total de chaque appel comme décrit dans [Suivre les coûts en mode d'entrée en streaming](#suivre-les-coûts-en-mode-dentrée-en-streaming). Pour un appel terminé par un plantage, voir [Récupérer les totaux après un plantage de session](#récupérer-les-totaux-après-un-plantage-de-session).

Les exemples suivants exécutent deux appels `query()` successifs, ajoutent le `total_cost_usd` de chaque appel à un total courant et affichent le coût par appel et le coût combiné :

### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

// Suivre le coût cumulé sur plusieurs appels query()
let totalSpend = 0;

const prompts = [
  "Read the files in src/ and summarize the architecture",
  "List all exported functions in src/auth.ts"
];

for (const prompt of prompts) {
  try {
    for await (const message of query({ prompt })) {
      if (message.type === "result") {
        totalSpend += message.total_cost_usd;
        console.log(`This call: $${message.total_cost_usd}`);
      }
    }
  } catch (error) {
    // Un query() single-shot lève une exception après avoir émis un résultat d'erreur. Si
    // l'échec était un résultat d'erreur, le coût de cet appel a déjà été compté ;
    // les échecs de connexion ou de processus n'émettent aucun message de résultat.
    // Continuer avec le prompt suivant.
    console.error(`Call failed: ${error}`);
  }
}

console.log(`Total spend: $${totalSpend.toFixed(4)}`);
```

### Python

```python
from claude_agent_sdk import query, ResultMessage
import asyncio


async def main():
    # Suivre le coût cumulé sur plusieurs appels query()
    total_spend = 0.0

    prompts = [
        "Read the files in src/ and summarize the architecture",
        "List all exported functions in src/auth.ts",
    ]

    for prompt in prompts:
        try:
            async for message in query(prompt=prompt):
                if isinstance(message, ResultMessage):
                    cost = message.total_cost_usd or 0
                    total_spend += cost
                    print(f"This call: ${cost}")
        except Exception as error:
            # Un query() single-shot lève une exception après avoir émis un résultat d'erreur.
            # Si l'échec était un résultat d'erreur, le coût de cet appel a déjà été
            # compté ; les échecs de connexion ou de processus n'émettent aucun message
            # de résultat. Continuer avec le prompt suivant.
            print(f"Call failed: {error}")

    print(f"Total spend: ${total_spend:.4f}")


asyncio.run(main())
```

## Gérer les erreurs, le cache et les comptes de tokens de sortie

Pour un suivi des coûts précis, tenez compte du compte de sortie provisoire des messages assistant, des tokens consommés par une conversation en échec et de la tarification des tokens de cache.

### Lire les tokens de sortie depuis le message de résultat

Claude Code construit chaque message assistant à partir de l'utilisation que l'API a rapportée au début de la réponse : le `output_tokens` du message n'est donc que le compte rapporté par l'API à `message_start`, avant la génération de la réponse. Une seule réponse de l'API peut produire plusieurs messages assistant, et chacun porte ce même placeholder.

L'API rapporte le vrai compte de sortie à la fin de la réponse, et Claude Code l'ajoute au message de résultat. Lisez les tokens de sortie dans l'`usage` du résultat, ou dans `modelUsage` pour une ventilation par modèle.

Pour voir le compte de sortie d'une réponse augmenter pendant le streaming, activez `includePartialMessages` (ou `include_partial_messages` en Python) et lisez `usage` dans chaque événement de flux `message_delta`, typé [`SDKPartialAssistantMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkpartialassistantmessage) en TypeScript et [`StreamEvent`](https://code.claude.com/docs/en/agent-sdk/python#streamevent) en Python.

### Suivre les coûts des conversations en échec

Les messages de résultat de succès comme d'erreur incluent `usage` et `total_cost_usd` ; en Python, ces deux champs sont typés optionnels : vérifiez qu'ils ne sont pas `None` avant de les lire.

Si une conversation échoue en cours de route, vous avez quand même consommé des tokens jusqu'au point d'échec. Lisez les données de coût de chaque message de résultat, que son `subtype` soit `success` ou l'un des sous-types d'erreur. Sur certains résultats d'erreur, `usage` rapporte moins que ce que l'appel a dépensé :

* **`error_during_execution` après un [plantage de session](#récupérer-les-totaux-après-un-plantage-de-session)** : tous les champs de coût peuvent être à zéro.
* **`error_max_budget_usd`** : `usage` omet la réponse qui a fait dépasser le budget, alors que `total_cost_usd` et `modelUsage` l'incluent.

Quand vous avez le choix, comptabilisez à partir de `total_cost_usd` ou de `modelUsage` plutôt que d'`usage`.

### Récupérer les totaux après un plantage de session

Quand le processus Claude Code plante, il émet un dernier résultat `error_during_execution` puis se termine, en mode single-shot comme en entrée en streaming. Ce résultat peut porter `usage`, `total_cost_usd` et `modelUsage` à zéro : récupérez donc les totaux de l'appel à partir de ce qui est arrivé avant. L'étape 1 récupère les totaux complets dès qu'un résultat antérieur existe ; le repli de l'étape 2 ne récupère que les tokens d'entrée et de cache de la boucle principale.

1. Utilisez le résultat du tour précédant le plantage. En mode d'entrée en streaming, il contient le total courant décrit dans [Suivre les coûts en mode d'entrée en streaming](#suivre-les-coûts-en-mode-dentrée-en-streaming). Passez plutôt à l'étape 2 quand ce résultat ne peut pas vous aider :
   * L'appel était single-shot, donc aucun résultat antérieur n'existe.
   * Le plantage est survenu au premier tour.
   * Le tour précédant le plantage était le `/clear` lui-même, son résultat ne couvrant donc que la réinitialisation.
2. Additionnez plutôt l'`usage` des messages assistant, en comptant chaque réponse de l'API une seule fois, comme le fait l'exemple [Suivre l'utilisation par étape](#suivre-lutilisation-par-étape). En mode single-shot, additionnez-les tous ; en mode d'entrée en streaming, additionnez ceux arrivés après le dernier résultat. Vous obtenez ainsi les tokens d'entrée et de cache de la boucle principale. L'utilisation des sous-agents n'est pas récupérable de cette façon, pas plus que les tokens de sortie ou le coût en USD, car [le `output_tokens` par étape est un placeholder](#lire-les-tokens-de-sortie-depuis-le-message-de-résultat).

### Suivre les tokens de cache

L'Agent SDK utilise automatiquement le [cache de prompt](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) pour réduire le coût des contenus répétés. Vous n'avez pas à configurer le cache vous-même. L'objet usage inclut deux champs supplémentaires pour le suivi du cache :

* `cache_creation_input_tokens` : tokens utilisés pour créer de nouvelles entrées de cache (facturés à un tarif plus élevé que les tokens d'entrée standard).
* `cache_read_input_tokens` : tokens lus depuis des entrées de cache existantes (facturés à tarif réduit).

Suivez-les séparément de `input_tokens` pour comprendre les économies réalisées grâce au cache. En TypeScript, ces champs sont typés sur l'objet [`Usage`](https://code.claude.com/docs/en/agent-sdk/typescript#usage). En Python, ils apparaissent comme clés du dict [`ResultMessage.usage`](https://code.claude.com/docs/en/agent-sdk/python#resultmessage) (par exemple `message.usage.get("cache_read_input_tokens", 0)`).

### Allonger le TTL du cache de prompt à une heure

Vos propres tours relèvent du [compartiment TTL de la conversation principale](https://code.claude.com/docs/en/prompt-caching#which-ttl-each-request-gets), avec les requêtes d'assistance que Claude Code exécute en ligne avec eux. Les requêtes que Claude Code fait hors de cette conversation, comme les [sous-agents](https://code.claude.com/docs/en/agent-sdk/subagents), ont un [contrôle de TTL séparé](https://code.claude.com/docs/en/prompt-caching#choose-the-ttl-yourself).

Les entrées de cache de vos propres tours utilisent par défaut un TTL de 5 minutes quand vous vous authentifiez avec une clé API ou que vous tournez sur Amazon Bedrock, Agent Platform de Google Cloud, Microsoft Foundry ou [Claude Platform on AWS](https://code.claude.com/docs/en/claude-platform-on-aws). Si votre charge exécute de nombreuses sessions courtes avec le même prompt système et le même contexte, espacées de plus de 5 minutes, le cache expire entre les sessions et chaque nouvelle session paie le plein tarif d'entrée.

Pour demander un TTL d'une heure sur les écritures en cache, définissez la variable d'environnement [`ENABLE_PROMPT_CACHING_1H`](https://code.claude.com/docs/en/env-vars). Vous pouvez l'exporter dans l'environnement de votre shell ou conteneur, ou la passer via `options.env`.

L'exemple suivant active le TTL d'une heure pour un agent tournant sur Amazon Bedrock. Comme il définit `CLAUDE_CODE_USE_BEDROCK`, il nécessite des identifiants AWS valides pour [Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock) ; sans eux, la requête échoue.

#### Python

```python
from claude_agent_sdk import ClaudeAgentOptions, query
import asyncio


async def main():
    options = ClaudeAgentOptions(
        env={
            "CLAUDE_CODE_USE_BEDROCK": "1",
            "ENABLE_PROMPT_CACHING_1H": "1",
        },
    )

    async for message in query(prompt="Summarize this project", options=options):
        print(message)


asyncio.run(main())
```

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

const options = {
  env: {
    ...process.env,
    CLAUDE_CODE_USE_BEDROCK: "1",
    ENABLE_PROMPT_CACHING_1H: "1",
  },
};

for await (const message of query({ prompt: "Summarize this project", options })) {
  console.log(message);
}
```

Les écritures en cache avec un TTL d'une heure sont facturées plus cher que les écritures à 5 minutes : activer cette option échange un coût d'écriture plus élevé contre davantage de lectures en cache. Voir la [tarification du cache de prompt](https://platform.claude.com/docs/en/build-with-claude/prompt-caching). Avec un abonnement Claude, dans l'utilisation incluse de votre forfait, vous bénéficiez du TTL d'une heure sur vos propres tours, et sur une partie des requêtes d'assistance que Claude Code fait à côté, sans définir cette variable ; Claude Code repasse ces tours au TTL de 5 minutes dès que vous puisez dans les [crédits d'utilisation](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans).

`ENABLE_PROMPT_CACHING_1H` demande le TTL d'une heure pour toutes les requêtes des deux compartiments. Pour choisir le TTL de chaque compartiment séparément, utilisez plutôt les contrôles suivants. Chacun accepte `5m` ou `1h` et prime sur `ENABLE_PROMPT_CACHING_1H` :

* Conversation principale : la [variable d'environnement](https://code.claude.com/docs/en/env-vars) `CLAUDE_CODE_PROMPT_CACHE_TTL`, ou le réglage [`promptCacheTtl`](https://code.claude.com/docs/en/settings-reference#promptcachettl)
* Tout le reste : la variable d'environnement `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL`, ou le réglage [`subagentPromptCacheTtl`](https://code.claude.com/docs/en/settings-reference#subagentpromptcachettl)

Définir `promptCacheTtl` à `1h` conserve le cache d'une heure sur la conversation principale pendant que vous puisez dans les crédits d'utilisation. Pour l'ordre de priorité complet, voir [choisir le TTL vous-même](https://code.claude.com/docs/en/prompt-caching#choose-the-ttl-yourself).

## Documentation associée

* [Référence du SDK TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript) : documentation complète de l'API
* [Vue d'ensemble du SDK](https://code.claude.com/docs/en/agent-sdk/overview) : bien démarrer avec le SDK
* [Permissions du SDK](https://code.claude.com/docs/en/agent-sdk/permissions) : gérer les permissions des outils
