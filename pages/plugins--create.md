# Créer un plugin Claude Code

> Source : https://code.claude.com/docs/en/plugins/create · [← Index](../31-index-documentation-officielle.md)

*Construire votre premier plugin Claude Code à partir d'un répertoire vide, le tester sans marketplace, et convertir une configuration `.claude/` existante.*

Un plugin est un répertoire de skills, d'agents, de hooks et de serveurs MCP, accompagné d'un fichier `plugin.json`, appelé **manifeste**, qui nomme le plugin. Claude Code charge le répertoire comme une seule unité : vous pouvez donc le partager avec vos coéquipiers, l'installer dans plusieurs projets ou le publier dans une marketplace.

Cette page s'adresse aux personnes qui écrivent leurs propres plugins.

> **Note** : ces cas sont traités sur d'autres pages :
>
> - **Installer le plugin de quelqu'un d'autre** : voir [Installer des plugins](https://code.claude.com/docs/en/plugins/install).
> - **Pas sûr d'avoir besoin d'un plugin** : voir [Décider si vous avez besoin d'un plugin](https://code.claude.com/docs/en/plugins/overview#decide-whether-you-need-a-plugin) dans la vue d'ensemble.
> - **Les utilisateurs de votre plugin sont sur claude.ai ou dans Cowork** : le même dossier s'y installe, avec un sous-ensemble différent de composants. Voir [Plugin structure and testing](https://claude.com/docs/plugins/build) et le [tableau de prise en charge des composants](https://claude.com/docs/plugins/platform-support#compare-component-support-by-app).

Commencez par la section qui correspond à ce que vous avez déjà :

- **Rien encore** : suivez [Créer votre premier plugin](#créer-votre-premier-plugin), puis [Développer sans marketplace](#développer-sans-marketplace) et [Tester et déboguer](#tester-et-déboguer).
- **Des fichiers déjà présents sous `.claude/`** : faites une fois le parcours du premier plugin pour apprendre la disposition, puis suivez [Convertir une configuration `.claude/` existante](#convertir-une-configuration-claude-existante).

## Décider quand utiliser un plugin

Les skills, agents, hooks et serveurs MCP fonctionnent tous de manière autonome dans votre projet ou votre répertoire personnel. Gardez cette configuration autonome tant qu'elle ne sert qu'un projet ou que vous seul. Créez un plugin quand vous voulez partager la configuration avec vos coéquipiers, l'installer dans plusieurs projets ou publier des versions numérotées.

Quand vous déplacez des skills, agents, hooks et configurations MCP autonomes dans un plugin, leur emplacement et leurs noms changent :

- **Où vont les fichiers** : sous le répertoire propre du plugin, appelé **racine du plugin**, sous la forme `skills/`, `agents/`, `hooks/hooks.json` et `.mcp.json`.
- **Comment ils sont nommés** : les skills et agents du plugin reçoivent le nom du plugin en préfixe, par exemple `/my-plugin:hello`, ce qui permet à deux plugins de fournir chacun un skill `hello` sans collision.

Pour déplacer une configuration existante dans un plugin, voir [Convertir une configuration `.claude/` existante](#convertir-une-configuration-claude-existante).

## Créer votre premier plugin

Dans ce parcours, vous créez un plugin dont le seul composant est un skill de salutation, et vous le lancez avec `--plugin-dir`, qui charge un plugin pour une session sans l'installer. Un plugin peut contenir n'importe quelle combinaison de [composants](https://code.claude.com/docs/en/plugins/components) (skills, agents, hooks, serveurs MCP…) et **aucun n'est obligatoire** ; un seul skill est le plus petit exemple qui montre la disposition.

Prérequis : Claude Code [installé et connecté](https://code.claude.com/docs/en/quickstart#step-1-install-claude-code).

Ouvrez un terminal dans le répertoire où vous voulez garder le plugin, par exemple `~/projects`, et exécutez-y les commandes de ces étapes. Un plugin peut se trouver n'importe où, car vous passez son chemin à Claude Code au démarrage d'une session.

### Étape 1 — Créer le répertoire du plugin

Créez le répertoire du plugin, avec à l'intérieur un dossier `.claude-plugin/` pour contenir le manifeste :

```bash
mkdir -p my-first-plugin/.claude-plugin
```

### Étape 2 — Écrire le manifeste

Le [manifeste](https://code.claude.com/docs/en/plugins/manifest-reference) est un fichier JSON nommé `plugin.json` qui indique à Claude Code le nom du plugin et le décrit. Enregistrez celui-ci sous `my-first-plugin/.claude-plugin/plugin.json` :

```json my-first-plugin/.claude-plugin/plugin.json
{
  "name": "my-first-plugin",
  "description": "A greeting plugin to learn the basics",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  }
}
```

Les quatre champs servent à ceci :

- **`name`** : obligatoire. Il identifie le plugin et devient le préfixe de chaque skill et agent fourni par le plugin. N'y mettez pas d'espaces.
- **`description`** : le texte que les utilisateurs voient pour le plugin dans `/plugin`.
- **`version`** : facultatif. Le définir maintient les utilisateurs sur cette version jusqu'à ce que vous la changiez ; [Publier une nouvelle version](https://code.claude.com/docs/en/plugins/host-marketplace#release-a-new-version) explique quand le définir ou l'omettre.
- **`author`** : à qui attribuer le plugin. `name` y est obligatoire ; `email` et `url` sont facultatifs.

Tous les autres champs figurent dans la [référence du manifeste](https://code.claude.com/docs/en/plugins/manifest-reference#fields).

Seul `plugin.json` va dans `.claude-plugin/`. Le skill que vous ajoutez ensuite se place directement sous `my-first-plugin/`, à côté de ce dossier.

### Étape 3 — Ajouter un skill

Le seul composant de ce plugin est un skill. Chaque skill est un répertoire sous `skills/` qui contient un fichier `SKILL.md`. Créez le répertoire du skill :

```bash
mkdir -p my-first-plugin/skills/hello
```

Puis créez `my-first-plugin/skills/hello/SKILL.md` avec ce contenu :

```markdown my-first-plugin/skills/hello/SKILL.md
---
name: hello
description: Greet the user with a friendly message
disable-model-invocation: true
---

Greet the user warmly and ask how you can help them today.
```

La ligne `disable-model-invocation: true` signifie que Claude ne lance pas le skill de lui-même. Retirez cette ligne d'un skill que vous voulez que Claude lance de sa propre initiative. La commande du skill combine le nom du plugin et le nom du skill : vous lancez donc celui-ci avec `/my-first-plugin:hello`. Pour les autres champs de frontmatter, voir la [référence du frontmatter des skills](https://code.claude.com/docs/en/skills#frontmatter-reference).

### Étape 4 — Valider le plugin

Vérifiez le manifeste et le frontmatter du skill avant de lancer quoi que ce soit :

```bash
claude plugin validate ./my-first-plugin
```

La commande affiche le chemin du manifeste vérifié et `✔ Validation passed`. Si elle affiche plutôt `✘ Validation failed`, chaque ligne au-dessus de ce résultat indique le champ à corriger. Recherchez chaque message dans [`claude plugin validate` signale des erreurs](https://code.claude.com/docs/en/plugins/troubleshooting#claude-plugin-validate-reports-errors).

### Étape 5 — Lancer Claude Code avec le plugin

Démarrez une session avec le plugin chargé :

```bash
claude --plugin-dir ./my-first-plugin
```

Une fois Claude Code démarré, lancez le skill :

```text
/my-first-plugin:hello
```

Claude répond par une salutation.

---

Le plugin ne se charge que dans les sessions démarrées avec `--plugin-dir`. Pour continuer à travailler dessus sans ce flag, ou pour tester une version `.zip`, voir [Développer sans marketplace](#développer-sans-marketplace).

Pour que Claude échafaude et vérifie avec vous un plugin plus conséquent, [installez](https://code.claude.com/docs/en/plugins/install#install-a-plugin) le plugin `plugin-dev` d'Anthropic depuis la marketplace `claude-plugins-official`. Il ajoute des skills et des agents pour écrire des composants (skills, hooks, serveurs MCP…) et pour valider le plugin terminé. Une fois installé, lancez `/plugin-dev:create-plugin` suivi d'une description du plugin voulu : Claude vous guide pour le concevoir, le créer et le valider.

### Partager votre plugin

*(ancre d'origine : `#share-the-plugin`)*

Un plugin construit avec [Créer votre premier plugin](#créer-votre-premier-plugin) n'existe que sur votre machine. Quand il est prêt pour d'autres personnes, il y a trois façons de le leur transmettre :

- **L'envoyer directement à quelques personnes** : donnez-leur le répertoire du plugin ou un `.zip` de celui-ci ; rien n'a besoin d'être publié. Voir [Partager un plugin sans marketplace](https://code.claude.com/docs/en/plugins/publish#share-a-plugin-without-a-marketplace).
- **Le lister dans votre propre marketplace** : vos coéquipiers ajoutent une fois votre marketplace, installent le plugin par son nom et reçoivent vos mises à jour. Voir [Publier via votre propre marketplace](https://code.claude.com/docs/en/plugins/publish#publish-through-your-own-marketplace).
- **Le soumettre à l'annuaire d'Anthropic** : après validation de la revue, les gens peuvent l'ajouter sur claude.ai et dans Cowork, et il atteint Claude Code via leur compte. Voir [Soumettre à l'annuaire d'Anthropic](https://code.claude.com/docs/en/plugins/publish#submit-to-anthropics-directory).

### Disposition d'un plugin

Chaque type de [composant](https://code.claude.com/docs/en/plugins/components) (skills, agents, hooks, serveurs MCP…) va dans un répertoire fixe sous la racine du plugin, c'est-à-dire le répertoire que vous passez à `--plugin-dir`. N'ajoutez que les répertoires que vous utilisez. Pour parcourir un répertoire de plugin complet et lire le rôle de chaque fichier, ouvrez l'[explorateur de plugin](https://code.claude.com/docs/en/plugins/components#explore-the-plugin-directory).

Le tableau liste les répertoires par lesquels commencent la plupart des plugins ; la [disposition complète](https://code.claude.com/docs/en/plugins/manifest-reference#standard-layout) liste les autres.

| Emplacement | Contenu |
| :- | :- |
| `.claude-plugin/plugin.json` | Le manifeste. Quand vous chargez un plugin avec `--plugin-dir` et qu'il n'a pas de manifeste, Claude Code nomme le plugin d'après son répertoire |
| `skills/` | Un répertoire `<name>/SKILL.md` par skill |
| `commands/` | Fichiers Markdown à plat, l'ancienne forme des skills. Utilisez `skills/` pour les nouveaux plugins |
| `agents/` | Un fichier Markdown par subagent |
| `hooks/hooks.json` | Configuration des hooks : une clé `"hooks"` de premier niveau dont la valeur a la même forme que `hooks` dans un fichier de réglages |
| `.mcp.json` | Définitions des serveurs MCP |

> **Avertissement** : seul `plugin.json` va dans `.claude-plugin/`. Les composants enregistrés à cet endroit **ne se chargent pas**.
>
> La racine du plugin est le répertoire propre du plugin, pas `~/.claude/` lui-même. Un `.mcp.json` enregistré sous `~/.claude/.mcp.json` ne se charge pas.

## Développer sans marketplace

Vous n'avez pas besoin d'une [marketplace](https://code.claude.com/docs/en/plugins/overview#get-plugins-from-a-marketplace) pour exécuter un plugin que vous êtes en train d'écrire. Chargez-le directement depuis le disque ou une URL :

- [`--plugin-dir`](#charger-un-plugin-pour-une-session) : charge un répertoire ou une archive `.zip` pour une session.
- [`--plugin-url`](#charger-un-plugin-depuis-une-url) : récupère une archive `.zip` depuis une URL pour une session.
- [`claude plugin init`](#faire-charger-un-plugin-dans-chaque-session) : échafaude un plugin sous `~/.claude/skills/` qui se charge dans chaque session.

Si deux plugins chargés de façons différentes portent le même nom, voir [Conflits de noms](https://code.claude.com/docs/en/plugins/loading#name-conflicts) pour savoir lequel Claude Code conserve.

### Charger un plugin pour une session

*(ancre d'origine : `#load-a-directory-or-archive-for-one-session`)*

Vous pouvez charger un plugin pour une seule session de trois façons : depuis un répertoire ou une archive `.zip` sur disque avec `--plugin-dir`, depuis une URL avec `--plugin-url`, ou depuis une variable d'environnement quand vous ne pouvez pas ajouter de flag. Chaque plugin se charge pour cette session uniquement, et **rien n'est écrit dans vos réglages**. Quand vous modifiez les fichiers du plugin pendant la session, lancez `/reload-plugins` pour charger les changements.

#### Charger un plugin depuis un répertoire ou un `.zip`

*(ancre d'origine : `#from-a-directory-or-zip`)*

Quand vous démarrez `claude` depuis votre shell, passez `--plugin-dir` avec le répertoire racine du plugin ou une archive `.zip` de celui-ci. Répétez le flag pour charger plusieurs plugins :

```bash
claude --plugin-dir ./my-first-plugin --plugin-dir ./other-plugin.zip
```

#### Charger un dossier de plugins

Pour charger plusieurs plugins depuis un même endroit, passez un dossier qui les contient, par exemple `--plugin-dir ./plugins` (v2.1.265+).

Si le dossier n'a ni répertoire `.claude-plugin/` ni composant de plugin à son premier niveau, Claude Code le traite comme un **dossier de plugins**. Chaque sous-dossier immédiat qui possède un manifeste `.claude-plugin/plugin.json` se charge alors comme un plugin distinct. Tout le reste du dossier est ignoré sans erreur, y compris un sous-dossier sans manifeste. Si un plugin du dossier ne se charge pas, vérifiez que son sous-dossier contient bien un `.claude-plugin/plugin.json`.

Vous pouvez aussi passer un dossier qui conserve un `.claude-plugin/marketplace.json` à côté de ses dossiers de plugins. Tant que ce répertoire `.claude-plugin/` ne contient pas de `plugin.json`, les dossiers de plugins se chargent quand même. Rien n'est installé ni activé depuis le fichier de marketplace, car Claude Code ne le lit pas (v2.1.281+ pour charger des plugins depuis un tel dossier).

Dans une session interactive, vous pouvez aussi ajouter et retirer des plugins du dossier après le démarrage :

- Un sous-dossier ajouté se charge comme nouveau plugin dès que son manifeste existe.
- Quand vous retirez un sous-dossier, son plugin est déchargé.

Un message apparaît dans la session pour chacun de ces changements. Si charger ou décharger un plugin en pleine conversation devait [invalider le cache de prompt](https://code.claude.com/docs/en/prompt-caching#enabling-or-disabling-a-plugin), le changement est mis en attente, et le message vous indique de lancer `/reload-plugins` pour l'appliquer.

#### Charger un plugin depuis une URL

*(ancre d'origine : `#fetch-an-archive-from-a-url-for-one-session`)*

Quand vous démarrez `claude` depuis votre shell, passez `--plugin-url` avec l'adresse d'une archive `.zip`, par exemple un artefact de build publié par votre CI :

```bash
claude --plugin-url https://example.com/my-first-plugin.zip
```

Claude Code télécharge l'archive au démarrage. Pour en charger plusieurs, répétez le flag ou passez les URL séparées par des espaces dans un seul argument entre guillemets.

> **Avertissement (sécurité)** : ne pointez ce flag que vers des archives que vous contrôlez ou en lesquelles vous avez confiance.

Si Claude Code ne peut pas récupérer l'archive, ou si l'archive est invalide, il démarre sans le plugin et enregistre une erreur de chargement de plugin consultable dans l'onglet **Errors** du gestionnaire `/plugin`.

#### Charger des plugins depuis une variable d'environnement

*(ancre d'origine : `#from-an-environment-variable`)*

Pour charger des plugins dans une session où vous ne pouvez pas ajouter le flag `--plugin-dir`, listez leurs chemins absolus dans la variable d'environnement [`CLAUDE_CODE_PLUGIN_DIRS`](https://code.claude.com/docs/en/env-vars#variables). Claude Code charge chaque chemin comme un chemin `--plugin-dir`. Ces plugins se chargent **en plus** de ceux passés avec `--plugin-dir`. [Les réglages de projet et locaux ne peuvent pas définir cette variable](https://code.claude.com/docs/en/settings-reference#variables-claude-code-ignores-in-env). `CLAUDE_CODE_PLUGIN_DIRS` : v2.1.280+.

Les réglages gérés (managed settings) peuvent désactiver `--plugin-dir` et `CLAUDE_CODE_PLUGIN_DIRS`. Voir [Flags qui chargent un plugin pour une session](https://code.claude.com/docs/en/plugins/cli-reference#flags-that-load-a-plugin-for-one-session). Pour tester un plugin avec un plugin dont il dépend, voir [Tester un plugin et sa dépendance en local](https://code.claude.com/docs/en/plugins/dependencies#test-a-plugin-and-its-dependency-locally).

### Faire charger un plugin dans chaque session

*(ancre d'origine : `#scaffold-a-plugin-that-loads-every-session`)*

Votre répertoire personnel de skills est `~/.claude/skills/`. Claude Code charge tout dossier qui s'y trouve et contient un `.claude-plugin/plugin.json` comme un plugin, **dans chaque session**, sans flag ni étape d'installation. `claude plugin init` échafaude pour vous un plugin de ce type.

#### Échafauder le plugin avec `claude plugin init`

`claude plugin init` écrit un plugin de départ sous `~/.claude/skills/`. Échafaudez-en un depuis votre shell :

```bash
claude plugin init my-tool
```

La commande crée `~/.claude/skills/my-tool/` avec un `.claude-plugin/plugin.json` et un `SKILL.md` à la racine. Elle affiche `✔ Created plugin "my-tool" at ~/.claude/skills/my-tool` suivi de `It will auto-load next session as my-tool@skills-dir. Run /reload-plugins to load it now.`

Passez `--with skills` pour que `claude plugin init` échafaude aussi un skill sous `skills/`. Les autres valeurs de `--with` figurent dans la [référence des commandes de plugin](https://code.claude.com/docs/en/plugins/cli-reference#plugin-init).

#### Nommer les skills du plugin

*(ancre d'origine : `#skill-names-in-a-scaffolded-plugin`)*

Le skill racine `~/.claude/skills/my-tool/SKILL.md` est aussi un skill personnel : vous l'invoquez donc avec `/my-tool`, et non `/my-tool:my-tool`. Les skills que vous ajoutez sous `skills/` dans le plugin reçoivent le préfixe du nom du plugin, par exemple `/my-tool:example`.

#### Arrêter de charger le plugin

Pour arrêter de charger un plugin échafaudé, supprimez son répertoire, ou lancez `claude plugin disable my-tool@skills-dir` dans votre shell avec le nom `my-tool@skills-dir` affiché par `claude plugin init`. Dans l'identifiant `my-tool@skills-dir`, `skills-dir` occupe la place d'un nom de marketplace, car le plugin se charge depuis votre répertoire de skills et non depuis une marketplace.

#### Partager le plugin via un dépôt

*(ancre d'origine : `#load-a-plugin-for-everyone-in-one-repository`)*

`claude plugin init` écrit le plugin dans votre répertoire personnel de skills `~/.claude/skills/` ; il se charge donc pour vous dans tous vos projets. Pour qu'un plugin se charge pour tout le monde dans un dépôt donné, créez vous-même la même disposition sous `<project>/.claude/skills/<name>/`, y compris son `.claude-plugin/plugin.json`. Voir [Plugins partagés via un dépôt](https://code.claude.com/docs/en/plugins/loading#plugins-shared-through-a-repository) pour les conditions dans lesquelles Claude Code le charge.

## Tester et déboguer

Quand une modification de votre plugin n'apparaît pas, effectuez ces vérifications dans l'ordre. Chacune vous indique ce que Claude Code a fait du plugin :

1. Dans votre shell, lancez `claude plugin validate <path>`. Il vérifie le manifeste et le frontmatter de chaque fichier de skill, d'agent et de commande, et se termine avec le code `0` sur `Validation passed`. Ajoutez `--strict` pour échouer aussi sur les avertissements. Les codes de sortie et la gestion des répertoires figurent dans la [référence des commandes de plugin](https://code.claude.com/docs/en/plugins/cli-reference#plugin-validate).
2. Dans la session en cours, lancez `/reload-plugins` pour appliquer les modifications faites sur disque. Il affiche une ligne `Reloaded:` avec des compteurs. Confirmez ensuite qu'un skill est chargé en tapant sa commande `/plugin-name:skill`, ou en trouvant le plugin dans l'onglet **Installed** de `/plugin`.
3. Dans la même session, lancez `/plugin`. L'onglet **Installed** liste votre plugin et, dans ses détails, les composants que Claude Code a trouvés. L'onglet **Errors** liste ce qui n'a pas pu se charger et pourquoi, par exemple un chemin de votre manifeste qui n'existe pas.
4. De retour dans votre shell, lancez `claude plugin list`. Il affiche les plugins de session et ceux du répertoire de skills dans des sections distinctes, avec `Status: ✔ loaded` ou l'erreur de chargement. Pour inclure le plugin en cours de développement, passez `--plugin-dir` avec son chemin avant `plugin list`.

Pour vérifier un serveur MCP, lancez `/mcp` dans la session pour voir son état. Quand le serveur fonctionne, `/mcp` l'indique comme connecté. Sinon, voir [Serveurs MCP qui ne démarrent pas](https://code.claude.com/docs/en/plugins/troubleshooting#invalid-mcp-server-config-for-and-mcp-servers-that-dont-start).

Pour vérifier un hook, déclenchez l'événement qu'il cible. Par exemple, demandez à Claude de modifier un fichier pour déclencher un hook `PostToolUse`. Lisez ensuite le [journal de débogage](https://code.claude.com/docs/en/hooks#debug-hooks), qui montre quels hooks ont correspondu, leurs codes de sortie et leur sortie.

Les sections suivantes couvrent les échecs les plus probables pendant le développement ; la [page de dépannage](https://code.claude.com/docs/en/plugins/troubleshooting#build-a-plugin) contient l'entrée complète de chacun.

### Un chemin de composant est introuvable

L'onglet **Errors** de `/plugin` affiche `<component> path not found: <path>`, par exemple `commands path not found`. Un chemin de composant de votre manifeste, comme `commands`, `skills`, `agents` ou `hooks`, ne pointe vers rien. Corrigez le chemin ou créez le répertoire, puis lancez `/reload-plugins` dans la session. Voir [`commands path not found`](https://code.claude.com/docs/en/plugins/troubleshooting#commands-path-not-found).

### `--plugin-dir` sur la racine d'une marketplace ne charge pas les plugins sous `plugins/`

`--plugin-dir` attend le répertoire racine du plugin, celui qui contient `.claude-plugin/plugin.json` et les répertoires de composants comme `skills/`. Si vous le pointez vers la racine d'une marketplace, Claude Code ne lit pas `marketplace.json` : un plugin sous `plugins/` ne se charge pas, et **aucune erreur n'apparaît**. Pointez le flag vers le dossier d'un plugin, ou ajoutez la marketplace. Voir [l'entrée de dépannage](https://code.claude.com/docs/en/plugins/troubleshooting#plugin-dir-loads-a-plugin-with-no-components).

### Le plugin se charge mais ses skills manquent

Le répertoire `skills/` se trouve dans `.claude-plugin/`, ou une entrée `skills` du manifeste pointe vers un fichier. Déplacez `skills/` à la racine du plugin, faites pointer chaque entrée `skills` vers un répertoire qui contient un `SKILL.md`, et lancez `/reload-plugins` dans la session. Voir [Le plugin se charge mais ses skills manquent](https://code.claude.com/docs/en/plugins/troubleshooting#plugin-loads-but-its-skills-are-missing).

### La boîte de dialogue `userConfig` n'apparaît jamais

La boîte de dialogue des options [`userConfig`](https://code.claude.com/docs/en/plugins/components#user-configuration) de votre plugin fait partie de l'installation via `/plugin` dans une session. Le chargement avec `--plugin-dir` ne l'affiche pas, pas plus que `claude plugin install` dans le shell. Avec le plugin chargé, lancez `/plugin configure <plugin-name>` dans la session pour l'ouvrir. Voir [La boîte de dialogue `userConfig` n'apparaît jamais](https://code.claude.com/docs/en/plugins/troubleshooting#the-userconfig-dialog-never-appears).

### Vérifier que le plugin modifie le comportement de Claude

Un plugin qui se charge sans erreur peut tout de même ne pas orienter Claude comme vous le souhaitez. `claude plugin eval`, à lancer dans votre shell, exécute vos cas de test avec et sans le plugin et note la différence. Voir [Tester les plugins avec des evals](https://code.claude.com/docs/en/plugin-evals), en commençant par [Créer votre première suite d'evals](https://code.claude.com/docs/en/plugin-evals#create-your-first-eval-suite).

## Convertir une configuration `.claude/` existante

*(ancre d'origine : `#convert-an-existing-claude-setup`)*

Si vous avez déjà des skills, agents ou hooks sous le répertoire `.claude/` d'un projet, vous pouvez les déplacer dans un plugin sans les réécrire.

Exécutez les commandes de ces étapes depuis la racine du projet, c'est-à-dire le répertoire qui contient `.claude/`, car les chemins des `cp` sont relatifs à celle-ci.

### Étape 1 — Créer la structure du plugin

Créez le répertoire du plugin et son dossier `.claude-plugin/` à côté de `.claude/`. Vous pourrez déplacer le plugin ailleurs ensuite.

```bash
mkdir -p my-plugin/.claude-plugin
```

Créez `my-plugin/.claude-plugin/plugin.json` :

```json my-plugin/.claude-plugin/plugin.json
{
  "name": "my-plugin",
  "description": "Migrated from standalone configuration",
  "version": "1.0.0"
}
```

### Étape 2 — Copier vos fichiers existants

Copiez chaque répertoire de configuration que vous possédez à la racine du plugin, et sautez la commande correspondant à tout répertoire que vous n'avez pas.

```bash
cp -r .claude/commands my-plugin/
```

```bash
cp -r .claude/agents my-plugin/
```

```bash
cp -r .claude/skills my-plugin/
```

Lancez `ls -a my-plugin` pour confirmer que chaque répertoire copié apparaît à côté de `.claude-plugin`.

### Étape 3 — Déplacer vos hooks

Si vous avez des hooks dans `.claude/settings.json` ou `.claude/settings.local.json`, créez un répertoire de hooks :

```bash
mkdir -p my-plugin/hooks
```

Créez `my-plugin/hooks/hooks.json` et copiez-y l'objet `hooks` de votre fichier de réglages. Le format est identique.

Cet exemple montre la forme avec un hook qui lance un linter sur chaque fichier que Claude écrit ou modifie. Remplacez l'exemple par votre propre objet `hooks`.

```json my-plugin/hooks/hooks.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{ "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npm run lint:fix" }]
      }
    ]
  }
}
```

### Étape 4 — Tester le plugin migré

Chargez le plugin pour une session :

```bash
claude --plugin-dir ./my-plugin
```

Vérifiez chaque composant sous son nouveau nom :

- **Skills** : lancez `/my-plugin:deploy` pour un skill qui s'appelait `/deploy`.
- **Subagents** : demandez à Claude d'utiliser l'agent `my-plugin:reviewer` pour un agent qui s'appelait `reviewer`.
- **Hooks** : déclenchez l'événement ciblé par chaque hook.

S'il manque quelque chose, suivez [Tester et déboguer](#tester-et-déboguer).

---

Tant que les originaux sont encore sous `.claude/`, ils restent chargés à côté des copies du plugin :

- **Skills et agents** : les deux ensembles n'entrent pas en collision, car les skills et agents du plugin portent le préfixe `my-plugin:`. `/deploy` et `/my-plugin:deploy` fonctionnent tous deux, et Claude voit `reviewer` et `my-plugin:reviewer` comme deux subagents distincts.
- **Hooks** : les hooks n'ont pas de préfixe ; un hook présent à la fois dans votre fichier de réglages et dans `hooks/hooks.json` **s'exécute deux fois** à chaque déclenchement de son événement.

Une fois que vous avez confirmé que le plugin fonctionne, supprimez les originaux de `.claude/` et retirez l'objet `hooks` de votre fichier de réglages.

## Étapes suivantes

- [Composants de plugin](https://code.claude.com/docs/en/plugins/components) : ajouter agents, hooks, serveurs MCP, serveurs LSP et configuration utilisateur à votre plugin.
- [Tester les plugins avec des evals](https://code.claude.com/docs/en/plugin-evals) : écrire des cas d'eval et les exécuter avec `claude plugin eval` pour vérifier avec quelle fiabilité le plugin guide le comportement de Claude.
- [Publier un plugin](https://code.claude.com/docs/en/plugins/publish) : le versionner, le placer dans une marketplace et le soumettre à la revue.
- [Plugin structure and testing](https://claude.com/docs/plugins/build) : le même dossier de plugin s'installe sur claude.ai et dans Cowork. Certains composants sont propres à Claude Code, et le [tableau de prise en charge des composants](https://claude.com/docs/plugins/platform-support#compare-component-support-by-app) liste ceux qui se chargent sur chaque surface.
- [Référence du manifeste de plugin](https://code.claude.com/docs/en/plugins/manifest-reference) : chaque champ de `plugin.json`, règle de chemin et répertoire.
- [Skills](https://code.claude.com/docs/en/skills) : écrire les skills que fournit votre plugin.
- [Plugins d'Anthropic dans le dépôt claude-code](https://github.com/anthropics/claude-code/tree/main/plugins) : exemples complets de la disposition décrite sur cette page, comme `feature-dev` et `code-review`.
