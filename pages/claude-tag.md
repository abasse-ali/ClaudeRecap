# Travailler avec Claude Tag

> Source : https://code.claude.com/docs/en/claude-tag · [← Index](../31-index-documentation-officielle.md)

*Claude Tag place Claude dans vos canaux Slack, avec un accès gouverné par les administrateurs. Cette page présente ce qu'on peut lui confier, comment se déroule la configuration, et par où commencer selon que l'on est administrateur ou utilisateur final.*

> **Note** : l'URL `code.claude.com/docs/en/claude-tag` aboutit à la documentation **Claude.ai** (site « Claude.ai Documentation », hébergé sur Mintlify). La page canonique est https://claude.com/docs/claude-tag/overview, intitulée « Work with Claude Tag » (« Overview » dans la barre latérale). La source téléchargée est la page HTML complète : l'essentiel de son volume est du CSS, du JavaScript et des données de rendu (Next.js). Tout son contenu textuel est repris ci-dessous, ainsi que les données utiles qu'elle embarque (métadonnées, table des matières, plan complet de la documentation Claude Tag avec la description de chaque page). Les liens internes de cette documentation pointent vers `https://claude.com/docs/...`.

## Métadonnées de la page

| Champ | Valeur |
|---|---|
| Titre | `Work with Claude Tag` (« Travailler avec Claude Tag ») |
| Titre dans la barre latérale (`sidebarTitle`) | `Overview` (« Vue d'ensemble ») |
| Titre de l'onglet du navigateur | `Work with Claude Tag - Claude.ai Documentation` |
| URL canonique | https://claude.com/docs/claude-tag/overview |
| Fichier source d'origine | `claude-tag/overview.mdx` |
| Version Markdown de la page | `/docs/claude-tag/overview.md` (type `text/markdown`, soit https://claude.com/docs/claude-tag/overview.md) |
| Plan du site | `/docs/sitemap.xml` |
| Mode d'affichage | `wide` (page large, sans colonne « Sur cette page ») |
| Dernière modification | 2026-10-05 à 18:53:22 UTC (`2026-10-05T18:53:22.206Z`) |
| Statut du produit | **Public Beta** (bêta publique) |
| Fil d'Ariane | Claude.ai Documentation (https://claude.com/docs) › Work with Claude Tag |
| Éditeur | Claude.ai Documentation (organisation `https://claude.com/#organization`) |
| Type de contenu (données structurées) | `Article`, `TechArticle`, `WebPage` |
| Image de partage (Open Graph / Twitter) | image générée par Mintlify à partir du titre et de la description |

**Description officielle** (champ `description`, reprise en `og:description` et `twitter:description`) : « Claude Tag puts Claude in your Slack channels with admin-governed access. See what to hand it, how setup works, and where to start as an admin or end user. » — Claude Tag met Claude dans vos canaux Slack avec un accès gouverné par les admins ; voyez quoi lui confier, comment fonctionne la configuration et par où commencer en tant qu'admin ou utilisateur final.

**Mots-clés de recherche déclarés** (tels quels, en anglais) : `what is claude tag` · `claude in slack` · `claude slack app` · `slack bot` · `slack agent` · `@claude` · `tag claude` · `claude tag pricing` · `cost` · `how is claude tag billed` · `which plans include claude tag` · `team plan` · `enterprise plan` · `pro plan` · `max plan` · `personal account` · `claude tag vs cowork` · `use cases` · `public beta`. Autrement dit, la page répond à : qu'est-ce que Claude Tag, Claude dans Slack, l'app Slack Claude, bot ou agent Slack, comment taguer Claude, tarif et coût, mode de facturation, forfaits concernés (Team, Enterprise, Pro, Max), compte personnel, différence avec Cowork, cas d'usage, bêta publique.

**Table des matières de la page** (extraite des données de la page) :

1. Forfaits incluant Claude Tag (`#plans-that-include-claude-tag`)
   - Qui a besoin d'un siège Claude (`#who-needs-a-claude-seat`)
2. Où Claude Tag s'exécute (`#where-claude-tag-runs`)
3. Facturation et limites de dépense (`#billing-and-spend-limits`)
4. Mettre Claude Tag au travail (`#put-claude-tag-to-work`)
   - Usages courants (`#common-uses`)
5. Configurer Claude Tag une fois pour tout le monde (`#set-claude-tag-up-once-for-everyone`)
6. Par où commencer avec Claude Tag (`#where-to-start-with-claude-tag`)

> **Note** : en haut de chaque page du site figure aussi l'encart « Documentation Index » : l'index complet de la documentation est disponible à `/docs/llms.txt` (https://claude.com/docs/llms.txt), à utiliser pour découvrir toutes les pages disponibles avant d'explorer plus loin.

## Bandeau d'accueil

Le haut de la page est un bandeau illustré (il remplace le titre habituel de la page) :

- **Pastille** : `Public Beta` (bêta publique).
- **Accroche** : « Tag `@Claude` in. Get results back in the thread. » — Taguez `@Claude`. Récupérez les résultats dans le fil.
- **Texte** : n'importe qui dans un canal Slack peut taguer Claude sur un problème et lui confier du travail : reproduire un bug et ouvrir une pull request, transformer un fil de décision en document, rassembler l'état d'un projet. Claude publie une checklist dans le fil au fur et à mesure, et tout l'échange reste visible pour le canal.
- **Deux boutons d'appel à l'action** :
  - « I'm setting it up → » (« Je le configure → ») : mène à [Set up Claude Tag](https://claude.com/docs/claude-tag/admins/setup-overview), le parcours de configuration pour les administrateurs.
  - « Use it in your channel ↓ » (« L'utiliser dans votre canal ↓ ») : descend vers la section [Mettre Claude Tag au travail](#mettre-claude-tag-au-travail) de cette page.

### Exemple de fil Slack (maquette)

À côté du texte, une maquette montre un échange typique dans le canal `# platform-eng` (38 membres) :

1. **Dana** (14 h 14 / *2:14 PM*) : « checkout has felt slow all morning — anyone else seeing it? » — le checkout est lent depuis ce matin, quelqu'un d'autre le constate ?
2. **Leo** (14 h 15 / *2:15 PM*) : « same. `@Claude` can you investigate? Compare latency against this morning's deploy and find what's causing it. » — pareil. `@Claude`, tu peux enquêter ? Compare la latence avec le déploiement de ce matin et trouve ce qui la cause.
3. **Claude** (badge `APP`, 14 h 15 / *2:15 PM*) : « On it. I'll compare latency before and after the deploy, track down the cause, and report back here. » — Je m'en occupe. Je compare la latence avant et après le déploiement, je trouve la cause et je fais un retour ici.

La réponse de Claude est suivie de sa checklist de progression (les étapes terminées sont cochées, l'étape en cours est signalée comme telle) :

| État | Étape (texte d'origine) | Traduction |
|---|---|---|
| Fait (*Done*) | Pulled p99 latency from Datadog | Latence p99 récupérée depuis Datadog |
| Fait (*Done*) | Diffed deploy 4f2c1 against main | Comparaison (diff) du déploiement `4f2c1` avec `main` |
| Fait (*Done*) | Reproduced the slow query locally | Requête lente reproduite en local |
| En cours (*In progress*) | Opening a pull request with the fix… | Ouverture d'une pull request avec le correctif… |

L'exemple illustre le fonctionnement décrit plus bas : quelqu'un tague `@Claude` dans un fil, Claude annonce son plan, avance étape par étape en publiant sa checklist dans le fil, s'appuie sur une connexion (ici Datadog) et sur le dépôt de code, et termine par une pull request, le tout visible par tout le canal.

## Forfaits incluant Claude Tag

Claude Tag est disponible sur les forfaits **Team** et **Enterprise**, sur le service first-party d'Anthropic. Il **n'est pas** disponible :

- sur les forfaits individuels (**Free**, **Pro** ou **Max**) ;
- pour les déploiements tiers (*third-party deployments*).

Pour l'utiliser, votre organisation associe (*pairs*) son espace de travail Slack à son organisation Claude ; les prérequis complets sont dans [la vue d'ensemble de la configuration](https://claude.com/docs/claude-tag/admins/setup-overview).

Si vous hésitez entre plusieurs produits Claude pour du travail « en forme de Slack », la section [en quoi Claude Tag diffère de Cowork et de Claude Code](https://claude.com/docs/claude-tag/concepts/how-it-works#how-claude-tag-differs-from-cowork-and-claude-code) les compare directement : le travail d'équipe dans des canaux partagés relève de Claude Tag ; le travail personnel sur vos propres fichiers relève de Cowork ou de Claude Code.

### Qui a besoin d'un siège Claude

Les utilisateurs Slack n'ont **pas** besoin, chacun, d'un siège Claude pour travailler avec Claude dans les canaux. Selon l'endroit :

- **Dans les canaux** : par défaut, n'importe qui dans l'espace de travail Slack associé peut taguer `@Claude` dans un canal. Un **Owner** peut [restreindre qui peut utiliser Claude](https://claude.com/docs/claude-tag/admins/restrict-access#restrict-who-can-use-claude) aux personnes de votre organisation Claude ou, sur **Enterprise**, à des rôles précis. Le travail en canal est facturé à l'usage sur le solde d'usage (*usage balance*) de votre organisation, dans la limite d'une [limite de dépense](https://claude.com/docs/claude-tag/admins/set-spend-limit) (*spend limit*) fixée par un Owner.
- **Dans les DM de groupe** (*group DMs*) : la même restriction sur qui peut utiliser Claude s'applique, et le travail est facturé au solde d'usage de votre organisation. Voir [Group DMs](https://claude.com/docs/claude-tag/admins/restrict-access#group-dms).
- **Dans les DM en tête-à-tête** (*one-to-one DMs*) : un DM envoyé par un membre qui a connecté un compte Claude s'exécute sur ce compte et est facturé sur le siège de cette personne. Ce siège doit inclure **Claude Code** ou, sur le forfait **Enterprise**, être un siège **Standard** ou **Usage-Based Chat** détenu par quelqu'un qui a aussi **Cowork**. Un [DM d'un membre qui n'a pas connecté de compte Claude](https://claude.com/docs/claude-tag/admins/restrict-access#direct-messages-from-members-without-a-claude-account) peut être facturé à votre organisation pendant une durée limitée.

En résumé :

| Où l'on parle à Claude | Qui peut l'utiliser | Sur quoi c'est facturé |
|---|---|---|
| Canal (et ses fils) | Par défaut tout l'espace Slack associé ; restreignable par un Owner (organisation Claude, ou rôles sur Enterprise) | Solde d'usage de l'organisation, plafonné par la limite de dépense |
| DM de groupe | Même restriction que pour les canaux | Solde d'usage de l'organisation |
| DM en tête-à-tête, membre avec compte Claude connecté | Le membre lui-même (siège avec Claude Code, ou siège Enterprise Standard / Usage-Based Chat + Cowork) | Siège de la personne (ses limites d'usage habituelles) |
| DM en tête-à-tête, membre sans compte Claude connecté | Selon les réglages de l'organisation | Organisation, pendant une durée limitée |

## Où Claude Tag s'exécute

Claude Tag fonctionne dans **Slack**. Vous interagissez avec lui en écrivant dans un canal, un fil ou un message direct Slack, et il répond au même endroit. Mentionnez `@Claude` dans un canal pour être sûr qu'il prenne le message en compte.

Quand Claude travaille sur une tâche, il s'exécute dans un **bac à sable éphémère** (*ephemeral sandbox*), pas sur votre ordinateur. Ce bac à sable :

- est créé quand une conversation commence ;
- contient le code ou les fichiers sur lesquels Claude travaille ;
- est supprimé quand la conversation devient inactive (*goes idle*).

Le cycle de vie complet est décrit dans [How Claude Tag works](https://claude.com/docs/claude-tag/concepts/how-it-works).

Vous étendez ce que Claude peut atteindre — vos dépôts, vos systèmes de tickets, vos entrepôts de données, vos outils maison — au moyen des [connecteurs Claude Tag](https://claude.com/docs/claude-tag/admins/add-connections) (*Claude Tag connectors*), des [plugins et des skills](https://claude.com/docs/claude-tag/admins/customize). Un Owner ou un [administrateur Claude Tag](https://claude.com/docs/claude-tag/admins/restrict-access#delegate-claude-tag-administration) (*Claude Tag admin*) les configure à l'un de ces trois niveaux :

- pour tout Slack ;
- pour un espace de travail ;
- pour un canal.

Les connecteurs claude.ai propres à chaque membre sont distincts de cette configuration : Claude peut les utiliser dans un canal pour les demandes de ce membre, comme l'explique [Personal connectors in channels](https://claude.com/docs/claude-tag/concepts/personal-connectors) (connecteurs personnels dans les canaux).

### Cartes d'orientation

Sous cette section, deux grandes cartes illustrées orientent le lecteur selon son profil.

**Pour les administrateurs — « Set up Claude Tag »** (configurer Claude Tag ; carte menant à [setup-overview](https://claude.com/docs/claude-tag/admins/setup-overview)) :

| Question | Réponse proposée | Lien |
|---|---|---|
| Where do I start? (Par où commencer ?) | Associer votre espace Slack, lancer, et vérifier que ça marche | [Set up Claude Tag](https://claude.com/docs/claude-tag/admins/setup-overview) |
| What can Claude Tag access? (À quoi Claude Tag peut-il accéder ?) | Comment les admins règlent l'accès par canal, et où sont stockés les identifiants | [How agent identity works](https://claude.com/docs/claude-tag/concepts/agent-identity) |
| How do I connect each service? (Comment connecter chaque service ?) | Types d'identifiants, hôtes autorisés, et ce que chaque connexion permet à Claude d'atteindre | [Give Claude access to your tools](https://claude.com/docs/claude-tag/admins/add-connections) |

**Pour les utilisateurs finaux — « Put Claude Tag to work »** (mettre Claude Tag au travail ; carte menant à la section [Mettre Claude Tag au travail](#mettre-claude-tag-au-travail)) :

| Question | Réponse proposée | Lien |
|---|---|---|
| How do I hand Claude Tag a task? (Comment confier une tâche à Claude Tag ?) | Mentionnez Claude dans n'importe quel canal où il est présent, sans rien installer | [Get started](https://claude.com/docs/claude-tag/users/getting-started) |
| What is Claude Tag good at? (En quoi Claude Tag est-il bon ?) | Cas d'usage pour le code, les données, les incidents et le go-to-market | [Use case library](https://claude.com/docs/claude-tag/users/use-cases) |
| How do I get good results? (Comment obtenir de bons résultats ?) | Bonnes habitudes pour cadrer et relire le travail | [Good habits](https://claude.com/docs/claude-tag/users/good-habits) |
| What does Claude Tag remember? (De quoi Claude Tag se souvient-il ?) | Mémoire de canal, ce qui est partagé à l'échelle de l'espace de travail, et qui peut voir quoi | [What Claude Tag remembers](https://claude.com/docs/claude-tag/users/memory) |
| Can Claude Tag run tasks on a schedule? (Claude Tag peut-il exécuter des tâches planifiées ?) | Tâches planifiées, surveillance de canal et déclencheurs | [Set up routines](https://claude.com/docs/claude-tag/users/proactivity) |

## Facturation et limites de dépense

Ajouter Claude à Slack **n'ajoute pas de coût par siège**. Le travail en canal, en fil et en DM de groupe est facturé **à l'usage** : il puise dans un **solde d'usage** (*usage balance*), un montant dans la devise de facturation de votre organisation, alimenté par un Owner. Une [limite de dépense](https://claude.com/docs/claude-tag/admins/set-spend-limit) (*spend limit*) plafonne la part de ce solde que Claude Tag peut consommer à chaque période de facturation.

Les DM en tête-à-tête envoyés par des membres qui ont connecté un compte Claude **ne puisent pas** dans ce solde. Un tel DM s'exécute sur le compte claude.ai de l'expéditeur et suit les limites d'usage habituelles de ce siège ; la limite de dépense de l'organisation ne s'y applique donc pas. En revanche, un [DM d'un membre qui n'a pas connecté de compte Claude](https://claude.com/docs/claude-tag/admins/restrict-access#direct-messages-from-members-without-a-claude-account) peut puiser dans ce solde.

Pour savoir ce que coûte l'usage de votre équipe : lancez un **pilote** avec une limite de dépense définie et suivez la répartition **par canal** sur la [page d'usage de vos paramètres d'administration](https://claude.ai/admin-settings/usage/claude-tag) (`claude.ai/admin-settings/usage/claude-tag`). La page [Set a spend limit](https://claude.com/docs/claude-tag/admins/set-spend-limit) explique comment alimenter le solde selon le forfait, comment fixer la limite, et ce qui se passe quand l'usage l'atteint.

| Type de conversation | Puise dans le solde d'usage de l'organisation ? | Soumis à la limite de dépense de l'organisation ? |
|---|---|---|
| Canal, fil, DM de groupe | Oui | Oui |
| DM en tête-à-tête d'un membre avec compte Claude connecté | Non (siège claude.ai de l'expéditeur) | Non (limites d'usage du siège) |
| DM en tête-à-tête d'un membre sans compte Claude connecté | Peut puiser dans le solde | Oui, puisqu'il consomme le solde |

## Mettre Claude Tag au travail

*(Section destinée aux utilisateurs finaux — « For end users ».)*

Si Claude Tag est dans votre canal, vous pouvez l'utiliser dès maintenant. (S'il n'y est pas encore, c'est un Owner de votre organisation Claude qui lance la configuration : voir [Set up Claude Tag](https://claude.com/docs/claude-tag/admins/setup-overview).) N'importe qui dans le canal peut lui confier du travail, et le travail en canal est facturé à l'organisation, pas à vous.

Ce que Claude peut atteindre dépend d'abord du canal où vous êtes. Le plus rapide pour le savoir est de lui demander :

- en écrivant `@Claude what can you access from this channel?` (« à quoi as-tu accès depuis ce canal ? ») ;
- ou, si vous êtes connecté à votre organisation Claude, en cliquant sur **Configure** dans le pied d'une réponse de Claude dans le canal, pour voir ses [connexions](https://claude.com/docs/claude-tag/concepts/glossary#connection) (*connections*), c'est-à-dire les services externes qu'un admin a connectés pour ce canal. Les réponses dans les canaux partagés entre organisations (*org-shared channels*) n'ont **pas** de lien **Configure**.

Pour vos propres demandes, Claude peut aussi [utiliser les connecteurs de votre compte claude.ai](https://claude.com/docs/claude-tag/concepts/personal-connectors), une fois que vous l'avez autorisé.

Dans un DM en tête-à-tête, Claude s'exécute sur votre propre compte claude.ai plutôt que sur la configuration d'un canal. Un Owner peut désactiver les DM pour toute l'organisation ; voir [Allow or disable direct messages](https://claude.com/docs/claude-tag/admins/restrict-access#allow-or-disable-direct-messages).

### Usages courants

Chaque lien ouvre un guide avec les prompts à coller et les connexions dont la tâche a besoin.

| Usage | Ce que fait Claude | Connexion nécessaire |
|---|---|---|
| [Watch monitors and alerts](https://claude.com/docs/claude-tag/users/use-cases/watch-monitors) (surveiller moniteurs et alertes) | Vérifications planifiées de tableaux de bord, et alertes analysées dès leur arrivée | Une connexion de monitoring, comme Datadog, Sentry ou PagerDuty |
| [Triage requests](https://claude.com/docs/claude-tag/users/use-cases/triage-requests) (trier les demandes) | Un canal d'entrée (*intake*) où Claude répond à ce qu'il peut, signale les doublons et oriente le reste | Aucune : fonctionne avec le seul contenu Slack |
| [Find answers in your docs](https://claude.com/docs/claude-tag/users/use-cases/find-answers) (trouver des réponses dans vos docs) | Questions de politique interne et de runbook répondues avec la source | Une connexion documentaire, comme Google Drive, Notion ou Confluence |
| [Answer data questions](https://claude.com/docs/claude-tag/users/use-cases/answer-data-questions) (répondre aux questions de données) | Une question en langage courant devient une requête sur l'entrepôt de données et un graphique | Une connexion à un entrepôt de données |
| [Track projects and chase approvals](https://claude.com/docs/claude-tag/users/use-cases/track-projects) (suivre les projets et relancer les validations) | Synthèses d'état récurrentes et relances qui continuent jusqu'à ce qu'une validation tombe | — |
| [Turn threads into docs and tickets](https://claude.com/docs/claude-tag/users/use-cases/create-artifacts) (transformer des fils en docs et tickets) | Une discussion tranchée devient le document de décision, la réponse au client ou les tickets créés | — |
| [Fix bugs](https://claude.com/docs/claude-tag/users/use-cases/fix-bugs) (corriger des bugs) | Un bug signalé dans le canal revient sous forme de pull request en brouillon | GitHub |
| [Work from your own channel](https://claude.com/docs/claude-tag/users/use-cases/your-own-channel) (travailler depuis votre propre canal) | Questions brouillon, synthèses de canaux que vous ne suivez pas, et relances sur ce que vous avez dit que vous feriez | — |

La page [Get started](https://claude.com/docs/claude-tag/users/getting-started) couvre votre premier message, ce que vous voyez pendant que Claude travaille, et comment façonner le comportement de Claude dans votre canal.

## Configurer Claude Tag une fois pour tout le monde

*(Section destinée aux administrateurs — « For administrators ».)*

Vous configurez Claude Tag **une seule fois**, à l'adresse [`claude.ai/admin-settings/claude-tag`](https://claude.ai/admin-settings/claude-tag), et vous devez être **Owner** de votre organisation Claude pour le faire. La page de configuration à cette adresse vous guide :

1. **Associer votre espace de travail Slack** : envoyez `@Claude connect` dans Slack pour obtenir un code d'association (*pairing code*), puis saisissez-le sur la page de configuration.
2. **Fixer une limite de dépense mensuelle, ajouter Claude aux canaux, et lancer** (*launch*).

Une fois le lancement fait, toute personne présente dans un canal où se trouve Claude peut utiliser Claude Tag immédiatement, **sans configuration par utilisateur**.

Claude Tag démarre **sans aucun accès propre** à vos systèmes externes. Après le lancement, vous connectez les services dans lesquels Claude travaillera — par exemple votre outil de suivi de tickets ou votre entrepôt de données — et vous accordez des dépôts à la **Claude GitHub App**. Les membres peuvent aussi laisser Claude utiliser leurs propres [connecteurs personnels](https://claude.com/docs/claude-tag/concepts/personal-connectors) pour leurs demandes.

La page [Set up Claude Tag](https://claude.com/docs/claude-tag/admins/setup-overview) détaille la configuration et la connexion des outils : ce qu'il faut avoir sous la main, ce que signifie chaque choix, et comment vérifier que Claude Tag fonctionne une fois lancé.

### Encart « Revue de sécurité »

Un encart **Security review** (revue de sécurité) renvoie vers [Security and data handling](https://claude.com/docs/claude-tag/concepts/security-and-data) (sécurité et traitement des données) : le modèle de sécurité, ce que les admins peuvent et ne peuvent pas restreindre, les pistes d'audit (*audit trails*) et les exigences réseau.

## Par où commencer avec Claude Tag

Quatre cartes horizontales (avec flèche) terminent la page :

| Carte | Icône | Texte | Lien |
|---|---|---|---|
| Set up Claude Tag (configurer Claude Tag) | `gear` (engrenage) | Admins : associez votre espace Slack et lancez | https://claude.com/docs/claude-tag/admins/setup-overview |
| Hand Claude Tag your first task (confier sa première tâche à Claude Tag) | `paper-plane` (avion en papier) | Il est déjà dans votre canal : envoyez votre premier message | https://claude.com/docs/claude-tag/users/getting-started |
| How Claude Tag works (comment fonctionne Claude Tag) | `diagram-project` (schéma) | Le modèle de session, ce qu'il peut lire, et comment la mémoire suit les lieux | https://claude.com/docs/claude-tag/concepts/how-it-works |
| Use case library (bibliothèque de cas d'usage) | `list` (liste) | Prompts à coller, par équipe et par connexion | https://claude.com/docs/claude-tag/users/use-cases |

En bas de page : la question « Was this page helpful? » (Cette page vous a-t-elle été utile ?) avec les boutons **Yes** / **No**, puis le lien vers la page suivante, [How Claude Tag works](https://claude.com/docs/claude-tag/concepts/how-it-works).

## Annexe A : plan complet de la documentation Claude Tag

La page embarque l'arborescence complète de l'onglet **Claude Tag** de la documentation Claude.ai : titre de chaque page, titre court dans la barre latérale, description et mots-clés de recherche. Dans la barre latérale affichée, les sous-groupes administrateurs (« Before you install », « Set up Claude Tag », « Roll out », « Customize », « Operate ») sont repliés ; leur contenu est détaillé ici. Les mots-clés sont recopiés tels quels (en anglais) : ce sont les termes de recherche sous lesquels chaque page est indexée, et ils donnent une idée précise de son contenu (messages d'erreur, commandes, noms de réglages).

### Vue d'ensemble (groupe sans titre)

- **[Work with Claude Tag](https://claude.com/docs/claude-tag/overview)** (barre latérale : *Overview*) — la présente page.

### Fonctionnement (*How it works*)

- **[How Claude Tag works](https://claude.com/docs/claude-tag/concepts/how-it-works)** (comment fonctionne Claude Tag) — Chaque fil Claude Tag dans Slack exécute une session de travail dans un bac à sable. La page montre comment la progression s'affiche dans le fil, comment réorienter Claude en cours de tâche, ce qui survit entre les tours, et quelle est la portée de la mémoire.
  - Mots-clés : `how claude tag works` · `session` · `thread vs channel` · `top-level message` · `new session` · `when does a session end` · `what survives between replies` · `sandbox` · `where does claude tag run` · `checklist` · `progress` · `steer mid-task` · `read the channel` · `other channels` · `search slack` · `cross-channel` · `channel and workspace memory` · `claude tag vs cowork` · `claude tag vs claude code` · `one-off and scheduled tasks`
- **[Glossary](https://claude.com/docs/claude-tag/concepts/glossary)** (glossaire) — Les termes de Claude Tag définis en un seul endroit, notamment *agent identity* (identité d'agent), *bundle* (lot d'accès), *connector* (connecteur), *personal connector* (connecteur personnel), *scope* (portée), *Agent Proxy*, *routine* et *session*.

### Pour les administrateurs (*For administrators*)

#### Avant d'installer (*Before you install*)

- **[How agent identity works](https://claude.com/docs/claude-tag/concepts/agent-identity)** (comment fonctionne l'identité d'agent) — Dans les canaux Slack, Claude Tag agit sous ses propres comptes de service, pas en votre nom. La page explique comment l'accès d'un canal est borné, comment les identifiants lui parviennent, comment Claude utilise vos connecteurs personnels pour vos propres tâches dans un canal, et pourquoi les DM sont différents.
  - Mots-clés : `service account` · `agent identity` · `whose credentials` · `acts as itself` · `not as you` · `on behalf of` · `who is the author` · `pull request author` · `claude github app` · `attribution` · `dm vs channel` · `group dm` · `group direct message` · `personal connectors in dms` · `personal connectors in channels` · `agent proxy` · `credential injection`
- **[Personal connectors in channels](https://claude.com/docs/claude-tag/concepts/personal-connectors)** (connecteurs personnels dans les canaux) — Claude peut utiliser vos propres connecteurs claude.ai, appelés connecteurs personnels, pour vos tâches dans un canal Slack. La page explique comment vous approuvez l'usage d'un connecteur, quand Claude vous demande de relire un résultat avant de le publier, et ce que les autres personnes du canal peuvent atteindre.
- **[Security and data handling](https://claude.com/docs/claude-tag/concepts/security-and-data)** (sécurité et traitement des données) — Comment Claude Tag garde les identifiants hors du bac à sable, limite les destinations possibles des requêtes d'une session de canal, et contrôle qui peut voir les artefacts et invoquer Claude.
  - Mots-clés : `security` · `privacy` · `zdr` · `zero data retention` · `compliance` · `restricted compliance configuration` · `where does claude tag run` · `sandbox` · `isolation` · `credential storage` · `network egress` · `service account` · `who can see artifacts` · `private channel data` · `isolate credentials between channels` · `member access` · `self-hosted` · `byoc` · `bring your own cloud` · `cross-channel posting` · `post in another channel` · `ask in one channel act in another channel`
- **[Data lifecycle and deletion](https://claude.com/docs/claude-tag/concepts/data-lifecycle)** (barre latérale : *Data lifecycle* — cycle de vie et suppression des données) — Ce que Claude Tag stocke côté Anthropic, combien de temps les transcriptions de session et la mémoire sont conservées, et ce que deviennent ces données quand on déconnecte un espace de travail, désinstalle l'app, supprime un canal Slack, retire une portée (*scope*) ou quitte l'organisation.
  - Mots-clés : `data retention` · `retention period` · `delete data` · `deletion` · `uninstall` · `disconnect workspace` · `remove claude from channel` · `what gets deleted` · `transcripts` · `memory retention` · `zero data retention` · `zdr` · `audit log` · `archived session` · `!restart`
- **[What the Claude Slack app can access](https://claude.com/docs/claude-tag/admins/for-slack-admins)** (barre latérale : *For Slack admins* — ce à quoi l'app Slack Claude a accès) — Ce que l'app Claude lit et publie dans Slack, les portées OAuth (*OAuth scopes*) qu'elle demande, et ce que son installation n'accorde **pas**. Rédigée pour l'administrateur Slack qui approuve l'installation.
  - Mots-clés : `slack admin` · `approve the app` · `oauth scopes` · `scopes` · `permissions the app requests` · `channels:join` · `what the app can read` · `where claude posts` · `uninstall the app` · `slack app approval`

#### Configurer Claude Tag (*Set up Claude Tag*)

Page racine du groupe : **[Set up Claude Tag](https://claude.com/docs/claude-tag/admins/setup-overview)** (configurer Claude Tag) — Configurer Claude Tag pour votre organisation : associer votre espace Slack, ajouter Claude aux canaux, lancer, donner à Claude l'accès à vos outils, et vérifier que tout fonctionne.

- Mots-clés : `set up claude in slack` · `add claude to slack` · `install the claude slack app` · `@claude connect` · `claim code` · `pairing code` · `pair slack workspace` · `connect slack workspace` · `slack admin` · `slack workspace owner` · `claude owner` · `enable claude tag` · `turn on claude tag` · `start setup` · `admin-settings/claude-tag` · `enterprise grid` · `workspace_ code` · `enterprise_ code` · `zdr` · `enable routines` · `routines not enabled` · `prerequisites` · `usage credits` · `launch claude tag` · `verify setup`

Pages du groupe :

- **[Give Claude access to your tools](https://claude.com/docs/claude-tag/admins/add-connections)** (barre latérale : *Give Claude access* — donner à Claude l'accès à vos outils) — Ajouter des connecteurs pour que Claude Tag agisse dans vos outils avec ses propres identifiants. Couvre les comptes de service, les *bundles*, les sites web autorisés (*allowed websites*), les entrées de domaine et les plugins.
  - Mots-clés : `connector` · `connectors tab` · `add a connector` · `bundle` · `access bundle` · `claude's access` · `custom connector` · `allowed websites` · `allowlist` · `whitelist domain` · `egress` · `host not allowed` · `network access` · `internet access` · `web search` · `api key` · `bearer token` · `service account` · `dedicated account` · `setup link` · `connection vs connector` · `personal connectors` · `attach plugins` · `test connection` · `restrict access` · `assign access` · `new thread after adding a connection`
- **[Configure GitHub access](https://claude.com/docs/claude-tag/admins/configure-github)** (barre latérale : *GitHub access* — configurer l'accès GitHub) — Donner à Claude sa propre identité GitHub via la **Claude GitHub App**. Couvre la liaison de votre organisation GitHub, l'octroi de dépôts et GitHub Actions.
- **[Configure GitLab access](https://claude.com/docs/claude-tag/admins/configure-gitlab)** (barre latérale : *GitLab access* — configurer l'accès GitLab) — Donner à Claude sa propre identité GitLab pour lire des projets, gérer des issues, relire des merge requests et vérifier des pipelines. Couvre le compte, le jeton et le connecteur.
  - Mots-clés : `gitlab` · `self-managed gitlab` · `gitlab.com` · `merge request` · `gitlab api` · `gitlab token` · `git host other than github` · `gitlab clone` · `gitlab limitations`
- **[Set a spend limit](https://claude.com/docs/claude-tag/admins/set-spend-limit)** (fixer une limite de dépense) — Le travail de Claude Tag en canal est facturé au solde d'usage de votre organisation. Fixer la limite de dépense et voir ce qui se passe quand elle est atteinte.
  - Mots-clés : `pricing` · `cost` · `how much does claude tag cost` · `billing` · `usage balance` · `credits` · `usage credits` · `buy usage credits` · `per seat` · `seats` · `subscription or usage` · `consumption billing` · `budget` · `spend per channel` · `per-channel limit` · `spend per user` · `usage limit` · `who pays for dms` · `dm billing` · `rate limit` · `spend limit reached` · `discount` · `list price` · `does reading a channel cost` · `proactive replies cost`
- **[Set up Claude Tag for on-call](https://claude.com/docs/claude-tag/admins/setup-oncall)** (barre latérale : *On-call setup* — configurer Claude Tag pour l'astreinte) — Mettre Claude d'astreinte pour la réponse aux incidents dans Slack : lancer la configuration sur `claude.ai/oncall`, choisir les canaux d'incident ou les rejoindre automatiquement (*auto-join*), plafonner la dépense, et déployer.

##### Guides de connexion (*Connection guides*)

Page racine du sous-groupe : **[Per-service connection guides](https://claude.com/docs/claude-tag/admins/connections/overview)** (barre latérale : *Connection guides* — guides de connexion par service) — Configuration pas à pas pour chaque outil auquel Claude Tag peut se connecter. Chaque guide couvre le compte dédié à créer, l'identifiant à saisir et l'URL à autoriser.

- Mots-clés : `connection guides` · `supported services` · `supported tools` · `integrations` · `which tools can claude connect to` · `datadog` · `sentry` · `pagerduty` · `linear` · `asana` · `jira` · `confluence` · `notion` · `google drive` · `google calendar` · `gmail` · `hubspot` · `salesforce` · `gong` · `gitlab` · `bigquery` · `snowflake` · `amplitude` · `stripe` · `vercel` · `service not listed` · `service without a guide` · `no preset` · `other services` · `custom tool`

| Guide (titre court) | Page | Ce que la connexion permet à Claude, et ce que couvre le guide |
|---|---|---|
| Custom connector (connecteur personnalisé) | [Connect a service that isn't in the list](https://claude.com/docs/claude-tag/admins/connections/custom) | Connecter un outil absent de la liste des connecteurs de Claude Tag. Couvre les types d'identifiants, la signification de chaque champ du formulaire, et l'ajout d'un serveur MCP personnalisé. |
| Amplitude | [Connect Amplitude](https://claude.com/docs/claude-tag/admins/connections/amplitude) | Connexion avec une clé d'API de projet et une clé secrète, ou en se connectant avec Amplitude, pour que Claude réponde aux questions d'analytique produit. |
| Asana | [Connect Asana](https://claude.com/docs/claude-tag/admins/connections/asana) | Créer des tâches et lire l'état des projets. Couvre le compte dédié à créer, les champs de jeton et l'URL à autoriser. |
| Jira and Confluence | [Connect Jira and Confluence](https://claude.com/docs/claude-tag/admins/connections/atlassian) | Connecter Atlassian Cloud pour lire et mettre à jour des issues Jira et chercher dans les pages Confluence. |
| BigQuery | [Connect BigQuery](https://claude.com/docs/claude-tag/admins/connections/bigquery) | Exécuter des requêtes en lecture seule sur vos datasets, avec une clé de compte de service GCP. |
| Datadog | [Connect Datadog](https://claude.com/docs/claude-tag/admins/connections/datadog) | Interroger métriques, logs et moniteurs. Couvre le compte dédié à créer, les champs de clé d'API et l'URL à autoriser. |
| GitLab | [Connect GitLab](https://claude.com/docs/claude-tag/admins/connections/gitlab) | Lire le code, gérer les issues, commenter les merge requests et vérifier les pipelines via l'API GitLab. Couvre les permissions du jeton, les noms d'hôte auto-hébergés (*self-managed*) et les différences avec GitHub. |
| Gong | [Connect Gong](https://claude.com/docs/claude-tag/admins/connections/gong) | Récupérer les résumés d'appels et le contexte des deals. Couvre le compte dédié, les champs de jeton et l'URL à autoriser. |
| Google Drive, Calendar, and Gmail | [Connect Google Drive, Calendar, and Gmail](https://claude.com/docs/claude-tag/admins/connections/google) | Lire docs, feuilles de calcul, événements et e-mails. Couvre la configuration OAuth, l'option compte de service, et ce que chacune accorde. |
| HubSpot | [Connect HubSpot](https://claude.com/docs/claude-tag/admins/connections/hubspot) | Lire les données de pipeline, de deals et de contacts. Couvre le compte dédié, les champs de jeton et l'URL à autoriser. |
| Linear | [Connect Linear](https://claude.com/docs/claude-tag/admins/connections/linear) | Créer des tickets et publier des mises à jour d'état. Couvre le compte dédié, les champs de jeton et l'URL à autoriser. |
| Notion | [Connect Notion](https://claude.com/docs/claude-tag/admins/connections/notion) | Lire et chercher dans votre espace Notion. Couvre le compte dédié, les champs de jeton et l'URL à autoriser. |
| PagerDuty | [Connect PagerDuty](https://claude.com/docs/claude-tag/admins/connections/pagerduty) | Lire les incidents et les plannings d'astreinte. Couvre le compte dédié, les champs de jeton et l'URL à autoriser. Mots-clés : `pagerduty` · `pager duty` · `on-call` · `on-call schedule` · `incidents` · `incident response` · `pagerduty api key` · `pagerduty token`. |
| Salesforce | [Connect Salesforce](https://claude.com/docs/claude-tag/admins/connections/salesforce) | Lire et mettre à jour des enregistrements CRM. Couvre l'app connectée (*connected app*) à créer, les champs d'identifiants client et l'hôte à autoriser. |
| Sentry | [Connect Sentry](https://claude.com/docs/claude-tag/admins/connections/sentry) | Ramener erreurs et stack traces dans les fils. Couvre le compte dédié, les champs de jeton et l'URL à autoriser. |
| Snowflake | [Connect Snowflake](https://claude.com/docs/claude-tag/admins/connections/snowflake) | Exécuter des requêtes en lecture seule sur votre entrepôt. Couvre l'utilisateur dédié à créer, le champ de jeton d'accès et l'hôte à autoriser. |
| Stripe | [Connect Stripe](https://claude.com/docs/claude-tag/admins/connections/stripe) | Répondre aux questions de facturation et d'abonnements. Couvre le compte dédié, les champs de clé et l'URL à autoriser. |
| Vercel | [Connect Vercel](https://claude.com/docs/claude-tag/admins/connections/vercel) | Vérifier l'état des déploiements et les logs. Couvre le compte dédié, les champs de jeton et l'URL à autoriser. |

Mots-clés de la page « Custom connector » : `custom mcp` · `mcp server` · `remote mcp server` · `add mcp to claude tag` · `custom connection` · `custom connector` · `custom tool` · `connect another tool` · `bearer token` · `custom header` · `authorization header` · `oauth client credentials` · `api key` · `internal api` · `internal service` · `rest api` · `service not in the list` · `credential types` · `token format`

##### Accès agent fédéré (*Federated agent access*)

Page racine du sous-groupe : **[Federated agent access](https://claude.com/docs/claude-tag/admins/federated-access/overview)** (barre latérale : *Overview*) — Claude Tag prouve son identité auprès de vos systèmes avec un **jeton signé de courte durée** plutôt qu'avec un identifiant stocké dans Claude. Explique le fonctionnement du jeton et lequel des **quatre types de connexion** utiliser.

| Page (titre court) | Lien | Contenu |
|---|---|---|
| Connect a gateway (connecter une passerelle) | [Connect a gateway](https://claude.com/docs/claude-tag/admins/federated-access/connect-a-gateway) | Connecter une passerelle que vous exploitez pour que Claude Tag appelle vos services internes avec un jeton d'identité de courte durée au lieu d'un identifiant stocké. Couvre ce que la passerelle doit vérifier, comment l'enregistrer dans la console, et comment vérifier la connexion. |
| AWS role (rôle AWS) | [Connect an AWS role](https://claude.com/docs/claude-tag/admins/federated-access/aws) | Laisser Claude se connecter à un rôle IAM de votre compte AWS avec un jeton d'identité de courte durée au lieu de clés d'accès stockées. Couvre le fournisseur d'identité et la politique d'approbation (*trust policy*) à créer dans AWS, la connexion du rôle dans la console, et la vérification de la connexion dans CloudTrail. |
| Google Cloud identity (identité Google Cloud) | [Connect a Google Cloud identity](https://claude.com/docs/claude-tag/admins/federated-access/gcp) | Laisser Claude appeler Google Cloud via la fédération d'identité de charge de travail (*workload identity federation*) avec un jeton d'identité de courte durée au lieu d'une clé de compte de service. Couvre le pool, le fournisseur et les autorisations IAM à créer, avec ou sans compte de service, et la connexion de l'identité dans la console. |
| Authorization server (serveur d'autorisation) | [Connect an authorization server](https://claude.com/docs/claude-tag/admins/federated-access/authorization-server) | Laisser Claude échanger un jeton d'identité de courte durée contre un jeton d'accès auprès d'un serveur d'autorisation OAuth 2.0 que vous exploitez, puis appeler vos API avec. Couvre ce que votre endpoint de jeton reçoit et doit vérifier, ce qu'il renvoie, et comment l'enregistrer dans la console. |
| Token reference (référence du jeton) | [Identity token reference](https://claude.com/docs/claude-tag/admins/federated-access/token-reference) | Les *claims*, l'émetteur (*issuer*), les clés de signature, la durée de vie et le format du sujet (*subject*) du jeton d'identité que Claude Tag présente à une passerelle, un fournisseur cloud ou un serveur d'autorisation. |
| Limits (limites) | [Limits for federated agent access](https://claude.com/docs/claude-tag/admins/federated-access/limits) | Nombres, longueurs, durées de vie et configurations non prises en charge pour l'accès agent fédéré : passerelles, rôles AWS, identités Google Cloud et serveurs d'autorisation. |
| Troubleshooting (dépannage) | [Troubleshoot federated agent access](https://claude.com/docs/claude-tag/admins/federated-access/troubleshooting) | Les erreurs de l'accès agent fédéré et leur correctif : messages des boîtes de dialogue de la console, requêtes que Claude signale comme bloquées ou en échec, et rejets enregistrés par votre passerelle, AWS, Google Cloud ou votre serveur d'autorisation. |

Les quatre types de connexion fédérée sont donc : **passerelle** (*gateway*), **rôle AWS**, **identité Google Cloud** et **serveur d'autorisation** OAuth 2.0.

#### Déploiement (*Roll out*)

- **[Configure per-channel access](https://claude.com/docs/claude-tag/admins/attach-to-scope)** (barre latérale : *Per-channel access* — configurer l'accès par canal) — Choisir à quels espaces de travail et canaux Slack s'applique un *bundle* Claude Tag, attacher des bundles aux canaux par leur nom, et voir comment les accès se cumulent quand des bundles se chevauchent.
  - Mots-clés : `scope` · `per channel` · `specific channel` · `assign bundle to channel` · `attach bundle` · `bind bundle to channel` · `where it applies` · `add place` · `add bundle` · `add a bundle` · `channel rule` · `channel group` · `channels tab` · `channel instructions` · `custom instructions` · `system prompt` · `instruction layers` · `inheritance` · `precedence` · `which credential wins` · `workspace default` · `different access per channel` · `channel not listed` · `search channels` · `channel id` · `restrict channel instructions`
- **[Restrict where Claude Tag operates](https://claude.com/docs/claude-tag/admins/restrict-access)** (restreindre où Claude Tag opère) — Claude Tag ne répond que là où il a été ajouté et sollicité. La page montre qui peut l'invoquer, ce qui change dans les canaux avec invités (*guests*) et Slack Connect, le réglage de version par portée, comment le limiter à des canaux choisis, comment déléguer l'administration ou la configuration d'un canal, et comment le faire taire ou le retirer.
  - Mots-clés : `private channel` · `guest` · `guests` · `slack connect` · `externally shared channel` · `shared channel` · `limit to specific channels` · `only certain channels` · `who can use claude` · `restrict users` · `roles` · `rbac` · `permissions` · `custom roles` · `claude tag admin` · `delegate administration` · `claude seat` · `without a claude account` · `channel manager` · `delegate setup` · `remove claude from a channel` · `turn off claude tag` · `disable dms` · `allow direct messages` · `group dm` · `group direct message` · `multi-person dm` · `dm without a claude account` · `billed to your organization` · `auto-join` · `block channels by name` · `channels claude can search` · `limit search` · `search scope` · `email domain` · `controls that aren't available` · `per-user spend cap`
- **[Manage workspaces and versions](https://claude.com/docs/claude-tag/admins/workspaces)** (barre latérale : *Workspaces and versions* — gérer espaces de travail et versions) — Connecter d'autres espaces Slack ou une **Enterprise Grid** à Claude Tag, choisir quelle version de Claude Tag utilise chaque canal, et déconnecter un espace de travail.
- **[Use Claude Tag at a healthcare organization](https://claude.com/docs/claude-tag/admins/healthcare)** (barre latérale : *Healthcare organizations* — utiliser Claude Tag dans un établissement de santé) — Configurer Claude Tag pour que des informations de santé protégées (PHI) n'atteignent jamais un canal, un message direct ou un outil que Claude peut lire : limiter Claude aux canaux approuvés, désactiver les messages directs, et ne connecter que des outils sans PHI. Couvre aussi la conduite à tenir si des PHI sont publiées là où Claude peut les lire.

> **Avertissement** : Claude Tag **n'est pas couvert** par le Business Associate Agreement (BAA) d'Anthropic. Dans un établissement de santé, il faut donc le configurer de sorte qu'aucune information de santé protégée (PHI) ne lui parvienne.

#### Personnalisation (*Customize*)

- **[Customize Claude Tag](https://claude.com/docs/claude-tag/admins/customize)** (personnaliser Claude Tag) — Claude Tag se personnalise **par canal et par espace de travail** (une portée, *scope*), pas par utilisateur. La page montre ce que les admins règlent dans claude.ai, ce que n'importe qui peut changer depuis le canal, et ce qui reste fixe.
  - Mots-clés : `environment` · `environment variables` · `env vars` · `setup script` · `install dependencies` · `install packages` · `python` · `runtime` · `language runtime` · `cloud environment` · `organization default environment` · `default model` · `change model` · `allowed models` · `auto mode` · `allow rules` · `rename claude` · `rename the agent` · `agent name` · `custom agent name` · `bot name` · `change the bot name` · `avatar` · `handle` · `personal connectors` · `respond automatically` · `configure page` · `settings admins control`
- **[Claude Tag settings map](https://claude.com/docs/claude-tag/concepts/settings-map)** (barre latérale : *Settings map* — carte des réglages) — Où se trouve chaque réglage : la page d'administration pour l'accès et le comportement, la page d'usage pour les limites de dépense, le lien **Configure** dans Slack pour les instructions de canal, et les connecteurs personnels pour les DM et vos propres tâches en canal. **Claude Managed Agents** se configure séparément, sur la Claude Platform.
- **[Manage a channel's instructions from another channel](https://claude.com/docs/claude-tag/admins/managed-by)** (barre latérale : *Managed by* — gérer les instructions d'un canal depuis un autre canal) — La fonction **Managed by** permet à une équipe centrale de rédiger, depuis son propre canal, les instructions permanentes de Claude pour un autre canal Slack. Couvre la mise en place, la carte de confirmation (*Confirm card*) et les limites.
  - Mots-clés : `managed by` · `managed instructions` · `managing channel` · `managed channel` · `central team` · `help desk instructions` · `on-call channel instructions` · `instructions from another channel` · `core instructions` · `reference files` · `confirm card` · `private channel instructions` · `correct claude privately` · `private correction`
- **[Set up a skills repository Claude can update](https://claude.com/docs/claude-tag/admins/skills-repo)** (barre latérale : *Add custom skills* — mettre en place un dépôt de skills que Claude peut mettre à jour) — Placer les skills Claude Tag de votre organisation dans un dépôt git avec synchronisation automatique (*auto-sync*), accorder à Claude l'accès en écriture ; Claude peut alors ouvrir des pull requests pour améliorer ses propres skills à partir de ce qu'il apprend dans les canaux.

#### Exploitation (*Operate*)

- **[Review what Claude Tag has done](https://claude.com/docs/claude-tag/admins/audit)** (barre latérale : *Audit and review* — revoir ce qu'a fait Claude Tag) — Les actions de Claude Tag apparaissent sous ses propres comptes de service dans le journal d'audit de chaque outil connecté. La page présente ce que couvre la page **Activity**, comment remonter une action jusqu'à son origine, et où chaque outil connecté conserve ses journaux.
  - Mots-clés : `audit` · `audit log` · `activity` · `what claude did` · `trace an action` · `who asked` · `network events` · `export` · `memory files` · `scheduled work in a channel` · `compliance`
- **[Attribute costs to users](https://claude.com/docs/claude-tag/admins/attribute-costs)** (attribuer les coûts aux utilisateurs) — Récupérer la dépense des canaux par utilisateur Slack via l'**API Claude Enterprise Analytics**, pour du *showback* ou du *chargeback* (refacturation interne), et voir comment le travail de Claude est attribué aux personnes.
- **[Troubleshoot Claude Tag setup](https://claude.com/docs/claude-tag/admins/troubleshooting)** (barre latérale : *Troubleshooting* — dépanner la configuration) — Les messages d'erreur de la configuration de Claude Tag et leur correctif : permissions discordantes, accès GitHub manquants, échecs de démarrage de session et erreurs de compte.
  - Mots-clés (dont plusieurs messages d'erreur, tels qu'affichés) : `not responding` · `silent` · `nothing happens` · `no reply` · `claude isn't on in this channel yet` · `claude in slack is not available for your organization` · `this workspace isn't set up for claude tag yet` · `using the legacy claude in slack bot` · `installation is out of date` · `unapproved permissions` · `only slack workspace admins can link this workspace` · `claim code invalid` · `already connected to a different organization` · `couldn't load slack scopes` · `guests` · `shared across multiple workspaces` · `enterprise grid` · `wrong environment` · `no runners` · `session failed to start` · `authentication error` · `routines aren't enabled` · `enable routines` · `permission checker blocks api calls` · `turned off claude sonnet 5` · `!restart`
- **[Network requirements](https://claude.com/docs/claude-tag/admins/network-requirements)** (exigences réseau) — Claude Tag atteint vos services depuis une plage de sortie (*egress*) publiée, **uniquement en HTTP et HTTPS**. La page donne le bloc d'adresses IP à autoriser (`160.79.104.0/21`, d'après ses mots-clés) et explique le lien entre cette liste d'autorisation IP et le réglage « sites web autorisés » (*allowed websites*) d'une connexion.
  - Mots-clés : `ip allowlist` · `allowlist ip` · `egress ip` · `ip range` · `160.79.104.0/21` · `firewall` · `source ip` · `static ip` · `dedicated ip` · `internet reachable` · `private network` · `vpn` · `webhooks` · `inbound events` · `allowed websites vs ip allowlist`
- **[Migrate from the earlier Claude in Slack](https://claude.com/docs/claude-tag/admins/migrate-from-earlier)** (barre latérale : *Migrate from legacy Claude in Slack* — migrer depuis l'ancien Claude in Slack) — Claude Tag remplace **sur place** l'ancienne app Claude in Slack, qui fonctionnait par utilisateur. La page explique ce qui change, ce qui reste, comment la version est choisie par canal, et ce que remarquent les utilisateurs existants.
  - Mots-clés : `legacy claude in slack` · `old claude slack app` · `earlier claude in slack` · `claude for slack` · `is it the same app` · `migrate` · `switch to claude tag` · `difference from the old app` · `version` · `inherit` · `legacy` · `new` · `off` (les quatre derniers correspondent aux valeurs du réglage de version par canal : hériter, ancienne version, nouvelle version, désactivé)

### Pour les utilisateurs finaux (*For end users*)

- **[Get started](https://claude.com/docs/claude-tag/users/getting-started)** (premiers pas) — Claude Tag fonctionne dans vos canaux Slack. Comment vérifier qu'il est activé, envoyer votre premier message, ce qu'il peut lire, et pourquoi les DM fonctionnent différemment.
- **[Claude Tag for Claude Code users](https://claude.com/docs/claude-tag/concepts/for-claude-code-users)** (barre latérale : *For Claude Code users* — Claude Tag pour les utilisateurs de Claude Code) — Quelles parties d'une configuration Claude Code se retrouvent dans Claude Tag, lesquelles passent dans les réglages d'administration, et comment les fils Slack correspondent aux sessions.
  - Mots-clés : `coming from claude code` · `claude code vs claude tag` · `CLAUDE.md` · `system prompt` · `hooks` · `subagents` · `effort` · `env var` · `environment variable` · `startup script` · `settings.json` · `.mcp.json` · `worktree` · `--continue` · `resume session` · `/model` · `/fast` · `fast mode` · `managed agents`
- **[Commands Claude Tag understands](https://claude.com/docs/claude-tag/users/commands)** (barre latérale : *Commands* — commandes comprises par Claude Tag) — Quelques mots exacts préfixés par un point d'exclamation (*bang*), placés après une @-mention, exécutent une action fixe au lieu de démarrer un tour normal. D'après la description de la page, ils permettent de :

  | Commande | Action (d'après la description et les mots-clés) |
  |---|---|
  | `!help` | Afficher la liste des commandes |
  | `!configure` | Obtenir le lien vers la page de réglages du canal |
  | `!restart` | Redémarrer une session bloquée ou partie dans un mauvais contexte |
  | `!status` | Vérifier si Claude travaille encore dans un fil ou un canal (et depuis combien de temps) |
  | `!mute` / `!unmute` | Rendre muet un fil, ou le réactiver |
  | `!fast` | Activer ou désactiver le mode rapide (*fast mode*) |
  | `!feedback` | Envoyer un retour (signaler un bug) |
  | `!routines` | Lister les routines d'un canal |
  | `!fork` | Dupliquer la conversation d'un fil dans un nouveau fil, ici ou dans un autre canal |

  - Mots-clés : `!help` · `!configure` · `!restart` · `!status` · `!feedback` · `!mute` · `!unmute` · `!fast` · `!routines` · `!fork` · `bang command` · `commands` · `channel settings` · `configure a channel` · `reset session` · `restart session` · `status check` · `is Claude still working` · `check progress` · `how long has Claude been working` · `mute a thread` · `silence a thread` · `thumbs down` · `thumbs-down button` · `muted in this thread` · `fast mode` · `faster replies` · `speed up Claude` · `send feedback` · `report a bug` · `list routines` · `fork a thread` · `continue a conversation in a new thread` · `continue a conversation in another channel` · `move a thread`
- **[Control when Claude Tag responds](https://claude.com/docs/claude-tag/users/when-claude-responds)** (contrôler quand Claude Tag répond) — Ce que fait Claude Tag d'un message de canal dans lequel personne ne l'a tagué, et comment désactiver les réponses spontanées pour un fil ou un canal. À lire si Claude répond trop, ou s'est tu.
  - Mots-clés : `ambient` · `ambience` · `auto-respond` · `auto respond` · `autorespond` · `respond automatically` · `proactivity` · `without mention` · `without @-mention` · `no tag` · `unprompted` · `proactive replies` · `channel awareness` · `stopped replying` · `went quiet` · `bot message` · `workflow builder` · `default`
- **[Use Claude Tag in a group DM](https://claude.com/docs/claude-tag/users/group-dms)** (barre latérale : *Group DMs* — utiliser Claude Tag dans un DM de groupe) — Ajouter Claude à un DM de groupe Slack pour partager une tâche avec quelques personnes. Quels accès et quelle facturation s'appliquent, quand Claude répond, et ce qui diffère d'un canal.
  - Mots-clés : `group dm` · `group direct message` · `multi-person dm` · `mpdm` · `add claude to a group dm` · `group dms aren't supported` · `claude in a group chat` · `dm with several people` · `group dm billing` · `group dm memory` · `mute claude in a group dm` · `quiet claude` · `reply only when mentioned`
- **[Good habits for working with Claude Tag](https://claude.com/docs/claude-tag/users/good-habits)** (barre latérale : *Good habits* — bonnes habitudes) — Nommer le résultat attendu, donner à chaque tâche une définition de « terminé », et choisir le bon canal. Comment rédiger des tâches que Claude peut mener à bout, et comment garder de nombreux fils faciles à relire.
- **[Set up routines](https://claude.com/docs/claude-tag/users/proactivity)** (mettre en place des routines) — Claude Tag exécute des routines que vous configurez depuis un canal : tâches planifiées, surveillance de canal, abonnements à des pull requests, endroit où les résultats sont publiés, et comment les mettre en pause.
  - Mots-clés : `routine` · `routines` · `scheduled` · `schedule` · `scheduled task` · `scheduled job` · `cron` · `recurring` · `daily report` · `daily standup` · `weekly digest` · `monitor a channel` · `watch channel` · `alert channel` · `follow a pull request` · `subscribe to a pull request` · `automation` · `run automatically` · `standing work` · `!routines` · `list routines` · `pause routine` · `stop routine`
- **[What Claude Tag remembers](https://claude.com/docs/claude-tag/users/memory)** (ce dont Claude Tag se souvient) — La mémoire de Claude Tag appartient **au canal**, pas à vous. Ce que chaque canal conserve, quelles notes sont partagées à l'échelle de l'espace de travail, et comment les vérifier ou les corriger.
  - Mots-clés : `memory` · `remember` · `forget` · `workspace memory` · `channel memory` · `memory store` · `where is memory stored` · `memory files` · `view memory` · `edit memory` · `delete memory` · `prune memory` · `private channel memory` · `dm memory` · `group dm memory` · `channel instructions vs memory` · `correct claude`
- **[Choose the model Claude Tag uses](https://claude.com/docs/claude-tag/users/models)** (barre latérale : *Choose the model* — choisir le modèle) — Demander à Claude de changer de modèle dans un fil Slack, définir le modèle par défaut d'un canal, choisir le modèle de vos messages directs, ou passer un fil en mode rapide. Quels modèles sont disponibles et comment confirmer lequel a répondu.
  - Mots-clés : `change model` · `switch model` · `default model` · `which models` · `opus` · `sonnet` · `model per channel` · `model per thread` · `what models can i use` · `available models` · `allowed models` · `model list` · `more models` · `model availability` · `make it the default` · `fast mode` · `!fast` · `faster replies` · `speed up Claude`
- **[Prompt library](https://claude.com/docs/claude-tag/users/prompt-library)** (bibliothèque de prompts) — Des prompts à copier-coller pour Claude Tag dans Slack, chacun avec la raison pour laquelle il fonctionne : premiers messages, passations (*handoffs*), règles de canal, vérifications de mémoire, routines, répartition en parallèle (*fan-out*) et réorientation (*steering*).
- **[Troubleshoot Claude Tag in channels and DMs](https://claude.com/docs/claude-tag/users/troubleshooting)** (barre latérale : *Troubleshooting* — dépanner Claude Tag dans les canaux et DM) — Correctifs pour les problèmes courants de Claude Tag dans Slack : pas de réponse après une réaction, sessions en file d'attente ou en échec, travail perdu, connexions manquantes, liens et sites web bloqués, erreurs de DM ou de compte.
  - Mots-clés (dont plusieurs messages d'erreur, tels qu'affichés) : `not responding` · `no reply` · `claude isn't on in this channel yet` · `claude is disabled in this channel` · `claude went quiet` · `claude ignores me` · `channels with guests` · `dm not working` · `seat that includes claude code` · `can only use claude in channels` · `agents can only be added at the start of direct messages` · `add claude to an existing dm` · `can't access this organization` · `turned off claude opus 5` · `wrong answer` · `too long` · `verbose` · `stop replying` · `report a bug` · `!feedback` · `!restart`

### Bibliothèque de cas d'usage (*Use case library*)

| Page (titre court) | Lien | Contenu |
|---|---|---|
| Browse the library (parcourir la bibliothèque) | [Use case library](https://claude.com/docs/claude-tag/users/use-cases) | Les types de travail que les équipes confient à Claude Tag dans Slack, chacun avec des prompts à coller : tri, rattrapage, docs et tickets, suivi de projets, données, deals, monitoring et correction de bugs. |
| Triage requests (trier les demandes) | [Triage requests](https://claude.com/docs/claude-tag/users/use-cases/triage-requests) | Claude Tag trie un canal de demandes Slack avec **deux messages de configuration** : réponses dans le fil, signalement des doublons, orientation vers le bon responsable, synthèses hebdomadaires par thème, création de tickets en option, et runbooks de réponses standard. Mots-clés : `runbook` · `standing answers` · `canned responses` · `prescribed answers` · `FAQ` · `playbook` · `recurring questions` · `same questions` · `repeat questions`. |
| Catch up (se remettre à jour) | [Catch up](https://claude.com/docs/claude-tag/users/use-cases/catch-up) | Claude Tag résume à la demande des fils et des canaux Slack : récapitulatifs ponctuels, synthèse matinale planifiée des fils ouverts, et ce qui vous attend. |
| Work from your own channel (travailler depuis votre propre canal) | [Work from your own channel](https://claude.com/docs/claude-tag/users/use-cases/your-own-channel) | Créer un canal Slack pour vous et Claude Tag seuls, et piloter votre travail depuis celui-ci : questions brouillon, synthèses de canaux que vous ne suivez pas, synthèse d'état hebdomadaire, relances poursuivies jusqu'à clôture, et une passation qui couvre vos absences. |
| Turn threads into docs and tickets (transformer des fils en docs et tickets) | [Turn threads into docs and tickets](https://claude.com/docs/claude-tag/users/use-cases/create-artifacts) | Claude Tag transforme une discussion Slack en l'artefact que vous nommez : réponses prêtes à envoyer, documents de décision, notes d'état, tickets créés, pages web hébergées, commentaires de page qui parviennent à Claude, et un modèle de « canal de capture ». |
| Track projects and chase approvals (suivre les projets et relancer les validations) | [Track projects and chase approvals](https://claude.com/docs/claude-tag/users/use-cases/track-projects) | Claude Tag publie des synthèses de projet que personne n'a à compiler : points d'état récurrents et relances sur contrats, revues de design et pull requests jusqu'à leur clôture. |
| Answer data questions (répondre aux questions de données) | [Answer data questions](https://claude.com/docs/claude-tag/users/use-cases/answer-data-questions) | Claude Tag répond aux questions de données dans le fil Slack : requêtes sur l'entrepôt avec graphiques, rapports de métriques planifiés, et graphiques construits à partir du seul historique du canal. |
| Find answers in your docs (trouver des réponses dans vos docs) | [Find answers in your docs](https://claude.com/docs/claude-tag/users/use-cases/find-answers) | Claude Tag trouve des réponses dans les docs connectées et répond dans le fil : recherche de politiques internes, vérification de runbooks, décisions antérieures, et réponses tirées du seul canal Slack. |
| Review documents against a checklist (relire des documents selon une checklist) | [Review documents against a checklist](https://claude.com/docs/claude-tag/users/use-cases/review-documents) | Claude Tag relit des documents d'un outil connecté selon une checklist ou une politique, et publie ses constats dans le fil : contrôle d'un document, revues par lots, listes de dépôt, comparaisons avec des revues passées, et un passage hebdomadaire planifié. |
| Pull deal and account state (récupérer l'état des deals et comptes) | [Pull deal and account state](https://claude.com/docs/claude-tag/users/use-cases/pull-deal-state) | Claude Tag ramène l'état des comptes et des deals dans le canal Slack : réponses sur un compte dans le fil, briefs avant appel, et synthèse hebdomadaire planifiée du pipeline. |
| Marketing teams (équipes marketing) | [Claude Tag for marketing teams](https://claude.com/docs/claude-tag/users/use-cases/marketing-team) | Recettes Claude Tag pour les équipes marketing dans Slack : réponses sur les politiques dans un canal d'entrée, récapitulatifs de canaux de campagne et brouillons prêts à envoyer, état des leads et campagnes depuis le CRM, synthèse hebdomadaire des métriques, et règles de ton de marque enregistrées dans la mémoire du canal. |
| Watch monitors and alerts (surveiller moniteurs et alertes) | [Watch monitors and alerts](https://claude.com/docs/claude-tag/users/use-cases/watch-monitors) | Claude Tag surveille tableaux de bord et canaux d'alertes pour que vous ne voyiez qu'une ligne par problème : vérifications planifiées, analyse des alertes avant que quiconque ne le demande, et prompts pour mettre en place les deux. |
| Fix bugs (corriger des bugs) | [Fix bugs](https://claude.com/docs/claude-tag/users/use-cases/fix-bugs) | Claude Tag mène un rapport de bug Slack jusqu'à une pull request en brouillon : reproduction du problème, surveillance d'un canal de bugs, recherche de cause racine, et suivi de la CI jusqu'au vert. |
| Work with GitHub (travailler avec GitHub) | [Work with your GitHub repositories](https://claude.com/docs/claude-tag/users/use-cases/work-with-github) | Claude Tag travaille avec vos dépôts GitHub depuis Slack : répondre à des questions dans le fil, s'abonner à des pull requests, et rendre des corvées sous forme de pull requests en brouillon. |

## Annexe B : contexte du site Claude.ai Documentation

### Onglets de la documentation

La barre de navigation du site propose ces onglets (la page appartient à l'onglet **Claude Tag**) :

| Onglet | Page d'entrée |
|---|---|
| Welcome (accueil) | https://claude.com/docs |
| Customize Claude (personnaliser Claude) | https://claude.com/docs/extend/overview |
| Build for Claude (développer pour Claude) | https://claude.com/docs/build/overview |
| Cowork | https://claude.com/docs/cowork/overview |
| Claude Tag | https://claude.com/docs/claude-tag/overview |
| Claude for M365 | https://claude.com/docs/office-agents/overview |
| Claude Science | https://claude.com/docs/claude-science/overview |
| Claude on third-party platforms (Claude sur des plateformes tierces) | https://claude.com/docs/third-party/claude-desktop/overview |
| Claude for Government (Claude pour le secteur public) | https://claude.com/docs/government/overview |

### En-tête et assistant

- Recherche : « Search... » avec le raccourci `⌘K`.
- Assistant : « Ask Assistant » avec le raccourci `⌘I` ; le panneau de l'assistant précise que les réponses sont générées par IA et peuvent contenir des erreurs (*Responses are generated using AI and may contain mistakes*).
- Liens : **Support** (https://support.anthropic.com) et **Go to Claude** (https://claude.ai).
- Lien d'accessibilité « Skip to main content » (aller au contenu principal).

> **Note** : la page charge un petit script (`desktop-app-nav.js`) propre à l'app **Claude Desktop** : le bouton « Go to Claude » ouvre normalement claude.ai dans une nouvelle fenêtre, ce que l'app de bureau refuse ; dans l'app (détectée par un user-agent contenant `Claude/` et `Electron/`), le script transforme donc les clics vers `https://claude.ai` (y compris avec cmd, ctrl ou shift) en navigation dans le même onglet, pour que l'utilisateur ne reste pas bloqué dans la documentation. Il ne fait rien dans un navigateur ordinaire, et n'agit que sur les liens dont l'origine est exactement `https://claude.ai`.

### Pied de page

- Réseaux : X (https://x.com/AnthropicAI) et LinkedIn (https://www.linkedin.com/company/anthropicresearch).

| Colonne | Liens |
|---|---|
| Company (entreprise) | [Anthropic](https://www.anthropic.com/company) · [Careers](https://www.anthropic.com/careers) · [Economic Futures](https://www.anthropic.com/economic-futures) · [Research](https://www.anthropic.com/research) · [News](https://www.anthropic.com/news) · [Trust center](https://trust.anthropic.com/) · [Transparency](https://www.anthropic.com/transparency) |
| Help and security (aide et sécurité) | [Availability](https://www.anthropic.com/supported-countries) · [Status](https://status.anthropic.com/) · [Support center](https://support.claude.com/) |
| Learn (apprendre) | [Courses](https://academy.claude.com/) · [MCP connectors](https://claude.com/partners/mcp) · [Customer stories](https://www.claude.com/customers) · [Engineering blog](https://www.anthropic.com/engineering) · [Events](https://www.anthropic.com/events) · [Powered by Claude](https://claude.com/partners/powered-by-claude) · [Service partners](https://claude.com/partners/services) · [Startups program](https://claude.com/programs/startups) |
| Terms and policies (conditions et politiques) | [Privacy policy](https://www.anthropic.com/legal/privacy) · [Disclosure policy](https://www.anthropic.com/responsible-disclosure-policy) · [Usage policy](https://www.anthropic.com/legal/aup) · [Commercial terms](https://www.anthropic.com/legal/commercial-terms) · [Consumer terms](https://www.anthropic.com/legal/consumer-terms) |
