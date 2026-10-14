# Créer une marketplace

> Source : https://code.claude.com/docs/en/plugins/create-marketplace · [← Index](../31-index-documentation-officielle.md)

*Construire une marketplace de plugins à partir d'un fichier `marketplace.json` et la tester en local avant de l'héberger.*

Une marketplace de plugins est un répertoire ou un dépôt contenant un fichier `.claude-plugin/marketplace.json` qui liste vos plugins et indique où récupérer chacun d'eux. Vous poussez le répertoire sur un hébergeur git, et toute personne qui y a accès l'enregistre dans Claude Code en une commande, puis installe vos plugins depuis celle-ci.

Créez votre propre marketplace quand vous voulez qu'un groupe de votre choix, comme votre équipe ou votre organisation, installe vos plugins et continue de recevoir vos mises à jour depuis un catalogue que vous contrôlez. Le dépôt peut être **privé**, il peut lister autant de plugins que vous le souhaitez, et un administrateur peut [l'imposer sur chaque machine](https://code.claude.com/docs/en/plugins/org).

> **Note** : ces cas sont traités sur d'autres pages :
>
> - **Partager un plugin avec quelques personnes** : envoyez-leur le répertoire du plugin ou un `.zip` de celui-ci. Voir [Partager un plugin sans marketplace](https://code.claude.com/docs/en/plugins/publish#share-a-plugin-without-a-marketplace).
> - **Proposer un plugin à tout le monde** : soumettez-le à l'annuaire d'Anthropic. Voir [Soumettre à l'annuaire d'Anthropic](https://code.claude.com/docs/en/plugins/publish#submit-to-anthropics-directory).
> - **Utiliser un plugin vous-même** : chargez-le avec `--plugin-dir` ou enregistrez-le dans votre répertoire de skills. Voir [Développer sans marketplace](https://code.claude.com/docs/en/plugins/create#develop-without-a-marketplace).

Commencez par [Créer une marketplace](#créer-une-marketplace-pas-à-pas) pour en construire une sur votre machine et y installer un plugin, puis [ajoutez d'autres entrées de plugin](#ajouter-des-entrées-de-plugin).

## Créer une marketplace (pas à pas)

*(ancre d'origine : `#create-a-marketplace`)*

Les étapes suivantes créent une marketplace sur votre machine, y ajoutent un plugin, l'enregistrent dans Claude Code et installent le plugin depuis celle-ci. C'est la boucle complète, et c'est la même que suivront vos utilisateurs une fois la marketplace hébergée à un endroit qu'ils peuvent atteindre. Exécutez chaque commande dans votre shell, depuis le répertoire où vous voulez créer `my-marketplace/`.

Il vous faut un plugin à lister. L'exemple utilise `my-first-plugin` de [Créer votre premier plugin](https://code.claude.com/docs/en/plugins/create#create-your-first-plugin), un plugin avec un seul skill que vous lancez avec `/my-first-plugin:hello` ; construisez-le d'abord si vous n'avez pas encore de plugin. Pour utiliser un de vos plugins à la place, substituez son répertoire et son `name` partout où les étapes indiquent `my-first-plugin`. Pour ce qu'un répertoire de plugin peut contenir, voir l'[explorateur du répertoire de plugin](https://code.claude.com/docs/en/plugins/components#explore-the-plugin-directory).

### Étape 1 — Préparer le répertoire de la marketplace

Une marketplace est un répertoire contenant un fichier `.claude-plugin/marketplace.json`, plus les plugins qu'il liste. Créez le répertoire de la marketplace et son dossier `.claude-plugin/`, puis copiez votre plugin sous `plugins/` :

```bash
mkdir -p my-marketplace/.claude-plugin my-marketplace/plugins
cp -r my-first-plugin my-marketplace/plugins/
```

Vérifiez que le plugin est valide à son nouvel emplacement, pour que toute erreur ultérieure concerne la marketplace et non le plugin :

```bash
claude plugin validate ./my-marketplace/plugins/my-first-plugin
```

La dernière ligne de la sortie indique `✔ Validation passed`.

### Étape 2 — Créer le fichier de marketplace

Enregistrez `marketplace.json` sous `my-marketplace/.claude-plugin/marketplace.json`. Le fichier exige un `name`, un `owner` et un tableau `plugins`.

Chaque objet de `plugins` est une **entrée de plugin** et nécessite un `name` et une `source`. Écrivez la `source` de l'entrée comme un chemin depuis la racine de la marketplace. La racine est `my-marketplace/`, le répertoire qui contient `.claude-plugin/`.

```json my-marketplace/.claude-plugin/marketplace.json
{
  "name": "my-marketplace",
  "description": "Plugins for my team",
  "owner": {
    "name": "Your Name"
  },
  "plugins": [
    {
      "name": "my-first-plugin",
      "source": "./plugins/my-first-plugin",
      "description": "A greeting plugin to learn the basics"
    }
  ]
}
```

### Étape 3 — Valider la marketplace

Lancez `claude plugin validate` sur le répertoire de la marketplace pour vérifier la syntaxe JSON, les champs obligatoires et chaque entrée de plugin de son `.claude-plugin/marketplace.json`.

```bash
claude plugin validate ./my-marketplace
```

Pour le fichier tel qu'écrit à l'étape 2, la dernière ligne de la sortie indique `✔ Validation passed`.

### Étape 4 — Ajouter la marketplace et installer le plugin

Enregistrez le répertoire comme marketplace.

```bash
claude plugin marketplace add ./my-marketplace
```

La commande affiche `✔ Successfully added marketplace: my-marketplace (declared in user settings)`, ce qui signifie que la marketplace est enregistrée dans votre fichier de réglages utilisateur.

Installez le plugin. L'identifiant d'installation est le `name` de l'entrée, un `@`, puis le `name` de la marketplace.

```bash
claude plugin install my-first-plugin@my-marketplace
```

La commande affiche `✔ Successfully installed plugin: my-first-plugin@my-marketplace (scope: user)`.

Dans une session, `/plugin marketplace add ./my-marketplace` enregistre la marketplace de la même façon. `/plugin install my-first-plugin@my-marketplace` ouvre les détails du plugin dans le panneau `/plugin`, où vous l'installez. Pour ce parcours, voir [Installer et gérer des plugins](https://code.claude.com/docs/en/plugins/install).

### Étape 5 — Confirmer que le plugin est chargé

Listez les plugins installés.

```bash
claude plugin list
```

La sortie liste `my-first-plugin@my-marketplace` avec `Status: ✔ enabled`.

Pour voir ce que le plugin a chargé, affichez ses détails.

```bash
claude plugin details my-first-plugin
```

La section `Component inventory` indique `Skills (1)  hello`.

Pour lancer le skill, démarrez une session et saisissez `/my-first-plugin:hello`. Claude vous salue. La commande porte le nom du plugin en préfixe, comme le nom de tout skill de plugin.

## Ajouter des entrées de plugin

Chaque plugin que vous distribuez est un objet du tableau `plugins` de `marketplace.json`. Pour ajouter un deuxième plugin, ajoutez un deuxième objet. Ces champs couvrent la plupart des entrées :

- **`name`** : l'identifiant que les gens tapent avant `@` lors de l'installation. [Entrées de plugin](https://code.claude.com/docs/en/plugins/marketplace-reference#plugin-entries) indique les caractères autorisés dans un nom.
- **`source`** : l'endroit d'où Claude Code récupère le plugin. Écrivez une chaîne de chemin relatif pour un plugin situé dans le répertoire de la marketplace, comme dans [le parcours](#créer-une-marketplace-pas-à-pas), ou un objet source pour un plugin situé ailleurs. Voir [Choisir une source de plugin](#choisir-une-source-de-plugin).
- **`description`** : la ligne que les gens voient à côté du plugin quand ils parcourent votre marketplace dans `/plugin`.

Pour la liste complète des champs, voir [Entrées de plugin](https://code.claude.com/docs/en/plugins/marketplace-reference#plugin-entries), qui couvre aussi les champs de [`plugin.json`](https://code.claude.com/docs/en/plugins/manifest-reference) qu'une entrée peut définir et quand ils s'appliquent.

## Règles pour les entrées de plugin

La plupart des installations qui échouent depuis une nouvelle marketplace viennent d'un chemin relatif écrit depuis le mauvais répertoire, ou d'un nom d'entrée différent du `name` figurant dans le `plugin.json` du plugin.

### Écrire les chemins relatifs depuis la racine de la marketplace

La racine de la marketplace est le répertoire qui contient `.claude-plugin/`. Dans [le parcours](#créer-une-marketplace-pas-à-pas), c'est `my-marketplace/` ; la `source` de l'entrée est donc `"./plugins/my-first-plugin"`. Le chemin ne part pas de l'intérieur de `.claude-plugin/` : n'utilisez donc pas `..` pour en sortir.

Un chemin contenant `..` et un chemin vers un répertoire manquant échouent à des commandes différentes :

- **Un chemin avec `..`** : `claude plugin validate` signale l'entrée comme invalide. Le message commence par `Path contains "..": ./../plugins/my-first-plugin`.
- **Un chemin vers un répertoire inexistant** : `claude plugin validate` **réussit**. C'est `claude plugin install` qui échoue avec `Source path does not exist: <path>`, où `<path>` est l'emplacement absolu vérifié par Claude Code.

### Garder le nom d'entrée et le nom du manifeste identiques

Un plugin de marketplace possède un `name` d'entrée dans `marketplace.json` et un `name` dans son propre `plugin.json`, appelé **nom du manifeste**. Chaque nom apparaît à des endroits différents :

- **Nom d'entrée** : l'identifiant d'installation, `<entry-name>@<marketplace>`. C'est ce que les gens tapent pour installer, ce qu'affiche `claude plugin list`, et la clé que Claude Code écrit sous [`enabledPlugins`](https://code.claude.com/docs/en/settings-reference#enabledplugins) dans leur fichier de réglages.
- **Nom du manifeste** : le préfixe des skills du plugin, et le nom qu'attend `claude plugin details`.

Quand les deux noms diffèrent et que quelqu'un installe par le nom du manifeste, Claude Code signale `Plugin "<manifest-name>" not found in marketplace "<marketplace>"`. **Gardez les deux noms identiques.** Pour en savoir plus sur l'usage des deux noms par Claude Code, voir [Référence du chargement des plugins](https://code.claude.com/docs/en/plugins/loading#find-where-a-plugin-came-from).

## Choisir une source de plugin

Chaque entrée de plugin de `marketplace.json` possède une `source` qui indique à Claude Code où récupérer ce plugin précis. Choisissez la source selon l'endroit où sont stockés les fichiers du plugin. Le tableau liste les sources les plus utilisées par les propriétaires de marketplaces.

| Source | À utiliser quand | Valeur `source` minimale |
| :- | :- | :- |
| Chemin relatif | Les fichiers du plugin se trouvent dans le répertoire même de la marketplace | `"./plugins/my-first-plugin"` |
| `github` | Le plugin est un dépôt GitHub à part entière | `{ "source": "github", "repo": "your-org/my-first-plugin" }` |
| `git-subdir` | Le plugin est un sous-répertoire d'un autre dépôt, par exemple un monorepo | `{ "source": "git-subdir", "url": "your-org/monorepo", "path": "tools/my-first-plugin" }` |

Dans une source `git-subdir`, `url` accepte une URL git ou un raccourci GitHub `owner/repo`.

Un plugin peut aussi provenir de l'un de ces types de source :

- **`url`** : un dépôt git désigné par URL, sur n'importe quel hébergeur ;
- **`archive`** : un fichier zip téléchargé en HTTPS ;
- **`npm`** : un paquet npm ;
- **`command`** : un répertoire produit en exécutant une commande sur la machine où le plugin est installé.

Pour les champs de chaque type de source, et pour épingler une source git sur un `ref` ou un `sha`, voir [Sources de plugin](https://code.claude.com/docs/en/plugins/marketplace-reference#plugin-sources).

## Valider et tester

Au fur et à mesure que vous ajoutez des plugins, lancez `claude plugin validate ./my-marketplace` dans votre shell après chaque modification, et installez depuis la marketplace sur votre propre machine avant de la partager. La validation et l'installation détectent des problèmes différents.

### Problèmes signalés par la validation

`claude plugin validate` ne lit que les fichiers situés dans le répertoire de la marketplace. Il signale :

- les erreurs de syntaxe JSON, sous la forme `json: Invalid JSON syntax: <reason>` ;
- les champs obligatoires manquants, par exemple `owner: Invalid input` ;
- un nom de marketplace ou de plugin qui enfreint les règles de nommage de la [référence des marketplaces](https://code.claude.com/docs/en/plugins/marketplace-reference#top-level-fields) ;
- une `source` relative contenant `..` ;
- les champs inconnus au premier niveau ou dans une entrée de plugin, sous forme d'**avertissements** ;
- les problèmes dans le `plugin.json` de chaque plugin à chemin relatif, sous la forme `plugins[N] plugin.json → <field>: <message>`.

Pour chaque message que `validate` peut afficher, voir [Messages de validation](https://code.claude.com/docs/en/plugins/marketplace-reference#validation-messages). Pour ses flags et codes de sortie, voir [`plugin validate`](https://code.claude.com/docs/en/plugins/cli-reference#plugin-validate).

### Problèmes qui apparaissent à l'ajout ou à l'installation

Les problèmes que `claude plugin validate` ne signale pas apparaissent quand vous ajoutez la marketplace ou installez depuis celle-ci :

- **À l'ajout de la marketplace** : les [noms exacts des marketplaces officielles](https://code.claude.com/docs/en/plugins/marketplace-reference#reserved-names), comme `claude-plugins-official`, passent la validation. Mais quand vous ajoutez une marketplace portant l'un de ces noms, Claude Code la refuse avec un message qui commence par `The name '<name>' is reserved for official Anthropic marketplaces`.
- **À l'installation d'un plugin** :
  - Claude Code ne récupère une source `github`, `git-subdir` ou autre source distante qu'au moment de l'installation du plugin ; un `repo` ou un `path` erroné apparaît donc à ce moment-là.
  - Une `source` relative dont le répertoire n'existe pas échoue aussi à l'installation, avec `Source path does not exist: <path>`.

### Tester une modification d'un plugin

Dans [le parcours](#créer-une-marketplace-pas-à-pas), vous avez ajouté `my-marketplace` depuis un répertoire local avec une `source` à chemin relatif. Avec cette configuration, Claude Code lit les fichiers du plugin directement depuis `my-marketplace/plugins/`. Vos modifications prennent effet au prochain démarrage de session ou quand vous lancez `/reload-plugins` dans une session, **sans changer la `version` du plugin**.

Les personnes qui installent depuis votre marketplace hébergée reçoivent en revanche une copie dans le cache des plugins. Pour savoir comment elles reçoivent une nouvelle version, voir [Tenir les utilisateurs à jour](https://code.claude.com/docs/en/plugins/host-marketplace#keep-users-up-to-date).

### Supprimer la marketplace pour repartir de zéro

Pour tout supprimer et recommencer, lancez `claude plugin marketplace remove my-marketplace` dans votre shell. La commande supprime la marketplace **et désinstalle ses plugins**.

## Héberger votre marketplace

Une fois que vous pouvez installer un plugin depuis la marketplace sur votre propre machine, comme dans [le parcours](#créer-une-marketplace-pas-à-pas), poussez le répertoire de la marketplace sur un hébergeur git.

Vos coéquipiers lancent alors `claude plugin marketplace add <owner>/<repo>` dans leur shell pour un dépôt GitHub, ou la même commande avec l'URL du dépôt. Ils installent ensuite un plugin par son nom comme dans [le parcours](#créer-une-marketplace-pas-à-pas).

Pour l'accès aux dépôts privés, les mises à jour, le versionnage, et le renommage ou la suppression d'entrées, voir [Héberger et maintenir une marketplace](https://code.claude.com/docs/en/plugins/host-marketplace).

## Étapes suivantes

- [Héberger et maintenir une marketplace](https://code.claude.com/docs/en/plugins/host-marketplace) : choisir un hébergeur, tenir les utilisateurs à jour, et renommer ou retirer des plugins en toute sécurité.
- [Référence des marketplaces](https://code.claude.com/docs/en/plugins/marketplace-reference) : champs de `marketplace.json` et types de source.
- [Gérer les plugins pour votre organisation](https://code.claude.com/docs/en/plugins/org) : imposer votre marketplace et ses plugins sur chaque machine.
- [Suggérer des plugins selon leur pertinence](https://code.claude.com/docs/en/plugins/relevance) : faire en sorte que Claude Code suggère un plugin de votre marketplace quand une session s'y prête.
