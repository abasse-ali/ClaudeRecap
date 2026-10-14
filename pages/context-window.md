# Explorer la fenêtre de contexte

> Source : https://code.claude.com/docs/en/context-window · [← Index](../31-index-documentation-officielle.md)

> Une simulation interactive de la façon dont la fenêtre de contexte de Claude Code se remplit au cours d'une session. Voyez ce qui se charge automatiquement, ce que coûte chaque lecture de fichier, et quand les règles et les hooks se déclenchent.

La fenêtre de contexte de Claude Code contient tout ce que Claude sait de votre session : vos instructions, les fichiers qu'il lit, ses propres réponses, et du contenu qui n'apparaît jamais dans votre terminal. La chronologie interactive de la page officielle rejoue une session complète, du démarrage jusqu'à la compaction : ce qui se charge avant que vous ne tapiez quoi que ce soit, ce qu'ajoute chaque lecture de fichier, chaque règle et chaque hook pendant que Claude travaille, et comment un sous-agent garde les lectures volumineuses hors de votre contexte. Voir [la description écrite](#ce-que-montre-la-chronologie) pour le même contenu sous forme de liste.

## La simulation interactive (composant `ContextWindow`)

La page officielle embarque un composant interactif intitulé **« Explore the context window »**, sous-titré **« A simulated session showing what enters context and what it costs »** (« Une session simulée montrant ce qui entre dans le contexte et ce que cela coûte »). Toutes les données qu'il contient sont reprises ci-dessous.

### Paramètres de la simulation

* **Taille de fenêtre simulée** : 200 000 tokens (affichée « / 200K · illustrative »).
* **Infobulle sur le compteur** : « Les nombres de tokens sont illustratifs. Les valeurs réelles varient selon la taille de votre CLAUDE.md, vos serveurs MCP et la longueur de vos fichiers. »
* **Compteur principal** : « ~N tokens » (format abrégé, ex. `22.2K`), dont la couleur dépend du remplissage : vert `#558A42` jusqu'à 50 %, ocre `#B8860B` au-delà de 50 %, orange `#D97757` au-delà de 75 %.
* **Barre de remplissage** : une fine barre de progression, puis une barre empilée où chaque bloc a une largeur proportionnelle à ses tokens (minimum 0,15 % pour rester visible) et la couleur de sa catégorie ; survoler ou cliquer un bloc affiche son détail.
* **Fin de la phase de démarrage** : à 20 % de la chronologie (`STARTUP_END = 0.2`) ; tout ce qui est chargé avant ce point est du contenu « de démarrage ».
* **Vitesse** : la chronologie avance de 3,2 % par seconde de lecture (une trentaine de secondes au total hors pauses).
* Dans la simulation, la session atteint environ **22,2K tokens** juste avant `/compact` (≈ 11 % des 200K, barre verte), puis redescend à **16,6K** après compaction.

### Déroulé : les points d'arrêt où vous intervenez

La lecture s'interrompt à cinq moments où « vous » devez envoyer une saisie (bouton **Send ↵** pour un prompt, **Run ↵** pour une commande) :

| Position | Type | Texte saisi | Reprise à |
| :- | :- | :- | :- |
| 18 % | prompt | `Fix the auth bug where users get 401 after token refresh` | 22 % |
| 70,5 % | prompt | `Use a subagent to research session timeout handling, then fix it` | 72 % |
| 86,5 % | commande `!` (bang) | `!git status` | 87,5 % |
| 88 % | commande slash | `/commit-push` | 89 % |
| 90 % | compaction | `/compact` | 100 % |

Au point d'arrêt `/compact`, le panneau affiche : « Le contexte est à N tokens. Exécutez `/compact` pour résumer les échanges plus anciens et libérer de l'espace pour davantage de travail. »

### Phase 1 — Avant que vous ne tapiez quoi que ce soit

Ces éléments sont chargés automatiquement (badge `auto`) dès le lancement de `claude`, avant votre premier message. Total : 7 850 tokens.

| Position | Élément | Tokens | Visibilité dans le terminal | Après `/compact` |
| :- | :- | :- | :- | :- |
| 1,5 % | System prompt | 4 200 | Invisible | Rechargé |
| 3,5 % | Auto memory (`MEMORY.md`) | 680 | Invisible | Rechargé |
| 6 % | Informations d'environnement | 280 | Invisible | Rechargé |
| 8 % | Outils MCP (différés) | 120 | Invisible | Rechargé |
| 10 % | Descriptions des skills | 450 | Invisible | **Non réinjecté** |
| 12 % | `~/.claude/CLAUDE.md` | 320 | Invisible | Rechargé |
| 14 % | `CLAUDE.md` du projet | 1 800 | Invisible | Rechargé |

Détail de chaque élément :

* **System prompt** (4 200 tokens) — Instructions de base pour le comportement, l'utilisation des outils et la mise en forme des réponses. Toujours chargé en premier. Vous ne le voyez jamais.
* **Auto memory (`MEMORY.md`)** (680 tokens) — Les notes que Claude s'est laissées lors de sessions précédentes : commandes de build apprises, schémas remarqués, erreurs à éviter. Les 200 premières lignes ou les 25 Ko premiers, selon la limite atteinte en premier, sont chargés dans le contexte de la conversation. En savoir plus : https://code.claude.com/docs/en/memory#auto-memory
* **Informations d'environnement** (280 tokens) — Répertoire de travail, plateforme, shell, version de l'OS, et indication s'il s'agit d'un dépôt git. La branche git, le statut et les commits récents sont chargés dans un bloc séparé.
* **Outils MCP (différés)** (120 tokens) — Les noms des outils MCP sont listés pour que Claude sache ce qui est disponible. Par défaut, les schémas complets restent différés et Claude charge à la demande ceux dont une tâche a besoin, via la recherche d'outils (tool search). Définissez `ENABLE_TOOL_SEARCH=auto` pour charger les schémas dès le départ lorsqu'ils tiennent dans 10 % de la fenêtre de contexte, ou `ENABLE_TOOL_SEARCH=false` pour tout charger. En savoir plus : https://code.claude.com/docs/en/mcp#scale-with-mcp-tool-search
* **Descriptions des skills** (450 tokens) — Descriptions d'une ligne des skills disponibles, pour que Claude sache ce qu'il peut invoquer. Le contenu complet d'un skill ne se charge que lorsque Claude l'utilise réellement. Les skills avec `disable-model-invocation: true` ne figurent pas dans cette liste : ils restent totalement hors du contexte jusqu'à ce que vous les invoquiez avec `/name`. Contrairement au reste du contenu de démarrage, cette liste **n'est pas réinjectée après `/compact`** ; seuls les skills effectivement invoqués sont préservés. En savoir plus : https://code.claude.com/docs/en/skills
* **`~/.claude/CLAUDE.md`** (320 tokens) — Vos préférences globales. S'appliquent à tous les projets. Chargées en même temps que les instructions du projet au début de chaque conversation. En savoir plus : https://code.claude.com/docs/en/memory#choose-where-to-put-claude-md-files
* **`CLAUDE.md` du projet** (1 800 tokens) — Conventions du projet, commandes de build, notes d'architecture. Le fichier le plus important que vous puissiez créer. Il se trouve à la racine du projet, donc toute l'équipe reçoit les mêmes instructions. En savoir plus : https://code.claude.com/docs/en/memory
  * 💡 *Économiser du contexte* : gardez-le sous 200 lignes. Déplacez le contenu de référence vers des skills ou des règles limitées à des chemins (path-scoped rules) pour qu'il ne se charge que lorsque c'est nécessaire.

### Phase 2 — Votre prompt et le travail de Claude

Premier prompt (badge `you`) puis travail de Claude (badge `claude`), avec règles chargées automatiquement et hooks.

| Position | Élément | Type | Tokens | Visibilité | Restauré après `/compact` |
| :- | :- | :- | :- | :- | :- |
| 22 % | Votre prompt | vous | 45 | Affiché | Non (résumé) |
| 28 % | Read `src/api/auth.ts` | claude | 2 400 | Une ligne | Oui (relu) |
| 32 % | Read `src/lib/tokens.ts` | claude | 1 100 | Une ligne | Oui (relu) |
| 35 % | Règle : `api-conventions.md` | auto | 380 | Une ligne | Non (résumé) |
| 38 % | Read `middleware.ts` | claude | 1 800 | Une ligne | Oui (relu) |
| 41 % | Read `auth.test.ts` | claude | 1 600 | Une ligne | Oui (relu) |
| 44 % | Règle : `testing.md` | auto | 290 | Une ligne | Non (résumé) |
| 47 % | `grep "refreshToken"` | claude | 600 | Une ligne | Non (résumé) |
| 53 % | Analyse de Claude | claude | 800 | Affiché | Non (résumé) |
| 57 % | Edit `auth.ts` | claude | 400 | Affiché | Non (résumé) |
| 59 % | Hook : prettier | hook | 120 | Invisible | Non (résumé) |
| 62 % | Edit `auth.test.ts` | claude | 600 | Affiché | Non (résumé) |
| 64 % | Hook : prettier | hook | 100 | Invisible | Non (résumé) |
| 67 % | Sortie de `npm test` | claude | 1 200 | Une ligne | Non (résumé) |
| 70 % | Résumé | claude | 400 | Affiché | Non (résumé) |

Détail de chaque élément :

* **Votre prompt** (45 tokens) — « Fix the auth bug where users get 401 after token refresh » (corriger le bug d'authentification où les utilisateurs reçoivent un 401 après le rafraîchissement du jeton).
* **Read `src/api/auth.ts`** (2 400 tokens) — Fichier principal d'authentification. Vous voyez « Read auth.ts » dans votre terminal, mais les 2 400 tokens de contenu du fichier, seul Claude les voit.
  * 💡 *Économiser du contexte* : les lectures de fichiers dominent l'utilisation du contexte. Soyez précis dans vos prompts (« fix the bug in auth.ts ») pour que Claude lise moins de fichiers. Pour les tâches qui demandent beaucoup de recherche, utilisez un sous-agent.
* **Read `src/lib/tokens.ts`** (1 100 tokens) — Claude suit les imports jusqu'au module de jetons. Affiché sous forme d'une ligne dans votre terminal.
* **Règle : `api-conventions.md`** (380 tokens, badge `auto`) — Cette règle dans `.claude/rules/` possède un motif `paths:` correspondant à `src/api/**`. Elle s'est chargée automatiquement lorsque Claude a lu un fichier de ce répertoire. Vous voyez « Loaded .claude/rules/api-conventions.md » dans votre terminal, mais pas le contenu de la règle. En savoir plus : https://code.claude.com/docs/en/memory#path-specific-rules
* **Read `middleware.ts`** (1 800 tokens) — Claude suit le flux d'authentification plus en profondeur.
* **Read `auth.test.ts`** (1 600 tokens) — Vérification des tests existants pour connaître le comportement attendu.
* **Règle : `testing.md`** (290 tokens, badge `auto`) — Une autre règle limitée à des chemins, correspondant cette fois aux fichiers `*.test.ts`. Déclenchée lorsque Claude a lu `auth.test.ts`. Affichée sous forme d'un avis « Loaded » d'une ligne. En savoir plus : https://code.claude.com/docs/en/memory#path-specific-rules
* **`grep "refreshToken"`** (600 tokens) — Résultats de recherche dans toute la base de code. Vous voyez que la commande s'est exécutée, pas la sortie complète.
* **Analyse de Claude** (800 tokens) — Explique le bug : le jeton est invalidé trop tôt dans la rotation. Ce texte apparaît dans votre terminal.
* **Edit `auth.ts`** (400 tokens) — Corrige l'ordre de rotation du jeton. Le diff apparaît dans votre terminal.
* **Hook : prettier** (120 tokens, badge `hook`) — Un hook `PostToolUse` dans `settings.json` exécute prettier après chaque modification de fichier et fait remonter son retour via `hookSpecificOutput.additionalContext`. Ce champ entre dans le contexte de Claude. La sortie standard brute (stdout) avec un code de sortie 0, elle, n'y entre pas : elle est seulement écrite dans le journal de débogage. En savoir plus : https://code.claude.com/docs/en/hooks-guide
  * 💡 *Économiser du contexte* : produisez du JSON avec `additionalContext` pour envoyer des informations à Claude. Pour les hooks `PostToolUse`, le code de sortie 2 fait remonter stderr comme une erreur mais ne peut pas bloquer, puisque l'outil a déjà été exécuté. Une sortie de plus de 10 000 caractères est enregistrée dans un fichier ; Claude reçoit alors un aperçu et le chemin du fichier.
* **Edit `auth.test.ts`** (600 tokens) — Ajoute un test de non-régression pour la correction. Le diff apparaît dans votre terminal.
* **Hook : prettier** (100 tokens, badge `hook`) — Le même hook se déclenche à nouveau pour le fichier de test. Chaque événement d'outil correspondant le déclenche. En savoir plus : https://code.claude.com/docs/en/hooks-guide
* **Sortie de `npm test`** (1 200 tokens) — Exécute la suite de tests. Vous voyez « Running npm test... » et le nombre de tests réussis, pas les 1 200 tokens de sortie complète.
* **Résumé** (400 tokens) — « Fixed token rotation. Added regression test. All tests pass. » (rotation du jeton corrigée, test de non-régression ajouté, tous les tests passent).

À ce stade, le contexte simulé atteint 19 685 tokens.

### Phase 3 — La question de suivi et le sous-agent

| Position | Élément | Type | Tokens (contexte principal) | Tokens (contexte du sous-agent) | Visibilité |
| :- | :- | :- | :- | :- | :- |
| 72 % | Votre question de suivi | vous | 40 | — | Affiché |
| 79 % | Lancement du sous-agent de recherche | claude | 80 | — | Une ligne |
| 79,5 % | System prompt (sous-agent) | sous-agent | 0 | 900 | Invisible |
| 80 % | `CLAUDE.md` du projet (copie propre) | sous-agent | 0 | 1 800 | Invisible |
| 80,5 % | Outils MCP + skills | sous-agent | 0 | 970 | Invisible |
| 81 % | Prompt de tâche venant de la session principale | sous-agent | 0 | 120 | Invisible |
| 82 % | Read `session.ts` | sous-agent | 0 | 2 200 | Invisible |
| 82,5 % | Read `timeouts.ts` | sous-agent | 0 | 800 | Invisible |
| 83 % | Read `config/*.ts` | sous-agent | 0 | 3 100 | Invisible |
| 85 % | Le sous-agent renvoie son résumé | claude | 420 | — | Une ligne |
| 86 % | Réponse de Claude | claude | 1 200 | — | Affiché |

Détail de chaque élément :

* **Votre question de suivi** (40 tokens) — « Use a subagent to research session timeout handling, then fix it » (utilise un sous-agent pour étudier la gestion de l'expiration de session, puis corrige-la).
  * 💡 *Économiser du contexte* : les questions de suivi s'ajoutent au même contexte. Déléguer la recherche à un sous-agent garde les lectures de fichiers volumineuses hors de votre fenêtre principale.
* **Lancement du sous-agent de recherche** (80 tokens) — Claude délègue la recherche à un sous-agent doté d'une fenêtre de contexte neuve et séparée. Celui-ci charge CLAUDE.md et la même configuration MCP et de skills, mais démarre sans l'historique de votre conversation ni l'auto memory de la session principale. En savoir plus : https://code.claude.com/docs/en/sub-agents
* **System prompt du sous-agent** (900 tokens dans son contexte) — Le sous-agent reçoit son propre system prompt, plus court que celui de la session principale. Pour l'agent généraliste (general-purpose), c'est un prompt bref plus les détails d'environnement. L'auto memory de la session principale n'est pas incluse. Si un agent personnalisé a `memory:` dans son frontmatter, il charge ici son propre `MEMORY.md` séparé à la place. En savoir plus : https://code.claude.com/docs/en/sub-agents#enable-persistent-memory
* **`CLAUDE.md` du projet (copie propre)** (1 800 tokens dans son contexte) — Le sous-agent charge lui aussi CLAUDE.md. Même fichier, même contenu, mais il est décompté du contexte du sous-agent, pas du vôtre. Les agents intégrés Explore et Plan s'en dispensent pour avoir un contexte plus réduit.
* **Outils MCP + skills** (970 tokens dans son contexte) — Le sous-agent a accès aux mêmes serveurs MCP et skills. Il reçoit la plupart des outils du parent, moins plusieurs qui ne s'appliquent pas dans un contexte imbriqué : notamment les contrôles du mode plan, les outils de tâches d'arrière-plan et, par défaut, l'outil Agent lui-même pour éviter la récursion.
* **Prompt de tâche venant de la session principale** (120 tokens dans son contexte) — Au lieu d'un prompt utilisateur, le sous-agent reçoit la tâche que Claude a rédigée pour lui : « Research session timeout handling in this codebase. »
* **Read `session.ts`** (2 200 tokens dans son contexte) — Le sous-agent fait maintenant son travail. Cette lecture de fichier remplit le contexte du sous-agent, pas le vôtre.
* **Read `timeouts.ts`** (800 tokens dans son contexte) — Une autre lecture de fichier dans le contexte séparé du sous-agent.
* **Read `config/*.ts`** (3 100 tokens dans son contexte) — Le sous-agent peut lire autant de fichiers que nécessaire. Rien de tout cela ne touche votre contexte principal.
* **Le sous-agent renvoie son résumé** (420 tokens) — Seule la réponse textuelle finale du sous-agent revient dans votre contexte, plus un petit bloc de métadonnées (trailer) avec le nombre de tokens et la durée. Le sous-agent a lu 6 100 tokens de fichiers ; vous avez reçu un résultat de 420 tokens. C'est là l'économie de contexte.
* **Réponse de Claude** (1 200 tokens) — Analyse et correction de l'expiration des sessions. Ce texte apparaît dans votre terminal.

Au total, 9 890 tokens sont restés dans le contexte du sous-agent ; la simulation l'indique par la mention « ↓ 9.9K tokens stayed in subagent's context · only the summary returns » (9,9K tokens sont restés dans le contexte du sous-agent ; seul le résumé revient). Les liens « En savoir plus » des éléments du sous-agent pointent vers https://code.claude.com/docs/en/sub-agents.

### Phase 4 — Commande `!` et skill invoqué manuellement

| Position | Élément | Type | Tokens | Visibilité | Restauré après `/compact` |
| :- | :- | :- | :- | :- | :- |
| 87,5 % | `!git status` | vous | 180 | Affiché | Non (résumé) |
| 89 % | `/commit-push` | vous | 620 | Une ligne | Oui (corps du skill réinjecté) |

* **`!git status`** (180 tokens) — Vous avez exécuté une commande shell avec le préfixe `!` pour voir quels fichiers Claude a modifiés. La commande et sa sortie entrent toutes deux dans le contexte, comme partie de votre message. Utile pour ancrer Claude dans la sortie d'une commande sans que Claude ne l'exécute lui-même. En savoir plus : https://code.claude.com/docs/en/interactive-mode#shell-mode-with-prefix
* **`/commit-push`** (620 tokens) — Vous avez invoqué un skill qui a `disable-model-invocation: true`. Sa description ne figurait pas dans l'index des skills au démarrage, il n'a donc rien coûté en contexte jusqu'à ce moment. Le contenu complet du skill se charge maintenant et Claude suit ses instructions pour indexer (stage), commiter et pousser vos modifications. Après `/compact`, Claude Code réinjecte le corps de chaque skill que vous avez invoqué, plafonné à 5 000 tokens par skill. En savoir plus : https://code.claude.com/docs/en/skills#control-who-invokes-a-skill
  * 💡 *Économiser du contexte* : définissez `disable-model-invocation: true` sur les skills qui ont des effets de bord, comme commiter, déployer ou envoyer des messages. Ils restent entièrement hors du contexte jusqu'à ce que vous en ayez besoin.

### Phase 5 — `/compact`

* **`/compact`** (93 %, badge `compact`, une ligne dans le terminal) — Remplace la conversation par un résumé structuré. Vous voyez un message « Conversation compacted ». La synthèse se fait sans apparaître dans votre terminal. En savoir plus : https://code.claude.com/docs/en/how-claude-code-works#the-context-window

Ce que la simulation conserve après compaction :

1. **Rechargé après compaction** (en-tête « Reloaded after compact ») : le contenu automatique de démarrage, sauf les descriptions des skills — system prompt, auto memory, informations d'environnement, outils MCP, `~/.claude/CLAUDE.md` et `CLAUDE.md` du projet (7 400 tokens).
2. **Résumé de la conversation** (en-tête « Summarized by /compact », invisible dans le terminal) : un bloc unique dont la taille vaut environ 12 % des tokens résumés (1 725 tokens pour 14 375 tokens d'événements). Sa description : « Les 21 événements de la conversation condensés en un seul résumé structuré. Le résumé conserve : vos demandes et votre intention, les concepts techniques clés, les fichiers examinés ou modifiés avec les extraits de code importants, les erreurs et la manière dont elles ont été corrigées, les tâches en attente et le travail en cours. Il remplace la conversation mot pour mot : les sorties complètes des outils et le raisonnement intermédiaire disparaissent. Claude Code relit les fichiers modifiés le plus récemment et réinjecte les skills que vous avez invoqués, listés ci-dessous. Claude peut encore faire référence au reste du travail mais n'en aura plus le contenu exact. » Les éléments du sous-agent ne sont pas comptés dans ce résumé (ils n'étaient pas dans votre contexte).
3. **Restauré après /compact** (en-tête « Restored after /compact ») : les fichiers relus (`src/api/auth.ts`, `src/lib/tokens.ts`, `middleware.ts`, `auth.test.ts`) et le corps du skill `/commit-push` (7 520 tokens).

Encadré « After /compact » affiché par la simulation : **22.2K → 16.6K tokens · freed 5.6K** (5,6K tokens libérés), avec le texte : « Voici ce qui reste dans le contexte : le contenu de démarrage, qui vit en dehors de l'historique des messages et se recharge après compaction, un résumé structuré de toute la conversation, les fichiers modifiés le plus récemment, et le corps de chaque skill que vous avez invoqué. Les descriptions des skills ne se rechargent pas. »

Dans la vue compactée, la liste remonte en haut, sous cet encadré, et les lignes du résumé et des éléments restaurés apparaissent avec une courte animation ; dans le terminal, vous ne voyez que le message « Conversation compacted » (voir plus bas).

### Types d'événements (badges)

| Badge | Libellé détaillé | Signification |
| :- | :- | :- |
| `auto` | Auto-loaded | Chargé automatiquement (system prompt, CLAUDE.md, mémoire, skills, MCP, règles) |
| `you` | You typed this | Ce que vous avez tapé (prompts, commande `!`, commande slash) |
| `claude` | Claude's work | Le travail de Claude (lectures, recherches, analyses, modifications, réponses) |
| `hook` | Hook (automatic) | Sortie d'un hook déclenché automatiquement |
| `compact` | Compaction | La compaction et le résumé de conversation qui en résulte |
| `subagent` | In subagent's context | Contenu présent uniquement dans le contexte du sous-agent |

### Niveaux de visibilité dans le terminal

| Niveau | Symbole | Libellé | Explication |
| :- | :- | :- | :- |
| `hidden` | ○ | Invisible dans votre terminal | Ce contenu n'apparaît pas dans votre terminal. |
| `brief` | ◐ | Une ligne dans votre terminal | Vous voyez une brève mention, pas le contenu complet. |
| `full` | ● | Affiché dans votre terminal | Le contenu réel apparaît dans votre terminal. |

Dans la liste, une icône d'œil marque les éléments qui apparaissent dans le terminal (verte pour `full`, atténuée pour `brief`, absente pour `hidden`) ; la légende précise « 👁 = appears in your terminal » (= apparaît dans votre terminal).

### Légende des couleurs (catégories)

| Couleur | Catégorie | Éléments concernés |
| :- | :- | :- |
| `#6B6964` | System | System prompt, informations d'environnement, system prompt du sous-agent |
| `#6A9BCC` | CLAUDE.md | `~/.claude/CLAUDE.md`, `CLAUDE.md` du projet (et sa copie dans le sous-agent) |
| `#E8A45C` | Memory | Auto memory (`MEMORY.md`) |
| `#D4A843` | Skills | Descriptions des skills |
| `#9B7BC4` | MCP | Outils MCP (et outils MCP + skills du sous-agent) |
| `#4A9B8E` | Rules | Règles limitées à des chemins (`api-conventions.md`, `testing.md`) |
| `#558A42` | You | Vos prompts, `!git status`, `/commit-push`, prompt de tâche du sous-agent |
| `#8A8880` | Files | Lectures de fichiers |
| `#A09E96` | Output | Sorties d'outils (`grep`, `npm test`) et résumé de conversation |
| `#D97757` | Claude | Analyses, modifications, réponses, lancement et retour du sous-agent, `/compact` |
| `#B8860B` | Hooks | Sorties des hooks (prettier) |

Survoler une catégorie de la légende met en évidence tous les blocs et toutes les lignes de cette couleur.

### Messages « À retenir » (Key takeaway) selon la phase

Le panneau latéral affiche un message clé qui change selon la position dans la chronologie (ou la position de l'événement survolé) :

| Phase | Message à retenir |
| :- | :- |
| Démarrage (avant 20 %) | Beaucoup de choses se chargent avant que vous ne tapiez quoi que ce soit. CLAUDE.md, la mémoire, les skills et les outils MCP sont tous dans le contexte avant votre premier prompt. |
| Premier prompt (20–28 %) | Votre prompt est minuscule comparé à ce qui est déjà chargé. L'essentiel du contexte de Claude est de la connaissance du projet, pas vos mots. |
| Lectures (28–50 %) | Chaque fichier lu par Claude fait grossir le contexte. Les règles limitées à des chemins se chargent automatiquement avec les fichiers correspondants. |
| Modifications et hooks (50–71 %) | Les hooks se déclenchent automatiquement sur les événements d'outils. Leur sortie atteint Claude via le JSON `additionalContext`. Le code de sortie 2 fait remonter stderr à Claude. La sortie stdout brute avec le code 0 va dans le journal de débogage, pas dans la transcription. |
| Question de suivi (71–79 %) | Les questions de suivi continuent de s'appuyer sur le même contexte. Tout ce qui précède est toujours là. |
| Sous-agent (79–87 %) | Le sous-agent travaille dans sa propre fenêtre de contexte séparée. Aucune de ses lectures de fichiers ne touche la vôtre. Seul le résumé final revient. |
| Commande `!` (87–88 %) | Les commandes `!` (bang) s'exécutent dans votre shell et ajoutent leur sortie en tête de votre message suivant. Utile pour ancrer Claude dans des résultats de commande sans qu'il les exécute. |
| Skill manuel (88–90 %) | Les skills réservés à l'utilisateur restent entièrement hors du contexte jusqu'à ce que vous les invoquiez. L'index des skills au démarrage ne liste que les skills que Claude peut appeler de lui-même. |
| Avant `/compact` (90 % et plus) | `/compact` résume la conversation pour libérer de l'espace tout en conservant les informations clés. Dans une vraie session, exécutez-le lorsque le contexte commence à affecter les performances ou avant une longue nouvelle tâche. |
| Après compaction | La compaction remplace la conversation par un résumé structuré. Le system prompt, CLAUDE.md, la mémoire et les outils MCP se rechargent automatiquement. Claude Code relit aussi jusqu'à cinq des fichiers modifiés le plus récemment et réinjecte les skills que vous avez invoqués. La liste des skills ne se recharge pas. |

### « Dans votre terminal, vous voyez » selon la phase

| Phase | Ce que vous voyez dans votre terminal |
| :- | :- |
| Démarrage (avant 20 %) | La zone de saisie, qui attend votre premier message. Tout ce qui précède se charge silencieusement avant que vous ne tapiez quoi que ce soit. |
| Premier prompt (20–28 %) | Votre prompt. Claude n'a pas encore commencé à travailler. |
| Lectures (28–52 %) | Votre prompt et « Reading files... ». Les règles s'affichent sous forme d'avis « Loaded » d'une ligne, pas leur contenu. |
| Modifications et hooks (52–72 %) | La réponse de Claude et les diffs des fichiers. Les hooks se déclenchent silencieusement. Une sortie d'outil comme `npm test` s'affiche sous forme de bref résumé, pas de contenu complet. |
| Question de suivi (72–79 %) | Votre prompt de suivi. |
| Sous-agent (79–86 %) | Un bref avis indiquant qu'un sous-agent travaille, puis son résultat. Vous ne voyez pas les lectures de fichiers individuelles du sous-agent. |
| Fin de session (86–90 %) | La réponse de Claude, la sortie de votre `git status` et l'exécution du skill commit-push. |
| Avant `/compact` (90 % et plus) | Toute votre conversation. `/compact` est disponible. |
| Après compaction | Un message « Conversation compacted », puis une ligne « Read auth.ts » pour chaque fichier relu et « Skills restored (commit-push) ». Aucun contenu lui-même n'apparaît. |

### Interface et commandes de la simulation

* **Écran d'accueil** : une invite `$ claude` avec curseur clignotant, un bouton **▶ Start session** et le texte « Observez ce qui se charge dans le contexte, du moment où vous lancez `claude` jusqu'à une conversation complète. »
* **En-têtes de phase dans la liste** : « Before you type anything » (avant que vous ne tapiez quoi que ce soit), « You » (vous), « Claude works » (Claude travaille), « Subagent's separate context window » (fenêtre de contexte séparée du sous-agent, avec retrait et filet violet), « Summarized by /compact », « Restored after /compact », « Reloaded after compact ».
* **Chaque ligne** affiche : badge de type, libellé, `+N` tokens (en violet pour les tokens du sous-agent), une mini-barre proportionnelle (pleine à 5 000 tokens) et l'icône de visibilité.
* **Zone de saisie simulée** : « You type in your terminal » (vous tapez dans votre terminal), avec le texte du point d'arrêt et le bouton **Send ↵** (prompt) ou **Run ↵** (commande `!`, commande slash, `/compact`).
* **Panneau de détail** : survoler un événement en affiche l'aperçu, cliquer l'épingle pour pouvoir faire défiler (message par défaut : « Hover or click any event — Hover to preview. Click to pin so you can scroll. »). Il montre le libellé, le type détaillé, le nombre de tokens (ou « N tokens in the subagent's context » pour le sous-agent), la description, le niveau de visibilité, l'éventuel conseil **💡 Save context** (économiser du contexte) et un lien **Learn more →**. Viennent ensuite les encadrés **Key takeaway** et **In your terminal you see** décrits plus haut.
* **Barre de commandes** : bouton lecture/pause/continuer/recommencer (▶, ⏸, ↺), barre de progression avec pourcentage, bouton plein écran (⛶ / ⤡).
* **Clavier** : la touche Espace (une fois que vous avez cliqué dans la simulation et qu'elle est visible à l'écran) démarre la lecture, envoie la saisie à un point d'arrêt, recommence à la fin, ou bascule lecture/pause. Elle est ignorée lorsque le focus est dans un champ de saisie, un bouton, une zone de texte, une liste déroulante ou un élément éditable.
* **Petits écrans** : sous 700 px de large, la simulation est masquée et remplacée par le message « Cette chronologie interactive fonctionne mieux sur un écran plus grand. Voir la description écrite ci-dessous pour les mêmes concepts. »

## Ce que montre la chronologie

La session parcourt un déroulé réaliste avec des nombres de tokens représentatifs :

* **Avant que vous ne tapiez quoi que ce soit** : CLAUDE.md, l'auto memory, les noms des outils MCP et les descriptions des skills se chargent tous dans le contexte. Des [fichiers AGENTS.md](https://code.claude.com/docs/en/memory#agents-md) peuvent se charger à la place de CLAUDE.md. Votre propre configuration peut ajouter d'autres éléments ici, comme un [style de sortie (output style)](https://code.claude.com/docs/en/output-styles) ou du texte provenant de [`--append-system-prompt`](https://code.claude.com/docs/en/cli-reference).
* **Pendant que Claude travaille** : chaque lecture de fichier s'ajoute au contexte, les [règles limitées à des chemins](https://code.claude.com/docs/en/memory#path-specific-rules) se chargent automatiquement avec les fichiers correspondants, et un [hook PostToolUse](https://code.claude.com/docs/en/hooks-guide) se déclenche après chaque modification.
* **Le prompt de suivi** : un [sous-agent](https://code.claude.com/docs/en/sub-agents) traite la recherche dans sa propre fenêtre de contexte séparée, de sorte que les lectures de fichiers volumineuses restent hors de la vôtre. Seuls le résumé et un petit bloc de métadonnées reviennent.
* **À la fin du parcours** : vous exécutez `/compact`, qui remplace la conversation par un résumé structuré. La majeure partie du contenu de démarrage se recharge automatiquement ; le tableau ci-dessous montre ce qu'il advient de chaque mécanisme.

## Ce qui survit à la compaction

Lorsqu'une longue session est compactée, Claude Code résume l'historique de la conversation pour qu'il tienne dans la fenêtre de contexte. Depuis la v2.1.198, la requête de synthèse hérite de la configuration de [réflexion étendue (extended thinking)](https://code.claude.com/docs/en/model-config#extended-thinking) de votre session : elle raisonne avec la réflexion activée lorsque votre session l'a activée, et reste sans réflexion sinon. La réflexion n'affecte que la manière dont le résumé est produit ; les réglages de votre session restent inchangés ensuite. Ce qu'il advient de chaque type de contenu dépend de la façon dont il a été chargé :

| Mécanisme | Après compaction |
| :- | :- |
| System prompt et style de sortie | Les deux s'appliquent toujours |
| CLAUDE.md à la racine du projet et règles sans portée (unscoped) | Réinjectés depuis le disque |
| Auto memory | Réinjectée depuis le disque |
| [Instantané du statut git](https://code.claude.com/docs/en/settings-reference#includegitinstructions) | Claude Code en lit un nouveau depuis votre dépôt |
| Le plan rédigé par Claude en [mode plan](https://code.claude.com/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode) | Réinjecté depuis le disque |
| Règles avec un frontmatter `paths:` | Claude Code les recharge [à la demande](https://code.claude.com/docs/en/memory#path-specific-rules) |
| CLAUDE.md imbriqués dans des sous-répertoires | Claude Code les recharge [à la demande](https://code.claude.com/docs/en/memory#how-claude-md-files-load) |
| Fichiers lus ou modifiés par Claude | Claude Code en relit jusqu'à cinq, les plus récemment modifiés d'abord |
| Corps des skills invoqués | Réinjectés, plafonnés à 5 000 tokens par skill et 25 000 tokens au total ; les plus anciens sont abandonnés en premier |
| [Commandes d'arrière-plan](https://code.claude.com/docs/en/interactive-mode#background-bash-commands) et [sous-agents](https://code.claude.com/docs/en/sub-agents#run-subagents-in-foreground-or-background) d'arrière-plan | Continuent de tourner. Claude Code rappelle à Claude lesquels sont encore en cours pour qu'il n'en démarre pas un doublon |
| Contexte ajouté précédemment par des hooks | Résumé avec le reste de la conversation |
| [Hooks SessionStart](https://code.claude.com/docs/en/hooks-guide#re-inject-context-after-compaction) qui correspondent à la source `compact` | Claude Code les exécute et ajoute leur sortie au contexte compacté |

Juste après la compaction, Claude Code relit jusqu'à cinq des fichiers que Claude a lus ou modifiés pendant la session, en choisissant ceux modifiés le plus récemment. Un fichier de plus de 5 000 tokens revient sous forme de référence de chemin sans son contenu, affiché `Referenced file` au lieu de `Read`.

Les règles limitées à des chemins et les fichiers CLAUDE.md imbriqués se chargent dans l'historique des messages lorsque Claude lit, écrit ou modifie leur fichier déclencheur ; la compaction les fait donc disparaître dans le résumé avec tout le reste. Si une règle doit persister après la compaction, supprimez son frontmatter `paths:` ou déplacez-la dans le CLAUDE.md à la racine du projet.

Les corps des skills sont réinjectés après la compaction, mais les skills volumineux sont tronqués pour respecter le plafond par skill, et les skills invoqués les plus anciens sont abandonnés une fois le budget total dépassé. La troncature conserve le début du fichier : placez donc les instructions les plus importantes en haut de `SKILL.md`.

## Quand votre contexte se remplit

Claude Code compacte automatiquement à l'approche de la limite, si bien qu'une fenêtre de contexte pleine ne met pas fin à votre session. Le passage automatique fonctionne de la même façon que l'étape `/compact` de la chronologie. Voir [Quand le contexte se remplit](https://code.claude.com/docs/en/how-claude-code-works#when-context-fills-up) pour ce qu'il préserve. Pour savoir quand Claude Code compacte aussi une conversation que vous laissez inactive, voir la [compaction en cas d'inactivité (idle compaction)](https://code.claude.com/docs/en/settings-reference#idlecompaction).

Vous pouvez aussi agir avant que le passage automatique ne s'exécute :

* **Compacter avec un objectif** : exécutez `/compact` avec des instructions, comme `/compact focus on the auth bug fix`, avant de commencer une longue nouvelle tâche. Le résumé conserve ce que vous choisissez au lieu de ce que le passage automatique devine être important.
* **Compacter une partie de la conversation** : exécutez `/rewind`, sélectionnez un message et choisissez **Summarize from here** (résumer à partir d'ici) ou **Summarize up to here** (résumer jusqu'ici). Voir [Rewind and summarize](https://code.claude.com/docs/en/checkpointing#rewind-and-summarize) pour ce que conserve chaque option et comment orienter le résumé.
* **Compacter plus tôt** : exécutez [`/autocompact`](https://code.claude.com/docs/en/commands#all-commands) avec un nombre de tokens, comme `/autocompact 500k`, pour définir le niveau de remplissage de la fenêtre de contexte à partir duquel le passage automatique s'exécute. Voir [Set the auto-compact window](https://code.claude.com/docs/en/model-config#set-the-auto-compact-window) pour les valeurs acceptées et les surcharges.
* **Effacer entre les tâches** : exécutez `/clear` lorsque vous passez à un travail sans rapport. L'ancienne conversation évince les fichiers dont vous aurez besoin ensuite et coûte des tokens à chaque message.
* **Déléguer les lectures volumineuses** : confiez la recherche à un [sous-agent](https://code.claude.com/docs/en/sub-agents) pour que le contenu des fichiers reste dans sa fenêtre de contexte, pas dans la vôtre.

Si vous avez besoin d'une fenêtre plus grande plutôt que d'une conversation plus petite : les modèles Fable, Sonnet 5 et ultérieurs, Haiku 5.5, Opus 4.6 et ultérieurs, et Sonnet 4.6 prennent en charge une fenêtre de contexte d'un million de tokens. Voir [Extended context](https://code.claude.com/docs/en/model-config#extended-context) pour la disponibilité selon le forfait et la manière de sélectionner une variante de modèle `[1m]`. La compaction fonctionne de la même façon avec cette limite plus élevée.

Le point auquel la compaction automatique s'exécute dépend de votre modèle et de votre configuration. Voir [Default auto-compact thresholds](https://code.claude.com/docs/en/model-config#default-auto-compact-thresholds) pour les seuils par modèle, et [Correct the window for a gateway or custom model ID](https://code.claude.com/docs/en/model-config#correct-the-window-for-a-gateway-or-custom-model-id) si Claude Code suppose une mauvaise fenêtre pour votre identifiant de modèle, par exemple un alias de [passerelle LLM (LLM gateway)](https://code.claude.com/docs/en/llm-gateway).

## Vérifier votre propre session

La visualisation utilise des nombres représentatifs. Pour voir votre utilisation réelle du contexte à tout moment, exécutez `/context` : vous obtenez une répartition en direct par catégorie avec des suggestions d'optimisation, y compris les fichiers CLAUDE.md et d'auto memory qui ont été chargés. Exécutez `/memory` pour ouvrir et modifier ces fichiers.

## Ressources associées

Pour une couverture plus approfondie des fonctionnalités montrées dans la chronologie, voir ces pages :

* [Extend Claude Code](https://code.claude.com/docs/en/features-overview) : quand utiliser CLAUDE.md, les skills, les règles, les hooks ou MCP
* [Store instructions and memories](https://code.claude.com/docs/en/memory) : la hiérarchie des CLAUDE.md et l'auto memory
* [Subagents](https://code.claude.com/docs/en/sub-agents) : déléguer la recherche à une fenêtre de contexte séparée
* [Best practices](https://code.claude.com/docs/en/best-practices) : gérer le contexte comme votre contrainte principale
* [Prompt caching](https://code.claude.com/docs/en/prompt-caching) : quelles actions invalident le préfixe mis en cache
* [Reduce token usage](https://code.claude.com/docs/en/costs#reduce-token-usage) : stratégies pour limiter l'utilisation du contexte
