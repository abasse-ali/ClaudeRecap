# Tous les réglages — partie 2/4

> Source : https://code.claude.com/docs/en/settings-reference · [← Index](../31-index-documentation-officielle.md)

> Cette partie couvre les réglages de **permissions**, de **bac à sable (sandbox)** et de **mémoire et contexte**. Les portées citées (`Any file` = tout fichier, `User or managed` = utilisateur ou géré, `User, local, or managed` = utilisateur, local ou géré, `Managed` = géré uniquement) sont définies dans la section [Scopes](https://code.claude.com/docs/en/settings-reference#scopes) (partie 1). Les liens vers `settings-reference#…` renvoient aux autres clés de la même page officielle.

## Réglages de permissions

Décidez ce que Claude peut faire sans demander, dans quel mode de permission une session démarre, et ce que le classifieur du mode auto autorise. Pour la syntaxe des règles et le modèle de permissions, voir [Configure permissions](https://code.claude.com/docs/en/permissions).

### `allowManagedPermissionRulesOnly`

Fait des réglages gérés (managed settings) la **seule** source de règles de permission. Claude Code ignore alors les règles `allow`, `ask` et `deny` des fichiers utilisateur, projet, local et `--settings`, ignore `--allowedTools`, masque les choix « toujours autoriser » dans les invites de permission et cesse d'enregistrer de nouvelles règles.

Lorsque des [réglages parents fournis par un hôte d'intégration](https://code.claude.com/docs/en/managed-settings#let-an-embedding-host-add-policy) s'appliquent, Claude Code les traite comme faisant partie du niveau géré. Il abandonne leurs règles `allow` et leurs `additionalDirectories`, et conserve leurs règles `deny` et `ask`, sauf les règles `Read` et `Edit` dont le motif commence par `!`. Un hôte ne peut pas soustraire des chemins aux règles gérées avec une règle `!`, que vous définissiez cette clé ou non.

Les règles `--disallowedTools` et les règles `deny` et `ask` de la session en cours continuent de s'appliquer, y compris après un rechargement des réglages en cours de session. Elles ne font que restreindre : elles ne peuvent pas élargir ce que les règles gérées accordent. Avant la v2.1.257, Claude Code abandonnait ces règles de ligne de commande et de session au premier rechargement des réglages.

Pour ce qu'un motif `!` dans une règle `--disallowedTools` ou de session peut exclure, voir [Read and Edit rules](https://code.claude.com/docs/en/permissions#read-and-edit).

Quand cette clé est définie, Claude Code (v2.1.282+) ignore aussi le frontmatter [`allowed-tools`](https://code.claude.com/docs/en/skills#pre-approve-tools-for-a-skill) des skills et des fichiers `.claude/commands/` provenant de ces sources :

* le répertoire `.claude/` d'un dépôt ;
* vos répertoires `~/.claude/skills/` et `~/.claude/commands/`, y compris les [skills synchronisés depuis claude.ai](https://code.claude.com/docs/en/skills#where-synced-skills-load) ;
* un répertoire ajouté par `--add-dir` ;
* les [plugins déclarés avec un manifeste `.claude-plugin`](https://code.claude.com/docs/en/plugins/loading#plugins-shared-through-a-repository) situés dans `~/.claude/skills/` ou dans le `.claude/skills/` du projet.

Les skills issus des réglages gérés et les skills intégrés (bundled) conservent leur `allowed-tools`. Le `disallowed-tools` d'un skill continue de s'appliquer. Pour ce que voit un développeur quand Claude Code ignore ce champ, voir [When only managed permission rules apply](https://code.claude.com/docs/en/skills#when-only-managed-permission-rules-apply).

* **Portée** : `Managed`
* **Type** : booléen
  * `true` : les réglages gérés deviennent la seule source de règles de permission
  * `false` : Claude Code applique les règles de permission des fichiers utilisateur, projet, local et `--settings`, en plus des règles gérées
* **Défaut** : non défini ; Claude Code applique donc les règles des réglages utilisateur, projet et local et de `--settings`, en plus des règles gérées

```json managed-settings.json
{
  "allowManagedPermissionRulesOnly": true
}
```

Cette clé ne verrouille pas la liste d'autorisation des serveurs MCP ; pour cela, définissez [`allowManagedMcpServersOnly`](https://code.claude.com/docs/en/settings-reference#allowmanagedmcpserversonly). Voir [Managed-only settings](https://code.claude.com/docs/en/managed-settings#managed-only-settings).

### `autoMode`

Ajoute vos propres règles à ce que le classifieur du [mode auto](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) bloque et autorise. Servez-vous-en pour indiquer au classifieur quels dépôts, buckets et domaines votre organisation considère comme fiables, afin qu'il cesse de bloquer les opérations internes de routine. Le classifieur est livré avec des [règles intégrées d'autorisation et de refus](https://code.claude.com/docs/en/auto-mode-config#inspect-the-defaults-and-your-effective-config). Incluez la chaîne littérale `"$defaults"` dans un tableau pour conserver ces règles intégrées à cette position et ajouter les vôtres autour ; omettez-la pour les remplacer par les vôtres.

* **Portée** : `User or managed`
* **Type** : objet avec les tableaux de règles rédigées en prose `environment`, `allow`, `soft_deny` et `hard_deny`, plus le booléen [`classifyAllShell`](https://code.claude.com/docs/en/settings-reference#automode-classifyallshell)
* **Défaut** : non défini ; le classifieur n'utilise que ses [règles intégrées](https://code.claude.com/docs/en/auto-mode-config#inspect-the-defaults-and-your-effective-config)

Cet exemple conserve les règles `soft_deny` intégrées, grâce à `"$defaults"`, et en ajoute une qui bloque `terraform apply` :

```json settings.json
{
  "autoMode": {
    "soft_deny": ["$defaults", "Never run terraform apply"]
  }
}
```

Quand plusieurs de ces fichiers définissent le même tableau, Claude Code concatène les entrées. Pour le format des règles et la façon dont chaque tableau est appliqué, voir [Configure auto mode](https://code.claude.com/docs/en/auto-mode-config).

### `autoMode.classifyAllShell`

Envoie **chaque** commande Bash et PowerShell au classifieur du mode auto tant que le mode auto est actif. Par défaut, le mode auto ne suspend que les règles d'autorisation qui pourraient exécuter du code arbitraire : les règles portant sur l'outil entier ou à joker comme `Bash(*)`, et les préfixes d'interpréteur ou d'enveloppe de shell comme `Bash(python *)`. Une commande que n'importe quelle autre règle d'autorisation couvre, comme `Bash(npm test)`, contourne le classifieur, sauf si elle porte des [domaines autorisés par commande](https://code.claude.com/docs/en/sandboxing#per-command-allowed-domains-in-auto-mode). Dans ce cas, un argument destructeur que le préfixe de la règle n'avait pas prévu peut passer inaperçu. Définir cette clé suspend toutes les règles d'autorisation shell pour la session, de sorte que le classifieur voit chaque commande. (v2.1.193+)

* **Portée** : `User or managed`. Lue partout où [`autoMode`](https://code.claude.com/docs/en/settings-reference#automode) est lue.
* **Type** : booléen
  * `true` : tant que le mode auto est actif, Claude Code envoie chaque commande Bash et PowerShell au classifieur et suspend vos règles d'autorisation shell ; hors mode auto, les règles s'appliquent toujours
  * `false` : le mode auto ne suspend que les règles d'autorisation capables d'exécuter du code arbitraire, comme `Bash(*)` et `Bash(python *)` ; une commande couverte par une autre règle d'autorisation contourne le classifieur, sauf si elle porte des [domaines autorisés par commande](https://code.claude.com/docs/en/sandboxing#per-command-allowed-domains-in-auto-mode), et toutes les autres commandes shell passent par lui
* **Défaut** : `false`

```json settings.json
{
  "autoMode": {
    "classifyAllShell": true
  }
}
```

Voir [Route all shell commands through the classifier](https://code.claude.com/docs/en/auto-mode-config#route-all-shell-commands-through-the-classifier).

### `disableAutoMode`

Retire le [mode auto](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) du cycle `Shift+Tab`. Toute session qui aurait sinon [démarré en mode auto](https://code.claude.com/docs/en/permission-modes#which-mode-a-session-starts-in) — via `--permission-mode auto`, un fichier de réglages ou le défaut intégré — démarre en `default` à la place. Les administrateurs la définissent dans les réglages gérés pour empêcher les développeurs de leur organisation d'utiliser le mode auto.

* **Portée** : `Any file`. Surtout utile dans les [réglages gérés](https://code.claude.com/docs/en/managed-settings), où les utilisateurs ne peuvent pas la surcharger. Acceptée aussi sous `permissions`, sous la forme `permissions.disableAutoMode`.
* **Type** : la chaîne `"disable"`
* **Défaut** : non défini

```json settings.json
{
  "disableAutoMode": "disable"
}
```

### `permissions`

Contrôle quels outils Claude peut utiliser sans demander, lesquels déclenchent toujours une invite et lesquels sont bloqués, et définit le [mode de permission](https://code.claude.com/docs/en/permission-modes) dans lequel une session démarre. Toutes les clés `permissions.*` ci-dessous s'imbriquent dans cet objet.

* **Portée** : `Any file`
* **Type** : objet avec `allow`, `ask`, `deny`, `additionalDirectories`, `blockReadsOutsideWorkingDirectories`, `defaultMode`, `disableBypassPermissionsMode` et `disableAutoMode`
* **Défaut** : non défini

Cet exemple approuve les commandes `npm run` sans demander, demande confirmation avant `git push`, bloque la lecture de `.env` et démarre les sessions en `acceptEdits` :

```json settings.json
{
  "permissions": {
    "allow": ["Bash(npm run *)"],
    "ask": ["Bash(git push *)"],
    "deny": ["Read(./.env)"],
    "defaultMode": "acceptEdits"
  }
}
```

Les trois tableaux de règles partagent une même syntaxe ; voir [Syntaxe des règles de permission](#syntaxe-des-règles-de-permission) sous `permissions.allow`. Pour la façon dont les règles de permission de différents fichiers se combinent, voir [how permission rules merge across scopes](https://code.claude.com/docs/en/permissions#settings-precedence) ; pour la combinaison des clés de réglages en général, voir [Settings precedence](https://code.claude.com/docs/en/settings#settings-precedence) dans le guide des réglages.

### `useAutoModeDuringPlan`

Choisit si Claude Code utilise le classifieur du mode auto pour examiner les commandes shell en mode plan. Avec la valeur par défaut `true`, le classifieur examine chaque commande pendant la planification quand le mode auto est disponible, et vous ne voyez aucune invite, sauf pour les [suppressions de chemins critiques](https://code.claude.com/docs/en/permission-modes#critical-paths). Mettez `false` pour obtenir une invite de permission pour chaque commande hors de l'ensemble intégré en lecture seule. Apparaît dans `/config` sous le nom **Use auto mode during plan**.

* **Portée** : `User, local, or managed`. Un dépôt ne peut pas la désactiver à votre place.
* **Type** : booléen
  * `true` : identique à non défini ; quand le mode auto est disponible, le classifieur examine chaque commande shell pendant la planification au lieu de vous la soumettre, sauf les [suppressions de chemins critiques](https://code.claude.com/docs/en/permission-modes#critical-paths). Un `false` dans l'un de ces fichiers la désactive malgré tout
  * `false` : vous obtenez une invite de permission pour chaque commande hors de l'ensemble intégré en lecture seule
* **Défaut** : `true`

```json settings.json
{
  "useAutoModeDuringPlan": false
}
```

### `permissions.allow`

Liste les utilisations d'outils que Claude Code approuve sans vous demander. Dans une règle MCP, `*` ne peut apparaître que dans le nom de l'outil après le préfixe `mcp__<server>__`, par exemple `mcp__github__get_*` ; il ne peut pas apparaître dans le nom du serveur.

* **Portée** : `Any file`
* **Type** : tableau de chaînes de règles de permission
* **Défaut** : non défini
* **Surcharges par session** : `--allowedTools` ajoute des règles d'autorisation pour une session, et une règle de refus de n'importe quel fichier de réglages bloque toujours un outil qu'elle nomme

Cet exemple approuve `git diff` et laisse Claude Code lire votre `.zshrc` sans demander :

```json settings.json
{
  "permissions": {
    "allow": ["Bash(git diff *)", "Read(~/.zshrc)"]
  }
}
```

Claude Code n'applique les règles `allow` du `.claude/settings.json` d'un projet qu'après que vous avez accepté la [boîte de dialogue de confiance de l'espace de travail](https://code.claude.com/docs/en/permissions#project-allow-rules-and-workspace-trust) pour ce dossier.

#### Syntaxe des règles de permission

Les règles de permission suivent le format `Tool` ou `Tool(specifier)`. Claude Code évalue d'abord les règles `deny`, puis `ask`, puis `allow`, et la première correspondance décide, quelle que soit la spécificité de chaque règle ; voir l'[ordre d'évaluation des règles de permission](https://code.claude.com/docs/en/permissions#manage-permissions).

Chaque ligne montre une forme de règle et ce qu'elle couvre.

| Règle | Ce qu'elle couvre |
| :- | :- |
| `Bash` | Toutes les commandes Bash |
| `Bash(npm run *)` | Les commandes commençant par `npm run` |
| `Read(./.env)` | Les lectures du fichier `.env` |
| `WebFetch(domain:example.com)` | Les requêtes de récupération vers example.com |

Pour la syntaxe complète — comportement des jokers, motifs spécifiques aux outils Read, Edit, WebFetch, MCP et Agent, et limites de sécurité des motifs Bash — voir [Permission rule syntax](https://code.claude.com/docs/en/permissions#permission-rule-syntax).

### `permissions.ask`

Liste les utilisations d'outils qui vous demandent confirmation même dans un mode de permission qui les approuverait sinon, comme `acceptEdits` ou `bypassPermissions`. En mode `dontAsk`, Claude Code refuse l'utilisation correspondante au lieu de demander.

* **Portée** : `Any file`
* **Type** : tableau de chaînes de règles de permission
* **Défaut** : non défini

```json settings.json
{
  "permissions": {
    "ask": ["Bash(git push *)"]
  }
}
```

### `permissions.deny`

*(Ancre historique : `exclude-sensitive-files`.)*

Liste les utilisations d'outils que Claude Code bloque. Utilisez-la pour les fichiers qui contiennent des clés d'API, des secrets ou des valeurs d'environnement : Claude Code exclut les fichiers correspondants de la découverte de fichiers et des résultats de recherche, refuse leur lecture et bloque les [outils Edit et Write](https://code.claude.com/docs/en/permissions#read-and-edit) sur les chemins correspondants.

Les règles de refus Read et Edit s'appliquent aux outils de fichiers intégrés de Claude, aux commandes de fichiers que Claude Code reconnaît dans Bash, comme `cat`, `head`, `tail`, `sed` et `tee`, et aux cibles des [redirections](https://code.claude.com/docs/en/permissions#redirections) Bash comme `> file` et `< file` ; elles ne s'appliquent pas à une commande qui lit des fichiers sans les nommer, comme `grep -r pattern .`, ni aux sous-processus arbitraires. Pour une application au niveau du système d'exploitation, [activez le bac à sable](https://code.claude.com/docs/en/sandboxing).

* **Portée** : `Any file`
* **Type** : tableau de chaînes de règles de permission
* **Défaut** : non défini
* **Surcharges par session** : `--disallowedTools` ajoute des règles de refus pour une session, en plus de cette clé

Cet exemple refuse la lecture des fichiers `.env`, du répertoire `secrets` et d'un fichier d'identifiants, et bloque les commandes `curl` :

```json settings.json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./config/credentials.json)",
      "Bash(curl *)"
    ]
  }
}
```

Les noms d'outils acceptent des motifs glob : `"*"` refuse tous les outils et `"mcp__*"` refuse tous les outils MCP. Claude Code ignore une règle de refus visant l'outil [`EndConversation`](https://code.claude.com/docs/en/tools-reference#endconversation-tool-behavior) tant qu'un autre outil reste disponible pour Claude. Une règle de refus `Bash` porte sur la commande telle que Claude l'écrit : `Bash(curl *)` n'arrête donc ni `/usr/bin/curl` ni `sh -c 'curl …'` ; voir [what a Bash rule doesn't match](https://code.claude.com/docs/en/permissions#bash-rule-limits). Cette clé remplace la configuration obsolète `ignorePatterns`.

### `permissions.additionalDirectories`

Donne à Claude l'accès aux fichiers de répertoires situés hors de celui où vous avez démarré, en tant que [répertoires de travail](https://code.claude.com/docs/en/permissions#working-directories) supplémentaires. La plupart de la configuration `.claude/` n'est [pas découverte](https://code.claude.com/docs/en/permissions#additional-directories-grant-file-access-not-configuration) dans ces répertoires.

* **Portée** : `Any file`, avec des [limites sur l'accès en écriture du bac à sable](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox) que donnent les entrées de projet et locales
* **Type** : tableau de chemins de répertoires
* **Défaut** : non défini
* **Surcharges par session** : `--add-dir` et `/add-dir` ajoutent des répertoires pour une session, en plus de cette clé

```json settings.json
{
  "permissions": {
    "additionalDirectories": ["../docs/"]
  }
}
```

Comme pour les règles `allow`, les entrées du `.claude/settings.json` d'un projet ne prennent effet qu'après acceptation de la [boîte de dialogue de confiance de l'espace de travail](https://code.claude.com/docs/en/permissions#project-allow-rules-and-workspace-trust) pour ce dossier.

### `permissions.blockReadsOutsideWorkingDirectories`

Fait refuser par les outils de fichiers de Claude les lectures hors de vos [répertoires de travail](https://code.claude.com/docs/en/permissions#working-directories), dans **tous** les modes de permission, y compris `bypassPermissions`. Claude Code refuse les appels `Read`, `Grep`, `Glob` et `LSP` sur ces chemins et indique à Claude de vous demander d'ajouter le répertoire avec `/add-dir`. Les fichiers dont Claude Code a lui-même besoin restent lisibles, comme vos skills, plugins, règles, agents, commandes et le fichier mémoire `CLAUDE.md` sous `~/.claude/`. (v2.1.257+)

Claude Code ne refuse pas les commandes shell de la même façon :

* [Actions no mode auto-approves](https://code.claude.com/docs/en/permission-modes#actions-no-mode-auto-approves) décrit quand une commande shell qui lit un tel chemin vous soumet une invite ;
* [Commandes en bac à sable sous le blocage](#commandes-en-bac-à-sable-sous-le-blocage) décrit ce qu'une commande en bac à sable peut lire.

Une commande Bash que l'analyseur shell ne peut pas suivre, par exemple une qui change de répertoire plus d'une fois ou lance un sous-shell, vous soumet une invite même en mode auto et en mode `bypassPermissions`. L'invite apparaît même quand la commande ne nomme aucun chemin hors des répertoires de travail. Cette invite ne s'applique pas quand la commande s'exécute dans le [bac à sable](https://code.claude.com/docs/en/sandboxing) et que le bac à sable applique le blocage.

Claude Code écrit aussi `true` ici lorsque vous choisissez de bloquer ces lectures dans l'[invite que le mode auto affiche avant la première lecture hors des répertoires de travail](https://code.claude.com/docs/en/permission-modes#first-read-outside-the-working-directories).

* **Portée** : `Any file`. Un `true` dans n'importe quel fichier s'applique : un dépôt peut activer le blocage pour lui-même mais ne peut pas lever le vôtre.
* **Type** : booléen
  * `true` : les outils de fichiers de Claude refusent les lectures hors des répertoires de travail
  * `false` : identique à non défini ; le blocage s'applique quand même si un autre fichier définit `true`
* **Défaut** : non défini ; les lectures hors des répertoires de travail suivent votre [mode de permission](https://code.claude.com/docs/en/permission-modes)

```json settings.json
{
  "permissions": {
    "blockReadsOutsideWorkingDirectories": true
  }
}
```

Les répertoires ajoutés avec `--add-dir`, `/add-dir` ou `additionalDirectories` dans vos réglages utilisateur ou gérés comptent comme répertoires de travail pour le blocage. Les répertoires ajoutés uniquement dans les réglages du dépôt ne comptent pas : ceux de `.claude/settings.json`, et ceux de `.claude/settings.local.json` sauf si git signale ce fichier comme non suivi (untracked). Dans un répertoire qui n'est pas un dépôt git, ou quand git suit ce fichier, Claude Code traite `.claude/settings.local.json` comme des réglages du dépôt ; placez donc les répertoires que vous voulez garder lisibles dans vos réglages utilisateur.

Lorsque [`autoMemoryDirectory`](https://code.claude.com/docs/en/settings-reference#automemorydirectory) provient du `.claude/settings.json` du projet, ou d'un `.claude/settings.local.json` [traité comme fourni par le dépôt](https://code.claude.com/docs/en/permissions#when-your-local-settings-file-needs-trust), Claude Code ne charge aucune [mémoire automatique](https://code.claude.com/docs/en/memory#storage-location) depuis ce répertoire et n'y enregistre rien.

Pour lever le blocage, retirez la clé de tous les fichiers de réglages qui la définissent, puis démarrez une nouvelle session.

#### Commandes en bac à sable sous le blocage

Quand le [bac à sable](https://code.claude.com/docs/en/sandboxing) est activé, le blocage couvre aussi les commandes en bac à sable. Claude Code leur refuse l'accès en lecture à votre répertoire personnel et aux autres racines qui contiennent des fichiers utilisateur : `/Users`, `/home`, `/root`, `/Volumes`, `/mnt`, `/media`, `/run/media` et `/srv`. Il rouvre ensuite les répertoires de travail, les [worktrees](https://code.claude.com/docs/en/worktrees) créés par Claude Code pendant la session, le répertoire temporaire de la session et les parties de `~/.claude` dont les commandes ont besoin, comme les skills et les plugins. Tant que le blocage est en vigueur, les entrées `allowRead` et `allowWrite` provenant des réglages du dépôt ne comptent pas.

Quand le répertoire de travail de la session est un [worktree git](https://code.claude.com/docs/en/worktrees) lié, y compris un worktree dans lequel Claude Code est entré en cours de session, le répertoire `.git` commun du dépôt reste lisible et inscriptible pour les commandes en bac à sable, afin que git continue de fonctionner.

Dans les cas suivants, le blocage n'atteint pas les commandes en bac à sable, tandis que les outils de fichiers de Claude continuent de l'appliquer :

* l'isolation du système de fichiers est désactivée via [`sandbox.filesystem.disabled`](https://code.claude.com/docs/en/settings-reference#sandbox-filesystem-disabled) ;
* [`allowManagedReadPathsOnly`](https://code.claude.com/docs/en/settings-reference#sandbox-filesystem-allowmanagedreadpathsonly) est défini ;
* le chemin du répertoire où vous avez lancé Claude Code contient un caractère glob comme `*`, `?` ou `[`.

Sous le blocage, Claude Code rouvre aux commandes en bac à sable vos fichiers de configuration git globaux, pour que `git` conserve votre identité et vos réglages :

* `~/.gitconfig` ;
* les fichiers `config`, `ignore` et `attributes` sous `$XDG_CONFIG_HOME/git`, qui vaut par défaut `~/.config/git` ;
* les fichiers que votre configuration git globale nomme via `[include]`, `[includeIf]`, `core.excludesFile` ou `core.attributesFile`.

Claude Code évalue chaque fichier séparément. Lorsqu'un fichier se trouve à un endroit où une commande en bac à sable peut écrire, directement ou via un lien symbolique, Claude Code ne rouvre pas les fichiers qu'il nomme.

Sous Linux et WSL2, un fichier de configuration qui est un lien symbolique peut rester illisible à son propre chemin, et `git` s'exécute alors sans lui. `~/.git-credentials` et `$XDG_CONFIG_HOME/git/credentials` restent bloqués.

Si un fichier rouvert contient un secret, comme un jeton `http.extraHeader`, ajoutez son chemin à [`sandbox.filesystem.denyRead`](https://code.claude.com/docs/en/settings-reference#sandbox-filesystem-denyread). Une entrée `denyRead` qui couvre un fichier l'emporte toujours sur cette réouverture.

### `permissions.defaultMode`

Définit le [mode de permission](https://code.claude.com/docs/en/permission-modes) dans lequel les nouvelles sessions démarrent. Si vous ne la définissez pas, les sessions démarrent dans le [défaut intégré](https://code.claude.com/docs/en/permission-modes#which-mode-a-session-starts-in) de votre surface.

* **Portée** : `Any file`. `auto` et `bypassPermissions` ne prennent pas effet depuis les réglages de projet ou locaux : définissez-les plutôt dans `~/.claude/settings.json`. Avant la v2.1.257, `bypassPermissions` prenait effet depuis n'importe quel fichier. Pour les conversations lancées par l'extension VS Code, Claude Code ne lit que les valeurs utilisateur, gérées et `--settings`.
* **Type** : chaîne, l'une de :
  * `"default"` : Claude Code n'exécute sans demander que les lectures
  * `"acceptEdits"` : Claude Code exécute aussi sans demander les modifications de fichiers et les commandes courantes du système de fichiers comme `mkdir` et `mv`
  * `"plan"` : Claude Code lit et planifie, mais bloque les modifications jusqu'à ce que vous approuviez un plan
  * `"auto"` : Claude Code s'exécute sans invites de routine ; avant que des actions comme les commandes shell et les requêtes réseau ne s'exécutent, un classifieur en arrière-plan vérifie qu'elles correspondent à votre demande
  * `"dontAsk"` : Claude Code refuse automatiquement tout appel qui déclencherait sinon une invite ; les lectures de fichiers dans vos répertoires de travail, les autres actions qui ne nécessitent pas d'approbation et les outils pré-approuvés s'exécutent toujours, à l'exception des lectures sur des [chemins réseau](https://code.claude.com/docs/en/permissions#network-paths)
  * `"bypassPermissions"` : Claude Code exécute tout sans demander
  * `"manual"` : alias de `"default"`
* **Défaut** : non défini
* **Surcharges par session** : `--permission-mode`, et son équivalent `--dangerously-skip-permissions` pour `bypassPermissions`, l'emportent sur cette clé pour une session

```json settings.json
{
  "permissions": {
    "defaultMode": "acceptEdits"
  }
}
```

Les règles de permission se superposent à chaque mode : les règles `deny` bloquent dans tous les modes, y compris `bypassPermissions`. Voir [Permission modes](https://code.claude.com/docs/en/permission-modes). Dans les sessions cloud, Claude Code n'honore que `acceptEdits`, `plan`, `default` et `auto` pour cette clé. Pour les conversations lancées par l'extension VS Code, voir [quel réglage l'extension lit pour le mode de permission de départ](https://code.claude.com/docs/en/permission-modes#switch-permission-modes).

### `permissions.disableBypassPermissionsMode`

Empêche quiconque d'entrer en mode `bypassPermissions`. Claude Code rejette alors le flag `--dangerously-skip-permissions` et ignore le `permissionMode: bypassPermissions` d'une [définition d'agent](https://code.claude.com/docs/en/sub-agents#permission-modes) : le sous-agent s'exécute alors avec le mode de permission de la session parente.

* **Portée** : `Any file`. Généralement définie dans les [réglages gérés](https://code.claude.com/docs/en/managed-settings) pour imposer une politique d'organisation.
* **Type** : la chaîne `"disable"`
* **Défaut** : non défini
* **Surcharges par session** : cette clé l'emporte sur `--dangerously-skip-permissions`, que Claude Code rejette tant que la clé est définie

```json settings.json
{
  "permissions": {
    "disableBypassPermissionsMode": "disable"
  }
}
```

Avant la v2.1.223, Claude Code appliquait le mode de permission du frontmatter même avec le contournement désactivé.

### `skipAutoPermissionPrompt`

Ignore l'avis unique décrivant le [mode auto](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) que Claude Code affiche la première fois que vous entrez vous-même en mode auto, par exemple via vos propres réglages ou le sélecteur de mode, plutôt que lorsque le défaut intégré démarre une session dans ce mode. Claude Code affiche cet avis une fois, puis enregistre qu'il a été affiché : cette clé n'a donc d'effet que là où l'avis n'est pas encore apparu.

* **Portée** : `User or managed`. Un dépôt ne peut pas la définir à votre place.
* **Type** : booléen
  * `true` : Claude Code ignore l'avis
  * `false` : identique à non défini ; l'avis apparaît une fois, sauf si un autre de ces fichiers définit `true`
* **Défaut** : non défini ; l'avis apparaît une fois

```json settings.json
{
  "skipAutoPermissionPrompt": true
}
```

### `skipDangerousModePermissionPrompt`

Ignore la boîte de dialogue de confirmation que Claude Code affiche avant qu'une session entre en mode `bypassPermissions`, que ce soit via `--dangerously-skip-permissions` ou via `defaultMode: "bypassPermissions"`. Claude Code écrit `true` ici dans vos réglages utilisateur lorsque vous acceptez cette boîte de dialogue une fois.

* **Portée** : `User, local, or managed`. Un dépôt non fiable ne peut pas faire ignorer la boîte de dialogue à votre place.
* **Type** : booléen
  * `true` : Claude Code ignore la boîte de dialogue de confirmation avant qu'une session entre en mode `bypassPermissions`
  * `false` : identique à non défini ; la boîte de dialogue apparaît, sauf si un autre de ces fichiers définit `true`
* **Défaut** : non défini ; la boîte de dialogue apparaît

```json settings.json
{
  "skipDangerousModePermissionPrompt": true
}
```

## Réglages du bac à sable (sandbox)

Isolez les commandes que Claude exécute de votre système de fichiers, de votre réseau et de vos identifiants. Pour le fonctionnement du bac à sable et les exigences de plateforme, voir [Sandboxing](https://code.claude.com/docs/en/sandboxing).

### `sandbox`

Isole les commandes Bash que Claude exécute de votre système de fichiers et de votre réseau grâce au [bac à sable](https://code.claude.com/docs/en/sandboxing). Activez le bac à sable avec `enabled`, puis restreignez ou élargissez ce que les commandes en bac à sable peuvent toucher avec les sous-objets `filesystem`, `network` et `credentials`. Le bac à sable fonctionne sous macOS, Linux et WSL2.

* **Portée** : `Any file`
* **Type** : objet avec `enabled`, `failIfUnavailable`, `autoAllowBashIfSandboxed`, `excludedCommands`, `allowUnsandboxedCommands`, `enableWeakerNestedSandbox`, `enableWeakerNetworkIsolation`, `allowAppleEvents`, `bwrapPath`, `socatPath`, `ignoreViolations` et `ripgrep`, plus les objets `filesystem`, `network` et `credentials`
* **Défaut** : non défini ; Claude Code exécute les commandes sans bac à sable

Cet exemple active le bac à sable, supprime les invites de permission pour les commandes en bac à sable, exécute `docker` hors du bac à sable, ouvre deux chemins d'écriture supplémentaires, masque votre fichier d'identifiants AWS et pré-autorise GitHub et npm :

```json settings.json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["docker *"],
    "filesystem": {
      "allowWrite": ["/tmp/build", "~/.kube"],
      "denyRead": ["~/.aws/credentials"]
    },
    "network": {
      "allowedDomains": ["github.com", "*.npmjs.org"]
    }
  }
}
```

Quand les réglages gérés définissent une clé booléenne comme `enabled` ou `failIfUnavailable`, cette valeur l'emporte sur tout ce qu'un développeur définit. Claude Code fusionne les clés de type tableau entre les portées de réglages que la session charge, de sorte qu'un développeur peut ajouter des entrées ; voir [Keep developers from widening the policy](https://code.claude.com/docs/en/sandboxing#keep-developers-from-widening-the-policy) pour les verrous réservés aux réglages gérés. Pour imposer le bac à sable à une organisation, voir [Enforce sandboxing with managed settings](https://code.claude.com/docs/en/sandboxing#enforce-sandboxing-with-managed-settings).

### `sandbox.enabled`

Active le [bac à sable](https://code.claude.com/docs/en/sandboxing) pour les commandes Bash. Lorsque vous choisissez un mode dans le panneau `/sandbox`, Claude Code écrit cette clé dans `.claude/settings.local.json` pour le projet courant ; définissez-la dans `~/.claude/settings.json` pour mettre en bac à sable tous les projets.

* **Portée** : `Any file`, avec des [limites pour les réglages de projet et locaux](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox)
* **Type** : booléen
  * `true` : Claude Code met les commandes Bash en bac à sable
  * `false` : les commandes Bash s'exécutent hors bac à sable
* **Défaut** : `false`

```json settings.json
{
  "sandbox": {
    "enabled": true
  }
}
```

Sous Linux et WSL2, le bac à sable nécessite `bubblewrap` et `socat` ; voir [Set up Linux and WSL2](https://code.claude.com/docs/en/sandboxing#set-up-linux-and-wsl2). Quand le bac à sable ne peut pas démarrer, Claude Code exécute les commandes hors bac à sable, sauf si vous définissez aussi [`failIfUnavailable`](https://code.claude.com/docs/en/settings-reference#sandbox-failifunavailable).

### `sandbox.failIfUnavailable`

Fait quitter Claude Code avec une erreur au démarrage lorsque `sandbox.enabled` vaut `true` mais que le bac à sable ne peut pas démarrer, parce qu'une dépendance manque ou que la plateforme n'est pas prise en charge. Sans cette clé, Claude Code exécute les commandes hors bac à sable. Les déploiements gérés qui exigent le bac à sable comme barrière de sécurité peuvent utiliser ce réglage.

Sur une plateforme que le bac à sable ne prend pas en charge, Claude Code ne démarre pas avec cette clé activée. Voir [Enforce sandboxing with managed settings](https://code.claude.com/docs/en/sandboxing#enforce-sandboxing-with-managed-settings).

* **Portée** : `Any file`, avec des [limites pour les réglages de projet et locaux](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox)
* **Type** : booléen
  * `true` : Claude Code quitte avec une erreur au démarrage lorsque `sandbox.enabled` vaut `true` mais que le bac à sable ne peut pas démarrer
  * `false` : Claude Code exécute les commandes hors bac à sable lorsque le bac à sable ne peut pas démarrer
* **Défaut** : `false`

Cet exemple oblige chaque machine gérée à mettre les commandes en bac à sable ou à refuser de démarrer :

```json managed-settings.json
{
  "sandbox": {
    "enabled": true,
    "failIfUnavailable": true
  }
}
```

Voir [Enforce sandboxing with managed settings](https://code.claude.com/docs/en/sandboxing#enforce-sandboxing-with-managed-settings).

### `sandbox.autoAllowBashIfSandboxed`

Laisse Claude Code exécuter les commandes Bash en bac à sable sans invite de permission. Les commandes qui ne peuvent pas s'exécuter dans le bac à sable passent toujours par le flux de permission habituel, et les règles `deny` ainsi que les règles `ask` ciblées sur un contenu, comme `Bash(git push *)`, s'appliquent toujours ; une règle `ask` portant sur `Bash` seul est ignorée pour les commandes en bac à sable. Mettez `false` pour faire passer aussi les commandes en bac à sable par le flux de permission habituel, ce que l'onglet **Mode** de `/sandbox` appelle le mode « permissions habituelles » (regular permissions mode).

* **Portée** : `Any file`
* **Type** : booléen
  * `true` : Claude Code exécute les commandes Bash en bac à sable sans invite de permission, sauf dans les cas que décrit [Auto-allow mode](https://code.claude.com/docs/en/sandboxing#auto-allow-mode) ; `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` désactive l'auto-autorisation
  * `false` : les commandes en bac à sable passent par le flux de permission habituel, si bien que vos règles d'autorisation et votre mode de permission décident. L'onglet **Mode** de `/sandbox` appelle cela le mode « permissions habituelles »
* **Défaut** : `true`

Cet exemple garde le bac à sable actif et fait passer les commandes en bac à sable par le flux de permission habituel :

```json settings.json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": false
  }
}
```

Voir [Auto-allow mode](https://code.claude.com/docs/en/sandboxing#auto-allow-mode) pour ce sur quoi le mode d'auto-autorisation demande encore confirmation et son comportement en mode plan.

### `sandbox.excludedCommands`

Nomme les commandes que Claude Code exécute hors du bac à sable, comme les outils qui n'y fonctionnent pas. Chaque entrée utilise la même syntaxe que le contenu d'une [règle de permission](https://code.claude.com/docs/en/permissions#permission-rule-syntax) `Bash(...)` : une commande exacte, un préfixe comme `docker *`, ou un motif à joker. Un motif sans joker est une correspondance exacte : `docker` ne correspond qu'à `docker` sans argument.

Vos entrées ne sortent un appel Bash du bac à sable que si elles couvrent **chaque** commande qu'il contient, et certaines formes d'appel restent en bac à sable même dans ce cas. Une entrée `docker *` seule ne sort pas `npm ci && docker build .` du bac à sable.

* **Portée** : `Any file`, avec des [limites pour les réglages de projet et locaux](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox)
* **Type** : tableau de motifs de commandes
* **Défaut** : non défini ; aucune commande n'est exclue

```json settings.json
{
  "sandbox": {
    "excludedCommands": ["docker *"]
  }
}
```

Claude Code garde un appel Bash en bac à sable lorsqu'il a l'une de ces formes, entre autres :

* une commande commençant par `sudo`, `eval` ou `xargs` ;
* un `cd`, `pushd` ou `popd`, où qu'il apparaisse dans l'appel ;
* une substitution de commande, un sous-shell ou un bloc de contrôle comme `if` ou `for` ;
* une redirection, comme `docker build . > build.log`, autre qu'une redirection qui ne fait que dupliquer un descripteur de fichier, comme `2>&1` ;
* un nom de commande qui provient d'une variable ;
* un `git clone`, `git init`, `git worktree add`, `git worktree move` ou `git bundle create` avec un argument de chemin absolu, commençant par `~` ou contenant un segment `..`.

Par exemple, `cd build && docker compose up` reste en bac à sable sous une entrée `docker *`, et ajouter une entrée `cd` n'y change rien. Sous une entrée `git *`, `git clone <url> vendor/lib` s'exécute hors du bac à sable, mais `git clone <url> ~/tools` reste en bac à sable. Un clonage écrit toute une arborescence de fichiers, éventuellement exécutables, là où pointe son chemin de destination.

Les commandes exclues passent toujours par le flux de permission habituel. L'exclusion est une commodité, pas une frontière de sécurité : quand un outil a seulement besoin d'écrire à un endroit précis, [`filesystem.allowWrite`](https://code.claude.com/docs/en/settings-reference#sandbox-filesystem-allowwrite) le garde en bac à sable.

Les entrées des portées de réglages chargées par la session se combinent en une seule liste, sauf si le bac à sable est [exigé par l'administrateur](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox). Dans ce cas, Claude Code ignore les entrées de `.claude/settings.json` et `.claude/settings.local.json`, afin qu'un dépôt cloné ne puisse pas sortir des commandes du bac à sable. Les entrées des réglages gérés, de `--settings` et de votre `~/.claude/settings.json` s'appliquent toujours, et aucun verrou réservé aux réglages gérés ne couvre cette liste.

### `sandbox.allowUnsandboxedCommands`

Permet à Claude de relancer une commande hors du bac à sable avec le paramètre `dangerouslyDisableSandbox` après que le bac à sable l'a bloquée. Quand elle vaut `false`, Claude Code ignore ce paramètre. Tant que le bac à sable tourne, les commandes que Claude exécute sont alors en bac à sable, sauf si elles correspondent à une entrée [`excludedCommands`](https://code.claude.com/docs/en/settings-reference#sandbox-excludedcommands). L'onglet **Overrides** de `/sandbox` affiche cet état sous le nom **Strict sandbox mode**. Un `false` dans les réglages gérés active le mode bac à sable strict pour les développeurs couverts.

* **Portée** : `Any file`, avec [une limite pour les réglages de projet et locaux](https://code.claude.com/docs/en/sandboxing#turn-off-the-retry-with-strict-sandbox-mode)
* **Type** : booléen
  * `true` : Claude peut relancer une commande hors du bac à sable avec le paramètre `dangerouslyDisableSandbox` après que le bac à sable l'a bloquée
  * `false` : Claude Code ignore ce paramètre ; tant que le bac à sable tourne, les commandes que Claude exécute sont en bac à sable, sauf si elles correspondent à une entrée `excludedCommands`
* **Défaut** : `true`

Cet exemple impose le mode bac à sable strict à tous ceux que couvrent les réglages gérés :

```json managed-settings.json
{
  "sandbox": {
    "enabled": true,
    "allowUnsandboxedCommands": false
  }
}
```

Un `false` provenant des réglages gérés ou de `--settings` rend aussi le bac à sable [exigé par l'administrateur](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox). Un `false` dans vos réglages utilisateur tient face à un `true` de projet, mais ne rend pas le bac à sable exigé par l'administrateur. Tenir face à la valeur d'un projet nécessite la v2.1.285+.

Qui approuve une relance hors bac à sable dépend de votre mode de permission et de vos règles d'autorisation. Voir [The unsandboxed retry escape hatch](https://code.claude.com/docs/en/sandboxing#the-unsandboxed-retry-escape-hatch).

Pour savoir quand les commandes que vous tapez vous-même à l'[invite du mode shell `!`](https://code.claude.com/docs/en/interactive-mode#shell-mode-with-prefix) s'exécutent en bac à sable, voir [strict sandbox mode](https://code.claude.com/docs/en/sandboxing#turn-off-the-retry-with-strict-sandbox-mode).

### `sandbox.filesystem`

Contrôle les chemins que les commandes en bac à sable peuvent lire et écrire. Par défaut, elles peuvent écrire dans le répertoire de travail, dans le répertoire temporaire propre à l'utilisateur et dans les répertoires ajoutés avec `--add-dir`, `/add-dir` ou `permissions.additionalDirectories`, et peuvent lire le reste du système de fichiers, **y compris les fichiers d'identifiants**. Élargissez ou restreignez cela avec les quatre listes de chemins, ou désactivez la couche système de fichiers avec `disabled`. Voir [Filesystem isolation](https://code.claude.com/docs/en/sandboxing#filesystem-isolation) pour les limites par défaut.

* **Portée** : `Any file`
* **Type** : objet avec les tableaux `allowWrite`, `denyWrite`, `denyRead` et `allowRead`, plus les booléens `allowManagedReadPathsOnly` et `disabled`
* **Défaut** : non défini ; les limites de lecture et d'écriture par défaut s'appliquent

Cet exemple permet aux commandes en bac à sable d'écrire dans un répertoire de build et dans votre kubeconfig, et masque votre fichier d'identifiants AWS :

```json settings.json
{
  "sandbox": {
    "filesystem": {
      "allowWrite": ["/tmp/build", "~/.kube"],
      "denyRead": ["~/.aws/credentials"]
    }
  }
}
```

Claude Code applique ces listes à la frontière du bac à sable du système d'exploitation : elles s'appliquent donc à chaque sous-processus que lance une commande en bac à sable, comme `kubectl`, `terraform` ou `npm`. Claude Code ajoute vos [règles de permission](https://code.claude.com/docs/en/sandboxing#permission-rules) à ces mêmes listes : les règles d'autorisation et de refus `Edit` à `allowWrite` et `denyWrite`, les règles de refus `Read` à `denyRead`, et les règles d'autorisation et de refus `WebFetch(domain:...)` aux listes de domaines de [`network`](https://code.claude.com/docs/en/settings-reference#sandbox-network).

Sauf si un verrou s'applique, Claude Code fusionne ces listes entre les fichiers de réglages chargés par la session. [`allowManagedReadPathsOnly`](https://code.claude.com/docs/en/settings-reference#sandbox-filesystem-allowmanagedreadpathsonly) limite `allowRead` aux entrées des réglages gérés, et [`allowManagedDomainsOnly`](https://code.claude.com/docs/en/settings-reference#sandbox-network-allowmanageddomainsonly) fait de même pour les domaines autorisés. Les [verrous de dépôt](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox) écartent les entrées des fichiers de réglages d'un dépôt.

[Configure sandboxing](https://code.claude.com/docs/en/sandboxing#configure-sandboxing) traite des sources que vous excluez avec `--setting-sources`. Quand vous modifiez une liste pendant une session, Claude Code [applique le changement à la session en cours](https://code.claude.com/docs/en/settings#when-edits-take-effect).

#### Préfixes de chemins du bac à sable

Les chemins de `allowWrite`, `denyWrite`, `denyRead`, `allowRead` et [`credentials.files`](https://code.claude.com/docs/en/settings-reference#sandbox-credentials-files) se résolvent selon leur préfixe :

| Préfixe | Signification | Exemple |
| :- | :- | :- |
| `/` | Chemin absolu depuis la racine du système de fichiers | `/tmp/build` reste `/tmp/build` |
| `~/` | Relatif au répertoire personnel | `~/.kube` devient `$HOME/.kube` |
| `./` ou aucun préfixe | Relatif à la racine du projet pour les réglages de projet, ou à `~/.claude` pour les réglages utilisateur | `./output` dans `.claude/settings.json` se résout en `<project-root>/output` |

Le préfixe `//path` pour les chemins absolus fonctionne aussi. Si vous utilisez `/path` avec un seul slash en attendant une résolution relative au projet, passez à `./path`. Cette syntaxe diffère de celle des [règles de permission Read et Edit](https://code.claude.com/docs/en/permissions#read-and-edit), qui utilisent `//path` pour l'absolu et `/path` pour le relatif au projet : les chemins de système de fichiers du bac à sable suivent les conventions standard, donc `/tmp/build` est un chemin absolu.

Claude Code retire la barre oblique finale d'un chemin de répertoire : `~/.aws` et `~/.aws/` désignent le même répertoire. Avant la v2.1.224, Claude Code transmettait la barre finale au bac à sable, et Claude pouvait encore lire ou écrire des chemins situés sous une entrée `denyRead` ou `denyWrite` écrite avec une barre finale.

Claude Code retire aussi un `/**` final : `~/build/**` et `~/build` couvrent le même répertoire. Le fonctionnement d'un joker comme `*` dépend de la liste où se trouve l'entrée et de la plateforme :

* **`allowWrite` et `denyWrite`** : sous macOS, les jokers fonctionnent. Sous Linux et WSL2, le bac à sable monte des chemins concrets : Claude Code ignore donc une entrée contenant `*`, `?` ou `[` une fois le `/**` final retiré, et cette entrée n'a aucun effet. Claude Code ajoute à ces listes les chemins de vos règles de permission `Edit`, la même limite s'y applique donc, et l'onglet **Config** de `/sandbox` avertit des règles de permission `Edit` et `Read` qui contiennent des jokers.
* **`denyRead` et `allowRead`** : les jokers fonctionnent sur toutes les plateformes. Sous Linux et WSL2, Claude Code développe une entrée de lecture en chemins concrets correspondants, ce qu'il ne fait pas pour les listes d'écriture.

### `sandbox.filesystem.allowWrite`

Ajoute des chemins où les commandes en bac à sable peuvent écrire, au-delà du répertoire de travail, du répertoire temporaire propre à l'utilisateur et des répertoires ajoutés avec `--add-dir`, `/add-dir` ou `permissions.additionalDirectories`. Utilisez-la quand un sous-processus comme `kubectl` ou un outil de build doit écrire hors du projet.

* **Portée** : `Any file`, avec des [limites pour les réglages de projet et locaux](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox)
* **Type** : tableau de chaînes de chemins, utilisant les [préfixes de chemins du bac à sable](#préfixes-de-chemins-du-bac-à-sable)
* **Défaut** : non défini ; les commandes en bac à sable peuvent écrire dans le répertoire de travail, le répertoire temporaire propre à l'utilisateur, les répertoires ajoutés avec `--add-dir` ou `/add-dir`, et les répertoires de [`permissions.additionalDirectories`](https://code.claude.com/docs/en/settings-reference#permissions-additionaldirectories)

Cet exemple permet à un build d'écrire sous `/tmp/build` et à `kubectl` de mettre à jour votre kubeconfig :

```json settings.json
{
  "sandbox": {
    "filesystem": {
      "allowWrite": ["/tmp/build", "~/.kube"]
    }
  }
}
```

Claude Code fusionne les entrées `allowWrite` et les chemins de vos règles d'autorisation `Edit(...)` entre les portées de réglages chargées par la session, en écartant celles des réglages du dépôt tant que [`permissions.blockReadsOutsideWorkingDirectories`](#commandes-en-bac-à-sable-sous-le-blocage) est actif. Les [verrous de dépôt](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox) peuvent aussi écarter les entrées d'un dépôt. Une entrée `allowWrite` ne peut pas lever la protection d'un [chemin protégé](https://code.claude.com/docs/en/sandboxing#protected-paths).

### `sandbox.filesystem.denyWrite`

Empêche les commandes en bac à sable d'écrire sur des chemins précis, y compris des chemins à l'intérieur d'un répertoire autrement inscriptible.

* **Portée** : `Any file`
* **Type** : tableau de chaînes de chemins, utilisant les [préfixes de chemins du bac à sable](#préfixes-de-chemins-du-bac-à-sable)
* **Défaut** : non défini

Cet exemple empêche les commandes en bac à sable de modifier la configuration système ou d'installer des binaires :

```json settings.json
{
  "sandbox": {
    "filesystem": {
      "denyWrite": ["/etc", "/usr/local/bin"]
    }
  }
}
```

Claude Code fusionne les entrées de toutes les portées de réglages chargées par la session, et ajoute les chemins de vos règles de refus `Edit(...)`.

### `sandbox.filesystem.denyRead`

Empêche les commandes en bac à sable de lire des chemins précis, comme des fichiers d'identifiants que la politique de lecture par défaut exposerait sinon. Pour protéger un fichier d'identifiants tout en le gardant utilisable via le proxy du bac à sable, voir plutôt [`sandbox.credentials`](https://code.claude.com/docs/en/settings-reference#sandbox-credentials).

* **Portée** : `Any file`
* **Type** : tableau de chaînes de chemins, utilisant les [préfixes de chemins du bac à sable](#préfixes-de-chemins-du-bac-à-sable)
* **Défaut** : non défini ; les commandes en bac à sable conservent l'[accès en lecture par défaut](https://code.claude.com/docs/en/sandboxing#filesystem-isolation), qui inclut des fichiers d'identifiants comme `~/.aws/credentials`

```json settings.json
{
  "sandbox": {
    "filesystem": {
      "denyRead": ["~/.aws/credentials"]
    }
  }
}
```

Claude Code fusionne les entrées de toutes les portées de réglages chargées par la session, et ajoute les chemins de vos règles de refus `Read(...)`. Quand [`filesystem.disabled`](https://code.claude.com/docs/en/settings-reference#sandbox-filesystem-disabled) vaut `true`, Claude Code n'applique pas ces entrées.

### `sandbox.filesystem.allowRead`

Rouvre la lecture de chemins précis à l'intérieur d'une zone que [`denyRead`](https://code.claude.com/docs/en/settings-reference#sandbox-filesystem-denyread) bloque, pour construire un accès en lecture limité à l'espace de travail. Une entrée `denyRead` exacte ou à joker reste bloquée à l'intérieur d'un `allowRead` plus large, comme le montre le [tableau des chevauchements](https://code.claude.com/docs/en/sandboxing#configure-sandboxing). Quand une entrée `denyRead` à joker comme `~/**/.env` correspond à un répertoire, Claude Code bloque aussi la lecture de son contenu. Avant la v2.1.236 sous macOS, Claude Code rouvrait les chemins correspondant à une entrée `denyRead` à joker partout où une entrée `allowRead` plus large les couvrait, et laissait lisible le contenu d'un répertoire correspondant.

* **Portée** : `Any file`, avec des [limites pour les réglages de projet et locaux](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox)
* **Type** : tableau de chaînes de chemins, utilisant les [préfixes de chemins du bac à sable](#préfixes-de-chemins-du-bac-à-sable)
* **Défaut** : non défini

Cet exemple bloque la lecture de votre répertoire personnel, à l'exception du projet lui-même :

```json settings.json
{
  "sandbox": {
    "filesystem": {
      "denyRead": ["~/"],
      "allowRead": ["."]
    }
  }
}
```

Claude Code résout une entrée `.` en racine du projet dans les réglages de projet, et en `~/.claude` dans les réglages utilisateur. Claude Code fusionne les entrées des fichiers de réglages chargés par la session, sauf si [`allowManagedReadPathsOnly`](https://code.claude.com/docs/en/settings-reference#sandbox-filesystem-allowmanagedreadpathsonly) est défini, et écarte les entrées des réglages du dépôt tant que [`permissions.blockReadsOutsideWorkingDirectories`](#commandes-en-bac-à-sable-sous-le-blocage) est actif. Les [verrous de dépôt](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox) peuvent aussi écarter les entrées d'un dépôt.

### `sandbox.filesystem.allowManagedReadPathsOnly`

N'honore que les entrées [`allowRead`](https://code.claude.com/docs/en/settings-reference#sandbox-filesystem-allowread) provenant des réglages gérés, afin que les développeurs ne puissent pas rouvrir l'accès en lecture à des chemins que votre organisation a bloqués. Claude Code fusionne toujours les entrées `denyRead` de toutes les portées de réglages chargées par la session.

* **Portée** : `Managed`
* **Type** : booléen
  * `true` : Claude Code n'honore que les entrées `allowRead` des réglages gérés
  * `false` : les entrées `allowRead` des autres fichiers de réglages peuvent être fusionnées
* **Défaut** : `false`

Cet exemple bloque la lecture du répertoire personnel, rouvre `~/work` et empêche les développeurs de rouvrir quoi que ce soit d'autre :

```json managed-settings.json
{
  "sandbox": {
    "filesystem": {
      "denyRead": ["~/"],
      "allowRead": ["~/work"],
      "allowManagedReadPathsOnly": true
    }
  }
}
```

Voir [Keep developers from widening the policy](https://code.claude.com/docs/en/sandboxing#keep-developers-from-widening-the-policy).

### `sandbox.filesystem.disabled`

Supprime l'isolation du système de fichiers tout en conservant l'isolation réseau. Les commandes en bac à sable obtiennent un accès illimité en lecture et en écriture au système de fichiers hôte, et leurs sorties réseau restent confinées à [`network.allowedDomains`](https://code.claude.com/docs/en/settings-reference#sandbox-network-alloweddomains). Utilisez-la quand vous employez le bac à sable pour contrôler où les commandes se connectent plutôt que ce qu'elles écrivent. (v2.1.216+)

* **Portée** : `User or managed`. Quand les réglages gérés configurent `sandbox.filesystem` d'une façon ou d'une autre, ou listent une entrée `sandbox.credentials.files` avec `"mode": "deny"`, seuls les réglages gérés peuvent la définir.
* **Type** : booléen
  * `true` : Claude Code supprime l'isolation du système de fichiers et conserve l'isolation réseau
  * `false` : l'isolation du système de fichiers reste active
* **Défaut** : `false` ; l'isolation du système de fichiers reste active

Cet exemple laisse le système de fichiers ouvert et confine les sorties réseau à GitHub et npm :

```json settings.json
{
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "disabled": true
    },
    "network": {
      "allowedDomains": ["github.com", "*.npmjs.org"]
    }
  }
}
```

Avec cette couche désactivée, Claude Code n'applique ni `denyRead` ni les entrées `deny` de `credentials.files`, tandis que les entrées `credentials.envVars` et les entrées `mask` appliquées continuent de fonctionner. [`autoAllowBashIfSandboxed`](https://code.claude.com/docs/en/settings-reference#sandbox-autoallowbashifsandboxed) vaut toujours `true` par défaut : mettez-la à `false` pour continuer à recevoir des invites. Voir [Disable filesystem isolation](https://code.claude.com/docs/en/sandboxing#disable-filesystem-isolation) pour la liste complète des sources qui peuvent la définir et ce qui change quand l'isolation est désactivée. (v2.1.216+)

### `sandbox.ignoreViolations`

Fait taire les rapports de violation du bac à sable pour des chemins qu'une commande est censée sonder et se voir refuser, comme un outil qui vérifie `/etc/hosts` au démarrage, afin que ces refus n'apparaissent ni comme violations ni dans ce que voit Claude. Le bac à sable bloque toujours l'accès ; seul le rapport est supprimé. Les clés sont des sous-chaînes à comparer à la commande, `*` correspondant à toutes les commandes, et les valeurs sont des sous-chaînes de la violation à ignorer pour cette commande, comme un chemin de fichier.

* **Portée** : `Any file`, avec des [limites pour les réglages de projet et locaux](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox)
* **Type** : objet associant une sous-chaîne de commande à un tableau de sous-chaînes de violation, généralement des chemins
* **Défaut** : non défini ; chaque violation est signalée

```json settings.json
{
  "sandbox": {
    "ignoreViolations": {
      "*": ["/etc/hosts"]
    }
  }
}
```

### `sandbox.enableWeakerNestedSandbox`

Exécute le bac à sable Linux à l'intérieur d'un conteneur Docker non privilégié, où bubblewrap ne peut pas monter un `/proc` neuf. Le bac à sable interne monte alors en bind le `/proc` existant du conteneur, ce qui expose des informations de processus qu'un montage neuf cacherait. Cela **réduit la sécurité** ; ne l'utilisez que lorsque le conteneur externe fournit déjà l'isolation dont vous avez besoin.

* **Portée** : `Any file`, avec des [limites pour les réglages de projet et locaux](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox)
* **Type** : booléen
  * `true` : le bac à sable interne monte en bind le `/proc` existant du conteneur au lieu d'en monter un neuf
  * `false` : le bac à sable monte un `/proc` neuf, ce qui ne fonctionne pas dans un conteneur Docker non privilégié
* **Défaut** : `false`

```json settings.json
{
  "sandbox": {
    "enabled": true,
    "enableWeakerNestedSandbox": true
  }
}
```

Linux et WSL2 uniquement. Voir [Bubblewrap fails to start inside a container](https://code.claude.com/docs/en/sandboxing#bubblewrap-fails-to-start-inside-a-container).

### `sandbox.enableWeakerNetworkIsolation`

Permet aux commandes en bac à sable sous macOS d'atteindre le service système de confiance TLS, `com.apple.trustd.agent`. Les outils écrits en Go comme `gh`, `gcloud` et `terraform` en ont besoin pour vérifier les certificats TLS quand vous utilisez [`network.httpProxyPort`](https://code.claude.com/docs/en/settings-reference#sandbox-network-httpproxyport) avec un proxy MITM et une autorité de certification personnalisée. Cela **réduit la sécurité** en ouvrant une voie potentielle d'exfiltration de données via le service de confiance.

* **Portée** : `Any file`, avec des [limites pour les réglages de projet et locaux](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox)
* **Type** : booléen
  * `true` : les commandes en bac à sable sous macOS peuvent atteindre `com.apple.trustd.agent`
  * `false` : les commandes en bac à sable sous macOS ne peuvent pas atteindre le service système de confiance TLS
* **Défaut** : `false`

```json settings.json
{
  "sandbox": {
    "enabled": true,
    "enableWeakerNetworkIsolation": true
  }
}
```

Si vous n'utilisez pas de proxy MITM, listez plutôt les outils en échec dans [`excludedCommands`](https://code.claude.com/docs/en/settings-reference#sandbox-excludedcommands) ; voir [Go-based CLIs fail TLS verification on macOS](https://code.claude.com/docs/en/sandboxing#go-based-clis-fail-tls-verification-on-macos).

### `sandbox.allowAppleEvents`

Permet aux commandes en bac à sable sous macOS d'envoyer des Apple Events, dont ont besoin `open`, `osascript` et les outils qui ouvrent des URL dans un navigateur ; sans cela, ils échouent avec l'erreur `-600`. Cela **supprime l'isolation de l'exécution de code** : les commandes en bac à sable peuvent lancer d'autres applications hors bac à sable sans aucune invite, et envoyer des commandes AppleScript à des applications en cours d'exécution comme Terminal, sous réserve de l'invite macOS de consentement à l'automatisation par application (TCC).

* **Portée** : `User or managed`
* **Type** : booléen
  * `true` : les commandes en bac à sable sous macOS peuvent envoyer des Apple Events
  * `false` : les commandes en bac à sable sous macOS ne peuvent pas envoyer d'Apple Events, si bien que `open` et `osascript` échouent avec l'erreur `-600`
* **Défaut** : `false`

```json settings.json
{
  "sandbox": {
    "enabled": true,
    "allowAppleEvents": true
  }
}
```

Pour conserver l'isolation tout en exécutant un tel outil, ajoutez-le plutôt à [`excludedCommands`](https://code.claude.com/docs/en/settings-reference#sandbox-excludedcommands). Voir [Apple Events on macOS](https://code.claude.com/docs/en/sandboxing#security-limitations).

### `sandbox.ripgrep`

Fait pointer le bac à sable vers votre propre binaire ripgrep au lieu de celui qu'utilise Claude Code, par exemple quand votre plateforme a besoin d'un `rg` compilé différemment.

* **Portée** : `User or managed`
* **Type** : objet avec `command`, le chemin du binaire ripgrep, et `args` (facultatif), un tableau d'arguments à ajouter en tête
* **Défaut** : non défini ; le bac à sable utilise le même binaire ripgrep que Claude Code, c'est-à-dire le binaire intégré sauf si vous mettez [`USE_BUILTIN_RIPGREP`](https://code.claude.com/docs/en/env-vars) à `0`

```json settings.json
{
  "sandbox": {
    "ripgrep": {
      "command": "/usr/local/bin/rg"
    }
  }
}
```

### `sandbox.bwrapPath`

Fait pointer le bac à sable vers un binaire bubblewrap installé hors du `PATH`, comme une copie embarquée sur un hôte isolé (air-gapped). Claude Code utilise ce chemin à la fois pour la vérification des dépendances au démarrage et quand il enveloppe chaque commande en bac à sable.

* **Portée** : `Managed`. Claude Code ne la lit que depuis les réglages gérés, afin qu'un fichier utilisateur, de projet ou local ne puisse pas faire pointer le bac à sable vers un autre binaire.
* **Type** : chaîne, un chemin absolu ; Claude Code ignore un chemin relatif et se rabat sur la recherche dans le `PATH`
* **Défaut** : non défini ; Claude Code trouve `bwrap` dans le `PATH`

```json managed-settings.json
{
  "sandbox": {
    "enabled": true,
    "bwrapPath": "/opt/admin/bwrap"
  }
}
```

Linux et WSL2 uniquement.

### `sandbox.socatPath`

Fait pointer le proxy réseau du bac à sable vers un binaire `socat` installé hors du `PATH`.

* **Portée** : `Managed`
* **Type** : chaîne, un chemin absolu ; Claude Code ignore un chemin relatif et se rabat sur la recherche dans le `PATH`
* **Défaut** : non défini ; Claude Code trouve `socat` dans le `PATH`

```json managed-settings.json
{
  "sandbox": {
    "enabled": true,
    "socatPath": "/opt/admin/socat"
  }
}
```

Linux et WSL2 uniquement.

### `sandbox.credentials`

Déclare les fichiers d'identifiants et les variables d'environnement à [protéger des commandes en bac à sable](https://code.claude.com/docs/en/sandboxing#protect-credentials). Chaque entrée nomme un fichier (`path`) ou une variable (`name`) et un `mode` : `deny` cache l'identifiant à l'intérieur du bac à sable, et `mask` montre aux commandes en bac à sable une valeur factice tandis que le [proxy du bac à sable](https://code.claude.com/docs/en/sandboxing#mask-credentials) substitue la vraie valeur dans les requêtes sortantes. Claude Code ne protège que les entrées que vous listez ; il n'existe **aucune** liste intégrée d'identifiants à refuser.

* **Portée** : `Any file`. Claude Code n'honore les entrées `mask`, `allowPlaintextInject`, `awsPairs` et `sigv4` que depuis les réglages utilisateur, les réglages gérés et le flag `--settings`.
* **Type** : objet avec `files`, `envVars`, `allowPlaintextInject`, `awsPairs` et `sigv4`
* **Défaut** : non défini ; aucun identifiant n'est protégé

Cet exemple masque votre fichier d'identifiants AWS et retire `GITHUB_TOKEN` de l'environnement des commandes en bac à sable :

```json settings.json
{
  "sandbox": {
    "credentials": {
      "files": [{ "path": "~/.aws/credentials", "mode": "deny" }],
      "envVars": [{ "name": "GITHUB_TOKEN", "mode": "deny" }]
    }
  }
}
```

La protection `deny` des fichiers fait partie de la couche système de fichiers : elle ne s'applique donc pas quand vous [désactivez l'isolation du système de fichiers](https://code.claude.com/docs/en/sandboxing#disable-filesystem-isolation) ; la protection des variables d'environnement, elle, s'applique toujours.

#### Entrées d'identifiants invalides dans les réglages gérés

Quand une entrée `sandbox.credentials` gérée échoue à la validation, Claude Code continue de protéger l'identifiant là où il le peut :

* Une entrée de `files` ou `envVars` qui a toujours un `path` ou un `name` valide et un `mode` valant `mask` ou `deny` — par exemple une entrée dont le motif `extract` n'a pas de groupe capturant — est rétrogradée en `mode: "deny"` avec un avertissement : l'identifiant reste bloqué, et non masqué, jusqu'à ce que vous corrigiez l'entrée. Une entrée `files` rétrogradée verrouille [`filesystem.disabled`](https://code.claude.com/docs/en/sandboxing#disable-filesystem-isolation) comme le ferait une entrée `deny` explicite, et l'avertissement précise que son blocage de lecture n'est pas appliqué si les réglages gérés désactivent l'isolation du système de fichiers.
* Une entrée avec un `mode` inconnu ou un `path` ou `name` invalide est supprimée.
* Chaque cas émet un avertissement ; qu'une entrée soit rétrogradée ou supprimée, les autres entrées valides restent appliquées.

S'applique en v2.1.191+ ; avant la v2.1.221, toute entrée invalide était supprimée. Pour les autres clés gérées traitées champ par champ, voir [Invalid entries in managed settings](https://code.claude.com/docs/en/managed-settings#invalid-entries-in-managed-settings).

### `sandbox.credentials.files`

Protège des fichiers ou répertoires d'identifiants contre les commandes en bac à sable. Avec `"mode": "deny"`, Claude Code bloque la lecture du chemin à l'intérieur du bac à sable — le même blocage de lecture que [`sandbox.filesystem.denyRead`](https://code.claude.com/docs/en/settings-reference#sandbox-filesystem-denyread). Avec `"mode": "mask"`, les commandes en bac à sable sous Linux et WSL2 lisent une copie sentinelle du fichier, et le proxy du bac à sable substitue la vraie valeur dans les requêtes sortantes vers les `injectHosts` de cette entrée ; sous macOS, le fichier est au contraire illisible dans le bac à sable. `"mode": "mask"` nécessite la v2.1.221+.

* **Portée** : `Any file`. Claude Code abandonne les entrées `mask` du `.claude/settings.json` de projet et du `.claude/settings.local.json` local.
* **Type** : tableau d'objets, chacun avec `path` et un `mode` valant `"deny"` ou `"mask"`, plus les [champs de masquage facultatifs pour les fichiers](#champs-de-masquage-pour-les-fichiers)
* **Défaut** : non défini ; aucun fichier d'identifiants n'est protégé

Cet exemple cache votre fichier d'identifiants AWS et masque le fichier d'hôtes de `gh`, en ne substituant la vraie valeur que dans les requêtes vers `api.github.com` :

```json settings.json
{
  "sandbox": {
    "credentials": {
      "files": [
        { "path": "~/.aws/credentials", "mode": "deny" },
        { "path": "~/.config/gh/hosts.yml", "mode": "mask", "injectHosts": ["api.github.com"] }
      ]
    }
  }
}
```

Les chemins utilisent les mêmes [préfixes](#préfixes-de-chemins-du-bac-à-sable) que les réglages `sandbox.filesystem.*`, et Claude Code fusionne les tableaux de toutes les portées de réglages chargées par la session. [Protect credentials](https://code.claude.com/docs/en/sandboxing#protect-credentials) indique ce qui s'applique encore depuis les sources que vous excluez avec `--setting-sources`. Les entrées `mask` nécessitent la v2.1.221+.

La substitution `mask` ne passe que par le proxy du bac à sable : définissez donc [`sandbox.network.tlsTerminate`](https://code.claude.com/docs/en/settings-reference#sandbox-network-tlsterminate), ou [`allowPlaintextInject`](https://code.claude.com/docs/en/settings-reference#sandbox-credentials-allowplaintextinject) pour des réseaux de test en HTTP simple. `mask` s'applique à un seul fichier : listez donc chaque fichier d'identifiants individuellement. Claude Code accepte mais ignore les champs `mask` sur une entrée `deny`. [Mask credentials](https://code.claude.com/docs/en/sandboxing#mask-credentials) indique quelles sources de réglages sont honorées, et [Mask credential files](https://code.claude.com/docs/en/sandboxing#mask-credential-files) indique quand une entrée se rabat sur `deny`.

#### Champs de masquage pour les fichiers

*(Ancres d'origine : `sandbox-credentials-files-extract`, `-onextractnomatch`, `-decode`, `-maskclaims`, `-maskduplicates`, `-injecthosts`.)*

Une entrée `mask` accepte ces champs facultatifs. Sans `extract` ni `decode`, Claude Code remplace tout le contenu du fichier par une seule sentinelle. Sous macOS avec l'isolation du système de fichiers active, Claude Code applique une entrée `mask` comme un `deny` avant même que `extract` ou `decode` ne s'exécute ; voir [Mask credential files](https://code.claude.com/docs/en/sandboxing#mask-credential-files).

| Champ | Type | Rôle |
| :- | :- | :- |
| `extract` | chaîne, une expression régulière avec au moins un groupe capturant | Ne masque que le texte capturé par le groupe 1 de chaque correspondance, pour que le reste du fichier reste analysable. Si `decode` est aussi défini, Claude Code examine chaque capture comme un JWT potentiel au lieu de la remplacer directement. (v2.1.221+) |
| `onExtractNoMatch` | `"warn"`, `"deny"` ou `"error"` ; défaut `"warn"` | Ce qui se passe quand `extract` ou `decode` ne trouve rien à masquer. `warn` laisse le fichier lisible tel quel dans le bac à sable, `deny` le rend illisible, et `error` arrête la mise en place du bac à sable jusqu'à ce que vous corrigiez la configuration. Claude Code traite `deny` comme `error` quand le blocage de lecture ne serait pas appliqué, parce que vous [désactivez l'isolation du système de fichiers](https://code.claude.com/docs/en/sandboxing#disable-filesystem-isolation) ou qu'une entrée [`sandbox.filesystem.allowRead`](https://code.claude.com/docs/en/settings-reference#sandbox-filesystem-allowread) rouvre le chemin. (v2.1.221+ ; le cas `decode` nécessite la v2.1.224+) |
| `decode` | la chaîne `"jwt"` | Recherche les JSON Web Tokens (JWT) dans le fichier, avec un motif intégré ou avec `extract` s'il est défini, vérifie chaque candidat et le remplace par un faux jeton structurellement valide, pour que le code qui décode le jeton dans le bac à sable continue de fonctionner. Si aucun candidat n'est vérifié, `onExtractNoMatch` détermine l'issue. (v2.1.224+) |
| `maskClaims` | tableau de chaînes, au moins un nom de claim ; nécessite `decode` | Ne masque que les claims de premier niveau nommés dans la charge utile de chaque JWT vérifié et reconstruit le jeton autour de la charge modifiée, pour que les autres claims restent lisibles. Si aucun claim nommé ne correspond, `onExtractNoMatch` détermine l'issue. (v2.1.224+) |
| `maskDuplicates` | booléen, défaut `false` | Remplace aussi les copies textuelles de chaque valeur masquée ailleurs dans le fichier, comme un secret collé dans un commentaire. Claude Code compare des sous-chaînes brutes : réservez-le aux secrets longs à forte entropie. Consulté seulement quand `extract` ou `decode` est défini. (v2.1.221+) |
| `injectHosts` | tableau de chaînes, chacune un hôte que [`sandbox.network.allowedDomains`](https://code.claude.com/docs/en/settings-reference#sandbox-network-alloweddomains) admet aussi | Restreint les hôtes pour lesquels le proxy du bac à sable substitue la vraie valeur. Si non défini, le proxy la substitue dans les requêtes vers tous les hôtes de `sandbox.network.allowedDomains`. (v2.1.221+) |

Cet exemple ne masque que la valeur `oauth_token` du fichier d'hôtes de `gh`, remplace toutes les autres copies de cette valeur dans le fichier, rend le fichier illisible si le motif ne trouve rien, et ne substitue le vrai jeton que dans les requêtes vers `api.github.com` :

```json settings.json
{
  "sandbox": {
    "credentials": {
      "files": [
        {
          "path": "~/.config/gh/hosts.yml",
          "mode": "mask",
          "extract": "oauth_token:\\s*(\\S+)",
          "maskDuplicates": true,
          "onExtractNoMatch": "deny",
          "injectHosts": ["api.github.com"]
        }
      ]
    }
  }
}
```

### `sandbox.credentials.envVars`

Protège des variables d'environnement contre les commandes en bac à sable. Avec `"mode": "deny"`, Claude Code retire la variable de l'environnement des commandes en bac à sable. Avec `"mode": "mask"`, les commandes en bac à sable voient une valeur sentinelle propre à la session, et le proxy du bac à sable substitue la vraie valeur dans les requêtes sortantes vers les `injectHosts` de cette entrée : des outils comme `gh` et `npm` continuent ainsi de s'authentifier sans jamais détenir le vrai identifiant.

* **Portée** : `Any file`. Claude Code abandonne les entrées `mask` du `.claude/settings.json` de projet et du `.claude/settings.local.json` local.
* **Type** : tableau d'objets, chacun avec `name` et un `mode` valant `"deny"` ou `"mask"`, plus les [champs de masquage facultatifs pour les variables d'environnement](#champs-de-masquage-pour-les-variables-denvironnement)
* **Défaut** : non défini ; aucune variable d'environnement n'est protégée

Cet exemple retire `NPM_TOKEN` des commandes en bac à sable et masque `GITHUB_TOKEN`, en ne substituant la vraie valeur que dans les requêtes vers `api.github.com` :

```json settings.json
{
  "sandbox": {
    "credentials": {
      "envVars": [
        { "name": "NPM_TOKEN", "mode": "deny" },
        { "name": "GITHUB_TOKEN", "mode": "mask", "injectHosts": ["api.github.com"] }
      ]
    }
  }
}
```

Le `name` doit commencer par une lettre ou un tiret bas et ne contenir que des lettres, des chiffres et des tirets bas. Claude Code fusionne les tableaux de toutes les portées de réglages chargées par la session, et applique `deny` lorsqu'une même variable apparaît avec les deux modes. [Protect credentials](https://code.claude.com/docs/en/sandboxing#protect-credentials) indique ce qui s'applique encore depuis les sources que vous excluez avec `--setting-sources`.

La substitution `mask` ne passe que par le proxy du bac à sable : définissez donc [`sandbox.network.tlsTerminate`](https://code.claude.com/docs/en/settings-reference#sandbox-network-tlsterminate), ou [`allowPlaintextInject`](https://code.claude.com/docs/en/settings-reference#sandbox-credentials-allowplaintextinject) pour des réseaux de test en HTTP simple ; voir [Mask credentials](https://code.claude.com/docs/en/sandboxing#mask-credentials). Claude Code accepte mais ignore les champs `mask` sur une entrée `deny`.

#### Champs de masquage pour les variables d'environnement

*(Ancres d'origine : `sandbox-credentials-envvars-extract`, `-onextractnomatch`, `-decode`, `-maskclaims`, `-injecthosts`.)*

Une entrée `mask` accepte ces champs facultatifs. Sans `extract` ni `decode`, Claude Code remplace toute la valeur par une seule sentinelle. `extract` et `decode` ne peuvent pas être combinés sur la même entrée.

| Champ | Type | Rôle |
| :- | :- | :- |
| `extract` | chaîne, une expression régulière avec au moins un groupe capturant | Ne masque que le texte capturé par le groupe 1 de chaque correspondance, comme le mot de passe dans une chaîne de connexion `DATABASE_URL`, pour que le reste de la valeur reste analysable. (v2.1.224+) |
| `onExtractNoMatch` | `"warn"`, `"deny"` ou `"error"` ; défaut `"warn"`. Sur une entrée avec `decode`, seul `"warn"` est accepté | Ce qui se passe quand `extract` ne trouve rien. `warn` transmet la variable sans masquage, `deny` la supprime (unset) dans le bac à sable, et `error` arrête la mise en place du bac à sable jusqu'à ce que vous corrigiez la configuration. (v2.1.224+) |
| `decode` | la chaîne `"jwt"` | Vérifie que la valeur entière est un JWT et la remplace par un faux jeton structurellement valide, pour que le code qui décode le jeton dans le bac à sable continue de fonctionner ; le proxy substitue le vrai jeton entier en sortie. Une valeur qui ne se vérifie pas est transmise sans masquage, avec un avertissement. (v2.1.224+) |
| `maskClaims` | tableau de chaînes, au moins un nom de claim ; nécessite `decode` | Ne masque que les claims de premier niveau nommés dans la charge utile du JWT décodé et reconstruit le jeton autour de la charge modifiée, pour que les autres claims restent lisibles. Si aucun claim nommé ne correspond, la variable est transmise sans masquage, avec un avertissement. (v2.1.224+) |
| `injectHosts` | tableau de chaînes, chacune un hôte que [`sandbox.network.allowedDomains`](https://code.claude.com/docs/en/settings-reference#sandbox-network-alloweddomains) admet aussi | Restreint les hôtes pour lesquels le proxy du bac à sable substitue la vraie valeur. Si non défini, le proxy la substitue dans les requêtes vers tous les hôtes de `sandbox.network.allowedDomains`. Écrivez une destination IPv6 sous forme d'adresse compressée nue, comme `"::1"`, et non entre crochets ; voir [IPv6 destinations in `injectHosts`](https://code.claude.com/docs/en/sandboxing#ipv6-destinations-in-injecthosts) |

Cet exemple ne masque que le mot de passe dans `DATABASE_URL`, supprime la variable si le motif ne trouve rien, et masque un JWT dans `SERVICE_JWT` en laissant lisibles tous les claims sauf `api_key` :

```json settings.json
{
  "sandbox": {
    "credentials": {
      "envVars": [
        {
          "name": "DATABASE_URL",
          "mode": "mask",
          "extract": "://[^:]+:([^@]+)@",
          "onExtractNoMatch": "deny"
        },
        {
          "name": "SERVICE_JWT",
          "mode": "mask",
          "decode": "jwt",
          "maskClaims": ["api_key"]
        }
      ]
    }
  }
}
```

### `sandbox.credentials.allowPlaintextInject`

Autorise la substitution `mask` dans les requêtes HTTP en clair, en plus du HTTPS à terminaison TLS. En HTTP simple, l'identité du serveur en amont n'est pas vérifiée et l'identifiant circule en clair : **laissez ce réglage désactivé** hors des réseaux de test de confiance.

* **Portée** : `User or managed`
* **Type** : booléen
  * `true` : Claude Code autorise la substitution `mask` dans les requêtes HTTP en clair en plus du HTTPS à terminaison TLS
  * `false` : Claude Code n'autorise la substitution `mask` que dans le HTTPS à terminaison TLS
* **Défaut** : `false`

```json settings.json
{
  "sandbox": {
    "credentials": {
      "allowPlaintextInject": true
    }
  }
}
```

### `sandbox.credentials.awsPairs`

Regroupe des variables d'environnement masquées qui forment un même identifiant AWS, pour la [re-signature SigV4](https://code.claude.com/docs/en/sandboxing#re-sign-aws-requests), quand votre identifiant se trouve dans des variables aux noms non standard. Claude Code associe automatiquement le trio conventionnel `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` et `AWS_SESSION_TOKEN` lorsque vous masquez leurs valeurs entières : cette clé n'est donc nécessaire que pour d'autres noms. (v2.1.224+)

* **Portée** : `User or managed`
* **Type** : tableau d'objets, chacun avec `accessKeyIdVar`, `secretAccessKeyVar` et éventuellement `sessionTokenVar`, nommant des entrées de `sandbox.credentials.envVars`
* **Défaut** : non défini ; seul le trio conventionnel est associé

Cet exemple associe trois variables aux noms personnalisés en un seul identifiant AWS pour la re-signature :

```json settings.json
{
  "sandbox": {
    "credentials": {
      "awsPairs": [
        {
          "accessKeyIdVar": "MY_KEY_ID",
          "secretAccessKeyVar": "MY_SECRET_KEY",
          "sessionTokenVar": "MY_SESSION_TOKEN"
        }
      ]
    }
  }
}
```

Chaque variable nommée doit être une entrée `mask` à valeur entière dans [`sandbox.credentials.envVars`](https://code.claude.com/docs/en/settings-reference#sandbox-credentials-envvars), sans `extract` ni `decode`, et ne peut occuper qu'un seul emplacement sur l'ensemble des paires. Ces règles s'appliquent aussi :

* le proxy re-signe les requêtes vers les hôtes listés dans les `injectHosts` de l'entrée de l'identifiant de clé d'accès ;
* quand `sessionTokenVar` est défini, le proxy envoie le vrai jeton dans `x-amz-security-token` sur les requêtes re-signées ;
* nommer l'une des variables conventionnelles dans une paire remplace l'association automatique.

### `sandbox.credentials.sigv4`

Choisit ce que fait le proxy du bac à sable des formes de requêtes AWS qu'il [ne peut pas re-signer](https://code.claude.com/docs/en/sandboxing#re-sign-aws-requests) : `streaming` pour les téléversements en streaming aws-chunked, `presigned` pour les URL pré-signées, et `sigv4a` pour les signatures asymétriques SigV4A. Cela ne s'applique qu'aux requêtes signées avec l'identifiant de clé d'accès factice d'une paire masquée. (v2.1.224+)

* **Portée** : `User or managed`
* **Type** : objet avec `streaming`, `presigned` et `sigv4a`, chacun valant l'une de :
  * `"deny"` : le proxy fait échouer la requête
  * `"passthrough"` : le proxy transmet la requête signée avec la valeur factice masquée, si bien que l'outil reçoit le propre rejet d'AWS
* **Défaut** : non défini ; chaque forme vaut `"deny"`

Cet exemple transmet les téléversements en streaming au lieu de les faire échouer au niveau du proxy :

```json settings.json
{
  "sandbox": {
    "credentials": {
      "sigv4": {
        "streaming": "passthrough"
      }
    }
  }
}
```

Avec `deny`, le proxy fait échouer la requête. Avec `passthrough`, le proxy transmet la requête avec sa signature calculée à partir de la valeur factice masquée : AWS la rejette, et l'outil appelant reçoit la réponse d'AWS elle-même au lieu d'une erreur du proxy.

### `sandbox.network`

Contrôle quels hôtes, ports et sockets les commandes en bac à sable peuvent atteindre. Le bac à sable fait passer le trafic sortant par un proxy qui applique ces listes ; voir [Network isolation](https://code.claude.com/docs/en/sandboxing#network-isolation) pour la façon dont le proxy décide et quand il demande confirmation.

* **Portée** : `Any file`. `strictAllowlist`, `allowManagedDomainsOnly` et `tlsTerminate` sont lues depuis moins de sources, comme l'indiquent leurs entrées.
* **Type** : objet avec les sous-clés ci-dessous
* **Défaut** : non défini ; aucun domaine n'est pré-autorisé et votre mode de permission décide [de ce qui arrive à chaque nouvel hôte](https://code.claude.com/docs/en/sandboxing#hosts-outside-your-allowed-domains)

Cet exemple pré-autorise GitHub et npm, bloque `uploads.github.com` et permet aux commandes d'écouter sur localhost :

```json settings.json
{
  "sandbox": {
    "network": {
      "allowedDomains": ["github.com", "*.npmjs.org"],
      "deniedDomains": ["uploads.github.com"],
      "allowLocalBinding": true
    }
  }
}
```

Claude Code fusionne les sous-clés de type tableau entre les portées de réglages : un projet peut donc ajouter des domaines à votre liste utilisateur, sauf si un [verrou de dépôt](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox) s'applique. Les [règles de permission](https://code.claude.com/docs/en/sandboxing#permission-rules) d'autorisation et de refus `WebFetch(domain:...)` alimentent les mêmes listes d'autorisation et de refus.

### `sandbox.network.allowUnixSockets`

Liste les chemins de sockets Unix auxquels les commandes en bac à sable peuvent se connecter sous macOS. Claude Code ignore cette liste sous Linux et WSL2, où le filtre seccomp ne peut pas inspecter les chemins de sockets ; utilisez-y plutôt [`allowAllUnixSockets`](https://code.claude.com/docs/en/settings-reference#sandbox-network-allowallunixsockets).

* **Portée** : `Any file`, avec des [limites pour les réglages de projet et locaux](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox)
* **Type** : tableau de chaînes, chacune un chemin de socket
* **Défaut** : non défini ; le bac à sable macOS bloque tous les sockets Unix

```json settings.json
{
  "sandbox": {
    "network": {
      "allowUnixSockets": ["~/.ssh/agent-socket"]
    }
  }
}
```

Un chemin de socket peut accorder un accès étendu : autoriser `/var/run/docker.sock`, par exemple, permet à une commande en bac à sable de contrôler le démon Docker. Voir [Security limitations](https://code.claude.com/docs/en/sandboxing#security-limitations).

### `sandbox.network.allowAllUnixSockets`

Permet aux commandes en bac à sable de se connecter à n'importe quel socket Unix. Sous Linux et WSL2, le [filtre seccomp](https://code.claude.com/docs/en/sandboxing#set-up-linux-and-wsl2) du bac à sable bloque les appels `socket(AF_UNIX, ...)` : c'est donc le seul moyen d'y autoriser les sockets Unix. Quand le filtre est absent, ce que `/sandbox` signale dans son onglet Dependencies, le bac à sable ne bloque pas les appels de sockets Unix. Voir [Set up Linux and WSL2](https://code.claude.com/docs/en/sandboxing#set-up-linux-and-wsl2) pour l'origine du filtre.

* **Portée** : `Any file`, avec des [limites pour les réglages de projet et locaux](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox)
* **Type** : booléen
  * `true` : les commandes en bac à sable peuvent se connecter à n'importe quel socket Unix
  * `false` : le bac à sable bloque les connexions aux sockets Unix : sous macOS sauf pour les chemins de `allowUnixSockets`, et sous Linux et WSL2 via le filtre seccomp lorsqu'il est présent
* **Défaut** : `false`

```json settings.json
{
  "sandbox": {
    "network": {
      "allowAllUnixSockets": true
    }
  }
}
```

Sous WSL2, `true` rouvre aussi le socket d'interopérabilité qui lance des binaires Windows comme `cmd.exe` et `powershell.exe`.

### `sandbox.network.allowLocalBinding`

Permet aux commandes en bac à sable sous macOS d'écouter sur des ports réseau, par exemple pour lancer un serveur de développement, et de se connecter à n'importe quel port de localhost. Une commande qui écoute sur une adresse autre que la boucle locale accepte des connexions d'autres machines. La clé n'a aucun effet sous Linux et WSL2, où chaque commande en bac à sable a sa propre interface de boucle locale. Pour atteindre un serveur de l'hôte depuis Linux ou WSL2, voir [A command fails to reach a server on localhost](https://code.claude.com/docs/en/sandboxing#a-command-fails-to-reach-a-server-on-localhost).

* **Portée** : `Any file`, avec des [limites pour les réglages de projet et locaux](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox)
* **Type** : booléen
  * `true` : les commandes en bac à sable sous macOS peuvent écouter sur n'importe quelle adresse locale et se connecter à n'importe quel port de localhost
  * `false` : les commandes en bac à sable sous macOS ne peuvent ni écouter sur un port ni se connecter directement aux serveurs de localhost
* **Défaut** : `false`

```json settings.json
{
  "sandbox": {
    "network": {
      "allowLocalBinding": true
    }
  }
}
```

### `sandbox.network.allowMachLookup`

Liste des noms de services XPC et Mach supplémentaires que le bac à sable macOS peut rechercher. Les outils qui communiquent via XPC, comme le simulateur iOS ou Playwright, ont besoin que leurs services soient listés ici.

* **Portée** : `Any file`, avec des [limites pour les réglages de projet et locaux](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox)
* **Type** : tableau de chaînes, chacune un nom de service ; un unique `*` final correspond à un préfixe, et `"*"` seul correspond à tous les services
* **Défaut** : non défini

Cet exemple autorise tous les services sous le préfixe `com.apple.coresimulator.` :

```json settings.json
{
  "sandbox": {
    "network": {
      "allowMachLookup": ["com.apple.coresimulator.*"]
    }
  }
}
```

### `sandbox.network.allowedDomains`

Pré-autorise des domaines pour le trafic sortant des commandes en bac à sable, afin que le bac à sable ne demande pas confirmation pour eux. Les jokers comme `*.example.com` correspondent aux sous-domaines, et un suffixe `:port` facultatif limite une entrée à un seul port ; une entrée sans port correspond à tous les ports.

* **Portée** : `Any file`, avec des [limites pour les réglages de projet et locaux](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox). Uniquement les réglages gérés lorsque [`allowManagedDomainsOnly`](https://code.claude.com/docs/en/settings-reference#sandbox-network-allowmanageddomainsonly) est défini.
* **Type** : tableau de chaînes, chacune un domaine, un motif à joker ou une adresse IP littérale, avec un suffixe `:port` facultatif
* **Défaut** : non défini ; votre mode de permission décide [de ce qui arrive à chaque nouvel hôte](https://code.claude.com/docs/en/sandboxing#hosts-outside-your-allowed-domains)

Cet exemple pré-autorise GitHub sur tous les ports, tous les sous-domaines npm, et un hôte d'API sur le port 443 uniquement :

```json settings.json
{
  "sandbox": {
    "network": {
      "allowedDomains": ["github.com", "*.npmjs.org", "api.example.com:443"]
    }
  }
}
```

Écrivez les adresses IPv6 littérales entre crochets, avec un port facultatif : `"[::1]"` autorise tous les ports et `"[::1]:443"` un seul port. La forme entre crochets nécessite la v2.1.229+. Voir [IPv6 addresses in domain lists](https://code.claude.com/docs/en/sandboxing#ipv6-addresses-in-domain-lists).

### `sandbox.network.deniedDomains`

Bloque des domaines pour le trafic sortant des commandes en bac à sable, avec la même syntaxe de jokers, de port et d'IPv6 que [`allowedDomains`](https://code.claude.com/docs/en/settings-reference#sandbox-network-alloweddomains). Un domaine refusé reste bloqué même si une entrée `allowedDomains` lui correspond aussi.

* **Portée** : `Any file`
* **Type** : tableau de chaînes, chacune un domaine, un motif à joker ou une adresse IP littérale, avec un suffixe `:port` facultatif
* **Défaut** : non défini

```json settings.json
{
  "sandbox": {
    "network": {
      "deniedDomains": ["sensitive.cloud.example.com"]
    }
  }
}
```

Claude Code fusionne cette liste depuis toutes les sources de réglages chargées par la session, même quand `allowManagedDomainsOnly` est défini : un développeur peut donc toujours durcir la liste de refus. Pour les adresses IPv6 littérales, voir [IPv6 addresses in domain lists](https://code.claude.com/docs/en/sandboxing#ipv6-addresses-in-domain-lists).

Une entrée écrite avec le point final qui marque un nom de domaine pleinement qualifié (FQDN), comme `example.com.`, bloque les mêmes connexions que `example.com`.

### `sandbox.network.strictAllowlist`

Refuse aux commandes en bac à sable l'accès aux hôtes hors de la liste d'autorisation au lieu de demander une approbation. La liste d'autorisation est [`allowedDomains`](https://code.claude.com/docs/en/settings-reference#sandbox-network-alloweddomains) plus les domaines des règles d'autorisation `WebFetch(domain:...)`, ou uniquement les entrées des réglages gérés lorsque [`allowManagedDomainsOnly`](https://code.claude.com/docs/en/settings-reference#sandbox-network-allowmanageddomainsonly) est défini. [Locks that apply without an admin-required sandbox](https://code.claude.com/docs/en/sandboxing#locks-that-apply-without-an-admin-required-sandbox) traite des entrées d'un dépôt. (v2.1.219+)

* **Portée** : `User or managed`. Un dépôt ne peut ni l'activer ni la désactiver.
* **Type** : booléen
  * `true` : Claude Code refuse aux commandes en bac à sable l'accès aux hôtes hors de la liste d'autorisation
  * `false` : sauf si un autre fichier de réglages de confiance définit `true`, Claude Code décide du sort d'un hôte hors liste selon le mode de permission au lieu de le refuser d'office : en mode auto, il compare l'hôte aux [domaines autorisés par commande](https://code.claude.com/docs/en/sandboxing#per-command-allowed-domains-in-auto-mode) ; en mode `dontAsk`, il refuse ; en mode `bypassPermissions` et dans les sessions interactives en terminal en mode plan où le contournement est disponible, il autorise ; sinon, il vous demande
* **Défaut** : `false`

```json settings.json
{
  "sandbox": {
    "network": {
      "strictAllowlist": true
    }
  }
}
```

Claude Code n'applique ce réglage qu'aux commandes en bac à sable ; les outils internes au processus comme `WebFetch` suivent toujours leurs [règles de permission](https://code.claude.com/docs/en/sandboxing#permission-rules). Dès qu'une des sources honorées la met à `true`, elle reste active. Voir [Network isolation](https://code.claude.com/docs/en/sandboxing#network-isolation). (v2.1.219+)

### `sandbox.network.allowManagedDomainsOnly`

Verrouille la liste d'autorisation réseau sur ce que définissent les réglages gérés. Claude Code n'honore alors que les `allowedDomains` et les règles d'autorisation `WebFetch(domain:...)` des réglages gérés, ignore les domaines des réglages utilisateur, projet, local et `--settings`, et bloque automatiquement un domaine non autorisé au lieu de demander confirmation.

* **Portée** : `Managed`
* **Type** : booléen
  * `true` : Claude Code n'honore que les `allowedDomains` et les règles d'autorisation `WebFetch(domain:...)` des réglages gérés, et bloque un domaine non autorisé au lieu de demander
  * `false` : les domaines des autres fichiers de réglages peuvent être fusionnés dans la liste d'autorisation
* **Défaut** : `false`

Cet exemple verrouille la liste d'autorisation sur GitHub et npm et ignore tout domaine ajouté par les développeurs :

```json managed-settings.json
{
  "sandbox": {
    "network": {
      "allowManagedDomainsOnly": true,
      "allowedDomains": ["github.com", "*.npmjs.org"]
    }
  }
}
```

Tant que la clé vaut `true`, le bac à sable est [exigé par l'administrateur](https://code.claude.com/docs/en/sandboxing#repository-settings-under-an-admin-required-sandbox), et seuls les réglages gérés peuvent définir un [port de proxy](https://code.claude.com/docs/en/settings-reference#sandbox-network-httpproxyport).

Les domaines refusés sont toujours fusionnés depuis toutes les sources chargées par la session. Voir [Keep developers from widening the policy](https://code.claude.com/docs/en/sandboxing#keep-developers-from-widening-the-policy).

### `sandbox.network.httpProxyPort`

Fait pointer le bac à sable vers votre propre proxy HTTP au lieu de celui que lance Claude Code. Les organisations le font pour inspecter le trafic HTTPS, appliquer leurs propres règles de filtrage ou journaliser les requêtes. Votre proxy prend en charge le filtrage, et Claude Code cesse d'appliquer ses listes de domaines et ses invites réseau au trafic qui y est envoyé. Si non défini, Claude Code démarre son propre proxy pour le trafic HTTP.

* **Portée** : `Any file`, sauf si [d'autres réglages du bac à sable limitent les fichiers pouvant définir un port](https://code.claude.com/docs/en/sandboxing#custom-proxy-configuration)
* **Type** : nombre, un port TCP local
* **Défaut** : non défini ; Claude Code exécute son propre proxy

```json settings.json
{
  "sandbox": {
    "network": {
      "httpProxyPort": 8080
    }
  }
}
```

Définissez aussi [`socksProxyPort`](https://code.claude.com/docs/en/settings-reference#sandbox-network-socksproxyport) si votre proxy doit également transporter le trafic SOCKS ; avec un seul des deux défini, Claude Code exécute toujours son propre proxy pour l'autre protocole. Voir [Custom proxy configuration](https://code.claude.com/docs/en/sandboxing#custom-proxy-configuration).

### `sandbox.network.socksProxyPort`

Fait pointer le bac à sable vers votre propre proxy SOCKS5 au lieu de celui que lance Claude Code. Votre proxy prend en charge le filtrage, et Claude Code cesse d'appliquer ses listes de domaines et ses invites réseau au trafic qui y est envoyé. Si non défini, Claude Code démarre son propre proxy pour le trafic SOCKS.

* **Portée** : `Any file`, sauf si [d'autres réglages du bac à sable limitent les fichiers pouvant définir un port](https://code.claude.com/docs/en/sandboxing#custom-proxy-configuration)
* **Type** : nombre, un port TCP local
* **Défaut** : non défini ; Claude Code exécute son propre proxy

```json settings.json
{
  "sandbox": {
    "network": {
      "socksProxyPort": 8081
    }
  }
}
```

Voir [Custom proxy configuration](https://code.claude.com/docs/en/sandboxing#custom-proxy-configuration).

### `sandbox.network.tlsTerminate`

Fait terminer le TLS par le proxy du bac à sable afin qu'il puisse lire le contenu des requêtes HTTPS. C'est **expérimental**, et la [substitution d'identifiants](https://code.claude.com/docs/en/sandboxing#mask-credentials) `mask` l'exige. Mettez `{}` pour générer une autorité de certification éphémère pour la session, ou définissez `caCertPath` et `caKeyPath` pour utiliser la vôtre.

* **Portée** : `User or managed`. Un dépôt ne peut ni l'activer ni fournir une autorité de certification.
* **Type** : objet avec les chaînes facultatives `caCertPath` et `caKeyPath`, chacune un chemin de fichier
* **Défaut** : non défini ; le proxy ne termine ni n'inspecte le TLS

```json settings.json
{
  "sandbox": {
    "network": {
      "tlsTerminate": {}
    }
  }
}
```

Quand plusieurs sources honorées la définissent, Claude Code utilise la valeur de la source de plus haute priorité : réglages gérés, puis flag `--settings`, puis réglages utilisateur.

## Mémoire et contexte

*(Ancre historique : `context-and-memory`.)*

Contrôlez ce que Claude Code charge dans le contexte, comment il compacte, et où il conserve la mémoire et les plans. Voir [Manage context](https://code.claude.com/docs/en/context-window) et [Memory](https://code.claude.com/docs/en/memory).

### `autoCompactEnabled`

Fait [compacter automatiquement la conversation](https://code.claude.com/docs/en/context-window#when-your-context-fills-up) par Claude Code lorsque le contexte approche de la limite. Apparaît dans `/config` sous le nom **Auto-compact**, et la basculer à cet endroit écrit cette clé dans vos réglages utilisateur.

* **Portée** : `Any file`
* **Type** : booléen
  * `true` : Claude Code compacte automatiquement la conversation lorsque le contexte approche de la limite
  * `false` : Claude Code ne compacte pas automatiquement
* **Défaut** : `true`
* **Surcharges par session** : [`DISABLE_AUTO_COMPACT`](https://code.claude.com/docs/en/env-vars) désactive l'auto-compactage pour une session ; quel que soit celui des deux qui le désactive, l'autre ne peut pas le réactiver

```json settings.json
{
  "autoCompactEnabled": false
}
```

La commande manuelle `/compact` continue de fonctionner quand l'auto-compactage est désactivé. Si vous désactivez l'auto-compactage, le [compactage en période d'inactivité](https://code.claude.com/docs/en/settings-reference#idlecompaction) est désactivé aussi.

### `autoCompactWindow`

Définit jusqu'à quel remplissage de la fenêtre de contexte Claude Code attend avant de [compacter automatiquement](https://code.claude.com/docs/en/context-window#when-your-context-fills-up).

* **Portée** : `Any file`
* **Type** : nombre de tokens, de `100000` à `1000000`. Claude Code plafonne la valeur à la fenêtre de contexte de votre modèle ; la [vue d'ensemble des modèles](https://platform.claude.com/docs/en/about-claude/models/overview) indique la fenêtre de chaque modèle
* **Défaut** : non défini ; Claude Code choisit une fenêtre adaptée à votre modèle
* **Surcharges par session** : [`--autocompact`](https://code.claude.com/docs/en/cli-reference#cli-flags) l'emporte sur cette clé pour une session, et [`CLAUDE_CODE_AUTO_COMPACT_WINDOW`](https://code.claude.com/docs/en/env-vars) l'emporte sur les deux

```json settings.json
{
  "autoCompactWindow": 500000
}
```

La commande [`/autocompact`](https://code.claude.com/docs/en/commands#all-commands) enregistre une fenêtre pour le modèle courant sous [`modelSettings`](https://code.claude.com/docs/en/settings-reference#modelsettings), qui l'emporte sur cette clé dans le même fichier pour ce modèle. [Set the auto-compact window](https://code.claude.com/docs/en/model-config#set-the-auto-compact-window) explique comment la commande, le flag, la variable et le réglage interagissent.

### `autoMemoryDirectory`

Stocke la [mémoire automatique](https://code.claude.com/docs/en/memory#storage-location) dans un répertoire de votre choix au lieu de l'emplacement par défaut propre à chaque projet.

* **Portée** : `Any file`
* **Type** : chaîne, un chemin de répertoire absolu ou préfixé par `~/`
* **Défaut** : non défini ; Claude Code utilise `~/.claude/projects/<project>/memory/`

```json settings.json
{
  "autoMemoryDirectory": "~/my-memory-dir"
}
```

Depuis les réglages de projet ou locaux, Claude Code honore cette clé selon la même [règle de confiance de l'espace de travail que les hooks](https://code.claude.com/docs/en/permissions#what-runs-before-you-trust-a-folder), puisqu'un dépôt cloné peut fournir ces fichiers.

### `autoMemoryEnabled`

Active ou désactive la [mémoire automatique](https://code.claude.com/docs/en/memory#enable-or-disable-auto-memory). Avec `false`, Claude ne lit ni n'écrit dans le répertoire de mémoire automatique. Vous pouvez aussi la basculer avec `/memory` pendant une session, ce qui écrit cette clé dans vos réglages utilisateur.

* **Portée** : `Any file`
* **Type** : booléen
  * `true` : identique à non défini ; la mémoire automatique reste active, sauf si un élément qui prime sur cette clé la désactive pour la session, comme `--bare`, le mode sans échec (safe mode) ou `CLAUDE_CODE_DISABLE_AUTO_MEMORY`
  * `false` : Claude ne lit ni n'écrit dans le répertoire de mémoire automatique
* **Défaut** : `true`
* **Surcharges par session** : [`CLAUDE_CODE_DISABLE_AUTO_MEMORY`](https://code.claude.com/docs/en/env-vars) l'emporte sur cette clé pour une session, dans un sens comme dans l'autre

```json settings.json
{
  "autoMemoryEnabled": false
}
```

### `bashOutputMaxChars`

Définit combien de caractères de la sortie d'une commande Bash ou PowerShell réussie [Claude reçoit en ligne](https://code.claude.com/docs/en/tools-reference#output-limits). Quand la sortie dépasse la limite, Claude Code l'enregistre dans un fichier et Claude reçoit un court aperçu plus le chemin du fichier. Augmentez la limite quand la sortie des commandes — un build verbeux ou le journal complet d'une suite de tests — dépasse régulièrement la valeur par défaut et que vous voulez que Claude la lise sans ouvrir le fichier. (v2.1.261+)

* **Portée** : `Any file`
* **Type** : nombre de caractères, entier positif. Claude Code ramène la valeur dans l'intervalle `4000` à `128000`
* **Défaut** : non défini ; Claude reçoit jusqu'à 30 000 caractères en ligne

```json settings.json
{
  "bashOutputMaxChars": 100000
}
```

Quand vous définissez cette clé, Claude Code ignore la variable d'environnement [`BASH_MAX_OUTPUT_LENGTH`](https://code.claude.com/docs/en/env-vars).

### `claudeMd`

Injecte des instructions de type CLAUDE.md comme mémoire gérée par l'organisation, sans déployer de fichier séparé. Claude Code charge le texte comme une entrée de mémoire gérée, avant les fichiers CLAUDE.md utilisateur et de projet.

* **Portée** : `Managed`
* **Type** : chaîne, le texte d'un fichier CLAUDE.md ; écrivez-le comme le fichier, Markdown compris, avec les sauts de ligne sous forme `\n`
* **Défaut** : non défini

Cet exemple déploie deux règles sous forme d'une courte liste Markdown :

```json managed-settings.json
{
  "claudeMd": "# Engineering rules\n\n- Always run make lint before committing.\n- Never push directly to main."
}
```

Voir [Deploy organization-wide CLAUDE.md](https://code.claude.com/docs/en/memory#deploy-organization-wide-claude-md).

### `claudeMdExcludes`

Ignore certains fichiers `CLAUDE.md` lorsque Claude Code charge la [mémoire](https://code.claude.com/docs/en/memory#exclude-specific-claude-md-files). Dans un grand monorepo, utilisez-la pour ignorer les CLAUDE.md d'autres équipes qui ne concernent pas votre travail ; [Exclude irrelevant CLAUDE.md files](https://code.claude.com/docs/en/large-codebases#exclude-irrelevant-claude-md-files), dans le guide des grandes bases de code, détaille ce cas. Les motifs sont comparés aux chemins absolus des fichiers.

* **Portée** : `Any file`
* **Type** : tableau de chaînes, chacune un motif glob ou un chemin absolu
* **Défaut** : non défini ; Claude Code charge tous les CLAUDE.md qu'il trouve

```json settings.json
{
  "claudeMdExcludes": ["**/vendor/**/CLAUDE.md"]
}
```

Les exclusions ne s'appliquent qu'aux fichiers de mémoire utilisateur, de projet et locaux ; les fichiers CLAUDE.md de politique gérée ne peuvent pas être exclus.

### `env`

*(Ancre historique : `environment-variables`.)*

Définit des variables d'environnement pour chaque session et pour les sous-processus que Claude Code y lance. La plupart des variables de la [référence des variables d'environnement](https://code.claude.com/docs/en/env-vars) peuvent être placées ici : c'est ainsi que vous en appliquez une à toutes les sessions ou que vous la déployez auprès de votre équipe. Les réglages de projet et locaux ne peuvent pas définir [certaines d'entre elles](#variables-que-claude-code-ignore-dans-env).

* **Portée** : `Any file`
* **Type** : objet associant des noms de variables à des valeurs chaînes
* **Défaut** : non défini

Cet exemple désactive le compactage automatique et fait passer les requêtes d'API par un proxy :

```json settings.json
{
  "env": {
    "DISABLE_AUTO_COMPACT": "1",
    "ANTHROPIC_BASE_URL": "https://proxy.example.com"
  }
}
```

#### Interaction des valeurs `env` avec votre shell

* Une valeur définie ici écrase la même variable exportée dans votre shell, et quand plusieurs fichiers de réglages définissent une variable, celui de [plus haute priorité](https://code.claude.com/docs/en/settings#settings-precedence) s'applique. [Variables que Claude Code ignore dans `env`](#variables-que-claude-code-ignore-dans-env) liste les exceptions pour les réglages de projet et locaux.
* Quand l'application Claude Desktop ou un runner d'[environnement auto-hébergé](https://code.claude.com/docs/en/self-hosted-environments) démarre la session, l'environnement de lancement qu'il construit prend le pas : Claude Code ignore une valeur `env` de n'importe quel fichier de réglages pour une variable que l'environnement de lancement définit déjà. Le [journal de débogage](https://code.claude.com/docs/en/debug-your-config) nomme chaque variable ignorée.
* Pour annuler un export du shell, mettez la variable à `""`. Claude Code traite une valeur vide comme non définie pour la sélection du fournisseur, et les sous-processus héritent de la valeur vide.
* `NO_COLOR` et `FORCE_COLOR` définies ici n'atteignent que les sous-processus. Pour changer les couleurs de l'interface de Claude Code lui-même, définissez-les dans votre shell avant de lancer `claude`.
* Les valeurs définies ici sont en clair dans le fichier de réglages et atteignent tous les sous-processus que lance Claude Code. Pour un jeton bearer OTLP qui tourne, utilisez [`otelHeadersHelper`](https://code.claude.com/docs/en/settings-reference#otelheadershelper) ; pour les identifiants d'API, utilisez [`apiKeyHelper`](https://code.claude.com/docs/en/settings-reference#apikeyhelper).

#### Quand Claude Code applique les valeurs `env`

* Depuis les réglages utilisateur, `--settings` et les réglages gérés : au démarrage, puis de nouveau dans la session en cours lorsqu'une modification enregistrée change l'`env` fusionné.
* Depuis les réglages de projet et locaux : après que vous avez accordé votre confiance à l'espace de travail, ou au démarrage en mode `-p`, qui n'affiche jamais la boîte de dialogue de confiance, puis de nouveau lorsqu'une modification enregistrée change l'`env` fusionné.
* Les variables que Claude Code classe comme sûres, comme la sélection du modèle, les délais et limites, et les bascules de fonctionnalités : au démarrage depuis tous les fichiers de réglages, à l'exception des [variables que les réglages de projet et locaux ne peuvent pas définir](#variables-que-claude-code-ignore-dans-env).
* Après avoir [déplacé la session avec `/cd`](https://code.claude.com/docs/en/permissions#move-the-session-to-another-directory) (v2.1.246+) : les valeurs `env` de projet et locales du nouveau répertoire, par-dessus celles du répertoire précédent.

#### Variables que Claude Code ignore dans `env`

* Les réglages de projet et locaux ne peuvent pas définir des variables qu'un dépôt récupéré ne devrait pas contrôler ; définissez-les plutôt dans votre shell, vos réglages utilisateur ou les réglages gérés. Claude Code abandonne chacune d'elles, à l'exception de quelques valeurs qui désactivent la télémétrie, et journalise un avertissement visible avec `claude --debug`. Il s'agit notamment de :

  * Les variables qui choisissent où Claude Code stocke ou écrit ses propres fichiers : `CLAUDE_CONFIG_DIR`, `CLAUDE_CODE_TMPDIR`, et les variables de répertoires du système d'exploitation comme `HOME`, `TMPDIR`, `TMP`, `TEMP` et la famille `XDG_*`.
  * Les variables Windows qui choisissent les programmes et la configuration globale de la machine pour les processus que lance Claude Code, comme `SystemRoot`, `ComSpec`, `ProgramData`, `LOCALAPPDATA`, `PATHEXT`, `PSModulePath` et la famille `ProgramFiles`.
  * Les variables qui exportent le contenu de la session : [`OTEL_LOG_RAW_API_BODIES`](https://code.claude.com/docs/en/env-vars#variables) et la paire de traçage bêta détaillé `ENABLE_BETA_TRACING_DETAILED` et `BETA_TRACING_ENDPOINT`.
  * Les variables de l'[exportateur OpenTelemetry](https://code.claude.com/docs/en/monitoring-usage) qui activent la télémétrie, choisissent sa destination ou choisissent le contenu qu'elle capture :

    * `CLAUDE_CODE_ENABLE_TELEMETRY`, plus la paire de télémétrie améliorée bêta `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA` et `ENABLE_ENHANCED_TELEMETRY_BETA`
    * Les sélecteurs d'exportateur `OTEL_LOGS_EXPORTER`, `OTEL_METRICS_EXPORTER` et `OTEL_TRACES_EXPORTER`
    * Les variables de contenu `OTEL_LOG_USER_PROMPTS`, `OTEL_LOG_ASSISTANT_RESPONSES`, `OTEL_LOG_TOOL_CONTENT` et `OTEL_LOG_TOOL_DETAILS`
    * Les variables `OTEL_EXPORTER_OTLP_*` dont le nom se termine par `_ENDPOINT`, `_HEADERS`, `_PROTOCOL`, `_CERTIFICATE`, `_CLIENT_KEY` ou `_INSECURE`, sous leurs formes génériques et par signal, comme `OTEL_EXPORTER_OTLP_ENDPOINT` et `OTEL_EXPORTER_OTLP_METRICS_HEADERS`
    * `OTEL_EXPORTER_PROMETHEUS_HOST` et `OTEL_EXPORTER_PROMETHEUS_PORT`

    Seules ces valeurs s'appliquent encore depuis les réglages de projet et locaux, parce qu'elles désactivent quelque chose : `none` pour les trois sélecteurs d'exportateur, et une valeur de désactivation comme `0` pour `OTEL_LOG_USER_PROMPTS`, `OTEL_LOG_TOOL_CONTENT` et `OTEL_LOG_TOOL_DETAILS`. Une telle valeur l'emporte sur la même variable dans vos réglages utilisateur, mais pas sur une variable définie par l'environnement depuis lequel vous lancez Claude Code, par un fichier `--settings` ou par les réglages gérés.

    Quand un fichier de réglages de projet ou local définit une variable de ce groupe, une session interactive locale affiche un avis au démarrage. Lancez `/status` ou `claude doctor` pour voir lesquelles Claude Code a ignorées et lesquelles ont désactivé la télémétrie ; les deux listent les noms, jamais les valeurs. Une exécution non interactive avec `-p` ou une session Agent SDK n'affiche aucun avis : vérifiez donc que votre collecteur reçoit toujours des données après une mise à jour. Si ce n'est pas le cas, définissez les variables dans vos réglages utilisateur, les réglages gérés, l'environnement du job ou un fichier passé avec `--settings`.

    Ignorer ce groupe dans les réglages de projet et locaux nécessite la v2.1.282+.
  * Les variables qui changent la façon dont Claude Code démarre ou se synchronise, comme `CLAUDE_CODE_PROCESS_WRAPPER`, `CLAUDE_CODE_SYNC_SKILLS`, `CLAUDE_CODE_SYNC_PLUGINS`, `CLAUDE_CODE_PLUGIN_CACHE_DIR` et `CLAUDE_CODE_PLUGIN_SEED_DIR`.
  * Les variables qui règlent les minuteries d'une boîte de dialogue restée sans réponse : [`CLAUDE_CODE_USER_DIALOG_TIMEOUT_MS`, `CLAUDE_AFK_TIMEOUT_MS` et `CLAUDE_AFK_COUNTDOWN_MS`](https://code.claude.com/docs/en/env-vars#variables).
  * [`CLAUDE_CODE_DISABLE_ATTACHMENTS`](https://code.claude.com/docs/en/env-vars#variables), qui désactive le traitement des pièces jointes.
  * [`CLAUDE_CODE_DISABLE_FILE_MENTIONS`](https://code.claude.com/docs/en/env-vars#variables), qui empêche Claude Code de développer les mentions `@` de fichiers et de répertoires.

  Avant la v2.1.251, les réglages de projet et locaux pouvaient aussi définir les variables de cette liste qui choisissent où Claude Code écrit ses fichiers ou qui exportent le contenu de la session, sauf `HOME` et `XDG_CONFIG_HOME`. Avant la v2.1.290, ils pouvaient aussi définir les variables de minuterie de dialogue et `CLAUDE_CODE_DISABLE_ATTACHMENTS`.
* Les variables d'identité que possèdent les environnements d'hébergement de Claude Code, comme `CLAUDE_CODE_REMOTE` et `CLAUDE_CODE_ACCOUNT_UUID`, sont ignorées depuis tous les fichiers.
* [`CLAUDE_CODE_MESSAGING_SOCKET` et `CLAUDE_CODE_MESSAGING_TOKEN`](https://code.claude.com/docs/en/env-vars#variables), que Claude Code exporte lui-même, sont ignorées depuis tous les fichiers. Ignorer la variable de socket nécessite la v2.1.224+, et ignorer le jeton la v2.1.228+.
* [`CLAUDE_CODE_PROJECT_DIR_NAME`](https://code.claude.com/docs/en/sessions#name-the-project-directory-yourself), que Claude Code ne lit que depuis l'environnement de lancement, est ignorée depuis tous les fichiers (v2.1.234+).
* [`CLAUDE_CODE_RESTRICTED`](https://code.claude.com/docs/en/env-vars#variables), que Claude Code ne lit que depuis l'environnement de lancement, est ignorée depuis tous les fichiers.
* [`CLAUDE_CODE_DISABLE_POWERSHELL_CMD_RM_DENY`](https://code.claude.com/docs/en/env-vars#variables), que Claude Code ne lit que depuis l'environnement de lancement, est ignorée depuis tous les fichiers. La variable nécessite la v2.1.283+.
* [`CLAUDE_CODE_DISABLE_DANGEROUS_RM_TIMEOUT`, `CLAUDE_CODE_DISABLE_SUBSTITUTION_RM_PROMPT` et `CLAUDE_CODE_DISABLE_INLINE_SHELL_RM_PROMPT`](https://code.claude.com/docs/en/env-vars#variables), que Claude Code ne lit que depuis l'environnement de lancement, sont ignorées depuis tous les fichiers.

### `fileCheckpointingEnabled`

Fait prendre à Claude Code un instantané des fichiers avant chaque modification, pour que [`/rewind`](https://code.claude.com/docs/en/checkpointing) puisse les restaurer. Apparaît dans `/config` sous le nom **Rewind code (checkpoints)**, et la basculer à cet endroit écrit cette clé dans vos réglages utilisateur.

* **Portée** : `Any file`
* **Type** : booléen
  * `true` : Claude Code prend un instantané des fichiers avant chaque modification, pour que `/rewind` puisse les restaurer
  * `false` : Claude Code ne prend pas d'instantané, donc `/rewind` ne peut pas restaurer les fichiers
* **Défaut** : `true`
* **Surcharges par session** : [`CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING`](https://code.claude.com/docs/en/env-vars) désactive les points de contrôle pour une session ; quel que soit celui des deux qui les désactive, l'autre ne peut pas les réactiver

```json settings.json
{
  "fileCheckpointingEnabled": false
}
```

Dans une exécution `-p` ou une session Agent SDK, Claude Code ignore cette clé. Le SDK active les points de contrôle avec son option `enableFileCheckpointing`, et une exécution `-p` nue nécessite `CLAUDE_CODE_ENABLE_SDK_FILE_CHECKPOINTING=true`. Voir [File checkpointing in the Agent SDK](https://code.claude.com/docs/en/agent-sdk/file-checkpointing).

### `idleCompaction`

Le compactage en période d'inactivité [compacte](https://code.claude.com/docs/en/context-window#what-survives-compaction) une longue conversation que vous avez laissée de côté, peu avant l'expiration de son cache de prompt. Votre message suivant envoie alors un résumé au lieu de toute la conversation. À votre retour, la conversation affiche `Compacted while idle, before the prompt cache expired`.

Le compactage en période d'inactivité est actif par défaut pour les conversations qui remplissent **toutes** ces conditions :

* **Session** : vous êtes dans une session de terminal interactive
* **Cache de prompt** : la conversation utilise le [TTL de cache de prompt d'une heure](https://code.claude.com/docs/en/prompt-caching#which-ttl-each-request-gets)
* **Feature flags** : Claude Code peut [récupérer les feature flags auprès d'Anthropic](https://code.claude.com/docs/en/env-vars#features-that-need-feature-flag-fetching)
* **Taille** : le contexte de la conversation a atteint le seuil de tokens, que vous pouvez [modifier](#modifier-le-seuil-du-compactage-en-période-dinactivité) et qui vaut 200 000 par défaut
* **Temps d'inactivité** : environ 54 minutes se sont écoulées depuis la dernière requête, et votre ordinateur est éveillé à ce moment-là

Pour désactiver le compactage en période d'inactivité, mettez cette clé à `false`.

* **Portée** : `Any file`
* **Type** : booléen
  * `true` : Claude Code compacte les conversations inactives qui remplissent les conditions
  * `false` : Claude Code ne compacte pas pendant que votre session est inactive
* **Défaut** : `true`
* **Surcharges par session** : [`DISABLE_AUTO_COMPACT`](https://code.claude.com/docs/en/env-vars) désactive le compactage en période d'inactivité en même temps que l'auto-compactage

```json settings.json
{
  "idleCompaction": false
}
```

La clé nécessite la v2.1.290+. Sur une version antérieure, qui ignore cette clé, désactivez [`autoCompactEnabled`](https://code.claude.com/docs/en/settings-reference#autocompactenabled) pour arrêter le compactage en période d'inactivité.

#### Modifier le seuil du compactage en période d'inactivité

Définissez [`CLAUDE_CODE_IDLE_COMPACT_MIN_TOKENS`](https://code.claude.com/docs/en/env-vars) au nombre de tokens que le contexte d'une conversation doit atteindre avant que le compactage en période d'inactivité ne s'y applique. Utilisez un nombre entier positif. Les valeurs inférieures à `100000` sont relevées à `100000`, le seuil le plus bas.

Avec la valeur par défaut, une conversation sur une fenêtre de contexte de 200K s'auto-compacte avant d'atteindre 200 000 tokens. Cet exemple abaisse le seuil à 150 000 pour que ces conversations soient éligibles :

```json settings.json
{
  "env": {
    "CLAUDE_CODE_IDLE_COMPACT_MIN_TOKENS": "150000"
  }
}
```

### `plansDirectory`

Choisit où Claude Code stocke les fichiers de plan qu'il écrit en [mode plan](https://code.claude.com/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode). Claude Code résout le chemin relativement à la racine du projet.

* **Portée** : `Any file`
* **Type** : chaîne, un chemin relatif à la racine du projet
* **Défaut** : non défini ; Claude Code utilise `~/.claude/plans`

```json settings.json
{
  "plansDirectory": "./plans"
}
```

Claude Code stocke les plans dans `~/.claude/plans` au lieu du répertoire que vous avez défini dans des cas comme ceux-ci :

* **Hors de la racine du projet** : le chemin se résout hors de la racine du projet, comme `"../plans"`.
* **Barre oblique inverse sous macOS, Linux et WSL** : le chemin résolu contient une barre oblique inverse, comme le `"docs\\plans"` de style Windows. Écrivez `"docs/plans"`, qui fonctionne aussi sous Windows.

### `skillListingBudgetFraction`

À chaque tour, Claude voit une [liste de vos skills](https://code.claude.com/docs/en/skills#skill-descriptions-are-cut-short) avec leurs descriptions, et Claude Code plafonne cette liste à une part de la fenêtre de contexte. Quand la liste dépasse le plafond, Claude Code conserve le nom de chaque skill mais retire les descriptions des skills les moins utilisés : Claude peut toujours invoquer ces skills, mais il est moins susceptible d'en choisir un de lui-même. Augmentez cette clé pour garder plus de descriptions visibles, au prix de plus de contexte par tour.

* **Portée** : `Any file`
* **Type** : nombre, une fraction strictement supérieure à `0` et au plus égale à `1`
* **Défaut** : `0.01`, soit 1 % de la fenêtre de contexte réservé

```json settings.json
{
  "skillListingBudgetFraction": 0.02
}
```

Pour voir combien de contexte la liste utilise et quels skills y contribuent le plus, lancez `/doctor`.

### `skillListingMaxDescChars`

À chaque tour, Claude voit une [liste de vos skills](https://code.claude.com/docs/en/skills#skill-descriptions-are-cut-short) qui affiche le texte `description` et `when_to_use` de chaque skill. Cette clé plafonne le nombre de caractères de ce texte que Claude Code affiche par skill ; un texte plus long est coupé au plafond.

* **Portée** : `Any file`
* **Type** : nombre de caractères, entier positif
* **Défaut** : `1536`

```json settings.json
{
  "skillListingMaxDescChars": 2048
}
```

Augmentez-la pour garder intactes les longues descriptions, au prix de plus de contexte par tour ; diminuez-la pour faire tenir plus de skills sous [`skillListingBudgetFraction`](https://code.claude.com/docs/en/settings-reference#skilllistingbudgetfraction).

### `taskOutputMaxChars`

> **Avertissement** : supprimée en v2.1.277, en même temps que l'outil `TaskOutput` qu'elle dimensionnait. La définir n'a aucun effet sur les versions actuelles. Claude lit désormais le [fichier de sortie](https://code.claude.com/docs/en/tools-reference#background-commands) d'une tâche en arrière-plan avec `Read`.

Jusqu'à la v2.1.276, cette clé définissait le nombre de caractères de la sortie d'une [tâche en arrière-plan](https://code.claude.com/docs/en/tools-reference#background-commands) que Claude recevait en ligne lorsqu'il lisait la tâche avec l'outil `TaskOutput`.
