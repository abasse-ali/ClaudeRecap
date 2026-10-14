# Travailler avec Claude Tag

> Source : https://code.claude.com/docs/en/claude-tag · [← Index](../31-index-documentation-officielle.md)

*Claude Tag place Claude dans vos canaux Slack, avec un accès gouverné par les administrateurs. Ce qu'on peut lui confier, comment se passe la configuration, et par où commencer, en tant qu'admin ou en tant qu'utilisateur final.*

> **Note** : cette URL redirige vers la documentation Claude.ai, page canonique https://claude.com/docs/claude-tag/overview (titre « Work with Claude Tag », intitulé « Overview » dans la barre latérale, dernière modification le 2026-10-05). La page est surtout composée de HTML et de composants visuels ; tout son contenu textuel est repris ci-dessous. Les liens internes pointent vers `https://claude.com/docs/...`.

## Bandeau d'accueil

- Pastille : **Public Beta** (bêta publique).
- Titre : « Taguez `@Claude`. Récupérez les résultats dans le fil. » (*Tag @Claude in. Get results back in the thread.*)
- N'importe qui dans un canal Slack peut taguer Claude sur un problème et lui confier du travail : reproduire un bug et ouvrir une pull request, transformer un fil de décision en document, rassembler l'état d'un projet. Claude publie une checklist dans le fil au fur et à mesure, et tout l'échange reste visible pour le canal.
- Deux boutons :
  - « Je le configure → » (*I'm setting it up*) : mène à [Set up Claude Tag](https://claude.com/docs/claude-tag/admins/setup-overview).
  - « L'utiliser dans votre canal ↓ » (*Use it in your channel*) : mène à la section [Mettre Claude Tag au travail](#mettre-claude-tag-au-travail).

### Exemple de fil Slack (maquette)

Canal `# platform-eng` (38 membres) :

1. **Dana**, 14 h 14 : « le checkout est lent depuis ce matin, quelqu'un d'autre le voit ? »
2. **Leo**, 14 h 15 : « pareil. `@Claude` tu peux enquêter ? Compare la latence avec le déploiement de ce matin et trouve ce qui la cause. »
3. **Claude** (badge `APP`), 14 h 15 : « Je m'en occupe. Je compare la latence avant et après le déploiement, je trouve la cause, et je fais un retour ici. » La checklist qui accompagne la réponse :
   - Fait : latence p99 récupérée depuis Datadog.
   - Fait : comparaison (diff) du déploiement `4f2c1` avec `main`.
   - Fait : requête lente reproduite en local.
   - En cours : ouverture d'une pull request avec le correctif…

## Forfaits incluant Claude Tag

Claude Tag est disponible sur les forfaits **Team** et **Enterprise**, sur le service first-party d'Anthropic. Il n'est **pas** disponible sur les forfaits individuels (Free, Pro ou Max), ni pour les déploiements tiers (*third-party*). Pour l'utiliser, votre organisation associe (*pairs*) son espace de travail Slack à son organisation Claude. Les prérequis complets sont dans [la vue d'ensemble de la configuration](https://claude.com/docs/claude-tag/admins/setup-overview).

Si vous hésitez entre plusieurs produits Claude pour du travail qui se passe dans Slack, la page [comment Claude Tag diffère de Cowork et de Claude Code](https://claude.com/docs/claude-tag/concepts/how-it-works#how-claude-tag-differs-from-cowork-and-claude-code) les compare directement. En résumé, le travail d'équipe dans des canaux partagés relève de Claude Tag, et le travail personnel sur vos propres fichiers relève de Cowork ou de Claude Code.

### Qui a besoin d'un siège Claude

Les utilisateurs Slack n'ont pas besoin chacun d'un siège Claude pour travailler avec Claude dans les canaux.

- **Dans les canaux** : par défaut, n'importe qui dans l'espace de travail Slack associé peut taguer `@Claude` dans un canal. Un Owner peut [restreindre qui peut utiliser Claude](https://claude.com/docs/claude-tag/admins/restrict-access#restrict-who-can-use-claude) aux membres de votre organisation Claude ou, sur Enterprise, à des rôles précis. Le travail en canal est facturé à l'usage sur le solde d'utilisation (*usage balance*) de votre organisation, dans la limite d'une [limite de dépense](https://claude.com/docs/claude-tag/admins/set-spend-limit) fixée par un Owner.
- **Dans les DM de groupe** : la même restriction sur qui peut utiliser Claude s'applique, et le travail est facturé au solde d'utilisation de votre organisation. Voir [Group DMs](https://claude.com/docs/claude-tag/admins/restrict-access#group-dms).
- **Dans les DM individuels** (un à un) : le DM d'un membre qui a connecté un compte Claude s'exécute sur ce compte et est facturé au siège de cette personne. Ce siège doit inclure Claude Code ou, sur le forfait Enterprise, être un siège **Standard** ou **Usage-Based Chat** détenu par quelqu'un qui a aussi Cowork. Le [DM d'un membre qui n'a pas connecté de compte Claude](https://claude.com/docs/claude-tag/admins/restrict-access#direct-messages-from-members-without-a-claude-account) peut être facturé à votre organisation pendant une durée limitée.

## Où Claude Tag s'exécute

Claude Tag fonctionne dans Slack. Vous interagissez avec lui en écrivant dans un canal, un fil ou un message direct Slack, et il répond au même endroit. Dans un canal, mentionnez `@Claude` pour être sûr qu'il prenne le message en compte.

Quand Claude travaille sur une tâche, il s'exécute dans un **bac à sable éphémère**, pas sur votre ordinateur. Ce bac à sable est créé au début d'une conversation. Il contient le code ou les fichiers sur lesquels Claude travaille et il est détruit quand la conversation devient inactive. Le cycle de vie complet est décrit dans [How Claude Tag works](https://claude.com/docs/claude-tag/concepts/how-it-works).

Vous étendez ce que Claude peut atteindre (vos dépôts, systèmes de tickets, entrepôts de données, outils personnalisés…) avec les [connecteurs Claude Tag](https://claude.com/docs/claude-tag/admins/add-connections), les [plugins et les skills](https://claude.com/docs/claude-tag/admins/customize). Un Owner ou un [admin Claude Tag](https://claude.com/docs/claude-tag/admins/restrict-access#delegate-claude-tag-administration) les configure à l'un de ces trois niveaux :

- pour tout Slack ;
- pour un espace de travail ;
- pour un canal.

Les connecteurs claude.ai propres à chaque membre sont distincts de cette configuration. Claude peut les utiliser dans un canal pour les demandes de ce membre, comme l'explique [Personal connectors in channels](https://claude.com/docs/claude-tag/concepts/personal-connectors).

### Cartes d'orientation

**Pour les administrateurs : [Configurer Claude Tag](https://claude.com/docs/claude-tag/admins/setup-overview)**

| Question | Ce que la page couvre | Lien |
| :- | :- | :- |
| Par où commencer ? | Associer votre espace de travail Slack, lancer, et vérifier que tout fonctionne | [Set up Claude Tag](https://claude.com/docs/claude-tag/admins/setup-overview) |
| À quoi Claude Tag peut-il accéder ? | Comment les admins règlent l'accès par canal, et où les identifiants sont stockés | [How agent identity works](https://claude.com/docs/claude-tag/concepts/agent-identity) |
| Comment connecter chaque service ? | Types d'identifiants, hôtes autorisés, et ce que chaque connexion permet à Claude d'atteindre | [Give Claude access to your tools](https://claude.com/docs/claude-tag/admins/add-connections) |

**Pour les utilisateurs finaux : [Mettre Claude Tag au travail](#mettre-claude-tag-au-travail)**

| Question | Ce que la page couvre | Lien |
| :- | :- | :- |
| Comment confier une tâche à Claude Tag ? | Mentionner Claude dans n'importe quel canal où il se trouve, sans rien installer | [Get started](https://claude.com/docs/claude-tag/users/getting-started) |
| En quoi Claude Tag est-il bon ? | Cas d'usage : code, données, incidents, go-to-market | [Use case library](https://claude.com/docs/claude-tag/users/use-cases) |
| Comment obtenir de bons résultats ? | Bonnes habitudes pour cadrer et relire le travail | [Good habits](https://claude.com/docs/claude-tag/users/good-habits) |
| De quoi Claude Tag se souvient-il ? | Mémoire de canal, ce qui est partagé dans tout l'espace de travail, et qui voit quoi | [What Claude Tag remembers](https://claude.com/docs/claude-tag/users/memory) |
| Claude Tag peut-il exécuter des tâches planifiées ? | Tâches planifiées, surveillance de canal et déclencheurs | [Set up routines](https://claude.com/docs/claude-tag/users/proactivity) |

## Facturation et limites de dépense

Ajouter Claude à Slack n'ajoute **aucun coût par siège**. Le travail en canal, en fil et en DM de groupe est facturé à l'usage. Il est prélevé sur un **solde d'utilisation** (*usage balance*), un montant dans la devise de facturation de votre organisation, alimenté par un Owner. Une [limite de dépense](https://claude.com/docs/claude-tag/admins/set-spend-limit) (*spend limit*) plafonne la part de ce solde que Claude Tag peut consommer à chaque période de facturation.

Les DM individuels des membres qui ont connecté un compte Claude ne sont **pas** prélevés sur ce solde. Un tel DM s'exécute sur le compte claude.ai de l'expéditeur et suit les limites d'usage habituelles de son siège : la limite de dépense de l'organisation ne s'y applique donc pas. En revanche, le [DM d'un membre qui n'a pas connecté de compte Claude](https://claude.com/docs/claude-tag/admins/restrict-access#direct-messages-from-members-without-a-claude-account) peut être prélevé sur ce solde.

Pour connaître le coût de l'usage de votre équipe, lancez un pilote avec une limite de dépense définie et suivez la ventilation par canal sur la [page d'usage de vos réglages d'administration](https://claude.ai/admin-settings/usage/claude-tag) (`claude.ai/admin-settings/usage/claude-tag`). La page [Set a spend limit](https://claude.com/docs/claude-tag/admins/set-spend-limit) explique comment alimenter le solde sur chaque forfait, comment fixer la limite, et ce qui se passe quand l'usage l'atteint.

| Type de conversation | Qui paie | Limite applicable |
| :- | :- | :- |
| Canal, fil, DM de groupe | Solde d'utilisation de l'organisation | Limite de dépense de l'organisation |
| DM individuel d'un membre qui a connecté un compte Claude | Siège claude.ai de l'expéditeur | Limites d'usage habituelles de ce siège (pas la limite de l'organisation) |
| DM individuel d'un membre sans compte Claude connecté | Peut être prélevé sur le solde de l'organisation (durée limitée) | Voir [restrict-access](https://claude.com/docs/claude-tag/admins/restrict-access#direct-messages-from-members-without-a-claude-account) |

*(Tableau de synthèse ajouté pour la fiche, construit uniquement à partir des trois paragraphes ci-dessus.)*

---

*Pour les utilisateurs finaux*

## Mettre Claude Tag au travail

Si Claude Tag est dans votre canal, vous pouvez l'utiliser tout de suite. S'il n'y est pas encore, c'est un Owner de votre organisation Claude qui lance la configuration (voir [Set up Claude Tag](https://claude.com/docs/claude-tag/admins/setup-overview)). N'importe qui dans le canal peut lui confier du travail, et le travail en canal est facturé à l'organisation, pas à vous.

Ce que Claude peut atteindre dépend d'abord du canal où vous êtes. Le plus rapide pour le savoir est de lui demander :

```text
@Claude what can you access from this channel?
```

*(« Qu'est-ce que tu peux atteindre depuis ce canal ? »)*

Si vous êtes connecté à votre organisation Claude, vous pouvez aussi cliquer sur **Configure** dans le pied d'une réponse de Claude dans le canal. Vous y voyez ses [connexions](https://claude.com/docs/claude-tag/concepts/glossary#connection), c'est-à-dire les services externes qu'un admin a connectés pour ce canal. Les réponses publiées dans des canaux partagés à l'échelle de l'organisation (*org-shared*) n'ont pas de lien Configure.

Pour vos propres demandes, Claude peut aussi [utiliser les connecteurs de votre compte claude.ai](https://claude.com/docs/claude-tag/concepts/personal-connectors), une fois que vous l'avez autorisé.

Dans un DM individuel, Claude s'exécute sur votre propre compte claude.ai et non sur la configuration d'un canal. Un Owner peut désactiver les DM pour toute l'organisation : voir [Allow or disable direct messages](https://claude.com/docs/claude-tag/admins/restrict-access#allow-or-disable-direct-messages).

### Usages courants

Chaque lien ouvre un guide avec les prompts à coller et les connexions dont la tâche a besoin.

| Cas d'usage | Ce que fait Claude | Connexion requise |
| :- | :- | :- |
| [Watch monitors and alerts](https://claude.com/docs/claude-tag/users/use-cases/watch-monitors) (surveiller monitors et alertes) | Vérifications planifiées des tableaux de bord, et investigation des alertes dès leur arrivée | Une connexion de monitoring comme Datadog, Sentry ou PagerDuty |
| [Triage requests](https://claude.com/docs/claude-tag/users/use-cases/triage-requests) (trier les demandes) | Un canal d'intake où Claude répond à ce qu'il peut, signale les doublons et route le reste | Aucune : fonctionne avec le seul contenu Slack |
| [Find answers in your docs](https://claude.com/docs/claude-tag/users/use-cases/find-answers) (trouver des réponses dans vos docs) | Réponses aux questions de politique interne et de runbook, avec la source | Une connexion documentaire comme Google Drive, Notion ou Confluence |
| [Answer data questions](https://claude.com/docs/claude-tag/users/use-cases/answer-data-questions) (répondre aux questions de données) | Une question en langage courant devient une requête d'entrepôt et un graphique | Une connexion à un entrepôt de données |
| [Track projects and chase approvals](https://claude.com/docs/claude-tag/users/use-cases/track-projects) (suivre les projets et relancer les validations) | Digests de statut permanents et relances qui continuent jusqu'à l'obtention d'une validation | Non précisée |
| [Turn threads into docs and tickets](https://claude.com/docs/claude-tag/users/use-cases/create-artifacts) (transformer des fils en docs et tickets) | Une discussion tranchée devient le doc de décision, la réponse client ou les tickets déposés | Non précisée |
| [Fix bugs](https://claude.com/docs/claude-tag/users/use-cases/fix-bugs) (corriger des bugs) | Un bug signalé dans le canal revient sous forme de pull request brouillon (*draft*) | GitHub |
| [Work from your own channel](https://claude.com/docs/claude-tag/users/use-cases/your-own-channel) (travailler depuis votre propre canal) | Questions brouillon, digests de canaux que vous ne suivez pas, relances sur ce que vous avez dit que vous feriez | Non précisée |

La page [Get started](https://claude.com/docs/claude-tag/users/getting-started) couvre votre premier message, ce que vous voyez pendant que Claude travaille, et comment orienter le comportement de Claude dans votre canal.

---

*Pour les administrateurs*

## Configurer Claude Tag une fois pour tout le monde

La configuration se fait **une seule fois**, sur [`claude.ai/admin-settings/claude-tag`](https://claude.ai/admin-settings/claude-tag). Il faut être **Owner** de votre organisation Claude. La page de configuration à cette adresse vous guide pas à pas :

1. **Associer votre espace de travail Slack** : envoyez `@Claude connect` dans Slack pour obtenir un code d'association (*pairing code*), puis saisissez-le sur la page de configuration.
2. **Fixer une limite de dépense mensuelle, ajouter Claude aux canaux, et lancer.**

Une fois le lancement fait, toutes les personnes d'un canal où se trouve Claude peuvent utiliser Claude Tag immédiatement, sans configuration individuelle.

Au départ, Claude Tag n'a **aucun accès propre** à vos systèmes externes. Après le lancement, vous connectez les services dans lesquels Claude travaillera (votre outil de tickets ou votre entrepôt de données, par exemple) et vous accordez des dépôts à la **Claude GitHub App**. Les membres peuvent aussi laisser Claude utiliser leurs propres [connecteurs personnels](https://claude.com/docs/claude-tag/concepts/personal-connectors) pour leurs demandes.

La page [Set up Claude Tag](https://claude.com/docs/claude-tag/admins/setup-overview) détaille la configuration et la connexion des outils : ce qu'il faut avoir sous la main, ce que signifie chaque choix, et comment vérifier que Claude Tag fonctionne une fois lancé.

> **Revue de sécurité** : la page [Security and data handling](https://claude.com/docs/claude-tag/concepts/security-and-data) présente le modèle de sécurité, ce que les admins peuvent et ne peuvent pas restreindre, les pistes d'audit et les exigences réseau.

## Par où commencer avec Claude Tag

| Carte | Description | Lien |
| :- | :- | :- |
| Set up Claude Tag (icône `gear`) | Admins : associer votre espace de travail Slack et lancer | https://claude.com/docs/claude-tag/admins/setup-overview |
| Hand Claude Tag your first task (icône `paper-plane`) | Il est déjà dans votre canal : envoyez votre premier message | https://claude.com/docs/claude-tag/users/getting-started |
| How Claude Tag works (icône `diagram-project`) | Le modèle de session, ce qu'il peut lire, et comment la mémoire suit les lieux | https://claude.com/docs/claude-tag/concepts/how-it-works |
| Use case library (icône `list`) | Prompts à coller, par équipe et par connexion | https://claude.com/docs/claude-tag/users/use-cases |

Page suivante dans la documentation : [How Claude Tag works](https://claude.com/docs/claude-tag/concepts/how-it-works).

---

## Annexe : plan de la documentation Claude Tag

*(Données extraites de la navigation embarquée dans la page. Elles donnent la liste complète des pages de la section Claude Tag avec leur description officielle, traduite. Entre parenthèses : l'intitulé court affiché dans la barre latérale quand il diffère du titre.)*

### Sans groupe

| Page | Description |
| :- | :- |
| [Work with Claude Tag](https://claude.com/docs/claude-tag/overview) (« Overview ») | Cette page. |

### Fonctionnement (*How it works*)

| Page | Description |
| :- | :- |
| [How Claude Tag works](https://claude.com/docs/claude-tag/concepts/how-it-works) | Chaque fil Claude Tag dans Slack exécute une session de travail dans un bac à sable. Comment la progression s'affiche dans le fil, comment réorienter Claude en cours de tâche, ce qui survit entre les tours, et comment la portée de la mémoire est délimitée. |
| [Glossary](https://claude.com/docs/claude-tag/concepts/glossary) | Les termes de Claude Tag définis en un seul endroit, notamment *agent identity*, *bundle*, *connector*, *personal connector*, *scope*, *Agent Proxy*, *routine* et *session*. |

### Pour les administrateurs (*For administrators*)

#### Avant d'installer (*Before you install*)

| Page | Description |
| :- | :- |
| [How agent identity works](https://claude.com/docs/claude-tag/concepts/agent-identity) | Dans les canaux Slack, Claude Tag agit sous ses propres comptes de service, pas en votre nom. Comment l'accès d'un canal est borné, comment les identifiants lui parviennent, comment Claude utilise vos connecteurs personnels pour vos propres tâches dans un canal, et pourquoi les DM fonctionnent autrement. |
| [Personal connectors in channels](https://claude.com/docs/claude-tag/concepts/personal-connectors) | Claude peut utiliser vos propres connecteurs claude.ai, appelés connecteurs personnels, pour vos tâches dans un canal Slack. Comment vous approuvez l'usage d'un connecteur, quand Claude vous demande de relire un résultat avant de le publier, et ce que les autres personnes du canal peuvent atteindre. |
| [Security and data handling](https://claude.com/docs/claude-tag/concepts/security-and-data) | Comment Claude Tag garde les identifiants hors du bac à sable, limite les destinations possibles des requêtes d'une session de canal, et contrôle qui peut voir les artefacts et invoquer Claude. |
| [Data lifecycle and deletion](https://claude.com/docs/claude-tag/concepts/data-lifecycle) | Ce que Claude Tag stocke côté Anthropic, la durée de conservation des transcriptions de session et de la mémoire, et l'effet sur ces données de chacune de ces actions : déconnecter un espace de travail, désinstaller l'app, supprimer un canal Slack, retirer un scope, quitter l'organisation. |
| [What the Claude Slack app can access](https://claude.com/docs/claude-tag/admins/for-slack-admins) | Ce que l'app Claude lit et publie dans Slack, les scopes OAuth qu'elle demande, et ce que son installation n'accorde pas. Rédigée pour l'admin Slack qui approuve l'installation. |

#### Configurer Claude Tag (*Set up Claude Tag*, page de groupe : [setup-overview](https://claude.com/docs/claude-tag/admins/setup-overview))

| Page | Description |
| :- | :- |
| [Give Claude access to your tools](https://claude.com/docs/claude-tag/admins/add-connections) | Ajouter des connecteurs pour que Claude Tag agisse dans vos outils avec ses propres identifiants. Couvre les comptes de service, les bundles, les sites web autorisés, les entrées de domaine et les plugins. |
| [Configure GitHub access](https://claude.com/docs/claude-tag/admins/configure-github) | Donner à Claude sa propre identité GitHub via la Claude GitHub App. Couvre la liaison de votre organisation GitHub, l'octroi de dépôts et GitHub Actions. |
| [Configure GitLab access](https://claude.com/docs/claude-tag/admins/configure-gitlab) | Donner à Claude sa propre identité GitLab pour lire des projets, gérer des issues, relire des merge requests et vérifier des pipelines. Couvre le compte, le jeton et le connecteur. |
| [Set a spend limit](https://claude.com/docs/claude-tag/admins/set-spend-limit) | Le travail de Claude Tag en canal est facturé au solde d'utilisation de votre organisation. Définir la limite de dépense et ce qui se passe quand elle est atteinte. |
| [Set up Claude Tag for on-call](https://claude.com/docs/claude-tag/admins/setup-oncall) | Mettre Claude d'astreinte pour la réponse aux incidents dans Slack : lancer la configuration sur `claude.ai/oncall`, choisir les canaux d'incident ou les faire rejoindre automatiquement, plafonner la dépense, puis déployer. |

##### Guides de connexion (*Connection guides*)

| Page | Description |
| :- | :- |
| [Connect a service that isn't in the list](https://claude.com/docs/claude-tag/admins/connections/custom) | Connecter un outil absent de la liste des connecteurs de Claude Tag. Couvre les types d'identifiants, le sens de chaque champ du formulaire, et l'ajout d'un serveur MCP personnalisé. |
| [Connect Amplitude](https://claude.com/docs/claude-tag/admins/connections/amplitude) | Avec une clé API de projet et une clé secrète, ou en se connectant via Amplitude, pour que Claude réponde aux questions d'analytique produit. |
| [Connect Asana](https://claude.com/docs/claude-tag/admins/connections/asana) | Pour déposer des tâches et lire l'état des projets. Couvre le compte dédié à créer, les champs de jeton et l'URL à autoriser. |
| [Connect Jira and Confluence](https://claude.com/docs/claude-tag/admins/connections/atlassian) | Connecter Atlassian Cloud pour lire et mettre à jour des issues Jira et rechercher dans les pages Confluence. |
| [Connect BigQuery](https://claude.com/docs/claude-tag/admins/connections/bigquery) | Pour exécuter des requêtes en lecture seule sur vos datasets, avec une clé de compte de service GCP. |
| [Connect Datadog](https://claude.com/docs/claude-tag/admins/connections/datadog) | Pour interroger métriques, logs et monitors. Couvre le compte dédié à créer, les champs de clé API et l'URL à autoriser. |
| [Connect GitLab](https://claude.com/docs/claude-tag/admins/connections/gitlab) | Pour lire du code, gérer des issues, commenter des merge requests et vérifier des pipelines via l'API GitLab. Couvre les permissions du jeton, les noms d'hôte des instances auto-hébergées (*self-managed*) et la différence avec GitHub. |
| [Connect Gong](https://claude.com/docs/claude-tag/admins/connections/gong) | Pour récupérer des résumés d'appels et le contexte des deals. Couvre le compte dédié à créer, les champs de jeton et l'URL à autoriser. |
| [Connect Google Drive, Calendar, and Gmail](https://claude.com/docs/claude-tag/admins/connections/google) | Pour lire documents, feuilles de calcul, événements et e-mails. Couvre la configuration OAuth, l'option compte de service, et ce que chacune accorde. |
| [Connect HubSpot](https://claude.com/docs/claude-tag/admins/connections/hubspot) | Pour lire pipeline, deals et contacts. Couvre le compte dédié à créer, les champs de jeton et l'URL à autoriser. |
| [Connect Linear](https://claude.com/docs/claude-tag/admins/connections/linear) | Pour déposer des tickets et publier des mises à jour de statut. Couvre le compte dédié à créer, les champs de jeton et l'URL à autoriser. |
| [Connect Notion](https://claude.com/docs/claude-tag/admins/connections/notion) | Pour lire et rechercher dans votre espace Notion. Couvre le compte dédié à créer, les champs de jeton et l'URL à autoriser. |
| [Connect PagerDuty](https://claude.com/docs/claude-tag/admins/connections/pagerduty) | Pour lire les incidents et les plannings d'astreinte. Couvre le compte dédié à créer, les champs de jeton et l'URL à autoriser. |
| [Connect Salesforce](https://claude.com/docs/claude-tag/admins/connections/salesforce) | Pour lire et mettre à jour des enregistrements CRM. Couvre l'app connectée (*connected app*) à créer, les champs d'identifiants client et l'hôte à autoriser. |
| [Connect Sentry](https://claude.com/docs/claude-tag/admins/connections/sentry) | Pour ramener erreurs et stack traces dans les fils. Couvre le compte dédié à créer, les champs de jeton et l'URL à autoriser. |
| [Connect Snowflake](https://claude.com/docs/claude-tag/admins/connections/snowflake) | Pour exécuter des requêtes en lecture seule sur votre entrepôt. Couvre l'utilisateur dédié à créer, le champ de jeton d'accès et l'hôte à autoriser. |
| [Connect Stripe](https://claude.com/docs/claude-tag/admins/connections/stripe) | Pour répondre aux questions de facturation et d'abonnement. Couvre le compte dédié à créer, les champs de clés et l'URL à autoriser. |
| [Connect Vercel](https://claude.com/docs/claude-tag/admins/connections/vercel) | Pour vérifier l'état et les logs des déploiements. Couvre le compte dédié à créer, les champs de jeton et l'URL à autoriser. |

##### Accès agent fédéré (*Federated agent access*)

| Page | Description |
| :- | :- |
| [Connect a gateway](https://claude.com/docs/claude-tag/admins/federated-access/connect-a-gateway) | Connecter une passerelle (*gateway*) que vous exploitez, pour que Claude Tag appelle vos services internes avec un jeton d'identité de courte durée au lieu d'un identifiant stocké. Couvre ce que la passerelle doit vérifier, son enregistrement dans la console, et la vérification de la connexion. |
| [Connect an AWS role](https://claude.com/docs/claude-tag/admins/federated-access/aws) | Laisser Claude se connecter à un rôle IAM de votre compte AWS avec un jeton d'identité de courte durée au lieu de clés d'accès stockées. Couvre le fournisseur d'identité et la *trust policy* à créer dans AWS, la connexion du rôle dans la console, et la vérification de la connexion dans CloudTrail. |
| [Connect a Google Cloud identity](https://claude.com/docs/claude-tag/admins/federated-access/gcp) | Laisser Claude appeler Google Cloud via *workload identity federation* avec un jeton d'identité de courte durée au lieu d'une clé de compte de service. Couvre le pool, le provider et les octrois IAM à créer, avec ou sans compte de service, et la connexion de l'identité dans la console. |
| [Connect an authorization server](https://claude.com/docs/claude-tag/admins/federated-access/authorization-server) | Laisser Claude échanger un jeton d'identité de courte durée contre un jeton d'accès auprès d'un serveur d'autorisation OAuth 2.0 que vous exploitez, puis appeler vos API avec. Couvre ce que votre *token endpoint* reçoit et doit vérifier, ce qu'il renvoie, et son enregistrement dans la console. |
| [Identity token reference](https://claude.com/docs/claude-tag/admins/federated-access/token-reference) | Les claims, l'émetteur (*issuer*), les clés de signature, la durée de vie et le format du sujet (*subject*) du jeton d'identité que Claude Tag présente à une passerelle, un fournisseur cloud ou un serveur d'autorisation. |
| [Limits for federated agent access](https://claude.com/docs/claude-tag/admins/federated-access/limits) | Nombres, longueurs, durées de vie et configurations non prises en charge pour l'accès agent fédéré : passerelles, rôles AWS, identités Google Cloud et serveurs d'autorisation. |
| [Troubleshoot federated agent access](https://claude.com/docs/claude-tag/admins/federated-access/troubleshooting) | Les erreurs de l'accès agent fédéré et le correctif de chacune : messages des boîtes de dialogue de la console, requêtes que Claude signale comme bloquées ou en échec, et rejets enregistrés par votre passerelle, AWS, Google Cloud ou votre serveur d'autorisation. |

#### Déploiement (*Roll out*)

| Page | Description |
| :- | :- |
| [Configure per-channel access](https://claude.com/docs/claude-tag/admins/attach-to-scope) | Choisir à quels espaces de travail et canaux Slack s'applique un bundle Claude Tag, rattacher des bundles aux canaux par leur nom, et voir comment les accès se cumulent quand des bundles se chevauchent. |
| [Restrict where Claude Tag operates](https://claude.com/docs/claude-tag/admins/restrict-access) | Claude Tag ne répond que là où il a été ajouté et sollicité. Qui peut l'invoquer, ce qui change dans les canaux avec invités et les canaux Slack Connect, le réglage de version par scope, comment le limiter à des canaux choisis, comment déléguer l'administration ou la configuration d'un canal, et comment le rendre silencieux ou le retirer. |
| [Manage workspaces and versions](https://claude.com/docs/claude-tag/admins/workspaces) | Connecter d'autres espaces de travail Slack ou une Enterprise Grid à Claude Tag, choisir la version de Claude Tag utilisée par chaque canal, et déconnecter un espace de travail. |
| [Use Claude Tag at a healthcare organization](https://claude.com/docs/claude-tag/admins/healthcare) | Voir l'avertissement ci-dessous. |

> **Avertissement** (organisations de santé) : Claude Tag **n'est pas couvert** par le Business Associate Agreement (BAA) d'Anthropic. Configurez-le pour que les informations de santé protégées (PHI) n'atteignent jamais un canal, un DM ou un outil que Claude peut lire : limitez Claude aux canaux approuvés, désactivez les DM, et ne connectez que des outils sans PHI. La page [Use Claude Tag at a healthcare organization](https://claude.com/docs/claude-tag/admins/healthcare) explique aussi quoi faire si des PHI sont publiées là où Claude peut les lire.

#### Personnalisation (*Customize*)

| Page | Description |
| :- | :- |
| [Customize Claude Tag](https://claude.com/docs/claude-tag/admins/customize) | Claude Tag se personnalise par canal et par espace de travail (un *scope*), pas par utilisateur. Ce que les admins règlent dans claude.ai, ce que chacun peut changer depuis le canal, et ce qui reste fixe. |
| [Claude Tag settings map](https://claude.com/docs/claude-tag/concepts/settings-map) | La carte des réglages : la page d'admin pour l'accès et le comportement, la page d'usage pour les limites de dépense, le lien Configure dans Slack pour les instructions de canal, et les connecteurs personnels pour les DM et vos propres tâches en canal. Claude Managed Agents se configure à part, sur la Claude Platform. |
| [Manage a channel's instructions from another channel](https://claude.com/docs/claude-tag/admins/managed-by) | *Managed by* permet à une équipe centrale d'écrire, depuis son propre canal, les instructions permanentes de Claude pour un autre canal Slack. Couvre la configuration, la carte *Confirm* et les limites. |
| [Set up a skills repository Claude can update](https://claude.com/docs/claude-tag/admins/skills-repo) | Placer les skills Claude Tag de votre organisation dans un dépôt git avec synchronisation automatique et accorder à Claude l'accès en écriture. Claude peut alors ouvrir des pull requests pour améliorer ses propres skills à partir de ce qu'il apprend dans les canaux. |

#### Exploitation (*Operate*)

| Page | Description |
| :- | :- |
| [Review what Claude Tag has done](https://claude.com/docs/claude-tag/admins/audit) | Les actions de Claude Tag apparaissent sous ses propres comptes de service dans le journal d'audit de chaque outil connecté. Ce que couvre la page *Activity*, comment remonter d'une action à sa source, et où chaque outil connecté conserve ses logs. |
| [Attribute costs to users](https://claude.com/docs/claude-tag/admins/attribute-costs) | Récupérer la dépense des canaux par utilisateur Slack depuis la Claude Enterprise Analytics API, pour du *showback* ou du *chargeback*, et voir comment le travail de Claude est attribué aux personnes. |
| [Troubleshoot Claude Tag setup](https://claude.com/docs/claude-tag/admins/troubleshooting) | Les messages d'erreur de la configuration et le correctif de chacun : incohérences de permissions, accès GitHub manquants, échecs de démarrage de session et erreurs de compte. |
| [Network requirements](https://claude.com/docs/claude-tag/admins/network-requirements) | Claude Tag atteint vos services depuis une plage d'egress publiée, en HTTP et HTTPS uniquement. Le bloc IP à mettre en liste d'autorisation, et le lien entre cette liste et le réglage « sites web autorisés » (*allowed websites*) d'une connexion. |
| [Migrate from the earlier Claude in Slack](https://claude.com/docs/claude-tag/admins/migrate-from-earlier) | Claude Tag remplace sur place l'ancienne app Claude in Slack, qui fonctionnait par utilisateur. Ce qui change, ce qui reste, comment la version est choisie par canal, et ce que remarquent les utilisateurs existants. |

### Pour les utilisateurs finaux (*For end users*)

| Page | Description |
| :- | :- |
| [Get started](https://claude.com/docs/claude-tag/users/getting-started) | Claude Tag fonctionne dans vos canaux Slack. Vérifier qu'il est activé, envoyer votre premier message, ce qu'il peut lire, et pourquoi les DM fonctionnent autrement. |
| [Claude Tag for Claude Code users](https://claude.com/docs/claude-tag/concepts/for-claude-code-users) (« For Claude Code users ») | Quelles parties d'une configuration Claude Code passent dans Claude Tag, lesquelles basculent dans les réglages d'admin, et comment les fils Slack correspondent aux sessions. |
| [Commands Claude Tag understands](https://claude.com/docs/claude-tag/users/commands) (« Commands ») | Quelques mots exacts préfixés par `!` (*bang*), placés après une @-mention, exécutent une action fixe au lieu de démarrer un tour normal. La page couvre : la liste des commandes, obtenir le lien vers la page de réglages d'un canal, redémarrer une session bloquée ou partie sur le mauvais contexte, vérifier si Claude travaille encore dans un fil ou un canal, rendre un fil muet ou le réactiver, activer ou désactiver le *fast mode*, envoyer un retour, lister les routines d'un canal, et forker la conversation d'un fil dans un nouveau fil, dans le même canal ou dans un autre. |
| [Control when Claude Tag responds](https://claude.com/docs/claude-tag/users/when-claude-responds) | Ce que fait Claude Tag d'un message de canal où personne ne l'a tagué, et comment désactiver ses réponses spontanées pour un fil ou un canal. À lire si Claude répond trop, ou s'il s'est tu. |
| [Use Claude Tag in a group DM](https://claude.com/docs/claude-tag/users/group-dms) (« Group DMs ») | Ajouter Claude à un DM de groupe Slack pour partager une tâche avec quelques personnes. Quels accès et quelle facturation s'appliquent, quand Claude répond, et ce qui diffère d'un canal. |
| [Good habits for working with Claude Tag](https://claude.com/docs/claude-tag/users/good-habits) (« Good habits ») | Nommer le résultat attendu, donner à chaque tâche une définition de « terminé », et choisir le bon canal. Comment écrire des tâches que Claude peut mener à bout et garder de nombreux fils faciles à relire. |
| [Set up routines](https://claude.com/docs/claude-tag/users/proactivity) | Claude Tag exécute les routines que vous configurez depuis un canal : tâches planifiées, surveillance de canal, abonnements aux pull requests, lieu de publication des résultats, et mise en pause. |
| [What Claude Tag remembers](https://claude.com/docs/claude-tag/users/memory) | La mémoire de Claude Tag appartient au canal, pas à vous. Ce que chaque canal conserve, quelles notes sont partagées dans tout l'espace de travail, et comment les vérifier ou les corriger. |
| [Choose the model Claude Tag uses](https://claude.com/docs/claude-tag/users/models) (« Choose the model ») | Demander à Claude de changer de modèle dans un fil Slack, définir le modèle par défaut d'un canal, choisir le modèle de vos DM, ou exécuter un fil en *fast mode*. Quels modèles vous pouvez utiliser, et comment savoir lequel a répondu. |
| [Prompt library](https://claude.com/docs/claude-tag/users/prompt-library) | Prompts à copier-coller pour Claude Tag dans Slack, chacun avec l'explication de son efficacité : premiers messages, passations, règles de canal, vérifications de mémoire, routines, *fan-out* (répartition sur plusieurs tâches) et pilotage. |
| [Troubleshoot Claude Tag in channels and DMs](https://claude.com/docs/claude-tag/users/troubleshooting) (« Troubleshooting ») | Correctifs aux problèmes courants dans Slack : pas de réponse après une réaction, sessions en file d'attente ou en échec, travail perdu, connexions manquantes, liens et sites web bloqués, erreurs de DM ou de compte. |

### Bibliothèque de cas d'usage (*Use case library*)

| Page | Description |
| :- | :- |
| [Use case library](https://claude.com/docs/claude-tag/users/use-cases) (« Browse the library ») | Les formes de travail que les équipes confient à Claude Tag dans Slack, chacune avec des prompts à coller : triage, rattrapage, docs et tickets, suivi de projet, données, deals, monitoring et corrections de bugs. |
| [Triage requests](https://claude.com/docs/claude-tag/users/use-cases/triage-requests) | Claude Tag trie un canal de demandes à partir de deux messages de configuration : réponses dans le fil, signalement des doublons, routage vers les responsables, synthèses hebdomadaires par thème, dépôt de tickets en option, et runbooks de réponses permanentes. |
| [Catch up](https://claude.com/docs/claude-tag/users/use-cases/catch-up) | Claude Tag résume fils et canaux à la demande : récapitulatifs ponctuels, synthèse matinale planifiée des fils ouverts, et ce qui attend votre action. |
| [Work from your own channel](https://claude.com/docs/claude-tag/users/use-cases/your-own-channel) | Créer un canal Slack réservé à vous et à Claude Tag et y piloter votre travail : questions brouillon, digests de canaux que vous ne suivez pas, digest de statut hebdomadaire, relances poursuivies jusqu'à leur clôture, et une passation qui couvre votre absence. |
| [Turn threads into docs and tickets](https://claude.com/docs/claude-tag/users/use-cases/create-artifacts) | Claude Tag transforme une discussion Slack en l'artefact que vous nommez : réponses prêtes à envoyer, docs de décision, mémos de statut, tickets déposés, pages web hébergées, commentaires de page qui parviennent à Claude, et un modèle de « canal de capture ». |
| [Track projects and chase approvals](https://claude.com/docs/claude-tag/users/use-cases/track-projects) | Claude Tag publie des digests de projet que personne n'a à compiler : mises à jour de statut permanentes et relances sur contrats, revues de design et pull requests jusqu'à leur clôture. |
| [Answer data questions](https://claude.com/docs/claude-tag/users/use-cases/answer-data-questions) | Claude Tag répond aux questions de données dans le fil Slack : requêtes d'entrepôt avec graphiques, rapports de métriques planifiés, et graphiques construits à partir du seul historique du canal. |
| [Find answers in your docs](https://claude.com/docs/claude-tag/users/use-cases/find-answers) | Claude Tag trouve les réponses dans les docs connectées et répond dans le fil : recherches de politiques internes, vérifications de runbooks, décisions antérieures, et réponses tirées du seul canal Slack. |
| [Review documents against a checklist](https://claude.com/docs/claude-tag/users/use-cases/review-documents) | Claude Tag relit des documents d'un outil connecté au regard d'une checklist ou d'une politique et publie ses constats dans le fil : vérification d'un document, revues par lots, listes de dépôt (*filing lists*), comparaisons avec des revues passées, et balayage hebdomadaire planifié. |
| [Pull deal and account state](https://claude.com/docs/claude-tag/users/use-cases/pull-deal-state) | Claude Tag ramène l'état des comptes et des deals dans le canal Slack : réponses sur un compte dans le fil, briefs avant un appel, et digest hebdomadaire planifié du pipeline. |
| [Claude Tag for marketing teams](https://claude.com/docs/claude-tag/users/use-cases/marketing-team) (« Marketing teams ») | Recettes Claude Tag pour les équipes marketing dans Slack : réponses de politique interne dans un canal d'intake, récapitulatifs de canaux de campagne et brouillons prêts à envoyer, état des leads et des campagnes depuis le CRM, digest hebdomadaire des métriques, et règles de voix de marque enregistrées dans la mémoire du canal. |
| [Watch monitors and alerts](https://claude.com/docs/claude-tag/users/use-cases/watch-monitors) | Claude Tag surveille tableaux de bord et canaux d'alertes pour que vous ne voyiez qu'une ligne par problème : vérifications planifiées, investigation des alertes avant que quiconque ne le demande, et les prompts pour mettre en place les deux. |
| [Fix bugs](https://claude.com/docs/claude-tag/users/use-cases/fix-bugs) | Claude Tag mène un rapport de bug Slack jusqu'à une pull request brouillon : reproduction du problème, surveillance d'un canal de bugs, recherche de cause racine, et suivi de la CI jusqu'au vert. |
| [Work with your GitHub repositories](https://claude.com/docs/claude-tag/users/use-cases/work-with-github) (« Work with GitHub ») | Claude Tag travaille avec vos dépôts GitHub depuis Slack : répondre aux questions dans le fil, s'abonner aux pull requests, et rendre de petites tâches sous forme de pull requests brouillons. |

### Contexte du site

- Claude Tag est l'un des onglets de la documentation Claude.ai (`https://claude.com/docs`), à côté de : Welcome, Customize Claude, Build for Claude, Cowork, Claude for M365, Claude Science, Claude on third-party platforms et Claude for Government.
- D'après les compteurs de recherche du site, la section Claude Tag compte 76 pages : 61 étiquetées « For administrators », 30 sous « Set up Claude Tag », dont 18 guides de connexion.
- Mots-clés déclarés pour cette page : *what is claude tag, claude in slack, claude slack app, slack bot, slack agent, @claude, tag claude, claude tag pricing, cost, how is claude tag billed, which plans include claude tag, team plan, enterprise plan, pro plan, max plan, personal account, claude tag vs cowork, use cases, public beta*.
