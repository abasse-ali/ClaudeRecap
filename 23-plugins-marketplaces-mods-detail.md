# 23 — Plugins, marketplaces et mods en détail

[← Sommaire](README.md) · Introduction : [11 — MCP, plugins, mods](11-mcp-plugins-mods.md)

## 1. Ce qu'un plugin peut contenir

| Composant | Emplacement par défaut | Contenu |
|-----------|------------------------|---------|
| Manifeste | `.claude-plugin/plugin.json` | Nom, version, métadonnées (facultatif mais recommandé) |
| Skills | `skills/<nom>/SKILL.md` | Skills appelables en `/plugin:skill` (ou `SKILL.md` à la racine pour un plugin à skill unique) |
| Commandes | `commands/*.md` | Ancien format de commandes (préférer `skills/`) |
| Agents | `agents/*.md` | Sous-agents (les sous-dossiers font partie du nom : `plugin:review:security`) |
| Hooks | `hooks/hooks.json` | Hooks ; peut aussi pointer vers un **module de hooks** JavaScript (= mod) |
| Serveurs MCP | `.mcp.json` | Serveurs MCP démarrés quand le plugin est actif |
| Serveurs LSP | `.lsp.json` | Serveurs de langage (intelligence de code) |
| Styles de sortie | `output-styles/` | Styles Markdown |
| Workflows | `workflows/*.js` | Workflows dynamiques réutilisables |
| Thèmes | `themes/*.json` | Thèmes de couleurs |
| Moniteurs | `monitors/monitors.json` | Processus de fond qui renvoient leur sortie à Claude |
| Exécutables | `bin/` | Ajoutés au `PATH` de Bash tant que le plugin est actif |
| Réglages | `settings.json` | Valeurs par défaut de `agent` et `subagentStatusLine` |
| Channels | (dans le manifeste) | Canaux de messages liés à un serveur MCP |
| Évaluations | `evals/` | Cas de test du plugin |

Variables utilisables dans les configurations : `${CLAUDE_PLUGIN_ROOT}` (dossier installé du plugin), `${CLAUDE_PLUGIN_DATA}` (`~/.claude/plugins/data/<id>/`, conservé entre les mises à jour, idéal pour installer des dépendances), `${CLAUDE_PROJECT_DIR}`, `${user_config.CLÉ}`.

## 2. Le manifeste `plugin.json`

Seul `name` est obligatoire.

| Champ | Rôle |
|-------|------|
| `name` | Identifiant (kebab-case) ; tous les composants sont préfixés par lui |
| `displayName` | Nom affiché |
| `version` | Version ; tant qu'elle ne change pas, les utilisateurs restent dessus |
| `description`, `author` (`name`, `email`, `url`), `homepage`, `repository`, `license`, `keywords` | Métadonnées |
| `icon`, `documentationUrl`, `supportUrl`, `privacyPolicyUrl`, `termsOfServiceUrl` | Fiche dans l'annuaire d'Anthropic |
| `metadata` | Données libres |
| `defaultEnabled` | Activé par défaut (true) |
| `dependencies` | Plugins requis (`"nom"`, `"nom@marketplace"` ou objet avec `version` semver : `~2.1.0`, `^2.0`, `>=1.4`) |
| `settings` | `agent`, `subagentStatusLine` appliqués quand le plugin est actif |
| `userConfig` | Options demandées à l'utilisateur à l'activation (`type` string/number/boolean/directory/file, `title`, `description`, `required`, `default`, `options`, `multiple`, `sensitive` → stockage sécurisé, `min`/`max`) |
| `channels` | Canaux (`server`, `displayName`, `userConfig`) |
| `skills`, `commands`, `agents`, `hooks`, `mcpServers`, `lspServers`, `outputStyles`, `workflows` | Chemins personnalisés (remplacent l'emplacement par défaut, sauf `skills` qui s'ajoute) ; `commands` accepte aussi un objet `{ nom: { source \| content, description, argumentHint, model, allowedTools } }` |
| `experimental.themes`, `experimental.monitors`, `experimental.evals` | Thèmes, moniteurs, dossier d'évals |
| `types` | Fichier `.d.ts` d'un mod |

**Serveur LSP** (`.lsp.json`) : `command`, `extensionToLanguage` (obligatoires), `args`, `transport`, `env`, `initializationOptions`, `settings`, `workspaceFolder`, `startupTimeout`, `shutdownTimeout`, `requestTimeout`, `restartOnCrash`, `maxRestarts`, `diagnostics`.

**Moniteur** : `name`, `command`, `description`, `when` (`always` ou `on-skill-invoke:<skill>`).

## 3. Commandes `claude plugin …` (shell)

| Commande | Rôle |
|----------|------|
| `claude plugin init <nom>` (alias `new`) | Créer un squelette de plugin dans `~/.claude/skills/<nom>/` |
| `claude plugin install <plugin>@<marketplace> [--scope user\|project\|local] [--marketplace <source>]` | Installer (en ajoutant la marketplace si besoin) |
| `claude plugin uninstall`, `enable`, `disable` | Gérer |
| `claude plugin update` | Mettre à jour |
| `claude plugin list [--json]` | Lister (version, portée, statut) |
| `claude plugin details <nom>` | Inventaire des composants et **coût en jetons** projeté |
| `claude plugin configure <plugin>` | Voir/enregistrer les options `userConfig` |
| `claude plugin prune` (alias `autoremove`) | Supprimer les dépendances installées automatiquement devenues inutiles |
| `claude plugin validate <chemin> [--strict] [--json]` | Valider un plugin, une marketplace ou un dossier de skills/agents (codes de sortie pour la CI) ; liste aussi ce qu'un mod est capable de faire |
| `claude plugin eval [init]` | Lancer / générer des **évaluations** |
| `claude plugin test [dossier]` | Lancer les tests d'un mod (sans session ni réseau) |
| `claude plugin tag` | Créer un tag git de release `<nom>--v<version>` après vérification des versions |
| `claude plugin marketplace add \| list \| remove \| update` | Gérer les marketplaces |

En session : `/plugin` (onglets Discover, Installed, Marketplaces, Errors), `/plugin install|enable|disable|uninstall|marketplace …`, `/reload-plugins [--force]`. Pour une session : `--plugin-dir`, `--plugin-url`, `CLAUDE_CODE_PLUGIN_DIRS`.

## 4. Évaluer un plugin (`claude plugin eval`)

- Une suite d'évaluation vit dans `evals/` : un sous-dossier par **cas** (un `prompt.md` + des **graders**).
- Chaque cas tourne plusieurs fois (3 par défaut) dans une session isolée avec seulement votre plugin, **et sans le plugin** (référence) : le rapport montre les scores `WITH` et `W/OUT` et leur différence.
- Graders déterministes ou `llm` (verdict d'un modèle) ; poids ; limites de tours et d'outils ; `--disallow-tools`.
- `claude plugin eval init` : Claude écrit les cas pour vous. Utilisable en CI avec un seuil de score.

## 5. Mesurer et promouvoir un plugin

- **Coût** : `claude plugin details` (« every turn » = ce que le plugin ajoute à chaque message ; « on use » = quand il sert). La marketplace officielle affiche ce coût avant installation.
- **Usage** : enregistré sur la machine de chaque utilisateur ; événements OpenTelemetry pour les questions d'organisation.
- **Suggestion depuis votre CLI** : si votre outil en ligne de commande détecte `CLAUDECODE`, il peut afficher sur sa propre ligne `<claude-code-hint v="1" type="plugin" value="nom@claude-plugins-official" />` pour proposer l'installation du plugin (marketplace officielle, sessions interactives seulement).
- **Pertinence** : un bloc `relevance` dans `marketplace.json` (sujet + signaux : fichiers lus, commandes lancées…) fait suggérer le plugin quand le travail correspond (« Working with Terraform? »).

## 6. Marketplaces

Une marketplace = un dépôt (ou dossier, ou URL) contenant `.claude-plugin/marketplace.json`.

| Champ racine | Rôle |
|--------------|------|
| `name` | Identifiant de la marketplace |
| `owner` | Mainteneur (`name` obligatoire) |
| `plugins` | Entrées de plugins |
| `description`, `version`, `metadata.pluginRoot` | Description, version, dossier de base des plugins |
| `forceRemoveDeletedPlugins` | Désinstaller chez les utilisateurs un plugin retiré |
| `allowCrossMarketplaceDependenciesOn` | Marketplaces autorisées pour les dépendances |
| `renames` | Renommages de plugins (ou `null` pour un retrait) |

**Entrée de plugin** : `name`, `source`, `description`, `version`, `category`, `tags`, `strict` (le `plugin.json` fait foi), `relevance`, `dependencies`, `defaultEnabled`, `displayName`, `metadata`, `headers`/`headersHelper` (téléchargements authentifiés), plus des champs de composants.

**Sources de plugin** : chemin relatif, `github` (`repo`, `ref`, `sha`), `url` (dépôt git), `git-subdir` (sous-dossier d'un dépôt), `npm` (`package`, `version`, `registry`), `archive` (zip HTTPS + `sha256`), `command` (dossier produit par une commande).

**Niveaux de confiance** : officiel (`claude-plugins-official`…), communautaire (`claude-community`, `claude-plugins-community`, `healthcare`), tiers (tous les autres). Les noms officiels ne sont acceptés que depuis `github.com/anthropics/`.

**Distribution** : sans marketplace (dossier ou `.zip` via `--plugin-url`), votre propre marketplace (dépôt git, éventuellement privé), ou l'**annuaire d'Anthropic** (claude.ai, Cowork, puis Claude Code via la synchronisation).

**Créer et héberger une marketplace** :
1. Écrire `.claude-plugin/marketplace.json` (un objet par plugin dans `plugins` : `name`, `source`, `description`…), tester en local avec `/plugin marketplace add ./ma-marketplace`.
2. Héberger sur GitHub, un autre hôte git, une URL de `marketplace.json` (éviter alors les sources en chemin relatif) ou un partage de fichiers.
3. Partager pour tout un dépôt : `claude plugin marketplace add votre-org/marketplace --scope project` puis committer `.claude/settings.json`.
4. Publier des mises à jour (versions, `claude plugin tag`), renommer sans casser les installations (`renames`), retirer des plugins (`forceRemoveDeletedPlugins`).

**Comment un plugin se charge** (pour comprendre un plugin qui ne marche pas) : *déclaré* dans les réglages (`enabledPlugins`, `extraKnownMarketplaces`) → *récupéré* sur le disque (`~/.claude/plugins/` : `known_marketplaces.json`, `installed_plugins.json`, cache) → *chargé* par la session (au démarrage ou avec `/reload-plugins`). Les conflits de noms et les versions figées expliquent la plupart des « la mise à jour n'a rien changé ».

**Dépanner un mod** : `claude plugin validate` (ce que Claude Code lit), `claude plugin test` depuis un dossier vide (les mods peuvent-ils se charger ?), lignes grisées dans la transcription avec `--plugin-dir`, sinon journal `--debug`.

## 7. Plugins pour une organisation (réglages gérés)

- **Pré-installer et imposer** : `extraKnownMarketplaces` + `enabledPlugins` dans les réglages gérés.
- **Restreindre** : `strictKnownMarketplaces` (liste blanche), `blockedMarketplaces`, `disableCommandPluginSources`, `strictPluginOnlyCustomization` (skills/agents/hooks/MCP seulement depuis plugins et réglages gérés), `disableSideloadFlags`.
- **Suggestions** : `pluginSuggestionMarketplaces` ; message de confiance : `pluginTrustMessage`.
- **Mods** : `allowManagedModsOnly`, `allowModsToOverrideDenyRules` (options du garde intégré), `prependPlugins`/`appendPlugins` (mods de l'organisation exécutés avant/après ceux des utilisateurs).

## 8. Mods : référence

### Fichiers

| Fichier | Rôle |
|---------|------|
| `.claude-plugin/plugin.json` | Manifeste |
| `hooks/hooks.json` | `{ "modules": ["./register.js"] }` |
| `hooks/register.js` | Le **module de hooks** : exporte `register(on, options)` |
| `types/index.d.ts` | Types de `$.state` (si utilisé) |
| `*.test.ts(x)` | Tests (`claude plugin test`) |

### La fonction hook

```js
export function register(on) {
  on('tool.call', { tool: 'Bash' }, async ($, e, next) => {
    if (e.input.command.includes('rm -rf')) return { deny: 'Commande refusée par mon mod' };
    return next(e); // laisser passer (ou next({...e, input: …}) pour réécrire)
  });
}
```

Arguments : `$` (l'API des mods), `e` (l'événement, figé), `next(e)` (passer au suivant, comme un middleware) ; `next.signal`, `next.origin`, `next.budget`, `next.to(e, tier)`, `.catch` pour gérer les échecs. Un hook peut **observer**, **réécrire** ou **répondre** (court-circuiter).

### Événements

| Famille | Événements |
|---------|------------|
| Outils | `tool.call`, `tool.check`, `tool.describe` |
| Prompts | `prompt.submit`, `prompt.fill`, `prompt.suggest`, `prompt.edit`, `prompt.autocomplete`, `prompt.compose`, `prompt.section`, `prompt.context`, `prompt.attachment`, `prompt.mention`, `skill.prompt`, `attribution.text` |
| Commandes et config | `command.run`, `command.describe`, `config.set`, `config.describe` |
| Tours | `turn.start`, `turn.step`, `turn.complete` |
| Session | `session.start`, `session.end`, `session.compact`, `session.receive`, `session.send`, `session.append`, `session.attach`, `session.detach`, `session.measure` |
| Sous-agents | `agent.offer`, `agent.spawn` |
| Interface | `ui.render`, `ui.resolve`, `ui.press`, `ui.input`, `ui.select`, `ui.focus`, `ui.scroll`, `ui.close`, `ui.message`, `ui.fault` |
| Autres mods | `plugin.register`, `engine.create` |
| Télémétrie | `telemetry.log`, `telemetry.mark` |

### API des mods (`$`)

| Espace | Méthodes |
|--------|----------|
| `$.plugin` | `name`, `root` |
| `$.ui` | `resolve`, `invalidate`, `open`, `close`, `panes`, `focus`, `scroll`, `toast`, `status`, `log`, `notice`, `ask`, `copy`, `selection`, `notify`… |
| `$.command` | `register`, `run`, `list` |
| `$.tool` | `register` (outil exposé à Claude), `call`, `check`, `list` |
| `$.agent` | `register`, `spawn`, `list` |
| `$.model` | `complete` (requête indépendante), `fork` (avec la conversation), `classify` |
| `$.prompt` | `submit`, `read`, `fill`, `suggest`, `compose` |
| `$.turn` | `abort` |
| `$.session` | `messages`, `cwd`, `root`, `model`, `turns`, `id`, `repo`, `surfaces`, `usage`, `version`, `compact`, `send`, `append`, `authorize` |
| `$.config`, `$.settings`, `$.env` | Réglages `/config`, lecture des réglages, variables d'environnement |
| `$.fs` | `read`, `write`, `list`, `exists`, `stat`, `ancestors` |
| `$.store` | Stockage clé-valeur partagé par toutes les sessions de la machine (4 Mio) |
| `$.state` | État réactif (`atom`, `derive`…) |
| `$.clock` | `now`, `sleep`, `after`, `every` |
| `$.http` | `fetch` |
| `$.process` | `run`, `spawn` |
| `$.mcp` | `call`, `connect` |
| `$.audio` | `play`, `speak` |
| `$.telemetry` | `log`, `mark` |

### Dessiner dans l'interface

- **Volet** (pane) à côté de la transcription (terminal large, plein écran) ou région au-dessus du prompt ; **bande** au-dessus du prompt ; options `title`, `focus`, `closeOnEscape`…
- **Sites de rendu existants** que l'on peut restyler ou remplacer : `UserMessage`, `AssistantMessage`, `ToolUse`, `ToolResult`, `ToolGroup`, `CommandOutput`, `AskUserQuestion`, `Spinner`, `ToolProgress`, `TurnDuration`, `InfoNotice`, `SessionMode`, `PromptHint`.
- **Éléments** : `Box` (mise en page flex, bordures `single`/`double`/`round`/`bold`/`dashed`/`quote`…), `Text`, `Button`, `Link`, `Code` (code coloré ou diff), `Markdown`, `Input`, `Select`, `Svg`, `Raster` (grille de cellules colorées), `Image` (PNG), `Client`.
- **Limites** : `$.process.run` 30 s (max 10 min), `$.model.complete` 1 024 jetons par défaut, fichiers 4 Mio, `$.store` 4 Mio, 10 rafraîchissements/s, toasts de 4 s.

### Créer, tester, administrer

- **Demander à Claude** : décrivez le mod ; il utilise le skill `plugin-authoring` et l'écrit dans la session (puis copiez-le dans un plugin pour le garder).
- **Écrire soi-même** : exemple officiel `first-mod` qui compte les appels d'outils, l'affiche près du spinner et ajoute `/tally`.
- **Rechargement à chaud** avec `--plugin-dir` ; types fournis (`claude-code/index.d.ts`, outils, MCP) pour l'autocomplétion.
- **Tests** : `claude plugin test` ; on simule (stub) les réponses de Claude Code (`$.store.get`, `$.model.complete`, `$.process.run`…), on déclenche des événements, on appuie sur des boutons.
- **Galerie** : exemples visuels de chaque élément (texte, boutons, champs, Markdown, code, diffs).
- **Administration** : garde intégré `cc-plugin-sec-default`, `allowManagedModsOnly`, revue des capacités avec `claude plugin validate` (`$.fs`, `$.process`, `$.http` = accès fichiers, programmes, réseau).
- **Sécurité** : un mod voit chaque prompt et appel d'outil, peut les modifier, approuver des appels, appeler des modèles, lire vos secrets. N'installez que des mods de confiance.
