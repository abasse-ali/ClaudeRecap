# Claude Code GitHub Actions

> Source : https://code.claude.com/docs/en/github-actions · [← Index](../31-index-documentation-officielle.md)

Exécuter Claude Code dans des workflows GitHub Actions pour répondre aux mentions `@claude`, automatiser des tâches et transformer des issues en pull requests.

[Claude Code GitHub Actions](https://github.com/anthropics/claude-code-action) est une GitHub Action qui exécute Claude Code dans les workflows de votre dépôt. Mentionnez `@claude` dans une pull request ou un commentaire d'issue pour que Claude analyse le code, implémente des modifications et pousse des commits. Vous pouvez aussi fournir à la Claude Code GitHub Action un prompt à exécuter automatiquement sur n'importe quel événement GitHub. Usages : transformer des issues en pull requests, corriger des bugs depuis un commentaire, automatiser des tâches récurrentes.

Plusieurs produits portent le nom Claude Code. Cette page couvre l'intégration de workflow `claude-code-action`, que vous configurez avec des fichiers de workflow dans votre dépôt. Produits apparentés :

- [Code Review](https://code.claude.com/docs/en/code-review) : revue automatique de chaque pull request, sans écrire de workflow.
- [Claude Code dans le cloud](https://code.claude.com/docs/en/claude-code-on-the-web) : sessions Claude Code exécutées sur une infrastructure cloud plutôt que sur votre machine.
- [Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview) : automatisation personnalisée en dehors de GitHub Actions. La Claude Code GitHub Action est construite sur le SDK.
- [GitHub Enterprise Server](https://code.claude.com/docs/en/github-enterprise-server) : Claude Code avec un GitHub auto-hébergé.

## Installation

Vous pouvez installer la Claude Code GitHub Action de deux façons :

- **Installation rapide** : lancez `/install-github-app` depuis Claude Code. Claude Code installe la GitHub App, ajoute votre secret d'authentification et prépare la pull request du workflow pour vous.
- **Installation manuelle** : installez l'application, ajoutez le secret et copiez vous-même le fichier de workflow dans votre dépôt. Choisissez cette voie si vous n'exécutez pas Claude Code localement, si la commande échoue, ou si vous voulez un contrôle total des fichiers de workflow.

Dans les deux cas, vous devez avoir un **accès administrateur** au dépôt.

### Installation rapide

`/install-github-app` ne fonctionne qu'avec les dépôts github.com. Si le remote git de votre dépôt est sur gitlab.com ou bitbucket.org, la commande affiche un avis et s'arrête au lieu de lancer l'installation. Pour exécuter Claude Code depuis des pipelines GitLab, voir [Claude Code GitLab CI/CD](https://code.claude.com/docs/en/gitlab-ci-cd).

Avant de commencer, installez la [GitHub CLI](https://cli.github.com) et authentifiez-la avec `gh auth login`. Claude Code vérifie sa présence et vous avertit si elle manque.

Ouvrez `claude` dans le dépôt à connecter, lancez `/install-github-app` et suivez les invites. Claude Code installe la Claude GitHub App, puis configure un secret d'authentification pour les workflows :

- Si Claude Code dispose déjà d'une clé API, il réutilise cette clé et propose de conserver le secret `ANTHROPIC_API_KEY` existant du dépôt s'il est déjà défini.
- Sinon, choisissez entre créer un jeton longue durée avec votre abonnement Claude ou coller une clé API.

Claude Code enregistre l'identifiant comme secret du dépôt, nommé `ANTHROPIC_API_KEY` pour une clé API ou `CLAUDE_CODE_OAUTH_TOKEN` pour un jeton d'abonnement.

Claude Code pousse ensuite une branche contenant les fichiers de workflow que vous avez sélectionnés, déjà configurés pour utiliser ce secret, et ouvre GitHub dans votre navigateur avec une pull request prête à être créée. Créez et fusionnez cette pull request : `@claude` fonctionne alors dans le dépôt.

Pour interrompre l'installation en cours de route, appuyez sur Esc. Une étape déjà en cours se termine, et aucune étape ultérieure ne démarre. Le message de clôture liste ce qui a déjà été fait dans le dépôt, comme une branche poussée ou un secret enregistré.

Si vous sélectionnez le workflow de revue, Claude publie chaque revue sur la pull request elle-même, sous forme de commentaire en ligne sur chaque problème trouvé, ou d'un seul commentaire de synthèse s'il n'en trouve aucun. Claude ignore certaines pull requests, comme les brouillons (drafts). L'[exemple de workflow de revue](#exécuter-un-skill) utilise le même skill et les liste. Avant la v2.1.229, Claude écrivait sa revue uniquement dans le journal d'exécution du workflow.

Pour mettre à jour un workflow de revue généré par une version antérieure, faites l'une des choses suivantes :

- Relancez `/install-github-app`. Lorsque le dépôt contient déjà un `claude.yml`, sélectionnez **Update workflow file with latest version**. Claude Code pousse des copies fraîches des fichiers de workflow sur une nouvelle branche et ouvre la pull request, comme lors d'une première installation.
- Ajoutez vous-même l'argument `--comment` et la ligne `claude_args` de l'[exemple de workflow de revue](#exécuter-un-skill) au fichier versionné, ce qui conserve vos autres modifications éventuelles.

Après l'installation de la GitHub App, Claude Code demande s'il faut poursuivre avec la configuration de GitHub Actions. Choisissez **Skip for now** pour vous arrêter avec seulement la GitHub App installée. Relancez `/install-github-app` plus tard pour terminer les étapes du workflow et du secret.

> **Note** :
> - Lorsque vous installez la GitHub App, vous lui accordez plusieurs permissions. Voir [Permissions de la GitHub App](#permissions-de-la-github-app) pour l'ensemble complet.
> - L'installation rapide fonctionne avec l'API Claude et les abonnements Claude. Si vous utilisez Amazon Bedrock, Agent Platform de Google Cloud ou Microsoft Foundry, voir [Use Claude Code GitHub Actions with cloud providers](https://code.claude.com/docs/en/github-actions-cloud-providers).

### Installation manuelle

Pour configurer la Claude Code GitHub Action sans lancer `/install-github-app`, installez l'application, ajoutez un secret et copiez vous-même un fichier de workflow :

**Étape 1 — Installer la Claude GitHub App**

Installez la [Claude GitHub App](https://github.com/apps/claude) sur votre dépôt. La Claude Code GitHub Action s'appuie sur trois des permissions de l'application :

- **Contents** : lecture et écriture, pour que Claude puisse modifier les fichiers du dépôt.
- **Issues** : lecture et écriture, pour que Claude puisse répondre aux issues.
- **Pull requests** : lecture et écriture, pour que Claude puisse créer des PR et pousser des modifications.

Lors de l'installation, vous accordez aussi des permissions utilisées par d'autres fonctionnalités Claude. Voir [Permissions de la GitHub App](#permissions-de-la-github-app) pour l'ensemble complet.

**Étape 2 — Ajouter un secret d'authentification**

Ajoutez l'un des secrets suivants à votre dépôt, selon votre mode d'authentification. Voir le guide GitHub [using secrets in GitHub Actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions).

- `ANTHROPIC_API_KEY` : une clé API Claude provenant de la [Claude Console](https://platform.claude.com).
- `CLAUDE_CODE_OAUTH_TOKEN` : un jeton OAuth qui s'authentifie avec votre abonnement Claude, disponible sur les forfaits Pro, Max, Team et Enterprise. Générez-le en lançant `claude setup-token` localement. Voir [Generate a long-lived token](https://code.claude.com/docs/en/authentication#generate-a-long-lived-token).

Dans les fichiers de workflow, passez le secret à l'entrée correspondante : `anthropic_api_key` pour une clé API, ou `claude_code_oauth_token` pour un jeton OAuth.

**Étape 3 — Copier le fichier de workflow**

Copiez [examples/claude.yml](https://github.com/anthropics/claude-code-action/blob/main/examples/claude.yml) dans le répertoire `.github/workflows/` de votre dépôt. Ce fichier est un workflow fonctionnel, pas seulement un exemple. Tel quel, Claude répond chaque fois que quelqu'un mentionne `@claude` dans une issue ou une pull request, en s'authentifiant avec le secret `ANTHROPIC_API_KEY`. Si vous avez ajouté `CLAUDE_CODE_OAUTH_TOKEN` à la place, remplacez la ligne `anthropic_api_key` du workflow par `claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}`.

> **Astuce** : après l'installation, testez la Claude Code GitHub Action en mentionnant `@claude` dans un commentaire d'issue ou de PR.

### Installation pour une organisation

Avec l'installation rapide ou manuelle, vous configurez un dépôt à la fois. Pour déployer la Claude Code GitHub Action à l'échelle d'une organisation :

- Installez la [Claude GitHub App](https://github.com/apps/claude) une seule fois au niveau de l'organisation, en choisissant tous les dépôts ou une liste sélectionnée.
- Stockez le secret d'authentification comme secret Actions au niveau de l'organisation, pour que chaque dépôt n'ait pas besoin de sa propre copie.
- Ajoutez le fichier de workflow à chaque dépôt qui doit exécuter la Claude Code GitHub Action, ou définissez le job une seule fois comme [workflow réutilisable](https://docs.github.com/en/actions/using-workflows/reusing-workflows) appelé par chaque dépôt.

Pour un secret partagé entre dépôts, authentifiez-vous avec une clé API de la [Claude Console](https://platform.claude.com) plutôt qu'avec un jeton OAuth, car un jeton OAuth est lié à l'abonnement de la personne qui a lancé `claude setup-token`.

Pour éviter complètement de stocker un secret longue durée, authentifiez-vous par **fédération d'identité de charge de travail** (workload identity federation) : la Claude Code GitHub Action échange le jeton GitHub OpenID Connect (OIDC) du workflow contre un accès à l'API Claude via un compte de service de la Claude Console. Définissez ces entrées :

- `anthropic_federation_rule_id` : l'ID de la règle de fédération, `fdrl_...`.
- `anthropic_organization_id` : l'ID de votre organisation Anthropic.
- `anthropic_service_account_id` : l'ID du compte de service, `svac_...`. Facultatif, puisque la règle de fédération créée dans la Console cible déjà un compte de service.
- `anthropic_workspace_id` : l'ID de l'espace de travail, `wrkspc_...`. Facultatif lorsque la règle de fédération cible un seul espace de travail.

Accordez au workflow la permission `id-token: write`, dont la Claude Code GitHub Action a besoin pour l'échange de fédération, même lorsque vous passez votre propre `github_token`. Voir le [guide d'installation de la Claude Code GitHub Action](https://github.com/anthropics/claude-code-action/blob/main/docs/setup.md) pour la configuration côté Console.

Pour les questions de traitement et de conservation des données dans une revue de sécurité, voir [data usage](https://code.claude.com/docs/en/data-usage) et [security](https://code.claude.com/docs/en/security).

### Désinstallation

Pour retirer la Claude Code GitHub Action, annulez chaque élément de l'installation qui s'applique à votre cas :

- **Fichiers de workflow** : supprimez de `.github/workflows/` les workflows qui utilisent `anthropics/claude-code-action`. Si vous avez utilisé l'installation rapide, cherchez `claude.yml` et, si vous avez sélectionné le workflow de revue, `claude-code-review.yml`. Une fois les workflows supprimés, la Claude Code GitHub Action ne s'exécute plus.
- **Secrets** : supprimez le secret `ANTHROPIC_API_KEY` ou `CLAUDE_CODE_OAUTH_TOKEN` du dépôt, ainsi que des secrets Actions au niveau de l'organisation si vous l'avez [partagé entre dépôts](#installation-pour-une-organisation). Supprimer un secret ne révoque pas l'identifiant qu'il contenait. Pour retirer définitivement une clé API, supprimez aussi la clé dans la [Claude Console](https://platform.claude.com).
- **GitHub App** : désinstallez la Claude GitHub App dans les réglages de votre dépôt ou de votre organisation, sous GitHub Apps, mais seulement si vous ne l'utilisez pas pour une autre fonctionnalité Claude, comme Code Review ou l'auto-fix web.

Si vous avez configuré un [fournisseur cloud](https://code.claude.com/docs/en/github-actions-cloud-providers), supprimez aussi les secrets du fournisseur, comme `AWS_ROLE_TO_ASSUME`, les secrets `GCP_*` ou les secrets `AZURE_*`, et désinstallez la GitHub App personnalisée ainsi que ses secrets `APP_ID` et `APP_PRIVATE_KEY`.

### Permissions de la GitHub App

La [Claude GitHub App](https://github.com/apps/claude) est partagée par toutes les fonctionnalités Claude qui s'intègrent à GitHub, dont la Claude Code GitHub Action, [Code Review](https://code.claude.com/docs/en/code-review) et l'[auto-fix des pull requests](https://code.claude.com/docs/en/claude-code-on-the-web#auto-fix-pull-requests) dans les sessions cloud. Une GitHub App possède un seul ensemble de permissions couvrant toutes ses fonctionnalités : cet ensemble inclut donc des permissions que la Claude Code GitHub Action n'utilise pas.

En installant l'application, vous accordez les permissions suivantes :

| Permission | Accès |
| - | - |
| Actions | Lecture et écriture |
| Administration | Lecture |
| Checks | Lecture et écriture |
| Contents | Lecture et écriture |
| Discussions | Lecture et écriture |
| Issues | Lecture et écriture |
| Members | Lecture |
| Merge queues | Lecture |
| Metadata | Lecture |
| Pull requests | Lecture et écriture |
| Repository hooks | Lecture et écriture |
| Statuses | Lecture |
| Workflows | Lecture et écriture |

L'ensemble de permissions peut aussi évoluer avant les fonctionnalités qui l'utilisent. Lorsque l'application demande une permission qu'elle n'avait pas auparavant, GitHub invite le propriétaire du compte à l'approuver (un propriétaire d'organisation pour une installation au niveau organisation), et l'installation garde ses anciennes permissions tant qu'il ne l'a pas fait. Par exemple, lorsque l'accès Actions passe de lecture à écriture, l'application peut relancer des workflows au lieu de seulement consulter les exécutions et les journaux : GitHub demande donc au propriétaire d'approuver ce changement.

En installant l'application, vous acceptez l'ensemble complet de ses permissions. GitHub ne permet pas d'en accepter seulement une partie. Si votre organisation n'autorise que les permissions utilisées par la Claude Code GitHub Action, créez plutôt une GitHub App personnalisée avec Contents, Issues et Pull requests, en suivant le [guide d'installation de la Claude Code GitHub Action](https://github.com/anthropics/claude-code-action/blob/main/docs/setup.md). Une application personnalisée ne couvre que la Claude Code GitHub Action : Code Review et l'auto-fix web exigent toujours l'application officielle.

Pour savoir comment la Claude Code GitHub Action limite ce que Claude peut faire avec ces permissions, voir la [documentation de sécurité](https://github.com/anthropics/claude-code-action/blob/main/docs/security.md).

## Modes interactif et automatisation

La Claude Code GitHub Action détermine son mode d'exécution à partir de la configuration de votre workflow :

- **Mode interactif** : lorsque le workflow ne fournit pas d'entrée `prompt`, Claude attend la phrase de déclenchement (`@claude` par défaut) dans un commentaire d'issue ou de pull request, dans une revue de pull request, ou dans le corps ou le titre d'une issue nouvellement ouverte, puis répond à cette demande. La progression et les résultats apparaissent dans un commentaire sur l'issue ou la PR déclencheuse.
- **Mode automatisation** : lorsque le workflow fournit une entrée `prompt`, Claude s'exécute sans attendre de mention, sous réserve uniquement des [contrôles sur qui peut déclencher des exécutions](#qui-peut-déclencher-des-exécutions). Par défaut, les résultats apparaissent dans le journal d'exécution du workflow plutôt que dans un commentaire. Claude peut publier sur l'issue ou la pull request lorsque le prompt le lui demande et qu'il dispose d'un outil capable de publier, comme dans l'[exemple de revue de code](#exécuter-un-skill).

### Qui peut déclencher des exécutions

Dans les deux modes, la Claude Code GitHub Action effectue deux contrôles sur l'acteur déclencheur avant que Claude ne démarre, et l'exécution échoue si l'un des deux la rejette :

- **Accès en écriture** : sur les événements d'issue et de pull request, l'utilisateur déclencheur doit avoir un accès en écriture au dépôt. Pour autoriser des utilisateurs précis sans accès en écriture, définissez `allowed_non_write_users` et passez votre propre entrée `github_token`. Les événements qu'aucun utilisateur n'est l'auteur, comme un déclencheur `schedule`, ignorent ce contrôle.
- **Acteur humain** : sur chaque événement, la Claude Code GitHub Action rejette un acteur bot sauf s'il figure dans `allowed_bots`, ce qui empêche des bots de déclencher Claude en boucle. Ce contrôle s'applique aussi aux exécutions planifiées, que GitHub attribue à un utilisateur du dépôt, généralement celui qui a modifié en dernier la planification `cron` du workflow. Si cet utilisateur est un bot, ajoutez-le à `allowed_bots`.

## Exemples d'utilisation

Le [répertoire d'exemples](https://github.com/anthropics/claude-code-action/tree/main/examples) contient des workflows prêts à l'emploi pour différents scénarios.

Les exemples de cette page utilisent l'authentification par clé API. Si vous vous authentifiez avec un abonnement Claude, remplacez dans chaque exemple la ligne `anthropic_api_key` par `claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}`.

### Répondre aux mentions @claude

Ce workflow exécute la Claude Code GitHub Action en mode interactif : Claude répond chaque fois que quelqu'un mentionne `@claude` dans un commentaire d'issue ou de PR.

```yaml theme={null}
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
jobs:
  claude:
    if: contains(github.event.comment.body, '@claude')
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      issues: write
      id-token: write
      actions: read
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 1
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

Les parties de ce workflow qui ne sont pas du code standard :

- `id-token: write` : requis pour l'authentification par défaut de la Claude Code GitHub Action via la GitHub App.
- `actions: read` : permet à Claude de lire les résultats de CI sur les PR.
- `actions/checkout` : donne à Claude une copie locale du dépôt sur laquelle travailler.
- `if` : évite de démarrer des runners pour des commentaires qui ne mentionnent pas `@claude`. La Claude Code GitHub Action vérifie aussi elle-même la phrase de déclenchement avant de répondre.

Une fois le workflow en place, mentionnez `@claude` dans n'importe quel commentaire d'issue ou de PR avec une demande :

```text wrap theme={null}
@claude implement this feature based on the issue description
@claude how should I implement user authentication for this endpoint?
@claude fix the TypeError in the user dashboard component
```

(Exemples : « implémente cette fonctionnalité d'après la description de l'issue », « comment implémenter l'authentification utilisateur pour ce point de terminaison ? », « corrige la TypeError dans le composant du tableau de bord utilisateur ».)

Claude répond dans un commentaire sur la même issue ou PR et le met à jour au fil de son travail.

### Exécuter un skill

L'entrée `prompt` accepte une invocation de [skill](https://code.claude.com/docs/en/skills) aussi bien que du texte brut :

- Pour un skill situé dans le répertoire `.claude/skills/` de votre dépôt, exécutez `actions/checkout` avant l'étape `anthropics/claude-code-action` pour que les fichiers du skill soient disponibles sur le runner, puis passez `/skill-name` comme `prompt`.
- Pour un skill fourni dans un [plugin](https://code.claude.com/docs/en/plugins/overview), installez le plugin avec les entrées `plugin_marketplaces` et `plugins`, puis passez le nom avec espace de noms `/plugin-name:skill-name` comme `prompt`. L'entrée `plugins` prend `plugin-name@marketplace-name`, où le nom de la marketplace provient du manifeste de la marketplace elle-même, et non de l'URL de son dépôt.

Le workflow suivant installe le plugin `code-review` et exécute son skill lorsqu'une pull request est ouverte, mise à jour, rouverte ou marquée prête pour la revue. Il exécute le même plugin que le workflow de revue de l'installation rapide. Utilisez un workflow de ce type lorsque vous voulez contrôler vous-même le prompt, le modèle et les déclencheurs. Pour des revues automatiques sans maintenir de fichier de workflow, voir [Code Review](https://code.claude.com/docs/en/code-review). Sur les dépôts publics, GitHub retient les secrets pour les exécutions déclenchées par des pull requests de forks : la revue ne s'exécute donc que sur les pull requests provenant de branches du même dépôt.

```yaml theme={null}
name: Code Review
on:
  pull_request:
    types: [opened, synchronize, ready_for_review, reopened]
jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: read
      issues: read
      id-token: write
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 1
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          plugin_marketplaces: "https://github.com/anthropics/claude-code.git"
          plugins: "code-review@claude-code-plugins"
          prompt: "/code-review:code-review --comment ${{ github.repository }}/pull/${{ github.event.pull_request.number }}"
          claude_args: '--allowedTools "mcp__github_inline_comment__create_inline_comment"'
```

Deux lignes de ce workflow déterminent où va la revue :

- **`--comment`** : Claude publie sa revue sur la pull request, sous forme de commentaire en ligne sur chaque problème trouvé, ou d'un seul commentaire de synthèse s'il n'en trouve aucun. Sans cet argument, Claude ne publie rien, et vous lisez les constats dans le journal d'exécution du workflow.
- **`claude_args`** : conservez cette ligne même si le frontmatter `allowed-tools` du skill nomme déjà le même outil, car la Claude Code GitHub Action ne démarre le serveur MCP qui publie les commentaires en ligne que lorsque `--allowedTools` dans `claude_args` le nomme.

Claude ignore les pull requests en brouillon et fermées, les pull requests qu'il juge ne pas nécessiter de revue (par exemple automatisées ou triviales), et les pull requests qui ont déjà un commentaire de Claude.

### Exécuter selon une planification

Avec une entrée `prompt`, la Claude Code GitHub Action s'exécute en mode automatisation sur n'importe quel événement GitHub, y compris une planification cron.

- Pour un prompt en texte brut, Claude n'a ni accès au shell ni à l'API GitHub tant que vous n'accordez pas les outils dont le prompt a besoin, avec `--allowedTools` dans `claude_args` ou une [règle `permissions.allow`](https://code.claude.com/docs/en/permissions#permission-rule-syntax) dans l'entrée `settings`.
- Si vous invoquez plutôt un skill, Claude peut utiliser les outils accordés par son [frontmatter `allowed-tools`](https://code.claude.com/docs/en/skills#pre-approve-tools-for-a-skill).
- GitHub n'exécute les workflows planifiés que depuis la branche par défaut et, dans les dépôts publics, désactive la planification après 60 jours sans activité sur le dépôt.

Ce workflow génère un rapport dans le journal d'exécution du workflow chaque jour à 09:00 UTC. Sa ligne `claude_args` [passe des arguments CLI](#passer-des-arguments-cli) qui sélectionnent le modèle et autorisent deux outils MCP GitHub. Claude lit les commits et les issues via l'API GitHub avec ces outils : vous pouvez donc omettre l'étape de checkout.

```yaml theme={null}
name: Daily Report
on:
  schedule:
    - cron: "0 9 * * *"
jobs:
  report:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      issues: read
      id-token: write
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: "Generate a summary of yesterday's commits and open issues"
          claude_args: |
            --model claude-opus-5-5
            --allowedTools "mcp__github__list_commits,mcp__github__list_issues"
```

(Le prompt demande : « Génère un résumé des commits d'hier et des issues ouvertes ».)

## Bonnes pratiques

### Définir les standards du projet dans CLAUDE.md

Créez un fichier `CLAUDE.md` à la racine du dépôt pour définir les conventions de style de code, les critères de revue, les règles propres au projet et les patterns préférés. Claude suit ces consignes lorsqu'il crée des PR et répond aux demandes. Voir la [documentation sur la mémoire](https://code.claude.com/docs/en/memory) pour les détails.

### Protéger vos identifiants

> **Avertissement** : ne commitez jamais de clés API ou de jetons OAuth directement dans votre dépôt. Stockez-les toujours comme GitHub Secrets et référencez-les dans les workflows, par exemple `anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}`.

N'accordez au workflow que les permissions dont il a besoin, et relisez les modifications de Claude avant de fusionner.

Pour les recommandations de sécurité, dont les permissions et l'authentification, voir la [documentation de sécurité de Claude Code Action](https://github.com/anthropics/claude-code-action/blob/main/docs/security.md).

### Maîtriser les coûts

Chaque exécution consomme deux types de ressources :

- **Minutes GitHub Actions** : la Claude Code GitHub Action s'exécute sur des runners hébergés par GitHub, qui consomment vos minutes GitHub Actions. Voir la [documentation de facturation de GitHub](https://docs.github.com/en/billing/managing-billing-for-your-products/managing-billing-for-github-actions/about-billing-for-github-actions) pour les tarifs et les limites de minutes.
- **Jetons API** : chaque interaction consomme des jetons selon la longueur des prompts et des réponses, la complexité de la tâche et la taille de la base de code. Voir la [page de tarification de Claude](https://claude.com/platform/api) pour les tarifs actuels. Si vous vous authentifiez avec un jeton OAuth, les exécutions utilisent votre abonnement Claude au lieu de la facturation API.

Vous pouvez réduire les deux types de coûts en donnant à Claude un contexte plus clair et en plafonnant la quantité de travail par exécution :

- Rédigez des demandes `@claude` précises pour que Claude ait besoin de moins de tours.
- Utilisez des modèles d'issue pour fournir le contexte dès le départ.
- Gardez votre `CLAUDE.md` concis, car Claude le lit à chaque exécution.
- Définissez `--max-turns` dans `claude_args` pour limiter les itérations.
- Définissez des délais d'expiration (timeouts) au niveau du workflow pour éviter les jobs incontrôlés.
- Utilisez les contrôles de concurrence de GitHub pour limiter les exécutions parallèles.

Pour le suivi de l'utilisation à l'échelle de l'organisation, voir le [tableau de bord d'analytique](https://code.claude.com/docs/en/analytics) et le [monitoring](https://code.claude.com/docs/en/monitoring-usage). Pour la mesure et la facturation de l'utilisation, voir [costs](https://code.claude.com/docs/en/costs).

## Utiliser un fournisseur cloud

Par défaut, la Claude Code GitHub Action appelle directement l'API Claude avec votre clé API ou votre jeton OAuth. Pour faire passer l'inférence par votre propre compte cloud, définissez l'entrée correspondant à votre fournisseur et suivez [Use Claude Code GitHub Actions with cloud providers](https://code.claude.com/docs/en/github-actions-cloud-providers) :

- **Amazon Bedrock** : `use_bedrock: "true"`
- **Agent Platform de Google Cloud** : `use_vertex: "true"`
- **Microsoft Foundry** : `use_foundry: "true"`

Avec ces trois fournisseurs, vous vous authentifiez par fédération d'identité OIDC au lieu d'une clé API Claude : vous ne stockez donc aucun identifiant cloud statique dans votre dépôt.

## Dépannage

### Claude ne répond pas aux commandes @claude

- Vérifiez que la GitHub App est installée sur le dépôt.
- Vérifiez que les workflows sont activés pour le dépôt.
- Assurez-vous que votre clé API ou votre jeton OAuth est défini dans les secrets du dépôt.
- Confirmez que le commentaire contient `@claude` en tant que mot complet, et non `/claude` ou `@claude-bot`.
- Confirmez que l'utilisateur qui commente a un accès en écriture au dépôt. Voir [Qui peut déclencher des exécutions](#qui-peut-déclencher-des-exécutions) pour les exceptions.

### La CI ne s'exécute pas sur les commits de Claude

- GitHub ne déclenche pas de workflows sur les commits faits avec le `GITHUB_TOKEN` par défaut. Si vous passez `github_token: ${{ secrets.GITHUB_TOKEN }}` à la Claude Code GitHub Action, retirez-le pour qu'elle s'authentifie en tant que Claude GitHub App, ou passez à la place un jeton d'application personnalisée.
- Vérifiez que les déclencheurs de votre workflow de CI incluent les événements produits par les pushes de Claude, comme `push` ou `pull_request`.

### Erreurs d'authentification

- Confirmez que la clé API ou le jeton OAuth est valide en le testant localement avec `claude` avant de déboguer le workflow.
- Pour Bedrock, Agent Platform et Foundry, voir la [section de dépannage](https://code.claude.com/docs/en/github-actions-cloud-providers#troubleshooting) de la page sur les fournisseurs cloud.

Pour d'autres solutions, voir la [FAQ](https://github.com/anthropics/claude-code-action/blob/main/docs/faq.md) de la Claude Code GitHub Action.

## Configuration avancée

### Paramètres de l'Action

Voici les entrées les plus utilisées. Chacune correspond à une clé `with:` dans l'étape `anthropics/claude-code-action`.

| Paramètre | Description | Requis |
| - | - | - |
| `prompt` | Instructions pour Claude, en texte brut ou sous forme d'invocation de [skill](https://code.claude.com/docs/en/skills). En son absence, Claude répond à la [phrase de déclenchement](#modes-interactif-et-automatisation) | Non |
| `claude_args` | Arguments CLI passés à Claude Code | Non |
| `anthropic_api_key` | Clé API Claude | Pour l'API Claude, sauf si vous utilisez `claude_code_oauth_token` ou la [fédération d'identité de charge de travail](#installation-pour-une-organisation). Non utilisé pour Bedrock, Agent Platform ou Foundry |
| `claude_code_oauth_token` | Jeton OAuth pour s'authentifier avec un abonnement Claude, généré avec `claude setup-token` | Non |
| `github_token` | Jeton pour les opérations GitHub. En son absence, la Claude Code GitHub Action s'authentifie en tant que Claude GitHub App | Non |
| `plugin_marketplaces` | Liste d'URL Git de marketplaces de plugins, séparées par des retours à la ligne | Non |
| `plugins` | Liste de noms de plugins à installer avant l'exécution, séparés par des retours à la ligne | Non |
| `settings` | Réglages de Claude Code, sous forme de chaîne JSON ou de chemin vers un fichier JSON de réglages | Non |
| `trigger_phrase` | Phrase de déclenchement à laquelle Claude répond. Par défaut : `@claude` | Non |
| `use_bedrock` | Utiliser Amazon Bedrock au lieu de l'API Claude | Non |
| `use_vertex` | Utiliser Agent Platform de Google Cloud au lieu de l'API Claude | Non |
| `use_foundry` | Utiliser Microsoft Foundry au lieu de l'API Claude | Non |

Pour la liste complète des entrées, voir la [référence de configuration](https://github.com/anthropics/claude-code-action/blob/main/docs/usage.md#inputs) de la Claude Code GitHub Action.

(Autres entrées mentionnées ailleurs sur la page : `allowed_non_write_users`, `allowed_bots`, `anthropic_federation_rule_id`, `anthropic_organization_id`, `anthropic_service_account_id`, `anthropic_workspace_id`.)

### Passer des arguments CLI

Le paramètre `claude_args` accepte n'importe quel [argument de la CLI Claude Code](https://code.claude.com/docs/en/cli-reference) :

```yaml theme={null}
claude_args: "--max-turns 5 --model claude-sonnet-5 --mcp-config /path/to/config.json"
```

Arguments courants :

- `--max-turns` : limite le nombre de tours de conversation.
- `--model` : modèle à utiliser, par exemple `claude-sonnet-5`. Sans cet argument, la Claude Code GitHub Action utilise le [modèle par défaut](https://code.claude.com/docs/en/model-config) de Claude Code.
- `--mcp-config` : chemin vers la [configuration MCP](https://code.claude.com/docs/en/mcp).
- `--allowedTools` : liste d'outils autorisés, séparés par des virgules. L'alias `--allowed-tools` fonctionne aussi.
- `--debug` : active la sortie de débogage.

## Migrer depuis la bêta

Si vos workflows référencent encore `anthropics/claude-code-action@beta`, passez-les en v1 :

1. Remplacez `@beta` par `@v1` dans la ligne `uses`.
2. Supprimez l'entrée `mode`, car la Claude Code GitHub Action [détecte désormais le mode automatiquement](#modes-interactif-et-automatisation).
3. Remplacez `direct_prompt` par `prompt`.
4. Déplacez les options CLI telles que `max_turns` et `model` dans `claude_args`. `custom_instructions` n'a pas de flag du même nom et devient `--append-system-prompt`.

Pour la correspondance complète des entrées et des exemples avant/après, voir le [guide de migration](https://github.com/anthropics/claude-code-action/blob/main/docs/migration-guide.md).

## Et ensuite

- [Use Claude Code GitHub Actions with cloud providers](https://code.claude.com/docs/en/github-actions-cloud-providers) : faire passer l'inférence par Amazon Bedrock, Agent Platform de Google Cloud ou Microsoft Foundry.
- [Référence de configuration](https://github.com/anthropics/claude-code-action/blob/main/docs/usage.md#inputs) : la liste complète des entrées de l'Action.
- [Répertoire d'exemples](https://github.com/anthropics/claude-code-action/tree/main/examples) : des workflows prêts à l'emploi pour d'autres scénarios.
- [Code Review](https://code.claude.com/docs/en/code-review) : revue automatique des pull requests sans maintenir de fichier de workflow.
