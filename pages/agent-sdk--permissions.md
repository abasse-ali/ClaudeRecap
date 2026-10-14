# Configurer les permissions (Agent SDK)

> Source : https://code.claude.com/docs/en/agent-sdk/permissions · [← Index](../31-index-documentation-officielle.md)

> Contrôler la manière dont votre agent utilise les outils grâce aux modes de permission, aux hooks et aux règles déclaratives allow/deny.

Le Claude Agent SDK fournit des contrôles de permission pour encadrer l'usage des outils par Claude. On utilise les modes de permission et les règles pour définir ce qui est autorisé automatiquement, et le [callback `canUseTool`](https://code.claude.com/docs/en/agent-sdk/user-input) pour traiter tout le reste à l'exécution.

## Comment les permissions sont évaluées

Quand Claude demande un outil, le SDK vérifie les permissions dans cet ordre :

### Étape 1 — Hooks

Les [hooks](https://code.claude.com/docs/en/agent-sdk/hooks) s'exécutent en premier. Un hook peut refuser l'appel directement ou le laisser passer.
- Un hook qui renvoie `allow` ne court-circuite **pas** les règles deny et ask qui suivent : elles sont évaluées quel que soit le résultat du hook.
- Un `allow` d'un hook `PreToolUse` ne peut pas non plus approuver une suppression `rm` ou `rmdir` visant un [chemin critique](https://code.claude.com/docs/en/permission-modes#critical-paths).

### Étape 2 — Règles deny

Vérification des règles `deny` (provenant de `disallowed_tools` et de [settings.json](https://code.claude.com/docs/en/settings-reference#permission-settings)).
- Si une règle deny correspond, l'outil est bloqué, **même en mode `bypassPermissions`**.
- Les règles deny par nom nu, comme `Bash`, retirent l'outil du contexte de Claude avant même le début de cette évaluation ; seules les règles à portée, comme `Bash(rm *)`, sont donc vérifiées à cette étape.

### Étape 3 — Règles ask

Vérification des règles `ask` de [settings.json](https://code.claude.com/docs/en/settings-reference#permission-settings). Si une règle ask correspond, l'appel est transmis à votre [callback `canUseTool`](https://code.claude.com/docs/en/agent-sdk/user-input) pour confirmation, **même en mode `bypassPermissions`**.

Les outils qui exigent une interaction utilisateur se comportent de la même façon : `AskUserQuestion` et les outils MCP dont le serveur définit [`_meta["anthropic/requiresUserInteraction"]`](https://code.claude.com/docs/en/mcp#require-approval-for-a-specific-tool) passent toujours au callback, même quand une règle allow correspond. En mode `dontAsk`, ces deux cas sont refusés à la place, car ce mode ne demande jamais rien. L'annotation MCP exige Claude Code v2.1.199+.

Les outils de [connecteurs claude.ai](https://code.claude.com/docs/en/mcp#organization-controls-on-connector-tools) que votre organisation a réglés sur `ask` sortent aussi du flux à cette étape. Chaque appel passe au callback, même en mode `bypassPermissions` et même quand une règle allow correspond. Le callback reçoit la raison `Your organization requires approval for this tool`. En mode `dontAsk`, l'appel est refusé à la place, car ce mode ne demande jamais rien.

### Étape 4 — Mode de permission

Application du [mode de permission](#modes-de-permission) actif :

- En mode `bypassPermissions`, Claude Code approuve tout ce qui atteint cette étape, sauf les suppressions `rm` et `rmdir` visant un [chemin critique](https://code.claude.com/docs/en/permission-modes#critical-paths), qui continuent leur chemin dans le flux.
- En mode `acceptEdits`, Claude Code approuve les opérations sur fichiers listées dans [Mode accept edits](#mode-accept-edits-acceptedits).
- En mode `plan`, Claude Code envoie les outils d'édition de fichiers et d'écriture shell à votre callback `canUseTool`, quelles que soient les règles allow, pour que les opérations d'écriture ne puissent pas être auto-approuvées pendant la planification.
- Dans les autres modes, la requête continue vers l'étape suivante.

### Étape 5 — Règles allow

Vérification des règles `allow` (provenant de `allowed_tools` et de settings.json). Si une règle correspond, l'outil est approuvé. Un appel que l'outil approuve de lui-même est également résolu à cette étape, sans règle nécessaire : par exemple une lecture de fichier à l'intérieur de vos répertoires de travail ou une [commande Bash en lecture seule](https://code.claude.com/docs/en/permissions#read-only-commands).

- Les suppressions `rm` et `rmdir` visant un [chemin critique](https://code.claude.com/docs/en/permission-modes#critical-paths) ne sont **jamais** approuvées par une règle allow. Qu'elles atteignent ensuite votre callback dépend du mode de permission : dans une session Agent SDK en mode `auto`, par exemple, Claude Code les refuse par défaut sans l'appeler. Le tableau des modes de la section [Critical paths](https://code.claude.com/docs/en/permission-modes#critical-paths) indique ce que fait chaque mode.
- Une règle allow n'approuve pas une lecture depuis un [chemin réseau](https://code.claude.com/docs/en/permissions#network-paths).

### Étape 6 — Callback canUseTool

Si rien de ce qui précède n'a résolu l'appel, votre [callback `canUseTool`](https://code.claude.com/docs/en/agent-sdk/user-input) est appelé pour décider. En mode `dontAsk`, cette étape est sautée et l'outil est refusé.

Dans le SDK TypeScript, si vous définissez [`permissionPrompts: 'none'`](https://code.claude.com/docs/en/agent-sdk/typescript#options), votre callback n'est pas appelé à cette étape. Un [hook `PermissionRequest`](https://code.claude.com/docs/en/hooks#permissionrequest) a encore la possibilité de décider ; s'il ne le fait pas, Claude Code refuse l'appel. Cette option exige Claude Code v2.1.259+.

*(Schéma de la doc : la requête d'outil traverse hooks → règles deny → règles ask → mode de permission → règles allow → canUseTool. Hooks, règles deny et canUseTool peuvent mener à « Bloqué » ; le mode bypass, les règles allow et canUseTool peuvent mener à « Exécuter » ; les règles ask renvoient vers canUseTool.)*

### Avertissement `CLAUDE_SDK_CAN_USE_TOOL_SHADOWED`

Si vous passez un callback `canUseTool` dans une configuration où le SDK TypeScript s'attend à ce que l'ordre d'évaluation auto-approuve les appels avant que le callback ne soit consulté, le SDK émet une fois un avertissement de processus Node.js à la construction de la requête. Le code de l'avertissement est `CLAUDE_SDK_CAN_USE_TOOL_SHADOWED`. Deux configurations le déclenchent :

- `permissionMode: 'bypassPermissions'`, qui auto-approuve tout appel atteignant l'étape du mode de permission, à l'exception des [actions qu'aucun mode n'auto-approuve](https://code.claude.com/docs/en/permission-modes#actions-no-mode-auto-approves).
- Chaque entrée nue de `allowedTools` comme `"Read"`, qui auto-approuve l'outil entier avant que le callback ne soit consulté, à l'exception des [actions qu'aucun mode n'auto-approuve](https://code.claude.com/docs/en/permission-modes#actions-no-mode-auto-approves) et des [lectures depuis des chemins réseau](https://code.claude.com/docs/en/permissions#network-paths).

Les entrées avec un spécificateur, comme `Bash(ls *)`, et le mode `acceptEdits` ne le déclenchent pas ; les règles allow issues des fichiers de réglages ne sont pas visibles par cette vérification.

Écoutez avec `process.on('warning', ...)` et filtrez sur le code pour journaliser ou supprimer l'avertissement. Pour contrôler chaque appel d'outil quels que soient le mode et les règles, utilisez plutôt un [hook `PreToolUse`](https://code.claude.com/docs/en/agent-sdk/hooks).

Cette page se concentre sur les **règles allow et deny** et sur les **modes de permission**. Pour les autres étapes :

- **Hooks** : exécuter du code personnalisé pour autoriser, refuser ou modifier les requêtes d'outils. Voir [Control execution with hooks](https://code.claude.com/docs/en/agent-sdk/hooks).
- **Callback canUseTool** : demander l'approbation de l'utilisateur à l'exécution, quand aucune étape antérieure n'a résolu l'appel. Voir [Handle approvals and user input](https://code.claude.com/docs/en/agent-sdk/user-input).

## Règles allow et deny

`allowed_tools` et `disallowed_tools` (TypeScript : `allowedTools` / `disallowedTools`) ajoutent des entrées aux listes de règles allow et deny du flux d'évaluation ci-dessus.
- Si vous nommez l'un des [outils de suivi de tâches](https://code.claude.com/docs/en/agent-sdk/todo-tracking#model-availability) dans `allowed_tools`, Claude Code active aussi ces outils pour la session.
- Tout autre outil non listé dans `allowed_tools` reste disponible pour Claude, et un appel à cet outil qui nécessite une approbation passe au mode de permission.
- Les règles deny se comportent différemment selon qu'elles nomment un outil ou qu'elles délimitent un motif à l'intérieur d'un outil.

| Option | Effet |
| :- | :- |
| `allowed_tools=["Read", "Grep"]` | `Read` et `Grep` sont auto-approuvés, sauf pour les [lectures depuis des chemins réseau](https://code.claude.com/docs/en/permissions#network-paths). Les autres outils non listés existent toujours, et les appels qui nécessitent une approbation passent au mode de permission puis à `canUseTool`. |
| `disallowed_tools=["Bash"]` | La définition de l'outil `Bash` est retirée de la requête. Claude ne voit pas l'outil et ne peut pas tenter de l'utiliser. |
| `disallowed_tools=["Bash(rm *)"]` | `Bash` reste disponible. Les appels correspondant à `rm *` [tel qu'écrit](https://code.claude.com/docs/en/permissions#bash-rule-limits) sont refusés dans tous les modes de permission, y compris `bypassPermissions`. Les autres appels `Bash`, y compris `/bin/rm`, passent au mode de permission. |
| `disallowed_tools=["*"]` | Toutes les définitions d'outils sont retirées de la requête. Les globs de noms d'outils sont pris en charge dans les règles deny : `"*"` correspond à tous les outils et `"mcp__*"` à tous les outils MCP de tous les serveurs. |

**Globs dans les règles allow** : les règles allow n'acceptent les globs de noms d'outils qu'après un préfixe littéral `mcp__<server>__`. Le segment serveur doit être sans glob, pour que la règle désigne un serveur précis que vous avez configuré : `mcp__puppeteer__*` correspond à tous les outils du serveur `puppeteer`, et `mcp__github__get_*` à ses outils `get_`. Une entrée non ancrée comme `allowed_tools=["*"]` ou `allowed_tools=["mcp__*"]` est ignorée avec un avertissement au démarrage et n'auto-approuve rien.

**Règles `Read` et `Edit` à portée** : elles prennent un motif de chemin. Les règles `Edit(path)` régissent tous les outils intégrés qui écrivent des fichiers, y compris `Write` et `NotebookEdit` ; une règle `Write(path)` n'est jamais prise en compte par les vérifications de permission sur les fichiers.

**Chemins absolus** : utilisez `//path` pour un chemin absolu du système de fichiers : une règle deny `Edit(//secrets/**)` bloque les écritures n'importe où sous `/secrets` sur le disque. Avec une seule barre oblique initiale, `Edit(/secrets/**)` s'ancre à la source de la règle. Pour les règles passées via `allowed_tools` ou `disallowed_tools`, cela signifie le répertoire de travail de la session : la règle ne bloque donc pas `/secrets` sur le disque. Voir [Read and Edit rules](https://code.claude.com/docs/en/permissions#read-and-edit) pour les quatre formes d'ancrage et la résolution des règles issues des fichiers de réglages.

> **Avertissement** : **Les outils auto-approuvés n'atteignent jamais `canUseTool`.** Un appel d'outil approuvé à une étape antérieure — par `acceptEdits` ou `bypassPermissions`, ou par une règle allow — saute votre callback `canUseTool` : les vérifications de permission que vous y avez placées sont donc silencieusement contournées pour cet outil.
>
> Une règle allow n'auto-approuve jamais `AskUserQuestion`, les outils MCP marqués [`_meta["anthropic/requiresUserInteraction"]`](https://code.claude.com/docs/en/mcp#require-approval-for-a-specific-tool), les outils de connecteurs que [votre organisation a réglés sur `ask`](https://code.claude.com/docs/en/mcp#organization-controls-on-connector-tools), ni les suppressions `rm` et `rmdir` visant un [chemin critique](https://code.claude.com/docs/en/permission-modes#critical-paths). En mode `dontAsk`, Claude Code refuse ces appels sans invoquer le callback. Dans les autres modes, les trois premiers atteignent le callback. Selon le [mode de permission](https://code.claude.com/docs/en/permission-modes#critical-paths), une suppression de chemin critique atteint aussi le callback, ou bien Claude Code la refuse sans l'appeler, comme il le fait par défaut pour une session Agent SDK en mode `auto`.
>
> La couverture dépend de la forme de l'entrée : un nom nu comme `Read` ou `mcp__github__get_issue` auto-approuve tous les appels à cet outil, hormis ces exceptions et les [lectures depuis des chemins réseau](https://code.claude.com/docs/en/permissions#network-paths) ; une règle à portée comme `Bash(npm test *)` n'auto-approuve que les appels correspondants, et les autres appels `Bash` nécessitant une approbation passent toujours au callback. Pour des vérifications qui doivent s'exécuter à chaque appel d'outil, utilisez un [hook `PreToolUse`](https://code.claude.com/docs/en/agent-sdk/hooks) : les hooks s'exécutent avant toutes les autres étapes, et un refus par hook s'applique même en mode `bypassPermissions`.

### Agent verrouillé : `allowedTools` + `dontAsk`

Pour un agent verrouillé, associez `allowedTools` à `permissionMode: "dontAsk"` :

```typescript theme={null}
const options = {
  allowedTools: ["Read", "Glob", "Grep"],
  permissionMode: "dontAsk"
};
```

Les outils listés sont approuvés, à l'exception des [actions qu'aucun mode n'auto-approuve](https://code.claude.com/docs/en/permission-modes#actions-no-mode-auto-approves) et des [lectures depuis des chemins réseau](https://code.claude.com/docs/en/permissions#network-paths), et tout autre appel qui demanderait une confirmation est refusé à la place. Les appels qui ne nécessitent aucune approbation en mode `default` s'exécutent que vous les listiez ou non : par exemple les [commandes Bash en lecture seule](https://code.claude.com/docs/en/permissions#read-only-commands), des outils comme `Agent` qui ne demandent rien avant de s'exécuter, et les lectures de fichiers dans vos répertoires de travail. Pour retirer complètement un outil de la requête, ajoutez son nom nu à `disallowedTools`.

> **Avertissement** : **`allowed_tools` ne restreint pas `bypassPermissions`.** `allowed_tools` pré-approuve les outils que vous listez. Les autres outils non listés ne correspondent à aucune règle allow et passent au mode de permission, où `bypassPermissions` les approuve. Définir `allowed_tools=["Read"]` avec `permission_mode="bypassPermissions"` approuve quand même tous les outils, y compris `Bash`, `Write` et `Edit`. Si vous avez besoin de `bypassPermissions` mais voulez bloquer certains outils, utilisez `disallowed_tools`.

### Règles déclaratives dans `.claude/settings.json`

Vous pouvez aussi configurer des règles allow, deny et ask de façon déclarative dans `.claude/settings.json`. Ces règles sont lues quand la source de réglages `project` est activée, ce qui est le cas avec les options `query()` par défaut. Si vous définissez `setting_sources` (TypeScript : `settingSources`) explicitement, incluez `"project"` pour qu'elles s'appliquent. Voir [Permission settings](https://code.claude.com/docs/en/settings-reference#permission-settings) pour la syntaxe des règles.

## Modes de permission

Les modes de permission offrent un contrôle global sur la manière dont Claude utilise les outils. Vous pouvez définir le mode à l'appel de `query()` ou le changer dynamiquement pendant les sessions en streaming.

Si vous n'en définissez pas, Claude Code choisit le mode de départ selon les règles de [Which mode a session starts in](https://code.claude.com/docs/en/permission-modes#which-mode-a-session-starts-in) :

- Un `permissions.defaultMode` provenant des [fichiers de réglages](https://code.claude.com/docs/en/settings#where-settings-live) de la session, quand il y en a un qui s'applique ;
- Sinon le défaut intégré, qui peut être le [mode auto](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode).

Une session qui démarre en mode auto abandonne les règles allow larges, comme une entrée `Bash` nue, comme le décrit [How auto mode evaluates actions](https://code.claude.com/docs/en/permission-modes#how-auto-mode-evaluates-actions). Si votre application dépend du mode `default` ou d'une telle règle, passez `default` explicitement.

Avant le SDK Agent TypeScript v0.3.286, omettre `permissionMode` équivalait à passer `default`.

### Modes disponibles

Le SDK prend en charge ces modes de permission :

| Mode | Description | Comportement des outils |
| :- | :- | :- |
| `default` | Comportement de permission standard | Aucune auto-approbation liée au mode ; les appels qui nécessitent une approbation et ne correspondent à aucune règle allow déclenchent votre callback `canUseTool` |
| `dontAsk` | Refuser au lieu de demander | Tout appel qui demanderait une confirmation est refusé. Les appels approuvés par `allowed_tools` ou par des règles s'exécutent, de même que ceux qui ne nécessitent pas d'approbation en mode `default` ; les outils de connecteurs que [votre organisation a réglés sur `ask`](https://code.claude.com/docs/en/mcp#organization-controls-on-connector-tools) et les outils exigeant une interaction utilisateur sont refusés même si vous les avez pré-approuvés, tout comme les [lectures depuis des chemins réseau](https://code.claude.com/docs/en/permissions#network-paths) et les suppressions `rm`/`rmdir` visant un [chemin critique](https://code.claude.com/docs/en/permission-modes#critical-paths). `canUseTool` n'est jamais appelé |
| `acceptEdits` | Accepter automatiquement les modifications de fichiers | Les modifications de fichiers et les [opérations sur le système de fichiers](#mode-accept-edits-acceptedits) (`mkdir`, `rm`, `mv`, etc.) sont approuvées automatiquement |
| `bypassPermissions` | Contourner les vérifications de permission | Les outils s'exécutent sans demande de permission, sauf pour les [actions qu'aucun mode n'auto-approuve](https://code.claude.com/docs/en/permission-modes#actions-no-mode-auto-approves). À utiliser avec prudence |
| `plan` | Mode planification | Claude explore et planifie sans modifier vos fichiers source ; les modifications de fichiers ne sont jamais auto-approuvées et passent par votre callback `canUseTool` |
| `auto` | Approbations classées par un modèle | Un classifieur (modèle) examine des actions comme les commandes shell et les requêtes réseau, et autorise ou bloque chacune de celles qu'il examine. Voir [Auto mode](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) pour la disponibilité et l'ordre de décision |

> **Avertissement** : **Héritage par les sous-agents** : un sous-agent s'exécute dans le mode de permission de la session parente, sauf si vous définissez `permissionMode` dans son [`AgentDefinition`](https://code.claude.com/docs/en/agent-sdk/typescript#agentdefinition) **et** que la session parente est en mode `default`, `dontAsk` ou `plan`. Même dans ce cas, Claude Code n'applique jamais une valeur `"bypassPermissions"`, et n'applique une valeur `"auto"` que si le [mode auto est disponible](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) pour ce sous-agent. Un sous-agent ne s'exécute en mode `bypassPermissions` que si la session parente elle-même l'est. L'exception `bypassPermissions` exige Claude Code v2.1.267+.
>
> Les sous-agents peuvent avoir des prompts système différents et un comportement moins contraint que votre agent principal ; hériter de `bypassPermissions` leur accorde donc un accès système complet et autonome. Les [actions qu'aucun mode n'auto-approuve](https://code.claude.com/docs/en/permission-modes#actions-no-mode-auto-approves) restent applicables.

### Définir le mode de permission

Vous pouvez définir le mode une fois au démarrage d'une requête, ou le changer dynamiquement pendant que la session est active.

#### Onglet « Au moment de la requête »

Passez `permission_mode` (Python) ou `permissionMode` (TypeScript) à la création de la requête. Ce mode s'applique à toute la session, sauf changement dynamique.

**Python**

```python Python theme={null}
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions


async def main():
    async for message in query(
        prompt="Help me refactor this code",
        options=ClaudeAgentOptions(
            permission_mode="default",  # Définir le mode ici
        ),
    ):
        if hasattr(message, "result"):
            print(message.result)


asyncio.run(main())
```

**TypeScript**

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

async function main() {
  for await (const message of query({
    prompt: "Help me refactor this code",
    options: {
      permissionMode: "default" // Définir le mode ici
    }
  })) {
    if ("result" in message) {
      console.log(message.result);
    }
  }
}

main();
```

#### Onglet « Pendant le streaming »

Appelez `set_permission_mode()` (Python) ou `setPermissionMode()` (TypeScript) pour changer de mode en cours de session. Le nouveau mode prend effet immédiatement pour toutes les requêtes d'outils suivantes. Cela permet de commencer de façon restrictive puis d'assouplir les permissions à mesure que la confiance s'établit, par exemple en passant à `acceptEdits` après avoir examiné l'approche initiale de Claude.

**Python**

```python Python theme={null}
import asyncio
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions


async def main():
    async with ClaudeSDKClient(
        options=ClaudeAgentOptions(
            permission_mode="default",  # Démarrer en mode default
        )
    ) as client:
        await client.query("Help me refactor this code")

        # Changer de mode dynamiquement en cours de session
        await client.set_permission_mode("acceptEdits")

        # Traiter les messages avec le nouveau mode de permission
        async for message in client.receive_response():
            if hasattr(message, "result"):
                print(message.result)


asyncio.run(main())
```

**TypeScript**

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

async function main() {
  const q = query({
    prompt: "Help me refactor this code",
    options: {
      permissionMode: "default" // Démarrer en mode default
    }
  });

  // Changer de mode dynamiquement en cours de session
  await q.setPermissionMode("acceptEdits");

  // Traiter les messages avec le nouveau mode de permission
  for await (const message of q) {
    if ("result" in message) {
      console.log(message.result);
    }
  }
}

main();
```

### Détails des modes

#### Mode accept edits (`acceptEdits`)

Auto-approuve les opérations sur fichiers pour que Claude puisse modifier le code sans demander. Les autres outils (comme les commandes Bash qui ne sont pas des opérations sur le système de fichiers) exigent toujours les permissions normales.

**Opérations auto-approuvées :**

- Modifications de fichiers (outils Edit, Write)
- Commandes du système de fichiers : `mkdir`, `touch`, `rm`, `rmdir`, `mv`, `cp`, `sed`

Les deux ne s'appliquent qu'aux chemins situés dans le répertoire de travail ou dans `additionalDirectories`. En mode `acceptEdits`, Claude Code n'auto-approuve pas la requête quand Claude :

- travaille sur un chemin hors de ce périmètre ;
- écrit dans un chemin protégé ;
- supprime un [chemin critique](https://code.claude.com/docs/en/permission-modes#critical-paths) avec `rm` ou `rmdir`.

**À utiliser quand** : vous faites confiance aux modifications de Claude et voulez itérer plus vite, par exemple pendant du prototypage ou en travaillant dans un répertoire isolé.

#### Mode don't ask (`dontAsk`)

Convertit toute demande de permission en refus, sans appeler `canUseTool`. Les outils pré-approuvés par `allowed_tools`, par des règles allow de `settings.json` ou par un hook s'exécutent normalement, de même que les appels qui ne nécessitent pas d'approbation en mode `default`, comme les lectures de fichiers dans vos répertoires de travail et les appels à `Agent`.

Sont refusés même quand une règle allow correspond : les outils de connecteurs que [votre organisation a réglés sur `ask`](https://code.claude.com/docs/en/mcp#organization-controls-on-connector-tools), les outils qui exigent une interaction utilisateur, les [lectures depuis des chemins réseau](https://code.claude.com/docs/en/permissions#network-paths) et les suppressions `rm` et `rmdir` visant un [chemin critique](https://code.claude.com/docs/en/permission-modes#critical-paths). Un `allow` d'un hook `PreToolUse` ne débloque pas non plus une suppression de chemin critique ni une lecture depuis un chemin réseau.

**À utiliser quand** : vous voulez une surface d'outils fixe et explicite pour un agent headless, et préférez un refus net plutôt que de compter silencieusement sur l'absence de `canUseTool`.

#### Mode bypass permissions (`bypassPermissions`)

Auto-approuve les usages d'outils sans demander, sauf dans les cas listés dans l'avertissement ci-dessous. Les hooks s'exécutent toujours et peuvent bloquer des opérations si nécessaire. Sous Linux et macOS, Claude Code refuse de démarrer dans ce mode en tant que root ou sous `sudo` hors d'un [sandbox reconnu](https://code.claude.com/docs/en/permission-modes#skip-all-checks-with-bypasspermissions-mode), et la requête échoue avant le premier tour.

> **Avertissement** : à utiliser avec une extrême prudence. Claude dispose d'un accès complet au système dans ce mode. À n'utiliser que dans des environnements contrôlés où vous faites confiance à toutes les opérations possibles.
>
> `allowed_tools` ne restreint pas ce mode. Tous les outils sont approuvés, pas seulement ceux que vous avez listés. Ces contrôles s'appliquent encore :
>
> - Les règles deny, les règles `ask` explicites et les hooks sont évalués avant la vérification du mode et peuvent toujours bloquer un outil.
> - Les outils de connecteurs que [votre organisation a réglés sur `ask`](https://code.claude.com/docs/en/mcp#organization-controls-on-connector-tools), les outils qui exigent une interaction utilisateur et les suppressions `rm` et `rmdir` visant un [chemin critique](https://code.claude.com/docs/en/permission-modes#critical-paths) passent toujours à votre callback `canUseTool`.
> - Les [garde-fous de messagerie inter-sessions](https://code.claude.com/docs/en/permission-modes#skip-all-checks-with-bypasspermissions-mode) s'appliquent toujours.

#### Mode plan (`plan`)

Claude explore la base de code et produit un plan sans modifier vos fichiers source. Les outils en lecture seule s'exécutent comme en mode `default`.

Les modifications de fichiers ne sont jamais auto-approuvées en mode plan, même quand une règle allow correspond : elles passent par votre callback `canUseTool`. À partir de Claude Code v2.1.212, les commandes shell qui modifient des fichiers, comme `touch` et `rm`, atteignent votre callback `canUseTool` de la même manière.

Dans le SDK TypeScript, si vous définissez `allowDangerouslySkipPermissions: true` avec `permissionMode: 'plan'`, les modifications de fichiers et les commandes shell qui modifient des fichiers atteignent toujours votre callback `canUseTool`. Cette option vous permet de passer plus tard à `bypassPermissions` avec `setPermissionMode()`.

Claude peut utiliser `AskUserQuestion` pour clarifier les exigences avant de finaliser le plan. Voir [Handle approvals and user input](https://code.claude.com/docs/en/agent-sdk/user-input#handle-clarifying-questions) pour la gestion de ces demandes.

**À utiliser quand** : vous voulez que Claude propose des changements sans les exécuter, par exemple pendant une revue de code ou quand vous devez approuver les changements avant qu'ils ne soient faits.

## Ressources associées

Pour les autres étapes du flux d'évaluation des permissions :

- [Handle approvals and user input](https://code.claude.com/docs/en/agent-sdk/user-input) : demandes d'approbation interactives et questions de clarification.
- [Hooks guide](https://code.claude.com/docs/en/agent-sdk/hooks) : exécuter du code personnalisé à des moments clés du cycle de vie de l'agent.
- [Permission rules](https://code.claude.com/docs/en/settings-reference#permission-settings) : règles allow/deny déclaratives dans `settings.json`.
