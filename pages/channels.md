# Pousser des événements dans une session en cours avec les channels

> Source : https://code.claude.com/docs/en/channels · [← Index](../31-index-documentation-officielle.md)

*Utilisez les channels (canaux) pour pousser des messages, des alertes et des webhooks dans votre session Claude Code depuis un serveur MCP. Transférez des résultats de CI, des messages de chat et des événements de supervision pour que Claude puisse réagir pendant votre absence.*

> **Note** : les channels sont en [aperçu de recherche (research preview)](#aperçu-de-recherche-research-preview). Ils exigent une authentification Anthropic via claude.ai ou une clé API Console, et ne sont **pas disponibles** sur Amazon Bedrock, Google Cloud Agent Platform ni Microsoft Foundry. Les organisations Team et Enterprise doivent [les activer explicitement](#contrôles-entreprise).

Un channel est un serveur MCP qui pousse des événements dans votre session Claude Code en cours, afin que Claude puisse réagir à ce qui se passe pendant que vous n'êtes pas au terminal. Les channels peuvent être bidirectionnels : Claude lit l'événement et répond via le même channel, comme une passerelle de chat. Les événements n'arrivent que tant que la session est ouverte ; pour un fonctionnement permanent, exécutez Claude dans un processus d'arrière-plan ou un terminal persistant.

Contrairement aux intégrations qui lancent une nouvelle session cloud ou attendent d'être interrogées (polling), l'événement arrive dans la session que vous avez déjà ouverte : voir [comparaison des channels](#comparaison-des-channels).

On installe un channel sous forme de plugin et on le configure avec ses propres identifiants. Telegram, Discord et iMessage sont inclus dans l'aperçu de recherche.

Quand Claude répond via un channel, vous voyez le message entrant dans votre terminal, mais pas le texte de la réponse. Le terminal affiche l'appel d'outil et une confirmation (comme « sent »), et la réponse elle-même apparaît sur l'autre plateforme.

Si vous administrez une organisation Team, Enterprise ou Console, voir [Activer les channels pour votre organisation](#contrôles-entreprise). Pour construire votre propre channel, voir la [référence des channels](https://code.claude.com/docs/en/channels-reference).

## Channels pris en charge

Chaque channel pris en charge est un plugin qui nécessite [Bun](https://bun.sh). Pour une démonstration pratique du fonctionnement des plugins avant de connecter une vraie plateforme, essayez le [démarrage rapide fakechat](#démarrage-rapide).

### Telegram

Voir le [code source complet du plugin Telegram](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/telegram).

#### 1. Créer un bot Telegram

Ouvrez [BotFather](https://t.me/BotFather) dans Telegram et envoyez `/newbot`. Donnez-lui un nom d'affichage et un nom d'utilisateur unique se terminant par `bot`. Copiez le token renvoyé par BotFather.

#### 2. Installer le plugin

Démarrez Claude Code en lançant `claude` dans votre terminal, puis saisissez ceci à son invite :

```
/plugin install telegram@claude-plugins-official
```

Si l'installation échoue, repérez le message signalé par Claude Code :

* `Marketplace "claude-plugins-official" not found` : ajoutez la marketplace avec `/plugin marketplace add anthropics/claude-plugins-official`, puis relancez l'installation.
* Le plugin est [introuvable dans la marketplace](https://code.claude.com/docs/en/plugins/install#install-a-plugin) : vérifiez le nom du plugin.

Quand l'installation demande une portée (scope) d'installation, choisissez l'option de portée utilisateur (user scope) pour que le plugin soit disponible dans tous vos projets. Vérifiez le résumé d'installation : s'il indique `Run /reload-plugins to apply.`, voir [Appliquer les changements de plugins sans redémarrer](https://code.claude.com/docs/en/plugins/cli-reference#reload-plugins) pour rendre disponible la commande de configuration du plugin.

#### 3. Configurer votre token

Lancez la commande de configuration avec le token fourni par BotFather :

```
/telegram:configure <token>
```

Cela l'enregistre dans `~/.claude/channels/telegram/.env`. Vous pouvez aussi définir `TELEGRAM_BOT_TOKEN` dans l'environnement de votre shell avant de lancer Claude Code.

#### 4. Redémarrer avec les channels activés

Quittez Claude Code et redémarrez avec le flag de channel. Cela démarre le plugin Telegram, qui commence à interroger (polling) les messages adressés à votre bot :

```bash theme={null}
claude --channels plugin:telegram@claude-plugins-official
```

#### 5. Appairer votre compte

Ouvrez Telegram et envoyez n'importe quel message à votre bot. Le bot répond avec un code d'appairage.

> **Note** : si votre bot ne répond pas, vérifiez que Claude Code tourne avec `--channels` (étape précédente). Le bot ne peut répondre que lorsque le channel est actif.

De retour dans Claude Code, lancez :

```
/telegram:access pair <code>
```

Puis verrouillez l'accès pour que seul votre compte puisse envoyer des messages :

```
/telegram:access policy allowlist
```

### Discord

Voir le [code source complet du plugin Discord](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/discord).

#### 1. Créer un bot Discord

Allez sur le [Discord Developer Portal](https://discord.com/developers/applications), cliquez sur **New Application** et nommez-la. Dans la section **Bot**, créez un nom d'utilisateur, puis cliquez sur **Reset Token** et copiez le token.

#### 2. Activer Message Content Intent

Dans les réglages de votre bot, descendez jusqu'à **Privileged Gateway Intents** et activez **Message Content Intent**.

#### 3. Inviter le bot sur votre serveur

Allez dans **OAuth2 > URL Generator**. Sélectionnez la portée `bot` et activez ces permissions :

* View Channels
* Send Messages
* Send Messages in Threads
* Read Message History
* Attach Files
* Add Reactions

Ouvrez l'URL générée pour ajouter le bot à votre serveur.

#### 4. Installer le plugin

Démarrez Claude Code en lançant `claude` dans votre terminal, puis saisissez ceci à son invite :

```
/plugin install discord@claude-plugins-official
```

Si l'installation échoue, repérez le message signalé par Claude Code :

* `Marketplace "claude-plugins-official" not found` : ajoutez la marketplace avec `/plugin marketplace add anthropics/claude-plugins-official`, puis relancez l'installation.
* Le plugin est [introuvable dans la marketplace](https://code.claude.com/docs/en/plugins/install#install-a-plugin) : vérifiez le nom du plugin.

Quand l'installation demande une portée d'installation, choisissez l'option de portée utilisateur pour que le plugin soit disponible dans tous vos projets. Vérifiez le résumé d'installation : s'il indique `Run /reload-plugins to apply.`, voir [Appliquer les changements de plugins sans redémarrer](https://code.claude.com/docs/en/plugins/cli-reference#reload-plugins) pour rendre disponible la commande de configuration du plugin.

#### 5. Configurer votre token

Lancez la commande de configuration avec le token du bot que vous avez copié :

```
/discord:configure <token>
```

Cela l'enregistre dans `~/.claude/channels/discord/.env`. Vous pouvez aussi définir `DISCORD_BOT_TOKEN` dans l'environnement de votre shell avant de lancer Claude Code.

#### 6. Redémarrer avec les channels activés

Quittez Claude Code et redémarrez avec le flag de channel. Cela connecte le plugin Discord pour que votre bot puisse recevoir des messages et y répondre :

```bash theme={null}
claude --channels plugin:discord@claude-plugins-official
```

#### 7. Appairer votre compte

Envoyez un message privé (DM) à votre bot sur Discord. Le bot répond avec un code d'appairage.

> **Note** : si votre bot ne répond pas, vérifiez que Claude Code tourne avec `--channels` (étape précédente). Le bot ne peut répondre que lorsque le channel est actif.

De retour dans Claude Code, lancez :

```
/discord:access pair <code>
```

Puis verrouillez l'accès pour que seul votre compte puisse envoyer des messages :

```
/discord:access policy allowlist
```

### iMessage

Voir le [code source complet du plugin iMessage](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/imessage).

Le channel iMessage lit directement votre base de données Messages et envoie les réponses via AppleScript. Il nécessite **macOS** et ne demande ni token de bot ni service externe.

#### 1. Accorder l'accès complet au disque (Full Disk Access)

La base de données Messages située dans `~/Library/Messages/chat.db` est protégée par macOS. La première fois que le serveur la lit, macOS demande l'accès : cliquez sur **Allow**. L'invite nomme l'application qui a lancé Bun, par exemple Terminal, iTerm ou votre IDE.

Si l'invite n'apparaît pas ou si vous avez cliqué sur Don't Allow, accordez l'accès manuellement dans **System Settings > Privacy & Security > Full Disk Access** en y ajoutant votre terminal. Sans cela, le serveur s'arrête immédiatement avec `authorization denied`.

#### 2. Installer le plugin

Démarrez Claude Code en lançant `claude` dans votre terminal, puis saisissez ceci à son invite :

```
/plugin install imessage@claude-plugins-official
```

Si l'installation échoue, repérez le message signalé par Claude Code :

* `Marketplace "claude-plugins-official" not found` : ajoutez la marketplace avec `/plugin marketplace add anthropics/claude-plugins-official`, puis relancez l'installation.
* Le plugin est [introuvable dans la marketplace](https://code.claude.com/docs/en/plugins/install#install-a-plugin) : vérifiez le nom du plugin.

Quand l'installation demande une portée d'installation, choisissez l'option de portée utilisateur pour que le plugin soit disponible dans tous vos projets.

Si le résumé d'installation indique `Run /reload-plugins to apply.`, inutile d'agir ici : le redémarrage de l'étape suivante prend en compte le plugin.

#### 3. Redémarrer avec les channels activés

Quittez Claude Code et redémarrez avec le flag de channel :

```bash theme={null}
claude --channels plugin:imessage@claude-plugins-official
```

#### 4. Vous envoyer un message

Ouvrez Messages sur n'importe quel appareil connecté à votre identifiant Apple et envoyez-vous un message à vous-même. Il atteint Claude immédiatement : la conversation avec soi-même (self-chat) contourne le contrôle d'accès sans aucune configuration.

> **Note** : la première réponse envoyée par Claude déclenche une invite macOS Automation demandant si votre terminal peut contrôler Messages. Cliquez sur **OK**.

#### 5. Autoriser d'autres expéditeurs

Par défaut, seuls vos propres messages passent. Pour permettre à un autre contact de joindre Claude, ajoutez son identifiant (handle) :

```
/imessage:access allow +15551234567
```

Les handles sont des numéros de téléphone au format `+indicatif` ou des e-mails d'identifiant Apple comme `user@example.com`.

## Démarrage rapide

Fakechat est un channel de démonstration officiellement pris en charge qui fait tourner une interface de chat sur localhost, sans rien à authentifier ni service externe à configurer.

Une fois fakechat installé et activé, vous pouvez taper dans le navigateur et le message arrive dans votre session Claude Code. Claude répond, et la réponse s'affiche dans le navigateur. Après avoir testé l'interface fakechat, essayez [Telegram](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/telegram), [Discord](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/discord) ou [iMessage](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/imessage).

Pour essayer la démo fakechat, il vous faut :

* Claude Code [installé et authentifié](https://code.claude.com/docs/en/quickstart#step-1-install-claude-code) avec un compte claude.ai ou une clé API Claude Console ;
* [Bun](https://bun.sh) installé. Les plugins de channel préconstruits sont des scripts Bun. Vérifiez avec `bun --version` ; si cela échoue, [installez Bun](https://bun.sh/docs/installation) ;
* **Organisation Team, Enterprise ou Console gérée** : votre administrateur doit [activer les channels](#contrôles-entreprise) dans les réglages gérés (managed settings).

### 1. Installer le plugin de channel fakechat

Démarrez Claude Code en lançant `claude` dans votre terminal, puis saisissez la commande d'installation à son invite :

```text theme={null}
/plugin install fakechat@claude-plugins-official
```

Si l'installation échoue, repérez le message signalé par Claude Code :

* `Marketplace "claude-plugins-official" not found` : ajoutez la marketplace avec `/plugin marketplace add anthropics/claude-plugins-official`, puis relancez l'installation.
* Le plugin est [introuvable dans la marketplace](https://code.claude.com/docs/en/plugins/install#install-a-plugin) : vérifiez le nom du plugin.

Quand l'installation demande une portée d'installation, choisissez l'option de portée utilisateur pour que le plugin soit disponible dans tous vos projets.

Si le résumé d'installation indique `Run /reload-plugins to apply.`, inutile d'agir ici : le redémarrage de l'étape suivante prend en compte le plugin.

### 2. Redémarrer avec le channel activé

Quittez Claude Code, puis redémarrez avec `--channels` en passant le plugin fakechat installé :

```bash theme={null}
claude --channels plugin:fakechat@claude-plugins-official
```

Le serveur fakechat démarre automatiquement. L'écran de démarrage affiche un avis de channels indiquant que les messages provenant de `plugin:fakechat@claude-plugins-official` sont injectés directement dans cette session. Si le plugin n'est pas installé ou ne figure pas sur la liste d'autorisation (allowlist) approuvée, une ligne d'avertissement nommant le problème apparaît sous cet avis.

> **Astuce** : vous pouvez passer plusieurs plugins à `--channels`, séparés par des espaces.

### 3. Pousser un message

Ouvrez l'interface fakechat sur [http://localhost:8787](http://localhost:8787) et tapez un message :

```text theme={null}
what's in my working directory?
```

*(« Qu'y a-t-il dans mon répertoire de travail ? »)*

Le message arrive dans votre session Claude Code. Le terminal l'affiche comme une ligne de channel entrante, par exemple `← fakechat · web: what's in my working directory?`, tandis que le modèle le reçoit sous la forme d'un événement `<channel source="plugin:fakechat:fakechat">`, utilisant le nom de serveur scopé du plugin. Claude le lit, fait le travail et appelle l'outil `reply` de fakechat. Si Claude Code demande une permission pour la première réponse, approuvez-la. La réponse s'affiche dans l'interface de chat.

Si Claude rencontre une demande de permission pendant que vous êtes loin du terminal, la session se met en pause jusqu'à votre réponse. Les serveurs de channel qui déclarent la [capacité de relais des permissions](https://code.claude.com/docs/en/channels-reference#relay-permission-prompts) peuvent vous transférer ces demandes pour que vous approuviez ou refusiez à distance. Pour un usage sans surveillance, [`--dangerously-skip-permissions`](https://code.claude.com/docs/en/permission-modes#skip-all-checks-with-bypasspermissions-mode) contourne la plupart des demandes, mais ne l'utilisez que dans des environnements de confiance. Même dans ce cas, les [actions qu'aucun mode n'approuve automatiquement](https://code.claude.com/docs/en/permission-modes#actions-no-mode-auto-approves) restent soumises à approbation.

Quand vous exécutez des channels en mode non interactif avec `-p`, les outils nécessitant une saisie au terminal, comme les questions à choix multiples et l'approbation du mode plan, sont désactivés pour que la session ne reste jamais bloquée en attente d'une saisie.

## Sécurité

Chaque plugin de channel approuvé maintient une liste d'autorisation d'expéditeurs (sender allowlist) : seuls les identifiants que vous avez ajoutés peuvent pousser des messages, et tous les autres sont ignorés silencieusement.

Telegram et Discord initialisent la liste par appairage :

1. Trouvez votre bot dans Telegram ou Discord et envoyez-lui n'importe quel message.
2. Le bot répond avec un code d'appairage.
3. Dans votre session Claude Code, approuvez le code lorsque c'est demandé.
4. Votre identifiant d'expéditeur est ajouté à la liste d'autorisation.

iMessage fonctionne différemment : vous envoyer un message à vous-même contourne automatiquement le filtre, et vous ajoutez d'autres contacts par leur handle avec `/imessage:access allow`.

En plus de cela, vous contrôlez quels serveurs sont activés à chaque session avec `--channels`, et votre organisation contrôle la disponibilité avec [`channelsEnabled`](#contrôles-entreprise) sur les forfaits claude.ai Team et Enterprise ainsi que sur les organisations Console qui déploient des réglages gérés.

Figurer dans `.mcp.json` ne suffit pas pour pousser des messages : un serveur doit aussi être nommé dans `--channels`.

La liste d'autorisation conditionne aussi le [relais des permissions](https://code.claude.com/docs/en/channels-reference#relay-permission-prompts) si le channel le déclare. Toute personne pouvant répondre via le channel peut approuver ou refuser l'utilisation d'outils dans votre session : n'ajoutez à la liste que des expéditeurs auxquels vous confiez cette autorité.

## Contrôles entreprise

Les administrateurs contrôlent la disponibilité via deux [réglages gérés](https://code.claude.com/docs/en/settings) que les utilisateurs ne peuvent pas remplacer. La valeur par défaut dépend du mode d'authentification :

* **claude.ai Team et Enterprise** : les channels sont bloqués jusqu'à ce qu'un Owner (propriétaire) [les active](#activer-les-channels-pour-votre-organisation).
* **Anthropic Console avec authentification par clé API** : les channels sont autorisés par défaut. Ce réglage n'est nécessaire que si votre organisation déploie des réglages gérés.

Dans tous les cas, aucun channel ne s'exécute tant qu'un utilisateur ne l'a pas activé pour la session avec `--channels`.

| Réglage | Rôle | Si non configuré |
| :- | :- | :- |
| `channelsEnabled` | Interrupteur principal. Doit valoir `true` pour qu'un channel puisse délivrer des messages. Quand il est désactivé, bloque tous les channels, y compris le flag de développement. Voir [Activer les channels pour votre organisation](#activer-les-channels-pour-votre-organisation). | claude.ai Team et Enterprise : channels bloqués. Console : channels autorisés, sauf si votre organisation déploie des réglages gérés, auquel cas ils sont bloqués jusqu'à ce que cette clé soit définie |
| `allowedChannelPlugins` | Quels plugins peuvent s'enregistrer une fois les channels activés. Remplace la liste maintenue par Anthropic lorsqu'il est défini. | La liste par défaut d'Anthropic s'applique |

Les utilisateurs Pro et Max sans organisation ne sont pas soumis à ces vérifications : les channels sont disponibles et les utilisateurs les activent par session avec `--channels`.

### Activer les channels pour votre organisation

Activez les channels pour votre organisation depuis [**Organization settings > Claude Code > Channels**](https://claude.ai/admin-settings/claude-code), ce qui nécessite le rôle Owner, ou en définissant `channelsEnabled` à `true` dans les réglages gérés.

Une fois activé, les utilisateurs de votre organisation peuvent utiliser `--channels` pour activer des serveurs de channel dans des sessions individuelles. Si le réglage est désactivé ou non défini, le serveur MCP se connecte quand même et ses outils fonctionnent, mais les messages de channel n'arrivent pas. Un avertissement au démarrage indique à l'utilisateur de demander à un administrateur d'activer le réglage.

### Restreindre les plugins de channel autorisés

Par défaut, tout plugin figurant sur la liste d'autorisation maintenue par Anthropic peut s'enregistrer comme channel. Les administrateurs des forfaits Team et Enterprise peuvent remplacer cette liste par la leur en définissant `allowedChannelPlugins` dans les réglages gérés. Utilisez-le pour restreindre les plugins officiels autorisés, approuver des channels issus de votre propre marketplace interne, ou les deux. Chaque entrée nomme un plugin et la marketplace dont il provient :

```json theme={null}
{
  "channelsEnabled": true,
  "allowedChannelPlugins": [
    { "marketplace": "claude-plugins-official", "plugin": "telegram" },
    { "marketplace": "claude-plugins-official", "plugin": "discord" },
    { "marketplace": "acme-corp-plugins", "plugin": "internal-alerts" }
  ]
}
```

Si vous définissez un tableau vide, vous bloquez tous les plugins de channel de la liste d'autorisation, mais `--dangerously-load-development-channels` peut toujours contourner ce blocage pour des tests locaux. Pour bloquer entièrement les channels, flag de développement compris, laissez plutôt `channelsEnabled` non défini.

Ce réglage nécessite `channelsEnabled: true`. Si un utilisateur passe à `--channels` un plugin absent de votre liste, Claude Code démarre normalement mais le channel ne s'enregistre pas, et l'avis de démarrage explique que le plugin ne figure pas sur la liste approuvée de l'organisation. Avec le runtime client MCP v2, un channel peut aussi échouer à s'enregistrer parce que Claude Code [n'enregistre pas un serveur de channel qui négocie la révision de protocole 2026-07-28](https://code.claude.com/docs/en/mcp#push-messages-with-channels).

## Aperçu de recherche (research preview)

Les channels sont une fonctionnalité en aperçu de recherche. La disponibilité est déployée progressivement, et la syntaxe du flag `--channels` ainsi que le contrat de protocole peuvent changer en fonction des retours.

Ni `--channels` ni `--dangerously-load-development-channels` n'apparaissent dans `claude --help` tant que la fonctionnalité est en aperçu. Les flags fonctionnent bien qu'ils ne soient pas listés.

Pendant l'aperçu, `--channels` n'accepte que les plugins d'une liste d'autorisation maintenue par Anthropic, ou de la liste de votre organisation si un administrateur a défini [`allowedChannelPlugins`](#restreindre-les-plugins-de-channel-autorisés). Les plugins de channel de [claude-plugins-official](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins) constituent l'ensemble approuvé par défaut. Si vous passez quelque chose qui ne figure pas dans la liste effective, Claude Code démarre normalement mais le channel ne s'enregistre pas, et l'avis de démarrage vous en indique la raison.

Pour tester un channel que vous construisez, passez-le à `--dangerously-load-development-channels` sous la forme `plugin:<name>@<marketplace>` ou `server:<name>`. Voir [Tester pendant l'aperçu de recherche](https://code.claude.com/docs/en/channels-reference#test-during-the-research-preview) pour tester les channels personnalisés que vous construisez.

Signalez problèmes et retours sur le [dépôt GitHub de Claude Code](https://github.com/anthropics/claude-code/issues).

## Comparaison des channels

Plusieurs fonctionnalités de Claude Code se connectent à des systèmes extérieurs au terminal, chacune adaptée à un type de travail différent :

| Fonctionnalité | Ce qu'elle fait | Idéale pour |
| - | - | - |
| [Sessions cloud](https://code.claude.com/docs/en/claude-code-on-the-web) | Exécute des tâches dans un nouveau bac à sable cloud, cloné depuis GitHub | Déléguer un travail asynchrone autonome que l'on vérifie plus tard |
| [Claude dans Slack](https://code.claude.com/docs/en/slack) | Lance une session cloud à partir d'une mention `@Claude` dans un canal ou un fil | Démarrer des tâches directement depuis le contexte d'une conversation d'équipe |
| [Serveur MCP](https://code.claude.com/docs/en/mcp) standard | Claude l'interroge pendant une tâche ; rien n'est poussé dans la session | Donner à Claude un accès à la demande pour lire ou interroger un système |
| [Remote Control](https://code.claude.com/docs/en/remote-control) | Vous pilotez votre session locale depuis claude.ai ou l'application mobile Claude | Diriger une session en cours quand vous êtes loin de votre bureau |

Les channels comblent le manque de cette liste en poussant des événements provenant de sources autres que Claude dans votre session locale déjà en cours.

* **Passerelle de chat** : posez une question à Claude depuis votre téléphone via Telegram, Discord ou iMessage, et la réponse revient dans le même chat pendant que le travail s'exécute sur votre machine, sur vos vrais fichiers.
* **[Récepteur de webhooks](https://code.claude.com/docs/en/channels-reference#example-build-a-webhook-receiver)** : un webhook provenant de la CI, de votre outil de suivi d'erreurs, d'un pipeline de déploiement ou d'un autre service externe arrive là où Claude a déjà vos fichiers ouverts et se souvient de ce que vous étiez en train de déboguer.

## Étapes suivantes

Une fois un channel en fonctionnement, explorez ces fonctionnalités liées :

* [Construire votre propre channel](https://code.claude.com/docs/en/channels-reference) pour les systèmes qui n'ont pas encore de plugin ;
* [Remote Control](https://code.claude.com/docs/en/remote-control) pour piloter une session locale depuis votre téléphone au lieu d'y transférer des événements ;
* [Tâches planifiées](https://code.claude.com/docs/en/scheduled-tasks) pour interroger à intervalles réguliers au lieu de réagir à des événements poussés.
