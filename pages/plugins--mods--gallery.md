# Galerie d'interface pour les mods

> Source : https://code.claude.com/docs/en/plugins/mods/gallery · [← Index](../31-index-documentation-officielle.md)

*Voir les éléments d'interface qu'un mod Claude Code peut dessiner, comme du texte, des boutons, des champs, du Markdown, du code et des diffs, avec du code d'exemple et des captures d'écran du terminal.*

Un mod dessine son interface à partir d'éléments : du texte, des boîtes, des boutons, des champs, et quelques éléments qui mettent en forme le contenu pour vous. Les exemples de cette page montrent le code qui dessine un élément, et la plupart sont accompagnés d'une capture du résultat dans un panneau du terminal, pour que vous puissiez choisir un élément d'après son apparence (les captures sont décrites ici en texte).

Pour apprendre comment fonctionne le dessin, commencez par [Dessiner dans l'interface](https://code.claude.com/docs/en/plugins/mods/interface). Pour les props principales et les applications qui dessinent chaque élément, voir la [référence des éléments](https://code.claude.com/docs/en/plugins/mods/reference#elements). Les [déclarations de types](https://code.claude.com/docs/en/plugins/mods/create#get-the-types-for-your-build) listent toutes les props.

## Essayer un exemple

Les exemples de cette page sont des extraits, pas des mods complets. Chacun est le code d'un élément et de tout ce qui y est imbriqué.

Pour voir un exemple dans votre propre terminal, créez le petit mod décrit dans ces étapes et collez-y l'exemple. Le mod ajoute une commande `/gallery` qui ouvre un panneau et y dessine l'exemple. Un [panneau](https://code.claude.com/docs/en/plugins/mods/interface#pick-where-to-draw) est une barre latérale à côté de la transcription dans un terminal plein écran large, ou sinon une zone encadrée au-dessus du prompt.

#### Étape 1 — Créer le mod

Créez un répertoire nommé `gallery` contenant les répertoires `.claude-plugin` et `hooks`. [Créer un mod](https://code.claude.com/docs/en/plugins/mods/create#write-a-mod-yourself) explique les fichiers.

Enregistrez le manifeste sous `gallery/.claude-plugin/plugin.json` :

```json gallery/.claude-plugin/plugin.json theme={null}
{
  "name": "gallery",
  "version": "0.1.0",
  "description": "Opens a pane that draws one sample",
  "author": { "name": "Your Name" }
}
```

Nommez votre point d'entrée dans `gallery/hooks/hooks.json` :

```json gallery/hooks/hooks.json theme={null}
{
  "modules": ["./register.js"]
}
```

Enregistrez le code sous `gallery/hooks/register.js`. Il ajoute une commande `/gallery` qui ouvre un panneau, et dessine `Plain text` dans ce panneau :

```javascript gallery/hooks/register.js theme={null}
// Remplace votre propre callback dans les exemples qui en prennent un
const noop = () => {}
// L'exemple Select garde son choix ici
let picked = 'md'

// L'exemple Raster empaquette ses cellules avec cette fonction
const DEFAULT_COLOR = 0x01000000
function cellsOf(rows) {
  const numbers = rows.flat().flatMap(([char, color]) => [char.codePointAt(0), color, DEFAULT_COLOR])
  return new Uint8Array(Uint32Array.from(numbers).buffer).toBase64()
}

export function register(on) {
  on('session.start', async ($, e, next) => {
    await $.command.register({ name: 'gallery', description: 'Open the sample pane' })
    return next(e)
  })

  on('command.run', { command: 'gallery' }, async ($) => {
    await $.ui.open({ id: 'gallery', focus: true, closeOnEscape: true })
    return {}
  })

  on('ui.render', { component: 'Pane' }, async ($, e, next) => {
    if (e.requestId !== 'gallery') return next(e)
    const { Box, Text, Button, Input, Select, Link, Markdown, Code, Raster, Svg } = $.ui.resolve(e)
    // Remplacez l'élément après return par un exemple
    return Text({ children: ['Plain text'] })
  })
}
```

#### Étape 2 — Lancer le mod

Dans votre shell, démarrez Claude Code depuis le répertoire qui contient `gallery` :

```bash theme={null}
claude --plugin-dir ./gallery
```

Dans le prompt de Claude Code, lancez `/gallery`. Un panneau s'ouvre avec `Plain text` dedans.

#### Étape 3 — Substituer un exemple

Copiez un exemple de cette page. Dans `register.js`, collez-le à la place de `Text({ children: ['Plain text'] })`, de sorte qu'il suive `return`, et enregistrez le fichier. Claude Code recharge le module à chaque enregistrement, donc relancez `/gallery` pour voir le nouvel exemple.

## Choisir un élément

Les exemples sont regroupés selon ce que vous voulez afficher à l'écran :

* **[Afficher du texte](#afficher-du-texte)** : `Text`, `Markdown` et `Link`
* **[Afficher du code et des modifications](#afficher-du-code-et-des-modifications)** : `Code`
* **[Disposer des éléments](#disposer-des-éléments)** : `Box`
* **[Recueillir une saisie](#recueillir-une-saisie)** : `Button`, `Input` et `Select`
* **[Dessiner des images](#dessiner-des-images)** : `Raster`, `Svg`, `Image` et `Client`

## Afficher du texte

Trois éléments affichent des mots à l'écran : `Text` pour votre propre mise en forme, `Markdown` pour du contenu déjà formaté, et `Link` pour une URL.

### `Text`

`Text` dessine une chaîne avec les styles que vous lui donnez. Cet exemple affiche une ligne par style :

```javascript theme={null}
Box({
  flexDirection: 'column',
  children: [
    Text({ children: ['Plain text'] }),
    Text({ bold: true, children: ['bold'] }),
    Text({ italic: true, children: ['italic'] }),
    Text({ underline: true, children: ['underline'] }),
    Text({ strikethrough: true, children: ['strikethrough'] }),
    Text({ dimColor: true, children: ['dimColor'] }),
    Text({ inverse: true, children: ['inverse'] }),
    Text({ color: 'red', children: ["color: 'red'"] }),
    Text({ backgroundColor: 'blue', children: ["backgroundColor: 'blue'"] }),
  ],
})
```

*Capture : un panneau avec neuf lignes de texte, chacune portant le nom de son style : normal, gras (bold), italique (italic), souligné (underline), barré (strikethrough), dimColor en gris, inversé (inverse), couleur rouge, et fond bleu (backgroundColor).*

`dimColor` dessine le texte en gris. `backgroundColor` ne remplit que la largeur du texte.

### `Markdown`

`Markdown` met en forme le texte comme sont mises en forme les réponses de Claude. Passez le contenu dans `text`, pas dans `children` :

```javascript theme={null}
Markdown({
  text: '## Release notes\n\nThis build has **two** fixes and one `flag`:\n\n- Faster start\n- Fewer prompts\n\n> Quoted text',
})
```

*Capture : un panneau avec un titre en gras, Release notes, puis une phrase avec un mot en gras et un mot de code en couleur, une liste de deux éléments, et une citation dessinée en italique avec une barre à sa gauche.*

Un titre est dessiné en gras sans ses marques `#`. Le code en ligne est dessiné en couleur sans ses backticks. Une citation est dessinée en italique avec une barre à sa gauche.

### `Link`

`Link` dessine un libellé suivi de son URL :

```javascript theme={null}
Link({ href: 'https://code.claude.com/docs', label: 'Claude Code docs' })
```

*Capture : un panneau avec une ligne : le libellé Claude Code docs, puis l'URL en gris.*

Le terminal dessine l'URL sous forme de texte après le libellé. Qu'un clic l'ouvre ou non dépend du terminal de l'utilisateur.

## Afficher du code et des modifications

`Code` dessine du texte source avec les couleurs syntaxiques propres à Claude Code, ou un diff.

### `Code`

Nommez le `language`, ou passez un `path` pour que Claude Code le déduise. Avec `startLine`, les lignes sont numérotées à partir de ce nombre :

```javascript theme={null}
Code({
  language: 'javascript',
  startLine: 1,
  source: "const name = 'mods'\nconsole.log('hello ' + name)",
})
```

*Capture : un panneau avec deux lignes numérotées de JavaScript en couleurs syntaxiques.*

Les couleurs viennent du thème de l'utilisateur.

### `Code` en diff

Avec `format: 'diff'`, `source` est un ou plusieurs hunks de diff unifié :

```javascript theme={null}
Code({
  format: 'diff',
  source: '@@ -1,3 +1,3 @@\n # Mods\n-A mod is a plugin.\n+A mod is a plugin that runs code.\n Read on.',
})
```

*Capture : un panneau avec un diff de quatre lignes. La ligne supprimée est ombrée en rouge et la ligne ajoutée en vert, chacune avec son numéro de ligne. Dans la ligne ajoutée, les mots « that runs code » ont une teinte plus marquée.*

Claude Code dessine des numéros de ligne à la place de la ligne `@@`. Là où une ligne supprimée et une ligne ajoutée se ressemblent, les mots qui ont changé reçoivent une teinte plus marquée.

## Disposer des éléments

### `Box`

`Box` dispose ce qu'il contient en ligne ou en colonne, et peut dessiner une bordure. Cet exemple place une ligne de mots au-dessus d'une boîte bordée :

```javascript theme={null}
Box({
  flexDirection: 'column',
  gap: 1,
  children: [
    Box({
      flexDirection: 'row',
      columnGap: 4,
      children: [Text({ children: ['a row'] }), Text({ children: ['of three'] }), Text({ children: ['items'] })],
    }),
    Box({
      borderStyle: 'round',
      paddingX: 1,
      children: [Text({ children: ["borderStyle: 'round'"] })],
    }),
  ],
})
```

*Capture : un panneau avec trois mots sur une ligne, espacés de quatre colonnes, puis une ligne vide, puis une bordure arrondie autour d'une ligne de texte. La bordure occupe toute la largeur du panneau.*

La bordure s'étire sur la largeur du panneau.

## Recueillir une saisie

`Button`, `Input` et `Select` sont des contrôles : l'utilisateur passe de l'un à l'autre avec Tab et utilise celui qui a le focus. [Focus clavier et raccourcis](https://code.claude.com/docs/en/plugins/mods/interface#know-which-keys-your-mod-can-receive) indique quelles touches les atteignent.

Ouvrir un panneau avec `focus: true` donne le focus clavier au panneau. Les lettres tapées atteignent un `Input` une fois qu'il a le focus, donc ajoutez `autoFocus: true` à un champ qui doit recevoir la saisie dès l'ouverture du panneau.

### `Button`

Un bouton exécute `onPress`. Cet exemple montre la forme par défaut, un bouton `plain` avec un raccourci (`hotkey`), et un bouton atténué :

```javascript theme={null}
Box({
  flexDirection: 'column',
  children: [
    Button({ key: 'save', label: 'Save', onPress: noop }),
    Button({ key: 'next', label: 'Next', hotkey: 'n', plain: true, onPress: noop }),
    Button({ key: 'skip', label: 'Skip', dimColor: true, onPress: noop }),
  ],
})
```

*Capture : un panneau avec trois boutons, un par ligne : Save entre crochets, n: Next sans crochets avec le n en couleur, et Skip entre crochets en gris.*

Un bouton qui a le focus est dessiné en vidéo inverse. Ici, l'utilisateur a appuyé deux fois sur Tab :

*Capture : les trois mêmes boutons, avec le deuxième, n: Next, dessiné en vidéo inverse.*

### `Input`

Un `Input` est un champ de texte d'une ligne qui exécute `onSubmit` quand l'utilisateur appuie sur Entrée :

```javascript theme={null}
Input({
  key: 'title',
  label: 'Title',
  placeholder: 'Type a title and press Enter',
  value: '',
  submitLabel: 'save',
  onSubmit: noop,
})
```

Sans le focus, le champ affiche son libellé et son texte indicatif (placeholder) :

*Capture : un panneau avec une ligne : le libellé Title, puis le placeholder « Type a title and press Enter » en gris.*

Avec le focus, le libellé passe en gras, un curseur apparaît, et le `submitLabel` s'affiche après `⏎` :

*Capture : le même champ avec son libellé en gras, un curseur bloc sur la première lettre du placeholder, et un signe retour suivi du mot save.*

La saisie remplace le placeholder :

*Capture : le même champ contenant les lettres tapées « Rel », suivies du signe retour et du mot save.*

### `Select`

Un `Select` permet à l'utilisateur de choisir une option parmi plusieurs, et exécute `onSelect` avec la `value` de l'option :

```javascript theme={null}
Select({
  key: 'format',
  label: 'Format',
  value: picked,
  options: [
    { value: 'md', label: 'Markdown' },
    { value: 'html', label: 'HTML' },
    { value: 'txt', label: 'Plain text' },
  ],
  onSelect: (value) => {
    picked = value
  },
})
```

Fermé, il affiche son libellé et l'option courante :

*Capture : un panneau avec une ligne : le libellé Format, l'option courante Markdown, et une petite flèche vers le bas.*

Ouvert, il liste ses options et en marque une :

*Capture : le sélecteur ouvert, avec ses trois options listées sous le libellé. La deuxième option, HTML, est dessinée en vidéo inverse.*

Après que l'utilisateur a choisi une option, la liste se ferme :

*Capture : le sélecteur de nouveau fermé, affichant désormais HTML comme option courante.*

## Dessiner des images

### `Raster`

Un `Raster` est une grille de cellules de caractères colorées, pour une carte de chaleur, une sparkline ou un plateau de jeu. C'est le terminal qui le dessine. Cet exemple utilise la fonction `cellsOf` du module de départ, qui empaquette les cellules dans la chaîne qu'attend un `Raster`. [Dessiner une grille de cellules colorées](https://code.claude.com/docs/en/plugins/mods/interface#draw-a-grid-of-colored-cells) l'explique :

```javascript theme={null}
Raster({
  key: 'grid',
  columns: 3,
  rows: 2,
  cells: cellsOf([
    [['█', 0x2e7d32], ['█', 0xf9a825], ['█', 0xc62828]],
    [['█', 0x2e7d32], ['█', 0x2e7d32], ['█', 0xf9a825]],
  ]),
})
```

*Capture : un panneau avec une petite grille de blocs colorés, deux lignes de trois : vert, ambre et rouge, puis vert, vert et ambre.*

Un `Raster` arrondit chaque couleur vers une palette plus réduite, donc `0x2e7d32` est dessiné comme `#337733`.

### `Svg`

Un `Svg` dessine un document SVG dans l'application Desktop :

```javascript theme={null}
Svg({
  alt: 'Three bars of rising height',
  width: 120,
  height: 60,
  source:
    '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 120 60"><rect x="10" y="40" width="20" height="20" fill="#2e7d32"/><rect x="50" y="25" width="20" height="35" fill="#f9a825"/><rect x="90" y="5" width="20" height="55" fill="#c62828"/></svg>',
})
```

Dans le terminal, un panneau qui ne renvoie qu'un `Svg` s'ouvre vide. Pour y dessiner autre chose, vérifiez [`e.surface`](https://code.claude.com/docs/en/plugins/mods/interface#pick-where-to-draw) et renvoyez un arbre différent.

### `Image` et `Client`

Deux autres éléments n'ont pas d'exemple ici. `Image` dessine un PNG ou des pixels bruts dans le terminal. `Client` est une zone que dessine un second fichier à vous, pour l'animation et la saisie au pointeur. La [référence des éléments](https://code.claude.com/docs/en/plugins/mods/reference#elements) liste leurs props.

Sauf si Claude Code détecte que le terminal dessine les images du protocole graphique kitty avec des espaces réservés Unicode (Unicode placeholders), l'utilisateur voit le texte `alt` d'une `Image`, atténué, à la place de l'image. Rédigez un texte `alt` qui se suffit à lui-même. La détection s'exécute au démarrage : elle réussit dans kitty 0.28 ou ultérieur et dans Ghostty, une fois que le terminal a répondu à la requête graphique de Claude Code, et échoue dans ces cas :

* **Autres terminaux** : tout terminal qui n'est pas l'un de ces deux, ou qui ne répond pas à la requête.
* **tmux et screen** : une session qui tourne dans tmux ou screen, quel que soit le terminal, kitty et Ghostty compris.
* **Sessions en arrière-plan** : toute [session en arrière-plan](https://code.claude.com/docs/en/agent-view), quel que soit le terminal depuis lequel elle est attachée.

Si les utilisateurs de votre mod voient le texte atténué dans un terminal qui dessine bien ces images à espaces réservés, ils peuvent régler [`CLAUDE_CODE_FORCE_TERMINAL_IMAGES`](https://code.claude.com/docs/en/env-vars) sur `1`, ce qui saute la détection. Dans tmux ou screen, cela n'aide pas : le texte `alt` disparaît, et Claude Code envoie l'image sans l'encapsuler pour le passthrough de tmux ou screen.

## Voir où un mod peut dessiner

Les exemples dessinent tous dans un panneau. Un mod peut aussi dessiner à d'autres endroits, et appeler Claude Code pour afficher quelque chose à sa place :

* **Panneau (pane) et bande (band)** : [Choisir où dessiner](https://code.claude.com/docs/en/plugins/mods/interface#pick-where-to-draw)
* **Les lignes propres à Claude Code, comme le spinner** : [Modifier ce que Claude Code dessine déjà](https://code.claude.com/docs/en/plugins/mods/interface#change-what-claude-code-already-draws)
* **Toast, ligne de statut et ligne de journal** : [Afficher quelque chose sans démarrer de tour](https://code.claude.com/docs/en/plugins/mods/api#show-something-without-starting-a-turn)
* **Boîte de dialogue de question** : [Suspendre un appel d'outil jusqu'à la décision de l'utilisateur](https://code.claude.com/docs/en/plugins/mods/events#hold-a-tool-call-until-the-user-decides)

## Étapes suivantes

* [Dessiner dans l'interface](https://code.claude.com/docs/en/plugins/mods/interface) : construire pas à pas un panneau avec des onglets
* [Tester un dessin](https://code.claude.com/docs/en/plugins/mods/test#test-a-drawing) : appuyer sur vos boutons depuis un test
* [Référence des éléments](https://code.claude.com/docs/en/plugins/mods/reference#elements) : les props principales de chaque élément et les applications qui le dessinent
