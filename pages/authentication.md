# Authentification

> Source : https://code.claude.com/docs/en/authentication · [← Index](../31-index-documentation-officielle.md)

Se connecter à Claude Code et configurer l'authentification pour les particuliers, les équipes et les organisations.

Claude Code prend en charge plusieurs méthodes d'authentification selon votre configuration. Les utilisateurs individuels peuvent se connecter avec un compte claude.ai, tandis que les équipes peuvent utiliser Claude for Teams ou Enterprise, la Claude Console, ou un fournisseur cloud comme Amazon Bedrock, Agent Platform de Google Cloud ou Microsoft Foundry.

## Se connecter à Claude Code

Après avoir [installé Claude Code](https://code.claude.com/docs/en/setup#install-claude-code), lancez `claude` dans votre terminal. Au premier lancement, Claude Code ouvre une fenêtre de navigateur pour que vous vous connectiez. Si vous avez défini la variable d'environnement `ANTHROPIC_API_KEY` et que vous approuvez la clé lorsque Claude Code vous demande s'il doit l'utiliser, Claude Code saute l'invite de connexion.

Si le navigateur ne s'ouvre pas automatiquement, appuyez sur `c` pour copier l'URL de connexion dans le presse-papiers, puis collez-la dans votre navigateur.

Si votre navigateur affiche un code de connexion au lieu de vous rediriger après la connexion, collez-le dans le terminal à l'invite `Paste code here if prompted`. Cela arrive quand le navigateur ne peut pas joindre le serveur de rappel (callback) local de Claude Code, ce qui est courant sous WSL2, en session SSH et dans les conteneurs.

Une fois la connexion terminée, le terminal affiche `Login successful` et vous invite à appuyer sur `Enter` pour continuer.

Vous pouvez vous authentifier avec l'un de ces types de compte :

* **Abonnement Claude Pro ou Max** : connectez-vous avec votre compte claude.ai. Abonnement sur [claude.com/pricing](https://claude.com/pricing?utm_source=claude_code\&utm_medium=docs\&utm_content=authentication_pro_max).
* **Claude for Teams ou Enterprise** : connectez-vous avec le compte claude.ai auquel l'administrateur de votre équipe vous a invité.
* **Claude Console** : connectez-vous avec vos identifiants Console. Votre administrateur doit d'abord [vous avoir invité](#authentification-claude-console). Vous pouvez vous connecter avec ou sans [création de clé API](#se-connecter-sans-clé-api).
* **Fournisseurs cloud** : si votre organisation utilise [Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock), [Agent Platform de Google Cloud](https://code.claude.com/docs/en/google-vertex-ai) ou [Microsoft Foundry](https://code.claude.com/docs/en/microsoft-foundry), définissez les variables d'environnement requises avant de lancer `claude`, ou sélectionnez **3rd-party platform** à l'invite de connexion, ce qui lance un assistant de configuration interactif pour Bedrock et Vertex AI. Aucune connexion par navigateur n'est nécessaire.
* **Passerelle cloud (Cloud gateway)** : si votre organisation exploite une [Claude apps gateway](https://code.claude.com/docs/en/claude-apps-gateway) auto-hébergée, connectez-vous via le SSO de l'entreprise avec `/login`. Le jeton émis par la passerelle est le seul identifiant de la session.

Les administrateurs peuvent imposer la méthode de connexion des développeurs et exiger que les connexions claude.ai appartiennent à une organisation donnée ; voir [Restreindre la connexion à votre organisation](#restreindre-la-connexion-à-votre-organisation).

Pour vous déconnecter et vous ré-authentifier, tapez `/logout` à l'invite de Claude Code. La déconnexion réinitialise aussi l'état de configuration du premier lancement : la prochaine fois que vous lancerez `claude`, il vous guidera à nouveau dans la connexion et la configuration.

En cas de problème de connexion, voir le [dépannage de l'authentification](https://code.claude.com/docs/en/troubleshoot-install#login-and-authentication).

### Se connecter avec plusieurs comptes

Pour rester connecté à plusieurs comptes à la fois (par exemple professionnel et personnel), donnez à chaque compte son propre répertoire de configuration. Au lancement de `claude`, définissez la variable d'environnement [`CLAUDE_CONFIG_DIR`](https://code.claude.com/docs/en/env-vars#variables) sur le répertoire du compte voulu. Chaque répertoire a ses propres réglages, son historique de sessions et sa connexion claude.ai ou sa clé API. Par exemple, sous Bash ou Zsh, ajoutez cet alias à `~/.bashrc` ou `~/.zshrc` pour que `claude-work` utilise votre compte professionnel tandis que `claude` garde le compte personnel :

```bash theme={null}
alias claude-work='CLAUDE_CONFIG_DIR=~/.claude-work claude'
```

Après avoir ouvert un nouveau terminal et lancé `claude-work` pour la première fois, Claude Code vous guide dans la connexion et la configuration pour ce nouveau répertoire. Des répertoires séparés ne maintiennent pas distinctes deux connexions Claude Console [sans clé API](#se-connecter-sans-clé-api), car Claude Code stocke ce type de connexion en dehors du répertoire de configuration.

## Configurer l'authentification d'équipe

Pour les équipes et organisations, vous pouvez configurer l'accès à Claude Code de l'une de ces façons :

* [Claude for Teams ou Enterprise](#claude-for-teams-ou-enterprise), recommandé pour la plupart des équipes
* [Claude Console](#authentification-claude-console)
* [Claude apps gateway](https://code.claude.com/docs/en/claude-apps-gateway), une passerelle auto-hébergée qui connecte les développeurs via votre IdP et achemine l'inférence vers le fournisseur cloud que vous configurez
* [Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock)
* [Agent Platform de Google Cloud](https://code.claude.com/docs/en/google-vertex-ai)
* [Microsoft Foundry](https://code.claude.com/docs/en/microsoft-foundry)

### Claude for Teams ou Enterprise

[Claude for Teams](https://claude.com/pricing?utm_source=claude_code\&utm_medium=docs\&utm_content=authentication_teams#team-&-enterprise) et [Claude for Enterprise](https://anthropic.com/contact-sales?utm_source=claude_code\&utm_medium=docs\&utm_content=authentication_enterprise) offrent la meilleure expérience aux organisations utilisant Claude Code. Les membres de l'équipe ont accès à la fois à Claude Code et à Claude sur le web, avec facturation centralisée et gestion d'équipe.

* **Claude for Teams** : forfait en libre-service avec fonctions de collaboration, outils d'administration, SSO, gestion de la facturation et [réglages gérés par le serveur](https://code.claude.com/docs/en/server-managed-settings) pour configurer Claude Code à l'échelle de l'organisation. Idéal pour les petites équipes.
* **Claude for Enterprise** : ajoute la capture de domaine (domain capture), les permissions basées sur les rôles et l'API de conformité (compliance API). Idéal pour les grandes organisations ayant des exigences de sécurité et de conformité.

Étapes :

1. **S'abonner** : abonnez-vous à [Claude for Teams](https://claude.com/pricing?utm_source=claude_code\&utm_medium=docs\&utm_content=authentication_teams_step#team-&-enterprise) ou contactez le service commercial pour [Claude for Enterprise](https://anthropic.com/contact-sales?utm_source=claude_code\&utm_medium=docs\&utm_content=authentication_enterprise_step).
2. **Inviter les membres de l'équipe** : invitez-les depuis le tableau de bord d'administration.
3. **Installer et se connecter** : les membres installent Claude Code et se connectent avec leur compte claude.ai.

### Authentification Claude Console

Pour les organisations qui préfèrent une facturation à l'usage de l'API, vous pouvez configurer l'accès via la Claude Console.

Étapes :

1. **Créer ou utiliser un compte Console** : utilisez votre compte Claude Console existant ou créez-en un.
2. **Ajouter des utilisateurs** : par l'une ou l'autre méthode :
   * Inviter des utilisateurs en masse depuis la page Members de la Console, [platform.claude.com/settings/members](https://platform.claude.com/settings/members) : cliquez sur **Invite**
   * [Configurer le SSO](https://support.claude.com/en/articles/13132885-setting-up-single-sign-on-sso)
3. **Attribuer des rôles** : lors de l'invitation, attribuez l'un des rôles suivants :
   * Rôle **Claude Code** : les utilisateurs peuvent uniquement créer des clés API Claude Code
   * Rôle **Developer** : les utilisateurs peuvent créer tout type de clé API
4. **Les utilisateurs terminent la configuration** : chaque utilisateur invité doit :
   * Accepter l'invitation Console
   * [Vérifier la configuration système requise](https://code.claude.com/docs/en/setup#system-requirements)
   * [Installer Claude Code](https://code.claude.com/docs/en/setup#install-claude-code)
   * Se connecter avec les identifiants de son compte Console

#### Se connecter sans clé API

Vous pouvez vous connecter à votre compte Console sans créer de clé API, même quand votre organisation n'autorise pas les développeurs à en créer. Choisissez le compte Anthropic Console à l'invite `/login` et Claude Code vous demande comment vous voulez vous connecter. Nécessite Claude Code v2.1.242+. Les deux options vous connectent à la Console dans le navigateur et diffèrent par ce que Claude Code stocke ensuite :

* **Sign in with your Console account** (« Se connecter avec votre compte Console »), marqué `(recommended)` : Claude Code conserve le jeton OAuth issu de cette connexion et le stocke comme [profil Anthropic](#profils-anthropic-et-identifiants-de-fédération). Aucune clé API n'est créée
* **Create an API key** (« Créer une clé API »), marqué `(legacy)` : Claude Code crée pour vous une clé API Console et la stocke avec vos autres identifiants

En pratique, le profil stocke une connexion OAuth alors qu'une clé API est un identifiant statique. Claude Code renouvelle automatiquement la connexion du profil ; si le renouvellement échoue, les requêtes échouent avec une [erreur de connexion de profil expirée](https://code.claude.com/docs/en/errors#anthropic-profile-login-expired) jusqu'à ce que vous vous reconnectiez.

Le choix n'est pas proposé sur toutes les machines. Claude Code crée une clé API sans demander dans ces cas :

* Vous utilisez un fournisseur cloud, comme [Amazon Bedrock, Agent Platform de Google Cloud ou Microsoft Foundry](https://code.claude.com/docs/en/third-party-integrations), ou [Claude Platform on AWS](https://code.claude.com/docs/en/claude-platform-on-aws)
* Un fichier de réglages définit [`forceLoginOrgUUID`](#restreindre-la-connexion-à-votre-organisation), ou définit `forceLoginMethod` à `"claudeai"` ou `"console"`
* Une source de réglages gérés sur votre machine (fichier de réglages gérés, profil MDM ou réglages gérés par le serveur en cache) existe mais Claude Code [ne peut pas la lire](https://code.claude.com/docs/en/managed-settings#invalid-entries-in-managed-settings), et aucune autre source gérée ne fournit de politique

Retirez (`unset`) `ANTHROPIC_API_KEY` avant de vous connecter sans clé.

Après une connexion sans clé, vous disposez d'un profil au lieu d'une clé API stockée :

* **Quel profil est écrit** : Claude Code écrit le profil nommé par `ANTHROPIC_PROFILE`, sinon votre profil actif, sinon `default`. Si ce profil est un profil de fédération, Claude Code refuse la connexion au lieu de l'écraser
* **De quoi cela vous déconnecte** : Claude Code vous déconnecte de toute connexion claude.ai stockée sur la machine
* **Comment annuler** : lancez `/logout`, qui supprime et révoque l'identifiant écrit par cette connexion

Tout le reste du fonctionnement des profils s'applique à cette connexion, notamment son rang par rapport à vos autres identifiants, la ligne `Profile` affichée dans `/status` et les fonctionnalités qui nécessitent une connexion claude.ai. Voir [Profils Anthropic et identifiants de fédération](#profils-anthropic-et-identifiants-de-fédération).

### Authentification via un fournisseur cloud

Pour les équipes utilisant Amazon Bedrock, Agent Platform de Google Cloud ou Microsoft Foundry :

1. **Suivre la configuration du fournisseur** : suivez la [documentation Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock), la [documentation Agent Platform de Google Cloud](https://code.claude.com/docs/en/google-vertex-ai) ou la [documentation Microsoft Foundry](https://code.claude.com/docs/en/microsoft-foundry).
2. **Distribuer la configuration** : distribuez à vos utilisateurs les variables d'environnement et les instructions pour générer des identifiants cloud. En savoir plus sur la [gestion de la configuration](https://code.claude.com/docs/en/settings).
3. **Installer Claude Code** : les utilisateurs peuvent [installer Claude Code](https://code.claude.com/docs/en/setup#install-claude-code).

### Restreindre la connexion à votre organisation

Pour exiger que les connexions claude.ai des développeurs appartiennent à une organisation Anthropic précise, définissez [`forceLoginMethod`](https://code.claude.com/docs/en/settings-reference#forceloginmethod) et [`forceLoginOrgUUID`](https://code.claude.com/docs/en/settings-reference#forceloginorguuid) dans les [réglages gérés](https://code.claude.com/docs/en/managed-settings). Définissez `forceLoginOrgUUID` sur l'identifiant de votre organisation, affiché dans les [réglages d'administration claude.ai](https://claude.ai/admin-settings/organization) pour les organisations Claude for Teams ou Enterprise. Claude Code signale une erreur pour toute connexion claude.ai à une autre organisation, et s'arrête au démarrage si l'identifiant claude.ai utilisé appartient à une organisation qui n'est pas listée.

Pour les connexions Claude Console, Claude Code utilise `forceLoginOrgUUID` pour présélectionner l'organisation sur la page de connexion Console lorsque vous le définissez sur un seul identifiant d'organisation Console, affiché sur [platform.claude.com/settings/organization](https://platform.claude.com/settings/organization). Il ne vérifie pas à quelle organisation appartient l'identifiant Console obtenu, ni à la connexion ni au démarrage. Un développeur connecté avec un compte Console avant le déploiement des clés reste connecté, et cette clé enregistrée est bloquée sur une machine qui exige aussi la connexion à la [passerelle](https://code.claude.com/docs/en/claude-apps-gateway) ou dans une session qui sélectionne un fournisseur cloud.

Si vous définissez `forceLoginOrgUUID` dans un fichier de réglages quelconque, Claude Code cesse de proposer la [connexion Console sans clé](#se-connecter-sans-clé-api) dans les sessions concernées par ce fichier et crée une clé API à la place. Pour orienter les développeurs vers la connexion claude.ai, définissez `forceLoginMethod` à `"claudeai"`.

Depuis Claude Code v2.1.212+, tous les chemins de connexion listés ici appliquent `forceLoginMethod`. Sur l'écran de connexion interactif du terminal (atteint via `/login` ou l'onboarding du premier lancement), Claude Code présélectionne une méthode `claudeai` ou `console` sans l'imposer : même avec `forceLoginMethod` à `"claudeai"`, un développeur peut encore y effectuer une connexion Console.

Les chemins diffèrent pour `forceLoginOrgUUID` :

* **Connexions depuis le terminal, l'[extension VS Code](https://code.claude.com/docs/en/vs-code) et l'Agent SDK** : vérifient `forceLoginOrgUUID` pour les connexions de compte claude.ai
* **`claude setup-token` et `/install-github-app`** : n'imposent que `forceLoginMethod`, et peuvent donc émettre un jeton dans une autre organisation
* **Connexion à la [passerelle](https://code.claude.com/docs/en/claude-apps-gateway)** : sélectionnée par `forceLoginMethod: "gateway"` plutôt que restreinte par lui, et ne s'authentifie pas auprès d'une organisation Anthropic, donc `forceLoginOrgUUID` ne s'applique pas ; utilisez le fournisseur d'identité de votre passerelle pour restreindre l'accès

Déployez les clés via vos outils de gestion des appareils. Les [réglages gérés par le serveur](https://code.claude.com/docs/en/server-managed-settings) n'atteignent que les comptes déjà authentifiés dans votre organisation : ils ne peuvent donc pas rediriger la première connexion d'un développeur. Si votre organisation distribue aussi des réglages gérés par le serveur, définissez les clés aux deux endroits : les sources de réglages gérés [ne fusionnent pas](https://code.claude.com/docs/en/server-managed-settings#settings-precedence), et les réglages gérés par le serveur en cache remplacent le fichier géré sur l'appareil, à part quelques [exceptions par clé](https://code.claude.com/docs/en/server-managed-settings#per-key-exceptions-across-managed-sources). `forceLoginOrgUUID` et les valeurs `"claudeai"` et `"console"` de `forceLoginMethod` ne font pas partie de ces exceptions : gardez-les aux deux endroits.

Dans un déploiement avec [passerelle](https://code.claude.com/docs/en/claude-apps-gateway), gardez aussi `forceLoginMethod` et `forceLoginOrgUUID` hors des [réglages servis par la passerelle](https://code.claude.com/docs/en/claude-apps-gateway-config#managed).

Ces clés déterminent aussi si une session qui n'utilise pas un identifiant de connexion peut démarrer. Voir [`forceLoginOrgUUID`](https://code.claude.com/docs/en/settings-reference#forceloginorguuid) dans la référence des réglages pour le comportement complet.

* **`ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN` ou `apiKeyHelper`** : bloqués au démarrage. Avec `forceLoginOrgUUID`, l'appartenance à l'organisation ne peut pas être vérifiée pour un identifiant d'environnement ; avec `forceLoginMethod`, l'identifiant se substituerait à la connexion exigée. Quand les réglages gérés exigent aussi la connexion à la [passerelle](https://code.claude.com/docs/en/claude-apps-gateway), Claude Code bloque de la même façon une clé API enregistrée par une connexion Claude Console antérieure. Voir [Administrator policy requires a Cloud gateway sign-in](https://code.claude.com/docs/en/errors#administrator-policy-requires-a-cloud-gateway-sign-in)
* **Sessions fournisseur cloud comme Amazon Bedrock** : bloquées seulement tant qu'un identifiant `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN` ou `apiKeyHelper`, ou une clé API enregistrée par une connexion Claude Console antérieure, est encore présent sur la machine. Supprimez-le et la session démarre. Ces sessions s'authentifient auprès de votre fournisseur cloud, dont les politiques d'accès les régissent
* **[Profil Anthropic ou identifiants de fédération](#profils-anthropic-et-identifiants-de-fédération)** : non bloqués, sauf si un identifiant `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN` ou `apiKeyHelper`, ou une clé API enregistrée par une connexion Claude Console antérieure, est aussi présent sur la machine. Les clés ne vérifient pas à quelle organisation appartient le profil

### Restreindre les fournisseurs d'API qu'une machine peut utiliser

[`allowedProviders`](https://code.claude.com/docs/en/settings-reference#allowedproviders), dans les [réglages gérés](https://code.claude.com/docs/en/managed-settings), liste les services par lesquels une machine gérée peut joindre Claude, comme l'API Anthropic, Amazon Bedrock ou une passerelle LLM. Il complète `forceLoginMethod` et `forceLoginOrgUUID`, qui régissent le compte qu'une session utilise lorsqu'elle parle à Anthropic. Nécessite Claude Code v2.1.285+.

```json managed-settings.json theme={null}
{
  "forceLoginMethod": "claudeai",
  "forceLoginOrgUUID": ["xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"],
  "allowedProviders": ["anthropic", "bedrock"]
}
```

Avec ce fichier, un développeur connecté à votre organisation claude.ai ou configuré pour Amazon Bedrock démarre normalement. Une session configurée pour tout autre fournisseur est refusée au démarrage, et une session en cours qui bascule vers un tel fournisseur est refusée à sa requête suivante. [Managed settings don't allow this API provider](https://code.claude.com/docs/en/errors#managed-settings-dont-allow-this-api-provider) présente chaque message.

* **Autoriser une passerelle ou un proxy LLM** : listez `"customEndpoint"` et définissez l'URL de la passerelle dans le bloc `env` géré de la même source. La [référence des réglages](https://code.claude.com/docs/en/settings-reference#allowedproviders) liste toutes les valeurs et indique quelles variables de point de terminaison doivent être fixées dans un `env` géré.
* **Déployer sur des machines gérées** : placez la liste dans la source gérée qui porte le reste de votre politique. La [note de portée (Scope)](https://code.claude.com/docs/en/settings-reference#allowedproviders) de l'entrée explique comment une liste gérée par le serveur se combine avec elle.
* **Réglages gérés par le serveur uniquement** : une liste définie seulement dans les [réglages gérés par le serveur](https://code.claude.com/docs/en/server-managed-settings) n'atteint que les sessions qui récupèrent les réglages de votre organisation ; considérez-la comme une commodité pour les machines que vous ne pouvez pas atteindre via la gestion des appareils, pas comme une mesure contraignante. [Platform availability](https://code.claude.com/docs/en/server-managed-settings#platform-availability) liste les sessions qui les récupèrent.

## Gestion des identifiants

Claude Code gère vos identifiants d'authentification de manière sécurisée :

* **Emplacement de stockage** :
  * Sur macOS, les identifiants sont stockés dans le Trousseau macOS chiffré (Keychain). Quand le Trousseau refuse l'écriture (par exemple s'il est verrouillé dans une session SSH), Claude Code stocke votre connexion dans `~/.claude/.credentials.json` avec le mode de fichier `0600`, le même stockage que sous Linux. Une connexion Console qui crée une clé API échoue tant que le Trousseau n'est pas accessible en écriture. Pour remettre votre connexion dans le Trousseau, suivez [les étapes de récupération](https://code.claude.com/docs/en/troubleshoot-install#not-logged-in-or-token-expired).
  * Sur Linux, les identifiants sont stockés dans `~/.claude/.credentials.json` avec le mode de fichier `0600`.
  * Sur Windows, les identifiants sont stockés dans `%USERPROFILE%\.claude\.credentials.json` et héritent des contrôles d'accès de votre répertoire de profil utilisateur, ce qui limite par défaut le fichier à votre compte utilisateur.
  * Si vous avez défini la variable d'environnement `CLAUDE_CONFIG_DIR`, Claude Code conserve le fichier `.credentials.json` dans ce répertoire (y compris le fichier écrit par le repli macOS) et associe aussi l'entrée du Trousseau macOS à ce répertoire : une session avec un autre `CLAUDE_CONFIG_DIR` lit une autre entrée.
  * Claude Code gère `.credentials.json` via `/login` et `/logout`. Pour acheminer les requêtes vers un point de terminaison d'API personnalisé, définissez plutôt la variable d'environnement [`ANTHROPIC_BASE_URL`](https://code.claude.com/docs/en/env-vars).
* **Types d'authentification pris en charge** : identifiants claude.ai, identifiants Claude API, Microsoft Foundry Auth, Bedrock Auth, Vertex Auth, profil Anthropic et identifiants [Workload Identity Federation](https://platform.claude.com/docs/en/manage-claude/workload-identity-federation), et jetons de session [Claude apps gateway](https://code.claude.com/docs/en/claude-apps-gateway).
* **Scripts d'identifiants personnalisés** : configurez le réglage [`apiKeyHelper`](https://code.claude.com/docs/en/settings-reference#apikeyhelper) pour exécuter un script shell qui renvoie une clé API.
* **Intervalles de renouvellement** : voir [`apiKeyHelper`](https://code.claude.com/docs/en/settings-reference#apikeyhelper) pour les cas où Claude Code ré-exécute le helper.
* **Avertissement de helper lent** : si `apiKeyHelper` met plus de 10 secondes à renvoyer une clé, Claude Code affiche un avertissement dans la barre d'invite avec le temps écoulé. Si vous voyez souvent cet avertissement, vérifiez si votre script d'identifiants peut être optimisé.
* **Échecs du helper** : quand le script se termine en erreur, dépasse le délai ou n'affiche rien, les requêtes échouent avec [`Your apiKeyHelper script is failing`](https://code.claude.com/docs/en/errors#your-apikeyhelper-script-is-failing) en trois tentatives au plus.

`apiKeyHelper`, `ANTHROPIC_API_KEY` et `ANTHROPIC_AUTH_TOKEN` s'appliquent à la CLI et aux surfaces qui l'encapsulent, notamment l'extension VS Code, l'Agent SDK et GitHub Actions. Claude Desktop et les sessions cloud n'appellent pas `apiKeyHelper` et ne lisent pas ces variables d'environnement : ils utilisent OAuth, sauf les sessions desktop qui exécutent une [configuration d'inférence tierce](https://code.claude.com/docs/en/llm-gateway-connect#desktop-app), qui s'authentifient avec l'identifiant de cette configuration.

### Renouveler une connexion qui expire

Quand la connexion créée avec `/login` expire dans moins de trois jours, Claude Code affiche un avertissement au démarrage : `Your login expires in 3 days · run /login to renew`.

Lancez `/login` pour renouveler. L'avertissement est informatif et ne bloque jamais une requête : l'authentification continue de fonctionner jusqu'à l'expiration effective.

Une fois la connexion stockée expirée et impossible à renouveler, chaque requête au modèle échoue avec [`Login expired · Please run /login`](https://code.claude.com/docs/en/errors#login-expired) jusqu'à ce que vous vous reconnectiez.

Vous pouvez détecter cet état avant qu'une requête échoue : [`/status`](https://code.claude.com/docs/en/commands) affiche une ligne `Login` indiquant `Expired — log in again`, ainsi que l'organisation et l'e-mail enregistrés pour la connexion expirée. La ligne n'apparaît que lorsque la connexion claude.ai enregistrée est l'identifiant actif. Cette ligne nécessite Claude Code v2.1.210+.

L'avertissement n'apparaît que lorsqu'une connexion claude.ai est l'identifiant actif, et pas quand l'identifiant provient d'un fournisseur cloud, de `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN` ou `apiKeyHelper`.

Renouveler tôt compte surtout pour les sessions sans surveillance. Une [session en arrière-plan dans l'agent view](https://code.claude.com/docs/en/agent-view) ou une session [Remote Control](https://code.claude.com/docs/en/remote-control) qui survit à la connexion cesse de progresser dès que l'identifiant expire et ne peut pas reprendre tant que vous ne vous reconnectez pas.

### Ordre de priorité de l'authentification

Quand plusieurs identifiants sont présents, Claude Code en choisit un dans cet ordre :

1. Identifiants du fournisseur cloud, quand `CLAUDE_CODE_USE_BEDROCK`, `CLAUDE_CODE_USE_VERTEX` ou `CLAUDE_CODE_USE_FOUNDRY` est défini. Voir [intégrations tierces](https://code.claude.com/docs/en/third-party-integrations) pour la configuration.
2. Variable d'environnement `ANTHROPIC_AUTH_TOKEN`. Envoyée dans l'en-tête `Authorization: Bearer`. À utiliser pour passer par une [passerelle ou un proxy LLM](https://code.claude.com/docs/en/llm-gateway) qui s'authentifie avec des jetons bearer plutôt qu'avec des clés API Anthropic.
3. Variable d'environnement `ANTHROPIC_API_KEY`. Envoyée dans l'en-tête `X-Api-Key`. À utiliser pour un accès direct à l'API Anthropic avec une clé de la [Claude Console](https://platform.claude.com). En mode interactif, on vous demande une fois d'approuver ou de refuser la clé, et votre choix est mémorisé. Pour le changer ensuite, utilisez le bouton « Use custom API key » dans `/config` ; ce bouton n'apparaît que tant que `ANTHROPIC_API_KEY` est défini dans votre environnement. En mode non interactif (`-p`), la clé est toujours utilisée lorsqu'elle est présente.
4. Sortie du script [`apiKeyHelper`](https://code.claude.com/docs/en/settings-reference#apikeyhelper). À utiliser pour des identifiants dynamiques ou tournants, comme des jetons de courte durée récupérés dans un coffre (vault).
5. Variable d'environnement `CLAUDE_CODE_OAUTH_TOKEN`. Un jeton OAuth longue durée généré par [`claude setup-token`](#générer-un-jeton-longue-durée). À utiliser pour les pipelines CI et les scripts où la connexion par navigateur n'est pas possible. Si vous lancez `/login` alors que la variable est définie, Claude Code bascule la session courante sur la nouvelle connexion, mais relit la variable à chaque nouvelle session tant que vous ne l'avez pas retirée de votre profil shell ou du bloc `env` d'un [fichier de réglages](https://code.claude.com/docs/en/settings).
6. Profil Anthropic et identifiants de fédération, c'est-à-dire les identifiants utilisés par la CLI `ant` et Workload Identity Federation. Un profil écrit par `ant auth login` ne se classe ici que si vous le nommez dans `ANTHROPIC_PROFILE` ; sinon il se classe sous `/login`. Voir [Profils Anthropic et identifiants de fédération](#profils-anthropic-et-identifiants-de-fédération).
7. Identifiants OAuth d'abonnement issus de `/login`. C'est le comportement par défaut pour les utilisateurs Claude Pro, Max, Team et Enterprise.

Une session [Claude apps gateway](https://code.claude.com/docs/en/claude-apps-gateway) connectée est hors de cette liste : c'est une sélection de fournisseur, comme Amazon Bedrock ou Agent Platform de Google Cloud, et elle prime sur eux. Quand une session de passerelle existe, la CLI s'authentifie avec le jeton de la passerelle même si `CLAUDE_CODE_USE_BEDROCK`, `CLAUDE_CODE_USE_VERTEX` ou `CLAUDE_CODE_USE_FOUNDRY` est défini, et les sources d'identifiants ci-dessus (jeton bearer, clé API, `apiKeyHelper`, profils) ne sont pas utilisées.

Si les [réglages gérés](https://code.claude.com/docs/en/managed-settings) de votre machine définissent [`forceLoginMethod`](https://code.claude.com/docs/en/settings-reference#forceloginmethod) à `"gateway"` ou définissent [`forceLoginGatewayUrl`](https://code.claude.com/docs/en/settings-reference#forcelogingatewayurl), et que vous ne sélectionnez pas de fournisseur cloud via une variable comme `CLAUDE_CODE_USE_BEDROCK` ou `CLAUDE_CODE_USE_VERTEX`, votre session n'utilise que la connexion à la passerelle. Claude Code ignore les autres sources d'identifiants et vous demande de vous connecter avec `/login`. Voir [Administrator policy requires a Cloud gateway sign-in](https://code.claude.com/docs/en/errors#administrator-policy-requires-a-cloud-gateway-sign-in) pour ce que vous voyez avec chaque identifiant restant. Nécessite Claude Code v2.1.261+, ou v2.1.265+ sur une machine qui définit uniquement `forceLoginGatewayUrl`.

Si vous avez un abonnement Claude actif mais aussi `ANTHROPIC_API_KEY` défini dans votre environnement, Claude Code utilise la clé API une fois que vous l'avez approuvée. Cela peut provoquer des échecs d'authentification si la clé appartient à une organisation désactivée ou expirée.

Lancez `unset ANTHROPIC_API_KEY` pour revenir à votre abonnement, et vérifiez avec `/status` quelle méthode est active. Quand une connexion et une clé API sont toutes deux configurées, `/status` signale l'identifiant non utilisé.

Les [sessions cloud](https://code.claude.com/docs/en/claude-code-on-the-web) utilisent toujours les identifiants de votre abonnement. Si vous définissez `ANTHROPIC_API_KEY` ou `ANTHROPIC_AUTH_TOKEN` dans l'environnement cloud, cela ne remplace pas les identifiants de votre abonnement.

#### Profils Anthropic et identifiants de fédération

Un profil est un fichier nommé de configuration d'identifiants situé dans votre [répertoire de configuration Anthropic](https://platform.claude.com/docs/en/manage-claude/wif-reference#configuration-directory), par défaut `~/.config/anthropic` sur macOS et Linux ou `%APPDATA%\Anthropic` sur Windows. Le mode d'authentification d'un profil est `oidc_federation` si vous l'avez configuré pour [Workload Identity Federation (WIF)](https://platform.claude.com/docs/en/manage-claude/workload-identity-federation), ou `user_oauth` s'il a été écrit par [`ant auth login`](https://platform.claude.com/docs/en/cli-sdks-libraries/cli/authentication) ou si vous vous êtes [connecté à un compte Console sans clé API](#se-connecter-sans-clé-api).

Claude Code ne lit pas les profils ni les variables de fédération en [mode bare](https://code.claude.com/docs/en/headless#start-faster-with-bare-mode), dans Claude Desktop ni dans les sessions cloud. Dans ces sessions, `/status` n'affiche pas de ligne `Profile`.

Claude Code examine trois sources dans cet ordre et s'arrête à la première qui est définie. Le tableau indique ce qui définit chaque source et son rang par rapport à votre identifiant `/login`.

| Source | Définie par | Rang par rapport à `/login` |
| :- | :- | :- |
| Profil nommé | `ANTHROPIC_PROFILE` | Au-dessus, quel que soit le mode d'authentification du profil |
| Variables de fédération | `ANTHROPIC_FEDERATION_RULE_ID` et `ANTHROPIC_ORGANIZATION_ID`, toutes deux définies | Au-dessus |
| Profil actif | Le [fichier `active_config`](https://platform.claude.com/docs/en/manage-claude/wif-reference#active-profile) de votre répertoire de configuration, ou un profil nommé `default` | Au-dessus si son mode est `oidc_federation` ; en dessous d'un identifiant `/login` fonctionnel si son mode est `user_oauth` |

Pour les variables de fédération, Claude Code lit aussi les autres variables de la [référence WIF](https://platform.claude.com/docs/en/manage-claude/wif-reference#environment-variables), comme `ANTHROPIC_IDENTITY_TOKEN_FILE`, lorsqu'il échange votre jeton d'identité. Pour le format du fichier de profil, voir la [référence WIF](https://platform.claude.com/docs/en/manage-claude/wif-reference#profile-configuration-file).

Pour confirmer la source choisie par Claude Code, lancez `/status`. Une ligne `Profile` nomme la source à la place de la ligne `Login method`. Quand le profil est l'identifiant utilisé, des lignes `Organization` et `Email` affichent son compte.

Quand la connexion d'un profil `user_oauth` a expiré et que Claude Code ne peut pas la renouveler, les requêtes échouent avec une [erreur de connexion de profil expirée](https://code.claude.com/docs/en/errors#anthropic-profile-login-expired).

Les fonctionnalités qui nécessitent votre connexion claude.ai, comme les [connecteurs claude.ai](https://code.claude.com/docs/en/mcp#use-mcp-servers-from-claude-ai) et [`/schedule`](https://code.claude.com/docs/en/routines), ne sont pas disponibles tant qu'une de ces sources est sélectionnée. Pour empêcher Claude Code de sélectionner une source :

* **Profil nommé ou variables de fédération** : retirez `ANTHROPIC_PROFILE`, ou retirez l'une des deux variables de fédération
* **Profil actif** : lancez `/logout` pour un profil `user_oauth` dont l'identifiant actuel a été écrit par une [connexion à un compte Console sans clé API](#se-connecter-sans-clé-api), lancez `ant auth logout` pour un profil dont l'identifiant actuel a été écrit par `ant auth login`, ou supprimez le fichier du profil dans `configs/` de votre répertoire de configuration (valable pour les deux modes d'authentification)

### Générer un jeton longue durée

Pour les pipelines CI, les scripts ou d'autres environnements où la connexion interactive par navigateur n'est pas disponible, générez un jeton OAuth valable un an avec `claude setup-token` :

```bash theme={null}
claude setup-token
```

La commande ouvre le même flux d'autorisation par navigateur que `/login`, et le jeton s'affiche dans le terminal après que vous avez approuvé l'accès dans le navigateur. Elle n'enregistre le jeton nulle part : copiez-le et définissez-le comme variable d'environnement `CLAUDE_CODE_OAUTH_TOKEN` là où vous voulez vous authentifier :

#### macOS, Linux, WSL

```bash theme={null}
export CLAUDE_CODE_OAUTH_TOKEN=your-token
```

#### Windows PowerShell

```powershell theme={null}
$env:CLAUDE_CODE_OAUTH_TOKEN = "your-token"
```

#### Windows CMD

```batch theme={null}
set CLAUDE_CODE_OAUTH_TOKEN=your-token
```

Ce jeton s'authentifie avec votre abonnement Claude et nécessite un forfait Pro, Max, Team ou Enterprise. Il ne peut faire que des requêtes au modèle : il ne peut donc pas établir de sessions [Remote Control](https://code.claude.com/docs/en/remote-control) ni récupérer les [connecteurs claude.ai](https://code.claude.com/docs/en/mcp#use-mcp-servers-from-claude-ai). Les serveurs MCP configurés localement fonctionnent toujours.

Le [mode bare](https://code.claude.com/docs/en/headless#start-faster-with-bare-mode) ne lit pas `CLAUDE_CODE_OAUTH_TOKEN`. Si votre script passe `--bare`, authentifiez-vous plutôt avec `ANTHROPIC_API_KEY` ou un `apiKeyHelper`.
