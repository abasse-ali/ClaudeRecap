# Sous-agents dans le SDK

> Source : https://code.claude.com/docs/en/agent-sdk/subagents · [← Index](../31-index-documentation-officielle.md)

*Définir et invoquer des sous-agents pour isoler le contexte, exécuter des tâches en parallèle et appliquer des instructions spécialisées dans vos applications Claude Agent SDK.*

Les sous-agents (subagents) sont des instances d'agent distinctes que votre agent principal peut lancer pour traiter des sous-tâches ciblées. Utilisez-les pour isoler le contexte, mener plusieurs analyses en parallèle et appliquer des instructions spécialisées sans alourdir le prompt de l'agent principal.

## Vue d'ensemble

Il existe trois façons de créer des sous-agents :

- **Par programme** : utilisez le paramètre `agents` dans les options de `query()`. Voir les références [TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript#agentdefinition) et [Python](https://code.claude.com/docs/en/agent-sdk/python#agentdefinition).
- **Par fichiers** : définissez les agents comme fichiers Markdown dans des répertoires `.claude/agents/`. Voir [définir des sous-agents sous forme de fichiers](https://code.claude.com/docs/en/sub-agents).
- **Agent intégré généraliste** : Claude peut invoquer à tout moment le sous-agent intégré `general-purpose` via l'outil Agent, sans que vous ayez à définir quoi que ce soit.

Ce guide se concentre sur l'approche par programme, recommandée pour les applications SDK.

## Avantages des sous-agents

Comme les sous-agents sont des instances d'agent distinctes, leur déléguer du travail apporte quatre avantages :

- **Isolation du contexte** : chaque sous-agent s'exécute dans sa propre conversation, qui démarre vierge sauf si le sous-agent est un [fork](https://code.claude.com/docs/en/sub-agents#fork-the-current-conversation). Dans tous les cas, les appels d'outils intermédiaires et leurs résultats restent à l'intérieur du sous-agent ; seul son message final revient au parent. Un sous-agent `research-assistant` peut explorer des dizaines de fichiers sans que ce contenu ne s'accumule dans la conversation principale. Le parent reçoit un résumé concis, pas chaque fichier lu par le sous-agent. Voir la section « Ce dont héritent les sous-agents » pour savoir exactement ce que contient le contexte du sous-agent.
- **Parallélisation** : plusieurs sous-agents peuvent s'exécuter simultanément ; des sous-tâches indépendantes se terminent donc dans le temps de la plus lente, et non dans la somme de toutes. Lors d'une revue de code, vous pouvez lancer les sous-agents `style-checker`, `security-scanner` et `test-coverage` en même temps plutôt que l'un après l'autre.
- **Instructions et connaissances spécialisées** : chaque sous-agent peut avoir un prompt système sur mesure, avec une expertise, des bonnes pratiques et des contraintes spécifiques. Un sous-agent `database-migration` peut disposer de connaissances détaillées sur les bonnes pratiques SQL, les stratégies de rollback et les contrôles d'intégrité des données, qui seraient du bruit inutile dans les instructions de l'agent principal.
- **Restrictions d'outils** : les sous-agents peuvent être limités à certains outils, ce qui réduit le risque d'actions non voulues. Un sous-agent `doc-reviewer` peut n'avoir accès qu'aux outils Read et Grep, garantissant qu'il peut analyser mais jamais modifier accidentellement vos fichiers de documentation.

## Créer des sous-agents

### Définition par programme (recommandée)

Définissez les sous-agents directement dans votre code avec le paramètre `agents`. Claude invoque les sous-agents via l'outil `Agent`.

La plupart des exemples de cette page n'affichent que le résultat final. Pour confirmer que Claude a bien délégué à un sous-agent plutôt que de répondre directement, voir la section « Détecter l'invocation d'un sous-agent ».

Cet exemple crée deux sous-agents : un relecteur de code en lecture seule et un exécuteur de tests capable de lancer des commandes.

#### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition


async def main():
    async for message in query(
        prompt="Review the authentication module for security issues",
        options=ClaudeAgentOptions(
            # Approuve automatiquement ces outils
            allowed_tools=["Read", "Grep", "Glob", "Agent"],
            agents={
                "code-reviewer": AgentDefinition(
                    # description indique à Claude quand utiliser ce sous-agent
                    description="Expert code review specialist. Use for quality, security, and maintainability reviews.",
                    # prompt définit le comportement et l'expertise du sous-agent
                    prompt="""You are a code review specialist with expertise in security, performance, and best practices.

When reviewing code:
- Identify security vulnerabilities
- Check for performance issues
- Verify adherence to coding standards
- Suggest specific improvements

Be thorough but concise in your feedback.""",
                    # tools restreint ce que le sous-agent peut faire (ici en lecture seule)
                    tools=["Read", "Grep", "Glob"],
                    # model remplace le modèle par défaut pour ce sous-agent
                    model="sonnet",
                ),
                "test-runner": AgentDefinition(
                    description="Runs and analyzes test suites. Use for test execution and coverage analysis.",
                    prompt="""You are a test execution specialist. Run tests and provide clear analysis of results.

Focus on:
- Running test commands
- Analyzing test output
- Identifying failing tests
- Suggesting fixes for failures""",
                    # L'accès à Bash permet à ce sous-agent de lancer les commandes de test
                    tools=["Bash", "Read", "Grep"],
                ),
            },
        ),
    ):
        if hasattr(message, "result"):
            print(message.result)


asyncio.run(main())
```

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Review the authentication module for security issues",
  options: {
    // Approuve automatiquement ces outils
    allowedTools: ["Read", "Grep", "Glob", "Agent"],
    agents: {
      "code-reviewer": {
        // description indique à Claude quand utiliser ce sous-agent
        description:
          "Expert code review specialist. Use for quality, security, and maintainability reviews.",
        // prompt définit le comportement et l'expertise du sous-agent
        prompt: `You are a code review specialist with expertise in security, performance, and best practices.

When reviewing code:
- Identify security vulnerabilities
- Check for performance issues
- Verify adherence to coding standards
- Suggest specific improvements

Be thorough but concise in your feedback.`,
        // tools restreint ce que le sous-agent peut faire (ici en lecture seule)
        tools: ["Read", "Grep", "Glob"],
        // model remplace le modèle par défaut pour ce sous-agent
        model: "sonnet"
      },
      "test-runner": {
        description:
          "Runs and analyzes test suites. Use for test execution and coverage analysis.",
        prompt: `You are a test execution specialist. Run tests and provide clear analysis of results.

Focus on:
- Running test commands
- Analyzing test output
- Identifying failing tests
- Suggesting fixes for failures`,
        // L'accès à Bash permet à ce sous-agent de lancer les commandes de test
        tools: ["Bash", "Read", "Grep"]
      }
    }
  }
})) {
  if ("result" in message) console.log(message.result);
}
```

### Configuration d'AgentDefinition

| Champ | Type | Obligatoire | Description |
| :- | :- | :- | :- |
| `description` | `string` | Oui | Description en langage naturel indiquant quand utiliser cet agent |
| `prompt` | `string` | Oui | Prompt système de l'agent, qui définit son rôle et son comportement |
| `tools` | `string[]` | Non | Tableau des noms d'outils autorisés. S'il est omis, l'agent hérite de chaque [outil disponible pour les sous-agents](https://code.claude.com/docs/en/sub-agents#available-tools) |
| `disallowedTools` | `string[]` | Non | Tableau de noms d'outils à retirer du jeu d'outils de l'agent. Les motifs au niveau d'un serveur MCP sont aussi acceptés : `mcp__server` ou `mcp__server__*` retire tous les outils de ce serveur, et `mcp__*` retire tous les outils MCP de tous les serveurs |
| `model` | `string` | Non | Modèle spécifique pour cet agent. Accepte un alias comme `'fable'`, `'opus'`, `'sonnet'`, `'haiku'`, `'inherit'`, ou un ID de modèle complet. `'inherit'` utilise le modèle principal. Si vous l'omettez, Claude Code choisit le modèle selon l'[ordre de choix du modèle des sous-agents](https://code.claude.com/docs/en/sub-agents#choose-a-model) |
| `skills` | `string[]` | Non | Liste de noms de skills à précharger dans le contexte de l'agent au démarrage. Les skills non listés restent invocables via l'outil Skill |
| `memory` | `'user' \| 'project' \| 'local'` | Non | Source de mémoire de cet agent |
| `mcpServers` | `(string \| object)[]` | Non | Serveurs MCP disponibles pour cet agent, par nom ou par configuration en ligne |
| `initialPrompt` | `string` | Non | Soumis automatiquement comme premier tour utilisateur quand cet agent s'exécute comme agent du thread principal. Ignoré quand l'agent est invoqué comme sous-agent |
| `maxTurns` | `number` | Non | Nombre maximal de tours agentiques avant l'arrêt de l'agent. Quand l'agent atteint la limite, Claude Code renvoie sa sortie marquée comme partielle, et vous pouvez [reprendre l'agent](#reprendre-des-sous-agents) pour continuer. Le marquage « partiel » exige Claude Code v2.1.246+ |
| `background` | `boolean` | Non | Exécute cet agent comme tâche d'arrière-plan non bloquante lorsqu'il est invoqué |
| `omitClaudeMd` | `boolean` | Non | Exclut les fichiers CLAUDE.md utilisateur, projet et local du contexte initial de l'agent ; les fichiers de politique gérée (managed policy) sont toujours chargés. Ignoré quand l'agent s'exécute comme agent du thread principal. Exige le TypeScript Agent SDK v0.3.271+. L'[`AgentDefinition`](https://code.claude.com/docs/en/agent-sdk/python#agentdefinition) du SDK Python n'a pas ce champ |
| `effort` | `'low' \| 'medium' \| 'high' \| 'xhigh' \| 'max' \| number` | Non | Niveau d'effort de raisonnement de cet agent |
| `permissionMode` | `PermissionMode` | Non | Mode de permission pour l'exécution des outils dans cet agent. Les [règles d'héritage des sous-agents](https://code.claude.com/docs/en/agent-sdk/permissions#available-modes) déterminent quand il s'applique |

Dans le SDK Python, les noms de champs composés de plusieurs mots, comme `disallowedTools` et `mcpServers`, gardent leur orthographe camelCase pour correspondre au format filaire (wire format), au lieu de suivre la convention snake_case de Python. Voir la [référence `AgentDefinition`](https://code.claude.com/docs/en/agent-sdk/python#agentdefinition) pour les détails.

Les sous-agents s'exécutent en arrière-plan par défaut. Un appel à l'outil Agent qui omet l'entrée [`run_in_background`](https://code.claude.com/docs/en/sub-agents#run-subagents-in-foreground-or-background) lance un sous-agent en arrière-plan, et Claude définit `run_in_background: false` lorsqu'il a besoin du résultat avant de continuer. Définissez le champ `background` à `true` pour forcer l'exécution en arrière-plan d'un agent donné, quoi que Claude demande. Avant Claude Code v2.1.198, ce défaut « arrière-plan » était déployé progressivement, et un appel à l'outil Agent sans `run_in_background` pouvait exécuter le sous-agent de façon synchrone.

Les sous-agents peuvent aussi lancer leurs propres sous-agents. Pour limiter la profondeur de cette imbrication, le nombre de sous-agents simultanés et la dépense d'une requête, voir la section « Plafonner la profondeur, la concurrence et la dépense des sous-agents ».

### Définition par fichiers (alternative)

Vous pouvez aussi définir les sous-agents comme fichiers Markdown dans des répertoires `.claude/agents/`. Voir la [documentation des sous-agents de Claude Code](https://code.claude.com/docs/en/sub-agents) pour cette approche. Les agents définis par programme ont priorité sur les agents définis par fichiers portant le même nom.

> **Note** : quand Claude appelle l'outil Agent sans `subagent_type`, il obtient le sous-agent intégré `general-purpose`, que Claude peut lancer même si vous ne définissez aucun agent. Définir [`CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS=1`](https://code.claude.com/docs/en/env-vars) supprime ce défaut, et un tel appel échoue alors avec [`subagent_type is required`](https://code.claude.com/docs/en/errors#subagent-type-is-required).

## Ce dont héritent les sous-agents

Sauf si le sous-agent est un [fork](https://code.claude.com/docs/en/sub-agents#fork-the-current-conversation), sa fenêtre de contexte démarre vierge, sans la conversation du parent, mais elle n'est pas vide. Le seul contenu transmis du parent au sous-agent est la chaîne de prompt de l'outil Agent : incluez donc directement dans ce prompt les chemins de fichiers, messages d'erreur ou décisions dont le sous-agent a besoin.

Un sous-agent qui dispose de l'outil [`SendMessage`](https://code.claude.com/docs/en/tools-reference) démarre avec la liste des autres agents nommés en cours d'exécution dans la session, afin de savoir à quels noms il peut envoyer des messages. Claude Code ajoute automatiquement cette liste au premier tour du sous-agent. Un [fork](https://code.claude.com/docs/en/sub-agents#fork-the-current-conversation) ne reçoit pas cette liste, car il hérite à la place de la conversation du parent.

Un sous-agent hérite aussi de la configuration de réflexion étendue (extended thinking) de la session principale.

Le tableau ci-dessous indique ce que contient le contexte d'un sous-agent non-fork et ce qu'il exclut.

| Le sous-agent reçoit | Le sous-agent ne reçoit pas |
| :- | :- |
| Son propre prompt système (`AgentDefinition.prompt`) et le prompt de l'outil Agent | L'historique de conversation du parent ni ses résultats d'outils |
| Le CLAUDE.md du projet (chargé via [`settingSources`](https://code.claude.com/docs/en/agent-sdk/claude-code-features#control-filesystem-settings-with-settingsources)). Un agent qui définit `omitClaudeMd` démarre sans lui | Le contenu des skills préchargés, sauf s'ils sont listés dans `AgentDefinition.skills` |
| Les définitions d'outils (héritées du parent, ou le sous-ensemble défini dans `tools`, [filtrées pour les exécutions en arrière-plan](https://code.claude.com/docs/en/sub-agents#available-tools)) | Le prompt système du parent |

> **Note** : le parent reçoit le rapport final du sous-agent, mais peut le résumer dans sa propre réponse. Pour conserver la sortie du sous-agent mot pour mot dans la réponse destinée à l'utilisateur, ajoutez une instruction en ce sens dans le prompt ou dans l'option `systemPrompt` que vous passez à l'appel `query()` principal.
>
> En v2.1.210+, Claude Code [analyse le message final à la recherche de motifs ressemblant à des instructions](https://code.claude.com/docs/en/sub-agents#subagent-output-scanning) avant que le parent ne le lise. L'analyse traite trois types de motifs différemment :
>
> - **Imitation de balises de contrôle** : Claude Code neutralise sur place une balise que seul le harnais émet, comme un bloc `<system-reminder>`. Il insère une barre oblique inverse (backslash) après le chevron ouvrant et ne supprime rien.
> - **Mentions de la configuration des permissions** : Claude Code conserve telles quelles les références à la configuration des permissions, comme `.claude/settings.json`, `bypassPermissions` ou `--dangerously-skip-permissions`.
> - **Marqueurs de tour** : une ligne qui commence par `Human:` ou `Assistant:` reçoit une barre oblique inverse avant les deux-points, pour que le message ne puisse pas imiter une frontière de tour de conversation.
>
> Pour une correspondance de type balise de contrôle ou configuration des permissions, Claude Code ajoute en tête une ligne de marqueur `[harness: ...]` qui nomme les motifs détectés ; une correspondance de type marqueur de tour n'ajoute pas cette ligne. Ce sont les seules modifications effectuées par l'analyse : elle ne supprime ni ne reformule jamais le texte du sous-agent.

Une erreur d'API qui interrompt prématurément le sous-agent, comme une limite de débit (rate limit), n'est jamais livrée comme son résultat. Voir [Erreurs d'API dans les sous-agents](https://code.claude.com/docs/en/sub-agents#api-errors-in-subagents) pour le comportement au premier plan et en arrière-plan.

## Invoquer des sous-agents

### Invocation automatique

Claude décide automatiquement quand invoquer les sous-agents en fonction de la tâche et de la `description` de chaque sous-agent. Par exemple, si vous définissez un sous-agent `performance-optimizer` avec la description « Performance optimization specialist for query tuning », Claude l'invoquera quand votre prompt parlera d'optimiser des requêtes.

Rédigez des descriptions claires et précises pour que Claude puisse associer les tâches au bon sous-agent.

### Invocation explicite

Pour garantir que Claude utilise un sous-agent donné, mentionnez-le par son nom dans votre prompt :

```text
"Use the code-reviewer agent to check the authentication module"
```

Cela contourne la correspondance automatique et invoque directement le sous-agent nommé.

### Configuration dynamique des agents

Vous pouvez créer des définitions d'agents dynamiquement selon les conditions d'exécution. Cet exemple crée un relecteur de sécurité avec différents niveaux de sévérité, en utilisant un modèle plus capable pour les revues strictes.

#### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition


# Fonction fabrique qui renvoie un AgentDefinition
# Ce motif permet de personnaliser les agents selon les conditions d'exécution
def create_security_agent(security_level: str) -> AgentDefinition:
    is_strict = security_level == "strict"
    return AgentDefinition(
        description="Security code reviewer",
        # Personnalise le prompt selon le niveau de sévérité
        prompt=f"You are a {'strict' if is_strict else 'balanced'} security reviewer...",
        tools=["Read", "Grep", "Glob"],
        # Idée clé : utiliser un modèle plus capable pour les revues à fort enjeu
        model="opus" if is_strict else "sonnet",
    )


async def main():
    # L'agent est créé au moment de la requête : chaque requête peut donc utiliser des réglages différents
    async for message in query(
        prompt="Review this PR for security issues",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Grep", "Glob", "Agent"],
            agents={
                # Appelle la fabrique avec la configuration souhaitée
                "security-reviewer": create_security_agent("strict")
            },
        ),
    ):
        if hasattr(message, "result"):
            print(message.result)


asyncio.run(main())
```

#### TypeScript

```typescript
import { query, type AgentDefinition } from "@anthropic-ai/claude-agent-sdk";

// Fonction fabrique qui renvoie un AgentDefinition
// Ce motif permet de personnaliser les agents selon les conditions d'exécution
function createSecurityAgent(securityLevel: "basic" | "strict"): AgentDefinition {
  const isStrict = securityLevel === "strict";
  return {
    description: "Security code reviewer",
    // Personnalise le prompt selon le niveau de sévérité
    prompt: `You are a ${isStrict ? "strict" : "balanced"} security reviewer...`,
    tools: ["Read", "Grep", "Glob"],
    // Idée clé : utiliser un modèle plus capable pour les revues à fort enjeu
    model: isStrict ? "opus" : "sonnet"
  };
}

// L'agent est créé au moment de la requête : chaque requête peut donc utiliser des réglages différents
for await (const message of query({
  prompt: "Review this PR for security issues",
  options: {
    allowedTools: ["Read", "Grep", "Glob", "Agent"],
    agents: {
      // Appelle la fabrique avec la configuration souhaitée
      "security-reviewer": createSecurityAgent("strict")
    }
  }
})) {
  if ("result" in message) console.log(message.result);
}
```

## Détecter l'invocation d'un sous-agent

Claude invoque les sous-agents via l'outil Agent. Pour détecter l'invocation d'un sous-agent, repérez les blocs `tool_use` dont le `name` vaut `"Agent"`.

Les messages produits dans le contexte d'un sous-agent comportent un champ `parent_tool_use_id`. En TypeScript, chaque message assistant et utilisateur produit par un sous-agent porte aussi [`agent_id`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkassistantmessage) : le `task_id` des [événements de tâche](https://code.claude.com/docs/en/agent-sdk/typescript#sdktaskstartedmessage) de ce sous-agent. `agent_id` exige le TypeScript Agent SDK v0.3.292+.

> **Note** : l'outil apparaît sous le nom `"Agent"` dans les blocs `tool_use`, mais sous le nom `"Task"` dans la liste d'outils de `system:init`. Avant Claude Code v2.1.63, les blocs `tool_use` le nommaient aussi `"Task"`. Pour que la détection fonctionne quelle que soit la version du SDK, testez les deux valeurs dans `block.name`.

La structure des messages diffère selon le SDK. En Python, vous accédez directement aux blocs de contenu via `message.content`. En TypeScript, `SDKAssistantMessage` enveloppe le message de l'API Claude : vous accédez donc au contenu via `message.message.content`.

Cet exemple parcourt les messages diffusés en flux, journalise l'invocation d'un sous-agent et signale les messages suivants qui proviennent du contexte d'exécution de ce sous-agent. La version TypeScript journalise aussi l'`agent_id` de chaque message de sous-agent qui en porte un.

#### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition, ToolUseBlock


async def main():
    async for message in query(
        prompt="Use the code-reviewer agent to review this codebase",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Glob", "Grep", "Agent"],
            agents={
                "code-reviewer": AgentDefinition(
                    description="Expert code reviewer.",
                    prompt="Analyze code quality and suggest improvements.",
                    tools=["Read", "Glob", "Grep"],
                )
            },
        ),
    ):
        # Vérifie l'invocation d'un sous-agent. Teste les deux noms : les anciennes
        # versions du SDK émettaient "Task", les versions actuelles émettent "Agent".
        if hasattr(message, "content") and message.content:
            for block in message.content:
                if isinstance(block, ToolUseBlock) and block.name in (
                    "Task",
                    "Agent",
                ):
                    print(f"Subagent invoked: {block.input.get('subagent_type')}")

        # Vérifie si ce message provient du contexte d'un sous-agent
        if hasattr(message, "parent_tool_use_id") and message.parent_tool_use_id:
            print("  (running inside subagent)")

        if hasattr(message, "result"):
            print(message.result)


asyncio.run(main())
```

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Use the code-reviewer agent to review this codebase",
  options: {
    allowedTools: ["Read", "Glob", "Grep", "Agent"],
    agents: {
      "code-reviewer": {
        description: "Expert code reviewer.",
        prompt: "Analyze code quality and suggest improvements.",
        tools: ["Read", "Glob", "Grep"]
      }
    }
  }
})) {
  const msg = message as any;

  // Vérifie l'invocation d'un sous-agent. Teste les deux noms : les anciennes
  // versions du SDK émettaient "Task", les versions actuelles émettent "Agent".
  for (const block of msg.message?.content ?? []) {
    if (block.type === "tool_use" && (block.name === "Task" || block.name === "Agent")) {
      console.log(`Subagent invoked: ${block.input.subagent_type}`);
    }
  }

  // Vérifie si ce message provient du contexte d'un sous-agent
  if (msg.parent_tool_use_id) {
    console.log("  (running inside subagent)");
    // Sur les messages assistant et utilisateur, agent_id correspond au task_id
    // présent sur task_started et les autres événements de tâche de ce sous-agent
    if (msg.agent_id) {
      console.log(`  agent_id: ${msg.agent_id}`);
    }
  }

  if ("result" in message) {
    console.log(message.result);
  }
}
```

## Reprendre des sous-agents

Vous pouvez reprendre un sous-agent pour qu'il continue là où il s'était arrêté, au lieu de repartir de zéro. Un sous-agent repris conserve tout son historique de conversation, y compris tous ses appels d'outils, résultats et raisonnements antérieurs.

Quand un sous-agent s'arrête à sa limite `maxTurns` (voir « Configuration d'AgentDefinition »), Claude Code marque la sortie dans le résultat de l'outil Agent comme partielle, pour que Claude sache que l'exécution n'est pas terminée.

Quand un sous-agent se termine, le résultat de l'outil Agent contient un bloc de texte avec `agentId: <id>`. Les agents intégrés [`Explore` et `Plan`](https://code.claude.com/docs/en/sub-agents#built-in-subagents) sont à usage unique (one-shot) et ne renvoient pas d'`agentId` : utilisez donc un agent personnalisé ou `general-purpose` si vous devez reprendre. Pour reprendre un sous-agent par programme :

1. **Capturer l'ID de session** : extrayez `session_id` des messages pendant la première requête.
2. **Extraire l'ID de l'agent** : analysez `agentId` dans le texte du résultat de l'outil Agent.
3. **Reprendre la session** : passez `resume: sessionId` dans les options de la deuxième requête et incluez l'ID de l'agent dans votre prompt. Chaque appel `query()` démarre une nouvelle session par défaut, et vous devez reprendre la même session pour accéder à la transcription du sous-agent.

> **Note** : si vous utilisez un agent personnalisé, passez la même définition d'agent dans le paramètre `agents` pour les deux requêtes.

L'exemple ci-dessous définit un agent personnalisé `endpoint-finder`. La première requête l'exécute et capture l'ID de session et l'ID de l'agent depuis le résultat de l'outil Agent ; la seconde reprend la session pour poser une question de suivi qui nécessite le contexte de la première analyse.

#### Python

```python
import asyncio
import re
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition, ToolResultBlock

AGENTS = {
    "endpoint-finder": AgentDefinition(
        description="Locates and catalogs API endpoints in a codebase.",
        prompt="You find and document API endpoints. Report each endpoint's path, method, and handler.",
        tools=["Read", "Grep", "Glob"],
    )
}


def extract_agent_id(block: ToolResultBlock) -> str | None:
    """Extrait agentId du contenu texte d'un résultat de l'outil Agent."""
    parts = block.content if isinstance(block.content, list) else [{"text": block.content}]
    for part in parts:
        if match := re.search(r"agentId:\s*([\w-]+)", part.get("text") or ""):
            return match.group(1)
    return None


async def main():
    agent_id = None
    session_id = None

    # Première invocation - exécute le sous-agent endpoint-finder
    try:
        async for message in query(
            prompt="Use the endpoint-finder agent to find all API endpoints in this codebase",
            options=ClaudeAgentOptions(allowed_tools=["Read", "Grep", "Glob", "Agent"], agents=AGENTS),
        ):
            # Capture session_id depuis ResultMessage (nécessaire pour reprendre cette session)
            if hasattr(message, "session_id"):
                session_id = message.session_id
            # Cherche la mention agentId dans les résultats d'outils
            for block in getattr(message, "content", None) or []:
                if isinstance(block, ToolResultBlock):
                    agent_id = extract_agent_id(block) or agent_id
            # Affiche le résultat final
            if hasattr(message, "result"):
                print(message.result)
    except Exception as error:
        # Un query() en un seul coup lève une exception après avoir produit un résultat d'erreur,
        # donc session_id et agent_id ont déjà été capturés par la boucle ci-dessus.
        print(f"Session ended with an error: {error}")

    # Deuxième invocation - reprend et pose une question de suivi
    if agent_id and session_id:
        async for message in query(
            prompt=f"Resume agent {agent_id} and list the top 3 most complex endpoints",
            options=ClaudeAgentOptions(
                allowed_tools=["Read", "Grep", "Glob", "Agent"], agents=AGENTS, resume=session_id
            ),
        ):
            if hasattr(message, "result"):
                print(message.result)
    else:
        print("No agentId found in the first query, so there is no subagent to resume.")


asyncio.run(main())
```

#### TypeScript

```typescript
import { query, type SDKMessage } from "@anthropic-ai/claude-agent-sdk";

const agents = {
  "endpoint-finder": {
    description: "Locates and catalogs API endpoints in a codebase.",
    prompt: "You find and document API endpoints. Report each endpoint's path, method, and handler.",
    tools: ["Read", "Grep", "Glob"]
  }
};

// Sérialise le contenu pour chercher agentId sans parcourir les types de blocs imbriqués
function extractAgentId(message: SDKMessage): string | undefined {
  if (message.type !== "assistant" && message.type !== "user") return undefined;
  const content = JSON.stringify(message.message.content);
  const match = content.match(/agentId:\s*([\w-]+)/);
  return match?.[1];
}

let agentId: string | undefined;
let sessionId: string | undefined;

// Première invocation - exécute le sous-agent endpoint-finder
try {
  for await (const message of query({
    prompt: "Use the endpoint-finder agent to find all API endpoints in this codebase",
    options: { allowedTools: ["Read", "Grep", "Glob", "Agent"], agents }
  })) {
    // Capture session_id depuis ResultMessage (nécessaire pour reprendre cette session)
    if ("session_id" in message) sessionId = message.session_id;
    // Cherche agentId dans le contenu des messages (il apparaît dans les résultats de l'outil Agent)
    const extractedId = extractAgentId(message);
    if (extractedId) agentId = extractedId;
    // Affiche le résultat final
    if ("result" in message) console.log(message.result);
  }
} catch (error) {
  // Un query() en un seul coup lève une exception après avoir produit un résultat d'erreur,
  // donc sessionId et agentId ont déjà été capturés par la boucle ci-dessus.
  console.error(`Session ended with an error: ${error}`);
}

// Deuxième invocation - reprend et pose une question de suivi
if (agentId && sessionId) {
  for await (const message of query({
    prompt: `Resume agent ${agentId} and list the top 3 most complex endpoints`,
    options: { allowedTools: ["Read", "Grep", "Glob", "Agent"], agents, resume: sessionId }
  })) {
    if ("result" in message) console.log(message.result);
  }
} else {
  console.log("No agentId found in the first query, so there is no subagent to resume.");
}
```

Les transcriptions des sous-agents sont stockées dans des fichiers séparés et persistent indépendamment de la conversation principale. Voir [reprendre des sous-agents dans Claude Code](https://code.claude.com/docs/en/sub-agents#resume-subagents) pour le comportement de compaction et la période de nettoyage `cleanupPeriodDays`.

## Restrictions d'outils

Utilisez le champ `tools` pour limiter ce qu'un sous-agent peut faire :

- **Omettre `tools`** : le sous-agent obtient chaque [outil disponible pour les sous-agents](https://code.claude.com/docs/en/sub-agents#available-tools).
- **Lister des outils** : le sous-agent n'obtient que ceux-là. Un relecteur de code qui ne doit jamais modifier de fichiers, par exemple, reçoit `["Read", "Grep", "Glob"]`.

Un outil que vous omettez n'existe pas du tout dans la session du sous-agent : Claude travaille sans lui, sans demande de permission ni erreur.

Cet exemple crée un agent d'analyse en lecture seule, capable d'examiner le code mais pas de modifier des fichiers ni d'exécuter des commandes.

#### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition


async def main():
    async for message in query(
        prompt="Analyze the architecture of this codebase",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Grep", "Glob", "Agent"],
            agents={
                "code-analyzer": AgentDefinition(
                    description="Static code analysis and architecture review",
                    prompt="""You are a code architecture analyst. Analyze code structure,
identify patterns, and suggest improvements without making changes.""",
                    # Outils en lecture seule : pas d'accès à Edit, Write ni Bash
                    tools=["Read", "Grep", "Glob"],
                )
            },
        ),
    ):
        if hasattr(message, "result"):
            print(message.result)


asyncio.run(main())
```

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Analyze the architecture of this codebase",
  options: {
    allowedTools: ["Read", "Grep", "Glob", "Agent"],
    agents: {
      "code-analyzer": {
        description: "Static code analysis and architecture review",
        prompt: `You are a code architecture analyst. Analyze code structure,
identify patterns, and suggest improvements without making changes.`,
        // Outils en lecture seule : pas d'accès à Edit, Write ni Bash
        tools: ["Read", "Grep", "Glob"]
      }
    }
  }
})) {
  if ("result" in message) console.log(message.result);
}
```

### Combinaisons d'outils courantes

| Cas d'usage | Outils | Description |
| :- | :- | :- |
| Analyse en lecture seule | `Read`, `Grep`, `Glob` | Peut examiner le code, mais ni le modifier ni exécuter quoi que ce soit |
| Exécution de tests | `Bash`, `Read`, `Grep` | Peut lancer des commandes et analyser leur sortie |
| Modification de code | `Read`, `Edit`, `Write`, `Grep`, `Glob` | Accès complet en lecture/écriture sans exécution de commandes |
| Accès complet | Tous les outils | Hérite des outils disponibles pour les sous-agents (omettre le champ `tools`) |

## Plafonner la profondeur, la concurrence et la dépense des sous-agents

> **Note** : cette section décrit le TypeScript SDK v0.3.219+ et le Python SDK v0.2.127+, c'est-à-dire les versions qui embarquent Claude Code v2.1.219+. Sur les versions antérieures, certaines de ces limites sont absentes ou ont d'autres valeurs par défaut : mettez à jour avant de compter sur elles pour borner une exécution. La [référence des variables d'environnement](https://code.claude.com/docs/en/env-vars) et la page [tours et budget](https://code.claude.com/docs/en/agent-sdk/agent-loop#turns-and-budget) indiquent la version de Claude Code qui a ajouté chaque variable ainsi que l'application du plafond de dépense aux sous-agents.

Claude décide seul quand lancer un sous-agent et combien en lancer. Chaque sous-agent effectue ses propres requêtes d'API, qui comptent dans le `total_cost_usd` de la requête, et un sous-agent peut lancer ses propres sous-agents : un seul prompt peut donc se transformer en arbre d'agents.

Vous pouvez plafonner cette croissance de trois façons : la profondeur d'imbrication des sous-agents, le nombre de sous-agents simultanés et la dépense totale de la requête. Définissez les limites de profondeur et de concurrence comme variables d'environnement via l'option [`env`](https://code.claude.com/docs/en/agent-sdk/typescript#options), et la limite de dépense comme option de requête :

| Limite | Comment la définir | Valeur par défaut | Ce que fait Claude Code à la limite |
| :- | :- | :- | :- |
| Profondeur | [`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`](https://code.claude.com/docs/en/env-vars) | `3` niveaux de sous-agents sous votre agent principal. `1` empêche vos sous-agents d'en lancer d'autres | Un sous-agent du niveau le plus bas ne peut plus en lancer, et effectue donc lui-même le travail délégué. Voir [sous-agents imbriqués](https://code.claude.com/docs/en/sub-agents#let-subagents-spawn-their-own-subagents) |
| Concurrence | [`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS`](https://code.claude.com/docs/en/env-vars) | `20` sous-agents simultanés, en comptant chaque sous-agent que Claude lance avec l'outil Agent | Refuse de lancer un sous-agent supplémentaire, en renvoyant `Concurrent subagent limit reached`, jusqu'à ce que le nombre en cours repasse sous la limite. Les sessions où [ultracode](https://code.claude.com/docs/en/model-config#adjust-effort-level) est actif ne sont jamais refusées. Voir la [limite de sous-agents simultanés](https://code.claude.com/docs/en/sub-agents#concurrent-subagent-limit) |
| Dépense | `maxBudgetUsd` en TypeScript, `max_budget_usd` en Python | Aucune limite. Compte la dépense propre de l'appel, requêtes des sous-agents incluses | Applique le plafond de trois façons : refuse de lancer d'autres sous-agents en renvoyant `Budget limit reached`, arrête les sous-agents d'arrière-plan encore en cours, et termine la requête avec le sous-type de résultat `error_max_budget_usd`. Pour le comportement des plafonds sur une session, voir [tours et budget](https://code.claude.com/docs/en/agent-sdk/agent-loop#turns-and-budget) |

Les deux SDK traitent l'option `env` différemment : le SDK TypeScript remplace l'environnement du sous-processus par cette valeur, donc étalez `process.env` dedans pour conserver des variables comme `PATH`, tandis que le SDK Python la fusionne avec l'environnement hérité. Cet exemple désactive l'imbrication, autorise au plus cinq sous-agents simultanés et arrête la requête dès que la dépense estimée atteint 5 $ :

#### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


async def main():
    try:
        async for message in query(
            prompt="Audit every service in this repo for unhandled promise rejections",
            options=ClaudeAgentOptions(
                allowed_tools=["Read", "Grep", "Glob", "Agent"],
                # env est fusionné par-dessus l'environnement hérité
                env={
                    "CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH": "1",
                    "CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS": "5",
                },
                max_budget_usd=5.0,
            ),
        ):
            if isinstance(message, ResultMessage):
                print(f"{message.subtype}: ${message.total_cost_usd}")
    except Exception as error:
        # Un query() en un seul coup lève une exception après avoir produit un résultat d'erreur,
        # donc le résultat plafonné par le budget a déjà été affiché ci-dessus.
        print(f"Session ended with an error: {error}")


asyncio.run(main())
```

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

try {
  for await (const message of query({
    prompt: "Audit every service in this repo for unhandled promise rejections",
    options: {
      allowedTools: ["Read", "Grep", "Glob", "Agent"],
      // env remplace l'environnement du sous-processus : étalez process.env pour garder PATH
      env: {
        ...process.env,
        CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH: "1",
        CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS: "5",
      },
      maxBudgetUsd: 5,
    },
  })) {
    if (message.type === "result") {
      console.log(`${message.subtype}: $${message.total_cost_usd}`);
    }
  }
} catch (error) {
  // Un query() en un seul coup lève une exception après avoir produit un résultat d'erreur,
  // donc le résultat plafonné par le budget a déjà été journalisé ci-dessus.
  console.error(`Session ended with an error: ${error}`);
}
```

Ce que vous voyez dépend de la limite éventuellement atteinte par la requête :

- **Sous le plafond de dépense** : vous voyez `success` et le coût estimé.
- **Au plafond de dépense** : vous voyez `error_max_budget_usd` avec un coût supérieur ou égal à `5`, puis votre gestionnaire d'erreur s'exécute.
- **À la limite de concurrence** : vous voyez dans le flux de messages un bloc `tool_result` contenant `Concurrent subagent limit reached`. Claude reçoit ce même bloc comme résultat de l'outil Agent.

### Exécuter Opus 5 avec des sous-agents

Claude Opus 5 délègue aux sous-agents plus volontiers que les modèles précédents : les limites de profondeur, de concurrence et de dépense (section précédente) comptent donc surtout pour les requêtes qui exécutent Opus 5. Le [guide de prompting d'Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5#controlling-subagent-spawning) propose une instruction de délégation que vous pouvez ajouter à n'importe quel prompt. Le fait que Claude Code ajoute sa propre instruction dépend du [prompt système](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts#how-system-prompts-work) que vous utilisez :

- **Preset `claude_code`** : quand le modèle est Opus 5, Claude Code ajoute à son prompt système une ligne demandant à Claude de ne pas appeler l'outil Agent sauf si on le lui demande. L'outil Agent reste disponible.
- **Un prompt personnalisé, ou pas de `systemPrompt`** : Claude Code ne construit pas son prompt système, donc cette ligne est absente. Ajoutez l'instruction de délégation du guide de prompting à votre propre prompt.

Dans les deux cas, l'instruction ne fait qu'orienter Claude : définissez donc aussi les limites. Claude Code les applique quelle que soit la façon dont Claude décide de déléguer.

## Passer à l'échelle avec les workflows dynamiques

Les sous-agents conviennent bien pour quelques tâches déléguées par tour. Pour des exécutions qui coordonnent des dizaines à des centaines d'agents, utilisez l'outil `Workflow`, qui déplace l'orchestration dans un script que le runtime exécute en dehors du contexte de la conversation. Voir [workflows dynamiques](https://code.claude.com/docs/en/workflows) pour la différence entre les workflows et la délégation tour par tour à des sous-agents.

L'outil `Workflow` est disponible dans le TypeScript Agent SDK v0.3.149+. Ajoutez `Workflow` à `allowedTools` pour approuver automatiquement les exécutions de workflows. Les schémas d'entrée et de sortie de l'outil figurent dans la [référence TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript#workflow).

## Dépannage

### Claude ne délègue pas aux sous-agents

Si Claude accomplit les tâches directement au lieu de les déléguer à votre sous-agent :

- **Utilisez un prompt explicite** : mentionnez le sous-agent par son nom dans votre prompt, par exemple « Use the code-reviewer agent to check the authentication module ».
- **Rédigez une description claire** : expliquez précisément quand utiliser le sous-agent pour que Claude puisse associer les tâches correctement.

### Les agents définis par fichiers ne se chargent pas

Claude Code surveille `~/.claude/agents/` et `.claude/agents/` et prend en compte un fichier d'agent nouveau ou modifié en quelques secondes, sans redémarrage. Si une définition n'apparaît jamais, examinez ces causes :

- **Nouveau répertoire `agents`** : la surveillance ne couvre que les répertoires qui existaient au démarrage de la session ; le premier fichier d'un nouveau répertoire nécessite donc un redémarrage de session. C'est la cause la plus fréquente.
- **Frontmatter invalide ou `name` en double** : vérifiez le YAML du fichier et si un agent existant utilise déjà ce `name`.
- **`--disable-slash-commands`** : les sessions démarrées avec ce flag ne surveillent pas ces répertoires et nécessitent toujours un redémarrage pour charger de nouveaux fichiers.
- **Un fichier dans un répertoire ajouté** : Claude Code charge `.claude/agents/` depuis les répertoires ajoutés avec l'option `add_dirs` (Python) ou `additionalDirectories` (TypeScript), ou avec `--add-dir` ou `/add-dir` en CLI, mais ne les surveille pas : un fichier nouveau ou modifié à cet endroit nécessite un redémarrage de session.
- **Un agent programmatique du même nom** : les `agents` passés à `query()` remplacent un agent défini par fichier portant le même nom.

Pour le format de fichier, voir [comment écrire des fichiers de sous-agents](https://code.claude.com/docs/en/sub-agents#write-subagent-files).

## Documentation associée

- [Sous-agents de Claude Code](https://code.claude.com/docs/en/sub-agents) : documentation des sous-agents, y compris les définitions par fichiers.
- [Workflows dynamiques](https://code.claude.com/docs/en/workflows) : orchestrer de nombreux sous-agents depuis un script pour des tâches trop volumineuses pour une seule conversation.
- [Vue d'ensemble du SDK](https://code.claude.com/docs/en/agent-sdk/overview) : bien démarrer avec le Claude Agent SDK.
