# Référence des hooks — partie 2/3

> Source : https://code.claude.com/docs/en/hooks · [← Index](../31-index-documentation-officielle.md)

> Cette partie couvre les événements de hook, de `SessionStart` jusqu'à `PostToolBatch` et au-delà selon l'ordre du cycle de vie. Les ancres du type `#common-input-fields` renvoient à la page officielle https://code.claude.com/docs/en/hooks.

## Événements de hook

Chaque événement correspond à un point du cycle de vie de Claude Code où des hooks peuvent s'exécuter. Les sections ci-dessous suivent l'ordre du cycle de vie : de la mise en place de la session, à travers la boucle agentique, jusqu'à la fin de la session. Chaque section décrit quand l'événement se déclenche, quels matchers il prend en charge, l'entrée JSON qu'il reçoit et comment contrôler le comportement via la sortie.

### SessionStart

S'exécute quand Claude Code démarre une nouvelle session ou reprend une session existante. Utile pour charger du contexte de développement (tickets existants, changements récents dans votre base de code) ou pour définir des variables d'environnement. Pour un contexte statique qui ne nécessite pas de script, utilisez plutôt [CLAUDE.md](https://code.claude.com/docs/en/memory).

SessionStart s'exécute à chaque session : gardez ces hooks rapides. Seuls les hooks `type: "command"` et `type: "mcp_tool"` sont pris en charge. Voir [Champs des hooks d'outil MCP](https://code.claude.com/docs/en/hooks#mcp-tool-hook-fields) pour savoir quand les hooks `mcp_tool` s'exécutent.

La valeur du matcher correspond à la façon dont la session a été lancée :

| Matcher | Quand il se déclenche |
| :- | :- |
| `startup` | Nouvelle session |
| `resume` | `--resume`, `--continue` ou `/resume` |
| `clear` | `/clear` |
| `compact` | Compaction automatique ou manuelle |
| `fork` | Une nouvelle session dérivée (fork) d'une session existante : `--fork-session` avec `--resume` ou `--continue`, la copie d'arrière-plan `/fork`, `/branch`, ou une conversation que vous [passez en arrière-plan](https://code.claude.com/docs/en/agent-view#from-inside-a-session) |

Avant la v2.1.214, les sessions forkées signalaient la source `"resume"`.

Quand vous démarrez une session interactive, reprenez une conversation au lancement avec `--continue` ou `--resume`, ou exécutez `/clear`, les hooks SessionStart s'exécutent en arrière-plan. Vous pouvez taper immédiatement, et une conversation reprise s'affiche sans attendre les hooks. La première réponse de Claude attend toutefois la fin des hooks, afin que leur contexte parvienne à Claude.

Quand vous changez de conversation avec `/resume` à l'intérieur d'une session, le changement attend au contraire la fin des hooks. Si vous lancez `/clear` ou basculez vers une autre conversation alors que des hooks d'arrière-plan tournent encore, rien de ce qu'ils renvoient ne s'applique à la session.

La même attente s'applique au lancement, y compris pour une session reprise : un prompt envoyé pendant que les hooks SessionStart tournent encore n'atteint Claude qu'une fois ceux-ci terminés.

Pendant l'une ou l'autre de ces attentes, appuyez sur `Esc` pour récupérer le prompt dans la zone de saisie sans l'envoyer. Les hooks continuent de s'exécuter.

#### Entrée de SessionStart

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks SessionStart reçoivent `source` et, éventuellement, `model`, `agent_type` et `session_title` :

| Champ | Description |
| :- | :- |
| `source` | Comment la session a démarré : `"startup"` pour une nouvelle session, `"resume"` pour une session reprise, `"clear"` après `/clear`, `"compact"` après une compaction, ou `"fork"` pour une nouvelle session forkée d'une session existante |
| `model` | L'identifiant du modèle actif. Il peut être absent, par exemple après `/clear` ou quand une session est restaurée via la récupération de conversation : vérifiez la présence du champ avant de le lire |
| `agent_type` | Le nom de l'agent, présent quand vous lancez Claude Code avec `claude --agent <name>` |
| `session_title` | Le titre personnalisé de la session, présent s'il a été défini, par exemple avec `--name`, `/rename`, la sortie `sessionTitle` d'un hook ou `renameSession()` de l'Agent SDK. Un hook qui émet `sessionTitle` peut d'abord vérifier ce champ pour éviter d'écraser un titre personnalisé existant |

Une session que vous n'avez pas nommée peut quand même avoir un [titre généré](https://code.claude.com/docs/en/sessions#name-your-sessions). Ce titre n'est pas un titre personnalisé et n'apparaît pas dans `session_title`.

Quand `source` vaut `"resume"` ou `"fork"` et que la transcription contient au moins une réponse de Claude, les hooks SessionStart reçoivent aussi les quatre champs ci-dessous. Votre hook peut s'en servir pour indiquer ce que coûte la reprise d'une conversation ancienne avant la première requête, par exemple dans un [`systemMessage`](https://code.claude.com/docs/en/hooks#json-output). Ces champs exigent Claude Code v2.1.251+.

| Champ | Description |
| :- | :- |
| `seconds_since_last_response` | Secondes réelles (horloge murale) écoulées depuis la dernière réponse de la transcription reprise |
| `context_tokens` | Tokens que la première requête de la session reprise renvoie comme prompt |
| `prompt_cache_likely_expired` | `true` quand la dernière réponse est plus ancienne que la [durée de vie du cache de prompt](https://code.claude.com/docs/en/prompt-caching#cache-lifetime) de la session, ou qu'une compaction ultérieure a remplacé la conversation mise en cache |
| `estimated_cache_write_usd` | Coût estimé, en dollars US, de l'écriture de `context_tokens` dans le cache de prompt avec le modèle de la session, hors réponse |

Cet exemple montre l'entrée pour une session reprise 90 minutes après sa dernière réponse :

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "cwd": "/Users/...",
  "hook_event_name": "SessionStart",
  "source": "resume",
  "model": "claude-opus-5",
  "seconds_since_last_response": 5400,
  "context_tokens": 182340,
  "prompt_cache_likely_expired": true,
  "estimated_cache_write_usd": 1.1396
}
```

#### Contrôle de décision de SessionStart

Un hook SessionStart peut ajouter du contexte pour Claude, fournir le premier message utilisateur, définir le titre de la session, surveiller des fichiers et recharger les skills. Renvoyez le champ correspondant à chacune de ces actions, en plus des [champs de sortie JSON](https://code.claude.com/docs/en/hooks#json-output) disponibles pour tous les hooks :

| Champ | Description |
| :- | :- |
| `additionalContext` | Chaîne ajoutée au contexte de Claude au début de la conversation, avant le premier prompt. Voir [Ajouter du contexte pour Claude](https://code.claude.com/docs/en/hooks#add-context-for-claude) pour la façon dont le texte est transmis et ce qu'il faut y mettre |
| `initialUserMessage` | Chaîne utilisée comme premier message utilisateur de la session, en [mode non interactif](https://code.claude.com/docs/en/headless) avec le flag `-p`. Elle devient le premier tour même si vous ne passez aucun prompt. Un prompt que vous passez suit comme tour suivant |
| `sessionTitle` | Définit le titre de la session, avec le même effet que `/rename`. S'applique quand `source` vaut `"startup"`, `"resume"` ou `"fork"` |
| `watchPaths` | Tableau de chemins absolus à surveiller pour les événements [FileChanged](https://code.claude.com/docs/en/hooks#filechanged) pendant cette session |
| `reloadSkills` | Booléen. Quand il vaut `true`, Claude Code ré-analyse les répertoires de [skills](https://code.claude.com/docs/en/skills) et de commandes une fois les hooks SessionStart terminés. Voir [Recharger les skills qu'un hook installe](#recharger-les-skills-quun-hook-installe) |

Cette sortie ajoute du contexte et nomme la session :

```json theme={null}
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "Current branch: feat/auth-refactor\nUncommitted changes: src/auth.ts, src/login.tsx\nActive issue: #4211 Migrate to OAuth2",
    "sessionTitle": "auth-refactor"
  }
}
```

Un hook qui ne fait qu'ajouter du contexte peut simplement l'imprimer sans construire de JSON, car Claude Code ajoute la [sortie stdout en texte brut](https://code.claude.com/docs/en/hooks#exit-code-0) d'un hook SessionStart au contexte de Claude.

Si le hook SessionStart de votre plugin fournit `initialUserMessage` ou `sessionTitle`, installez le plugin avant le démarrage de la session. Claude Code ignore ces deux champs lorsqu'ils proviennent d'un plugin dont l'installation se termine après l'exécution des hooks SessionStart.

#### Recharger les skills qu'un hook installe

Pour rendre disponibles dans la même session les skills qu'un hook SessionStart installe, renvoyez `reloadSkills`. La découverte des skills s'exécute normalement avant la fin des hooks SessionStart ; sans ce champ, les fichiers qu'un hook écrit dans `~/.claude/skills/` ou `.claude/skills/` peuvent manquer lors de l'exécution du premier prompt.

Cet exemple synchronise un dépôt de skills partagé et demande la ré-analyse :

```bash theme={null}
#!/bin/bash

git -C ~/.claude/skills/team-skills pull --quiet 2>/dev/null || \
  git clone --quiet https://git.example.com/your-org/team-skills.git ~/.claude/skills/team-skills

echo '{"hookSpecificOutput": {"hookEventName": "SessionStart", "reloadSkills": true}}'
```

L'URL du dépôt est un exemple fictif. Remplacez-la par celle de votre propre dépôt de skills.

#### Persister des variables d'environnement

Les hooks SessionStart ont accès à la variable d'environnement `CLAUDE_ENV_FILE`, qui fournit un chemin de fichier où persister des variables d'environnement pour les commandes shell que Claude exécutera plus tard dans la session.

Pour définir des variables individuelles, écrivez des instructions `export` dans `CLAUDE_ENV_FILE`. Utilisez l'ajout (`>>`) pour préserver les variables définies par d'autres hooks :

```bash theme={null}
#!/bin/bash

if [ -n "$CLAUDE_ENV_FILE" ]; then
  echo 'export NODE_ENV=production' >> "$CLAUDE_ENV_FILE"
  echo 'export DEBUG_LOG=true' >> "$CLAUDE_ENV_FILE"
  echo 'export PATH="$PATH:./node_modules/.bin"' >> "$CLAUDE_ENV_FILE"
fi

exit 0
```

Pour capturer toutes les modifications d'environnement faites par des commandes de configuration, comparez les variables exportées avant et après :

```bash theme={null}
#!/bin/bash

ENV_BEFORE=$(export -p | sort)

# Exécutez vos commandes de configuration qui modifient l'environnement
source ~/.nvm/nvm.sh
nvm use 20

if [ -n "$CLAUDE_ENV_FILE" ]; then
  ENV_AFTER=$(export -p | sort)
  comm -13 <(echo "$ENV_BEFORE") <(echo "$ENV_AFTER") >> "$CLAUDE_ENV_FILE"
fi

exit 0
```

Chaque commande Bash exécute le contenu du fichier comme du code shell avant la commande elle-même ; une ligne de ce fichier peut donc utiliser tout ce que Bash évalue, comme la référence `$PATH` dans `export PATH="$PATH:./node_modules/.bin"`.

##### Variables persistées dans les commandes PowerShell

Les commandes [PowerShell](https://code.claude.com/docs/en/tools-reference#powershell-tool) reçoivent elles aussi les variables de `CLAUDE_ENV_FILE` (v2.1.296+), mais PowerShell n'exécute jamais le fichier. Claude Code en extrait les affectations et les copie dans l'environnement de la commande PowerShell. Il ne le fait que si chaque ligne relève de l'un des cas suivants, en considérant tout ce qu'ont écrit l'ensemble des hooks de la session ainsi que tout script vers lequel vous avez [fait pointer `CLAUDE_ENV_FILE`](https://code.claude.com/docs/en/env-vars) avant le lancement :

* Une ligne vide ou un commentaire `#`
* Une affectation unique en début de ligne, écrite `export NAME=value`, `declare -x NAME=value` ou `NAME=value`, avec une valeur que Bash utiliserait exactement telle qu'écrite, composée d'un mélange quelconque de : texte non entre guillemets n'utilisant que des lettres, des chiffres et les caractères `_ @ % + = : , . / -` ; texte entre apostrophes simples ; texte entre guillemets doubles où tout `$`, accent grave ou `"` intérieur est échappé par une barre oblique inverse

Si une ligne est d'un autre type — par exemple `export PATH="$PATH:./node_modules/.bin"` avec son `$PATH` non échappé, une commande `source`, ou les chaînes `$'...'` qu'imprime `direnv export bash` — les commandes PowerShell ne reçoivent aucune des variables, et `claude --debug` journalise `Session environment is not all plain assignments`. Les commandes Bash, elles, les reçoivent toutes. Sous Windows, les commandes PowerShell ne reçoivent pas non plus une variable dont la valeur contient `/` ou `\`, car Git Bash et Windows écrivent les chemins différemment. Une commande PowerShell [en sandbox](https://code.claude.com/docs/en/sandboxing) ne reçoit aucune des variables.

> **Note** : `CLAUDE_ENV_FILE` est disponible pour les hooks SessionStart, [Setup](#setup), [CwdChanged](https://code.claude.com/docs/en/hooks#cwdchanged) et [FileChanged](https://code.claude.com/docs/en/hooks#filechanged). Les autres événements de hook n'ont pas accès à cette variable, pas plus qu'un hook qui s'exécute dans PowerShell, que ce soit via [`"shell": "powershell"`](https://code.claude.com/docs/en/hooks#command-hook-fields) ou par défaut sous Windows sans Git Bash.

### Setup

Se déclenche uniquement quand vous lancez Claude Code avec `--init-only`, ou avec `--init` ou `--maintenance` en [mode non interactif](https://code.claude.com/docs/en/headless) avec le flag `-p`. Il ne se déclenche pas au démarrage normal. Utilisez-le pour une installation ponctuelle de dépendances ou un nettoyage planifié que vous déclenchez explicitement depuis la CI ou des scripts, séparément du démarrage normal de session. Pour une initialisation par session, utilisez plutôt [SessionStart](#sessionstart).

La valeur du matcher correspond au flag CLI qui a déclenché le hook :

| Matcher | Quand il se déclenche |
| :- | :- |
| `init` | `claude --init-only` ou `claude -p --init` |
| `maintenance` | `claude -p --maintenance` |

Quand vous exécutez `claude --init-only`, Claude Code exécute les hooks Setup et les hooks `SessionStart` ayant le matcher `startup`, puis se termine sans démarrer de conversation.

Quand vous démarrez ou poursuivez une conversation avec `-p`, vous devez aussi fournir un prompt, en argument ou via stdin. Vous pouvez vous en passer quand un hook `SessionStart` fournit [`initialUserMessage`](#contrôle-de-décision-de-sessionstart) ou quand vous reprenez une session avec un [appel d'outil différé](#différer-un-appel-doutil-pour-plus-tard).

En cas de succès, `--init-only` n'affiche rien dans le terminal. Pour confirmer que les hooks se sont exécutés, lancez `claude --debug-file <path> --init-only` en remplaçant `<path>` par l'emplacement d'un fichier journal, puis cherchez dans le journal les entrées des hooks Setup et SessionStart.

Comme Setup ne se déclenche pas à chaque lancement, un plugin qui a besoin d'une dépendance installée ne peut pas compter uniquement sur Setup. Le schéma pratique consiste à vérifier la dépendance à la première utilisation et à l'installer si elle manque, par exemple un hook ou une skill qui teste la présence de `${CLAUDE_PLUGIN_DATA}/node_modules` et exécute `npm install` en son absence. Voir le [répertoire de données persistant](https://code.claude.com/docs/en/plugins/components#path-variables-and-persistent-data) pour savoir où stocker les dépendances installées. Si vous distribuez votre plugin via une marketplace, ce schéma n'est peut-être pas nécessaire : Claude Code [installe automatiquement les dépendances de paquets Node.js éligibles](https://code.claude.com/docs/en/plugins/loading#node-js-package-dependencies) lorsqu'il met le plugin en cache.

#### Entrée de Setup

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks Setup reçoivent un champ `trigger` valant `"init"` ou `"maintenance"` :

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "cwd": "/Users/...",
  "hook_event_name": "Setup",
  "trigger": "init"
}
```

#### Contrôle de décision de Setup

Les hooks Setup ne peuvent pas bloquer ; l'exécution continue quel que soit le code de sortie. Quel que soit ce code, Claude Code ignore les [champs de sortie JSON](https://code.claude.com/docs/en/hooks#json-output) d'un hook Setup, comme `systemMessage`, `continue` et `hookSpecificOutput.additionalContext`. Avec `-p`, la sortie stdout, stderr et le code de sortie d'un hook Setup n'apparaissent dans la sortie de l'exécution que sous forme d'[événements `hook_response`](https://code.claude.com/docs/en/headless#read-session-metadata) quand vous lancez avec `--output-format stream-json --verbose`.

Les hooks Setup ont accès à `CLAUDE_ENV_FILE`. Les variables écrites dans ce fichier persistent dans les commandes Bash suivantes de la session, comme pour les [hooks SessionStart](#persister-des-variables-denvironnement). Seuls les hooks `type: "command"` s'exécutent sur `Setup`. Un hook `type: "mcp_tool"` sur `Setup` est toujours ignoré, comme décrit dans [Champs des hooks d'outil MCP](https://code.claude.com/docs/en/hooks#mcp-tool-hook-fields).

### InstructionsLoaded

Se déclenche quand un fichier `CLAUDE.md` ou `.claude/rules/*.md` est chargé dans le contexte. Cet événement se déclenche au démarrage de la session pour les fichiers chargés d'emblée, puis plus tard lorsque des fichiers sont chargés à la demande (paresseusement), par exemple quand Claude accède à un sous-répertoire contenant un `CLAUDE.md` imbriqué ou quand des règles conditionnelles avec un frontmatter `paths:` correspondent. Le hook ne prend en charge ni le blocage ni le contrôle de décision. Il s'exécute de façon asynchrone, à des fins d'observabilité.

Cet événement ne se déclenche pas quand Claude [lit `AGENTS.md` directement](https://code.claude.com/docs/en/memory#agents-md) via le réglage **Project instructions**. Il se déclenche en revanche quand un `CLAUDE.md` importe votre `AGENTS.md`, avec `load_reason` à `include` comme pour tout autre fichier importé, et quand `CLAUDE.md` est un lien symbolique vers lui, comme un chargement normal de `CLAUDE.md`.

Le matcher s'applique à `load_reason`. Par exemple, utilisez `"matcher": "session_start"` pour ne déclencher que sur les fichiers chargés au démarrage de la session, ou `"matcher": "path_glob_match|nested_traversal"` pour ne déclencher que sur les chargements paresseux.

#### Entrée d'InstructionsLoaded

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks InstructionsLoaded reçoivent ces champs :

| Champ | Description |
| :- | :- |
| `file_path` | Chemin absolu du fichier d'instructions chargé |
| `memory_type` | Portée du fichier : `"User"`, `"Project"`, `"Local"` ou `"Managed"` |
| `load_reason` | Raison du chargement : `"session_start"`, `"nested_traversal"`, `"path_glob_match"`, `"include"` ou `"compact"`. La valeur `"compact"` apparaît quand les fichiers d'instructions sont rechargés après une compaction |
| `globs` | Motifs glob de chemin issus du frontmatter `paths:` du fichier, s'il y en a. Présent uniquement pour les chargements `path_glob_match` |
| `trigger_file_path` | Chemin du fichier dont l'accès a déclenché ce chargement, pour les chargements paresseux |
| `parent_file_path` | Chemin du fichier d'instructions parent qui a inclus celui-ci, pour les chargements `include` |

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../transcript.jsonl",
  "cwd": "/Users/my-project",
  "hook_event_name": "InstructionsLoaded",
  "file_path": "/Users/my-project/CLAUDE.md",
  "memory_type": "Project",
  "load_reason": "session_start"
}
```

#### Contrôle de décision d'InstructionsLoaded

Les hooks InstructionsLoaded n'ont aucun contrôle de décision. Ils ne peuvent ni bloquer ni modifier le chargement des instructions. Claude Code ignore leurs [champs de sortie JSON](https://code.claude.com/docs/en/hooks#json-output), comme `systemMessage` et `continue`. Utilisez cet événement pour la journalisation d'audit, le suivi de conformité ou l'observabilité.

### UserPromptSubmit

S'exécute quand un prompt est soumis, avant que Claude ne le traite. Cela permet d'ajouter du contexte en fonction du prompt ou de la conversation, de valider des prompts, ou de bloquer certains types de prompts.

Les hooks `UserPromptSubmit` ne se déclenchent pas seulement sur les prompts que vous tapez. Claude Code les exécute aussi lors de :

* Le déclenchement d'une [tâche planifiée](https://code.claude.com/docs/en/scheduled-tasks), y compris une itération de `/loop`
* Le compte rendu d'un [sous-agent d'arrière-plan](https://code.claude.com/docs/en/sub-agents#run-subagents-in-foreground-or-background) à la session qui l'a lancé
* Un [message qu'une autre session envoie](https://code.claude.com/docs/en/cross-session-messaging) à votre conversation principale

Les hooks `UserPromptSubmit` ont un délai d'expiration par défaut de 30 secondes pour les types `command`, `http` et `mcp_tool`, plus court que le défaut de 600 secondes de ces types sur la plupart des autres événements. Comme ce hook s'exécute avant chaque prompt et bloque le traitement par le modèle jusqu'à sa fin, un hook bloqué fige la session. Si votre hook a besoin de plus de temps, définissez le champ `timeout` dans l'entrée du hook.

Hormis un hook command lancé avec [`async: true`](https://code.claude.com/docs/en/hooks#run-hooks-in-the-background), un hook command, HTTP ou outil MCP sur `UserPromptSubmit` qui atteint son délai d'expiration est annulé et sa sortie, y compris tout `additionalContext`, est ignorée. Le prompt parvient quand même à Claude, sans ce contexte. Pour bloquer le prompt à la place, définissez [`onFailure: "block"`](https://code.claude.com/docs/en/hooks#block-the-action-when-a-hook-fails) sur un hook command ou HTTP. La transcription affiche un avis nommant le hook, le délai qui a expiré, et indiquant que la sortie a été ignorée.

Un [hook callback de l'Agent SDK](https://code.claude.com/docs/en/agent-sdk/hooks) sur `UserPromptSubmit` qui atteint son délai d'expiration bloque le prompt avec un message nommant le hook et le délai, car un callback à cet endroit peut jouer le rôle de garde-fou de politique qui ne doit pas « échouer en mode ouvert ». La session continue. Avant la v2.1.208, l'expiration d'un callback sur cet événement terminait le tour avec une erreur d'exécution.

#### Entrée d'UserPromptSubmit

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks UserPromptSubmit reçoivent le champ `prompt` contenant le texte soumis. Le contenu collé qui a été réduit à un espace réservé `[Pasted text #N]` arrive développé sur place. Dans les sessions où Claude Code [marque le texte collé pour Claude](https://code.claude.com/docs/en/terminal-config#how-claude-treats-pasted-text), ce contenu développé se trouve entre une ligne `<pasted_content id="…">` et une ligne `</pasted_content id="…">` : tenez compte de ces lignes si votre hook analyse le prompt.

Les hooks UserPromptSubmit reçoivent aussi `session_title` quand la session a un titre personnalisé, avec la même signification que le [champ `session_title` de SessionStart](#entrée-de-sessionstart).

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "cwd": "/Users/...",
  "permission_mode": "default",
  "hook_event_name": "UserPromptSubmit",
  "prompt": "Write a function to calculate the factorial of a number"
}
```

#### Contrôle de décision d'UserPromptSubmit

Les hooks `UserPromptSubmit` peuvent contrôler si un prompt soumis est traité et ajouter du contexte. Tous les [champs de sortie JSON](https://code.claude.com/docs/en/hooks#json-output) sont disponibles.

Il y a deux façons d'ajouter du contexte à la conversation avec le code de sortie 0 :

* **Stdout en texte brut** : Claude Code ajoute au contexte de Claude la sortie stdout qu'il [traite comme du texte brut](https://code.claude.com/docs/en/hooks#exit-code-0)
* **JSON avec `additionalContext`** : utilisez le format JSON ci-dessous pour plus de contrôle. Le champ `additionalContext` est ajouté comme contexte

Aucun de ces deux canaux ne produit d'entrée visible dans la transcription. Le stdout brut et la valeur `additionalContext` sont chacun injectés sous forme de rappel système (system reminder) commençant par le nom du hook ; Claude lit les deux. Pour confirmer la transmission, consultez le [journal de débogage](https://code.claude.com/docs/en/hooks#debug-hooks).

Pour bloquer un prompt, renvoyez un objet JSON avec `decision` à `"block"` :

| Champ | Description |
| :- | :- |
| `decision` | `"block"` arrête le prompt avant qu'il n'atteigne Claude. Omettez-le pour laisser le prompt continuer |
| `reason` | Affiché à l'utilisateur quand `decision` vaut `"block"`. Non ajouté au contexte |
| `additionalContext` | Chaîne ajoutée au contexte de Claude en même temps que le prompt soumis. Voir [Ajouter du contexte pour Claude](https://code.claude.com/docs/en/hooks#add-context-for-claude) |
| `sessionTitle` | Définit le titre de la session. Sert à nommer automatiquement les sessions selon le contenu du prompt |
| `suppressOriginalPrompt` | Si `true` quand le hook bloque le prompt, omet le texte du prompt dans le message de blocage. Voir [Ce que laisse derrière lui un prompt bloqué](#ce-que-laisse-derrière-lui-un-prompt-bloqué) |

Un hook qui bloque en sortant avec le code 2 est traité comme `reason` : le message de blocage montre le texte stderr à l'utilisateur, et il n'est pas ajouté au contexte.

```json theme={null}
{
  "decision": "block",
  "reason": "Explanation for decision",
  "hookSpecificOutput": {
    "hookEventName": "UserPromptSubmit",
    "additionalContext": "My additional context here",
    "sessionTitle": "My session title",
    "suppressOriginalPrompt": true
  }
}
```

#### Ce que laisse derrière lui un prompt bloqué

Un prompt bloqué n'atteint jamais Claude, mais son texte n'est pas supprimé partout. Par défaut, le message de blocage montré à l'utilisateur se termine par `Original prompt:` suivi du texte soumis, et Claude Code écrit ce message dans le fichier de transcription de la session sur le disque. Pour omettre le texte du message, imprimez un JSON avec `"suppressOriginalPrompt": true` dans `hookSpecificOutput`. Cela fonctionne que le hook bloque avec `decision: "block"` ou en sortant avec le code 2.

`suppressOriginalPrompt` ne modifie que le message de blocage. Le texte soumis peut toujours apparaître dans des fichiers locaux comme la transcription de session et votre historique de prompts : un hook bloquant n'est donc pas un moyen de garder un secret hors du disque. Pour limiter ou supprimer ces fichiers, voir [Stockage en clair](https://code.claude.com/docs/en/claude-directory#plaintext-storage) et [Effacer les données locales](https://code.claude.com/docs/en/claude-directory#clear-local-data).

### UserPromptExpansion

S'exécute quand une commande tapée par l'utilisateur se développe en prompt avant d'atteindre Claude. Utilisez-le pour bloquer l'invocation directe de certaines commandes, injecter du contexte pour une skill particulière ou journaliser les commandes invoquées par les utilisateurs. Par exemple, un hook correspondant à `deploy` peut bloquer `/deploy` sauf si un fichier d'approbation est présent, ou un hook correspondant à une skill de revue peut ajouter la checklist de revue de l'équipe sous forme d'`additionalContext`.

Cet événement couvre le chemin que `PreToolUse` ne couvre pas : un hook `PreToolUse` correspondant à l'outil `Skill` ne se déclenche que lorsque Claude appelle l'outil, alors que taper directement `/skillname` contourne `PreToolUse`. `UserPromptExpansion` se déclenche sur ce chemin direct.

Le matcher s'applique à `command_name`. Laissez le matcher vide pour déclencher sur chaque commande de type prompt.

#### Entrée d'UserPromptExpansion

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks UserPromptExpansion reçoivent `expansion_type`, `command_name`, `command_args`, `command_source` et la chaîne `prompt` d'origine. Le champ `expansion_type` vaut `slash_command` pour les skills et commandes personnalisées, ou `mcp_prompt` pour les prompts de serveurs MCP.

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../00893aaf.jsonl",
  "cwd": "/Users/...",
  "permission_mode": "default",
  "hook_event_name": "UserPromptExpansion",
  "expansion_type": "slash_command",
  "command_name": "example-skill",
  "command_args": "arg1 arg2",
  "command_source": "plugin",
  "prompt": "/example-skill arg1 arg2"
}
```

#### Contrôle de décision d'UserPromptExpansion

Les hooks `UserPromptExpansion` peuvent bloquer l'expansion ou ajouter du contexte. Tous les [champs de sortie JSON](https://code.claude.com/docs/en/hooks#json-output) sont disponibles.

| Champ | Description |
| :- | :- |
| `decision` | `"block"` empêche la commande de se développer. Omettez-le pour la laisser continuer |
| `reason` | Affiché à l'utilisateur quand `decision` vaut `"block"` |
| `additionalContext` | Chaîne ajoutée au contexte de Claude en même temps que le prompt développé. Voir [Ajouter du contexte pour Claude](https://code.claude.com/docs/en/hooks#add-context-for-claude) |

Un hook qui bloque en sortant avec le code 2 est traité comme `reason` : le message de blocage montre le texte stderr à l'utilisateur.

```json theme={null}
{
  "decision": "block",
  "reason": "This slash command is not available",
  "hookSpecificOutput": {
    "hookEventName": "UserPromptExpansion",
    "additionalContext": "Additional context for this expansion"
  }
}
```

### MessageDisplay

S'exécute pendant qu'un message de l'assistant s'affiche en streaming à l'écran. Claude Code affiche le message par incréments : chaque fois qu'un lot de lignes nouvellement complètes est prêt à être rendu, le hook s'exécute une fois avec ces lignes, et Claude Code affiche à leur place le texte de remplacement fourni par le hook. Un long message produit plusieurs appels ; un message court peut n'en produire qu'un.

Utilisez MessageDisplay pour :

* supprimer le markdown pour un affichage minimal
* transformer le texte qu'une application Agent SDK montre à ses utilisateurs
* masquer des clés d'API ou des noms d'hôtes internes dans les réponses de Claude

Claude Code retient chaque lot jusqu'au retour de votre hook : gardez donc le hook rapide. Si le hook échoue ou expire, Claude Code affiche le texte d'origine. Le délai d'expiration par défaut pour cet événement est de 10 secondes ; si votre hook a besoin de plus de temps, définissez le champ `timeout` dans l'entrée du hook.

MessageDisplay ne concerne que l'affichage : le texte de remplacement ne change que ce qui est rendu à l'écran. La transcription et ce que voit Claude conservent le texte d'origine ; Claude ne voit donc jamais le remplacement, et le mode verbeux affiche l'original. Le hook ne reçoit que le texte des messages de l'assistant : les résultats d'outils et le texte que vous tapez s'affichent sans changement.

MessageDisplay ne prend pas en charge les matchers et se déclenche pour chaque message de l'assistant qui diffuse du texte ; les messages sans texte, comme les réponses ne contenant que des appels d'outils, ne le déclenchent pas.

Dans les exécutions non interactives, y compris les requêtes Agent SDK et `claude -p`, MessageDisplay s'exécute une fois par message de l'assistant au lieu d'une fois par lot de lignes. L'appel unique arrive après la fin du message et transporte le texte complet du message : `index` vaut `0`, `final` vaut `true`, et `delta` contient le message entier. Un hook qui collecte le texte `delta` de chaque message reçoit donc le même texte total dans les deux modes.

#### Entrée de MessageDisplay

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks MessageDisplay reçoivent des identifiants pour le tour et le message, la position de cet appel dans le message, et le nouveau texte dans `delta`. Les limites des lots dépendent de la façon dont le texte est diffusé : utilisez `index` et `final` pour suivre la progression dans un message plutôt que de supposer un regroupement particulier des lignes.

| Champ | Description |
| :- | :- |
| `turn_id` | UUID du tour en cours |
| `message_id` | UUID du message de l'assistant affiché. Stable pour tous les lots d'un même message. Ce n'est pas l'identifiant d'API `msg_…`, il ne peut donc pas être corrélé avec les identifiants de messages de la transcription |
| `index` | Index (à partir de zéro) de ce lot dans le message |
| `final` | `true` sur le dernier lot du message. Chaque message a exactement un lot final |
| `delta` | Les lignes nouvellement complètes depuis le lot précédent, sauts de ligne finaux inclus. Toujours des lignes entières, sauf le lot final qui peut se terminer en milieu de ligne. En exécution interactive, le delta du lot final est vide quand le message se termine par un saut de ligne : considérez donc `final`, et non un delta non vide, comme le signal de fin de message. En exécution Agent SDK et `claude -p`, l'appel unique transporte le message entier |

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../transcript.jsonl",
  "cwd": "/Users/my-project",
  "hook_event_name": "MessageDisplay",
  "turn_id": "0c9e6a2f-7d41-4f4e-9a15-3f4f7c2b8d10",
  "message_id": "5b2a9c8e-1f63-4d8a-b7c4-9e0d2a6f1c3b",
  "index": 0,
  "final": false,
  "delta": "Here is the plan:\n"
}
```

#### Sortie de MessageDisplay

En plus des [champs de sortie JSON](https://code.claude.com/docs/en/hooks#json-output) disponibles pour tous les hooks, les hooks MessageDisplay peuvent renvoyer `displayContent` pour remplacer le delta à l'écran :

| Champ | Description |
| :- | :- |
| `displayContent` | Texte affiché à la place du delta. Omettez-le pour afficher l'original |

Les hooks MessageDisplay n'ont pas de contrôle de décision. Ils ne peuvent ni bloquer le message ni modifier ce qui est stocké dans la transcription ou envoyé à Claude. Claude Code applique le `displayContent` de leur sortie JSON et ignore `systemMessage` et `continue`.

Cet exemple supprime la mise en forme markdown des réponses de Claude pour un affichage en texte brut. Le script lit chaque lot depuis stdin, retire du `delta` les marqueurs de gras et les accents graves du code en ligne, et renvoie le résultat dans `displayContent`.

##### Onglet « macOS/Linux »

Enregistrez un hook command pour l'événement dans votre fichier de réglages :

```json theme={null}
{
  "hooks": {
    "MessageDisplay": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/plain-display.sh",
            "args": []
          }
        ]
      }
    ]
  }
}
```

Enregistrez ce script dans `.claude/hooks/plain-display.sh` dans votre projet et rendez-le exécutable avec `chmod +x` :

```bash theme={null}
#!/bin/bash
jq '{hookSpecificOutput: {hookEventName: "MessageDisplay", displayContent: (.delta | gsub("\\*\\*"; "") | gsub("`"; ""))}}'
```

##### Onglet « Windows (PowerShell) »

Enregistrez un hook command qui exécute le script via PowerShell :

```json theme={null}
{
  "hooks": {
    "MessageDisplay": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe",
            "args": [
              "-NoProfile",
              "-ExecutionPolicy",
              "Bypass",
              "-File",
              "${CLAUDE_PROJECT_DIR}/.claude/hooks/plain-display.ps1"
            ]
          }
        ]
      }
    ]
  }
}
```

Le flag `-NoProfile` évite de charger votre profil PowerShell pour que le hook démarre vite, et `-ExecutionPolicy Bypass` permet à PowerShell d'exécuter le fichier de script local.

Enregistrez ce script dans `.claude/hooks/plain-display.ps1` dans votre projet :

```powershell theme={null}
$batch = [Console]::In.ReadToEnd() | ConvertFrom-Json
$text = $batch.delta -replace '\*\*', '' -replace '`', ''
@{
  hookSpecificOutput = @{
    hookEventName = "MessageDisplay"
    displayContent = $text
  }
} | ConvertTo-Json
```

Les lots sans markdown passent sans changement. Si le script échoue, par exemple parce que `jq` est absent, Claude Code affiche le texte d'origine et ne signale l'échec que dans la [sortie de débogage](https://code.claude.com/docs/en/hooks#debug-hooks), pas dans la session.

### PreToolUse

S'exécute après que Claude a créé les paramètres de l'outil et avant le traitement de l'appel d'outil. Correspond à n'importe quel nom d'outil sauf `EndConversation` : outils intégrés comme `Bash`, `PowerShell`, `Edit`, `Write`, `Read`, `Glob`, `Grep`, `Agent`, `Workflow`, `WebFetch`, `WebSearch`, `AskUserQuestion` et `ExitPlanMode`, ainsi que tous les [noms d'outils MCP](https://code.claude.com/docs/en/hooks#match-mcp-tools).

Pour exécuter un hook quand un fichier précis change sur le disque, quel qu'en soit l'auteur, utilisez [FileChanged](https://code.claude.com/docs/en/hooks#filechanged) plutôt que de faire correspondre les outils d'édition de fichiers par leur nom. Contrairement à PreToolUse, Claude Code exécute les hooks FileChanged après la modification, et ils n'ont pas de contrôle de décision : ils ne peuvent donc pas bloquer l'écriture.

> **Avertissement** : PreToolUse ne s'exécute que lorsque Claude appelle un outil. Les fichiers que vous [référencez avec `@` dans votre prompt](https://code.claude.com/docs/en/common-workflows#reference-files-and-directories) sont ajoutés sans aucun appel d'outil : Claude Code insère leur contenu en construisant le prompt, donc aucun hook PreToolUse ne se déclenche pour eux, y compris les hooks correspondant à `Read`. Pour bloquer certains chemins dans les références `@`, utilisez plutôt une [règle de refus `Read`](https://code.claude.com/docs/en/permissions#read-and-edit).
>
> PreToolUse ne se déclenche pas non plus pour [`EndConversation`](https://code.claude.com/docs/en/tools-reference#endconversation-tool-behavior).

Utilisez le [contrôle de décision de PreToolUse](#contrôle-de-décision-de-pretooluse) pour autoriser, refuser, demander confirmation ou différer l'appel d'outil.

Un [hook callback de l'Agent SDK](https://code.claude.com/docs/en/agent-sdk/hooks) sur `PreToolUse` qui dépasse son délai d'expiration bloque l'appel d'outil, et Claude reçoit un résultat d'erreur nommant ce délai. Un refus explicite renvoyé par un autre hook reste prioritaire.

#### Entrée de PreToolUse

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks PreToolUse reçoivent `tool_name`, `tool_input` et `tool_use_id`.

Pour un [outil MCP](https://code.claude.com/docs/en/hooks#match-mcp-tools), l'entrée contient aussi `mcp_server`, un objet avec le `name` du serveur et une `source` indiquant d'où provient la définition du serveur. Les valeurs de `source` incluent `plugin`, `sdk` et des portées de configuration comme `user` et `project`. [`McpServerProvenance`](https://code.claude.com/docs/en/agent-sdk/typescript#mcpserverprovenance), dans la référence de l'Agent SDK, les liste toutes et explique comment traiter une valeur inconnue. Fondez vos décisions de confiance sur `source` plutôt que sur `name` ou sur le préfixe de nom d'outil `mcp__<server>__`. Le champ `mcp_server` exige Claude Code v2.1.274+.

Pour les outils de fichiers `Write`, `Edit` et `Read`, `tool_input.file_path` est toujours absolu :

* Claude Code développe `~` et les chemins relatifs avant l'exécution des hooks ; un hook qui filtre sur les chemins ne peut donc pas être contourné via `~` ou une écriture relative du même chemin
* Sous Windows, le chemin arrive avec des barres obliques inverses comme séparateurs, même quand votre hook s'exécute sous Git Bash où `$PWD` ressemble à `/c/project`
* Une comparaison écrite avec des barres obliques, comme un test `/src/`, ne correspond jamais à un chemin avec barres obliques inverses, et l'appel d'outil se poursuit comme si le hook n'avait rien à bloquer
* Normalisez les séparateurs avant de comparer : `FILE_PATH="${FILE_PATH//\\//}"` en Bash, ou `file_path.replace("\\", "/")` en Python, puis cherchez un segment de chemin comme `/src/` plutôt que d'ancrer avec `^`, puisque le chemin est absolu

Un appel `Write` sous Windows transmet :

```json theme={null}
{
  "hook_event_name": "PreToolUse",
  "tool_name": "Write",
  "tool_input": {
    "file_path": "C:\\project\\src\\index.ts",
    "content": "..."
  },
  ...
}
```

Les champs de `tool_input` dépendent de l'outil :

##### Bash

Exécute des commandes shell.

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `command` | string | `"npm test"` | La commande shell à exécuter |
| `description` | string | `"Run test suite"` | Description facultative de ce que fait la commande |
| `timeout` | number | `120000` | Délai d'expiration facultatif en millisecondes. Les valeurs supérieures au [maximum](https://code.claude.com/docs/en/tools-reference#bash-tool-behavior) sont ramenées au maximum plutôt que rejetées |
| `run_in_background` | boolean | `false` | Indique s'il faut exécuter la commande en arrière-plan |

Quand une commande Bash modifie des fichiers dans un dépôt Git, Claude Code peut enregistrer ce qui a changé. Il enregistre les modifications dans tous les modes de permission quand le réglage [`bashEditDiffEnabled`](https://code.claude.com/docs/en/settings-reference#basheditdiffenabled) active l'enregistrement ; l'entrée de ce réglage précise quels fichiers peuvent le définir. Sinon, il ne les enregistre qu'en mode auto et en mode `bypassPermissions`, et seulement lorsque Claude Code demande à Claude d'éditer des fichiers via Bash. Définissez `bashEditDiffEnabled` à `false` pour désactiver l'enregistrement. Les commandes d'arrière-plan et les commandes en lecture seule ne portent aucun diff.

Votre [hook PostToolUse](https://code.claude.com/docs/en/hooks#posttooluse) reçoit alors les fichiers modifiés dans `tool_response.bashEditDiff`. La liste couvre ce qui a changé dans le dépôt pendant l'exécution de la commande. Les fichiers ignorés par Git et les fichiers dans des sous-modules ne sont pas listés. Exige Claude Code v2.1.269+.

> **Note** : La liste est fournie au mieux (best effort) et en bêta publique. Claude Code peut manquer une modification, inclure un fichier qu'un autre processus a modifié au même moment, ou s'arrêter à ses limites de taille. La forme du champ peut changer. Servez-vous de la liste pour trouver quoi relire, pas pour appliquer une politique.

`changedFiles` et `files` listent ce que la commande a modifié ; les autres champs indiquent à quel point cette liste est complète et fiable.

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `changedFiles` | array | `["/path/to/src/app.ts"]` | Chemins absolus des fichiers modifiés par la commande, 200 au maximum. Présent dès que `files` contient un diff ou que `moreFiles` est supérieur à zéro |
| `files` | array | `[{"filePath": "/path/to/src/app.ts", "hunks": [...]}]` | Diffs de 5 fichiers modifiés au plus, pour l'affichage. `created` ou `deleted` vaut `true` pour un fichier ajouté ou supprimé par la commande |
| `moreFiles` | number | `2` | Nombre de fichiers modifiés sans diff dans `files` |
| `unavailable` | boolean | `true` | Défini quand le diff est incomplet ou n'a pas pu être pris |
| `skipped` | boolean | `true` | Défini pour une commande Git qui déplace l'arbre de travail, comme `git checkout` ou `git stash`, auquel cas Claude Code ne prend aucun diff |
| `shared` | boolean | `true` | Défini quand un autre appel de l'outil Bash, par exemple celui d'un sous-agent, s'est exécuté dans le même dépôt au même moment : certaines modifications listées peuvent alors provenir de cette autre commande |

##### PowerShell

Exécute des commandes PowerShell. Voir l'[outil PowerShell](https://code.claude.com/docs/en/tools-reference#powershell-tool) pour la disponibilité selon la plateforme.

Les champs sont les mêmes que pour l'outil Bash, avec la chaîne de commande dans `command` :

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `command` | string | `"Get-ChildItem -Recurse"` | La commande PowerShell à exécuter |
| `description` | string | `"List files recursively"` | Description facultative de ce que fait la commande |
| `timeout` | number | `120000` | Délai d'expiration facultatif en millisecondes |
| `run_in_background` | boolean | `false` | Indique s'il faut exécuter la commande en arrière-plan |

Utilisez le matcher `Bash|PowerShell` dans les hooks qui inspectent des commandes shell, afin de couvrir les deux outils :

* Sous Windows, partout où l'outil PowerShell est activé, Claude traite PowerShell comme le shell principal et y fait passer les commandes shell.
* Sous Windows sans Git Bash, l'outil est activé automatiquement et Claude Code n'enregistre pas du tout l'outil Bash.
* Un hook qui ne correspond qu'à `Bash` ne s'y déclenche jamais.

##### Write

Crée ou écrase un fichier.

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `file_path` | string | `"/path/to/file.txt"` | Chemin absolu du fichier à écrire |
| `content` | string | `"file content"` | Contenu à écrire dans le fichier |

##### Edit

Remplace une chaîne dans un fichier existant.

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `file_path` | string | `"/path/to/file.txt"` | Chemin absolu du fichier à éditer |
| `old_string` | string | `"original text"` | Texte à rechercher et remplacer |
| `new_string` | string | `"replacement text"` | Texte de remplacement |
| `replace_all` | boolean | `false` | Indique s'il faut remplacer toutes les occurrences |

##### Read

Lit le contenu d'un fichier.

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `file_path` | string | `"/path/to/file.txt"` | Chemin absolu du fichier à lire |
| `offset` | number | `10` | Numéro de ligne facultatif à partir duquel lire |
| `limit` | number | `50` | Nombre facultatif de lignes à lire |

##### Glob

Trouve les fichiers correspondant à un motif glob.

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `pattern` | string | `"**/*.ts"` | Motif glob auquel comparer les fichiers |
| `path` | string | `"/path/to/dir"` | Répertoire facultatif dans lequel chercher. Par défaut, le répertoire de travail courant |

##### Grep

Recherche dans le contenu des fichiers avec des expressions régulières.

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `pattern` | string | `"TODO.*fix"` | Motif d'expression régulière à rechercher |
| `path` | string | `"/path/to/dir"` | Fichier ou répertoire facultatif dans lequel chercher |
| `glob` | string | `"*.ts"` | Motif glob facultatif pour filtrer les fichiers |
| `output_mode` | string | `"content"` | `"content"`, `"files_with_matches"` ou `"count"`. Par défaut `"files_with_matches"` |
| `-i` | boolean | `true` | Recherche insensible à la casse |
| `multiline` | boolean | `false` | Active la correspondance multiligne |

##### WebFetch

Récupère et traite du contenu web.

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `url` | string | `"https://example.com/api"` | URL dont récupérer le contenu |
| `prompt` | string | `"Extract the API endpoints"` | Prompt à exécuter sur le contenu récupéré |
| `offset` | number | `100000` | Nombre facultatif de caractères à sauter depuis le début de la page. Claude le définit pour continuer à lire une longue page. Exige Claude Code v2.1.290+ |

##### WebSearch

Effectue une recherche sur le web.

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `query` | string | `"react hooks best practices"` | Requête de recherche |
| `allowed_domains` | array | `["docs.example.com"]` | Facultatif : n'inclure que les résultats de ces domaines |
| `blocked_domains` | array | `["spam.example.com"]` | Facultatif : exclure les résultats de ces domaines |

##### Agent

Lance un [sous-agent](https://code.claude.com/docs/en/sub-agents).

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `prompt` | string | `"Find all API endpoints"` | La tâche que l'agent doit accomplir |
| `description` | string | `"Find API endpoints"` | Courte description de la tâche |
| `subagent_type` | string | `"Explore"` | Type d'agent spécialisé à utiliser |
| `model` | string | `"sonnet"` | Alias de modèle facultatif pour remplacer le modèle par défaut |

Quand un appel Agent au premier plan se termine, votre [hook PostToolUse](https://code.claude.com/docs/en/hooks#posttooluse) reçoit le résultat du sous-agent et la télémétrie de l'exécution dans `tool_response`. Lisez ces champs pour examiner l'exécution ; pour des cumuls de tokens et de coûts sur l'ensemble des sous-agents, utilisez les [compteurs de tokens et de coûts](https://code.claude.com/docs/en/monitoring-usage#token-counter) filtrés sur `query_source` `"subagent"`, car `totalTokens` et `usage` ne couvrent que la dernière requête :

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `status` | string | `"completed"` | `"completed"` pour les sous-agents au premier plan, `"async_launched"` pour les sous-agents en arrière-plan. Les sous-agents s'exécutent en arrière-plan par défaut : un appel Agent qui omet `run_in_background` produit donc aussi `"async_launched"` |
| `agentId` | string | `"a4d2c8f1e0b3a297"` | Identifiant de l'exécution du sous-agent |
| `content` | array | `[{"type": "text", "text": "Found 12 endpoints..."}]` | Les blocs de texte finaux du sous-agent ou, pour un sous-agent dont le rapport passe par `SubagentHandback`, une courte note sur cette remise à leur place |
| `resolvedModel` | string | `"claude-sonnet-4-5"` | Modèle sur lequel le sous-agent a démarré, qui peut différer du modèle demandé |
| `modelsUsed` | array | `["claude-sonnet-4-5", "claude-haiku-4-5"]` | Modèles utilisés dans l'ordre, les répétitions consécutives étant fusionnées ; défini uniquement si le modèle a été changé en cours d'exécution. Exige Claude Code v2.1.212+ |
| `totalTokens` | number | `12450` | Nombre de tokens de la dernière requête d'API du sous-agent : entrée, sortie et cache combinés. Ce n'est pas un total sur toute l'exécution |
| `totalDurationMs` | number | `48211` | Durée réelle (horloge murale) de l'exécution du sous-agent |
| `totalToolUseCount` | number | `7` | Nombre d'appels d'outils effectués par le sous-agent |
| `usage` | object | `{"input_tokens": 8320, ...}` | Détail des tokens par type pour la dernière requête d'API : `input_tokens`, `output_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens` |

À partir de Claude Code v2.1.271, un sous-agent qui s'exécute avec l'outil [`SubagentHandback`](https://code.claude.com/docs/en/tools-reference), que Claude Code fournit en [mode auto](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode), remet son rapport via cet outil au lieu de le renvoyer sous forme de texte. Le champ `content` de son résultat `completed` contient alors une courte note sur cette remise plutôt que le rapport lui-même. Pour lire le rapport, faites correspondre un hook `PreToolUse` ou `PostToolUse` à `SubagentHandback` et lisez `tool_input.message`.

Pour les sous-agents d'arrière-plan, l'outil rend la main quand la tâche passe en arrière-plan ; `tool_response` ne contient donc aucun champ d'utilisation : un lancement en arrière-plan rend la main immédiatement, et une tâche au premier plan que Claude Code passe en arrière-plan en cours d'exécution rend la main à cette transition. Il contient `status: "async_launched"`, `agentId`, `description`, `prompt`, `outputFile` et `resolvedModel`.

Sur une réponse `completed`, `resolvedModel` nomme le modèle sur lequel le sous-agent a démarré, qui peut différer de la valeur `model` de `tool_input`, par exemple quand `availableModels` ou une autre surcharge s'applique. Sur une réponse `async_launched`, `resolvedModel` nomme le modèle utilisé au moment où l'agent est passé en arrière-plan : un changement survenu avant ce passage y est donc reflété. `modelsUsed` et ce comportement de `resolvedModel` au passage en arrière-plan exigent Claude Code v2.1.212+.

##### AskUserQuestion

Pose à l'utilisateur une à quatre questions à choix multiples.

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `questions` | array | `[{"question": "Which framework?", "header": "Framework", "options": [{"label": "React", "description": "Component library"}, {"label": "Vue", "description": "Progressive framework"}], "multiSelect": false}]` | Questions à présenter, chacune avec une chaîne `question`, un court `header`, un tableau `options` et un drapeau facultatif `multiSelect` |
| `answers` | object | `{"Which framework?": "React"}` | Facultatif. Associe le texte de chaque question au libellé de l'option choisie. Les réponses à sélection multiple joignent les libellés par des virgules. Claude ne définit pas ce champ ; fournissez-le via `updatedInput` pour répondre par programme |

##### ExitPlanMode

Présente un plan et demande à l'utilisateur de l'approuver avant que Claude ne quitte le [mode plan](https://code.claude.com/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode). Claude écrit le plan dans un fichier sur le disque avant d'appeler l'outil ; le `tool_input` littéral du modèle est donc généralement vide. Claude Code injecte le contenu du plan et le chemin du fichier avant de transmettre l'entrée aux hooks.

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `plan` | string | `"## Refactor auth\n1. Extract..."` | Contenu du plan en Markdown. Injecté depuis le fichier de plan sur le disque |
| `planFilePath` | string | `"/Users/.../plans/refactor-auth.md"` | Chemin du fichier de plan. Injecté |
| `allowedPrompts` | array | `[{"tool": "Bash", "prompt": "run tests"}]` | Obsolète. Claude Code accepte le champ mais l'ignore. Avant la v2.1.205, il transportait les permissions basées sur des prompts que Claude demandait pour mettre en œuvre le plan |

Dans `PostToolUse`, `tool_response` est un objet avec les champs `plan` et `filePath` contenant le plan approuvé, plus des indicateurs d'état internes. Lisez `tool_response.plan` pour obtenir le contenu du plan plutôt que de relire le fichier sur le disque.

#### Contrôle de décision de PreToolUse

Les hooks `PreToolUse` peuvent contrôler si un appel d'outil se poursuit. Contrairement aux autres hooks qui utilisent un champ `decision` de premier niveau, PreToolUse renvoie sa décision dans un objet `hookSpecificOutput`. Cela lui donne un contrôle plus riche : quatre issues (allow, deny, ask ou defer) plus la possibilité de modifier l'entrée de l'outil avant son exécution.

| Champ | Description |
| :- | :- |
| `permissionDecision` | `"allow"` évite la demande de permission, sauf pour les [actions qu'aucun mode n'approuve automatiquement](https://code.claude.com/docs/en/permission-modes#actions-no-mode-auto-approves), pour les [lectures sur des chemins réseau](https://code.claude.com/docs/en/permissions#network-paths), et pour `AskUserQuestion` et `ExitPlanMode`, qui exigent un [`updatedInput` associé](#outils-nécessitant-une-interaction-utilisateur). `"deny"` empêche l'appel d'outil. `"ask"` demande confirmation à l'utilisateur. `"defer"` sort proprement pour que l'outil puisse être repris plus tard. Les [règles deny et ask](https://code.claude.com/docs/en/permissions#manage-permissions) sont évaluées quoi que renvoie le hook |
| `permissionDecisionReason` | Pour `"ask"`, affiché à l'utilisateur dans la demande de permission. Quand Claude Code [refuse l'appel](https://code.claude.com/docs/en/headless#turn-off-permission-prompts-in-unattended-runs) dans une exécution `-p` où personne ne peut répondre à cette demande, Claude lit la raison dans le résultat de l'outil. Pour `"deny"`, affiché à Claude. Pour `"allow"` et `"defer"`, écrit uniquement dans le [journal de débogage](https://code.claude.com/docs/en/hooks#debug-hooks) |
| `updatedInput` | Modifie les paramètres d'entrée de l'outil avant l'exécution. Remplace l'objet d'entrée entier : incluez les champs inchangés avec ceux modifiés. Claude Code évalue les règles de permission et l'[éligibilité au passage automatique en arrière-plan](https://code.claude.com/docs/en/tools-reference#foreground-commands-that-move-to-the-background) d'une commande Bash sur l'entrée renvoyée par votre hook, pas sur celle envoyée par Claude. Combinez avec `"allow"` pour approuver automatiquement, ou avec `"ask"` pour montrer l'entrée modifiée à l'utilisateur. Ignoré pour `"defer"` |
| `additionalContext` | Chaîne ajoutée au contexte de Claude en même temps que le résultat de l'outil. Ignorée quand `permissionDecision` vaut `"defer"`. Voir [Ajouter du contexte pour Claude](https://code.claude.com/docs/en/hooks#add-context-for-claude) |

Quand plusieurs hooks PreToolUse renvoient des décisions différentes, l'ordre de priorité est `deny` > `defer` > `ask` > `allow`.

Un hook qui bloque en sortant avec le code 2 est traité comme `"deny"` : Claude voit le message stderr comme raison du refus.

Quand un hook renvoie `"ask"`, la demande de permission affichée à l'utilisateur comporte une étiquette indiquant l'origine du hook : `[settings]` pour un hook issu d'un fichier de réglages quelconque ou du frontmatter d'un agent, `[plugin:<name>]` pour le hook d'un plugin, ou `[skill]` pour un hook issu du frontmatter d'une skill. Cela aide les utilisateurs à comprendre quelle source de configuration demande une confirmation.

Le `"ask"` d'un hook force aussi une demande de permission en [mode auto](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) : le classifieur peut toujours refuser l'appel d'outil, mais il ne peut pas l'approuver silencieusement. Avant la v2.1.211, le classifieur pouvait approuver une commande Bash s'exécutant hors de la [sandbox](https://code.claude.com/docs/en/sandboxing) sans afficher la demande requise par le hook ; le classifieur appliquait toutefois ses propres règles de sécurité à cette commande, et un `"deny"` de hook était toujours respecté.

```json theme={null}
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow",
    "permissionDecisionReason": "My reason here",
    "updatedInput": {
      "field_to_modify": "new value"
    },
    "additionalContext": "Current environment: production. Proceed with caution."
  }
}
```

> **Note** : PreToolUse utilisait auparavant les champs de premier niveau `decision` et `reason`, mais ils sont obsolètes pour cet événement. Utilisez plutôt `hookSpecificOutput.permissionDecision` et `hookSpecificOutput.permissionDecisionReason`. Les anciennes valeurs `"approve"` et `"block"` correspondent respectivement à `"allow"` et `"deny"`. Les autres événements, comme PostToolUse et Stop, continuent d'utiliser `decision` et `reason` de premier niveau comme format actuel.

#### Outils nécessitant une interaction utilisateur

`AskUserQuestion` et `ExitPlanMode` nécessitent une interaction utilisateur. En [mode non interactif](https://code.claude.com/docs/en/headless) avec le flag `-p`, Claude Code ne les propose que si l'exécution dispose d'un [hôte de permissions](https://code.claude.com/docs/en/headless#turn-off-permission-prompts-in-unattended-runs) pour recevoir la demande, comme un callback `canUseTool` de l'Agent SDK.

Un hook `PreToolUse` satisfait cette exigence lorsqu'il :

1. Lit l'entrée de l'outil depuis stdin
2. Recueille la réponse via votre propre interface
3. Renvoie `permissionDecision: "allow"` avec un `updatedInput` contenant la réponse, afin que l'outil s'exécute sans demande

Renvoyer `"allow"` seul ne suffit pas pour ces outils.

Pour `AskUserQuestion`, renvoyez le tableau `questions` d'origine et ajoutez un objet [`answers`](#askuserquestion) associant le texte de chaque question à la réponse choisie. Cette sortie répond `React` à une question :

```json theme={null}
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow",
    "updatedInput": {
      "questions": [
        {
          "question": "Which framework?",
          "header": "Framework",
          "options": [{"label": "React", "description": "Component library"}, {"label": "Vue", "description": "Progressive framework"}],
          "multiSelect": false
        }
      ],
      "answers": {"Which framework?": "React"}
    }
  }
}
```

Un outil MCP que son serveur marque avec [`_meta["anthropic/requiresUserInteraction"]`](https://code.claude.com/docs/en/mcp#require-approval-for-a-specific-tool) est plus strict : un hook ne peut pas éviter sa demande d'approbation avec `"allow"`, avec ou sans `updatedInput`, car Claude Code ne peut pas confirmer que le hook a recueilli l'interaction dont l'outil a besoin.

#### Différer un appel d'outil pour plus tard

`"defer"` est destiné aux intégrations qui exécutent `claude -p` comme sous-processus et lisent sa sortie JSON, comme une application Agent SDK ou une interface personnalisée construite sur Claude Code. Il permet à ce processus appelant de mettre Claude en pause sur un appel d'outil, de recueillir une saisie via sa propre interface, puis de reprendre là où il s'était arrêté. Claude Code ne respecte cette valeur qu'en [mode non interactif](https://code.claude.com/docs/en/headless) avec le flag `-p`. En session interactive, il journalise un avertissement et ignore le résultat du hook.

L'outil `AskUserQuestion` est le cas typique : Claude veut poser une question à l'utilisateur, mais il n'y a pas de terminal pour y répondre. Une exécution `-p` ne propose `AskUserQuestion` que si elle dispose d'un [hôte de permissions](https://code.claude.com/docs/en/headless#turn-off-permission-prompts-in-unattended-runs), comme un outil MCP passé avec `--permission-prompt-tool` : démarrez donc l'exécution avec un tel hôte. L'aller-retour se déroule ainsi :

1. Claude appelle `AskUserQuestion`. Le hook `PreToolUse` se déclenche.
2. Le hook renvoie `permissionDecision: "defer"`. L'outil ne s'exécute pas. Le processus se termine avec `stop_reason: "tool_deferred"` et l'appel d'outil en attente est conservé dans la transcription.
3. Le processus appelant lit `deferred_tool_use` dans le résultat du SDK, affiche la question dans sa propre interface et attend une réponse.
4. Le processus appelant exécute `claude -p --resume <session-id>` avec le même hôte de permissions. Le même appel d'outil déclenche à nouveau `PreToolUse`.
5. Le hook renvoie `permissionDecision: "allow"` avec la réponse dans `updatedInput`. L'outil s'exécute et Claude continue.

Le champ `deferred_tool_use` contient l'`id`, le `name` et l'`input` de l'outil. L'`input` correspond aux paramètres que Claude a générés pour l'appel d'outil, capturés avant l'exécution :

```json theme={null}
{
  "type": "result",
  "subtype": "success",
  "stop_reason": "tool_deferred",
  "session_id": "abc123",
  "deferred_tool_use": {
    "id": "toolu_01abc",
    "name": "AskUserQuestion",
    "input": { "questions": [{ "question": "Which framework?", "header": "Framework", "options": [{"label": "React", "description": "Component library"}, {"label": "Vue", "description": "Progressive framework"}], "multiSelect": false }] }
  }
}
```

Il n'y a ni délai d'expiration ni limite de tentatives. La session reste sur le disque jusqu'à ce que vous la repreniez, sous réserve du balayage de rétention [`cleanupPeriodDays`](https://code.claude.com/docs/en/settings-reference#cleanupperioddays), qui supprime les fichiers de session après 30 jours par défaut, selon les [règles du balayage de rétention](https://code.claude.com/docs/en/claude-directory#cleaned-up-automatically). Si la réponse n'est pas prête lors de la reprise, le hook peut renvoyer `"defer"` à nouveau et le processus se termine de la même façon. Le processus appelant décide quand sortir de la boucle en renvoyant finalement `"allow"` ou `"deny"` depuis le hook.

`"defer"` ne fonctionne que si Claude fait un seul appel d'outil dans le tour. Si Claude fait plusieurs appels d'outils à la fois, `"defer"` est ignoré avec un avertissement et l'outil suit le circuit de permission normal. Cette contrainte existe parce que la reprise ne peut ré-exécuter qu'un seul outil : il est impossible de différer un appel d'un lot sans laisser les autres non résolus.

Si l'outil différé n'est plus disponible lors de la reprise, le processus se termine avec `stop_reason: "tool_deferred_unavailable"` et `is_error: true` avant que le hook ne se déclenche. Cela arrive quand un serveur MCP qui fournissait l'outil n'est pas connecté pour la session reprise. La charge utile `deferred_tool_use` est tout de même incluse pour que vous puissiez identifier l'outil manquant.

> **Note** : Pour reprendre une session différée en mode plan, passez [`--permission-prompt-tool`](https://code.claude.com/docs/en/cli-reference#cli-flags) avec `--resume` afin que Claude Code puisse présenter le plan pour approbation. Pour les autres conditions, voir [Reprendre en mode plan avec `-p`](https://code.claude.com/docs/en/sessions#resume-in-plan-mode-with-p). Exige Claude Code v2.1.246+.
>
> Quand vous reprenez avec `-p`, Claude Code ne restaure aucun autre mode de permission enregistré. Il démarre l'exécution dans le mode de permission dans lequel démarrerait une nouvelle exécution `claude -p` : repassez donc `--permission-mode` ou `--dangerously-skip-permissions` si la session différée en utilisait un. Quand vous reprenez avec `claude --resume <session-id>` sans `-p`, Claude Code restaure le mode de permission enregistré, avec les exceptions listées dans [Mode de permission à la reprise](https://code.claude.com/docs/en/sessions#permission-mode-on-resume).

### PermissionRequest

S'exécute quand Claude Code est sur le point de vous demander la permission d'utiliser un outil. Dans les sessions qui ne peuvent pas afficher de demande, comme les sous-agents d'arrière-plan en [mode non interactif](https://code.claude.com/docs/en/headless), Claude Code exécute quand même ces hooks et, si aucun hook ne renvoie de décision, il refuse l'appel d'outil. Pour un appel qui atteint un `--permission-prompt-tool` ou le [callback `canUseTool`](https://code.claude.com/docs/en/agent-sdk/permissions) de l'Agent SDK, les hooks s'exécutent en parallèle de votre hôte, et la première décision rendue s'applique.
Utilisez le [contrôle de décision de PermissionRequest](#contrôle-de-décision-de-permissionrequest) pour autoriser ou refuser au nom de l'utilisateur.

Utilisez cet événement quand vous avez besoin d'un signal dès l'instant où Claude demande la permission d'utiliser un outil. Claude Code n'exécute un hook [Notification](#notification) de type `permission_prompt` qu'après que la demande a attendu environ six secondes.

Claude Code n'exécute pas les hooks PermissionRequest pour la [requête réseau](https://code.claude.com/docs/en/sandboxing#network-isolation) d'une commande en sandbox. Pour obtenir un signal pour cette demande, utilisez le type de notification `permission_prompt`.

Le matcher porte sur le nom de l'outil, avec les mêmes valeurs que PreToolUse.

#### Entrée de PermissionRequest

Les hooks PermissionRequest reçoivent les champs `tool_name` et `tool_input` comme les hooks PreToolUse, mais sans `tool_use_id`. Pour un outil MCP, ils reçoivent aussi l'objet [`mcp_server`](#entrée-de-pretooluse). Un tableau facultatif `permission_suggestions` contient les [mises à jour de permissions](#entrées-de-mise-à-jour-de-permissions) que Claude Code suggère pour cette demande, comme l'ajout d'une règle d'autorisation ou le changement de mode de permission.

Le tableau `permission_suggestions` n'est pas la liste exacte des options que vous voyez, car chaque boîte de dialogue de permission construit ses propres options. Certaines boîtes de dialogue, comme celle des modifications de fichiers, ne lisent pas du tout ce tableau et déduisent leurs options de la demande elle-même. Une boîte de dialogue qui le lit peut néanmoins masquer une option dont la suggestion reste dans le tableau, par exemple quand [`allowManagedPermissionRulesOnly`](https://code.claude.com/docs/en/settings-reference#allowmanagedpermissionrulesonly) masque les options d'enregistrement de règles. Elle peut aussi proposer des options sans entrée de suggestion, comme [**Yes, and switch to auto mode**](https://code.claude.com/docs/en/permission-modes#switch-permission-modes), qui change directement le mode de permission plutôt que de passer par une mise à jour de permissions.

Les hooks PreToolUse s'exécutent avant chaque appel d'outil, qu'il nécessite une permission ou non. Les hooks PermissionRequest ne s'exécutent que lorsque Claude Code est sur le point de vous demander la permission, ou lorsqu'il refuserait automatiquement un appel qui ne peut pas afficher de demande. Aucun de ces deux événements ne se déclenche pour [`EndConversation`](https://code.claude.com/docs/en/tools-reference#endconversation-tool-behavior).

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "cwd": "/Users/...",
  "permission_mode": "default",
  "hook_event_name": "PermissionRequest",
  "tool_name": "Bash",
  "tool_input": {
    "command": "rm -rf node_modules",
    "description": "Remove node_modules directory"
  },
  "permission_suggestions": [
    {
      "type": "addRules",
      "rules": [{ "toolName": "Bash", "ruleContent": "rm -rf node_modules" }],
      "behavior": "allow",
      "destination": "localSettings"
    }
  ]
}
```

#### Contrôle de décision de PermissionRequest

Les hooks `PermissionRequest` peuvent autoriser ou refuser des demandes de permission. En plus des [champs de sortie JSON](https://code.claude.com/docs/en/hooks#json-output) disponibles pour tous les hooks, votre script de hook peut renvoyer un objet `decision` avec ces champs propres à l'événement :

| Champ | Description |
| :- | :- |
| `behavior` | `"allow"` accorde la permission, `"deny"` la refuse. Les [règles deny et ask](https://code.claude.com/docs/en/permissions#manage-permissions) sont toujours évaluées : un hook renvoyant `"allow"` ne l'emporte donc pas sur une règle deny correspondante |
| `updatedInput` | Pour `"allow"` uniquement : modifie les paramètres d'entrée de l'outil avant l'exécution. Remplace l'objet d'entrée entier : incluez les champs inchangés avec ceux modifiés. L'entrée modifiée est réévaluée par rapport aux règles deny et ask |
| `updatedPermissions` | Pour `"allow"` uniquement : tableau d'[entrées de mise à jour de permissions](#entrées-de-mise-à-jour-de-permissions) à appliquer, comme l'ajout d'une règle d'autorisation ou le changement du mode de permission de la session |
| `message` | Pour `"deny"` uniquement : indique à Claude pourquoi la permission a été refusée |
| `interrupt` | Pour `"deny"` uniquement : si `true`, arrête Claude |

Un hook qui sort avec le code 2 sans objet `decision` laisse le circuit de permission inchangé, et son stderr est ignoré. Pour accorder ou refuser la demande, renvoyez l'objet `decision`.

```json theme={null}
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "decision": {
      "behavior": "allow",
      "updatedInput": {
        "command": "npm run lint"
      }
    }
  }
}
```

#### Entrées de mise à jour de permissions

Le champ de sortie `updatedPermissions` et le [champ d'entrée `permission_suggestions`](#entrée-de-permissionrequest) utilisent le même tableau d'objets d'entrée. Chaque entrée a un `type` qui détermine ses autres champs, et une `destination` qui contrôle où la modification est écrite.

| `type` | Champs | Effet |
| :- | :- | :- |
| `addRules` | `rules`, `behavior`, `destination` | Ajoute des règles de permission. `rules` est un tableau d'objets `{toolName, ruleContent?}`. Omettez `ruleContent` pour cibler l'outil entier. `behavior` vaut `"allow"`, `"deny"` ou `"ask"` |
| `replaceRules` | `rules`, `behavior`, `destination` | Remplace toutes les règles du `behavior` donné à la `destination` par les `rules` fournies |
| `removeRules` | `rules`, `behavior`, `destination` | Supprime les règles correspondantes du `behavior` donné |
| `setMode` | `mode`, `destination` | Change le mode de permission. Modes valides : `default`, `auto`, `acceptEdits`, `dontAsk`, `bypassPermissions`, `plan`, et `manual` comme alias de `default` |
| `addDirectories` | `directories`, `destination` | Ajoute des répertoires de travail. `directories` est un tableau de chemins (chaînes) |
| `removeDirectories` | `directories`, `destination` | Supprime des répertoires de travail |

> **Note** : `setMode` avec `bypassPermissions` ne prend effet que si vous avez lancé la session avec le mode bypass déjà disponible : `--dangerously-skip-permissions`, `--permission-mode bypassPermissions`, `--allow-dangerously-skip-permissions`, ou `permissions.defaultMode: "bypassPermissions"` dans les [réglages utilisateur, `--settings` ou gérés](https://code.claude.com/docs/en/settings-reference#permissions-defaultmode). Sinon, la mise à jour est sans effet. Elle est aussi sans effet quand [`permissions.disableBypassPermissionsMode`](https://code.claude.com/docs/en/permissions#managed-settings) désactive le mode, ou quand la session démarre en [mode restreint](https://code.claude.com/docs/en/cli-reference#cli-flags).
>
> `bypassPermissions` n'est jamais persisté comme `defaultMode`, quelle que soit la `destination`.

Le champ `destination` de chaque entrée détermine si la modification reste en mémoire ou est persistée dans un fichier de réglages.

| `destination` | Écrit dans |
| :- | :- |
| `session` | en mémoire uniquement, abandonné à la fin de la session |
| `localSettings` | `.claude/settings.local.json` |
| `projectSettings` | `.claude/settings.json` |
| `userSettings` | `~/.claude/settings.json` |

Un hook peut renvoyer telle quelle l'une des `permission_suggestions` reçues comme sa propre sortie `updatedPermissions`.

### PostToolUse

S'exécute immédiatement après qu'un outil s'est terminé avec succès.

Le matcher porte sur le nom de l'outil, avec les mêmes valeurs que PreToolUse.

Élargissez la correspondance quand le nom de l'outil n'est pas le bon filtre :

* Pour exécuter un hook après la réussite de n'importe quel outil, omettez le `matcher` ou mettez-le à `"*"`. Votre hook peut alors découvrir lui-même ce qui a changé, par exemple en exécutant `git status --porcelain`, qui liste aussi les fichiers non suivis que `git diff` ne voit pas. Pour les appels d'outils qui échouent, ajoutez le même hook sous [PostToolUseFailure](#posttoolusefailure).
* Pour exécuter un hook quand un fichier précis change sur le disque, quel qu'en soit l'auteur, utilisez [FileChanged](https://code.claude.com/docs/en/hooks#filechanged). Claude Code n'exécute pas un hook `PostToolUse` correspondant à `Edit|Write` quand une commande `Bash` ou un processus extérieur à Claude Code réécrit le même fichier.

#### Entrée de PostToolUse

Les hooks `PostToolUse` se déclenchent après qu'un outil s'est déjà exécuté avec succès. L'entrée inclut à la fois `tool_input`, les arguments envoyés à l'outil, et `tool_response`, le résultat qu'il a renvoyé. Le schéma exact des deux dépend de l'outil. Les chemins `tool_input` des outils de fichiers arrivent dans le même format que pour [PreToolUse](#entrée-de-pretooluse) : toujours absolus, avec les séparateurs natifs de la plateforme, donc des barres obliques inverses sous Windows. Pour un outil MCP, l'entrée contient aussi l'objet [`mcp_server`](#entrée-de-pretooluse).

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "cwd": "/Users/...",
  "permission_mode": "default",
  "hook_event_name": "PostToolUse",
  "tool_name": "Write",
  "tool_input": {
    "file_path": "/path/to/file.txt",
    "content": "file content"
  },
  "tool_response": {
    "filePath": "/path/to/file.txt",
    "type": "create"
  },
  "tool_use_id": "toolu_01ABC123...",
  "duration_ms": 12
}
```

| Champ | Description |
| :- | :- |
| `duration_ms` | Facultatif. Durée d'exécution de l'outil en millisecondes. Exclut le temps passé dans les demandes de permission et les hooks PreToolUse |

#### Contrôle de décision de PostToolUse

Les hooks `PostToolUse` peuvent fournir un retour à Claude après l'exécution de l'outil. En plus des [champs de sortie JSON](https://code.claude.com/docs/en/hooks#json-output) disponibles pour tous les hooks, votre script de hook peut renvoyer ces champs propres à l'événement :

| Champ | Description |
| :- | :- |
| `decision` | `"block"` ajoute la `reason` à côté du résultat de l'outil. Claude voit toujours la sortie d'origine ; pour la remplacer, utilisez `updatedToolOutput` |
| `reason` | Explication montrée à Claude quand `decision` vaut `"block"` |
| `additionalContext` | Chaîne ajoutée au contexte de Claude en même temps que le résultat de l'outil. Voir [Ajouter du contexte pour Claude](https://code.claude.com/docs/en/hooks#add-context-for-claude) |
| `classifierContext` | Courte note sur le résultat de cet appel, destinée au classifieur du [mode auto](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) plutôt qu'à Claude. Voir [Annoter un résultat pour le classifieur du mode auto](#annoter-un-résultat-pour-le-classifieur-du-mode-auto). Exige Claude Code v2.1.236+ |
| `updatedToolOutput` | Remplace la sortie de l'outil par la valeur fournie avant qu'elle ne soit envoyée à Claude. La valeur doit respecter la forme de sortie de l'outil |
| `updatedMCPToolOutput` | Remplace la sortie pour les [outils MCP](https://code.claude.com/docs/en/hooks#match-mcp-tools) uniquement. Préférez `updatedToolOutput`, qui fonctionne pour tous les outils |

L'exemple ci-dessous remplace la sortie d'un appel `Bash`. La valeur de remplacement respecte la forme de sortie de l'outil `Bash` :

```json theme={null}
{
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "additionalContext": "Additional information for Claude",
    "updatedToolOutput": {
      "stdout": "[redacted]",
      "stderr": "",
      "interrupted": false,
      "isImage": false
    }
  }
}
```

> **Avertissement** : `updatedToolOutput` ne modifie que ce que voit Claude. L'outil s'est déjà exécuté au moment où le hook se déclenche : tout fichier écrit, toute commande exécutée ou toute requête réseau envoyée a déjà pris effet. La télémétrie, comme les spans d'outils OpenTelemetry et les événements d'analytique, capture aussi la sortie d'origine avant l'exécution du hook. Pour empêcher ou modifier un appel d'outil avant son exécution, utilisez plutôt un hook [PreToolUse](#pretooluse).
>
> La valeur de remplacement doit respecter la forme de sortie de l'outil. Les outils intégrés renvoient des objets structurés plutôt que de simples chaînes. Par exemple, `Bash` renvoie un objet avec les champs `stdout`, `stderr`, `interrupted` et `isImage`. Pour les outils intégrés, une valeur qui ne respecte pas le schéma de sortie de l'outil est ignorée et la sortie d'origine est utilisée. La sortie des outils MCP est transmise sans validation de schéma. Supprimer des détails d'erreur dont Claude a besoin peut l'amener à poursuivre sur une hypothèse fausse.

#### Annoter un résultat pour le classifieur du mode auto

Renvoyez `classifierContext` pour envoyer une courte note sur le résultat de l'appel d'outil au classifieur du [mode auto](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) plutôt qu'à Claude. Le classifieur [ne reçoit jamais les résultats d'outils eux-mêmes](https://code.claude.com/docs/en/permission-modes#how-the-classifier-evaluates-actions) : ce champ est donc le moyen pris en charge pour lui dire quelque chose sur ce qu'un appel a renvoyé avant qu'il n'examine les actions suivantes. Le champ exige Claude Code v2.1.236+.

L'exemple ci-dessous indique au classifieur d'où provient la sortie d'une requête :

```json theme={null}
{
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "classifierContext": "This query ran against the staging database, not production."
  }
}
```

Le poids que le classifieur accorde à la note dépend de l'endroit où vous avez configuré le hook :

* **Hooks configurés dans Claude Code** : pour les hooks issus de fichiers de réglages, de plugins, de skills et du frontmatter d'agents, le classifieur traite la note comme un contexte non vérifié fourni par l'application. La note n'établit jamais l'intention de l'utilisateur, et si elle affirme que vous avez approuvé ou demandé quelque chose, le classifieur vérifie cette affirmation par rapport à vos propres messages dans la conversation
* **Callbacks Agent SDK en cours de processus** : quand une application qui embarque Claude Code enregistre le hook comme [callback du SDK TypeScript](https://code.claude.com/docs/en/agent-sdk/hooks) et renvoie la note pendant la session en direct, le classifieur peut considérer une déclaration de l'utilisateur relayée dans la note comme une intention de l'utilisateur. Une telle déclaration peut satisfaire une exigence de consentement que le classifieur accepterait d'un message que vous envoyez, mais elle ne lève jamais un blocage que votre propre message ne pourrait pas lever non plus. Après la reprise d'une session, Claude Code traite les notes restaurées comme un contexte non vérifié. Quand des hooks des deux groupes annotent le même appel, le classifieur traite la note combinée comme non vérifiée

Claude Code applique ces limites lors de la transmission de la note :

* **Longueur** : Claude Code plafonne les notes d'un appel d'outil à 2 000 caractères et tronque le reste. Le plafond est partagé entre tous les hooks qui répondent à cet appel
* **Réponses synchrones uniquement** : Claude Code ignore le champ dans la réponse d'un hook qui [s'exécute en arrière-plan](https://code.claude.com/docs/en/hooks#run-hooks-in-the-background), car cette réponse arrive après que Claude Code a enregistré le résultat de l'outil
* **Appels que le classifieur n'enregistre pas** : la transcription du classifieur omet les consultations en lecture seule comme les lectures de fichiers et les recherches. Claude Code ignore une note attachée à l'un de ces appels
* **Interaction avec les réécritures** : quand la note décrit une sortie que vous remplacez avec `updatedToolOutput`, renvoyez les deux champs dans la même réponse de hook. Claude Code abandonne la note si cette réécriture est rejetée ou si la réécriture d'un autre hook la remplace. Claude Code transmet une note renvoyée sans réécriture même si un autre hook réécrit la sortie

> **Avertissement** : Le classifieur lit le contenu placé dans `classifierContext` comme une information provenant de l'application qui héberge la session : n'y copiez donc pas de sortie d'outil non fiable ni de texte tiers. Limitez la note à une courte affirmation sur cet appel précis, comme un fait sur son origine ou une déclaration de l'utilisateur à son sujet ; n'utilisez pas ce champ pour transmettre des messages sans rapport ou un flux d'événements.

### PostToolUseFailure

S'exécute quand un outil dont l'exécution a commencé échoue : l'outil a levé une erreur, ou un outil MCP a renvoyé un résultat d'erreur. Utilisez-le pour journaliser les échecs, envoyer des alertes ou fournir un retour correctif à Claude.

Le matcher porte sur le nom de l'outil, avec les mêmes valeurs que PreToolUse.

> **Note** : Cet événement ne se déclenche pas pour les appels d'outils rejetés avant exécution : nom d'outil inconnu, entrée qui échoue à la validation de schéma ou à une validation propre à l'outil, ou refus de permission. Les rejets de validation sont renvoyés comme résultats `tool_use_error` et surviennent avant l'exécution des hooks : ils ne déclenchent donc ni `PreToolUse` ni `PostToolUseFailure`. Les refus de permission déclenchent `PreToolUse` mais pas cet événement ; voir [PermissionDenied](#permissiondenied).

#### Entrée de PostToolUseFailure

Les hooks PostToolUseFailure reçoivent les mêmes champs `tool_name` et `tool_input` que PostToolUse, ainsi que des informations d'erreur sous forme de champs de premier niveau. Pour un outil MCP, ils reçoivent aussi l'objet [`mcp_server`](#entrée-de-pretooluse). Par exemple, une commande `npm test` en échec pourrait transmettre :

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "cwd": "/Users/...",
  "permission_mode": "default",
  "hook_event_name": "PostToolUseFailure",
  "tool_name": "Bash",
  "tool_input": {
    "command": "npm test",
    "description": "Run test suite"
  },
  "tool_use_id": "toolu_01ABC123...",
  "error": "Exit code 1\nError: Cannot find module 'express'",
  "is_interrupt": false,
  "duration_ms": 4187
}
```

| Champ | Description |
| :- | :- |
| `error` | Chaîne décrivant ce qui s'est mal passé. Le format dépend de l'outil en échec |
| `is_interrupt` | Booléen facultatif. Vrai quand l'échec est parvenu à Claude Code comme un abandon plutôt que comme une erreur signalée par l'outil. Annuler un outil en cours d'exécution ne déclenche pas ce hook ; le résultat de l'outil contient alors le message d'interruption |
| `duration_ms` | Facultatif. Durée d'exécution de l'outil en millisecondes. Exclut le temps passé dans les demandes de permission et les hooks PreToolUse |

La chaîne `error` est généralement le même texte que Claude reçoit comme résultat de l'outil en échec. Son format varie selon l'outil et le type d'échec. Basez votre hook sur `tool_name`, `is_interrupt` et la première ligne `Exit code N` ; traitez le reste de la chaîne comme du texte d'affichage, pas comme un format stable.

* Pour Bash et PowerShell, une commande qui s'est exécutée et s'est terminée produit une première ligne `Exit code N`, puis toute la sortie de la commande en un seul bloc, stdout et stderr entremêlés
* Une charge utile peut aussi contenir un simple message d'échec sans ligne de code de sortie, quand Claude Code n'a pas pu démarrer le processus shell lui-même
* Claude Code tronque par le milieu les longues chaînes autour d'un marqueur `... [N characters truncated] ...`, et peut insérer ses propres lignes, comme `Command timed out after 2m 0s`

#### Contrôle de décision de PostToolUseFailure

Les hooks `PostToolUseFailure` peuvent fournir du contexte à Claude après l'échec d'un outil. En plus des [champs de sortie JSON](https://code.claude.com/docs/en/hooks#json-output) disponibles pour tous les hooks, votre script de hook peut renvoyer ces champs propres à l'événement :

| Champ | Description |
| :- | :- |
| `additionalContext` | Chaîne ajoutée au contexte de Claude en même temps que l'erreur. Voir [Ajouter du contexte pour Claude](https://code.claude.com/docs/en/hooks#add-context-for-claude) |

```json theme={null}
{
  "hookSpecificOutput": {
    "hookEventName": "PostToolUseFailure",
    "additionalContext": "Additional information about the failure for Claude"
  }
}
```

### PostToolBatch

S'exécute une fois après la résolution de tous les appels d'outils d'un lot, avant que Claude Code n'envoie la requête suivante au modèle. `PostToolUse` se déclenche une fois par outil, ce qui signifie qu'il se déclenche en parallèle quand Claude fait des appels d'outils parallèles. `PostToolBatch` se déclenche exactement une fois avec le lot complet : c'est donc le bon endroit pour injecter du contexte qui dépend de l'ensemble des outils exécutés plutôt que d'un outil isolé. Il n'y a pas de matcher pour cet événement.

#### Entrée de PostToolBatch

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks PostToolBatch reçoivent `tool_calls`, un tableau décrivant chaque appel d'outil du lot :

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "cwd": "/Users/...",
  "permission_mode": "default",
  "hook_event_name": "PostToolBatch",
  "tool_calls": [
    {
      "tool_name": "Read",
      "tool_input": {"file_path": "/.../ledger/accounts.py"},
      "tool_use_id": "toolu_01...",
      "tool_response": "1\tfrom __future__ import annotations\n2\t..."
    },
    {
      "tool_name": "Read",
      "tool_input": {"file_path": "/.../ledger/transactions.py"},
      "tool_use_id": "toolu_02...",
      "tool_response": "1\tfrom __future__ import annotations\n2\t..."
    }
  ]
}
```

`tool_response` contient le même contenu que celui que le modèle reçoit dans le bloc `tool_result` correspondant. La valeur est une chaîne sérialisée ou un tableau de blocs de contenu, exactement tel que l'outil l'a émis. Pour `Read`, il s'agit donc d'un texte préfixé par des numéros de ligne plutôt que du contenu brut du fichier. Les réponses peuvent être volumineuses : n'analysez que les champs dont vous avez besoin.

> **Note** : La forme de `tool_response` diffère de celle de `PostToolUse`. `PostToolUse` transmet l'objet `Output` structuré de l'outil, comme `{filePath: "...", type: "create"}` pour `Write` ; `PostToolBatch` transmet le contenu `tool_result` sérialisé que voit le modèle.

#### Contrôle de décision de PostToolBatch

Les hooks `PostToolBatch` peuvent injecter du contexte pour Claude. En plus des [champs de sortie JSON](https://code.claude.com/docs/en/hooks#json-output) disponibles pour tous les hooks, votre script de hook peut renvoyer ces champs propres à l'événement :

| Champ | Description |
| :- | :- |
| `additionalContext` | Chaîne de contexte injectée une seule fois avant le prochain appel au modèle. Voir [Ajouter du contexte pour Claude](https://code.claude.com/docs/en/hooks#add-context-for-claude) pour les détails de transmission, ce qu'il faut y mettre et la façon dont les sessions reprises traitent les valeurs passées |

```json theme={null}
{
  "hookSpecificOutput": {
    "hookEventName": "PostToolBatch",
    "additionalContext": "These files are part of the ledger module. Run pytest before marking the task complete."
  }
}
```

Renvoyer `decision: "block"` ou `continue: false` arrête la boucle agentique avant le prochain appel au modèle. Le message de blocage provient du `reason` ou du `stopReason` JSON, ou de stderr en cas de sortie avec le code 2. Vous le voyez comme un avertissement dans la transcription, et il reste dans la conversation : Claude le voit donc quand la conversation continue.

### PermissionDenied

S'exécute quand le [mode auto](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) refuse un appel d'outil, y compris quand il refuse sans verdict du classifieur parce qu'[un contrôle de sécurité distinct du mode auto a refusé la requête propre du classifieur](https://code.claude.com/docs/en/errors#auto-mode-cannot-determine-the-safety-of-an-action) ou que sa réponse n'a pas pu être analysée. Ce hook ne se déclenche qu'en mode auto : il ne s'exécute pas quand vous refusez manuellement une boîte de dialogue de permission, quand un hook `PreToolUse` bloque un appel ou quand une règle `deny` correspond. Utilisez-le pour journaliser les refus, ajuster la configuration ou indiquer au modèle qu'il peut réessayer l'appel d'outil.

Le matcher porte sur le nom de l'outil, avec les mêmes valeurs que PreToolUse.

#### Entrée de PermissionDenied

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks PermissionDenied reçoivent `tool_name`, `tool_input`, `tool_use_id` et `reason`. Pour un outil MCP, ils reçoivent aussi l'objet [`mcp_server`](#entrée-de-pretooluse).

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "cwd": "/Users/...",
  "permission_mode": "auto",
  "hook_event_name": "PermissionDenied",
  "tool_name": "Bash",
  "tool_input": {
    "command": "rm -rf /tmp/build",
    "description": "Clean build directory"
  },
  "tool_use_id": "toolu_01ABC123...",
  "reason": "[Irreversible Local Destruction]"
}
```

| Champ | Description |
| :- | :- |
| `reason` | La raison du refus. Pour un verdict du classifieur, dans la plupart des sessions, elle nomme la règle correspondante entre crochets, comme `[Data Exfiltration]` ; voir [Examiner les refus](https://code.claude.com/docs/en/auto-mode-config#review-denials) pour les autres formes. Pour un [refus sans verdict](#contrôle-de-décision-de-permissiondenied), elle commence par `Auto mode could not evaluate this action and is blocking it for safety`. Pour un refus dû à l'indisponibilité du modèle classifieur, c'est le texte fixe `Classifier unavailable` |

#### Contrôle de décision de PermissionDenied

Les hooks PermissionDenied peuvent indiquer au modèle qu'il peut réessayer l'appel d'outil refusé. Renvoyez un objet JSON avec `hookSpecificOutput.retry` à `true` :

```json theme={null}
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionDenied",
    "retry": true
  }
}
```

Quand `retry` vaut `true`, Claude Code ajoute à la conversation un message indiquant au modèle qu'il peut réessayer l'appel d'outil. Claude Code n'annule pas le refus lui-même. Si votre hook ne renvoie pas de JSON, ou renvoie `retry: false`, le refus est maintenu et le modèle reçoit le message de rejet d'origine.

Claude Code ignore `retry: true` quand le classifieur n'a produit [aucun verdict sur l'action](https://code.claude.com/docs/en/errors#auto-mode-cannot-determine-the-safety-of-an-action) : sa réponse n'a pas pu être analysée, ou un contrôle de sécurité distinct du mode auto a refusé la requête propre du classifieur. Pour ces refus, Claude Code indique déjà au modèle, dans le message de rejet, s'il doit réessayer plus tard ou passer à autre chose.

### Notification

S'exécute quand Claude Code envoie des notifications. Le matcher porte sur le type de notification. Omettez le matcher pour exécuter les hooks pour tous les types de notification.

Vous recevez ces événements de hook même avec les notifications de bureau désactivées : le réglage `preferredNotifChannel`, y compris `notifications_disabled`, ne change que la façon dont vous êtes alerté, pas le fait que votre hook s'exécute.

| Matcher | Quand il se déclenche |
| :- | :- |
| `permission_prompt` | Claude a besoin que vous approuviez une utilisation d'outil ou la [requête réseau](https://code.claude.com/docs/en/sandboxing#network-isolation) d'une commande en sandbox, et la demande attend depuis environ six secondes |
| `idle_prompt` | Claude a fini de répondre il y a environ 60 secondes et vous n'avez rien tapé depuis |
| `auth_success` | L'authentification se termine |
| `elicitation_dialog` | Un serveur MCP ouvre un formulaire d'élicitation et vous n'avez rien tapé depuis environ six secondes |
| `elicitation_url_dialog` | Un serveur MCP vous demande d'ouvrir une URL dans le navigateur et vous n'avez rien tapé depuis environ six secondes |
| `elicitation_complete` | Un serveur MCP signale qu'une [élicitation en mode URL](https://code.claude.com/docs/en/hooks#elicitation-input) est terminée |
| `elicitation_response` | Une réponse d'élicitation MCP est renvoyée au serveur |
| `agent_needs_input` | Une session d'arrière-plan commence à attendre votre saisie pendant que la [vue des agents](https://code.claude.com/docs/en/agent-view) est ouverte dans un terminal. Se déclenche aussi quand une session de terminal vous montre la [question de configuration du terminal d'un coéquipier d'agent team](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode) ou l'avis du mode auto sur les [frais de requêtes du classifieur](https://code.claude.com/docs/en/auto-mode-classifier-billing) et que vous n'avez rien tapé depuis environ six secondes |
| `agent_completed` | Une session d'arrière-plan se termine ou échoue. Ne se déclenche que pendant que la [vue des agents](https://code.claude.com/docs/en/agent-view) est ouverte dans un terminal |
| `quota_auto_resume_fired` | Claude Code reprend votre tâche après qu'une limite d'utilisation claude.ai l'a mise en pause : à la réinitialisation, ou plus tôt quand une action de votre part dans Claude Code pendant l'attente — ajouter des crédits d'utilisation, passer à un forfait supérieur ou changer de modèle — rend à nouveau de l'utilisation disponible, avec l'[exception liée au réglage du modèle](https://code.claude.com/docs/en/interactive-mode#wait-for-a-usage-limit-to-reset) |
| `quota_auto_resume_stale` | Une limite d'utilisation claude.ai s'est réinitialisée pendant que votre ordinateur était en veille depuis plus d'environ 30 minutes. Claude Code attend que vous appuyiez sur `Enter` au lieu de continuer. Après une veille plus courte, il continue et déclenche plutôt `quota_auto_resume_fired` |
| `quota_auto_resume_disabled` | Claude Code met fin à son attente d'une limite d'utilisation claude.ai sans reprendre votre tâche : [`autoContinueAtUsageLimit`](https://code.claude.com/docs/en/settings-reference#autocontinueatusagelimit) a été désactivé ou la réinitialisation a été repoussée à plus de 24 heures pendant une attente que Claude Code avait lancée de lui-même, la tâche reprise a continué à atteindre la limite, ou la reprise a été bloquée avant d'atteindre le modèle. Ne se déclenche pas quand vous appuyez sur `Esc` ou `Ctrl+C`, ni quand vous choisissez **Don't continue automatically** |

Les types `quota_auto_resume_fired`, `quota_auto_resume_stale` et `quota_auto_resume_disabled` exigent Claude Code v2.1.234+.

Dans les sessions de terminal, `permission_prompt` pour la requête réseau d'une commande en sandbox exige Claude Code v2.1.246+.

`agent_needs_input` pour la question de configuration du terminal d'un coéquipier exige Claude Code v2.1.248+.

> **Note** : Les types `permission_prompt`, `idle_prompt`, `elicitation_dialog` et `elicitation_url_dialog` partagent leur minutage avec les notifications de bureau ; dans les sessions de terminal, vous ne les voyez donc que lorsque vous semblez absent du terminal :
>
> * Attendez-vous à `permission_prompt` après environ six secondes sans frappe. Le minuteur démarre à l'apparition de la demande de permission, et chaque frappe le repousse. Pour exécuter un hook immédiatement quand Claude demande la permission d'utiliser un outil, utilisez plutôt [PermissionRequest](#permissionrequest).
> * Attendez-vous à `idle_prompt` environ 60 secondes après que Claude a fini de répondre, et seulement si vous n'avez rien tapé depuis et qu'aucun agent d'arrière-plan, comme un [sous-agent](https://code.claude.com/docs/en/sub-agents) d'arrière-plan, ne tourne encore. Claude Code n'envoie pas `idle_prompt` pendant qu'il attend la réinitialisation d'une limite d'utilisation claude.ai. Quand l'attente se termine d'elle-même, c'est l'un des types `quota_auto_resume_*` qui se déclenche.
> * Attendez-vous à `elicitation_dialog` pour un formulaire d'élicitation, ou à `elicitation_url_dialog` pour une demande d'URL de navigateur, après environ six secondes sans frappe. Les deux partagent le même seuil de six secondes que `permission_prompt` : le minuteur démarre à l'apparition de la boîte de dialogue, et chaque frappe le repousse.
>
> Une demande de permission ou une élicitation qui arrive pendant qu'une autre boîte de dialogue est affichée conserve le même seuil de six secondes, compté à partir de son arrivée. Sa notification peut donc vous parvenir alors que la demande attend encore derrière la boîte de dialogue ouverte.

Claude Code minute `permission_prompt` différemment dans les sessions où il envoie les demandes de permission au [callback `canUseTool`](https://code.claude.com/docs/en/agent-sdk/user-input) de l'Agent SDK, ce qui est la façon dont Claude Desktop et l'extension VS Code hébergent Claude Code :

* Attendez-vous à `permission_prompt` environ six secondes après que Claude a demandé la permission. Claude Code ne le repousse pas pendant que vous tapez.
* Si vous, ou un hook [PermissionRequest](#permissionrequest), répondez plus tôt, Claude Code n'exécute pas `permission_prompt`.
* Définissez [`CLAUDE_CODE_DISABLE_PERMISSION_PROMPT_NOTIFY_HOOKS`](https://code.claude.com/docs/en/env-vars) à `1` pour désactiver `permission_prompt` dans ces sessions.

Avant la v2.1.233, `permission_prompt` ne se déclenchait pas dans ces sessions.

Utilisez des matchers séparés pour exécuter des gestionnaires différents selon le type de notification. Cette configuration déclenche un script d'alerte propre aux permissions quand Claude a besoin d'une approbation, et une notification différente quand Claude est inactif :

```json theme={null}
{
  "hooks": {
    "Notification": [
      {
        "matcher": "permission_prompt",
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/permission-alert.sh"
          }
        ]
      },
      {
        "matcher": "idle_prompt",
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/idle-notification.sh"
          }
        ]
      }
    ]
  }
}
```

#### Entrée de Notification

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks Notification reçoivent `message` avec le texte de la notification, un `title` facultatif, et `notification_type` indiquant quel type s'est déclenché.

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "cwd": "/Users/...",
  "hook_event_name": "Notification",
  "message": "Claude needs your permission",
  "title": "Permission needed",
  "notification_type": "permission_prompt"
}
```

Les hooks Notification ne peuvent ni bloquer ni modifier les notifications. Claude Code ignore leurs champs `systemMessage` et `continue`, mais émet tout de même [`terminalSequence`](https://code.claude.com/docs/en/hooks#emit-terminal-notifications), sur lequel repose l'exemple de notification de bureau. Les hooks Notification sont destinés aux effets de bord, comme le transfert de la notification vers un service externe.

### SubagentStart

S'exécute quand Claude lance un sous-agent avec l'outil Agent, quand Claude [reprend un sous-agent](https://code.claude.com/docs/en/sub-agents#resume-subagents), et chaque fois qu'un coéquipier d'[agent team](https://code.claude.com/docs/en/agent-teams) en cours de processus traite un nouveau message. Prend en charge les matchers pour filtrer par nom de type d'agent. Pour les agents intégrés, il s'agit du nom de l'agent, comme `general-purpose`, `Explore` ou `Plan`. Pour les [sous-agents personnalisés](https://code.claude.com/docs/en/sub-agents), il s'agit du champ `name` du frontmatter de l'agent, pas du nom de fichier.

Pour les sous-agents fournis par un [plugin](https://code.claude.com/docs/en/plugins/overview), le type d'agent est l'identifiant à portée de plugin, comme `my-plugin:reviewer`, et non le simple nom du frontmatter. Les deux-points font passer un nom à portée de plugin par le chemin des expressions régulières : ancrez donc le matcher avec `^` et `$` pour une correspondance exacte : `^my-plugin:reviewer$`.

#### Entrée de SubagentStart

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks SubagentStart reçoivent `agent_id`, l'identifiant unique du sous-agent, et `agent_type`, le nom d'agent sur lequel filtre le matcher.

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "cwd": "/Users/...",
  "hook_event_name": "SubagentStart",
  "agent_id": "agent-abc123",
  "agent_type": "Explore"
}
```

Les hooks SubagentStart ne peuvent pas bloquer la création du sous-agent, mais ils peuvent injecter du contexte dans le sous-agent. En plus des [champs de sortie JSON](https://code.claude.com/docs/en/hooks#json-output) disponibles pour tous les hooks, vous pouvez renvoyer :

| Champ | Description |
| :- | :- |
| `additionalContext` | Chaîne ajoutée au contexte du sous-agent au début de sa conversation, avant son premier prompt. Voir [Ajouter du contexte pour Claude](https://code.claude.com/docs/en/hooks#add-context-for-claude) |

```json theme={null}
{
  "hookSpecificOutput": {
    "hookEventName": "SubagentStart",
    "additionalContext": "Follow security guidelines for this task"
  }
}
```

Quand le hook s'exécute à nouveau pour le même sous-agent, Claude Code n'injecte le contexte renvoyé que si le contexte du sous-agent ne contient pas déjà la copie issue d'une exécution précédente. La copie injectée au lancement reste en place, ce qui préserve le [cache de prompt](https://code.claude.com/docs/en/prompt-caching#subagents-and-the-cache) du sous-agent. Après qu'une [auto-compaction](https://code.claude.com/docs/en/sub-agents#auto-compaction) a supprimé cette copie, Claude Code injecte à nouveau le contexte de l'exécution suivante.

### SubagentStop

S'exécute quand un sous-agent Claude Code a fini de répondre. Le matcher porte sur le type d'agent, avec les mêmes valeurs que SubagentStart.

#### Entrée de SubagentStop

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks SubagentStop reçoivent `stop_hook_active`, `agent_id`, `agent_type`, `agent_transcript_path` et `last_assistant_message`. Le champ `agent_type` est la valeur utilisée pour le filtrage par matcher. `transcript_path` est la transcription de la session principale, tandis qu'`agent_transcript_path` est la propre transcription du sous-agent, stockée dans un dossier imbriqué `subagents/`. Le champ `last_assistant_message` contient le texte de la réponse finale du sous-agent, pour que les hooks puissent y accéder sans analyser le fichier de transcription.

Tous les événements SubagentStop ne proviennent pas d'un sous-agent lancé par Claude. Claude Code exécute aussi des agents internes pour certaines de ses propres fonctionnalités, comme les [suggestions de prompt](https://code.claude.com/docs/en/interactive-mode#prompt-suggestions) et les [questions annexes `/btw`](https://code.claude.com/docs/en/interactive-mode#side-questions-with-%2Fbtw), et SubagentStop se déclenche aussi quand l'un d'eux se termine. Pour ces événements, `agent_type` est le nom d'agent sous lequel la session elle-même s'exécute, par exemple défini avec [`--agent`](https://code.claude.com/docs/en/cli-reference#cli-flags) ou le [réglage `agent`](https://code.claude.com/docs/en/settings-reference#agent), et une chaîne vide quand la session s'exécute sans agent.

Un `matcher` qui nomme des types d'agents ne correspond pas à un `agent_type` vide. Un hook dont le matcher est omis, `""` ou `"*"`, ou est une expression régulière qui correspond à une chaîne vide, s'exécute aussi pour les événements à `agent_type` vide.

À partir de Claude Code v2.1.271, un sous-agent qui s'exécute avec l'outil [`SubagentHandback`](https://code.claude.com/docs/en/tools-reference) remet son rapport via cet outil avant de s'arrêter. Le champ `last_assistant_message` contient alors le texte de clôture du sous-agent, s'il y en a un, qui n'est pas le rapport remis. Le rapport est l'entrée `message` de cet appel, qu'un hook `PreToolUse` ou `PostToolUse` correspondant à `SubagentHandback` reçoit sous la forme `tool_input.message`.

Les hooks SubagentStop reçoivent aussi les tableaux `background_tasks` et `session_crons` décrits dans [Entrée de Stop](#entrée-de-stop). Les deux tableaux portent sur la session parente, pas sur le sous-agent.

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "~/.claude/projects/.../abc123.jsonl",
  "cwd": "/Users/...",
  "permission_mode": "default",
  "hook_event_name": "SubagentStop",
  "stop_hook_active": false,
  "agent_id": "def456",
  "agent_type": "Explore",
  "agent_transcript_path": "~/.claude/projects/.../abc123/subagents/agent-def456.jsonl",
  "last_assistant_message": "Analysis complete. Found 3 potential issues...",
  "background_tasks": [],
  "session_crons": []
}
```

Les hooks SubagentStop utilisent le même format de contrôle de décision que les [hooks Stop](#contrôle-de-décision-de-stop), y compris `hookSpecificOutput.additionalContext` avec `hookEventName` à `"SubagentStop"`, pour un retour hors erreur qui maintient le sous-agent en activité. Renvoyer `decision: "block"` avec une `reason` maintient le sous-agent en activité et lui transmet `reason` comme prochaine instruction. Un hook qui bloque en sortant avec le code 2 transmet son message stderr de la même façon. Pour injecter du contexte dans la session parente après le retour d'un sous-agent, utilisez plutôt un hook [`PostToolUse`](#posttooluse) sur l'outil `Agent`.

### TaskCreated

S'exécute quand une tâche est en cours de création via l'outil `TaskCreate`. Utilisez-le pour imposer des conventions de nommage, exiger des descriptions de tâches ou empêcher la création de certaines tâches. Dans une [session sans les outils Task](https://code.claude.com/docs/en/tools-reference#task-tool-availability), cet événement ne se déclenche pas.

Les hooks TaskCreated ne prennent pas en charge les matchers et se déclenchent à chaque occurrence.

#### Entrée de TaskCreated

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks TaskCreated reçoivent `task_id`, `task_subject` et, éventuellement, `task_description`, `teammate_name` et `team_name`.

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "cwd": "/Users/...",
  "hook_event_name": "TaskCreated",
  "task_id": "task-001",
  "task_subject": "Implement user authentication",
  "task_description": "Add login and signup endpoints",
  "teammate_name": "implementer",
  "team_name": "session-a1b2c3d4"
}
```

| Champ | Description |
| :- | :- |
| `task_id` | Identifiant de la tâche en cours de création |
| `task_subject` | Titre de la tâche |
| `task_description` | Description détaillée de la tâche. Peut être absent |
| `teammate_name` | Nom du coéquipier qui crée la tâche. Peut être absent |
| `team_name` | Obsolète. Nom d'équipe dérivé de la session ; sera supprimé dans une version future |
| `agent_id` | Sur cet événement, le [champ d'entrée commun](https://code.claude.com/docs/en/hooks#common-input-fields) identifie le sous-agent ou le [coéquipier en cours de processus](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode) qui crée la tâche. Peut être absent. Exige Claude Code v2.1.290+ |

#### Contrôle de décision de TaskCreated

Un hook TaskCreated peut bloquer la création avec le code de sortie 2 ou avec une décision JSON. Dans les deux cas, Claude Code supprime la tâche et renvoie votre message à Claude comme erreur de l'outil. Claude Code ignore `continue: false` pour cet événement, et Claude continue de travailler.

* **Code de sortie 2** : Claude Code renvoie le texte stderr comme message.
* **JSON `{"decision": "block", "reason": "..."}`** : Claude Code renvoie `reason` comme message.

Cet exemple bloque les tâches dont le sujet ne respecte pas le format requis :

```bash theme={null}
#!/bin/bash
INPUT=$(cat)
TASK_SUBJECT=$(echo "$INPUT" | jq -r '.task_subject')

if [[ ! "$TASK_SUBJECT" =~ ^\[TICKET-[0-9]+\] ]]; then
  echo "Task subject must start with a ticket number, e.g. '[TICKET-123] Add feature'" >&2
  exit 2
fi

exit 0
```

### TaskCompleted

S'exécute quand une tâche est en cours de marquage comme terminée. Il se déclenche dans deux situations : quand un agent quelconque marque explicitement une tâche comme terminée via l'outil TaskUpdate, ou quand un coéquipier d'[agent team](https://code.claude.com/docs/en/agent-teams) termine son tour avec des tâches en cours. Utilisez-le pour imposer des critères d'achèvement, comme des tests ou des vérifications de lint réussis, avant qu'une tâche puisse être close.

Les hooks TaskCompleted ne prennent pas en charge les matchers et se déclenchent à chaque occurrence.

#### Entrée de TaskCompleted

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks TaskCompleted reçoivent `task_id`, `task_subject` et, éventuellement, `task_description`, `teammate_name` et `team_name`.

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "cwd": "/Users/...",
  "permission_mode": "default",
  "hook_event_name": "TaskCompleted",
  "task_id": "task-001",
  "task_subject": "Implement user authentication",
  "task_description": "Add login and signup endpoints",
  "teammate_name": "implementer",
  "team_name": "session-a1b2c3d4"
}
```

| Champ | Description |
| :- | :- |
| `task_id` | Identifiant de la tâche en cours d'achèvement |
| `task_subject` | Titre de la tâche |
| `task_description` | Description détaillée de la tâche. Peut être absent |
| `teammate_name` | Nom du coéquipier qui termine la tâche. Peut être absent |
| `team_name` | Obsolète. Nom d'équipe dérivé de la session ; sera supprimé dans une version future |
| `agent_id` | Sur cet événement, le [champ d'entrée commun](https://code.claude.com/docs/en/hooks#common-input-fields) identifie le sous-agent ou le [coéquipier en cours de processus](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode) qui termine la tâche. Peut être absent. Exige Claude Code v2.1.290+ |

#### Contrôle de décision de TaskCompleted

Les hooks TaskCompleted offrent deux façons de contrôler l'achèvement d'une tâche :

* **Code de sortie 2** : la tâche n'est pas marquée comme terminée et le message stderr est renvoyé au modèle comme retour.
* **JSON `{"continue": false, "stopReason": "..."}`** : quand l'événement a été déclenché par un coéquipier qui termine son tour, arrête complètement le coéquipier, comme le comportement d'un hook `Stop`. Le `stopReason` est affiché à l'utilisateur. Quand l'événement a été déclenché par l'outil `TaskUpdate`, Claude Code ignore `continue: false` ; le code de sortie 2 bloque toujours l'achèvement.

Cet exemple exécute les tests et bloque l'achèvement de la tâche s'ils échouent :

```bash theme={null}
#!/bin/bash
INPUT=$(cat)
TASK_SUBJECT=$(echo "$INPUT" | jq -r '.task_subject')

# Exécuter la suite de tests
if ! npm test 2>&1; then
  echo "Tests not passing. Fix failing tests before completing: $TASK_SUBJECT" >&2
  exit 2
fi

exit 0
```

### Stop

S'exécute quand l'agent principal de Claude Code a fini de répondre. Ne s'exécute pas si l'arrêt est dû à une interruption par l'utilisateur. Les erreurs d'API déclenchent plutôt [StopFailure](#stopfailure).

> **Astuce** : La commande [`/goal`](https://code.claude.com/docs/en/goal) est un raccourci intégré pour un hook Stop basé sur un prompt, limité à la session. Utilisez-la quand vous voulez que Claude continue à travailler vers une condition sans écrire de configuration de hook.

#### Entrée de Stop

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks Stop reçoivent `stop_hook_active`, `last_assistant_message`, `background_tasks` et `session_crons`. Le champ `stop_hook_active` vaut `true` quand Claude Code continue déjà à la suite d'un hook Stop. Vérifiez cette valeur ou analysez la transcription pour éviter de bloquer sur une condition qui ne se résoudra jamais.

Claude Code applique un plafond de 8 continuations consécutives : après que des hooks Stop ont prolongé le tour huit fois de suite, Claude Code passe outre le blocage suivant et termine le tour. Le compteur de continuations consécutives se réinitialise chaque fois que Claude appelle un outil. Pour relever ce plafond, définissez [`CLAUDE_CODE_STOP_HOOK_BLOCK_CAP`](https://code.claude.com/docs/en/env-vars).

Le champ `last_assistant_message` contient le texte de la réponse finale de Claude, pour que les hooks puissent y accéder sans analyser le fichier de transcription. Pour les hooks qui agissent sur le tour qui vient de se terminer, comme les hooks de lecture à voix haute ou de notification, utilisez ce champ plutôt que de lire `transcript_path` : il n'est pas garanti, sur toutes les versions, que le fichier de transcription contienne le message final au moment de Stop.

Les tableaux `background_tasks` et `session_crons` permettent aux hooks de distinguer « la session est terminée » de « la session est en pause, en attente qu'un travail d'arrière-plan la réveille ». Les deux tableaux sont présents quand le registre des tâches est accessible, et vides quand rien n'est en cours ni planifié.

Chaque entrée de `background_tasks` décrit une tâche en cours et utilise ces champs :

| Champ | Description |
| :- | :- |
| `id` | Identifiant de la tâche |
| `type` | Libellé lisible du type de tâche, comme `shell`, `subagent`, `monitor`, `workflow`, `teammate`, `cloud session` ou `MCP task`. Chaque libellé identifie la fonctionnalité de Claude Code qui a créé la tâche. Se rabat sur le discriminant brut pour les types non reconnus |
| `status` | État actuel de la tâche |
| `description` | Description en texte libre, plafonnée à 1000 caractères avec un marqueur `… [+N chars]` dans la chaîne en cas de troncature |
| `command` | Ligne de commande shell, plafonnée à 1000 caractères. Présent uniquement pour les tâches `shell` |
| `agent_type` | Nom du type de sous-agent. Présent uniquement pour les tâches `subagent` |
| `server` | Nom du serveur MCP. Présent uniquement pour les tâches `monitor` et `MCP task` |
| `tool` | Nom de l'outil MCP. Présent uniquement pour les tâches `monitor` et `MCP task` |
| `name` | Nom du workflow. Présent uniquement pour les tâches `workflow` |

Chaque entrée de `session_crons` décrit un réveil planifié limité à la session, issu de `CronCreate`, `ScheduleWakeup` et `/loop` :

| Champ | Description |
| :- | :- |
| `id` | Identifiant de la tâche cron |
| `schedule` | Expression cron, par exemple `0 9 * * 1-5` |
| `recurring` | `false` pour les réveils ponctuels dont la planification encode une heure de déclenchement unique, `true` pour les tâches qui se redéclenchent à chaque correspondance |
| `prompt` | Prompt soumis au déclenchement du cron, plafonné à 1000 caractères avec le même marqueur `… [+N chars]` |

Cet exemple montre une entrée Stop avec une tâche shell en cours et un cron récurrent :

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "~/.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "cwd": "/Users/...",
  "permission_mode": "default",
  "hook_event_name": "Stop",
  "stop_hook_active": true,
  "last_assistant_message": "I've completed the refactoring. Here's a summary...",
  "background_tasks": [
    {
      "id": "task-001",
      "type": "shell",
      "status": "running",
      "description": "tail logs",
      "command": "tail -f /var/log/syslog"
    }
  ],
  "session_crons": [
    {
      "id": "cron-001",
      "schedule": "0 9 * * 1-5",
      "recurring": true,
      "prompt": "check the build"
    }
  ]
}
```

#### Contrôle de décision de Stop

Les hooks `Stop` et `SubagentStop` peuvent contrôler si Claude continue. En plus des [champs de sortie JSON](https://code.claude.com/docs/en/hooks#json-output) disponibles pour tous les hooks, votre script de hook peut renvoyer ces champs propres à l'événement :

| Champ | Description |
| :- | :- |
| `decision` | `"block"` empêche Claude de s'arrêter. Omettez-le pour laisser Claude s'arrêter |
| `reason` | Obligatoire quand `decision` vaut `"block"`. Indique à Claude pourquoi il doit continuer |
| `hookSpecificOutput.additionalContext` | Retour hors erreur pour Claude. La conversation continue pour que Claude puisse en tenir compte mais, contrairement à `decision: "block"`, il apparaît dans la transcription comme un retour de hook plutôt que comme une erreur de hook |

Un hook qui bloque en sortant avec le code 2 est traité comme `reason` : Claude reçoit le message stderr comme explication de la raison pour laquelle il doit continuer.

```json theme={null}
{
  "decision": "block",
  "reason": "Must be provided when Claude is blocked from stopping"
}
```

Utilisez `additionalContext` quand le hook fonctionne comme prévu et donne des consignes à Claude, comme « exécute la suite de tests avant de terminer ». Il prolonge la conversation avec les mêmes protections contre les boucles que `decision: "block"`, à savoir l'entrée `stop_hook_active` et le plafond de 8 continuations consécutives, mais la transcription l'étiquette `Stop hook feedback` et aucune notification d'erreur de hook n'est affichée :

```json theme={null}
{
  "hookSpecificOutput": {
    "hookEventName": "Stop",
    "additionalContext": "Please run the test suite before finishing"
  }
}
```

### StopFailure

S'exécute à la place de [Stop](#stop) quand le tour se termine à cause d'une erreur d'API. Claude Code ignore la sortie et le code de sortie du hook, à l'exception de [`terminalSequence`](https://code.claude.com/docs/en/hooks#emit-terminal-notifications). Utilisez-le pour journaliser les échecs, envoyer des alertes ou engager des actions de récupération quand Claude ne peut pas terminer une réponse à cause de limites de débit, de problèmes d'authentification ou d'autres erreurs d'API.

#### Entrée de StopFailure

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks StopFailure reçoivent `error`, un `error_details` facultatif et un `last_assistant_message` facultatif. Le champ `error` identifie le type d'erreur et sert au filtrage par matcher.

| Champ | Description |
| :- | :- |
| `error` | Type d'erreur : `rate_limit`, `overloaded`, `authentication_failed`, `oauth_org_not_allowed`, `account_on_hold`, `billing_error`, `invalid_request`, `model_not_found`, `server_error`, `max_output_tokens`, `cloud_credential_error` ou `unknown` |
| `error_details` | Détails supplémentaires sur l'erreur, quand ils sont disponibles |
| `last_assistant_message` | Le texte d'erreur rendu affiché dans la conversation. Contrairement à `Stop` et `SubagentStop`, où ce champ contient la sortie conversationnelle de Claude, pour `StopFailure` il contient la chaîne d'erreur d'API elle-même, comme `"API Error: Rate limit reached"` |

```json theme={null}
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "cwd": "/Users/...",
  "hook_event_name": "StopFailure",
  "error": "rate_limit",
  "error_details": "429 Too Many Requests",
  "last_assistant_message": "API Error: Rate limit reached"
}
```

Les hooks StopFailure n'ont pas de contrôle de décision. Ils ne servent qu'à des fins de notification et de journalisation.

