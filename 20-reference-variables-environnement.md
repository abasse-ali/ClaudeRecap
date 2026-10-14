# 20 — Référence : toutes les variables d'environnement

[← Sommaire](README.md)

Claude Code lit **près de 400 variables d'environnement** (v2.1.299). On les définit dans le shell (`export VAR=valeur` avant `claude`) ou dans la clé `env` d'un fichier de réglages (elles s'appliquent alors à chaque session et à ses sous-processus).

- Quand une variable est à la fois dans le shell et dans un bloc `env`, **la valeur du fichier de réglages l'emporte** dans la plupart des sessions ; entre fichiers, la priorité habituelle s'applique (géré > … > utilisateur).
- Pour neutraliser une variable héritée du shell, mettez-la à `""` dans `env`.
- Les variables du shell sont lues au démarrage ; celles de `env` sont réappliquées quand le fichier change.
- `--model`/`/model` priment sur `ANTHROPIC_MODEL`, mais `CLAUDE_CODE_EFFORT_LEVEL` prime sur `--effort`/`/effort`.

> Valeurs booléennes : en général `1`/`true` active et `0`/`false` désactive. **Exceptions** : `DISABLE_TELEMETRY`, `DISABLE_ERROR_REPORTING`, `FALLBACK_FOR_ALL_PRIMARY_MODELS`, `IS_DEMO`, `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` et `CLAUDE_CODE_TMUX_TRUECOLOR` s'activent avec **n'importe quelle valeur non vide** (même `0`) : il faut supprimer la variable pour revenir en arrière.

## 1. Authentification et fournisseurs

| Variable | Rôle |
|----------|------|
| `ANTHROPIC_API_KEY` | Clé API (en-tête `X-Api-Key`) ; prime sur l'abonnement même si vous êtes connecté |
| `ANTHROPIC_AUTH_TOKEN` | Valeur personnalisée de l'en-tête `Authorization` (préfixée par `Bearer `) |
| `ANTHROPIC_PROFILE` | Profil Anthropic à utiliser (créé par `ant auth login` ou connexion Console) |
| `ANTHROPIC_BASE_URL` | Point d'accès de l'API (proxy, gateway) ; désactive par défaut la recherche d'outils MCP hors hôte Anthropic |
| `ANTHROPIC_CUSTOM_HEADERS` | En-têtes supplémentaires (`Nom: Valeur`, un par ligne) |
| `ANTHROPIC_BETAS` | En-têtes `anthropic-beta` supplémentaires |
| `ANTHROPIC_FEDERATION_RULE_ID`, `ANTHROPIC_ORGANIZATION_ID`, `ANTHROPIC_WORKSPACE_ID` | Workload Identity Federation (règle, organisation, espace de travail) |
| `CLAUDE_CODE_OAUTH_TOKEN` | Jeton d'accès OAuth claude.ai (alternative à `/login` pour SDK et automatisation ; généré par `claude setup-token`) |
| `CLAUDE_CODE_OAUTH_REFRESH_TOKEN`, `CLAUDE_CODE_OAUTH_SCOPES` | Jeton de rafraîchissement OAuth et ses scopes (connexion sans navigateur) |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | Intervalle de rafraîchissement des identifiants produits par `apiKeyHelper` |
| `CLAUDE_CODE_PROVIDER_MANAGED_BY_HOST` | Défini par une plateforme hôte qui gère le fournisseur à la place de Claude Code |
| **Amazon Bedrock** | |
| `CLAUDE_CODE_USE_BEDROCK` | Utiliser Amazon Bedrock |
| `AWS_BEARER_TOKEN_BEDROCK` | Clé API Bedrock |
| `ANTHROPIC_BEDROCK_BASE_URL` | Point d'accès Bedrock personnalisé |
| `ANTHROPIC_BEDROCK_REGION_PREFIX` | Préfixe de profil d'inférence inter-régions (`us`, `eu`, `apac`, `jp`, `au`, `global`) |
| `ANTHROPIC_BEDROCK_SERVICE_TIER` | Niveau de service (`default`, `flex`, `priority`) |
| `CLAUDE_CODE_USE_MANTLE`, `ANTHROPIC_BEDROCK_MANTLE_BASE_URL`, `CLAUDE_CODE_SKIP_MANTLE_AUTH` | Point d'accès Bedrock Mantle (activer, URL, ignorer l'authentification) |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH` | Ne pas s'authentifier auprès d'AWS (gateway qui signe elle-même) |
| `CLAUDE_CODE_AWS_CHAIN_RESOLVE_TIMEOUT_MS` | Délai d'obtention des identifiants AWS par la chaîne par défaut |
| `CLAUDE_CODE_SKIP_AWS_CRED_CACHE` | Ne pas mettre en cache les identifiants AWS |
| `CLAUDE_CODE_DISABLE_BEDROCK_CONTENT_TYPE_DEFAULT`, `CLAUDE_CODE_DISABLE_BEDROCK_CONTENT_TYPE_GUARD` | Compatibilité avec des réponses Bedrock au `Content-Type` atypique |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION` | Région AWS du modèle de classe Haiku |
| **Claude Platform on AWS** | |
| `CLAUDE_CODE_USE_ANTHROPIC_AWS` | Utiliser Claude Platform on AWS |
| `ANTHROPIC_AWS_API_KEY`, `ANTHROPIC_AWS_BASE_URL`, `ANTHROPIC_AWS_WORKSPACE_ID` | Clé, URL et espace de travail |
| `CLAUDE_CODE_SKIP_ANTHROPIC_AWS_AUTH` | Ignorer l'authentification côté client |
| **Google Cloud Agent Platform (Vertex AI)** | |
| `CLAUDE_CODE_USE_VERTEX` | Utiliser Google Cloud |
| `ANTHROPIC_VERTEX_PROJECT_ID`, `ANTHROPIC_VERTEX_BASE_URL` | Projet GCP, URL personnalisée |
| `CLAUDE_CODE_SKIP_VERTEX_AUTH` | Ignorer l'authentification Google |
| `VERTEX_REGION_CLAUDE_<MODÈLE>` | Région par modèle : `3_5_HAIKU`, `3_5_SONNET`, `3_7_SONNET`, `4_0_OPUS`, `4_0_SONNET`, `4_1_OPUS`, `4_5_OPUS`, `4_5_SONNET`, `4_6_OPUS`, `4_6_SONNET`, `4_7_OPUS`, `4_8_OPUS`, `5_OPUS`, `5_SONNET`, `5_5_OPUS`, `5_5_SONNET`, `FABLE_5`, `FABLE_5_1`, `HAIKU_4_5`, `HAIKU_5_5` |
| **Microsoft Foundry** | |
| `CLAUDE_CODE_USE_FOUNDRY` | Utiliser Microsoft Foundry |
| `ANTHROPIC_FOUNDRY_API_KEY`, `ANTHROPIC_FOUNDRY_AUTH_TOKEN` | Clé API / jeton Bearer (Entra) |
| `ANTHROPIC_FOUNDRY_RESOURCE`, `ANTHROPIC_FOUNDRY_BASE_URL` | Nom de ressource ou URL complète |
| `CLAUDE_CODE_SKIP_FOUNDRY_AUTH` | Ignorer l'authentification Azure |
| **Accès au modèle** | |
| `CLAUDE_CODE_SKIP_MODEL_ACCESS_MEMORY` | Ne pas mémoriser les modèles inaccessibles (Bedrock, Google Cloud) |
| `CLAUDE_CODE_DISABLE_MODEL_ACCESS_FALLBACK` | Ne pas basculer vers un modèle plus ancien en cas de perte d'accès |

## 2. Modèles, effort et réflexion

| Variable | Rôle |
|----------|------|
| `ANTHROPIC_MODEL` | Modèle à utiliser |
| `ANTHROPIC_DEFAULT_MODEL` | Modèle de démarrage des nouvelles sessions |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` / `_SONNET_MODEL` / `_HAIKU_MODEL` / `_FABLE_MODEL` | ID vers lequel pointe chaque alias (`opusplan` utilise Opus en plan, Sonnet sinon ; Haiku sert aussi aux tâches de fond) |
| `ANTHROPIC_DEFAULT_<FAMILLE>_MODEL_NAME` / `_DESCRIPTION` / `_SUPPORTED_CAPABILITIES` | Nom, description et capacités (`effort,thinking`…) affichés dans `/model` pour un modèle épinglé |
| `ANTHROPIC_CUSTOM_MODEL_OPTION` (+ `_NAME`, `_DESCRIPTION`, `_SUPPORTED_CAPABILITIES`) | Ajouter une entrée personnalisée au sélecteur `/model` |
| `ANTHROPIC_SMALL_FAST_MODEL` | Obsolète : modèle de classe Haiku pour les tâches de fond |
| `CLAUDE_CODE_EFFORT_LEVEL` | Niveau d'effort (`low`…`max` ou `auto`) |
| `CLAUDE_CODE_ALWAYS_ENABLE_EFFORT` | Toujours envoyer le paramètre d'effort (modèles inconnus derrière une gateway) |
| `MAX_THINKING_TOKENS` | Budget fixe de réflexion étendue |
| `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` | Désactiver la réflexion adaptative (Opus/Sonnet 4.6) au profit d'un budget fixe |
| `CLAUDE_CODE_DISABLE_THINKING` | Ne jamais envoyer le paramètre `thinking` (proxys incompatibles) |
| `DISABLE_INTERLEAVED_THINKING` | Ne pas envoyer l'en-tête de réflexion entrelacée |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | Nombre max de jetons de sortie |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT` | Désactiver la fenêtre de 1 million de jetons |
| `CLAUDE_CODE_MAX_CONTEXT_TOKENS` | Taille de contexte supposée pour le modèle |
| `CLAUDE_CODE_DISABLE_LEGACY_MODEL_REMAP` | Ne pas remapper Opus 4/4.1 vers l'Opus actuel |
| `CLAUDE_CODE_DISABLE_FAST_MODE` | Désactiver le mode rapide |
| `CLAUDE_CODE_SKIP_FAST_MODE_NETWORK_ERRORS`, `CLAUDE_CODE_SKIP_FAST_MODE_ORG_CHECK` | Contourner la vérification de disponibilité du mode rapide (réseaux filtrés) |
| `CLAUDE_CODE_DISABLE_ADVISOR_TOOL` | Désactiver l'advisor |
| `CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK` | Pas de changement automatique de modèle quand un classifieur de sécurité signale une requête |
| `FALLBACK_FOR_ALL_PRIMARY_MODELS` | Arrêter les tentatives sur surcharge répétée pour tous les modèles sans secours |
| `CLAUDE_CODE_SIMPLE_SYSTEM_PROMPT` | Prompt système complet ou abrégé |
| `CLAUDE_CODE_DISABLE_STRUCTURED_OUTPUTS` | Ne pas envoyer le champ de sortie structurée (gateways incompatibles) |
| `MAX_STRUCTURED_OUTPUT_RETRIES` | Tentatives quand la sortie ne respecte pas `--json-schema` |
| `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY` (+ `_TIMEOUT_MS`) | Remplir `/model` depuis l'endpoint `/v1/models` d'une gateway |

## 3. Réseau, délais et nouvelles tentatives

| Variable | Rôle |
|----------|------|
| `HTTP_PROXY`, `HTTPS_PROXY`, `NO_PROXY` | Proxy HTTP/HTTPS et exceptions |
| `CLAUDE_CODE_PROXY_RESOLVES_HOSTS` | Laisser le proxy résoudre les noms DNS |
| `CLAUDE_CODE_CERT_STORE` | Sources de certificats CA (`bundled`, `system`) |
| `CLAUDE_CODE_CLIENT_CERT`, `CLAUDE_CODE_CLIENT_KEY`, `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE` | Authentification mTLS |
| `CLAUDE_CODE_DISABLE_MTLS_RELOAD_ON_STALE_CONNECTION` | Ne pas relire le certificat mTLS après une erreur de connexion |
| `API_TIMEOUT_MS` | Délai des requêtes API (10 min par défaut) |
| `API_FORCE_IDLE_TIMEOUT` | Délai d'inactivité du flux de réponse (5 min ; `0` = aucun) |
| `CLAUDE_STREAM_IDLE_TIMEOUT_MS`, `CLAUDE_BYTE_STREAM_IDLE_TIMEOUT_MS`, `CLAUDE_STREAM_FIRST_BYTE_TIMEOUT_MS` | Délais des chiens de garde du streaming |
| `CLAUDE_ENABLE_STREAM_WATCHDOG`, `CLAUDE_ENABLE_BYTE_WATCHDOG`, `CLAUDE_ENABLE_BYTE_WATCHDOG_BEDROCK` | Activer/désactiver ces chiens de garde |
| `CLAUDE_CODE_MAX_RETRIES` | Nombre de nouvelles tentatives (10 par défaut, max 15) |
| `CLAUDE_CODE_OVERLOADED_RETRY_BASE_DELAY_MS`, `CLAUDE_CODE_OVERLOADED_RETRY_MAX_DELAY_MS` | Délais de backoff sur erreur 529 (surcharge) |
| `CLAUDE_CODE_RETRY_WATCHDOG` (+ `_MAX_WAIT_MS`) | Réessayer indéfiniment les erreurs 429/529 (CI, évaluations) |
| `CLAUDE_CODE_NONSTREAMING_TIMEOUT_RETRIES` | Tentatives d'une requête non-streaming expirée |
| `CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK` | Pas de repli non-streaming en cas d'échec |
| `CLAUDE_CODE_EXTRA_BODY` | Objet JSON fusionné dans chaque requête API |
| `CLAUDE_CODE_GZIP_REQUEST_BODIES` | Compression gzip des requêtes (`0` pour désactiver) |
| `CLAUDE_CODE_GATEWAY_HINT_HEADERS` | Envoyer les en-têtes indicatifs (`x-claude-code-request-class`…) à un proxy |
| `CLAUDE_CODE_PROPAGATE_TRACEPARENT` | Propager le contexte de trace W3C vers un proxy |
| `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` | Retirer les en-têtes et champs bêta |
| `CLAUDE_CODE_ATTRIBUTION_HEADER` | `0` = retirer le bloc d'attribution (version, empreinte) du prompt système |

## 4. Cache de prompt

| Variable | Rôle |
|----------|------|
| `CLAUDE_CODE_PROMPT_CACHE_TTL` | Durée du cache de la conversation principale (`5m` ou `1h`) |
| `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL` | Idem pour sous-agents, workflows, requêtes de fond |
| `ENABLE_PROMPT_CACHING_1H` | Demander un cache d'1 h (clé API, Bedrock, Google Cloud, Foundry) ; `ENABLE_PROMPT_CACHING_1H_BEDROCK` est obsolète |
| `FORCE_PROMPT_CACHING_5M` | Forcer 5 minutes |
| `DISABLE_PROMPT_CACHING` | Désactiver le cache pour tous les modèles |
| `DISABLE_PROMPT_CACHING_FABLE` / `_OPUS` / `_SONNET` / `_HAIKU` | Désactiver pour une famille |

## 5. Contexte, compaction et mémoire

| Variable | Rôle |
|----------|------|
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | Seuil de compaction en jetons (100 000 à 1 000 000) |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | Pourcentage de la fenêtre déclenchant la compaction (pour compacter plus tôt) |
| `DISABLE_AUTO_COMPACT` | Désactiver la compaction automatique (et en veille) |
| `DISABLE_COMPACT` | Désactiver toute compaction, même `/compact` |
| `CLAUDE_CODE_IDLE_COMPACT_MIN_TOKENS` | Taille minimale pour la compaction en veille (200 000) |
| `CLAUDE_CODE_DISABLE_UNKNOWN_MODEL_WINDOW_ENFORCEMENT` | Pas de compaction anticipée pour un modèle inconnu |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | Désactiver (ou forcer avec `0`) la mémoire automatique |
| `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | Ne charger aucun `CLAUDE.md` |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` | Charger les `CLAUDE.md` et règles des dossiers `--add-dir` |
| `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS` | Retirer les instructions git intégrées et l'instantané `git status` |
| `CLAUDE_CODE_DISABLE_ATTACHMENTS` | Envoyer les `@mentions` en texte brut |
| `CLAUDE_CODE_DISABLE_FILE_MENTIONS` | Ne pas développer les `@mentions` : Claude reçoit le chemin et lit lui-même |
| `SLASH_COMMAND_TOOL_CHAR_BUDGET` | Budget de caractères de la liste des skills |
| `CLAUDE_CODE_MAX_MCP_DESCRIPTION_LENGTH` | Longueur max des descriptions d'outils MCP envoyées |
| `CLAUDE_CODE_TRANSCRIPT_LOCAL_GC` | Limiter la taille des transcriptions des longues sessions `-p`/SDK |

## 6. Outils

| Variable | Rôle |
|----------|------|
| `BASH_DEFAULT_TIMEOUT_MS` | Délai par défaut d'une commande (2 min) |
| `BASH_MAX_TIMEOUT_MS` | Délai maximum demandable (10 min) |
| `BASH_MAX_OUTPUT_LENGTH` | Caractères de sortie relus (30 000 ; max 150 000) |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | Revenir au dossier du projet après chaque commande |
| `CLAUDE_CODE_SHELL` | Shell utilisé (chemin vers `bash` ou `zsh`) |
| `CLAUDE_CODE_SHELL_PREFIX` | Préfixe qui enveloppe toutes les commandes lancées (journalisation, audit) |
| `CLAUDE_ENV_FILE` | Script exécuté avant chaque commande Bash (variables persistantes, ex. virtualenv) |
| `CLAUDE_CODE_GIT_BASH_PATH` | Windows : chemin de Git Bash |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | Activer/désactiver l'outil PowerShell |
| `CLAUDE_CODE_POWERSHELL_RESPECT_EXECUTION_POLICY` | Respecter la politique d'exécution PowerShell |
| `CLAUDE_CODE_DISABLE_WINDOWS_SHELL_LAUNCHER` | Lancer PowerShell sans passer par `cmd.exe` |
| `CLAUDE_CODE_TOOL_MEMORY_LIMIT` | Linux/WSL : plafond mémoire des commandes (ex. `4G`) |
| `CLAUDE_CODE_TOOL_MEMORY_CGROUP_EXCLUDE` | Processus exclus de ce plafond (`mcp`, `lsp`…) |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS` | Limite de jetons d'une lecture de fichier (25 000) |
| `CLAUDE_CODE_GLOB_HIDDEN`, `CLAUDE_CODE_GLOB_NO_IGNORE`, `CLAUDE_CODE_GLOB_TIMEOUT_SECONDS` | Fichiers cachés, `.gitignore` et délai de l'outil Glob |
| `USE_BUILTIN_RIPGREP` | `0` = utiliser le `rg` du système |
| `CLAUDE_CODE_USE_NATIVE_FILE_SEARCH` | Chercher commandes/agents/styles sans ripgrep |
| `CLAUDE_CODE_DISABLE_WEB_FETCH` | Désactiver WebFetch |
| `CLAUDE_CODE_WEBFETCH_CACHE_TTL_MS`, `CLAUDE_CODE_WEBFETCH_DEADLINE_MS` | Cache (15 min) et délai de téléchargement de WebFetch |
| `CLAUDE_CODE_MAX_WEB_SEARCHES_PER_SESSION`, `CLAUDE_CODE_WEB_SEARCH_REFILLS_PER_HOUR` | Quota et recharge des recherches web |
| `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY` | Outils en lecture seule et sous-agents en parallèle (10) |
| `CLAUDE_CODE_PERFORCE_MODE` | Protection d'écriture compatible Perforce (`p4 edit`) |
| `CLAUDE_CODE_ENABLE_FINE_GRAINED_TOOL_STREAMING` | Streaming des entrées d'outils pendant leur génération |
| `CLAUDE_CODE_ENABLE_TASKS`, `CLAUDE_CODE_ENABLE_TODO_TOOLS` | Choix des outils de liste de tâches / les activer sur tous les modèles |
| `CLAUDE_CODE_TASK_LIST_ID` | Partager une liste de tâches entre sessions |
| `TASK_MAX_OUTPUT_LENGTH` | Retiré |

## 7. Agents, workflows et arrière-plan

| Variable | Rôle |
|----------|------|
| `CLAUDE_CODE_SUBAGENT_MODEL` | Modèle par défaut des sous-agents, coéquipiers et agents de workflow |
| `CLAUDE_CODE_SUBAGENT_MODEL_FORCE` | Imposer un modèle unique à tous ces agents |
| `CLAUDE_CODE_WORKFLOW_SUBAGENT_MODEL` | Modèle unique pour les agents de workflow |
| `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` | Sous-agents simultanés max (20) |
| `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` | Profondeur d'imbrication des sous-agents (3) |
| `CLAUDE_CODE_DISABLE_EXPLORE_PLAN_AGENTS` | Retirer les agents Explore et Plan |
| `CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS` | Retirer tous les agents intégrés (`-p`/SDK) |
| `CLAUDE_CODE_FORK_SUBAGENT` | Mode fork (sous-agents forkés) en `-p`/SDK |
| `CLAUDE_ASYNC_AGENT_STALL_TIMEOUT_MS` | Délai de blocage des sous-agents (10 min) |
| `CLAUDE_AUTO_BACKGROUND_TASKS` | Passer automatiquement les longs sous-agents en arrière-plan (~2 min) |
| `CLAUDE_CODE_WORKER_CHECKIN_SCHEDULE` | Rappels pour vérifier les sous-agents de fond |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | Désactiver toutes les tâches de fond (`Ctrl+B`, auto-arrière-plan…) |
| `CLAUDE_CODE_DISABLE_BG_SHELL_PRESSURE_REAP` | Ne pas tuer les shells de fond sous pression mémoire |
| `CLAUDE_CODE_DISABLE_BG_EXIT_HANDOFF` | Arrêter le travail de fond quand le superviseur s'arrête |
| `CLAUDE_DISABLE_ADOPT` | Ne pas transférer le travail en cours quand on passe la session en arrière-plan |
| `CLAUDE_CODE_DISABLE_AGENT_VIEW` | Désactiver agents d'arrière-plan et agent view |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | Activer les équipes d'agents |
| `CLAUDE_CODE_TEAM_TEARDOWN_PARK_TIMEOUT_MS` | Attente de fin d'équipe en `-p` |
| `CLAUDE_CODE_DISABLE_WORKFLOWS` | Désactiver les workflows |
| `CLAUDE_CODE_WORKFLOW_MAX_CONCURRENT_AGENTS` | Agents simultanés par workflow (16 par défaut, 1–256) |
| `CLAUDE_CODE_WORKFLOW_PREFIX_STAGGER_MS` | Étalement des premières requêtes d'un workflow (cache) |
| `CLAUDE_CODE_PRINT_BG_WAIT_CEILING_MS` | Attente max du travail de fond après le dernier tour en `-p` |
| `CLAUDE_CODE_BG_TASKS_REPORT_RUNNING` | Statut rapporté à l'hôte pendant le travail de fond |
| `CLAUDE_CODE_GOAL_CHECKIN_MINUTES` | Rappel de vérification d'un `/goal` en attente (30 min) |
| `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` | Blocages consécutifs max d'un hook Stop (8) |
| `CLAUDE_CODE_DISABLE_CRON` | Désactiver `/loop` et les tâches planifiées |

## 8. MCP

| Variable | Rôle |
|----------|------|
| `MCP_TIMEOUT` | Délai de démarrage d'un serveur (30 s) |
| `MCP_TOOL_TIMEOUT` | Délai d'exécution d'un outil |
| `CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT` | Délai sans réponse ni progression |
| `CLAUDE_CODE_MCP_AUTO_BACKGROUND_MS` | Passage en arrière-plan d'un appel long (2 min ; `0` = jamais) |
| `MAX_MCP_OUTPUT_TOKENS` | Jetons max d'une réponse d'outil MCP (25 000) |
| `ENABLE_TOOL_SEARCH` | Recherche d'outils MCP (chargement différé) |
| `MCP_CONNECTION_NONBLOCKING`, `MCP_CONNECT_TIMEOUT_MS`, `CLAUDE_CODE_MCP_STARTUP_WAIT_MS` | Attendre ou non les serveurs au démarrage |
| `MCP_SERVER_CONNECTION_BATCH_SIZE`, `MCP_REMOTE_SERVER_CONNECTION_BATCH_SIZE` | Connexions parallèles locales (3) / distantes (20) |
| `MCP_DISCOVERY_CACHE` (+ `_TTL_S`, `_MAX_STALE_S`, `_STRIKES`) | Cache de découverte des serveurs distants |
| `MCP_CLIENT_SECRET`, `MCP_OAUTH_CALLBACK_PORT` | Secret client OAuth, port de rappel fixe |
| `MCP_SDK_GENERATION`, `MCP_PROTOCOL_NEGOTIATION` | Moteur client MCP (v1/v2) et négociation de protocole |
| `CLAUDE_CODE_MCP_ALLOWLIST_ENV` | Lancer les serveurs stdio avec un environnement minimal |
| `ENABLE_CLAUDEAI_MCP_SERVERS` | `false` = ne pas récupérer les connecteurs claude.ai |
| `CLAUDE_AGENT_SDK_MCP_NO_PREFIX` | Pas de préfixe `mcp__serveur__` pour les serveurs créés par le SDK |

## 9. Plugins et skills

| Variable | Rôle |
|----------|------|
| `CLAUDE_CODE_PLUGIN_DIRS` | Dossiers de plugins à charger (comme `--plugin-dir`) |
| `CLAUDE_CODE_PLUGIN_CACHE_DIR` | Dossier racine des plugins |
| `CLAUDE_CODE_PLUGIN_SEED_DIR` | Dossiers de plugins pré-remplis en lecture seule (images de conteneur) |
| `CLAUDE_CODE_PLUGIN_DIR_WATCH` | Recharger un mod quand ses fichiers changent |
| `CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS` | Délai de clonage d'une marketplace (2 min) |
| `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` | Garder la copie existante si la mise à jour échoue |
| `CLAUDE_CODE_PLUGIN_PREFER_HTTPS` | Cloner en HTTPS plutôt qu'en SSH |
| `CLAUDE_CODE_DISABLE_OFFICIAL_MARKETPLACE_AUTOINSTALL` | Ne pas ajouter la marketplace officielle |
| `FORCE_AUTOUPDATE_PLUGINS` | Mettre à jour les plugins même si l'auto-update est désactivé |
| `CLAUDE_CODE_SYNC_PLUGIN_INSTALL` (+ `_TIMEOUT_MS`) | En `-p`, attendre l'installation des plugins |
| `CLAUDE_CODE_ENABLE_BACKGROUND_PLUGIN_REFRESH` | Rafraîchir les plugins entre les tours en `-p` |
| `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS` | Retirer les skills et workflows intégrés |
| `CLAUDE_CODE_DISABLE_POLICY_SKILLS` | Ne pas charger les skills gérés du système |
| `CLAUDE_CODE_SYNC_SKILLS` (+ `_WAIT_TIMEOUT_MS`, `_INSTALL_TIMEOUT_MS`) | En `-p`, télécharger et attendre les skills du compte claude.ai |
| `CLAUDE_CODE_NEW_INIT` | `/init` interactif (CLAUDE.md, skills, hooks) |

## 10. Interface, terminal et accessibilité

| Variable | Rôle |
|----------|------|
| `CLAUDE_CODE_NO_FLICKER` | Activer le rendu plein écran |
| `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN` | Forcer le rendu classique |
| `CLAUDE_CODE_ALT_SCREEN_FULL_REPAINT` | Redessiner tout l'écran à chaque image |
| `CLAUDE_CODE_DISABLE_VIRTUAL_SCROLL` | Désactiver le défilement virtuel |
| `CLAUDE_CODE_DISABLE_MOUSE`, `CLAUDE_CODE_DISABLE_MOUSE_CLICKS` | Désactiver la souris / seulement les clics |
| `CLAUDE_CODE_SCROLL_SPEED` | Multiplicateur de la molette (jusqu'à 20) |
| `CLAUDE_CODE_NATIVE_CURSOR` | Curseur natif du terminal |
| `CLAUDE_CODE_ACCESSIBILITY` | Curseur natif visible (loupes d'écran) |
| `CLAUDE_AX_SCREEN_READER`, `CLAUDE_AX_PREPARK_MS`, `CLAUDE_AX_STARTUP_QUIET_MS` | Mode lecteur d'écran et ses délais |
| `CLAUDE_CODE_SYNTAX_HIGHLIGHT` | `false` = pas de coloration dans les diffs |
| `CLAUDE_CODE_FORCE_STRIKETHROUGH`, `CLAUDE_CODE_FORCE_SYNC_OUTPUT`, `CLAUDE_CODE_FORCE_TERMINAL_IMAGES` | Forcer barré, sortie synchronisée, images (mods) |
| `CLAUDE_CODE_TMUX_TRUECOLOR` | Couleurs 24 bits dans tmux |
| `FORCE_HYPERLINK` | Forcer (`1`) ou couper (`0`) les liens cliquables |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE` | Ne pas modifier le titre du terminal |
| `CLAUDE_CODE_HIDE_CWD` | Cacher le dossier dans le logo de démarrage (captures d'écran) |
| `IS_DEMO` | Mode démo : masque e-mail et organisation |
| `CLAUDE_CODE_BS_AS_CTRL_BACKSPACE` | Interprétation de la touche `^H` |
| `CLAUDE_CODE_NONBLOCKING_STDOUT` | Sortie non bloquante (terminal qui ne lit plus) |
| `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` | Suggestions de prompt |
| `CLAUDE_CODE_ENABLE_AWAY_SUMMARY` | Forcer le récapitulatif au retour on/off |
| `CLAUDE_AFK_TIMEOUT_MS`, `CLAUDE_AFK_COUNTDOWN_MS` | Continuation automatique d'une question sans réponse et son compte à rebours |
| `CLAUDE_CLIENT_PRESENCE_FILE` | Fichier indiquant que vous êtes présent (écran déverrouillé) |
| `CLAUDE_CODE_USER_DIALOG_TIMEOUT_MS` | Délai des boîtes transmises à Remote Control / SDK |
| `CLAUDE_CODE_DISABLE_NOTIFICATION_PRESENCE_CHECK` | Notifier même quand vous êtes devant le terminal |
| `CLAUDE_CODE_AUTO_CONNECT_IDE`, `CLAUDE_CODE_IDE_HOST_OVERRIDE`, `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL`, `CLAUDE_CODE_IDE_SKIP_VALID_CHECK` | Connexion à l'IDE |
| `CLAUDE_CODE_ENABLE_CFC`, `CLAUDE_CODE_DISABLE_CFC_PROMPT` | Claude in Chrome au démarrage / sans section de prompt |
| `CLAUDE_CODE_YOU_SHOULD_KNOW_DEBUG` | Voir les vérifications du mod « You should know » |

## 11. Sessions, persistance, mises à jour et distant

| Variable | Rôle |
|----------|------|
| `CLAUDE_CONFIG_DIR` | Dossier de configuration (au lieu de `~/.claude`) — un par compte |
| `CLAUDE_CODE_PROJECT_DIR_NAME` | Nom du dossier `projects/` d'une session |
| `CLAUDE_CODE_TMPDIR` | Dossier temporaire interne |
| `CLAUDE_CODE_SKIP_PROMPT_HISTORY` | Ne rien enregistrer (historique, transcriptions) |
| `CLAUDE_CODE_FORCE_SESSION_PERSISTENCE` | Forcer l'enregistrement d'une session lancée depuis une autre |
| `CLAUDE_CODE_RESUME_INTERRUPTED_TURN` (+ `_MAX_AGE_MS`), `CLAUDE_CODE_RESUME_PROMPT` | Reprendre automatiquement un tour interrompu |
| `CLAUDE_CODE_EXIT_AFTER_STOP_DELAY` | Quitter automatiquement après inactivité (scripts) |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | Budget de temps des hooks `SessionEnd` |
| `CLAUDE_CODE_STARTUP_FAILURE_RESULTS` | Message de résultat expliquant un échec de démarrage (`stream-json`) |
| `CLAUDE_CODE_EMIT_SESSION_STATE_EVENTS` | Messages `session_state_changed` dans le flux |
| `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX` | Préfixe des noms de sessions Remote Control |
| `CCR_FORCE_BUNDLE` | `claude --cloud` téléverse le dépôt local au lieu de cloner |
| `CLAUDE_CODE_PROCESS_WRAPPER` | Lancer les processus de fond via un launcher d'entreprise |
| `CLAUDE_CODE_ARTIFACT_AUTO_OPEN`, `CLAUDE_CODE_ARTIFACT_COMMENTS`, `CLAUDE_CODE_ARTIFACT_COMMENTS_AUTOREACT` | Ouverture auto des artifacts, lecture et réponse automatique aux commentaires |
| `CLAUDE_CODE_DISABLE_ARTIFACT` | Désactiver l'outil Artifact |
| `CLAUDE_CODE_SEND_FEEDBACK` | Retours rédigés par Claude on/off |
| `DISABLE_AUTOUPDATER` | Pas de mise à jour automatique (`claude update` reste possible) |
| `DISABLE_UPDATES` | Bloquer toute mise à jour |
| `CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE` | Mise à jour en arrière-plan via Homebrew/WinGet |
| `DISABLE_INSTALLATION_CHECKS` | Masquer les avertissements d'installation |

## 12. Désactiver des commandes et fonctions

| Variable | Rôle |
|----------|------|
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | Couper tout trafic non essentiel (mises à jour, télémétrie, rapports d'erreurs, `/feedback`…) |
| `DISABLE_GROWTHBOOK` | Ne pas récupérer les feature flags (désactive les fonctions qui en dépendent, comme Remote Control) |
| `DISABLE_COST_WARNINGS` | Pas d'avertissements de coût |
| `DISABLE_DOCTOR_COMMAND` | Masquer `/doctor` |
| `DISABLE_EXTRA_USAGE_COMMAND` | Masquer `/usage-credits` |
| `DISABLE_FEEDBACK_COMMAND` | Désactiver `/feedback`, `/bug`, `/share` |
| `DISABLE_INSTALL_GITHUB_APP_COMMAND` | Masquer `/install-github-app` |
| `DISABLE_LOGIN_COMMAND`, `DISABLE_LOGOUT_COMMAND`, `DISABLE_UPGRADE_COMMAND` | Masquer `/login`, `/logout`, `/upgrade` |
| `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` | Pas de sondage « How is Claude doing? » |
| `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | Désactiver les points de contrôle de fichiers |
| `CLAUDE_CODE_BASH_EDIT_DIFF` | Diff des fichiers modifiés par une commande Bash |

## 13. Sécurité et isolation

| Variable | Rôle |
|----------|------|
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | Retirer les identifiants de l'environnement des sous-processus (Bash, hooks, MCP) |
| `CLAUDE_CODE_SCRIPT_CAPS` | Limiter le nombre d'appels de certains scripts par session (avec le scrub) |
| `CLAUDE_CODE_RESTRICTED` | Démarrer en mode restreint (`--restricted`) |
| `CLAUDE_CODE_SAFE_MODE` | Démarrer en mode sans échec (`--safe-mode`) |
| `CLAUDE_CODE_SIMPLE` | Mode minimal (`--bare`) |
| `CLAUDE_CODE_DISABLE_DANGEROUS_RM_TIMEOUT` | Pas de délai sur les demandes de suppression de chemins critiques |
| `CLAUDE_CODE_DISABLE_INLINE_SHELL_RM_PROMPT` | Ne pas analyser les scripts `bash -c '…'` pour les suppressions critiques |
| `CLAUDE_CODE_DISABLE_SUBSTITUTION_RM_PROMPT` | Ne pas vérifier `rm -rf "$(…)"` |
| `CLAUDE_CODE_DISABLE_POWERSHELL_CMD_RM_DENY` | Désactiver le refus de `rd`/`del` sur un chemin système |
| `CLAUDE_CODE_DISABLE_AUTH_REFRESH_LOCK` | Chaque processus lance lui-même `gcpAuthRefresh`/`awsAuthRefresh` |
| `CLAUDE_CODE_DISABLE_ADMIN_ENV_UNION` | Ne pas fusionner les blocs `env` des sources gérées |
| `CLAUDE_CODE_AUTO_MODE_SERVER` | Revue du mode auto côté serveur ou classifieur local |
| `CLAUDE_CODE_DISABLE_PERMISSION_PROMPT_NOTIFY_HOOKS` | Ne pas lancer les hooks `Notification` pour les permissions transmises au SDK |

**Ce que le scrub retire** : clés et secrets reconnaissables (`ANTHROPIC_API_KEY`, `AWS_SECRET_ACCESS_KEY`, `NPM_TOKEN`, `DB_PASSWORD`, URL contenant un mot de passe…), `CLAUDE_CONFIG_DIR`. Il **garde** `GITHUB_TOKEN`/`GH_TOKEN` (pour `gh`), `HTTP(S)_PROXY`, `GIT_CONFIG_*`.

## 14. Télémétrie, OpenTelemetry et débogage

| Variable | Rôle |
|----------|------|
| `DISABLE_TELEMETRY`, `DO_NOT_TRACK` | Désactiver la télémétrie |
| `DISABLE_ERROR_REPORTING` | Désactiver les rapports d'erreurs |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | Activer l'export OpenTelemetry (métriques, logs) |
| `OTEL_LOG_USER_PROMPTS`, `OTEL_LOG_ASSISTANT_RESPONSES` | Inclure le texte des prompts / réponses |
| `OTEL_LOG_TOOL_DETAILS`, `OTEL_LOG_TOOL_CONTENT` | Inclure les arguments / le contenu des outils |
| `OTEL_LOG_RAW_API_BODIES` | Journaliser les corps bruts des requêtes API |
| `OTEL_LOG_MANAGED_SETTINGS` | Journaliser les réglages gérés (expurgés) |
| `OTEL_METRICS_INCLUDE_ACCOUNT_UUID` / `_SESSION_ID` / `_VERSION` / `_ENTRYPOINT` / `_REPOSITORY` / `_RESOURCE_ATTRIBUTES` | Attributs ajoutés aux métriques |
| `OTEL_ATTRIBUTE_VALUE_LENGTH_LIMIT`, `CLAUDE_CODE_OTEL_CONTENT_MAX_LENGTH` | Longueur max des attributs |
| `CLAUDE_CODE_OTEL_FLUSH_TIMEOUT_MS`, `CLAUDE_CODE_OTEL_SHUTDOWN_TIMEOUT_MS`, `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS`, `CLAUDE_CODE_OTEL_DIAG_STDERR` | Délais et diagnostics de l'exportateur |
| `ENABLE_BETA_TRACING_DETAILED`, `BETA_TRACING_ENDPOINT` | Traçage détaillé bêta |
| `CLAUDE_CODE_ENABLE_FEEDBACK_SURVEY_FOR_OTEL` | Envoyer le sondage de qualité vers votre collecteur |
| `DEBUG` | Mode débogage (comme `--debug`) ; logs dans `~/.claude/debug/<session>.txt` |
| `CLAUDE_CODE_DEBUG_LOGS_DIR`, `CLAUDE_CODE_DEBUG_LOG_LEVEL` | Chemin et niveau du journal de débogage |

## 15. Variables définies par Claude Code (à lire dans vos scripts et hooks)

| Variable | Contenu |
|----------|---------|
| `CLAUDECODE` | `1` dans les sous-processus lancés par Claude Code |
| `CLAUDE_CODE_CHILD_SESSION` | `1` dans les sous-processus Bash/PowerShell/Monitor/hooks/barre d'état |
| `CLAUDE_CODE_SESSION_ID` | ID de la session |
| `CLAUDE_CODE_REMOTE`, `CLAUDE_CODE_REMOTE_SESSION_ID` | `true` et ID dans une session cloud |
| `CLAUDE_CODE_BRIDGE_SESSION_ID` | ID de session Remote Control quand elle est connectée |
| `CLAUDE_EFFORT` | Niveau d'effort en cours |
| `CLAUDE_PID` | PID de Claude Code |
| `CLAUDE_JOB_DIR` | Dossier d'une session d'arrière-plan (`~/.claude/jobs/<id>`) |
| `CLAUDE_CODE_MESSAGING_SOCKET`, `CLAUDE_CODE_MESSAGING_TOKEN` | Socket et jeton de messagerie entre sessions |
| `CLAUDE_PROJECT_DIR` | Racine du projet (hooks, serveurs MCP) |

## Variables retirées (sans effet)

`CLAUDE_CODE_CONNECT_TIMEOUT_MS`, `CLAUDE_CODE_MAX_SUBAGENTS_PER_SESSION`, `CLAUDE_CODE_AUTO_BACKGROUND_WORKER_CHECKIN_SECONDS`, `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE`, `CLAUDE_CODE_ENABLE_OPUS_4_7_FAST_MODE`, `CLAUDE_SUBAGENT_BG_SHELL_MAX_MS`, `TASK_MAX_OUTPUT_LENGTH`, `CLAUDE_CODE_ENABLE_AUTO_MODE` (le mode auto est disponible partout).
