# Glossaire

> Source : https://code.claude.com/docs/en/glossary · [← Index](../31-index-documentation-officielle.md)

*Définitions de la terminologie de Claude Code : ce que signifient la boucle agentique, la compaction, CLAUDE.md, les hooks, les subagents, MCP et les autres concepts clés.*

Ce glossaire définit la terminologie de Claude Code. Chaque entrée renvoie à la page où le concept est traité en profondeur. Pour les concepts liés au modèle comme les tokens, la température ou le RAG, voir le [glossaire de la plateforme](https://platform.claude.com/docs/en/about-claude/glossary). Pour les termes de Claude Desktop comme desktop extension, MCPB et DXT, voir le [Centre d'aide Claude](https://support.claude.com/).

Les termes sont conservés en anglais (titres des entrées), avec leur équivalent français indiqué entre parenthèses quand il est utile.

## A

### AGENTS.md

Fichier Markdown d'instructions de projet que vous rédigez à l'intention des agents de codage IA. Si votre dépôt en contient un et pas de [CLAUDE.md](#claudemd), Claude le lit comme vos instructions de projet, sans que vous ayez à ajouter un second fichier. Vous pouvez modifier le réglage **Project instructions** dans `/config` pour que Claude lise les deux fichiers ou seulement `CLAUDE.md`. La lecture directe d'`AGENTS.md` nécessite Claude Code v2.1.277+. Dans certaines sessions, Claude [ne peut pas lire `AGENTS.md`](https://code.claude.com/docs/en/memory#when-agents-md-support-is-unavailable) : [importez-le alors depuis un `CLAUDE.md`](https://code.claude.com/docs/en/memory#share-one-file-with-other-coding-tools).

En savoir plus : [AGENTS.md](https://code.claude.com/docs/en/memory#agents-md)

### Agent teams

(Équipes d'agents.) Plusieurs sessions Claude Code indépendantes coordonnées par un chef d'équipe (team lead), avec une liste de tâches partagée et une messagerie de pair à pair. Contrairement aux [subagents](#subagent), qui s'exécutent au sein d'une seule session et ne rendent compte qu'au parent, chaque coéquipier a sa propre fenêtre de contexte et vous pouvez interagir directement avec n'importe lequel d'entre eux. Les équipes d'agents sont **expérimentales et désactivées par défaut** ; voir [Activer les équipes d'agents](https://code.claude.com/docs/en/agent-teams#enable-agent-teams).

En savoir plus : [Exécuter des équipes d'agents](https://code.claude.com/docs/en/agent-teams)

### Agentic coding

(Codage agentique.) Flux de travail dans lequel l'IA peut lire des fichiers, exécuter des commandes et effectuer des modifications de façon autonome pendant que vous observez, réorientez ou vous absentez — par opposition aux assistants conversationnels qui ne répondent que par du texte que vous devez appliquer vous-même. Claude Code est agentique parce qu'il dispose d'[outils](#tool) qui lui permettent d'agir, et pas seulement de conseiller.

En savoir plus : [Fonctionnement de Claude Code](https://code.claude.com/docs/en/how-claude-code-works)

### Agentic harness

(Harnais agentique.) Les outils, la gestion du contexte et l'environnement d'exécution qui transforment un modèle de langage en agent de codage compétent. Claude Code est le harnais ; Claude est le modèle à l'intérieur. Le harnais fournit l'accès aux fichiers, l'exécution shell, le contrôle des permissions, le chargement de la mémoire et la boucle qui enchaîne les actions.

En savoir plus : [Fonctionnement de Claude Code](https://code.claude.com/docs/en/how-claude-code-works)

### Agentic loop

(Boucle agentique.) Le cycle que Claude parcourt pour chaque tâche : recueillir le contexte, agir, vérifier les résultats, et recommencer jusqu'à la fin. Chaque utilisation d'outil renvoie des informations qui orientent l'étape suivante. Vous pouvez interrompre la boucle à tout moment pour réorienter. La plupart des points d'extension, dont les [hooks](#hook), les [skills](#skill) et [MCP](#mcp-model-context-protocol), se branchent sur des phases précises de cette boucle.

En savoir plus : [Fonctionnement de Claude Code](https://code.claude.com/docs/en/how-claude-code-works#the-agentic-loop)

### Artifact

(Artefact.) Page web interactive et vivante que Claude Code publie depuis votre session vers une URL privée sur claude.ai, pour voir un résultat visuellement ou le partager au lieu de lire du texte dans le terminal. La page se met à jour sur place quand la session la republie. Les artefacts créés depuis Claude Code apparaissent dans la même galerie que ceux créés dans les conversations claude.ai. Les options de partage dépendent de votre forfait : voir [Partager un artefact](https://code.claude.com/docs/en/artifacts#share-an-artifact).

En savoir plus : [Partager le résultat d'une session sous forme d'artefacts](https://code.claude.com/docs/en/artifacts)

### Auto memory

(Mémoire automatique.) Notes que Claude écrit pour lui-même à partir de vos corrections et préférences, stockées par dépôt git sous `~/.claude/projects/`. Tous les worktrees d'un même dépôt partagent un seul répertoire de mémoire automatique. Les 200 premières lignes ou 25 Ko de l'index `MEMORY.md` sont chargés au début de chaque session. La mémoire automatique est le pendant, écrit par Claude, de [CLAUDE.md](#claudemd), que vous écrivez.

En savoir plus : [Mémoire automatique](https://code.claude.com/docs/en/memory#auto-memory)

### Auto mode

(Mode auto.) [Mode de permission](#permission-mode) dans lequel un modèle classificateur distinct examine les actions à votre place, de sorte que Claude Code exécute la plupart d'entre elles sans vous demander. Claude Code vous demande quand même avant les actions correspondant à vos règles `ask` explicites. Depuis Claude Code v2.1.283+, le mode auto est le [mode de permission de départ intégré](https://code.claude.com/docs/en/permission-modes#which-mode-a-session-starts-in) pour les sessions interactives du terminal et de VS Code ; sur les versions antérieures, seulement sur les forfaits Pro, Max et Team. Le classificateur bloque l'escalade de portée, l'infrastructure non fiable et l'[injection de prompt](#prompt-injection). Les résultats d'outils sont retirés de ce qu'il voit, si bien qu'un contenu hostile dans un fichier ou une page web ne peut pas le manipuler directement.

En savoir plus : [Supprimer les demandes avec le mode auto](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode)

## B

### Bare mode

(Mode dépouillé.) Avec `--bare`, Claude Code démarre sans charger les hooks, skills, commandes personnalisées, subagents, plugins installés, serveurs MCP, mémoire automatique ni CLAUDE.md — à l'exception des skills d'un répertoire passé avec `--add-dir`. Recommandé pour la CI et les appels scriptés, quand il faut le même résultat sur chaque machine.

En savoir plus : [Démarrer plus vite avec le mode bare](https://code.claude.com/docs/en/headless#start-faster-with-bare-mode)

### Bundled skills

(Skills fournies.) Playbooks à base de prompts inclus avec Claude Code, comme `/batch`, `/code-review`, `/debug` et `/loop`. Contrairement aux commandes intégrées, qui exécutent une logique fixe, les skills fournies donnent à Claude un prompt détaillé et le laissent orchestrer le travail : elles peuvent donc lancer des agents, lire des fichiers et s'adapter à votre base de code.

En savoir plus : [Skills fournies](https://code.claude.com/docs/en/skills#bundled-skills)

## C

### Channel

(Canal.) [Serveur MCP](#mcp-model-context-protocol) qui pousse des événements dans votre session en cours, pour que Claude puisse réagir à ce qui se passe pendant que vous êtes loin du terminal. Les canaux peuvent être bidirectionnels : Claude lit un événement entrant et répond par le même canal. Telegram, Discord et iMessage sont inclus dans l'aperçu de recherche (research preview).

En savoir plus : [Channels](https://code.claude.com/docs/en/channels)

### Checkpoint

(Point de restauration.) Point de restauration créé à chaque prompt envoyé qui démarre un tour. Claude Code prend un instantané des fichiers avant chaque modification afin qu'un checkpoint puisse les rétablir. Appuyez deux fois sur `Esc` ou lancez `/rewind` pour restaurer le code, la conversation, ou les deux, à un point antérieur, ou pour résumer une partie de la conversation à partir d'un message choisi. Les checkpoints sont enregistrés avec la conversation : une session reprise peut donc toujours y revenir avec `/rewind`. Ils sont distincts de git et **ne suivent pas les modifications faites via l'outil Bash**.

En savoir plus : [Checkpointing](https://code.claude.com/docs/en/checkpointing)

### `.claude` directory

(Répertoire `.claude`.) Répertoire dans lequel Claude Code lit la configuration de portée projet : réglages, hooks, skills, subagents, règles et mémoire automatique. Un projet a `.claude/` à sa racine ; vos valeurs par défaut de niveau utilisateur sont dans `~/.claude/`.

En savoir plus : [Le répertoire `.claude`](https://code.claude.com/docs/en/claude-directory)

### CLAUDE.md

Fichier Markdown d'instructions persistantes que vous écrivez pour Claude, chargé au début de chaque session comme message utilisateur après le prompt système. Mettez-y les conventions du projet, les notes d'architecture et les règles « toujours faire X ». Le CLAUDE.md à la racine du projet survit à la [compaction](#compaction) et est relu à neuf depuis le disque ensuite.

Vous pouvez placer CLAUDE.md en portée projet dans `./CLAUDE.md` ou `./.claude/CLAUDE.md`, en portée utilisateur dans `~/.claude/CLAUDE.md`, ou en tant que [politique gérée](#managed-settings) pour votre organisation. Tous les fichiers découverts sont concaténés dans le contexte au lieu de se remplacer, du plus large au plus spécifique. Claude Code peut aussi charger les fichiers [AGENTS.md](#agentsmd) d'un projet à la place de CLAUDE.md.

En savoir plus : [Fichiers CLAUDE.md](https://code.claude.com/docs/en/memory#claude-md-files)

### Cloud session

(Session cloud.) Session Claude Code qui continue de tourner après la fermeture de votre ordinateur portable, car elle s'exécute sur une infrastructure cloud et non sur votre machine : gérée par Anthropic par défaut, ou un [environnement auto-hébergé](https://code.claude.com/docs/en/self-hosted-environments) exploité par votre organisation. Vous en démarrez une depuis claude.ai/code, l'application mobile Claude, l'application Desktop avec **Cloud** sélectionné, `claude --cloud`, ou une [routine](https://code.claude.com/docs/en/routines). Une session dans votre terminal, votre IDE ou l'application Desktop avec **Local** sélectionné est une session locale ; pour atteindre une session locale depuis un autre appareil, utilisez [Remote Control](#remote-control).

En savoir plus : [Utiliser Claude Code dans le cloud](https://code.claude.com/docs/en/claude-code-on-the-web)

### Command

(Commande.) Instruction réutilisable que vous invoquez en tapant `/name` dans le prompt. Les commandes intégrées comme `/clear`, `/model` et `/compact` contrôlent la session. Vous pouvez définir vos propres commandes sous forme de fichiers dans `.claude/commands/`, ou les installer depuis un [plugin](#plugin). Les [skills](#skill) sont la façon recommandée d'empaqueter des commandes en plusieurs étapes.

Deux autres usages du mot sont sans rapport : les sous-commandes du CLI `claude` comme `claude mcp add`, listées dans la [référence CLI](https://code.claude.com/docs/en/cli-reference#cli-commands), et le champ `command` d'une entrée de [serveur MCP](#mcp-server) stdio, qui indique l'exécutable que Claude Code lance pour démarrer le serveur.

En savoir plus : [Commandes](https://code.claude.com/docs/en/commands) · [Skills](https://code.claude.com/docs/en/skills)

### Compaction

Résumé automatique de votre conversation lorsque la [fenêtre de contexte](#context-window) approche de sa limite. Les anciennes sorties d'outils sont effacées d'abord, puis la conversation est résumée. Le CLAUDE.md à la racine du projet et la mémoire automatique survivent à la compaction et sont rechargés depuis le disque ; les instructions données seulement dans la conversation peuvent être perdues. Lancez `/compact` pour la déclencher manuellement, éventuellement avec un axe comme `/compact focus on the API changes`.

En savoir plus : [Ce qui survit à la compaction](https://code.claude.com/docs/en/context-window#what-survives-compaction) · [Quand le contexte se remplit](https://code.claude.com/docs/en/how-claude-code-works#when-context-fills-up)

### Connector

(Connecteur.) [Serveur MCP](#mcp-server) ajouté à votre compte claude.ai plutôt que configuré dans Claude Code. Lorsque vous vous connectez à Claude Code avec ce compte, vos connecteurs apparaissent dans `/mcp` à côté des serveurs ajoutés localement. Les organisations peuvent aussi provisionner des connecteurs et y définir des contrôles par outil.

En savoir plus : [Utiliser les serveurs MCP de claude.ai](https://code.claude.com/docs/en/mcp#use-mcp-servers-from-claude-ai)

### Context window

(Fenêtre de contexte.) La mémoire de travail d'une session, qui contient l'historique de conversation, le contenu des fichiers, les sorties de commandes, CLAUDE.md, la mémoire automatique, les skills chargées et les instructions système. Au fil du travail, le contexte se remplit jusqu'à ce que la [compaction](#compaction) le résume. Lancez `/context` pour voir ce qui occupe l'espace. Pour le concept sous-jacent au niveau du modèle, voir le [glossaire de la plateforme](https://platform.claude.com/docs/en/about-claude/glossary#context-window).

En savoir plus : [Explorer la fenêtre de contexte](https://code.claude.com/docs/en/context-window)

## D

### Dispatch

Routeur de tâches lancé depuis le téléphone, qui démarre une session Claude Code dans l'application Desktop lorsque vous envoyez une tâche de codage depuis l'application mobile Claude. Votre prompt est automatiquement acheminé vers le bon outil. Disponible sur les forfaits **Pro et Max**.

En savoir plus : [Sessions depuis Dispatch](https://code.claude.com/docs/en/desktop#sessions-from-dispatch)

## E

### Effort level

(Niveau d'effort.) Réglage qui contrôle le raisonnement adaptatif, lequel laisse le modèle décider s'il réfléchit, et combien, à chaque étape. Un effort plus élevé signifie plus de tokens de réflexion et un raisonnement plus profond ; un effort plus faible est plus rapide et moins cher. L'effort est pris en charge sur les modèles Fable, sur Opus 4.6 et ultérieurs, sur Sonnet 4.6 et ultérieurs, et sur Haiku 5.5.

En savoir plus : [Régler le niveau d'effort](https://code.claude.com/docs/en/model-config#adjust-effort-level)

### Extended thinking

(Réflexion étendue.) Raisonnement visible, étape par étape, que le modèle effectue avant de répondre. Vous pouvez l'ajuster avec le [niveau d'effort](#effort-level), ou plafonner les tokens de réflexion avec `MAX_THINKING_TOKENS` sur les modèles à budget de réflexion fixe. La réflexion apparaît en italique gris dans le terminal.

En savoir plus : [Utiliser la réflexion étendue](https://code.claude.com/docs/en/model-config#extended-thinking)

## F

### Frontmatter

(En-tête YAML.) Bloc de réglages YAML tout en haut d'un fichier Markdown, entre une ligne `---` ouvrante et une ligne `---` fermante. Les skills, subagents, styles de sortie et règles lisent chacun leur configuration depuis le frontmatter — par exemple la `description` d'une skill ou les `tools` d'un subagent — et traitent tout ce qui suit le `---` fermant comme les instructions. Le `---` ouvrant doit être la première ligne du fichier. Chaque type de fichier accepte son propre ensemble de champs.

En savoir plus : [Frontmatter des skills](https://code.claude.com/docs/en/skills#frontmatter-reference), [Frontmatter des subagents](https://code.claude.com/docs/en/sub-agents#supported-frontmatter-fields), [Frontmatter des styles de sortie](https://code.claude.com/docs/en/output-styles#frontmatter), [Frontmatter des règles](https://code.claude.com/docs/en/memory#rules-frontmatter-reference)

## H

### Hook

Gestionnaire défini par l'utilisateur qui s'exécute automatiquement à un point précis du cycle de vie de Claude Code, par exemple avant l'exécution d'un outil, après la modification d'un fichier ou au démarrage d'une session. Les gestionnaires peuvent être une commande shell, un point de terminaison HTTP, un outil MCP, un prompt LLM ou un subagent. Les hooks sont **déterministes** : ils se déclenchent à des points fixes du cycle de vie, et non à la discrétion du modèle.

Une configuration de hook comporte trois niveaux :

* **Hook event** (événement de hook) : le point du cycle de vie
* **Matcher** (filtre) : filtre les événements qui le déclenchent
* **Hook handler** (gestionnaire) : ce qui s'exécute

En savoir plus : [Bien démarrer avec les hooks](https://code.claude.com/docs/en/hooks-guide) · [Référence des hooks](https://code.claude.com/docs/en/hooks)

## M

### Managed settings

(Réglages gérés.) Réglages imposés à l'échelle de l'organisation par l'IT ou le DevOps, distribués depuis les serveurs d'Anthropic via la console d'administration ou déployés sur les appareils à un chemin de niveau OS situé hors de `~/.claude`. Les réglages utilisateur et projet **ne peuvent pas** remplacer les réglages gérés. La distribution gérée par serveur s'applique sur les [configurations éligibles](https://code.claude.com/docs/en/server-managed-settings#platform-availability) ; voir [Considérations de sécurité](https://code.claude.com/docs/en/server-managed-settings#security-considerations). À utiliser pour les politiques de sécurité, les exigences de conformité ou un outillage standardisé sur un parc.

En savoir plus : [Réglages gérés par serveur](https://code.claude.com/docs/en/server-managed-settings) · [Fichiers de réglages](https://code.claude.com/docs/en/settings#where-settings-live)

### MCP (Model Context Protocol)

Standard ouvert pour connecter des outils d'IA à des sources de données et services externes. Les serveurs MCP donnent à Claude de nouveaux outils pour Slack, Jira, des bases de données, des navigateurs et des centaines d'autres intégrations. Vous connectez des serveurs via `/mcp` ou en les ajoutant à `.mcp.json`. Pour le protocole lui-même, voir le [glossaire de la plateforme](https://platform.claude.com/docs/en/about-claude/glossary#mcp-model-context-protocol).

En savoir plus : [Model Context Protocol](https://code.claude.com/docs/en/mcp)

### MCP server

(Serveur MCP.) Programme qui fournit à Claude des outils, des prompts ou des ressources via [MCP](#mcp-model-context-protocol). Vous ajoutez des serveurs avec `claude mcp add`, dans `.mcp.json`, via un [plugin](#plugin), ou en tant que [connecteur](#connector) claude.ai. Un serveur stdio local s'exécute comme un processus que Claude Code démarre à partir des champs `command` et `args` de sa configuration, qui n'ont rien à voir avec les [commandes](#command) que vous tapez dans le prompt.

En savoir plus : [Model Context Protocol](https://code.claude.com/docs/en/mcp)

### MCP Tool Search

(Recherche d'outils MCP.) Mécanisme d'économie de contexte qui diffère le chargement des schémas d'outils MCP jusqu'à ce qu'ils soient nécessaires. Seuls les noms d'outils et les instructions des serveurs sont chargés au démarrage ; Claude récupère le schéma complet à la demande lorsqu'il décide d'utiliser un outil précis. Cela évite que des serveurs MCP inactifs consomment beaucoup de contexte.

En savoir plus : [Passer à l'échelle avec MCP Tool Search](https://code.claude.com/docs/en/mcp#scale-with-mcp-tool-search)

## N

### Non-interactive mode

(Mode non interactif.) Mode qui exécute un seul prompt puis se termine, sans prompt interactif, invoqué avec `-p` ou `--print`. Utilisé pour la CI, les scripts et les pipes. L'exécution est tout de même enregistrée comme session reprenable, sauf si vous passez `--no-session-persistence`. L'[Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview) en est l'équivalent Python et TypeScript. Anciennement appelé mode headless.

En savoir plus : [Exécuter Claude Code par programme](https://code.claude.com/docs/en/headless)

## O

### Output style

(Style de sortie.) Configuration qui modifie les instructions que Claude Code donne à Claude, pour définir le comportement, le ton ou le format des réponses. Contrairement à [CLAUDE.md](#claudemd), qui ajoute du contexte de projet à côté des instructions par défaut de Claude Code, un style de sortie personnalisé ajoute ses propres instructions et peut omettre les instructions par défaut d'ingénierie logicielle.

En savoir plus : [Styles de sortie](https://code.claude.com/docs/en/output-styles)

## P

### Permission mode

(Mode de permission.) Comportement d'approbation de référence pour la session. Passez de l'un à l'autre avec `Shift+Tab` dans le CLI, ou utilisez le sélecteur de mode dans VS Code, Desktop et claude.ai. Modes disponibles : `default`, `acceptEdits`, `plan`, `auto`, `dontAsk` et `bypassPermissions`.

Le mode `default` est libellé **Manual** dans le CLI, dans les extensions VS Code et JetBrains et dans l'application desktop, et Claude Code accepte `manual` comme alias de cette valeur.

En savoir plus : [Choisir un mode de permission](https://code.claude.com/docs/en/permission-modes)

### Permission rule

(Règle de permission.) Entrée de réglages qui autorise, soumet à confirmation ou refuse un appel d'outil selon le nom de l'outil et un motif d'arguments. Les règles sont évaluées dans l'ordre deny→ask→allow, la première correspondance l'emporte. Les règles de permission sont des contrôles fins superposés au [mode de permission](#permission-mode), plus général.

En savoir plus : [Configurer les permissions](https://code.claude.com/docs/en/permissions)

### Plan mode

(Mode plan.) [Mode de permission](#permission-mode) dans lequel Claude étudie et propose des modifications sans éditer vos fichiers source. Il peut lire, chercher et exécuter des commandes d'exploration, puis présente un plan à approuver avant de toucher à quoi que ce soit. Entrez en mode plan avec `/plan` ou en appuyant sur `Shift+Tab`.

En savoir plus : [Analyser avant de modifier avec le mode plan](https://code.claude.com/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode)

### Plugin

Ensemble de skills, hooks, subagents et serveurs MCP empaquetés en une seule unité installable. Les skills de plugin ont un espace de noms `plugin-name:skill-name`, ce qui permet à plusieurs plugins de coexister. Distribuez des plugins entre équipes via une [marketplace](https://code.claude.com/docs/en/plugins/overview).

En savoir plus : [Plugins](https://code.claude.com/docs/en/plugins/overview)

### Project trust

(Confiance du projet.) Boîte de dialogue d'acceptation d'un répertoire avant que Claude Code ne charge sa configuration. L'acceptation est enregistrée par répertoire de projet, sauf pour votre répertoire personnel, où la confiance n'est conservée que pour la session en cours et où la demande réapparaît à chaque lancement. Tant que vous n'avez pas accordé votre confiance à un répertoire, Claude Code retient une partie du contenu fourni par son dépôt, comme les règles `allow` du projet et les marketplaces de `.claude/settings.json`. [Ce qui s'exécute avant que vous fassiez confiance à un dossier](https://code.claude.com/docs/en/permissions#what-runs-before-you-trust-a-folder) liste chaque type de contenu, y compris ce qu'une session `-p` exécute sans boîte de dialogue.

En savoir plus : [Le répertoire `.claude`](https://code.claude.com/docs/en/claude-directory)

### Prompt injection

(Injection de prompt.) Instructions hostiles intégrées dans un fichier, une page web ou un résultat d'outil, qui tentent de pousser Claude vers des actions que vous n'avez jamais demandées. Les défenses de Claude Code comprennent le système de permissions, la détection d'injection de commandes et la vérification de confiance. Le [mode auto](#auto-mode) ajoute une sonde côté serveur qui analyse les résultats d'outils à la recherche de contenu suspect, et un classificateur qui examine les actions sans les résultats d'outils, afin que le texte injecté ne puisse pas le manipuler directement.

En savoir plus : [Se protéger contre l'injection de prompt](https://code.claude.com/docs/en/security#protect-against-prompt-injection)

## R

### Remote Control

(Contrôle à distance.) Moyen de poursuivre une session Claude Code locale depuis votre téléphone ou votre navigateur via claude.ai. L'exécution du code et les fichiers restent sur votre machine ; seule l'interface est distante. À distinguer d'une [session cloud](https://code.claude.com/docs/en/claude-code-on-the-web), qui s'exécute dans un bac à sable cloud.

En savoir plus : [Remote Control](https://code.claude.com/docs/en/remote-control)

### Rules

(Règles.) Fichiers d'instructions modulaires dans `.claude/rules/` qui se chargent à côté de CLAUDE.md. Une règle peut être limitée à certains chemins via le frontmatter YAML `paths:`, de sorte qu'elle ne se charge que lorsque Claude lit, écrit ou modifie un fichier correspondant, ce qui garde le contexte léger tant qu'elle n'est pas pertinente.

En savoir plus : [Organiser les règles avec `.claude/rules/`](https://code.claude.com/docs/en/memory#organize-rules-with-claude/rules/)

## S

### Sandboxing

(Bac à sable.) Isolation du système de fichiers et du réseau au niveau de l'OS pour l'outil Bash. Les commandes s'exécutent à l'intérieur d'une frontière que vous définissez à l'avance, si bien que Claude peut y travailler librement sans demande d'approbation par commande. Le sandboxing est une couche distincte des [règles de permission](#permission-rule).

En savoir plus : [Sandboxing](https://code.claude.com/docs/en/sandboxing)

### Session

Conversation liée à votre répertoire courant, avec sa propre [fenêtre de contexte](#context-window) indépendante. Les sessions peuvent être reprises avec `claude -c`, dupliquées avec `--fork-session` pour conserver l'historique sous un nouvel ID de session, ou exécutées en parallèle dans plusieurs terminaux. Lancer `/clear` démarre une nouvelle session ; la précédente reste enregistrée et accessible via `/resume`. La transcription de chaque session est stockée sous `~/.claude/projects/`.

En savoir plus : [Travailler avec les sessions](https://code.claude.com/docs/en/how-claude-code-works#work-with-sessions)

### Settings layers

(Couches de réglages.) Hiérarchie depuis laquelle Claude Code lit sa configuration, par ordre de priorité décroissante : [politique gérée](#managed-settings), réglages passés avec le flag `--settings`, réglages locaux dans `.claude/settings.local.json`, réglages de projet dans `.claude/settings.json`, puis réglages utilisateur dans `~/.claude/settings.json`. Les tableaux (arrays) fusionnent entre les couches ; les valeurs scalaires d'une couche supérieure remplacent celles des couches inférieures. Voir [Priorité des réglages](https://code.claude.com/docs/en/settings#settings-precedence).

En savoir plus : [Fichiers de réglages](https://code.claude.com/docs/en/settings#where-settings-live)

### Skill

Fichier `SKILL.md` contenant des instructions, des connaissances ou un workflow que Claude ajoute à sa boîte à outils. Claude charge une skill automatiquement quand elle est pertinente, ou vous l'invoquez directement avec `/skill-name`. Les skills suivent le standard ouvert Agent Skills ; Claude Code l'étend avec le contrôle de l'invocation et l'exécution en subagent.

Les skills sont le successeur recommandé des commandes personnalisées. Un fichier `.claude/commands/deploy.md` et un fichier `.claude/skills/deploy/SKILL.md` créent tous deux `/deploy` et fonctionnent de la même façon ; les fichiers de commandes existants continuent de fonctionner.

En savoir plus : [Étendre Claude avec des skills](https://code.claude.com/docs/en/skills)

### Subagent

(Sous-agent.) Assistant IA spécialisé qui s'exécute dans sa propre fenêtre de contexte, avec un prompt système personnalisé, un accès à des outils précis et des permissions indépendantes. Il travaille sur une tâche déléguée et renvoie un résumé à la conversation principale. Utilisez des subagents pour garder les grosses explorations hors de votre contexte principal ou pour mener des recherches en parallèle. Un subagent reste à l'intérieur de la session qui l'a lancé. Pour faire passer des résultats entre des sessions distinctes que vous lancez vous-même, utilisez la [messagerie inter-sessions](https://code.claude.com/docs/en/cross-session-messaging).

Les subagents intégrés comprennent Explore, Plan et general-purpose.

En savoir plus : [Créer des subagents personnalisés](https://code.claude.com/docs/en/sub-agents)

### Surface

Tout endroit où vous accédez à Claude Code : le CLI, VS Code, JetBrains, Desktop ou claude.ai. Toutes les surfaces partagent le même moteur. Les sessions sur votre machine lisent votre CLAUDE.md, vos réglages et vos skills locaux ; les [sessions cloud](https://code.claude.com/docs/en/cloud-environments#what-carries-over-from-your-setup) partent d'un clone neuf de votre dépôt et ne lisent pas le `~/.claude/` de votre machine. Slack et l'extension Chrome sont des intégrations qui se connectent à une surface, et non des surfaces en elles-mêmes.

En savoir plus : [Plateformes et intégrations](https://code.claude.com/docs/en/platforms)

### System prompt

(Prompt système.) Instructions que Claude Code envoie avant votre conversation à chaque requête, couvrant la façon dont Claude utilise les outils, se comporte de manière sûre et formate ses réponses. Vous pouvez compléter le prompt système avec `--append-system-prompt` ou le remplacer avec `--system-prompt`. Le prompt système est la première couche du [cache de prompt](https://code.claude.com/docs/en/prompt-caching#how-the-cache-is-organized).

Vos fichiers [CLAUDE.md](#claudemd) et les instructions de votre [style de sortie](#output-style) ne font pas partie du prompt système. Claude Code les transmet dans la conversation sous forme de [system reminders](#system-reminder).

En savoir plus : [Flags du prompt système](https://code.claude.com/docs/en/cli-reference#system-prompt-flags)

### System reminder

(Rappel système.) Message que Claude Code, en tant que [harnais](#agentic-harness), ajoute à la conversation pour donner du contexte à Claude. Vous n'envoyez pas vous-même de system reminders. Claude Code les insère au fil de la session, par exemple au démarrage de la session, lorsqu'un hook renvoie du texte ou lorsqu'un fichier change sur le disque. Claude les lit en même temps que vos messages. Tous les éléments suivants parviennent à Claude sous forme de system reminders :

* vos fichiers [CLAUDE.md](#claudemd) ;
* les instructions de votre [style de sortie](#output-style) ;
* le texte qu'un [hook](#hook) renvoie comme `additionalContext` ;
* la liste des [skills](#skill) disponibles ;
* une note signalant qu'un fichier que Claude a lu plus tôt a changé sur le disque ;
* les lignes d'attribution des commits et pull requests.

Dans une requête API journalisée, un system reminder apparaît encadré par des balises `<system-reminder>` à l'intérieur d'un message utilisateur, ou, sur certains modèles, comme un message distinct de rôle `system`.

En savoir plus : [Contexte que Claude Code ajoute hors du prompt système](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts#context-claude-code-adds-outside-the-system-prompt)

## T

### Teleport

(Téléportation.) Commande, `/teleport`, qui rapatrie une session Claude Code cloud dans votre terminal local. Claude récupère la branche, charge l'historique de conversation et reprend à partir du dernier état de la session cloud. Le sens inverse est `--cloud`, qui envoie une tâche locale s'exécuter dans le cloud.

En savoir plus : [Poursuivre une session cloud dans votre terminal](https://code.claude.com/docs/en/claude-code-on-the-web#from-cloud-to-terminal)

### Tool

(Outil.) Action que Claude peut effectuer : lire un fichier, modifier du code, exécuter une commande shell, chercher sur le web, lancer un subagent. Les outils sont ce qui rend Claude Code agentique. Sans eux, Claude ne peut répondre que par du texte. Chaque utilisation d'outil renvoie un résultat qui oriente la décision suivante de Claude dans la [boucle agentique](#agentic-loop).

En savoir plus : [Outils disponibles pour Claude](https://code.claude.com/docs/en/tools-reference)

### Transcript

(Transcription.) L'enregistrement stocké d'une [session](#session). La conversation est ce que vous et Claude échangez ; la transcription est cette conversation conservée dans un fichier, par défaut dans `~/.claude/projects/<project>/<session-id>.jsonl`. Claude Code relit ce fichier lorsque vous reprenez la session : c'est ainsi qu'une conversation se poursuit après la fin de la session. Pour la vue à l'écran de cette même conversation, voir le [visualiseur de transcription](https://code.claude.com/docs/en/interactive-mode#transcript-viewer).

En savoir plus : [Où sont stockées les transcriptions](https://code.claude.com/docs/en/sessions#where-transcripts-are-stored)

### Turn

(Tour.) Une réponse complète de Claude au sein d'une [session](#session). Un tour commence lorsque vous envoyez un message et se termine lorsque Claude a fini de répondre, avec un nombre quelconque d'appels d'[outils](#tool) entre les deux. Les [hooks Stop](#hook) se déclenchent à la fin de chaque tour. Une session se compose de nombreux tours, et la [boucle agentique](#agentic-loop) décrit ce qui se passe à l'intérieur d'un tour.

En savoir plus : [Fonctionnement de Claude Code](https://code.claude.com/docs/en/how-claude-code-works#the-agentic-loop)

## V

### Verification loop

(Boucle de vérification.) Façon dont une session sait que le travail est réellement terminé au lieu d'accepter un résultat qui semble plausible. Vous donnez à Claude une vérification qu'il peut exécuter — une suite de tests, un build, une comparaison de captures d'écran — et Claude itère jusqu'à ce qu'elle passe, au lieu de s'arrêter après une seule tentative. Une boucle de vérification est le prérequis de [`/goal`](https://code.claude.com/docs/en/goal), des exécutions sans surveillance et des [workflows dynamiques](https://code.claude.com/docs/en/workflows) : sans elle, la seule chose qui décide que l'agent a terminé, c'est l'agent lui-même.

En savoir plus : [Donner à Claude un moyen de vérifier son travail](https://code.claude.com/docs/en/best-practices#give-claude-a-way-to-verify-its-work)

## W

### Worktree isolation

(Isolation par worktree.) Mode d'isolation qui exécute Claude dans un worktree git séparé sous `.claude/worktrees/`, activé avec le flag `-w` ou `isolation: worktree` dans la configuration d'un subagent. Les modifications restent sur une branche séparée, dans un répertoire séparé, de sorte que des agents parallèles modifient chacun leur propre copie des fichiers.

En savoir plus : [Exécuter des sessions parallèles avec les worktrees git](https://code.claude.com/docs/en/worktrees)

***

## Termes obsolètes et renommés

Ces termes apparaissent dans d'anciennes docs, des articles de blog et des contenus communautaires. Utilisez le nom actuel pour vos recherches sur ce site.

| Ancien terme | Nom actuel | Remarques |
| - | - | - |
| Headless mode (mode headless) | [Non-interactive mode](#non-interactive-mode) (mode non interactif) | Même flag `-p`, même comportement |
| Web session (session web) ; « Claude Code on the web » comme nom de toute session cloud | [Cloud session](#cloud-session) (session cloud) | « Claude Code on the web » ne désigne plus que la surface navigateur sur claude.ai/code |
| Custom commands (commandes personnalisées) | [Skills](#skill) | Les fichiers `.claude/commands/` fonctionnent toujours |
| Slash commands (commandes slash) | Commands (commandes) | « Slash » a été retiré des textes du produit |
