# 28 — Glossaire

[← Sommaire](README.md)

| Terme | Définition |
|-------|------------|
| **AGENTS.md** | Fichier d'instructions de projet destiné aux agents de codage (format partagé avec d'autres outils). Lu par Claude Code comme instructions de projet, notamment en l'absence de `CLAUDE.md`. |
| **Advisor** | Second modèle, plus fort, que Claude consulte aux moments clés d'une tâche (`/advisor`). |
| **Agent view** | Écran qui liste toutes vos sessions Claude Code d'arrière-plan (`claude agents`). |
| **Agent teams (équipes d'agents)** | Plusieurs sessions Claude Code indépendantes coordonnées par un chef d'équipe, avec une liste de tâches partagée et une messagerie entre pairs. |
| **Agentic coding (codage agentique)** | L'IA lit des fichiers, lance des commandes et fait des modifications de manière autonome pendant que vous observez, réorientez ou vous absentez. |
| **Agentic harness (harnais)** | Outils, gestion du contexte et environnement d'exécution qui transforment un modèle en agent : Claude Code est le harnais, Claude est le modèle. |
| **Agentic loop (boucle agentique)** | Cycle de chaque tâche : rassembler du contexte, agir, vérifier, recommencer. |
| **Artifact** | Page web interactive publiée par Claude Code depuis la session sur une URL privée de claude.ai ; partageable. |
| **Auto memory (mémoire automatique)** | Notes que Claude écrit lui-même (préférences, corrections), stockées par dépôt sous `~/.claude/projects/`. |
| **Auto mode** | Mode de permission où un classifieur examine les actions à votre place. |
| **Background session** | Session qui tourne en arrière-plan, pilotée depuis l'agent view. |
| **Bare mode** | Démarrage avec `--bare` sans hooks, skills, plugins, MCP, mémoire ni `CLAUDE.md` (pour la CI). |
| **Bundled skills** | Skills fournis avec Claude Code (`/batch`, `/code-review`, `/debug`, `/loop`…). |
| **Channel** | Serveur MCP qui pousse des événements dans votre session en cours (et peut répondre dans le même canal). |
| **Checkpoint (point de contrôle)** | Point de restauration créé à chaque prompt ; `Esc` `Esc` ou `/rewind`. |
| **Claude Tag** | Claude dans les canaux Slack d'une organisation (Team/Enterprise), sous une identité partagée gérée par les admins. |
| **Cloud session** | Session qui continue sur une infrastructure cloud (Anthropic ou auto-hébergée) quand vous fermez votre ordinateur. |
| **Command (commande)** | Instruction réutilisable invoquée par `/nom` ; intégrée, ou définie par vous (skills, `.claude/commands/`). |
| **Compaction** | Résumé automatique de la conversation quand le contexte approche de sa limite. |
| **Connector (connecteur)** | Serveur MCP ajouté à votre compte claude.ai, disponible dans Claude Code quand vous êtes connecté avec ce compte. |
| **Context window (fenêtre de contexte)** | Mémoire de travail d'une session : conversation, fichiers lus, sorties de commandes, `CLAUDE.md`, skills, instructions système. |
| **Cowork** | Onglet de l'app Claude pour le travail agentique long et Dispatch. |
| **Dispatch** | Routeur de tâches lancé depuis le téléphone qui ouvre une session Code dans l'app Desktop (Pro/Max). |
| **Dynamic workflow (workflow dynamique)** | Script écrit par Claude qui orchestre de nombreux sous-agents en arrière-plan. |
| **Effort level (niveau d'effort)** | Réglage de la profondeur de réflexion (`low` → `max`). |
| **Extended thinking (réflexion étendue)** | Raisonnement pas à pas visible (en gris italique) avant la réponse. |
| **Fast mode (mode rapide)** | Réponses Opus plus rapides, à un tarif plus élevé. |
| **Fork** | Copie d'une conversation : `/fork` (nouvelle session de fond), `/subtask` (sous-agent forké), `--fork-session`. |
| **Frontmatter** | Bloc YAML entre deux lignes `---` en tête d'un fichier Markdown (skills, sous-agents, styles, règles). |
| **Hook** | Action exécutée automatiquement à un moment du cycle de vie : commande shell, requête HTTP, outil MCP, prompt LLM ou sous-agent. |
| **Managed settings (réglages gérés)** | Réglages imposés par l'organisation, non modifiables par l'utilisateur. |
| **Marketplace** | Catalogue de plugins (fichier `marketplace.json` dans un dépôt). |
| **MCP (Model Context Protocol)** | Standard ouvert pour connecter des outils d'IA à des sources de données et services externes. |
| **MCP server** | Programme qui fournit des outils, prompts ou ressources via MCP (local stdio ou distant HTTP). |
| **MCP Tool Search** | Chargement différé des schémas d'outils MCP pour économiser le contexte. |
| **Mod** | Plugin dont le code JavaScript s'exécute dans Claude Code pour modifier son interface et son comportement. |
| **Non-interactive mode (mode non interactif)** | Exécution d'un prompt unique avec `-p`/`--print` (CI, scripts, pipes). |
| **Output style (style de sortie)** | Configuration qui change le rôle, le ton ou le format des réponses. |
| **Permission mode (mode de permission)** | Comportement d'approbation de base : `default` (Manual), `acceptEdits`, `plan`, `auto`, `dontAsk`, `bypassPermissions`. |
| **Permission rule (règle de permission)** | Entrée allow/ask/deny selon l'outil et ses arguments (ordre deny → ask → allow). |
| **Plan mode** | Mode où Claude étudie et propose un plan sans modifier vos fichiers. |
| **Plugin** | Paquet installable regroupant skills, hooks, sous-agents, serveurs MCP… |
| **Project (projet)** | Conversation de coordination qui lance et suit des fils de sessions cloud parallèles. |
| **Project trust (confiance)** | Acceptation d'un dossier avant que Claude Code charge sa configuration exécutable. |
| **Prompt caching (cache de prompt)** | Réutilisation par l'API du début identique des requêtes, pour aller plus vite et coûter moins. |
| **Prompt injection** | Instructions hostiles cachées dans un fichier, une page web ou un résultat d'outil. |
| **Remote Control** | Continuer une session locale depuis le téléphone ou un navigateur ; l'exécution reste sur votre machine. |
| **Routine** | Tâche automatisée dans le cloud (planifiée, déclenchée par API ou par GitHub). |
| **Rules (règles)** | Fichiers d'instructions modulaires dans `.claude/rules/`, éventuellement limités à des chemins. |
| **Sandboxing** | Isolation (fichiers et réseau) des commandes Bash au niveau du système d'exploitation. |
| **Session** | Conversation liée à un dossier, avec sa propre fenêtre de contexte. |
| **Settings layers (couches de réglages)** | Ordre de priorité : géré > `--settings` > local > projet > utilisateur. |
| **Skill** | Fichier `SKILL.md` (instructions, connaissances ou workflow) chargé automatiquement ou via `/nom`. |
| **Subagent (sous-agent)** | Assistant spécialisé avec son propre contexte, son prompt, ses outils ; renvoie un résumé. |
| **Surface** | Endroit d'où l'on utilise Claude Code : CLI, VS Code, JetBrains, Desktop, claude.ai, mobile. |
| **System prompt** | Instructions envoyées avant la conversation à chaque requête. |
| **System reminder (rappel système)** | Message que Claude Code ajoute à la conversation pour donner du contexte à Claude. |
| **Teleport** | Ramener une session cloud dans votre terminal (`/teleport`, `--teleport`). |
| **Tool (outil)** | Action que Claude peut effectuer : lire, modifier, exécuter, chercher, lancer un sous-agent… |
| **Transcript (transcription)** | Enregistrement d'une session (`~/.claude/projects/<projet>/<session>.jsonl`). |
| **Turn (tour)** | Une réponse complète de Claude, du message envoyé jusqu'à la fin de sa réponse (avec ses appels d'outils). |
| **Ultracode** | Réglage qui fait planifier un workflow multi-agents pour chaque tâche importante (ou mot-clé dans un prompt). |
| **Ultrareview** | Revue de code approfondie multi-agents dans le cloud. |
| **Ultrathink** | Mot-clé dans un prompt pour une réflexion plus profonde sur ce tour. |
| **Verification loop (boucle de vérification)** | Donner à Claude un moyen de vérifier (tests, build, capture) pour qu'il itère jusqu'à réussite. |
| **Worktree isolation** | Exécution dans un worktree git séparé (`-w`, `isolation: worktree`) pour éviter les conflits. |
