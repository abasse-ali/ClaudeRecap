# Utiliser l'API des mods

> Source : https://code.claude.com/docs/en/plugins/mods/api · [← Index](../31-index-documentation-officielle.md)

*Appeler l'API des mods depuis un mod Claude Code pour ajouter des commandes et des outils, appeler un modèle, exécuter du travail sur minuterie, envoyer des messages à d'autres sessions, et accéder aux fichiers et au réseau.*

L'API des mods est l'ensemble des méthodes qu'un mod appelle pour agir : ajouter des commandes et des outils, appeler un modèle, exécuter du travail entre les événements, et accéder au système de fichiers, aux processus et au réseau. Chaque hook la reçoit comme premier argument, `$`, avec les méthodes regroupées dans des espaces de noms comme `$.ui` et `$.fs`. Les [événements](https://code.claude.com/docs/en/plugins/mods/events) décident quand un hook s'exécute, et l'API des mods est ce que le hook appelle une fois lancé.

Construisez votre [premier mod](https://code.claude.com/docs/en/plugins/mods/create) avant de commencer ici. Pour chaque méthode, voir [méthodes de l'API des mods](https://code.claude.com/docs/en/plugins/mods/reference#mods-api-methods) ou lisez [les types de votre build](https://code.claude.com/docs/en/plugins/mods/create#get-the-types-for-your-build).

## Ajouter une commande ou un outil

Un mod peut ajouter une commande que l'utilisateur exécute et un outil que Claude appelle. Enregistrez les deux dans un hook [`session.start`](https://code.claude.com/docs/en/plugins/mods/reference#session). Claude Code attend ce hook avant le premier prompt, donc ce que vous enregistrez est disponible dès le premier tour.

### Ajouter une commande

Une commande est destinée à l'utilisateur. Enregistrez-la, puis traitez [`command.run`](https://code.claude.com/docs/en/plugins/mods/reference#commands-and-configuration) pour son nom. Cet exemple ajoute une commande `/standup` qui accepte un nombre de jours facultatif :

```javascript theme={null}
on('session.start', async ($, e, next) => {
  // Ajoute /standup à la liste des commandes, avec la description que l'utilisateur y voit
  await $.command.register({ name: 'standup', description: 'Summarize what changed today', argumentHint: '[days]' })
  return next(e)
})

// Le matcher limite le hook à /standup, donc les autres commandes ne l'atteignent pas
on('command.run', { command: 'standup' }, async ($, e) => {
  // e.args est le texte tapé après le nom de la commande, ou une chaîne vide
  return { text: 'Summary for the last ' + (e.args || '1') + ' day(s): ...' }
})
```

Une fois la session démarrée, `/standup` apparaît avec sa description dans la liste affichée quand vous tapez `/`. L'`argumentHint` s'affiche dans le prompt après que vous avez tapé la commande et une espace, comme dans `/standup [days]`. Quand vous lancez `/standup 3`, le second hook renvoie `Summary for the last 3 day(s): ...`, et la transcription affiche ce texte après le nom du plugin. Le hook n'appelle jamais `next`, car la commande n'a aucun autre comportement que le vôtre.

Le `text` que vous renvoyez s'imprime dans la transcription et Claude le lit. Pour n'imprimer rien, comme le fait une commande qui ne fait qu'ouvrir un [panneau](https://code.claude.com/docs/en/plugins/mods/interface#pick-where-to-draw), renvoyez `{}`. Pour permettre à la commande de s'exécuter pendant que Claude travaille, ajoutez `immediate: true` à l'enregistrement.

Choisissez un nom qu'aucune commande intégrée n'utilise. Tapez `/` dans une session pour les voir. `$.command.register` lève une exception pour un nom déjà pris, avec un message tel que `"/focus" refused: it is the built-in /focus`. Un hook qui lève une exception est ignoré, donc le reste de votre hook `session.start` ne s'exécute pas non plus. Enregistrez les commandes en dernier dans ce hook, ou encadrez l'appel par `try` et `catch`.

### Ajouter un outil

Un outil est destiné à Claude. Enregistrez-le avec un nom, une description que Claude lit, et un JSON Schema pour son entrée. Claude le voit sous un nom plus long composé de `mcp__`, du nom de votre plugin, de deux underscores, et du nom que vous avez enregistré. Vous traitez ses appels dans un hook [`tool.call`](https://code.claude.com/docs/en/plugins/mods/events#guard-or-change-a-tool-call) filtré sur ce nom complet. Cet exemple, issu d'un plugin nommé `my-mod`, enregistre `ticket`, donc le nom complet est `mcp__my-mod__ticket`. Il donne à Claude un outil qui consulte un ticket dans un outil de suivi d'incidents :

```javascript theme={null}
on('session.start', async ($, e, next) => {
  await $.tool.register({
    name: 'ticket',
    // Claude décide quand appeler l'outil à partir de cette description
    description: 'Look up a ticket by its id and return its title and status',
    // Les arguments que Claude doit envoyer : une chaîne requise nommée id
    inputSchema: { type: 'object', properties: { id: { type: 'string' } }, required: ['id'] },
  })
  return next(e)
})

// Le nom complet de l'outil est mcp__, le nom du plugin, et le nom enregistré
on('tool.call', { tool: 'mcp__my-mod__ticket' }, async ($, e) => {
  // Les arguments de l'outil sont des champs de e, donc l'id est e.id
  const response = await $.http.fetch('https://tickets.example.com/api/' + encodeURIComponent(e.id))
  // Renvoie un résultat dans tous les cas, pour que Claude sache quand la recherche a échoué
  return { result: response.ok ? response.text : 'Lookup failed with status ' + response.status }
})
```

Quand vous posez une question sur un ticket, Claude peut appeler `mcp__my-mod__ticket` avec son id. Le second hook récupère le ticket et renvoie le corps de la réponse, que Claude lit comme résultat de l'outil. Quand le serveur répond avec un statut d'erreur, Claude lit `Lookup failed with status` suivi du numéro.

> **Astuce** : quand la [recherche d'outils MCP](https://code.claude.com/docs/en/mcp#scale-with-mcp-tool-search) diffère un outil enregistré, Claude voit son nom mais pas sa description tant qu'il ne l'a pas recherché. Si Claude doit envisager l'outil à chaque tour, ajoutez [`isDeferred: false`](https://code.claude.com/docs/en/plugins/mods/reference#tools) à l'enregistrement pour [charger l'outil complet dès le départ](https://code.claude.com/docs/en/mcp#exempt-a-server-from-deferral). Ce champ requiert Claude Code v2.1.293 ou ultérieur (v2.1.293+), et les versions antérieures l'ignorent.

## Appeler un modèle

Un mod peut envoyer ses propres requêtes à un modèle pour une petite tâche, comme classer ou résumer du texte. `$.model.complete` envoie votre prompt seul, et `$.model.fork({ prompt })` envoie la conversation en cours avec votre prompt à la fin.

Ce tableau compare le contenu de chaque requête :

| Dans la requête | `$.model.complete` | `$.model.fork` |
| :- | :- | :- |
| Modèle | Le `model` que vous passez | Le modèle de la session |
| Prompt système | Un court [bloc d'attribution](https://code.claude.com/docs/en/llm-gateway-protocol#system-prompt-attribution-block), puis votre `system` si vous en passez un | Le prompt système de la session |
| Messages | Un seul message utilisateur, votre `prompt` | La conversation jusqu'ici, puis votre `prompt` comme message utilisateur |
| CLAUDE.md et autre contexte du projet | Non inclus | Inclus, comme dans la dernière requête de la conversation |
| Outils | Aucun | Les outils de Claude, que le modèle ne peut pas appeler |

Un fork répète la dernière requête de la conversation, donc la Claude API en sert l'essentiel depuis le [cache de prompt](https://code.claude.com/docs/en/prompt-caching) tant que la conversation est encore en cache.

Les deux appels utilisent les identifiants de la session, donc ils sont facturés sur le forfait de l'utilisateur, sa clé API ou son fournisseur cloud. [Les types de votre build](https://code.claude.com/docs/en/plugins/mods/create#get-the-types-for-your-build) documentent chaque méthode `$.model`.

### Envoyer un seul prompt

Passez `model` et `prompt` à `$.model.complete`. `prompt` devient le message utilisateur. Pour donner des instructions au modèle, comme un rôle ou un format de sortie, passez aussi `system`, qui devient le prompt système.

Ce hook répond à une commande `/triage`, [enregistrée comme commande](#ajouter-une-commande), en demandant à un petit modèle d'étiqueter le texte tapé après elle :

```javascript theme={null}
on('command.run', { command: 'triage' }, async ($, e) => {
  const r = await $.model.complete({
    model: 'haiku',
    // Le prompt système fixe la tâche, et le prompt porte le texte à étiqueter
    system: 'Reply with one word: bug, feature, or question.',
    prompt: e.args,
    // Un mot nécessite peu de tokens, et l'appel abandonne après 15 secondes
    maxTokens: 20,
    timeoutMs: 15000,
  })
  // r.text n'existe que si le modèle a répondu, donc vérifier d'abord r.isAnswered
  const label = r.isAnswered ? r.text.trim() : 'unknown'
  return { text: 'Label: ' + label }
})
```

Quand vous lancez `/triage the export button does nothing`, le mod envoie ce texte au modèle et imprime sa réponse, par exemple `Label: bug`. Quand le modèle ne répond pas, l'étiquette est `unknown`.

Un échec de la Claude API ne rejette pas l'appel : vérifiez donc `r.isAnswered`, et lisez `r.reason` quand il vaut `false`. L'appel est rejeté pour une requête que Claude Code n'enverra pas, comme un modèle que votre organisation bloque.

[Les types de votre build](https://code.claude.com/docs/en/plugins/mods/create#get-the-types-for-your-build) listent les autres options, comme `effort`, et les [limites](https://code.claude.com/docs/en/plugins/mods/reference#limits) donnent la valeur par défaut de `maxTokens`.

### Utiliser le cache de prompt

`$.model.complete` prend en charge le [cache de prompt](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) de la Claude API. L'API met en cache le début d'une requête, appelé le préfixe, jusqu'à un [point d'arrêt de cache](https://platform.claude.com/docs/en/build-with-claude/prompt-caching#explicit-cache-breakpoints) que vous définissez. Quand chaque appel commence par le même long contenu statique, comme des instructions ou du matériel de référence, placez un point d'arrêt à la fin de ce contenu. Les appels suivants le lisent alors depuis le cache au lieu de payer le prix d'entrée complet.

Pour définir un point d'arrêt, passez `prompt` sous forme de tableau de blocs `{ text }` au lieu d'une chaîne, et ajoutez `cache: true` au dernier bloc du contenu statique. Claude Code envoie ce bloc avec le champ `cache_control` de l'API. `system` accepte la même forme de tableau. Pour choisir entre les deux, voir [Choisir entre `prompt` et `system`](#choisir-entre-prompt-et-system).

> **Note** : les tableaux de blocs requièrent Claude Code v2.1.292 ou ultérieur (v2.1.292+). Les versions antérieures rejettent un tableau dans `prompt` avec une erreur se terminant par `takes { model, prompt } (host check)`, et omettent un tableau dans `system` de la requête.

Cette version du [hook `/triage`](#envoyer-un-seul-prompt) envoie un long ensemble de règles d'étiquetage avant le texte à étiqueter, avec un point d'arrêt après les règles. `RULES` est une chaîne qui vous appartient :

```javascript theme={null}
on('command.run', { command: 'triage' }, async ($, e) => {
  const r = await $.model.complete({
    model: 'haiku',
    prompt: [
      // Identique à chaque appel, donc il forme le préfixe mis en cache
      { text: RULES, cache: true },
      // Change à chaque appel, donc il va après le point d'arrêt
      { text: e.args },
    ],
  })
  return { text: 'Label: ' + (r.isAnswered ? r.text.trim() : 'unknown') }
})
```

Le TTL et le nombre de points d'arrêt ont ces limites :

* **TTL** : une entrée de cache dure cinq minutes après sa dernière utilisation. Le TTL vient des réglages Claude Code de l'utilisateur, pas de l'appel. Pour une heure, réglez [`subagentPromptCacheTtl`](https://code.claude.com/docs/en/prompt-caching#choose-the-ttl-yourself) sur `1h`.
* **Points d'arrêt par requête** : l'API en accepte [jusqu'à quatre](https://platform.claude.com/docs/en/build-with-claude/prompt-caching#when-to-use-multiple-breakpoints), et un de plus revient sous forme d'`api-error` dans `r.reason`.

#### Choisir entre `prompt` et `system`

Placez le contenu statique que vos appels partagent au début de `prompt`, sauf si vous savez que vos requêtes vont directement à la Claude API :

* **Directement vers la Claude API, avec une clé API ou un abonnement Claude** : les deux champs fonctionnent
* **Via [Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock), [Claude Platform on AWS](https://code.claude.com/docs/en/claude-platform-on-aws), [Agent Platform de Google Cloud](https://code.claude.com/docs/en/google-vertex-ai), [Microsoft Foundry](https://code.claude.com/docs/en/microsoft-foundry), ou une [passerelle LLM](https://code.claude.com/docs/en/llm-gateway)** : utilisez `prompt`. Claude Code commence le prompt système par un [bloc d'attribution](https://code.claude.com/docs/en/llm-gateway-protocol#system-prompt-attribution-block) dont l'empreinte provient du début du message utilisateur. Le point de terminaison `api.anthropic.com` retire ce bloc avant la mise en cache. Les autres points de terminaison le reçoivent comme partie du prompt, donc un point d'arrêt dans `system` peut manquer le cache quand `prompt` commence différemment.
* **Dans un mod que d'autres personnes exécutent** : utilisez `prompt`, car vous ne choisissez pas leur fournisseur

`system` précède `prompt` dans le préfixe, donc un point d'arrêt dans `prompt` couvre aussi `system`, et un appel avec un `system` différent manque le cache.

#### Vérifier les succès de cache

Le résultat de `$.model.complete` contient un objet `usage` avec les [champs de cache](https://platform.claude.com/docs/en/build-with-claude/prompt-caching#tracking-cache-performance) de l'API. `usage.cache_creation_input_tokens` compte les tokens que l'appel a écrits dans le cache, et `usage.cache_read_input_tokens` compte les tokens qu'il a lus depuis le cache. Attendez-vous à une écriture au premier appel et à des lectures lors des appels suivants dans le TTL.

Si chaque appel écrit et qu'aucun ne lit, le préfixe diffère d'un appel à l'autre ou les appels sont plus espacés que le TTL. Pour un préfixe qui diffère, voir [Choisir entre `prompt` et `system`](#choisir-entre-prompt-et-system).

Si les deux champs restent à zéro sur des appels auxquels le modèle a répondu, rien n'a été mis en cache. Vérifiez chacune de ces causes :

* **Le préfixe est trop court** : l'API ne met pas en cache un préfixe inférieur à la [longueur minimale](https://platform.claude.com/docs/en/build-with-claude/prompt-caching#cache-limitations) du modèle, et ne renvoie aucune erreur
* **Le cache de prompt est désactivé** : quand une [variable `DISABLE_PROMPT_CACHING`](https://code.claude.com/docs/en/prompt-caching#disable-prompt-caching) s'applique au modèle, Claude Code retire les points d'arrêt et envoie le texte sans cache
* **Votre passerelle retire `cache_control`** : une passerelle peut [supprimer le champ et renvoyer quand même un succès](https://code.claude.com/docs/en/prompt-caching#where-the-cache-lives)
* **Un autre mod réécrit le début du texte** : Claude Code l'[envoie alors sans points d'arrêt](#ce-que-reçoit-un-hook-modelcomplete)

### Ce que reçoit un hook `model.complete`

Si vous accrochez l'événement [`model.complete`](https://code.claude.com/docs/en/plugins/mods/reference#mods-api-calls) pour inspecter ou modifier les requêtes d'autres mods, lisez le texte depuis ces champs :

* **`e.prompt`** : toujours une chaîne. Quand l'appelant a passé un tableau, c'est le texte des blocs concaténé dans l'ordre.
* **`e.system`** : une chaîne construite de la même manière, ou absent quand l'appelant n'a passé aucun `system`
* **`e.promptBlocks` et `e.systemBlocks`** : les tableaux de l'appelant, chacun présent quand l'appelant a passé un tableau pour ce champ

Claude Code envoie les chaînes que votre hook passe à `next` et utilise les tableaux que vous passez avec elles pour placer les [points d'arrêt de cache](#utiliser-le-cache-de-prompt). Il conserve les blocs de tête qui correspondent encore au début de la chaîne, avec leurs points d'arrêt, et envoie le reste de la chaîne sans point d'arrêt. Par exemple, `next({ ...e, prompt: e.prompt + NOTE })` conserve les points d'arrêt de l'appelant, et un hook qui modifie le début de `prompt` les supprime.

## Exécuter du travail en arrière-plan

Le travail qui dure plus d'un événement, comme vérifier quelque chose une fois par minute, s'exécute sur une minuterie que vous démarrez depuis `session.start`. Un hook lui-même s'exécute pour un seul événement et a une [limite de temps](https://code.claude.com/docs/en/plugins/mods/reference#limits) sur son propre temps d'exécution. Le temps passé à attendre `next` ou un appel à l'API des mods ne compte pas, sauf un `$.clock.sleep`. `$.clock.every` et `$.clock.after` remplacent `setInterval` et `setTimeout`, avec le délai en millisecondes en premier : `$.clock.after(5000, fn)` appelle `fn` une fois, dans cinq secondes. Chacun renvoie une minuterie dotée d'une méthode `cancel()`, et `await $.clock.now()` donne l'heure en millisecondes.

Ce hook consulte les checks d'une pull request une fois par minute et affiche le résultat sous le prompt. `summarize` est une fonction à vous qui transforme la sortie JSON de la commande en quelques mots :

```javascript theme={null}
on('session.start', async ($, e, next) => {
  // Appelle la fonction toutes les 60 000 millisecondes, à partir d'une minute d'ici
  $.clock.every(60_000, async () => {
    const status = await $.process.run(['gh', 'pr', 'checks', '--json', 'state'])
    // Remplace la ligne sous le prompt par le dernier résumé
    $.ui.status('checks: ' + summarize(status.stdout))
  })
  // Retourne sans attendre la minuterie, pour que la session démarre tout de suite
  return next(e)
})
```

La session démarre normalement. Une minute plus tard, une ligne apparaît sous le prompt avec un `⚠`, le nom du mod, puis `checks:` et votre résumé. Elle est remplacée une fois par minute ensuite. Le callback de la minuterie s'exécute en dehors de tout événement, donc il continue de tourner entre les tours et n'en démarre pas. Si le callback lève une exception, l'erreur va dans le [journal de débogage](https://code.claude.com/docs/en/plugins/mods/troubleshoot#read-the-debug-log) et la minuterie s'exécute de nouveau à l'intervalle suivant.

### Afficher quelque chose sans démarrer de tour

Une tâche d'arrière-plan peut montrer quelque chose à l'utilisateur sans démarrer de tour. Chacun de ces appels place du texte à un endroit différent :

| Appel | Ce que voit l'utilisateur |
| :- | :- |
| `$.ui.status(text)` | Une ligne sous le prompt qui reste jusqu'à ce que vous la changiez. Elle commence par `⚠` et le nom du mod, comme dans `⚠ my-mod: checks: 3 passing`. |
| `$.ui.toast(text)` | Une notification toast avec le nom du mod, qui disparaît après quelques secondes. C'est une boîte en haut à droite en [rendu plein écran](https://code.claude.com/docs/en/fullscreen), et une ligne à droite sous le prompt dans le moteur de rendu classique. |
| `$.ui.log(text)` | Une ligne atténuée dans la transcription, que Claude ne lit pas. Elle commence par `●` et le nom du mod, comme dans `● my-mod: build finished`. |

### Démarrer un tour depuis une tâche d'arrière-plan

Quand une tâche d'arrière-plan trouve quelque chose qui requiert l'attention de Claude, elle peut démarrer un tour en soumettant un prompt avec `$.prompt.submit({ text })`. Claude lit le texte après une phrase qui désigne votre mod comme expéditeur. Pour l'envoyer comme les propres mots de l'utilisateur, sans cette phrase, ajoutez `asUser: true`. L'appel attend que la session soit inactive, puis démarre un nouveau tour. Il se résout quand ce tour démarre, donc ne l'attendez pas avec `await` dans un handler qui s'exécute pendant que Claude travaille.

### Arrêter le travail en arrière-plan

Les minuteries s'arrêtent quand le module se recharge. Pour un travail de longue durée à l'intérieur d'un hook, [`next.signal`](https://code.claude.com/docs/en/plugins/mods/reference#the-hook-function) est un `AbortSignal` qui s'interrompt quand l'événement traité par votre hook est abandonné, par exemple quand l'utilisateur interrompt : passez-le donc à tout ce qui dure longtemps.

## Envoyer et recevoir des messages entre sessions

Un mod peut envoyer un message en texte brut à une autre de vos sessions, à l'un des sous-agents de cette session, ou à un coéquipier de son [équipe d'agents](https://code.claude.com/docs/en/agent-teams). Il peut aussi observer les messages qui arrivent et qui partent.

Pour en envoyer un, appelez `$.session.send({ to, text })`, qui effectue la même livraison que l'outil SendMessage. Réglez `to` selon le destinataire du message :

* **Une autre de vos sessions** : `{ sessionId }`
* **Un sous-agent ou un coéquipier** : `{ agentId }`, avec un id issu de `$.agent.list()`
* **L'expéditeur d'un message que vous avez reçu** : l'adresse sous forme de chaîne d'où provenait ce message

L'appel se résout une fois le message mis en file, avec `{ isDelivered: true }`. Quand rien n'a été livré, il se résout avec `{ isDelivered: false, reason }`, et `reason` indique pourquoi.

Ce hook répond à une commande `/ping`, [enregistrée comme commande](#ajouter-une-commande), en demandant un état à la session dont vous tapez l'id après elle :

```javascript theme={null}
on('command.run', { command: 'ping' }, async ($, e) => {
  // e.args est l'id de session tapé après /ping
  const sent = await $.session.send({ to: { sessionId: e.args }, text: 'Status? One line.' })
  // L'appel se résout dans tous les cas, donc vérifier isDelivered pour savoir ce qui s'est passé
  if (!sent.isDelivered) $.ui.toast('Not delivered: ' + sent.reason)
  // Un résultat vide n'imprime rien dans la transcription de cette session
  return {}
})
```

Quand le message est mis en file, rien n'apparaît dans votre session, et le Claude de l'autre session lit `Status? One line.` Quand rien n'a été livré, une notification toast en donne la raison.

`session.receive` et `session.send` permettent à un mod d'observer les messages. Renvoyez `next(e)` dans les deux pour laisser passer chaque message inchangé :

| Événement | Se déclenche quand | Champs utiles |
| :- | :- | :- |
| `session.receive` | Un message arrive pour cette session, avant que Claude ne le lise | `e.text`, et `e.origin.kind`, comme `peer` ou `peer-send-message` pour une autre session ou un autre agent, `task-notification`, ou `scheduled-trigger`. Renvoyez `{ consumed: reason }` pour le cacher à Claude. |
| `session.send` | Un message est sur le point de partir, depuis l'outil SendMessage ou un mod | `e.to`, `e.text`, et `e.origin.kind`, qui vaut `model` ou `plugin` |

Une session réglée pour [refuser les messages entrants](https://code.claude.com/docs/en/cross-session-messaging#control-inbound-messages) refuse un message avant que `session.receive` ne se déclenche, donc un hook ne le voit jamais. Un message retenu en attente de votre approbation atteint d'abord le hook, donc un mod peut lire un message que vous n'avez pas encore approuvé. Le `next(e)` du hook est rejeté quand le message n'est pas livré.

Le nom de l'expéditeur sur un message reçu est ce que l'expéditeur a écrit : ne fondez donc aucune décision dessus.

## Accéder aux fichiers, aux processus et au réseau

Un mod accède au système de fichiers, aux processus et au réseau via l'API des mods, avec les mêmes permissions que l'utilisateur qui exécute Claude Code. Le module de hooks lui-même n'a aucune API Node.js, aucune minuterie globale comme `setTimeout`, et aucun accès réseau ou fichier propre. Les API JavaScript et web standard comme `URL`, `TextEncoder`, `AbortController` et `crypto.subtle` sont disponibles. Chaque espace de noms ci-dessous couvre un type d'accès :

| Espace de noms | Ce qu'il fait |
| :- | :- |
| `$.fs` | `read(path)`, `write(path, text)`, `exists(path)`, `stat(path)` et `list(path)` agissent sur les fichiers et répertoires |
| `$.process` | `run(['git', 'status'])` lance une commande et se résout quand elle se termine. `spawn` diffuse en continu la sortie d'une commande de longue durée. |
| `$.http` | `fetch(url, init)` sur `http` ou `https`. Se résout en `{ status, ok, headers, text }` une fois le corps lu. |
| `$.store` | Un magasin clé-valeur JSON propre à votre plugin, conservé entre les sessions |
| `$.env` | `get` et `set` des variables d'environnement. Écrivez le nom comme une chaîne littérale. |
| `$.settings` | `read` ce que contiennent les fichiers de réglages et la politique gérée |
| `$.session` | `messages()` renvoie la transcription sous forme de liste de `{ role, text, toolUses }`. Aussi le répertoire de travail, le modèle, et plus. [`usage()`](https://code.claude.com/docs/en/plugins/mods/reference#mods-api-methods) renvoie l'utilisation de la fenêtre de contexte et les limites du forfait. |
| `$.mcp` | `call` un outil sur un serveur MCP connecté |

Les fichiers, processus et requêtes ont quelques règles propres :

* **Chemins** : un chemin relatif se résout par rapport au répertoire de travail de la session, ou du sous-agent dont le hook traite l'événement
* **`$.fs.list`** : renvoie les entrées d'un seul répertoire sous la forme `{ name, kind, size, isLink }` et n'est pas récursif
* **`$.process.run`** : prend une liste d'arguments et n'utilise aucun shell. Se résout en `{ exitCode, stdout, stderr }` quel que soit le code de sortie. Est rejeté si le programme ne peut pas démarrer ou tourne encore au délai d'expiration, de 30 secondes par défaut : encadrez-le donc par `try` et `catch`.
* **`$.http.fetch`** : suit jusqu'à cinq redirections. Sur une redirection vers une origine différente, il ne conserve que les en-têtes de requête `accept`, `accept-language`, `content-type` et `user-agent` que vous avez définis et supprime les autres, donc une requête qui dépend d'un autre en-tête, comme `Authorization`, peut échouer après cette redirection. Les [limites](https://code.claude.com/docs/en/plugins/mods/reference#limits) donnent son délai d'expiration et ses tailles de corps.

Chacun de ces appels est lui-même un événement, nommé d'après son espace de noms et sa méthode sans le `$.`, comme `fs.read` pour `$.fs.read`. Un mod [plus tôt dans la chaîne](https://code.claude.com/docs/en/plugins/mods/events#the-order-mods-run-in) peut observer, réécrire ou refuser votre appel : c'est ainsi qu'une organisation restreint ce que les mods atteignent.

Un mod peut refuser votre appel `$.process.spawn` après que la commande a produit une sortie ou s'est terminée, et rien de ce que la commande a fait n'est annulé. L'appel est alors rejeté avec un message qui se termine par l'une de ces chaînes suivie de la raison du mod qui refuse :

* **`$.process.spawn started, and a plugin withheld its result:`** : le mod qui refuse n'avait pas lu la sortie de la commande jusqu'au bout. Claude Code arrête la commande si elle tourne encore.
* **`$.process.spawn ran, and a plugin withheld its result:`** : le mod qui refuse avait lu la sortie de la commande jusqu'au bout, donc la commande s'était terminée

## Étapes suivantes

* [Réagir aux événements](https://code.claude.com/docs/en/plugins/mods/events) : accrocher les appels d'outils, les prompts et les tours
* [Dessiner dans l'interface](https://code.claude.com/docs/en/plugins/mods/interface) : afficher ce que votre mod collecte dans un panneau ou au-dessus du prompt
* [Tester un mod](https://code.claude.com/docs/en/plugins/mods/test) : remplacer n'importe lequel de ces appels par un stub dans un test
* [Référence des mods](https://code.claude.com/docs/en/plugins/mods/reference) : événements, méthodes de l'API des mods et limites
