# Référence de l'Agent SDK — TypeScript — partie 2/4

> Source : https://code.claude.com/docs/en/agent-sdk/typescript · [← Index](../31-index-documentation-officielle.md)

Cette partie couvre les types de messages renvoyés par `query()` (`SDKMessage` et toutes ses variantes : messages assistant, utilisateur, résultat, système, flux partiels, compaction, permissions, usage du contexte, rapport `/usage`, origine des messages, etc.).

## Types de messages (Message Types)

### `SDKMessage`

Type union de tous les messages possibles renvoyés par la requête.

```typescript theme={null}
type SDKMessage =
  | SDKAssistantMessage
  | SDKUserMessage
  | SDKUserMessageReplay
  | SDKResultMessage
  | SDKSystemMessage
  | SDKPartialAssistantMessage
  | SDKCompactBoundaryMessage
  | SDKStatusMessage
  | SDKLocalCommandOutputMessage
  | SDKHookStartedMessage
  | SDKHookProgressMessage
  | SDKHookResponseMessage
  | SDKPluginInstallMessage
  | SDKToolProgressMessage
  | SDKAuthStatusMessage
  | SDKTaskNotificationMessage
  | SDKTaskStartedMessage
  | SDKTaskProgressMessage
  | SDKTaskUpdatedMessage
  | SDKBackgroundTasksChangedMessage
  | SDKThinkingTokensMessage
  | SDKSessionStateChangedMessage
  | SDKWorkerShuttingDownMessage
  | SDKCommandsChangedMessage
  | SDKNotificationMessage
  | SDKFilesPersistedEvent
  | SDKToolUseSummaryMessage
  | SDKMemoryRecallMessage
  | SDKRateLimitEvent
  | SDKElicitationCompleteMessage
  | SDKPermissionDeniedMessage
  | SDKPromptSuggestionMessage
  | SDKAPIRetryMessage
  | SDKMirrorErrorMessage
  | SDKInformationalMessage
  | SDKConversationResetMessage;
```

### `SDKAssistantMessage`

Message de réponse de l'assistant.

```typescript theme={null}
type SDKAssistantMessage = {
  type: "assistant";
  uuid: UUID;
  session_id: string;
  message: BetaMessage; // Issu du SDK Anthropic
  parent_tool_use_id: string | null;
  error?: SDKAssistantMessageError;
  aborted?: true;
  agent_id?: string;
  timestamp?: string;
  context_usage?: SDKContextUsage;
  usage_report?: SDKUsageReport;
  user_message_uuid?: string;
  user_message_uuids?: string[];
  resume_reason?: string;
};
```

Le champ `message` est un [`BetaMessage`](https://platform.claude.com/docs/en/api/messages/create) issu du SDK Anthropic. Il contient des champs comme `id`, `content`, `model`, `stop_reason` et `usage`.

`SDKAssistantMessageError` vaut l'une des valeurs suivantes : `'authentication_failed'`, `'oauth_org_not_allowed'`, `'account_on_hold'`, `'billing_error'`, `'rate_limit'`, `'overloaded'`, `'invalid_request'`, `'model_not_found'`, `'server_error'`, `'max_output_tokens'`, `'cloud_credential_error'` ou `'unknown'`. Quatre de ces valeurs signifient plus que ce que leur nom indique :

* `'model_not_found'` : le modèle sélectionné n'existe pas ou n'est pas disponible pour votre compte ou votre déploiement.
* `'overloaded'` : l'API a renvoyé un 529 parce que le serveur est saturé, par opposition à `'rate_limit'`, qui est un 429 lié à votre quota.
* `'account_on_hold'` : [votre compte est suspendu](https://code.claude.com/docs/en/errors#your-account-is-on-hold).
* `'cloud_credential_error'` : Claude Code n'a pas pu obtenir d'identifiants AWS ou Google Cloud utilisables sur la machine où il s'exécute, donc aucune requête n'a atteint le fournisseur cloud. La cause habituelle est une connexion cloud expirée ou jamais terminée sur cette machine, mais un service d'identifiants brièvement injoignable renvoie la même valeur. Voir [Could not load AWS or Google Cloud credentials](https://code.claude.com/docs/en/errors#could-not-load-aws-or-google-cloud-credentials). Nécessite l'Agent SDK TypeScript v0.3.267+ (qui embarque Claude Code v2.1.267).

`aborted` vaut `true` lorsqu'une interruption ou un abandon a tronqué le message de l'assistant avant la fin du flux : le message n'a pas de `stop_reason` et le contenu peut s'arrêter au milieu d'un mot. Le champ est absent des messages terminés normalement. Nécessite l'Agent SDK v0.3.214+.

`agent_id` identifie le sous-agent qui a produit le message ; il est absent des messages du fil principal. Sa valeur est égale au `task_id` des événements [`task_started`](#sdktaskstartedmessage) et autres événements de tâche de ce sous-agent, et reste inchangée lorsque le sous-agent est [repris](https://code.claude.com/docs/en/agent-sdk/subagents#resume-subagents). Nécessite l'Agent SDK v0.3.292+.

Associez les messages d'un sous-agent à ses événements de tâche via `agent_id` plutôt qu'en appariant le `parent_tool_use_id` d'un message avec le `tool_use_id` d'un événement de tâche. Lorsqu'un appel d'outil reprend le sous-agent, les événements de tâche portent le `tool_use_id` de cet appel, alors que les messages conservent le `parent_tool_use_id` de l'appel d'outil qui a démarré le sous-agent à l'origine : les deux ne correspondent donc plus.

Claude Code renseigne `user_message_uuid` et `user_message_uuids` sur le premier message assistant du tour, dans les conditions décrites dans [`user_message_uuid`](#user_message_uuid). Lorsque le tour poursuit un tour interrompu par un redémarrage, les messages assistant qui portent ces champs portent aussi [`resume_reason`](#resume_reason).

`timestamp` est l'heure ISO 8601 à laquelle le contenu du message a fini d'être généré sur le processus qui l'a produit. La valeur provient de l'horloge de cette machine : utilisez-la uniquement pour l'affichage et ne triez pas les messages selon elle. Un même tour d'API peut produire plusieurs messages assistant partageant un `message.id`, chacun avec son propre `timestamp`. Si le champ est absent, rabattez-vous sur l'heure de réception du message.

`context_usage` est une copie structurée du rapport `/context`, typée [`SDKContextUsage`](#sdkcontextusage) ; nécessite l'Agent SDK v0.3.232+. Quand vous envoyez `/context` comme prompt, Claude Code livre le rapport sous forme d'un message assistant dont `message.content` contient le tableau Markdown, et attache `context_usage` à ce même message. Claude Code ne renseigne ce champ sur aucun autre message assistant, et les versions antérieures livrent le tableau `/context` sans lui : lisez donc la ventilation depuis le champ lorsqu'il est présent et rabattez-vous sur le texte Markdown sinon.

`usage_report` est une copie structurée du rapport `/usage`, typée [`SDKUsageReport`](#sdkusagereport) ; nécessite l'Agent SDK v0.3.273+. Quand vous envoyez `/usage` comme prompt, Claude Code livre le rapport sous forme d'un message assistant dont `message.content` contient le texte. Il n'attache `usage_report` à ce même message que si la session remplit toutes ces conditions :

* La session s'authentifie avec un identifiant claude.ai
* L'identifiant indique un type de forfait connu ou porte le scope `user:profile`
* Le compte n'est pas en facturation à l'usage

Un jeton `claude setup-token` passé via `CLAUDE_CODE_OAUTH_TOKEN` ne remplit pas ces conditions par défaut, car il ne porte que le scope `user:inference`. Les autres sessions, comme les sessions avec clé API, livrent le texte sans le champ, tout comme les versions antérieures. Lisez le rapport depuis le champ lorsqu'il est présent et rabattez-vous sur le texte sinon.

### `SDKUserMessage`

Message d'entrée utilisateur.

```typescript theme={null}
type SDKUserMessage = {
  type: "user";
  uuid?: UUID;
  session_id?: string;
  agent_id?: string;
  message: MessageParam; // Issu du SDK Anthropic
  pasted_content?: MessageParam["content"][];
  parent_tool_use_id: string | null;
  isSynthetic?: boolean;
  shouldQuery?: boolean;
  client_composed?: true;
  tool_use_result?: unknown;
  priority?: "now" | "next" | "later";
  origin?: SDKMessageOrigin;
  inline_pastes?: string[];
};
```

Renseignez `pasted_content` pour envoyer du contenu que l'utilisateur a collé dans votre interface de saisie plutôt que tapé, une entrée par collage, chacune étant une chaîne ou un tableau de blocs de contenu. Claude Code ajoute le texte de chaque entrée après le texte tapé, dans l'ordre, et peut envelopper chaque collage dans des balises `<pasted_content>`. Les blocs autres que texte sont ignorés : envoyez donc les images et documents dans `message.content`. Nécessite l'Agent SDK v0.3.277+.

Renseignez `inline_pastes` pour indiquer à Claude Code quelles parties de `message.content` l'utilisateur a collées plutôt que tapées, une chaîne par collage. Le texte du prompt reste là où l'utilisateur l'a placé. Claude Code peut envelopper chaque collage listé dans des balises `<pasted_content>` à son emplacement, afin que Claude distingue le contenu collé des propres mots de l'utilisateur. Seuls les collages situés dans le dernier bloc de texte du prompt sont enveloppés. Nécessite l'Agent SDK TypeScript v0.3.280+.

Chaque champ de collage a une limite de taille :

* `pasted_content` : si les entrées plus les blocs de contenu qu'elles contiennent dépassent 1 000, Claude Code ignore le champ entier.
* `inline_pastes` : Claude Code utilise les 100 premières entrées non vides et ignore le reste.

Renseignez `shouldQuery`, `client_composed` ou `priority` pour modifier la manière dont Claude Code traite un message que vous envoyez :

* `shouldQuery` : mettez-le à `false` pour ajouter le message à la transcription sans déclencher de tour de l'assistant. Le message est conservé et fusionné dans le prochain message utilisateur qui déclenche un tour. Utilisez-le pour injecter du contexte, comme la sortie d'une commande exécutée hors bande, sans dépenser un appel au modèle.
* `client_composed` : mettez-le à `true` pour que Claude Code livre le texte du message tel quel. Claude Code ne développe alors pas les mentions `@path` ni [`@server:resource`](https://code.claude.com/docs/en/mcp#use-mcp-resources), et n'exécute pas comme commande un texte commençant par `/`. Tant que l'option [`verbatimPrompts`](#options) est activée, le SDK renseigne ce champ sur chaque message. Nécessite l'Agent SDK TypeScript v0.3.280+ et Claude Code v2.1.248+.
* `priority` : contrôle le moment où un message envoyé pendant un tour en cours atteint Claude :
  * `'next'`, ou absence de champ `priority` : Claude lit le message dans le même tour, dès que les appels d'outils en cours se terminent. Si le tour se termine avant, le message démarre le tour suivant.
  * `'later'` : Claude Code retient le message jusqu'à la fin du tour et l'envoie comme nouveau tour.
  * `'now'` avec [`origin: { kind: "human" }`](#sdkmessageorigin) : sur Claude Code v2.1.286+, le travail pouvant continuer en arrière-plan y est déplacé, et Claude lit le message dans le même tour. Le travail déplaçable comprend les commandes shell, les sous-agents et les appels d'outils MCP ; sur v2.1.287+, il comprend aussi les appels WebFetch et WebSearch. Lorsque Claude est seulement en train d'écrire une réponse, ou que le travail en cours ne peut pas être déplacé, Claude Code interrompt plutôt le tour et Claude lit le message ensuite.
  * `'now'` sans cette origine : Claude Code interrompt le tour et Claude lit le message ensuite.

Ce message, envoyé pendant un tour en cours, demande à Claude de changer de cap sans perdre une commande shell encore en cours d'exécution :

```typescript theme={null}
const message: SDKUserMessage = {
  type: "user",
  message: { role: "user", content: "Skip the integration tests and summarize what you have so far" },
  parent_tool_use_id: null,
  priority: "now",
  origin: { kind: "human" },
};
```

Un message utilisateur produit par un sous-agent, comme le `tool_result` d'un de ses propres appels d'outils, porte `agent_id`. Voir [`SDKAssistantMessage`](#sdkassistantmessage), qui définit ce champ et sa version requise.

Sur un message portant un bloc `tool_result`, `tool_use_result` est l'objet de sortie structuré de l'outil plutôt que le texte envoyé au modèle. Sa forme dépend de l'outil nommé par le bloc `tool_use` correspondant, d'où le type `unknown` ; les formes intégrées sont listées dans [Types de sortie des outils](#tool-output-types). Ces résultats demandent un traitement au-delà de leur forme listée :

* L'outil `Agent` : `tool_use_result` est un [`AgentOutput`](#agent-2). Faites le rendu à partir de lui plutôt que d'analyser le texte du `tool_result`. Le `content` d'un résultat `completed` contient le rapport du sous-agent ou, pour un sous-agent dont le rapport passe par un appel d'outil `SubagentHandback`, une courte note sur cette remise à la place du rapport. En [mode auto](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) sur Claude Code v2.1.271+, chaque sous-agent produisant un résultat `completed` fait son rapport de cette façon, sauf s'il s'agit d'un [fork](https://code.claude.com/docs/en/sub-agents#fork-the-current-conversation), et Claude reçoit le rapport comme un message distinct du sous-agent.
* Un appel WebFetch ou WebSearch que Claude Code a déplacé en arrière-plan pour livrer un message `'now'` : le message utilisateur portant le `tool_result` de cet appel a `tool_use_result` égal à `{ detachedToolCall: true }`. L'appel est toujours en cours, et Claude reçoit son résultat lorsqu'il se termine. Aucun second `tool_result` pour ce `tool_use_id` ne suit : si votre application affiche une ligne par appel d'outil, marquez cette ligne comme déplacée en arrière-plan à l'arrivée de ce message. Nécessite Claude Code v2.1.287+.
* Un outil MCP dont le résultat contient des blocs `resource_link` : `tool_use_result` est un objet doté d'un tableau `resourceLinks` d'entrées [`SDKMcpResourceLink`](#sdkmcpresourcelink). Claude reçoit chaque lien comme une ligne de texte dans le bloc `tool_result` : lisez donc `resourceLinks` pour afficher les fichiers renvoyés par le serveur au lieu d'analyser ce texte. Claude Code omet `resourceLinks` quand le résultat n'a pas de lien et sur les résultats des sous-agents, conserve au plus 50 liens par résultat, et cesse d'ajouter des liens une fois que le tableau atteint 64 Kio de JSON sérialisé. `resourceLinks` nécessite l'Agent SDK v0.3.257+.
* Un outil MCP qui renvoie [`structuredContent`](#calltoolresult) : `tool_use_result` est un objet dont le membre `structuredContent` contient ce que le serveur a envoyé et dont le membre `content` contient la valeur [`McpOutput`](#mcpoutput). Les résultats des sous-agents ne portent pas `structuredContent`.
* Un outil MCP dont le `structuredContent` sérialisé dépasse 1 048 576 caractères de JSON : Claude Code retire `structuredContent` de `tool_use_result` et place `structuredContentOmitted: true` à la place, afin que votre application distingue un objet supprimé d'un outil qui n'en a envoyé aucun. Les autres membres, comme `content` et `resourceLinks`, restent, et ce que Claude reçoit ne change pas. Claude Code v2.1.287+ applique ce plafond. Deux types d'outils diffèrent :
  * Les outils des [serveurs SDK in-process](https://code.claude.com/docs/en/agent-sdk/custom-tools) sont exemptés et livrent l'objet en entier.
  * Un outil dont l'entrée `tools/list` déclare une [ressource MCP Apps `ui://`](#mcpserverstatus) a une limite de 8 388 608 caractères sur Claude Code v2.1.295+ ; les versions antérieures à v2.1.295 l'exemptent.

### `SDKUserMessageReplay`

Message utilisateur rejoué, avec UUID obligatoire.

```typescript theme={null}
type SDKUserMessageReplay = {
  type: "user";
  uuid: UUID;
  session_id: string;
  message: MessageParam;
  parent_tool_use_id: string | null;
  isSynthetic?: boolean;
  client_composed?: true;
  tool_use_result?: unknown;
  origin?: SDKMessageOrigin;
  isReplay: true;
};
```

Un tour utilisateur injecté depuis l'extérieur de la session, c'est-à-dire dont le kind d'[`origin`](#sdkmessageorigin) est `peer` ou `channel`, arrive dans le flux sous forme de rejeu (replay), qu'il ait été livré pendant un tour actif ou qu'il ait démarré un nouveau tour alors que la session était inactive. Avant v2.1.207, un tour injecté livré pendant que la session était inactive ne produisait aucun message dans le flux et n'apparaissait qu'en relisant la transcription.

### `SDKResultMessage`

Message de résultat final.

```typescript theme={null}
type SDKResultMessage =
  | {
      type: "result";
      subtype: "success";
      uuid: UUID;
      session_id: string;
      duration_ms: number;
      duration_api_ms: number;
      is_error: boolean;
      api_error_status?: number | null;
      num_turns: number;
      result: string;
      stop_reason: string | null;
      ttft_ms?: number;
      ttft_stream_ms?: number;
      user_message_uuid?: string;
      user_message_uuids?: string[];
      resume_reason?: string;
      local_command?: string;
      request_sent_wall_ms?: number;
      first_content_frame_ms?: number;
      first_stream_post_ms?: number;
      first_stream_post_ack_ms?: number;
      first_stream_post_queue_wait_ms?: number;
      first_stream_post_queued_behind?: "durable_post" | "ephemeral_post" | "retry_backoff" | "hold" | "none";
      first_stream_post_wall_ms?: number;
      first_text_post_ms?: number;
      first_text_post_queue_wait_ms?: number;
      first_text_post_queued_behind?: "durable_post" | "ephemeral_post" | "retry_backoff" | "hold" | "none";
      first_text_post_wall_ms?: number;
      total_cost_usd: number;
      usage: NonNullableUsage;
      modelUsage: { [modelName: string]: ModelUsage };
      permission_denials: SDKPermissionDenial[];
      queued_turn_count?: number;
      structured_output?: unknown;
      deferred_tool_use?: { id: string; name: string; input: Record<string, unknown> };
      terminal_reason?: TerminalReason;
      result_index?: number;
      fast_mode_state?: FastModeState;
      fast_mode_disabled_reason?: FastModeDisabledReason;
      origin?: SDKMessageOrigin;
    }
  | {
      type: "result";
      subtype:
        | "error_max_turns"
        | "error_during_execution"
        | "error_max_budget_usd"
        | "error_max_structured_output_retries";
      uuid: UUID;
      session_id: string;
      duration_ms: number;
      duration_api_ms: number;
      is_error: boolean;
      num_turns: number;
      stop_reason: string | null;
      total_cost_usd: number;
      usage: NonNullableUsage;
      modelUsage: { [modelName: string]: ModelUsage };
      permission_denials: SDKPermissionDenial[];
      queued_turn_count?: number;
      errors: string[];
      startup_failure_reason?: SDKStartupFailureReason;
      user_message_uuid?: string;
      user_message_uuids?: string[];
      resume_reason?: string;
      terminal_reason?: TerminalReason;
      result_index?: number;
      fast_mode_state?: FastModeState;
      fast_mode_disabled_reason?: FastModeDisabledReason;
      origin?: SDKMessageOrigin;
    };
```

Plusieurs champs du résultat apportent des détails de diagnostic au-delà de `subtype` :

* `api_error_status` : le code de statut HTTP de l'erreur d'API qui a terminé la conversation. Absent ou `null` lorsque le tour s'est terminé sans erreur d'API.
* `ttft_ms` : temps jusqu'au premier token, en millisecondes, mesuré à l'arrivée du premier message assistant complet. Présent uniquement dans la branche succès.
* `ttft_stream_ms` : temps en millisecondes jusqu'au premier événement de flux `message_start`, quand le flux de réponse s'ouvre. Inférieur à `ttft_ms` ; l'écart entre les deux est le temps passé à streamer le premier message. Présent uniquement dans la branche succès.
* `user_message_uuid` : l'`uuid` du message que vous avez envoyé et auquel ce tour a répondu. Voir [`user_message_uuid`](#user_message_uuid) pour savoir quels résultats le portent.
* `user_message_uuids` : les `uuid` de tous les messages que vous avez envoyés et auxquels Claude Code a répondu dans ce tour. Voir [`user_message_uuids`](#user_message_uuids).
* `resume_reason` : la raison pour laquelle ce tour poursuit un tour interrompu par un redémarrage. Présent dans les deux branches. Voir [`resume_reason`](#resume_reason).
* `local_command` : le nom de la commande que le tour a déclenchée, sur le résultat de succès d'un tour qu'une commande a accompli sans entrer dans la boucle de l'agent, comme `/compact`. Le nom est ramené en lettres minuscules et underscores : `/reload-plugins` donne `reload_plugins`. Une commande fournie par un serveur MCP, ainsi que la commande intégrée `/mcp`, donnent `mcp`. Une commande que vous avez définie vous-même donne `custom`. Les arguments ne sont jamais inclus. Absent de tout tour entré dans la boucle de l'agent et des envois n'ayant exécuté aucune commande. Nécessite l'Agent SDK v0.3.268+.
* `request_sent_wall_ms` : horodatage epoch en millisecondes auquel Claude Code a envoyé la requête d'API, pour des jointures avec des horodatages côté serveur. Présent uniquement en même temps que [`user_message_uuid`](#user_message_uuid), sur un résultat de succès avec `is_error` à false dont le tour a envoyé une requête d'API.
* `first_content_frame_ms` : temps en millisecondes jusqu'au premier événement de flux `content_block_start` ou `content_block_delta`, les blocs de réflexion (thinking) comptant comme contenu. Présent uniquement dans la branche succès, quand `is_error` vaut false. Nécessite l'Agent SDK v0.3.260+.
* `first_stream_post_ms`, `first_stream_post_ack_ms`, `first_stream_post_wall_ms` : mesures de temps de l'envoi du premier événement de flux du tour. Claude Code ne les enregistre que dans les sessions qu'il streame vers claude.ai, comme les [sessions cloud](https://code.claude.com/docs/en/claude-code-on-the-web), et les résultats émis par `query()` ne les portent pas. Nécessite l'Agent SDK v0.3.260+.
* `usage` : boucle principale de l'agent uniquement. Exclut les appels de modèle des sous-agents et auxiliaires, et est par tour dans les sessions en entrée streaming. Préférez `modelUsage` pour la comptabilité des tokens et des coûts.
* `modelUsage` : totaux par modèle de tous les appels de modèle effectués via le pipeline de requête pendant cet appel `query()`, y compris la boucle principale, les sous-agents et les appels internes comme la compaction et les agents Workflow. Les appels auxiliaires hors de ce pipeline, comme le classifieur de permissions et les requêtes de comptage de tokens, sont exclus. Un appel qui reprend une session compte aussi les [totaux par modèle restaurés depuis les appels antérieurs de la session](https://code.claude.com/docs/en/agent-sdk/cost-tracking#accumulate-costs-across-multiple-calls). Dans les sessions en entrée streaming, les totaux sont cumulés sur les tours : lisez le dernier résultat plutôt que d'additionner les résultats. Voir [Track costs in streaming input mode](https://code.claude.com/docs/en/agent-sdk/cost-tracking#track-costs-in-streaming-input-mode) pour les remises à zéro et [Recover totals after a session crash](https://code.claude.com/docs/en/agent-sdk/cost-tracking#recover-totals-after-a-session-crash) pour les résultats à zéro.
* `total_cost_usd` : coût estimé cumulé en USD, couvrant les mêmes appels que `modelUsage` et remis à zéro aux mêmes moments. Un appel qui reprend une session compte aussi les [totaux restaurés depuis les appels antérieurs de la session](https://code.claude.com/docs/en/agent-sdk/cost-tracking#accumulate-costs-across-multiple-calls). C'est une estimation, pas un relevé de facturation. Voir [Track cost and usage](https://code.claude.com/docs/en/agent-sdk/cost-tracking) pour les réserves sur la précision.
* `queued_turn_count` : le nombre de messages envoyés avec `origin: { kind: "human" }` encore en attente au moment où Claude Code a produit le résultat. Voir [`queued_turn_count`](#queued_turn_count) pour ce que signifient `0` et un champ absent.
* `result_index` : la position de ce résultat dans l'ordre de livraison de l'exécution, en comptant à partir de 0 sur tous les résultats écrits par le processus. Présent dans les deux branches. Un résultat dont l'écriture échoue consomme quand même son numéro : un trou dans la séquence signifie donc qu'un résultat a été perdu. Nécessite l'Agent SDK v0.3.268+.
* `startup_failure_reason` : la raison pour laquelle Claude Code a refusé de démarrer, sur le résultat `error_during_execution` qu'il écrit avant de quitter lors d'un échec de démarrage connu. Voir [`startup_failure_reason`](#startup_failure_reason) pour les valeurs et les échecs qui le portent. Nécessite l'Agent SDK v0.3.274+.
* `terminal_reason` : la raison de la fin de la boucle. L'une des valeurs `"completed"`, `"max_turns"`, `"tool_deferred"`, `"aborted_streaming"`, `"aborted_tools"`, `"hook_stopped"`, `"stop_hook_prevented"`, `"background_requested"`, `"blocking_limit"`, `"rapid_refill_breaker"`, `"prompt_too_long"`, `"image_error"`, `"model_error"`, `"api_error"`, `"malformed_tool_use_exhausted"`, `"budget_exhausted"`, `"structured_output_retry_exhausted"`, `"tool_deferred_unavailable"` ou `"turn_setup_failed"`.
* `fast_mode_state` : l'une des valeurs `"on"`, `"off"` ou `"cooldown"`.
* `fast_mode_disabled_reason` : la raison pour laquelle le [mode rapide (fast mode)](https://code.claude.com/docs/en/fast-mode) n'est pas disponible actuellement. Absent quand rien ne bloque le mode rapide, même si une requête peut tout de même s'exécuter à vitesse standard. Pendant la période de refroidissement (cooldown) qui suit une limite de débit du mode rapide, Claude Code indique `fast_mode_state: "cooldown"` sans code de raison et réactive le mode rapide à l'expiration du refroidissement. Nécessite Claude Code v2.1.219+.

Utilisez le code de raison pour expliquer dans votre propre interface pourquoi le mode rapide est désactivé, au lieu de recalculer la disponibilité. Chaque code nomme la vérification qui a bloqué le mode rapide :

| Code de raison | Signification |
| - | - |
| `free` | Le compte n'a pas l'abonnement payant ou les crédits d'usage requis par le mode rapide |
| `preference` | L'organisation a désactivé le mode rapide |
| `extra_usage_disabled` | Les crédits d'usage sont désactivés pour le compte |
| `network_error` | La [vérification de disponibilité](https://code.claude.com/docs/en/fast-mode#use-fast-mode-behind-proxies-and-llm-gateways) n'a pas pu joindre `api.anthropic.com` |
| `unknown` | Claude Code n'a pas pu déterminer la disponibilité |
| `not_first_party` | La session utilise un fournisseur autre que l'API Anthropic |
| `disabled_by_env` | [`CLAUDE_CODE_DISABLE_FAST_MODE`](https://code.claude.com/docs/en/env-vars) est défini |
| `model_not_allowed` | Le modèle Opus du mode rapide ne figure pas dans la liste d'autorisation [`availableModels`](https://code.claude.com/docs/en/model-config#restrict-model-selection) de l'organisation |
| `sdk_opt_in_required` | La session n'a pas activé (opt-in) le mode rapide : passez `fastMode: true` dans l'option [`settings`](#options) ou via [`applyFlagSettings()`](#applyflagsettings) |
| `pending` | La vérification de disponibilité n'est pas encore terminée |

La même paire de champs apparaît sur [`SDKSystemMessage`](#sdksystemmessage) et sur [`SDKControlInitializeResponse`](#sdkcontrolinitializeresponse), ce qui permet de lire l'état du mode rapide avant le premier tour.

Le champ `origin` transmet l'[`SDKMessageOrigin`](#sdkmessageorigin) du message utilisateur qui a déclenché ce résultat. Lorsque le SDK injecte un tour de suivi synthétique, par exemple pour une tâche d'arrière-plan terminée, le `SDKResultMessage` résultant porte `origin: { kind: "task-notification" }`. Les routines dont le déclencheur s'est activé et les messages vérifiés par le serveur provenant de vos autres sessions arrivent aussi avec ce kind, chacun avec le `subkind` décrit dans [Sous-types (subkinds) de task-notification](#task-notification-subkinds). Vérifiez `kind` pour distinguer les résultats qui répondent à votre prompt des suivis injectés avant de les router ou de les supprimer. Si votre application [déclare des exécutions planifiées](#declare-a-scheduled-run), leurs résultats portent aussi `kind: "task-notification"` : ne supprimez donc pas sur la seule base de `kind`.

Lorsque plusieurs achèvements de tâches d'arrière-plan sont en file ensemble, Claude Code peut y répondre en un seul tour plutôt qu'en un tour chacun. Chaque achèvement produit tout de même son propre résultat avec cette origine. Tous les achèvements traités ensemble, sauf le dernier, produisent des résultats vides avec `num_turns: 0`, dans l'ordre, et le résultat du dernier porte le tour qui répond à tous.

Le champ est absent des résultats émis avant tout tour utilisateur, comme les erreurs de démarrage.

Lorsqu'un hook `PreToolUse` renvoie `permissionDecision: "defer"`, le résultat a `stop_reason: "tool_deferred"` et `deferred_tool_use` porte l'`id`, le `name` et l'`input` de l'outil en attente. Lisez ce champ pour présenter la demande dans votre propre interface, puis reprenez avec le même `session_id` pour continuer. Voir [Defer a tool call for later](https://code.claude.com/docs/en/hooks#defer-a-tool-call-for-later) pour l'aller-retour complet.

#### `user_message_uuid`

L'`uuid` du [`SDKUserMessage`](#sdkusermessage) auquel le tour répond, renvoyé en écho pour que vous puissiez associer la réponse de Claude Code au message envoyé. Claude Code ne renvoie un `uuid` que si vous en avez défini un sur le message. Le champ est facultatif sur `SDKUserMessage`, et un prompt sous forme de chaîne passé à `query()` n'en porte aucun.

Le message auquel un tour répond dépend de la façon dont le tour a démarré :

* **Un message ordinaire que vous avez envoyé**, c'est-à-dire sans `isSynthetic: true` : le tour répond à ce message pendant toute son exécution. Quand vous envoyez plusieurs messages rapprochés, Claude Code peut les fusionner en un seul tour, et le champ ne porte alors que l'`uuid` du dernier message. Pour associer la réponse à n'importe lequel des messages fusionnés, utilisez [`user_message_uuids`](#user_message_uuids).
* **Un message envoyé avec `isSynthetic: true`** : le tour répond d'abord à ce message. Si Claude Code récupère un de vos messages ordinaires entre deux appels d'outils, le tour répond ensuite au message récupéré. Le renvoi de l'`uuid` d'un message synthétique nécessite l'Agent SDK v0.3.265+ ; les versions antérieures ne renvoient rien sur les tours synthétiques.
* **Le prompt que Claude Code génère pour poursuivre un tour interrompu sous [`CLAUDE_CODE_RESUME_INTERRUPTED_TURN`](https://code.claude.com/docs/en/env-vars)** : lorsque le dernier prompt du tour interrompu est un message ordinaire que vous avez envoyé, qu'il ait ouvert le tour ou que Claude Code l'ait récupéré pendant le tour, le tour poursuivi répond d'abord à ce message. [`resume_reason`](#resume_reason) distingue les frames du tour poursuivi de celles de la tentative interrompue. Lorsque le dernier prompt n'est pas un de vos messages ordinaires, le tour poursuivi ne répond d'abord à aucun de vos messages. Si Claude Code récupère un de vos messages ordinaires entre deux appels d'outils, le tour répond ensuite au message récupéré. Le renvoi du prompt du tour interrompu nécessite l'Agent SDK v0.3.268+.
* **Tout autre prompt généré par Claude Code lui-même** : le tour ne répond d'abord à aucun de vos messages et ses frames ne portent aucun écho. Si Claude Code récupère un de vos messages ordinaires entre deux appels d'outils, le tour répond ensuite à ce message. L'écho de récupération nécessite l'Agent SDK v0.3.265+ ; les versions antérieures ne renvoient rien sur ces tours.

Claude Code renvoie l'`uuid` du message traité sur trois types de frames :

* **Le résultat** : chaque résultat d'un tour ayant répondu à un message que vous avez envoyé. Tous ces résultats le portent sur l'Agent SDK v0.3.265+. Avant v0.3.265, le résultat de succès d'un tour démarré par un message ordinaire ne le portait pas lorsque le tour n'envoyait aucune requête d'API ou se terminait par un appel d'outil différé. Avant v0.3.246, les résultats d'erreur ne le portaient pas non plus, et avant v0.3.216 aucun résultat ne le portait.
* **La première réponse du tour** : le premier [message assistant](#sdkassistantmessage), et avec `includePartialMessages` aussi le premier [événement de flux](#sdkpartialassistantmessage) dont `event.type` n'est pas `ping`, pour que vous puissiez lier la réponse avant l'arrivée du résultat. L'écho sur la première réponse nécessite l'Agent SDK v0.3.246+. Avant v0.3.269, avec `includePartialMessages`, Claude Code ne le renseignait que sur ce premier événement de flux, ou sur le premier message assistant lorsque le tour ne streamait rien. Lorsque le message auquel répond le tour change en cours de tour, la première réponse après le changement porte aussi le champ, sur l'Agent SDK v0.3.265+ ; les versions antérieures le renseignaient sur une seule frame de réponse par tour.
* **Chaque frame [`thinking_tokens`](#sdkthinkingtokensmessage) du tour** : pour que vous puissiez attribuer la progression de la réflexion au message envoyé sans attendre la première réponse du tour. Nécessite l'Agent SDK v0.3.260+.

Claude Code omet le champ dans ces cas :

* Les frames de réponse autres que ces premières réponses
* Les frames de sous-agents
* Les tours qui ne répondent à aucun de vos messages, ou qui répondent à un message envoyé sans `uuid`
* Les résultats qui ne répondent à aucun message envoyé, comme le résultat à zéro après le plantage d'un processus worker

#### `user_message_uuids`

Les `uuid` de tous les messages que vous avez envoyés et auxquels Claude Code a répondu dans ce tour. Quand vous envoyez plusieurs messages rapprochés, Claude Code peut les fusionner en un seul tour, et `user_message_uuid` ne nomme alors que le dernier. Pour associer la réponse à n'importe lequel des messages fusionnés, cherchez l'`uuid` de ce message n'importe où dans cette liste. Nécessite l'Agent SDK v0.3.259+.

Claude Code renseigne la liste en même temps que `user_message_uuid` sur chaque frame de réponse qui porte ce champ et sur le résultat. Pour l'ensemble complet des frames du tour qui renvoient l'`uuid` du message traité, et la version requise pour chacune, voir [`user_message_uuid`](#user_message_uuid). La liste contient toujours `user_message_uuid` et compte au plus 64 entrées.

Lorsque Claude Code récupère un message ordinaire envoyé pendant qu'un tour était en cours, il ajoute l'`uuid` de ce message à la liste du résultat.

Lorsqu'une première réponse ou un résultat porte `user_message_uuid` sans la liste, il provient d'une version antérieure de Claude Code : rabattez-vous alors sur le champ unique.

#### `resume_reason`

La raison pour laquelle ce tour poursuit un tour interrompu par un redémarrage. Claude Code renseigne ce champ sur un tour qui poursuit un tour interrompu sous [`CLAUDE_CODE_RESUME_INTERRUPTED_TURN`](https://code.claude.com/docs/en/env-vars), pour que vous puissiez distinguer la réponse et le résultat du tour poursuivi de ceux de la tentative interrompue. Nécessite l'Agent SDK v0.3.268+.

Claude Code renseigne le champ sur deux types de frames :

* **Le résultat du tour poursuivi** : dans la branche succès comme dans la branche erreur, que le résultat porte ou non `user_message_uuid`.
* **Les frames de réponse du tour poursuivi** : celles qui portent [`user_message_uuid`](#user_message_uuid).

La valeur est un court jeton en minuscules, comme `interrupted_turn`.

#### `queued_turn_count`

Le nombre de messages envoyés avec [`origin: { kind: "human" }`](#sdkmessageorigin) encore en attente dans la file de commandes au moment où Claude Code a produit le résultat. Nécessite l'Agent SDK v0.3.242+.

Ce que signifient `0` et un champ absent :

* **`0`** : Claude Code ne compte pas les messages envoyés sans cette `origin`, ni les notifications de tâches : un tour peut donc encore suivre.
* **Absent** : le résultat final que Claude Code émet après un plantage ou une erreur fatale au démarrage omet le champ, et [peut porter des totaux à zéro](https://code.claude.com/docs/en/agent-sdk/cost-tracking#recover-totals-after-a-session-crash).

#### `startup_failure_reason`

La raison pour laquelle Claude Code a refusé de démarrer, afin que votre application propose le correctif plutôt qu'une nouvelle tentative. Claude Code le renseigne sur le résultat `error_during_execution` qu'il écrit avant de quitter lors d'un échec de démarrage connu. Ce résultat porte des totaux à zéro, et son tableau `errors` contient le même texte que stderr. Le champ est absent de tout autre résultat. Nécessite l'Agent SDK v0.3.274+.

Définissez `CLAUDE_CODE_STARTUP_FAILURE_RESULTS` à `1` dans [`env`](#options) pour recevoir ce résultat pour chaque valeur de `SDKStartupFailureReason`. Sans cette variable, Claude Code n'écrit le résultat que pour les échecs suivants ; les autres se terminent par une sortie sur stderr, un code de sortie non nul et aucun message de résultat :

* Une reprise que Claude Code arrête parce qu'il [ne peut pas ramener la session dans son worktree](https://code.claude.com/docs/en/worktrees#the-session-resumes-outside-its-worktree), avec `worktree_unverified` ou `worktree_resume_refused`. Cette section indique quelle erreur porte quelle valeur.
* Un [`continue`](#options) refusé d'une conversation détenue par une session d'arrière-plan, avec `session_held_by_background`. Pour un [`resume`](#options) refusé d'une telle conversation, Claude Code n'écrit le résultat que si la variable est définie.

```typescript theme={null}
type SDKStartupFailureReason =
  | "org_pin_api_key_conflict"
  | "provider_not_allowed"
  | "org_verify_failed"
  | "org_pin_mismatch"
  | "managed_settings_invalid"
  | "remote_settings_required_unavailable"
  | "gateway_signin_required"
  | "gateway_access_denied"
  | "proxy_invalid"
  | "temp_dir_unusable"
  | "cwd_unavailable"
  | "shell_tool_missing"
  | "session_held_by_background"
  | "worktree_resume_refused"
  | "worktree_unverified"
  | "cli_version_too_old"
  | "bypass_root"
  | "org_config_required_unavailable"
  | "org_config_refused";
```

Chaque valeur désigne un refus :

| Valeur | Ce qui a arrêté la session |
| :- | :- |
| `org_pin_api_key_conflict` | Les réglages gérés [exigent une connexion first-party ou via le Cloud gateway](https://code.claude.com/docs/en/authentication#restrict-login-to-your-organization), alors qu'une clé API Anthropic, un jeton d'authentification ou un `apiKeyHelper` est configuré à la place |
| `provider_not_allowed` | Les réglages gérés [listent les fournisseurs d'API que cette machine peut utiliser](https://code.claude.com/docs/en/settings-reference#allowedproviders), et la session est configurée pour un fournisseur non listé, ou pour un endpoint que les réglages n'épinglent pas. Nécessite Claude Code v2.1.285+ |
| `org_verify_failed` | L'organisation de la connexion n'a pas pu être vérifiée par rapport à l'épinglage, par exemple à cause d'une panne réseau ou d'un jeton révoqué |
| `org_pin_mismatch` | La connexion appartient à une organisation que l'épinglage n'autorise pas |
| `managed_settings_invalid` | Les réglages de politique gérés n'ont pas pu être lus, l'épinglage ne nomme aucune organisation, ou les [restrictions de modèles gérées](https://code.claude.com/docs/en/errors#managed-settings-block-the-default-model) ne laissent aucun modèle autorisé pour l'option Default |
| `remote_settings_required_unavailable` | Les réglages gérés exigés par l'organisation n'ont pas pu être chargés |
| `gateway_signin_required` | Le [Cloud gateway](https://code.claude.com/docs/en/claude-apps-gateway) a mis fin à cette connexion |
| `gateway_access_denied` | La requête de réglages gérés vers le Cloud gateway a renvoyé un 403, cas traité dans le [tableau de dépannage](https://code.claude.com/docs/en/claude-apps-gateway-deploy#troubleshooting) du gateway |
| `proxy_invalid` | Un réglage de proxy n'est pas une URL complète |
| `temp_dir_unusable` | Le répertoire temporaire propre à l'utilisateur n'est pas sûr ou n'a pas pu être créé |
| `cwd_unavailable` | Le répertoire de travail a été supprimé, déplacé ou ne peut pas être lu |
| `shell_tool_missing` | Sous Windows, aucun outil shell n'est disponible : Git Bash est absent, et PowerShell est absent ou désactivé via `CLAUDE_CODE_USE_POWERSHELL_TOOL` |
| `session_held_by_background` | La conversation à reprendre ou continuer s'exécute en tant que [session d'arrière-plan](https://code.claude.com/docs/en/agent-view) |
| `worktree_resume_refused` | Le worktree de la session a échoué à ses contrôles de sécurité, ou la reprise a été lancée depuis l'intérieur de celui-ci. `errors` indique si relancer la même reprise continue sans le worktree |
| `worktree_unverified` | Le worktree de la session n'a pas pu être vérifié pour l'instant, et une nouvelle tentative peut réussir |
| `cli_version_too_old` | Cette version de Claude Code est inférieure au minimum exigé par Anthropic |
| `bypass_root` | Le mode de contournement des permissions (bypass permissions) a été demandé alors que le processus s'exécute en root |
| `org_config_required_unavailable` | La session a besoin des politiques et réglages gérés de l'organisation avant de pouvoir démarrer, et ils n'ont pas pu être chargés, par exemple à cause d'une panne réseau ou d'une erreur serveur Anthropic. Nécessite l'Agent SDK v0.3.293+ |
| `org_config_refused` | Anthropic a refusé de fournir les politiques et réglages gérés de l'organisation pour cette connexion, par exemple parce que la connexion a expiré ou a été révoquée, ou que l'organisation n'autorise pas Claude Code pour ce compte. Nécessite l'Agent SDK v0.3.293+ |

<!-- SUITE -->
