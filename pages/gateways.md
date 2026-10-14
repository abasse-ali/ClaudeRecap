# Faire passer Claude Code par une passerelle (gateway)

> Source : https://code.claude.com/docs/en/gateways · [← Index](../31-index-documentation-officielle.md)

Faire passer Claude Code par une passerelle auto-hébergée pour centraliser les identifiants, le suivi de l'utilisation et le contrôle des coûts. Cette page couvre l'architecture, la passerelle Claude apps gateway d'Anthropic et l'utilisation d'autres produits de passerelle.

Une **passerelle** (gateway) est un proxy que votre organisation exploite entre Claude Code et un fournisseur de modèles. Claude Code envoie son trafic API à la passerelle au lieu de l'envoyer directement au fournisseur, et la passerelle le transmet en utilisant un identifiant détenu par votre organisation. Les développeurs s'authentifient auprès de la passerelle au lieu de détenir des identifiants du fournisseur : l'authentification, le suivi de l'utilisation, les budgets et la journalisation d'audit se font donc en un seul endroit que vous contrôlez.

Claude Code inclut une passerelle auto-hébergée, [Claude apps gateway](https://code.claude.com/docs/en/claude-apps-gateway), dans le binaire `claude` : vous n'avez pas besoin d'adopter un produit de passerelle séparé pour en exploiter une. Si votre organisation exploite déjà une [passerelle LLM](https://code.claude.com/docs/en/llm-gateway), Claude Code fonctionne aussi avec.

Cette page couvre :

- [Comment une passerelle s'insère entre Claude Code et votre fournisseur](#fonctionnement-dune-passerelle)
- [Choisir entre Claude apps gateway et une passerelle que vous exploitez déjà](#choisir-une-passerelle)
- [Comment les passerelles interagissent avec les abonnements claude.ai](#abonnements-et-passerelles)
- [Ce qui se configure séparément de la passerelle](#à-configurer-séparément-de-la-passerelle)

## Fonctionnement d'une passerelle

Le Claude Code de chaque développeur envoie ses requêtes à l'adresse de la passerelle et s'authentifie avec un identifiant émis par la passerelle.

La passerelle authentifie le développeur, applique les règles d'accès et de budget que vous avez configurées, et transmet la requête à votre fournisseur avec l'identifiant de l'organisation. Le fournisseur peut être l'API d'Anthropic ou un [fournisseur cloud](https://code.claude.com/docs/en/third-party-integrations) tel qu'Amazon Bedrock, Agent Platform de Google Cloud ou Microsoft Foundry ; c'est la configuration de la passerelle qui décide. Avec Claude apps gateway, ou une autre passerelle exposant un unique point de terminaison au format Anthropic, changer de fournisseur ne nécessite pas d'intervenir sur les machines des développeurs.

Schéma (décrit) : dans la zone « machines des développeurs », la CLI Claude Code et l'extension VS Code envoient leurs requêtes à l'adresse de la passerelle avec un identifiant propre à chaque développeur. Dans la zone « votre infrastructure », la passerelle gère l'authentification, le suivi de l'utilisation, les budgets et le routage, et transmet les requêtes avec l'identifiant de votre organisation. Dans la zone « fournisseurs de modèles », une flèche pleine mène au fournisseur configuré (illustré par l'API Anthropic) et des flèches en pointillés mènent à d'autres options (Amazon Bedrock, Google Cloud, Microsoft Foundry, à titre d'exemples).

Deux types d'identifiants sont en jeu :

- **Identifiant développeur** : chaque développeur détient le sien, émis par la passerelle. Il l'authentifie auprès de la passerelle et l'identifie dans le suivi de l'utilisation.
- **Identifiant fournisseur** : la passerelle détient un seul identifiant pour votre compte fournisseur, partagé par tout le trafic transmis.

## Choisir une passerelle

Claude Code fonctionne avec la passerelle d'Anthropic ou avec une passerelle que votre organisation exploite déjà.

### Claude apps gateway

Claude apps gateway est la passerelle auto-hébergée d'Anthropic, incluse dans le binaire `claude`.

- **Fournisseurs en amont** : Amazon Bedrock, Claude Platform on AWS, Google Cloud, Microsoft Foundry ou l'API Anthropic.
- **Connexion** : les développeurs se connectent avec le fournisseur d'identité (IdP) de votre entreprise via `/login`.
- **Gouvernance** : la passerelle applique l'accès aux modèles et les [réglages gérés (managed settings)](https://code.claude.com/docs/en/managed-settings) par groupe de l'IdP.
- **Télémétrie** : elle émet des métriques d'utilisation [OpenTelemetry Protocol (OTLP)](https://code.claude.com/docs/en/monitoring-usage) vers votre propre pile d'observabilité.

Comme elle est construite et testée avec chaque version de Claude Code, elle transmet les en-têtes et champs de requête que Claude Code envoie. Une passerelle maintenue séparément doit voir ses [règles de transmission mises à jour](https://code.claude.com/docs/en/llm-gateway-protocol#forward-as-open-lists) à mesure que ces en-têtes et champs évoluent à chaque version ; Claude apps gateway est publiée avec la CLI, il n'y a donc aucune liste à tenir à jour. Voir [Availability and limitations](https://code.claude.com/docs/en/claude-apps-gateway#availability-and-limitations) pour le petit ensemble de fonctionnalités qui se comportent différemment dans une session via passerelle.

La connexion à la passerelle est une étape SSO dans le navigateur, et il n'existe pas de flux par jeton de service : un pipeline CI sans développeur pour approuver la connexion ne peut donc pas s'authentifier par ce biais ; configurez ces pipelines directement auprès de votre fournisseur. Les sessions Agent SDK et les exécutions `claude -p` sur une machine où un développeur s'est connecté utilisent la session de passerelle de cette machine et sont soumises à ses politiques. Voir [CI pipelines and remote machines](https://code.claude.com/docs/en/claude-apps-gateway#ci-pipelines-and-remote-machines).

Voir [Claude apps gateway](https://code.claude.com/docs/en/claude-apps-gateway) pour la déployer.

### Autres passerelles

Si votre organisation exploite déjà une passerelle LLM ou une passerelle API, vous pouvez l'utiliser à la place. Anthropic n'approuve, ne maintient ni n'audite les autres produits de passerelle, et ne prend pas en charge le routage de Claude Code vers des modèles autres que Claude, quelle que soit la passerelle. Voir [Other LLM gateways](https://code.claude.com/docs/en/llm-gateway) pour la liste de contrôle de déploiement côté administrateur, ce qu'une passerelle doit implémenter et comment y diriger Claude Code.

## Abonnements et passerelles

Lorsque les développeurs se connectent via une passerelle avec un identifiant de passerelle, l'utilisation est facturée au compte fournisseur de votre organisation **aux tarifs API**, et leurs abonnements claude.ai ne sont ni utilisés ni débités. Définir [`ANTHROPIC_AUTH_TOKEN`](https://code.claude.com/docs/en/env-vars) pour une passerelle que vous exploitez, ou se connecter à une Claude apps gateway avec `/login`, désactive la connexion par abonnement pour cette session. Chaque requête transmise sous cet identifiant est facturée au compte associé à l'identifiant fournisseur de la passerelle.

**Exception** : définir uniquement `ANTHROPIC_BASE_URL`, sans identifiant de passerelle. Les requêtes passent toujours par la passerelle, mais une connexion claude.ai enregistrée reste l'identifiant actif : les limites d'utilisation et la facturation de l'abonnement s'appliquent alors. [Other LLM gateways](https://code.claude.com/docs/en/llm-gateway#subscriptions-and-gateways) décrit cette configuration et ce que la passerelle doit transmettre pour qu'elle fonctionne.

## À configurer séparément de la passerelle

Une passerelle route les requêtes API vers les modèles. Certaines choses que l'on pourrait attendre d'elle se configurent ailleurs :

- **Quel modèle répond** : choisissez le modèle avec la commande `/model` ou les [variables d'environnement de modèle](https://code.claude.com/docs/en/model-config#setting-your-model). La passerelle décide où vont les requêtes, pas quel modèle le développeur sélectionne. Claude apps gateway peut borner ce choix avec une liste autorisée `availableModels` par groupe, mais le développeur choisit toujours à l'intérieur de cette liste.
- **Autre trafic réseau** : Claude Code envoie lui-même les vérifications de version et les téléchargements directement à Anthropic, en dehors du chemin de la passerelle. Votre réseau doit toujours autoriser la sortie vers les [domaines requis](https://code.claude.com/docs/en/network-config), ou définissez [`CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`](https://code.claude.com/docs/en/env-vars) pour désactiver les flux facultatifs.
- **Télémétrie client** : Claude Code désactive ses analyses client à destination d'Anthropic lorsqu'une session se connecte à une Claude apps gateway. Pour désactiver aussi les analyses de démarrage antérieures à la connexion, fournissez [`DISABLE_TELEMETRY`](https://code.claude.com/docs/en/managed-settings#turn-telemetry-off-for-your-organization) dans les [réglages gérés côté client](https://code.claude.com/docs/en/claude-apps-gateway-config#client-side-managed-settings) sur chaque appareil.
- **Télémétrie client sur les autres passerelles** : l'envoi du flux facultatif de télémétrie client par Claude Code dépend de votre fournisseur ; le [tableau des comportements par défaut de la télémétrie](https://code.claude.com/docs/en/data-usage#default-behaviors-by-api-provider) couvre chaque cas.
- **Destinations de la télémétrie** : l'endroit où Claude Code envoie la télémétrie d'une session via passerelle dépend de la manière dont la session s'est connectée ; [What's enforced on developers](https://code.claude.com/docs/en/claude-apps-gateway#whats-enforced-on-developers) indique où vont les exports de chaque type de session.
- **Proxys HTTP d'entreprise** : un `HTTPS_PROXY` se place entre Claude Code et chaque serveur avec lequel il communique, passerelle comprise. Si votre réseau en exige un, [configurez le proxy](https://code.claude.com/docs/en/network-config) en plus de la passerelle. Pour une Claude apps gateway que vous hébergez, [la connexion vérifie que l'hôte du proxy se trouve lui aussi sur un réseau privé](https://code.claude.com/docs/en/claude-apps-gateway#prerequisites) ; si ce n'est pas le cas, ajoutez l'hôte de la passerelle à `NO_PROXY` pour que la CLI s'y connecte directement.

## Étapes suivantes

La page suivante dépend de qui exploite la passerelle. La passerelle d'Anthropic s'exécute depuis le binaire `claude` et dispose de son propre guide d'installation ; une passerelle déjà exploitée par votre organisation dispose d'un guide de compatibilité à suivre et d'une liste de contrôle de déploiement pour l'administrateur.

- [Claude apps gateway](https://code.claude.com/docs/en/claude-apps-gateway) : déployer la passerelle auto-hébergée d'Anthropic avec connexion SSO et télémétrie OTLP.
- [Other LLM gateways](https://code.claude.com/docs/en/llm-gateway) : ce qu'une passerelle déjà exploitée par votre organisation doit implémenter, et comment y diriger Claude Code.
- [Set up Claude Code for your organization](https://code.claude.com/docs/en/admin-setup) : les décisions de déploiement plus larges dont une passerelle n'est qu'une partie.
