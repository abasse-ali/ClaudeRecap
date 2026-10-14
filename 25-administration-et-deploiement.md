# 25 — Administration, déploiement et environnements

[← Sommaire](README.md)

Ce fichier s'adresse aux administrateurs qui déploient Claude Code dans une organisation.

## 1. La carte des décisions

| Décision | Ce que l'on choisit |
|----------|---------------------|
| **Fournisseur d'API** | Où Claude Code s'authentifie et comment c'est facturé |
| **Distribution des réglages** | Comment la politique arrive sur les postes |
| **Ce qu'on impose** | Outils, commandes, intégrations autorisés |
| **Visibilité de l'usage** | Suivi des dépenses et de l'adoption |
| **Données** | Conservation, conformité |

## 2. Fournisseurs

| Fournisseur | À choisir quand… | Configuration |
|-------------|------------------|---------------|
| **Claude for Teams / Enterprise** (recommandé) | Claude Code + claude.ai sous un abonnement par siège, sans infrastructure | Connexion claude.ai, SSO, console d'admin |
| **Claude Console** | Approche API, paiement à l'usage | `claude auth login --console`, invitations |
| **Amazon Bedrock** | Hériter de la conformité et de la facturation AWS | `CLAUDE_CODE_USE_BEDROCK=1`, identifiants AWS (`aws configure`, SSO, clé d'accès, clé API Bedrock), profils d'inférence, `/setup-bedrock` |
| **Google Cloud Agent Platform** | Conformité et facturation GCP | `CLAUDE_CODE_USE_VERTEX=1`, projet, région, IAM, `/setup-vertex` |
| **Microsoft Foundry** | Conformité et facturation Azure | `CLAUDE_CODE_USE_FOUNDRY=1`, ressource, clé API ou Entra ID, RBAC |
| **Claude Platform on AWS** | Plateforme Claude dans AWS | `CLAUDE_CODE_USE_ANTHROPIC_AWS=1`, SigV4 ou clé d'espace de travail |

**Épingler les versions de modèles** chez les fournisseurs cloud (`ANTHROPIC_DEFAULT_OPUS_MODEL`, etc.) pour éviter les surprises lors des nouvelles versions.

### Disponibilité des fonctions selon le forfait (claude.ai)

| Fonction | Pro | Max | Team | Enterprise |
|----------|:---:|:---:|:----:|:----------:|
| Sessions cloud | ✓ | ✓ | ✓ | ✓ |
| Routines | ✓ | ✓ | ✓ | ✓ |
| Remote Control | ✓ | ✓ | Activé par l'admin | Activé par l'admin |
| Channels | ✓ | ✓ | Activé par l'admin | Activé par l'admin |
| Computer use | ✓ | ✓ | ✗ | ✗ |
| Dispatch (Desktop) | ✓ | ✓ | ✗ | ✗ |
| Code Review automatique | ✗ | ✗ | ✓ | ✓ |
| Artifacts | ✓ | ✓ | ✓ | ✓ |
| Tableau de bord analytique | ✗ | ✗ | ✓ | ✓ |
| API Enterprise Analytics | ✗ | ✗ | ✗ | ✓ |
| Réglages gérés par le serveur | ✗ | ✗ | ✓ | ✓ |
| SSO | ✗ | ✗ | ✓ | ✓ |
| SCIM, API de conformité, Zero Data Retention | ✗ | ✗ | ✗ | ✓ |

**Fonctions qui exigent un compte claude.ai** (pas de clé Console ni de fournisseur tiers) : sessions cloud et mobile, Slack, application Desktop, routines, ultrareview, Code Review, Remote Control, extension Chrome, computer use, artifacts, dictée vocale.

## 3. Distribuer les réglages gérés

| Mécanisme | Emplacement | Priorité |
|-----------|-------------|----------|
| **Réglages gérés par le serveur** | Console claude.ai › Organization settings › Claude Code › Managed settings (ou une Claude apps gateway) | La plus haute |
| **MDM / politique OS** | macOS : profil `com.anthropic.claudecode` ; Windows : `HKLM\SOFTWARE\Policies\ClaudeCode` (Jamf, Intune, GPO…) | Haute |
| **Fichier** | macOS `/Library/Application Support/ClaudeCode/managed-settings.json` ; Linux/WSL `/etc/claude-code/managed-settings.json` ; Windows `C:\Program Files\ClaudeCode\managed-settings.json` ; découpage possible dans `managed-settings.d/` | — |
| **Registre utilisateur Windows** | `HKCU\SOFTWARE\Policies\ClaudeCode` | La plus basse |

- Par défaut, seule la source la plus prioritaire s'applique ; `managedSourcesBehavior` permet de les combiner.
- Les réglages serveur peuvent déclencher une **boîte d'approbation de sécurité** chez l'utilisateur ; `forceRemoteSettingsRefresh` bloque le démarrage jusqu'à leur récupération.
- `policyHelper` : un exécutable qui calcule la politique au démarrage.
- Vérifier : `/status` (ligne *Setting sources*), `claude doctor`.
- Clés réservées aux réglages gérés : voir [19](19-reference-tous-les-reglages.md) (portée « Géré »).

## 4. Ce qu'on peut imposer

| Domaine | Moyens |
|---------|--------|
| Permissions | `permissions.allow/ask/deny`, `allowManagedPermissionRulesOnly`, `disableBypassPermissionsMode`, `defaultMode` |
| Mode auto | `disableAutoMode`, `autoMode.environment` (infrastructure de confiance : dépôts, buckets, domaines), règles de blocage/autorisation personnalisées, `claude auto-mode defaults/config` |
| Sandbox | `sandbox.*`, `allowManagedDomainsOnly`, `allowManagedReadPathsOnly`, `failIfUnavailable` |
| MCP | `managed-mcp.json` (contrôle exclusif), `managedMcpServers` (serveurs fournis), `allowedMcpServers` + `allowManagedMcpServersOnly` (catalogue approuvé), `deniedMcpServers`, `strictPluginOnlyCustomization.mcp` |
| Plugins et mods | `extraKnownMarketplaces`, `enabledPlugins`, `strictKnownMarketplaces`, `blockedMarketplaces`, `allowManagedModsOnly`, `prependPlugins`/`appendPlugins` |
| Hooks | `allowManagedHooksOnly`, `allowedHttpHookUrls` |
| Modèles | `availableModels`, `deniedModels`, `maxEffortLevel`, modèle et effort par défaut de l'organisation |
| Connexion | `forceLoginMethod`, `forceLoginOrgUUID`, `allowedProviders` |
| Versions | `requiredMinimumVersion`, `requiredMaximumVersion`, `autoUpdatesChannel`, `DISABLE_UPDATES` |
| Fonctions | `disableRemoteControl`, `channelsEnabled`, `enableArtifact`, `disableWorkflows`, `disableAgentView`, `disableDesktopLocalSessions`, `sshHostAllowlist`, `disableSideloadFlags`, `DISABLE_*_COMMAND`… |
| Instructions | `CLAUDE.md` d'organisation (fichier système ou `claudeMd`), skills d'entreprise |

### Modèles de restriction MCP

| Schéma | Effet |
|--------|-------|
| Désactiver MCP | `managed-mcp.json` vide |
| Déploiement fixe | `managed-mcp.json` avec vos serveurs, rien d'autre |
| Serveurs fournis | `managedMcpServers` : vos serveurs + ceux de l'utilisateur |
| Catalogue approuvé | `allowedMcpServers` + `allowManagedMcpServersOnly: true` |
| Plugins seulement | `strictPluginOnlyCustomization` |
| Liste blanche souple | `allowedMcpServers` seul |
| Liste noire | `deniedMcpServers` |

## 5. Passerelles (gateways)

- **Claude apps gateway** : passerelle d'Anthropic intégrée au binaire (`claude gateway --config gateway.yaml`). Connexion des développeurs par **SSO (OIDC)**, accès aux modèles par groupe, routage vers Bedrock, Claude Platform on AWS, Google Cloud, Foundry ou l'API Anthropic, **plafonds de dépense** (API d'administration, audit), télémétrie OTLP, réglages gérés poussés aux utilisateurs connectés. Stockage Postgres ; déploiement Docker Compose, AWS ou Google Cloud.
- **Autres passerelles LLM** (LiteLLM, etc.) : `ANTHROPIC_BASE_URL` + identifiant de passerelle par développeur (`ANTHROPIC_AUTH_TOKEN` ou `apiKeyHelper`) ; découverte des modèles (`CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY`) ; guide de compatibilité du protocole.

## 6. Réseau d'entreprise

- **Proxy** : `HTTPS_PROXY`, `HTTP_PROXY`, `NO_PROXY` (authentification basique dans l'URL).
- **Certificats** : magasin Mozilla intégré + magasin du système ; `CLAUDE_CODE_CERT_STORE`, `NODE_EXTRA_CA_CERTS` pour une CA d'entreprise.
- **mTLS** : `CLAUDE_CODE_CLIENT_CERT`, `CLAUDE_CODE_CLIENT_KEY`, `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE`.
- **Domaines requis** : API Anthropic, authentification, télémétrie (désactivable), mises à jour.
- **Launcher d'entreprise** : `CLAUDE_CODE_PROCESS_WRAPPER` / `processWrapper` pour faire passer les processus de fond par un lanceur obligatoire (script qui finit par `exec "$@"`).
- **Dev containers** : Feature officielle, pare-feu de sortie, utilisateur non-root.

## 7. Environnements cloud

### Environnements gérés par Anthropic

- Un environnement **Default** ; on en crée d'autres avec : nom, **accès réseau** (None, **Trusted** par défaut, Full, Custom), variables d'environnement, **secrets réseau** (injectés par le proxy pour certains hôtes), **script d'installation** (mis en cache).
- **Proxy GitHub** (le jeton n'entre jamais dans le conteneur) et **proxy de sécurité**.
- Environnements **partagés** par l'organisation (page Cloud environments de l'admin).
- Ce qui vient de votre configuration : `CLAUDE.md`, `.claude/settings.json`, skills, hooks et MCP du dépôt ; pas votre `~/.claude/`.
- Outils préinstallés (langages, gestionnaires de paquets, navigateurs, bases de données courantes…).

### Environnements auto-hébergés

Les sessions « cloud » tournent sur **votre infrastructure** grâce à des **runners** (`claude self-hosted-runner`) qui s'enregistrent dans un environnement (`ccpool_…`). Le code et l'exécution restent chez vous ; l'interface (claude.ai, Desktop, mobile) reste la même.

- Démarrage guidé : `claude self-hosted-runner setup` ; diagnostic : `doctor` ; runners à la demande : `orchestrator`.
- Personnalisation des sessions (images, outils, hooks semés par l'opérateur, permissions), tests de bout en bout, vérification de l'identité des sessions.
- Lancer une session dessus : `claude -p "…" --environment ccpool_… [--ref main]`.

## 8. Suivi de l'usage

| Outil | Contenu |
|-------|---------|
| **Analytics** (claude.ai/analytics/claude-code) | Utilisateurs actifs, lignes acceptées, taux d'acceptation, PR et lignes livrées avec Claude (intégration GitHub), classement, export CSV |
| **Console** (platform.claude.com/claude-code) | Usage, dépenses, équipes |
| **OpenTelemetry** | Métriques (sessions, lignes, PR, commits, coût, jetons, temps actif), événements (prompts, outils, requêtes API), traces ; destination verrouillable par l'admin |
| **Plafonds** | Limites de dépense (Console, gateway), crédits d'usage |
| **Coûts** | `/usage`, conseils de réduction (voir [17](17-sdk-entreprise-couts.md)) |

## 9. Sécurité et données

- **Fondations** : programme de sécurité d'Anthropic (SOC 2 Type 2, ISO 27001), architecture fondée sur les permissions, sandbox, limite du dossier de travail, liste blanche de commandes, mode Accept edits limité, protection contre l'injection de prompt (analyse du contexte, isolement des requêtes web, commandes à risque toujours soumises à approbation), vérification de la confiance des dossiers.
- **Politique de données** : consommateurs (Free/Pro/Max) selon leur choix d'entraînement ; commercial (Team, Enterprise, API) non utilisé pour l'entraînement par défaut ; programme Development Partner sur adhésion ; `/feedback` envoie la transcription choisie.
- **Zero Data Retention** (Enterprise éligible) : prompts et réponses non conservés pour l'inférence Claude Code ; certaines fonctions (cloud, artifacts…) suivent la politique standard.
- **HIPAA** (mode local) : configuration d'organisation, versions minimales (Claude Code v2.1.285+), connexion directe à l'API avec un compte Enterprise, gestion des données locales.
- **Rétention locale** : `cleanupPeriodDays`, `claude purge`.

### Cadre juridique

- **Conditions** : conditions commerciales (Team, Enterprise, API) ou conditions grand public (Free, Pro, Max) ; via Bedrock ou Google Cloud, votre accord commercial existant s'applique.
- **Intégrer Claude Code dans vos produits** (préinstallé, sandbox hébergée, infrastructure d'agents) : nécessite d'accepter les conditions commerciales.
- **Santé (BAA)** : un accord BAA signé avec Anthropic s'étend à Claude Code dans certaines configurations ; sous la configuration HIPAA, les sessions cloud, Remote Control, le mobile et les fournisseurs tiers/gateways ne sont pas couverts.
- Certifications et documents de sécurité sur le **Trust Center** (trust.anthropic.com) et le Transparency Hub ; signalement de vulnérabilités via **HackerOne**.

### Exemples de fichiers de réglages (doc officielle)

La doc fournit trois modèles prêts à copier : `~/.claude/settings.json` d'un développeur (modèle, effort, terminal, quelques autorisations), `.claude/settings.json` d'une équipe (permissions, hook qui bloque `rm`, marketplace, plugins) et `managed-settings.json` d'une organisation (`forceLoginMethod`, `forceLoginOrgUUID`, `availableModels`, `enforceAvailableModels`, règles imposées, télémétrie).

## 10. Adoption

- **Kit de communication** : annonce de lancement (version standard et version « sponsor exécutif »), messages de campagne, FAQ (« où va mon code ? »).
- **Kit des champions** : rôle des référents internes, quoi partager (techniques réutilisables), répondre aux questions, faire grandir l'usage.
- **Bonnes pratiques** : investir dans les `CLAUDE.md`, installation « en un clic », commencer par les questions sur le code et les petites corrections, épingler les modèles, politiques de sécurité gérées.
- `/team-onboarding` génère un guide d'intégration à partir de votre propre usage.
