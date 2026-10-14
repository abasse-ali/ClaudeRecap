# 05 — Les outils intégrés de Claude

[← Sommaire](README.md)

Les **outils** sont les « mains » de Claude : c'est par eux qu'il lit, écrit, exécute et communique. Vous ne les appelez pas vous-même ; Claude les choisit. Vous contrôlez leur usage avec les **permissions** (voir [06](06-permissions-securite-sandbox.md)) et les **hooks** (voir [10](10-hooks-et-automatisation.md)). Les noms ci-dessous sont ceux à utiliser dans les règles de permission (`Bash(...)`, `Edit(...)`…), les matchers de hooks et les champs `tools` des sous-agents.

## Fichiers et code

| Outil | Rôle | Permission ? |
|-------|------|:------------:|
| `Read` | Lit un fichier (avec numéros de ligne). Gère aussi les **images** (Claude les « voit »), les **PDF** (par plages de 20 pages max pour les longs), les **notebooks Jupyter** | Non |
| `Write` | Crée ou écrase un fichier | Oui |
| `Edit` | Remplacements ciblés dans un fichier | Oui |
| `NotebookEdit` | Modifie des cellules de notebook Jupyter | Oui |
| `Glob` | Trouve des fichiers par motif (`**/*.ts`) — absent par défaut sur macOS/Linux/WSL, où Claude utilise `find` via Bash | Non |
| `Grep` | Cherche dans le contenu des fichiers — idem, remplacé par `grep` (ripgrep intégré) via Bash sur macOS/Linux/WSL | Non |
| `LSP` | **Intelligence de code** via un serveur de langage : aller à la définition, trouver les références, types, erreurs signalées après chaque modification (nécessite un plugin de code intelligence) | Non |

## Exécution

| Outil | Rôle | Permission ? |
|-------|------|:------------:|
| `Bash` | Exécute des commandes shell. Délai par défaut 2 min (max 10 min, réglable) ; peut passer en arrière-plan (`Ctrl+B`) ; le dossier courant persiste entre les commandes | Oui (sauf commandes en lecture seule intégrées) |
| `PowerShell` | Exécute des commandes PowerShell nativement (Windows, selon disponibilité) | Oui |
| `Monitor` | Lance une commande en arrière-plan et renvoie chaque ligne de sortie à Claude (surveiller un log, une PR, un dossier, un WebSocket) pour qu'il réagisse en cours de conversation | Oui |

## Web

| Outil | Rôle | Permission ? |
|-------|------|:------------:|
| `WebSearch` | Recherche web (titres et URL) ; filtrage par domaines autorisés/bloqués | Oui |
| `WebFetch` | Récupère une URL, la convertit en Markdown et en extrait ce qui est demandé (certains domaines de documentation sont pré-approuvés) | Oui |

## Agents et orchestration

| Outil | Rôle | Permission ? |
|-------|------|:------------:|
| `Agent` | Lance un **sous-agent** avec son propre contexte (ou un coéquipier d'une équipe d'agents) | Non |
| `SendMessage` | Envoie un message à un coéquipier, à un sous-agent repris ou à une autre session Claude Code | — |
| `ListAgents` | Liste les agents et sessions joignables (base de `/list-agents`) | — |
| `SubagentHandback` | Remet le rapport final d'un sous-agent (mode auto) | — |
| `TaskStop` | Arrête une tâche d'arrière-plan, un coéquipier ou un agent | Non |
| `Workflow` | Exécute un **workflow dynamique** (script qui orchestre de nombreux sous-agents) | Oui |
| `Skill` | Exécute un skill dans la conversation | Oui |
| `ToolSearch` | Recherche et charge à la demande des outils « différés » (utile avec beaucoup d'outils MCP) | Non |
| `WaitForMcpServers` | Attend qu'un serveur MCP encore en connexion soit prêt | Non |

## Planification et suivi

| Outil | Rôle | Permission ? |
|-------|------|:------------:|
| `EnterPlanMode` / `ExitPlanMode` | Entrer en mode plan / présenter le plan pour approbation | Non / Oui |
| `TaskCreate`, `TaskGet`, `TaskList`, `TaskUpdate` | La **liste de tâches** de Claude (checklist visible avec `Ctrl+T`) | Non |
| `TodoWrite` | Ancienne checklist (désactivée par défaut) | Non |
| `CronCreate`, `CronDelete`, `CronList` | Tâches planifiées **dans la session** (base de `/loop`) | Non |
| `ScheduleWakeup` | Choisit quand relancer la prochaine itération d'un `/loop` auto-rythmé | — |
| `RemoteTrigger` | Crée, met à jour, lance et liste les **routines** cloud (base de `/schedule`) | — |
| `EnterWorktree` / `ExitWorktree` | Créer/entrer dans un **worktree git** isolé, et en sortir | Non |

## Interaction avec vous

| Outil | Rôle | Permission ? |
|-------|------|:------------:|
| `AskUserQuestion` | Pose des **questions à choix multiples** (avec réponse libre « Other ») | Non |
| `PushNotification` | Notification de bureau, et push sur téléphone si Remote Control est connecté | Non |
| `SendUserFile` | Vous envoie un fichier (rapport, capture, diagramme…) affiché ou en pièce jointe | — |
| `Artifact` | Publie un fichier HTML/Markdown comme **artifact** (page privée et partageable sur claude.ai) | — |
| `SendFeedback` | Rédige un retour sur Claude Code que vous relisez avant envoi | Non |
| `ReportFindings` | Restitue les résultats d'une revue de code sous forme de liste structurée | Non |
| `ShareOnboardingGuide` | Partage le guide généré par `/team-onboarding` | Oui |
| `EndConversation` | Met fin à la session (cas rares d'abus persistant) | Non |

## MCP

| Outil | Rôle |
|-------|------|
| `ListMcpResourcesTool` | Liste les ressources exposées par les serveurs MCP |
| `ReadMcpResourceTool` | Lit une ressource MCP par son URI |
| `mcp__<serveur>__<outil>` | Chaque outil fourni par un serveur MCP connecté (ex. `mcp__github__create_pull_request`) |

## Commandes en arrière-plan

- Claude peut lancer une commande en arrière-plan, ou vous appuyez sur `Ctrl+B` pendant qu'elle tourne.
- Une commande qui atteint son délai passe automatiquement en arrière-plan (sauf `sleep`).
- La sortie est écrite dans un fichier que Claude relit ; `/tasks` liste et arrête les tâches.
- Les tâches de fond sont arrêtées à la sortie de Claude Code (sauf si vous passez la session elle-même en arrière-plan).
- Un appel d'outil MCP qui dure plus de 2 minutes passe aussi en arrière-plan.

## Voir les outils disponibles

Demandez simplement à Claude « quels outils as-tu ? », ou consultez `/permissions`, `/mcp` (outils par serveur), `/context` (coût en contexte des outils).

## Restreindre les outils

```bash
claude --tools "Bash,Edit,Read"          # seulement ces outils intégrés
claude --disallowedTools "WebFetch" "mcp__*"   # retirer des outils
```

Ou dans les réglages : une règle `deny` avec un nom d'outil seul (ex. `"WebSearch"`) le retire complètement du contexte de Claude.
