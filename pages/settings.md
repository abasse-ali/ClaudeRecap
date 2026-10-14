# Fichiers de réglages et ordre de priorité

> Source : https://code.claude.com/docs/en/settings · [← Index](../31-index-documentation-officielle.md)

*Modifier les réglages de Claude Code, choisir la portée (scope) à laquelle appartient une clé, vérifier la modification, et savoir quelle valeur Claude Code utilise lorsqu'une clé est définie à plusieurs endroits.*

Les réglages (settings) sont les clés JSON qui changent le comportement de Claude Code : le modèle avec lequel il démarre, ce qu'il peut exécuter sans demander, les fichiers qu'il ne peut pas lire, son apparence dans le terminal, et ce que votre organisation impose.

> **Astuce** : pour chercher une clé précise, allez sur [All settings](https://code.claude.com/docs/en/settings-reference), qui liste chaque clé avec le fichier où la définir, sa valeur par défaut et un exemple.

Claude Code lit ses réglages dans des fichiers JSON comme `~/.claude/settings.json`. Il les cherche à plusieurs emplacements, et [le fichier dans lequel il lit un réglage détermine à qui ce réglage s'applique](#fichiers-de-réglages-et-qui-ils-concernent). Cette page couvre ces fichiers : dans lequel placer un réglage, comment modifier un réglage et confirmer qu'il s'applique, et quelle valeur Claude Code retient quand la même clé est définie dans plusieurs fichiers. [Configure permissions](https://code.claude.com/docs/en/permissions) explique ce que Claude Code peut exécuter sans demander et comment écrire les règles `allow`, `ask` et `deny`.

> **Note** : cette page concerne Claude Code exécuté sur votre machine : le terminal, les extensions [VS Code](https://code.claude.com/docs/en/vs-code) et [JetBrains](https://code.claude.com/docs/en/jetbrains), et l'[application desktop](https://code.claude.com/docs/en/desktop), qui lisent tous les mêmes fichiers de réglages. Une [session cloud](https://code.claude.com/docs/en/claude-code-on-the-web) s'exécute sur une autre machine et n'en lit qu'une partie ; voir [Réglages dans les sessions cloud](#réglages-dans-les-sessions-cloud).

## Fichiers de réglages et qui ils concernent

Claude Code lit ses réglages dans quatre fichiers, et une organisation peut aussi fournir des réglages gérés (managed settings) depuis la console claude.ai. Chaque source a une portée (scope) : l'ensemble des personnes et des projets auxquels s'applique un réglage enregistré dans cette source, que ce soit vous seul, tout le monde dans un projet, ou toute votre organisation.

| Portée | Fichier | Qui est concerné | À utiliser pour |
| :- | :- | :- | :- |
| User (utilisateur) | `~/.claude/settings.json` | Vous, dans tous les projets de cette machine | Préférences personnelles : thème, mode d'édition, modèle par défaut, vos propres règles de permission |
| Shared project (projet partagé) | `.claude/settings.json` | Toute personne travaillant dans le dossier qui le contient. Dans un dépôt git, committez-le pour que vos coéquipiers l'obtiennent | Permissions d'équipe, hooks, plugins et variables d'environnement nécessaires au projet |
| Project local (projet local) | `.claude/settings.local.json` | Vous, dans ce seul projet. Claude Code le tient hors de git lorsqu'il crée le fichier ; si vous le créez à la main, ajoutez-le vous-même à `.gitignore` | Surcharges personnelles pour un projet, et tests avant de partager |
| Managed (géré) | `managed-settings.json` et les autres [sources gérées](https://code.claude.com/docs/en/managed-settings#delivery-mechanisms) | Toutes les personnes auxquelles votre organisation le déploie ; [Ordre de priorité des réglages](#ordre-de-priorité-des-réglages) indique ce qui peut le surcharger | Politique de sécurité et exigences de conformité |

Dans la colonne Fichier, `~/.claude` désigne le dossier `.claude` de votre répertoire personnel (home), et un `.claude` seul désigne le dossier `.claude` à l'intérieur de votre projet.

### Comparer la portée de chaque fichier de réglages

Supposons que vous ayez trois projets sur votre machine, `website/`, `api/` et `acme-app/`, qu'un coéquipier ait son propre clone de `acme-app/`, et que vous lanciez une [session cloud](#réglages-dans-les-sessions-cloud) sur `acme-app/`.

Le graphique interactif de la page officielle montre dans lesquels de ces dossiers un réglage s'applique quand vous lancez Claude Code depuis eux (on clique sur un fichier de réglages pour voir les dossiers qu'il atteint). Contenu du graphique :

| Fichier sélectionné | Dossiers atteints | Où se trouve le fichier |
| :- | :- | :- |
| `~/.claude/settings.json` | `website/`, `api/` et votre `acme-app/` (votre machine) | Sur votre machine |
| `acme-app/.claude/settings.json` | Votre `acme-app/`, le clone du coéquipier (« leur clone, une fois que vous avez committé le fichier ») et la session cloud (« clone neuf, une fois que vous avez committé le fichier ») | Dans chaque `acme-app/` |
| `acme-app/.claude/settings.local.json` | Votre `acme-app/` uniquement | Dans votre `acme-app/` |
| Réglages gérés (`managed-settings.json`, MDM, ou la console claude.ai) | Tous les dossiers : `website/`, `api/`, votre `acme-app/`, celui du coéquipier et la session cloud (pour celle-ci : « server-managed uniquement ») | Fourni par l'organisation |

En détail :

* **`~/.claude/settings.json`** : tous les projets de votre machine, et rien chez votre coéquipier ni dans la session cloud.
* **`acme-app/.claude/settings.json`** : votre `acme-app/`. Il n'atteint le clone de votre coéquipier et la session cloud que si vous committez le fichier dans le contrôle de version ; tant que vous ne l'avez pas fait, c'est un fichier sur votre disque comme un autre et personne d'autre ne l'a.
* **`acme-app/.claude/settings.local.json`** : votre `acme-app/` uniquement. Claude Code l'ajoute à vos exclusions git globales la première fois qu'il écrit le fichier, afin qu'il reste hors de vos commits ; si vous créez le fichier à la main, [ajoutez-le vous-même à `.gitignore`](#garder-les-réglages-personnels-hors-dun-dépôt).
* **Réglages gérés**, qu'il s'agisse d'un fichier `managed-settings.json`, d'une politique MDM ou de [server-managed settings](https://code.claude.com/docs/en/server-managed-settings) depuis la console claude.ai : tous les projets de toutes les machines sur lesquelles votre organisation le déploie, ou sur lesquelles vous vous connectez avec votre compte d'organisation. Seuls les server-managed settings atteignent la session cloud.

### Trouver ou créer vos fichiers de réglages

L'installation de Claude Code ne crée aucun fichier de réglages. Si votre machine ou votre projet en possède déjà un, il provient de l'une de ces sources :

* **Managed** : votre organisation le déploie. Vous ne le créez pas et ne le modifiez pas.
* **Shared project** : un projet qui utilise déjà Claude Code peut en avoir un committé. Sinon, créez-le dans `.claude/settings.json` dans le dossier du projet.
* **User** et **Project local** : créez-les vous-même, ou laissez Claude Code les créer. Il écrit `~/.claude/settings.json` la première fois que vous changez dans le menu `/config` une option qu'il stocke dans les réglages utilisateur, comme le thème, et `.claude/settings.local.json` la première fois que vous donnez une approbation permanente sur une demande de permission, comme « Yes, and don't ask again » pour une commande Bash. Quelques options de `/config`, dont **Show tips**, sont enregistrées dans `.claude/settings.local.json` au lieu du fichier utilisateur.

> **Info** : sous Windows, `~/.claude` signifie `%USERPROFILE%\.claude`. Pour placer les fichiers du répertoire personnel ailleurs, définissez [`CLAUDE_CONFIG_DIR`](https://code.claude.com/docs/en/env-vars) ; Claude Code y stocke alors vos réglages, votre historique de sessions et vos plugins.

Claude Code tient aussi un cinquième fichier, [`~/.claude.json`](https://code.claude.com/docs/en/claude-directory#ce-claude-json), qu'il écrit pour lui-même ; vous n'avez pas besoin de le modifier. Il contient votre session de connexion, les configurations de [serveurs MCP](https://code.claude.com/docs/en/mcp), l'état par projet comme les décisions de confiance (trust), et les [clés de configuration globale](https://code.claude.com/docs/en/settings-reference#global-config-settings) que `/config` écrit pour vous.

### Partager des réglages avec votre équipe

Committez `.claude/settings.json` pour que toute personne qui clone le dépôt obtienne les mêmes permissions, hooks et plugins. Chaque coéquipier peut toujours le surcharger pour lui-même dans son propre `.claude/settings.local.json`, donc les exceptions personnelles ne nécessitent pas de commit. Pour un fichier d'équipe complet, voir [les réglages partagés d'une équipe](https://code.claude.com/docs/en/settings-example#a-teams-shared-settings).

Une partie de ce que vous committez attend que chaque coéquipier [fasse confiance au dossier](https://code.claude.com/docs/en/permissions#project-allow-rules-and-workspace-trust), et quelques clés ne prennent jamais effet depuis un fichier de dépôt ; [Dépanner un réglage qui ne s'applique pas](#dépanner-un-réglage-qui-ne-sapplique-pas) couvre les deux cas.

### Garder les réglages personnels hors d'un dépôt

Pour modifier un réglage pour vous seul dans un projet sans le modifier pour vos coéquipiers, enregistrez-le dans `.claude/settings.local.json` à l'intérieur du projet. Claude Code applique ce fichier par-dessus le `.claude/settings.json` committé : si le fichier de votre équipe définit `"model": "claude-sonnet-5"` et que vous voulez Opus, mettez `"model": "claude-opus-5-5"` dans votre fichier local et seules vos sessions changent.

Claude Code écrit aussi dans ce fichier, le tient hors de vos commits et applique ses règles allow sans l'étape de confiance :

* **Claude Code l'écrit aussi.** Quand Claude demande la permission d'exécuter une commande Bash et que vous choisissez « Yes, and don't ask again », Claude Code enregistre cette [approbation de permission](https://code.claude.com/docs/en/permissions#permission-system) ici sous forme de règle `allow`.
* **Vous n'avez pas besoin de l'ignorer dans git vous-même, sauf si vous l'avez créé à la main.** La première fois que Claude Code écrit le fichier dans un dépôt git qui ne l'ignore pas déjà, il ajoute `**/.claude/settings.local.json` à votre fichier d'exclusions git global, si bien que le fichier reste hors de vos commits dans tous les dépôts. Ce fichier est `core.excludesFile` lorsque votre config git globale le définit sur un chemin absolu ou préfixé par `~` ; sinon c'est `$XDG_CONFIG_HOME/git/ignore`, ou `~/.config/git/ignore` quand `XDG_CONFIG_HOME` n'est pas défini. Si vous avez créé le fichier à la main et que Claude Code n'y a pas encore écrit, ajoutez-le vous-même à `.gitignore`.
* **Ses règles allow n'attendent pas la confiance tant que le fichier reste non suivi (untracked).** Comme le fichier vous appartient et n'appartient pas au dépôt, Claude Code applique ses règles `allow` sans l'étape de [confiance de l'espace de travail (workspace trust)](https://code.claude.com/docs/en/permissions#project-allow-rules-and-workspace-trust) qu'il exige pour le fichier committé. Si le fichier est suivi par git, l'étape de confiance s'applique à lui aussi ; voir [When your local settings file needs trust](https://code.claude.com/docs/en/permissions#when-your-local-settings-file-needs-trust).

#### Où Claude Code place le fichier local dans un dépôt git

Quand Claude demande la permission d'exécuter une commande Bash et que vous choisissez « Yes, and don't ask again », Claude Code enregistre cette approbation comme règle `allow` dans `.claude/settings.local.json`. Si vous lancez Claude Code dans un sous-répertoire d'un dépôt git, il lit et écrit ce fichier à la racine du dépôt et applique l'approbation à l'ensemble du dépôt. Dans un [worktree](https://code.claude.com/docs/en/worktrees), il utilise le fichier situé à la racine du checkout principal.

Deux règles nuancent cet emplacement à la racine :

* **Quand le fichier reste à côté de `.claude/settings.json`** : hors d'un dépôt git, quand la racine du dépôt est votre répertoire personnel, sous Windows, ou quand la racine du dépôt ou son entrée `.git` ou `.claude` n'appartient pas à votre utilisateur.
* **Les chemins du fichier ne sont pas ancrés à la racine du dépôt** : une règle de permission qui commence par `/` ou un chemin de sandbox relatif [s'ancre au répertoire de travail principal de la session](https://code.claude.com/docs/en/permissions#read-and-edit).

Avant la v2.1.211, Claude Code gardait le fichier dans le répertoire de démarrage. Il lit encore un fichier qu'une version antérieure y a laissé, en plus du fichier à la racine ; lorsque les deux définissent la même clé, la valeur de la racine s'applique, et les règles de permission des deux fichiers s'appliquent. L'utilitaire [`resolveSettings()`](https://code.claude.com/docs/en/agent-sdk/typescript#resolvesettings) de l'Agent SDK lit toujours le fichier dans le répertoire de démarrage.

Claude Code lit le `.claude/settings.json` partagé depuis le [répertoire de travail principal](https://code.claude.com/docs/en/permissions#working-directories) de la session ; pour utiliser un fichier committé à la racine du dépôt, lancez donc Claude Code depuis cette racine. Après avoir [déplacé la session avec `/cd`](https://code.claude.com/docs/en/permissions#move-the-session-to-another-directory), Claude Code lit les deux fichiers de projet depuis le nouveau répertoire, en plaçant le fichier local selon les mêmes règles. Les lire depuis le répertoire vers lequel vous vous êtes déplacé nécessite la v2.1.246+. Pour une session worktree lancée depuis l'application desktop, voir [What worktrees share with the main checkout](https://code.claude.com/docs/en/worktrees#what-worktrees-share-with-the-main-checkout).

### Vérifier ce que votre organisation impose

Si votre organisation gère Claude Code, certains réglages sont décidés pour vous et rien de ce que vous mettez dans vos propres fichiers ne les modifie. Pour savoir lesquels, lancez `/status` : la ligne `Setting sources` nomme la source gérée qui s'applique à vous. Les réglages gérés s'appliquent partout où Claude Code s'exécute sur cette machine ; [What a developer can change](https://code.claude.com/docs/en/managed-settings#what-a-developer-can-change) traite des droits d'administrateur local et des outils autres que Claude Code.

Les réglages gérés vous parviennent par les [mécanismes de distribution](https://code.claude.com/docs/en/managed-settings#delivery-mechanisms) décrits sur la page des réglages gérés, le plus souvent :

* Les [server-managed settings](https://code.claude.com/docs/en/server-managed-settings), que Claude Code récupère depuis la console d'administration claude.ai ou une [Claude apps gateway](https://code.claude.com/docs/en/claude-apps-gateway) auto-hébergée
* Des politiques MDM ou au niveau de l'OS, et des fichiers `managed-settings.json` dans un répertoire système
* Un hôte qui embarque Claude Code, comme Claude Desktop, via l'option SDK `managedSettings` ; voir [Control policy from an embedding host](https://code.claude.com/docs/en/managed-settings#parent-settings-from-embedding-hosts)

Dans une session [Cowork](https://claude.com/docs/cowork/overview) exécutée sur votre machine dans l'application Claude Desktop, Claude Code ne récupère pas les server-managed settings de la console d'administration claude.ai, et il lit la politique déployée sur votre appareil sauf si la configuration Claude Desktop de votre organisation définit `requireCoworkFullVmSandbox`. [Where and when a policy applies](https://code.claude.com/docs/en/managed-settings#where-and-when-a-policy-applies) couvre les sessions Cowork et cloud.

Si vous êtes l'administrateur, [Set up Claude Code for your organization](https://code.claude.com/docs/en/admin-setup) explique comment choisir ce qu'il faut imposer, et [Deploy managed settings](https://code.claude.com/docs/en/managed-settings) couvre la distribution et la manière de confirmer qu'une politique est en vigueur. Pour l'avertissement que peut afficher l'éditeur de réglages gérés de la console d'administration claude.ai, voir [Configure server-managed settings](https://code.claude.com/docs/en/server-managed-settings#configure-server-managed-settings).

## Modifier un réglage

Vous pouvez modifier un réglage depuis le menu `/config`, en éditant un fichier de réglages, ou pour une seule session depuis la ligne de commande.

Le prompt système de Claude Code n'est pas publié. Pour donner des instructions permanentes à Claude, utilisez les [fichiers `CLAUDE.md`](https://code.claude.com/docs/en/memory) ou le flag `--append-system-prompt`.

### Utiliser le menu /config

Lancez `/config` dans Claude Code et ouvrez l'onglet **Config**. Il liste un petit ensemble d'options personnelles comme le thème, le mode d'édition et la sortie verbeuse, pas toutes les clés de réglages. Sélectionnez une option pour la modifier ; Claude Code l'enregistre pour vous :

* **La plupart des options** : `~/.claude/settings.json`
* **Quelques options, comme Show tips** : `.claude/settings.local.json`
* **Les [options de configuration globale](https://code.claude.com/docs/en/settings-reference#global-config-settings)** : `~/.claude.json`

Pour définir une option sans passer par le menu, passez `key=value`, par exemple `/config verbose=true`.

> **Note** : `/config` fait partie de l'interface terminal. Le panneau de chat [VS Code](https://code.claude.com/docs/en/vs-code) et l'[application desktop](https://code.claude.com/docs/en/desktop) ne l'ouvrent pas ; modifiez-y les réglages en éditant un fichier de réglages ou via les propres réglages de ces applications.

### Éditer un fichier de réglages

Ouvrez dans votre éditeur le fichier de réglages de la portée voulue et ajoutez ou modifiez une clé. Les fichiers de réglages sont du JSON strict : un commentaire `//` ou une virgule finale est une erreur de syntaxe, et Claude Code signale le fichier comme une [Settings Error](#réparer-un-fichier-de-réglages-cassé) au démarrage suivant. Par exemple, pour laisser Claude Code exécuter vos commandes de lint et de test sans demander et l'empêcher de lire les fichiers `.env`, ajoutez ceci à `~/.claude/settings.json` :

```json ~/.claude/settings.json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)"
    ]
  }
}
```

Chaque entrée sous `permissions` est une règle qui nomme un outil et ce qu'il peut faire ; [Configure permissions](https://code.claude.com/docs/en/permissions) en explique la syntaxe. La ligne `$schema` pointe vers le [schéma JSON publié](https://json.schemastore.org/claude-code-settings.json) des réglages de Claude Code, qui vous donne l'autocomplétion et la validation en ligne dans VS Code, Cursor et tout autre éditeur prenant en charge JSON Schema. Le schéma peut être en retard sur les dernières versions de la CLI : un avertissement de validation sur une clé récemment documentée ne signifie donc pas que votre configuration est invalide.

Après avoir enregistré, lancez `/status` dans Claude Code pour confirmer que le fichier a été chargé ; [Confirmer ce qui a été chargé](#confirmer-ce-qui-a-été-chargé) explique ce qu'affiche la ligne `Setting sources` et comment un fichier cassé est signalé.

Pour un fichier personnel, un fichier d'équipe et un fichier d'organisation complets, chacun avec un commentaire sur chaque clé qu'il définit, voir les [exemples de fichiers de réglages](https://code.claude.com/docs/en/settings-example).

### Modifier un réglage pour une seule session

Pour essayer une valeur sans l'enregistrer, définissez-la au lancement de Claude Code. La valeur s'applique à cette session et vos fichiers de réglages restent inchangés. Trois façons de le faire :

* **`--settings`** : passez une clé en JSON, en ligne ou sous forme de chemin vers un fichier. Claude Code l'applique au-dessus de vos fichiers user, project et local et en dessous des réglages gérés. Il peut définir toute clé que votre fichier de réglages utilisateur peut définir ; il ne peut pas définir les clés `Managed` ou `Global config`.
* **Un flag dédié à cette clé** : certaines clés ont leur propre flag, comme `--model` pour `model` et `--effort` pour `effortLevel` et `modelSettings`.
* **Une variable d'environnement** : exportez la variable associée à la clé avant de lancer `claude`, comme `ANTHROPIC_MODEL` pour `model`.

L'entrée de chaque clé dans la [référence des réglages](https://code.claude.com/docs/en/settings-reference) liste ses surcharges par session et celle qui l'emporte : vérifiez l'entrée de la clé que vous voulez modifier.

Les commandes que vous lancez dans une session enregistrent le plus souvent votre choix : quand vous modifiez un réglage dans `/config`, Claude Code l'écrit dans vos fichiers de réglages, et `/model` enregistre la valeur comme défaut pour les nouvelles sessions.

Si vous appuyez sur `s` dans le sélecteur `/model`, Claude Code change de modèle sans l'enregistrer comme défaut utilisateur. [Adjust effort level](https://code.claude.com/docs/en/model-config#adjust-effort-level) indique quels choix de `/effort` Claude Code enregistre comme défaut pour le modèle utilisé et lesquels ne s'appliquent qu'à la session en cours.

Par exemple, pour lancer une session sur Opus sans changer votre défaut :

```bash
claude --settings '{"model": "claude-opus-5-5"}'
```

### Quand les modifications prennent effet

Claude Code surveille vos fichiers de réglages et les recharge lorsqu'ils changent : il applique donc la plupart des modifications à la session en cours sans redémarrage, y compris les modifications de `permissions`, de `hooks` et des assistants d'identifiants (credential helpers) comme `apiKeyHelper`. Claude Code charge aussi un fichier de réglages créé en cours de session si son dossier existait au démarrage de la session. Pour le dossier `.claude/` du projet, il charge le fichier même si vous créez le dossier dans la même session.

Le rechargement couvre les réglages user, project, local et managed, et Claude Code exécute le [hook `ConfigChange`](https://code.claude.com/docs/en/hooks#configchange) pour chaque modification de fichier de réglages qu'il détecte, mais pas pour les réglages gérés qui arrivent par MDM ou depuis la console claude.ai. Les réglages gérés arrivant par MDM ou depuis la console claude.ai atteignent une session en cours selon un calendrier plutôt qu'à l'enregistrement ; le [tableau de distribution](https://code.claude.com/docs/en/managed-settings#choose-a-delivery-mechanism) le précise par source.

Claude Code ne lit certaines clés qu'une fois, au démarrage de la session : une modification de l'une d'elles n'atteint donc pas la session en cours. Les clés côté administrateur qui attendent aussi un redémarrage, comme `requiredMinimumVersion`, sont listées dans [where and when a policy applies](https://code.claude.com/docs/en/managed-settings#where-and-when-a-policy-applies). Celles que vous êtes le plus susceptible de modifier en cours de session :

* [`model`](https://code.claude.com/docs/en/settings-reference#model) : utilisez [`/model`](https://code.claude.com/docs/en/model-config#setting-your-model) pour changer en cours de session. Chaque modèle a son propre cache de prompt, donc la première requête après un changement relit toute la conversation sans cache ; voir [Switching models](https://code.claude.com/docs/en/prompt-caching#switching-models).
* [`effortLevel`](https://code.claude.com/docs/en/settings-reference#effortlevel) et [`modelSettings`](https://code.claude.com/docs/en/settings-reference#modelsettings) : utilisez [`/effort`](https://code.claude.com/docs/en/model-config#adjust-effort-level) pour changer l'effort en cours de session.

### Confirmer ce qui a été chargé

Lancez `/status` dans Claude Code pour voir quelles sources de réglages sont actives. L'onglet **Status** comporte une ligne `Setting sources` qui liste chaque fichier de réglages que Claude Code a chargé pour la session en cours, comme `User settings` ou `Project local settings`. Quand des [réglages gérés](https://code.claude.com/docs/en/admin-setup#decide-how-settings-reach-devices) sont en vigueur, l'entrée des réglages gérés indique entre parenthèses comment ils sont arrivés sur votre machine.

Cette ligne confirme quels fichiers Claude Code a lus ; elle n'indique pas quel fichier a fourni chaque clé. Pour lister les entrées que Claude Code a rejetées, lancez [`claude doctor`](https://code.claude.com/docs/en/debug-your-config) ; pour un modèle défini par les réglages project ou managed, l'en-tête de démarrage nomme le fichier qui l'a défini. `/status` et `/config` ouvrent la même boîte de dialogue sur des onglets différents, et l'onglet **Config** n'est pas une vue du contenu de votre `settings.json`.

### Réparer un fichier de réglages cassé

Si vous faites une faute de frappe dans le JSON ou attribuez à une clé une valeur que Claude Code n'accepte pas, Claude Code vous le signale au démarrage d'une session interactive. Ce qu'il affiche dépend de la part du fichier touchée :

* **Settings Error** : un fichier user, project ou local contient du JSON invalide ou une valeur rejetée par le schéma. Au démarrage d'une session interactive, Claude Code affiche une boîte de dialogue qui vous permet de corriger le fichier avec l'aide de Claude, de quitter, ou de continuer sans les réglages cassés.
* **Settings Warning** : seules certaines entrées échouent, comme une règle de permission mal formée ou un nom d'événement de hook inconnu. Claude Code ignore ces valeurs et garde le reste du fichier en vigueur.
* **Réglages gérés** : Claude Code continue d'appliquer le reste du fichier. [Invalid entries in managed settings](https://code.claude.com/docs/en/managed-settings#invalid-entries-in-managed-settings) indique ce qu'il écarte et quelles clés retombent sur une valeur plus stricte jusqu'à correction. Pour un document de réglages gérés qui n'est pas du JSON valide, voir [Managed settings document could not be parsed](https://code.claude.com/docs/en/errors#managed-settings-document-could-not-be-parsed).
* **Configuration error** : `~/.claude.json` ne peut pas être analysé. Claude Code copie le fichier cassé dans `~/.claude/backups/.claude.json.corrupted.<timestamp>` et demande s'il faut quitter pour le corriger à la main ou réinitialiser à la configuration par défaut ; une exécution `-p` affiche l'erreur et quitte. Pour récupérer votre état précédent, recopiez l'un des cinq fichiers `.claude.json.backup.<timestamp>` les plus récents de `~/.claude/backups/`, que Claude Code enregistre avant d'écrire le fichier.

Après avoir continué, lancez `/status` pour voir les fichiers concernés et `claude doctor` pour le détail de chaque erreur.

Une exécution `-p` n'affiche aucune boîte de dialogue. Sauf si [un document de réglages gérés ne peut pas être analysé](https://code.claude.com/docs/en/errors#managed-settings-document-could-not-be-parsed), Claude Code ignore le fichier ou les valeurs cassés et continue avec le reste ; après une exécution `-p` qui ignore un réglage, lancez donc `claude doctor` pour voir ce qui a été écarté.

## Ordre de priorité des réglages

Lorsque la même clé apparaît à plusieurs endroits, Claude Code utilise la valeur du niveau le plus élevé qui la définit. La pile ci-dessous montre les niveaux, le plus élevé en haut ; une clé à un niveau supérieur surcharge la même clé partout en dessous.

Contenu du schéma de la page officielle (priorité la plus haute en haut, flèche « overrides » = surcharge) :

| Niveau | Fichier / source | Qui |
| :- | :- | :- |
| 1. Managed settings | `managed-settings.json`, MDM, ou la console claude.ai | Votre organisation |
| 2. Command line | `claude --settings` | Vous, pour cette session |
| 3. Project local | `.claude/settings.local.json` | Vous, pour ce projet |
| 4. Shared project | `.claude/settings.json` | Tout le monde dans le projet |
| 5. User | `~/.claude/settings.json` | Vous, dans tous les projets |

Dans l'ordre, de la priorité la plus haute à la plus basse :

1. **Réglages gérés (managed)** : réglages que votre organisation déploie, par un fichier `managed-settings.json`, une politique MDM, ou des [server-managed settings](https://code.claude.com/docs/en/server-managed-settings) depuis la console claude.ai. Rien dans vos propres fichiers de réglages ni dans `--settings` ne surcharge une clé gérée, et un flag comme `--model` ne choisit que parmi les modèles que votre organisation autorise. Un [`model`](https://code.claude.com/docs/en/settings-reference#model) géré est un défaut de départ, pas un verrou ; les verrous sont [`availableModels`](https://code.claude.com/docs/en/settings-reference#availablemodels) et [`deniedModels`](https://code.claude.com/docs/en/settings-reference#deniedmodels). Quand votre organisation fournit plusieurs sources gérées, les règles de [priorité au sein du niveau géré](https://code.claude.com/docs/en/managed-settings#precedence-within-the-managed-tier) indiquent ce que Claude Code lit dans chacune.
2. **Ligne de commande** : le JSON que vous passez avec `--settings <file-or-json>` au lancement de `claude`, pour cette session uniquement ; voir [Modifier un réglage pour une seule session](#modifier-un-réglage-pour-une-seule-session). Une clé définie là surcharge la même clé dans vos fichiers de réglages project et user, et une clé que vous omettez conserve sa valeur issue de ces fichiers. Les autres flags, comme `--model`, définissent une seule chose pour la session et ne font pas partie de cette pile ; l'entrée d'une clé dans la [référence des réglages](https://code.claude.com/docs/en/settings-reference) indique quels flags la surchargent.
3. **Réglages projet locaux** (`.claude/settings.local.json`) : vos réglages personnels pour ce projet.
4. **Réglages projet partagés** (`.claude/settings.json`) : réglages que votre équipe versionne dans le contrôle de source.
5. **Réglages utilisateur** (`~/.claude/settings.json`) : vos réglages personnels pour tous les projets.

Les variables d'environnement ne sont pas un niveau de cette pile. Lorsqu'un comportement a à la fois une variable shell et une clé de réglage, celle qui s'applique est décidée paire par paire, et non par niveau : `ANTHROPIC_MODEL` exportée dans votre shell s'applique par-dessus la clé `model` de n'importe quel fichier, tandis que `ANTHROPIC_DEFAULT_MODEL` ne s'applique que si aucun fichier ne définit `model`. La [référence des variables d'environnement](https://code.claude.com/docs/en/env-vars#precedence) indique quelles clés ont une paire et laquelle Claude Code lit en premier. Un bloc `env` dans un fichier de réglages est une clé ordinaire et suit les niveaux ci-dessus.

Pour quelques clés sensibles en matière de sécurité, Claude Code respecte une valeur plus stricte venant d'un niveau inférieur plutôt qu'une valeur gérée ; [Exceptions à la priorité des réglages gérés](#exceptions-à-la-priorité-des-réglages-gérés) les liste.

### Les listes fusionnent au lieu de se surcharger

Quand vous définissez la même clé de type liste, comme `permissions.allow`, dans plusieurs fichiers, Claude Code combine les listes au lieu d'en choisir une : chaque fichier peut ainsi ajouter des entrées sans retirer celles d'un autre fichier. Quatre clés contenant des listes de modèles ou des entrées par modèle suivent leurs propres règles :

* [`fallbackModel`](https://code.claude.com/docs/en/settings-reference#fallbackmodel) est une chaîne ordonnée où la position a un sens : Claude Code prend la valeur entière dans le fichier de plus haute priorité qui la définit.
* [`modelPicker`](https://code.claude.com/docs/en/settings-reference#modelpicker) contient une liste ordonnée de lignes plus un indicateur de remplacement : Claude Code ne fusionne donc jamais les lignes de deux sources. Il prend la valeur entière dans la plus haute des sources managed, `--settings` et user qui la définit, et ignore la clé dans les réglages project et local. (v2.1.242+)
* [`availableModels`](https://code.claude.com/docs/en/settings-reference#availablemodels) : quand les réglages gérés appliqués par Claude Code la définissent, Claude Code applique cette liste telle quelle et ignore les entrées ajoutées dans les réglages user, project ou local, sauf si une application qui embarque Claude Code fournit sa propre liste de modèles ; voir [Exceptions à la priorité des réglages gérés](#exceptions-à-la-priorité-des-réglages-gérés). Entre sources gérées, la liste ne fusionne jamais non plus ; [how Claude Code combines managed sources](https://code.claude.com/docs/en/managed-settings#how-claude-code-combines-managed-sources) indique quelle liste de source s'applique. Entre portées non gérées, Claude Code fusionne les tableaux comme d'habitude.
* [`modelSettings`](https://code.claude.com/docs/en/settings-reference#modelsettings) : Claude Code la résout modèle par modèle, conjointement avec [`effortLevel`](https://code.claude.com/docs/en/settings-reference#effortlevel). L'entrée `modelSettings` précise quelle valeur de fichier s'applique à un modèle.

### Exemples de priorité

Pendant que Claude travaille, Claude Code affiche une astuce d'une ligne sous le spinner, comme « Use /config to change your default permission mode (including Plan Mode) ». Supposons que vous vouliez désactiver ces astuces : vous définissez [`spinnerTipsEnabled`](https://code.claude.com/docs/en/settings-reference#spinnertipsenabled) à `false` dans `~/.claude/settings.json`. Chaque scénario ci-dessous est une chose qui peut les réactiver, avec ce que vous pouvez faire.

#### Les réglages d'équipe surchargent les réglages personnels

Le `.claude/settings.json` de votre équipe la définit à `true`. Claude Code utilise la valeur du projet car shared project est au-dessus de user : vous voyez donc les astuces dans ce projet et nulle part ailleurs.

Vous pouvez récupérer votre valeur : ajoutez `"spinnerTipsEnabled": false` à `.claude/settings.local.json` dans ce projet. Project local est au-dessus de shared project : vos sessions n'y affichent plus d'astuces et celles de vos coéquipiers ne changent pas.

#### Les réglages de l'organisation surchargent tout

Les réglages gérés de votre organisation la définissent à `true`. Rien de ce que vous mettez dans les réglages user, project ou local ne désactive les astuces, et `--settings` non plus. Managed est le niveau le plus élevé.

Vous ne pouvez pas récupérer votre valeur. Lancez `/status` pour voir quelle source gérée s'applique, et demandez à votre administrateur si la politique doit changer.

#### La ligne de commande surcharge vos fichiers pour une session

Vous avez lancé la session avec `claude --settings '{"spinnerTipsEnabled": true}'`. La ligne de commande est au-dessus de tous les fichiers sauf managed : cette session affiche donc les astuces même si vos fichiers indiquent `false`.

Vous récupérez votre valeur à la session suivante ; `--settings` dure une session et n'écrit dans aucun fichier.

#### Un flag ou une variable d'environnement définit la même chose

Certaines clés ont un flag de ligne de commande ou une variable d'environnement qui surcharge la valeur du réglage quel que soit le fichier qui l'a définie : `ANTHROPIC_MODEL` surcharge le réglage [`model`](https://code.claude.com/docs/en/settings-reference#model), et `--model` surcharge les deux pour une session.

Le fait de pouvoir récupérer votre valeur dépend de la clé : supprimez la variable ou retirez le flag, et consultez l'entrée de la clé dans la [référence des réglages](https://code.claude.com/docs/en/settings-reference) et la ligne de la variable dans la [référence des variables d'environnement](https://code.claude.com/docs/en/env-vars) pour savoir laquelle Claude Code utilise.

### Dépanner un réglage qui ne s'applique pas

Quand vous définissez une clé et que Claude Code ne se comporte pas comme si vous l'aviez fait, commencez par `/status` pour voir quels fichiers il a chargés, puis trouvez votre symptôme ci-dessous. [Debug your configuration](https://code.claude.com/docs/en/debug-your-config) couvre les vérifications plus larges, dont un test avec une configuration vierge.

#### Une valeur que vous avez définie est ignorée

Autre chose définit la même clé, le fichier ne peut pas définir cette valeur, ou le fichier n'a pas été chargé :

* **Un niveau supérieur la définit.** Un autre fichier de réglages, un flag `--settings` ou une source gérée définit la clé au-dessus de la vôtre ; la [pile](#ordre-de-priorité-des-réglages) indique laquelle. Un flag ou une variable d'environnement peut aussi surcharger la clé de lui-même, au cas par cas ; l'entrée de la clé dans la [référence des réglages](https://code.claude.com/docs/en/settings-reference) indique celle que Claude Code utilise, et l'[entrée `env`](https://code.claude.com/docs/en/settings-reference#env) traite d'une valeur `env` gérée face à un export shell.
* **Une clé de sécurité garde sa valeur stricte.** Pour quelques clés, Claude Code respecte la valeur restrictive venant de n'importe quel fichier : un `true` de projet pour [`disableClaudeAiConnectors`](https://code.claude.com/docs/en/settings-reference#disableclaudeaiconnectors) reste donc actif ; voir [Exceptions à la priorité des réglages gérés](#exceptions-à-la-priorité-des-réglages-gérés).
* **Le fichier ne peut pas définir cette valeur.** Les valeurs `auto` et `bypassPermissions` de [`permissions.defaultMode`](https://code.claude.com/docs/en/settings-reference#permissions-defaultmode) ne prennent pas effet depuis les réglages project ou local ; définissez-les dans les réglages user ou managed, ou passez `--permission-mode` pour une session. Avant la v2.1.257, `bypassPermissions` prenait effet depuis n'importe quel fichier.

  Une variable d'export de télémétrie dans un bloc [`env`](https://code.claude.com/docs/en/settings-reference#env) ne prend pas non plus effet depuis les réglages project ou local, à part quelques valeurs de désactivation. [Variables Claude Code ignores in `env`](https://code.claude.com/docs/en/settings-reference#variables-claude-code-ignores-in-env) liste les variables et ces valeurs.
* **Le fichier est cassé.** Du JSON invalide ou une valeur rejetée amène Claude Code à ignorer le fichier ou l'entrée ; voir [Réparer un fichier de réglages cassé](#réparer-un-fichier-de-réglages-cassé).

#### Une modification faite dans Claude Code est perdue dans les nouvelles sessions

Quand vous enregistrez un choix pour les nouvelles sessions depuis Claude Code, comme un modèle par défaut avec `/model`, Claude Code l'écrit dans votre fichier de réglages utilisateur, `~/.claude/settings.json`. Si vous ne pouvez pas écrire dans ce fichier, par exemple parce qu'un autre outil le génère ou le lie à une copie en lecture seule, la modification s'applique à la session en cours et disparaît à la suivante. Définissez la clé dans l'outil qui génère le fichier, ou remplacez le fichier par un fichier dans lequel vous pouvez écrire.

Si vous pouvez écrire dans le fichier et que la modification ne dure toujours pas, vérifiez si la modification était [pour une seule session](#modifier-un-réglage-pour-une-seule-session) ou si [un niveau supérieur définit la même clé](#une-valeur-que-vous-avez-définie-est-ignorée). Pour la clé `model`, [A new session starts on a different model than you picked](https://code.claude.com/docs/en/model-config#a-new-session-starts-on-a-different-model-than-you-picked) liste d'autres causes.

#### Une modification gérée ne vous est pas parvenue

Les sources gérées atteignent une session en cours selon le calendrier du [tableau de distribution](https://code.claude.com/docs/en/managed-settings#choose-a-delivery-mechanism) : redémarrez donc d'abord la session. Si `/status` nomme ensuite une autre source que celle que votre administrateur a modifiée, une source de priorité supérieure s'applique ; [How Claude Code combines managed sources](https://code.claude.com/docs/en/managed-settings#how-claude-code-combines-managed-sources) donne l'ordre.

#### Une clé committée n'atteint pas les coéquipiers

Deux choses empêchent une clé de `.claude/settings.json` de s'appliquer à toutes les personnes qui le clonent :

* **Claude Code ignore la clé dans un fichier de dépôt.** Cherchez `User, local, or managed`, `User or managed`, `User`, `Managed` ou `Global config` dans la colonne Scope de l'[index des réglages](https://code.claude.com/docs/en/settings-reference#settings-index). Ces clés ne s'appliquent jamais depuis le fichier partagé, à part quelques-unes qu'un fichier de dépôt peut quand même désactiver ; chacune de ces entrées le précise sur sa ligne Scope. Les clés `Global config` ne s'appliquent que depuis `~/.claude.json`.

  Dans la clé `env`, les variables d'export de télémétrie ne s'appliquent jamais non plus depuis le fichier partagé, à part quelques valeurs de désactivation ; voir [Variables Claude Code ignores in `env`](https://code.claude.com/docs/en/settings-reference#variables-claude-code-ignores-in-env).
* **La clé attend la confiance.** Les règles `permissions.allow`, `permissions.additionalDirectories`, `extraKnownMarketplaces` et la plupart des valeurs [`env`](https://code.claude.com/docs/en/settings-reference#env) ne s'appliquent qu'après que chaque coéquipier a [fait confiance au dossier](https://code.claude.com/docs/en/permissions#project-allow-rules-and-workspace-trust). D'ici là, ils voient toujours les demandes de permission et n'obtiennent pas les plugins d'une marketplace déclarée par le fichier. Les règles `deny` et `ask` s'appliquent immédiatement.

#### Les règles de permission se combinent autrement que prévu

* **Vous avez choisi « Yes, and don't ask again » sur une demande de permission mais on vous redemande pour le même outil.** Ce choix a enregistré une règle `allow` dans votre fichier local, et une règle `allow` à cet endroit ne l'emporte pas sur une règle `ask` venant d'un fichier project ou managed ; [how permission rules combine](https://code.claude.com/docs/en/permissions#settings-precedence) explique l'ordre. Dans l'extension VS Code, la carte d'approbation vous laisse choisir le fichier de destination, y compris le fichier partagé du projet, ce qui change la règle pour tout le monde ; dans la CLI, Claude Code n'écrit que dans votre fichier local.
* **Les règles allow de votre organisation s'appliquent toujours en plus des vôtres.** C'est normal : Claude Code fusionne [`permissions.allow`](https://code.claude.com/docs/en/settings-reference#permissions-allow) entre les portées, sauf si votre organisation définit [`allowManagedPermissionRulesOnly`](https://code.claude.com/docs/en/settings-reference#allowmanagedpermissionrulesonly).

### Exceptions à la priorité des réglages gérés

Pour quelques clés dont les valeurs restreignent une session, Claude Code respecte une valeur restrictive venant d'une portée qui, autrement, ne pourrait pas surcharger les réglages gérés. Trouvez la clé dans ce tableau pour voir quelle valeur il respecte et d'où.

| Clé | Valeur respectée par Claude Code | Remarques |
| :- | :- | :- |
| [`disableClaudeAiConnectors`](https://code.claude.com/docs/en/settings-reference#disableclaudeaiconnectors) | `true` depuis n'importe quelle portée | Respectée même quand une source gérée définit `false` |
| [`enableArtifact`](https://code.claude.com/docs/en/settings-reference#enableartifact) | `false` depuis n'importe quelle portée, et `disableArtifact: true` depuis n'importe quelle portée | Respectée même quand une source gérée définit `true` ; rien ne réactive l'[outil Artifact](https://code.claude.com/docs/en/artifacts#disable-artifacts). (v2.1.242+) |
| [`isolatePeerMachines`](https://code.claude.com/docs/en/settings-reference#isolatepeermachines) | `true` depuis n'importe quelle portée | Respectée même quand une source gérée définit `false` |
| [`permissions.blockReadsOutsideWorkingDirectories`](https://code.claude.com/docs/en/settings-reference#permissions-blockreadsoutsideworkingdirectories) | `true` depuis n'importe quelle portée | Respectée même quand une source gérée définit `false`. (v2.1.257+) |
| [`autoMode.classifyAllShell`](https://code.claude.com/docs/en/settings-reference#automode-classifyallshell) | `true` depuis `~/.claude/settings.json` ou `--settings` | Respectée même quand une source gérée définit `false` |
| [`remoteControlAtStartup`](https://code.claude.com/docs/en/settings-reference#remotecontrolatstartup) | `false` depuis `.claude/settings.json` ou `.claude/settings.local.json` | Respectée même quand une source gérée définit `true` ; un `true` project ou local est ignoré |
| [`crossSessionInbound`](https://code.claude.com/docs/en/settings-reference#crosssessioninbound) | Une valeur plus stricte depuis `.claude/settings.json` ou `.claude/settings.local.json`, sur l'échelle `accept` < `hold` < `refuse` | Respectée par-dessus les valeurs managed, `--settings` et user ; une valeur project ou local qui n'est pas plus stricte est ignorée |
| [`useAutoModeDuringPlan`](https://code.claude.com/docs/en/settings-reference#useautomodeduringplan) | `false` depuis n'importe quelle source gérée, `--settings`, `~/.claude/settings.json` ou `.claude/settings.local.json` | Respectée même quand la source gérée gagnante définit `true` ; un `false` dans `.claude/settings.json` est ignoré |
| [`syncClaudeAiSkills`](https://code.claude.com/docs/en/settings-reference#syncclaudeaiskills) | `false` depuis n'importe quelle source gérée, `--settings`, `~/.claude/settings.json` ou `.claude/settings.local.json` | Respectée même quand la source gérée gagnante définit `true` ; un `false` dans `.claude/settings.json` est ignoré |
| [`syncClaudeAiPlugins`](https://code.claude.com/docs/en/settings-reference#syncclaudeaiplugins) | `false` depuis n'importe quelle source gérée, `--settings`, `~/.claude/settings.json` ou `.claude/settings.local.json` | Respectée même quand la source gérée gagnante définit `true` ; un `false` dans `.claude/settings.json` est ignoré |
| [`maxEffortLevel`](https://code.claude.com/docs/en/settings-reference#maxeffortlevel) | Un plafond plus bas depuis n'importe quelle portée, y compris `--settings` | Respecté même quand les réglages gérés appliqués par Claude Code définissent un plafond plus élevé ; c'est le plafond le plus bas qui s'applique. (v2.1.267+) |

Une application qui exécute Claude Code en son sein et définit [`CLAUDE_CODE_PROVIDER_MANAGED_BY_HOST`](https://code.claude.com/docs/en/env-vars) est aussi une exception. Claude Code retient la configuration de modèle de cette application par-dessus les clés `model`, `fallbackModel`, `modelPicker` et `modelOverrides` de toutes les sources gérées, et par-dessus les variables de sélection de modèle d'un bloc `env` géré, comme `ANTHROPIC_MODEL` et la famille `ANTHROPIC_DEFAULT_*_MODEL`. Claude Code maintient en vigueur une liste d'autorisation [`availableModels`](https://code.claude.com/docs/en/settings-reference#availablemodels) gérée, sauf si l'application fournit la sienne.

## Réglages dans les sessions cloud

Une [session cloud](https://code.claude.com/docs/en/claude-code-on-the-web) s'exécute dans un [environnement cloud](https://code.claude.com/docs/en/cloud-environments) sur un clone neuf de votre dépôt, pas sur votre machine. Cela change les réglages qui l'atteignent :

* **Réglages projet partagés** (`.claude/settings.json`) : lus dans une session avec un seul dépôt, car le fichier fait partie du clone et la session démarre à l'intérieur. Committez-y un réglage pour l'appliquer dans ces sessions. Dans un environnement hébergé par Anthropic, une session avec plusieurs dépôts démarre au-dessus des clones et ne lit que les clés `enabledPlugins` et `extraKnownMarketplaces` du `.claude/settings.json` de chaque dépôt, pas les règles de permission, les hooks, `env` ni les autres clés. Les marketplaces et plugins déclarés par ces deux clés [ne se chargent toujours pas dans une session cloud](https://code.claude.com/docs/en/cloud-environments#what-carries-over-from-your-setup). Pour un environnement auto-hébergé, voir [which repository's settings apply](https://code.claude.com/docs/en/self-hosted-environments-configuration#repository-settings-in-sessions-with-several-repositories).
* **Réglages user et project local** (`~/.claude/settings.json` et `.claude/settings.local.json`) : non lus. Les deux restent sur votre machine, et le fichier local n'est pas dans le clone.
* **Réglages gérés** : un fichier `managed-settings.json` ou un profil MDM sur votre appareil n'atteint pas une session cloud. Les [server-managed settings](https://code.claude.com/docs/en/server-managed-settings) de votre organisation l'atteignent ; [surface coverage](https://code.claude.com/docs/en/model-config#surface-coverage) liste les sessions cloud qui les reçoivent. Un [environnement auto-hébergé](https://code.claude.com/docs/en/self-hosted-environments) lit aussi le fichier de réglages gérés présent dans son image de runner. [How Claude Code combines managed sources](https://code.claude.com/docs/en/managed-settings#how-claude-code-combines-managed-sources) indique quand ce fichier s'applique.
* **`/config`** : dans votre navigateur sur claude.ai/code, ouvre la section Claude Code de vos réglages claude.ai au lieu de modifier une valeur. Pour modifier un réglage d'une session cloud, définissez une [variable d'environnement](https://code.claude.com/docs/en/cloud-environments#set-environment-variables) sur l'environnement ou, dans une session avec un seul dépôt, committez la clé dans le `.claude/settings.json` de ce dépôt.

[What carries over from your setup](https://code.claude.com/docs/en/cloud-environments#what-carries-over-from-your-setup) liste le reste : `CLAUDE.md`, skills, serveurs MCP, plugins et identifiants.

## Pour aller plus loin

* [All settings](https://code.claude.com/docs/en/settings-reference) : chaque clé, avec l'endroit où la définir et un exemple
* [Example settings files](https://code.claude.com/docs/en/settings-example) : un fichier personnel, un fichier d'équipe et le fichier géré d'une organisation
* [Configure permissions](https://code.claude.com/docs/en/permissions) : règles allow, ask et deny, et ce que Claude Code exécute sans demander
* [Environment variables](https://code.claude.com/docs/en/env-vars) : les variables lues par Claude Code et le bloc `env`
* [Debug your configuration](https://code.claude.com/docs/en/debug-your-config) : quand un réglage ne s'applique pas
* [Claude directory reference](https://code.claude.com/docs/en/claude-directory) : chaque fichier lu par Claude Code, y compris sous-agents, serveurs MCP, plugins et `CLAUDE.md`
