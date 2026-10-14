# Tous les réglages — partie 4/4

> Source : https://code.claude.com/docs/en/settings-reference · [← Index](../31-index-documentation-officielle.md)

> Cette partie couvre les catégories MCP ; agents, sessions et worktrees ; Remote Control, desktop et notifications ; authentification et fournisseurs ; mises à jour et versions ; outils ; confidentialité et télémétrie ; réglages entreprise et gérés ; réglages de configuration globale (`~/.claude.json`) ; et « Voir aussi ». Les liens vers `#ancre` renvoient aux sections de la même page officielle (parties 1 à 4 comprises), en particulier [Portées (scopes)](https://code.claude.com/docs/en/settings-reference#scopes).

## MCP

Contrôlent à quels serveurs MCP Claude Code se connecte et lesquels une organisation autorise. Voir [Connect to external tools with MCP](https://code.claude.com/docs/en/mcp) et [Managed MCP configuration](https://code.claude.com/docs/en/managed-mcp).

### `allowAllClaudeAiMcps`

Charge les [connecteurs claude.ai](https://code.claude.com/docs/en/mcp#use-mcp-servers-from-claude-ai) que Claude Code récupère lui-même en plus d'un `managed-mcp.json` déployé. Sans cette clé, `managed-mcp.json` prend le contrôle exclusif des serveurs MCP et supprime ces connecteurs.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes). Les utilisateurs ne peuvent pas réactiver les connecteurs supprimés par le contrôle exclusif.
* **Type** : booléen
  * `true` : Claude Code charge les connecteurs claude.ai en plus d'un `managed-mcp.json` déployé
  * `false` : un `managed-mcp.json` déployé prend le contrôle exclusif des serveurs MCP et supprime les connecteurs claude.ai que [Claude Code récupère lui-même](https://code.claude.com/docs/en/mcp#how-connectors-reach-claude-code)
* **Défaut** : `false`, donc un `managed-mcp.json` déployé supprime les connecteurs claude.ai que Claude Code récupère lui-même

```json managed-settings.json theme={null}
{
  "allowAllClaudeAiMcps": true
}
```

[`allowedMcpServers`](#allowedmcpservers) et [`deniedMcpServers`](#deniedmcpservers) s'appliquent toujours aux connecteurs chargés par cette clé. Les connecteurs fournis à une [session cloud](https://code.claude.com/docs/en/claude-code-on-the-web) dont l'hôte porte un `managed-mcp.json`, comme un runner auto-hébergé (self-hosted), restent supprimés. Voir [Allow claude.ai connectors alongside the managed set](https://code.claude.com/docs/en/managed-mcp#allow-claude-ai-connectors-alongside-the-managed-set).

### `allowClaudeInChromeWithManagedMcp`

Permet au serveur intégré [Claude in Chrome](https://code.claude.com/docs/en/chrome) de fonctionner à côté d'un `managed-mcp.json` déployé. Sans cette clé, un `managed-mcp.json` déployé bloque Claude in Chrome dans les sessions terminal. (v2.1.282+)

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes), uniquement depuis les réglages gérés propres à l'appareil : un plist déployé par MDM ou une clé de registre HKLM, ou un fichier système `managed-settings.json`. Claude Code l'ignore dans les réglages gérés par serveur (server-managed), dans le registre HKCU modifiable par l'utilisateur, et dans les réglages utilisateur ou projet.
* **Type** : booléen
  * `true` : le serveur intégré Claude in Chrome peut fonctionner à côté d'un `managed-mcp.json` déployé
  * `false` : un `managed-mcp.json` déployé bloque Claude in Chrome dans les sessions terminal
* **Défaut** : `false`, donc un `managed-mcp.json` déployé bloque Claude in Chrome dans les sessions terminal

```json managed-settings.json theme={null}
{
  "allowClaudeInChromeWithManagedMcp": true
}
```

Une entrée [`deniedMcpServers`](#deniedmcpservers) pour `claude-in-chrome` bloque toujours le serveur même avec cette clé activée. Voir [Allow Claude in Chrome alongside the managed set](https://code.claude.com/docs/en/managed-mcp#allow-claude-in-chrome-alongside-the-managed-set).

### `allowedMcpServers`

Liste d'autorisation (allowlist) des serveurs MCP que les personnes peuvent ajouter. Claude Code bloque tout serveur qui ne correspond à aucune entrée, où qu'il soit défini, y compris les serveurs de plugins, les serveurs passés par l'utilisateur avec `--mcp-config` et les serveurs venant de claude.ai.

Les serveurs intégrés comme Claude in Chrome, le serveur `ide` auquel Claude Code se connecte dans un IDE [VS Code](https://code.claude.com/docs/en/vs-code#the-built-in-ide-mcp-server) ou [JetBrains](https://code.claude.com/docs/en/jetbrains#the-built-in-ide-mcp-server) en cours d'exécution, et les serveurs que la CLI configure elle-même sont exemptés de l'allowlist, mais la denylist s'applique toujours à eux. Depuis la v2.1.268, les outils Slack d'une session [Claude Tag](https://code.claude.com/docs/en/claude-tag) sont aussi exemptés de l'allowlist, la denylist s'appliquant toujours à eux. Les serveurs in-process `type: "sdk"` sont exemptés des deux listes ; c'est l'[application qui a démarré la session](https://code.claude.com/docs/en/mcp#how-connectors-reach-claude-code) qui les enregistre.

Les serveurs fournis par votre organisation sont aussi exemptés de l'allowlist, la denylist s'appliquant toujours à eux. L'exemption couvre chaque entrée [`managedMcpServers`](#managedmcpservers), et toute entrée de [`managed-mcp.json`](https://code.claude.com/docs/en/managed-mcp#exclusive-control-with-managed-mcp-json) dont les valeurs n'utilisent aucune expansion `${VAR}`. Voir [How a server is evaluated](https://code.claude.com/docs/en/managed-mcp#how-a-server-is-evaluated) pour l'ordre complet des vérifications. Avant la v2.1.259, les serveurs de `managed-mcp.json` devaient aussi correspondre.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes) (n'importe quel fichier). Les entrées de tous les fichiers fusionnent en une seule allowlist, sauf si [`allowManagedMcpServersOnly`](#allowmanagedmcpserversonly) est défini. Déployez-la dans les réglages gérés pour l'imposer.
* **Type** : tableau d'objets, chacun avec exactement une clé : `serverName`, une chaîne limitée aux lettres, chiffres, tirets et underscores ; `serverCommand`, un tableau de la commande et de ses arguments, comparé exactement ; ou `serverUrl`, un motif d'URL avec jokers `*`
* **Défaut** : non défini, donc tout serveur est autorisé ; un tableau vide bloque tous les serveurs ajoutés par les utilisateurs

Cet exemple n'autorise que le serveur stdio démarré par la commande `npx` indiquée :

```json settings.json theme={null}
{
  "allowedMcpServers": [
    { "serverCommand": ["npx", "-y", "@modelcontextprotocol/server-filesystem"] }
  ]
}
```

Une entrée [`deniedMcpServers`](#deniedmcpservers) l'emporte : un serveur présent sur les deux listes est bloqué. Dès que la liste contient une entrée `serverCommand`, un serveur stdio doit correspondre à une entrée `serverCommand`, et dès qu'elle contient une entrée `serverUrl`, un serveur distant doit correspondre à une entrée `serverUrl` : une correspondance `serverName` n'admet plus ce type de serveur. Voir [Policy-based control with allowlists and denylists](https://code.claude.com/docs/en/managed-mcp#policy-based-control-with-allowlists-and-denylists).

### `allowManagedMcpServersOnly`

Fait de l'allowlist gérée la seule qui s'applique. Claude Code lit alors [`allowedMcpServers`](#allowedmcpservers) uniquement depuis les réglages gérés et ignore les allowlists des réglages utilisateur, projet et local ; [`deniedMcpServers`](#deniedmcpservers) fusionne toujours depuis toutes les portées, si bien que les utilisateurs peuvent encore bloquer des serveurs pour eux-mêmes. Les administrateurs la définissent pour que les réglages propres d'un utilisateur ne puissent pas élargir ce que l'allowlist gérée permet.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code lit `allowedMcpServers` uniquement depuis les réglages gérés et ignore les allowlists des réglages utilisateur, projet et local
  * `false` : les allowlists de toutes les portées fusionnent
* **Défaut** : `false`, donc les allowlists de toutes les portées fusionnent

Cet exemple verrouille l'allowlist sur les réglages gérés et n'autorise que le serveur nommé `github` :

```json managed-settings.json theme={null}
{
  "allowManagedMcpServersOnly": true,
  "allowedMcpServers": [
    { "serverName": "github" }
  ]
}
```

Les utilisateurs peuvent toujours ajouter leurs propres serveurs MCP ; seuls ceux qui correspondent à l'allowlist gérée se chargent. Voir [Restrict the allowlist to managed settings only](https://code.claude.com/docs/en/managed-mcp#restrict-the-allowlist-to-managed-settings-only).

### `deniedMcpServers`

Bloque des serveurs MCP précis. Claude Code refuse de charger un serveur correspondant où qu'il soit défini, y compris les serveurs de plugins, les serveurs passés avec `--mcp-config`, les serveurs de `managed-mcp.json`, les serveurs de [`managedMcpServers`](#managedmcpservers), et les connecteurs claude.ai [qu'il récupère lui-même](https://code.claude.com/docs/en/mcp#how-connectors-reach-claude-code). Les serveurs in-process `type: "sdk"` sont exemptés ; c'est l'application qui a démarré la session qui les enregistre.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Les entrées de tous les fichiers fusionnent en une seule denylist, et [`allowManagedMcpServersOnly`](#allowmanagedmcpserversonly) n'y change rien. Déployez-la dans les réglages gérés pour l'imposer.
* **Type** : tableau d'objets, chacun avec exactement une clé : `serverName`, une chaîne (le nom affiché d'un connecteur claude.ai comme `"claude.ai Slack"` fonctionne donc) ; `serverCommand`, un tableau de la commande et de ses arguments, comparé exactement ; ou `serverUrl`, un motif d'URL avec jokers `*`
* **Défaut** : non défini, donc aucun serveur n'est bloqué ; un tableau vide ne bloque rien non plus

```json settings.json theme={null}
{
  "deniedMcpServers": [
    { "serverName": "filesystem" }
  ]
}
```

La denylist l'emporte sur [`allowedMcpServers`](#allowedmcpservers) : un serveur présent sur les deux listes est bloqué. Voir [Policy-based control with allowlists and denylists](https://code.claude.com/docs/en/managed-mcp#policy-based-control-with-allowlists-and-denylists).

### `disableClaudeAiConnectors`

Désactive les [connecteurs MCP claude.ai](https://code.claude.com/docs/en/mcp#use-mcp-servers-from-claude-ai) que [Claude Code récupère lui-même](https://code.claude.com/docs/en/mcp#how-connectors-reach-claude-code) : il ne les récupère ni ne les connecte. Un `true` dans n'importe quel fichier de réglages s'applique : un `.claude/settings.json` de projet versionné peut exclure un dépôt de ces connecteurs, mais un `false` au niveau projet ne peut pas surcharger un `true` au niveau utilisateur ou géré.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code ne récupère ni ne connecte ces connecteurs
  * `false` : identique à non défini ; Claude Code récupère vos connecteurs sauf si un autre fichier de réglages ou `ENABLE_CLAUDEAI_MCP_SERVERS` les désactive
* **Défaut** : `false`, donc Claude Code récupère vos connecteurs
* **Surcharges par session** : [`ENABLE_CLAUDEAI_MCP_SERVERS`](https://code.claude.com/docs/en/env-vars) défini à `false` désactive les connecteurs pour une session ; quel que soit celui des deux qui les désactive, l'autre ne peut pas les réactiver

```json settings.json theme={null}
{
  "disableClaudeAiConnectors": true
}
```

Les serveurs passés explicitement avec `--mcp-config` ne sont pas concernés. Pour bloquer des connecteurs individuels plutôt que tous, utilisez [`deniedMcpServers`](#deniedmcpservers). Voir [Disable claude.ai connectors](https://code.claude.com/docs/en/mcp#disable-claude-ai-connectors).

### `disabledMcpjsonServers`

Rejette des serveurs précis définis dans le fichier `.mcp.json` d'un projet, pour que Claude Code ne les connecte jamais ni ne vous demande de les approuver. Un rejet dans n'importe quel fichier de réglages s'applique, y compris un `.claude/settings.json` de projet versionné dans le dépôt.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : tableau de chaînes, les noms des serveurs tels qu'ils apparaissent dans `.mcp.json`
* **Défaut** : non défini

```json settings.json theme={null}
{
  "disabledMcpjsonServers": ["filesystem"]
}
```

Claude Code écrit cette clé dans `.claude/settings.local.json` quand vous rejetez un serveur dans la boîte de dialogue d'approbation. `claude mcp get <name>` affiche un serveur rejeté comme `✘ Rejected (see disabledMcpjsonServers in settings)`. Le rejet l'emporte sur [`enabledMcpjsonServers`](#enabledmcpjsonservers) et [`enableAllProjectMcpServers`](#enableallprojectmcpservers).

### `enableAllProjectMcpServers`

Approuve sans demande tous les serveurs MCP définis dans les fichiers `.mcp.json` de projet. Claude Code écrit cette clé dans `.claude/settings.local.json` quand vous choisissez d'approuver tous les serveurs dans la boîte de dialogue d'approbation.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Dans un dossier dont vous n'avez pas accepté la boîte de dialogue de confiance (trust), Claude Code l'honore depuis les réglages utilisateur, les réglages gérés et `--settings`, et l'ignore dans le fichier de projet partagé, aussi bien dans la session que pour `claude mcp list` et `claude mcp get` ; [Project server approvals and workspace trust](https://code.claude.com/docs/en/mcp#project-server-approvals-and-workspace-trust) précise quand un `.claude/settings.local.json` non suivi par git compte aussi.
* **Type** : booléen
  * `true` : Claude Code approuve sans demande tous les serveurs MCP définis dans les fichiers `.mcp.json` de projet
  * `false` : Claude Code vous demande d'approuver chaque serveur. Dans un dossier de confiance, un `false` dans un fichier de priorité plus haute surcharge un `true` dans un fichier plus bas ; dans un dossier non approuvé, un `true` dans n'importe quel fichier honoré suffit
* **Défaut** : non défini, donc Claude Code vous demande d'approuver chaque serveur

```json settings.json theme={null}
{
  "enableAllProjectMcpServers": true
}
```

Une entrée [`disabledMcpjsonServers`](#disabledmcpjsonservers) rejette toujours un serveur.

### `enabledMcpjsonServers`

Approuve des serveurs précis définis dans les fichiers `.mcp.json` de projet pour que Claude Code les connecte sans demander. Claude Code écrit cette clé dans `.claude/settings.local.json` quand vous approuvez un serveur dans la boîte de dialogue d'approbation.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Dans un dossier dont vous n'avez pas accepté la boîte de dialogue de confiance, Claude Code l'honore depuis les réglages utilisateur, les réglages gérés et `--settings`, et l'ignore dans le fichier de projet partagé, aussi bien dans la session que pour `claude mcp list` et `claude mcp get` ; [Project server approvals and workspace trust](https://code.claude.com/docs/en/mcp#project-server-approvals-and-workspace-trust) précise quand un `.claude/settings.local.json` non suivi compte aussi.
* **Type** : tableau de chaînes, les noms des serveurs tels qu'ils apparaissent dans `.mcp.json`
* **Défaut** : non défini

Cet exemple approuve les serveurs `memory` et `github` du `.mcp.json` du projet :

```json settings.json theme={null}
{
  "enabledMcpjsonServers": ["memory", "github"]
}
```

Une entrée [`disabledMcpjsonServers`](#disabledmcpjsonservers) rejette toujours un serveur.

### `managedMcpServers`

Fournit des serveurs MCP distants à tous les utilisateurs depuis les réglages gérés. Les utilisateurs conservent les serveurs qu'ils ajoutent eux-mêmes et ne peuvent ni modifier ni supprimer ceux que vous fournissez. (v2.1.259+)

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code abandonne la clé avec un avertissement dans les réglages utilisateur, projet et local, et ne la lit pas dans l'onglet Code de l'application Claude Desktop sur un déploiement tiers (third-party) ni dans les sessions Cowork de l'application, où Claude Desktop fournit et verrouille lui-même les serveurs MCP de ces sessions.
* **Type** : objet indexé par nom de serveur. Chaque entrée a la forme `.mcp.json` d'un serveur `http` ou `sse` : une `url` `https://` obligatoire, et optionnellement `headers`, `oauth` et les autres options HTTP et SSE. Claude Code abandonne les entrées qui échouent à la validation ; [What an entry can contain](https://code.claude.com/docs/en/managed-mcp#what-an-entry-can-contain) liste les conditions
* **Défaut** : non défini, donc les réglages gérés ne fournissent aucun serveur

Cet exemple fournit un serveur HTTP nommé `search` :

```json managed-settings.json theme={null}
{
  "managedMcpServers": {
    "search": {
      "type": "http",
      "url": "https://search.example.com/mcp"
    }
  }
}
```

Pour la priorité, la façon dont les serveurs fournis se combinent avec `managed-mcp.json` et les listes d'autorisation et de refus, et ce que voient les utilisateurs, voir [Provide servers through managed settings](https://code.claude.com/docs/en/managed-mcp#provide-servers-through-managed-settings).

## Agents, sessions et worktrees

Définissent l'agent par défaut, contrôlent les coéquipiers (teammates) et la messagerie entre sessions, et configurent les worktrees. Voir [Subagents](https://code.claude.com/docs/en/sub-agents) et [Worktrees](https://code.claude.com/docs/en/worktrees).

### `agent`

Exécute le fil principal comme un [sous-agent](https://code.claude.com/docs/en/sub-agents#invoke-subagents-explicitly) nommé : Claude Code applique à votre session le prompt système, les restrictions d'outils et le modèle de ce sous-agent. La même clé définit l'agent par défaut des sessions que vous lancez depuis `claude agents`.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, le nom d'un agent intégré ou personnalisé
* **Défaut** : non défini, donc le fil principal s'exécute comme l'agent par défaut de Claude Code
* **Surcharges par session** : `--agent` l'emporte sur cette clé pour une session

```json settings.json theme={null}
{
  "agent": "code-reviewer"
}
```

Le `settings.json` propre d'un plugin peut aussi fournir cette clé ; voir [Ship default settings with your plugin](https://code.claude.com/docs/en/plugins/components#default-settings).

### `crossSessionInbound`

Choisit ce que fait cette session des [messages arrivant de vos autres sessions Claude Code](https://code.claude.com/docs/en/cross-session-messaging#control-inbound-messages). Quand aucune valeur ne s'applique, Claude Code décide message par message à partir des classes de mode de permission des deux sessions. (v2.1.224+)

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Une valeur de projet ou locale ne s'applique que si elle est plus stricte que celle donnée par les réglages gérés, le flag `--settings` ou les réglages utilisateur.
* **Type** : chaîne, parmi :
  * `"accept"` : Claude Code remet le message à Claude
  * `"hold"` : Claude Code affiche un avis pour le message sans le remettre
  * `"refuse"` : Claude Code abandonne le message
* **Défaut** : non défini, donc Claude Code décide message par message

```json settings.json theme={null}
{
  "crossSessionInbound": "hold"
}
```

Claude Code lit d'abord les réglages gérés, puis le flag `--settings`, puis les réglages utilisateur, et applique la première valeur trouvée. `refuse` est plus strict que `hold`, et `hold` plus strict que `accept`. Quand aucune de ces sources de confiance ne définit de valeur, un `hold` ou `refuse` de projet ou local s'applique quand même, à la place du comportement par défaut message par message. Dans les sessions avec messagerie inter-sessions, cette clé apparaît dans `/config` sous **Messages from your other sessions**, qui l'écrit dans les réglages utilisateur ; la ligne requiert la v2.1.232+, et Claude Code la masque tant que le flag `--settings` ou les réglages gérés définissent la clé.

Claude Code [avertit](https://code.claude.com/docs/en/errors#crosssessioninbound-must-be-one-of-accept-hold-refuse) quand vous définissez une valeur qu'il ne reconnaît pas. Tant que cette valeur est présente dans un fichier utilisateur, projet, local ou `--settings`, Claude Code retient (hold) les messages entrants, même si une source prioritaire définit `accept`. Un `refuse` défini par une autre source s'applique toujours. Corrigez ou supprimez la valeur pour lever la retenue.

Quand la valeur non reconnue est dans les [réglages gérés](https://code.claude.com/docs/en/managed-settings), Claude Code la traite plutôt comme `refuse` jusqu'à ce qu'un administrateur la corrige. Avant la v2.1.248, Claude Code ignorait une valeur non reconnue sans avertissement.

### `disableAgentView`

Désactive les [agents en arrière-plan et la vue agents (agent view)](https://code.claude.com/docs/en/agent-view) : `claude agents`, `--bg`, `/background` et le superviseur à la demande. Définissez-la dans les [réglages gérés](https://code.claude.com/docs/en/managed-settings) pour l'imposer à une organisation.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code désactive `claude agents`, `--bg`, `/background` et le superviseur à la demande
  * `false` : la vue agents est disponible
* **Défaut** : non défini, donc la vue agents est disponible
* **Surcharges par session** : [`CLAUDE_CODE_DISABLE_AGENT_VIEW`](https://code.claude.com/docs/en/env-vars) désactive la vue agents pour une session ; quel que soit celui des deux qui la désactive, l'autre ne peut pas la réactiver

```json settings.json theme={null}
{
  "disableAgentView": true
}
```

### `isolatePeerMachines`

Exige votre approbation explicite avant que le `SendMessage` de Claude n'atteigne une de vos sessions située sur une autre machine ; voir [Require approval for cross-machine messages](https://code.claude.com/docs/en/cross-session-messaging#require-approval-for-cross-machine-messages). La demande d'approbation apparaît même en [mode `bypassPermissions`](https://code.claude.com/docs/en/permission-modes#skip-all-checks-with-bypasspermissions-mode).

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Un `true` de n'importe quelle portée s'applique : un fichier de projet versionné peut activer l'exigence mais pas la désactiver.
* **Type** : booléen
  * `true` : Claude Code demande votre approbation avant que le `SendMessage` de Claude n'atteigne une de vos sessions sur une autre machine
  * `false` : les messages inter-machines ne déclenchent pas de demande
* **Défaut** : non défini, donc les messages inter-machines ne déclenchent pas de demande

```json settings.json theme={null}
{
  "isolatePeerMachines": true
}
```

L'approbation du `SendMessage` inter-machines requiert la v2.1.224+.

### `processWrapper`

Sur macOS et Linux, place une commande de lanceur d'entreprise (corporate launcher) devant les [processus en arrière-plan que Claude Code démarre](https://code.claude.com/docs/en/corporate-launcher#what-the-launcher-covers). Claude Code exécute le lanceur avec sa propre ligne de commande ajoutée à la fin, donc le lanceur doit faire un `exec` vers Claude Code ; voir [Run Claude Code behind a corporate launcher](https://code.claude.com/docs/en/corporate-launcher) pour le contrat du lanceur. (v2.1.210+)

* **Portée** : [`User or managed`](https://code.claude.com/docs/en/settings-reference#scopes) (utilisateur ou géré)
* **Type** : chaîne, la commande du lanceur sous forme de préfixe argv, comme un chemin absolu avec des arguments optionnels
* **Défaut** : non défini, donc les processus en arrière-plan démarrent sans enveloppe
* **Surcharges par session** : [`CLAUDE_CODE_PROCESS_WRAPPER`](https://code.claude.com/docs/en/env-vars) l'emporte sur cette clé pour une session

```json settings.json theme={null}
{
  "processWrapper": "/opt/corp/launcher --profile claude"
}
```

Claude Code ignore le lanceur sous Windows et démarre chaque processus sans enveloppe. (v2.1.210+)

### `teammateMode`

Choisit où Claude Code affiche les coéquipiers d'une [équipe d'agents (agent team)](https://code.claude.com/docs/en/agent-teams) : dans votre panneau de terminal principal, ou dans des panneaux divisés quand votre terminal le permet. Voir [Choose a display mode](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode).

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code lit aussi une valeur laissée dans `~/.claude.json` par d'anciennes versions.
* **Type** : chaîne, parmi :
  * `"in-process"` : les coéquipiers s'exécutent dans votre panneau de terminal principal
  * `"auto"` : panneaux divisés quand vous êtes dans tmux, ou dans iTerm2 avec `it2` dans votre `PATH` ou tmux installé ; in-process sinon
  * `"tmux"` : panneaux divisés via tmux ou iTerm2, détectés d'après votre terminal
  * `"iterm2"` : panneaux divisés natifs d'iTerm2 via la CLI `it2`
* **Défaut** : `"in-process"`
* **Surcharges par session** : `--teammate-mode` l'emporte sur cette clé pour une session

```json settings.json theme={null}
{
  "teammateMode": "auto"
}
```

### `worktree`

Configure la façon dont Claude Code crée et gère les [worktrees git](https://code.claude.com/docs/en/worktrees) pour `--worktree`, l'outil `EnterWorktree`, ainsi que les sous-agents isolés et les sessions en arrière-plan.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : objet avec `baseRef`, `symlinkDirectories`, `sparsePaths` et `bgIsolation`
* **Défaut** : non défini

Cet exemple fait partir les nouveaux worktrees de votre `HEAD` actuel et crée un lien symbolique de `node_modules` dans chacun :

```json settings.json theme={null}
{
  "worktree": {
    "baseRef": "head",
    "symlinkDirectories": ["node_modules"]
  }
}
```

Pour copier des fichiers ignorés par git comme `.env` dans les nouveaux worktrees, ajoutez plutôt un [fichier `.worktreeinclude`](https://code.claude.com/docs/en/worktrees#copy-gitignored-files-into-worktrees) à la racine de votre projet, au lieu d'un réglage.

### `worktree.baseRef`

Choisit la référence (ref) à partir de laquelle les nouveaux worktrees créent leur branche. `"fresh"` part de `origin/<default-branch>` pour un arbre propre identique au distant ; `"head"` part de votre `HEAD` local actuel, de sorte que les commits non poussés et l'état de la branche de fonctionnalité sont présents dans le worktree.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, parmi :
  * `"fresh"` : les nouveaux worktrees partent de `origin/<default-branch>`
  * `"head"` : les nouveaux worktrees partent de votre `HEAD` local actuel, commits non poussés compris
* **Défaut** : `"fresh"`

```json settings.json theme={null}
{
  "worktree": {
    "baseRef": "head"
  }
}
```

Dans un worktree lié, `"head"` se résout vers le `HEAD` de ce worktree, pas celui du checkout principal.

### `worktree.symlinkDirectories`

Crée des liens symboliques de dossiers du dépôt principal dans chaque worktree, pour ne pas dupliquer de gros dossiers sur le disque.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : tableau de chaînes, chemins de dossiers relatifs à la racine du dépôt
* **Défaut** : non défini, donc Claude Code ne crée aucun lien symbolique

Cet exemple lie `node_modules` et `.cache` du dépôt principal dans chaque nouveau worktree :

```json settings.json theme={null}
{
  "worktree": {
    "symlinkDirectories": ["node_modules", ".cache"]
  }
}
```

### `worktree.sparsePaths`

N'extrait (checkout) que les dossiers listés dans chaque worktree, via le sparse-checkout de git. Claude Code n'écrit sur le disque que ces dossiers plus les fichiers à la racine, ce qui est plus rapide dans les gros monorepos ; voir [Check out only the directories you need](https://code.claude.com/docs/en/large-codebases#check-out-only-the-directories-you-need).

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : tableau de chaînes, chemins de dossiers relatifs à la racine du dépôt
* **Défaut** : non défini, donc chaque worktree extrait l'arbre complet

Cet exemple n'extrait que `packages/my-app` et `shared/utils`, plus les fichiers racine, dans chaque worktree :

```json settings.json theme={null}
{
  "worktree": {
    "sparsePaths": ["packages/my-app", "shared/utils"]
  }
}
```

Tant qu'un worktree sparse existe, git active `extensions.worktreeConfig` dans le `.git/config` partagé du dépôt.

### `worktree.bgIsolation`

Choisit comment les [sessions en arrière-plan](https://code.claude.com/docs/en/agent-view#how-file-edits-are-isolated) isolent leurs modifications de fichiers. Si vous avez envoyé une session en arrière-plan avec `←` ou `/background`, elle modifie les fichiers sur place quelle que soit cette clé. Avec `"worktree"`, Claude Code bloque `Edit` et `Write` dans le checkout principal jusqu'à ce que la session appelle `EnterWorktree` ; avec `"none"`, les tâches en arrière-plan modifient directement la copie de travail. Choisissez `"none"` pour un dépôt où les worktrees git sont peu pratiques.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, parmi :
  * `"worktree"` : Claude Code bloque `Edit` et `Write` dans le checkout principal jusqu'à ce que la session appelle `EnterWorktree`
  * `"none"` : les tâches en arrière-plan modifient directement la copie de travail
* **Défaut** : `"worktree"`

```json settings.json theme={null}
{
  "worktree": {
    "bgIsolation": "none"
  }
}
```

En dehors d'un dépôt git, un [hook `WorktreeCreate`](https://code.claude.com/docs/en/worktrees#non-git-version-control) qui échoue lève le blocage pour que la session puisse modifier le répertoire de travail sur place.

### `worktree.location`

Choisit le dossier, sur une machine distante, où les [sessions SSH de Desktop](https://code.claude.com/docs/en/desktop#choose-where-ssh-session-worktrees-go) créent leurs worktrees, au lieu de `<project-root>/.claude/worktrees/`. Seule l'application desktop lit cette clé : `--worktree`, l'outil `EnterWorktree`, les sous-agents isolés et les sessions en arrière-plan l'ignorent. (Claude Desktop v1.44121.0+)

* **Portée** : [`User`](https://code.claude.com/docs/en/settings-reference#scopes), dans `~/.claude/settings.json` sur la machine distante
* **Type** : chaîne, un chemin absolu ou commençant par `~/`
* **Défaut** : non défini, donc les worktrees vont à l'intérieur du projet

Cet exemple définit le dossier `~/worktrees` :

```json settings.json theme={null}
{
  "worktree": {
    "location": "~/worktrees"
  }
}
```

Un **Worktree folder** défini sur la connexion SSH dans Desktop l'emporte. Si votre organisation restreint les dossiers que les sessions peuvent utiliser, Desktop garde les worktrees à l'intérieur du projet.

## Remote Control, desktop et notifications

Configurent Remote Control, les environnements cloud, l'application desktop et les notifications que Claude Code envoie quand il a besoin de vous. Voir [Remote Control](https://code.claude.com/docs/en/remote-control).

### `agentPushNotifEnabled`

Autorise Claude à envoyer une notification push sur votre téléphone quand il juge que cela en vaut la peine, par exemple à la fin d'une longue tâche. Claude Code synchronise ce choix avec votre compte, et les push arrivent tant que [Remote Control](https://code.claude.com/docs/en/remote-control) est connecté. Apparaît dans `/config` sous **Push when Claude decides**.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code lit aussi une valeur laissée dans `~/.claude.json` par d'anciennes versions.
* **Type** : booléen
  * `true` : Claude peut envoyer une notification push sur votre téléphone quand il le juge utile
  * `false` : Claude n'envoie pas ces notifications
* **Défaut** : `false`

```json settings.json theme={null}
{
  "agentPushNotifEnabled": true
}
```

Voir [Mobile push notifications](https://code.claude.com/docs/en/remote-control#mobile-push-notifications).

### `awaySummaryEnabled`

Affiche un récapitulatif de session d'une ligne quand vous revenez au terminal après quelques minutes d'absence. Définissez-la à `false`, ou désactivez **Session recap** dans `/config`, pour supprimer ce récapitulatif.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : vous voyez un récapitulatif d'une ligne à votre retour après quelques minutes d'absence
  * `false` : Claude Code n'affiche aucun récapitulatif
* **Défaut** : non défini, donc le récapitulatif est activé
* **Surcharges par session** : [`CLAUDE_CODE_ENABLE_AWAY_SUMMARY`](https://code.claude.com/docs/en/env-vars) l'emporte sur cette clé pour une session, dans les deux sens

```json settings.json theme={null}
{
  "awaySummaryEnabled": false
}
```

Claude Code n'affiche jamais le récapitulatif en mode non interactif.

### `disableArtifact`

> **Avertissement** : obsolète (deprecated), remplacée par [`enableArtifact`](#enableartifact). Claude Code honore encore `disableArtifact: true` comme équivalent à `enableArtifact: false`, et ignore `disableArtifact: false`.

Utilisez plutôt [`enableArtifact`](#enableartifact) pour désactiver l'outil [Artifact](https://code.claude.com/docs/en/artifacts), qui publie la sortie d'une session sous forme de page web privée sur claude.ai. Quand vous désactivez la ligne **Artifacts** dans `/config`, Claude Code écrit `enableArtifact` dans vos réglages utilisateur et efface cette clé.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code désactive l'outil Artifact pour chaque session à laquelle le fichier s'applique, et aucun autre fichier ne le réactive
  * `false` : ignoré ; pour laisser l'outil activé, supprimez la clé
* **Défaut** : non défini, donc l'outil suit la [disponibilité](https://code.claude.com/docs/en/artifacts#availability) de votre compte
* **Surcharges par session** : [`CLAUDE_CODE_DISABLE_ARTIFACT`](https://code.claude.com/docs/en/env-vars) défini à `1` désactive l'outil pour une session

```json settings.json theme={null}
{
  "disableArtifact": true
}
```

[Disable artifacts](https://code.claude.com/docs/en/artifacts#disable-artifacts) liste toutes les façons de désactiver l'outil.

### `disableDeepLinkRegistration`

Empêche Claude Code d'enregistrer le gestionnaire de protocole `claude-cli://` auprès du système d'exploitation, ce qu'il fait sinon après l'envoi du premier prompt d'une session interactive. Les [liens profonds (deep links)](https://code.claude.com/docs/en/deep-links) permettent à des outils externes d'ouvrir une session Claude Code avec un prompt prérempli. Définissez-la dans les environnements où l'enregistrement de gestionnaires de protocole est restreint ou géré séparément.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : la chaîne `"disable"`
* **Défaut** : non défini, donc Claude Code enregistre le gestionnaire

```json settings.json theme={null}
{
  "disableDeepLinkRegistration": "disable"
}
```

### `disableDesktopLocalSessions`

Désactive les sessions Code qui s'exécutent sur l'appareil dans l'[application desktop](https://code.claude.com/docs/en/desktop#local-sessions-on-managed-devices), pour les déploiements où les développeurs doivent travailler sur des machines distantes via SSH. Dans l'onglet Code, l'environnement **Local** reste dans la liste déroulante des environnements mais il est grisé et non sélectionnable, avec une infobulle indiquant que votre organisation l'a désactivé ; sous Windows, l'entrée WSL est grisée de la même façon, mais le fait que des sessions WSL s'exécutent ou non sur un appareil géré est [régi séparément](https://code.claude.com/docs/en/admin-setup#wsl-sessions-in-claude-code-desktop). Les nouvelles sessions utilisent par défaut la première [connexion SSH](https://code.claude.com/docs/en/desktop#ssh-sessions) si une est configurée, et l'application refuse de démarrer ou reprendre une session sur l'appareil, y compris une connexion SSH vers la même machine. Les sessions SSH vers d'autres hôtes et les sessions cloud ne sont pas concernées. L'application desktop lit cette clé ; la CLI en terminal l'ignore. (Claude Desktop v1.37937.0+)

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes). Par défaut, l'application desktop lit la clé depuis [une seule source gérée](https://code.claude.com/docs/en/managed-settings#how-claude-code-combines-managed-sources).
* **Type** : booléen ; seul le booléen JSON `true` a un effet
  * `true` : l'application desktop ne propose aucune session Code sur l'appareil ; les sessions locales existantes restent listées mais ne peuvent pas continuer
  * `false` : les sessions locales restent disponibles
* **Défaut** : non défini, donc les sessions locales sont disponibles

```json managed-settings.json theme={null}
{
  "disableDesktopLocalSessions": true
}
```

L'application desktop ignore toute autre valeur, et une valeur non booléenne, comme la chaîne `"true"` ou `1`, journalise aussi un avertissement. Associez-la à [`sshConfigs`](#sshconfigs) pour que les utilisateurs arrivent sur une connexion fonctionnelle, et à [`sshHostAllowlist`](#sshhostallowlist) pour limiter les hôtes accessibles. Voir [Local sessions on managed devices](https://code.claude.com/docs/en/desktop#local-sessions-on-managed-devices).

Claude Desktop fournit aux sessions Code une politique dérivée de votre configuration desktop, par exemple l'allowlist de sortie réseau (egress), le bac à sable du système de fichiers et les restrictions MCP des déploiements tiers. Claude Code ignore ces réglages parents dès qu'une [source d'administration](https://code.claude.com/docs/en/managed-settings#how-claude-code-combines-managed-sources) est présente : réglages gérés par serveur, politique MDM ou de niveau OS, ou fichier de réglages gérés. Déployer cette clé par l'un de ces moyens sur un appareil qui n'en avait aucun auparavant, comme dans les déploiements tiers, empêche donc les politiques dérivées du desktop de s'appliquer. [Let an embedding host add policy](https://code.claude.com/docs/en/managed-settings#let-an-embedding-host-add-policy) explique quand les réglages parents peuvent encore fusionner ; cela vaut pour toute clé déployée de cette façon, pas seulement celle-ci.

### `disableRemoteControl`

Désactive [Remote Control](https://code.claude.com/docs/en/remote-control) : Claude Code refuse alors `claude remote-control`, le flag `--remote-control`, le démarrage automatique et la bascule en cours de session, et indique que la politique de votre organisation l'a désactivé. Placez-la dans les [réglages gérés](https://code.claude.com/docs/en/managed-settings) pour une application par appareil via MDM.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code refuse `claude remote-control`, le flag `--remote-control`, le démarrage automatique et la bascule en cours de session
  * `false` : Remote Control reste disponible
* **Défaut** : `false`

```json settings.json theme={null}
{
  "disableRemoteControl": true
}
```

### `enableArtifact`

Désactive l'outil [Artifact](https://code.claude.com/docs/en/artifacts), qui publie la sortie d'une session sous forme de page web privée sur claude.ai. Quand vous désactivez la ligne **Artifacts** dans `/config`, Claude Code écrit cette clé dans vos réglages utilisateur ; vous ne la modifiez donc généralement pas à la main.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Chaque fichier peut désactiver l'outil, et aucun ne peut le réactiver.
* **Type** : booléen
  * `false` : Claude Code désactive l'outil Artifact pour chaque session à laquelle le fichier s'applique
  * `true` : identique à laisser la clé non définie, car elle ne surcharge jamais un `false` venant d'un autre fichier, de [`CLAUDE_CODE_DISABLE_ARTIFACT`](https://code.claude.com/docs/en/env-vars) ou du [réglage d'administration](https://code.claude.com/docs/en/artifacts#manage-artifacts-for-your-organization) de votre organisation
* **Défaut** : non défini, donc l'outil suit la [disponibilité](https://code.claude.com/docs/en/artifacts#availability) de votre compte

```json settings.json theme={null}
{
  "enableArtifact": false
}
```

Tant qu'une source autre que vos propres réglages utilisateur maintient l'outil désactivé, Claude Code masque la ligne **Artifacts** dans `/config`, car l'activer là ne changerait rien. [Disable artifacts](https://code.claude.com/docs/en/artifacts#disable-artifacts) liste toutes les façons de désactiver l'outil.

### `inputNeededNotifEnabled`

Envoie une notification push sur votre téléphone quand une demande de permission ou une question attend votre réponse. Claude Code ne les envoie que tant que [Remote Control](https://code.claude.com/docs/en/remote-control) est connecté. Apparaît dans `/config` sous **Push when actions required**.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code lit aussi une valeur laissée dans `~/.claude.json` par d'anciennes versions.
* **Type** : booléen
  * `true` : vous recevez une notification push sur votre téléphone quand une demande de permission ou une question attend, tant que Remote Control est connecté
  * `false` : Claude Code n'envoie pas de telles notifications
* **Défaut** : `false`

```json settings.json theme={null}
{
  "inputNeededNotifEnabled": true
}
```

Voir [Mobile push notifications](https://code.claude.com/docs/en/remote-control#mobile-push-notifications).

### `preferredNotifChannel`

Choisit comment Claude Code vous prévient quand une tâche se termine ou qu'une demande de permission attend. Apparaît dans `/config` sous **Local notifications**.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code lit aussi une valeur laissée dans `~/.claude.json` par d'anciennes versions.
* **Type** : chaîne, parmi :
  * `"auto"` : Claude Code envoie une notification de bureau dans iTerm2, Ghostty et Kitty, fait sonner la cloche dans Terminal.app uniquement quand sa cloche audible est désactivée, et ne fait rien ailleurs
  * `"terminal_bell"` : Claude Code émet le caractère cloche (bell) dans n'importe quel terminal
  * `"iterm2"` : Claude Code envoie une notification de bureau iTerm2
  * `"iterm2_with_bell"` : Claude Code envoie une notification de bureau iTerm2 et fait sonner la cloche
  * `"kitty"` : Claude Code envoie une notification de bureau Kitty
  * `"ghostty"` : Claude Code envoie une notification de bureau Ghostty
  * `"notifications_disabled"` : Claude Code n'envoie aucune notification
* **Défaut** : `"auto"`

```json settings.json theme={null}
{
  "preferredNotifChannel": "terminal_bell"
}
```

Avec `"auto"`, Claude Code envoie une notification de bureau dans iTerm2, Ghostty et Kitty. Dans Terminal.app, il émet le caractère cloche uniquement si vous avez désactivé la cloche audible de Terminal, et dans les autres terminaux il ne fait rien. Choisissez `"terminal_bell"` pour émettre le caractère cloche dans n'importe quel terminal. Voir [Get a terminal bell or notification](https://code.claude.com/docs/en/terminal-config#get-a-terminal-bell-or-notification).

### `remote.defaultEnvironmentId`

Choisit l'[environnement cloud](https://code.claude.com/docs/en/cloud-environments) par défaut des sessions cloud que vous créez depuis la CLI, par exemple avec `claude --cloud`. Claude Code écrit cette clé dans vos réglages utilisateur quand vous choisissez un environnement avec [`/remote-env`](https://code.claude.com/docs/en/cloud-environments#select-an-environment-from-the-cli).

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Pour un ID d'environnement auto-hébergé, uniquement les réglages utilisateur ou gérés, ou le flag `--settings`.
* **Type** : chaîne, un ID d'environnement comme `env_...` ou `ccpool_...`
* **Défaut** : non défini, donc Claude Code utilise l'environnement hébergé par Anthropic quand votre liste en contient un, sinon le premier environnement de votre liste qui n'est pas un [environnement passerelle (bridge) Remote Control](https://code.claude.com/docs/en/cloud-environments#the-default-environment), ou le premier environnement quand tous sont des environnements passerelle
* **Surcharges par session** : `--environment` l'emporte sur cette clé pour la session cloud qu'il crée

```json settings.json theme={null}
{
  "remote": {
    "defaultEnvironmentId": "env_0123abcd"
  }
}
```

Un ID d'environnement hébergé par Anthropic, qui commence par `env_`, suit l'ordre de priorité standard des réglages : une valeur dans les réglages de projet d'un dépôt surcharge votre choix au niveau utilisateur. Un ID d'[environnement auto-hébergé](https://code.claude.com/docs/en/self-hosted-environments), qui commence par `ccpool_`, n'est honoré que depuis les réglages utilisateur, les réglages gérés et le flag `--settings` ; Claude Code l'ignore dans les réglages de projet ou locaux d'un dépôt, et `/remote-env` indique quelle valeur il a ignorée — ainsi, un fichier versionné ne peut pas diriger des sessions vers un environnement auto-hébergé que vous n'avez pas choisi.

### `remoteControlAtStartup`

Connecte automatiquement [Remote Control](https://code.claude.com/docs/en/remote-control) au démarrage de chaque session interactive, au lieu d'attendre `/remote-control`. Définissez-la à `true` pour activer la connexion automatique, `false` pour la désactiver. Apparaît dans `/config` sous **Enable Remote Control for all sessions**.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code lit aussi une valeur laissée dans `~/.claude.json` par d'anciennes versions.
* **Type** : booléen
  * `true` : Claude Code connecte Remote Control automatiquement au démarrage de chaque session interactive
  * `false` : Claude Code attend `/remote-control`
* **Défaut** : non défini, donc le [comportement par défaut de connexion automatique](https://code.claude.com/docs/en/remote-control#enable-remote-control-for-all-sessions) s'applique
* **Surcharges par session** : `--remote-control` active Remote Control pour une session même quand cette clé vaut `false`, et aucun flag ne le désactive pour une session

```json settings.json theme={null}
{
  "remoteControlAtStartup": true
}
```

Claude Code ignore un `true` venant des réglages de projet ou locaux : un dépôt peut désactiver la connexion automatique pour son checkout, mais pas l'activer. Pour le comportement complet par portée, voir [Enable Remote Control for all sessions](https://code.claude.com/docs/en/remote-control#enable-remote-control-for-all-sessions) et les [clés de sécurité où la valeur la plus stricte s'applique](https://code.claude.com/docs/en/settings#security-keys-where-the-stricter-value-applies).

### `sshConfigs`

Ajoute des connexions SSH à la liste déroulante des environnements de [Desktop](https://code.claude.com/docs/en/desktop#pre-configure-ssh-connections-for-your-team). Les administrateurs l'utilisent pour distribuer des connexions partagées à une équipe. Les connexions définies dans les réglages gérés apparaissent comme gérées. Les utilisateurs peuvent les sélectionner et [définir leur propre **Worktree folder**](https://code.claude.com/docs/en/desktop#choose-where-ssh-session-worktrees-go) pour elles, mais ne peuvent rien modifier d'autre ni les supprimer dans l'application.

* **Portée** : [`User or managed`](https://code.claude.com/docs/en/settings-reference#scopes). L'application desktop lit cette clé. Par défaut, elle lit les connexions gérées depuis [une seule source gérée](https://code.claude.com/docs/en/managed-settings#how-claude-code-combines-managed-sources).
* **Type** : tableau d'objets, chacun avec `id`, `name` et `sshHost` obligatoires, et `sshPort` et `sshIdentityFile` optionnels
* **Défaut** : non défini

Cet exemple ajoute une connexion nommée `Dev VM` qui se connecte à `user@dev.example.com` :

```json settings.json theme={null}
{
  "sshConfigs": [
    {
      "id": "dev-vm",
      "name": "Dev VM",
      "sshHost": "user@dev.example.com"
    }
  ]
}
```

### `sshHostAllowlist`

Limite les hôtes auxquels une [session SSH de Desktop](https://code.claude.com/docs/en/desktop#restrict-which-ssh-hosts-users-can-connect-to) peut se connecter. Seule l'application Desktop lit cette clé ; la CLI non. Les motifs sont insensibles à la casse : `*` correspond à n'importe quel hôte, `*.example.com` correspond à `example.com` et à tous ses sous-domaines, et toute autre valeur est une correspondance exacte avec le nom d'hôte après résolution par `~/.ssh/config`. Un tableau vide désactive les sessions SSH.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes). Par défaut, Desktop lit la clé depuis [une seule source gérée](https://code.claude.com/docs/en/managed-settings#how-claude-code-combines-managed-sources).
* **Type** : tableau de motifs de noms d'hôte
* **Défaut** : non défini, donc tout hôte est autorisé

Cet exemple autorise `devboxes.example.com` et ses sous-domaines, plus l'hôte exact `bastion.example.com` :

```json managed-settings.json theme={null}
{
  "sshHostAllowlist": ["*.devboxes.example.com", "bastion.example.com"]
}
```

Une valeur que Desktop ne peut pas lire comme une liste d'hôtes, comme `true` ou un objet, compte comme un tableau vide jusqu'à correction, sauf `null`, qui compte comme non défini. (Claude Desktop v2.26454.0+)

Si vous définissez [`managedSourcesBehavior`](#managedsourcesbehavior) à `"merge"` dans votre source de rang le plus élevé, Desktop combine les listes de toutes les [sources d'administration](https://code.claude.com/docs/en/managed-settings#how-claude-code-combines-managed-sources) et autorise un hôte qui correspond à l'une d'elles. Si vous définissez un tableau vide dans une source, les sessions SSH restent actives pour les hôtes listés par une autre source.

## Authentification et fournisseurs

Fournissent des identifiants via des scripts auxiliaires (helpers) et, pour les organisations, imposent une méthode de connexion ou une organisation. Voir [Authentication](https://code.claude.com/docs/en/authentication).

### `allowedProviders`

Liste les services par lesquels une machine peut joindre Claude, comme l'API Anthropic, Amazon Bedrock ou une passerelle LLM (LLM gateway). Une session sur un fournisseur non listé est refusée au démarrage, à la connexion et lors de son prochain contact avec l'API ; passer à un fournisseur non listé en cours de session est donc refusé aussi. Le [message de refus](https://code.claude.com/docs/en/errors#managed-settings-dont-allow-this-api-provider) indique ce qui a sélectionné le fournisseur et les étapes pour continuer. (v2.1.285+)

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes). Une liste définie par les propres sources d'administration de la machine (politiques MDM et fichiers de réglages gérés) continue de s'appliquer quand les réglages gérés par serveur en fournissent aussi une : une session ne peut alors utiliser que les fournisseurs présents sur les deux listes, si bien qu'une liste gérée par serveur peut restreindre ce que la machine autorise mais jamais l'élargir. La source de la machine dont l'`allowedProviders` compte suit [la façon dont Claude Code combine les sources gérées](https://code.claude.com/docs/en/managed-settings#how-claude-code-combines-managed-sources). Une liste fournie uniquement par les réglages gérés par serveur n'atteint que les sessions qui [récupèrent les réglages gérés par serveur](https://code.claude.com/docs/en/server-managed-settings#platform-availability).
* **Type** : tableau de chaînes, chacune parmi :
  * `"anthropic"` : l'API Anthropic sur l'hôte propre d'Anthropic, via une connexion claude.ai ou Console ou une clé API. Associez-la à [`forceLoginMethod`](#forceloginmethod) ou [`forceLoginOrgUUID`](#forceloginorguuid) pour restreindre aussi la connexion
  * `"bedrock"` : [Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock)
  * `"vertex"` : l'[Agent Platform de Google Cloud](https://code.claude.com/docs/en/google-vertex-ai), anciennement Vertex AI
  * `"foundry"` : [Microsoft Foundry](https://code.claude.com/docs/en/microsoft-foundry)
  * `"anthropicAws"` : [Claude Platform on AWS](https://code.claude.com/docs/en/claude-platform-on-aws)
  * `"mantle"` : l'[endpoint Mantle](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint) d'Amazon Bedrock. Une session qui [exécute Mantle en parallèle de l'API Invoke](https://code.claude.com/docs/en/amazon-bedrock#run-mantle-alongside-the-invoke-api) utilise les deux fournisseurs : listez alors `"bedrock"` et `"mantle"` ensemble
  * `"customEndpoint"` : l'API Anthropic ou l'API d'un fournisseur cloud envoyée vers un autre hôte, comme une [passerelle LLM](https://code.claude.com/docs/en/llm-gateway) désignée par `ANTHROPIC_BASE_URL` ou par la variable d'endpoint d'un fournisseur comme `ANTHROPIC_BEDROCK_BASE_URL`. Claude Code ne l'admet que pour la valeur exacte qu'un bloc [`env`](#env) géré épingle (pin)
  * `"gateway"` : une connexion via une [passerelle cloud (Cloud gateway)](https://code.claude.com/docs/en/claude-apps-gateway)
* **Défaut** : non défini, donc n'importe quel fournisseur peut être utilisé

```json managed-settings.json theme={null}
{
  "allowedProviders": ["anthropic", "bedrock"]
}
```

L'entrée de chaque fournisseur cloud désigne le service propre de ce fournisseur, y compris ses endpoints régionaux, FIPS et privés.

Une entrée que Claude Code ne reconnaît pas comme nom de fournisseur est abandonnée et signalée, et le reste de la liste reste appliqué. Avec une liste vide, ou dont toutes les entrées sont non reconnues, Claude Code refuse tous les fournisseurs et ne démarre pas sur la machine.

#### Endpoints qui nécessitent un épinglage dans `env` géré

Un épinglage (pin) est la valeur d'une variable d'endpoint définie dans un bloc [`env`](#env) géré. Quand une session envoie le trafic d'un fournisseur ailleurs que vers le service propre de ce fournisseur, Claude Code ne l'admet que si la valeur de la session est identique à l'épinglage. Ces endpoints en nécessitent un :

* **Sessions `"customEndpoint"`** : la variable qui désigne l'hôte, comme `ANTHROPIC_BASE_URL`
* **Amazon Bedrock** : les variables du SDK AWS `AWS_ENDPOINT_URL`, `AWS_ENDPOINT_URL_BEDROCK` et `AWS_ENDPOINT_URL_BEDROCK_RUNTIME` quand elles pointent hors du service propre de Bedrock. La session reste sous `"bedrock"` et non `"customEndpoint"`
* **L'URL d'une connexion via passerelle** : la session reste sous `"gateway"`, et [`forceLoginGatewayUrl`](#forcelogingatewayurl) compte aussi comme épinglage

Les blocs `env` qui comptent comme épinglages dépendent de l'endroit où la liste est définie :

* **Une source d'administration sur la machine définit une liste** : seuls les blocs `env` des propres sources d'administration de la machine comptent
* **Seuls les réglages gérés par serveur définissent une liste** : une valeur `env` dans ces réglages gérés par serveur compte aussi

La liste ne juge pas les variables d'identifiants et de tenancy d'un fournisseur cloud, ni le chemin réseau, comme `HTTPS_PROXY` et les réglages de certificats. Définissez-les pour l'ensemble du parc dans le bloc `env` géré.

### `apiKeyHelper`

Exécute votre propre commande pour produire l'identifiant que Claude Code envoie avec les requêtes au modèle. Claude Code exécute la commande via le shell système, `/bin/sh` sur macOS et Linux et `cmd` sous Windows, et envoie sa sortie à la fois dans les en-têtes `X-Api-Key` et `Authorization: Bearer`. Utilisez-la pour des identifiants dynamiques ou tournants, comme des jetons à courte durée récupérés dans un coffre (vault).

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, une ligne de commande shell
* **Défaut** : non défini, donc Claude Code n'exécute aucun helper

```json settings.json theme={null}
{
  "apiKeyHelper": "/bin/generate_temp_api_key.sh"
}
```

Claude Code met la valeur en cache et réexécute la commande dans ces cas :

* Après la durée de vie du cache, cinq minutes par défaut ou l'intervalle défini avec [`CLAUDE_CODE_API_KEY_HELPER_TTL_MS`](https://code.claude.com/docs/en/env-vars).
* Quand une requête à l'API Anthropic, directement ou via une [passerelle LLM](https://code.claude.com/docs/en/llm-gateway), échoue avec `401` ou `403`.
* Avant d'envoyer une requête à l'API Anthropic, directement ou via une passerelle LLM, quand la sortie en cache est un JWT qui a expiré après sa production par le helper. (v2.1.246+)

Les deux derniers cas ne s'appliquent que lorsque la sortie du helper est l'identifiant envoyé par Claude Code et que `ANTHROPIC_AUTH_TOKEN` n'est pas défini.

Dans les sessions interactives, quand la commande provient des réglages de projet ou locaux, Claude Code ne l'exécute pas tant que vous n'avez pas accepté la demande de confiance de l'espace de travail (workspace trust). Voir [Credential management](https://code.claude.com/docs/en/authentication#credential-management).

### `awsAuthRefresh`

Exécute votre propre commande, comme `aws sso login`, pour rafraîchir les identifiants de votre dossier `.aws` quand ceux dont Claude Code dispose pour [Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock) ne fonctionnent plus. Claude Code vérifie d'abord les identifiants actuels auprès de STS et n'exécute la commande que si cette vérification échoue, puis relit le dossier `.aws` rafraîchi.

Quand la vérification échoue au même moment dans plusieurs processus Claude Code qui utilisent la même commande et les mêmes identifiants, comme des terminaux ou fenêtres d'IDE séparés, un seul processus exécute la commande et les autres attendent ce lancement au lieu de lancer le leur. Un processus qui a attendu 60 secondes avec une requête en attente exécute lui-même la commande. Pour désactiver ce mécanisme, définissez [`CLAUDE_CODE_DISABLE_AUTH_REFRESH_LOCK`](https://code.claude.com/docs/en/env-vars) à `1`.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, une ligne de commande shell
* **Défaut** : non défini, donc Claude Code ne rafraîchit pas les identifiants AWS pour vous

```json settings.json theme={null}
{
  "awsAuthRefresh": "aws sso login --profile myprofile"
}
```

Utilisez cette clé quand votre flux de rafraîchissement écrit dans `.aws` ; utilisez [`awsCredentialExport`](#awscredentialexport) quand il affiche plutôt les identifiants. Voir [advanced credential configuration](https://code.claude.com/docs/en/amazon-bedrock#advanced-credential-configuration).

### `awsCredentialExport`

Exécute votre propre commande qui affiche des identifiants AWS au format JSON, pour que Claude Code puisse appeler [Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock) avec des identifiants qui ne se trouvent pas dans votre dossier `.aws`. Claude Code accepte la forme de sortie de `aws sts` et la forme plate de `aws configure export-credentials`, et limite ces identifiants à son propre client Bedrock : les commandes shell que Claude exécute voient donc toujours vos identifiants ambiants.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, une ligne de commande shell
* **Défaut** : non défini, donc Claude Code utilise la chaîne d'identifiants AWS ambiante

```json settings.json theme={null}
{
  "awsCredentialExport": "/bin/generate_aws_grant.sh"
}
```

Contrairement à [`awsAuthRefresh`](#awsauthrefresh), Claude Code exécute toujours cette commande quand elle est définie, sans vérifier d'abord les identifiants ambiants. Voir [advanced credential configuration](https://code.claude.com/docs/en/amazon-bedrock#advanced-credential-configuration).

### `forceLoginMethod`

Restreint le type de compte avec lequel on peut se connecter. `"claudeai"` n'autorise que les comptes claude.ai, `"console"` que les comptes Claude Console, et `"gateway"` envoie les personnes vers une [passerelle cloud](https://code.claude.com/docs/en/claude-apps-gateway) au lieu d'une connexion first-party. Les administrateurs la définissent dans les réglages gérés et l'associent à [`forceLoginOrgUUID`](#forceloginorguuid) pour garder les connexions claude.ai des développeurs dans une seule organisation. Si vous la définissez à `"claudeai"` ou `"console"` dans n'importe quel fichier de réglages, Claude Code cesse aussi de proposer la [connexion Console sans clé](https://code.claude.com/docs/en/authentication#sign-in-without-an-api-key) dans les sessions auxquelles ce fichier s'applique.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code honore `"gateway"` depuis les mêmes sources que [`forceLoginGatewayUrl`](#forcelogingatewayurl) et la traite comme non définie partout ailleurs.
* **Type** : chaîne, parmi :
  * `"claudeai"` : seuls les comptes claude.ai peuvent se connecter
  * `"console"` : seuls les comptes Claude Console peuvent se connecter
  * `"gateway"` : Claude Code envoie les personnes vers une passerelle cloud au lieu d'une connexion first-party
* **Défaut** : non défini, donc chacun choisit une méthode de connexion

```json settings.json theme={null}
{
  "forceLoginMethod": "claudeai"
}
```

Tous les chemins de connexion first-party appliquent la restriction, y compris l'[extension VS Code](https://code.claude.com/docs/en/vs-code), l'Agent SDK, `claude setup-token` et `/install-github-app`, sauf l'écran de connexion interactif du terminal, atteint par `/login` ou l'onboarding au premier lancement, qui présélectionne la méthode sans l'imposer. Avant la v2.1.212, seules les connexions en terminal l'appliquaient. Voir [Restrict login to your organization](https://code.claude.com/docs/en/authentication#restrict-login-to-your-organization) pour le traitement de chaque chemin de connexion, des identifiants d'environnement et des fournisseurs tiers.

Quand une source gérée sur la machine définit `"gateway"`, Claude Code n'utilise pas une connexion, une clé API ou un identifiant `apiKeyHelper` restant d'avant. Voir [Administrator policy requires a Cloud gateway sign-in](https://code.claude.com/docs/en/errors#administrator-policy-requires-a-cloud-gateway-sign-in) pour le message produit par chacun. Si vous sélectionnez un fournisseur cloud via `CLAUDE_CODE_USE_BEDROCK` ou une variable d'environnement similaire, la session n'a pas besoin de la connexion via passerelle. Avant la v2.1.261, Claude Code utilisait une connexion restante sur ces machines.

### `forceLoginGatewayUrl`

Définit l'URL de passerelle à laquelle se connecte l'écran Cloud gateway de `/login`, pour que les personnes atteignent votre [passerelle cloud](https://code.claude.com/docs/en/claude-apps-gateway) sans taper son adresse. L'écran n'a pas de champ URL : avec cette clé, il affiche l'URL de votre passerelle et se connecte quand la personne appuie sur Entrée ; sans elle, il lui dit de contacter son administrateur informatique.

Dans les réglages gérés, cette clé ou `forceLoginMethod: "gateway"` rend la machine exclusivement passerelle (gateway-only), sauf pour les sessions qui sélectionnent un fournisseur cloud avec `CLAUDE_CODE_USE_*`. `/login` s'ouvre alors sur l'écran Cloud gateway sans sélecteur de méthode de connexion. Voir [Administrator policy requires a Cloud gateway sign-in](https://code.claude.com/docs/en/errors#administrator-policy-requires-a-cloud-gateway-sign-in) pour ce qu'il advient d'une connexion first-party ou d'une clé API restante. Définissez les deux clés pour que l'écran se connecte au lieu d'afficher une erreur.

* **Portée** : [`User or managed`](https://code.claude.com/docs/en/settings-reference#scopes). Lue depuis une source gérée sur la machine : `managed-settings.json`, le plist macOS ou le registre Windows HKLM, ou un helper de politique (policy helper). Sur une machine sans aucune de ces sources, Claude Code v2.1.295+ la lit aussi depuis les [réglages utilisateur](https://code.claude.com/docs/en/claude-apps-gateway#set-the-gateway-url-in-user-settings). Claude Code l'ignore dans HKCU et dans les réglages gérés par serveur.
* **Type** : chaîne, une URL complète incluant le schéma
* **Défaut** : non défini, donc l'écran Cloud gateway affiche une erreur demandant de contacter l'administrateur informatique

```json managed-settings.json theme={null}
{
  "forceLoginGatewayUrl": "https://claude-gateway.example.com"
}
```

Si la valeur n'est pas une URL valide, l'écran de connexion le signale, et le reste du fichier de réglages gérés s'applique toujours. Voir [Set the gateway URL](https://code.claude.com/docs/en/claude-apps-gateway#set-the-gateway-url).

### `forceLoginOrgUUID`

Depuis une source gérée, exige que les connexions par compte claude.ai appartiennent à une organisation Anthropic, donnée par un UUID unique, ou à l'une de plusieurs organisations, données par un tableau. Depuis n'importe quel fichier de réglages, Claude Code utilise aussi un UUID unique pour présélectionner cette organisation lors d'une connexion claude.ai ou Claude Console, et ne présélectionne rien pour un tableau. Si vous définissez la clé dans n'importe quel fichier, Claude Code cesse aussi de proposer la [connexion Console sans clé](https://code.claude.com/docs/en/authentication#sign-in-without-an-api-key) dans les sessions auxquelles ce fichier s'applique et crée plutôt une clé API.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Seule une source gérée impose la restriction ; un UUID unique dans tout autre fichier présélectionne l'organisation à la connexion sans restreindre.
* **Type** : chaîne, un UUID, ou tableau de chaînes, plusieurs UUID
* **Défaut** : non défini, donc n'importe quelle organisation peut se connecter

Cet exemple accepte les connexions de deux organisations sans en présélectionner une :

```json managed-settings.json theme={null}
{
  "forceLoginOrgUUID": ["xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy"]
}
```

Si une source gérée définit un tableau vide, ou une valeur qui n'est ni une chaîne ni un tableau de chaînes, les utilisateurs qui se connectent avec un compte Anthropic ne peuvent ni démarrer Claude Code ni terminer une connexion. Ils voient un message qui nomme `forceLoginOrgUUID` et leur dit de contacter leur administrateur. Un [`policyHelper`](#policyhelper) qui émet une valeur du mauvais type [fait échouer son exécution](#échecs-du-helper) à la place.

Voir [Restrict login to your organization](https://code.claude.com/docs/en/authentication#restrict-login-to-your-organization) pour le traitement par Claude Code des connexions Claude Console, des autres chemins de connexion et des identifiants d'environnement.

### `gatewayInternalNetworks`

Déclare les blocs IPv4 publics à partir desquels votre organisation numérote son réseau interne, pour que `/login` y accepte une [passerelle cloud](https://code.claude.com/docs/en/claude-apps-gateway). (v2.1.268+)

Sans cette clé, `/login` se connecte à toute passerelle sur une adresse privée et à rien d'autre. Avec elle, `/login` accepte aussi une passerelle située dans un bloc listé, via une connexion directe uniquement. L'adresse propre de la machine sur cette connexion doit aussi se trouver dans le même bloc.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes). Lue uniquement depuis une source sur la machine : `managed-settings.json`, le plist macOS ou le registre Windows HKLM, ou un helper de politique. Claude Code l'ignore dans HKCU et dans les réglages gérés par serveur.
* **Type** : tableau de chaînes, au plus quatre blocs CIDR IPv4, chacun de `/8` à `/32`, ne se chevauchant pas entre eux, et aucun ne chevauchant l'espace privé.
* **Défaut** : non défini, donc `/login` n'accepte que les passerelles sur des adresses privées

```json managed-settings.json theme={null}
{
  "gatewayInternalNetworks": ["203.0.113.0/24"]
}
```

Remplacez la plage de documentation de l'exemple par votre propre bloc. Claude Code refuse les plages de documentation, les plages que les clients VPN et NAT64 utilisent localement, et l'espace réservé à partir duquel aucun réseau n'est numéroté, comme le multicast.

Si une entrée est invalide, ou si la valeur n'est pas une liste de chaînes, `/login` nomme le problème et refuse toute nouvelle connexion via passerelle sur la machine jusqu'à correction de la valeur. Les connexions existantes continuent de fonctionner. Voir [Allow a gateway on public address space you own](https://code.claude.com/docs/en/claude-apps-gateway#allow-a-gateway-on-public-address-space-you-own) pour les règles complètes et ce que voient les développeurs.

### `gcpAuthRefresh`

Exécute votre propre commande pour rafraîchir les Application Default Credentials de Google Cloud quand Claude Code constate qu'elles ont expiré ou ne peuvent pas être chargées, pour que les requêtes vers l'[Agent Platform de Google Cloud](https://code.claude.com/docs/en/google-vertex-ai) continuent de fonctionner sans réauthentification manuelle.

Quand plusieurs processus Claude Code qui utilisent la même commande et les mêmes identifiants, comme des terminaux ou fenêtres d'IDE séparés, les trouvent expirés au même moment, un seul processus exécute la commande et les autres attendent ce lancement au lieu de lancer le leur. Un processus qui a attendu 60 secondes avec une requête en attente exécute lui-même la commande. Pour désactiver ce mécanisme, définissez [`CLAUDE_CODE_DISABLE_AUTH_REFRESH_LOCK`](https://code.claude.com/docs/en/env-vars) à `1`.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, une ligne de commande shell
* **Défaut** : non défini, donc l'erreur d'identifiants de Claude Code vous dit d'exécuter vous-même `gcloud auth application-default login`

```json settings.json theme={null}
{
  "gcpAuthRefresh": "gcloud auth application-default login"
}
```

Voir [advanced credential configuration](https://code.claude.com/docs/en/google-vertex-ai#advanced-credential-configuration).

### `otelHeadersHelper`

Exécute votre propre commande pour générer les en-têtes que Claude Code envoie avec les exports OpenTelemetry, pour les backends dont les jetons tournent. Claude Code l'exécute au démarrage puis périodiquement, et attend sur stdout un objet JSON de valeurs d'en-têtes de type chaîne.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, un chemin d'exécutable ou une ligne de commande shell
* **Défaut** : non défini, donc Claude Code n'ajoute aucun en-tête généré par un helper

```json settings.json theme={null}
{
  "otelHeadersHelper": "/bin/generate_otel_headers.sh"
}
```

Définissez l'intervalle de rafraîchissement avec [`CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS`](https://code.claude.com/docs/en/env-vars). Voir [Dynamic headers](https://code.claude.com/docs/en/monitoring-usage#dynamic-headers) pour les exigences du script et ce qui se passe quand le helper échoue.

## Mises à jour et versions

Choisissent un canal de mise à jour et, pour les organisations, épinglent les versions que les personnes peuvent exécuter. Voir [Update Claude Code](https://code.claude.com/docs/en/setup#update-claude-code).

### `autoUpdatesChannel`

Choisit le [canal de publication](https://code.claude.com/docs/en/setup#configure-release-channel) que suivent les mises à jour automatiques en arrière-plan et `claude update`. `"stable"` donne une version généralement vieille d'environ une semaine et qui saute les publications ayant des régressions majeures ; `"latest"` donne la publication la plus récente.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Définissez-la dans les réglages gérés pour imposer un canal à toute votre organisation.
* **Type** : chaîne, parmi :
  * `"latest"` : les mises à jour suivent la publication la plus récente
  * `"stable"` : les mises à jour suivent une version généralement vieille d'environ une semaine, qui saute les publications ayant des régressions majeures
* **Défaut** : non défini, donc Claude Code suit `"latest"`

```json settings.json theme={null}
{
  "autoUpdatesChannel": "stable"
}
```

Claude Code écrit `"stable"` dans vos réglages utilisateur quand vous le choisissez sous **Auto-update channel** dans `/config`, et supprime la clé quand vous revenez à latest au même endroit. `claude install stable` et `claude install latest` enregistrent aussi le canal nommé. Passer de `"latest"` à `"stable"` dans `/config` demande s'il faut autoriser une rétrogradation (downgrade) ou rester sur la version actuelle ; rester définit [`minimumVersion`](#minimumversion). Les installations Homebrew ignorent cette clé : le cask `claude-code` suit stable et `claude-code@latest` suit latest, et `claude update` s'en remet à `brew upgrade`. Pour désactiver complètement les mises à jour automatiques, définissez [`DISABLE_AUTOUPDATER`](https://code.claude.com/docs/en/setup#disable-auto-updates) dans `env`.

### `minimumVersion`

Empêche les mises à jour automatiques en arrière-plan et `claude update` d'installer une version inférieure à celle-ci, pour que passer au canal `"stable"` ne vous fasse pas rétrograder depuis une build `"latest"` plus récente. Claude Code écrit cette clé pour vous quand vous choisissez de rester sur votre version actuelle en changeant de canal dans `/config`, et l'efface quand vous revenez à `"latest"`.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Définissez-la dans les réglages gérés pour épingler un minimum à l'échelle de l'organisation que les réglages utilisateur et projet ne peuvent pas abaisser.
* **Type** : chaîne, un numéro de version comme `"2.1.100"` ; une valeur qui n'est pas une version valide est ignorée
* **Défaut** : non défini, donc les mises à jour peuvent installer n'importe quelle version proposée par le canal

Cet exemple suit le canal stable et refuse d'installer toute version inférieure à 2.1.100 :

```json settings.json theme={null}
{
  "autoUpdatesChannel": "stable",
  "minimumVersion": "2.1.100"
}
```

Cette clé ne contraint que les mises à jour. Pour que Claude Code refuse de démarrer en dessous d'une version, utilisez plutôt [`requiredMinimumVersion`](#requiredminimumversion). Voir [Pin a minimum version](https://code.claude.com/docs/en/setup#pin-a-minimum-version).

### `requiredMaximumVersion`

Définit la version la plus récente de Claude Code que votre organisation autorise à démarrer. Quand la version en cours est plus récente, Claude Code quitte au démarrage et dit à l'utilisateur d'installer une version approuvée par la méthode approuvée de votre organisation ; `claude install <version>` peut aussi fonctionner. (v2.1.163+)

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code ne donne aucun avertissement quand il ignore la clé ailleurs.
* **Type** : chaîne, un numéro de version comme `"2.1.150"` ; une valeur qui n'est pas une version valide est ignorée
* **Défaut** : non défini, donc aucun plafond ne s'applique

```json managed-settings.json theme={null}
{
  "requiredMaximumVersion": "2.1.150"
}
```

Les mises à jour automatiques en arrière-plan et `claude update` sautent les versions au-dessus du plafond, si bien qu'une installation dans la plage y reste. `claude update`, `claude install` et `claude doctor` continuent de fonctionner au-dessus du plafond pour que les utilisateurs puissent récupérer. Associez-la à [`requiredMinimumVersion`](#requiredminimumversion) pour imposer une plage.

### `requiredMinimumVersion`

Définit la version la plus ancienne de Claude Code que votre organisation autorise à démarrer. Quand la version en cours est plus ancienne, Claude Code quitte au démarrage et dit à l'utilisateur de mettre à jour par la méthode approuvée de votre organisation. La vérification n'a lieu qu'au démarrage : une session déjà en cours continue. (v2.1.163+)

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code ne donne aucun avertissement quand il ignore la clé ailleurs.
* **Type** : chaîne, un numéro de version comme `"2.1.150"` ; une valeur qui n'est pas une version valide est ignorée
* **Défaut** : non défini, donc aucun plancher ne s'applique

```json managed-settings.json theme={null}
{
  "requiredMinimumVersion": "2.1.150"
}
```

`claude update`, `claude install` et `claude doctor` continuent de fonctionner sous le plancher pour que les utilisateurs puissent récupérer. Contrairement à [`minimumVersion`](#minimumversion), qui empêche seulement les rétrogradations, cette clé bloque le démarrage. Associez-la à [`requiredMaximumVersion`](#requiredmaximumversion) pour imposer une plage.

## Outils

Désactivent des outils précis dans l'[application desktop Claude Code](https://code.claude.com/docs/en/desktop). La CLI en terminal ignore ces clés. Pour les outils eux-mêmes, voir [Tools available to Claude](https://code.claude.com/docs/en/tools-reference).

### `browserExternalPageTools`

Empêche Claude d'utiliser ses outils pour lire des pages externes ou agir dessus dans le [panneau Browser](https://code.claude.com/docs/en/desktop#browse-external-sites) de l'application desktop. Les personnes de votre organisation peuvent toujours ouvrir elles-mêmes des sites externes, et les aperçus de serveurs de développement locaux continuent de fonctionner avec les outils de Claude. L'application desktop lit cette clé ; la CLI en terminal l'ignore.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, `"disabled"` ; l'application desktop accepte aussi `"disable"`, quelle que soit la casse
* **Défaut** : non défini, donc les outils de Claude fonctionnent sur les pages externes

```json managed-settings.json theme={null}
{
  "browserExternalPageTools": "disabled"
}
```

Toute autre valeur laisse les outils de Claude actifs, et une chaîne non vide qui n'est pas l'une des deux valeurs acceptées journalise un avertissement. Pour bloquer les sites externes aussi bien pour les personnes que pour Claude, définissez plutôt [`disableBrowserExternalNavigation`](#disablebrowserexternalnavigation). Voir [Restrict external browsing for your organization](https://code.claude.com/docs/en/desktop#restrict-external-browsing-for-your-organization).

### `disableBrowserExternalNavigation`

Désactive la navigation externe dans le [panneau Browser](https://code.claude.com/docs/en/desktop#browse-external-sites) de l'application desktop, pour les personnes comme pour Claude. Les aperçus de serveurs de développement en localhost continuent de fonctionner. L'application desktop lit cette clé ; la CLI en terminal l'ignore.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen ; seul le booléen JSON `true` a un effet
  * `true` : l'application desktop désactive la navigation externe dans le panneau Browser pour les personnes comme pour Claude ; les aperçus localhost continuent de fonctionner
  * `false` : la navigation externe reste active
* **Défaut** : non défini, donc la navigation externe est active

```json managed-settings.json theme={null}
{
  "disableBrowserExternalNavigation": true
}
```

L'application desktop ignore toute autre valeur, et une valeur non booléenne, comme la chaîne `"true"` ou `1`, journalise aussi un avertissement. Pour laisser la navigation externe active tout en tenant les outils de Claude à l'écart des pages externes, définissez plutôt [`browserExternalPageTools`](#browserexternalpagetools). Voir [Restrict external browsing for your organization](https://code.claude.com/docs/en/desktop#restrict-external-browsing-for-your-organization).

### `disableMobileSimulatorTools`

Bloque les outils de Claude pour le [panneau iOS Simulator](https://code.claude.com/docs/en/desktop-ios-simulator#turn-off-simulator-access) de l'application desktop. Les personnes gardent l'usage manuel du panneau ; seul l'accès de Claude est retiré, et personne ne peut le réactiver depuis l'application. L'application desktop lit cette clé ; la CLI en terminal l'ignore.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen ; seul le booléen JSON `true` a un effet
  * `true` : l'application desktop bloque les outils de Claude pour le panneau iOS Simulator
  * `false` : les outils de simulateur de Claude suivent la bascule de réglages de chaque personne dans l'application desktop
* **Défaut** : non défini, donc les outils de simulateur de Claude suivent la bascule de réglages de chaque personne dans l'application desktop

```json managed-settings.json theme={null}
{
  "disableMobileSimulatorTools": true
}
```

L'application desktop ignore toute autre valeur, et une valeur non booléenne, comme la chaîne `"true"` ou `1`, journalise aussi un avertissement.

## Confidentialité et télémétrie

Contrôlent combien de temps Claude Code conserve les données de session et ce qu'il envoie. Les interrupteurs qui désactivent les métriques d'usage et les rapports d'erreurs sont des variables d'environnement, pas des clés de réglages : définissez `DISABLE_TELEMETRY`, `DISABLE_ERROR_REPORTING` ou `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` dans la clé [`env`](#env) ou dans le shell. [Telemetry services](https://code.claude.com/docs/en/data-usage#telemetry-services) indique ce que chacune arrête. Deux exceptions se désactivent depuis un fichier de réglages : [`feedbackDrafts`](#feedbackdrafts) ci-dessous pour le feedback rédigé par Claude, et [`feedbackSurveyRate`](#feedbacksurveyrate) ci-dessous pour le sondage de session.

### `cleanupPeriodDays`

Définit pendant combien de jours Claude Code conserve les [transcriptions de session et autres données d'application](https://code.claude.com/docs/en/claude-directory#cleaned-up-automatically) avant de les supprimer. Claude Code effectue la suppression par un balayage en arrière-plan après le démarrage d'une session, dès lors qu'il peut déterminer la durée de rétention en toute sécurité. Le balayage supprime les transcriptions sans afficher de message : une session inutilisée depuis plus longtemps que la durée de rétention n'apparaît donc plus dans le sélecteur [`/resume`](https://code.claude.com/docs/en/sessions#resume-a-session).

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : nombre de jours, entier, minimum `1`
* **Défaut** : `30`

```json settings.json theme={null}
{
  "cleanupPeriodDays": 20
}
```

La valeur `0` échoue à la validation ; choisissez une grande valeur comme `3650` pour une longue rétention. Pour empêcher totalement Claude Code d'écrire des transcriptions, voir [Plaintext storage](https://code.claude.com/docs/en/claude-directory#plaintext-storage).

### `desktopSessionCleanupPeriodDays`

Définit une limite d'âge en jours pour les transcriptions des sessions que vous avez démarrées ou reprises en dernier dans Claude Desktop ou Cowork. Sans cette clé, Claude Code [conserve ces transcriptions quel que soit leur âge](https://code.claude.com/docs/en/claude-directory#cleaned-up-automatically). Claude Code supprime chacune dès qu'elle est plus ancienne à la fois que cette limite et que [`cleanupPeriodDays`](#cleanupperioddays) : avec `cleanupPeriodDays` à sa valeur par défaut de 30, une valeur de `7` les conserve donc quand même 30 jours. [Cleaned up automatically](https://code.claude.com/docs/en/claude-directory#cleaned-up-automatically) liste les cas où `cleanupPeriodDays` s'applique à la place et où Claude Code ignore cette clé. (v2.1.248+)

* **Portée** : [`User or managed`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code lit aussi la clé depuis un fichier passé avec `--settings`, et l'ignore dans les réglages de projet et locaux.
* **Type** : nombre de jours, entier, minimum `0`
* **Défaut** : `0`, qui ne fixe aucune limite d'âge

```json settings.json theme={null}
{
  "desktopSessionCleanupPeriodDays": 90
}
```

### `feedbackDrafts`

Contrôle le [feedback rédigé par Claude](https://code.claude.com/docs/en/tools-reference#sendfeedback-tool-behavior) : si Claude peut mettre en file d'attente des brouillons de feedback que vous relirez, et si Claude Code affiche une carte quand Claude en met un en file.

* **Portée** : [`User or managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, parmi `"notify"`, `"quiet"` ou `"off"`
  * `"notify"` : Claude Code affiche une carte au-dessus du prompt quand Claude met un brouillon en file, jusqu'à [trois cartes par session](https://code.claude.com/docs/en/tools-reference#what-you-see-when-claude-drafts) par défaut
  * `"quiet"` : Claude rédige sans carte. Vous voyez le nombre de brouillons en file dans le pied du prompt et les relisez dans `/feedback`
  * `"off"` : Claude Code retire l'outil SendFeedback, donc Claude ne peut pas mettre de brouillons en file
* **Défaut** : `"notify"`
* **Surcharges par session** : [`CLAUDE_CODE_SEND_FEEDBACK`](https://code.claude.com/docs/en/env-vars) défini à `0` désactive la fonctionnalité pour une session

```json settings.json theme={null}
{
  "feedbackDrafts": "quiet"
}
```

Apparaît dans `/config` sous **Claude-drafted feedback**, qui écrit cette clé dans vos réglages utilisateur. Vous ne voyez la ligne `/config` que dans les sessions [où Claude peut rédiger du feedback](https://code.claude.com/docs/en/tools-reference#sessions-without-claude-drafted-feedback) ; la valeur `"off"` ne la masque pas, si bien que vous pouvez réactiver la fonctionnalité depuis la même ligne. Une valeur dans les réglages gérés l'emporte sur votre réglage utilisateur : quand un administrateur définit cette clé, la ligne affiche la valeur gérée et la modifier n'a aucun effet. Claude Code ignore cette clé dans les réglages de projet et locaux.

### `feedbackSurveyRate`

Définit la probabilité que le [sondage de qualité de session](https://code.claude.com/docs/en/data-usage#session-quality-surveys) apparaisse quand une session y est éligible. Définissez `0` pour que le sondage n'apparaisse jamais.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : nombre entre `0` et `1`
* **Défaut** : non défini, donc Claude Code utilise le taux fixé à distance par Anthropic, ou son taux intégré de `0.005` sur Amazon Bedrock, l'Agent Platform de Google Cloud et Microsoft Foundry, qui ne reçoivent pas de configuration à distance
* **Surcharges par session** : [`CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY`](https://code.claude.com/docs/en/env-vars) défini à `1` désactive le sondage pour une session, quel que soit le taux défini par cette clé

```json settings.json theme={null}
{
  "feedbackSurveyRate": 0.05
}
```

Le même taux s'applique au sondage dans l'extension VS Code.

### `skipWebFetchPreflight`

Saute la [vérification de sécurité de domaine de WebFetch](https://code.claude.com/docs/en/data-usage#webfetch-domain-safety-check), qui envoie chaque nom d'hôte demandé à `api.anthropic.com` avant la récupération. Définissez `true` dans les environnements qui bloquent le trafic vers Anthropic, comme les déploiements Amazon Bedrock, Agent Platform de Google Cloud ou Microsoft Foundry avec une sortie réseau restrictive.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code saute la vérification de sécurité de domaine de WebFetch
  * `false` : la vérification a lieu avant la première récupération vers chaque nom d'hôte dans une session, et à nouveau pour un nom d'hôte dont la vérification précédente a été bloquée ou a échoué
* **Défaut** : non défini, donc la vérification a lieu avant la première récupération vers chaque nom d'hôte dans une session

```json settings.json theme={null}
{
  "skipWebFetchPreflight": true
}
```

Avec la vérification sautée, WebFetch tente n'importe quelle URL sans consulter la liste de blocage ; associez-la donc à des [règles de permission `WebFetch`](https://code.claude.com/docs/en/permissions#webfetch) si vous devez restreindre les domaines que Claude peut atteindre.

## Entreprise et réglages gérés

Clés qu'une organisation utilise pour calculer, rafraîchir et combiner les réglages gérés. Voir [Set up managed settings](https://code.claude.com/docs/en/admin-setup).

### `disableSideloadFlags`

Rejette au démarrage les flags CLI `--plugin-dir`, `--plugin-url`, `--agents` et `--mcp-config`, que les utilisateurs pourraient sinon passer pour contourner [`strictKnownMarketplaces`](#strictknownmarketplaces) le temps d'une exécution. Claude Code quitte avec une erreur qui nomme les flags rejetés. Dans les [sessions cloud](https://code.claude.com/docs/en/claude-code-on-the-web), Claude Code démarre plutôt la session et abandonne chaque entrée `--mcp-config` fournie par le serveur, sauf les entrées in-process `type: "sdk"` et les outils Slack d'une session [Claude Tag](https://code.claude.com/docs/en/claude-tag). (v2.1.193+)

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code rejette `--plugin-dir`, `--plugin-url`, `--agents` et `--mcp-config` au démarrage et quitte avec une erreur qui les nomme. Dans les sessions cloud, il démarre plutôt la session et abandonne chaque entrée `--mcp-config` fournie par le serveur, sauf les entrées in-process `type: "sdk"` et les outils Slack d'une session Claude Tag
  * `false` : Claude Code accepte ces flags
* **Défaut** : `false`

```json managed-settings.json theme={null}
{
  "disableSideloadFlags": true
}
```

Claude Code accepte toujours un `--mcp-config` dont tous les serveurs sont des entrées in-process `type: "sdk"`, pour que l'Agent SDK et l'extension VS Code continuent de fonctionner. Les utilisateurs peuvent toujours ajouter des serveurs avec `claude mcp add` ou un fichier `.mcp.json` ; pour un contrôle par serveur, définissez aussi [`allowedMcpServers`](https://code.claude.com/docs/en/managed-mcp). (v2.1.193+)

La même vérification couvre les dossiers de plugins nommés dans la variable d'environnement [`CLAUDE_CODE_PLUGIN_DIRS`](https://code.claude.com/docs/en/env-vars#variables), ce qui requiert la v2.1.280+. Quand la variable nomme un dossier, Claude Code quitte avec la même erreur, et l'erreur indique de supprimer (unset) la variable.

Dans les sessions cloud, Claude Code ignore aussi les mises à jour MCP en cours de session fournies par le serveur, le mécanisme derrière la configuration des sessions cloud et les appels SDK `setMcpServers()` qui atteignent ces sessions. Les entrées in-process `type: "sdk"` et les outils Slack d'une session Claude Tag y restent aussi exemptés. Avant la v2.1.268, cet abandon comme celui du démarrage retiraient aussi les outils Slack d'une session Claude Tag. Avant la v2.1.239, un `--mcp-config` fourni par le serveur empêchait une session cloud de démarrer.

L'application desktop gère elle-même certains plugins, y compris les plugins synchronisés depuis claude.ai et ceux que votre organisation déploie via l'application. Si vous déployez cette clé sur un appareil via MDM, politique de niveau OS ou fichier de réglages gérés, l'application desktop ne transmet pas ces plugins aux sessions suivantes sur cet appareil :

* **[Sessions Code sur la machine de l'utilisateur](https://code.claude.com/docs/en/desktop#environment-configuration)** : elles démarrent aussi sans les skills activés pour le compte claude.ai de l'utilisateur. Les plugins que Claude Code installe depuis les marketplaces de vos réglages gérés se chargent toujours. Dans [Claude Desktop on 3P](https://claude.com/docs/third-party/claude-desktop/overview), les serveurs MCP des plugins que vous déployez dans le dossier `org-plugins` de l'appareil restent aussi disponibles, car l'application desktop s'y connecte elle-même. Avant Claude Desktop v1.37937.0, ces sessions échouaient plutôt au démarrage.
* **[Sessions Cowork sur la machine de l'utilisateur](https://code.claude.com/docs/en/managed-settings#where-and-when-a-policy-applies)** : les skills contenus dans ces plugins et les skills activés pour le compte claude.ai de l'utilisateur restent disponibles. Dans [Claude Desktop on 3P](https://claude.com/docs/third-party/claude-desktop/overview), les serveurs MCP des plugins que vous déployez dans le dossier `org-plugins` de l'appareil restent aussi disponibles, car l'application desktop s'y connecte elle-même. Avant Claude Desktop v1.44121.0, ces sessions échouaient plutôt au démarrage.

### `forceRemoteSettingsRefresh`

Bloque le démarrage de la CLI jusqu'à ce que Claude Code ait récupéré à neuf les [réglages gérés par serveur](https://code.claude.com/docs/en/server-managed-settings). Si la récupération échoue, Claude Code quitte au lieu de continuer avec des réglages en cache ou sans réglages. Définissez-la quand votre environnement ne peut accepter même une courte fenêtre pendant laquelle une session tourne sans sa politique gérée.

Quand la clé n'est pas définie, Claude Code ne bloque pas le démarrage sur la récupération ; toutefois, quand le développeur se connecte au démarrage, il attend jusqu'à cinq secondes cette récupération. Une session via Cloud gateway attend toujours, et quitte si la passerelle est injoignable.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code honore un `true` venant de n'importe quelle source gérée contrôlée par l'administrateur, même si ce n'est pas la source de plus haute priorité.
* **Type** : booléen
  * `true` : Claude Code bloque le démarrage jusqu'à avoir récupéré à neuf les réglages gérés par serveur, et quitte si la récupération échoue
  * `false` : Claude Code ne bloque pas le démarrage sur la récupération, mais lors d'un démarrage avec connexion il attend jusqu'à cinq secondes la récupération
* **Défaut** : `false`

```json managed-settings.json theme={null}
{
  "forceRemoteSettingsRefresh": true
}
```

Définissez-la dans un profil MDM ou le fichier de réglages gérés pour imposer un démarrage en échec fermé (fail-closed) avant l'arrivée de la première charge utile du serveur. Claude Code n'applique la vérification que dans les sessions qui récupèrent les réglages gérés par serveur : une session qui [ne les récupère pas](https://code.claude.com/docs/en/server-managed-settings#platform-availability) démarre sans attendre. Les sous-commandes `claude auth` sont exemptées, pour que les utilisateurs puissent se réauthentifier quand des identifiants expirés sont la cause de l'échec. Voir [Enforce fail-closed startup](https://code.claude.com/docs/en/server-managed-settings#enforce-fail-closed-startup).

### `managedSourcesBehavior`

Choisit si Claude Code applique seulement la [source gérée](https://code.claude.com/docs/en/managed-settings#how-claude-code-combines-managed-sources) de plus haute priorité que votre organisation fournit, ou combine toutes les sources d'administration qu'elle fournit. Par défaut, Claude Code prend la source de plus haute priorité qui porte une [clé de politique](https://code.claude.com/docs/en/managed-settings#how-claude-code-combines-managed-sources) et ignore les autres. Une clé de politique est toute clé de réglages autre que celle-ci et `wslInheritsWindowsSettings`. Avec ce comportement par défaut, dès que les réglages gérés par serveur ou une politique MDM fournissent une clé de politique, un fichier `managed-settings.json` ne contribue que les [clés que Claude Code lit dans toutes les sources d'administration](https://code.claude.com/docs/en/managed-settings#keys-read-from-every-admin-source). Avec `"merge"`, chaque source d'administration que vous fournissez contribue ses clés à une politique combinée unique. (v2.1.242+)

> **Avertissement de sécurité** : ne définissez `"merge"` que là où chaque source [classée](https://code.claude.com/docs/en/managed-settings#how-claude-code-combines-managed-sources) sous votre source la plus haute est sous le contrôle d'un administrateur, car Claude Code ajoute alors à la politique les entrées d'une source inférieure, comme des règles `permissions.allow`.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code lit cette clé depuis la source de plus haute priorité qui porte soit cette clé soit une clé de politique, et l'ignore dans toutes les sources de rang inférieur : une source inférieure ne peut donc pas s'inscrire elle-même pour se combiner avec la source au-dessus. Ni le registre Windows HKCU ni les [réglages parents d'un hôte embarquant](https://code.claude.com/docs/en/managed-settings#let-an-embedding-host-add-policy) ne participent à la fusion.
* **Type** : chaîne, parmi :
  * `"first-wins"` : la source de plus haute priorité qui porte une clé de politique fournit la politique, et les sources inférieures ne contribuent que les [clés que Claude Code lit dans toutes les sources d'administration](https://code.claude.com/docs/en/managed-settings#keys-read-from-every-admin-source)
  * `"merge"` : chaque source d'administration que vous fournissez contribue ses clés, combinées selon les règles ci-dessous
* **Défaut** : `"first-wins"`

Fournissez la clé dans la source de plus haute priorité que vous déployez. Une machine qui ne reçoit jamais de réglages gérés par serveur a aussi besoin de la clé dans son profil MDM, car Claude Code lit la clé depuis la source de plus haute priorité qui la porte ou qui porte une clé de politique. Un fichier `managed-settings.json` est la source d'administration de plus bas rang, donc un `"merge"` défini là n'a aucune source inférieure avec laquelle se combiner. Dans les réglages gérés par serveur, la clé se présente ainsi :

```json theme={null}
{
  "managedSourcesBehavior": "merge"
}
```

Avec `"merge"`, Claude Code combine chaque clé selon son type. Ce tableau donne la règle pour chaque type. Les lignes « listes d'autorisation restrictives », « valeurs prises en entier » et « source la plus haute uniquement » nomment toutes les clés qu'elles couvrent ; les autres lignes donnent des exemples :

| Type de clé | Comment Claude Code la combine | Clés |
| :- | :- | :- |
| Listes | Combine les entrées de toutes les sources | [`permissions.allow`](#permissions-allow), [`sandbox.network.allowedDomains`](#sandbox-network-alloweddomains) et les autres clés de type liste |
| Verrous | Applique la valeur la plus stricte définie par une source. Quand aucune source ne définit de valeur stricte, n'applique une valeur plus souple que depuis la source la plus haute | [`allowManagedPermissionRulesOnly`](#allowmanagedpermissionrulesonly), [`permissions.disableBypassPermissionsMode`](#permissions-disablebypasspermissionsmode) et les autres verrous booléens ou énumérés |
| Listes d'autorisation restrictives | Prend la liste en entier depuis la source la plus haute qui la définit, sans ajouter d'entrées des sources inférieures. Quand la source la plus haute n'en définit pas, la prend en entier depuis la source suivante | [`availableModels`](#availablemodels), [`allowedMcpServers`](#allowedmcpservers), [`allowedProviders`](#allowedproviders), [`strictKnownMarketplaces`](#strictknownmarketplaces), [`allowedChannelPlugins`](#allowedchannelplugins) et la chaîne [`fallbackModel`](#fallbackmodel) |
| Valeurs prises en entier | Prend la valeur en entier depuis la source la plus haute qui la définit, sans combiner d'entrées ni de champs des sources inférieures. Quand la source la plus haute ne la définit pas, la prend en entier depuis la source suivante | [`sandbox.credentials.awsPairs`](#sandbox-credentials-awspairs), [`sandbox.ripgrep`](#sandbox-ripgrep) |
| Serveurs MCP fournis | Combine les noms de serveurs de toutes les sources. Quand deux sources définissent le même nom, applique l'entrée entière de la source la plus haute | [`managedMcpServers`](#managedmcpservers) |
| Lue depuis la source de plus haute priorité uniquement | Ne lit la clé que depuis la source de plus haute priorité qui porte une clé de politique, si bien que la valeur d'une source inférieure est ignorée même quand la source la plus haute n'en définit aucune | [`apiKeyHelper`](#apikeyhelper), [`awsAuthRefresh`](#awsauthrefresh), [`awsCredentialExport`](#awscredentialexport), [`gcpAuthRefresh`](#gcpauthrefresh), [`otelHeadersHelper`](#otelheadershelper), `proxyAuthHelper`, [`forceLoginOrgUUID`](#forceloginorguuid), les valeurs `"claudeai"` et `"console"` de [`forceLoginMethod`](#forceloginmethod), [`parentSettingsBehavior`](#parentsettingsbehavior), [`modelPicker`](#modelpicker), [`policyHelper`](#policyhelper), [`permissions.defaultMode`](#permissions-defaultmode) |
| `env` | [Fusionne variable par variable entre sources d'administration](https://code.claude.com/docs/en/managed-settings#keys-read-from-every-admin-source), aussi bien avec `"first-wins"` qu'avec `"merge"` | [`env`](#env) |
| Toute autre clé | Prend la valeur de la source la plus haute qui la définit | [`cleanupPeriodDays`](#cleanupperioddays), [`model`](#model) |

Prendre `sandbox.credentials.awsPairs` et `sandbox.ripgrep` en entier requiert la v2.1.257+.

Quelques clés ajoutent une condition que le tableau ne montre pas :

* **[`policyHelper`](#policyhelper)** : Claude Code ne l'honore que si la source la plus haute qui porte une clé de politique est une politique MDM ou un fichier de réglages gérés ; sous des réglages gérés par serveur, elle ne s'applique donc pas.
* **[`modelOverrides`](#modeloverrides)** : va de pair avec `availableModels`. Claude Code prend `modelOverrides` depuis la source la plus haute qui la définit, sauf si une source plus haute définit `availableModels` sans `modelOverrides`. Dans ce cas, il ignore `modelOverrides` de toutes les sources.
* **[`forceLoginGatewayUrl`](#forcelogingatewayurl), [`gatewayInternalNetworks`](#gatewayinternalnetworks) et la valeur `"gateway"` de [`forceLoginMethod`](#forceloginmethod)** : Claude Code ne lit jamais aucune d'elles depuis les réglages gérés par serveur ; une valeur là-bas ne s'applique donc pas et ne masque pas non plus une valeur définie dans une politique MDM ou un fichier de réglages gérés. Parmi les sources d'administration de la machine, seule celle de plus haut rang qui porte une clé de politique les fournit, que des réglages gérés par serveur soient présents ou non.
* **[`allowedProviders`](#allowedproviders)** : après la règle du tableau, la propre liste de la machine limite encore le résultat, comme l'indique la note Portée de son entrée.

Pour confirmer quelles sources se sont combinées sur une machine, lancez `/status` et [lisez la ligne `Setting sources`](https://code.claude.com/docs/en/managed-settings#read-the-source-in-/status).

### `parentSettingsBehavior`

Choisit si Claude Code applique les réglages gérés fournis par un processus hôte embarquant (embedding host), comme l'Agent SDK ou une extension d'IDE, quand un niveau géré déployé par un administrateur est aussi présent. Avec `"first-wins"`, Claude Code abandonne les réglages fournis par l'hôte ; avec `"merge"`, il les applique sous le niveau administrateur à travers un filtre qui ne garde que les valeurs restrictives. Choisissez `"merge"` quand un hôte doit transmettre ses propres restrictions aux sessions qu'il lance, par exemple Claude Desktop fournissant l'allowlist de sortie réseau d'une passerelle.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code la lit depuis la source gérée contrôlée par l'administrateur de plus haute priorité.
* **Type** : chaîne, parmi :
  * `"first-wins"` : Claude Code abandonne les réglages fournis par l'hôte quand un niveau géré déployé par un administrateur est présent
  * `"merge"` : Claude Code applique les réglages fournis par l'hôte sous le niveau administrateur à travers un filtre ne gardant que les valeurs restrictives
* **Défaut** : `"first-wins"`

```json managed-settings.json theme={null}
{
  "parentSettingsBehavior": "merge"
}
```

Cette clé n'a aucun effet quand aucun niveau géré déployé par un administrateur n'existe : les réglages de l'hôte s'appliquent alors comme seul niveau géré, toujours filtrés aux valeurs restrictives. Pour les limites du filtre et l'interaction des sources gérées, voir [Parent settings from embedding hosts](https://code.claude.com/docs/en/managed-settings#parent-settings-from-embedding-hosts) et [Restrict parent settings](https://code.claude.com/docs/en/claude-apps-gateway#restrict-parent-settings).

### `policyHelper`

Exécute un exécutable que vous déployez et qui calcule les réglages gérés au démarrage, pour dériver la politique de l'état de l'appareil (device posture), de l'identité ou d'un service distant au lieu d'un fichier statique. Claude Code exécute le helper avant d'accepter le premier prompt et traite les réglages qu'il émet comme les réglages gérés de la session.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes). Lue depuis le plist macOS, le registre Windows HKLM ou le fichier de réglages gérés. Claude Code lit la clé depuis la source gérée de plus haute priorité qui porte une [clé de politique](https://code.claude.com/docs/en/managed-settings#how-claude-code-combines-managed-sources) et n'exécute le helper que si cette source est l'une de ces trois ; il ignore la clé dans les réglages gérés par serveur, le registre HKCU et les réglages parents fournis par un hôte.
* **Type** : objet avec `path`, `timeoutMs` et `refreshIntervalMs`
* **Défaut** : non défini, donc aucun helper ne s'exécute

Quand les réglages gérés par serveur fournissent la politique au lancement, ils l'emportent sur la source du helper et le helper ne s'exécute pas.

Si une récupération ultérieure des réglages signale que les réglages gérés par serveur ont été supprimés, Claude Code exécute le helper à ce moment-là plutôt que d'attendre le lancement suivant. Sa sortie régit le reste de la session, et une exécution qui échoue termine la session avec le même message qu'une [exécution de démarrage échouée](#échecs-du-helper).

Cet exemple exécute le helper avec un délai d'expiration de 5 secondes et le relance toutes les cinq minutes :

```json managed-settings.json theme={null}
{
  "policyHelper": {
    "path": "/usr/local/bin/claude-policy",
    "timeoutMs": 5000,
    "refreshIntervalMs": 300000
  }
}
```

#### Écrire la sortie du helper

Claude Code exécute le helper sans argument, définit `CLAUDE_CODE_VERSION` dans son environnement, et lit sur stdout une enveloppe JSON, plafonnée à 1 Mio.

Placez les réglages sous une clé `managedSettings`. Un objet de réglages nu sans clé `managedSettings` est analysé avec `managedSettings` indéfini et n'applique rien, et Claude Code ne signale aucune erreur :

```json theme={null}
{
  "managedSettings": {
    "permissions": { "deny": ["Read(//etc/secrets/**)"] }
  }
}
```

Quand le helper émet `managedSettings`, cet objet devient la seule source de réglages gérés pour l'exécution : Claude Code ignore les sources MDM, fichier et HKCU, lit les [clés inter-sources](https://code.claude.com/docs/en/managed-settings#keys-read-from-every-admin-source) uniquement depuis la sortie du helper, et ne fusionne jamais les [réglages parents](https://code.claude.com/docs/en/managed-settings#parent-settings-from-embedding-hosts).

La vérification `forceRemoteSettingsRefresh` de démarrage s'exécute avant le helper et lit n'importe quelle source d'administration. Un helper qui sort avec `0` et une enveloppe sans `managedSettings` ne contribue aucun réglage géré, et les autres sources s'appliquent normalement.

#### Échecs du helper

Une exécution du helper échoue quand :

* `path` enfreint les règles de [`policyHelper.path`](#policyhelperpath).
* Aucun fichier régulier ne se trouve à `path`. Claude Code vérifie la présence du fichier avant de lancer le helper, dans le même budget `timeoutMs` ; un montage réseau qui ne répond pas peut donc faire échouer l'exécution.
* Le helper sort avec un code non nul, tourne encore quand `timeoutMs` est écoulé, ou ne démarre pas du tout, par exemple parce qu'il n'est pas exécutable.
* Le helper écrit plus de 1 Mio sur stdout ou sur stderr.
* stdout n'est pas un unique objet JSON, ou son `managedSettings` présente une [violation de schéma que Claude Code ne peut pas réparer](https://code.claude.com/docs/en/managed-settings#find-entries-claude-code-dropped).

Quand l'exécution de démarrage échoue, Claude Code affiche la raison et refuse de démarrer. Après une sortie non nulle, la raison inclut le stderr du helper, ou son stdout si stderr est vide. Après un dépassement de délai, la raison nomme la limite `timeoutMs` et n'inclut aucune sortie du helper. Le refus couvre les sessions interactives, `claude -p`, les sessions Agent SDK, les [sessions en arrière-plan](https://code.claude.com/docs/en/agent-view) et la plupart des sous-commandes.

Le refus est délibéré : un helper qui doit résister aux pannes doit servir depuis son propre cache et sortir avec `0`.

Quand un rafraîchissement en arrière-plan échoue, Claude Code garde en vigueur la dernière politique réussie, et `/status` affiche le rafraîchissement en échec avec sa raison jusqu'à ce qu'un rafraîchissement réussisse. Chaque rafraîchissement s'exécute avec le même `timeoutMs` et les mêmes règles d'échec que l'exécution de démarrage.

Avec `--debug`, Claude Code écrit le stderr du helper de chaque exécution dans le [journal de débogage](https://code.claude.com/docs/en/debug-your-config).

Claude Code signale une valeur `policyHelper` invalide comme une [entrée abandonnée](https://code.claude.com/docs/en/managed-settings#find-entries-claude-code-dropped) et démarre la session sur les réglages gérés restants sans exécuter de helper. Les valeurs invalides incluent une simple chaîne de chemin et un `timeoutMs` inférieur à [son minimum](#policyhelpertimeoutms).

Pour désactiver un helper, supprimez la clé de la source qui la définit.

### `policyHelper.path`

Nomme l'exécutable helper que Claude Code exécute. Pour ce qui se passe quand le chemin enfreint les règles ci-dessous, voir [Échecs du helper](#échecs-du-helper).

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes). Lue depuis le plist macOS, le registre Windows HKLM ou le fichier de réglages gérés, là où [`policyHelper`](#policyhelper) est lue.
* **Type** : chaîne, un chemin absolu sous forme normalisée, sans segments `.` ou `..` ; sous Windows, un chemin à lettre de lecteur ou UNC se terminant par `.exe`
* **Défaut** : aucun ; obligatoire quand `policyHelper` est définie

```json managed-settings.json theme={null}
{
  "policyHelper": {
    "path": "/usr/local/bin/claude-policy"
  }
}
```

### `policyHelper.timeoutMs`

Définit combien de temps Claude Code attend le helper avant de considérer l'exécution comme échouée. Une exécution qui dépasse le délai échoue comme une sortie non nulle : au démarrage, Claude Code refuse donc de démarrer.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes). Lue depuis le plist macOS, le registre Windows HKLM ou le fichier de réglages gérés, là où [`policyHelper`](#policyhelper) est lue.
* **Type** : entier, en millisecondes, minimum `1000`
* **Défaut** : `10000`

```json managed-settings.json theme={null}
{
  "policyHelper": {
    "path": "/usr/local/bin/claude-policy",
    "timeoutMs": 5000
  }
}
```

### `policyHelper.refreshIntervalMs`

Fait relancer le helper par Claude Code en arrière-plan à intervalle régulier, pour que les changements de politique atteignent une session en cours. Quand un rafraîchissement réussit, sa sortie remplace les réglages gérés précédents sans redémarrage ; quand il échoue, Claude Code garde la politique qu'il a déjà.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes). Lue depuis le plist macOS, le registre Windows HKLM ou le fichier de réglages gérés, là où [`policyHelper`](#policyhelper) est lue.
* **Type** : entier, en millisecondes : `0` pour désactiver le rafraîchissement, sinon au moins `60000`
* **Défaut** : non défini, donc Claude Code n'exécute le helper qu'une fois au démarrage

Cet exemple relance le helper toutes les cinq minutes :

```json managed-settings.json theme={null}
{
  "policyHelper": {
    "path": "/usr/local/bin/claude-policy",
    "refreshIntervalMs": 300000
  }
}
```

### `wslInheritsWindowsSettings`

Fait lire à Claude Code sous WSL les réglages gérés depuis la chaîne de politique Windows, HKLM et le fichier de réglages gérés Windows ayant priorité sur `/etc/claude-code`, et HKCU venant en dessous. Tant que la chaîne est active, Claude Code ne lit `/etc/claude-code` que lorsqu'[aucun document d'administration Windows n'est présent](https://code.claude.com/docs/en/managed-settings#present-admin-documents) dans la valeur de registre HKLM ou le dossier `C:\Program Files\ClaudeCode\`. Définissez-la pour étendre aux sessions WSL de la même machine la politique que vous déployez déjà sous Windows, afin qu'elles suivent les mêmes règles que les sessions de l'hôte. Claude Code ne l'honore que si elle est définie dans la clé de registre HKLM ou dans un fichier de réglages gérés ou un fichier drop-in sous `C:\Program Files\ClaudeCode\`, deux emplacements dont l'écriture requiert les droits administrateur Windows.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes). Dans une source Windows contrôlée par l'administrateur.
* **Type** : booléen
  * `true` : Claude Code sous WSL lit les réglages gérés depuis la chaîne de politique Windows, et ne lit `/etc/claude-code` que lorsqu'aucun document d'administration Windows n'est présent
  * `false` : WSL ne lit que `/etc/claude-code`
* **Défaut** : `false`, donc WSL ne lit que `/etc/claude-code`

```json managed-settings.json theme={null}
{
  "wslInheritsWindowsSettings": true
}
```

Une fois qu'une source d'administration active la chaîne, la politique HKCU ne la rejoint sous WSL que si HKCU définit aussi la clé à `true`. Cette copie n'active pas la chaîne à elle seule. Une source Windows qui ne contient que cette clé, à `true` ou `false`, ne compte pas comme source de politique ; une source de priorité inférieure fournit donc toujours la politique. Cette clé n'a aucun effet sous Windows natif.

Claude Code lit `true` et `false` avec ou sans guillemets, et lit `null` comme une suppression de la clé. Une source Windows contrôlée par l'administrateur qui contient toute autre valeur compte comme un [document d'administration présent](https://code.claude.com/docs/en/managed-settings#present-admin-documents) avec la chaîne activée : ni `/etc/claude-code` ni HKCU ne s'appliquent, et un avertissement au démarrage nomme la clé. Une valeur HKLM ou un fichier du dossier Windows qui existe mais ne peut pas être lu empêche aussi `/etc/claude-code` de s'appliquer, que la chaîne soit active ou non. (v2.1.282+)

## Réglages de configuration globale

Enregistrez ces clés dans `~/.claude.json`, pas dans un fichier de réglages. Claude Code les ignore partout ailleurs. Claude Code et `/config` écrivent la plupart d'entre elles pour vous, et vous pouvez aussi les modifier à la main.

### `autoConnectIde`

Se connecte automatiquement à un IDE en cours d'exécution quand vous démarrez Claude Code depuis un terminal externe. Apparaît dans `/config` sous **Auto-connect to IDE (external terminal)** quand vous exécutez Claude Code hors d'un terminal VS Code ou JetBrains.

* **Portée** : [`Global config`](https://code.claude.com/docs/en/settings-reference#scopes) (configuration globale)
* **Type** : booléen
  * `true` : Claude Code se connecte automatiquement à un IDE en cours d'exécution quand vous le démarrez depuis un terminal externe
  * `false` : Claude Code ne se connecte pas automatiquement depuis un terminal externe ; dans un terminal VS Code ou JetBrains, ou avec `--ide`, il se connecte quand même
* **Défaut** : `false`
* **Surcharges par session** : [`CLAUDE_CODE_AUTO_CONNECT_IDE`](https://code.claude.com/docs/en/env-vars) l'emporte sur cette clé pour une session, dans les deux sens

```json ~/.claude.json theme={null}
{
  "autoConnectIde": true
}
```

Claude Code ignore cette clé dans `settings.json`.

### `autoInstallIdeExtension`

Installe automatiquement l'extension IDE de Claude Code quand vous exécutez Claude Code depuis un terminal VS Code. Apparaît dans `/config` sous **Auto-install IDE extension** quand vous exécutez Claude Code dans un terminal VS Code ou JetBrains.

* **Portée** : [`Global config`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code installe automatiquement l'extension IDE quand vous l'exécutez depuis un terminal VS Code
  * `false` : Claude Code n'installe pas l'extension automatiquement
* **Défaut** : `true`
* **Surcharges par session** : [`CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL`](https://code.claude.com/docs/en/env-vars) défini à `1` saute l'installation pour une session même quand cette clé vaut `true`

```json ~/.claude.json theme={null}
{
  "autoInstallIdeExtension": false
}
```

Claude Code ignore cette clé dans `settings.json`.

### `claudeInChromeDefaultEnabled`

Démarre chaque session CLI interactive avec l'[intégration Chrome](https://code.claude.com/docs/en/chrome) activée, sans passer `--chrome` à chaque fois. Si vous exécutez [`claude remote-control`](https://code.claude.com/docs/en/remote-control), une session qu'il démarre pour l'un des fils de vos [projets](https://code.claude.com/docs/en/claude-projects) suit aussi cette clé, sauf en mode `bypassPermissions`. Depuis la v2.1.287, cette clé s'applique aussi aux sessions de l'[extension VS Code](https://code.claude.com/docs/en/vs-code#automate-browser-tasks-with-chrome) : voir [Enable Chrome by default](https://code.claude.com/docs/en/chrome#enable-chrome-by-default).

Exécuter `/chrome` et sélectionner **Enabled by default** définit cette clé pour vous. Apparaît dans `/config` sous **Claude in Chrome enabled by default**.

* **Portée** : [`Global config`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code active l'intégration Chrome au démarrage d'une session CLI interactive, comme lorsque vous passez `--chrome`. Dans l'extension VS Code, les sessions se connectent au navigateur dès leur démarrage
  * `false` : les sessions CLI interactives démarrent avec l'intégration Chrome désactivée, et Claude Code cesse de [proposer de la configurer](https://code.claude.com/docs/en/chrome#install-the-extension-when-claude-asks). Passez `--chrome` pour l'activer le temps d'une session interactive. Dans l'extension VS Code, une session se connecte quand vous tapez `@browser`, comme lorsque la clé n'est pas définie
* **Défaut** : non défini, donc l'intégration Chrome est désactivée et Claude Code peut encore proposer de la configurer
* **Surcharges par session** : `--chrome` et [`--no-chrome`](https://code.claude.com/docs/en/cli-reference) l'emportent sur cette clé pour une session interactive

```json ~/.claude.json theme={null}
{
  "claudeInChromeDefaultEnabled": true
}
```

Claude Code ignore cette clé dans `settings.json`.

### `copyFullResponse`

Fait que [`/copy`](https://code.claude.com/docs/en/commands) copie la réponse complète à chaque fois, sans afficher le sélecteur. Choisir **Always copy full response** dans ce sélecteur définit cette clé à `true`. Apparaît dans `/config` sous **Skip the /copy picker**.

* **Portée** : [`Global config`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : `/copy` copie la réponse complète sans afficher le sélecteur
  * `false` : quand la réponse contient des blocs de code ou des citations (blockquotes), `/copy` affiche un sélecteur où vous choisissez un bloc ou la réponse complète
* **Défaut** : `false`

```json ~/.claude.json theme={null}
{
  "copyFullResponse": true
}
```

Claude Code ignore cette clé dans `settings.json`.

### `copyOnSelect`

Copie automatiquement le texte dans votre presse-papiers quand vous finissez de le sélectionner à la souris en [rendu plein écran (fullscreen)](https://code.claude.com/docs/en/fullscreen#use-the-mouse) ou dans la [vue agents](https://code.claude.com/docs/en/agent-view). Apparaît dans `/config` sous **Copy on select** tant que le rendu plein écran est activé.

* **Portée** : [`Global config`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code copie le texte dans votre presse-papiers quand vous finissez de le sélectionner
  * `false` : sélectionner du texte laisse votre presse-papiers inchangé, et vous [copiez la sélection avec un raccourci clavier](https://code.claude.com/docs/en/fullscreen#use-the-mouse) à la place
* **Défaut** : `true`

```json ~/.claude.json theme={null}
{
  "copyOnSelect": false
}
```

Claude Code ignore cette clé dans `settings.json`.

### `defaultToAgentsView`

Ouvre la [vue agents](https://code.claude.com/docs/en/agent-view) au lieu d'une nouvelle conversation quand vous lancez `claude` sans argument. Apparaît dans `/config` sous **Open agents view by default**, sauf si la vue agents est [désactivée](#disableagentview).

* **Portée** : [`Global config`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : `claude` sans argument ouvre la vue agents, sauf si la vue agents est [désactivée](#disableagentview)
  * `false` : `claude` sans argument démarre une nouvelle conversation
* **Défaut** : `false`

```json ~/.claude.json theme={null}
{
  "defaultToAgentsView": true
}
```

Claude Code ignore cette clé dans `settings.json`.

### `diffTool`

Choisit où Claude Code affiche le diff d'une modification `Edit` ou `Write` qu'il propose lorsqu'un IDE [VS Code](https://code.claude.com/docs/en/vs-code) ou [JetBrains](https://code.claude.com/docs/en/jetbrains#features) est connecté : `"auto"` l'ouvre dans la visionneuse de diff de l'IDE, `"terminal"` le garde dans le terminal. Apparaît dans `/config` sous **Diff tool** uniquement tant que Claude Code est connecté à un IDE VS Code ou JetBrains.

* **Portée** : [`Global config`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, parmi :
  * `"auto"` : Claude Code ouvre le diff dans la visionneuse de diff de l'IDE quand un IDE VS Code ou JetBrains est connecté
  * `"terminal"` : Claude Code garde le diff dans le terminal
* **Défaut** : `"auto"`

```json ~/.claude.json theme={null}
{
  "diffTool": "terminal"
}
```

Claude Code ignore cette clé dans `settings.json`.

### `externalEditorContext`

Quand vous appuyez sur `Ctrl+G`, Claude Code ouvre le prompt que vous êtes en train de taper dans votre [éditeur externe](https://code.claude.com/docs/en/interactive-mode#general-controls). Avec cette clé activée, le tampon de l'éditeur commence par la réponse précédente de Claude sous forme de lignes de commentaire `#`, pour que vous puissiez la lire en écrivant, et Claude Code retire ces lignes à l'enregistrement. Apparaît dans `/config` sous **Show last response in external editor**.

* **Portée** : [`Global config`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : le tampon de l'éditeur commence par la réponse précédente de Claude sous forme de lignes de commentaire `#`, que Claude Code retire à l'enregistrement
  * `false` : le tampon de l'éditeur s'ouvre avec seulement votre prompt
* **Défaut** : `false`

```json ~/.claude.json theme={null}
{
  "externalEditorContext": true
}
```

Avec la clé activée, le tampon que Claude Code ouvre ressemble à ceci, et seul le texte sous la ligne de marqueur est envoyé comme prompt (contenu laissé tel quel, en anglais, car c'est ce que Claude Code affiche) :

```text theme={null}
# ─── Claude's last response (for reference; removed on save) ───
# I added the retry loop to fetchUser in src/api.ts and a test
# for the timeout case. Want me to wire the same retry into
# fetchOrders?
# ─── Write your reply below this line ──────────────────────────

Yes, and cap it at three attempts.
```

Claude Code conserve les 50 dernières lignes de la réponse et marque la coupure avec `# … (earlier output truncated)`.

Claude Code ignore cette clé dans `settings.json`.

### `leftArrowOpensAgents`

Appuyer sur `←` sur un prompt vide [envoie la session en arrière-plan et ouvre la vue agents](https://code.claude.com/docs/en/agent-view#switch-sessions-without-leaving-the-terminal). Définissez cette clé à `false` pour désactiver ce raccourci. Apparaît dans `/config` sous **← opens agents** quand la vue agents est disponible.

* **Portée** : [`Global config`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : appuyer sur `←` sur un prompt vide, dans une session démarrée dans le terminal, l'envoie en arrière-plan et ouvre la vue agents
  * `false` : Claude Code désactive le raccourci ; dans une session à laquelle vous vous êtes [attaché depuis la vue agents](https://code.claude.com/docs/en/agent-view#attach-to-a-session), `←` sur un prompt vide détache toujours
* **Défaut** : `true`

```json ~/.claude.json theme={null}
{
  "leftArrowOpensAgents": false
}
```

Claude Code ignore cette clé dans `settings.json`.

### `permissionExplainerEnabled`

> **Avertissement** : supprimée en v2.1.257, en même temps que l'explication de commande par `Ctrl+E` sur les demandes de permission Bash et PowerShell. La définir n'a aucun effet sur les versions actuelles.

Jusqu'à la v2.1.256, vous pouviez appuyer sur `Ctrl+E` sur une demande de permission Bash ou PowerShell pour voir une explication de la commande générée par le modèle, et définir cette clé à `false` pour désactiver ce raccourci.

* **Portée** : [`Global config`](https://code.claude.com/docs/en/settings-reference#scopes). En v2.1.256 et antérieures.
* **Type** : booléen
* **Défaut** : `true`

### `prStatusFooterEnabled`

Affiche dans le pied du prompt un badge pour la pull request ou merge request ouverte de la branche courante, avec un soulignement coloré qui indique son [statut](https://code.claude.com/docs/en/interactive-mode#pr-review-status). Apparaît dans `/config` sous **Show PR status footer**.

* **Portée** : [`Global config`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : le pied affiche le badge dans les conditions décrites dans [PR review status](https://code.claude.com/docs/en/interactive-mode#pr-review-status)
  * `false` : Claude Code saute la vérification des pull requests et merge requests du pied et n'affiche pas ce badge. Une session à laquelle vous vous êtes [attaché depuis la vue agents](https://code.claude.com/docs/en/agent-view#attach-to-a-session) peut encore afficher un simple lien vers une pull request [qui lui est liée](https://code.claude.com/docs/en/agent-view#pull-request-status)
* **Défaut** : `true`

```json ~/.claude.json theme={null}
{
  "prStatusFooterEnabled": false
}
```

Claude Code ignore cette clé dans `settings.json`.

### `teammateDefaultModel`

> **Avertissement** : supprimée en v2.1.234, en même temps que sa ligne `/config` **Default teammate model**. La définir n'a aucun effet sur les versions actuelles.

Jusqu'à la v2.1.233, vous définissiez cette clé comme modèle des coéquipiers d'une [équipe d'agents](https://code.claude.com/docs/en/agent-teams#specify-teammates-and-models) pour lesquels votre prompt ne nommait pas de modèle : un alias comme `"sonnet"`, ou `null` pour suivre le modèle du chef d'équipe (lead). Pour le modèle que Claude Code choisit désormais pour ces coéquipiers, voir [specify teammates and models](https://code.claude.com/docs/en/agent-teams#specify-teammates-and-models).

* **Portée** : [`Global config`](https://code.claude.com/docs/en/settings-reference#scopes). En v2.1.233 et antérieures.
* **Type** : chaîne, un alias de modèle ou un ID de modèle complet, ou `null`
* **Défaut** : non défini

## Voir aussi

* [Configure permissions](https://code.claude.com/docs/en/permissions) : syntaxe des règles, modes de permission et confiance de l'espace de travail (workspace trust)
* [Environment variables](https://code.claude.com/docs/en/env-vars) : toutes les variables `CLAUDE_*`, `ANTHROPIC_*` et de fournisseurs que Claude Code lit
* [Tools available to Claude](https://code.claude.com/docs/en/tools-reference) : les outils intégrés et ceux qui nécessitent une approbation
* [Example settings files](https://code.claude.com/docs/en/settings-example) : un fichier personnel, un fichier d'équipe et le fichier géré d'une organisation
* [Set up managed settings](https://code.claude.com/docs/en/admin-setup) : comment les organisations décident de ce qu'elles imposent
* [Deploy managed settings](https://code.claude.com/docs/en/managed-settings) : mécanismes de distribution, priorité au sein du niveau géré, et entrées invalides dans les réglages gérés
* [Debug your configuration](https://code.claude.com/docs/en/debug-your-config) : `claude doctor` et la boîte de dialogue Settings Error
