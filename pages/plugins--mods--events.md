# Réagir aux événements avec un mod

> Source : https://code.claude.com/docs/en/plugins/mods/events · [← Index](../31-index-documentation-officielle.md)

*Traiter les événements de Claude Code depuis un mod : observer, réécrire ou répondre aux appels d'outils, prompts et tours, filtrer les événements que traite un hook, et prévoir la cohabitation avec d'autres mods.*

Un hook est un gestionnaire d'événement : une fonction que Claude Code exécute quand un événement nommé se produit. Claude Code déclenche un événement à chaque point où il s'apprête à agir, par exemple quand il exécute un outil, soumet un prompt, envoie une requête au modèle, ou démarre ou termine une session. Votre hook s'exécute avant que Claude Code n'agisse, il peut donc observer l'événement, le réécrire, ou y répondre à la place de Claude Code. Vous enregistrez un hook avec [`on(eventName, handler)`](https://code.claude.com/docs/en/plugins/mods/reference#the-hook-function).

Construisez votre [premier mod](https://code.claude.com/docs/en/plugins/mods/create) avant de commencer ici. Pour chaque événement et ses champs exacts, voir la [référence](https://code.claude.com/docs/en/plugins/mods/reference#events) ou lisez [les types de votre build](https://code.claude.com/docs/en/plugins/mods/create#get-the-types-for-your-build).

## Comment un hook traite un événement

Un hook se place entre un événement et ce que Claude Code ferait en réponse, il peut donc observer l'événement, le réécrire, ou y répondre lui-même. Il reçoit trois arguments : l'[API des mods](https://code.claude.com/docs/en/plugins/mods/api) sous le nom `$`, l'événement sous le nom `e`, et le gestionnaire suivant sous le nom `next`. Les gestionnaires d'un événement forment une chaîne de middlewares. `next(e)` appelle le gestionnaire suivant, qui est le hook d'un autre mod ou, en fin de chaîne, le comportement propre de Claude Code, et se résout avec le résultat. Ce que votre hook fait de `next` détermine laquelle des trois actions il effectue.

### Observer un événement

Pour observer un événement sans le modifier, faites votre travail et renvoyez `next(e)`. Ce hook journalise chaque outil que Claude s'apprête à utiliser :

```javascript theme={null}
on('tool.call', async ($, e, next) => {
  // S'exécute avant l'outil
  $.ui.log('Claude is about to use ' + e.tool)
  // Transmet l'événement inchangé
  return next(e)
})
```

Avant chaque exécution d'outil, une ligne atténuée telle que `● my-mod: Claude is about to use Bash` apparaît dans la transcription, où `my-mod` est le nom de votre plugin. L'outil s'exécute comme il le ferait sans le mod.

Pour agir après l'événement, faites `await next(e)`, faites votre travail, et renvoyez le résultat. Ce hook journalise chaque outil après son exécution :

```javascript theme={null}
on('tool.call', async ($, e, next) => {
  // Laisse l'outil s'exécuter, et attend son résultat
  const result = await next(e)
  // S'exécute après l'outil
  $.ui.log(e.tool + ' finished')
  // Rend le résultat inchangé
  return result
})
```

La ligne apparaît désormais après la fin de chaque outil. Claude lit le même résultat dans les deux cas, car le hook renvoie ce en quoi `next(e)` s'est résolu.

### Réécrire un événement

Pour modifier ce sur quoi Claude Code agit, comme le texte d'un prompt, appelez `next` avec une copie modifiée de l'événement. L'événement lui-même est immuable : il est profondément gelé, et affecter un champ lève une exception. Ce hook supprime les espaces en début et fin de chaque prompt avant son envoi :

```javascript theme={null}
on('prompt.submit', async ($, e, next) => {
  // Transmet une copie de l'événement avec son texte modifié
  return next({ ...e, text: e.text.trim() })
})
```

Les gestionnaires suivants et Claude Code reçoivent le prompt nettoyé et ne voient jamais l'original. Vous pouvez aussi modifier le résultat : `await next(e)`, puis renvoyez une copie du résultat avec un champ remplacé.

### Répondre à un événement

Pour traiter un événement vous-même, renvoyez un résultat sans appeler `next`. Cela court-circuite la chaîne, donc les mods suivants et le comportement propre de Claude Code ne s'exécutent pas. Ce hook refuse toute commande Bash :

```javascript theme={null}
on('tool.call', { tool: 'Bash' }, async () => {
  // Aucun appel à next, donc la commande ne s'exécute jamais
  return { deny: 'Bash is turned off in this project. Use the file tools.' }
})
```

Quand Claude tente une commande Bash, la commande ne s'exécute pas, et Claude lit le texte de `deny` comme résultat de l'outil. Chaque événement a sa propre forme de résultat, que liste la [référence des événements](https://code.claude.com/docs/en/plugins/mods/reference#events).

### Filtrer les événements que traite un hook

Pour n'exécuter un hook que pour certains événements, passez un filtre en deuxième argument de `on`. Claude Code appelle ce filtre un matcher. C'est un objet dont les champs sont comparés à ceux de l'événement, et le hook ne s'exécute que lorsque tous les champs correspondent. Un champ peut être une valeur, un tableau de valeurs autorisées, ou une expression régulière.

Chaque ligne de cet exemple enregistre la même fonction, `hook`, pour un ensemble plus restreint d'appels d'outils :

```javascript theme={null}
// Une chaîne correspond à une valeur : les appels Bash uniquement
on('tool.call', { tool: 'Bash' }, hook)
// Un tableau correspond à n'importe laquelle de ses valeurs : les appels Edit et Write
on('tool.call', { tool: ['Edit', 'Write'] }, hook)
// Une expression régulière correspond par motif : tous les outils d'un serveur MCP
on('tool.call', { tool: /^mcp__github__/ }, hook)
```

`hook` s'exécute une fois pour un appel Bash, Edit ou Write, et une fois pour un appel à un outil dont le nom commence par `mcp__github__`. Un appel à tout autre outil, comme Read, ne correspond à aucun des trois, donc `hook` ne s'exécute pas pour lui.

Le nom de l'événement peut être un joker. `'classic.*'` correspond à chaque [événement de hook des réglages](#traiter-les-événements-de-hooks-des-réglages). `'*'` correspond à tous les événements sauf les [événements de télémétrie](https://code.claude.com/docs/en/plugins/mods/reference#telemetry), qui prennent leur propre nom et un filtre `{ to: 'collector' }`.

Enregistrez chaque événement une seule fois par matcher. Si vous appelez `on` deux fois pour `session.start` sans matcher, le module échoue au chargement avec `on("session.start") is registered twice without a matcher`. Regroupez tout ce que fait votre mod au démarrage de la session dans un seul hook.

## Accrocher ce que fait Claude

Traitez ces événements pour voir ou modifier un appel d'outil, un prompt ou un tour pendant qu'il se produit. Pour chaque événement et ce qu'un hook peut renvoyer, voir la [référence des événements](https://code.claude.com/docs/en/plugins/mods/reference#events).

### Garder ou modifier un appel d'outil

Un hook `tool.call` voit chaque outil que Claude s'apprête à utiliser, il peut donc refuser l'appel, modifier ses arguments, ou le laisser passer. `tool.call` se déclenche quand Claude Code s'apprête à exécuter un outil, y compris les appels faits par un sous-agent et les appels aux outils MCP. `e.tool` est le nom de l'outil et les arguments de l'outil sont des champs de `e`, comme `e.command` pour Bash. Quand vous appelez `next(e)`, Claude Code effectue la vérification des permissions puis exécute l'outil.

Ce hook refuse une commande Bash qui fait un push forcé, et explique pourquoi à Claude :

```javascript theme={null}
// Le matcher limite le hook aux appels Bash, donc e.command est la commande shell
on('tool.call', { tool: 'Bash' }, async ($, e, next) => {
  if (/git push .*--force/.test(e.command)) {
    // Retourner sans appeler next répond à l'événement, donc la commande ne s'exécute jamais
    return { deny: 'Force pushes are not allowed in this repository. Push to a new branch instead.' }
  }
  // Toute autre commande passe à la vérification des permissions puis à Bash
  return next(e)
})
```

Quand Claude tente `git push --force`, la commande ne s'exécute pas et aucune demande de permission n'apparaît, car le hook n'appelle jamais `next`. Claude lit le texte de `deny` comme résultat de l'outil : rédigez-le donc comme une instruction sur laquelle Claude peut agir. Toute autre commande Bash s'exécute comme sans le mod.

Pour agir après l'exécution d'un outil, faites `await next(e)`, faites votre travail, et renvoyez ce que `next` vous a donné. Ce hook journalise chaque fichier `.mdx` que Claude modifie, avec [`$.ui.log`](https://code.claude.com/docs/en/plugins/mods/api#show-something-without-starting-a-turn), qui ajoute à la transcription une ligne atténuée que Claude ne lit pas :

```javascript theme={null}
on('tool.call', { tool: ['Edit', 'Write'] }, async ($, e, next) => {
  // Attend la vérification des permissions et l'outil, et garde ce qu'ils ont produit
  const result = await next(e)
  // Un appel refusé revient sous la forme { deny }, et un appel échoué a isError défini
  const changed = !result.deny && !result.isError
  if (changed && e.file_path.endsWith('.mdx')) $.ui.log('Claude changed ' + e.file_path)
  // Renvoie le résultat tel quel, pour que Claude lise ce que l'outil a renvoyé
  return result
})
```

Après que Claude a modifié ou écrit un fichier `.mdx`, une ligne atténuée de la transcription nomme le fichier. Rien n'est journalisé pour un autre type de fichier, ni pour un appel refusé ou échoué. La vision qu'a Claude de l'appel ne change pas, car le hook renvoie le résultat qu'il a reçu.

Votre hook peut aussi modifier un appel, le réessayer, y répondre lui-même, ou retenir son résultat :

* **Modifier l'appel** : passez des arguments modifiés à `next`.
* **Réessayer l'appel** : appelez de nouveau `next(e)`. Un hook qui voit `isError` sur le premier résultat peut exécuter l'outil une seconde fois et renvoyer ce résultat.
* **Répondre vous-même à l'appel** : renvoyez un objet avec un champ `result`, sans appeler `next`, et pour un outil intégré, donnez à `result` la forme qu'a le résultat propre de cet outil dans [les types de votre build](https://code.claude.com/docs/en/plugins/mods/create#get-the-types-for-your-build). Aucune demande de permission n'apparaît et l'outil ne s'exécute pas, donc le résultat que vous renvoyez est tout ce que Claude apprend de ce qui s'est passé.
* **Retenir le résultat pour Claude** : renvoyez `{ deny: reason }` après `await next(e)`. Claude lit votre raison à la place de ce que `next` a renvoyé. Quand l'outil s'est exécuté, le deny cache son résultat à Claude et n'annule rien de ce que l'outil a fait. Quand l'outil s'est exécuté avec succès, la raison suit une note telle que `Bash ran, and a plugin withheld its result:`.

Les hooks des [réglages gérés](https://code.claude.com/docs/en/server-managed-settings) de votre organisation s'exécutent avant le hook `tool.call` de tout mod, et un blocage de l'un d'eux est définitif.

#### Suspendre un appel d'outil jusqu'à la décision de l'utilisateur

Un hook peut mettre en pause un appel d'outil et demander à l'utilisateur quoi faire avant de continuer. Un hook `tool.call` peut faire un `await` avant d'appeler `next` ou de retourner, et l'appel d'outil reste en attente jusque-là. Pour poser la question à l'utilisateur, appelez `$.ui.ask`. Il affiche votre question au-dessus d'une liste numérotée de vos options, dans la boîte de dialogue que Claude utilise pour vous demander quelque chose, et se résout avec le libellé que l'utilisateur choisit. Après vos options, la boîte de dialogue ajoute une ligne pour taper une autre réponse et une ligne **Chat about this**.

Le motif `RISKY` de cet exemple correspond à `rm -r`, `rm -rf`, `git reset --hard`, et `git push` avec `--force`, et il manque d'autres écritures comme `git push -f`. Ce module demande confirmation avant d'exécuter une commande Bash qui correspond au motif :

```javascript theme={null}
const RISKY = /\brm\s+-rf?\b|\bgit\s+reset\s+--hard\b|\bgit\s+push\b.*--force/

export function register(on) {
  on('tool.call', { tool: 'Bash' }, async ($, e, next) => {
    // Laisse passer toute autre commande sans question
    if (!RISKY.test(e.command)) return next(e)
    // Part de la réponse sûre, pour qu'une question sans réponse refuse la commande
    let answer = 'Refuse'
    try {
      // L'appel d'outil attend ici jusqu'à ce que l'utilisateur choisisse l'un des deux libellés
      answer = await $.ui.ask('Run this command? ' + e.command, ['Run it', 'Refuse'])
    } catch {
      // L'utilisateur a fermé la question, ou c'est une exécution claude -p sans personne à qui demander
    }
    if (answer !== 'Run it') {
      // Répond sans appeler next, donc la commande ne s'exécute pas
      return { deny: 'The user declined this command. Ask before trying a different approach.' }
    }
    return next(e)
  })
}
```

Quand Claude tente une commande comme `rm -rf build`, la question apparaît avec la commande, et la commande attend la réponse :

* **L'utilisateur choisit Run it** : le hook appelle `next(e)`, et la vérification habituelle des permissions s'exécute encore ensuite
* **L'utilisateur choisit Refuse** : la commande ne s'exécute pas, et Claude lit le texte de `deny`
* **L'utilisateur tape une réponse** : `$.ui.ask` se résout avec le texte tapé. Le hook le compare à `Run it`, donc tout autre texte refuse la commande.
* **Personne ne répond** : `$.ui.ask` est rejeté quand l'utilisateur ferme la question ou choisit **Chat about this**, ainsi que dans une exécution `claude -p`, donc le bloc `catch` laisse la réponse à `Refuse`

Gardez l'attente à l'intérieur d'un appel à l'API des mods comme `$.ui.ask`, car ce temps ne compte pas dans la [limite de temps](https://code.claude.com/docs/en/plugins/mods/reference#limits) du hook. Le temps passé à attendre une promesse à vous compte, lui. Claude Code ignore un hook qui dépasse le délai, et la commande suspendue s'exécuterait alors.

#### Approuver ou refuser un appel d'outil avant que l'utilisateur ne soit sollicité

Pour décider si un appel d'outil peut s'exécuter, traitez [`tool.check`](https://code.claude.com/docs/en/plugins/mods/reference#tools), l'événement où Claude Code prend cette décision. Il se déclenche après que les règles de permission et les hooks des réglages ont décidé, et `next(e)` se résout avec leur décision : `allow`, `ask` ou `deny`. Votre hook renvoie cette décision ou une autre. `e.input` contient les arguments de l'outil, comme `command` pour Bash.

Pour une commande ou un chemin fixe, utilisez une [règle de permission](https://code.claude.com/docs/en/permissions#permission-rule-syntax) comme `Bash(npm test)`, qui ne demande aucun code. Traitez `tool.check` quand la décision dépend de ce qui est vrai à ce moment, comme la branche Git courante ou une valeur enregistrée par un autre hook.

Ce hook refuse `git push` tant que la branche courante est `main` :

```javascript theme={null}
on('tool.check', { tool: 'Bash' }, async ($, e, next) => {
  // Ce qu'ont décidé les règles de permission et les hooks des réglages : 'allow', 'ask' ou 'deny'
  const decided = await next(e)
  if (!e.input.command.includes('git push')) return decided
  const branch = await $.process.run(['git', 'branch', '--show-current'])
  if (branch.stdout.trim() !== 'main') return decided
  return { decision: 'deny', reason: 'Push from a branch other than main' }
})
```

Sur `main`, le hook renvoie `deny`, même quand une règle autorise `git push`. Sur une autre branche, et pour les autres commandes, l'appel obtient la décision qu'il obtiendrait sans le mod.

Le hook se base sur le texte de la commande, traitez-le donc comme un rappel pour Claude. Pour bloquer les pushs vers `main` pour tout le monde, protégez la branche sur votre hébergeur Git.

Un hook peut renvoyer `allow`, `ask` ou `deny`, il peut donc aussi approuver un appel qu'un hook `PreToolUse` hors réglages gérés a bloqué. [Étendre les permissions avec des hooks](https://code.claude.com/docs/en/permissions#extend-permissions-with-hooks) liste les décisions qui prévalent sur un mod.

### Réécrire un prompt ou y ajouter du contenu

Un hook `prompt.submit` voit chaque prompt avant le début du tour, il peut donc réécrire le texte ou y ajouter du contenu. `e.text` est ce qui a été tapé.

| Pour faire ceci | Renvoyez ceci |
| :- | :- |
| Réécrire le prompt. La transcription et votre [historique des prompts](https://code.claude.com/docs/en/interactive-mode#command-history) affichent le nouveau texte. | `next({ ...e, text: newText })` |
| Ajouter, après le prompt, du texte que seul Claude lit | `next({ ...e, context: [...(e.context ?? []), extraText] })` |
| Empêcher l'envoi du prompt | `{ drop: 'the reason' }` |

Ce hook ajoute pour Claude le nom de la branche courante chaque fois qu'un prompt mentionne une pull request :

```javascript theme={null}
on('prompt.submit', async ($, e, next) => {
  // Transmet tel quel un prompt qui ne mentionne pas de pull request
  if (!/\bPR\b|pull request/i.test(e.text)) return next(e)
  const git = await $.process.run(['git', 'branch', '--show-current'])
  // Hors d'un dépôt git la commande échoue, donc il n'y a pas de branche à ajouter
  if (git.exitCode !== 0) return next(e)
  // Garde tout contexte ajouté par un hook précédent, et ajoute une ligne de plus pour Claude
  return next({ ...e, context: [...(e.context ?? []), 'Current branch: ' + git.stdout.trim()] })
})
```

Quand vous envoyez un prompt comme `open a PR for this change`, votre message a le même aspect dans la transcription, et Claude lit aussi une ligne comme `Current branch: feature/auth` après lui. Un prompt qui ne mentionne pas de pull request passe inchangé, et `git` ne s'exécute pas.

Pour arrêter un prompt, renvoyez `{ drop: 'the reason' }` sans appeler `next`. Le texte retourne dans la zone de saisie du prompt de l'utilisateur, qui voit `Prompt dropped by a hook:` suivi de votre raison : adressez donc la raison à l'utilisateur. Si votre hook renvoie un `drop` après que son appel `next(e)` a laissé passer le prompt, le tour s'exécute quand même, et le hook [échoue](#traiter-un-hook-qui-échoue) avec un message qui inclut `a drop after its next() was answered`.

D'[autres événements](https://code.claude.com/docs/en/plugins/mods/reference#prompts-and-what-claude-reads) couvrent le reste de ce que lit Claude : `prompt.section` pour chaque section du prompt système, `prompt.context` pour le contexte envoyé avec le premier message, et `skill.prompt` pour le texte d'un skill. Un texte issu de ces hooks qui change d'une requête à l'autre [invalide le cache de prompt](https://code.claude.com/docs/en/prompt-caching).

### Suivre un tour

Un tour est tout ce que fait Claude en réponse à un prompt. Traitez `turn.start`, `turn.step` et `turn.complete` pour en suivre un :

| Événement | Quand il se déclenche | Ce qu'un hook peut faire |
| :- | :- | :- |
| `turn.start` | Un tour commence | Observer. `e.turnId` identifie le tour dans les deux autres événements. |
| `turn.step` | Claude Code s'apprête à envoyer une requête au modèle. Un tour avec des appels d'outils en compte plusieurs. `e.agentId` est défini pour la requête d'un sous-agent. | Lire l'utilisation de tokens de chaque requête, l'envoyer à un autre modèle avec `next({ ...e, model })`, ou répondre sans appeler le modèle |
| `turn.complete` | Le tour s'est terminé, y compris un tour interrompu par l'utilisateur, où `e.isAborted` vaut `true`. `e.answer` est le texte final de Claude, `e.durationMs` sa durée, et `e.usage` les totaux de tokens du tour. Le tour d'un sous-agent le déclenche avec `e.agentId` défini. | Observer, ou renvoyer un objet avec un champ `text`, comme `{ text: 'Done in 12 seconds' }`, pour afficher une ligne sous la réponse |

Écrivez un hook `turn.step` comme un générateur asynchrone, car l'événement est diffusé en flux. `yield* next(e)` transmet la réponse au fil du flux et s'évalue en le résultat final. Ce hook journalise quelle part de chaque requête la Claude API a servie depuis le [cache de prompt](https://code.claude.com/docs/en/prompt-caching) :

```javascript theme={null}
// function* fait du hook un générateur, qui peut transmettre la réponse morceau par morceau
on('turn.step', async function* ($, e, next) {
  // Envoie la requête, transmet chaque morceau à son arrivée, et garde le résultat final
  const result = yield* next(e)
  // Ignore un résultat qui ne rapporte aucun compte de tokens
  if (result.usage) {
    $.ui.log('cache read ' + result.usage.cache_read_input_tokens + ' · wrote ' + result.usage.cache_creation_input_tokens)
  }
  // Renvoie le résultat inchangé, pour que le tour continue normalement
  return result
})
```

La réponse de Claude s'affiche en flux à l'écran comme sans le mod. Après la fin de chaque requête, une ligne atténuée de la transcription donne le nombre de tokens lus depuis le cache et le nombre écrits dedans. Un tour avec des appels d'outils comporte plusieurs requêtes, il ajoute donc plusieurs lignes.

`result.usage` contient les comptes de tokens que la Claude API rapporte pour une requête, plus le `model` qui a répondu : `input_tokens`, `output_tokens`, `cache_read_input_tokens` et `cache_creation_input_tokens`. Le hook s'exécute aussi pour les requêtes des sous-agents, donc vérifiez `e.agentId` si vous ne voulez que la conversation principale.

Pour voir les appels d'outils que l'API a exécutés elle-même pendant la requête, comme les appels à l'[outil advisor](https://code.claude.com/docs/en/advisor), lisez `result.serverToolUses`. Claude Code n'exécute pas ces appels, donc aucun hook `tool.call` ou `tool.check` ne se déclenche pour eux. Le champ est absent quand la réponse ne contient aucun appel de ce type, et il requiert Claude Code v2.1.290 ou ultérieur (v2.1.290+).

### Traiter les événements de hooks des réglages

Les hooks des réglages sont les hooks de type commande, HTTP, prompt et agent que vous configurez dans les fichiers de réglages. Chaque [événement de hook des réglages](https://code.claude.com/docs/en/hooks#hook-events), comme `Stop`, `SessionEnd` ou `PostToolUse`, est aussi un événement nommé `classic.` suivi du nom de l'événement de hook des réglages, comme `classic.Stop`. `e` est le JSON qu'un hook des réglages reçoit sur stdin, y compris `transcript_path`.

Ce hook utilise `Stop`, qui se déclenche quand Claude a fini de répondre, pour journaliser l'endroit où la transcription de la session est enregistrée :

```javascript theme={null}
on('classic.Stop', async ($, e, next) => {
  // e a les mêmes champs qu'un hook Stop d'un fichier de réglages lit sur stdin
  $.ui.log('Transcript saved at ' + e.transcript_path)
  // Transmet l'événement, pour que les hooks Stop de vos fichiers de réglages s'exécutent toujours
  return next(e)
})
```

Chaque fois que Claude a fini de répondre, une ligne atténuée de la transcription donne le chemin du fichier de transcription. Le hook renvoie `next(e)`, il observe donc l'événement et ne change rien à la façon dont le tour se termine.

## Cohabiter avec d'autres mods

Plusieurs mods peuvent traiter le même événement, et n'importe lequel d'entre eux peut échouer. Si votre mod bloque des appels d'outils, vérifiez sa position dans la chaîne et ce qui se passe quand son hook échoue.

### L'ordre d'exécution des mods

Les hooks d'un même événement forment une seule chaîne de middlewares. Le `next` de chaque mod appelle le hook du mod suivant, et le dernier `next` atteint le comportement propre de Claude Code. Le premier mod est le plus externe : il voit l'événement avant les autres et le résultat après eux, et il décide si les autres s'exécutent ou non. Un mod plus tardif ne peut pas empêcher un mod antérieur de voir un événement.

Claude Code ordonne la chaîne selon la provenance de chaque mod :

1. Le garde intégré `sec-default@builtin`, un mod intégré à Claude Code que `/plugin` liste sous le nom `cc-plugin-sec-default`, là où [il se charge](https://code.claude.com/docs/en/plugins/mods/admin#know-what-happens-by-default), les mods que votre organisation liste dans [`prependPlugins`](https://code.claude.com/docs/en/plugins/mods/admin#install-your-organizations-mods), puis tout autre mod qui compte comme appartenant à votre organisation et ne figure pas dans `appendPlugins`
2. Les mods que vous installez
3. Les mods que votre organisation liste dans `appendPlugins`
4. Les autres mods intégrés à Claude Code

Parmi les mods que vous installez, un mod s'exécute avant les mods qu'il liste sous `dependencies` dans son manifeste. Au sein d'un même module, les hooks s'exécutent dans l'ordre où `register` a appelé `on`.

#### Place des hooks des réglages dans l'ordre

Les hooks `PreToolUse` configurés dans les fichiers de réglages s'exécutent aussi pendant un appel d'outil, à des points fixes de la chaîne des mods :

* **Hooks `PreToolUse` des réglages gérés** : s'exécutent avant le hook `tool.call` du premier mod, et un blocage de l'un d'eux est définitif, donc aucun mod ne voit l'appel.
* **Hooks `PreToolUse` de tous les autres fichiers de réglages et des `hooks/hooks.json` des plugins** : s'exécutent après que le dernier mod a appelé `next`, comme partie du comportement propre de Claude Code. Un mod qui répond à `tool.call` sans appeler `next` les empêche de s'exécuter, et un mod qui appelle `next` voit leur décision dans le résultat qu'il renvoie.

[`tool.check`](#approuver-ou-refuser-un-appel-doutil-avant-que-lutilisateur-ne-soit-sollicité) se déclenche après que ces hooks et les règles de permission ont décidé, donc un hook sur cet événement peut approuver un appel qu'un hook du second groupe a bloqué.

### Traiter un hook qui échoue

Un hook qui échoue ne casse pas la session, et vous pouvez décider de ce qui se passe à la place. Quand un hook sans gestionnaire `.catch` lève une exception, dépasse le délai, ou renvoie un résultat de mauvaise forme, la suite dépend de s'il avait appelé `next` :

* **Il a échoué avant d'appeler `next`** : Claude Code l'ignore, et le gestionnaire suivant s'exécute à sa place
* **Il a échoué après la résolution de `next`** : ce résultat est conservé, et rien ne s'exécute une seconde fois

Une ligne nomme le mod, l'événement et la raison, comme `my-mod: tool.call hook skipped: threw Error: boom`. L'endroit où vous la lisez dépend de la session, comme le liste [Découvrir pourquoi un mod ne fait rien](https://code.claude.com/docs/en/plugins/mods/troubleshoot#find-out-why-a-mod-does-nothing). Un hook `ui.render` dont le dessin ne passe pas la validation est signalé différemment, comme le décrit [Construire un arbre à partir d'éléments](https://code.claude.com/docs/en/plugins/mods/interface#build-a-tree-from-elements).

Pour qu'un hook qui bloque des appels échoue en position fermée (fail closed), ajoutez un gestionnaire d'erreur `.catch` qui répond à sa place. Ici, `guard` est votre fonction de hook, et le gestionnaire teste [`next.called`](https://code.claude.com/docs/en/plugins/mods/reference#the-hook-function) pour savoir si `guard` avait déjà appelé `next` au moment de l'échec :

```javascript theme={null}
// on renvoie un enregistrement, et .catch attache un gestionnaire à ce seul hook
on('tool.call', { tool: 'Bash' }, guard).catch(async ($, e, next) => {
  // guard avait déjà appelé next, donc renvoyer ce qui est revenu
  if (next.called) return next(e)
  // next.error.kind indique pourquoi le gestionnaire a été sollicité, comme 'throw' ou 'timeout'
  return { deny: 'The command guard failed, so this command was not run: ' + next.error.kind }
})
```

Quand `guard` lève une exception ou dépasse le délai sur un appel Bash, Claude Code appelle le gestionnaire avec le même événement :

* **`guard` a échoué avant d'appeler `next`** : la commande ne s'exécute pas, et Claude lit le texte de `deny` avec le type d'erreur à la fin
* **`guard` a échoué après avoir appelé `next`** : le `next(e)` du gestionnaire se résout avec le résultat qu'avait produit l'appel de `guard`, sans réexécuter la commande, et Claude lit ce résultat

Le gestionnaire a sa propre [limite de temps](https://code.claude.com/docs/en/plugins/mods/reference#limits), plus courte. Si le gestionnaire lui-même lève une exception ou dépasse le délai, Claude Code ignore le hook comme s'il n'avait pas de gestionnaire. Quand `guard` n'avait pas appelé `next`, la commande se poursuit alors comme sans le mod.

La même forme de gestionnaire convient à un garde sur `prompt.submit` ou `config.set`. Quand `next.called` vaut false, renvoyez le refus que la [référence des événements](https://code.claude.com/docs/en/plugins/mods/reference#events) liste pour cet événement : `{ drop: 'the reason' }` pour `prompt.submit`, `{ deny: 'the reason' }` pour `config.set`.

Pour `tool.check` et `plugin.register`, un refus renvoyé après la résolution de `next` reste valable, renvoyez-le donc sans tester `next.called` :

* **`tool.check`** : renvoyez `{ decision: 'deny', reason: 'the reason' }`
* **`plugin.register`** : renvoyez `{ refuse: 'the reason' }`, comme le montre [Refuser les mods quand votre vérification échoue](https://code.claude.com/docs/en/plugins/mods/admin#refuse-mods-when-your-check-fails)

Pour `tool.call`, un `deny` renvoyé après la résolution de `next` [retient le résultat pour Claude](#garder-ou-modifier-un-appel-doutil).

## Étapes suivantes

* [Utiliser l'API des mods](https://code.claude.com/docs/en/plugins/mods/api) : ajouter des commandes et des outils, appeler un modèle, et exécuter du travail sur minuterie
* [Dessiner dans l'interface](https://code.claude.com/docs/en/plugins/mods/interface) : afficher ce que collectent vos hooks dans un panneau ou au-dessus du prompt
* [Tester un mod](https://code.claude.com/docs/en/plugins/mods/test) : déclencher n'importe lequel de ces événements depuis un test
* [Référence des mods](https://code.claude.com/docs/en/plugins/mods/reference) : chaque événement, chaque méthode de l'API des mods, et les limites
