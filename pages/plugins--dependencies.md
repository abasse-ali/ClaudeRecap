# Dépendances de plugins

> Source : https://code.claude.com/docs/en/plugins/dependencies · [← Index](../31-index-documentation-officielle.md)

*Déclarer les plugins dont votre plugin dépend, avec des plages de versions comme `^1.2`, et voir comment Claude Code les installe, les résout et les élague.*

Une dépendance de plugin est un autre plugin dont votre plugin a besoin, par exemple un plugin dont il appelle le serveur MCP ou un skill. Chaque dépendance suit la **dernière version** que fournit sa marketplace, sauf si vous déclarez une **contrainte de version** : une plage de versions sémantiques comme `^2.0` ou `~2.1.0` que vous avez testée.

Cette page s'adresse aux auteurs de plugins qui déclarent des dépendances dans `plugin.json` et aux mainteneurs de marketplaces qui étiquettent (tag) les versions publiées.

> **Note** : ces cas sont traités sur d'autres pages :
>
> - **Installer un plugin qui a des dépendances** : voir [Gérer les plugins installés](https://code.claude.com/docs/en/plugins/install#manage-installed-plugins).
> - **Lire une erreur de dépendance** : voir [Erreurs de dépendance](https://code.claude.com/docs/en/plugins/troubleshooting#dependency-errors).
> - **Déclarer les paquets npm et Bun dont le code de votre plugin a besoin** : voir [Dépendances de paquets Node.js](https://code.claude.com/docs/en/plugins/loading#node-js-package-dependencies).

Pour ajouter une contrainte, commencez par [Déclarer une dépendance avec une contrainte de version](#déclarer-une-dépendance-avec-une-contrainte-de-version). Si vous maintenez un plugin dont d'autres dépendent, [étiquetez vos versions](#publier-un-plugin-dont-dautres-dépendent) pour que leurs contraintes puissent se résoudre.

## Déclarer des dépendances

*(ancre secondaire d'origine : `#decide-whether-to-constrain-dependency-versions`)*

Sans contrainte de version, une dépendance passe à chaque nouvelle version publiée par sa marketplace lors de la prochaine mise à jour des utilisateurs. Si cette version renomme un outil MCP qu'appelle votre plugin, votre plugin casse pour tous ceux qui mettent à jour.

Avec une contrainte comme `~2.1.0` sur une dépendance provenant d'une source adossée à git, les utilisateurs qui ont installé votre plugin continuent de recevoir les correctifs `2.1.x` de la dépendance et ne passent **jamais** à `2.2`. Pour mettre à niveau à votre rythme, testez avec une version plus récente, puis publiez une nouvelle version de votre plugin avec une contrainte plus large.

### Déclarer une dépendance avec une contrainte de version

Listez les dépendances dans le tableau `dependencies` du fichier `.claude-plugin/plugin.json` de votre plugin. Le manifeste suivant déclare une dépendance sans version et une dépendance contrainte :

```json .claude-plugin/plugin.json
{
  "name": "deploy-kit",
  "version": "3.1.0",
  "dependencies": [
    "audit-logger",
    { "name": "secrets-vault", "version": "~2.1.0" }
  ]
}
```

Une entrée peut être une **chaîne** : le nom du plugin seul, comme `"audit-logger"` dans ce manifeste, ou `"name@marketplace"` pour le résoudre dans une autre marketplace. Avec une simple chaîne, votre plugin dépend de la version, quelle qu'elle soit, que fournit la marketplace de ce plugin.

Pour définir une contrainte de version, utilisez un **objet** avec ces champs, chacun de type chaîne :

| Champ | Description |
| :- | :- |
| `name` | Le nom de plugin de la dépendance, tel qu'il apparaît dans son entrée de marketplace. Claude Code le recherche dans la même marketplace que le plugin déclarant, sauf si vous définissez `marketplace`. Obligatoire. |
| `version` | Une [plage de versions sémantiques](https://github.com/npm/node-semver#ranges) comme `~2.1.0`, `^2.0`, `>=1.4` ou `=2.1.0`. La dépendance s'installe au tag git le plus élevé qui satisfait cette plage ; le mainteneur de la dépendance doit donc [étiqueter ses versions](#publier-un-plugin-dont-dautres-dépendent). |
| `marketplace` | Une autre marketplace dans laquelle résoudre `name`. Une liste d'autorisation contrôle les dépendances inter-marketplaces, décrite dans [Dépendre d'un plugin d'une autre marketplace](#dépendre-dun-plugin-dune-autre-marketplace). |

Une plage ne correspond pas aux versions de pré-publication comme `2.0.0-beta.1`, sauf si vous l'activez avec un suffixe de pré-publication comme `^2.0.0-0`.

### Regrouper des plugins pour une équipe

Pour permettre aux ingénieurs d'installer un ensemble choisi de plugins en une seule commande, publiez un plugin dont le manifeste contient un `name` et un tableau `dependencies`. Un manifeste de plugin n'exige que `name` : c'est donc un plugin valide, et l'installer installe toutes ses dépendances.

Par exemple, une équipe plateforme peut publier dans une marketplace interne des lots (bundles) par rôle, pour que les ingénieurs lancent un seul `claude plugin install` au lieu d'installer chaque plugin séparément :

```json .claude-plugin/plugin.json
{
  "name": "backend-standard",
  "version": "1.0.0",
  "description": "Standard plugin set for backend engineers",
  "dependencies": [
    "secrets-vault",
    "deploy-kit",
    { "name": "db-migrate", "version": "^3.0" },
    "oncall-runbook"
  ]
}
```

Pour ajouter plus tard un plugin à l'ensemble standard, publiez une nouvelle version de `backend-standard` avec la dépendance supplémentaire. Quand la marketplace ne se [met pas à jour automatiquement par défaut](https://code.claude.com/docs/en/plugins/loading#which-marketplaces-and-plugins-auto-update), les ingénieurs activent la mise à jour automatique de la marketplace ou mettent à jour manuellement :

- **Activer la mise à jour automatique de la marketplace** : la prochaine mise à jour automatique fait passer le lot à la nouvelle version et installe toutes les dépendances qu'il ajoute.
- **Mettre à jour manuellement** : lancer `claude plugin update backend-standard` dans un shell, puis `/reload-plugins` dans une session ouverte pour installer les dépendances nouvellement ajoutées.

Pour les étapes côté ingénieur, voir [Garder les plugins à jour](https://code.claude.com/docs/en/plugins/install#keep-plugins-updated).

Pour déployer un lot auprès de toute une organisation, un administrateur l'ajoute à `enabledPlugins` dans les réglages gérés (managed settings). Voir [Préinstaller et imposer des plugins](https://code.claude.com/docs/en/plugins/org#pre-install-and-require-plugins).

### Dépendre d'un plugin d'une autre marketplace

Par défaut, Claude Code **n'installe pas** une dépendance provenant d'une marketplace différente de celle du plugin déclarant, sauf si l'utilisateur a déjà cette dépendance installée et activée au même scope. Ce comportement par défaut empêche une marketplace d'installer silencieusement des plugins depuis une source que l'utilisateur n'a pas examinée.

Pour autoriser l'installation, ajoutez le nom de la marketplace cible à `allowCrossMarketplaceDependenciesOn` dans le `marketplace.json` de la **marketplace racine**. La marketplace racine est celle qui héberge le plugin que l'utilisateur installe. Seule la liste d'autorisation de la marketplace racine s'applique.

Le `marketplace.json` suivant autorise `deploy-kit` à dépendre d'un plugin de `your-shared-marketplace` :

```json .claude-plugin/marketplace.json
{
  "name": "your-marketplace",
  "owner": { "name": "Your Org" },
  "allowCrossMarketplaceDependenciesOn": ["your-shared-marketplace"],
  "plugins": [
    {
      "name": "deploy-kit",
      "source": "./deploy-kit",
      "dependencies": [
        { "name": "audit-logger", "marketplace": "your-shared-marketplace" }
      ]
    }
  ]
}
```

Si `allowCrossMarketplaceDependenciesOn` est absent ou n'inclut pas la marketplace cible, Claude Code n'installe pas la dépendance :

- **Dépendance déclarée dans l'entrée de marketplace** : l'installation elle-même est refusée avec un message qui commence par `Dependency "audit-logger@your-shared-marketplace" (required by deploy-kit@your-marketplace) is in marketplace "your-shared-marketplace", which is not in the allowlist` et qui nomme le champ à définir.
- **Dépendance déclarée dans `plugin.json`** : l'installation se termine sans la dépendance, et votre plugin **échoue ensuite à se charger**.

La vérification de la liste d'autorisation ne s'applique pas à une dépendance déjà activée. Si un utilisateur installe d'abord lui-même `audit-logger` depuis `your-shared-marketplace`, au même scope, `deploy-kit` s'installe ensuite sans aucune modification de la liste d'autorisation.

### Tester un plugin et sa dépendance en local

Si vous développez en même temps un plugin et le plugin dont il dépend, démarrez Claude Code depuis votre shell et chargez les deux avec [`--plugin-dir`](https://code.claude.com/docs/en/plugins/cli-reference#flags-that-load-a-plugin-for-one-session) :

```bash
claude --plugin-dir ./my-dependency --plugin-dir ./my-plugin
```

La copie locale de la dépendance satisfait l'entrée de dépendance de votre plugin ; vous n'avez donc pas besoin d'installer la dépendance depuis sa marketplace.

- **Pas besoin de `version`** : le `plugin.json` local n'a pas besoin de `version` non plus, car une [contrainte de version](#déclarer-une-dépendance-avec-une-contrainte-de-version) n'est pas vérifiée face à une copie locale.
- **Entrées qui nomment une marketplace** : une entrée qui nomme une marketplace correspond aussi à la copie locale (v2.1.242+).

Tant que vous n'avez pas installé la dépendance depuis sa marketplace, votre plugin cesse de se charger chaque fois que la copie locale est désactivée ou absente :

- **Vous avez désactivé la copie locale** : votre plugin est désactivé au prochain chargement des plugins, avec une erreur qui se termine par `is disabled — enable it or remove the dependency`. Quand l'erreur nomme la dépendance `<name>@inline`, cet identifiant désigne la copie chargée via `--plugin-dir`.
- **Vous avez démarré une session sans le flag `--plugin-dir` de la dépendance** : l'erreur indique que la dépendance n'est pas installée. Repassez le flag, ou installez la dépendance depuis sa marketplace.

Quand les deux plugins se trouvent dans un même dossier parent, vous pouvez passer ce dossier une seule fois à `--plugin-dir`. Si le dossier n'est pas lui-même un plugin, Claude Code charge chaque dossier enfant qui contient un `.claude-plugin/plugin.json` (v2.1.265+).

## Publier un plugin dont d'autres dépendent

*(ancre d'origine : `#tag-plugin-releases-for-version-resolution`)*

Si vous maintenez un plugin dont d'autres plugins dépendent avec une contrainte de version, étiquetez ses versions pour que ces contraintes puissent se résoudre. Une contrainte se résout d'après les **tags git** du dépôt qui héberge le plugin. Étiquetez le dépôt vers lequel pointe la [source de plugin](https://code.claude.com/docs/en/plugins/marketplace-reference#plugin-sources) dans `marketplace.json` :

- **Source `github`, `url` ou `git-subdir`** : le dépôt propre du plugin ; c'est donc l'auteur du plugin qui crée les tags.
- **Chemin relatif comme `./plugins/secrets-vault`** : le dépôt de la marketplace ; c'est donc le mainteneur de la marketplace qui crée les tags.

### Créer un tag de version

Étiquetez chaque version sous la forme `<plugin-name>--v<version>`, où `<version>` correspond au champ `version` du `plugin.json` de ce commit. Le préfixe du nom de plugin permet à un même dépôt de marketplace d'héberger plusieurs plugins avec des historiques de versions indépendants.

Créez le tag depuis le répertoire du plugin, avec un remote `origin` configuré pour recevoir le tag poussé, à l'aide de [`claude plugin tag`](https://code.claude.com/docs/en/plugins/cli-reference#plugin-tag) :

```bash
claude plugin tag --push
```

La commande construit le nom du tag à partir du manifeste du plugin. Avant de créer le tag, elle effectue ces vérifications :

- valide le plugin ;
- vérifie que `plugin.json` et l'entrée de marketplace s'accordent sur la version, lorsque le répertoire du plugin se trouve dans une copie (checkout) de marketplace ;
- exige un arbre de travail propre sous le répertoire du plugin ;
- refuse si le tag existe déjà.

Une exécution réussie affiche `Created tag secrets-vault--v2.1.0`. Avec `--push`, elle affiche aussi `Pushed to origin`. Sans `--push`, elle affiche la commande `git push` à lancer vous-même.

Passez `--dry-run` pour voir le plan sans rien créer.

La [référence de `claude plugin tag`](https://code.claude.com/docs/en/plugins/cli-reference#plugin-tag) liste les autres flags.

Vous pouvez aussi lancer directement `git tag secrets-vault--v2.1.0`, à condition de garder vous-même synchronisées la `version` de `plugin.json` et celle de l'entrée de marketplace.

### Contraindre une dépendance dont la source n'est pas git

La résolution par tags ne s'applique qu'aux sources adossées à git. Pour une dépendance avec une [source de plugin](https://code.claude.com/docs/en/plugins/marketplace-reference#plugin-sources) `npm`, `archive` ou `command`, la contrainte ne détermine pas quelle version est récupérée. Elle est tout de même vérifiée au chargement du plugin, et le plugin dépendant est désactivé si la version installée ne la satisfait pas.

Pour les sources `npm`, `archive` et `command`, la version vérifiée est la `version` du `plugin.json` de la dépendance. Définissez-en une avant de contraindre cette dépendance, car un `plugin.json` sans version **ne satisfait aucune contrainte**.

Claude Code n'installe jamais lui-même une dépendance ayant une source `command` : les utilisateurs doivent donc [l'installer d'abord](https://code.claude.com/docs/en/plugins/marketplace-reference#command-plugin-source). Il n'exécute jamais non plus le [`headersHelper`](https://code.claude.com/docs/en/plugins/host-marketplace#authenticate-archive-downloads) d'une dépendance : les utilisateurs installent donc aussi, avant votre plugin, toute dépendance dont l'entrée de marketplace en définit un.

Outre `claude plugin install`, ces opérations installent aussi toute dépendance déclarée manquante, et les limites `command` et `headersHelper` s'y appliquent également :

- `/reload-plugins`
- la mise à jour automatique de la marketplace du plugin dépendant
- la réexécution de `claude plugin install` sur le plugin dépendant
- `claude plugin marketplace add`

## Comportement des dépendances chez vos utilisateurs

Ces sections décrivent comment Claude Code résout, vérifie et combine les contraintes que vous déclarez une fois votre plugin installé à côté d'autres.

### Résolution d'une contrainte d'après les tags

Quand un utilisateur installe un plugin qui déclare `{ "name": "secrets-vault", "version": "~2.1.0" }`, la dépendance s'installe depuis le tag `secrets-vault--v` le plus élevé qui satisfait `~2.1.0` sur le dépôt qui héberge `secrets-vault`. Quand aucun tag ne satisfait la plage, l'installation échoue ou utilise la copie actuelle de la marketplace :

- **Plugin ayant son propre dépôt** : l'installation échoue avec un message contenant `Dependency "secrets-vault@your-marketplace" has no git tag satisfying`.
- **Plugin référencé par un chemin relatif** : l'installation utilise à la place la copie actuelle de la marketplace, et la contrainte est vérifiée au chargement du plugin. Si cette copie est hors de la plage, le plugin dépendant reste désactivé et `claude plugin list` affiche `Requires "secrets-vault@your-marketplace" ~2.1.0, installed 3.0.0`.

Pour un plugin que la marketplace référence par un chemin relatif, une marketplace ajoutée sous forme de chemin de dossier local résout aussi les contraintes d'après les tags git de ce dossier, lorsque le dossier est un dépôt git. Un dossier local qui n'est pas un dépôt git n'a pas de tags : Claude Code installe alors la dépendance depuis le contenu actuel du dossier.

### Confirmer la version résolue

Pour confirmer la version vers laquelle une contrainte s'est résolue, lancez `claude plugin list` dans votre shell. Une dépendance résolue par tag affiche sa version avec un suffixe de commit de 12 caractères, par exemple `2.1.0-8713c5b11005`.

Les vérifications de contrainte utilisent la version du **tag** plutôt que la `version` de `plugin.json`, même si `plugin.json` est en retard à ce commit.

Si vous déplacez de force un tag vers un autre commit, l'installation suivante récupère le contenu de ce commit au lieu de réutiliser une copie en cache périmée. Voir [Versions et mises à jour](https://code.claude.com/docs/en/plugins/loading#versions-and-updates) pour la façon dont la version d'un plugin devient sa clé de cache.

### Combiner les contraintes de plusieurs plugins

Quand plusieurs plugins installés contraignent la même dépendance, celle-ci se résout vers la version la plus élevée qui satisfait toutes leurs plages. Les combinaisons courantes se résolvent ainsi :

| Le plugin A exige | Le plugin B exige | Résultat |
| :- | :- | :- |
| `^2.0` | `>=2.1` | Une seule installation, au tag `2.x` le plus élevé à partir de `2.1.0`. Les deux plugins se chargent. |
| `~2.1` | `~3.0` | L'installation du plugin B échoue avec un message `has conflicting version requirements`. Le plugin A et la dépendance restent inchangés. |
| `=2.1.0` | aucune | La dépendance reste en `2.1.0`. La mise à jour automatique ignore les versions plus récentes tant que le plugin A est installé. |

La mise à jour automatique récupère une dépendance contrainte au tag git le plus élevé qui satisfait la plage de **chaque** plugin installé, plutôt qu'à la dernière version de la marketplace :

- Si les plages des plugins installés ne se chevauchent pas, la mise à jour automatique laisse cette dépendance à sa version actuelle, et l'onglet **Errors** de `/plugin` affiche une entrée nommant le plugin qui contraint.
- Si elles se chevauchent mais qu'aucun tag ne tombe dans la plage, la mise à jour automatique récupère la copie actuelle de la marketplace et saute la mise à jour lorsque la `version` de cette copie sort de la plage d'un des plugins installés.

Quand un utilisateur désinstalle le dernier plugin qui contraint une dépendance, celle-ci n'est plus contrainte à une plage de versions et recommence à suivre son entrée de marketplace à la prochaine mise à jour.

### Dépendances quand votre plugin se charge depuis claude.ai

Quand un utilisateur active votre plugin pour son compte claude.ai, Claude Code le charge sous le nom `<name>@synced`, [sans marketplace](https://code.claude.com/docs/en/plugins/loading#synced-plugins). Pour cette copie de votre plugin, Claude Code considère qu'une entrée de dépendance qui nomme une marketplace est satisfaite quand l'utilisateur dispose de l'un ou l'autre :

- **La dépendance depuis sa marketplace** : installée depuis la marketplace nommée par votre entrée, et activée. Claude Code vérifie une plage `version` de votre entrée face à la version de la copie installée.
- **Un plugin claude.ai du même nom** : activé depuis son compte claude.ai. Claude Code vérifie une plage `version` de votre entrée face à la `version` de ce plugin, et un plugin qui n'indique aucune `version` passe la vérification.

Si vous dites aux utilisateurs d'activer votre plugin depuis claude.ai, dites-leur d'y activer aussi la dépendance, ou de l'installer depuis sa marketplace.

Un plugin claude.ai ne satisfait l'entrée que lorsque votre plugin se charge lui aussi depuis claude.ai.

## Voir aussi

- [`claude plugin prune`](https://code.claude.com/docs/en/plugins/cli-reference#plugin-prune) : supprimer les dépendances installées automatiquement dont plus aucun plugin n'a besoin.
- [Héberger une marketplace](https://code.claude.com/docs/en/plugins/host-marketplace) : canaux de publication et recommandation d'autres plugins.
