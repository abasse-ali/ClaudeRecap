# Référence des hooks — partie 2/3

> Source : https://code.claude.com/docs/en/hooks · [← Index](../31-index-documentation-officielle.md)

> Cette partie couvre les événements de hooks, de `SessionStart` jusqu'aux événements de fin de session. Les liens « (#ancre) » renvoient aux sections de la même page officielle (parties 1 et 3 comprises).

## Événements de hooks (Hook events)

Chaque événement correspond à un point du cycle de vie de Claude Code où des hooks peuvent s'exécuter. Les sections ci-dessous suivent l'ordre du cycle de vie : de l'initialisation de la session, en passant par la boucle agentique, jusqu'à la fin de la session. Chaque section décrit quand l'événement se déclenche, quels matchers il prend en charge, l'entrée JSON qu'il reçoit, et comment contrôler le comportement via la sortie.

### SessionStart

S'exécute quand Claude Code démarre une nouvelle session ou reprend une session existante. Utile pour charger du contexte de développement (issues existantes, changements récents dans la base de code) ou pour définir des variables d'environnement. Pour du contexte statique qui ne nécessite pas de script, utilisez plutôt [CLAUDE.md](https://code.claude.com/docs/en/memory).

SessionStart s'exécute à chaque session : gardez ces hooks rapides. Seuls les hooks `type: "command"` et `type: "mcp_tool"` sont pris en charge. Voir [Champs des hooks d'outil MCP](https://code.claude.com/docs/en/hooks#mcp-tool-hook-fields) pour savoir quand les hooks `mcp_tool` s'exécutent.

La valeur du matcher correspond à la façon dont la session a été lancée :

| Matcher | Quand il se déclenche |
| :- | :- |
| `startup` | Nouvelle session |
| `resume` | `--resume`, `--continue` ou `/resume` |
| `clear` | `/clear` |
| `compact` | Compaction automatique ou manuelle |
| `fork` | Nouvelle session dérivée (fork) d'une session existante : `--fork-session` avec `--resume` ou `--continue`, la copie en arrière-plan de `/fork`, `/branch`, ou une conversation que vous [passez en arrière-plan](https://code.claude.com/docs/en/agent-view#from-inside-a-session) |

Avant la v2.1.214, les sessions forkées signalaient la source `"resume"`.

Quand vous démarrez une session interactive, reprenez une conversation au lancement avec `--continue` ou `--resume`, ou exécutez `/clear`, les hooks SessionStart s'exécutent en arrière-plan. Vous pouvez taper immédiatement, et une conversation reprise s'affiche sans attendre les hooks. La première réponse de Claude attend toutefois la fin des hooks, pour que leur contexte parvienne à Claude.

Quand vous changez de conversation avec `/resume` à l'intérieur d'une session, le changement attend au contraire la fin des hooks. Si vous exécutez `/clear` ou basculez vers une autre conversation pendant que des hooks en arrière-plan tournent encore, rien de ce qu'ils renvoient ne s'applique à la session.

La même attente s'applique au lancement, y compris pour une session reprise : un prompt envoyé pendant que les hooks SessionStart tournent encore n'atteint Claude qu'à leur fin.

Pendant l'une ou l'autre attente, appuyez sur `Esc` pour ramener le prompt dans la zone de saisie sans l'envoyer. Les hooks continuent de s'exécuter.

#### Entrée de SessionStart

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks SessionStart reçoivent `source` et, en option, `model`, `agent_type` et `session_title` :

| Champ | Description |
| :- | :- |
| `source` | Mode de démarrage de la session : `"startup"` pour une nouvelle session, `"resume"` pour une session reprise, `"clear"` après `/clear`, `"compact"` après une compaction, ou `"fork"` pour une nouvelle session forkée d'une existante |
| `model` | Identifiant du modèle actif. Peut être absent, par exemple après `/clear` ou quand une session est restaurée via la récupération de conversation : vérifiez la présence du champ avant de le lire |
| `agent_type` | Nom de l'agent, présent quand vous lancez Claude Code avec `claude --agent <name>` |
| `session_title` | Titre personnalisé de la session, présent s'il est défini, par exemple avec `--name`, `/rename`, la sortie `sessionTitle` d'un hook, ou `renameSession()` de l'Agent SDK. Un hook qui émet `sessionTitle` peut vérifier ce champ d'abord pour éviter d'écraser un titre personnalisé existant |

Une session que vous n'avez pas nommée peut quand même avoir un [titre généré](https://code.claude.com/docs/en/sessions#name-your-sessions). Ce titre n'est pas un titre personnalisé et n'apparaît pas dans `session_title`.

Quand `source` vaut `"resume"` ou `"fork"` et que le transcript contient au moins une réponse de Claude, les hooks SessionStart reçoivent aussi les quatre champs ci-dessous. Votre hook peut s'en servir pour signaler le coût de la reprise d'une conversation ancienne avant la première requête, par exemple dans un [`systemMessage`](https://code.claude.com/docs/en/hooks#json-output). Ces champs exigent Claude Code v2.1.251+.

| Champ | Description |
| :- | :- |
| `seconds_since_last_response` | Secondes (temps réel) écoulées depuis la dernière réponse du transcript repris |
| `context_tokens` | Tokens que la première requête de la session reprise renvoie comme prompt |
| `prompt_cache_likely_expired` | `true` quand la dernière réponse est plus ancienne que la [durée de vie du cache de prompt](https://code.claude.com/docs/en/prompt-caching#cache-lifetime) de la session, ou qu'une compaction ultérieure a remplacé la conversation en cache |
| `estimated_cache_write_usd` | Coût estimé en dollars US de l'écriture de `context_tokens` dans le cache de prompt sur le modèle de la session, hors réponse |

Exemple d'entrée pour une session reprise 90 minutes après sa dernière réponse :

```json
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

Un hook SessionStart peut ajouter du contexte pour Claude, fournir le premier message utilisateur, définir le titre de la session, surveiller des fichiers et recharger les skills. Renvoyez le champ correspondant à chaque action, en plus des [champs de sortie JSON](https://code.claude.com/docs/en/hooks#json-output) disponibles pour tous les hooks :

| Champ | Description |
| :- | :- |
| `additionalContext` | Chaîne ajoutée au contexte de Claude au début de la conversation, avant le premier prompt. Voir [Ajouter du contexte pour Claude](https://code.claude.com/docs/en/hooks#add-context-for-claude) pour la façon dont le texte est transmis et ce qu'il faut y mettre |
| `initialUserMessage` | Chaîne utilisée comme premier message utilisateur de la session, en [mode non interactif](https://code.claude.com/docs/en/headless) avec le flag `-p`. Elle devient le premier tour même si vous ne passez aucun prompt. Un prompt que vous passez suit comme tour suivant |
| `sessionTitle` | Définit le titre de la session, avec le même effet que `/rename`. S'applique quand `source` vaut `"startup"`, `"resume"` ou `"fork"` |
| `watchPaths` | Tableau de chemins absolus à surveiller pour les événements [FileChanged](https://code.claude.com/docs/en/hooks#filechanged) pendant cette session |
| `reloadSkills` | Booléen. À `true`, Claude Code rescanne les répertoires de [skills](https://code.claude.com/docs/en/skills) et de commandes une fois les hooks SessionStart terminés. Voir [Recharger les skills qu'un hook installe](#recharger-les-skills-quun-hook-installe) |

Cette sortie ajoute du contexte et nomme la session :

```json
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "Current branch: feat/auth-refactor\nUncommitted changes: src/auth.ts, src/login.tsx\nActive issue: #4211 Migrate to OAuth2",
    "sessionTitle": "auth-refactor"
  }
}
```

Un hook qui se contente d'ajouter du contexte peut l'imprimer sans construire de JSON, car Claude Code ajoute la [sortie stdout en texte brut](https://code.claude.com/docs/en/hooks#exit-code-0) d'un hook SessionStart au contexte de Claude.

Si le hook SessionStart de votre plugin fournit `initialUserMessage` ou `sessionTitle`, installez le plugin avant le démarrage de la session. Claude Code ignore ces deux champs provenant d'un plugin dont l'installation se termine après l'exécution des hooks SessionStart.

#### Recharger les skills qu'un hook installe

Pour rendre disponibles dans la même session les skills qu'un hook SessionStart installe, renvoyez `reloadSkills`. La découverte des skills s'exécute normalement avant la fin des hooks SessionStart ; sans ce champ, les fichiers qu'un hook écrit dans `~/.claude/skills/` ou `.claude/skills/` peuvent manquer lors du premier prompt.

Cet exemple synchronise un dépôt de skills partagé et demande le rescan :

```bash
#!/bin/bash

git -C ~/.claude/skills/team-skills pull --quiet 2>/dev/null || \
  git clone --quiet https://git.example.com/your-org/team-skills.git ~/.claude/skills/team-skills

echo '{"hookSpecificOutput": {"hookEventName": "SessionStart", "reloadSkills": true}}'
```

L'URL du dépôt est un exemple fictif : remplacez-la par votre propre dépôt de skills.

#### Persister des variables d'environnement

Les hooks SessionStart ont accès à la variable d'environnement `CLAUDE_ENV_FILE`, qui fournit un chemin de fichier où persister des variables d'environnement pour les commandes shell que Claude exécutera plus tard dans la session.

Pour définir des variables individuelles, écrivez des instructions `export` dans `CLAUDE_ENV_FILE`. Utilisez l'ajout (`>>`) pour préserver les variables définies par d'autres hooks :

```bash
#!/bin/bash

if [ -n "$CLAUDE_ENV_FILE" ]; then
  echo 'export NODE_ENV=production' >> "$CLAUDE_ENV_FILE"
  echo 'export DEBUG_LOG=true' >> "$CLAUDE_ENV_FILE"
  echo 'export PATH="$PATH:./node_modules/.bin"' >> "$CLAUDE_ENV_FILE"
fi

exit 0
```

Pour capturer tous les changements d'environnement effectués par des commandes de configuration, comparez les variables exportées avant et après :

```bash
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

Les commandes [PowerShell](https://code.claude.com/docs/en/tools-reference#powershell-tool) reçoivent elles aussi les variables de `CLAUDE_ENV_FILE` (v2.1.296+), mais PowerShell n'exécute jamais le fichier : Claude Code en extrait les affectations et les copie dans l'environnement de la commande PowerShell. Il ne le fait que si chaque ligne relève de l'un des cas suivants, sur l'ensemble de ce que chaque hook de la session a écrit et de tout script vers lequel vous avez [fait pointer `CLAUDE_ENV_FILE`](https://code.claude.com/docs/en/env-vars) avant le lancement :

* Une ligne vide ou un commentaire `#`
* Une seule affectation en début de ligne, écrite `export NAME=value`, `declare -x NAME=value` ou `NAME=value`, avec une valeur que Bash utiliserait exactement telle qu'écrite, composée de n'importe quel mélange de : texte non quoté n'utilisant que des lettres, chiffres et les caractères `_ @ % + = : , . / -` ; texte entre guillemets simples ; texte entre guillemets doubles où tout `$`, backtick ou `"` intérieur est échappé par une barre oblique inverse

Si une ligne est d'un autre type — par exemple `export PATH="$PATH:./node_modules/.bin"` avec son `$PATH` non échappé, une commande `source`, ou les chaînes `$'...'` qu'imprime `direnv export bash` —, les commandes PowerShell ne reçoivent aucune des variables, et `claude --debug` journalise `Session environment is not all plain assignments`. Les commandes Bash les reçoivent quand même toutes. Sous Windows, les commandes PowerShell ne reçoivent pas non plus une variable dont la valeur contient `/` ou `\`, car Git Bash et Windows écrivent les chemins différemment. Une commande PowerShell [en sandbox](https://code.claude.com/docs/en/sandboxing) ne reçoit aucune des variables.

> **Note** : `CLAUDE_ENV_FILE` est disponible pour les hooks SessionStart, [Setup](#setup), [CwdChanged](#cwdchanged) et [FileChanged](#filechanged). Les autres événements n'ont pas accès à cette variable, pas plus qu'un hook exécuté dans PowerShell, que ce soit via [`"shell": "powershell"`](https://code.claude.com/docs/en/hooks#command-hook-fields) ou par défaut sous Windows sans Git Bash.

### Setup

Se déclenche uniquement quand vous lancez Claude Code avec `--init-only`, ou avec `--init` ou `--maintenance` en [mode non interactif](https://code.claude.com/docs/en/headless) avec le flag `-p`. Il ne se déclenche pas au démarrage normal. Utilisez-le pour une installation ponctuelle de dépendances ou un nettoyage planifié que vous déclenchez explicitement depuis la CI ou des scripts, séparément du démarrage normal des sessions. Pour l'initialisation par session, utilisez plutôt [SessionStart](#sessionstart).

La valeur du matcher correspond au flag CLI qui a déclenché le hook :

| Matcher | Quand il se déclenche |
| :- | :- |
| `init` | `claude --init-only` ou `claude -p --init` |
| `maintenance` | `claude -p --maintenance` |

Quand vous exécutez `claude --init-only`, Claude Code exécute les hooks Setup puis les hooks `SessionStart` avec le matcher `startup`, puis quitte sans démarrer de conversation.

Quand vous démarrez ou continuez une conversation avec `-p`, vous devez aussi fournir un prompt, en argument ou via stdin. Vous pouvez omettre le prompt quand un hook `SessionStart` fournit [`initialUserMessage`](#contrôle-de-décision-de-sessionstart) ou quand vous reprenez une session avec un [appel d'outil différé](#différer-un-appel-doutil).

En cas de succès, `--init-only` n'imprime rien dans le terminal. Pour confirmer que les hooks ont tourné, lancez `claude --debug-file <path> --init-only` en remplaçant `<path>` par l'emplacement d'un fichier journal, puis cherchez dans le journal les entrées des hooks Setup et SessionStart.

Comme Setup ne se déclenche pas à chaque lancement, un plugin qui a besoin d'une dépendance installée ne peut pas compter sur Setup seul. Le modèle pratique consiste à vérifier la dépendance à la première utilisation et à l'installer si elle manque, par exemple un hook ou une skill qui teste la présence de `${CLAUDE_PLUGIN_DATA}/node_modules` et lance `npm install` en son absence. Voir le [répertoire de données persistantes](https://code.claude.com/docs/en/plugins/components#path-variables-and-persistent-data) pour l'emplacement où stocker les dépendances installées. Si vous distribuez votre plugin via une marketplace, ce modèle peut être inutile : Claude Code [installe automatiquement les dépendances de paquets Node.js éligibles](https://code.claude.com/docs/en/plugins/loading#node-js-package-dependencies) lorsqu'il met le plugin en cache.

#### Entrée de Setup

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks Setup reçoivent un champ `trigger` valant `"init"` ou `"maintenance"` :

```json
{
  "session_id": "abc123",
  "transcript_path": "/Users/.../.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "cwd": "/Users/...",
  "hook_event_name": "Setup",
  "trigger": "init"
}
```

#### Contrôle de décision de Setup

Les hooks Setup ne peuvent pas bloquer ; l'exécution continue quel que soit le code de sortie. Quel que soit le code de sortie, Claude Code ignore les [champs de sortie JSON](https://code.claude.com/docs/en/hooks#json-output) d'un hook Setup, comme `systemMessage`, `continue` et `hookSpecificOutput.additionalContext`. Avec `-p`, le stdout, le stderr et le code de sortie d'un hook Setup n'apparaissent dans la sortie de l'exécution que sous forme d'[événements `hook_response`](https://code.claude.com/docs/en/headless#read-session-metadata) quand vous lancez avec `--output-format stream-json --verbose`.

Les hooks Setup ont accès à `CLAUDE_ENV_FILE`. Les variables écrites dans ce fichier persistent dans les commandes Bash suivantes de la session, comme pour les [hooks SessionStart](#persister-des-variables-denvironnement). Seuls les hooks `type: "command"` s'exécutent sur `Setup`. Un hook `type: "mcp_tool"` sur `Setup` est toujours ignoré, comme décrit dans [Champs des hooks d'outil MCP](https://code.claude.com/docs/en/hooks#mcp-tool-hook-fields).

### InstructionsLoaded

Se déclenche quand un fichier `CLAUDE.md` ou `.claude/rules/*.md` est chargé dans le contexte. Cet événement se déclenche au démarrage de la session pour les fichiers chargés d'emblée, puis plus tard lors des chargements paresseux, par exemple quand Claude accède à un sous-répertoire contenant un `CLAUDE.md` imbriqué ou quand des règles conditionnelles avec un frontmatter `paths:` correspondent. Le hook ne prend en charge ni le blocage ni le contrôle de décision. Il s'exécute de façon asynchrone, à des fins d'observabilité.

Cet événement ne se déclenche pas quand Claude [lit `AGENTS.md` directement](https://code.claude.com/docs/en/memory#agents-md) via le réglage **Project instructions**. Il se déclenche en revanche quand un `CLAUDE.md` importe votre `AGENTS.md` (avec `load_reason` à `include`, comme pour tout fichier importé), et quand `CLAUDE.md` est un lien symbolique vers lui (comme un chargement normal de `CLAUDE.md`).

Le matcher s'applique à `load_reason`. Par exemple, utilisez `"matcher": "session_start"` pour ne réagir qu'aux fichiers chargés au démarrage, ou `"matcher": "path_glob_match|nested_traversal"` pour ne réagir qu'aux chargements paresseux.

#### Entrée d'InstructionsLoaded

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks InstructionsLoaded reçoivent ces champs :

| Champ | Description |
| :- | :- |
| `file_path` | Chemin absolu du fichier d'instructions chargé |
| `memory_type` | Portée du fichier : `"User"`, `"Project"`, `"Local"` ou `"Managed"` |
| `load_reason` | Raison du chargement : `"session_start"`, `"nested_traversal"`, `"path_glob_match"`, `"include"` ou `"compact"`. La valeur `"compact"` apparaît quand des fichiers d'instructions sont rechargés après une compaction |
| `globs` | Motifs glob de chemins issus du frontmatter `paths:` du fichier, le cas échéant. Présent uniquement pour les chargements `path_glob_match` |
| `trigger_file_path` | Chemin du fichier dont l'accès a déclenché ce chargement, pour les chargements paresseux |
| `parent_file_path` | Chemin du fichier d'instructions parent qui a inclus celui-ci, pour les chargements `include` |

```json
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

S'exécute quand un prompt est soumis, avant que Claude ne le traite. Permet d'ajouter du contexte en fonction du prompt ou de la conversation, de valider des prompts, ou de bloquer certains types de prompts.

Les hooks `UserPromptSubmit` ne se déclenchent pas seulement sur les prompts que vous tapez. Claude Code les exécute aussi sur :

* le déclenchement d'une [tâche planifiée](https://code.claude.com/docs/en/scheduled-tasks), y compris une itération de `/loop`
* un [sous-agent en arrière-plan](https://code.claude.com/docs/en/sub-agents#run-subagents-in-foreground-or-background) qui rend compte à la session qui l'a lancé
* un [message envoyé par une autre session](https://code.claude.com/docs/en/cross-session-messaging) à votre conversation principale

Les hooks `UserPromptSubmit` ont un timeout par défaut de 30 secondes pour les types `command`, `http` et `mcp_tool`, plus court que le défaut de 600 secondes de ces types sur la plupart des autres événements. Comme ce hook s'exécute avant chaque prompt et bloque le traitement par le modèle jusqu'à sa fin, un hook bloqué fige la session. Si votre hook a besoin de plus de temps, définissez le champ `timeout` dans l'entrée du hook.

Hormis un hook de commande lancé avec [`async: true`](https://code.claude.com/docs/en/hooks#run-hooks-in-the-background), un hook `UserPromptSubmit` de type commande, HTTP ou outil MCP qui atteint son timeout est annulé et sa sortie, y compris tout `additionalContext`, est ignorée. Le prompt parvient quand même à Claude, sans ce contexte. Pour bloquer le prompt à la place, définissez [`onFailure: "block"`](https://code.claude.com/docs/en/hooks#block-the-action-when-a-hook-fails) sur un hook de commande ou HTTP. Le transcript affiche un avis nommant le hook, le timeout déclenché, et indiquant que la sortie a été ignorée.

Un [hook callback de l'Agent SDK](https://code.claude.com/docs/en/agent-sdk/hooks) sur `UserPromptSubmit` qui atteint son timeout bloque le prompt avec un message nommant le hook et le timeout, car un callback à cet endroit peut jouer le rôle de garde-fou de politique qui ne doit pas échouer en mode ouvert (« fail open »). La session continue. Avant la v2.1.208, un timeout de callback sur cet événement terminait le tour avec une erreur d'exécution.

#### Entrée d'UserPromptSubmit

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks UserPromptSubmit reçoivent le champ `prompt` contenant le texte soumis. Le contenu collé replié en un espace réservé `[Pasted text #N]` arrive développé à sa place. Dans les sessions où Claude Code [balise le texte collé pour Claude](https://code.claude.com/docs/en/terminal-config#how-claude-treats-pasted-text), ce contenu développé se trouve entre une ligne `<pasted_content id="…">` et une ligne `</pasted_content id="…">` : tenez compte de ces lignes si votre hook analyse le prompt.

Les hooks UserPromptSubmit reçoivent aussi `session_title` quand la session a un titre personnalisé, avec la même signification que le [champ `session_title` de SessionStart](#entrée-de-sessionstart).

```json
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

* **Stdout en texte brut** : Claude Code ajoute au contexte de Claude le stdout qu'il [traite comme du texte brut](https://code.claude.com/docs/en/hooks#exit-code-0)
* **JSON avec `additionalContext`** : utilisez le format JSON ci-dessous pour plus de contrôle. Le champ `additionalContext` est ajouté comme contexte

Aucun de ces canaux ne produit d'entrée visible dans le transcript. Le stdout brut et la valeur de `additionalContext` sont chacun injectés comme un rappel système (system reminder) commençant par le nom du hook ; Claude lit les deux. Pour confirmer la transmission, consultez le [journal de débogage](https://code.claude.com/docs/en/hooks#debug-hooks).

Pour bloquer un prompt, renvoyez un objet JSON avec `decision` à `"block"` :

| Champ | Description |
| :- | :- |
| `decision` | `"block"` arrête le prompt avant qu'il n'atteigne Claude. Omettez-le pour laisser passer le prompt |
| `reason` | Affiché à l'utilisateur quand `decision` vaut `"block"`. Non ajouté au contexte |
| `additionalContext` | Chaîne ajoutée au contexte de Claude avec le prompt soumis. Voir [Ajouter du contexte pour Claude](https://code.claude.com/docs/en/hooks#add-context-for-claude) |
| `sessionTitle` | Définit le titre de la session. À utiliser pour nommer automatiquement les sessions selon le contenu du prompt |
| `suppressOriginalPrompt` | Si `true` quand le hook bloque le prompt, omet le texte du prompt du message de blocage. Voir [Ce que laisse un prompt bloqué](#ce-que-laisse-un-prompt-bloqué) |

Un hook qui bloque en sortant avec le code 2 est traité comme `reason` : le message de blocage affiche le texte stderr à l'utilisateur, et il n'est pas ajouté au contexte.

```json
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

#### Ce que laisse un prompt bloqué

Un prompt bloqué n'atteint jamais Claude, mais son texte n'est pas supprimé partout. Par défaut, le message de blocage affiché à l'utilisateur se termine par `Original prompt:` suivi du texte soumis, et Claude Code écrit ce message dans le fichier transcript de la session sur disque. Pour exclure le texte du message, imprimez un JSON avec `"suppressOriginalPrompt": true` dans `hookSpecificOutput`. Cela fonctionne que le hook bloque via `decision: "block"` ou en sortant avec le code 2.

`suppressOriginalPrompt` ne modifie que le message de blocage. Le texte soumis peut toujours apparaître dans des fichiers locaux comme le transcript de la session et votre historique de prompts : un hook bloquant n'est donc pas un moyen de garder un secret hors du disque. Pour limiter ou supprimer ces fichiers, voir [Stockage en clair](https://code.claude.com/docs/en/claude-directory#plaintext-storage) et [Effacer les données locales](https://code.claude.com/docs/en/claude-directory#clear-local-data).

### UserPromptExpansion

S'exécute quand une commande tapée par l'utilisateur se développe en prompt avant d'atteindre Claude. Utilisez-le pour empêcher l'invocation directe de certaines commandes, injecter du contexte pour une skill particulière, ou journaliser les commandes invoquées. Par exemple, un hook correspondant à `deploy` peut bloquer `/deploy` sauf si un fichier d'approbation est présent, ou un hook correspondant à une skill de revue peut ajouter la checklist de revue de l'équipe en `additionalContext`.

Cet événement couvre le chemin que `PreToolUse` ne couvre pas : un hook `PreToolUse` correspondant à l'outil `Skill` ne se déclenche que lorsque Claude appelle l'outil, alors que taper `/skillname` directement contourne `PreToolUse`. `UserPromptExpansion` se déclenche sur ce chemin direct.

Le matcher porte sur `command_name`. Laissez le matcher vide pour réagir à chaque commande de type prompt.

#### Entrée d'UserPromptExpansion

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks UserPromptExpansion reçoivent `expansion_type`, `command_name`, `command_args`, `command_source` et la chaîne `prompt` d'origine. Le champ `expansion_type` vaut `slash_command` pour les skills et commandes personnalisées, ou `mcp_prompt` pour les prompts de serveurs MCP.

```json
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
| `decision` | `"block"` empêche l'expansion de la commande. Omettez-le pour la laisser se poursuivre |
| `reason` | Affiché à l'utilisateur quand `decision` vaut `"block"` |
| `additionalContext` | Chaîne ajoutée au contexte de Claude avec le prompt développé. Voir [Ajouter du contexte pour Claude](https://code.claude.com/docs/en/hooks#add-context-for-claude) |

Un hook qui bloque en sortant avec le code 2 est traité comme `reason` : le message de blocage affiche le texte stderr à l'utilisateur.

```json
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

S'exécute pendant qu'un message de l'assistant s'affiche en streaming à l'écran. Claude Code affiche le message par incréments : chaque fois qu'un lot de lignes nouvellement terminées est prêt à être rendu, le hook s'exécute une fois avec ces lignes et Claude Code affiche le texte de remplacement du hook à leur place. Un message long produit plusieurs appels ; un message court peut n'en produire qu'un.

Utilisez MessageDisplay pour :

* retirer le markdown pour un affichage minimal
* transformer le texte qu'une application Agent SDK montre à ses utilisateurs
* masquer des clés d'API ou des noms d'hôtes internes dans les réponses de Claude

Claude Code retient chaque lot jusqu'au retour de votre hook : gardez-le rapide. Si le hook échoue ou dépasse son timeout, Claude Code affiche le texte d'origine. Le timeout par défaut de cet événement est de 10 secondes ; si votre hook a besoin de plus, définissez le champ `timeout` dans l'entrée du hook.

MessageDisplay ne concerne que l'affichage : le texte de remplacement ne modifie que ce qui est rendu à l'écran. Le transcript et ce que voit Claude conservent le texte d'origine ; Claude ne voit donc jamais le remplacement, et le mode verbeux affiche l'original. Le hook ne reçoit que le texte des messages de l'assistant : les résultats d'outils et le texte que vous tapez s'affichent sans changement.

MessageDisplay ne prend pas en charge les matchers et se déclenche pour chaque message de l'assistant qui diffuse du texte ; les messages sans texte, comme les réponses ne contenant que des appels d'outils, ne le déclenchent pas.

Dans les exécutions non interactives, y compris les requêtes Agent SDK et `claude -p`, MessageDisplay s'exécute une fois par message de l'assistant au lieu d'une fois par lot de lignes. L'appel unique arrive après la fin du message et porte le texte complet : `index` vaut `0`, `final` vaut `true`, et `delta` contient le message entier. Un hook qui accumule le texte `delta` de chaque message reçoit le même texte total dans les deux modes.

#### Entrée de MessageDisplay

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks MessageDisplay reçoivent les identifiants du tour et du message, la position de cet appel dans le message, et le nouveau texte dans `delta`. Les limites des lots dépendent de la façon dont le texte est diffusé : utilisez `index` et `final` pour suivre la progression dans un message plutôt que de supposer un regroupement particulier des lignes.

| Champ | Description |
| :- | :- |
| `turn_id` | UUID du tour en cours |
| `message_id` | UUID du message de l'assistant affiché. Stable sur tous les lots d'un même message. Ce n'est pas l'identifiant d'API `msg_…`, il ne peut donc pas être corrélé aux identifiants de messages du transcript |
| `index` | Index (à partir de zéro) de ce lot dans le message |
| `final` | `true` sur le dernier lot du message. Chaque message a exactement un lot final |
| `delta` | Les lignes nouvellement terminées depuis le lot précédent, sauts de ligne finaux inclus. Toujours des lignes entières, sauf le lot final qui peut s'arrêter en milieu de ligne. En exécution interactive, le delta du lot final est vide quand le message se termine par un saut de ligne : considérez `final`, et non un delta non vide, comme signal de fin de message. Dans les exécutions Agent SDK et `claude -p`, l'appel unique porte le message entier |

```json
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

Les hooks MessageDisplay n'ont aucun contrôle de décision. Ils ne peuvent ni bloquer le message ni modifier ce qui est stocké dans le transcript ou envoyé à Claude. Claude Code prend en compte `displayContent` de leur sortie JSON et ignore `systemMessage` et `continue`.

Cet exemple retire la mise en forme markdown des réponses de Claude pour un affichage en texte brut. Le script lit chaque lot depuis stdin, retire les marqueurs de gras et les backticks de code en ligne de `delta`, et renvoie le résultat en `displayContent`.

##### macOS/Linux

Enregistrez un hook de commande pour l'événement dans votre fichier de réglages :

```json
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

```bash
#!/bin/bash
jq '{hookSpecificOutput: {hookEventName: "MessageDisplay", displayContent: (.delta | gsub("\\*\\*"; "") | gsub("`"; ""))}}'
```

##### Windows (PowerShell)

Enregistrez un hook de commande qui exécute le script via PowerShell :

```json
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

```powershell
$batch = [Console]::In.ReadToEnd() | ConvertFrom-Json
$text = $batch.delta -replace '\*\*', '' -replace '`', ''
@{
  hookSpecificOutput = @{
    hookEventName = "MessageDisplay"
    displayContent = $text
  }
} | ConvertTo-Json
```

Les lots sans markdown passent inchangés. Si le script échoue, par exemple parce que `jq` est absent, Claude Code affiche le texte d'origine et ne signale l'échec que dans la [sortie de débogage](https://code.claude.com/docs/en/hooks#debug-hooks), pas dans la session.

### PreToolUse

S'exécute après que Claude a créé les paramètres de l'outil et avant le traitement de l'appel. Correspond à n'importe quel nom d'outil sauf `EndConversation` : outils intégrés comme `Bash`, `PowerShell`, `Edit`, `Write`, `Read`, `Glob`, `Grep`, `Agent`, `Workflow`, `WebFetch`, `WebSearch`, `AskUserQuestion` et `ExitPlanMode`, et tout [nom d'outil MCP](https://code.claude.com/docs/en/hooks#match-mcp-tools).

Pour exécuter un hook quand un fichier précis change sur le disque, quel qu'en soit l'auteur, utilisez [FileChanged](#filechanged) plutôt que de cibler par nom les outils d'édition de fichiers. Contrairement à PreToolUse, Claude Code exécute les hooks FileChanged après le changement, et ils n'ont pas de contrôle de décision : ils ne peuvent donc pas bloquer l'écriture.

> **Avertissement** : PreToolUse ne s'exécute que lorsque Claude appelle un outil. Les fichiers que vous [référencez avec `@` dans votre prompt](https://code.claude.com/docs/en/common-workflows#reference-files-and-directories) sont ajoutés sans appel d'outil : Claude Code insère leur contenu en construisant le prompt, donc aucun hook PreToolUse ne se déclenche pour eux, y compris les hooks correspondant à `Read`. Pour empêcher certains chemins d'être référencés avec `@`, utilisez plutôt une [règle de refus `Read`](https://code.claude.com/docs/en/permissions#read-and-edit).
>
> PreToolUse ne se déclenche pas non plus pour [`EndConversation`](https://code.claude.com/docs/en/tools-reference#endconversation-tool-behavior).

Utilisez le [contrôle de décision de PreToolUse](#contrôle-de-décision-de-pretooluse) pour autoriser, refuser, demander ou différer l'appel d'outil.

Un [hook callback de l'Agent SDK](https://code.claude.com/docs/en/agent-sdk/hooks) sur `PreToolUse` qui dépasse son timeout bloque l'appel d'outil, et Claude reçoit un résultat d'erreur nommant le timeout. Un refus explicite renvoyé par un autre hook garde la priorité.

#### Entrée de PreToolUse

En plus des [champs d'entrée communs](https://code.claude.com/docs/en/hooks#common-input-fields), les hooks PreToolUse reçoivent `tool_name`, `tool_input` et `tool_use_id`.

Pour un [outil MCP](https://code.claude.com/docs/en/hooks#match-mcp-tools), l'entrée porte aussi `mcp_server`, un objet avec le `name` du serveur et une `source` indiquant d'où vient la définition du serveur. Les valeurs de `source` incluent `plugin`, `sdk`, et des portées de configuration comme `user` et `project`. [`McpServerProvenance`](https://code.claude.com/docs/en/agent-sdk/typescript#mcpserverprovenance) dans la référence de l'Agent SDK les liste toutes et indique comment traiter une valeur inconnue. Fondez vos décisions de confiance sur `source` plutôt que sur `name` ou sur le préfixe de nom d'outil `mcp__<server>__`. Le champ `mcp_server` exige Claude Code v2.1.274+.

Pour les outils de fichiers `Write`, `Edit` et `Read`, `tool_input.file_path` est toujours absolu :

* Claude Code développe `~` et les chemins relatifs avant l'exécution des hooks ; un hook qui filtre sur les chemins ne peut donc pas être contourné via `~` ou une écriture relative du même chemin
* Sous Windows, le chemin arrive avec des séparateurs barre oblique inverse, même quand votre hook tourne sous Git Bash où `$PWD` ressemble à `/c/project`
* Une comparaison écrite avec des barres obliques, comme un test `/src/`, ne correspond jamais à un chemin à barres obliques inverses, et l'appel d'outil se poursuit comme si le hook n'avait rien à bloquer
* Normalisez les séparateurs avant de comparer : `FILE_PATH="${FILE_PATH//\\//}"` en Bash, ou `file_path.replace("\\", "/")` en Python, puis recherchez un segment de chemin comme `/src/` plutôt que d'ancrer avec `^`, puisque le chemin est absolu

Un appel `Write` sous Windows transmet :

```json
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
| `timeout` | number | `120000` | Timeout facultatif en millisecondes. Les valeurs au-dessus du [maximum](https://code.claude.com/docs/en/tools-reference#bash-tool-behavior) sont ramenées au maximum plutôt que rejetées |
| `run_in_background` | boolean | `false` | Exécuter ou non la commande en arrière-plan |

Quand une commande Bash modifie des fichiers dans un dépôt Git, Claude Code peut enregistrer ce qui a changé. Il enregistre les changements dans tous les modes de permission quand le réglage [`bashEditDiffEnabled`](https://code.claude.com/docs/en/settings-reference#basheditdiffenabled) active l'enregistrement (l'entrée de ce réglage indique quels fichiers peuvent le définir). Sinon, il ne les enregistre qu'en mode auto et en mode `bypassPermissions`, et seulement quand Claude Code demande à Claude d'éditer des fichiers via Bash. Définissez `bashEditDiffEnabled` à `false` pour désactiver l'enregistrement. Les commandes en arrière-plan et les commandes en lecture seule ne portent pas de diff.

Votre [hook PostToolUse](#posttooluse) reçoit alors les fichiers modifiés dans `tool_response.bashEditDiff`. La liste couvre ce qui a changé dans le dépôt pendant l'exécution de la commande. Les fichiers ignorés par Git et les fichiers des sous-modules ne sont pas listés. Exige Claude Code v2.1.269+.

> **Note** : La liste est établie au mieux (best effort) et en bêta publique. Claude Code peut manquer un changement, inclure un fichier qu'un autre processus a modifié en même temps, ou s'arrêter à ses limites de taille. La forme du champ peut changer. Utilisez la liste pour savoir quoi relire, pas pour appliquer une politique.

`changedFiles` et `files` listent ce que la commande a modifié ; les autres champs indiquent dans quelle mesure cette liste est complète et fiable.

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `changedFiles` | array | `["/path/to/src/app.ts"]` | Chemins absolus des fichiers modifiés par la commande, 200 au maximum. Présent dès que `files` contient un diff ou que `moreFiles` est supérieur à zéro |
| `files` | array | `[{"filePath": "/path/to/src/app.ts", "hunks": [...]}]` | Diffs de jusqu'à 5 fichiers modifiés, pour l'affichage. `created` ou `deleted` vaut `true` pour un fichier que la commande a ajouté ou supprimé |
| `moreFiles` | number | `2` | Nombre de fichiers modifiés sans diff dans `files` |
| `unavailable` | boolean | `true` | Défini quand le diff est incomplet ou n'a pas pu être pris |
| `skipped` | boolean | `true` | Défini pour une commande Git qui déplace l'arbre de travail, comme `git checkout` ou `git stash`, pour laquelle Claude Code ne prend aucun diff |
| `shared` | boolean | `true` | Défini quand un autre appel d'outil Bash, par exemple d'un sous-agent, a tourné dans le même dépôt au même moment : certains changements listés peuvent provenir de cette commande |

##### PowerShell

Exécute des commandes PowerShell. Voir l'[outil PowerShell](https://code.claude.com/docs/en/tools-reference#powershell-tool) pour la disponibilité selon la plateforme.

Les champs sont les mêmes que pour l'outil Bash, avec la chaîne de commande dans `command` :

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `command` | string | `"Get-ChildItem -Recurse"` | La commande PowerShell à exécuter |
| `description` | string | `"List files recursively"` | Description facultative de ce que fait la commande |
| `timeout` | number | `120000` | Timeout facultatif en millisecondes |
| `run_in_background` | boolean | `false` | Exécuter ou non la commande en arrière-plan |

Utilisez le matcher `Bash|PowerShell` dans les hooks qui inspectent les commandes shell, pour couvrir les deux outils :

* Sous Windows, partout où l'outil PowerShell est activé, Claude traite PowerShell comme shell principal et y fait passer les commandes shell.
* Sous Windows sans Git Bash, l'outil est activé automatiquement et Claude Code n'enregistre pas du tout l'outil Bash.
* Un hook qui ne cible que `Bash` ne s'y déclenche jamais.

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
| `replace_all` | boolean | `false` | Remplacer ou non toutes les occurrences |

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
| `path` | string | `"/path/to/dir"` | Répertoire facultatif où chercher. Par défaut, le répertoire de travail courant |

##### Grep

Recherche dans le contenu des fichiers avec des expressions régulières.

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `pattern` | string | `"TODO.*fix"` | Motif d'expression régulière à rechercher |
| `path` | string | `"/path/to/dir"` | Fichier ou répertoire facultatif où chercher |
| `glob` | string | `"*.ts"` | Motif glob facultatif pour filtrer les fichiers |
| `output_mode` | string | `"content"` | `"content"`, `"files_with_matches"` ou `"count"`. Par défaut `"files_with_matches"` |
| `-i` | boolean | `true` | Recherche insensible à la casse |
| `multiline` | boolean | `false` | Active la correspondance multiligne |

##### WebFetch

Récupère et traite du contenu web.

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `url` | string | `"https://example.com/api"` | URL dont récupérer le contenu |
| `prompt` | string | `"Extract the API endpoints"` | Prompt à appliquer au contenu récupéré |
| `offset` | number | `100000` | Nombre facultatif de caractères à sauter depuis le début de la page. Claude le définit pour continuer à lire une page longue. Exige Claude Code v2.1.290+ |

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

Quand un appel Agent au premier plan se termine, votre [hook PostToolUse](#posttooluse) reçoit le résultat du sous-agent et la télémétrie de son exécution dans `tool_response`. Lisez ces champs pour inspecter l'exécution ; pour des cumuls de tokens et de coûts sur l'ensemble des sous-agents, utilisez les [compteurs de tokens et de coûts](https://code.claude.com/docs/en/monitoring-usage#token-counter) filtrés sur `query_source` `"subagent"`, car `totalTokens` et `usage` ne couvrent que la dernière requête :

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `status` | string | `"completed"` | `"completed"` pour les sous-agents au premier plan, `"async_launched"` pour les sous-agents en arrière-plan. Les sous-agents tournent en arrière-plan par défaut : un appel Agent qui omet `run_in_background` produit donc aussi `"async_launched"` |
| `agentId` | string | `"a4d2c8f1e0b3a297"` | Identifiant de l'exécution du sous-agent |
| `content` | array | `[{"type": "text", "text": "Found 12 endpoints..."}]` | Les blocs de texte finaux du sous-agent ou, pour un sous-agent dont le rapport passe par `SubagentHandback`, une courte note sur cette remise à la place |
| `resolvedModel` | string | `"claude-sonnet-4-5"` | Modèle sur lequel le sous-agent a démarré, qui peut différer du modèle demandé |
| `modelsUsed` | array | `["claude-sonnet-4-5", "claude-haiku-4-5"]` | Modèles utilisés dans l'ordre, répétitions consécutives fusionnées ; défini seulement si le modèle a changé en cours d'exécution. Exige Claude Code v2.1.212+ |
| `totalTokens` | number | `12450` | Nombre de tokens de la dernière requête API du sous-agent : entrée, sortie et cache combinés. Ce n'est pas un total sur toute l'exécution |
| `totalDurationMs` | number | `48211` | Durée (temps réel) de l'exécution du sous-agent |
| `totalToolUseCount` | number | `7` | Nombre d'appels d'outils effectués par le sous-agent |
| `usage` | object | `{"input_tokens": 8320, ...}` | Ventilation par type des tokens de la dernière requête API : `input_tokens`, `output_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens` |

À partir de Claude Code v2.1.271, un sous-agent qui tourne avec l'outil [`SubagentHandback`](https://code.claude.com/docs/en/tools-reference), que Claude Code fournit en [mode auto](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode), remet son rapport via cet outil au lieu de le renvoyer en texte. Le champ `content` de son résultat `completed` porte alors une courte note sur cette remise plutôt que le rapport lui-même. Pour lire le rapport, ciblez `SubagentHandback` avec un hook `PreToolUse` ou `PostToolUse` et lisez `tool_input.message`.

Pour les sous-agents en arrière-plan, l'outil rend la main au moment où la tâche passe en arrière-plan, donc `tool_response` ne porte aucun champ d'usage : un lancement en arrière-plan rend la main immédiatement, et une tâche au premier plan que Claude Code passe en arrière-plan en cours d'exécution rend la main à cette transition. Il contient `status: "async_launched"`, `agentId`, `description`, `prompt`, `outputFile` et `resolvedModel`.

Sur une réponse `completed`, `resolvedModel` nomme le modèle sur lequel le sous-agent a démarré, qui peut différer de la valeur `model` de `tool_input`, par exemple quand `availableModels` ou une autre surcharge s'applique. Sur une réponse `async_launched`, `resolvedModel` nomme le modèle en usage au moment du passage en arrière-plan : un changement survenu avant ce passage y est donc reflété. `modelsUsed` et ce comportement de `resolvedModel` au passage en arrière-plan exigent Claude Code v2.1.212+.

##### AskUserQuestion

Pose à l'utilisateur une à quatre questions à choix multiples.

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `questions` | array | `[{"question": "Which framework?", "header": "Framework", "options": [{"label": "React", "description": "Component library"}, {"label": "Vue", "description": "Progressive framework"}], "multiSelect": false}]` | Questions à présenter, chacune avec une chaîne `question`, un `header` court, un tableau `options` et un flag `multiSelect` facultatif |
| `answers` | object | `{"Which framework?": "React"}` | Facultatif. Associe le texte de chaque question au libellé de l'option choisie. Les réponses multi-sélection joignent les libellés par des virgules. Claude ne définit pas ce champ ; fournissez-le via `updatedInput` pour répondre par programme |

##### ExitPlanMode

Présente un plan et demande à l'utilisateur de l'approuver avant que Claude ne quitte le [mode plan](https://code.claude.com/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode). Claude écrit le plan dans un fichier sur disque avant d'appeler l'outil, donc le `tool_input` littéral venant du modèle est généralement vide. Claude Code injecte le contenu du plan et le chemin du fichier avant de transmettre l'entrée aux hooks.

| Champ | Type | Exemple | Description |
| :- | :- | :- | :- |
| `plan` | string | `"## Refactor auth\n1. Extract..."` | Contenu du plan en Markdown. Injecté depuis le fichier de plan sur disque |
| `planFilePath` | string | `"/Users/.../plans/refactor-auth.md"` | Chemin du fichier de plan. Injecté |
| `allowedPrompts` | array | `[{"tool": "Bash", "prompt": "run tests"}]` | Obsolète. Claude Code accepte le champ mais l'ignore. Avant la v2.1.205, il portait les permissions par prompt demandées par Claude pour mettre en œuvre le plan |

Dans `PostToolUse`, `tool_response` est un objet avec les champs `plan` et `filePath` contenant le plan approuvé, plus des indicateurs d'état internes. Lisez `tool_response.plan` pour obtenir le contenu du plan plutôt que de relire le fichier sur disque.

#### Contrôle de décision de PreToolUse

Les hooks `PreToolUse` peuvent contrôler si un appel d'outil se poursuit. Contrairement aux autres hooks qui utilisent un champ `decision` de premier niveau, PreToolUse renvoie sa décision dans un objet `hookSpecificOutput`. Cela lui donne un contrôle plus riche : quatre issues (allow, deny, ask ou defer) plus la possibilité de modifier l'entrée de l'outil avant l'exécution.

| Champ | Description |
| :- | :- |
| `permissionDecision` | `"allow"` saute le prompt de permission, sauf pour les [actions qu'aucun mode n'approuve automatiquement](https://code.claude.com/docs/en/permission-modes#actions-no-mode-auto-approves), pour les [lectures depuis des chemins réseau](https://code.claude.com/docs/en/permissions#network-paths), et pour `AskUserQuestion` et `ExitPlanMode`, qui exigent d'être [couplés à `updatedInput`](#outils-nécessitant-une-interaction-utilisateur). `"deny"` empêche l'appel d'outil. `"ask"` demande confirmation à l'utilisateur. `"defer"` sort proprement pour que l'outil puisse être repris plus tard. Les [règles deny et ask](https://code.claude.com/docs/en/permissions#manage-permissions) sont évaluées quoi que renvoie le hook |
| `permissionDecisionReason` | Pour `"ask"`, affiché à l'utilisateur dans le prompt de permission. Quand Claude Code [refuse l'appel](https://code.claude.com/docs/en/headless#turn-off-permission-prompts-in-unattended-runs) dans une exécution `-p` où personne ne peut répondre à ce prompt, Claude lit la raison dans le résultat de l'outil. Pour `"deny"`, affiché à Claude. Pour `"allow"` et `"defer"`, écrit uniquement dans le [journal de débogage](https://code.claude.com/docs/en/hooks#debug-hooks) |
| `updatedInput` | Modifie les paramètres d'entrée de l'outil avant l'exécution. Remplace l'objet d'entrée entier : incluez les champs inchangés avec les champs modifiés. Claude Code évalue les règles de permission et l'[éligibilité au passage automatique en arrière-plan](https://code.claude.com/docs/en/tools-reference#foreground-commands-that-move-to-the-background) d'une commande Bash sur l'entrée renvoyée par votre hook, pas sur celle envoyée par Claude. À combiner avec `"allow"` pour approuver automatiquement, ou `"ask"` pour montrer l'entrée modifiée à l'utilisateur. Ignoré pour `"defer"` |
| `additionalContext` | Chaîne ajoutée au contexte de Claude avec le résultat de l'outil. Ignorée quand `permissionDecision` vaut `"defer"`. Voir [Ajouter du contexte pour Claude](https://code.claude.com/docs/en/hooks#add-context-for-claude) |

Quand plusieurs hooks PreToolUse renvoient des décisions différentes, l'ordre de priorité est `deny` > `defer` > `ask` > `allow`.

Un hook qui bloque en sortant avec le code 2 est traité comme `"deny"` : Claude voit le message stderr comme raison du refus.

Quand un hook renvoie `"ask"`, le prompt de permission affiché à l'utilisateur inclut une étiquette identifiant l'origine du hook : `[settings]` pour un hook issu de n'importe quel fichier de réglages ou du frontmatter d'un agent, `[plugin:<name>]` pour un hook de plugin, ou `[skill]` pour un hook issu du frontmatter d'une skill. Cela aide l'utilisateur à comprendre quelle source de configuration demande la confirmation.

Le `"ask"` d'un hook force aussi un prompt de permission en [mode auto](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) : le classifieur peut toujours refuser l'appel, mais il ne peut pas l'approuver silencieusement. Avant la v2.1.211, le classifieur pouvait approuver une commande Bash s'exécutant hors de la [sandbox](https://code.claude.com/docs/en/sandboxing) sans afficher le prompt demandé par le hook ; le classifieur appliquait tout de même ses propres règles de sécurité à cette commande, et un `"deny"` de hook était toujours respecté.

```json
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

> **Note** : PreToolUse utilisait auparavant les champs de premier niveau `decision` et `reason`, mais ils sont obsolètes pour cet événement. Utilisez plutôt `hookSpecificOutput.permissionDecision` et `hookSpecificOutput.permissionDecisionReason`. Les anciennes valeurs `"approve"` et `"block"` correspondent respectivement à `"allow"` et `"deny"`. Les autres événements comme PostToolUse et Stop continuent d'utiliser `decision` et `reason` de premier niveau comme format actuel.

#### Outils nécessitant une interaction utilisateur

`AskUserQuestion` et `ExitPlanMode` nécessitent une interaction utilisateur. En [mode non interactif](https://code.claude.com/docs/en/headless) avec le flag `-p`, Claude Code ne les propose que si l'exécution dispose d'un [hôte de permissions](https://code.claude.com/docs/en/headless#turn-off-permission-prompts-in-unattended-runs) pour recevoir le prompt, comme un callback `canUseTool` de l'Agent SDK.

Un hook `PreToolUse` satisfait cette exigence s'il :

1. lit l'entrée de l'outil depuis stdin ;
2. recueille la réponse via votre propre interface ;
3. renvoie `permissionDecision: "allow"` avec un `updatedInput` contenant la réponse, pour que l'outil s'exécute sans prompt.

Renvoyer `"allow"` seul ne suffit pas pour ces outils.

Pour `AskUserQuestion`, renvoyez le tableau `questions` d'origine et ajoutez un objet [`answers`](#askuserquestion) associant le texte de chaque question à la réponse choisie. Cette sortie répond `React` à une question :

```json
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

Un outil MCP que son serveur marque avec [`_meta["anthropic/requiresUserInteraction"]`](https://code.claude.com/docs/en/mcp#require-approval-for-a-specific-tool) est plus strict : un hook ne peut pas sauter son prompt d'approbation avec `"allow"`, avec ou sans `updatedInput`, car Claude Code ne peut pas confirmer que le hook a recueilli l'interaction dont l'outil a besoin.

#### Différer un appel d'outil

`"defer"` s'adresse aux intégrations qui lancent `claude -p` comme sous-processus et lisent sa sortie JSON, comme une application Agent SDK ou une interface personnalisée construite sur Claude Code. Il permet à ce processus appelant de mettre Claude en pause sur un appel d'outil, de recueillir une entrée via sa propre interface, puis de reprendre là où il s'était arrêté. Claude Code ne respecte cette valeur qu'en [mode non interactif](https://code.claude.com/docs/en/headless) avec le flag `-p`. En session interactive, il journalise un avertissement et ignore le résultat du hook.

L'outil `AskUserQuestion` est le cas typique : Claude veut poser une question à l'utilisateur, mais il n'y a pas de terminal pour y répondre. Une exécution `-p` ne propose `AskUserQuestion` que si elle dispose d'un [hôte de permissions](https://code.claude.com/docs/en/headless#turn-off-permission-prompts-in-unattended-runs), comme un outil MCP passé avec `--permission-prompt-tool` ; lancez donc l'exécution avec un tel hôte. L'aller-retour fonctionne ainsi :

1. Claude appelle `AskUserQuestion`. Le hook `PreToolUse` se déclenche.
2. Le hook renvoie `permissionDecision: "defer"`. L'outil ne s'exécute pas. Le processus se termine avec `stop_reason: "tool_deferred"` et l'appel d'outil en attente est conservé dans le transcript.
3. Le processus appelant lit `deferred_tool_use` dans le résultat du SDK, présente la question dans sa propre interface et attend une réponse.
4. Le processus appelant exécute `claude -p --resume <session-id>` avec le même hôte de permissions. Le même appel d'outil déclenche à nouveau `PreToolUse`.
5. Le hook renvoie `permissionDecision: "allow"` avec la réponse dans `updatedInput`. L'outil s'exécute et Claude continue.

Le champ `deferred_tool_use` porte l'`id`, le `name` et l'`input` de l'outil. L'`input` correspond aux paramètres que Claude a générés pour l'appel, capturés avant l'exécution :

```json
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

Il n'y a ni timeout ni limite de tentatives. La session reste sur disque jusqu'à sa reprise, sous réserve du balayage de rétention [`cleanupPeriodDays`](https://code.claude.com/docs/en/settings-reference#cleanupperioddays), qui supprime les fichiers de session après 30 jours par défaut, selon les [règles de balayage de rétention](https://code.claude.com/docs/en/claude-directory#cleaned-up-automatically). Si la réponse n'est pas prête à la reprise, le hook peut renvoyer `"defer"` de nouveau et le processus se termine de la même façon. Le processus appelant décide quand sortir de la boucle en renvoyant finalement `"allow"` ou `"deny"` depuis le hook.

`"defer"` ne fonctionne que lorsque Claude fait un seul appel d'outil dans le tour. Si Claude fait plusieurs appels à la fois, `"defer"` est ignoré avec un avertissement et l'outil suit le flux de permission normal. Cette contrainte existe parce que la reprise ne peut réexécuter qu'un seul outil : impossible de différer un appel d'un lot sans laisser les autres en suspens.

Si l'outil différé n'est plus disponible à la reprise, le processus se termine avec `stop_reason: "tool_deferred_unavailable"` et `is_error: true` avant le déclenchement du hook. Cela arrive quand un serveur MCP qui fournissait l'outil n'est pas connecté pour la session reprise. La charge utile `deferred_tool_use` est tout de même incluse pour vous permettre d'identifier l'outil manquant.

> **Note** : Pour reprendre une session différée en mode plan, passez [`--permission-prompt-tool`](https://code.claude.com/docs/en/cli-reference#cli-flags) avec `--resume` afin que Claude Code puisse présenter le plan pour approbation. Pour les autres conditions, voir [Reprendre en mode plan avec `-p`](https://code.claude.com/docs/en/sessions#resume-in-plan-mode-with-p). Exige Claude Code v2.1.246+.
>
> Quand vous reprenez avec `-p`, Claude Code ne restaure aucun autre mode de permission stocké. Il démarre l'exécution dans le mode de permission dans lequel démarrerait un nouveau `claude -p` : repassez donc `--permission-mode` ou `--dangerously-skip-permissions` si la session différée en utilisait un. Quand vous reprenez avec `claude --resume <session-id>` sans `-p`, Claude Code restaure le mode de permission stocké, avec les exceptions listées dans [mode de permission à la reprise](https://code.claude.com/docs/en/sessions#permission-mode-on-resume).

