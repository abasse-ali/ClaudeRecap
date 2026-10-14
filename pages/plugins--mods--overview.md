# Vue d'ensemble des mods

> Source : https://code.claude.com/docs/en/plugins/mods/overview · [← Index](../31-index-documentation-officielle.md)

*Ajoutez des panneaux, des commandes et des règles d'appel d'outil à Claude Code avec un mod. Voyez ce qu'un mod peut faire, comment en créer ou en installer un, et où les mods s'exécutent.*

Un mod est un [plugin](https://code.claude.com/docs/en/plugins/overview) qui change l'apparence et le comportement de Claude Code. Il est constitué de gestionnaires d'événements écrits en JavaScript ou en TypeScript : Claude Code en appelle un lorsqu'un événement se produit — un appel d'outil, un prompt soumis, ou le dessin d'une partie de l'interface — et le gestionnaire peut observer l'événement, le modifier ou le prendre en charge lui-même. Utilisez un mod pour ajouter une fonctionnalité à vous dans Claude Code, par exemple un panneau qui trace en graphique le remplissage de votre contexte après chaque requête. Pour les fichiers d'un mod et un exemple complet, voir [Fonctionnement d'un mod](#fonctionnement-dun-mod).

> **Note** : les [hooks](https://code.claude.com/docs/en/hooks) existants de Claude Code s'exécutent aussi sur des événements, sous la forme d'une commande shell, d'une requête HTTP ou d'un prompt que vous configurez dans un fichier de réglages. Les gestionnaires d'un mod, eux, sont des fonctions qui s'exécutent **à l'intérieur** de Claude Code. Claude Code appelle les deux « hooks » : sur ces pages, « hook » désigne un gestionnaire de mod, et le type défini dans un fichier de réglages est appelé « settings hook » (hook de réglages).

## Ce qu'un mod peut faire

Les hooks de réglages, les skills, les lignes de statut et les serveurs MCP agissent depuis l'extérieur de Claude Code : chacun exécute un script, ou fournit du texte ou des outils à Claude. Un mod s'exécute à l'intérieur de Claude Code, il peut donc faire ce qu'eux ne peuvent pas :

* **Dessiner une interface utilisable** : un panneau (pane) à côté de la transcription ou une bande au-dessus du prompt, avec des onglets, des boutons et des champs de texte. Voir [Dessiner dans l'interface](https://code.claude.com/docs/en/plugins/mods/interface).
* **Redessiner l'interface propre de Claude Code** : remplacer ou restyler des parties que Claude Code dessine lui-même, comme la ligne d'un appel d'outil, le spinner ou la boîte de dialogue dans laquelle Claude pose des questions. Voir [Changer ce que Claude Code dessine déjà](https://code.claude.com/docs/en/plugins/mods/interface#change-what-claude-code-already-draws).
* **Intervenir dans un appel d'outil ou une requête** : par exemple, retenir un appel d'outil pendant que vous posez une question à l'utilisateur, y répondre sans exécuter l'outil, ou envoyer une requête à un autre modèle. Voir [Garder ou modifier un appel d'outil](https://code.claude.com/docs/en/plugins/mods/events#guard-or-change-a-tool-call) et [Suivre un tour](https://code.claude.com/docs/en/plugins/mods/events#follow-a-turn).
* **Exécuter votre propre code sur une commande** : une `/commande` qui exécute votre fonction immédiatement, sans tour de Claude, même pendant que Claude travaille. Voir [Ajouter une commande ou un outil](https://code.claude.com/docs/en/plugins/mods/api#add-a-command-or-a-tool).
* **Partager des données entre hooks** : les hooks d'un mod partagent les variables de son fichier, donc ce qu'un hook enregistre, un autre peut l'afficher. Par exemple, un hook peut compter les appels d'outil pendant qu'un autre affiche le compte à côté du spinner, ou un hook peut lire l'utilisation de tokens de chaque requête pendant qu'un autre la trace dans un panneau. Voir [Réagir aux événements](https://code.claude.com/docs/en/plugins/mods/events).

Les mods fonctionnent dans la CLI Claude Code et dans l'onglet Code de l'application Claude Desktop. Voir [Où s'exécutent les mods](#où-sexécutent-les-mods) pour comprendre leur comportement ailleurs, comme dans l'extension VS Code, `claude -p` et les sessions cloud. Si un hook de réglages, une skill ou un serveur MCP fait déjà ce dont vous avez besoin, [comparez-les](#comparer-mods-hooks-de-réglages-skills-et-serveurs-mcp) avant d'écrire un mod. Pour gérer les mods d'une organisation, voir [Gérer les mods pour votre organisation](https://code.claude.com/docs/en/plugins/mods/admin).

## Obtenir un mod

Pour commencer avec un mod :

* **En utiliser un que vous avez déjà** : certaines fonctionnalités de Claude Code sont elles-mêmes des mods, comme `/diff`. Voir [Mods intégrés à Claude Code](#mods-intégrés-à-claude-code).
* **En créer un** : décrivez ce que vous voulez dans une session Claude Code, et Claude écrit le mod. Voir [Demander un mod à Claude](https://code.claude.com/docs/en/plugins/mods/create#ask-claude-for-a-mod). Pour apprendre comment fonctionne le code d'un mod, [écrivez-en un vous-même](https://code.claude.com/docs/en/plugins/mods/create#write-a-mod-yourself).
* **En installer un** : voir [Installer ou mettre à jour un mod](#installer-ou-mettre-à-jour-un-mod), ou [essayer un mod d'exemple](#essayer-un-mod-dexemple).

### Installer ou mettre à jour un mod

> **Avertissement** : un mod est du code qui s'exécute avec vos permissions. Il peut lire et écrire vos fichiers, lancer des processus et effectuer des requêtes réseau. N'installez des mods que depuis des auteurs et des marketplaces de confiance. Voir [Décider si l'on peut faire confiance à un mod](#décider-si-lon-peut-faire-confiance-à-un-mod).

Un mod s'installe comme un plugin, depuis une marketplace. Indiquez le nom du plugin, un `@`, puis le nom de la marketplace. Ces exemples installent un plugin nommé `token-chart` depuis une marketplace nommée `your-org` :

* Dans une session Claude Code, exécutez `/plugin install token-chart@your-org`.
* Dans votre shell, exécutez `claude plugin install token-chart@your-org`.

[Installer des plugins](https://code.claude.com/docs/en/plugins/install) couvre les marketplaces, les portées (scopes), l'extension VS Code et l'application Desktop, ainsi que la [mise à jour des plugins](https://code.claude.com/docs/en/plugins/install#keep-plugins-updated) ; tout cela s'applique sans changement à un plugin qui contient un mod.

Si vous installez ou mettez à jour un mod depuis votre shell alors qu'une session est ouverte, exécutez `/reload-plugins` dans cette session pour le charger. Sinon, il se charge au prochain démarrage de Claude Code.

### Essayer un mod d'exemple

Anthropic partage des mods d'exemple dans le [répertoire `claude-code/mods` du dépôt `claude-code-playground`](https://github.com/anthropics/claude-code-playground/tree/main/claude-code/mods). Chacun est un plugin complet, et son README explique comment il a été construit. Le dépôt les partage en l'état, sans support.

* [`token-weather`](https://github.com/anthropics/claude-code-playground/tree/main/claude-code/mods/token-weather) : dessine une « prévision météo » de votre fenêtre de contexte au-dessus du prompt
* [`blast-radius`](https://github.com/anthropics/claude-code-playground/tree/main/claude-code/mods/blast-radius) : retient une commande shell risquée, comme `rm -rf` ou un force push, et montre ce qu'elle changerait, avec des boutons pour continuer ou annuler
* [`replay-theater`](https://github.com/anthropics/claude-code-playground/tree/main/claude-code/mods/replay-theater) : ajoute une commande `/replay` qui parcourt pas à pas les modifications de fichiers faites par Claude au dernier tour

Un mod d'exemple s'exécute avec vos permissions. Pour voir ce qu'il fait avant de le charger, [listez ses hooks et ses appels](#lister-ce-que-fait-un-mod-avant-de-linstaller).

Pour en essayer un, clonez le dépôt et [chargez le répertoire du mod pour une session](https://code.claude.com/docs/en/plugins/create#load-a-directory-or-archive-for-one-session) avec `--plugin-dir`. Pour confirmer que le mod est chargé, [vérifiez quels mods la session a chargés](#voir-quels-mods-une-session-a-chargés).

Pour en garder un, [ajoutez le répertoire `claude-code/mods` de votre clone comme marketplace](https://code.claude.com/docs/en/plugins/install#add-a-marketplace), puis installez le mod depuis `claude-code-playground-mods`. La marketplace pointe vers votre clone : le mod cesse de se charger si vous le déplacez ou le supprimez.

## Décider si l'on peut faire confiance à un mod

Un mod est du code qui s'exécute avec vos permissions, à l'intérieur de Claude Code. N'installez des mods que depuis des auteurs et des [marketplaces de confiance](https://code.claude.com/docs/en/plugins/security).

### Ce à quoi un mod peut accéder

Un mod s'exécute avec vos permissions ; avant d'en installer un, sachez à quoi il a accès. Une fois chargé, un mod peut :

* **Agir sur votre machine en votre nom** : lire et écrire des fichiers partout où votre compte utilisateur le peut, lancer des programmes et effectuer des requêtes réseau
* **Lire vos secrets** : variables d'environnement et fichiers de réglages, y compris une clé API que vous conservez dans l'un ou l'autre
* **Voir votre session** : chaque prompt que vous envoyez et chaque appel d'outil que fait Claude
* **Modifier votre session** : réécrire un prompt ou un appel d'outil, soumettre un prompt comme si vous l'aviez tapé, ou envoyer un message à une autre de vos sessions
* **Agir sans vous demander** : approuver un appel d'outil avant que la question ne vous soit posée
* **Consommer votre usage** : appeler un modèle sur votre forfait ou votre clé API

Les mods ne sont pas sandboxés. Si vous activez le [sandboxing](https://code.claude.com/docs/en/sandboxing), le sandbox isole les commandes Bash exécutées par Claude, mais un processus lancé par un mod s'exécute en dehors.

Un mod qui approuve des appels d'outil peut approuver un appel pour lequel une règle `ask` aurait demandé confirmation, ou qu'un de vos propres hooks `PreToolUse` a bloqué. [Étendre les permissions avec des hooks](https://code.claude.com/docs/en/permissions#extend-permissions-with-hooks) liste ce qu'un tel mod peut approuver, y compris les cas où il peut approuver un appel refusé par une règle `deny`.

Un mod peut restyler une grande partie de l'interface de Claude Code, mais pas l'invite de permission (permission prompt). Il ne peut pas changer ce qu'une invite de permission vous affiche.

### Lister ce que fait un mod avant de l'installer

Avant d'installer un mod, vous pouvez lister les événements qu'il gère et ce qu'il demande à Claude Code de faire (lire un fichier, effectuer une requête réseau…), sans l'exécuter. Récupérez d'abord les fichiers du plugin, par exemple en clonant son dépôt. Puis, dans votre shell, exécutez `claude plugin validate` sur le répertoire du plugin :

```bash theme={null}
claude plugin validate ./some-mod
```

Les lignes `hooks:` et `calls:` de la sortie listent les événements que gère le mod et ce qu'il demande à Claude Code de faire. [Examiner ce qu'un mod peut faire](https://code.claude.com/docs/en/plugins/mods/admin#review-what-a-mod-can-do) montre la sortie et les appels à surveiller.

## Activer ou désactiver les mods

Les mods sont activés par défaut. Dans le terminal, utilisez Claude Code v2.1.287 ou ultérieure. L'application Desktop embarque sa propre copie de Claude Code, et les mods y fonctionnent à partir de la v2.1.286. Vérifiez la version là où vous utilisez les mods :

* **Terminal** : dans votre shell, exécutez `claude --version`. Si la vôtre est plus ancienne, [mettez Claude Code à jour](https://code.claude.com/docs/en/setup#update-claude-code).
* **Application Desktop** : dans une session locale de l'onglet Code, saisissez `/status` et lisez la ligne **Claude Code**, qui affiche une version comme `2.1.286`. Si la vôtre est plus ancienne, [mettez l'application Desktop à jour](https://code.claude.com/docs/en/desktop#claude-code-version-in-the-code-tab).

Pour désactiver les mods, choisissez combien en arrêter et pour combien de temps. Pour les réactiver, annulez le même changement :

* **Un seul mod** : désactivez ou désinstallez son plugin depuis l'[onglet **Installed** de `/plugin`](https://code.claude.com/docs/en/plugins/install#manage-installed-plugins)
* **Tous les mods installés, pour une session** : démarrez Claude Code avec [`--safe-mode`](https://code.claude.com/docs/en/cli-reference#cli-flags), qui désactive aussi vos autres personnalisations
* **Tous les mods que vous avez installés, dans toutes les sessions** : définissez [`"disableAllHooks": true`](https://code.claude.com/docs/en/settings-reference#disableallhooks) dans `~/.claude/settings.json`. Vos hooks de réglages et votre ligne de statut personnalisée s'arrêtent aussi. Ce que gère votre organisation continue de s'exécuter.

Si vous utilisez Claude Code via une organisation, un administrateur peut aussi limiter les mods qui se chargent. Les administrateurs commencent par [Empêcher le chargement des mods installés par les utilisateurs](https://code.claude.com/docs/en/plugins/mods/admin#stop-user-installed-mods-from-loading).

`disableAllHooks` et le réglage `allowManagedModsOnly` de votre organisation arrêtent un mod mais laissent le reste de son plugin en place : le plugin reste installé, et ses skills, commandes, agents et serveurs MCP se chargent. D'autres réglages et flags vont plus loin. [`disableAllHooks`](https://code.claude.com/docs/en/settings-reference#disableallhooks) et [Ce qui s'exécute sous `allowManagedHooksOnly`](https://code.claude.com/docs/en/settings-reference#what-runs-under-allowmanagedhooksonly) listent l'effet de chacun sur un plugin et ses hooks de réglages.

Pour savoir si les mods peuvent se charger chez vous, voir [Vérifier si les mods peuvent se charger](https://code.claude.com/docs/en/plugins/mods/troubleshoot#check-whether-mods-can-load).

> **Note** : si vous avez défini `CLAUDE_CODE_ENABLE_FUNCTION_HOOKS` pendant l'accès anticipé, supprimez-la. Claude Code v2.1.287 et ultérieures l'ignorent : la mettre à `0` ne maintient donc pas les mods désactivés.

### Voir quels mods une session a chargés

Pour voir quels mods une session de terminal a chargés, exécutez `/plugin` au prompt de Claude Code. Une ligne grisée sous les onglets donne le nombre et les noms, par exemple `1 mod active · first-mod`. Si un mod que vous avez installé n'y figure pas, voir [Comprendre pourquoi un mod ne fait rien](https://code.claude.com/docs/en/plugins/mods/troubleshoot#find-out-why-a-mod-does-nothing).

## Fonctionnement d'un mod

Un mod est un [plugin](https://code.claude.com/docs/en/plugins/overview) dont le code enregistre des gestionnaires d'événements, appelés hooks. Claude Code exécute un hook quand son événement se produit, par exemple quand Claude appelle un outil ou quand le spinner est dessiné. Un petit mod comporte trois fichiers :

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
* **`register.js`** : [votre code](https://code.claude.com/docs/en/plugins/mods/create#write-a-mod-yourself), appelé le module de hooks (hooks module). Il indique à Claude Code sur quels événements exécuter vos fonctions.

Voici un `register.js` complet. Il compte les appels d'outil que fait Claude et affiche le compte à côté du spinner pendant que Claude travaille, par exemple `Thinking · tool calls: 3…`.

```javascript hooks/register.js theme={null}
// Le compteur, partagé par les deux hooks ci-dessous
let calls = 0

// Claude Code appelle cette fonction une fois, au chargement du mod
export function register(on) {
  // S'exécute chaque fois que Claude s'apprête à utiliser un outil
  on('tool.call', async ($, e, next) => {
    calls += 1
    // Demande à Claude Code de redessiner l'interface, pour afficher le nouveau compte
    $.ui.invalidate('ui.render')
    // Laisse l'outil s'exécuter normalement
    return next(e)
  })

  // S'exécute chaque fois que Claude Code dessine le spinner
  on('ui.render', { component: 'Spinner' }, async ($, e, next) => {
    // Garde le spinner de Claude Code, avec le compte ajouté après son mot
    return next({ ...e, props: { ...e.props, suffix: ' · tool calls: ' + calls + '…' } })
  })
}
```

Le fichier enregistre deux hooks, et tous deux utilisent la variable `calls` définie en haut :

* **Le hook [`tool.call`](https://code.claude.com/docs/en/plugins/mods/reference#tools)** s'exécute chaque fois que Claude s'apprête à utiliser un outil. Il ajoute un à `calls`, demande à Claude Code de redessiner l'interface et laisse l'outil s'exécuter normalement.
* **Le hook [`ui.render`](https://code.claude.com/docs/en/plugins/mods/reference#interface)** s'exécute chaque fois que Claude Code dessine le spinner. Il conserve le spinner propre de Claude Code et ajoute le compte après le mot.

L'enregistrement vidéo de la page montre le mod en action : sur la ligne du spinner au-dessus de la zone de prompt, pendant que Claude liste un répertoire et lit deux fichiers (prompt « list the files here and read the README »), on lit `Thinking · tool calls: 1…`, puis `2…`, puis `3…`.

### Ce qu'un hook peut faire d'un événement

Claude Code exécute votre hook **avant** d'agir sur l'événement, c'est donc le hook qui décide de la suite. Il peut :

* **Observer** : noter ce qui se passe et laisser l'événement continuer inchangé, comme le fait le hook `tool.call` de l'exemple
* **Réécrire** : modifier l'événement avant qu'il ne continue, comme le fait le hook `ui.render` en ajoutant le compte au spinner
* **Répondre** : traiter l'événement lui-même, de sorte que le comportement habituel ne s'exécute pas, par exemple refuser une commande

Pour faire quoi que ce soit en dehors de son propre code — dessiner, ajouter une commande, appeler un modèle, lire un fichier, lancer un processus ou effectuer une requête réseau —, un hook appelle l'API des mods (mods API). Un hook n'a pas d'autre moyen de faire ces choses, ce qui explique que Claude Code puisse [lister ce que fait un mod](#lister-ce-que-fait-un-mod-avant-de-linstaller) avant que vous ne l'installiez.

Pour le code correspondant à chaque choix, voir [Réagir aux événements](https://code.claude.com/docs/en/plugins/mods/events#how-a-hook-handles-an-event). Pour ce qu'un hook peut appeler, voir [Utiliser l'API des mods](https://code.claude.com/docs/en/plugins/mods/api).

### Où s'exécutent les mods

Les hooks d'un mod s'exécutent dans tout type de session qui charge le plugin, quel que soit le fournisseur qui sert le modèle, y compris Amazon Bedrock, l'Agent Platform de Google Cloud et Microsoft Foundry. Le dessin est plus restreint : seuls le terminal et l'application Desktop affichent les panneaux, bandes et lignes remplacées d'un mod. Ce tableau liste chaque endroit où vous pouvez exécuter Claude Code :

| Où vous exécutez Claude Code | Les hooks s'exécutent | Ce que dessine le mod apparaît |
| :- | :- | :- |
| `claude` dans un terminal, y compris le terminal intégré d'un éditeur et le plugin JetBrains | Oui | Oui |
| L'onglet Code de l'application Desktop, sauf dans une session WSL | Oui | Oui, sauf les éléments que le [tableau des éléments](https://code.claude.com/docs/en/plugins/mods/reference#elements) marque comme réservés au terminal |
| Une [session WSL](https://code.claude.com/docs/en/desktop-wsl) dans l'application Desktop | Non, car les plugins ne sont pas disponibles dans les sessions WSL | Non |
| Le panneau de chat de l'extension VS Code | Oui | Non |
| `claude -p` et l'[Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview) | Oui | Non |
| [Remote Control](https://code.claude.com/docs/en/remote-control) depuis claude.ai ou l'application mobile | Oui, dans la session sur votre machine | Dans le terminal de votre machine |
| Une [session cloud](https://code.claude.com/docs/en/claude-code-on-the-web) | Oui, pour un plugin qui [parvient jusqu'à la session cloud](https://code.claude.com/docs/en/cloud-environments#what-carries-over-from-your-setup) | Non |

Un mod qui dessine peut vérifier dans quelle application il s'exécute, et se rabattre sur une ligne dans la transcription ou sur la réponse texte d'une commande là où rien ne se dessine.

## Contrôler les mods pour votre organisation

Les administrateurs décident si les mods s'exécutent, et lesquels, via les [réglages gérés (managed settings)](https://code.claude.com/docs/en/managed-settings). [Gérer les mods pour votre organisation](https://code.claude.com/docs/en/plugins/mods/admin) couvre le comportement par défaut, la manière d'examiner un mod et l'application d'une politique avec un mod de votre cru.

## Comparer mods, hooks de réglages, skills et serveurs MCP

Mods, hooks de réglages, skills et serveurs MCP se recoupent. Ce tableau montre ce qu'est chacun et quand le choisir.

| | Mod | Hook de réglages | Skill | Serveur MCP |
| :- | :- | :- | :- | :- |
| Ce que c'est | Des fonctions dans un plugin, que Claude Code appelle dans son propre processus | Une commande shell, une requête HTTP ou un prompt que Claude Code exécute sur un événement du cycle de vie | Un fichier `SKILL.md` d'instructions que Claude lit | Un processus ou service externe qui fournit des outils à Claude |
| Ce qu'il peut changer | Appels d'outil, prompts, commandes, tours, et ce que dessine l'interface | Si un appel d'outil ou un prompt est autorisé à continuer, les arguments et le résultat d'un appel d'outil, et le contexte ajouté pour Claude | Ce que Claude sait et fait | Les outils dont dispose Claude |
| Peut-il dessiner dans l'interface | Oui | Non | Non | Non |
| Ce que vous écrivez | JavaScript ou TypeScript | Un script et une entrée dans `settings.json` | Du Markdown | Un serveur dans n'importe quel langage |
| Choisissez-le quand | Vous voulez un panneau, une bande au-dessus du prompt, une commande personnalisée, ou réécrire un événement | Vous voulez bloquer, autoriser ou journaliser un événement avec un script que vous avez déjà | Vous collez sans cesse les mêmes instructions dans le chat | Claude doit atteindre un système externe |

Chacun des autres a sa propre page : [Hooks](https://code.claude.com/docs/en/hooks), [Skills](https://code.claude.com/docs/en/skills) et [MCP](https://code.claude.com/docs/en/mcp). Un plugin peut tous les contenir : un mod peut donc être livré dans le même plugin qu'une skill et un serveur MCP.

## Mods intégrés à Claude Code

Certaines fonctionnalités de Claude Code sont elles-mêmes des mods. Pour voir ceux de votre session, exécutez `/plugin` au prompt de Claude Code et allez dans l'onglet **Installed**, qui les liste sous **Built-in**. Vous ne pouvez ni mettre à jour ni désinstaller un mod intégré ; la dernière colonne du tableau indique comment désactiver chacun. La [ligne `mods active`](#voir-quels-mods-une-session-a-chargés) omet les mods intégrés.

Ce tableau liste chaque entrée sous le nom qu'affiche `/plugin` :

| Nom dans `/plugin` | Ce qu'il fait | Où il est actif | Comment le désactiver |
| :- | :- | :- | :- |
| `cc-plugin-agents-md` | Charge `AGENTS.md` comme instructions de projet | Chaque session, sauf [celles qui ne peuvent pas lire `AGENTS.md`](https://code.claude.com/docs/en/memory#when-agents-md-support-is-unavailable) | Le désactiver dans `/plugin`, ou [choisir quels fichiers d'instructions se chargent](https://code.claude.com/docs/en/memory#choose-which-instruction-files-load) |
| `cc-plugin-diff` | Prend en charge [`/diff`](https://code.claude.com/docs/en/interactive-mode#review-changes-with-%2Fdiff) et dessine son panneau | Sessions interactives dans le terminal | Le désactiver dans `/plugin`. `/diff` reste disponible, et c'est la version intégrée de la commande de Claude Code qui répond. |
| `cc-plugin-plugin-authoring` | Donne à Claude la [skill `plugin-authoring`](https://code.claude.com/docs/en/plugins/mods/create#ask-claude-for-a-mod) pour écrire des mods. Il contient une skill et aucun code de mod. | Sauf si Anthropic a désactivé à distance les mods installés | Le désactiver dans `/plugin` |
| `cc-plugin-sec-default` | Protège ce que gère votre organisation contre les mods qu'installe un utilisateur | [Là où la protection se charge](https://code.claude.com/docs/en/plugins/mods/admin#know-what-happens-by-default) | Impossible. Un administrateur [fixe l'ordre](https://code.claude.com/docs/en/plugins/mods/admin#install-your-organizations-mods) dans les réglages gérés |
| `cc-plugin-telemetry` | Envoie les enregistrements analytiques que journalisent Claude Code et ses mods intégrés | Partout où les analytics propres de Claude Code sont actives | Le désactiver dans `/plugin`, ou couper les analytics, par exemple avec [`DISABLE_TELEMETRY`](https://code.claude.com/docs/en/env-vars) |
| `cc-plugin-you-should-know` | [You should know](https://code.claude.com/docs/en/interactive-mode#you-should-know-suggestions) : un agent annexe affiche une note au-dessus du prompt quand il trouve quelque chose qui a pu vous échapper | Désactivé tant que vous ne l'[activez](https://code.claude.com/docs/en/interactive-mode#you-should-know-suggestions) pas | Le désactiver dans `/plugin` |

Les réglages et flags qui arrêtent les mods installés, comme `disableAllHooks`, `--bare` et `--safe-mode`, n'arrêtent pas les mods intégrés.

### Lire le code source des mods intégrés

Le code source de certains de ces mods est public dans le [répertoire `mods` du dépôt Claude Code](https://github.com/anthropics/claude-code/tree/main/mods). Chacun est un plugin complet avec son module de hooks et ses tests :

* [`diff`](https://github.com/anthropics/claude-code/tree/main/mods/diff) : le panneau `/diff`, avec des boutons liés à des actions clavier et un défilement que le mod gère lui-même
* [`agents-md`](https://github.com/anthropics/claude-code/tree/main/mods/agents-md) : charge `AGENTS.md` comme instructions de projet, avec une option [`userConfig`](https://code.claude.com/docs/en/plugins/components#user-configuration)
* [`sec-default`](https://github.com/anthropics/claude-code/tree/main/mods/sec-default) : la protection décrite dans [Savoir ce qui se passe par défaut](https://code.claude.com/docs/en/plugins/mods/admin#know-what-happens-by-default), un modèle de mod qui applique une politique
* [`telemetry`](https://github.com/anthropics/claude-code/tree/main/mods/telemetry) : ajoute des méthodes que d'autres mods peuvent appeler, et livre leurs types

## Étapes suivantes

* [Créer un mod](https://code.claude.com/docs/en/plugins/mods/create) : en construire un qui compte les appels d'outil, affiche le compte à côté du spinner et ajoute une commande, et apprendre la boucle modification / rechargement
* [Dessiner dans l'interface](https://code.claude.com/docs/en/plugins/mods/interface) : panneaux, bande au-dessus du prompt, boutons, champs de texte et état
* [Réagir aux événements](https://code.claude.com/docs/en/plugins/mods/events) : appels d'outil, prompts, tours, et ordre d'exécution des mods
* [Utiliser l'API des mods](https://code.claude.com/docs/en/plugins/mods/api) : commandes, outils, appels de modèle, minuteries et fichiers
* [Tester un mod](https://code.claude.com/docs/en/plugins/mods/test) : tests automatisés qui s'exécutent sans session
* [Dépanner un mod](https://code.claude.com/docs/en/plugins/mods/troubleshoot) : les raisons pour lesquelles un mod ne fait rien, et le journal de débogage
* [Gérer les mods pour votre organisation](https://code.claude.com/docs/en/plugins/mods/admin) : valeurs par défaut, réglages gérés, examen d'un mod et mods de politique
* [Référence des mods](https://code.claude.com/docs/en/plugins/mods/reference) : événements, méthodes, éléments et limites
