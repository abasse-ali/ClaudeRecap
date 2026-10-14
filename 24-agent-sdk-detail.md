# 24 — Agent SDK en détail

[← Sommaire](README.md) · Introduction : [17 — SDK, entreprise, coûts](17-sdk-entreprise-couts.md)

L'**Claude Agent SDK** (anciennement Claude Code SDK) permet de construire vos propres agents avec le moteur de Claude Code, en **Python** (`pip install claude-agent-sdk`) ou **TypeScript** (`npm install @anthropic-ai/claude-agent-sdk`). Le SDK lance le binaire Claude Code en sous-processus et vous renvoie un flux de messages.

## Ce qu'on peut faire avec le SDK

| Capacité | Description |
|----------|-------------|
| Outils intégrés | Lire, écrire, modifier des fichiers, lancer des commandes, chercher sur le web (les mêmes que Claude Code) |
| Outils personnalisés | Vos fonctions exposées comme outils via un serveur MCP **en processus** (`tool()` / `@tool`, `createSdkMcpServer()` / `create_sdk_mcp_server()`) |
| MCP | Connecter des serveurs MCP externes (`mcpServers`) |
| Recherche d'outils | Charger les outils à la demande quand il y en a beaucoup |
| Sous-agents | Définir des agents spécialisés (`agents`) |
| Hooks | Fonctions de rappel sur les événements (PreToolUse, PostToolUse, Stop…) |
| Permissions | Règles allow/deny, modes, rappel `canUseTool` pour approuver ou poser des questions à l'utilisateur |
| Sessions | Continuer, reprendre, forker une conversation ; stockage externe des sessions |
| Skills, commandes, mémoire | Chargés depuis `.claude/` et `~/.claude/` comme dans Claude Code (`settingSources`) |
| Plugins | Charger des plugins par chemin local |
| Sorties structurées | JSON validé par un schéma (Zod, Pydantic, JSON Schema) |
| Streaming | Messages partiels en temps réel, entrées en flux (conversation continue) |
| Points de contrôle | Revenir à l'état des fichiers d'un message donné (`rewindFiles`) |
| Coûts | Usage par étape et par modèle, coût total, budget max |
| Observabilité | OpenTelemetry |
| Suivi des tâches | La checklist de l'agent |
| Prompts système | Préréglage `claude_code` + ajouts, styles de sortie, ou prompt entièrement personnalisé |

## Exemple minimal

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Trouve et corrige le bug dans auth.ts",
  options: { allowedTools: ["Read", "Edit", "Bash"], permissionMode: "acceptEdits", maxTurns: 20 },
})) {
  if (message.type === "result") console.log(message.result, message.total_cost_usd);
}
```

## La boucle d'agent et les messages

Chaque tour : Claude produit du texte et des appels d'outils → le SDK exécute les outils → les résultats repartent vers Claude, jusqu'à la réponse finale.

| Message | Contenu |
|---------|---------|
| `SystemMessage` | Cycle de vie : `init` (métadonnées : outils, modèle, serveurs MCP…), `compact_boundary`, `informational`, `worker_shutting_down` |
| `AssistantMessage` | Un bloc de contenu de Claude (texte, appel d'outil, réflexion) |
| `UserMessage` | Résultat d'outil renvoyé à Claude, ou message que vous injectez |
| `StreamEvent` | Événements bruts de streaming (avec `includePartialMessages`) |
| `ResultMessage` | Fin de la boucle : résultat final, usage des jetons, coût, ID de session, `subtype` (succès ou limite atteinte) |

TypeScript a aussi : messages de hooks, de compaction, de permission refusée, d'installation de plugin, d'usage du contexte, etc.

## Options principales (`Options` / `ClaudeAgentOptions`)

| Catégorie | Options |
|-----------|---------|
| Outils | `tools`, `allowedTools`, `disallowedTools`, `mcpServers`, `strictMcpConfig` |
| Prompt | `systemPrompt` (préréglage `claude_code` + `append`, texte, ou fichier), `outputFormat` (sortie structurée) |
| Permissions | `permissionMode`, `canUseTool`, `permissionPromptToolName`, `sandbox` |
| Sessions | `continue`, `resume`, `sessionId`, `forkSession`, `resumeSessionAt`, `sessionStore` |
| Limites | `maxTurns`, `maxBudgetUsd`, `taskBudget` |
| Modèle | `model`, `fallbackModel`, `effort`, `thinking`, `maxThinkingTokens`, `betas` |
| Environnement | `cwd`, `addDirs`, `env`, `settings`, `settingSources` (`user`, `project`, `local`), `cliPath`, `extraArgs` |
| Extensions | `agents`, `plugins`, `skills`, `hooks` |
| Flux | `includePartialMessages`, `includeHookEvents`, `forwardSubagentText` |
| Divers | `enableFileCheckpointing`, `stderr`, `debugStderr`, `user`, `loadTimeoutMs` |

## Fonctions et méthodes

**Fonctions** : `query()`, `tool()`, `createSdkMcpServer()`, `listSessions()`, `getSessionMessages()`, `getSessionInfo()`, `renameSession()`, `tagSession()`, `forkSession()`, `resolveSettings()`, `startup()` / `prewarm()` (TypeScript : démarrer le processus à l'avance). Python : classe `ClaudeSDKClient` pour une conversation multi-tours dans un même processus.

**Méthodes de l'objet `Query`** (TypeScript, mode d'entrée en flux) : `interrupt()`, `rewindFiles()`, `setPermissionMode()`, `setModel()`, `applyFlagSettings()`, `updateSettings()`, `initializationResult()`, `reinitialize()`, `supportedCommands()`, `supportedModels()`, `supportedAgents()`, `mcpServerStatus()`, `getContextUsage()`, `readFile()`, `reloadPlugins()`, `reloadSkills()`, `reloadOutputStyles()`, `accountInfo()`, `reconnectMcpServer()`, `toggleMcpServer()`, `setMcpServers()`, `readMcpResource()`, `streamInput()`, `stopTask()`, `close()`.

## Approbations et questions (`canUseTool`)

Le rappel `canUseTool(toolName, input)` est appelé quand un outil n'est pas pré-approuvé : renvoyez `allow` (éventuellement avec une entrée modifiée) ou `deny` (avec un message). Quand Claude pose une question (`AskUserQuestion`), le même rappel reçoit les questions et vous renvoyez les réponses choisies.

## Sessions

- **Continuer** (`continue: true` / `continue_conversation=True`), **reprendre** par ID (`resume`), **forker** (`forkSession`).
- L'ID de session se lit dans le `ResultMessage`.
- **Stockage externe** : interface `SessionStore` (`append`, `load`, et facultatifs `listSessions`, `delete`…) ; `InMemorySessionStore` pour les tests ; adaptateurs de référence et suite de conformité fournis.

## Entrée en flux vs message unique

Le mode **flux** (recommandé) permet une session persistante : envoyer plusieurs messages, des images, interrompre, changer de mode ou de modèle en cours de route. Le mode **message unique** suffit pour une tâche ponctuelle sans interaction.

## Hooks dans le SDK

Deux façons, utilisables ensemble : les hooks des fichiers de réglages (commandes shell) et les **hooks en fonctions** dans `options.hooks` (`{ PreToolUse: [{ matcher: "Bash", hooks: [maFonction] }] }`). Ils peuvent modifier l'entrée d'un outil, ajouter du contexte, bloquer un outil. TypeScript gère des événements supplémentaires (SessionStart, SessionEnd, TeammateIdle, TaskCompleted…).

## Hébergement et sécurité

- **Modèle** : un sous-processus Claude Code par session ; l'état vit sur le disque local.
- **Schémas de session** : éphémères (une tâche puis on détruit), longues (agent permanent), hybrides, conteneur multi-agents.
- **Isolation** : sandbox runtime, conteneurs, gVisor, machines virtuelles ; principes de moindre privilège et de défense en profondeur ; contrôle du réseau sortant ; gestion des identifiants.

## Coûts

Usage par étape, par modèle, cumul sur plusieurs appels, coût en cas d'échec, reprise des totaux après crash. Les coûts affichés sont des **estimations** côté client.

## Migration

Depuis l'ancien « Claude Code SDK » : renommage du paquet, `ClaudeCodeOptions` → `ClaudeAgentOptions` (Python), le prompt système de Claude Code n'est plus appliqué par défaut (utiliser le préréglage `claude_code`), les fichiers de réglages ne sont plus chargés par défaut (utiliser `settingSources`).

## Variables utiles au SDK

`CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS`, `CLAUDE_AGENT_SDK_MCP_NO_PREFIX`, `CLAUDE_CODE_FORK_SUBAGENT`, `CLAUDE_CODE_RESUME_INTERRUPTED_TURN`, `CLAUDE_CODE_EXIT_AFTER_STOP_DELAY`, `CLAUDE_CODE_EMIT_SESSION_STATE_EVENTS`, `CLAUDE_CODE_SYNC_SKILLS`, `CLAUDE_CODE_SYNC_PLUGIN_INSTALL` (voir [20](20-reference-variables-environnement.md)).
