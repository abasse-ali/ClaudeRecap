# Créer un mod

> Source : https://code.claude.com/docs/en/plugins/mods/create · [← Index](../31-index-documentation-officielle.md)

*Demander à Claude d'écrire un mod Claude Code à partir d'une description, ou en écrire un vous-même qui compte les appels d'outils et ajoute une commande. Apprendre la boucle rechargement-validation.*

Un mod est un [plugin](https://code.claude.com/docs/en/plugins/overview) Claude Code doté d'un fichier d'entrée, appelé le module de hooks : un fichier JavaScript ou TypeScript dont Claude Code appelle les fonctions quand des événements se produisent. Pour en créer un :

* **Demander à Claude de l'écrire** : [décrivez ce que vous voulez](#demander-un-mod-à-claude) dans une session Claude Code
* **L'écrire vous-même** : [suivez le tutoriel](#écrire-un-mod-vous-même) pour apprendre comment fonctionne le code d'un mod. Vous n'avez besoin ni de Node.js, ni d'un bundler, ni d'une étape de build, car Claude Code charge directement les fichiers `.js` et `.ts`.

Si vous n'avez pas encore décidé si un mod est le bon outil, lisez d'abord la [comparaison de la vue d'ensemble](https://code.claude.com/docs/en/plugins/mods/overview#compare-mods-settings-hooks-skills-and-mcp-servers).

> **Note** : utilisez Claude Code v2.1.287 ou ultérieur (v2.1.287+). Dans votre shell, lancez `claude --version` pour vérifier. Pour savoir si les mods peuvent se charger chez vous, voir [Vérifier si les mods peuvent se charger](https://code.claude.com/docs/en/plugins/mods/troubleshoot#check-whether-mods-can-load).

## Demander un mod à Claude

Décrivez le mod que vous voulez dans une session Claude Code interactive, et Claude l'écrit. Claude s'appuie sur un [skill](https://code.claude.com/docs/en/skills) intégré nommé `plugin-authoring`, qui lui indique où écrire le mod, quels événements et méthodes votre version propose, et comment le mod est chargé. Claude peut charger le skill quand vous demandez un mod, ou vous pouvez le charger vous-même en lançant `/plugin-authoring` dans le prompt de Claude Code.

Le mod s'exécute une fois que vous l'avez approuvé, sauf dans les [sessions où un mod écrit par Claude ne peut pas se charger](#sessions-où-un-mod-écrit-par-claude-ne-peut-pas-se-charger).

#### Étape 1 — Décrire le mod

Demandez le mod avec vos propres mots, par exemple `make a mod that shows the current git branch above the prompt`. Claude écrit le mod dans un répertoire qui lui est propre, dans le dossier des mods de la session, qui est `~/.claude/dev-mods/` suivi de l'ID de la session. Le chemin complet d'un mod ressemble à `~/.claude/dev-mods/3f2a9c1e-5b7d-4e8a-9c21-6d0f4b8a7e13/git-branch/`.

> **Note** : dans les [modes de permission](https://code.claude.com/docs/en/permission-modes#protected-paths) `default` et `acceptEdits`, Claude Code demande confirmation avant que Claude ne crée chacun des fichiers du mod, car `~/.claude` est un chemin protégé. Approuvez chaque fichier à mesure qu'il se présente.

#### Étape 2 — Approuver le mod

Quand Claude enregistre le premier fichier, Claude Code demande s'il faut activer le rechargement à chaud pour la session. Le rechargement à chaud exécute les mods que Claude écrit dans cette session et prend en compte chaque modification ultérieure.

Choisissez l'une de ces réponses :

* **Enable for this session** (activer pour cette session) : les mods du dossier des mods de la session se chargent à la fin du tour, et se rechargent à la fin de chaque tour qui les modifie. Votre réponse vaut pour la session, y compris après sa reprise.
* **Not now** (pas maintenant) : rien ne se charge pour l'instant. Les fichiers restent là où Claude les a écrits, et les mods se chargent au prochain démarrage de cette session. Pour empêcher un mod de se charger à jamais, supprimez son répertoire.

#### Étape 3 — Vérifier que le mod s'est chargé

Lancez `/plugin` dans le prompt de Claude Code et appuyez sur Tab jusqu'à ce que l'onglet **Installed** soit sélectionné. Il liste le mod, et vous pouvez le désactiver à cet endroit.

#### Étape 4 — Essayer le mod

Utilisez ce que vous avez demandé. Pour le prompt d'exemple, le nom de la branche courante apparaît au-dessus de la zone de prompt. Si le mod ne fait pas ce que vous vouliez, dites à Claude quoi changer. Le mod se recharge à la fin de chaque tour qui modifie ses fichiers, donc vous pouvez essayer la modification dès que Claude a terminé.

### Utiliser le mod dans d'autres sessions

Un mod écrit par Claude ne se charge que dans la session qui l'a créé, et Claude Code supprime le dossier des mods de cette session une fois qu'il est plus ancien que [`cleanupPeriodDays`](https://code.claude.com/docs/en/settings-reference#cleanupperioddays). Pour conserver le mod, copiez son répertoire hors du dossier des mods vers un emplacement à vous, comme `~/mods/git-branch`. Puis choisissez comment le charger :

* **Dans une session que vous démarrez** : dans votre shell, lancez `claude --plugin-dir ~/mods/git-branch`
* **Pour d'autres personnes** : [ajoutez-le à une marketplace](#partager-votre-mod) pour qu'elles puissent l'installer

### Sessions où un mod écrit par Claude ne peut pas se charger

Un mod écrit par Claude ne se charge qu'après votre approbation, dans un espace de travail de confiance où les mods sont autorisés à s'exécuter. Dans ces sessions, il ne se charge pas :

* **Personne n'est là pour approuver** : la session ne peut pas vous afficher de demande, comme dans une exécution `claude -p` ou en [mode `dontAsk`](https://code.claude.com/docs/en/permission-modes)
* **L'espace de travail n'est pas de confiance** : vous n'avez pas accepté la demande de confiance pour le répertoire
* **Les mods sont désactivés** : vous avez démarré avec `--safe-mode` ou `--bare`, vous avez réglé `disableAllHooks`, ou les [réglages gérés de votre organisation le bloquent](https://code.claude.com/docs/en/plugins/mods/admin#choose-how-much-to-allow)

## Écrire un mod vous-même

Dans ce tutoriel, vous construisez un mod nommé `first-mod` qui compte les appels d'outils que fait Claude, affiche le compte à côté du spinner pendant que Claude travaille, et ajoute une commande `/tally` qui l'imprime. Vous lisez ensuite les déclarations de types que Claude Code écrit à côté de votre mod et lancez `claude plugin validate`. Ensemble, ils vous montrent les événements et méthodes que propose votre version et ce que Claude Code lit dans votre code.

Un enregistrement vidéo (non reproduit ici) montre le mod terminé : le spinner compte les appels d'outils, `/tally` imprime le compte, et une modification du code prend effet pendant que la session tourne. On y voit le prompt `list the files here and read the README` envoyé ; le spinner affiche `Thinking · tool calls: 1` et le compte augmente pendant que Claude travaille ; la commande `/tally` imprime `first-mod: Claude has made 3 tool calls since this mod loaded` ; une ligne indique que first-mod s'est rechargé et liste ses quatre hooks ; au prompt suivant, le spinner affiche `Thinking · tools used: 1`.

Vous écrivez trois fichiers :

```text theme={null}
first-mod/
├── .claude-plugin/
│   └── plugin.json
└── hooks/
    ├── hooks.json
    └── register.js
```

* **`plugin.json`** : le [manifeste](https://code.claude.com/docs/en/plugins/manifest-reference) du plugin
* **`hooks.json`** : [pointe vers votre fichier de code](https://code.claude.com/docs/en/plugins/mods/reference#files)
* **`register.js`** : votre code, appelé le module de hooks

#### Étape 1 — Créer le répertoire du plugin

Créez les deux répertoires qui contiennent les fichiers :

##### Bash ou Zsh

```bash theme={null}
mkdir -p first-mod/.claude-plugin first-mod/hooks
```

##### PowerShell

```powershell theme={null}
New-Item -ItemType Directory -Force first-mod\.claude-plugin, first-mod\hooks
```

#### Étape 2 — Écrire le manifeste

Un mod est un plugin, et un mod a besoin d'un [manifeste](https://code.claude.com/docs/en/plugins/manifest-reference). Le manifeste de ce mod n'a aucun champ spécial. Enregistrez ceci sous `first-mod/.claude-plugin/plugin.json` :

```json first-mod/.claude-plugin/plugin.json theme={null}
{
  "name": "first-mod",
  "version": "0.1.0",
  "description": "Counts Claude's tool calls, shows the count beside the spinner, and adds a /tally command",
  "author": { "name": "Your Name" }
}
```

#### Étape 3 — Indiquer à Claude Code où se trouve votre code

Quand Claude Code charge un plugin, il lit le fichier `hooks/hooks.json` du plugin. La clé `modules` de ce fichier donne le chemin de votre code, et c'est sa présence qui fait du plugin un mod. Listez un chemin, relatif à `hooks.json`. Ici, il pointe vers `register.js`, que vous écrivez à l'étape suivante.

Enregistrez ceci sous `first-mod/hooks/hooks.json` :

```json first-mod/hooks/hooks.json theme={null}
{
  "description": "The first-mod hooks module",
  "modules": ["./register.js"]
}
```

#### Étape 4 — Écrire le code

Ce fichier est le code du mod, appelé le module de hooks. Quand le mod se charge, Claude Code appelle la fonction `register` que le fichier exporte et lui passe une fonction nommée [`on`](https://code.claude.com/docs/en/plugins/mods/reference#the-hook-function). Chaque appel à `on` enregistre un gestionnaire d'événement, appelé hook, pour l'événement qu'il nomme.

Enregistrez ceci sous `first-mod/hooks/register.js` :

```javascript first-mod/hooks/register.js theme={null}
// Le compte, partagé par les hooks ci-dessous
let calls = 0

// Claude Code appelle ceci une fois, quand le mod se charge
export function register(on) {
  // S'exécute quand la session démarre, avant votre premier prompt
  on('session.start', async ($, e, next) => {
    // Ajoute la commande /tally
    await $.command.register({
      name: 'tally',
      description: 'Show how many tool calls Claude has made',
    })
    // Laisse la session démarrer normalement
    return next(e)
  })

  // S'exécute chaque fois que Claude s'apprête à utiliser un outil
  on('tool.call', async ($, e, next) => {
    calls += 1
    // Demande à Claude Code de redessiner l'interface, pour que le nouveau compte s'affiche
    $.ui.invalidate('ui.render')
    // Laisse l'outil s'exécuter normalement
    return next(e)
  })

  // S'exécute quand vous tapez /tally, et seulement alors, grâce au matcher
  on('command.run', { command: 'tally' }, async () => {
    // Le texte à imprimer dans la transcription
    return { text: 'Claude has made ' + calls + ' tool calls since this mod loaded' }
  })

  // S'exécute chaque fois que Claude Code dessine le spinner
  on('ui.render', { component: 'Spinner' }, async ($, e, next) => {
    // Garde le spinner de Claude Code, avec le compte ajouté après son mot
    return next({ ...e, props: { ...e.props, suffix: ' · tool calls: ' + calls + '…' } })
  })
}
```

Le fichier garde un compte dans `calls` et enregistre quatre hooks :

* **[`session.start`](https://code.claude.com/docs/en/plugins/mods/reference#session)** s'exécute quand la session démarre, avant votre premier prompt, et de nouveau à chaque rechargement du mod. Il ajoute la commande `/tally` à Claude Code.
* **[`tool.call`](https://code.claude.com/docs/en/plugins/mods/reference#tools)** s'exécute chaque fois que Claude s'apprête à utiliser un outil. Il ajoute un à `calls` et demande à Claude Code de redessiner l'interface.
* **[`command.run`](https://code.claude.com/docs/en/plugins/mods/reference#commands-and-configuration)** s'exécute quand vous tapez `/tally`. Il renvoie le texte à imprimer.
* **[`ui.render`](https://code.claude.com/docs/en/plugins/mods/reference#interface)** s'exécute chaque fois que Claude Code dessine le spinner. Il ajoute le compte après le mot du spinner.

[Comment fonctionne le mod d'exemple](#comment-fonctionne-le-mod-dexemple) explique les trois arguments que prend chaque hook et ce que chacun renvoie.

#### Étape 5 — Charger le mod

Démarrez Claude Code avec le flag `--plugin-dir`, qui charge un répertoire de plugin pour une session sans l'installer :

```bash theme={null}
claude --plugin-dir ./first-mod
```

#### Étape 6 — Essayer le mod

Demandez à Claude de faire quelque chose qui prend quelques appels d'outils, comme `list the files here and read the README`. Pendant que Claude travaille, le mot du spinner est suivi d'un compte qui augmente, comme dans `Thinking · tool calls: 2…`. Quand Claude a terminé, tapez `/tally` et appuyez sur Entrée. La transcription affiche `first-mod: Claude has made 2 tool calls since this mod loaded`, avec votre propre compte. Claude Code place le nom du plugin devant le texte de la commande.

Pour vérifier la commande sans session interactive, lancez-la en mode non interactif :

```bash theme={null}
claude -p "/tally" --plugin-dir ./first-mod
```

```text theme={null}
first-mod: Claude has made 0 tool calls since this mod loaded
```

Si `/tally` n'est pas dans la liste des commandes, le module ne s'est pas chargé. Voir [Découvrir pourquoi un mod ne fait rien](https://code.claude.com/docs/en/plugins/mods/troubleshoot#find-out-why-a-mod-does-nothing).

#### Étape 7 — Modifier le code pendant que la session tourne

Laissez la session ouverte. Dans `register.js`, remplacez `' · tool calls: '` par `' · tools used: '` dans le hook `ui.render` et enregistrez. La ligne mise en évidence (la 4e) est celle qui change :

```javascript first-mod/hooks/register.js {4} theme={null}
  // S'exécute chaque fois que Claude Code dessine le spinner
  on('ui.render', { component: 'Spinner' }, async ($, e, next) => {
    // Garde le spinner de Claude Code, avec le compte ajouté après son mot
    return next({ ...e, props: { ...e.props, suffix: ' · tools used: ' + calls + '…' } })
  })
```

Une ligne de la transcription indique que `first-mod` s'est rechargé et liste ses hooks, et le spinner suivant utilise le nouveau texte, comme dans `Thinking · tools used: 1…`.

### Comment fonctionne le mod d'exemple

Chaque fonction que vous passez à `on` est un hook, c'est-à-dire un gestionnaire d'événement. Claude Code passe à chaque hook les trois mêmes arguments :

* **L'API des mods**, nommée `$` : toutes les méthodes qu'un mod peut appeler pour agir hors de lui-même, dans des [espaces de noms](https://code.claude.com/docs/en/plugins/mods/reference#mods-api-methods) comme `$.ui` et `$.command`
* **L'événement**, nommé `e` : l'[entrée de l'événement](https://code.claude.com/docs/en/plugins/mods/reference#events) sous forme de données brutes, comme le nom et les arguments d'un appel d'outil
* **Le gestionnaire suivant**, nommé [`next`](https://code.claude.com/docs/en/plugins/mods/events#how-a-hook-handles-an-event) : une fonction qui transmet l'événement aux autres mods puis au comportement propre de Claude Code, et renvoie le résultat

Les hooks de `first-mod` traitent leurs événements de ces manières :

* **Observer** : le hook `session.start` enregistre la commande, et le hook `tool.call` compte l'appel et demande un redessin. Les deux renvoient `next(e)`, donc la session démarre et l'outil s'exécute normalement.
* **Répondre** : le hook `command.run` renvoie son propre résultat et n'appelle jamais `next`. Le deuxième argument de `on`, `{ command: 'tally' }`, est un filtre, appelé [matcher](https://code.claude.com/docs/en/plugins/mods/events#filter-which-events-a-hook-handles), donc le hook ne s'exécute que pour `/tally`.
* **Réécrire** : le hook `ui.render` appelle `next` avec une copie de `e` dont le `suffix` contient le compte, donc Claude Code dessine son spinner habituel avec votre texte après le mot

Claude Code surveille un répertoire chargé avec `--plugin-dir` et recharge à chaud le module de hooks quand un fichier qu'il contient change. Chaque rechargement exécute à nouveau `register`, donc `calls` repasse à `0` et `/tally` recommence à compter. Pour conserver une valeur entre les rechargements, voir [Conserver l'état](https://code.claude.com/docs/en/plugins/mods/interface#keep-state).

## Continuer à travailler sur un mod

Une fois un mod chargé, vous pouvez demander à Claude de le modifier, vérifier votre code par rapport aux définitions de types de votre version, lister les événements et appels que Claude Code y trouve, et le tester.

### Modifier un mod avec Claude

Pour modifier un mod que vous avez déjà, démarrez la session avec `--plugin-dir` pointant vers le répertoire du mod, afin que ce que Claude écrit se charge dans la même session :

```bash theme={null}
claude --plugin-dir ./first-mod
```

Demandez ensuite la modification, par exemple `add a /tally-reset command to this mod that sets the tally back to zero`. Claude modifie le module de hooks, lance `claude plugin validate`, et corrige ce qu'il signale. Un répertoire chargé avec `--plugin-dir` est un [chemin protégé](https://code.claude.com/docs/en/permission-modes#protected-paths), donc dans les modes `default` et `acceptEdits` on vous demande d'approuver chacune des modifications de Claude sur le mod. Le tableau des chemins protégés donne le résultat pour les autres modes de permission.

Les fichiers que Claude enregistre pendant son tour se rechargent à la fin du tour, donc vous pouvez essayer `/tally-reset` dès que Claude a terminé.

### Obtenir les définitions de types de votre version

Quand Claude Code charge un mod depuis `--plugin-dir` dans une session interactive, ou un mod [que Claude a écrit pour vous](#demander-un-mod-à-claude), il écrit des fichiers de déclaration TypeScript dans le répertoire `.claude-plugin/types/` du mod. Ils décrivent exactement les événements, méthodes de l'API des mods et éléments de la version de Claude Code que vous exécutez, pour que votre éditeur puisse autocompléter et vérifier les types de vos hooks. Le répertoire contient ces fichiers :

| Chemin | Ce qu'il déclare |
| :- | :- |
| `claude-code/index.d.ts` | Chaque événement avec son entrée et son résultat, chaque espace de noms et méthode de l'API des mods, et les éléments que chaque surface peut dessiner |
| `claude-code-tools/index.d.ts` | Les entrées et résultats des outils intégrés, pour que vérifier `e.tool === 'Bash'` affine le type de `e` |
| `claude-code-mcp/index.d.ts` | Les entrées des outils MCP qui étaient connectés la dernière fois que vous avez enregistré un fichier du mod |
| `index.d.ts` dans un répertoire portant le nom d'un plugin | Ce que ce plugin ajoute à l'API des mods. Il y a un répertoire pour chaque plugin que votre `plugin.json` liste sous `dependencies`. |
| `tsconfig.json` | Options du compilateur adaptées à un module de hooks |

Si votre mod n'a pas de `tsconfig.json` propre, Claude Code en ajoute un à la racine du mod qui étend celui généré, pour que votre éditeur et `tsc -p ./first-mod` vérifient les types du mod sans configuration supplémentaire.

Les événements et méthodes peuvent changer d'une version à l'autre : faites donc confiance à ces fichiers plutôt qu'à toute page, celle-ci comprise, en cas de désaccord.

`claude-code/index.d.ts` est la référence la plus complète pour votre build, avec un commentaire et un exemple pour chaque méthode de l'API des mods. Pour chercher quelque chose, cherchez son nom dans le fichier, comme `'tool.call'`.

### Vérifier ce que Claude Code lit dans votre mod

Pour voir votre mod comme Claude Code le voit, sans exécuter votre code ni démarrer de session, utilisez `claude plugin validate`. Il vérifie le manifeste et effectue sur le source du module de hooks la même analyse statique que Claude Code effectue quand il charge un mod. Dans votre shell, lancez-le sur le répertoire du mod :

```bash theme={null}
claude plugin validate ./first-mod
```

Pour `first-mod`, la sortie inclut ces lignes.

```text theme={null}
  ❯ ./register.js hooks: session.start, tool.call, command.run{command=tally}, ui.render{component=Spinner}
  ❯ ./register.js calls: $.command.register, $.ui.invalidate

✔ Validation passed
```

Vérifiez la ligne `hooks:` pour les événements que votre module accroche, chacun avec son filtre entre accolades, et `calls:` pour chaque méthode de l'API des mods qu'il appelle. Si votre module lit ou définit des variables d'environnement, cherchez aussi les lignes `env reads:` et `env writes:`, ainsi que `state reads:` et `state writes:` s'il utilise [`$.state`](https://code.claude.com/docs/en/plugins/mods/interface#keep-state). Vous voyez aussi une ligne pour chaque hook capable de refuser une action, comme `gating hook without .catch: tool.call`, qui indique si ce hook a un [gestionnaire `.catch`](https://code.claude.com/docs/en/plugins/mods/events#handle-a-hook-that-fails).

Si un événement que vous vouliez traiter manque à la première ligne, Claude Code n'appellera pas non plus ce hook. La cause habituelle est un nom d'événement mal orthographié, que la commande signale par une erreur comme `"tool.calls" is not an event`.

Suivez ces règles pour que l'analyse statique trouve chaque hook et chaque appel :

* Écrivez chaque appel à l'API des mods en entier : `$`, l'espace de noms, puis la méthode, comme dans `$.store.get('notes')`. Vous pouvez passer `$` à une fonction déclarée au niveau supérieur du même fichier, et pour une fonction à vous nommée `loadNotes`, la ligne `calls:` affiche alors `$.store.get (via loadNotes)`. Passer `$` à une méthode, à une fonction définie à l'intérieur du hook, ou à une fonction importée d'un autre de vos fichiers fait échouer la validation. Les fonctions `read` et `update` qu'utilise [`$.state`](https://code.claude.com/docs/en/plugins/mods/interface#keep-state) sont les imports qui peuvent le recevoir. N'affectez pas `$` ni l'un de ses espaces de noms à une variable, ne le déstructurez pas, et ne l'indexez pas avec un nom calculé. `const ui = $.ui` échoue avec `$.ui is used as a value`.
* Écrivez le nom de l'événement dans chaque appel `on` sous forme de chaîne littérale, comme `'tool.call'`. Une variable, ou une boucle sur une liste de noms, échoue avec `the event name passed to on() is not a string literal`.
* Dans `register`, ne déclarez pas une seconde variable ou un second paramètre nommé `on`. La validation échoue avec `"on" is declared again (shadowed)`.
* N'importez que depuis des fichiers situés dans le répertoire du plugin, par chemin relatif. Le seul import nu autorisé est `claude-code`, pour les types et quelques utilitaires.
* Utilisez des déclarations `import` en haut du fichier, comme dans `import { name } from './file.js'`. Un `import()` dynamique échoue avec `a dynamic import(); a hooks module imports its own files with an import declaration`.
* Écrivez chaque fichier comme un module ES, avec `import` et non `require`. La [référence](https://code.claude.com/docs/en/plugins/mods/reference#files) liste les extensions de fichiers que Claude Code charge.

### Tester le mod

Vous pouvez écrire des tests automatisés pour un mod et les lancer depuis votre shell avec `claude plugin test`, sans session, sans connexion et sans réseau. Un test déclenche les événements que vos hooks traitent et vérifie ce qu'ont fait les hooks.

Ce test déclenche deux appels d'outils, lance `/tally`, et vérifie que la réponse les compte tous les deux. Enregistrez-le sous `first-mod/tests/first-mod.test.ts` :

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

[Tester un mod](https://code.claude.com/docs/en/plugins/mods/test) couvre le remplacement par stub d'un appel de modèle ou du magasin, et le test des minuteries et des dessins.

## Partager votre mod

Un mod est un plugin : vous le versionnez donc dans le manifeste, et les gens l'installent et le mettent à jour avec les commandes `/plugin`. La manière de le partager dépend de son public :

* **Quelques personnes** : envoyez-leur le répertoire du plugin ou un `.zip` de celui-ci. Voir [Partager un plugin sans marketplace](https://code.claude.com/docs/en/plugins/publish#share-a-plugin-without-a-marketplace)
* **Votre équipe** : listez-le dans [votre propre marketplace](https://code.claude.com/docs/en/plugins/publish#publish-through-your-own-marketplace), comme un dépôt privé avec un répertoire par plugin. Pour ajouter cette marketplace pour tous ceux qui travaillent dans un dépôt, [enregistrez-la dans les réglages du dépôt](https://code.claude.com/docs/en/plugins/host-marketplace#register-the-marketplace-for-everyone-in-a-repository)
* **Toute votre organisation** : un administrateur peut [installer les mods de votre organisation](https://code.claude.com/docs/en/plugins/mods/admin#install-your-organizations-mods) via les réglages gérés
* **N'importe qui** : rendez public le dépôt de votre marketplace, ou [soumettez le plugin à l'annuaire d'Anthropic](https://code.claude.com/docs/en/plugins/publish#submit-to-anthropics-directory)

Avant cela, vérifiez le `name` du plugin : `claude plugin validate` fait échouer un nom qui [ressemble à l'un de ceux d'Anthropic](https://code.claude.com/docs/en/plugins/manifest-reference#name), comme un nom commençant par `claude-`. Les événements et méthodes peuvent changer d'une version à l'autre : votre README est donc l'endroit où indiquer avec quelle version de Claude Code vous avez testé.

Continuez à développer sur le répertoire avec `--plugin-dir`, et non sur une copie installée. Claude Code met en cache un plugin installé par version, donc vos modifications n'atteignent pas la copie installée tant que vous n'avez pas incrémenté la version et réinstallé.

## Étapes suivantes

* [Dessiner dans l'interface](https://code.claude.com/docs/en/plugins/mods/interface) : ouvrir un panneau, dessiner au-dessus du prompt, et ajouter des boutons et des champs de texte
* [Réagir aux événements](https://code.claude.com/docs/en/plugins/mods/events) : accrocher les appels d'outils, les prompts et les tours
* [Utiliser l'API des mods](https://code.claude.com/docs/en/plugins/mods/api) : ajouter des commandes et des outils, appeler un modèle, et exécuter du travail sur minuterie
* [Tester un mod](https://code.claude.com/docs/en/plugins/mods/test) : remplacer par stub ce que Claude Code répondrait, et tester minuteries et dessins
* [Dépanner un mod](https://code.claude.com/docs/en/plugins/mods/troubleshoot) : les raisons pour lesquelles un mod ne fait rien, et le journal de débogage
* [Lire le source des mods intégrés](https://code.claude.com/docs/en/plugins/mods/overview#read-the-source-of-built-in-mods) : des plugins complets, chacun avec son module de hooks et ses tests
