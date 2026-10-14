# Utiliser les fonctionnalités de Claude Code dans le SDK

> Source : https://code.claude.com/docs/en/agent-sdk/claude-code-features · [← Index](../31-index-documentation-officielle.md)

*Charger dans vos agents SDK les instructions de projet, les skills, les hooks et les autres fonctionnalités de Claude Code.*

L'Agent SDK repose sur les mêmes fondations que Claude Code : vos agents SDK ont donc accès aux mêmes fonctionnalités basées sur le système de fichiers : instructions de projet (`CLAUDE.md` et règles), skills, hooks, etc.

Quand vous omettez `settingSources`, `query()` lit les mêmes réglages du système de fichiers que la CLI Claude Code : réglages utilisateur, projet et local, fichiers CLAUDE.md, ainsi que les skills, agents et commandes de `.claude/`. Pour s'en passer, passez `settingSources: []`, ce qui limite l'agent à ce que vous configurez par programme. Les réglages de politique gérée (managed policy) et la configuration globale `~/.claude.json` sont lus **quelle que soit** cette option. Voir [Ce que settingSources ne contrôle pas](#ce-que-settingsources-ne-contrôle-pas).

## Contrôler les réglages du système de fichiers avec settingSources

L'option des sources de réglages ([`setting_sources`](https://code.claude.com/docs/en/agent-sdk/python#claudeagentoptions) en Python, [`settingSources`](https://code.claude.com/docs/en/agent-sdk/typescript#settingsource) en TypeScript) contrôle quels réglages issus du système de fichiers le SDK charge. Passez une liste explicite pour activer certaines sources, ou un tableau vide pour désactiver les réglages utilisateur, projet et local.

Cet exemple charge les réglages de niveau utilisateur et projet en fixant `settingSources` à `["user", "project"]` :

### Python

```python
from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage, ResultMessage
import asyncio


async def main():
    async for message in query(
        prompt="Help me refactor the auth module",
        options=ClaudeAgentOptions(
            # "user" charge depuis ~/.claude/, "project" charge depuis ./.claude/ dans le cwd.
            # Ensemble, ils donnent à l'agent accès aux CLAUDE.md, skills, hooks et
            # permissions des deux emplacements.
            setting_sources=["user", "project"],
            allowed_tools=["Read", "Edit", "Bash"],
        ),
    ):
        if isinstance(message, AssistantMessage):
            for block in message.content:
                if hasattr(block, "text"):
                    print(block.text)
        if isinstance(message, ResultMessage) and message.subtype == "success":
            print(f"\nResult: {message.result}")


asyncio.run(main())
```

### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Help me refactor the auth module",
  options: {
    // "user" charge depuis ~/.claude/, "project" charge depuis ./.claude/ dans le cwd.
    // Ensemble, ils donnent à l'agent accès aux CLAUDE.md, skills, hooks et
    // permissions des deux emplacements.
    settingSources: ["user", "project"],
    allowedTools: ["Read", "Edit", "Bash"]
  }
})) {
  if (message.type === "assistant") {
    for (const block of message.message.content) {
      if (block.type === "text") console.log(block.text);
    }
  }
  if (message.type === "result" && message.subtype === "success") {
    console.log(`\nResult: ${message.result}`);
  }
}
```

À l'exécution, la réponse de l'assistant s'affiche sur stdout, suivie d'une ligne de résultat final une fois l'exécution terminée.

Chaque source charge des réglages depuis un emplacement précis, où `<cwd>` est le répertoire de travail passé via l'option `cwd`, ou le répertoire courant du processus s'il n'est pas défini. Pour la définition complète du type, voir [`SettingSource`](https://code.claude.com/docs/en/agent-sdk/typescript#settingsource) (TypeScript) ou [`SettingSource`](https://code.claude.com/docs/en/agent-sdk/python#settingsource) (Python).

| Source | Ce qu'elle charge | Emplacement |
| :- | :- | :- |
| `"project"` | `settings.json` et hooks du projet ; CLAUDE.md du projet et `.claude/rules/*.md` ; skills, commandes et sous-agents du projet | `<cwd>/.claude/` pour `settings.json` et les hooks ; `<cwd>` et chaque répertoire parent pour CLAUDE.md et les règles ; `<cwd>` et chaque répertoire parent jusqu'à la racine du dépôt pour les skills, commandes et sous-agents, plus les dossiers `.claude/skills/`, `.claude/commands/` et `.claude/agents/` de chaque répertoire passé via l'option `additionalDirectories` ou `add_dirs`, que le SDK transmet à Claude Code sous forme de [`--add-dir`](https://code.claude.com/docs/en/permissions#additional-directories-grant-file-access-not-configuration) |
| `"user"` | `settings.json` utilisateur ; CLAUDE.md utilisateur et `~/.claude/rules/*.md` ; skills, commandes et sous-agents utilisateur | `~/.claude/` pour `settings.json`, CLAUDE.md et les règles ; `~/.claude/skills/`, `~/.claude/commands/` et `~/.claude/agents/` pour les skills, commandes et sous-agents |
| `"local"` | CLAUDE.local.md, `.claude/settings.local.json` | `<cwd>/.claude/` pour `settings.local.json` ; `<cwd>` et chaque répertoire parent pour CLAUDE.local.md |

Omettre `settingSources` équivaut à `["user", "project", "local"]`.

L'option `cwd` détermine où le SDK cherche les éléments de niveau projet. Le `settings.json` et les hooks du projet ne se chargent que depuis `<cwd>/.claude/`, **sans** repli sur les répertoires parents.

### Ce que settingSources ne contrôle pas

`settingSources` couvre les réglages utilisateur, projet et local. Quelques éléments sont lus quelle que soit sa valeur :

| Élément | Comportement | Pour le désactiver |
| :- | :- | :- |
| Réglages de politique gérée | La politique gérée au niveau du poste (plist MDM, stratégie du registre ou fichier de réglages gérés) est chargée depuis l'hôte. Les [réglages gérés côté serveur](https://code.claude.com/docs/en/server-managed-settings) sont récupérés sur une [configuration éligible](https://code.claude.com/docs/en/server-managed-settings#platform-availability) lorsque la session s'authentifie avec un identifiant qualifiant : connexion OAuth d'organisation, clé API configurée directement, ou [profil Anthropic](https://code.claude.com/docs/en/authentication#anthropic-profiles-and-federation-credentials) `user_oauth` | Politique du poste : supprimer de l'hôte le fichier de réglages gérés, le plist ou la stratégie du registre. Réglages gérés côté serveur : c'est un [Owner](https://code.claude.com/docs/en/server-managed-settings#access-control) de votre organisation Claude qui les contrôle ; impossible de les désactiver depuis le SDK |
| Configuration globale `~/.claude.json` | Toujours lue | La déplacer avec `CLAUDE_CONFIG_DIR` dans `env` |
| Mémoire automatique dans `~/.claude/projects/<project>/memory/` | Chargée dans le prompt système au démarrage de la session. L'agent y écrit de nouveaux souvenirs avec les outils standard `Write` et `Edit` (pas d'outil mémoire dédié) : ces outils doivent donc être activés pour que l'agent puisse enregistrer des souvenirs | Mettre `autoMemoryEnabled: false` dans les réglages, ou `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` dans `env` |
| [Connecteurs MCP claude.ai](https://code.claude.com/docs/en/mcp#use-mcp-servers-from-claude-ai) | Chargés quand la session s'authentifie avec votre connexion claude.ai. Non chargés quand `CLAUDE_CODE_OAUTH_TOKEN` contient un jeton issu de [`claude setup-token`](https://code.claude.com/docs/en/authentication#generate-a-long-lived-token), qui ne peut faire que des requêtes au modèle. Passer `mcpServers: {}` ne supprime **pas** les connecteurs | Mettre `strictMcpConfig: true`, [`disableClaudeAiConnectors: true`](https://code.claude.com/docs/en/mcp#disable-claude-ai-connectors) dans les réglages, ou `ENABLE_CLAUDEAI_MCP_SERVERS=false` dans `env` |
| Entrées `deny` de [`sandbox.credentials`](https://code.claude.com/docs/en/sandboxing#protect-credentials) et entrées de fichiers `mask` dans `~/.claude/settings.json` | Quand le [sandbox de commandes](https://code.claude.com/docs/en/sandboxing) s'exécute, Claude Code applique les entrées `deny` et conserve les entrées `mask` de `credentials.files` comme restrictions, même si `settingSources` exclut les réglages utilisateur. Claude Code n'utilise ces entrées que pour restreindre ce à quoi les commandes en sandbox peuvent accéder | Supprimer les entrées de `~/.claude/settings.json` |

> **Avertissement** : ne comptez pas sur les options par défaut de `query()` pour l'isolation multi-locataire (multi-tenant). Comme les éléments ci-dessus sont lus quel que soit `settingSources`, un processus SDK peut récupérer de la configuration au niveau de l'hôte et de la mémoire propre à un répertoire. Pour les déploiements multi-locataires, exécutez chaque locataire dans son propre système de fichiers et définissez `settingSources: []` ainsi que `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` dans `env`. Les [réglages gérés côté serveur](https://code.claude.com/docs/en/server-managed-settings) sont récupérés lorsque le processus s'authentifie avec un identifiant d'organisation ; l'isolation du système de fichiers ne les supprime pas. Voir [Déploiement sécurisé](https://code.claude.com/docs/en/agent-sdk/secure-deployment).

## Instructions de projet (CLAUDE.md et règles)

Les fichiers `CLAUDE.md` et `.claude/rules/*.md` donnent à l'agent un contexte persistant sur votre projet : conventions de code, commandes de build, décisions d'architecture et instructions. Quand `settingSources` inclut `"project"`, comme dans l'[exemple settingSources](#contrôler-les-réglages-du-système-de-fichiers-avec-settingsources), le SDK charge ces fichiers dans le contexte au démarrage de la session. L'agent suit alors les conventions du projet sans que vous les répétiez dans chaque prompt.

### Emplacements de chargement de CLAUDE.md

| Niveau | Emplacement | Quand il est chargé |
| :- | :- | :- |
| Projet (racine) | `<cwd>/CLAUDE.md` ou `<cwd>/.claude/CLAUDE.md` | `settingSources` inclut `"project"` |
| Règles du projet | `<cwd>/.claude/rules/*.md` et `.claude/rules/*.md` dans chaque répertoire parent | `settingSources` inclut `"project"` |
| Projet (répertoires parents) | Fichiers `CLAUDE.md` dans les répertoires au-dessus de `cwd` | `settingSources` inclut `"project"`, chargés au démarrage de la session |
| Projet (sous-répertoires) | Fichiers `CLAUDE.md` dans les sous-répertoires de `cwd` | `settingSources` inclut `"project"`, chargés [à la demande](https://code.claude.com/docs/en/memory#how-claude-md-files-load) |
| Local | `<cwd>/CLAUDE.local.md` et `CLAUDE.local.md` dans chaque répertoire parent | `settingSources` inclut `"local"` |
| Utilisateur | `~/.claude/CLAUDE.md` | `settingSources` inclut `"user"` |
| Règles utilisateur | `~/.claude/rules/*.md` | `settingSources` inclut `"user"` |

Tous les niveaux sont cumulatifs : si des CLAUDE.md projet et utilisateur existent, l'agent voit les deux. Il n'existe pas de règle de priorité stricte entre niveaux ; en cas d'instructions contradictoires, le résultat dépend de l'interprétation de Claude. Écrivez des règles non contradictoires, ou indiquez explicitement la priorité dans le fichier le plus spécifique (« Ces instructions de projet remplacent toute valeur par défaut contradictoire de niveau utilisateur »).

> **Astuce** : vous pouvez aussi injecter du contexte directement via `systemPrompt`, sans fichiers CLAUDE.md. Voir [Modifier les prompts système](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts). Utilisez CLAUDE.md quand vous voulez que le même contexte soit partagé entre les sessions interactives de Claude Code et vos agents SDK.

Pour structurer et organiser le contenu de CLAUDE.md, voir [Gérer la mémoire de Claude](https://code.claude.com/docs/en/memory).

## Skills

Les skills sont des fichiers Markdown qui donnent à l'agent des connaissances spécialisées et des workflows invocables. Contrairement à `CLAUDE.md` (chargé à chaque session), les skills se chargent à la demande : l'agent reçoit les descriptions des skills au démarrage et charge le contenu complet quand c'est pertinent.

Les skills sont découvertes depuis le système de fichiers via `settingSources`. Quand l'option `skills` de `query()` est omise, les skills utilisateur et projet découvertes sont activées et l'outil Skill est disponible, comme dans la CLI. Pour contrôler quelles skills sont activées, passez `skills` avec la valeur `"all"`, une liste de noms de skills, ou `[]` pour toutes les désactiver. Quand `skills` est défini, le SDK ajoute automatiquement l'outil Skill à `allowedTools`. Si vous passez aussi une liste `tools` explicite, incluez-y `"Skill"` pour que Claude puisse invoquer les skills.

### Python

```python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage
import asyncio


# Les skills dans .claude/skills/ sont découvertes automatiquement
# quand settingSources inclut "project"
async def main():
    async for message in query(
        prompt="Review this PR using our code review checklist",
        options=ClaudeAgentOptions(
            setting_sources=["user", "project"],
            skills="all",
            allowed_tools=["Read", "Grep", "Glob"],
        ),
    ):
        if isinstance(message, ResultMessage) and message.subtype == "success":
            print(message.result)


asyncio.run(main())
```

### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

// Les skills dans .claude/skills/ sont découvertes automatiquement
// quand settingSources inclut "project"
for await (const message of query({
  prompt: "Review this PR using our code review checklist",
  options: {
    settingSources: ["user", "project"],
    skills: "all",
    allowedTools: ["Read", "Grep", "Glob"]
  }
})) {
  if (message.type === "result" && message.subtype === "success") {
    console.log(message.result);
  }
}
```

> **Note** : les skills doivent être créées comme artefacts du système de fichiers (`.claude/skills/<name>/SKILL.md`). Le SDK n'a pas d'API programmatique pour enregistrer des skills. Voir [Agent Skills dans le SDK](https://code.claude.com/docs/en/agent-sdk/skills) pour tous les détails.

## Hooks

Le SDK permet de définir des hooks de deux façons, qui fonctionnent côte à côte :

* **Hooks du système de fichiers** : commandes shell définies dans `settings.json`, chargées quand `settingSources` inclut la source correspondante. Ce sont les mêmes hooks que ceux configurés pour les [sessions interactives de Claude Code](https://code.claude.com/docs/en/hooks-guide).
* **Hooks programmatiques** : fonctions de rappel (callbacks) passées directement à `query()`. Elles s'exécutent dans le processus de votre application et peuvent renvoyer des décisions structurées. Voir [Contrôler l'exécution avec les hooks](https://code.claude.com/docs/en/agent-sdk/hooks).

Les callbacks de hook reçoivent l'entrée de l'outil et renvoient un dictionnaire de décision. Renvoyer `{}` signifie autoriser l'outil à continuer. Pour bloquer l'exécution, renvoyez un objet `hookSpecificOutput` avec `permissionDecision: "deny"` et un `permissionDecisionReason`. La raison est envoyée à Claude comme résultat de l'outil. Voir le [guide des hooks](https://code.claude.com/docs/en/agent-sdk/hooks) pour la signature complète du callback et les types de retour.

### Python

```python
from claude_agent_sdk import query, ClaudeAgentOptions, HookMatcher, ResultMessage
import asyncio


# Callback de hook PreToolUse. Arguments positionnels :
#   input_data : dict HookInput avec tool_name, tool_input, hook_event_name
#   tool_use_id : str | None, l'ID de l'appel d'outil intercepté
#   context : HookContext, réservé à une future prise en charge du signal d'abandon
async def audit_bash(input_data, tool_use_id, context):
    command = input_data.get("tool_input", {}).get("command", "")
    if "rm -rf" in command:
        return {
            "hookSpecificOutput": {
                "hookEventName": "PreToolUse",
                "permissionDecision": "deny",
                "permissionDecisionReason": "Destructive command blocked",
            }
        }
    return {}  # Dict vide : autoriser l'outil à continuer


# Les hooks du système de fichiers de .claude/settings.json s'exécutent automatiquement
# quand settingSources les charge. Vous pouvez aussi ajouter des hooks programmatiques :
async def main():
    async for message in query(
        prompt="Refactor the auth module",
        options=ClaudeAgentOptions(
            setting_sources=["project"],  # Charge les hooks de .claude/settings.json
            hooks={
                "PreToolUse": [
                    HookMatcher(matcher="Bash", hooks=[audit_bash]),
                ]
            },
        ),
    ):
        if isinstance(message, ResultMessage) and message.subtype == "success":
            print(message.result)


asyncio.run(main())
```

### TypeScript

```typescript
import { query, type HookInput, type HookJSONOutput } from "@anthropic-ai/claude-agent-sdk";

// Callback de hook PreToolUse. HookInput est une union discriminée sur
// hook_event_name ; restreindre sur ce champ donne à TypeScript la bonne
// forme de tool_input pour cet événement.
const auditBash = async (input: HookInput): Promise<HookJSONOutput> => {
  if (input.hook_event_name !== "PreToolUse") return {};
  const toolInput = input.tool_input as { command?: string };
  if (toolInput.command?.includes("rm -rf")) {
    return {
      hookSpecificOutput: {
        hookEventName: "PreToolUse",
        permissionDecision: "deny",
        permissionDecisionReason: "Destructive command blocked",
      },
    };
  }
  return {}; // Objet vide : autoriser l'outil à continuer
};

// Les hooks du système de fichiers de .claude/settings.json s'exécutent automatiquement
// quand settingSources les charge. Vous pouvez aussi ajouter des hooks programmatiques :
for await (const message of query({
  prompt: "Refactor the auth module",
  options: {
    settingSources: ["project"], // Charge les hooks de .claude/settings.json
    hooks: {
      PreToolUse: [{ matcher: "Bash", hooks: [auditBash] }]
    }
  }
})) {
  if (message.type === "result" && message.subtype === "success") {
    console.log(message.result);
  }
}
```

### Quel type de hook utiliser

| Type de hook | Idéal pour |
| :- | :- |
| **Système de fichiers** (`settings.json`) | Partager des hooks entre sessions CLI et SDK. Prend en charge `"command"` (scripts shell), `"http"` (POST vers un endpoint), `"mcp_tool"` (appel d'un outil d'un serveur MCP connecté), `"prompt"` (un LLM évalue un prompt) et `"agent"` (lance un agent vérificateur). Ils se déclenchent dans l'agent principal et dans tous les sous-agents qu'il lance. |
| **Programmatique** (callbacks dans `query()`) | Logique propre à l'application, décisions structurées et intégration dans le processus. Ils se déclenchent aussi dans les sous-agents. L'entrée du hook (premier argument du callback) porte les champs `agent_id` et `agent_type` qui identifient l'agent ayant déclenché le hook. |

> **Note** : le SDK TypeScript prend en charge des événements de hook supplémentaires par rapport à Python, notamment `SessionStart`, `SessionEnd`, `TeammateIdle` et `TaskCompleted`. Voir le [guide des hooks](https://code.claude.com/docs/en/agent-sdk/hooks) pour le tableau complet de compatibilité des événements.

Pour tous les détails sur les hooks programmatiques, voir [Contrôler l'exécution avec les hooks](https://code.claude.com/docs/en/agent-sdk/hooks). Pour la syntaxe des hooks du système de fichiers, voir [Hooks](https://code.claude.com/docs/en/hooks).

## Choisir la bonne fonctionnalité

L'Agent SDK offre plusieurs façons d'étendre le comportement de l'agent. Si vous hésitez, ce tableau associe les objectifs courants à la bonne approche.

| Ce que vous voulez faire | Utiliser | Surface du SDK |
| :- | :- | :- |
| Fixer des conventions de projet que l'agent suit toujours | [CLAUDE.md](https://code.claude.com/docs/en/memory) | `settingSources: ["project"]` le charge automatiquement |
| Donner à l'agent du matériel de référence qu'il charge quand c'est pertinent | [Skills](https://code.claude.com/docs/en/agent-sdk/skills) | `settingSources` + option `skills` |
| Exécuter un workflow réutilisable (déploiement, revue, release) | [Skills invocables par l'utilisateur](https://code.claude.com/docs/en/agent-sdk/skills) | `settingSources` + option `skills` |
| Déléguer une sous-tâche isolée à un contexte neuf (recherche, revue) | [Sous-agents](https://code.claude.com/docs/en/agent-sdk/subagents) | Paramètre `agents` + `allowedTools: ["Agent"]` |
| Coordonner plusieurs instances de Claude Code avec des listes de tâches partagées et une messagerie directe entre agents | [Équipes d'agents](https://code.claude.com/docs/en/agent-teams) | Pas configuré directement via les options du SDK. Les équipes d'agents sont une fonctionnalité de la CLI où une session joue le rôle de chef d'équipe et coordonne le travail de coéquipiers indépendants |
| Exécuter une logique déterministe sur les appels d'outils (audit, blocage, transformation) | [Hooks](https://code.claude.com/docs/en/agent-sdk/hooks) | Paramètre `hooks` avec callbacks, ou scripts shell chargés via `settingSources` |
| Donner à Claude un accès outillé et structuré à un service externe | [MCP](https://code.claude.com/docs/en/agent-sdk/mcp) | Paramètre `mcpServers` |

Chaque fonctionnalité activée consomme de la fenêtre de contexte de l'agent. Pour le coût de chaque fonctionnalité et la façon dont elles se superposent, voir [Étendre Claude Code](https://code.claude.com/docs/en/features-overview#understand-context-costs).

## Ressources associées

* [Étendre Claude Code](https://code.claude.com/docs/en/features-overview) : vue d'ensemble conceptuelle de toutes les fonctionnalités d'extension, avec tableaux comparatifs et analyse du coût en contexte
* [Skills dans le SDK](https://code.claude.com/docs/en/agent-sdk/skills) : guide complet pour utiliser les skills par programme
* [Sous-agents](https://code.claude.com/docs/en/agent-sdk/subagents) : définir et invoquer des sous-agents pour des sous-tâches isolées
* [Hooks](https://code.claude.com/docs/en/agent-sdk/hooks) : intercepter et contrôler le comportement de l'agent aux points clés de l'exécution
* [Permissions](https://code.claude.com/docs/en/agent-sdk/permissions) : contrôler l'accès aux outils avec des modes, des règles et des callbacks
* [Prompts système](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts) : injecter du contexte sans fichiers CLAUDE.md
