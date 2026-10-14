# 30 — Nouveautés et historique (2026)

[← Sommaire](README.md)

Claude Code publie plusieurs versions par semaine. `/release-notes` affiche le journal des modifications dans la session ; la doc publie un résumé hebdomadaire (code.claude.com/docs/en/whats-new) et le changelog complet.

## Dernières versions (septembre–octobre 2026)

### Modèles

| Version | Date | Nouveauté |
|---------|------|-----------|
| 2.1.293 | 7 oct. | **Claude Haiku 5.5** (`claude-haiku-5-5`) devient le Haiku par défaut : 1 M de contexte, 0,10 $ / 0,50 $ par million de jetons |
| 2.1.284 | 28 sept. | **Claude Sonnet 5.5** (`claude-sonnet-5-5`) devient le Sonnet par défaut : 1 M de contexte, 2 $ / 10 $ |
| 2.1.280 | 22 sept. | **Claude Opus 5.5** (`claude-opus-5-5`) devient l'Opus par défaut : 1 M de contexte, 4 $ / 20 $ |
| (sem. 36) | sept. | **Claude Fable 5.1** disponible (`/model fable`) |

### Fonctions marquantes

| Version | Nouveauté |
|---------|-----------|
| 2.1.299 | **Compaction en veille** : une conversation de plus de 200 000 jetons inactive ~54 min est compactée avant l'expiration du cache d'1 h ; `/effort` et `/fast` dans l'agent view ; avertissements `/usage` (effort max, workflows, contexte permanent, gros résultats d'outils) ; `CLAUDE_CODE_DISABLE_FILE_MENTIONS` |
| 2.1.296 | `autoCompactWindow` par sous-agent ; option `allow_large` de l'outil Read (lire un gros fichier en une fois) ; `CLAUDE_CODE_WORKFLOW_SUBAGENT_MODEL` |
| 2.1.295 | Hooks `onFailure: "block"` ; prise en charge du **Program Status Protocol (OSC 7501)** (le terminal affiche si Claude travaille, attend ou a fini) ; VS Code : ligne cliquable pour les fichiers envoyés par Claude |
| 2.1.292 | `claude plugin install --marketplace <source>` ; paramètre `effort` de l'outil Agent (sous-agent à un effort donné) |
| 2.1.290 | `claude attach <nom>` / `claude logs <nom>` ; `/claude-api managed-agents-onboard <url \| modèle>` ; Claude Tag : `!fast` dans Slack |
| 2.1.288 | `↑` récupère un brouillon effacé par `Ctrl+C` ; `/code-review --max-findings` ; `Ctrl+F` (recherche) et `Alt+↑/↓` (groupes) dans l'agent view ; nouvelle authentification MCP quand un serveur demande plus de droits |
| 2.1.287 | Filtre `n:` dans l'agent view ; **mods** (modifier l'interface) ; VS Code : « Run in background » |
| 2.1.286 | Compteur « 2 sur 5 » quand plusieurs demandes de permission s'empilent ; VS Code : **signets**, historique des questions/réponses, aperçus des options |
| 2.1.285 | `claude --desktop` ; `claude plugin configure` ; `CLAUDE_CODE_DISABLE_WEB_FETCH` ; VS Code : lecture à la demande du panneau Problems |
| 2.1.284 | `/mcp reconnect all` ; réponse « Oui, mais redemande la prochaine fois » pour les lectures hors du dossier en mode auto ; raccourcis du curseur `/effort` personnalisables ; ultracode indépendant du niveau d'effort |
| 2.1.283 | `/doctor prompt-audit` ; `availableModelsMatch`, `deniedModels` ; **mode auto par défaut** dans le terminal et VS Code |
| 2.1.282 | `maxProseWidth` (largeur du texte dans un terminal large) ; sessions cloud : état de l'app GitHub, dépôts d'un autre propriétaire |
| 2.1.281 | `"attribution": false` (aucune mention dans les commits/PR) ; recommandation du mode auto dans `/insights` ; barres de défilement en plein écran ; mode rapide dans les sessions cloud |
| 2.1.277 | **Prise en charge d'`AGENTS.md`** ; VS Code : déconnexion, carte des tâches de fond, bouton Copier, archivage automatique |
| 2.1.275 | `Ctrl+Entrée` / `Ctrl+X Ctrl+S` pour envoyer la file d'attente immédiatement |
| 2.1.269 | Évaluations de plugins (`claude plugin eval`) |
| 2.1.265 | `/design`, `/slides` ; import depuis Cursor |

## Résumés hebdomadaires

| Semaine | Points forts |
|---------|--------------|
| **37** (7–11 sept.) | Tester ses plugins avec `claude plugin eval` ; détacher les volets de Desktop dans leurs propres fenêtres |
| **36** (31 août–4 sept.) | **Claude Fable 5.1** ; computer use en arrière-plan sur Desktop ; panneau **`/diff`** en direct ; `/skill-doctor` |
| **35** (24–28 août) | Reprendre des sessions du terminal dans l'app Desktop ; relire les retours rédigés par Claude (`/feedback`) ; **mode restreint** (`--restricted`) |
| **34** (17–21 août) | Skill **`/design`** (maquettes éditables) ; style de sortie **Concise** ; démarrer une session sur sa machine depuis le téléphone (`claude remote-control`) |
| **33** (10–14 août) | Desktop reprend seul après une limite d'usage ; **mode fork par défaut** ; merge requests et marketplaces **GitLab** |
| **32** (3–7 août) | **Messagerie entre sessions** ; **environnements auto-hébergés** ; le mode auto devient le mode de permission par défaut |
| **30** (20–24 juil.) | **Opus 5** par défaut ; volet **simulateur iOS** dans Desktop ; plugin **Claude Security** |
| **29** (13–17 juil.) | Données en direct dans les artifacts via les connecteurs MCP ; **mode lecteur d'écran** |
| **28** (6–10 juil.) | **Navigateur intégré** de Desktop pour les sites externes ; **`/doctor`** complet ; améliorations de l'agent view |
| **27** (29 juin–3 juil.) | **Sonnet 5** par défaut ; **Claude in Chrome** en disponibilité générale ; sous-agents en arrière-plan par défaut ; **Desktop sur Linux** (bêta) ; `/radio` |
| **26** (22–26 juin) | `claude mcp login` ; Claude répond aux commandes `!` ; `/rewind` revient avant un `/clear` |
| **25** (15–19 juin) | **Artifacts** (pages partageables) ; règles deny/ask par paramètre d'outil ; `/config clé=valeur` |
| **24** (8–12 juin) | **`/cd`** ; sous-agents qui lancent des sous-agents ; **mode sans échec** (`--safe-mode`) |
| **23** (1–5 juin) | Mode auto sur Bedrock, Google Cloud et Foundry ; demande avant d'écrire des fichiers exécutables en acceptEdits ; `/plugin list` ; plage de versions imposée par l'admin |
| **22** (25–29 mai) | **Opus 4.8** ; **workflows dynamiques** ; plugin **security-guidance** ; mode rapide moins cher sur Opus 4.8 |
| **21** (18–22 mai) | Mode auto sur Pro et Sonnet 4.6 ; ventilation de l'usage par skill/agent/MCP dans `/usage` ; nouvelle commande **`/code-review`** |
| **20** (11–15 mai) | **Agent view** ; **`/goal`** ; mode rapide sur Opus 4.7 |
| **19** (4–8 mai) | Plugins en `.zip` et par URL ; `Ctrl+R` dans tous les projets ; base des worktrees configurable ; règles « hard deny » du mode auto |
| **18** (27 avr.–1er mai) | Windows sans Git Bash ; code OAuth collé quand le navigateur ne peut pas joindre localhost ; nettoyage local par projet ; retrouver une session à partir d'une URL de PR dans `/resume` |
| **16** (13–17 avr.) | **Opus 4.7** et niveau d'effort **xhigh** ; **Routines** ; notifications push mobiles ; ventilation de `/usage` ; binaires natifs |
| **15** (6–10 avr.) | Ultraplan (depuis retiré) ; outil **Monitor** et `/loop` auto-rythmé ; `/team-onboarding` ; `/autofix-pr` |
| **14** (30 mars–3 avr.) | **Computer use dans le CLI** ; leçons interactives (`/powerup`) ; rendu sans scintillement ; exécutables de plugins dans le `PATH` |
| **13** (23–27 mars) | **Mode auto** ; computer use intégré ; auto-fix des PR dans le cloud ; recherche dans la transcription ; outil **PowerShell** pour Windows |

## Fonctions retirées ou remplacées

| Ancien | Remplacement |
|--------|--------------|
| `/ultraplan` | Mode plan |
| `/pr-comments` | Demander directement à Claude |
| `/vim` | `/config` › Editor mode |
| `/review` (ancienne revue de PR) | Alias de `/code-review` |
| `/agents` (interface de gestion) | Demander à Claude ou éditer `.claude/agents/` |
| `/extra-usage` | `/usage-credits` |
| Outil `TaskOutput` | Lecture du fichier de sortie avec `Read` |
| Outil `TodoWrite` | Outils `TaskCreate`/`TaskList`/`TaskUpdate`/`TaskGet` |
| `--enable-auto-mode` | `--permission-mode auto` (le mode auto est dans le cycle `Shift+Tab`) |
| `--remote` | `--cloud` |
| `keybindingFlavor`, `includeCoAuthoredBy`, `disableArtifact` | Sans effet / `attribution` / `enableArtifact` |
