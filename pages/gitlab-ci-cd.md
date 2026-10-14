# Claude Code avec GitLab CI/CD

> Source : https://code.claude.com/docs/en/gitlab-ci-cd · [← Index](../31-index-documentation-officielle.md)

*Découvrir comment intégrer Claude Code dans votre flux de développement avec GitLab CI/CD.*

> **Info** : Claude Code pour GitLab CI/CD est actuellement en **bêta**. Les fonctionnalités peuvent évoluer à mesure que l'expérience est affinée.
>
> Cette intégration est **maintenue par GitLab**. Pour le support, voir le [ticket GitLab](https://gitlab.com/gitlab-org/gitlab/-/issues/573776) correspondant.

> **Note** : cette intégration repose sur le [CLI Claude Code et l'Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview), ce qui permet d'utiliser Claude de façon programmatique dans vos jobs CI/CD et vos workflows d'automatisation personnalisés.

## Pourquoi utiliser Claude Code avec GitLab ?

* **Création instantanée de MR** : décrivez ce dont vous avez besoin, et Claude propose une MR (merge request) complète avec les modifications et leur explication.
* **Implémentation automatisée** : transformez des issues en code fonctionnel avec une seule commande ou mention.
* **Conscient du projet** : Claude suit vos consignes `CLAUDE.md` et les motifs de code existants.
* **Configuration simple** : ajoutez un seul job à `.gitlab-ci.yml` et une variable CI/CD masquée.
* **Prêt pour l'entreprise** : choisissez l'API Claude, Amazon Bedrock ou Google Cloud's Agent Platform pour répondre aux exigences de résidence des données et d'achats.
* **Sécurisé par défaut** : s'exécute sur vos runners GitLab, avec vos protections de branches et vos approbations.

## Fonctionnement

Claude Code utilise GitLab CI/CD pour exécuter des tâches d'IA dans des jobs isolés et renvoyer les résultats via des MR :

1. **Orchestration pilotée par les événements** : GitLab écoute les déclencheurs que vous avez choisis (par exemple un commentaire mentionnant `@claude` dans une issue, une MR ou un fil de revue). Le job collecte le contexte depuis le fil et le dépôt, construit des prompts à partir de ces entrées et lance Claude Code.

2. **Abstraction du fournisseur** : utilisez le fournisseur adapté à votre environnement :
   * API Claude (SaaS)
   * Amazon Bedrock (accès basé sur IAM, options inter-régions)
   * Google Cloud's Agent Platform (natif GCP, Workload Identity Federation)

3. **Exécution en bac à sable** : chaque interaction s'exécute dans un conteneur avec des règles strictes de réseau et de système de fichiers. Claude Code applique des permissions limitées à l'espace de travail pour restreindre les écritures. Chaque modification passe par une MR, de sorte que les relecteurs voient le diff et que les approbations s'appliquent toujours.

Choisissez des points de terminaison régionaux pour réduire la latence et respecter les exigences de souveraineté des données, tout en utilisant vos accords cloud existants.

## Que peut faire Claude ?

Dans un pipeline GitLab, Claude Code peut :

* créer et mettre à jour des MR à partir de descriptions d'issues ou de commentaires ;
* analyser des régressions de performance et proposer des optimisations ;
* implémenter des fonctionnalités directement dans une branche, puis ouvrir une MR ;
* corriger des bugs et régressions identifiés par des tests ou des commentaires ;
* répondre aux commentaires de suivi pour itérer sur les modifications demandées.

## Mise en place

### Configuration rapide

Le moyen le plus rapide de démarrer est d'ajouter un job minimal à votre `.gitlab-ci.yml` et de définir votre clé API comme variable masquée.

1. **Ajouter une variable CI/CD masquée**
   * Allez dans **Settings** → **CI/CD** → **Variables**
   * Ajoutez `ANTHROPIC_API_KEY` (masquée, protégée si nécessaire)

2. **Ajouter un job Claude à `.gitlab-ci.yml`**

```yaml
stages:
  - ai

claude:
  stage: ai
  image: node:24-alpine3.21
  # Adaptez les règles à la façon dont vous voulez déclencher le job :
  # - exécutions manuelles
  # - événements de merge request
  # - déclencheurs web/API quand un commentaire contient '@claude'
  rules:
    - if: '$CI_PIPELINE_SOURCE == "web"'
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  variables:
    GIT_STRATEGY: fetch
  before_script:
    - apk update
    - apk add --no-cache git curl bash
    - curl -fsSL https://claude.ai/install.sh | bash
    # L'installateur place claude dans ~/.local/bin, qui n'est pas dans le PATH de cette image
    - export PATH="$HOME/.local/bin:$PATH"
  script:
    # Optionnel : démarrer un serveur MCP GitLab si votre configuration en fournit un
    - /bin/gitlab-mcp-server || true
    # Utilisez les variables AI_FLOW_* lors d'une invocation via des déclencheurs web/API avec des charges de contexte
    - echo "$AI_FLOW_INPUT for $AI_FLOW_CONTEXT on $AI_FLOW_EVENT"
    - >
      claude
      -p "${AI_FLOW_INPUT:-'Review this MR and implement the requested changes'}"
      --permission-mode acceptEdits
      --allowedTools "Bash Read Edit Write mcp__gitlab"
      --debug
```

Après avoir ajouté le job et votre variable `ANTHROPIC_API_KEY`, testez en lançant le job manuellement depuis **CI/CD** → **Pipelines**, ou déclenchez-le depuis une MR pour que Claude propose des mises à jour dans une branche et ouvre une MR si nécessaire.

> **Note** : pour utiliser Amazon Bedrock ou Google Cloud's Agent Platform au lieu de l'API Claude, voir la section [Utilisation avec Amazon Bedrock et Google Cloud](#utilisation-avec-amazon-bedrock-et-google-cloud) ci-dessous pour l'authentification et la configuration de l'environnement.

### Configuration manuelle (recommandée en production)

Si vous préférez une configuration plus maîtrisée ou si vous avez besoin de fournisseurs entreprise :

1. **Configurer l'accès au fournisseur** :
   * **API Claude** : créez `ANTHROPIC_API_KEY` et stockez-la comme variable CI/CD masquée.
   * **Amazon Bedrock** : **configurez GitLab** → **AWS OIDC** et créez un rôle IAM pour Amazon Bedrock.
   * **Google Cloud's Agent Platform** : **configurez Workload Identity Federation pour GitLab** → **GCP**.

2. **Ajouter des identifiants de projet pour les opérations de l'API GitLab** :
   * Utilisez `CI_JOB_TOKEN` par défaut, ou créez un Project Access Token avec la portée `api`.
   * Stockez-le sous `GITLAB_ACCESS_TOKEN` (masquée) si vous utilisez un PAT.

3. **Ajouter le job Claude à `.gitlab-ci.yml`** : utilisez le job de la [configuration rapide](#configuration-rapide) pour l'API Claude, ou un job propre au fournisseur parmi les [exemples de configuration](#exemples-de-configuration).

4. **(Optionnel) Activer les déclencheurs par mention** :
   * Ajoutez un webhook de projet pour « Comments (notes) » vers votre écouteur d'événements (si vous en utilisez un).
   * Faites appeler par l'écouteur l'API de déclenchement de pipeline avec des variables comme `AI_FLOW_INPUT` et `AI_FLOW_CONTEXT` lorsqu'un commentaire contient `@claude`.

## Exemples de cas d'usage

### Transformer des issues en MR

Dans un commentaire d'issue :

```text
@claude implement this feature based on the issue description
```

(« implémente cette fonctionnalité d'après la description de l'issue »)

Claude analyse l'issue et la base de code, écrit les modifications dans une branche et ouvre une MR pour relecture.

### Obtenir de l'aide à l'implémentation

Dans une discussion de MR :

```text
@claude suggest a concrete approach to cache the results of this API call
```

(« propose une approche concrète pour mettre en cache les résultats de cet appel d'API »)

Claude propose des modifications, ajoute le code avec la mise en cache appropriée et met à jour la MR.

### Corriger rapidement des bugs

Dans un commentaire d'issue ou de MR :

```text
@claude fix the TypeError in the user dashboard component
```

(« corrige la TypeError dans le composant du tableau de bord utilisateur »)

Claude localise le bug, implémente un correctif et met à jour la branche ou ouvre une nouvelle MR.

## Utilisation avec Amazon Bedrock et Google Cloud

Pour les environnements d'entreprise, vous pouvez exécuter Claude Code entièrement sur votre infrastructure cloud, avec la même expérience développeur.

### Onglet « Amazon Bedrock »

#### Prérequis

Avant de configurer Claude Code avec Amazon Bedrock, il vous faut :

1. Un compte AWS avec un accès Amazon Bedrock aux modèles Claude souhaités.
2. GitLab configuré comme fournisseur d'identité OIDC dans AWS IAM.
3. Un rôle IAM avec les permissions Amazon Bedrock et une politique de confiance restreinte à votre projet/vos refs GitLab.
4. Des variables CI/CD GitLab pour l'endossement du rôle :
   * `AWS_ROLE_TO_ASSUME` (ARN du rôle)
   * `AWS_REGION` (région Amazon Bedrock)

#### Instructions de mise en place

Configurez AWS pour autoriser les jobs GitLab CI à endosser un rôle IAM via OIDC (aucune clé statique).

**Configuration requise :**

1. Activez Amazon Bedrock et demandez l'accès aux modèles Claude ciblés.
2. Créez un fournisseur OIDC IAM pour GitLab s'il n'existe pas déjà.
3. Créez un rôle IAM approuvé par le fournisseur OIDC GitLab, restreint à votre projet et à vos refs protégées.
4. Attachez des permissions au moindre privilège pour les API d'invocation d'Amazon Bedrock.

Utilisez l'[exemple de job Amazon Bedrock](#exemples-de-configuration) pour échanger, à l'exécution, le jeton OIDC du job contre des identifiants AWS temporaires.

### Onglet « Google Cloud's Agent Platform »

#### Prérequis

Avant de configurer Claude Code avec Google Cloud's Agent Platform, il vous faut :

1. Un projet Google Cloud avec :
   * l'API Google Cloud's Agent Platform activée ;
   * Workload Identity Federation configurée pour faire confiance à l'OIDC de GitLab.
2. Un compte de service dédié avec uniquement les rôles Google Cloud's Agent Platform nécessaires.
3. Des variables CI/CD GitLab :
   * `GCP_WORKLOAD_IDENTITY_PROVIDER` (nom de ressource du fournisseur sans le préfixe `//iam.googleapis.com/`, par exemple `projects/123456789/locations/global/workloadIdentityPools/my-pool/providers/my-provider`)
   * `GCP_SERVICE_ACCOUNT` (e-mail du compte de service)
   * `GCP_PROJECT_ID` (ID du projet Google Cloud)

#### Instructions de mise en place

Configurez Google Cloud pour autoriser les jobs GitLab CI à emprunter l'identité d'un compte de service via Workload Identity Federation.

**Configuration requise :**

1. Activez l'API IAM Credentials, l'API STS et l'API Google Cloud's Agent Platform.
2. Créez un Workload Identity Pool et un fournisseur pour l'OIDC GitLab.
3. Créez un compte de service dédié avec les rôles Google Cloud's Agent Platform.
4. Accordez au principal WIF la permission d'emprunter l'identité du compte de service.

Utilisez l'[exemple de job Agent Platform](#exemples-de-configuration) pour vous authentifier sans stocker de clés.

## Exemples de configuration

Voici des extraits prêts à l'emploi que vous pouvez adapter à votre pipeline.

### Exemple de job Amazon Bedrock (OIDC)

**Prérequis :**

* Amazon Bedrock activé avec accès au(x) modèle(s) Claude choisi(s).
* OIDC GitLab configuré dans AWS avec un rôle qui fait confiance à votre projet et à vos refs GitLab.
* Rôle IAM avec permissions Amazon Bedrock (moindre privilège recommandé).

**Variables CI/CD requises :**

* `AWS_ROLE_TO_ASSUME` : ARN du rôle IAM pour l'accès à Amazon Bedrock.
* `AWS_REGION` : région Amazon Bedrock (par exemple `us-west-2`).

GitLab génère le jeton OIDC du job à partir du bloc `id_tokens:` et l'expose sous `GITLAB_OIDC_TOKEN`. Réglez `aud` sur la valeur d'audience configurée sur le fournisseur d'identité OIDC IAM dans AWS, par exemple l'URL de votre instance GitLab.

```yaml
stages:
  - ai

claude-bedrock:
  stage: ai
  image: node:24-alpine3.21
  rules:
    - if: '$CI_PIPELINE_SOURCE == "web"'
  id_tokens:
    GITLAB_OIDC_TOKEN:
      aud: https://gitlab.example.com
  before_script:
    - apk add --no-cache bash curl jq git aws-cli
    - curl -fsSL https://claude.ai/install.sh | bash
    # L'installateur place claude dans ~/.local/bin, qui n'est pas dans le PATH de cette image
    - export PATH="$HOME/.local/bin:$PATH"
    # Échanger le jeton OIDC du job contre des identifiants AWS
    - export AWS_WEB_IDENTITY_TOKEN_FILE="/tmp/oidc_token"
    - printf "%s" "$GITLAB_OIDC_TOKEN" > "$AWS_WEB_IDENTITY_TOKEN_FILE"
    - >
      aws sts assume-role-with-web-identity
      --role-arn "$AWS_ROLE_TO_ASSUME"
      --role-session-name "gitlab-claude-$(date +%s)"
      --web-identity-token "file://$AWS_WEB_IDENTITY_TOKEN_FILE"
      --duration-seconds 3600 > /tmp/aws_creds.json
    - export AWS_ACCESS_KEY_ID="$(jq -r .Credentials.AccessKeyId /tmp/aws_creds.json)"
    - export AWS_SECRET_ACCESS_KEY="$(jq -r .Credentials.SecretAccessKey /tmp/aws_creds.json)"
    - export AWS_SESSION_TOKEN="$(jq -r .Credentials.SessionToken /tmp/aws_creds.json)"
  script:
    - /bin/gitlab-mcp-server || true
    - >
      claude
      -p "${AI_FLOW_INPUT:-'Implement the requested changes and open an MR'}"
      --permission-mode acceptEdits
      --allowedTools "Bash Read Edit Write mcp__gitlab"
      --debug
  variables:
    AWS_REGION: "us-west-2"
    CLAUDE_CODE_USE_BEDROCK: "1"
```

> **Note** : les ID de modèles Amazon Bedrock comportent des préfixes propres à la région (par exemple `us.anthropic.claude-sonnet-4-6`). Transmettez le modèle souhaité via la configuration de votre job ou le prompt, si votre workflow le permet.

### Exemple de job Agent Platform (Workload Identity Federation)

**Prérequis :**

* API Google Cloud's Agent Platform activée dans votre projet GCP.
* Workload Identity Federation configurée pour faire confiance à l'OIDC GitLab.
* Un compte de service avec les permissions Google Cloud's Agent Platform.

**Variables CI/CD requises :**

* `GCP_WORKLOAD_IDENTITY_PROVIDER` : nom de ressource du fournisseur sans le préfixe `//iam.googleapis.com/`, par exemple `projects/123456789/locations/global/workloadIdentityPools/my-pool/providers/my-provider`.
* `GCP_SERVICE_ACCOUNT` : e-mail du compte de service.
* `GCP_PROJECT_ID` : ID du projet Google Cloud.
* `CLOUD_ML_REGION` : région Google Cloud's Agent Platform (par exemple `us-east5`).

GitLab génère le jeton OIDC du job à partir du bloc `id_tokens:` et l'expose sous `GITLAB_OIDC_TOKEN`. Réglez `aud` sur la valeur d'audience configurée sur le fournisseur du Workload Identity Pool, par exemple l'URL de votre instance GitLab. Le job écrit le jeton dans un fichier, et l'entrée `credential_source` de la configuration d'identifiants indique aux bibliothèques d'authentification de Google d'aller le lire à cet endroit. Définir `GOOGLE_APPLICATION_CREDENTIALS` sur le fichier de configuration d'identifiants le rend disponible pour Claude Code via les [Application Default Credentials](https://code.claude.com/docs/en/google-vertex-ai#3-configure-gcp-credentials).

```yaml
stages:
  - ai

claude-vertex:
  stage: ai
  image: gcr.io/google.com/cloudsdktool/google-cloud-cli:slim
  rules:
    - if: '$CI_PIPELINE_SOURCE == "web"'
  id_tokens:
    GITLAB_OIDC_TOKEN:
      aud: https://gitlab.example.com
  before_script:
    - apt-get update && apt-get install -y git && apt-get clean
    - curl -fsSL https://claude.ai/install.sh | bash
    # L'installateur place claude dans ~/.local/bin, qui n'est pas dans le PATH de cette image
    - export PATH="$HOME/.local/bin:$PATH"
    # Écrire le jeton OIDC du job là où credential_source l'attend
    - printf "%s" "$GITLAB_OIDC_TOKEN" > /tmp/oidc_token
    # Écrire la configuration d'identifiants WIF dans un fichier (aucune clé téléchargée)
    - |
      cat > /tmp/cred.json <<EOF
      {
        "type": "external_account",
        "audience": "//iam.googleapis.com/${GCP_WORKLOAD_IDENTITY_PROVIDER}",
        "subject_token_type": "urn:ietf:params:oauth:token-type:jwt",
        "token_url": "https://sts.googleapis.com/v1/token",
        "credential_source": {
          "file": "/tmp/oidc_token"
        },
        "service_account_impersonation_url": "https://iamcredentials.googleapis.com/v1/projects/-/serviceAccounts/${GCP_SERVICE_ACCOUNT}:generateAccessToken"
      }
      EOF
    # Exposer les identifiants à Claude Code via les Application Default Credentials
    - export GOOGLE_APPLICATION_CREDENTIALS=/tmp/cred.json
    # Authentifier le CLI gcloud avec la même configuration d'identifiants
    - gcloud auth login --cred-file=/tmp/cred.json
    - gcloud config set project "$GCP_PROJECT_ID"
  script:
    - /bin/gitlab-mcp-server || true
    - >
      CLOUD_ML_REGION="${CLOUD_ML_REGION:-us-east5}"
      claude
      -p "${AI_FLOW_INPUT:-'Review and update code as requested'}"
      --permission-mode acceptEdits
      --allowedTools "Bash Read Edit Write mcp__gitlab"
      --debug
  variables:
    CLOUD_ML_REGION: "us-east5"
    CLAUDE_CODE_USE_VERTEX: "1"
    ANTHROPIC_VERTEX_PROJECT_ID: "$GCP_PROJECT_ID"
```

> **Note** : avec Workload Identity Federation, vous n'avez pas besoin de stocker de clés de compte de service. Utilisez des conditions de confiance propres au dépôt et des comptes de service au moindre privilège.

## Bonnes pratiques

### Configuration de CLAUDE.md

Créez un fichier `CLAUDE.md` à la racine du dépôt pour définir les standards de code, les critères de revue et les règles propres au projet. Claude lit ce fichier pendant les exécutions et suit vos conventions lorsqu'il propose des modifications.

### Considérations de sécurité

**Ne commitez jamais de clés API ni d'identifiants cloud dans votre dépôt.** Utilisez toujours les variables CI/CD GitLab :

* Ajoutez `ANTHROPIC_API_KEY` comme variable masquée (et protégez-la si nécessaire).
* Utilisez l'OIDC propre au fournisseur lorsque c'est possible (pas de clés à longue durée de vie).
* Limitez les permissions des jobs et le trafic réseau sortant.
* Relisez les MR de Claude comme celles de n'importe quel autre contributeur.

### Optimiser les performances

* Gardez `CLAUDE.md` ciblé et concis.
* Fournissez des descriptions d'issues/MR claires pour réduire les itérations.
* Mettez en cache npm et les installations de paquets sur les runners lorsque c'est possible.

### Coûts CI

Avec Claude Code dans GitLab CI/CD, tenez compte des coûts associés :

* **Temps de GitLab Runner** :
  * Claude s'exécute sur vos runners GitLab et consomme des minutes de calcul.
  * Voir la facturation des runners de votre forfait GitLab pour les détails.

* **Coûts d'API** :
  * Chaque interaction avec Claude consomme des tokens selon la taille du prompt et de la réponse.
  * La consommation de tokens varie selon la complexité de la tâche et la taille de la base de code.
  * Voir la [tarification Anthropic](https://platform.claude.com/docs/en/about-claude/pricing) pour les détails.

* **Conseils d'optimisation des coûts** :
  * Utilisez des commandes `@claude` précises pour réduire les tours inutiles.
  * Définissez des valeurs appropriées pour `--max-turns` et pour le `timeout` du job.
  * Limitez la concurrence pour contrôler les exécutions parallèles.

## Dépannage

### Claude ne répond pas aux commandes @claude

* Vérifiez que votre pipeline est bien déclenché (manuellement, par un événement de MR, ou via un écouteur/webhook d'événements de notes).
* Assurez-vous que votre `ANTHROPIC_API_KEY` ou les variables du fournisseur cloud sont présentes.
* Vérifiez que le commentaire contient `@claude` (et non `/claude`) et que votre déclencheur par mention est configuré.

### Le job ne peut pas écrire de commentaires ni ouvrir de MR

* Assurez-vous que `CI_JOB_TOKEN` dispose de permissions suffisantes sur le projet, ou utilisez un Project Access Token avec la portée `api`.
* Vérifiez que l'outil `mcp__gitlab` est activé dans `--allowedTools`.
* Confirmez que le job s'exécute dans le contexte de la MR ou dispose d'assez de contexte via les variables `AI_FLOW_*`.

### Erreurs d'authentification

* **Pour l'API Claude** : confirmez que `ANTHROPIC_API_KEY` est valide et non expirée.
* **Pour Amazon Bedrock ou Google Cloud's Agent Platform** : vérifiez la configuration OIDC/WIF, l'emprunt d'identité du rôle et les noms des secrets ; confirmez la disponibilité de la région et du modèle.

## Configuration avancée

### Paramètres et variables courants

Contrôlez les exécutions de Claude Code dans vos jobs avec ces flags CLI, mots-clés GitLab et variables :

* `-p` : fournir des instructions en ligne, par exemple `claude -p "Review this MR"`.
* `--max-turns` : limiter le nombre d'itérations aller-retour.
* `timeout` : limiter la durée totale d'exécution du job avec le mot-clé `timeout` de niveau job de GitLab, par exemple `timeout: 30m`.
* `ANTHROPIC_API_KEY` : requise pour l'API Claude (non utilisée pour Amazon Bedrock ni Google Cloud's Agent Platform).
* Environnement propre au fournisseur : `AWS_REGION`, variables de projet/région pour Google Cloud's Agent Platform.

> **Note** : les flags et paramètres exacts peuvent varier selon la version de `@anthropic-ai/claude-code`. Lancez `claude --help` dans votre job pour voir les options prises en charge.

### Personnaliser le comportement de Claude

Vous pouvez guider Claude de deux façons principales :

1. **CLAUDE.md** : définissez les standards de code, les exigences de sécurité et les conventions du projet. Claude le lit pendant les exécutions et suit vos règles.
2. **Prompts personnalisés** : transmettez des instructions propres à la tâche via `-p` dans le job. Utilisez des prompts différents pour des jobs différents (par exemple revue, implémentation, refactorisation).
