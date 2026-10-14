# 04 — La ligne de commande : sous-commandes, flags et mode script

[← Sommaire](README.md)

## Lancer Claude Code

| Commande | Effet |
|----------|-------|
| `claude` | Session interactive |
| `claude "demande"` | Session interactive avec une première demande |
| `claude -p "demande"` | **Mode script** (non interactif) : répond puis quitte |
| `cat fichier \| claude -p "explique"` | Traite du contenu envoyé par un pipe |
| `claude -c` | Reprend la conversation la plus récente du dossier |
| `claude -c -p "demande"` | Reprend et répond en mode script |
| `claude -r "<session>" "demande"` | Reprend une session par ID ou par nom |

## Sous-commandes `claude …`

| Sous-commande | Rôle |
|---------------|------|
| `claude update` | Mettre à jour |
| `claude install [version\|stable\|latest]` | Installer/réinstaller le binaire natif |
| `claude doctor` | Diagnostic en lecture seule de l'installation et des réglages |
| `claude auth login [--email] [--sso] [--console]` / `logout` / `status [--text]` | Authentification |
| `claude setup-token` | Jeton OAuth longue durée pour CI et scripts |
| `claude mcp …` | Gérer les serveurs MCP : `add`, `add-json`, `add-from-claude-desktop`, `list`, `get`, `remove`, `login`, `logout`, `serve`, `reset-project-choices` |
| `claude plugin …` (ou `plugins`) | Gérer les plugins : `install`, `uninstall`, `enable`, `disable`, `list`, `validate`, `marketplace add/list/…`, `eval` |
| `claude agents [--cwd] [--json [--all]]` | Ouvrir l'**agent view** (sessions en arrière-plan) ou lister en JSON |
| `claude attach <id\|nom>` | S'attacher à une session d'arrière-plan |
| `claude logs <id\|nom>` | Afficher la sortie récente d'une session d'arrière-plan |
| `claude stop <id>` (ou `kill`) | Arrêter une session d'arrière-plan |
| `claude respawn <id>` / `--all` | Redémarrer une ou toutes les sessions (ex. après une mise à jour) |
| `claude rm <id>` | Retirer une session de la liste (et son worktree si sûr) |
| `claude daemon status\|logs\|run\|stop` | Superviseur des sessions d'arrière-plan |
| `claude remote-control` | Serveur **Remote Control** (piloter depuis claude.ai / mobile) |
| `claude ultrareview [cible] [--json] [--timeout] [--post]` | Ultrareview non interactive (CI) |
| `claude auto-mode defaults\|config\|reset` | Règles du classifieur du mode auto |
| `claude import [source]` | Importer la config d'autres agents (Codex, Gemini, Cursor) |
| `claude purge [chemin]` | Effacer les données locales d'un projet |
| `claude gateway --config gateway.yaml` | Démarrer la **Claude apps gateway** (administrateurs) |
| `claude self-hosted-runner [setup\|doctor\|orchestrator]` | Exécuteur pour environnements cloud auto-hébergés |

Une faute de frappe (`claude udpate`) affiche « Did you mean claude update? ».

## Flags principaux

### Session et reprise

| Flag | Rôle |
|------|------|
| `-c`, `--continue` | Reprendre la dernière conversation du dossier |
| `-r`, `--resume [id\|nom\|chemin.jsonl]` | Reprendre une session précise ou ouvrir le sélecteur |
| `--fork-session` | En reprenant, créer un nouvel ID de session (copie) |
| `--from-pr <n°\|URL>` | Sélecteur filtré sur les sessions liées à une PR/MR |
| `-n`, `--name` | Nommer la session |
| `--session-id <uuid>` | Imposer un ID de session |
| `--no-session-persistence` | Ne pas sauvegarder la session (mode `-p`) |
| `-w`, `--worktree [nom\|#PR\|URL]` | Démarrer dans un **worktree git** isolé (`.claude/worktrees/<nom>`) |
| `--tmux` | Avec `--worktree`, ouvrir une session tmux (ou panneaux iTerm2) |
| `--bg`, `--background` | Démarrer directement en **agent d'arrière-plan** ; avec `--exec 'cmd'`, lancer une commande shell en tâche de fond |
| `--cloud "tâche"` | Créer une **session cloud** (ou envoyer un message à une session cloud existante) |
| `--teleport` | Reprendre une session cloud dans le terminal |
| `--desktop` | Ouvrir l'app Desktop sur le dossier courant |
| `--remote-control`, `--rc [nom]` | Session interactive pilotable à distance |

### Modèle et comportement

| Flag | Rôle |
|------|------|
| `--model <alias\|id>` | Modèle de la session (`sonnet`, `opus`, `haiku`, `fable`, `best`, ou ID complet) |
| `--effort <low\|medium\|high\|xhigh\|max\|ultracode>` | Niveau d'effort |
| `--fallback-model sonnet,haiku` | Modèles de secours si le principal est surchargé |
| `--advisor <modèle>` | Activer l'advisor |
| `--agent <nom>` | Faire tourner toute la session comme un sous-agent donné |
| `--agents '<json>'` | Définir des sous-agents à la volée |
| `--append-system-prompt "…"` / `--append-system-prompt-file` | Ajouter au prompt système |
| `--system-prompt "…"` / `--system-prompt-file` | Remplacer entièrement le prompt système |
| `--autocompact <auto\|jetons>` | Seuil de compaction automatique pour la session |
| `--betas` | En-têtes bêta de l'API (clé API seulement) |

### Permissions et outils

| Flag | Rôle |
|------|------|
| `--permission-mode <default\|manual\|acceptEdits\|plan\|auto\|dontAsk\|bypassPermissions>` | Mode de permission de départ |
| `--dangerously-skip-permissions` | Tout autoriser sans demander (= `bypassPermissions`) — **réservé aux conteneurs/VM isolés** |
| `--allow-dangerously-skip-permissions` | Ajoute bypass au cycle `Shift+Tab` sans démarrer dedans |
| `--allowedTools "Bash(git log *)" "Read"` | Outils autorisés sans demande |
| `--disallowedTools "Edit" "mcp__*"` | Outils interdits/retirés |
| `--tools "Bash,Edit,Read"` | Restreindre les outils intégrés disponibles (`""` = aucun, `"default"`) |
| `--add-dir ../lib` | Dossiers de travail supplémentaires |
| `--restricted` | Mode restreint (harnais d'évaluation : pas d'exécution de commandes, réglages limités) |
| `--permission-prompt-tool <outil_mcp>` | Outil MCP qui répond aux demandes de permission en mode `-p` |
| `--permission-prompts none` | En `-p`, refuser toute demande de permission |

### Configuration et extensions

| Flag | Rôle |
|------|------|
| `--settings <fichier\|json>` | Réglages supplémentaires pour la session |
| `--setting-sources user,project,local` | Quelles sources de réglages charger |
| `--mcp-config ./mcp.json` | Charger des serveurs MCP |
| `--strict-mcp-config` | N'utiliser que les serveurs de `--mcp-config` |
| `--plugin-dir <dossier\|zip>` / `--plugin-url <url>` | Charger un plugin pour la session |
| `--channels plugin:nom@marketplace` | Écouter des **channels** (aperçu de recherche) |
| `--chrome` / `--no-chrome` | Activer/désactiver Claude in Chrome |
| `--ide` | Se connecter automatiquement à l'IDE |
| `--disable-slash-commands` | Désactiver skills et commandes |
| `--bare` | Mode minimal : ne charge ni hooks, ni skills, ni plugins, ni MCP, ni mémoire, ni `CLAUDE.md` (démarrage rapide pour scripts) |
| `--safe-mode` | Désactive toutes les personnalisations pour dépanner une configuration cassée |
| `--teammate-mode <in-process\|auto\|tmux\|iterm2>` | Affichage des équipes d'agents |
| `--init`, `--init-only`, `--maintenance` | Exécuter les hooks `Setup` |

### Débogage et affichage

| Flag | Rôle |
|------|------|
| `--debug[=filtres]`, `--debug-file <chemin>` | Logs de débogage (ex. `--debug='mcp,startup'`) |
| `--verbose` | Sortie détaillée tour par tour |
| `--ax-screen-reader` | Sortie adaptée aux lecteurs d'écran |
| `-v`, `--version` | Version |

## Mode script / non interactif (`-p`)

Le flag `-p` (`--print`) transforme Claude Code en outil de ligne de commande utilisable dans des scripts, la CI ou des pipelines.

```bash
# Question simple
claude -p "Quels fichiers gèrent l'authentification ?"

# Pipe d'entrée et redirection de sortie
cat build-error.txt | claude -p "explique la cause de cette erreur" > explication.txt

# Sortie JSON (avec coût, durée, ID de session…)
claude -p "résume le projet" --output-format json

# Flux JSON en temps réel
claude -p "corrige les tests" --output-format stream-json --verbose

# Sortie structurée validée par un schéma JSON
claude -p --json-schema '{"type":"object","properties":{"bugs":{"type":"array"}}}' "liste les bugs"

# CI : liste blanche précise, aucune demande
claude -p "lance la suite de tests" --permission-mode dontAsk --allowedTools "Bash(npm test)" "Read"

# Limiter le coût et le nombre de tours
claude -p "refactorise utils.js" --max-budget-usd 5.00 --max-turns 10

# Démarrage rapide et prévisible
claude --bare -p "analyse ce diff" --append-system-prompt "Réponds en français"
```

Flags propres au mode `-p` :

| Flag | Rôle |
|------|------|
| `--output-format text\|json\|stream-json` | Format de sortie |
| `--input-format text\|stream-json` | Format d'entrée |
| `--json-schema` | Sortie JSON validée |
| `--max-turns` | Nombre maximum de tours agentiques |
| `--max-budget-usd` | Plafond de dépense estimée |
| `--include-partial-messages` | Événements de streaming partiels |
| `--include-hook-events` | Événements du cycle de vie des hooks |
| `--forward-subagent-text` | Texte et réflexion des sous-agents dans le flux |
| `--replay-user-messages` | Renvoyer les messages utilisateur en sortie |
| `--prompt-suggestions` | Émettre une suggestion du prochain prompt |
| `--append-subagent-system-prompt[-file]` | Ajouter du texte au prompt système des sous-agents |
| `--exclude-dynamic-system-prompt-sections` | Améliorer la réutilisation du cache entre utilisateurs |

- Arrêter avec SIGTERM → code de sortie 143.
- `/model`, `/effort`, `/config clé=valeur`, `/color`, `/rename`, `/mcp`, `/goal` fonctionnent aussi en `-p`.
- Pour aller plus loin (Python/TypeScript) : l'**Agent SDK** (voir [17](17-sdk-entreprise-couts.md)).

## Personnaliser le prompt système

| Flag | Effet |
|------|-------|
| `--system-prompt` / `--system-prompt-file` | Remplace tout le prompt par défaut (vous perdez les consignes d'outils et de sécurité) |
| `--append-system-prompt` / `--append-system-prompt-file` | Ajoute à la fin du prompt par défaut (recommandé) |
| `--system-prompt-snapshot off` | Reconstruit le prompt à chaque requête (utile pour itérer) |

Pour un persona réutilisable, préférez les **styles de sortie** ; pour des conventions de projet, le **`CLAUDE.md`**.

## Liens profonds `claude-cli://`

Un lien `claude-cli://open?q=<prompt>&cwd=<chemin>` (ou `&repo=owner/nom`) ouvre un terminal avec Claude Code dans le bon dossier et le prompt pré-rempli. Rien n'est envoyé tant que vous n'appuyez pas sur `Entrée` — pratique dans des runbooks, alertes et tableaux de bord.
