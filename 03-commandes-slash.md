# 03 — Toutes les commandes slash (`/`)

[← Sommaire](README.md)

Tapez `/` dans le prompt pour voir les commandes disponibles, puis des lettres pour filtrer. Une commande n'est reconnue **qu'en début de message** ; le texte qui suit devient ses arguments. Notation : `<arg>` obligatoire, `[arg]` facultatif.

- **(Skill)** = skill intégré : un prompt détaillé confié à Claude (il orchestre le travail avec ses outils).
- **(Workflow)** = workflow intégré qui lance de nombreux sous-agents en arrière-plan.
- Toutes les commandes ne s'affichent pas pour tout le monde : cela dépend de la plateforme, du forfait et du fournisseur. Quelques commandes sont **cachées** du menu et ne fonctionnent que tapées en entier (ex. `/heapdump`).
- Si vous envoyez une commande pendant que Claude répond, elle est mise en file d'attente (sauf celles qui s'exécutent immédiatement, comme `/status`, `/tasks`, `/usage`).
- On peut enchaîner jusqu'à 6 skills : `/skill-a /skill-b fais XYZ`.

---

## 1. Démarrage, aide et compte

| Commande | Rôle |
|----------|------|
| `/help` | Aide et liste des commandes |
| `/powerup` | Leçons interactives avec démos animées pour découvrir les fonctions |
| `/init` | Crée un `CLAUDE.md` décrivant le projet (avec `CLAUDE_CODE_NEW_INIT=1`, parcours interactif incluant skills, hooks, mémoire) ; propose d'importer une config Codex/Gemini |
| `/import [codex\|gemini\|cursor] [--dry-run] [--yes]` | Importe la configuration d'OpenAI Codex, Gemini CLI ou Cursor (instructions, MCP, commandes, sous-agents, skills) |
| `/doctor [prompt-audit [chemin]]` | **(Skill)** Bilan de l'installation, des réglages, des extensions et des `CLAUDE.md`, avec corrections après confirmation. `prompt-audit` audite vos instructions. Alias `/checkup` |
| `/status` | Réglages, onglet **Status** : version, modèle, compte, connectivité, sources de réglages |
| `/release-notes` | Notes de version dans un sélecteur interactif |
| `/insights` | Rapport HTML analysant vos sessions récentes (projets, habitudes, points de friction, fonctions à essayer) |
| `/team-onboarding` | Génère un guide d'intégration pour un coéquipier à partir de vos 30 derniers jours d'usage (+ lien de partage) |
| `/login` / `/logout` | Se connecter / se déconnecter du compte Anthropic |
| `/upgrade` | Ouvre la page pour passer à un forfait supérieur |
| `/privacy-settings` | Paramètres de confidentialité (Pro et Max) |
| `/feedback [rapport]` | Envoyer un retour produit ; ouvre aussi la file des retours rédigés par Claude |
| `/bug [rapport]` | Signaler un bug (avec consentement sur ce qui est envoyé). Alias `/share` |
| `/mobile` | QR code pour télécharger l'app mobile. Alias `/ios`, `/android` |
| `/desktop` | Continuer la session dans l'app Desktop (macOS, Windows x64). Alias `/app` |
| `/passes` | Offrir une semaine gratuite de Claude Code (si éligible) |
| `/stickers` | Commander des autocollants Claude Code |
| `/radio` | Ouvrir la radio lo-fi « Claude FM » |

## 2. Conversation, contexte et sessions

| Commande | Rôle |
|----------|------|
| `/clear [nom]` | Nouvelle conversation avec un contexte vide (l'ancienne reste reprenable). Alias `/reset`, `/new` |
| `/compact [instructions]` | Résume la conversation pour libérer du contexte, avec consignes facultatives (« garde la sortie des tests ») |
| `/autocompact [auto\|<jetons>]` | Règle le seuil de compaction automatique (ex. `500k`) |
| `/context [all]` | **Visualise l'occupation du contexte** en grille colorée, avec conseils d'optimisation |
| `/btw [question]` | Question annexe sans polluer l'historique |
| `/recap` | Résumé d'une ligne de la session |
| `/rewind` | Revenir en arrière (code et/ou conversation) ou résumer à partir d'un message. Alias `/checkpoint`, `/undo` |
| `/resume [session]` | Reprendre une conversation par ID ou nom, ou ouvrir le sélecteur. Alias `/continue` |
| `/branch [nom]` | Créer une **branche de la conversation** à ce point et basculer dessus (l'original reste reprenable) |
| `/fork [prompt]` | Copier la conversation dans une **nouvelle session en arrière-plan** et continuer ici |
| `/subtask <tâche>` | Lancer un **sous-agent forké** (hérite de toute la conversation) dont le résultat revient ici |
| `/rename [nom]` | Renommer la session (nom affiché sur la barre du prompt) ; sans argument, nom auto-généré |
| `/color [couleur\|default]` | Couleur de la barre du prompt : `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan` |
| `/export [fichier]` | Exporter la conversation en texte |
| `/copy [N]` | Copier la dernière réponse (ou la N-ième) ; sélecteur de blocs de code ; `w` = écrire dans un fichier |
| `/goal [condition\|clear]` | Fixer un **objectif** : Claude continue tour après tour jusqu'à ce que la condition soit remplie |
| `/exit` | Quitter (détache simplement une session d'arrière-plan). Alias `/quit` |

## 3. Modèle, réflexion et style

| Commande | Rôle |
|----------|------|
| `/model [modèle]` | Changer de modèle (et l'enregistrer par défaut) ; `←/→` pour l'effort ; `s` pour cette session seulement |
| `/effort [low\|medium\|high\|xhigh\|max\|auto\|status\|ultracode [on\|off]]` | Niveau d'effort de réflexion ; `ultracode` fait planifier un workflow pour chaque tâche importante |
| `/fast [on\|off]` | **Mode rapide** (réponses Opus plus rapides, facturé en crédits d'usage) |
| `/advisor [modèle\|off]` | Active l'**advisor** : un second modèle plus fort consulté aux moments clés |
| `/output-style [style]` | Changer de style de sortie (Default, Proactive, Concise, Explanatory, Learning ou personnalisé) |

## 4. Permissions et sécurité

| Commande | Rôle |
|----------|------|
| `/permissions` | Gérer les règles **allow / ask / deny**, les dossiers de travail, les refus récents du mode auto, les règles du classifieur (onglet **Auto mode**). Alias `/allowed-tools` |
| `/plan [description]` | Passer en **mode plan** (et démarrer une tâche, ex. `/plan corrige le bug d'auth`) |
| `/sandbox` | Activer/désactiver la **sandbox** Bash |
| `/auto-mode-setup` | Rédiger les entrées `autoMode.environment` (infrastructure de confiance) à partir du projet |
| `/fewer-permission-prompts` | **(Skill)** Analyse vos transcriptions et ajoute une liste blanche de commandes en lecture seule pour réduire les demandes |
| `/add-dir <chemin>` | Ajouter un dossier de travail supplémentaire pour la session |
| `/cd <chemin>` | Déplacer la session dans un autre dossier en gardant la conversation |
| `/security-review` | Analyse de sécurité des changements de la branche (injections, authentification, fuite de données…) |

## 5. Code, revue et vérification

| Commande | Rôle |
|----------|------|
| `/diff` | Voir les changements de l'arbre de travail (panneau en direct en plein écran) |
| `/code-review [low…max\|ultra] [--fix] [--comment] [--max-findings n\|all] [pr#\|branche\|chemin]` | **(Skill)** Relecture des bugs du diff ou d'une PR ; `--fix` applique les corrections, `--comment` publie sur la PR/MR, `ultra` lance une revue multi-agents dans le cloud. Alias `/review` |
| `/ultrareview [PR ou branche]` | Revue approfondie multi-agents dans le cloud (= `/code-review ultra`) |
| `/simplify [cible]` | **(Skill)** 4 agents en parallèle cherchent réutilisation, simplification, efficacité, bon niveau d'abstraction, puis appliquent les corrections |
| `/verify` | **(Skill)** Construit, lance l'app et observe le résultat pour confirmer qu'un changement fonctionne |
| `/run` | **(Skill)** Lance et pilote l'application du projet pour voir un changement fonctionner |
| `/run-skill-generator` | **(Skill)** Écrit un skill de projet qui apprend à `/run` et `/verify` comment construire et lancer votre app |
| `/batch <instruction>` | **(Skill)** Gros changement en parallèle : découpe en 5 à 30 unités, un sous-agent par unité dans son worktree, chacun teste et publie |
| `/debug [description]` | **(Skill)** Active les logs de débogage et analyse les problèmes de la session |
| `/autofix-pr [prompt]` | Lance une session cloud qui surveille la PR de la branche et pousse des corrections (CI, commentaires) |

## 6. Agents, tâches de fond et automatisation

| Commande | Rôle |
|----------|------|
| `/agents` | Rappel pour créer/gérer des sous-agents (demandez à Claude ou éditez `.claude/agents/`) |
| `/tasks` | Voir et gérer le travail en arrière-plan (shells, sous-agents, même terminés). Alias `/bashes` |
| `/background [prompt]` | Détacher la session en **agent d'arrière-plan** et libérer le terminal. Alias `/bg` |
| `/stop` | Arrêter la session d'arrière-plan à laquelle vous êtes attaché |
| `/list-agents` | Lister les sous-agents, coéquipiers et autres sessions auxquels Claude peut écrire. Alias `/peers` |
| `/workflows` | Vue de progression des **workflows** (pause, reprise, sauvegarde) |
| `/workflow-authoring` | **(Skill)** Référence pour écrire des scripts de workflow |
| `/deep-research <question>` | **(Workflow)** Recherche web en éventail, recoupement des sources, rapport cité |
| `/loop [intervalle] [prompt]` | **(Skill)** Répéter un prompt à intervalle (ex. `/loop 5m vérifie le déploiement`) ou à un rythme choisi par Claude. Alias `/proactive` |
| `/schedule [description]` | Créer/gérer des **routines** cloud (planifiées, via API ou événements GitHub). Alias `/routines` |

## 7. Configuration et extensions

| Commande | Rôle |
|----------|------|
| `/config [clé=valeur …]` | Interface des **réglages** (thème, modèle, style, éditeur…) ou réglage direct (`/config theme=dark`). Alias `/settings` |
| `/memory` | Éditer les `CLAUDE.md`, activer/désactiver la mémoire automatique, voir ses entrées |
| `/mcp [reconnect\|enable\|disable [serveur\|all]]` | Gérer les **serveurs MCP** et leur authentification OAuth |
| `/plugin [sous-commande]` | Gérer les **plugins** (Discover, Installed, Marketplaces, Errors) |
| `/reload-plugins [--force]` | Recharger les plugins sans redémarrer |
| `/plugin-authoring` | Référence pour écrire un **mod** |
| `/skills` | Lister les **skills** ; `t` trier par jetons ; `Espace`/`Entrée` changer leur visibilité |
| `/reload-skills` | Rescanner les dossiers de skills |
| `/skill-doctor` | Coût en contexte et fréquence d'usage de chaque skill |
| `/hooks` | Voir la configuration des **hooks** |
| `/update-config [demande]` | **(Skill)** Décrivez un changement de réglage, Claude modifie le bon `settings.json` |
| `/statusline` | Configurer la **barre d'état** (en langage naturel) |
| `/keybindings` | Ouvrir le fichier des **raccourcis clavier** |
| `/theme` | Changer de **thème** (ou en créer un) |
| `/tui [default\|fullscreen]` | Choisir le moteur de rendu (classique ou plein écran) |
| `/focus` | Vue « focus » (plein écran) |
| `/scroll-speed` | Vitesse de la molette (plein écran) |
| `/terminal-setup` | Installer `Shift+Entrée` pour les retours à la ligne (VS Code, Cursor, Alacritty, Zed…), régler Option sur Terminal Apple, presse-papiers iTerm2 |
| `/voice [hold\|tap\|off]` | **Dictée vocale** |
| `/ide` | Gérer l'intégration avec l'IDE |
| `/chrome` | Réglages de **Claude in Chrome** |

## 8. Cloud, appareils et intégrations

| Commande | Rôle |
|----------|------|
| `/remote-control` | Rendre la session pilotable depuis claude.ai ou l'app mobile. Alias `/rc` |
| `/remote-env` | Environnement cloud par défaut pour les sessions lancées depuis le CLI |
| `/teleport` | Ramener une **session cloud** dans ce terminal. Alias `/tp` |
| `/web-setup` | Connecter GitHub aux sessions cloud avec vos identifiants `gh` |
| `/install-github-app` | Installer l'app GitHub Claude et configurer **GitHub Actions** |
| `/install-slack-app` | Installer l'app Slack Claude |
| `/claude-in-chrome [tâche]` | **(Skill)** Faire réaliser une tâche dans votre navigateur (test, formulaire, logs console) |
| `/artifacts` | Lister vos **artifacts** (pages partageables), en joindre un, l'ouvrir, copier son lien |
| `/artifact-capabilities` | **(Skill)** Capacités d'exécution d'un artifact (connecteurs, téléchargements…) |
| `/artifact-diagramming` | **(Skill)** Conseils pour dessiner des diagrammes SVG dans les artifacts |
| `/design [brief]` | **(Skill)** Maquettes UI, parcours d'écrans, landing pages sur un canevas Claude Design (export PNG/PDF) |
| `/design-sync [indice]` / `/design-login` | **(Skill)** Convertir et téléverser votre design system React dans Claude Design |
| `/slides [brief]` | **(Skill)** Créer une présentation Claude Slides |
| `/dataviz [demande]` | **(Skill)** Conseils de conception de graphiques et tableaux de bord accessibles |
| `/claude-api [migrate\|upgrade\|managed-agents-onboard\|prompt-audit\|cost-optimize\|build-eval\|hillclimb\|…]` | **(Skill)** Référence de l'API Claude et des Managed Agents pour votre langage |

## 9. Consommation et dépannage

| Commande | Rôle |
|----------|------|
| `/usage` | Coût de la session, limites du forfait, statistiques d'activité, ventilation de ce qui consomme. Alias `/cost`, `/stats` |
| `/usage-credits` | Configurer ou demander des **crédits d'usage** (ex-`/extra-usage`) |
| `/rate-limit-options` | Options quand une limite bloque : attendre et reprendre automatiquement, crédits, changement de forfait |
| `/heapdump` | (caché) Instantané mémoire pour diagnostiquer une forte consommation |
| `/setup-bedrock` / `/setup-vertex` | (cachés) Assistants de configuration Amazon Bedrock / Google Cloud |

## Commandes retirées

`/pr-comments` (demandez directement à Claude), `/vim` (→ `/config` › Editor mode), `/ultraplan` (→ mode plan), `/review` ancienne version (→ alias de `/code-review`).

## Commandes ajoutées par vos extensions

- **Vos skills** : `/nom-du-skill` (dossiers `.claude/skills/` ou `~/.claude/skills/`, anciens fichiers `.claude/commands/`).
- **Skills de plugins** : `/nom-du-plugin:nom-du-skill`.
- **Skills synchronisés depuis claude.ai** : `/anthropic-skills:nom` (ou `/nom` s'il n'y a pas de conflit).
- **Prompts MCP** : les serveurs MCP peuvent exposer des prompts utilisables comme commandes.
- **Workflows sauvegardés** : un workflow peut être enregistré comme commande réutilisable.
