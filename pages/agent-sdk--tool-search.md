# Passer à de nombreux outils grâce à la recherche d'outils

> Source : https://code.claude.com/docs/en/agent-sdk/tool-search · [← Index](../31-index-documentation-officielle.md)

*Faire passer votre agent à des milliers d'outils en ne découvrant et ne chargeant que ce qui est nécessaire, à la demande.*

La recherche d'outils (tool search) permet à votre agent de travailler avec des centaines ou des milliers d'outils en les découvrant et en les chargeant dynamiquement, à la demande. Au lieu de charger toutes les définitions d'outils dans la fenêtre de contexte dès le départ, l'agent cherche dans votre catalogue d'outils et ne charge que ceux dont il a besoin.

Cette approche résout deux problèmes qui apparaissent quand les bibliothèques d'outils grossissent :

- **Efficacité du contexte** : les définitions d'outils peuvent occuper une grande partie de la fenêtre de contexte (50 outils peuvent consommer 10 à 20 K tokens), ce qui laisse moins de place pour le vrai travail.
- **Précision du choix des outils** : la précision de sélection des outils se dégrade au-delà de 30 à 50 outils chargés en même temps.

## Fonctionnement de la recherche d'outils

La recherche d'outils est activée par défaut, avec les exceptions listées dans « Configurer la recherche d'outils ».

Lorsqu'elle est active, les définitions d'outils sont retenues hors de la fenêtre de contexte. L'agent reçoit un résumé des outils disponibles et cherche les outils pertinents quand la tâche exige une capacité pas encore chargée. Par défaut, jusqu'à cinq des outils les plus pertinents sont chargés dans le contexte, où ils restent disponibles pour les tours suivants jusqu'à ce que le SDK compacte les messages dans lesquels l'agent les a découverts. Après cette compaction, l'agent recherche à nouveau ces outils la prochaine fois qu'il en a besoin.

La recherche d'outils ajoute un aller-retour supplémentaire chaque fois que Claude cherche des outils, mais pour de grands ensembles d'outils, ce coût est compensé par un contexte plus petit à chaque tour. Avec moins d'environ 10 outils dont les définitions tiennent confortablement dans la fenêtre de contexte, tout charger dès le départ est généralement plus rapide.

Pour les détails du mécanisme d'API sous-jacent, voir [Tool search dans l'API](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool).

> **Note** : la recherche d'outils n'est pas prise en charge sur les [déploiements Microsoft Foundry hébergés sur Azure](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry#hosting-options), qui la rejettent côté serveur : le SDK détecte ce rejet et charge alors les définitions d'outils dès le départ pour ce déploiement. `ENABLE_TOOL_SEARCH` ne peut pas outrepasser ce comportement, puisque le rejet vient du déploiement lui-même.

## Configurer la recherche d'outils

La recherche d'outils est activée par défaut. Pour les modèles figurant sur la liste des modèles non pris en charge du SDK, celui-ci charge les définitions d'outils dès le départ, et aucune valeur de `ENABLE_TOOL_SEARCH` ne l'outrepasse. Sur Agent Platform de Google Cloud, le SDK décide selon la génération du modèle :

- **Claude Opus 4.5, Sonnet 4.5, Haiku 4.5 et ultérieurs** : la recherche d'outils est activée par défaut.
- **Modèles Agent Platform antérieurs** : le SDK charge les définitions d'outils dès le départ, car leurs infrastructures de service rejettent l'en-tête bêta requis. `ENABLE_TOOL_SEARCH` ne peut pas outrepasser ce comportement.

Avant Claude Code v2.1.221, le SDK désactivait la recherche d'outils pour tous les modèles sur Agent Platform de Google Cloud, sauf si vous définissiez `ENABLE_TOOL_SEARCH`.

Le SDK désactive aussi la recherche d'outils lorsque `ANTHROPIC_BASE_URL` pointe vers un hôte qui n'est pas first-party (non Anthropic), car la plupart des proxys ne transmettent pas les blocs `tool_reference`. Vous pouvez outrepasser ce défaut avec la variable d'environnement `ENABLE_TOOL_SEARCH` :

| Valeur | Comportement |
| :- | :- |
| (non définie) | La recherche d'outils est active. Les définitions d'outils sont différées et découvertes à la demande. Repli sur le chargement dès le départ pour : les modèles antérieurs à Opus 4 et Sonnet 4 ; les modèles Agent Platform de Google Cloud antérieurs à Opus 4.5, Sonnet 4.5 et Haiku 4.5 ; un `ANTHROPIC_BASE_URL` non first-party ; un déploiement Microsoft Foundry hébergé sur Azure. |
| `true` | La recherche d'outils reste active avec un `ANTHROPIC_BASE_URL` non first-party. Les autres replis (du cas « non définie ») s'appliquent toujours. Le SDK envoie l'en-tête bêta à travers les proxys, et les requêtes échouent sur les proxys qui ne prennent pas en charge les blocs `tool_reference`. |
| `auto` | Compte les tokens des définitions d'outils que la recherche d'outils peut différer et compare le total à la fenêtre de contexte du modèle. Quand le total atteint 10 % de la fenêtre, la recherche d'outils s'active. En dessous, le SDK charge toutes les définitions d'outils dans le contexte dès le départ. |
| `auto:N` | Comme `auto`, avec un pourcentage personnalisé. `auto:5` s'active quand ces définitions atteignent 5 % de la fenêtre de contexte. Plus la valeur est basse, plus l'activation est précoce. |
| `false` | La recherche d'outils est désactivée. Toutes les définitions d'outils sont chargées dans le contexte à chaque tour. |

Définir [`CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS`](https://code.claude.com/docs/en/env-vars) maintient la recherche d'outils désactivée, et vous ne pouvez pas l'outrepasser en définissant vous-même `ENABLE_TOOL_SEARCH`. À partir de Claude Code v2.1.227, votre organisation peut maintenir la recherche d'outils activée malgré cette variable via les [réglages gérés (managed settings)](https://code.claude.com/docs/en/managed-settings). Dans une session qui [reçoit la configuration HIPAA](https://code.claude.com/docs/en/hipaa-setup#check-how-developers-sign-in-and-connect), la recherche d'outils est désactivée, et ni `ENABLE_TOOL_SEARCH` ni les réglages gérés ne la réactivent. [Désactiver les capacités en pré-version](https://code.claude.com/docs/en/llm-gateway-protocol#disable-pre-release-capabilities) précise où s'applique cette dérogation et ce que la variable retire.

La recherche d'outils s'applique à tous les outils enregistrés, qu'ils proviennent de serveurs MCP distants ou de [serveurs MCP SDK personnalisés](https://code.claude.com/docs/en/agent-sdk/custom-tools). Avec `auto`, le SDK compte chaque définition que la recherche d'outils peut différer dans un seuil combiné unique : chaque outil MCP non marqué [`alwaysLoad`](https://code.claude.com/docs/en/mcp#exempt-a-server-from-deferral), quel que soit son serveur, plus les outils intégrés qui se chargent à la demande. Le SDK charge dès le départ les outils intégrés essentiels comme Bash, Read et Edit, et ne les compte pas dans le seuil. Un [mod](https://code.claude.com/docs/en/plugins/mods/reference#tools) de l'un de vos plugins peut aussi différer un outil ou le charger dès le départ, ce qui modifie ce décompte.

Définissez la valeur dans l'option `env` de `query()`. En TypeScript, `env` remplace l'environnement du sous-processus : étalez `...process.env` pour conserver les variables héritées. En Python, `env` est fusionné par-dessus l'environnement hérité. Cet exemple se connecte à un serveur MCP distant qui expose de nombreux outils, les pré-approuve tous avec un joker (wildcard), et utilise `auto:5` pour que la recherche d'outils s'active lorsque les définitions qu'elle peut différer atteignent 5 % de la fenêtre de contexte :

### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

try {
  for await (const message of query({
    prompt: "Find and run the appropriate database query",
    options: {
      mcpServers: {
        "enterprise-tools": {
          // Se connecte à un serveur MCP distant
          type: "http",
          url: "https://tools.example.com/mcp"
        }
      },
      allowedTools: ["mcp__enterprise-tools__*"], // Le joker pré-approuve tous les outils de ce serveur
      env: {
        ...process.env, // env remplace l'environnement du sous-processus : on garde les variables héritées
        ENABLE_TOOL_SEARCH: "auto:5" // Active la recherche d'outils quand les définitions différables atteignent 5 % du contexte
      }
    }
  })) {
    if (message.type === "result" && message.subtype === "success") {
      console.log(message.result);
    }
  }
} catch (error) {
  // Un query() en un seul coup lève une exception après avoir produit un résultat d'erreur
  console.log(`Session ended with an error: ${error}`);
}
```

### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


async def main():
    options = ClaudeAgentOptions(
        mcp_servers={
            "enterprise-tools": {
                "type": "http",
                "url": "https://tools.example.com/mcp",
            }
        },
        allowed_tools=[
            "mcp__enterprise-tools__*"
        ],  # Le joker pré-approuve tous les outils de ce serveur
        env={
            "ENABLE_TOOL_SEARCH": "auto:5"  # Active la recherche d'outils quand les définitions différables atteignent 5 % du contexte
        },
    )

    try:
        async for message in query(
            prompt="Find and run the appropriate database query",
            options=options,
        ):
            if isinstance(message, ResultMessage) and message.subtype == "success":
                print(message.result)
    except Exception as error:
        # Un query() en un seul coup lève une exception après avoir produit un résultat d'erreur
        print(f"Session ended with an error: {error}")


asyncio.run(main())
```

Pour exécuter cet exemple, remplacez `https://tools.example.com/mcp` par l'URL de votre propre serveur MCP. En cas de succès, le texte du résultat s'affiche dans la console.

Comme il s'agit d'un appel `query()` en un seul coup (single-shot), le SDK lève une exception après avoir produit un résultat d'erreur ; l'exemple enveloppe donc la boucle dans un bloc try. Pour savoir pourquoi une exécution a échoué, vérifiez le `subtype` du message de résultat (par exemple `error_during_execution`) à l'intérieur de la boucle. Pour en savoir plus sur les messages de résultat, voir [Traiter le résultat](https://code.claude.com/docs/en/agent-sdk/agent-loop#handle-the-result).

## Optimiser la découverte des outils

Le mécanisme de recherche compare les requêtes aux noms et aux descriptions des outils. Un nom comme `search_slack_messages` ressort pour un éventail de demandes plus large que `query_slack`. Les descriptions contenant des mots-clés précis (« Search Slack messages by keyword, channel, or date range ») correspondent à davantage de requêtes que les descriptions génériques (« Query Slack »).

Vous pouvez aussi ajouter au prompt système une section listant les catégories d'outils disponibles. Cela donne à l'agent une idée des types d'outils qu'il peut rechercher. Passez le texte dans l'option `systemPrompt` de `query()` en TypeScript, ou `system_prompt` en Python. Utilisez le preset `claude_code` avec `append`, qui ajoute votre texte au prompt du preset au lieu de le remplacer :

### TypeScript

```typescript
options: {
  systemPrompt: {
    type: "preset",
    preset: "claude_code",
    append: "You can search for tools to interact with Slack, GitHub, and Jira."
  }
}
```

### Python

```python
options = ClaudeAgentOptions(
    system_prompt={
        "type": "preset",
        "preset": "claude_code",
        "append": "You can search for tools to interact with Slack, GitHub, and Jira.",
    }
)
```

Pour l'ensemble des options du prompt système, voir [Modifier les prompts système](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts).

## Limites

- **Nombre maximal d'outils** : 10 000 outils dans votre catalogue.
- **Résultats de recherche** : renvoie par défaut jusqu'à cinq outils parmi les plus pertinents par recherche.
- **Modèles pris en charge** : Opus 4, Sonnet 4, Haiku 4.5 et modèles ultérieurs ; ou Opus 4.5, Sonnet 4.5, Haiku 4.5 et modèles ultérieurs sur Agent Platform de Google Cloud. Sur les modèles antérieurs, le SDK charge les définitions d'outils dès le départ, comme décrit dans « Configurer la recherche d'outils ».

## Documentation associée

- [Tool search dans l'API](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool) : documentation complète de l'API pour la recherche d'outils, y compris les implémentations personnalisées.
- [Connecter des serveurs MCP](https://code.claude.com/docs/en/agent-sdk/mcp) : se connecter à des outils externes via des serveurs MCP.
- [Outils personnalisés](https://code.claude.com/docs/en/agent-sdk/custom-tools) : créer vos propres outils avec des serveurs MCP SDK.
- [Référence du SDK TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript) : référence complète de l'API.
- [Référence du SDK Python](https://code.claude.com/docs/en/agent-sdk/python) : référence complète de l'API.
