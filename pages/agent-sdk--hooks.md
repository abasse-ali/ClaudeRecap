# Intercepter et contrôler le comportement de l'agent avec les hooks

> Source : https://code.claude.com/docs/en/agent-sdk/hooks · [← Index](../31-index-documentation-officielle.md)

*Intercepter et personnaliser le comportement de l'agent aux points clés de son exécution grâce aux hooks.*

Les hooks sont des fonctions de rappel (callbacks) qui exécutent votre code en réponse à des événements de l'agent : appel d'un outil, démarrage d'une session, arrêt de l'exécution, etc. Avec les hooks, vous pouvez :

- **Bloquer les opérations dangereuses** avant leur exécution, comme des commandes shell destructrices ou un accès non autorisé à des fichiers.
- **Journaliser et auditer** chaque appel d'outil pour la conformité, le débogage ou l'analytique.
- **Transformer les entrées et sorties** pour assainir des données, injecter des identifiants ou rediriger des chemins de fichiers.
- **Exiger une approbation humaine** pour les actions sensibles, comme des écritures en base de données ou des appels d'API.
- **Suivre le cycle de vie de la session** pour gérer l'état, nettoyer des ressources ou envoyer des notifications.

## Fonctionnement des hooks

1. **Un événement se déclenche.** Quelque chose se produit pendant l'exécution de l'agent et le SDK émet un événement : un outil est sur le point d'être appelé (`PreToolUse`), un outil a renvoyé un résultat (`PostToolUse`), un sous-agent a démarré ou s'est arrêté, l'agent est inactif, ou l'exécution est terminée. Voir la [liste complète des événements](#hooks-disponibles).
2. **Le SDK collecte les hooks enregistrés.** Le SDK recherche les hooks enregistrés pour ce type d'événement. Cela inclut les hooks callback passés dans `options.hooks` **et** les hooks de commande shell provenant des fichiers de réglages lorsque l'entrée correspondante de [`settingSources`](https://code.claude.com/docs/en/agent-sdk/typescript#settingsource) ou [`setting_sources`](https://code.claude.com/docs/en/agent-sdk/python#settingsource) est activée — ce qui est le cas avec les options par défaut de `query()`.
3. **Les matchers filtrent les hooks à exécuter.** Si un hook possède un motif [`matcher`](#matchers) (comme `"Write|Edit"`), le SDK le teste contre la cible de l'événement (par exemple le nom de l'outil). Les hooks sans matcher s'exécutent pour chaque événement de ce type.
4. **Les fonctions callback s'exécutent.** Le [callback](#fonctions-callback) de chaque hook correspondant reçoit des informations sur ce qui se passe : nom de l'outil, ses arguments, ID de session et autres détails propres à l'événement.
5. **Votre callback renvoie une décision.** Après avoir effectué ses opérations (journalisation, appels d'API, validation), votre callback renvoie un [objet de sortie](#sorties) qui indique à l'agent quoi faire : autoriser l'opération, la bloquer, modifier l'entrée ou injecter du contexte dans la conversation.

L'exemple suivant assemble ces étapes. Il enregistre un hook `PreToolUse` (étape 1) avec un matcher `"Write|Edit"` (étape 3) pour que le callback ne se déclenche que pour les outils d'écriture de fichiers. Une fois déclenché, le callback reçoit l'entrée de l'outil (étape 4), vérifie si le chemin cible un fichier `.env` et renvoie `permissionDecision: "deny"` pour bloquer l'opération (étape 5) :

### Python

```python
import asyncio
from claude_agent_sdk import (
    AssistantMessage,
    ClaudeSDKClient,
    ClaudeAgentOptions,
    HookMatcher,
    ResultMessage,
)


# Définir un callback de hook qui reçoit les détails de l'appel d'outil
async def protect_env_files(input_data, tool_use_id, context):
    # Extraire le chemin du fichier depuis les arguments d'entrée de l'outil
    file_path = input_data["tool_input"].get("file_path", "")
    file_name = file_path.split("/")[-1]

    # Bloquer l'opération si elle cible un fichier .env
    if file_name == ".env":
        return {
            "hookSpecificOutput": {
                "hookEventName": input_data["hook_event_name"],
                "permissionDecision": "deny",
                "permissionDecisionReason": "Cannot modify .env files",
            }
        }

    # Renvoyer un objet vide pour autoriser l'opération
    return {}


async def main():
    options = ClaudeAgentOptions(
        hooks={
            # Enregistrer le hook pour les événements PreToolUse
            # Le matcher filtre pour ne garder que les appels Write et Edit
            "PreToolUse": [HookMatcher(matcher="Write|Edit", hooks=[protect_env_files])]
        }
    )

    async with ClaudeSDKClient(options=options) as client:
        await client.query("Create a .env file with the standard local development database configuration")
        async for message in client.receive_response():
            # Filtrer les messages assistant et résultat
            if isinstance(message, (AssistantMessage, ResultMessage)):
                print(message)


asyncio.run(main())
```

### TypeScript

```typescript
import { query, HookCallback, PreToolUseHookInput } from "@anthropic-ai/claude-agent-sdk";

// Définir un callback de hook avec le type HookCallback
const protectEnvFiles: HookCallback = async (input, toolUseID, { signal }) => {
  // Convertir l'entrée vers le type de hook spécifique pour la sûreté de typage
  const preInput = input as PreToolUseHookInput;

  // Convertir tool_input pour accéder à ses propriétés (typé unknown dans le SDK)
  const toolInput = preInput.tool_input as Record<string, unknown>;
  const filePath = toolInput?.file_path as string;
  const fileName = filePath?.split("/").pop();

  // Bloquer l'opération si elle cible un fichier .env
  if (fileName === ".env") {
    return {
      hookSpecificOutput: {
        hookEventName: preInput.hook_event_name,
        permissionDecision: "deny",
        permissionDecisionReason: "Cannot modify .env files"
      }
    };
  }

  // Renvoyer un objet vide pour autoriser l'opération
  return {};
};

for await (const message of query({
  prompt: "Create a .env file with the standard local development database configuration",
  options: {
    hooks: {
      // Enregistrer le hook pour les événements PreToolUse
      // Le matcher filtre pour ne garder que les appels Write et Edit
      PreToolUse: [{ matcher: "Write|Edit", hooks: [protectEnvFiles] }]
    }
  }
})) {
  // Filtrer les messages assistant et résultat
  if (message.type === "assistant" || message.type === "result") {
    console.log(message);
  }
}
```

Quel que soit le script lancé, Claude tente de créer le fichier `.env` et le hook refuse l'appel d'outil.

## Hooks disponibles

Le SDK fournit des hooks pour les différentes étapes de l'exécution de l'agent. Certains sont disponibles dans les deux SDK, d'autres uniquement en TypeScript.

| Événement de hook | SDK Python | SDK TypeScript | Déclencheur | Exemple de cas d'usage |
| - | - | - | - | - |
| `PreToolUse` | Oui | Oui | Demande d'appel d'outil (peut bloquer ou modifier) | Bloquer les commandes shell dangereuses |
| `PostToolUse` | Oui | Oui | Résultat de l'exécution d'un outil | Consigner toutes les modifications de fichiers dans une piste d'audit |
| `PostToolUseFailure` | Oui | Oui | Échec de l'exécution d'un outil | Gérer ou journaliser les erreurs d'outils |
| `PostToolBatch` | Non | Oui | Un lot complet d'appels d'outils est résolu, une fois par lot avant l'appel suivant au modèle | Injecter des conventions une seule fois pour tout le lot |
| [`UserPromptSubmit`](https://code.claude.com/docs/en/hooks#userpromptsubmit) | Oui | Oui | Un prompt est soumis, y compris un tour que Claude Code démarre de lui-même | Injecter du contexte supplémentaire dans les prompts |
| [`UserPromptExpansion`](https://code.claude.com/docs/en/hooks#userpromptexpansion) | Non | Oui | Une commande saisie par l'utilisateur, ou un prompt MCP, est développée en prompt avant d'atteindre Claude. Ne se déclenche pas quand Claude invoque lui-même un skill | Bloquer l'invocation directe d'une commande ou ajouter du contexte quand un skill est saisi |
| `MessageDisplay` | Non | Oui | Un message assistant contenant du texte se termine, une fois par message avec le texte complet | Masquer ou reformater le texte affiché sans modifier le transcript |
| `Stop` | Oui | Oui | Arrêt de l'exécution de l'agent | Sauvegarder l'état de la session avant de quitter |
| `StopFailure` | Non | Oui | Le tour se termine par une erreur d'API au lieu d'un arrêt normal | Journaliser les échecs ou envoyer des alertes |
| `SubagentStart` | Oui | Oui | Initialisation d'un sous-agent | Suivre le lancement de tâches parallèles |
| `SubagentStop` | Oui | Oui | Fin d'un sous-agent | Agréger les résultats des tâches parallèles |
| `PreCompact` | Oui | Oui | Demande de compaction de la conversation | Archiver le transcript complet avant le résumé |
| `PostCompact` | Non | Oui | La compaction de la conversation est terminée | Journaliser le résumé généré |
| [`PreModelSwitch`](https://code.claude.com/docs/en/hooks#premodelswitch) | Non | Oui | Un changement de modèle demandé, avant qu'il ait lieu (peut bloquer) | Bloquer le passage à un modèle précis |
| [`PostModelSwitch`](https://code.claude.com/docs/en/hooks#postmodelswitch) | Non | Oui | Le modèle de la session change, y compris lors d'un repli (fallback) automatique | Donner à Claude des consignes propres au nouveau modèle |
| `PermissionRequest` | Oui | Oui | Un appel d'outil nécessite une décision de permission | Gestion personnalisée des permissions |
| `PermissionDenied` | Non | Oui | Le mode auto refuse un appel d'outil, y compris les refus sans verdict du classifieur | Journaliser les refus, ou indiquer au modèle qu'il peut réessayer ; Claude Code ignore `retry: true` pour les refus sans verdict. Voir [PermissionDenied](https://code.claude.com/docs/en/hooks#permissiondenied) |
| `SessionStart` | Non | Oui | Initialisation de la session | Initialiser la journalisation et la télémétrie |
| `SessionEnd` | Non | Oui | Fin de la session | Nettoyer les ressources temporaires |
| `Notification` | Oui | Oui | Messages d'état de l'agent | Envoyer les mises à jour d'état de l'agent vers Slack ou PagerDuty |
| `Setup` | Non | Oui | Mise en place/maintenance de la session | Exécuter des tâches d'initialisation |
| `TeammateIdle` | Non | Oui | Un coéquipier (teammate) devient inactif | Réattribuer du travail ou notifier |
| `TaskCreated` | Non | Oui | Une tâche est créée via l'outil `TaskCreate` | Imposer des conventions de nommage des tâches |
| [`TaskCompleted`](https://code.claude.com/docs/en/hooks#taskcompleted) | Non | Oui | Une tâche est marquée comme terminée | Exiger que les tests passent avant de clore une tâche |
| `Elicitation` | Non | Oui | Un serveur MCP demande une saisie utilisateur en cours de tâche | Répondre par programme aux demandes de saisie MCP |
| `ElicitationResult` | Non | Oui | Un utilisateur répond à une élicitation MCP | Modifier ou bloquer la réponse avant son retour au serveur |
| `ConfigChange` | Non | Oui | Modification d'un fichier de configuration | Recharger les réglages dynamiquement |
| `InstructionsLoaded` | Non | Oui | Un fichier `CLAUDE.md` ou de règles est chargé dans le contexte | Auditer quels fichiers d'instructions sont chargés |
| `WorktreeCreate` | Non | Oui | Création d'un worktree git | Suivre les espaces de travail isolés |
| `WorktreeRemove` | Non | Oui | Un worktree créé par un hook `WorktreeCreate` est en cours de suppression | Nettoyer les ressources de l'espace de travail |
| `CwdChanged` | Non | Oui | Le répertoire de travail change pendant une session | Recharger les variables d'environnement par répertoire |
| `FileChanged` | Non | Oui | Un fichier surveillé est modifié, créé ou supprimé | Recharger la configuration quand les fichiers du projet changent |
| `DirectoryAdded` | Non | Oui | Un répertoire de travail est ajouté pendant une session | Installer les dépendances d'un dépôt ajouté en cours de session |

## Configurer les hooks

Pour configurer un hook, passez-le dans le champ `hooks` des options de l'agent (`ClaudeAgentOptions` en Python, l'objet `options` en TypeScript). Cet extrait suppose que vous avez déjà défini un callback de hook, comme `protect_env_files` (Python) ou `protectEnvFiles` (TypeScript) de l'exemple ci-dessus :

### Python

```python
options = ClaudeAgentOptions(
    hooks={"PreToolUse": [HookMatcher(matcher="Bash", hooks=[my_callback])]}
)

async with ClaudeSDKClient(options=options) as client:
    await client.query("Your prompt")
    async for message in client.receive_response():
        print(message)
```

### TypeScript

```typescript
for await (const message of query({
  prompt: "Your prompt",
  options: {
    hooks: {
      PreToolUse: [{ matcher: "Bash", hooks: [myCallback] }]
    }
  }
})) {
  console.log(message);
}
```

L'option `hooks` est un dictionnaire en Python ou un objet en TypeScript, où :

- **Clés** : les [noms d'événements de hook](#hooks-disponibles), comme `'PreToolUse'`, `'PostToolUse'` et `'Stop'`.
- **Valeurs** : des tableaux de [matchers](#matchers), chacun contenant un motif de filtre facultatif et vos [fonctions callback](#fonctions-callback).

### Matchers

Les matchers permettent de filtrer le moment où vos callbacks se déclenchent. Le champ `matcher` est comparé à une valeur différente selon le type d'événement : par exemple, les hooks liés aux outils comparent le nom de l'outil, tandis que les hooks `Notification` comparent le type de notification.

Les matchers du SDK suivent les mêmes règles que les [matchers des fichiers de réglages](https://code.claude.com/docs/en/hooks#matcher-patterns). Cette section documente les deux chemins d'évaluation (chaîne exacte et expression régulière), leurs exigences de version et les valeurs de matcher pour chaque type d'événement.

| Option | Type | Défaut | Description |
| - | - | - | - |
| `matcher` | `string` | `undefined` | Motif comparé au champ de filtre de l'événement, selon les [règles des matchers des fichiers de réglages](https://code.claude.com/docs/en/hooks#matcher-patterns). Pour les hooks d'outils, c'est le nom de l'outil. Les outils intégrés incluent `Bash`, `Read`, `Write`, `Edit`, `Glob`, `Grep`, `WebFetch`, `Agent` et d'autres (liste complète dans [Tool Input Types](https://code.claude.com/docs/en/agent-sdk/typescript#tool-input-types)). Les outils MCP suivent le motif `mcp__<server>__<action>`, où `<server>` est la clé utilisée dans la configuration `mcpServers`. |
| `hooks` | `HookCallback[]` | - | Obligatoire. Tableau de fonctions callback à exécuter quand le motif correspond |
| `timeout` | `number` | `undefined` | Délai d'expiration en secondes. S'il est omis, Claude Code applique le [délai par défaut de l'événement](#délai-dexpiration-des-hooks). Vos callbacks SDK suivent les valeurs par défaut des hooks `command` |

Utilisez le motif `matcher` pour cibler des outils précis chaque fois que possible. Un matcher `'Bash'` ne s'exécute que pour les commandes Bash, alors qu'omettre le motif exécute vos callbacks à chaque occurrence de l'événement. Omettez-le volontairement pour journaliser tous les appels d'outils de la session.

### Fonctions callback

#### Entrées

Chaque callback de hook reçoit trois arguments :

- **Données d'entrée** : un objet typé contenant les détails de l'événement. Chaque type de hook a sa propre forme d'entrée. Par exemple, `PreToolUseHookInput` inclut `tool_name` et `tool_input`, tandis que `NotificationHookInput` inclut `message`. Définitions de types complètes dans les références du SDK [TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript#hookinput) et [Python](https://code.claude.com/docs/en/agent-sdk/python#hookinput).
  - Toutes les entrées de hook partagent `session_id`, `cwd` et `hook_event_name`.
  - `agent_id` et `agent_type` sont renseignés quand le hook se déclenche à l'intérieur d'un sous-agent. En TypeScript, ils sont sur l'entrée de base et disponibles pour tous les types de hooks. En Python, ce sont des champs facultatifs sur `PreToolUse`, `PostToolUse`, `PostToolUseFailure` et `PermissionRequest`, et des champs obligatoires sur `SubagentStart` et `SubagentStop`.
- **ID d'utilisation d'outil** (`str | None` / `string | undefined`) : met en corrélation les événements `PreToolUse` et `PostToolUse` d'un même appel d'outil.
- **Contexte** : en TypeScript, contient une propriété `signal` (`AbortSignal`) pour l'annulation. En Python, cet argument est réservé pour un usage futur.

#### Sorties

Votre callback renvoie un objet comportant deux catégories de champs :

- **Champs de premier niveau**, acceptés pour tous les événements : `systemMessage` affiche un message à l'utilisateur, et `continue` (`continue_` en Python) détermine si l'agent continue à s'exécuter après ce hook. Certains événements les ignorent ou les livrent ailleurs ; la [section de chaque événement](https://code.claude.com/docs/en/hooks#hook-events) de la page des hooks indique où ils aboutissent.
- **`hookSpecificOutput`** contrôle l'opération en cours. Les champs à y placer dépendent du type d'événement :
  - Pour les hooks `PreToolUse`, c'est là que vous définissez `permissionDecision` (`"allow"`, `"deny"`, `"ask"` ou `"defer"`), `permissionDecisionReason` et `updatedInput`. Si vous renvoyez `"defer"`, le tour se termine par un message de résultat dont le `stop_reason` vaut `"tool_deferred"`, ce qui permet de [reprendre l'appel plus tard](https://code.claude.com/docs/en/hooks#defer-a-tool-call-for-later).
  - Pour les hooks `PostToolUse`, vous pouvez définir `additionalContext` pour ajouter des informations au résultat de l'outil. Pour remplacer la sortie de l'outil avant que Claude ne la voie, définissez `updatedToolOutput`, qui fonctionne pour n'importe quel outil dans les deux SDK. L'ancien champ `updatedMCPToolOutput` ne remplace que la sortie des outils MCP.
  - Dans le SDK TypeScript, un callback `PostToolUse` peut aussi renvoyer `classifierContext`, une courte note sur le résultat de l'appel d'outil destinée au classifieur de permissions du [mode auto](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode). Comme votre callback s'exécute dans le processus de votre propre application, le classifieur peut considérer une déclaration de l'utilisateur relayée dans la note comme une intention de l'utilisateur. Ce champ exige le SDK Agent TypeScript v0.3.236+. [Annotate a result for the auto mode classifier](https://code.claude.com/docs/en/hooks#annotate-a-result-for-the-auto-mode-classifier) détaille la limite de longueur, la règle « synchrone uniquement » et ce qu'il ne faut pas mettre dans la note.

Renvoyez `{}` pour autoriser l'opération sans modification. Les hooks callback du SDK utilisent le même format de sortie JSON que les [hooks de commande shell de Claude Code](https://code.claude.com/docs/en/hooks#json-output), page qui documente chaque champ et chaque option propre aux événements. Pour les définitions de types du SDK, voir les références [TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript#synchookjsonoutput) et [Python](https://code.claude.com/docs/en/agent-sdk/python#synchookjsonoutput).

> **Note** : lorsque plusieurs hooks ou règles de permission s'appliquent, `deny` l'emporte sur `defer`, qui l'emporte sur `ask`, qui l'emporte sur `allow`. Si un seul hook renvoie `deny`, l'opération est bloquée quels que soient les autres hooks.

#### Sortie asynchrone

Par défaut, l'agent attend que votre hook renvoie une valeur avant de continuer. Si votre hook produit un effet de bord (journalisation, envoi d'un webhook…) et n'a pas besoin d'influencer le comportement de l'agent, vous pouvez renvoyer une sortie asynchrone : l'agent continue alors immédiatement sans attendre la fin du hook. Dans cet extrait, `send_to_logging_service` (Python) et `sendToLoggingService` (TypeScript) représentent n'importe quelle fonction de journalisation que vous définissez :

##### Python

```python
async def async_hook(input_data, tool_use_id, context):
    # Lancer une tâche en arrière-plan, puis renvoyer immédiatement
    asyncio.create_task(send_to_logging_service(input_data))
    return {"async_": True, "asyncTimeout": 30000}
```

##### TypeScript

```typescript
const asyncHook: HookCallback = async (input, toolUseID, { signal }) => {
  // Lancer une tâche en arrière-plan, puis renvoyer immédiatement
  sendToLoggingService(input).catch(console.error);
  return { async: true, asyncTimeout: 30000 };
};
```

| Champ | Type | Description |
| - | - | - |
| `async` | `true` | Signale le mode asynchrone. L'agent continue sans attendre. En Python, utilisez `async_` pour éviter le mot-clé réservé. |
| `asyncTimeout` | `number` | Délai d'expiration facultatif, en millisecondes, pour l'opération en arrière-plan |

> **Note** : les sorties asynchrones ne peuvent ni bloquer, ni modifier, ni injecter de contexte dans l'opération, puisque l'agent est déjà passé à la suite. Réservez-les aux effets de bord : journalisation, métriques ou notifications.

## Exemples

Plusieurs exemples de cette section ne montrent que la fonction callback. Pour en exécuter un, enregistrez le callback sous l'événement correspondant dans le champ `hooks` de vos options, comme montré dans [Configurer les hooks](#configurer-les-hooks).

### Modifier l'entrée d'un outil

Cet exemple intercepte les appels à l'outil Write et réécrit l'argument `file_path` pour le préfixer de `/sandbox`, redirigeant toutes les écritures de fichiers vers un répertoire sandbox. Le callback renvoie `updatedInput` avec le chemin modifié et `permissionDecision: 'allow'` pour approuver automatiquement l'opération réécrite :

#### Python

```python
async def redirect_to_sandbox(input_data, tool_use_id, context):
    if input_data["hook_event_name"] != "PreToolUse":
        return {}

    if input_data["tool_name"] == "Write":
        original_path = input_data["tool_input"].get("file_path", "")
        return {
            "hookSpecificOutput": {
                "hookEventName": input_data["hook_event_name"],
                "permissionDecision": "allow",
                "updatedInput": {
                    **input_data["tool_input"],
                    "file_path": f"/sandbox{original_path}",
                },
            }
        }
    return {}
```

#### TypeScript

```typescript
const redirectToSandbox: HookCallback = async (input, toolUseID, { signal }) => {
  if (input.hook_event_name !== "PreToolUse") return {};

  const preInput = input as PreToolUseHookInput;
  const toolInput = preInput.tool_input as Record<string, unknown>;
  if (preInput.tool_name === "Write") {
    const originalPath = toolInput.file_path as string;
    return {
      hookSpecificOutput: {
        hookEventName: preInput.hook_event_name,
        permissionDecision: "allow",
        updatedInput: {
          ...toolInput,
          file_path: `/sandbox${originalPath}`
        }
      }
    };
  }
  return {};
};
```

> **Note** : associez `updatedInput` à `permissionDecision: 'allow'` pour approuver automatiquement l'entrée modifiée, ou à `permissionDecision: 'ask'` pour la montrer à l'utilisateur. Si vous omettez `permissionDecision`, l'entrée modifiée s'applique quand même et passe par l'évaluation normale des permissions. Avec `'defer'`, `updatedInput` est ignoré. Renvoyez toujours un nouvel objet plutôt que de muter le `tool_input` d'origine.

Pour vérifier la redirection, réglez le préfixe sur un chemin accessible en écriture, comme `./sandbox` ou `/tmp/sandbox` (macOS ne permet pas de créer un répertoire `/sandbox` à la racine), puis demandez à l'agent d'écrire un fichier : le résultat de l'outil Write dans le flux de messages indique le chemin avec votre préfixe sandbox plutôt que celui demandé par Claude.

### Ajouter du contexte et bloquer un outil

Cet exemple bloque les écritures dans le répertoire `/etc` et en explique la raison au modèle comme à l'utilisateur :

- `permissionDecision: 'deny'` arrête l'appel d'outil.
- `permissionDecisionReason` indique la raison au modèle, pour qu'il évite de réessayer.
- `systemMessage` montre à l'utilisateur ce qui s'est passé.

#### Python

```python
async def block_etc_writes(input_data, tool_use_id, context):
    file_path = input_data["tool_input"].get("file_path", "")

    if file_path.startswith("/etc"):
        return {
            # Champ de premier niveau : message affiché à l'utilisateur
            "systemMessage": "Remember: system directories like /etc are protected.",
            # hookSpecificOutput : bloquer l'opération
            "hookSpecificOutput": {
                "hookEventName": input_data["hook_event_name"],
                "permissionDecision": "deny",
                "permissionDecisionReason": "Writing to /etc is not allowed",
            },
        }
    return {}
```

#### TypeScript

```typescript
const blockEtcWrites: HookCallback = async (input, toolUseID, { signal }) => {
  const preInput = input as PreToolUseHookInput;
  const toolInput = preInput.tool_input as Record<string, unknown>;
  const filePath = toolInput?.file_path as string;

  if (filePath?.startsWith("/etc")) {
    return {
      // Champ de premier niveau : message affiché à l'utilisateur
      systemMessage: "Remember: system directories like /etc are protected.",
      // hookSpecificOutput : bloquer l'opération
      hookSpecificOutput: {
        hookEventName: preInput.hook_event_name,
        permissionDecision: "deny",
        permissionDecisionReason: "Writing to /etc is not allowed"
      }
    };
  }
  return {};
};
```

Pour vérifier le blocage, enregistrez le callback sous `PreToolUse` avec un matcher `Write|Edit` et demandez à l'agent de créer un fichier sous `/etc` : le résultat de l'outil Write dans le flux de messages contient `Writing to /etc is not allowed`, et aucun fichier n'est créé.

### Approuver automatiquement certains outils

Par défaut, l'agent peut demander une permission avant d'utiliser certains outils. Cet exemple approuve automatiquement les outils de système de fichiers en lecture seule (Read, Glob, Grep) en renvoyant `permissionDecision: 'allow'`, ce qui les laisse s'exécuter sans confirmation de l'utilisateur — sauf pour les lectures depuis des [chemins réseau](https://code.claude.com/docs/en/permissions#network-paths) — tandis que tous les autres outils restent soumis aux vérifications de permission normales :

#### Python

```python
async def auto_approve_read_only(input_data, tool_use_id, context):
    if input_data["hook_event_name"] != "PreToolUse":
        return {}

    read_only_tools = ["Read", "Glob", "Grep"]
    if input_data["tool_name"] in read_only_tools:
        return {
            "hookSpecificOutput": {
                "hookEventName": input_data["hook_event_name"],
                "permissionDecision": "allow",
                "permissionDecisionReason": "Read-only tool auto-approved",
            }
        }
    return {}
```

#### TypeScript

```typescript
const autoApproveReadOnly: HookCallback = async (input, toolUseID, { signal }) => {
  if (input.hook_event_name !== "PreToolUse") return {};

  const preInput = input as PreToolUseHookInput;
  const readOnlyTools = ["Read", "Glob", "Grep"];
  if (readOnlyTools.includes(preInput.tool_name)) {
    return {
      hookSpecificOutput: {
        hookEventName: preInput.hook_event_name,
        permissionDecision: "allow",
        permissionDecisionReason: "Read-only tool auto-approved"
      }
    };
  }
  return {};
};
```

### Enregistrer plusieurs hooks

Quand un événement se déclenche, tous les hooks correspondants s'exécutent **en parallèle**. Pour les décisions de permission, c'est le résultat le plus restrictif qui s'applique : un seul `deny` bloque l'appel d'outil, quoi que renvoient les autres hooks. Comme l'ordre d'achèvement n'est pas déterministe, écrivez chaque hook pour qu'il agisse indépendamment, sans supposer qu'un autre hook s'est exécuté avant lui.

L'exemple ci-dessous enregistre trois vérifications indépendantes pour chaque appel d'outil. Les noms de hooks, comme `audit_logger` (Python) ou `auditLogger` (TypeScript), représentent des callbacks que vous définissez :

#### Python

```python
options = ClaudeAgentOptions(
    hooks={
        "PreToolUse": [
            HookMatcher(hooks=[authorization_check]),
            HookMatcher(hooks=[input_validator]),
            HookMatcher(hooks=[audit_logger]),
        ]
    }
)
```

#### TypeScript

```typescript
const options = {
  hooks: {
    PreToolUse: [
      { hooks: [authorizationCheck] },
      { hooks: [inputValidator] },
      { hooks: [auditLogger] }
    ]
  }
};
```

### Filtrer avec des matchers multi-outils

Les matchers multi-outils permettent de partager un même callback entre des outils apparentés. Cet exemple enregistre trois matchers de portées différentes ; chaque hook nommé représente un callback que vous définissez :

- Une liste exacte séparée par des barres verticales (`Write|Edit|NotebookEdit`) déclenche `file_security_hook` uniquement pour les outils de modification de fichiers.
- Une regex (`^mcp__`) déclenche `mcp_audit_hook` pour tout outil MCP dont le nom commence par `mcp__`.
- Un matcher omis déclenche `global_logger` pour chaque appel d'outil, quel que soit son nom.

#### Python

```python
options = ClaudeAgentOptions(
    hooks={
        "PreToolUse": [
            # Correspond aux outils de modification de fichiers
            HookMatcher(matcher="Write|Edit|NotebookEdit", hooks=[file_security_hook]),
            # Correspond à tous les outils MCP
            HookMatcher(matcher="^mcp__", hooks=[mcp_audit_hook]),
            # Correspond à tout (pas de matcher)
            HookMatcher(hooks=[global_logger]),
        ]
    }
)
```

#### TypeScript

```typescript
const options = {
  hooks: {
    PreToolUse: [
      // Correspond aux outils de modification de fichiers
      { matcher: "Write|Edit|NotebookEdit", hooks: [fileSecurityHook] },

      // Correspond à tous les outils MCP
      { matcher: "^mcp__", hooks: [mcpAuditHook] },

      // Correspond à tout (pas de matcher)
      { hooks: [globalLogger] }
    ]
  }
};
```

### Suivre l'activité des sous-agents

Utilisez les hooks `SubagentStop` pour surveiller la fin du travail des sous-agents. Le type d'entrée complet est dans les références du SDK [TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript#hookinput) et [Python](https://code.claude.com/docs/en/agent-sdk/python#hookinput). Cet exemple journalise un résumé chaque fois qu'un sous-agent se termine :

#### Python

```python
async def subagent_tracker(input_data, tool_use_id, context):
    # Journaliser les détails du sous-agent quand il se termine
    print(f"[SUBAGENT] Completed: {input_data['agent_id']}")
    print(f"  Transcript: {input_data['agent_transcript_path']}")
    print(f"  Tool use ID: {tool_use_id}")
    print(f"  Stop hook active: {input_data.get('stop_hook_active')}")
    return {}


options = ClaudeAgentOptions(
    hooks={"SubagentStop": [HookMatcher(hooks=[subagent_tracker])]}
)
```

#### TypeScript

```typescript
import { HookCallback, SubagentStopHookInput } from "@anthropic-ai/claude-agent-sdk";

const subagentTracker: HookCallback = async (input, toolUseID, { signal }) => {
  // Convertir en SubagentStopHookInput pour accéder aux champs propres aux sous-agents
  const subInput = input as SubagentStopHookInput;

  // Journaliser les détails du sous-agent quand il se termine
  console.log(`[SUBAGENT] Completed: ${subInput.agent_id}`);
  console.log(`  Transcript: ${subInput.agent_transcript_path}`);
  console.log(`  Tool use ID: ${toolUseID}`);
  console.log(`  Stop hook active: ${subInput.stop_hook_active}`);
  return {};
};

const options = {
  hooks: {
    SubagentStop: [{ hooks: [subagentTracker] }]
  }
};
```

Pour vérifier que le hook se déclenche, enregistrez le callback et demandez à l'agent de déléguer une petite tâche à un sous-agent (par exemple lister les fichiers du répertoire courant) : à la fin du sous-agent, le callback affiche les lignes `[SUBAGENT] Completed:` avec l'ID du sous-agent et le chemin de son transcript.

### Effectuer des requêtes HTTP depuis les hooks

Les hooks peuvent effectuer des opérations asynchrones comme des requêtes HTTP. Interceptez les erreurs à l'intérieur du hook plutôt que de les laisser se propager.

Cet exemple envoie un webhook après chaque exécution d'outil, en indiquant quel outil a tourné et quand. Le hook intercepte les erreurs d'un webhook en échec :

#### Python

```python
import asyncio
import json
import urllib.request
from datetime import datetime


def _send_webhook(tool_name):
    """Utilitaire synchrone qui envoie en POST les données d'usage d'outil à un webhook externe."""
    data = json.dumps(
        {
            "tool": tool_name,
            "timestamp": datetime.now().isoformat(),
        }
    ).encode()
    req = urllib.request.Request(
        "https://api.example.com/webhook",
        data=data,
        headers={"Content-Type": "application/json"},
        method="POST",
    )
    urllib.request.urlopen(req)


async def webhook_notifier(input_data, tool_use_id, context):
    # Ne se déclencher qu'après la fin d'un outil (PostToolUse), pas avant
    if input_data["hook_event_name"] != "PostToolUse":
        return {}

    try:
        # Exécuter l'appel HTTP bloquant dans un thread pour ne pas bloquer la boucle d'événements
        await asyncio.to_thread(_send_webhook, input_data["tool_name"])
    except Exception as e:
        # Journaliser l'erreur sans la relancer
        print(f"Webhook request failed: {e}")

    return {}
```

#### TypeScript

```typescript
import { query, HookCallback, PostToolUseHookInput } from "@anthropic-ai/claude-agent-sdk";

const webhookNotifier: HookCallback = async (input, toolUseID, { signal }) => {
  // Ne se déclencher qu'après la fin d'un outil (PostToolUse), pas avant
  if (input.hook_event_name !== "PostToolUse") return {};

  try {
    await fetch("https://api.example.com/webhook", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        tool: (input as PostToolUseHookInput).tool_name,
        timestamp: new Date().toISOString()
      }),
      // Passer signal pour que la requête soit annulée si le hook expire
      signal
    });
  } catch (error) {
    // Traiter l'annulation séparément des autres erreurs
    if (error instanceof Error && error.name === "AbortError") {
      console.log("Webhook request cancelled");
    }
    // Ne pas relancer l'erreur
  }

  return {};
};

// Enregistrer comme hook PostToolUse
for await (const message of query({
  prompt: "Refactor the auth module",
  options: {
    hooks: {
      PostToolUse: [{ hooks: [webhookNotifier] }]
    }
  }
})) {
  console.log(message);
}
```

Pour vérifier que le hook se déclenche, faites pointer l'URL du webhook vers un point de terminaison que vous pouvez observer et envoyez un prompt qui utilise un outil : le hook envoie un POST avec le nom de l'outil et l'horodatage après chaque exécution d'outil.

### Transférer les notifications vers Slack

Utilisez les hooks `Notification` pour recevoir les notifications système de l'agent et les transférer vers des services externes. Dans les sessions SDK, Claude Code exécute ce hook pour les types de notification suivants :

- [`permission_prompt`](https://code.claude.com/docs/en/hooks#notification), une fois qu'une demande de permission a attendu environ six secondes sur votre [callback `canUseTool`](https://code.claude.com/docs/en/agent-sdk/user-input). Exige le SDK Agent TypeScript v0.3.233+ ou le SDK Agent Python v0.2.139+.
- `elicitation_complete` et `elicitation_response`, pour les flux d'élicitation de saisie utilisateur.

Claude Code émet les autres types, comme `idle_prompt`, `auth_success` et `elicitation_dialog`, depuis l'interface interactive, que les sessions SDK n'exécutent pas.

Chaque notification comprend un champ `message` contenant une description lisible et, éventuellement, un `title`.

Cet exemple transfère chaque notification vers un canal Slack. Il nécessite une [URL de webhook entrant Slack](https://docs.slack.dev/messaging/sending-messages-using-incoming-webhooks/), que vous créez en ajoutant une application à votre espace de travail Slack et en activant les webhooks entrants :

#### Python

```python
import asyncio
import json
import urllib.request

from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions, HookMatcher


def _send_slack_notification(message):
    """Utilitaire synchrone qui envoie un message à Slack via un webhook entrant."""
    data = json.dumps({"text": f"Agent status: {message}"}).encode()
    req = urllib.request.Request(
        "https://hooks.slack.com/services/YOUR/WEBHOOK/URL",
        data=data,
        headers={"Content-Type": "application/json"},
        method="POST",
    )
    urllib.request.urlopen(req)


async def notification_handler(input_data, tool_use_id, context):
    try:
        # Exécuter l'appel HTTP bloquant dans un thread pour ne pas bloquer la boucle d'événements
        await asyncio.to_thread(_send_slack_notification, input_data.get("message", ""))
    except Exception as e:
        print(f"Failed to send notification: {e}")

    # Renvoyer un objet vide. Les hooks Notification ne modifient pas le comportement de l'agent
    return {}


async def main():
    options = ClaudeAgentOptions(
        hooks={
            # Enregistrer le hook pour les événements Notification (pas besoin de matcher)
            "Notification": [HookMatcher(hooks=[notification_handler])],
        },
    )

    async with ClaudeSDKClient(options=options) as client:
        await client.query("Analyze this codebase")
        async for message in client.receive_response():
            print(message)


asyncio.run(main())
```

#### TypeScript

```typescript
import { query, HookCallback, NotificationHookInput } from "@anthropic-ai/claude-agent-sdk";

// Définir un callback de hook qui envoie les notifications à Slack
const notificationHandler: HookCallback = async (input, toolUseID, { signal }) => {
  // Convertir en NotificationHookInput pour accéder au champ message
  const notification = input as NotificationHookInput;

  try {
    // Envoyer en POST le message de notification à un webhook entrant Slack
    await fetch("https://hooks.slack.com/services/YOUR/WEBHOOK/URL", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        text: `Agent status: ${notification.message}`
      }),
      // Passer signal pour que la requête soit annulée si le hook expire
      signal
    });
  } catch (error) {
    if (error instanceof Error && error.name === "AbortError") {
      console.log("Notification cancelled");
    } else {
      console.error("Failed to send notification:", error);
    }
  }

  // Renvoyer un objet vide. Les hooks Notification ne modifient pas le comportement de l'agent
  return {};
};

// Enregistrer le hook pour les événements Notification (pas besoin de matcher)
for await (const message of query({
  prompt: "Analyze this codebase",
  options: {
    hooks: {
      Notification: [{ hooks: [notificationHandler] }]
    }
  }
})) {
  console.log(message);
}
```

Quand un événement `Notification` se déclenche, le hook publie le `message` de la notification, préfixé par `Agent status:`, dans le canal ciblé par votre webhook.

## Résoudre les problèmes courants

### Le hook ne se déclenche pas

- Vérifiez que le nom de l'événement est correct et respecte la casse (`PreToolUse`, pas `preToolUse`).
- Vérifiez que votre motif de matcher correspond exactement au nom de l'outil.
- Assurez-vous que le hook est placé sous le bon type d'événement dans `options.hooks`.
- Pour les hooks non liés aux outils qui acceptent des matchers, comme `Notification` et `SubagentStop`, les matchers portent sur d'autres champs, et `Stop` ignore complètement les matchers (voir [matcher patterns](https://code.claude.com/docs/en/hooks#matcher-patterns)).
- Les hooks peuvent ne pas se déclencher quand l'agent atteint la limite [`max_turns`](https://code.claude.com/docs/en/agent-sdk/python#claudeagentoptions), car la session se termine avant que les hooks puissent s'exécuter.

### Le matcher ne filtre pas comme prévu

Les matchers ne portent que sur les noms d'outils, pas sur les chemins de fichiers ni les autres arguments. Pour filtrer par chemin de fichier, vérifiez `tool_input.file_path` à l'intérieur du hook :

```typescript
const myHook: HookCallback = async (input, toolUseID, { signal }) => {
  const preInput = input as PreToolUseHookInput;
  const toolInput = preInput.tool_input as Record<string, unknown>;
  const filePath = toolInput?.file_path as string;
  if (!filePath?.endsWith(".md")) return {}; // Ignorer les fichiers non markdown
  // Traiter les fichiers markdown...
  return {};
};
```

### Délai d'expiration des hooks

Claude Code exécute chaque callback avec un délai d'expiration, que vous définissez en secondes via le champ `timeout` de son `HookMatcher`. À défaut, Claude Code utilise la valeur par défaut de l'événement :

| Événement(s) | Délai par défaut |
| - | - |
| La plupart des événements | 600 secondes |
| `UserPromptSubmit`, `PreModelSwitch`, `PostModelSwitch` | 30 secondes |
| `MessageDisplay` | 10 secondes |
| `SessionEnd` | Exécuté pendant l'arrêt, sous le [budget de délai SessionEnd](https://code.claude.com/docs/en/hooks#sessionend-input), plus court : 1,5 seconde par défaut |

Quand un callback dépasse son délai, Claude Code l'annule et ignore sa sortie ; la session continue au lieu de rester bloquée. La suite dépend de l'événement :

- **`PreToolUse`** : Claude Code n'exécute pas l'appel d'outil, Claude reçoit un résultat d'outil indiquant que le hook n'a pas répondu avant son délai, et le tour continue. Si un autre hook `PreToolUse` a renvoyé un refus explicite, Claude reçoit ce refus plutôt que l'erreur de délai. Avant la v2.1.210, Claude Code signalait le dépassement à Claude comme un rejet de l'utilisateur, ce qui faisait s'arrêter les sessions sans surveillance en attente d'une saisie.
- **`PostToolUse` et `PostToolUseFailure`** : Claude Code conserve le résultat de l'outil et le tour continue.
- **`UserPromptSubmit` et [`UserPromptExpansion`](https://code.claude.com/docs/en/hooks#userpromptexpansion)** : Claude Code bloque le prompt avec un message nommant le hook et le délai, et la session continue. Comme un callback sur ces événements peut servir de garde-fou de politique, Claude Code ne laisse jamais passer un prompt non contrôlé après un dépassement de délai. Avant la v2.1.208, Claude Code terminait la requête avec `error_during_execution` quand un callback sur ces événements expirait.
- **`Stop` et `SubagentStop`** : le callback expiré compte comme n'ayant renvoyé aucune décision. L'agent ou le sous-agent s'arrête comme si ce callback l'avait autorisé, et la décision de vos autres hooks sur l'événement s'applique toujours. Avant Claude Code v2.1.273, un callback `Stop` ou `SubagentStop` expiré comptait comme une exécution de hook en échec, et Claude Code ignorait les décisions de vos autres hooks sur l'événement.
- **`SessionStart`** : le callback expiré compte comme n'ayant renvoyé aucune sortie, et la session continue avec la sortie de vos autres hooks `SessionStart`.
- **`PreModelSwitch`** : Claude Code bloque le changement de modèle. Un hook qui ne répond pas n'a pas approuvé le changement.
- **Autres événements**, comme `Notification`, `PreCompact` et `PostModelSwitch` : Claude Code journalise l'échec et continue.

La première fois qu'un callback `Stop` ou `SessionStart` expire dans la session principale, Claude Code ajoute aussi au flux de messages un [`SDKInformationalMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkinformationalmessage) indiquant que l'application qui pilote la session n'a pas répondu. Les dépassements suivants ne répètent pas ce message tant que votre application reste sans réponse.

Si vous interrompez la requête alors qu'un callback est en attente, Claude Code annule l'appel d'outil en attente. Avant la v2.1.208, l'appel d'outil pouvait quand même s'exécuter si vous interrompiez pendant un callback `PreToolUse` en attente.

Si votre callback a besoin de plus de temps, définissez un `timeout` plus élevé sur son `HookMatcher`. En TypeScript, utilisez l'`AbortSignal` du troisième argument du callback pour gérer proprement l'annulation quand le délai expire.

### Outil bloqué de façon inattendue

- Vérifiez tous les hooks `PreToolUse` à la recherche de retours `permissionDecision: 'deny'`.
- Ajoutez de la journalisation à vos hooks pour voir quel `permissionDecisionReason` ils renvoient.
- Vérifiez que les motifs de matcher ne sont pas trop larges : un matcher vide correspond à tous les outils.

### L'entrée modifiée n'est pas appliquée

- Assurez-vous que `updatedInput` se trouve **dans** `hookSpecificOutput`, et non au premier niveau :

  ```typescript
  return {
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "allow",
      updatedInput: { command: "new command" }
    }
  };
  ```

- N'associez pas `updatedInput` à `permissionDecision: 'defer'`, qui abandonne l'entrée modifiée. Omettre `permissionDecision` ne pose pas de problème : l'entrée modifiée s'applique via l'évaluation normale des permissions. Vous pouvez aussi renvoyer `'allow'` pour approuver automatiquement l'entrée modifiée, ou `'ask'` pour la soumettre à l'approbation de l'utilisateur.
- Incluez `hookEventName` dans `hookSpecificOutput` pour identifier à quel type de hook la sortie se rapporte.

### Hooks de session indisponibles en Python

`SessionStart` et `SessionEnd` peuvent être enregistrés comme hooks callback du SDK en TypeScript, mais ne sont pas disponibles dans le SDK Python, car son type `HookEvent` les omet. En Python, ils ne sont disponibles que comme [hooks de commande shell](https://code.claude.com/docs/en/hooks#hook-events) définis dans des fichiers de réglages tels que `.claude/settings.json`. Les fichiers de réglages chargés par votre application SDK dépendent de [`setting_sources`](https://code.claude.com/docs/en/agent-sdk/python#settingsource) ou [`settingSources`](https://code.claude.com/docs/en/agent-sdk/typescript#settingsource). Si vous définissez cette option, incluez la source qui contient les hooks :

#### Python

```python
options = ClaudeAgentOptions(
    setting_sources=["project"],  # Charge .claude/settings.json, hooks compris
)
```

#### TypeScript

```typescript
const options = {
  settingSources: ["project"] // Charge .claude/settings.json, hooks compris
};
```

Pour exécuter plutôt une logique d'initialisation sous forme de callback du SDK Python, utilisez le premier message de `client.receive_response()` comme déclencheur.

### Multiplication des demandes de permission des sous-agents

Lorsque vous lancez plusieurs sous-agents, chacun peut demander des permissions séparément pour ses propres appels d'outils. Pour éviter les demandes répétées, utilisez des hooks `PreToolUse` pour approuver automatiquement certains outils, ou configurez des règles de permission, dont les sous-agents [héritent de la conversation parente](https://code.claude.com/docs/en/sub-agents#permission-modes).

### Boucles récursives de hooks avec des sous-agents

Un hook `UserPromptSubmit` qui lance des sous-agents peut créer des boucles infinies si ces sous-agents déclenchent le même hook. Pour l'éviter :

- Utilisez une variable partagée ou l'état de session pour savoir si vous êtes déjà à l'intérieur d'un sous-agent.
- Limitez les hooks à la seule session de l'agent de premier niveau.

### systemMessage n'apparaît pas dans la sortie

Le champ `systemMessage` affiche un message à l'**utilisateur**, pas au modèle. Avec Claude Code v2.1.227+, le `systemMessage` d'un hook peut apparaître dans le flux de messages sous forme de [`SDKInformationalMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkinformationalmessage) ; cela dépend de l'événement. La [section de chaque événement](https://code.claude.com/docs/en/hooks#hook-events) de la page des hooks indique comment la sortie est remontée. Pour transmettre plutôt du contexte au modèle, renvoyez [`additionalContext`](https://code.claude.com/docs/en/hooks#add-context-for-claude).

Avant la v2.1.227, le SDK ne remontait la sortie des hooks dans le flux de messages que pour les hooks `SessionStart` et `Setup`. Pour tout autre événement, la sortie n'apparaissait que dans les événements de cycle de vie ajoutés par [`includeHookEvents`](https://code.claude.com/docs/en/agent-sdk/typescript#options) (`include_hook_events` en Python). L'entrée de cette option précise quels événements de cycle de vie chaque événement de hook produit.

Si vous devez remonter les décisions des hooks à votre application de manière fiable, journalisez-les séparément ou utilisez un canal de sortie dédié.

## Ressources associées

- [Référence des hooks de Claude Code](https://code.claude.com/docs/en/hooks) : schémas JSON complets d'entrée/sortie, documentation des événements et motifs de matcher.
- [Guide des hooks de Claude Code](https://code.claude.com/docs/en/hooks-guide) : exemples et pas-à-pas de hooks de commande shell.
- [Référence du SDK TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript) : types de hooks, définitions d'entrée/sortie et options de configuration.
- [Référence du SDK Python](https://code.claude.com/docs/en/agent-sdk/python) : types de hooks, définitions d'entrée/sortie et options de configuration.
- [Permissions](https://code.claude.com/docs/en/agent-sdk/permissions) : contrôler ce que votre agent peut faire.
- [Outils personnalisés](https://code.claude.com/docs/en/agent-sdk/custom-tools) : créer des outils pour étendre les capacités de l'agent.
