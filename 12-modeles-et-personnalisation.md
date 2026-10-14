# 12 — Modèles et personnalisation

[← Sommaire](README.md)

## Choisir le modèle

`/model` (sélecteur ; `←/→` règle l'effort ; `s` = cette session seulement), `Option+P`/`Alt+P`, `claude --model <alias>`, réglage `model`, variable `ANTHROPIC_MODEL`.

| Alias | Modèle (API Anthropic, oct. 2026) | Pour |
|-------|-----------------------------------|------|
| `default` | Valeur par défaut de votre compte | Revenir au réglage par défaut |
| `best` | Fable si disponible, sinon Opus | Le plus capable |
| `fable` | Claude Fable 5.1 | Tâches les plus difficiles et les plus longues |
| `opus` | Claude Opus 5.5 | Raisonnement complexe |
| `sonnet` | Claude Sonnet 5.5 | Codage quotidien |
| `haiku` | Claude Haiku 5.5 | Tâches simples et rapides |
| `sonnet[1m]`, `opus[1m]` | Variante à **1 million de jetons** de contexte (si pas déjà native) | Très longues sessions |
| `opusplan` | Opus en mode plan, Sonnet pour exécuter | Bon compromis |

Chez d'autres fournisseurs (Bedrock, Google Cloud, Foundry), les alias peuvent pointer vers des versions plus anciennes ; on peut fixer un ID complet (ex. `claude-opus-5-5`). Autres options : **modèles de secours** (`--fallback-model`, `fallbackModel`), **bascule automatique** si le modèle est indisponible, liste de modèles autorisés par l'organisation.

## Niveau d'effort (profondeur de réflexion)

| Niveau | Quand l'utiliser |
|--------|------------------|
| `low` | Échanges rapides, brouillons, petits changements (renommage) |
| `medium` | **Défaut** sur Opus/Sonnet/Haiku 5.5 : travail quotidien bien cadré |
| `high` | Bugs dans du code existant, cas limites probables |
| `xhigh` | Raisonnement plus poussé, plus de jetons |
| `max` | Problèmes très durs (ex. recherche de failles), session uniquement |
| `ultracode` | Réglage (pas un niveau) : Claude planifie un **workflow** pour chaque tâche importante |

Régler : `/effort [niveau]` (curseur interactif, `Tab` pour ultracode), `/model` (flèches), `--effort`, `CLAUDE_CODE_EFFORT_LEVEL`, réglages `modelSettings`/`effortLevel`, frontmatter des skills et sous-agents. Le niveau s'affiche dans l'en-tête (« with low effort »).

- **`ultrathink`** dans un prompt : réflexion plus profonde pour ce seul tour.
- **Réflexion étendue** : `Option+T`/`Alt+T` (les modèles 5.x réfléchissent toujours) ; afficher la réflexion avec `Ctrl+O`.

## Mode rapide (`/fast`)

Réponses Opus plus rapides (`↯` à côté du prompt). `/fast`, `Option+O`/`Alt+O`, ou `"fastMode": true`. Tarif plus élevé (ex. 8 $ / 40 $ par million de jetons entrée/sortie sur Opus 5.5) ; sur les abonnements, il est facturé en **crédits d'usage**. Bascule automatiquement sur Opus si le modèle courant ne le prend pas en charge.

## Advisor (`/advisor`)

Associe à votre modèle principal un **conseiller plus fort** (`fable`, `opus`, `sonnet` ou ID), que Claude consulte aux moments clés (plan, erreur récurrente, vérification finale). `/advisor opus`, `/advisor off`, `--advisor`, réglage `advisorModel`.

## Styles de sortie (`/output-style`)

| Style | Effet |
|-------|-------|
| **Default** | Comportement standard d'ingénierie logicielle |
| **Proactive** | Démarre tout de suite, fait des hypothèses raisonnables au lieu de demander |
| **Concise** | Réponses directes, sans préambule ni récapitulatif |
| **Explanatory** | Ajoute des blocs « Insight » qui expliquent les choix |
| **Learning** | Explique et vous laisse écrire certains morceaux de code (`TODO(human)`) |

Styles personnalisés : fichiers Markdown dans `~/.claude/output-styles/` ou `.claude/output-styles/` ; réglage `"outputStyle": "Concise"`.

## Langue

Réglage `"language": "french"` : Claude répond en français (et la dictée vocale utilise cette langue).

## Thèmes et apparence

- `/theme` : `auto` (suit le terminal), clair, sombre, **daltoniens**, **ANSI** (couleurs du terminal), thèmes personnalisés (`~/.claude/themes/` ou plugins) ; « New custom theme… » pour en créer.
- `/color` : couleur de la barre du prompt ; `/rename` : nom affiché.
- `/tui fullscreen` : rendu plein écran ; `/focus` : vue focus.
- Réglages d'affichage : `spinnerVerbs`, `spinnerTipsEnabled`, `showTurnDuration`, `prefersReducedMotion`, `syntaxHighlightingDisabled`, `maxProseWidth`, `timeFormat`, `terminalTitleFromRename`, `companyAnnouncements`…

## Barre d'état (`/statusline`)

Une ligne personnalisée sous le prompt, produite par **votre script** qui reçoit en JSON l'état de la session (modèle, dossier, contexte utilisé, coût, git…).

- `/statusline affiche le modèle, la branche git et le % de contexte` : Claude écrit le script et le configure.
- `/statusline` sans argument : l'adapte à partir de votre prompt shell.
- Manuel : `"statusLine": {"type": "command", "command": "~/.claude/statusline.sh"}`. Supprimer : `/statusline delete`.
- `subagentStatusLine` : ligne d'état pour les sous-agents.

## Raccourcis clavier personnalisés

`/keybindings` ouvre `~/.claude/keybindings.json` (rechargé à chaud) :

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "bindings": [
    { "context": "Chat", "bindings": { "ctrl+e": "chat:externalEditor", "ctrl+s": null } }
  ]
}
```

- **Contextes** : `Global`, `Chat`, `Autocomplete`, `Settings`, `Confirmation`, `Tabs`, `Help`, `Transcript`, `HistorySearch`, `Task`, `ThemePicker`, `Attachments`, `Footer`, `MessageSelector`, `DiffDialog`, `DiffPanel`, `ModelPicker`, `EffortSlider`, `Select`, `Plugin`, `AbovePrompt…`, `Pane`, `Agents`, `Scroll`.
- **Actions** (exemples) : `app:interrupt`, `app:exit`, `app:toggleTodos`, `app:toggleTranscript`, `chat:submit`, `chat:newline`, `chat:cycleMode`, `chat:modelPicker`, `chat:fastMode`, `chat:thinkingToggle`, `chat:stash`, `chat:externalEditor`, `chat:imagePaste`, `chat:sendNow`, `chat:killAgents`…
- Accords (`ctrl+x ctrl+k`), touches spéciales, `null` pour désactiver un raccourci par défaut.

## Fichiers de réglages

| Portée | Fichier | Concerne |
|--------|---------|----------|
| **Géré** (entreprise) | `managed-settings.json`, MDM, ou console claude.ai | Toute l'organisation — non contournable |
| **Ligne de commande** | `--settings <fichier\|json>` | Cette session |
| **Projet local** | `.claude/settings.local.json` | Vous, dans ce projet (hors git) |
| **Projet partagé** | `.claude/settings.json` | L'équipe (à committer) |
| **Utilisateur** | `~/.claude/settings.json` | Vous, partout |

Priorité dans cet ordre (géré > ligne de commande > local > projet > utilisateur). JSON strict (pas de commentaires ni de virgule finale). `/status` montre les sources actives.

Modifier : `/config` (interface : thème, modèle, mode éditeur, verbosité, suggestions, récapitulatif, Remote Control au démarrage…), `/config clé=valeur`, `/update-config` (en langage naturel), ou à la main.

### Réglages notables (sélection)

| Catégorie | Clés |
|-----------|------|
| Modèle | `model`, `effortLevel`, `modelSettings`, `fastMode`, `advisorModel`, `fallbackModel`, `availableModels`, `alwaysThinkingEnabled`, `ultracode`, `outputStyle`, `language` |
| Permissions | `permissions.allow/ask/deny`, `permissions.defaultMode`, `permissions.additionalDirectories`, `disableAutoMode`, `autoMode`, `sandbox.*` |
| Contexte et mémoire | `autoCompactEnabled`, `autoCompactWindow`, `idleCompaction`, `autoMemoryEnabled`, `claudeMdExcludes`, `fileCheckpointingEnabled`, `cleanupPeriodDays`, `env` |
| Interface | `theme`, `editorMode`, `tui`, `viewMode`, `statusLine`, `spellcheck`, `voice`, `promptSuggestionEnabled`, `autoContinueAtUsageLimit`, `respondToBashCommands`, `emojiCompletionEnabled` |
| Git | `attribution.commit`, `attribution.pr`, `includeGitInstructions`, `prUrlTemplate` |
| Extensions | `hooks`, `disableAllHooks`, `enabledPlugins`, `extraKnownMarketplaces`, `skillOverrides`, `disableBundledSkills`, `enableAllProjectMcpServers` |
| Agents | `agent`, `teammateMode`, `worktree.*`, `crossSessionInbound`, `disableAgentView` |
| Distant | `remoteControlAtStartup`, `disableRemoteControl`, `sshConfigs`, `preferredNotifChannel` |
| Mises à jour | `autoUpdatesChannel` (`latest`/`stable`), `minimumVersion` |

### Variables d'environnement utiles

`ANTHROPIC_API_KEY`, `ANTHROPIC_MODEL`, `ANTHROPIC_BASE_URL`, `CLAUDE_CONFIG_DIR`, `CLAUDE_CODE_EFFORT_LEVEL`, `CLAUDE_CODE_SUBAGENT_MODEL`, `CLAUDE_CODE_USE_BEDROCK` / `CLAUDE_CODE_USE_VERTEX`, `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`, `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`, `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`, `DISABLE_TELEMETRY`, `BASH_DEFAULT_TIMEOUT_MS`, `BASH_MAX_TIMEOUT_MS`, `MCP_TIMEOUT`, `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION`, `CLAUDE_CODE_TASK_LIST_ID`… (on peut aussi les poser dans la clé `env` des réglages).

## Configuration du terminal

- `/terminal-setup` : `Shift+Entrée` pour les retours à la ligne (VS Code, Cursor, Alacritty, Zed…), Option comme Meta sur Terminal Apple.
- macOS : régler **Option = Meta** dans iTerm2/Terminal pour les raccourcis `Alt`.
- Notifications de fin de tâche, barre de progression du terminal (`terminalProgressBarEnabled`), titre du terminal.
- Lecteurs d'écran : `--ax-screen-reader`.
