# 01 — Présentation et installation

[← Sommaire](README.md)

## Qu'est-ce que Claude Code ?

Claude Code est un **outil de codage agentique** : un assistant qui ne se contente pas de répondre, mais qui **agit** dans votre projet. Il fonctionne en boucle (« boucle agentique ») :

1. il **comprend** la demande et rassemble du contexte (lecture de fichiers, recherche dans le code) ;
2. il **agit** avec des outils (modifier des fichiers, lancer des commandes, chercher sur le web, appeler des services via MCP) ;
3. il **vérifie** le résultat (tests, compilation, captures d'écran) ;
4. il recommence jusqu'à ce que la tâche soit terminée, puis il rend compte.

Ce qu'il sait faire, en résumé :

- **Explorer et expliquer** une base de code (« comment fonctionne l'authentification ici ? »).
- **Écrire, modifier, refactoriser** du code sur plusieurs fichiers à la fois.
- **Corriger des bugs** à partir d'un message d'erreur, d'un log, d'une capture d'écran.
- **Lancer des commandes** : tests, build, linters, serveurs, git, gh, docker…
- **Faire du git** : commits avec messages, branches, pull requests, résolution de conflits.
- **Relire du code** (`/code-review`, `/security-review`, ultrareview dans le cloud).
- **Chercher sur le web** et lire de la documentation.
- **Piloter un navigateur** (Claude in Chrome, navigateur intégré de l'app Desktop) et même **votre ordinateur** (computer use).
- **Se connecter à vos outils** (GitHub, Slack, Linear, Sentry, bases de données…) via MCP.
- **Travailler en parallèle** avec des sous-agents, des sessions en arrière-plan, des équipes d'agents et des workflows de dizaines d'agents.
- **Tourner sans vous** : tâches planifiées, routines dans le cloud, GitHub Actions, mode script `-p`.
- **Produire des pages partageables** (artifacts : rapports, tableaux de bord, diapositives, maquettes).

## Où l'utiliser ?

| Plateforme | Idéal pour | Points forts |
|------------|-----------|--------------|
| **CLI (terminal)** | Travail en terminal, scripts, serveurs distants | Toutes les fonctions, Agent SDK, computer use (macOS), fournisseurs tiers |
| **Desktop** (app Claude, onglet Code) | Revue visuelle, sessions parallèles | Visionneuse de diff, aperçu de l'app, navigateur intégré, terminal intégré, computer use et Dispatch (Pro/Max) |
| **VS Code** (+ Cursor, etc.) | Rester dans l'éditeur | Diffs en ligne, contexte du fichier ouvert, sélection partagée |
| **JetBrains** (IntelliJ, PyCharm, WebStorm, GoLand, PhpStorm, Android Studio…) | Rester dans l'IDE JetBrains | Diff dans l'IDE, partage de sélection et de diagnostics |
| **Web** (claude.ai/code) | Tâches longues sans surveillance | Sessions cloud qui continuent après déconnexion |
| **Mobile** (Claude iOS / Android) | Lancer et surveiller depuis le téléphone | Sessions cloud, Remote Control, Dispatch, notifications push |

Intégrations : **Chrome**, **GitHub Actions**, **GitLab CI/CD**, **Code Review** automatique des PR, **Slack**, **Claude Tag** (Slack d'équipe).

## Configuration requise

- **Système** : macOS 13+, Windows 10 1809+ / Server 2019+, Ubuntu 20.04+, Debian 10+, Alpine 3.19+.
- **Matériel** : 4 Go de RAM minimum, processeur x64 ou ARM64.
- **Réseau** : connexion internet obligatoire.
- **Shell** : Bash, Zsh, PowerShell ou CMD.
- **Pays** : pays pris en charge par Anthropic.
- `ripgrep` est normalement inclus.

## Installation

| Système | Commande |
|---------|----------|
| macOS, Linux, WSL | `curl -fsSL https://claude.ai/install.sh \| bash` |
| Windows PowerShell | `irm https://claude.ai/install.ps1 \| iex` |
| Windows CMD | `curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd` |
| Homebrew (macOS) | `brew install --cask claude-code` (pas de mise à jour automatique : `brew upgrade claude-code`) |
| WinGet (Windows) | `winget install Anthropic.ClaudeCode` |

- Version stable plutôt que la plus récente : `curl -fsSL https://claude.ai/install.sh | bash -s stable`.
- Installer/réinstaller une version précise : `claude install 2.1.118`, `claude install stable`, `claude install latest`.
- Mettre à jour : `claude update` (l'installation native se met à jour toute seule).
- Diagnostiquer l'installation : `claude doctor` (dans le shell) ou `/doctor` (dans une session, propose aussi des corrections).
- Application Desktop : à télécharger sur claude.com/download (macOS universel, Windows x64/ARM64, Linux via apt en bêta).
- Extension VS Code : depuis la marketplace VS Code ; plugin JetBrains : depuis la marketplace JetBrains.

## Connexion (authentification)

Au premier lancement de `claude`, un navigateur s'ouvre pour vous connecter. Types de comptes :

| Compte | Usage |
|--------|-------|
| **Claude Pro ou Max** | Abonnement individuel claude.ai |
| **Claude Team ou Enterprise** | Compte invité par l'administrateur (SSO, réglages gérés, etc.) |
| **Claude Console** | Facturation à l'usage de l'API (`claude auth login --console`) |
| **Fournisseurs cloud** | Amazon Bedrock, Google Cloud Agent Platform (Vertex AI), Microsoft Foundry, Claude Platform on AWS |
| **Claude apps gateway** | Passerelle auto-hébergée avec SSO d'entreprise |
| **Clé API** | Variable `ANTHROPIC_API_KEY` (Claude Code demande confirmation) |

Commandes utiles :

- `/login`, `/logout` dans une session ; `claude auth login | logout | status` dans le shell.
- `claude auth login --email vous@exemple.com`, `--sso` pour forcer le SSO.
- `claude setup-token` : génère un jeton OAuth longue durée pour la CI (nécessite un abonnement).
- Plusieurs comptes en parallèle : un dossier de configuration par compte avec `CLAUDE_CONFIG_DIR`.

## Premiers pas

```bash
cd mon-projet
claude                      # ouvre une session interactive
claude "explique ce projet" # ouvre une session avec une première demande
claude -p "résume README.md" # répond une fois puis quitte (mode script)
claude -c                   # reprend la dernière conversation du dossier
claude -r                   # choisit une conversation à reprendre
```

Dans la session :

1. `/init` → génère un `CLAUDE.md` décrivant le projet (commandes de build, conventions…).
2. Posez une question (« quelle est l'architecture ? »), puis demandez une modification.
3. Claude propose chaque action ; vous acceptez, refusez ou commentez selon le **mode de permission** (voir [06](06-permissions-securite-sandbox.md)).
4. Vérifiez avec `/diff`, faites relire avec `/code-review`, demandez « commit mes changements » ou « crée une PR ».

Autres aides intégrées : `/help`, `/powerup` (leçons interactives animées), `?` sur un prompt vide (aide des raccourcis), `/release-notes`, l'agent `claude-code-guide` (posez simplement des questions sur Claude Code).

## Où Claude Code range ses fichiers

| Emplacement | Contenu |
|-------------|---------|
| `~/.claude/settings.json` | Vos réglages personnels |
| `~/.claude/CLAUDE.md` | Vos instructions personnelles (tous projets) |
| `~/.claude/skills/`, `~/.claude/agents/`, `~/.claude/commands/` | Vos skills, sous-agents, anciennes commandes personnalisées |
| `~/.claude/keybindings.json` | Vos raccourcis clavier personnalisés |
| `~/.claude/themes/` | Thèmes personnalisés |
| `~/.claude/projects/<projet>/` | Transcriptions des sessions (`.jsonl`) et mémoire automatique (`memory/`) |
| `~/.claude/plugins/` | Plugins installés |
| `~/.claude.json` | Configuration globale, serveurs MCP de portée « user » et « local » |
| `.claude/settings.json` | Réglages partagés du projet (à committer) |
| `.claude/settings.local.json` | Vos réglages perso pour ce projet (hors git) |
| `.claude/skills/`, `.claude/agents/`, `.claude/rules/` | Skills, sous-agents et règles du projet |
| `.mcp.json` | Serveurs MCP partagés du projet |
| `CLAUDE.md`, `CLAUDE.local.md`, `AGENTS.md` | Instructions du projet |
| `.claude/worktrees/` | Worktrees git créés par Claude |

Effacer toutes les données locales d'un projet : `claude purge [chemin]` (`--dry-run` pour prévisualiser, `--all` pour tous les projets).
