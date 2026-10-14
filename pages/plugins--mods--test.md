# Tester un mod

> Source : https://code.claude.com/docs/en/plugins/mods/test · [← Index](../31-index-documentation-officielle.md)

*Écrire des tests automatisés pour un mod Claude Code qui déclenchent des événements, simulent (stub) les réponses de Claude Code et appuient sur des boutons, sans session, sans connexion et sans réseau.*

Vous pouvez écrire des tests automatisés pour un mod et les lancer depuis votre shell avec [`claude plugin test`](https://code.claude.com/docs/en/plugins/mods/reference#commands). Un test déclenche les événements que vos hooks gèrent et vérifie ce que les hooks ont fait, ce qui permet de détecter un problème avant qu'il n'atteigne une session. Le premier exemple teste le mod de [Créer un mod](https://code.claude.com/docs/en/plugins/mods/create).

## Écrire un test

Un test charge votre mod, envoie des événements à travers ses hooks comme le ferait Claude Code, et vérifie ce que les hooks ont fait, sans session, sans connexion (sign-in) et sans réseau. Vous lancez les tests depuis votre shell avec `claude plugin test`, et chaque fichier de test importe le kit de test, une bibliothèque de test fournie par le module `claude-code/testing`.

Donnez à chaque fichier de test un nom qui se termine par `.test.ts`, comme `first-mod.test.ts`, et enregistrez-le n'importe où dans le répertoire du plugin. Chaque fichier de test doit contenir au moins un `test()`, sinon l'exécution échoue avec `declares no test(): nothing ran`. Un fichier de test peut importer les fichiers de votre mod et des helpers `.ts` voisins, ce qui permet de tester unitairement des fonctions simples, comme les règles d'un jeu, sans le kit.

Ce test déclenche deux appels d'outils, exécute la commande `/tally` de [Créer un mod](https://code.claude.com/docs/en/plugins/mods/create), et vérifie que la réponse compte bien les deux. Sa première ligne est un [stub](#simuler-ce-que-claude-code-répondrait), qui répond aux appels d'outils à la place de Claude Code. Enregistrez-le sous `first-mod/tests/first-mod.test.ts` :

```typescript first-mod/tests/first-mod.test.ts theme={null}
import { expect, test } from 'claude-code/testing'

test('/tally reports the tool calls the mod has seen', async ($, on) => {
  // Répond à chaque appel d'outil à la place de Claude Code, donc aucun outil ne s'exécute
  on('tool.call', () => ({ result: 'ok' }))

  // Déclenche deux appels d'outils, que le hook tool.call du mod compte
  await $.tool.call({ tool: 'Bash', command: 'ls' })
  await $.tool.call({ tool: 'Read', file_path: 'README.md' })

  // Lance /tally et vérifie le texte que renvoie son hook
  const answer = await $.command.run({ command: 'tally', args: '' })
  expect(answer.text).toBe('Claude has made 2 tool calls since this mod loaded')
})
```

Dans votre shell, lancez les tests depuis le répertoire `first-mod` :

```bash theme={null}
claude plugin test
```

La sortie nomme chaque test et indique s'il a réussi, avec des durées qui varient d'une exécution à l'autre :

```text theme={null}
tests/first-mod.test.ts:
(pass) /tally reports the tool calls the mod has seen [22.87ms]

 1 pass
 0 fail
Ran 1 test across 1 file. [0.19s]
```

Chaque `$.tool.call` est passé par le hook [`tool.call`](https://code.claude.com/docs/en/plugins/mods/reference#tools) du mod, qui a ajouté un à son compteur et transmis l'appel au stub. Aucun `ls` ne s'est exécuté et aucun fichier n'a été lu. `$.command.run` est ensuite allé au hook [`command.run`](https://code.claude.com/docs/en/plugins/mods/reference#commands-and-configuration) du mod, et `answer` est l'objet renvoyé par ce hook.

La commande se termine avec le code de sortie 1 quand un test échoue, ce qui la rend utilisable en CI. Si vos propres mods ne peuvent pas se charger dans le shell qui la lance, elle affiche une ligne commençant par `claude plugin test: hooks modules are turned off` avec la raison, et se termine avec le code 1.

### Simuler ce que Claude Code répondrait

Aucun modèle, store ou outil ne s'exécute dans un test : partout où votre mod attend une réponse de Claude Code, le test fournit la réponse au moyen d'un stub. Une fonction de test reçoit deux arguments pour cela :

* **`$`** : le `$` propre au test, qui joue le rôle de Claude Code. Ce n'est pas l'[API des mods](https://code.claude.com/docs/en/plugins/mods/reference#mods-api-methods) que reçoit un hook. Chacune de ses méthodes déclenche l'événement du même nom, l'envoie à travers les hooks de votre mod, et se résout avec le résultat : `$.tool.call({ tool: 'Bash', command: 'ls' })` déclenche `tool.call`. `$.command.run`, `$.prompt.submit`, `$.session.start` et `$.turn.complete` fonctionnent de la même façon, et `$.classic.Stop` ainsi que les autres méthodes `$.classic` déclenchent un [événement de hook de réglages](https://code.claude.com/docs/en/plugins/mods/events#hook-the-settings-hook-events). Un test ne peut pas déclencher directement un appel de l'API des mods comme `ui.close`. Provoquez-le via votre mod, par exemple en appuyant sur le bouton qui ferme le panneau.
* **`on`** : appelez-le pour enregistrer des stubs, c'est-à-dire des hooks qui répondent à la place de Claude Code. Nommez un stub pour un appel de l'API des mods sans le `$.` : un stub enregistré sous `store.get` répond au `$.store.get` de votre mod. Quand votre mod appelle [`$.model.complete`](https://code.claude.com/docs/en/plugins/mods/api#call-a-model) ou [`$.store.get`](https://code.claude.com/docs/en/plugins/mods/interface#keep-state), c'est un stub qui fournit la réponse.

Cet exemple simule un appel de modèle. Le hook appartient à un mod nommé `grader` et gère une commande `/grade` qui envoie une phrase à un modèle et indique si la réponse commence par `PASS`. Le fichier ne contient que le hook testé : le mod a donc aussi besoin d'un `plugin.json` et d'un `hooks.json`, comme dans [Créer un mod](https://code.claude.com/docs/en/plugins/mods/create#write-a-mod-yourself). Pour taper `/grade` dans une session, le mod doit aussi [enregistrer la commande](https://code.claude.com/docs/en/plugins/mods/api#add-a-command) :

```javascript grader/hooks/register.js theme={null}
export function register(on) {
  on('command.run', { command: 'grade' }, async ($, e) => {
    // e.args est le texte tapé après /grade
    const reply = await $.model.complete({
      model: 'haiku',
      system: 'Grade the sentence. Start your reply with PASS or FAIL.',
      prompt: e.args,
    })
    const passed = reply.isAnswered && reply.text.startsWith('PASS')
    return { text: passed ? 'Passed' : 'Try again' }
  })
}
```

Ce test simule l'appel de modèle pour vérifier ce que fait le hook avec une réponse positive :

```typescript grader/tests/grader.test.ts theme={null}
import { expect, test } from 'claude-code/testing'

test('a passing grade is reported', async ($, on) => {
  // Répond à l'appel $.model.complete du mod avec une réponse fixe, donc aucun modèle ne s'exécute
  on('model.complete', () => ({
    value: {
      isAnswered: true,
      text: 'PASS\nNice sentence.',
      usage: { input_tokens: 10, output_tokens: 5, cache_read_input_tokens: 0, cache_creation_input_tokens: 0 },
    },
  }))

  // Lance /grade, ce qui fait appeler le modèle par le mod
  const answer = await $.command.run({ command: 'grade', args: 'The cat sat on the mat.' })
  expect(answer.text).toBe('Passed')
})
```

Le test réussit parce que le `reply` du hook est l'objet placé sous `value`, dont le `text` commence par `PASS`. Pour vérifier l'autre branche, ajoutez un second test dont le stub renvoie un `text` commençant par `FAIL`, et attendez `Try again`.

Un stub pour un appel de l'API des mods renvoie un objet doté d'un champ `value`, qui contient ce à quoi l'appel se résout dans votre mod : `{ value: 7 }` fait se résoudre `$.store.get` à `7`. Un stub pour l'un des événements de Claude Code, comme [`turn.step`](https://code.claude.com/docs/en/plugins/mods/reference#turns) ou `tool.call`, renvoie le résultat propre à cet événement, comme `{ result: 'ok' }`. `$.session.send` et `$.prompt.fill` prennent eux aussi le résultat de leur événement, comme le montre le tableau. [Savoir ce que renvoie un stub](#savoir-ce-que-renvoie-un-stub) indique quelle forme prend chaque nom courant. Les erreurs suivantes signifient qu'un stub est incorrect ou manquant. La sortie d'un test en échec contient un bloc intitulé `the engine reported:`, où chaque erreur apparaît :

* `returned neither { value } nor { deny }` : un stub pour un appel de l'API des mods a renvoyé une valeur nue, ce qui fait échouer le test
* `no implementation for` suivi d'un nom : votre mod a fait cet appel et aucun stub n'y répond

Le kit exporte aussi des mocks prêts à l'emploi pour l'horloge, le store, les variables d'environnement et les lignes ajoutées à la conversation :

* **`mock.clock(on)`** : répond à [`$.clock`](https://code.claude.com/docs/en/plugins/mods/api#run-work-in-the-background) et renvoie une horloge simulée que votre test fait avancer, pour qu'un test de minuteur n'ait pas à attendre.
* **`mock.store(on, { count: 7 })`** : répond à `$.store` à partir d'un store qui démarre avec ces entrées. Il ne renvoie rien : pour vérifier ce que votre mod a enregistré, écrivez vous-même les deux stubs `store`, comme le fait le [test de dessin](#tester-un-dessin).
* **`mock.env(on, { CI: 'true' })`** : répond à `$.env.get` à partir de ces variables.
* **`mock.session(on)`** : renvoie une session simulée dont la méthode `appended()` liste les lignes que votre mod a ajoutées avec [`$.session.append`](https://code.claude.com/docs/en/plugins/mods/reference#session), de la plus ancienne à la plus récente ; nécessite Claude Code v2.1.293 ou ultérieur (v2.1.293+).

### Respecter les règles du kit de test

Le kit de test a quelques règles qui lui sont propres, et en enfreindre une produit les erreurs que les nouveaux auteurs de tests rencontrent en premier :

* **Enregistrez chaque stub avant le premier appel du test sur `$`.** Appeler `on` après cela lève une erreur comme `on("ui.render") after the test first called $`.

* **[`session.start`](https://code.claude.com/docs/en/plugins/mods/reference#session) ne s'exécute pas tout seul.** Chaque test démarre avec votre module fraîchement chargé et aucun de ses hooks appelé : les variables de niveau module conservent leurs valeurs initiales. Si un hook dépend de ce que `session.start` met en place, déclenchez-le d'abord :

  ```typescript theme={null}
  // Répond à l'événement après que votre hook l'a transmis avec next(e)
  on('session.start', () => ({ cwd: '/work' }))
  // Répond à l'appel $.command.register que fait votre hook
  on('command.register', () => ({ value: undefined }))
  // Déclenche l'événement, ce qui exécute votre hook session.start
  await $.session.start({ surface: 'terminal', isInteractive: true, cwd: '/work' })
  ```

  Le second stub répond à l'appel `$.command.register` que fait un hook `session.start` comme celui du [tutoriel](https://code.claude.com/docs/en/plugins/mods/create#write-a-mod-yourself). Sans lui, cet appel est rejeté avec `no implementation for command.register` et le kit saute votre hook, de sorte que rien de ce qui suit l'appel dans le hook ne s'exécute. Le test n'échoue pas à ce moment-là. Le hook sauté n'est listé sous `the engine reported:` que si une vérification ultérieure échoue.

* **Un hook qui renvoie `next(e)` a besoin d'un stub pour répondre.** Quand votre hook [`ui.render`](https://code.claude.com/docs/en/plugins/mods/reference#interface) renvoie `next(e)`, par exemple pour ne rien dessiner quand Claude est inactif, [le monter](#tester-un-dessin) échoue avec `no implementation for ui.render`. Enregistrez un stub qui renvoie un élément sous forme de données simples :

  ```typescript theme={null}
  // Représente ce que Claude Code dessinerait à cet emplacement
  on('ui.render', () => ({ type: 'Text', props: {}, children: ['drawn by Claude Code'] }))
  ```

  Une fois le stub enregistré, le montage réussit, et `ui.find({ type: 'Text' })` renvoie cet élément chaque fois que votre hook a renvoyé `next(e)`.

* **Un stub pour `turn.step` est un générateur asynchrone**, et le test lit le flux jusqu'au bout pour obtenir le résultat :

  ```typescript theme={null}
  on('turn.step', async function* ($, e) {
    // Chaque yield est un morceau de la réponse diffusée (streamée) du modèle
    yield { kind: 'text', index: 0, text: 'ok' }
    // La valeur de retour est le résultat de la requête entière
    return { turnId: e.turnId, index: e.index, answer: 'ok', toolUses: [], stopReason: 'end_turn', usage: null }
  })

  // Déclenche une requête au modèle, ce qui exécute votre hook turn.step
  const stream = $.turn.step({ turnId: 't', index: 0, model: 'claude-test', messageCount: 1 })
  // Lit chaque morceau jusqu'à ce que le flux indique qu'il a terminé
  let step = await stream.next()
  while (step.done !== true) step = await stream.next()
  const result = step.value
  ```

  À la fin de la boucle, `result` est l'objet renvoyé par le stub, après que votre hook `turn.step` a eu l'occasion de le modifier. Ici, `result.answer` vaut `'ok'`.

* **Déclenchez un appel d'outil avec le nom de l'outil et ses arguments sous forme de champs**, comme `await $.tool.call({ tool: 'Bash', command: 'ls' })`, et enregistrez un stub `tool.call` qui renvoie `{ result }`.

### Savoir ce que renvoie un stub

Chaque appel de l'API des mods que fait votre mod dans un test a besoin d'un stub qui répond à la place de Claude Code, sauf les quelques appels auxquels le kit répond lui-même : [`$.ui.invalidate`](https://code.claude.com/docs/en/plugins/mods/interface#redraw-when-something-changes), [`$.state`](https://code.claude.com/docs/en/plugins/mods/interface#keep-state) et les appels `$.session.append`. Pour les appels `$.clock`, utilisez `mock.clock(on)`, sinon le `$.clock.now()` de votre mod échoue avec `no implementation for clock.now`.

Ce tableau liste ceux que les mods utilisent le plus. La première colonne est l'appel que fait votre mod ou l'événement qu'il transmet avec `next(e)`. La seconde est la fonction à passer à `on` sous ce nom : la ligne `$.store.get` devient ainsi `on('store.get', ($, e) => ({ value: saved.get(e.key) }))`. Un `'...'` dans un stub marque du texte à compléter :

| Votre mod appelle ou transmet | Stub |
| :- | :- |
| `$.command.register`, `$.tool.register`, `$.ui.toast`, `$.ui.log`, `$.ui.status`, `$.ui.close`, `$.store.set` | `() => ({ value: undefined })`. Pour `ui.toast` et `ui.log`, le texte est `e.text`. |
| `$.store.get` | `($, e) => ({ value: saved.get(e.key) })` |
| `$.fs.read` | `($, e) => ({ value: e.path.endsWith('notes.md') ? '# Notes' : '' })`. `e.path` arrive sous forme de chemin absolu, donc comparez avec `endsWith`. |
| `$.ui.open` | `() => ({ value: { isPlaced: true } })` |
| `$.ui.ask` | Un stub `tool.call`, car la question lui parvient sous forme d'appel à l'outil `AskUserQuestion` : `($, e) => ({ result: { answers: { [e.questions[0].question]: 'Run it' } } })`. Vérifiez d'abord `e.tool` si votre mod transmet d'autres appels d'outils. |
| `$.model.complete` | `() => ({ value: { isAnswered: true, text: '...', usage } })` |
| `$.process.run` | `($, e) => ({ value: { exitCode: 0, stdout: '...', stderr: '' } })`. `e.argv` est la liste d'arguments et `e.init` contient `cwd` et `timeoutMs`. |
| Tout appel de l'API des mods qui doit échouer | `() => ({ deny: 'the reason' })`, ce qui fait rejeter l'appel dans votre mod. Un stub qui lève une exception est sauté à la place. |
| `session.start` | `() => ({ cwd: '/work' })` |
| `turn.start` | `($, e) => ({ turnId: e.turnId })` |
| `tool.call` | `() => ({ result: '...' })` |
| `turn.complete` | `() => ({ text: '' })`. Déclenchez-le avec `$.turn.complete({ turnId, answer, durationMs, isAborted: false, usage: null })`. |
| `prompt.submit` | `($, e) => ({ text: e.text })` |
| `prompt.fill` | `() => ({ isFilled: true })` |
| `$.prompt.read` | `() => ({ value: { text: '...', cursor: 0 } })` |
| `$.ui.copy` | `() => ({ value: { isCopied: true } })` |
| `$.session.messages` | `() => ({ value: [{ role: 'assistant', text: '...', toolUses: [] }] })` |
| `$.session.id`, `$.agent.list` | `() => ({ value: 'abc123' })`, `() => ({ value: [] })` |
| `session.send` | `() => ({ isDelivered: true })`. `e.to` arrive sous forme de chaîne même quand votre mod a passé `{ sessionId }`. |
| `session.receive` | `($, e) => ({ text: e.text })`. Déclenchez-le avec `$.session.receive({ origin: { kind: 'peer-send-message' }, text })`. |
| `ui.render` | `() => ({ type: 'Text', props: {}, children: ['...'] })` |

`expect` dispose des assertions `toBe`, `toEqual`, `toMatch`, `toMatchObject`, `toContain`, `toBeDefined`, `toBeUndefined` et `toThrow`, et de `.not` devant n'importe laquelle d'entre elles.

Quand un `expect` échoue à l'intérieur d'un stub ou d'un hook que vous avez passé à `on` sous forme de fonction simple (et non de générateur asynchrone), le test échoue. Le moteur saute ce hook, et la sortie de l'échec le nomme, par exemple `in the test's store.set hook`.

## Tester un minuteur

Un mod qui exécute du travail sur un minuteur a besoin d'une horloge contrôlée par le test, pour que le test puisse faire avancer le temps au lieu d'attendre. `const clock = mock.clock(on)` renvoie une horloge simulée qui démarre à `0` et n'avance que lorsque votre test la fait avancer. Pour démarrer à un autre instant, passez-le en millisecondes, comme dans `mock.clock(on, { now: 5000 })`. L'horloge dispose de ces méthodes :

| Méthode | Ce qu'elle fait |
| :- | :- |
| `await clock.advance(1000)` | Avance le temps de ce nombre de millisecondes et exécute chaque minuteur qui arrive à échéance |
| `await clock.set(5000)` | Avance le temps jusqu'à cette valeur, comme le ferait `advance` |
| `clock.now()` | Renvoie l'heure, c'est-à-dire la valeur à laquelle se résout le `$.clock.now()` de votre mod |
| `await clock.settle()` | Exécute les minuteurs déjà échus, comme une chaîne d'appels `$.clock.after` à délai nul, sans faire avancer le temps |
| `await clock.sleep(2000)` | À l'intérieur d'un stub, fait répondre ce stub seulement une fois que le test a avancé jusque-là ; c'est ainsi qu'on simule un modèle ou un processus lent |

Ce hook appartient à un mod nommé `countdown` et gère une commande `/countdown` qui prend un nombre de secondes, démarre un minuteur `$.clock.every` d'une seconde, et affiche un toast à zéro. Comme pour `grader`, le fichier ne contient que le hook testé et n'enregistre pas la commande :

```javascript countdown/hooks/register.js theme={null}
export function register(on) {
  on('command.run', { command: 'countdown' }, async ($, e) => {
    // e.args est le texte tapé après /countdown
    let left = Number(e.args)
    const timer = $.clock.every(1000, () => {
      left -= 1
      if (left === 0) {
        timer.cancel()
        $.ui.toast('Time is up')
      }
    })
    // N'affiche rien dans la transcription
    return {}
  })
}
```

Ce test lance `/countdown 3` et fait avancer l'horloge simulée : il vérifie ainsi trois secondes de comportement sans attendre trois secondes :

```typescript countdown/tests/countdown.test.ts theme={null}
import { expect, mock, test } from 'claude-code/testing'

test('the countdown ends with a toast', async ($, on) => {
  // Répond à chaque appel $.clock à partir d'une horloge que le test contrôle
  const clock = mock.clock(on)
  // Collecte le texte de chaque toast que le mod affiche
  const toasts: string[] = []
  on('ui.toast', ($, e) => {
    toasts.push(e.text)
    return { value: undefined }
  })

  await $.command.run({ command: 'countdown', args: '3' })
  // Après deux secondes, le minuteur s'est déclenché deux fois et aucun toast n'est dû
  await clock.advance(2000)
  expect(toasts).toEqual([])
  // La troisième seconde amène le compte à zéro
  await clock.advance(1000)
  expect(toasts).toEqual(['Time is up'])
})
```

Le premier `expect` montre que le toast n'arrive pas trop tôt, et le second qu'il arrive une fois. Chaque `advance` se résout après l'exécution des minuteurs arrivés à échéance : la vérification de la ligne suivante voit donc leur effet.

## Tester un dessin

Un test peut dessiner l'un des [emplacements de rendu (render sites)](https://code.claude.com/docs/en/plugins/mods/reference#render-sites) de votre mod, puis appuyer sur les éléments qu'il a dessinés, taper dedans et les retrouver. `$.ui.mount` dessine l'emplacement via le hook `ui.render` de votre mod et renvoie un handle doté d'une méthode pour chacune de ces actions. Pour couvrir plusieurs applications dans un même test, réglez `surface` sur l'application pour laquelle dessiner. Ce test ouvre le panneau de [Construire un panneau avec des onglets](https://code.claude.com/docs/en/plugins/mods/interface#build-a-pane-with-tabs), change d'onglet, appuie sur le bouton, et vérifie le compteur dans le terminal et dans l'application Desktop :

```typescript hello-tabs/tests/hello-tabs.test.ts theme={null}
import { expect, test } from 'claude-code/testing'

// Ce que Claude Code passe à un hook ui.render pour ce panneau, hormis l'application
const PANE = {
  plugin: 'hello-tabs',
  component: 'Pane',
  requestId: 'hello-tabs',
  viewport: { columns: 100, rows: 30 },
  props: {
    title: 'Hello tabs',
    isFocused: true,
    bodyColumns: 60,
    placement: 'inline',
    scroll: { offset: 0, bodyRows: 10 },
    view: {},
  },
} as const

test('the second tab counts presses and saves the count', async ($, on) => {
  // Simule $.store avec une Map, pour que le test puisse lire ce que le mod a enregistré
  const saved = new Map<string, unknown>()
  on('store.get', ($, e) => ({ value: saved.get(e.key) }))
  on('store.set', ($, e) => {
    saved.set(e.key, e.value)
    return { value: undefined }
  })

  // Dessine le panneau une fois pour chaque application
  for (const surface of ['terminal', 'desktop'] as const) {
    const ui = await $.ui.mount({ ...PANE, surface })
    // Appuie sur les boutons via la clé (key) que le mod leur a donnée
    await ui.press({ key: 'tab-two' })
    await ui.press({ key: 'more' })
    // La ligne de compteur du second onglet figure dans le dessin
    expect(await ui.find({ type: 'Text', text: /^Count: \d+$/ })).toBeDefined()
    await ui.unmount()
  }

  // Un appui dans chaque application, cela fait deux
  expect(saved.get('count')).toBe(2)
})
```

Dans votre shell, lancez `claude plugin test` depuis le répertoire `hello-tabs`. Le test réussit quand les deux applications dessinent la ligne de compteur et que le mod a enregistré `2`. Le compteur est conservé de la première application à la seconde, car les deux montages utilisent le même module chargé.

Le handle que renvoie `$.ui.mount` dispose de ces méthodes, qui désignent les éléments par la `key` que vous leur avez donnée :

| Méthode | Ce qu'elle fait |
| :- | :- |
| `press({ key: 'more' })` | Appuie sur le `Button` portant cette clé |
| `input({ key: 'new-note', text: 'buy milk' })` | Tape le texte dans l'`Input` portant cette clé et appuie sur Entrée. Ajoutez `kind: 'change'` pour taper sans soumettre. |
| `select({ key: 'size', value: 'large' })` | Choisit l'option ayant cette valeur dans le `Select` portant cette clé |
| `find({ key: 'more' })` ou `find({ type: 'Text', text: 'Count: 2' })` | Renvoie le premier élément correspondant sous la forme `{ type, props, children }`, ou `undefined`. `text` peut être une chaîne ou une expression régulière. |
| `unmount()` | Supprime le dessin |

Chaque méthode se résout une fois que votre handler a terminé, ce qui permet de vérifier le résultat à la ligne suivante. Réglez `props` sur ce que Claude Code passerait pour cet emplacement. Le [tableau des emplacements de rendu](https://code.claude.com/docs/en/plugins/mods/reference#render-sites) liste les props de chaque emplacement, et [les types de votre build](https://code.claude.com/docs/en/plugins/mods/create#get-the-types-for-your-build) donnent leurs types.

Un test de dessin vérifie l'arbre que renvoie votre hook et s'il est valide pour cette application. Il ne vérifie pas comment l'application le peint : examinez donc aussi toute nouvelle mise en page dans une vraie session.

### Tester un dessin après `/clear`

Chaque test démarre avec chaque valeur de `$.state` à sa valeur par défaut, c'est-à-dire l'état dans lequel `/clear` les laisse. Pour tester ce que fait ensuite votre mod, sautez `session.start`, déclenchez `classic.SessionStart` avec `source: 'clear'`, et vérifiez ce que dessine votre mod.

Ce test vérifie le module de [Recharger une valeur enregistrée après `/clear`](https://code.claude.com/docs/en/plugins/mods/interface#load-a-saved-value-again-after-clear). Ajoutez-le au fichier de [Tester un dessin](#tester-un-dessin), où `PANE` est défini. Le premier test de ce fichier attend que le bouton enregistre le compteur, comme le fait le bouton de [Enregistrer depuis plus d'une session](https://code.claude.com/docs/en/plugins/mods/interface#save-from-more-than-one-session) :

```typescript hello-tabs/tests/hello-tabs.test.ts theme={null}
test('the saved count comes back after /clear', async ($, on) => {
  // Le store contient déjà un compteur à 7
  on('store.get', () => ({ value: 7 }))
  // Répond à l'événement après que votre hook l'a transmis avec next(e)
  on('classic.SessionStart', () => ({}))

  // Déclenche l'événement qui suit /clear, ce qui exécute votre hook
  await $.classic.SessionStart({ source: 'clear' })

  const ui = await $.ui.mount({ ...PANE, surface: 'terminal' })
  await ui.press({ key: 'tab-two' })
  // Le panneau affiche le compteur enregistré, et non la valeur par défaut 0
  expect(await ui.find({ type: 'Text', text: 'Count: 7' })).toBeDefined()
})
```

Le test réussit quand votre hook `classic.SessionStart` a copié le `7` enregistré dans `$.state` avant que le panneau ne se dessine. Sans ce hook dans votre module, le panneau dessine `Count: 0`, `find` renvoie `undefined`, et le test échoue sur `toBeDefined`.

## Tester un mod de politique

<a id="test-a-mod-that-judges-other-mods"></a>

Un mod que votre organisation liste dans [`prependPlugins`](https://code.claude.com/docs/en/plugins/mods/admin) peut refuser un autre mod avant son chargement. Pour en tester un, réglez le niveau (tier) de votre mod et donnez au test un second mod que le vôtre autorisera ou refusera :

* **`tier`** : appelez-le une fois en haut du fichier de test, comme dans `tier('prepend')`, pour charger votre mod en tant que `prepend`, `append` ou `builtin`, sa place dans l'[ordre d'exécution des mods](https://code.claude.com/docs/en/plugins/mods/events#the-order-mods-run-in). Sans cela, votre mod se charge en tant que `user`.
* **`plugins`** : passez à `test` un objet d'options avant le corps du test. Son tableau `plugins` contient des mods que vous écrivez en ligne, chacun avec un `name` et une fonction `register`. Pour en charger un ailleurs qu'en `user`, ajoutez-lui `tier`.

Ce fichier de test charge en premier le [mod de politique de la page d'administration](https://code.claude.com/docs/en/plugins/mods/admin#enforce-a-policy-with-a-mod-of-your-own). Il vérifie que le mod de politique refuse un mod qui démarre un processus et autorise un mod qui n'en démarre pas :

```typescript acme-guard/tests/guard.test.ts theme={null}
import { expect, test, tier } from 'claude-code/testing'

// Charge le mod testé avant tous les autres mods
tier('prepend')

// Un second mod dont le code appelle $.process.run, ce que la politique bloque
const runner = {
  name: 'runner',
  register(on) {
    on('tool.call', async ($, e, next) => {
      await $.process.run(['ls'])
      return { result: 'runner answered' }
    })
  },
}

// Un second mod qui n'appelle rien de ce que la politique bloque
const reader = {
  name: 'reader',
  register(on) {
    on('tool.call', async ($, e, next) => {
      return { result: 'reader answered' }
    })
  },
}

test('refuses a mod that starts a process', { plugins: [runner] }, async ($, on) => {
  on('tool.call', () => ({ result: 'claude code answered' }))
  let message = ''
  try {
    // Le premier appel sur $ charge les mods, donc le refus est levé ici
    await $.tool.call({ tool: 'Bash', command: 'ls' })
  } catch (error) {
    message = error.message
  }
  expect(message).toBe('runner: refused by acme-guard: Acme policy: mods may not call process.run')
})

test('admits a mod that starts no process', { plugins: [reader] }, async ($, on) => {
  on('tool.call', () => ({ result: 'claude code answered' }))
  const out = await $.tool.call({ tool: 'Bash', command: 'ls' })
  // La réponse vient de reader, ce qui montre qu'il s'est chargé
  expect(out).toEqual({ result: 'reader answered' })
})
```

Dans votre shell, lancez `claude plugin test` depuis le répertoire `acme-guard`. Les deux tests réussissent avec le mod de politique tel que le montre la page d'administration.

Le kit charge tous les mods au premier appel du test sur `$`. Quand votre mod en refuse un, cet appel lève une exception, et le message nomme le mod refusé, le mod qui l'a refusé et votre raison. Dans le second test, rien n'est refusé : `reader` répond donc à l'appel d'outil avant qu'il n'atteigne le stub.

## Étapes suivantes

* [Dépanner un mod](https://code.claude.com/docs/en/plugins/mods/troubleshoot) : découvrir pourquoi un mod ne fait rien dans une session
* [Référence des mods](https://code.claude.com/docs/en/plugins/mods/reference) : l'entrée et le résultat de chaque événement, pour écrire des stubs
