# Dessiner dans l'interface avec un mod

> Source : https://code.claude.com/docs/en/plugins/mods/interface · [← Index](../31-index-documentation-officielle.md)

*Dessinez des panneaux, une bande au-dessus du prompt, des boutons et des champs de texte depuis un mod Claude Code, gérez les appuis et la saisie, et conservez l'état entre les redessins et d'une session à l'autre.*

Un mod peut dessiner sa propre interface dans Claude Code et modifier des parties de l'interface que Claude Code dessine déjà. Chaque endroit où un mod peut dessiner s'appelle un [site de rendu](https://code.claude.com/docs/en/plugins/mods/reference#render-sites) (render site) : un panneau (pane), la bande au-dessus du prompt, le spinner, etc. Claude Code déclenche l'événement [`ui.render`](https://code.claude.com/docs/en/plugins/mods/reference#interface) chaque fois qu'il s'apprête à dessiner un site de rendu, et votre hook pour cet événement renvoie ce qu'il faut y dessiner.

La page contient une carte des endroits où un mod peut dessiner dans une session de terminal (rendu plein écran) : un mod peut ajouter un panneau en barre latérale à droite, un toast en haut à droite de la transcription, une ligne de journal dans la transcription, une bande au-dessus du prompt et une ligne de statut sous le prompt. Un mod peut redessiner les messages, les lignes d'appel d'outil et le spinner. Le prompt, lui, appartient à Claude Code.

Dans un terminal plus étroit, le panneau se place au-dessus du prompt au lieu d'être à côté de la transcription.

Construisez votre [premier mod](https://code.claude.com/docs/en/plugins/mods/create) avant de commencer ici. Commencez par l'exemple guidé, qui construit un panneau avec deux onglets et un compteur, puis lisez la section consacrée à chaque élément que vous voulez modifier.

> **Note** : pour retrouver une prop ou une limite précise, voir la [référence](https://code.claude.com/docs/en/plugins/mods/reference#render-sites).

## Construire un panneau avec des onglets

Dans cette section, vous construisez un mod qui ajoute une commande `/hello-tabs`, laquelle ouvre un panneau. Un panneau est une barre latérale à côté de la transcription dans un terminal large en plein écran, ou une zone encadrée au-dessus du prompt sinon. Ce panneau affiche deux onglets, et le second contient un bouton qui ajoute un à un compteur. Le compte est toujours là après un redémarrage de Claude Code.

Le mod terminé ressemble à ceci (vidéo dans la source) : la commande `/hello-tabs` est tapée au prompt et un panneau encadré s'ouvre au-dessus, avec `1: One` et `2: Two` en haut et le texte `This is the first tab.` Le second onglet montre un bouton `Add one` à côté de `Count: 1`, et le compte monte jusqu'à 3. Le panneau revient ensuite au premier onglet.

Les onglets sont deux boutons sur une même ligne. Le mod mémorise lequel est actif et dessine le contenu de cet onglet sous la ligne.

### Étape 1 — Créer le plugin

Un mod est un plugin avec un manifeste, un `hooks.json` qui pointe vers votre code, et le fichier de code. [Créer un mod](https://code.claude.com/docs/en/plugins/mods/create#write-a-mod-yourself) explique chacun. Créez un répertoire nommé `hello-tabs` contenant les répertoires `.claude-plugin` et `hooks`, puis enregistrez les deux premiers fichiers.

Enregistrez le manifeste sous `hello-tabs/.claude-plugin/plugin.json` :

```json hello-tabs/.claude-plugin/plugin.json theme={null}
{
  "name": "hello-tabs",
  "version": "0.1.0",
  "description": "Opens a pane with two tabs and a counter",
  "author": { "name": "Your Name" }
}
```

Nommez votre point d'entrée dans `hello-tabs/hooks/hooks.json` :

```json hello-tabs/hooks/hooks.json theme={null}
{
  "modules": ["./register.js"]
}
```

### Étape 2 — Écrire le code

Cette liste indique ce que fait chaque hook, dans l'ordre où ils apparaissent dans le code :

* Ajoute la commande `/hello-tabs`, et charge le compte enregistré par une session précédente
* Ouvre le panneau quand vous exécutez cette commande
* Dessine le contenu du panneau : la ligne d'onglets et le corps de l'onglet ouvert

Deux variables de niveau module, `tab` et `count`, contiennent l'état du panneau.

Enregistrez ceci sous `hello-tabs/hooks/register.js` :

```javascript hello-tabs/hooks/register.js theme={null}
// L'id du panneau, utilisé pour l'ouvrir et pour le reconnaître au moment du dessin
const PANE = 'hello-tabs'

// Ce que montre le panneau : l'onglet ouvert, et la valeur du compteur
let tab = 'one'
let count = 0

export function register(on) {
  // S'exécute avant votre premier prompt, et de nouveau après un rechargement
  on('session.start', async ($, e, next) => {
    await $.command.register({ name: 'hello-tabs', description: 'Open the hello-tabs pane' })
    // Charge le compte enregistré par une session précédente, s'il existe
    const saved = await $.store.get('count')
    if (typeof saved === 'number') count = saved
    return next(e)
  })

  // S'exécute quand vous tapez /hello-tabs
  on('command.run', { command: 'hello-tabs' }, async ($) => {
    // Ouvre le panneau, lui donne le clavier, et laisse Échap le fermer
    await $.ui.open({ id: PANE, title: 'Hello tabs', focus: true, closeOnEscape: true })
    // N'affiche rien dans la transcription
    return {}
  })

  // S'exécute chaque fois que Claude Code dessine un panneau
  on('ui.render', { component: 'Pane' }, async ($, e, next) => {
    // Ne touche pas aux panneaux des autres mods
    if (e.requestId !== PANE) return next(e)
    // Récupère les éléments que cette application sait dessiner
    const { Box, Text, Button } = $.ui.resolve(e)
    // Demande à Claude Code d'exécuter à nouveau ce hook
    const redraw = () => $.ui.invalidate('ui.render')

    // Un onglet : un bouton qui bascule sur son onglet quand on le presse
    const tabButton = (name, label, hotkey) =>
      Button({
        key: 'tab-' + name,
        label,
        hotkey,
        plain: true,
        // Grise l'onglet qui n'est pas ouvert
        dimColor: tab !== name,
        onPress: () => {
          tab = name
          redraw()
        },
      })

    // Ce qui va sous les onglets, selon celui qui est ouvert
    const body =
      tab === 'one'
        ? [Text({ children: ['This is the first tab.'] })]
        : [
            Box({
              flexDirection: 'row',
              columnGap: 2,
              children: [
                Button({
                  key: 'more',
                  label: 'Add one',
                  hotkey: 'a',
                  onPress: async () => {
                    count += 1
                    redraw()
                    // Enregistre le compte pour qu'il soit là après un redémarrage
                    await $.store.set('count', count)
                  },
                }),
                Text({ children: ['Count: ' + count] }),
              ],
            }),
          ]

    // Le panneau complet : la ligne d'onglets, une ligne vide, puis le corps
    return Box({
      flexDirection: 'column',
      children: [
        Box({
          flexDirection: 'row',
          columnGap: 3,
          children: [tabButton('one', 'One', '1'), tabButton('two', 'Two', '2')],
        }),
        Text({ children: [' '] }),
        ...body,
      ],
    })
  })
}
```

Chaque hook fait aussi quelque chose que le code ne rend pas évident :

* **[`session.start`](https://code.claude.com/docs/en/plugins/mods/reference#session)** lit aussi le compte enregistré depuis [`$.store`](#conserver-létat), un magasin clé-valeur qui persiste d'une session à l'autre.
* **[`command.run`](https://code.claude.com/docs/en/plugins/mods/api#add-a-command)** indique seulement à Claude Code que le panneau existe. Ouvrir un panneau ne dessine rien en soi : Claude Code déclenche ensuite `ui.render` pour demander ce qu'il contient.
* **`ui.render`** renvoie l'arbre d'éléments — une `Box` contenant d'autres boîtes, du texte et des boutons — et le reconstruit à partir de `tab` et `count` à chaque exécution.

Presser un bouton exécute son callback `onPress`, qui modifie une variable et appelle `redraw`. Claude Code exécute alors de nouveau le hook `ui.render`, qui construit un nouvel arbre à partir des nouvelles valeurs. Tout dessin interactif suit ce cycle de rendu : un callback modifie l'état, et le hook refait le rendu à partir du nouvel état.

### Étape 3 — Ouvrir le panneau

Dans votre shell, démarrez Claude Code avec `claude --plugin-dir ./hello-tabs`. Au prompt de Claude Code, exécutez `/hello-tabs`. Un panneau s'ouvre avec `1: One` et `2: Two` en haut. Appuyez sur `2`, puis plusieurs fois sur `a`, le raccourci (hotkey) de **Add one**. Le compte augmente.

### Étape 4 — Vérifier que le compte a été enregistré

Appuyez sur Échap pour fermer le panneau, puis quittez la session. Dans votre shell, redémarrez Claude Code avec la même commande `claude --plugin-dir ./hello-tabs`, et exécutez `/hello-tabs` au prompt. Le compte est là où vous l'aviez laissé.

Pour remettre le compte à zéro, faites appeler `$.store.delete('count')` par le mod. [Conserver l'état](#conserver-létat) explique combien de temps dure chaque type de valeur.

## Choisir où dessiner

Un hook `ui.render` s'exécute pour chaque site de rendu, à moins que vous ne le restreigniez à celui où vous voulez dessiner. Pour choisir le site, passez un filtre, appelé [matcher](https://code.claude.com/docs/en/plugins/mods/events#filter-which-events-a-hook-handles), comme second argument de `on`. `{ component: 'Pane' }` n'exécute le hook que pour les panneaux. Dans le hook, `e.component` nomme le site, `e.surface` indique quelle application dessine, et `e.props` contient les données propres au site. Pour un panneau, `e.requestId` est l'`id` avec lequel vous l'avez ouvert.

Le panneau et la bande sont vides tant qu'un mod ne les remplit pas. Voici ce qu'est chacun et comment y dessiner :

### Onglet « Panneau » (Pane)

Un panneau est une barre latérale à côté de la transcription dans un terminal large en plein écran, ou une zone encadrée au-dessus du prompt sinon. Quand plusieurs panneaux sont ouverts, chacun reçoit un onglet qui affiche son titre.

Un panneau apparaît quand votre mod appelle `$.ui.open` avec un `id` de votre choix, comme dans `$.ui.open({ id: 'hello-tabs' })`. [Ouvrir un panneau au bon moment](#ouvrir-un-panneau-au-bon-moment) couvre les autres champs et les cas où un panneau attend un terminal plus large.

Pour dessiner dans votre panneau, filtrez sur `{ component: 'Pane' }` et vérifiez que `e.requestId` est votre `id`.

### Onglet « Bande au-dessus du prompt » (Band above the prompt)

La bande est une bande située juste au-dessus de la zone de saisie du prompt. Elle est toujours présente, et tous les mods la partagent.

Votre hook renvoie un arbre pour afficher quelque chose dans la bande, ou `next(e)` pour ne rien afficher. Un arbre remplace ce que dessinent dans la bande les mods qui s'exécutent [après le vôtre](https://code.claude.com/docs/en/plugins/mods/events#the-order-mods-run-in). Pour conserver leur dessin, placez le résultat de `await next(e)` parmi les enfants (`children`) d'une [`Box`](#construire-un-arbre-à-partir-déléments) de votre arbre.

Pour dessiner dans la bande, filtrez sur `{ component: 'AbovePrompt' }`.

### Changer ce que Claude Code dessine déjà

Claude Code dessine lui-même la plus grande partie de son interface : messages, lignes d'appel d'outil, spinner, etc. Chacune de ces parties est aussi un site de rendu, qu'un mod peut donc restyler ou remplacer. Pour en modifier une, filtrez votre hook `ui.render` sur son nom, d'après ce tableau :

| Site | Ce que c'est |
| :- | :- |
| `UserMessage`, `AssistantMessage` | Un message dans la transcription |
| `ToolUse`, `ToolResult`, `ToolGroup` | La ligne d'un appel d'outil, son résultat, et un groupe d'appels replié |
| `CommandOutput` | La ligne qu'a affichée une commande |
| `AskUserQuestion` | La boîte de dialogue que Claude ouvre pour vous poser une question |
| `Spinner`, `ToolProgress`, `TurnDuration` | Les lignes de statut d'un tour : la ligne animée pendant que Claude travaille, la ligne de progression en direct d'un outil en cours, et la ligne qui clôt un tour |
| `InfoNotice`, `SessionMode`, `PromptHint` | Les lignes de statut sous le logo, les libellés de mode dans le pied de page, et la ligne d'indice sous le prompt |

Sur un site que Claude Code dessine déjà, votre hook peut changer un détail, remplacer le dessin ou le laisser tel quel. Chaque choix est appliqué ci-dessous au spinner. Les exemples lisent une variable `calls` qu'incrémente un autre hook, comme dans le [mod du tutoriel](https://code.claude.com/docs/en/plugins/mods/create#write-a-mod-yourself).

#### Onglet « Changer un détail »

Pour garder le dessin de Claude Code et en changer une partie, passez à `next` une copie de l'événement avec des `props` modifiées. Ce hook change le texte qui suit le mot du spinner :

```javascript theme={null}
on('ui.render', { component: 'Spinner' }, async ($, e, next) => {
  // Garde le spinner de Claude Code, et change le texte après son mot
  return next({ ...e, props: { ...e.props, suffix: ' · tool calls: ' + calls + '…' } })
})
```

Le spinner garde son animation et son mot, et votre texte suit le mot :

```text theme={null}
Thinking · tool calls: 2…
```

#### Onglet « Remplacer le dessin »

Pour dessiner quelque chose à vous à la place du site, renvoyez un arbre et n'appelez pas `next`. Ce hook dessine une ligne de texte là où serait le spinner :

```javascript theme={null}
on('ui.render', { component: 'Spinner' }, async ($, e) => {
  const { Text } = $.ui.resolve(e)
  // Pas d'appel à next : cette ligne est donc dessinée à la place du spinner
  return Text({ children: ['Claude has made ' + calls + ' tool calls'] })
})
```

Pendant que Claude travaille, votre ligne s'affiche et le spinner de Claude Code non :

```text theme={null}
Claude has made 2 tool calls
```

#### Onglet « Laisser tel quel »

Pour laisser le site tel que Claude Code le dessine, renvoyez `next(e)`. Un hook le fait souvent pour certains événements et pas pour d'autres. Ce hook laisse le spinner intact tant qu'il n'y a pas d'appel à compter :

```javascript theme={null}
on('ui.render', { component: 'Spinner' }, async ($, e, next) => {
  // Rien à afficher pour l'instant : on transmet l'événement inchangé
  if (calls === 0) return next(e)
  return next({ ...e, props: { ...e.props, suffix: ' · tool calls: ' + calls + '…' } })
})
```

Avant le premier appel d'outil, le spinner a le même aspect que sans le mod :

```text theme={null}
Thinking…
```

#### Référence au dessin de Claude Code

Sur ces sites, `next(e)` renvoie une référence au dessin de Claude Code, `{ type: 'engine', ref }`, sauf si un mod qui s'exécute après le vôtre a renvoyé un arbre à lui. Pour changer le contenu de ce dessin, passez à `next` une copie de l'événement avec d'autres props, comme dans l'onglet **Changer un détail**. Vous pouvez renvoyer la référence telle quelle, ou la placer dans une `Box` à côté d'éléments à vous :

```javascript theme={null}
on('ui.render', { component: 'Spinner' }, async ($, e, next) => {
  const { Box, Text } = $.ui.resolve(e)
  const theirs = await next(e)
  return Box({ flexDirection: 'column', children: [theirs, Text({ children: ['under the spinner'] })] })
})
```

Pendant que Claude travaille, le spinner s'anime comme avant, et `under the spinner` apparaît en dessous.

L'invite de permission (permission prompt) n'est pas un site de rendu : un mod ne peut donc pas changer ce qu'elle affiche. La boîte de dialogue de question, `AskUserQuestion`, en est un : un mod peut donc la modifier. Un arbre pour cette boîte de dialogue doit contenir la référence exactement une fois, avec vos éléments au-dessus. Sinon, Claude Code dessine sa propre boîte de dialogue.

Le terminal et l'application Desktop ne déclenchent pas tous les mêmes sites. `Pane`, `AbovePrompt`, `Spinner` et les sites de la transcription fonctionnent dans les deux. Quelques autres lignes de statut ne sont déclenchées que dans le terminal. Le [tableau des sites de rendu](https://code.claude.com/docs/en/plugins/mods/reference#render-sites) indique où chacun est déclenché.

### Ouvrir un panneau au bon moment

Un panneau n'apparaît que lorsque votre mod l'ouvre. La manière et le moment de l'ouverture déterminent s'il prend le focus clavier, combien de place il demande, et s'il s'affiche tout court dans un terminal étroit.

Pour ouvrir un panneau, appelez [`$.ui.open`](https://code.claude.com/docs/en/plugins/mods/reference#mods-api-methods) avec un `id` de votre choix. L'`id` est le nom du panneau : votre hook `ui.render` le vérifie, et vous le passez de nouveau pour fermer le panneau.

```javascript theme={null}
await $.ui.open({ id: 'hello-tabs', title: 'Hello tabs', focus: true })
```

Pour fermer le panneau, appelez `$.ui.close` avec l'`id` utilisé à l'ouverture :

```javascript theme={null}
await $.ui.close({ id: 'hello-tabs' })
```

En plus d'`id`, `$.ui.open` accepte ces champs facultatifs :

| Champ | Effet |
| :- | :- |
| `title` | Le libellé de l'onglet du panneau quand plusieurs panneaux sont ouverts |
| `focus` | Demande le [focus clavier](#focus-clavier-et-raccourcis) |
| `closeOnEscape` | Fait qu'Échap ferme le panneau |
| `holdToasts` | Dans le terminal, retient les toasts tant que ce panneau est celui qui est affiché. Voir [Retenir les toasts derrière une boîte de dialogue](#retenir-les-toasts-derrière-une-boîte-de-dialogue). |
| `rows` | La hauteur à demander quand le panneau est au-dessus du prompt. Par défaut, un tiers de l'espace. |
| `columns` | La largeur à demander quand le panneau est à côté de la transcription |

`focus`, `closeOnEscape` et `holdToasts` sont facultatifs et n'acceptent que `true`. Pour en désactiver un, omettez-le. Passer `false` lève une erreur comme `ui.open: focus is true or left out`. Pour en définir un de façon conditionnelle, n'ajoutez le champ que si la condition est remplie. Cet appel ne demande le focus clavier que si `items` n'est pas vide :

```javascript theme={null}
const pane = { id: 'hello-tabs', title: 'Hello tabs' }
await $.ui.open(items.length > 0 ? { ...pane, focus: true } : pane)
```

Pour permettre à une commande d'ouvrir le panneau pendant que Claude travaille, ajoutez `immediate: true` lorsque vous [enregistrez la commande](https://code.claude.com/docs/en/plugins/mods/api#add-a-command). Sans cela, une commande tapée pendant un tour attend la fin du tour.

#### Retenir les toasts derrière une boîte de dialogue

Passez `holdToasts: true` à `$.ui.open` quand le panneau est une boîte de dialogue à laquelle l'utilisateur répond puis qu'il quitte, afin que les toasts n'apparaissent pas pendant qu'il décide. Dans le terminal, la retenue dure tant que ce panneau est celui qui est affiché, et un toast déclenché pendant ce temps attend la fin de la retenue.

Claude Code retient les toasts des autres mods et ses propres notifications éphémères, en plus de ceux que votre mod déclenche avec [`$.ui.toast`](https://code.claude.com/docs/en/plugins/mods/api#show-something-without-starting-a-turn). N'utilisez pas ce champ pour un panneau qui reste ouvert, afin que l'utilisateur continue de les voir.

#### Quand un panneau attend un terminal plus large

Un panneau que votre mod ouvre sans qu'on le lui demande n'apparaît pas dans un terminal étroit, pour ne pas envahir un petit écran. Son affichage dépend de ce qui l'a ouvert :

* **Ouvert par une action de l'utilisateur**, comme une commande qu'il a exécutée ou un bouton qu'il a pressé : le panneau apparaît quelle que soit la largeur
* **Ouvert par votre mod de lui-même**, par exemple depuis une minuterie ou un hook [`turn.start`](https://code.claude.com/docs/en/plugins/mods/events#follow-a-turn) : le panneau n'apparaît que dans un terminal d'au moins 144 colonnes. Une fois que l'utilisateur a ouvert lui-même ce panneau, 110 colonnes suffisent.

Quand le panneau apparaît, `$.ui.open` se résout avec `{ isPlaced: true }`. Quand le panneau est en attente, `isPlaced` vaut `false` et `reason` est une chaîne qui explique pourquoi. Un panneau en attente apparaît quand l'utilisateur l'ouvre ou élargit le terminal. Pour signaler qu'une chose est disponible sans ouvrir de panneau, appelez `$.ui.toast('Your message')`, qui affiche une notification toast.

## Construire un arbre à partir d'éléments

Ce que renvoie un hook `ui.render` est un arbre d'éléments : une description de ce qu'il faut dessiner, faite de boîtes, de texte et de contrôles imbriqués les uns dans les autres. Vous décrivez le dessin, et Claude Code le rend dans le terminal ou dans l'application Desktop.

Pour obtenir les éléments, appelez `$.ui.resolve(e)` dans votre hook, comme dans `const { Box, Text, Button } = $.ui.resolve(e)`. Chaque élément est une fonction. Vous lui passez des props, et vous placez dans `children` les éléments et chaînes qu'il contient.

Voici chacun des éléments les plus courants et la façon dont le terminal le dessine :

### Onglet « Text »

`Text` dessine une chaîne, avec un style facultatif comme `bold` et `color` :

```javascript theme={null}
Text({ children: ['This is the first tab.'] })
```

```text theme={null}
This is the first tab.
```

### Onglet « Box »

`Box` dispose ce qu'elle contient, en ligne ou en colonne. Celle-ci place un bouton et une ligne de texte côte à côte, séparés de deux colonnes :

```javascript theme={null}
Box({
  flexDirection: 'row',
  columnGap: 2,
  children: [
    Button({ key: 'more', label: 'Add one', onPress: addOne }),
    Text({ children: ['Count: 0'] }),
  ],
})
```

```text theme={null}
[ Add one ]  Count: 0
```

### Onglet « Button »

`Button` est un contrôle que l'utilisateur peut presser. Il exécute votre callback `onPress`. Avec `plain: true`, il n'a pas de crochets et affiche son raccourci (hotkey) :

```javascript theme={null}
Button({ key: 'more', label: 'Add one', onPress: addOne })
Button({ key: 'tab-one', label: 'One', hotkey: '1', plain: true, onPress: showTabOne })
```

```text theme={null}
[ Add one ]
1: One
```

### Onglet « Input »

`Input` est un champ de texte. Il exécute votre callback `onSubmit` avec le texte quand l'utilisateur appuie sur Entrée :

```javascript theme={null}
Input({
  key: 'new-note',
  label: 'Note',
  placeholder: 'Type a note and press Enter',
  value: '',
  submitLabel: 'add',
  onSubmit: addNote,
})
```

```text theme={null}
Note: Type a note and press Enter
```

### Tous les éléments

La [galerie de l'interface](https://code.claude.com/docs/en/plugins/mods/gallery) propose des exemples et des captures de la plupart des éléments. Ce tableau liste tous les éléments :

| Élément | Ce qu'il dessine | Où |
| :- | :- | :- |
| `Box` | Un conteneur flex. Accepte des props de mise en page comme `flexDirection`, `columnGap`, `padding`, [`borderStyle`](https://code.claude.com/docs/en/plugins/mods/reference#box-border-styles) et `width`. | Partout |
| `Text` | Du texte stylé. Accepte `color`, `bold`, `dimColor`, `italic` et `wrap`. Une `color` est une clé de thème ou une couleur comme `'red'`. Un `wrap` vaut `'wrap'`, `'truncate'`, `'truncate-start'`, `'truncate-middle'` ou `'truncate-end'`. | Partout |
| `Button` | Un contrôle qui appelle `onPress` | Partout |
| `Link`, `Code`, `Markdown` | Un lien avec `href` et un `label` facultatif, un bloc de code, et du texte formaté comme les réponses de Claude. `Markdown` prend son contenu dans une prop `text`, pas dans `children`, et a besoin d'une `key` quand vous passez `onLinkPress`. | Partout |
| `Input`, `Select` | Un champ de texte et une liste déroulante | Terminal, Desktop |
| `Svg` | Un document SVG | Desktop |
| `Client` | Une zone dessinée par un second fichier à vous, pour l'animation et la saisie au pointeur. Ce fichier n'a pas accès à l'API des mods. Il communique avec vos hooks en postant des données, qui arrivent sous forme d'événement `ui.message`. S'il ne parvient pas à se charger, à se dessiner ou à s'exécuter, vos hooks reçoivent un événement [`ui.fault`](https://code.claude.com/docs/en/plugins/mods/reference#interface). | Terminal, Desktop |
| `Raster`, `Image` | Une [grille de cellules colorées](#dessiner-une-grille-de-cellules-colorées), et une image | Terminal |

Si votre module est un fichier `.tsx` ou `.jsx`, vous pouvez écrire l'arbre en JSX. Déstructurez d'abord les éléments depuis `$.ui.resolve(e)`.

Si un arbre utilise un élément que l'application ne possède pas, une prop qu'un élément n'accepte pas, ou un enfant là où il n'en faut pas, Claude Code dessine sa propre version du site.

Dans une session démarrée avec `--plugin-dir`, une ligne de la transcription le signale, par exemple `ui.render (Pane) refused: Text prop "bogusProp" is not allowed; the engine drew its own`. Le [journal de débogage](https://code.claude.com/docs/en/plugins/mods/troubleshoot#read-the-debug-log) l'enregistre sous la forme `ui.render (Pane): a hook returned a tree that does not validate`, avec la même raison. Rien d'autre n'apparaît dans la session : quand un dessin ne s'affiche pas, vérifiez cette ligne ou le journal.

### `Link` dans l'application Desktop

Dans l'application Desktop, un `Link` se dessine comme du texte brut sauf si son `href` respecte ces exigences :

* **Schéma et hôte** : une URL `https:`, ou une URL `http://localhost` comme `http://localhost:3000`
* **Pas de `@`** : écrivez un `@` du chemin ou de la requête sous la forme `%40`
* **Graphie** : ce que renvoie `new URL(href).href`, à l'exception d'un `/` manquant après l'hôte. Cela exclut un hôte en majuscules, une espace, et `:443` sur une URL `https:`.

Dans le terminal, ces exigences ne s'appliquent pas.

### Quand un `Client` échoue

Dans le terminal, quand le fichier qu'exécute un `Client` échoue, une ligne grisée comme `my-mod: Client client/spinner.js: boom` prend la place du `Client`, et le reste de votre dessin s'affiche toujours.

Si votre mod gère [`ui.fault`](https://code.claude.com/docs/en/plugins/mods/reference#interface), Claude Code [redessine ensuite le site](#quand-claude-code-redessine-sans-quon-le-lui-demande).

### Dessiner une grille de cellules colorées

Pour une carte de chaleur (heat map), une sparkline ou un plateau de jeu dans le terminal, dessinez un seul `Raster` plutôt qu'une `Box` par cellule. Un `Raster` prend une `key`, sa taille en `columns` et `rows`, et `cells`, une chaîne base64 qui encode toutes les cellules. Chaque cellule est composée de trois nombres : le point de code du caractère, sa couleur et sa couleur de fond. Une couleur est une valeur RVB 24 bits en hexadécimal, comme `0xc62828` pour un rouge. La valeur `0x01000000`, juste au-dessus de cette plage, désigne la couleur par défaut du terminal.

L'application Desktop n'a pas de `Raster` : vérifiez donc `e.surface` et dessinez du texte dans ce cas. Ce corps de panneau dessine une carte de chaleur de trois sur deux :

```javascript theme={null}
// La valeur qui signifie « utiliser la couleur par défaut du terminal »
const DEFAULT_COLOR = 0x01000000

// Encode des lignes de paires [caractère, couleur] dans la chaîne unique qu'attend un Raster
// Une cellule = trois nombres : le point de code du caractère, sa couleur et son fond
function cellsOf(rows) {
  const numbers = rows.flat().flatMap(([char, color]) => [char.codePointAt(0), color, DEFAULT_COLOR])
  return new Uint8Array(Uint32Array.from(numbers).buffer).toBase64()
}

on('ui.render', { component: 'Pane' }, async ($, e, next) => {
  // Ne dessine que dans le panneau ouvert avec l'id 'heat'
  if (e.requestId !== 'heat') return next(e)
  const { Box, Text, Raster } = $.ui.resolve(e)
  // Deux lignes de trois cellules, chacune un caractère bloc et sa couleur
  const rows = [
    [['█', 0x2e7d32], ['█', 0xf9a825], ['█', 0xc62828]],
    [['█', 0x2e7d32], ['█', 0x2e7d32], ['█', 0xf9a825]],
  ]
  if (e.surface !== 'terminal') {
    return Text({ children: ['The heat map needs the terminal.'] })
  }
  return Box({
    flexDirection: 'column',
    children: [Raster({ key: 'grid', columns: 3, rows: 2, cells: cellsOf(rows) })],
  })
})
```

Dans le terminal, le panneau affiche la grille : une petite grille de blocs colorés sur deux lignes de trois — la ligne du haut verte, ambre et rouge ; celle du bas verte, verte et ambre.

Le tableau `rows` est la partie que vous modifieriez, et `cellsOf` le transforme en chaîne encodée. Le hook ne dessine que dans un panneau dont l'`id` est `heat` : ouvrez-en un avec `$.ui.open({ id: 'heat' })` depuis une commande, comme l'[exemple `hello-tabs`](#construire-un-panneau-avec-des-onglets) ouvre son panneau.

Chaque caractère doit occuper une seule cellule de large. Pour animer un `Raster` déjà affiché, appelez `$.ui.blit` avec l'`id` du panneau comme `requestId`, la `key` du `Raster`, la même taille, et de nouvelles cellules. Pour cet exemple : `$.ui.blit({ requestId: 'heat', key: 'grid', columns: 3, rows: 2, cells: cellsOf(newRows) })`. Cela repeint ce seul élément sans réexécuter votre hook `ui.render`.

## Répondre aux appuis et à la saisie

Quand l'utilisateur presse un bouton, tape dans un champ ou fait un choix dans une liste que votre mod a dessinés, Claude Code appelle le callback de ce contrôle, qui s'exécute dans votre module. Chaque contrôle accepte ses propres callbacks :

* **`Button`** : accepte `onPress(e)`, où `e.surface` est l'application d'où vient l'appui
* **`Input`** : accepte `onSubmit(value)` et `onInput(value)`
* **`Select`** : accepte `onSelect(value)`, avec ses choix dans `options`, une liste d'au moins un choix aux valeurs uniques, comme `[{ value: 'sm', label: 'Small' }, { value: 'lg', label: 'Large' }]`

Un test presse un contrôle ou tape dedans via sa `key` : donnez-en donc une à chaque contrôle. Chaque utilisation d'un contrôle déclenche aussi [`ui.press`, `ui.input` ou `ui.select`](https://code.claude.com/docs/en/plugins/mods/reference#interface) avec la `key` dans `e.element`, et un autre mod peut gérer ces événements. Son hook s'exécute avant votre callback : il voit donc ce que l'utilisateur tape dans votre `Input` et peut le modifier ou répondre à la place de votre callback. L'API des mods n'a aucune méthode pour presser le bouton d'un autre mod.

### Focus clavier et raccourcis

Votre mod ne lit jamais le clavier lui-même. L'utilisateur appuie sur une touche, Claude Code décide auquel de vos contrôles elle est destinée, et le callback de ce contrôle s'exécute. Hormis un [raccourci numérique sur la bande](https://code.claude.com/docs/en/plugins/mods/reference#elements), cela n'arrive que lorsque votre panneau ou votre bande a le focus clavier. Le reste du temps, les touches vont au prompt.

#### Comment un panneau obtient le focus clavier

Un panneau obtient le focus clavier quand :

* Votre mod l'ouvre avec `focus: true` depuis une commande ou un appui
* L'utilisateur appuie sur Ctrl+X puis Tab
* L'utilisateur clique dessus

Claude Code n'accorde `focus: true` que tant que le prompt est vide et que rien d'autre n'a le focus clavier. Un panneau qui s'ouvre pendant que l'utilisateur tape ne lui prend pas ses frappes.

#### Ce que fait chaque touche

Ce tableau liste l'effet d'une touche pendant que votre panneau ou votre bande a le focus clavier :

| Touche | Effet |
| :- | :- |
| Tab | Passe au contrôle suivant |
| Haut et Bas | Se déplacent entre les contrôles tant que votre dessin tient. Quand le panneau ou la bande a plus de lignes qu'il ne peut en afficher, elles le font défiler. |
| Entrée | Presse le `Button` qui a le focus, soumet l'`Input` qui a le focus, ou valide le choix dans un `Select` |
| Le raccourci d'un bouton | Presse ce bouton. Pendant qu'un `Input` a le focus, toute touche imprimable va dans le champ. |
| Page Up, Page Down, Home et End | Font défiler votre panneau ou votre bande quand il a plus de lignes qu'il ne peut en afficher |
| Ctrl+X puis une flèche | Redimensionne votre panneau. Gauche ou Haut lui donne plus de place, Droite ou Bas la rend. |
| Ctrl+X puis X | Ferme votre panneau, même pendant qu'un de ses champs a le focus |
| Échap | Rend le focus clavier au prompt. Avec `closeOnEscape: true`, ferme aussi le panneau. |

Un mod ne peut pas affecter Tab ni les flèches à autre chose : un jeu se dirige donc avec `w`, `a`, `s` et `d`.

#### Définir un raccourci et le premier focus

Ces props d'un contrôle déterminent comment le clavier l'atteint :

* **`hotkey`** : pour permettre à l'utilisateur de presser un `Button` d'une seule touche, donnez-lui un `hotkey` d'un chiffre ou d'une lettre minuscule, comme `hotkey: 'a'`
* **`autoFocus`** : pour choisir le contrôle qui a le focus à l'ouverture du panneau, ajoutez-lui `autoFocus: true`. La prop n'accepte que `true` : omettez-la sur les autres contrôles.

L'affichage d'un raccourci dépend du bouton et de l'application :

| Bouton | Dans le terminal | Dans l'application Desktop |
| :- | :- | :- |
| Avec crochets, le défaut | `[ Add one ]`, sans raccourci affiché | Le libellé avec une petite touche à côté |
| Avec `plain: true` | `1: One` | Le libellé avec une petite touche à côté |

Dans le terminal, nommez la touche dans le libellé d'un bouton à crochets, ou utilisez `plain: true`, pour que l'utilisateur voie sur quoi appuyer. La [référence des éléments](https://code.claude.com/docs/en/plugins/mods/reference#elements) donne les autres règles de `Button` : `action`, raccourcis numériques sur la bande, et deux boutons sur le même raccourci.

### Recevoir une saisie et dessiner une ligne par élément

Beaucoup de panneaux sont un champ de texte suivi d'une liste. L'exemple de cette section est un panneau de notes : vous tapez une note et appuyez sur Entrée pour l'ajouter, et chaque note a un bouton `x` qui la supprime. Avec deux notes ajoutées, le terminal dessine le panneau ainsi :

```text theme={null}
╭────────────────────────────────────────────────────────✕─╮
│ Note: Type a note and press Enter ⏎ add                  │
│ x buy milk                                               │
│ x call bob                                               │
╰──────────────────────────────────────────────────────────╯
```

Le `✕` sur la bordure supérieure est la marque propre de Claude Code pour fermer le panneau.

L'exemple utilise ces techniques :

* **Recevoir une saisie** : un `Input` appelle `onSubmit(value)` avec le texte du champ quand l'utilisateur appuie sur Entrée, et `onInput(value)` à chaque modification
* **Dessiner une liste** : associez à chaque donnée une ligne, et donnez au bouton de chaque ligne sa propre `key`

Ce hook dessine le contenu du panneau :

```javascript theme={null}
// La liste que dessine le panneau
let notes = []

on('ui.render', { component: 'Pane' }, async ($, e, next) => {
  // Ne dessine que dans le panneau ouvert avec l'id 'notes'
  if (e.requestId !== 'notes') return next(e)
  const { Box, Text, Button, Input } = $.ui.resolve(e)
  const redraw = () => $.ui.invalidate('ui.render')

  return Box({
    flexDirection: 'column',
    children: [
      Input({
        key: 'new-note',
        label: 'Note',
        placeholder: 'Type a note and press Enter',
        // Dessine le champ vide à chaque fois, ce qui le vide après une soumission
        value: '',
        submitLabel: 'add',
        autoFocus: true,
        // S'exécute quand vous appuyez sur Entrée dans le champ
        onSubmit: async (value) => {
          // Ignore une ligne vide
          if (!value.trim()) return
          notes = [...notes, value.trim()]
          redraw()
          await $.store.set('notes', notes)
        },
      }),
      // Une ligne par note : un bouton de suppression, puis le texte de la note
      ...notes.map((note, i) =>
        Box({
          flexDirection: 'row',
          columnGap: 1,
          children: [
            Button({
              // Une key propre, pour distinguer le bouton de chaque ligne
              key: 'delete-' + i,
              label: 'x',
              plain: true,
              onPress: async () => {
                notes = notes.filter((_, j) => j !== i)
                redraw()
                await $.store.set('notes', notes)
              },
            }),
            Text({ children: [note] }),
          ],
        }),
      ),
    ],
  })
})
```

Pour essayer le panneau :

* **Ajouter une note** : tapez une ligne et appuyez sur Entrée. La ligne apparaît comme une nouvelle rangée, et le champ se vide.
* **Supprimer une note** : appuyez sur Tab jusqu'à ce que le bouton `x` de la note ait le focus, puis appuyez sur Entrée. Le `x` est le libellé du bouton et non un raccourci : taper la lettre ne le presse pas.

Chaque changement suit le même cycle de rendu que `hello-tabs` : le callback modifie `notes`, appelle `redraw`, et enregistre la liste dans `$.store`.

Le champ se vide après chaque soumission grâce à sa prop `value`. `value` est le texte que contient le champ au moment où il est dessiné, et la saisie de l'utilisateur le remplace jusqu'à ce que votre hook redessine le champ. L'exemple dessine toujours le champ avec `''`.

L'exemple enregistre les notes mais ne les recharge pas. Pour les retrouver à la session suivante, lisez-les dans un hook `session.start`, comme `hello-tabs` lit `count`.

Ces props composent la ligne du champ, `Note: Type a note and press Enter ⏎ add` :

| Prop | Dans l'exemple | Ce que c'est |
| :- | :- | :- |
| `label` | `Note` | Le texte avant le champ. Le terminal dessine `: ` après lui. |
| `placeholder` | `Type a note and press Enter` | Texte grisé affiché tant que le champ est vide |
| `submitLabel` | `add` | Le mot après `⏎` qui indique ce que fait Entrée |

Soumettre un `Input` ne démarre pas de tour, sauf si votre callback appelle [`$.prompt.submit`](https://code.claude.com/docs/en/plugins/mods/api#start-a-turn-from-a-background-job).

## Redessiner un site

Un dessin est un instantané : il montre ce que votre hook `ui.render` a renvoyé lors de sa dernière exécution. Pour afficher quelque chose de nouveau, le hook doit s'exécuter de nouveau. Claude Code le réexécute pour certains changements, et votre mod le demande pour les autres.

### Quand Claude Code redessine sans qu'on le lui demande

Claude Code réexécute votre hook `ui.render` quand les props du site changent ou quand la largeur du terminal change. Quand un `Client` du site échoue et que votre mod gère [`ui.fault`](https://code.claude.com/docs/en/plugins/mods/reference#interface), Claude Code exécute le hook une fois de plus après le retour de vos hooks `ui.fault`, pour que votre hook `ui.render` puisse omettre le `Client`. Il n'exécute pas le hook sur une minuterie, et il ne peut pas savoir quand une variable de votre module change.

### Redessiner quand vos données changent

Pour que vos sites soient redessinés après un changement de vos propres données, appelez `$.ui.invalidate('ui.render')`. Ce panneau compte les appuis. Le callback du bouton modifie `count`, puis demande un redessin :

```javascript theme={null}
let count = 0

on('ui.render', { component: 'Pane' }, async ($, e, next) => {
  if (e.requestId !== 'counter') return next(e)
  const { Box, Text, Button } = $.ui.resolve(e)
  return Box({
    flexDirection: 'row',
    columnGap: 2,
    children: [
      Button({
        key: 'more',
        label: 'Add one',
        onPress: () => {
          count += 1
          // Les données ont changé : on demande à Claude Code de redessiner le panneau
          $.ui.invalidate('ui.render')
        },
      }),
      Text({ children: ['Count: ' + count] }),
    ],
  })
})
```

Chaque appui augmente le nombre affiché dans le panneau. L'[exemple `hello-tabs`](#construire-un-panneau-avec-des-onglets) enveloppe le même appel dans sa fonction `redraw`.

Une valeur conservée dans [`$.state`](#conserver-une-valeur-dans-state) n'a pas besoin de cet appel, car écrire la valeur redessine les sites qui la lisent.

### Redessiner sur une minuterie

Pour maintenir à jour une horloge, un compte à rebours ou une valeur venant de l'extérieur de la session, redessinez selon un calendrier. Démarrez une minuterie dans le hook `session.start` du module. Si le module en a déjà un, comme `hello-tabs`, ajoutez-y la ligne [`$.clock.every`](https://code.claude.com/docs/en/plugins/mods/api#run-work-in-the-background) :

```javascript theme={null}
on('session.start', async ($, e, next) => {
  // Toutes les 1000 millisecondes, demande à Claude Code de redessiner vos sites
  $.clock.every(1000, () => $.ui.invalidate('ui.render'))
  return next(e)
})
```

Claude Code exécute désormais votre hook `ui.render` une fois par seconde. La minuterie s'arrête quand le module est rechargé, et la nouvelle instance du module démarre la sienne.

### Fréquence maximale de redessin d'un site

Claude Code limite (throttle) les redessins d'un site : votre mod peut donc appeler `$.ui.invalidate` aussi souvent que ses données changent. Pour la fréquence de redessin de chaque site, voir le [tableau des limites](https://code.claude.com/docs/en/plugins/mods/reference#limits).

Les appels plus rapides que la limite sont fusionnés en un seul redessin. Ce redessin exécute votre hook une fois, et le hook lit vos données telles qu'elles sont à cet instant : la dernière valeur s'affiche, pas les valeurs intermédiaires. Une animation ne peut pas aller plus vite que la limite.

## Conserver l'état

L'endroit où un mod conserve une valeur détermine combien de temps elle dure : jusqu'au rechargement du module, jusqu'à la fin de la session, ou d'une session à l'autre. Choisissez selon la durée de vie nécessaire :

| La conserver dans | Elle dure jusqu'à | À utiliser pour |
| :- | :- | :- |
| Une variable de niveau module | Le rechargement du module, qui a lieu à chaque enregistrement d'un fichier pendant le développement | Des valeurs que vous pouvez perdre, comme `tab` dans `hello-tabs` |
| `$.state` | La fin de la session, ou l'exécution par l'utilisateur de `/clear`, `/resume` ou `/branch` | Des valeurs dont dépend un dessin et qui doivent survivre à un rechargement |
| `$.store` | Sa suppression par votre mod, ou l'absence de toute lecture/écriture du magasin par une session pendant [`cleanupPeriodDays`](https://code.claude.com/docs/en/settings-reference#cleanupperioddays). Le magasin est un magasin clé-valeur, enregistré dans un fichier JSON propre à votre plugin sous `~/.claude/plugins/store/`. | Réglages, historique, tout ce que l'utilisateur s'attend à retrouver la fois suivante |

`$.store.get(key)` se résout avec la valeur ou `undefined`, et `$.store.set(key, value)` accepte n'importe quelle valeur JSON.

### Conserver une valeur dans `$.state`

`$.state` conserve des valeurs pendant toute la session, et redessine pour vous. C'est un état réactif : un hook `ui.render` qui lit une valeur s'y abonne, de sorte que Claude Code redessine ce site chaque fois que vous écrivez la valeur, sans que vous appeliez `$.ui.invalidate`. Une valeur dans `$.state` survit aussi à un rechargement du module, contrairement à une variable.

Pour le mettre en place, déclarez vos valeurs, faites pointer votre manifeste vers la déclaration, puis définissez et utilisez chaque valeur. Les exemples déplacent le `count` de `hello-tabs` dans `$.state`.

#### Déclarer les valeurs

Déclarez les valeurs dans un fichier de déclaration de types. La clé extérieure est le nom de votre plugin, et chaque entrée en dessous est une valeur et son type. Enregistrez ceci sous `hello-tabs/types/index.d.ts` :

```typescript hello-tabs/types/index.d.ts theme={null}
declare module 'claude-code' {
  interface PluginState {
    'hello-tabs': {
      tab: 'one' | 'two'
      count: number
    }
  }
}
```

#### Faire pointer le manifeste vers la déclaration

Pour que `claude plugin validate` vérifie votre code par rapport à ce fichier, ajoutez au manifeste un champ `types` contenant son chemin :

```json hello-tabs/.claude-plugin/plugin.json theme={null}
{
  "name": "hello-tabs",
  "version": "0.1.0",
  "description": "Opens a pane with two tabs and a counter",
  "author": { "name": "Your Name" },
  "types": "./types/index.d.ts"
}
```

#### Définir, lire et écrire une valeur

Dans votre module, définissez chaque valeur avec une valeur par défaut, lisez-la pendant le dessin, et écrivez-la depuis un callback. `atom` nomme une valeur et sa valeur par défaut, `read` la renvoie, et `update` l'écrit. Ces trois helpers appellent `$.state.get` et `$.state.set` pour vous :

```javascript theme={null}
import { atom, read, update } from 'claude-code'

// En haut du module : nomme la valeur et donne sa valeur par défaut
const count = atom({ plugin: 'hello-tabs', key: 'count' }, 0)

// Dans le hook ui.render : lit la valeur pour la dessiner
const n = await read($, count)

// Dans un Button : écrit une nouvelle valeur à partir de l'ancienne
onPress: () => update($, count, (value) => value + 1)
```

Comme le hook `ui.render` a lu `count`, Claude Code réexécute le hook chaque fois que le bouton l'écrit.

Ces règles s'appliquent au code :

* **Écrivez `plugin` et `key` sous forme de littéraux de chaîne** : `claude plugin validate` les lit dans votre source
* **Conservez le résultat de chaque appel à `atom` dans une `const`** : si vous déclarez `count` avec `let`, la validation échoue avec `takes a source the scan can read`
* **Déclarez chaque valeur dans le fichier de déclaration de types** : sinon la validation échoue avec `hello-tabs.count is not declared`
* **Écrivez depuis un callback ou depuis le hook d'un autre événement** : un hook `ui.render` peut lire l'état mais pas l'écrire ; écrivez donc depuis `onPress`, `onSubmit` ou un hook d'un autre événement

#### Modifier `hello-tabs` pour utiliser `$.state`

Pour déplacer `count` de `hello-tabs` dans `$.state`, modifiez chaque ligne qui l'utilise :

* **En haut du module** : ajoutez la ligne `import`, et remplacez `let count = 0` par la ligne `atom`
* **Dans le hook `ui.render`** : ajoutez la ligne `read` avant `tabButton`, et dessinez `'Count: ' + n` dans le `Text`
* **Dans le bouton Add one** : remplacez `onPress` par celui de [Enregistrer depuis plusieurs sessions](#enregistrer-depuis-plusieurs-sessions), qui enregistre le compte en plus de l'écrire
* **Dans le hook `session.start`** : remplacez les deux lignes qui lisent `saved` par l'appel à `loadCount` de [Recharger une valeur enregistrée après `/clear`](#recharger-une-valeur-enregistrée-après-clear)

Gardez `redraw` pour les boutons d'onglet, car `tab` reste une variable.

### Recharger une valeur enregistrée après `/clear`

Si votre mod copie une valeur enregistrée de `$.store` vers `$.state` au `session.start`, il doit la recopier après `/clear`, `/resume` ou `/branch`. Ces commandes remettent chaque valeur de `$.state` à sa valeur par défaut, et `session.start` ne se déclenche pas à nouveau. [`classic.SessionStart`](https://code.claude.com/docs/en/plugins/mods/events#hook-the-settings-hook-events), lui, se déclenche après chacune d'elles, avec `e.source` valant `clear`, `resume` ou `fork` : recopiez donc la valeur dans un hook sur cet événement. Sinon, votre dessin affiche la valeur par défaut, et un callback qui enregistre la valeur de `$.state` écrit cette valeur par défaut par-dessus ce que vous aviez stocké.

Ce code charge `count` depuis les deux hooks. Il s'appuie sur la version `$.state` de `hello-tabs`, où `count` est un atom et `update` est importé. Placez `loadCount` au-dessus de `register`, et ajoutez l'appel à `loadCount` au hook `session.start` que vous avez déjà. `classic.SessionStart` se déclenche aussi au démarrage et après une compaction, qui ne réinitialise pas `$.state` : le filtre sur `source` limite donc le hook aux trois réinitialisations :

```javascript theme={null}
// Copie le compte enregistré de $.store vers $.state, ou 0 si rien n'est enregistré
async function loadCount($) {
  const saved = Number((await $.store.get('count')) ?? 0)
  await update($, count, () => saved)
}

// S'exécute avant votre premier prompt, et de nouveau après un rechargement
on('session.start', async ($, e, next) => {
  await loadCount($)
  return next(e)
})

// S'exécute de nouveau après /clear, /resume et /branch (qui rapporte fork)
on('classic.SessionStart', { source: ['clear', 'resume', 'fork'] }, async ($, e, next) => {
  await loadCount($)
  return next(e)
})
```

Avec ces deux hooks en place, le panneau affiche le compte enregistré après `/clear`, et non `0`, et le prochain appui sur **Add one** s'ajoute au compte enregistré.

`loadCount` écrit la valeur stockée par-dessus celle de `$.state`, et `session.start` se déclenche de nouveau à chaque rechargement du module. Pour que le magasin ne prenne pas de retard, enregistrez à chaque changement, comme le fait le bouton **Add one**.

Pour vérifier le rechargement sans session, [testez le dessin après `/clear`](https://code.claude.com/docs/en/plugins/mods/test#test-a-drawing-after-clear).

### Enregistrer depuis plusieurs sessions

Toutes les sessions de votre machine qui exécutent votre mod partagent un même `$.store`. Un `get` suivi d'un `set` n'est pas atomique. Quand deux sessions lisent chacune une valeur, la modifient et la réécrivent, elles entrent en concurrence (race), et la seconde écriture remplace la première.

Pour rendre cela moins probable :

* **Donnez à chaque élément sa propre clé** : un `set` ne modifie que sa propre clé, donc des sessions qui écrivent des clés différentes ne s'écrasent pas
* **Relisez juste avant d'écrire** : pour une valeur que plusieurs sessions modifient, faites le `get` de la clé dans le callback et construisez la nouvelle valeur à partir de celui-ci, et non d'une copie chargée au `session.start`. L'écriture d'une autre session reste perdue si elle survient entre votre `get` et votre `set`.

Ce bouton ajoute un à ce que contient le magasin à cet instant, puis met à jour le dessin :

```javascript theme={null}
onPress: async () => {
  // Lit ce que contient le magasin maintenant, qu'une autre session a pu modifier
  const saved = Number((await $.store.get('count')) ?? 0)
  // Enregistre le nouveau compte, puis l'affiche
  await $.store.set('count', saved + 1)
  await update($, count, () => saved + 1)
}
```

Si une seconde session a pressé son propre bouton trois fois depuis le démarrage de celle-ci, cet appui affiche et enregistre un compte qui inclut ces trois appuis.

## Étapes suivantes

* [Réagir aux événements](https://code.claude.com/docs/en/plugins/mods/events) : alimenter votre dessin à partir des appels d'outil et des tours
* [Utiliser l'API des mods](https://code.claude.com/docs/en/plugins/mods/api) : alimenter votre dessin à partir de minuteries et d'appels de modèle
* [Tester un dessin](https://code.claude.com/docs/en/plugins/mods/test#test-a-drawing) : presser vos boutons depuis un test, sur plusieurs surfaces
* [Sites de rendu](https://code.claude.com/docs/en/plugins/mods/reference#render-sites) et [éléments](https://code.claude.com/docs/en/plugins/mods/reference#elements) : les props de chaque site et de chaque élément
