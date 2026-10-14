# 27 — Dépannage et messages d'erreur

[← Sommaire](README.md)

## 1. Les outils de diagnostic

| Outil | Usage |
|-------|-------|
| `claude doctor` (shell) | Santé de l'installation, erreurs dans les fichiers de réglages, éligibilité à Remote Control (lecture seule) |
| `/doctor` (session) | Bilan complet avec corrections appliquées après confirmation ; `/doctor prompt-audit` pour vos instructions |
| `/debug [description]` | Active le journal de débogage et l'analyse |
| `claude --debug[=mcp,hooks,…]`, `--debug-file` | Journal détaillé (`~/.claude/debug/<session>.txt`) |
| `/status` | Version, modèle, compte, sources de réglages, clé API utilisée |
| `/context` | Ce qui est chargé dans le contexte |
| `/mcp`, `/hooks`, `/plugin` (onglet Errors), `/skills`, `/permissions` | État de chaque extension |
| `claude --safe-mode` | Démarrer sans aucune personnalisation pour savoir si l'une d'elles est en cause |
| `claude --bare` | Démarrage minimal |
| `/heapdump` | Diagnostic de mémoire |
| `/bug` | Signaler un bug à Anthropic (avec consentement) |

### Déboguer sa configuration (méthode officielle)

1. **Voir ce qui est chargé** : `/context`, `/memory`.
2. **Vérifier les réglages résolus** : `/status`, `/config`.
3. **Vérifier les serveurs MCP** : `/mcp`, `claude mcp list`.
4. **Vérifier les hooks** : `/hooks`, journal `--debug=hooks`.
5. **Tester avec une configuration vierge** : `--safe-mode`, `--setting-sources user`.
6. **Causes fréquentes** : JSON invalide (commentaires, virgules finales), fichier au mauvais endroit, espace de travail non approuvé, réglage écrasé par un niveau plus prioritaire.

## 2. Problèmes d'installation

| Symptôme | Piste |
|----------|-------|
| Le script d'installation renvoie du HTML | Réseau/proxy qui intercepte ; vérifier l'accès à `claude.ai` |
| `command not found: claude` | Ajouter `~/.local/bin` au `PATH`, rouvrir le terminal |
| `permission denied` sur le PATH, erreurs de permissions | Droits du dossier d'installation ; éviter `sudo` |
| Erreurs TLS/SSL, `Failed to fetch version` | Proxy d'entreprise : `NODE_EXTRA_CA_CERTS`, `HTTPS_PROXY` |
| Installation tuée (peu de mémoire, Docker) | Au moins 4 Go de RAM ; voir les pages dédiées |
| Windows : mauvaise commande, fichier verrouillé, 32 bits non supporté | Utiliser la commande PowerShell ou CMD adaptée ; Git for Windows ou PowerShell requis |
| Linux : musl/glibc, `Illegal instruction` | Binaire adapté à la distribution / au processeur |
| macOS : `dyld: cannot load` | Version de macOS trop ancienne |
| WSL1 : `Exec format error` | Utiliser WSL2 |
| Plusieurs installations en conflit | `claude doctor` les détecte ; garder l'installation native |
| `claude update` bloqué | Vérifier le réseau ; `claude install` |

## 3. Connexion et authentification

| Symptôme | Piste |
|----------|-------|
| `Not logged in · Please run /login` | `/login` |
| `Invalid API key`, `401` | Vérifier/retirer `ANTHROPIC_API_KEY` |
| `Credit balance is too low` alors que vous avez un abonnement | Une `ANTHROPIC_API_KEY` approuvée prend le pas : `/status`, puis retirer la variable |
| `OAuth error: Invalid code`, `403 Forbidden` | Recommencer `/login` ; vérifier que l'organisation donne accès à Claude Code |
| OAuth impossible en SSH, WSL2, conteneur | Copier l'URL affichée dans un navigateur local, ou `claude setup-token` |
| Identifiants Bedrock / Google Cloud / Foundry non chargés | Vérifier le profil, la région, `/setup-bedrock`, `/setup-vertex` |
| Réinitialiser complètement la connexion | `/logout` puis `/login` (ou `claude auth logout`) |

## 4. Performances et affichage

| Symptôme | Piste |
|----------|-------|
| CPU ou mémoire élevés | `/compact`, redémarrer, `/heapdump` pour un rapport |
| Grands tableaux coupés | Élargir le terminal, mode plein écran |
| La compaction automatique échoue en boucle | `/clear`, réduire les fichiers volumineux lus |
| Commande bloquée | `Esc`, `Ctrl+C`, `/tasks` pour arrêter une tâche de fond |
| Texte corrompu dans le terminal d'un éditeur | `Ctrl+L`, `/tui default`, variables `CLAUDE_CODE_ALT_SCREEN_FULL_REPAINT`… |
| Molette lente en plein écran | `/scroll-speed`, `CLAUDE_CODE_SCROLL_SPEED` |
| Copier vers le presse-papiers local en SSH | Plein écran (copie OSC 52), `/copy` puis `w` pour écrire dans un fichier |
| `pbcopy` échoue dans la sandbox | Exclure la commande de la sandbox |
| Recherche lente/incomplète (WSL) | Placer le projet dans le système de fichiers Linux ; `USE_BUILTIN_RIPGREP` |
| Réponses de moins bonne qualité que d'habitude | Vérifier `/model` (bon modèle ?), `/effort` (niveau ?), `/context` (contexte saturé → `/compact` ou `/clear`), repli automatique de modèle signalé dans la transcription |

## 5. Catalogue des messages d'erreur (par famille)

### Erreurs serveur (temporaires)

`API Error: 500`, `Repeated 529 Overloaded errors` (API saturée pour tout le monde, ne compte pas dans votre quota), `<modèle> is experiencing high load`, `Request timed out`, `No response from API`, réponse interrompue (« Connection lost mid-response », « Your computer went to sleep », « Response stalled »…), erreurs du classifieur du mode auto (« Auto mode could not evaluate this action and is blocking it for safety »), `Agent terminated early due to an API error`.

→ Claude Code **réessaie automatiquement** (jusqu'à 10 fois avec backoff). Si cela persiste : https://status.claude.com, réessayer, changer de modèle, `--fallback-model`. Réglage des tentatives : `CLAUDE_CODE_MAX_RETRIES`, `CLAUDE_CODE_RETRY_WATCHDOG`.

### Limites d'usage

`You've hit your session limit` / `weekly limit` / `Opus limit` / `Sonnet limit`, `Usage credits required for 1M context`, `Server is temporarily limiting requests`, `Request rejected (429)` (limite de débit de votre clé ou projet cloud), `Credit balance is too low`, limites de dépense mensuelles (individuelle, organisation, canal, équipe).

→ Attendre la réinitialisation (Claude reprend seul), `/usage`, `/usage-credits`, `/rate-limit-options`, `/upgrade`.

### Authentification

`Not logged in`, `Could not resolve authentication method`, `Invalid API key`, `Your apiKeyHelper script is failing`, `Invalid ANTHROPIC_CUSTOM_HEADERS`, `This organization has been disabled`, `Your organization has disabled API key authentication`, `OAuth token revoked or expired`, `Login expired`, `Failed to start OAuth callback server`, `Your account is on hold`, erreurs Remote Control (« only available when using Claude via api.anthropic.com », « turned off by your organization's policy »), `Artifacts need a claude.ai login`, identifiants AWS / Google Cloud / Foundry expirés, session de gateway expirée, serveur MCP qui demande une nouvelle connexion.

### Réseau

`Unable to connect to API` (`ConnectionRefused` : pare-feu/proxy ; `ENOTFOUND` : DNS), `Unable to connect to Anthropic services`, `Socket is closed`, réponse vide ou mal formée, **erreurs de certificat SSL** (`UNABLE_TO_GET_ISSUER_CERT_LOCALLY`, `SELF_SIGNED_CERT_IN_CHAIN` → `NODE_EXTRA_CA_CERTS=/chemin/ca.pem`), hôte non autorisé dans une session cloud (réglage réseau de l'environnement).

### Requêtes

`Prompt is too long` / `Context limit reached · /compact or /clear to continue`, `Context exceeds the token limit` (affiché par `/context`), `Request too large`, image trop grande, erreurs PDF, `Extra inputs are not permitted`, schéma d'outil invalide, `Model not found` / modèle non reconnu, `Claude Opus is not available with the Claude Pro plan`, changement de modèle bloqué par un hook, budget de réflexion trop grand, refus de la politique d'usage, sujet de cybersécurité signalé, sortie bloquée par le filtrage de contenu.

### Ligne de commande

Fichier de réglages > 2 Mio, dossier courant supprimé, espace de travail non approuvé, `Unknown command: /hepl. Did you mean /help?`, entrée vide, diff trop grand pour ultrareview, pas de merge-base avec la branche de base, impossible de reprendre la conversation / aucune conversation pour cet ID, port de rappel OAuth déjà utilisé, `--agents` invalide, `--bg` combiné à `-p`.

### Plugins, outils, sessions de fond, IDE, rewind, sauvegarde, configuration

- **Plugins** : marketplace de source non fiable ou déjà ajoutée ailleurs, échec de vérification d'intégrité d'une archive, chemin qui sort du dossier du plugin, configuration de marketplace illisible.
- **Outils** : agent qui serait lancé sans aucun outil, fichier couvert par une règle `Read` deny, index de mémoire trop gros, motif `pkill` qui viserait Claude Code, message inter-sessions trop gros ou trop nombreux, écriture bloquée (chemin non résolu ou réseau), commande bloquée par l'isolation de worktree.
- **Sessions de fond** : pas de transcription enregistrée, processus hôte mort, erreurs du launcher (`CLAUDE_CODE_PROCESS_WRAPPER`), `EACCES`/`EUNKNOWN` au démarrage, service de fond injoignable (`claude daemon stop --any` puis relancer).
- **IDE / wrapper** : processus Claude Code terminé avec un code N.
- **Rewind** : fichiers liés (symlinks) non restaurés, messages envoyés pendant un tour marqués « No code restore ».
- **Sauvegarde** : échec d'écriture des transcriptions.
- **Configuration** : espace de travail non approuvé, dossier de travail sur un chemin réseau, `headersHelper` non exécuté, règle de permission mal formée (joker mal placé), refuser Bash désactive aussi PowerShell, fichiers de masque de sandbox laissés par une session tuée.

## 6. Problèmes fréquents par fonction

| Fonction | Où chercher |
|----------|-------------|
| Hook qui ne se déclenche pas | `/hooks`, matcher (sensible à la casse), confiance de l'espace de travail, `disableAllHooks` ; JSON pollué par le profil du shell |
| Serveur MCP en échec | `/mcp` (état, authentification), `/mcp reconnect all`, `MCP_TIMEOUT`, `claude --debug=mcp` |
| Skill pas proposé | Description trop vague, `disable-model-invocation`, `skillOverrides`, `/reload-skills`, `/skill-doctor` |
| Plugin inactif | `/plugin` › Errors, `/reload-plugins`, `claude plugin validate` |
| `CLAUDE.md` ignoré | Trop long ou contradictoire ; `/doctor prompt-audit` ; le mettre au bon endroit ; transformer une règle critique en hook |
| `AGENTS.md` non chargé | Mod `cc-plugin-agents-md` désactivé ou session qui ne le lit pas |
| Barre d'état absente | Script exécutable, confiance de l'espace de travail, sortie sur stdout |
| Remote Control indisponible | Compte claude.ai requis, politique de l'organisation, `claude doctor` |
| Agent view vide ou superviseur muet | `claude daemon status`, `claude daemon stop --any` |
| Diff/terminal du Desktop | Version de l'app, Git installé, `gh` pour les PR |
| Extension VS Code muette | Recharger la fenêtre, journaux (« Show Logs »), `Cmd+Esc` sur macOS |

## 7. Obtenir de l'aide

`/bug` ou `/feedback` (rapport avec contexte), documentation officielle (code.claude.com/docs), page d'état (status.claude.com), centre d'aide (support.claude.com), dépôt GitHub `anthropics/claude-code` (issues).
