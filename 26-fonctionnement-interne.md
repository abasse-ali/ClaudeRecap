# 26 — Fonctionnement interne : boucle, contexte, cache, fichiers, terminal

[← Sommaire](README.md)

## 1. La boucle agentique

Pour chaque tâche, Claude enchaîne trois phases qui se mélangent : **rassembler du contexte** → **agir** → **vérifier**. Exemple pour « corrige les tests qui échouent » : lancer la suite de tests, lire l'erreur, chercher les fichiers concernés, les lire, les modifier, relancer les tests.

| Catégorie d'outils | Ce que Claude peut faire |
|--------------------|--------------------------|
| Fichiers | Lire, modifier, créer, renommer, réorganiser |
| Recherche | Trouver des fichiers par motif, chercher par regex, explorer le code |
| Exécution | Commandes shell, serveurs, tests, git |
| Web | Rechercher, lire de la documentation, chercher un message d'erreur |
| Intelligence de code | Erreurs de type après modification, définitions, références (plugins LSP) |

**Ce à quoi Claude a accès** quand vous lancez `claude` dans un dossier : les fichiers du dossier et des sous-dossiers (et des dossiers ajoutés), votre terminal et ses outils (git, gh, npm, docker…), l'état git, les `CLAUDE.md`, la mémoire automatique, les extensions configurées (MCP, skills, hooks, plugins).

**Environnements d'exécution** : **local** (votre machine, par défaut), **cloud** (VM gérées par Anthropic ou environnements auto-hébergés), **Remote Control** (votre machine pilotée depuis un navigateur ou le téléphone).

## 2. Le contexte

### Ce qui se charge automatiquement

Prompt système et définitions d'outils → `CLAUDE.md` (organisation, utilisateur, projet, local) et règles sans `paths` → mémoire automatique (200 lignes / 25 Ko) → descriptions des skills et des sous-agents → instantané `git status` → puis la conversation (vos messages, réponses, résultats d'outils, fichiers lus).

### Ce qui survit à une compaction

| Élément | Après compaction |
|---------|------------------|
| Prompt système, style de sortie | Toujours appliqués |
| `CLAUDE.md` racine et règles sans `paths` | Réinjectés depuis le disque |
| Mémoire automatique | Réinjectée |
| Instantané git | Relu à neuf |
| Plan du mode plan | Réinjecté |
| Règles `paths:` et `CLAUDE.md` imbriqués | Rechargés à la demande |
| Fichiers lus ou modifiés | Les 5 plus récents relus |
| Skills invoqués | Réinjectés (5 000 jetons par skill, 25 000 au total) |
| Commandes et sous-agents de fond | Continuent de tourner |
| Contexte ajouté par des hooks | Résumé avec le reste |
| Hooks `SessionStart` avec matcher `compact` | Réexécutés, sortie ajoutée |

### Outils d'observation

`/context` (grille par catégorie, fichiers chargés, suggestions), `/memory`, indicateur de contexte (VS Code, Desktop), barre d'état (`context_window.used_percentage`), `/usage`.

## 3. Le cache de prompt

À chaque message, Claude Code renvoie **tout** le contexte au modèle ; le **cache** évite de le retraiter. Il est organisé en couches :

| Couche | Contenu | Change quand… |
|--------|---------|---------------|
| Prompt système | Instructions, définitions d'outils | L'ensemble des outils change |
| Contexte du projet | `CLAUDE.md`, mémoire, règles | Au démarrage, après `/clear` ou `/compact` |
| Conversation | Messages et résultats | À chaque tour |

**Ce qui invalide le cache** (un tour plus lent et plus cher) : changer de **modèle** (chaque modèle a son cache), changer d'**effort**, activer le **mode rapide**, **connecter ou retirer** un serveur MCP (ou un plugin qui apporte des serveurs MCP), retirer volontairement un outil, compacter. Les skills, commandes, agents, hooks, moniteurs et thèmes d'un plugin **n'invalident pas** le cache. Claude Code demande confirmation avant certains de ces changements quand le cache est chaud.

- Durée de vie : **5 minutes** ou **1 heure** (`CLAUDE_CODE_PROMPT_CACHE_TTL`, `promptCacheTtl`, `ENABLE_PROMPT_CACHING_1H`).
- Les modifications de `CLAUDE.md` en cours de session ne s'appliquent qu'après `/clear`, `/compact` ou une nouvelle session.
- Statistiques : ligne « Prompt cache (main) » de `/usage`, champ `prompt_cache` de la barre d'état (taux de succès, ratés, cause du dernier raté), horloge du cache dans VS Code.
- **Compaction en veille** : une longue conversation inactive est compactée juste avant l'expiration du cache d'1 h.

## 4. Les fichiers locaux de Claude Code

### Configuration (à vous)

`~/.claude/settings.json`, `~/.claude/CLAUDE.md`, `~/.claude/rules/`, `~/.claude/skills/`, `~/.claude/agents/`, `~/.claude/commands/`, `~/.claude/output-styles/`, `~/.claude/themes/`, `~/.claude/keybindings.json`, `~/.claude.json` (config globale, MCP utilisateur/local, état des projets) ; côté projet `.claude/` et `CLAUDE.md`, `.mcp.json`, `.worktreeinclude`, `.claude/launch.json` (Desktop).

### Données nettoyées automatiquement (après `cleanupPeriodDays`, 30 jours par défaut)

| Chemin sous `~/.claude/` | Contenu |
|--------------------------|---------|
| `projects/<projet>/<session>.jsonl` | Transcription complète |
| `projects/<projet>/<session>/subagents/` | Transcriptions des sous-agents |
| `projects/<projet>/<session>/tool-results/` | Sorties d'outils volumineuses |
| `file-history/<session>/` | Instantanés pour `/rewind` |
| `plans/` | Fichiers de plan |
| `debug/` | Journaux de débogage |
| `paste-cache/`, `image-cache/`, `uploads/` | Collages, images, pièces jointes Remote Control |
| `dev-mods/<session>/` | Mods écrits par Claude dans la session |
| `tasks/` | Listes de tâches |
| `shell-snapshots/` | Alias et fonctions du shell capturés |
| `backups/` | Anciennes versions de `~/.claude.json` |
| `feedback-bundles/`, `feedback/drafts/` | Rapports de bug et brouillons de retours |
| `usage-data/` | Rapports `/insights` |

### Données conservées

`history.jsonl` (tous vos prompts : `↑`, `Ctrl+R`), `stats-cache.json` (`/usage`), `remote-settings.json` (réglages serveur en cache), `cache/changelog.md`, `policy-limits.json`, `projects/<projet>/memory/` (mémoire automatique), `daemon.log` et `jobs/` (sessions d'arrière-plan).

Chaque session a aussi un **dossier de brouillon** (scratchpad) temporaire pour les fichiers de travail. Les données sont stockées en clair : `claude purge` efface tout pour un projet.

## 5. La barre d'état : données disponibles

Votre script reçoit en JSON sur stdin (et peut afficher plusieurs lignes, des couleurs ANSI, des liens OSC 8) :

| Champ | Contenu |
|-------|---------|
| `model.id`, `model.display_name` | Modèle |
| `workspace.current_dir`, `project_dir`, `added_dirs`, `git_worktree`, `repo.host/owner/name` | Dossiers et dépôt |
| `cost.total_cost_usd`, `total_duration_ms`, `total_api_duration_ms`, `total_lines_added/removed` | Coût, durée, lignes modifiées |
| `context_window.used_percentage`, `remaining_percentage`, `context_window_size`, `current_usage` | Contexte |
| `exceeds_200k_tokens`, `fast_mode`, `effort.level`, `thinking.enabled` | État du modèle |
| `rate_limits.five_hour.*`, `rate_limits.seven_day.*`, `rate_limits.spend_limit.*` | Limites d'usage (%, réinitialisation) |
| `prompt_cache.*` | Statistiques du cache (`hit_ratio`, `misses`, `last_miss_cause`, `expires_at`…) |
| `session_id`, `session_name`, `prompt_id`, `transcript_path`, `version` | Session |
| `output_style.name`, `vim.mode`, `agent.name` | Style, mode Vim, agent |
| `pr.number`, `pr.url`, `pr.review_state`, `pr.kind` | Pull request / merge request ouverte |
| `worktree.name`, `path`, `branch`, `original_cwd`, `original_branch` | Worktree |

Exemples officiels : usage du contexte, état git en couleurs, suivi coût/durée, plusieurs lignes, liens cliquables, limites d'usage, mise en cache des opérations coûteuses, Windows. `subagentStatusLine` personnalise aussi les lignes des sous-agents.

## 6. Configuration du terminal

| Besoin | Solution |
|--------|----------|
| Nouvelle ligne | `Ctrl+J` ou `\` + `Entrée` partout ; `Shift+Entrée` natif dans Ghostty, Kitty, iTerm2, WezTerm, Warp, Terminal Apple, Windows Terminal, foot, Alacritty ≥ 0.16 ; `/terminal-setup` pour VS Code, Cursor, Zed… |
| Raccourcis Option (macOS) | Régler Option = Meta (iTerm2, Terminal, Ghostty…) |
| Être prévenu | Cloche/notification quand Claude finit ou attend (`preferredNotifChannel`), hook `Notification` pour un son, protocole OSC 7501 (état des sessions dans le terminal) |
| tmux | Dans `~/.tmux.conf` : `set -g allow-passthrough on` (notifications, barre de progression), `set -s extended-keys on` et `set -as terminal-features 'xterm*:extkeys'` (`Shift+Entrée`) |
| Windows : Retour arrière supprime un mot | `CLAUDE_CODE_BS_AS_CTRL_BACKSPACE` |
| Couleurs | `/theme` (auto, clair/sombre, daltonien, ANSI) ; thème personnalisé JSON dans `~/.claude/themes/` (`name`, `base`, `overrides` des jetons de couleur) |
| Gros collages | Repliés en `[Pasted text #N]` ; contenu conservé dans `paste-cache/` |

## 7. Accessibilité

- **Mode lecteur d'écran** (VoiceOver, NVDA…) : `--ax-screen-reader`, `CLAUDE_AX_SCREEN_READER=1` ou `axScreenReader: true`. Texte linéaire sans bordures, préfixé : `you:`, `claude:`, `thinking:`, `tool:`, `tool error:`, `error:`, `warning:`, `Permission Required:`, `Cost:`.
- **Loupes d'écran** : `CLAUDE_CODE_ACCESSIBILITY=1` (curseur natif visible).
- **Mouvements réduits** : `prefersReducedMotion`.
- **Daltonisme** : thèmes `dark-daltonized`, `light-daltonized`.
- **Notifications** : `preferredNotifChannel: "terminal_bell"`.
- VS Code : navigation au clavier du panneau, commande « Focus last message ».

## 8. Grands dépôts et monorepos

| Objectif | Moyen |
|----------|-------|
| Charger seulement les conventions utiles | `CLAUDE.md` par dossier ou règles `.claude/rules/` avec `paths:` |
| Ignorer les `CLAUDE.md` de paquets inutiles | `claudeMdExcludes` |
| Empêcher la lecture du code généré ou vendored | Règles `Read` dans `permissions.deny` |
| Naviguer par symboles | Plugin d'intelligence de code (LSP) |
| Worktrees légers | `worktree.sparsePaths`, `worktree.symlinkDirectories` |
| Travailler sur un paquet voisin | `--add-dir`, `additionalDirectories` |
| Procédures propres à un sous-dossier | Skills imbriqués (`apps/web:deploy`) |
| Conventions communes à toute l'entreprise | Un plugin interne |

Démarrer à la racine donne accès à tout (CLAUDE.md des sous-dossiers chargés à la demande) ; démarrer dans un sous-dossier limite l'accès à cette partie.
