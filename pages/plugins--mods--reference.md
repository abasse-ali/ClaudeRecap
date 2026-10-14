# Référence des mods

> Source : https://code.claude.com/docs/en/plugins/mods/reference · [← Index](../31-index-documentation-officielle.md)

*Référence complète des mods de Claude Code : organisation du module de hooks, événements, méthodes de l'API des mods, sites de rendu, éléments par surface, limites et réglages.*

Retrouvez ici chaque événement qu'un [mod](https://code.claude.com/docs/en/plugins/mods/overview) peut gérer, chaque méthode de l'API des mods qu'il peut appeler et chaque site de rendu (render site) où il peut dessiner, pour la CLI Claude Code et l'application Desktop, à jour de la v2.1.290. Chaque entrée donne le nom et une description d'une ligne, avec un lien vers la section du guide qui l'explique lorsqu'elle existe.

> **Note** : la référence complète est constituée des [déclarations TypeScript des mods de Claude Code](https://github.com/anthropics/claude-code/blob/main/mods/types/claude-code.d.ts), qui décrivent chaque événement, méthode et élément, avec des exemples. La copie sur GitHub peut être plus ancienne que la version de Claude Code installée chez vous. En cas de désaccord, fiez-vous à [la copie que Claude Code écrit pour votre version](https://code.claude.com/docs/en/plugins/mods/create#get-the-types-for-your-build).

## Fichiers

Un mod est un répertoire de plugin contenant ces fichiers :

| Fichier | Obligatoire | Contenu |
| :- | :- | :- |
| `.claude-plugin/plugin.json` | Oui | Le [manifeste](https://code.claude.com/docs/en/plugins/manifest-reference) du plugin. Les mods n'ajoutent aucun champ obligatoire. |
| `hooks/hooks.json` | Oui | `modules` : un tableau contenant un seul chemin, relatif à ce fichier, vers le module de hooks, comme dans `"modules": ["./register.js"]`. Peut aussi contenir des [hooks de réglages](https://code.claude.com/docs/en/hooks) sous `hooks`. |
| Le module de hooks, par exemple [`hooks/register.js`](https://code.claude.com/docs/en/plugins/mods/create#write-a-mod-yourself) | Oui | Le point d'entrée du mod. Exporte `register(on, options)`. Extension `.js`, `.mjs`, `.cjs`, `.jsx`, `.ts`, `.mts`, `.cts` ou `.tsx`. Un module ES. |
| [`types/index.d.ts`](https://code.claude.com/docs/en/plugins/mods/interface#declare-the-values), désigné par `types` dans le manifeste | Quand le mod utilise `$.state` ou ajoute un namespace à l'API des mods | Déclare les valeurs `PluginState` et tout namespace qu'ajoute le mod |
| Fichiers dont le nom se termine par `.test.ts` ou `.test.tsx` | Non | Tests qu'exécute [`claude plugin test`](https://code.claude.com/docs/en/plugins/mods/test#write-a-test) |

`register` reçoit `on` et `options`. `options` contient les valeurs des champs [`userConfig`](https://code.claude.com/docs/en/plugins/components#user-configuration) que déclare le manifeste, avec les valeurs par défaut renseignées.

## La fonction de hook

Un mod enregistre chacun de ses hooks (gestionnaires d'événements) en appelant `on` à l'intérieur de `register`. `on` prend le nom de l'événement, un [matcher](https://code.claude.com/docs/en/plugins/mods/events#filter-which-events-a-hook-handles) facultatif (un filtre sur les champs de l'événement), et le hook, comme dans `on('tool.call', { tool: 'Bash' }, async ($, e, next) => next(e))`. `on` renvoie un enregistrement doté d'une seule méthode, `.catch(handler)`, qui définit le [gestionnaire d'erreur](https://code.claude.com/docs/en/plugins/mods/events#handle-a-hook-that-fails) du hook.

| Argument | Ce que c'est |
| :- | :- |
| [`$`](https://code.claude.com/docs/en/plugins/mods/events#how-a-hook-handles-an-event) | L'API des mods : toutes les méthodes de [Méthodes de l'API des mods](#méthodes-de-lapi-des-mods). Écrivez chaque appel en entier, namespace puis méthode, comme dans `$.fs.read('notes.md')`. |
| [`e`](https://code.claude.com/docs/en/plugins/mods/events#how-a-hook-handles-an-event) | L'entrée de l'événement, sous forme de données simples profondément gelées (deeply frozen). Pour la modifier, passez une copie à `next`. |
| [`next(e)`](https://code.claude.com/docs/en/plugins/mods/events#how-a-hook-handles-an-event) | Le gestionnaire suivant, comme dans un middleware. Exécute les hooks qui suivent celui-ci, puis le comportement de Claude Code. Se résout avec le résultat de l'événement. |
| [`next.signal`](https://code.claude.com/docs/en/plugins/mods/api#stop-background-work) | Un `AbortSignal` qui est déclenché quand l'événement est abandonné |
| `next.origin` | `{ plugin, tier }` de l'émetteur de l'événement. Claude Code lui-même est `{ plugin: 'engine', tier: 'core' }`. Le `tier` d'un mod est son groupe de priorité dans l'[ordre d'exécution des mods](https://code.claude.com/docs/en/plugins/mods/events#the-order-mods-run-in) : `prepend`, `user`, `append` ou `builtin`. |
| `next.budget` | La limite de temps du hook en millisecondes : `next.budget.ms` est la limite totale, et `next.budget.remainingMs` ce qui reste à cet instant |
| `next.to(e, tier)` | Saute vers un tier ultérieur : `append`, `builtin` ou `core`. `next.to(e, 'append')` saute les mods installés par un utilisateur. Seul un mod listé dans `prependPlugins` ou `appendPlugins` peut l'appeler. |
| `next.error` | Dans un gestionnaire `.catch` uniquement. `kind` vaut `throw` ou `timeout` quand le hook a échoué, et `message` est le texte de l'erreur. `kind` vaut `re-entry` quand le hook a été sauté parce que l'événement provenait de l'un de ses propres appels à l'API des mods, et `cause` vaut `lent` si c'est une méthode ajoutée à l'API des mods par un autre mod qui a déclenché cet événement. `re-entry` et `cause` : v2.1.292+. |
| `next.called` | Dans un gestionnaire `.catch` uniquement. `true` quand le hook avait appelé `next`. |

## Événements

Les événements sont regroupés selon ce qu'ils concernent, avec le moment où ils se déclenchent et ce qu'un hook peut renvoyer. Les hooks sur `turn.step` et `process.spawn` sont des générateurs asynchrones ; les autres hooks sont des fonctions asynchrones.

La dernière colonne de chaque tableau utilise une notation abrégée. `next(e)` transmet l'événement inchangé. `next({ ...e, text })` transmet une copie dont le champ nommé est modifié, comme dans `next({ ...e, text: e.text.trim() })`. Un objet répond à l'événement sans appeler `next`, et un mot comme `reason` représente une chaîne que vous écrivez, comme dans `{ deny: 'Use the file tools.' }`.

### Outils

Les événements d'outil se déclenchent autour de chaque appel d'outil de Claude, depuis la description que lit Claude jusqu'à la décision d'exécuter ou non l'appel :

| Événement | Se déclenche quand | Un hook peut renvoyer |
| :- | :- | :- |
| [`tool.call`](https://code.claude.com/docs/en/plugins/mods/events#guard-or-change-a-tool-call) | Un outil est sur le point de s'exécuter | `next(e)`, `{ deny: reason }` ou `{ result }` |
| [`tool.check`](https://code.claude.com/docs/en/plugins/mods/events#where-settings-hooks-run-in-the-order) | Claude Code décide si un appel d'outil peut s'exécuter, après les hooks `tool.call` et `PreToolUse`. `next(e)` se résout avec la décision à laquelle sont arrivés les règles, le mode de permission et ces hooks. | `{ decision }`, qui vaut `allow`, `ask` ou `deny` |
| `tool.describe` | Une fois pour chaque outil, quand sa description est envoyée à Claude pour la première fois. Une seconde fois pour un outil MCP quand Claude le charge via la [recherche d'outils](https://code.claude.com/docs/en/mcp#scale-with-mcp-tool-search), avec `e.description` contenant le texte que Claude lit pour l'outil chargé. | `{ description }`, éventuellement avec `isDeferred` à `true` pour placer l'outil derrière la recherche d'outils, ou à `false` pour le charger d'emblée |

#### Champs d'agent et d'organisation sur `tool.check`

Dans un hook `tool.check`, lisez ces champs pour distinguer l'appel d'un sous-agent de celui de la conversation principale, et pour savoir si votre organisation exige une approbation pour un outil de connecteur :

* **`e.agentId`** : défini quand l'appel est fait par un [sous-agent](https://code.claude.com/docs/en/sub-agents) ou un [coéquipier in-process](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode), absent quand c'est la conversation principale
* **`e.ceiling`** : `ask` pour un outil de connecteur que votre organisation a réglé sur `ask`, dans les [sessions où ce réglage parvient à Claude Code](https://code.claude.com/docs/en/mcp#organization-controls-on-connector-tools)

Sur `tool.check`, `e.agentId` et `e.ceiling` : v2.1.290+.

### Prompts et ce que lit Claude

Les événements de prompt couvrent le texte que tape l'utilisateur et le texte que Claude Code envoie de lui-même à Claude, comme le prompt système et les rappels :

| Événement | Se déclenche quand | Un hook peut renvoyer |
| :- | :- | :- |
| [`prompt.submit`](https://code.claude.com/docs/en/plugins/mods/events#rewrite-or-add-to-a-prompt) | Un prompt est soumis | `next({ ...e, text })`, `next({ ...e, context })` ou `{ drop: reason }` |
| `prompt.fill`, `prompt.suggest` | Du texte est sur le point d'entrer dans la zone de prompt comme brouillon, ou comme suggestion grisée | `next(e)` avec un texte modifié |
| `prompt.edit` | L'utilisateur modifie la zone de prompt | `next(e)` |
| `prompt.autocomplete` | L'utilisateur tape dans la zone de prompt et un mot se termine au niveau du curseur. `e.token` est ce mot. Claude Code n'attend pas le hook : ses propres lignes d'autocomplétion s'affichent d'abord. | `{ suggestions }`, une liste de lignes `{ text, label, description }` affichées sous celles de Claude Code. Choisir une ligne écrit son `text` à la place du mot. |
| `prompt.compose` | Claude Code produit un prompt système | `{ sections }`, une liste de `{ id, text, scope }` dans l'ordre d'envoi |
| [`prompt.section`](https://code.claude.com/docs/en/plugins/mods/events#rewrite-or-add-to-a-prompt) | Une fois pour chaque section nommée du prompt système. `e.name` est l'`id` de la section dans `prompt.compose`. | `{ text }`, ou `{ text: null }` pour omettre la section |
| [`prompt.context`](https://code.claude.com/docs/en/plugins/mods/events#rewrite-or-add-to-a-prompt) | Une fois par conversation, pour le contexte envoyé avec le premier message | `{ blocks }` |
| `prompt.attachment` | Claude Code ajoute un message de son cru pour Claude, comme un rappel. `e.type` en nomme le genre et, pour les genres que déclarent les types, `e.detail` contient les faits à partir desquels le texte a été rédigé. | `{ text }`, ou `{ text: null }` pour l'omettre |
| `prompt.mention` | Claude Code s'apprête à lire un fichier qu'un prompt @-mentionne. v2.1.290+. | `next({ ...e, path })` pour lire un autre fichier, ou `{ deny: reason }` |
| [`skill.prompt`](https://code.claude.com/docs/en/plugins/mods/events#rewrite-or-add-to-a-prompt) | Le texte d'une skill est développé pour Claude | `{ text }` |
| `attribution.text` | Claude Code compose le texte d'attribution d'un commit ou d'une pull request | `{ text }` |

### Commandes et configuration

Les événements de commande et de configuration se déclenchent quand une commande s'exécute ou est listée, et quand une ligne de `/config` est affichée ou modifiée :

| Événement | Se déclenche quand | Un hook peut renvoyer |
| :- | :- | :- |
| [`command.run`](https://code.claude.com/docs/en/plugins/mods/api#add-a-command) | Une commande est sur le point de s'exécuter | `{ text }`, `{}` ou `next(e)` |
| `command.describe` | Une fois pour chaque commande, pour la liste des commandes | `{ description, argumentHint, isHidden }` |
| `config.set` | Une ligne de `/config` est sur le point de changer | `next({ ...e, value })` ou `{ deny: reason }` |
| `config.describe` | Une fois pour chaque ligne de `/config` | `{ label, description, isHidden }` |

### Tours

Les événements de tour suivent une réponse du début à la fin, y compris chaque requête au modèle qu'elle comporte :

| Événement | Se déclenche quand | Un hook peut renvoyer |
| :- | :- | :- |
| [`turn.start`](https://code.claude.com/docs/en/plugins/mods/events#follow-a-turn) | Un tour commence | `next(e)` |
| [`turn.step`](https://code.claude.com/docs/en/plugins/mods/events#follow-a-turn) | Une requête est sur le point de partir vers le modèle | `yield* next(e)`, ou `next({ ...e, model })`, `next({ ...e, effort })` |
| [`turn.complete`](https://code.claude.com/docs/en/plugins/mods/events#follow-a-turn) | Un tour s'est terminé | `next(e)`, ou `{ text }` pour afficher une ligne sous la réponse |

### Session

Les événements de session marquent le démarrage, la fin et la compaction de la session, ainsi que l'échange de messages avec d'autres sessions :

| Événement | Se déclenche quand | Un hook peut renvoyer |
| :- | :- | :- |
| [`session.start`](https://code.claude.com/docs/en/plugins/mods/api#add-a-command-or-a-tool) | Une fois pour chaque mod chargé, avant le premier prompt, et de nouveau après un rechargement de ce mod. Pas après `/clear`, `/resume` ou `/branch`. | `next(e)` |
| `session.end` | La session se termine, ou `/clear`, `/resume` ou `/branch` s'exécute. `e.reason` vaut `clear`, `resume`, `logout`, `prompt_input_exit` ou `other`. `/branch` rapporte `resume`. | `next(e)` |
| `session.compact` | La conversation est sur le point d'être compactée | `{ skip: reason }` |
| [`session.receive`](https://code.claude.com/docs/en/plugins/mods/api#send-and-receive-messages-between-sessions), [`session.send`](https://code.claude.com/docs/en/plugins/mods/api#send-and-receive-messages-between-sessions) | Un message arrive d'un autre agent ou d'une autre session, ou est sur le point d'y partir. Voir [Envoyer et recevoir des messages entre sessions](https://code.claude.com/docs/en/plugins/mods/api#send-and-receive-messages-between-sessions). | `{ consumed: reason }` pour `receive`, `{ isDelivered: false, reason }` pour `send` |
| `session.append` | Une fois pour chaque ligne que conserve la conversation (prompt, bloc de réponse, résultat d'outil, avis…), avant son stockage | `next({ ...e, message })` avec un `message.content` modifié, pour réécrire les blocs de texte de la ligne ou le `content` d'un bloc `tool_result` qu'elle contient |
| `session.attach`, `session.detach` | Une autre application se connecte à la session ou s'en déconnecte | `next(e)` |
| `session.measure` | Après chaque tour, et quand le pourcentage utilisé d'une limite de forfait change | `next(e)` |

### Sous-agents

Les événements de sous-agent se déclenchent quand un type de sous-agent est proposé à Claude et quand un sous-agent ou un coéquipier d'équipe d'agents est sur le point de démarrer :

| Événement | Se déclenche quand | Un hook peut renvoyer |
| :- | :- | :- |
| `agent.offer` | Un type de sous-agent est proposé à Claude | `{ isOffered: false }` pour le retenir |
| `agent.spawn` | Un sous-agent ou un coéquipier d'[équipe d'agents](https://code.claude.com/docs/en/agent-teams) est sur le point de démarrer. Pour un coéquipier, `e.isTeammate` vaut `true`. | `next({ ...e, model })` pour choisir son modèle, ou `{ deny: reason }` |

Quand Claude reprend un sous-agent avec l'outil [`SendMessage`](https://code.claude.com/docs/en/sub-agents#resume-subagents), votre hook `agent.spawn` ne s'exécute pas à nouveau. Pour refuser les appels `SendMessage` qui reprennent un sous-agent, ciblez cet outil dans un hook [`tool.call`](https://code.claude.com/docs/en/plugins/mods/events#guard-or-change-a-tool-call).

### Interface

Les événements d'interface se déclenchent quand Claude Code dessine un site de rendu et quand l'utilisateur utilise un contrôle dessiné par un mod. [Dessiner dans l'interface](https://code.claude.com/docs/en/plugins/mods/interface) montre ce que renvoie un hook `ui.render` :

| Événement | Se déclenche quand |
| :- | :- |
| [`ui.render`](https://code.claude.com/docs/en/plugins/mods/interface#pick-where-to-draw) | Un [site de rendu](#sites-de-rendu) est sur le point d'être dessiné |
| `ui.resolve` | Au chargement des mods, une fois pour chaque application, site de rendu et mod. Le résultat est la table d'éléments que lit `$.ui.resolve(e)`. |
| [`ui.press`](https://code.claude.com/docs/en/plugins/mods/interface#respond-to-presses-and-typing), [`ui.input`](https://code.claude.com/docs/en/plugins/mods/interface#respond-to-presses-and-typing), [`ui.select`](https://code.claude.com/docs/en/plugins/mods/interface#respond-to-presses-and-typing) | Un `Button`, un `Input` ou un `Select` dessiné par un mod est utilisé |
| `ui.focus`, `ui.scroll` | Le contrôle ayant le focus, ou la position de défilement d'un panneau ou de la bande, est sur le point de changer |
| `ui.close` | Un panneau est sur le point de se fermer. `e.id` est le panneau et `e.origin.kind` vaut `plugin`, `person` ou `unload`. |
| [`ui.message`](https://code.claude.com/docs/en/plugins/mods/interface#build-a-tree-from-elements) | Un élément `Client` envoie des données à son mod |
| [`ui.fault`](https://code.claude.com/docs/en/plugins/mods/interface#redraw-when-something-changes) | Un élément `Client` dessiné par votre mod n'a pas pu se charger, se dessiner ou s'exécuter. `e.phase` vaut `load`, `render` ou `run`, et `e.reason` est le message d'erreur. v2.1.289+. |

### Autres mods

Ces événements permettent à un mod d'agir sur d'autres mods au moment où ils se chargent, pour en refuser un ou modifier l'API des mods qu'il reçoit :

| Événement | Se déclenche quand | Un hook peut renvoyer |
| :- | :- | :- |
| [`plugin.register`](https://code.claude.com/docs/en/plugins/mods/admin#enforce-a-policy-with-a-mod-of-your-own) | Un module de hooks est sur le point de se charger. `e.uses` liste ses événements, appels à l'API des mods, variables d'environnement et état, tels que les affiche `claude plugin validate`. Chaque appel est écrit sans le préfixe `$.`, par exemple `fs.read`. | `{ refuse: reason }` |
| `engine.create` | L'API des mods est en cours de construction pour ce mod | Une API des mods modifiée, pour ajouter un namespace. Un mod hors du [tier](#la-fonction-de-hook) `user` peut aussi en retirer un. |

Quand le hook d'un autre mod appelle une méthode d'un namespace que vous avez ajouté dans `engine.create`, les appels `$` de votre méthode s'exécutent dans le contexte de ce hook jusqu'à ce que tous les hooks de cet événement aient rendu la main. Par exemple, un chemin relatif se résout par rapport au répertoire de travail de ce hook, et `$.prompt.submit` est rejeté pendant que le tour attend ce hook. Les appels que fait votre méthode après cela s'exécutent dans le contexte propre de votre mod.

### Télémétrie

Les événements de télémétrie se déclenchent pour les enregistrements d'usage que journalise Claude Code :

| Événement | Se déclenche quand | Un hook peut renvoyer |
| :- | :- | :- |
| `telemetry.log`, `telemetry.mark` | Un enregistrement de télémétrie est sur le point d'être journalisé, ou une utilisation d'une fonctionnalité est marquée. Dans un mod que vous installez, donnez à un hook de télémétrie le filtre `{ to: 'collector' }`, comme dans `on('telemetry.log', { to: 'collector' }, hook)`. Sans ce filtre, le mod échoue à `claude plugin validate`. `*` ne correspond pas à ces événements. | `next(e)`, ou `{ deny: reason }` |

### Événements des hooks de réglages

Chaque [événement de hook de réglages](https://code.claude.com/docs/en/hooks#hook-events) est un événement nommé `classic.<Event>`, comme `classic.Stop` ou `classic.PostToolUse`. `e` est le JSON reçu sur stdin par le hook.

### Appels à l'API des mods

Chaque [méthode de l'API des mods](#méthodes-de-lapi-des-mods) est aussi un événement, nommé d'après son namespace et sa méthode, comme `fs.read`, `model.complete` ou `ui.open`. Un hook sur l'un d'eux intercepte les appels des mods qui s'exécutent après lui, et peut renvoyer `next(e)`, `{ deny: reason }` ou `{ value }`.

## Méthodes de l'API des mods

L'API des mods est l'argument `$` que reçoit chaque hook. Ses méthodes sont regroupées en namespaces, comme `$.ui`. Ce tableau liste les méthodes de chaque namespace par leur nom : `open` dans la ligne `$.ui` correspond à l'appel `$.ui.open(...)`. Les guides montrent l'usage des plus courantes, et [les types de votre build](https://code.claude.com/docs/en/plugins/mods/create#get-the-types-for-your-build) documentent chaque méthode avec un exemple.

| Namespace | Méthodes |
| :- | :- |
| `$.plugin` | `name`, `root` : le nom et le répertoire de ce plugin |
| [`$.ui`](https://code.claude.com/docs/en/plugins/mods/interface#pick-where-to-draw) | `resolve`, `invalidate`, `open`, `close`, `panes`, `focus`, `scroll`, `toast`, `status`, `log`, `notice`, `ask`, `copy`, `selection`, `blit` |
| [`$.command`](https://code.claude.com/docs/en/plugins/mods/api#add-a-command) | `register`, `run`, `list` |
| [`$.tool`](https://code.claude.com/docs/en/plugins/mods/api#add-a-tool) | `register`, `call`, `check`, `list` |
| `$.agent` | `register`, `spawn`, `list`. `list()` renvoie les sous-agents et coéquipiers de cette session, chacun avec un `status` valant `pending`, `running`, `waiting`, `idle`, `completed`, `failed` ou `killed` ; `idle` et `waiting` : v2.1.289+. |
| [`$.model`](https://code.claude.com/docs/en/plugins/mods/api#call-a-model) | `complete`, `fork`, `classify` |
| [`$.prompt`](https://code.claude.com/docs/en/plugins/mods/api#start-a-turn-from-a-background-job) | `submit`, `read`, `fill`, `suggest`, `compose`. Claude lit le texte de `submit({ text })` après une phrase qui désigne votre mod comme expéditeur. `submit({ text, asUser: true })` envoie le texte comme les propres mots de l'utilisateur, sans cette phrase. |
| `$.turn` | `abort` |
| [`$.session`](https://code.claude.com/docs/en/plugins/mods/api#send-and-receive-messages-between-sessions) | `messages`, `cwd`, `root`, `model`, `turns`, `id`, `repo`, `surfaces`, `usage`, `version`, `compact`, `send`, `append`, `authorize`. `usage()` renvoie `{ startedAt, context, rateLimits, cost }` : `context` contient `tokens`, `window` et `percent`, et `rateLimits` est une liste de `{ kind, percentUsed, resetsAt }`. |
| `$.config` | `list`, `set` |
| [`$.settings`](https://code.claude.com/docs/en/plugins/mods/api#reach-files-processes-and-the-network) | `read` |
| [`$.env`](https://code.claude.com/docs/en/plugins/mods/api#reach-files-processes-and-the-network) | `get`, `set` |
| [`$.fs`](https://code.claude.com/docs/en/plugins/mods/api#reach-files-processes-and-the-network) | `read`, `write`, `list`, `exists`, `stat`, `ancestors`. `write` n'est pas atomique : il remplace le contenu du fichier sur place, donc un autre processus peut lire un fichier partiellement écrit. Conservez dans `$.store` les données que plusieurs sessions modifient. |
| [`$.store`](https://code.claude.com/docs/en/plugins/mods/interface#keep-state) | `get`, `set`, `delete`, `keys`. Un magasin clé-valeur partagé par toutes les sessions de la machine. Voir [Enregistrer depuis plusieurs sessions](https://code.claude.com/docs/en/plugins/mods/interface#save-from-more-than-one-session). |
| [`$.state`](https://code.claude.com/docs/en/plugins/mods/interface#keep-a-value-in-\$-state) | État réactif : `get`, `set`, avec les helpers `atom`, `read`, `update`, `derive` et `memberOf` importés depuis `claude-code` |
| [`$.clock`](https://code.claude.com/docs/en/plugins/mods/api#run-work-in-the-background) | `now`, `sleep`, `after`, `every` |
| [`$.http`](https://code.claude.com/docs/en/plugins/mods/api#reach-files-processes-and-the-network) | `fetch` |
| [`$.process`](https://code.claude.com/docs/en/plugins/mods/api#reach-files-processes-and-the-network) | `run`, `spawn` |
| [`$.mcp`](https://code.claude.com/docs/en/plugins/mods/api#reach-files-processes-and-the-network) | `call`, `connect`. `connect(server)` connecte un serveur MCP que liste le manifeste de votre propre plugin. |
| `$.audio` | `play`, `speak` |
| `$.telemetry` | `log`, `mark`. Un enregistrement n'est envoyé que lorsque l'appel est fait par Claude Code ou un mod intégré. |

## Sites de rendu

Un site de rendu est un point d'extension de l'interface de Claude Code. Chaque ligne est une valeur de `e.component` dans un hook `ui.render`, avec les champs de `e.props` et les applications qui le rendent. `e.surface` vaut `terminal` ou `desktop`. [Changer ce que Claude Code dessine déjà](https://code.claude.com/docs/en/plugins/mods/interface#change-what-claude-code-already-draws) montre ce qu'un hook peut faire sur un site, avec un exemple de chaque choix.

| Site | `e.props` | `e.requestId` | Rendu dans |
| :- | :- | :- | :- |
| [`Pane`](https://code.claude.com/docs/en/plugins/mods/interface#pick-where-to-draw) | `title`, `isFocused`, `bodyColumns`, `placement`, `scroll`, `view` | L'`id` du panneau | Terminal, Desktop |
| [`AbovePrompt`](https://code.claude.com/docs/en/plugins/mods/interface#pick-where-to-draw) | `hasSurvey`, `isWorking`, `maxRows`, `bodyColumns`, `scroll`, `view` | Une seule instance | Terminal, Desktop |
| `UserMessage` | `text`, `origin`, `isExpanded`, et `task` ou `from` selon l'origine | L'id du message | Terminal, Desktop |
| `AssistantMessage` | Le texte de la réponse | L'id du message | Terminal, Desktop |
| `ToolUse`, `ToolResult`, `ToolGroup` | Le nom de l'outil, son entrée et son résultat | L'id de l'appel d'outil | Terminal, Desktop |
| `CommandOutput` | `command`, `text` | L'id du message | Terminal, Desktop |
| [`AskUserQuestion`](https://code.claude.com/docs/en/plugins/mods/interface#change-what-claude-code-already-draws) | La question et les options | L'id de l'appel d'outil | Terminal, Desktop |
| `ToolProgress` | `kind` | L'id de l'appel d'outil | Terminal |
| [`Spinner`](https://code.claude.com/docs/en/plugins/mods/interface#change-what-claude-code-already-draws) | `word`, `message`, `suffix`, `mode` | L'id de l'agent | Terminal, Desktop |
| `TurnDuration` | `word`, `durationMs` | L'id du message | Terminal |
| `InfoNotice` | `text`, `command` | L'id du message | Terminal |
| `SessionMode` | `modes` | Une seule instance | Terminal, Desktop |
| `PromptHint` | `isDraft`, `isWorking`, `hint` | Une seule instance | Terminal, Desktop |

`e.viewport` contient `columns`, `rows` et `isFullscreen`. Il est absent tant que l'application n'a pas mesuré sa fenêtre. Son `rows` est la hauteur de toute la fenêtre, pas celle de votre panneau.

Pour adapter un arbre à son site, lisez ces props dans le hook :

* **Largeur d'un `Pane` ou de la bande** : dessinez sur `e.props.bodyColumns` colonnes
* **Hauteur d'un `Pane` à côté de la transcription** : là où `e.props.placement` vaut `'dock'`, `e.props.scroll.bodyRows` est le nombre de lignes dont dispose le panneau pour votre arbre
* **Hauteur d'un `Pane` au-dessus du prompt** : là où `e.props.placement` vaut `'inline'`, le panneau grandit avec votre arbre jusqu'à une limite, et `bodyRows` est cette limite. Le [champ `rows` de `$.ui.open`](https://code.claude.com/docs/en/plugins/mods/interface#open-a-pane-at-the-right-time) en demande une autre.

Un arbre plus haut que le panneau défile d'un seul bloc.

## Éléments

Les éléments sont les briques de l'arbre que renvoie un hook `ui.render` ; on les obtient avec `$.ui.resolve(e)`. [Construire un arbre à partir d'éléments](https://code.claude.com/docs/en/plugins/mods/interface#build-a-tree-from-elements) montre les plus courants avec leur rendu dans le terminal, et la [galerie de l'interface](https://code.claude.com/docs/en/plugins/mods/gallery) contient des captures de la plupart. Une coche signifie que l'application sait dessiner l'élément.

| Élément | Props principales | Terminal | Desktop |
| :- | :- | :-: | :-: |
| [`Box`](https://code.claude.com/docs/en/plugins/mods/interface#build-a-tree-from-elements) | `key`, mise en page flex, `gap`, `padding`, `margin`, `width`, `height`, [`borderStyle`](#styles-de-bordure-de-box), `backgroundColor`, `position`, `hover` | ✓ | ✓ |
| [`Text`](https://code.claude.com/docs/en/plugins/mods/interface#build-a-tree-from-elements) | `color`, `backgroundColor`, `bold`, `italic`, `underline`, `dimColor`, `inverse`, `wrap` | ✓ | ✓ |
| [`Button`](https://code.claude.com/docs/en/plugins/mods/interface#respond-to-presses-and-typing) | `key`, `label`, `onPress`, `hotkey`, `plain`, `dimColor`, `autoFocus`, `action` | ✓ | ✓ |
| [`Link`](https://code.claude.com/docs/en/plugins/mods/interface#link-in-the-desktop-app) | `href`, `label`. Voir [Limites](#limites). | ✓ | ✓ |
| [`Code`](https://code.claude.com/docs/en/plugins/mods/gallery#show-code-and-changes) | `source`, `language`, `path`, `startLine`, `format`, `wrap` | ✓ | ✓ |
| `Markdown` | `text`, `key`, `dimColor`, `onLinkPress`, `pressableLinks` | ✓ | ✓ |
| [`Input`](https://code.claude.com/docs/en/plugins/mods/interface#take-typed-input-and-draw-a-row-for-each-item) | `key`, `label`, `placeholder`, `value`, `submitLabel`, `onSubmit`, `onInput`, `autoFocus` | ✓ | ✓ |
| `Select` | `key`, `label`, `options`, `value`, `onSelect`, `autoFocus` | ✓ | ✓ |
| `Svg` | Un document SVG, jusqu'à 131 072 caractères | | ✓ |
| [`Client`](https://code.claude.com/docs/en/plugins/mods/interface#build-a-tree-from-elements) | `module`, `key` | ✓ | ✓ |
| [`Raster`](https://code.claude.com/docs/en/plugins/mods/interface#draw-a-grid-of-colored-cells) | `key`, `columns` jusqu'à 512, `rows` jusqu'à 256, `cells`. Voir [Dessiner une grille de cellules colorées](https://code.claude.com/docs/en/plugins/mods/interface#draw-a-grid-of-colored-cells). | ✓ | |
| `Image` | Octets PNG ou RGBA jusqu'à 2 Mio, ou un chemin de fichier, `columns` et `rows` jusqu'à 255, et un texte `alt`. | ✓ | |

Autres règles de `Button` : `action` nomme l'une des [actions de raccourci clavier](https://code.claude.com/docs/en/keybindings) propres à Claude Code, et le raccourci de l'utilisateur pour cette action presse le bouton lorsque ce raccourci est un accord (chord) ou une touche avec modificateur. Un `hotkey` numérique sur un bouton de la bande se déclenche aussi quand l'utilisateur tape ce seul chiffre dans un prompt vide puis marque une pause. Quand deux boutons d'un même dessin déclarent le même `hotkey`, c'est le dernier qui l'obtient. `autoFocus` n'accepte que `true` sur tous les contrôles : omettez la prop pour le désactiver.

### Styles de bordure de `Box`

Pour dessiner une bordure autour d'une `Box`, réglez sa prop `borderStyle` sur l'un de ces noms, comme dans `borderStyle: 'round'`. Chaque ligne indique ce que le terminal dessine pour ce nom et montre le bord supérieur de la bordure.

| `borderStyle` | Ce que dessine le terminal | Bord supérieur |
| :- | :- | :- |
| `'single'` | Traits fins avec coins carrés | `┌──┐` |
| `'double'` | Traits doubles | `╔══╗` |
| `'round'` | Traits fins avec coins arrondis | `╭──╮` |
| `'bold'` | Traits épais | `┏━━┓` |
| `'singleDouble'` | Traits fins en haut et en bas, traits doubles sur les côtés | `╓──╖` |
| `'doubleSingle'` | Traits doubles en haut et en bas, traits fins sur les côtés | `╒══╕` |
| `'classic'` | Les caractères ASCII `+`, `-` et `\|` | `+--+` |
| `'arrow'` | Des flèches pointant vers l'intérieur de la `Box` | `↘↓↓↙` |
| `'dashed'` | Traits pointillés avec coins vides | `╌╌` |
| `'quote'` | Une barre, `▎`, le long du côté gauche, et des cellules vides sur les trois autres côtés | Vide |

Une `Box` dont le `borderStyle` porte un autre nom, comme `'rounded'`, se dessine sans bordure.

## Limites

Les hooks et les appels à l'API des mods s'exécutent sous des limites de temps et de taille. Claude Code saute un hook qui dépasse une limite de temps.

| Limite | Valeur |
| :- | :- |
| Temps d'exécution propre d'un hook pour un événement, sans compter le temps passé dans `next` ou dans un appel à l'API des mods autre que `$.clock.sleep` | 10 secondes, ou 50 millisecondes pour un hook `prompt.edit` |
| Temps d'exécution d'un gestionnaire `.catch` | 1 seconde |
| Tous les hooks `session.end` ensemble | Autant que le [budget du hook SessionEnd](https://code.claude.com/docs/en/hooks#sessionend-input), 1,5 seconde sauf si vous le modifiez, compté à partir de la fin de vos hooks de réglages `SessionEnd` |
| Délai d'expiration de `$.process.run` | 30 secondes par défaut, 10 minutes au maximum |
| `maxTokens` de `$.model.complete` | 1024 par défaut, jusqu'à 64 000 ou la limite de sortie du modèle |
| `$.fs.read` et `$.fs.write` | 4 Mio par fichier |
| Corps de requête de `$.http.fetch` | 4 Mio, comptés en caractères. Un appel avec un corps plus grand est rejeté. |
| Corps de réponse de `$.http.fetch` | 4 Mio. `text` contient les 4 premiers Mio et le reste n'est pas lu. Quand l'en-tête `Content-Length` annonce davantage, l'appel est rejeté à la place, avec une raison qui se termine par `is over the 4194304-byte limit`, sauf si la dernière requête après d'éventuelles redirections utilise la méthode `HEAD`. Exemption `HEAD` : v2.1.296+. |
| Un appel `$.http.fetch`, redirections et corps compris | 30 secondes |
| Redirections suivies par un appel `$.http.fetch` | 5 |
| Raison de `drop` d'un hook ou raison de `deny` de `config.set` | 4 096 caractères. La fin d'une raison plus longue est coupée, et le drop ou le deny s'applique quand même. Coupure : v2.1.292+ ; dans les versions antérieures, le hook [échoue](https://code.claude.com/docs/en/plugins/mods/events#handle-a-hook-that-fails) à la place. |
| Texte dans un arbre | Les 100 000 premiers caractères sont dessinés |
| `language` ou `path` d'un `Code`, `value` d'une option de `Select`, ou `module` d'un `Client` | 10 000 caractères. Si l'un est plus long, Claude Code [dessine sa propre version du site](https://code.claude.com/docs/en/plugins/mods/interface#build-a-tree-from-elements). |
| `href` d'un `Link` | 2 048 caractères. Un `href` plus long empêche tout l'arbre de se dessiner. |
| `$.store` | 4 Mio de JSON au total |
| `$.session.messages()` | Les 4 096 entrées les plus récentes |
| Redessins via `$.ui.invalidate('ui.render')` | Limités à 10 par seconde, ou 30 dans le terminal pour le panneau visible, la bande dépliée et la ligne d'indice sous le prompt. Les appels plus rapprochés sont fusionnés. |
| `$.ui.toast` | Affiché 4 secondes sauf si vous passez `{ timeoutMs }` |
| Un panneau ouvert sans que l'utilisateur le demande | Placé à partir de 144 colonnes de terminal, 110 une fois que l'utilisateur l'a ouvert lui-même une fois |
| Portées (fonctions, blocs, boucles…) imbriquées les unes dans les autres dans un même fichier d'un module de hooks | 2 000 |
| Noms de commande, d'outil, de type de sous-agent et de panneau | Lettres, chiffres, `_` et `-`, jusqu'à 64 caractères |
| Un test `claude plugin test` | 5 secondes sauf si le test définit `timeoutMs` |

## Réglages et variables d'environnement

Voici les réglages et variables d'environnement qui affectent les mods. La colonne « Où » indique dans quel fichier de réglages ou quel environnement chacun est lu :

| Nom | Où | Effet |
| :- | :- | :- |
| `CLAUDE_CODE_PLUGIN_DIRS` | Environnement, ou `env` dans `~/.claude/settings.json` | Répertoires de plugins à charger comme le fait `--plugin-dir`, pour les applications auxquelles on ne peut pas passer de flag. Chemins absolus séparés par `:`, ou `;` sous Windows. |
| `CLAUDE_CODE_PLUGIN_DIR_WATCH` | Environnement | `1` fait qu'une session non interactive de longue durée recharge les mods `--plugin-dir` à chaque enregistrement |
| `prependPlugins`, `appendPlugins` | Réglages gérés. Réglages utilisateur uniquement sur une machine sans réglages gérés, pour un utilisateur qui n'est pas connecté avec un forfait Team ou Enterprise. | Listes d'identifiants de plugins, comme `acme-guard@acme-tools`. Les mods de `prependPlugins` s'exécutent avant tous les mods qu'installe un utilisateur, et ceux d'`appendPlugins` après, dans l'ordre listé. Voir [L'ordre d'exécution des mods](https://code.claude.com/docs/en/plugins/mods/events#the-order-mods-run-in). |
| `allowManagedModsOnly` | Réglages gérés, en tant qu'[option de la protection intégrée](https://code.claude.com/docs/en/plugins/mods/admin#set-options-on-the-built-in-guard) | Seuls les mods qui [comptent comme ceux de votre organisation](https://code.claude.com/docs/en/plugins/mods/admin#install-your-organizations-mods), et les mods intégrés à Claude Code, exécutent leurs hooks. Les hooks de réglages des utilisateurs continuent de s'exécuter. |
| `allowModsToOverrideDenyRules` | Réglages gérés, en tant qu'[option de la protection intégrée](https://code.claude.com/docs/en/plugins/mods/admin#set-options-on-the-built-in-guard) | Permet à un mod installé par un utilisateur d'approuver un appel d'outil qu'une règle `deny` refuse |
| `allowManagedHooksOnly` | Réglages gérés | Bloque les hooks et les mods installés qui n'appartiennent pas à votre organisation. Voir [ce qui continue de s'exécuter](https://code.claude.com/docs/en/settings-reference#what-runs-under-allowmanagedhooksonly). |
| `disableAllHooks` | N'importe quel fichier de réglages | Dans les réglages gérés, aucun mod ni hook d'un plugin installé ne s'exécute. Dans vos propres réglages, ce que gère votre organisation continue de s'exécuter. Voir [`disableAllHooks`](https://code.claude.com/docs/en/settings-reference#disableallhooks). |
| `disableSideloadFlags` | Réglages gérés | Rejette `--plugin-dir` et `--plugin-url` au démarrage |
| `pluginConfigs` | Réglages utilisateur ou gérés | Contient les valeurs `userConfig` d'un mod, indexées par l'identifiant du plugin, comme `acme-guard@acme-tools`, ou par son nom suivi de `@inline`, comme `first-mod@inline`, pour un mod chargé avec `--plugin-dir` |

`sec-default@builtin` est une protection intégrée à Claude Code, listée comme `cc-plugin-sec-default` dans `/plugin` et dans le journal de débogage. Elle se charge avant tout mod qu'installe une personne sur une machine dotée de réglages gérés, ou pour un utilisateur connecté avec un forfait Team ou Enterprise. Si `prependPlugins` est défini dans les réglages gérés, la protection ne se charge que si cette liste la nomme, à la position indiquée. Son code source se trouve dans le [répertoire `mods/sec-default` du dépôt Claude Code](https://github.com/anthropics/claude-code/tree/main/mods/sec-default).

## Commandes

Ces commandes et flags chargent, inspectent et testent un mod. Les commandes `claude` s'exécutent dans votre shell, et les commandes `/` au prompt de Claude Code. Dans le tableau, `<directory>` représente un chemin que vous tapez, comme dans `claude plugin validate ./first-mod`. Les crochets marquent un argument facultatif.

| Commande | Effet |
| :- | :- |
| [`/plugin`](https://code.claude.com/docs/en/plugins/mods/overview#see-which-mods-a-session-loaded) | Affiche une ligne comme `1 mod active · first-mod` sous ses onglets quand un mod non intégré s'est chargé |
| [`claude plugin validate <directory>`](https://code.claude.com/docs/en/plugins/mods/create#check-what-claude-code-reads-from-your-mod) | Lit le manifeste et le module de hooks d'un plugin et signale les erreurs, les événements qu'il gère et les appels à l'API des mods qu'il effectue. `--strict` traite les avertissements comme des erreurs et `--json` affiche un rapport lisible par machine. |
| [`claude plugin test [directory]`](https://code.claude.com/docs/en/plugins/mods/test#write-a-test) | Exécute chaque fichier du répertoire (ou du répertoire courant si vous n'en donnez pas) dont le nom se termine par `.test.ts` ou `.test.tsx`. Se termine avec le statut 1 quand un test échoue. |
| [`claude --plugin-dir <directory>`](https://code.claude.com/docs/en/plugins/mods/create#write-a-mod-yourself) | Charge un répertoire de plugin pour une session et recharge son module de hooks à chaque enregistrement. Répétez le flag pour en charger plusieurs. |
| `/reload-plugins` | Recharge les plugins au moment où vous l'exécutez |
