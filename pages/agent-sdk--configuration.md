# Configurer votre agent

> Source : https://code.claude.com/docs/en/agent-sdk/configuration · [← Index](../31-index-documentation-officielle.md)

*Configurer les sessions de l'Agent SDK : composer l'objet d'options, définir le modèle, l'environnement et les limites, et trouver la page de chaque option de fonctionnalité.*

Une session de l'Agent SDK lit sa configuration depuis les fichiers de réglages, les variables d'environnement et l'objet `options` passé au démarrage. Cette page montre comment composer l'objet `options` et ce que contrôlent les fichiers de réglages et les variables d'environnement.

Pour le type et la valeur par défaut de chaque option, voir les références [`Options`](https://code.claude.com/docs/en/agent-sdk/typescript#options) (TypeScript) et [`ClaudeAgentOptions`](https://code.claude.com/docs/en/agent-sdk/python#claudeagentoptions) (Python).

## Passer des options à une session

Chaque appel `query()` accepte un objet d'options : `Options` en TypeScript, `ClaudeAgentOptions` en Python. Chaque champ est facultatif ; une session démarrée sans option tourne avec les valeurs par défaut du SDK. L'exemple ci-dessous configure une session en lecture seule qui résume les TODO ouverts d'un projet. Les paires se lisent TypeScript / Python quand l'orthographe diffère :

* **`model`** : choisit le modèle
* **`allowedTools` / `allowed_tools`** : pré-approuve une liste d'outils en lecture seule
* **`maxTurns` / `max_turns`** : plafonne le nombre de tours
* **`cwd`** : définit le répertoire de travail

### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Summarize the open TODOs in this repo",
  options: {
    model: "claude-sonnet-5",
    allowedTools: ["Read", "Glob", "Grep"],
    maxTurns: 8,
    cwd: "/path/to/repo",
  },
})) {
  if (message.type === "result" && message.subtype === "success" && !message.is_error) {
    console.log(message.result);
  }
}
```

### Python

```python
import asyncio

from claude_agent_sdk import ClaudeAgentOptions, ResultMessage, query

async def main():
    options = ClaudeAgentOptions(
        model="claude-sonnet-5",
        allowed_tools=["Read", "Glob", "Grep"],
        max_turns=8,
        cwd="/path/to/repo",
    )

    async for message in query(
        prompt="Summarize the open TODOs in this repo",
        options=options,
    ):
        if isinstance(message, ResultMessage) and not message.is_error:
            print(message.result)

asyncio.run(main())
```

Pointez `cwd` vers l'un de vos projets et lancez l'exemple : le résumé des TODO ouverts du projet s'affiche à l'arrivée du message de résultat.

`allowedTools` (TypeScript) ou `allowed_tools` (Python) pré-approuve les outils listés : leurs appels s'exécutent sans s'arrêter pour demander une approbation. Les outils hors de la liste **restent disponibles**. Quand Claude appelle un outil non listé, c'est le mode de permission qui décide si l'appel s'exécute. Voir [Règles allow et deny](https://code.claude.com/docs/en/agent-sdk/permissions#allow-and-deny-rules).

## Charger les fichiers de réglages

Les fichiers de réglages fournissent de la configuration au-delà de l'objet d'options. Deux options contrôlent leur chargement :

* **`settingSources` / `setting_sources`** : contrôle quelles sources du système de fichiers se chargent : user, project et local. Les fichiers de réglages et les fichiers CLAUDE.md arrivent par ces sources.
* **`settings`** : charge un chemin de fichier de réglages ou une chaîne JSON inline dans les deux langages ; TypeScript accepte aussi un objet de réglages. Quelle que soit la forme passée, elle prime sur les réglages utilisateur, projet et local du système de fichiers ; seuls les réglages de politique gérée ont un rang supérieur. Les références documentent l'ordre de priorité complet dans [Settings precedence](https://code.claude.com/docs/en/agent-sdk/typescript#settings-precedence) pour TypeScript et [Settings precedence](https://code.claude.com/docs/en/agent-sdk/python#settings-precedence) pour Python.

Passez `[]` pour désactiver les réglages utilisateur, projet et local. Voir [Utiliser les fonctionnalités de Claude Code dans le SDK](https://code.claude.com/docs/en/agent-sdk/claude-code-features).

## Choisir un modèle

Sauf si l'option `model`, vos réglages ou votre environnement sélectionnent un modèle, une nouvelle session démarre sur le [modèle par défaut de Claude Code](https://code.claude.com/docs/en/model-config#default-model-setting). Pour l'ordre de ces sources, voir [Définir votre modèle](https://code.claude.com/docs/en/model-config#setting-your-model). Définissez `model` pour épingler un modèle précis, ou pour choisir un modèle plus petit pour des agents plus rapides et moins chers. La valeur accepte un alias de modèle ou un nom de modèle complet ; les alias et les versions vers lesquelles ils pointent sont listés dans [Alias de modèles](https://code.claude.com/docs/en/model-config#model-aliases).

Définissez `fallbackModel` (TypeScript) ou `fallback_model` (Python) pour nommer un modèle de secours. Quand le modèle principal est surchargé ou indisponible, la session bascule sur le secours. Le principal est retenté au début de chaque tour utilisateur, si bien que la session y revient une fois la panne passée.

Dans les deux langages, l'option accepte un seul modèle ou une liste de secours séparés par des virgules. Pour l'ordre et le plafond de la chaîne, voir [Chaînes de modèles de secours](https://code.claude.com/docs/en/model-config#fallback-model-chains). En TypeScript, un modèle de secours égal à `model` lève une erreur au démarrage.

Les exemples ci-dessous montrent une liste de secours en TypeScript et un secours unique en Python :

### TypeScript

```typescript
const options = {
  model: "claude-fable-5",
  fallbackModel: "claude-opus-5,claude-sonnet-5",
};
```

### Python

```python
options = ClaudeAgentOptions(
    model="claude-fable-5",
    fallback_model="claude-opus-5",
)
```

### Paramètres d'échantillonnage

> **Note** : les paramètres de requête de la [Messages API](https://platform.claude.com/docs/en/api/messages) `temperature`, `top_p` et `max_tokens` n'ont **aucun champ** dans l'objet d'options, dans aucun des deux langages. Réglez plutôt le [niveau d'effort](https://code.claude.com/docs/en/agent-sdk/agent-loop#effort-level) ou un [plafond de dépense](#limiter-les-tours-et-la-dépense), ou appelez directement la Messages API si vous avez besoin de ces paramètres.

## Définir des variables d'environnement

L'option `env` définit les variables d'environnement du processus Claude Code qui exécute la session. Le fait que vos valeurs remplacent l'environnement hérité ou s'y fusionnent dépend du langage :

* **TypeScript** : `env` **remplace** l'environnement du sous-processus
* **Python** : le SDK **fusionne** vos valeurs par-dessus l'environnement hérité, vos valeurs prenant le pas sur celles héritées

En TypeScript, étalez `process.env` dans `env` pour conserver les variables héritées comme `PATH`, `HOME` et `ANTHROPIC_API_KEY`. Si vous laissez `env` non défini, le sous-processus hérite de votre environnement dans les deux langages.

L'exemple fait passer le trafic API par une passerelle (gateway) en définissant `ANTHROPIC_BASE_URL`.

### TypeScript

```typescript
const options = {
  env: { ...process.env, ANTHROPIC_BASE_URL: "https://gateway.example.com" },
};
```

### Python

```python
options = ClaudeAgentOptions(
    env={"ANTHROPIC_BASE_URL": "https://gateway.example.com"},
)
```

Les variables passées peuvent aussi configurer Claude Code lui-même. Pour les variables lues par le processus Claude Code, voir [Variables d'environnement](https://code.claude.com/docs/en/env-vars). Pour régler ainsi les délais d'expiration de l'API et la détection de blocage, suivez la section « Handle slow or stalled API responses » de la [référence TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript#handle-slow-or-stalled-api-responses) ou de la [référence Python](https://code.claude.com/docs/en/agent-sdk/python#handle-slow-or-stalled-api-responses).

## Définir le répertoire de travail

Définissez `cwd` pour exécuter la session dans un répertoire précis. Si `cwd` n'est pas défini, la session s'exécute dans le répertoire de travail de votre processus. Aucun des deux SDK n'a de setter pour `cwd` : pour travailler dans un autre répertoire, démarrez une autre session avec ce `cwd`.

Claude Code lit le répertoire de travail pour déterminer :

* **Réglages et hooks du projet** : quels [réglages et hooks de projet se chargent](https://code.claude.com/docs/en/agent-sdk/claude-code-features)
* **Skills** : où les [skills de session sont découvertes](https://code.claude.com/docs/en/agent-sdk/skills)
* **Stockage de session** : à quel projet [appartient une session stockée](https://code.claude.com/docs/en/agent-sdk/session-storage)

Pour permettre aux outils d'atteindre des fichiers hors du répertoire de travail, ajoutez des chemins avec `additionalDirectories` (TypeScript) ou `add_dirs` (Python). Pour la portée de cette autorisation, voir [Les répertoires supplémentaires donnent accès aux fichiers, pas à la configuration](https://code.claude.com/docs/en/permissions#additional-directories-grant-file-access-not-configuration).

## Limiter les tours et la dépense

Plafonnez les tours et la dépense avec `maxTurns` / `max_turns` et `maxBudgetUsd` / `max_budget_usd`. Les deux plafonds sont désactivés quand ils ne sont pas définis. Quand une session atteint un plafond, l'exécution se termine par un message de résultat dont le sous-type nomme le plafond : `error_max_turns` ou `error_max_budget_usd`. La suite dépend du mode d'entrée :

* **`query()` en un seul coup (single-shot)** : le SDK émet le résultat de plafond puis lève une exception ; entourez la boucle d'un bloc try pour continuer au-delà de l'erreur
* **Entrée en streaming** : la session reste vivante après un résultat de plafond, et le compteur de tours max repart de zéro pour chaque message en file. Le budget total s'accumule entre les messages ; une fois la dépense arrivée au plafond, les messages suivants de la même conversation se terminent par le même résultat de budget. Un [`/clear`](https://code.claude.com/docs/en/agent-sdk/cost-tracking) remet le budget à zéro

Les deux plafonds traitent `0` différemment :

* **`maxTurns` / `max_turns`** : `0` exécute la session sans limite de tours, comme si l'option n'était pas définie
* **`maxBudgetUsd` / `max_budget_usd`** : la CLI rejette `0` comme montant invalide au démarrage, et la session ne s'exécute jamais

Pour en savoir plus sur les deux plafonds, y compris la dépense des sous-agents, voir [Tours et budget](https://code.claude.com/docs/en/agent-sdk/agent-loop#turns-and-budget).

## Modifier la configuration en cours de session

Quand vous démarrez une session avec l'[entrée en streaming](https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode), vous pouvez changer son modèle et son mode de permission pendant qu'elle tourne. L'endroit où appeler les setters dépend du langage :

* **TypeScript** : méthodes de l'objet renvoyé par `query()`
* **Python** : méthodes de [`ClaudeSDKClient`](https://code.claude.com/docs/en/agent-sdk/python#claudesdkclient), car `query()` renvoie un simple itérateur sans méthodes de contrôle

Les deux langages ont les mêmes setters :

* **`setModel()` / `set_model()`** : change de modèle. Appelé sans modèle, il bascule sur le [modèle par défaut de Claude Code](https://code.claude.com/docs/en/model-config#default-model-setting) plutôt que sur le `model` passé dans les options.
* **`setPermissionMode()` / `set_permission_mode()`** : change le mode de permission

TypeScript dispose en plus de `applyFlagSettings()` et `updateSettings()` :

* **`applyFlagSettings()`** : applique des réglages à l'exécution, par exemple `await session.applyFlagSettings({ effortLevel: "high" })`. La méthode prend des clés de fichier de réglages et non des champs d'options ; consultez la [référence `applyFlagSettings()`](https://code.claude.com/docs/en/agent-sdk/typescript#applyflagsettings) pour le schéma et pour savoir quelles clés prennent effet en cours de session.
* **`updateSettings()`** : écrit une clé autorisée (allowlist) dans un fichier de réglages. La [référence `updateSettings()`](https://code.claude.com/docs/en/agent-sdk/typescript#updatesettings) indique la clé acceptée par chaque source et les versions minimales.
  * Passez `"localSettings"` pour écrire dans le fichier de réglages locaux du projet, par exemple `await session.updateSettings("localSettings", { outputStyle: "Explanatory" })`. La clé écrite prend effet à la requête suivante de la session et persiste pour les sessions ultérieures qui chargent les réglages `local`.
  * Passez `"userSettings"` pour écrire `effortLevel`, la seule clé acceptée par cette source. Claude Code l'enregistre comme niveau d'effort par défaut pour le modèle courant de la session ; l'effort de la session en cours ne change pas.

L'exemple ci-dessous exécute une session à deux tours, modifie la configuration entre les tours et affiche le modèle ayant répondu à chaque tour. En TypeScript, le flux de prompts retient le second message jusqu'à ce que les setters aient été exécutés, et le second tour s'exécute sur le nouveau modèle.

### TypeScript

```typescript
import { query, type SDKUserMessage } from "@anthropic-ai/claude-agent-sdk";

function userMessage(text: string): SDKUserMessage {
  return { type: "user", message: { role: "user", content: text }, parent_tool_use_id: null };
}

// Retenir le second prompt jusqu'à ce que les setters aient été exécutés.
let startSecondTurn!: () => void;
const secondTurnReady = new Promise<void>((resolve) => {
  startSecondTurn = resolve;
});

async function* turnPrompts(): AsyncGenerator<SDKUserMessage, void> {
  yield userMessage("Reply with exactly: ready");
  await secondTurnReady;
  yield userMessage("Reply with exactly: done");
}

const session = query({
  prompt: turnPrompts(),
  options: {
    model: "claude-sonnet-5",
  },
});

let turnModel = "";
let completedTurns = 0;

for await (const message of session) {
  if (message.type === "assistant") {
    turnModel = message.message.model;
  } else if (message.type === "result") {
    completedTurns += 1;
    if (completedTurns === 1) {
      console.log(`First turn model: ${turnModel}`);
      await session.setModel("claude-opus-5");
      await session.setPermissionMode("acceptEdits");
      startSecondTurn();
    } else {
      console.log(`Second turn model: ${turnModel}`);
      break;
    }
  }
}
```

### Python

```python
import asyncio

from claude_agent_sdk import AssistantMessage, ClaudeAgentOptions, ClaudeSDKClient

async def main():
    options = ClaudeAgentOptions(model="claude-sonnet-5")

    async with ClaudeSDKClient(options=options) as client:
        await client.query("Reply with exactly: ready")
        first_model = ""
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                first_model = message.model

        await client.set_model("claude-opus-5")
        await client.set_permission_mode("acceptEdits")

        await client.query("Reply with exactly: done")
        second_model = ""
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                second_model = message.model

    print(f"First turn model: {first_model}")
    print(f"Second turn model: {second_model}")

asyncio.run(main())
```

Sur la Claude API, le programme affiche `First turn model: claude-sonnet-5`, puis `Second turn model: claude-opus-5` après le changement.

> **Note** : chaque modèle a son propre cache de prompt ; après un changement de modèle en cours de session, la requête suivante recalcule toute la conversation sans cache, au tarif du nouveau modèle. Voir [Changer de modèle](https://code.claude.com/docs/en/prompt-caching#switching-models).

## Configurer des fonctionnalités spécifiques

Le tableau ci-dessous associe chaque option à la fonctionnalité qu'elle configure. Pour les options non couvertes ici, voir les références [TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript#options) et [Python](https://code.claude.com/docs/en/agent-sdk/python#claudeagentoptions). Si vous connaissez votre objectif mais pas l'option qui y répond, partez de [Choisir la bonne fonctionnalité](https://code.claude.com/docs/en/agent-sdk/claude-code-features#choose-the-right-feature).

| TypeScript | Python | Contrôle | Documenté dans |
| - | - | - | - |
| `permissionMode` | `permission_mode` | Ce que l'agent peut faire sans approbation | [Configurer les permissions](https://code.claude.com/docs/en/agent-sdk/permissions) |
| `allowedTools` | `allowed_tools` | Quels appels d'outils sont pré-approuvés | [Configurer les permissions](https://code.claude.com/docs/en/agent-sdk/permissions) |
| `canUseTool` | `can_use_tool` | Votre callback d'approbation des appels d'outils | [Gérer les demandes d'approbation d'outils](https://code.claude.com/docs/en/agent-sdk/user-input#handle-tool-approval-requests) |
| `systemPrompt` | `system_prompt` | Les instructions de l'agent | [Modifier les prompts système](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts) |
| `settingSources` | `setting_sources` | Quels réglages du système de fichiers se chargent | [Utiliser les fonctionnalités de Claude Code dans le SDK](https://code.claude.com/docs/en/agent-sdk/claude-code-features) |
| `mcpServers` | `mcp_servers` | Serveurs d'outils externes | [Se connecter à des outils externes avec MCP](https://code.claude.com/docs/en/agent-sdk/mcp) |
| `agents` | `agents` | Définitions de sous-agents | [Sous-agents](https://code.claude.com/docs/en/agent-sdk/subagents) |
| `hooks` | `hooks` | Callbacks aux points du cycle de vie | [Hooks](https://code.claude.com/docs/en/agent-sdk/hooks) |
| `skills` | `skills` | Quelles skills se chargent | [Étendre les agents avec des skills](https://code.claude.com/docs/en/agent-sdk/skills) |
| `plugins` | `plugins` | Quels plugins se chargent | [Plugins](https://code.claude.com/docs/en/agent-sdk/plugins) |
| `outputFormat` | `output_format` | Schémas de sortie structurée | [Sorties structurées](https://code.claude.com/docs/en/agent-sdk/structured-outputs) |
| `resume` | `resume` | Reprise d'une session stockée | [Sessions](https://code.claude.com/docs/en/agent-sdk/sessions) |
| `forkSession` | `fork_session` | Bifurcation (fork) d'une session | [Sessions](https://code.claude.com/docs/en/agent-sdk/sessions) |
| `sessionStore` | `session_store` | Persistance externe des sessions | [Stockage des sessions](https://code.claude.com/docs/en/agent-sdk/session-storage) |
| `enableFileCheckpointing` | `enable_file_checkpointing` | Modifications de fichiers réversibles (rembobinables) | [Points de contrôle de fichiers](https://code.claude.com/docs/en/agent-sdk/file-checkpointing) |
| `effort` | `effort` | Quantité de travail que Claude consacre aux réponses | [Niveau d'effort](https://code.claude.com/docs/en/agent-sdk/agent-loop#effort-level) |
| `sandbox` | `sandbox` | Comportement du sandbox pour l'exécution des outils | Références [TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript#sandbox-configuration) et [Python](https://code.claude.com/docs/en/agent-sdk/python#sandbox-configuration), avec le contexte de déploiement dans [Déploiement sécurisé](https://code.claude.com/docs/en/agent-sdk/secure-deployment) |

## Étapes suivantes

Pour voir la configuration assemblée dans des agents fonctionnels :

* **[Démarrage rapide](https://code.claude.com/docs/en/agent-sdk/quickstart)** : construire et exécuter un premier agent de bout en bout
* **[Exemples](https://code.claude.com/docs/en/agent-sdk/examples)** : trouver un projet complet et exécutable, ou une recette guidée du Claude Cookbook, correspondant à ce que vous voulez construire
* **[Isolation multi-locataire](https://code.claude.com/docs/en/agent-sdk/hosting#multi-tenant-isolation)** : isoler les réglages et la mémoire de chaque locataire avec `settingSources` / `setting_sources`, `env` et `cwd`
