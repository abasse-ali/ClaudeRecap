# Comment Claude mémorise votre projet

> Source : https://code.claude.com/docs/en/memory · [← Index](../31-index-documentation-officielle.md)

*Donner à Claude des instructions persistantes avec des fichiers CLAUDE.md ou AGENTS.md, et laisser Claude accumuler automatiquement des apprentissages grâce à l'auto memory (mémoire automatique).*

Chaque session Claude Code démarre avec une fenêtre de contexte vierge. Deux mécanismes transportent les connaissances d'une session à l'autre :

* **Fichiers CLAUDE.md** : des instructions que vous écrivez pour donner à Claude un contexte persistant. Claude peut aussi lire les [fichiers `AGENTS.md`](#agentsmd) d'un dépôt à la place de CLAUDE.md
* **Auto memory** : des notes que Claude écrit lui-même à partir de vos corrections et de vos préférences

Cette page explique comment :

* [Écrire et organiser des fichiers CLAUDE.md](#fichiers-claudemd)
* [Utiliser un `AGENTS.md` existant](#agentsmd) comme instructions de projet
* [Restreindre des règles à certains types de fichiers](#organiser-les-règles-avec-clauderules) avec `.claude/rules/`
* [Configurer l'auto memory](#auto-memory) pour que Claude prenne des notes automatiquement
* [Dépanner](#dépanner-les-problèmes-de-mémoire) lorsque les instructions ne sont pas suivies

## CLAUDE.md ou auto memory

Claude Code dispose de deux systèmes de mémoire complémentaires. Les deux sont chargés au début de chaque conversation. Claude les traite comme du contexte, pas comme une configuration imposée. Pour bloquer une action quelle que soit la décision de Claude, utilisez plutôt un [hook PreToolUse](https://code.claude.com/docs/en/hooks-guide). Plus vos instructions sont précises et concises, plus Claude les suit de façon cohérente.

| | Fichiers CLAUDE.md | Auto memory |
| :- | :- | :- |
| **Qui l'écrit** | Vous | Claude |
| **Ce qu'il contient** | Instructions et règles | Apprentissages et schémas récurrents |
| **Portée** | Projet, utilisateur ou organisation | Par dépôt, partagée entre les worktrees |
| **Chargé dans** | Chaque session | Chaque session (200 premières lignes ou 25 Ko) |
| **À utiliser pour** | Standards de code, workflows, architecture du projet | Vos préférences, les corrections que vous donnez à Claude, le contexte du projet que Claude ne peut pas déduire du code |

Utilisez les fichiers CLAUDE.md pour guider le comportement de Claude. L'auto memory permet à Claude d'apprendre de vos corrections sans effort manuel.

Les subagents peuvent aussi tenir leur propre auto memory. Voir la [configuration des subagents](https://code.claude.com/docs/en/sub-agents#enable-persistent-memory) pour les détails.

## Fichiers CLAUDE.md

Les fichiers CLAUDE.md sont des fichiers markdown qui donnent à Claude des instructions persistantes pour un projet, votre workflow personnel ou toute votre organisation. Vous les écrivez en texte brut ; Claude les lit au début de chaque session. Si votre dépôt utilise plutôt `AGENTS.md`, voir [AGENTS.md](#agentsmd).

### Quand ajouter quelque chose à CLAUDE.md

Considérez CLAUDE.md comme l'endroit où noter ce que vous auriez sinon à réexpliquer. Ajoutez-y quelque chose quand :

* Claude commet la même erreur une deuxième fois
* Une revue de code relève quelque chose que Claude aurait dû savoir sur cette base de code
* Vous tapez dans le chat la même correction ou précision que lors de la session précédente
* Un nouveau coéquipier aurait besoin du même contexte pour être productif

Limitez-le aux faits que Claude doit garder en tête à chaque session : commandes de build, conventions, organisation du projet, règles du type « toujours faire X ». Si une entrée est une procédure en plusieurs étapes ou ne concerne qu'une partie de la base de code, déplacez-la plutôt dans un [skill](https://code.claude.com/docs/en/skills) ou une [règle restreinte à des chemins](#organiser-les-règles-avec-clauderules). La [vue d'ensemble des extensions](https://code.claude.com/docs/en/features-overview#build-your-setup-over-time) explique quand utiliser chaque mécanisme.

### Choisir où placer les fichiers CLAUDE.md

Les fichiers CLAUDE.md peuvent se trouver à plusieurs emplacements, chacun avec une portée différente. Le tableau ci-dessous les liste dans l'ordre de chargement, de la portée la plus large à la plus spécifique : une instruction de projet apparaît donc dans le contexte après une instruction utilisateur.

| Portée | Emplacement | Rôle | Exemples d'usage | Partagé avec |
| - | - | - | - | - |
| **Politique gérée (managed policy)** | • macOS : `/Library/Application Support/ClaudeCode/CLAUDE.md`<br />• Linux et WSL : `/etc/claude-code/CLAUDE.md`<br />• Windows : `C:\Program Files\ClaudeCode\CLAUDE.md` | Instructions à l'échelle de l'organisation gérées par l'IT/DevOps | Standards de code de l'entreprise, politiques de sécurité, exigences de conformité | Tous les utilisateurs de l'organisation |
| **Instructions utilisateur** | `~/.claude/CLAUDE.md` | Préférences personnelles pour tous les projets | Préférences de style de code, raccourcis d'outillage personnels | Vous seul (tous les projets) |
| **Instructions de projet** | `./CLAUDE.md` ou `./.claude/CLAUDE.md`. Voir [AGENTS.md](#agentsmd) pour savoir quand `./AGENTS.md` est chargé à leur place ou en plus | Instructions partagées par l'équipe pour le projet | Architecture du projet, standards de code, workflows courants | Membres de l'équipe via le contrôle de version |
| **Instructions locales** | `./CLAUDE.local.md` | Préférences personnelles propres au projet ; à ajouter au `.gitignore` | Vos URL de sandbox, vos données de test préférées | Vous seul (projet courant) |

Les fichiers CLAUDE.md et CLAUDE.local.md situés dans la hiérarchie de répertoires au-dessus du répertoire de travail sont chargés au lancement. Ceux des sous-répertoires se chargent à la demande. Voir [Comment se chargent les fichiers CLAUDE.md](#comment-se-chargent-les-fichiers-claudemd) pour le moment du chargement et l'ordre de résolution complet.

Pour les gros projets, vous pouvez découper les instructions en fichiers thématiques grâce aux [règles de projet](#organiser-les-règles-avec-clauderules). Les règles permettent de restreindre des instructions à certains types de fichiers ou sous-répertoires.

### Mettre en place un CLAUDE.md de projet

Un CLAUDE.md de projet peut être stocké dans `./CLAUDE.md` ou dans `./.claude/CLAUDE.md`. Créez ce fichier et ajoutez-y les instructions qui s'appliquent à toute personne travaillant sur le projet : commandes de build et de test, standards de code, décisions d'architecture, conventions de nommage et workflows courants. Ces instructions sont partagées avec votre équipe via le contrôle de version : concentrez-vous donc sur les standards du projet plutôt que sur des préférences personnelles. Pour confirmer que le fichier est chargé, lancez `/context` dans une session et regardez la liste sous **Memory files**.

> **Astuce** : lancez `/init` pour générer automatiquement un CLAUDE.md de départ. Claude analyse votre base de code et crée un fichier contenant les commandes de build, les instructions de test et les conventions du projet qu'il découvre. Si un CLAUDE.md existe déjà, `/init` propose des améliorations au lieu de l'écraser. Affinez ensuite avec des instructions que Claude ne découvrirait pas seul.
>
> Pour un déroulé interactif en plusieurs phases, définissez la variable d'environnement `CLAUDE_CODE_NEW_INIT` à `1` avant de lancer `/init`. Définissez-la dans votre shell ou dans le bloc `env` d'un fichier de réglages, comme indiqué dans [Définir des variables d'environnement](https://code.claude.com/docs/en/env-vars#set-environment-variables). Avec cette variable, `/init` demande quels artefacts mettre en place : fichiers CLAUDE.md, skills et hooks. Il explore ensuite votre base de code avec un subagent, comble les lacunes par des questions complémentaires et présente une proposition à relire avant d'écrire le moindre fichier. La variable ne change que le déroulement de `/init`, vous pouvez donc la laisser définie.

### Écrire des instructions efficaces

Claude traite les fichiers CLAUDE.md comme du contexte, pas comme une configuration imposée : la façon dont vous rédigez les instructions influe donc sur la fiabilité avec laquelle Claude les suit. Écrivez des instructions assez concrètes pour être vérifiables :

* « Utiliser une indentation de 2 espaces » plutôt que « Bien formater le code »
* « Lancer `npm test` avant de committer » plutôt que « Tester vos modifications »
* « Les handlers d'API se trouvent dans `src/api/handlers/` » plutôt que « Garder les fichiers organisés »

Gardez vos fichiers courts, organisés et cohérents :

* **Taille** : visez moins de 200 lignes par fichier CLAUDE.md. Les fichiers plus longs consomment davantage de contexte et réduisent l'adhésion. Déplacez les instructions qui ne concernent qu'une partie de la base de code dans des [règles restreintes à des chemins](#règles-propres-à-des-chemins), qui ne se chargent que lorsque Claude travaille sur des fichiers correspondants. Les [imports](#importer-des-fichiers-supplémentaires) aident à organiser un long fichier mais n'en réduisent pas le coût en contexte, car les fichiers importés se chargent aussi au lancement.
* **Structure** : regroupez les instructions liées sous des titres markdown et des puces. Des sections organisées sont plus faciles à suivre pour Claude que des paragraphes denses.
* **Cohérence** : si deux instructions se contredisent, Claude peut en choisir une arbitrairement. Relisez périodiquement vos fichiers CLAUDE.md, les CLAUDE.md imbriqués dans les sous-répertoires et [`.claude/rules/`](#organiser-les-règles-avec-clauderules) pour retirer les instructions obsolètes ou contradictoires. Pour que Claude les trouve à votre place, [lancez un audit des prompts](#auditer-vos-fichiers-dinstructions).

#### Auditer vos fichiers d'instructions

Pour que Claude vérifie si vos fichiers d'instructions contiennent du contenu obsolète ou contradictoire, lancez `/doctor prompt-audit` dans une session. Claude cherche des problèmes tels que des instructions écrites pour d'anciens modèles, des références à des fichiers ou des commandes qui n'existent pas, et des fichiers qui se contredisent. Vous obtenez un rapport des constats avec des modifications proposées, et rien ne change dans vos fichiers tant que vous ne demandez pas à Claude de les appliquer.

Par défaut, l'audit couvre vos fichiers CLAUDE.md, CLAUDE.local.md et AGENTS.md, ainsi que les règles, skills, commandes, subagents et styles de sortie sous `.claude/` et `~/.claude/`. Pour n'auditer qu'un fichier ou un répertoire, passez son chemin, par exemple `/doctor prompt-audit .claude/skills/deploy`.

L'audit passe par le skill intégré `/claude-api`. Il est indisponible tant que ce skill est désactivé dans [`skillOverrides`](https://code.claude.com/docs/en/skills#override-skill-visibility-from-settings) ou avec [`disableBundledSkills`](https://code.claude.com/docs/en/settings-reference#disablebundledskills). `/doctor prompt-audit` nécessite Claude Code v2.1.283+.

### Importer des fichiers supplémentaires

Les fichiers CLAUDE.md peuvent importer d'autres fichiers avec la syntaxe `@chemin/vers/import`. Les fichiers importés sont développés et chargés dans le contexte au lancement, en même temps que le CLAUDE.md qui les référence.

Les chemins relatifs comme absolus sont autorisés. Les chemins relatifs sont résolus par rapport au fichier qui contient l'import, pas par rapport au répertoire de travail. Les fichiers importés peuvent à leur tour importer d'autres fichiers, avec une profondeur maximale de quatre sauts.

Pour importer un fichier dont le chemin contient des espaces, placez une barre oblique inverse avant chaque espace. Sans elles, le chemin s'arrête au premier espace, même si l'import est seul sur sa ligne. Un chemin entouré de guillemets n'est pas importé du tout, avec ou sans barres obliques inverses. Cet import charge un fichier depuis un dossier nommé `Design Docs` :

```text theme={null}
- API conventions @Design\ Docs/api-conventions.md
```

L'analyse des imports ignore les spans de code Markdown et les blocs de code délimités. Pour mentionner un chemin dans votre CLAUDE.md sans l'importer, entourez-le d'accents graves : écrire `` `@README` `` garde le texte littéral, alors que `@README` hors accents graves importe le fichier.

Pour inclure un README, un package.json et un guide de workflow, référencez-les avec la syntaxe `@` n'importe où dans votre CLAUDE.md :

```text theme={null}
See @README for project overview and @package.json for available npm commands for this project.

# Additional Instructions
- git workflow @docs/git-instructions.md
```

Pour des préférences privées propres à un projet qui ne doivent pas être versionnées, créez un `CLAUDE.local.md` à la racine du projet. Il se charge en même temps que `CLAUDE.md` et est traité de la même manière. Ajoutez `CLAUDE.local.md` à votre `.gitignore` pour qu'il ne soit pas committé. Avec `CLAUDE_CODE_NEW_INIT=1` défini, lancer `/init` et choisir l'option personnelle le fait pour vous.

Si vous travaillez avec plusieurs worktrees git du même dépôt, un `CLAUDE.local.md` ignoré par git n'existe que dans le worktree où vous l'avez créé. Pour partager des instructions personnelles entre worktrees, importez plutôt un fichier de votre répertoire personnel :

```text theme={null}
# Individual Preferences
- @~/.claude/my-project-instructions.md
```

> **Avertissement** : un import dans un fichier mémoire de niveau projet est externe lorsque son chemin se résout en dehors de votre répertoire de travail, comme l'import depuis le répertoire personnel ci-dessus. La première fois que Claude Code rencontre des imports externes dans un projet, il affiche une boîte de dialogue d'approbation qui liste les fichiers. Si vous refusez, les imports restent désactivés et la boîte de dialogue ne réapparaît plus.
>
> Claude Code affiche cette boîte de dialogue pour vous protéger des fichiers que d'autres personnes committent dans un projet partagé. Les fichiers mémoire de portée utilisateur, comme `~/.claude/CLAUDE.md` et `~/.claude/rules/`, sont des fichiers que vous avez écrits vous-même. Sauf dans les sessions [Cowork](https://claude.com/product/cowork) sur votre bureau, Claude Code charge leurs imports sans boîte de dialogue et leur fait confiance comme au reste de votre configuration personnelle.
>
> Dans les sessions Cowork sur votre bureau, Claude Code ignore tout import d'un fichier de portée utilisateur qui se résout vers un chemin hors du répertoire de travail de la session, et charge le reste du fichier. Dans ces sessions, il ignore aussi un `~/.claude/CLAUDE.md` qui est lui-même un lien symbolique ou un lien physique, ainsi qu'un répertoire `~/.claude/rules/` ou un fichier de règle en lien symbolique pointant hors du répertoire de travail.

### Comment se chargent les fichiers CLAUDE.md

Claude Code charge `CLAUDE.md` et `CLAUDE.local.md` depuis votre répertoire de travail courant et depuis chaque répertoire au-dessus. Lancez Claude Code dans `foo/bar/` et il charge les instructions de `foo/bar/CLAUDE.md`, de `foo/CLAUDE.md` et de tous les fichiers `CLAUDE.local.md` situés à côté.

Tous les fichiers découverts sont concaténés dans le contexte au lieu de se remplacer. À travers l'arborescence, le contenu est ordonné de la racine du système de fichiers jusqu'à votre répertoire de travail. Dans l'exemple `foo/bar/`, `foo/CLAUDE.md` apparaît dans le contexte avant `foo/bar/CLAUDE.md` : les instructions les plus proches de l'endroit où vous avez lancé Claude sont donc lues en dernier. Dans chaque répertoire, `CLAUDE.local.md` est ajouté après `CLAUDE.md`, si bien que vos notes personnelles sont la dernière chose que Claude lit à ce niveau.

Claude découvre aussi les fichiers `CLAUDE.md` et `CLAUDE.local.md` des sous-répertoires de votre répertoire de travail courant. Au lieu de les charger au lancement, Claude Code charge chacun d'eux dès que Claude lit, écrit ou modifie un autre fichier dans ce sous-répertoire. La lecture inclut l'affichage du fichier avec une commande Bash qui [compte comme une lecture](https://code.claude.com/docs/en/tools-reference#edit-tool-behavior), comme `cat` ou `head` sur un seul fichier. Pour les fichiers situés dans un worktree sous `.claude/worktrees/`, voir [Isoler des subagents avec des worktrees](https://code.claude.com/docs/en/worktrees#isolate-subagents-with-worktrees).

Si vous travaillez dans un gros monorepo où les CLAUDE.md d'autres équipes sont pris en compte, utilisez [`claudeMdExcludes`](#exclure-certains-fichiers-claudemd) pour les ignorer. Pour l'organisation complète des CLAUDE.md racine et par répertoire et des règles, voir [Monorepos et gros dépôts](https://code.claude.com/docs/en/large-codebases).

Les commentaires HTML de niveau bloc (`<!-- maintainer notes -->`) dans les fichiers CLAUDE.md sont retirés avant que le contenu soit injecté dans le contexte de Claude. Utilisez-les pour laisser des notes aux mainteneurs humains sans dépenser de tokens de contexte. Les commentaires à l'intérieur des blocs de code sont conservés. Quand vous ouvrez directement un fichier CLAUDE.md avec l'outil Read, les commentaires restent visibles.

#### Charger depuis des répertoires supplémentaires

Le flag `--add-dir` donne à Claude accès à des répertoires supplémentaires hors de votre répertoire de travail principal. Par défaut, les fichiers CLAUDE.md de ces répertoires ne sont pas chargés.

Pour charger aussi les fichiers mémoire des répertoires supplémentaires, définissez la variable d'environnement `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` :

```bash theme={null}
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1 claude --add-dir ../shared-config
```

La forme en ligne définit la variable pour ce seul lancement sous Bash ou Zsh. Pour la garder active dans toutes les sessions, ajoutez-la au bloc `env` de `~/.claude/settings.json` comme indiqué dans [Définir des variables d'environnement](https://code.claude.com/docs/en/env-vars#set-environment-variables).

Cela charge `CLAUDE.md`, `.claude/CLAUDE.md`, `.claude/rules/*.md` et `CLAUDE.local.md` depuis le répertoire supplémentaire. `CLAUDE.local.md` est ignoré si vous excluez `local` de [`--setting-sources`](https://code.claude.com/docs/en/cli-reference).

### Organiser les règles avec `.claude/rules/`

Pour les projets plus importants, vous pouvez organiser les instructions en plusieurs fichiers dans le répertoire `.claude/rules/`. Les instructions restent ainsi modulaires et plus faciles à maintenir pour les équipes. Les règles peuvent aussi être [restreintes à des chemins de fichiers](#règles-propres-à-des-chemins), de sorte qu'elles ne se chargent dans le contexte que lorsque Claude travaille sur des fichiers correspondants, ce qui réduit le bruit et économise de l'espace de contexte.

> **Note** : les règles se chargent dans le contexte à chaque session ou lorsque des fichiers correspondants sont ouverts. Pour des instructions propres à une tâche qui n'ont pas besoin d'être en permanence dans le contexte, utilisez plutôt des [skills](https://code.claude.com/docs/en/skills), qui ne se chargent que lorsque vous les invoquez ou que Claude juge qu'ils sont pertinents pour votre prompt.

#### Mettre en place des règles

Placez des fichiers markdown dans le répertoire `.claude/rules/` de votre projet. Chaque fichier doit traiter d'un seul sujet et porter un nom descriptif comme `testing.md` ou `api-design.md`. Tous les fichiers `.md` sont découverts récursivement, vous pouvez donc organiser les règles en sous-répertoires comme `frontend/` ou `backend/` :

```text theme={null}
your-project/
├── .claude/
│   ├── CLAUDE.md           # Instructions principales du projet
│   └── rules/
│       ├── code-style.md   # Règles de style de code
│       ├── testing.md      # Conventions de test
│       └── security.md     # Exigences de sécurité
```

Les règles sans [frontmatter `paths`](#règles-propres-à-des-chemins) sont chargées au lancement avec la même priorité que `.claude/CLAUDE.md`.

Les règles de projet sont ignorées si vous excluez `project` de [`--setting-sources`](https://code.claude.com/docs/en/cli-reference). Avant la v2.1.211, les règles chargées à la demande, y compris les règles restreintes à des chemins et celles des répertoires `.claude/rules/` imbriqués, se chargeaient même lorsque `project` était exclu.

#### Règles propres à des chemins

Les règles peuvent être restreintes à certains fichiers grâce à un frontmatter YAML avec le champ `paths`. Ces règles conditionnelles ne s'appliquent que lorsque Claude travaille sur des fichiers correspondant aux motifs indiqués.

```markdown theme={null}
---
paths:
  - "src/api/**/*.ts"
---

# API Development Rules

- All API endpoints must include input validation
- Use the standard error response format
- Include OpenAPI documentation comments
```

(Exemple : toutes les routes d'API doivent valider leurs entrées, utiliser le format standard de réponse d'erreur et inclure des commentaires de documentation OpenAPI.)

Les règles sans champ `paths` sont chargées sans condition et s'appliquent à tous les fichiers. Une règle restreinte à des chemins se charge lorsque Claude utilise l'outil Read, Write ou Edit sur un fichier correspondant. Elle se charge aussi lorsque Claude affiche un fichier correspondant avec une commande Bash qui [compte comme une lecture](https://code.claude.com/docs/en/tools-reference#edit-tool-behavior), comme `cat` ou `head` sur un seul fichier. La correspondance fonctionne aussi lorsque Claude atteint un fichier via un chemin en lien symbolique vers le répertoire du projet, par exemple dans un checkout en lien symbolique.

Utilisez des motifs glob dans le champ `paths` pour faire correspondre des fichiers par extension, par répertoire ou par toute combinaison :

| Motif | Correspond à |
| - | - |
| `**/*.ts` | Tous les fichiers TypeScript de n'importe quel répertoire |
| `src/**/*` | Tous les fichiers sous le répertoire `src/` |
| `*.md` | Les fichiers Markdown à la racine du projet |
| `src/components/*.tsx` | Les composants React d'un répertoire précis |

Vous pouvez indiquer plusieurs motifs et utiliser l'expansion d'accolades pour couvrir plusieurs extensions dans un seul motif :

```markdown theme={null}
---
paths:
  - "src/**/*.{ts,tsx}"
  - "lib/**/*.ts"
  - "tests/**/*.test.ts"
---
```

Chaque groupe d'accolades multiplie le nombre de motifs développés : `src/*.{ts,tsx}` se développe en deux motifs, et `{a,b}/{c,d}/*.{ts,tsx}` en huit. Pour borner l'expansion, toute la liste `paths` d'une règle partage un budget unique de 1 000 motifs développés et de 4 Mio ; les motifs sans accolades ne comptent pas dans ce budget.

Claude Code utilise sans le développer tout motif qui dépasserait le budget, et ses accolades littérales ne correspondent à aucun fichier. Avant la v2.1.217, une valeur `paths` comportant de nombreux groupes d'accolades bloquait ou faisait planter la CLI au démarrage.

La syntaxe glob traite `[` comme le début d'une expression entre crochets telle que `[abc]`. Un motif contenant un `[` qui ne peut pas être lu comme une expression entre crochets, comme `photos [2024/**`, est invalide : il ne correspond à rien, et les autres motifs de la règle continuent de fonctionner. Pour faire correspondre un `[` littéral dans un nom de fichier, échappez-le : `photos \[2024/**`.

#### Référence du frontmatter des règles

Configurez une règle avec un [frontmatter](https://code.claude.com/docs/en/glossary#frontmatter) YAML entre des marqueurs `---` en haut du fichier. `paths` est le seul champ que Claude Code lit dans une règle ; tout autre champ est ignoré sans erreur. Claude Code retire le frontmatter avant de charger la règle dans le contexte.

| Champ | Obligatoire | Description |
| :- | :- | :- |
| `paths` | Non | Motifs glob qui [restreignent la règle aux fichiers correspondants](#règles-propres-à-des-chemins). Accepte une liste YAML ou une chaîne séparée par des virgules |

Si le YAML entre les marqueurs ne s'analyse pas, Claude Code ignore le frontmatter et charge la règle comme si elle n'avait pas de `paths`. Lancez `claude --debug` pour voir l'erreur d'analyse.

#### Partager des règles entre projets avec des liens symboliques

Le répertoire `.claude/rules/` prend en charge les liens symboliques : vous pouvez donc maintenir un ensemble de règles partagées et le lier dans plusieurs projets. Les liens symboliques circulaires sont détectés et gérés proprement.

Claude Code traite un lien symbolique dont la cible est hors de votre répertoire de travail comme un [import externe](#importer-des-fichiers-supplémentaires). Les règles liées ne se chargent pas tant que vous n'avez pas approuvé les imports externes pour le projet, et ensuite seules celles sans [champ `paths`](#règles-propres-à-des-chemins) se chargent.

Claude Code demande cette approbation une fois par projet, dans une boîte de dialogue au début d'une session interactive. La boîte de dialogue liste les fichiers de règles liés ainsi que les éventuels imports `@path` externes. Pour charger des règles partagées sans cette approbation, conservez-les dans [`~/.claude/rules/`](#règles-de-niveau-utilisateur), où elles s'appliquent à tous les projets de votre machine.

Cet exemple lie à la fois un répertoire partagé et un fichier individuel :

```bash theme={null}
ln -s ~/shared-claude-rules .claude/rules/shared
ln -s ~/company-standards/security.md .claude/rules/security.md
```

Si vous faites pointer un lien symbolique `.claude/rules/` ou `CLAUDE.md` vers un chemin réseau, comme le partage UNC `\\server\share` ou un chemin sous `/net` ou `/Network`, les instructions liées ne se chargent pas. Claude Code ne suit pas le lien, car la résolution d'un tel chemin peut contacter l'hôte qu'il désigne. Les chemins `\\wsl$` ne comptent pas comme des chemins réseau.

#### Règles de niveau utilisateur

Les règles personnelles placées dans `~/.claude/rules/` s'appliquent à tous les projets de votre machine. Utilisez-les pour des préférences qui ne sont pas propres à un projet :

```text theme={null}
~/.claude/rules/
├── preferences.md    # Vos préférences de code personnelles
└── workflows.md      # Vos workflows préférés
```

Claude Code charge les règles utilisateur avant les règles de projet : une règle de projet apparaît donc plus tard dans le contexte de Claude qu'une règle utilisateur. Aucun des deux ensembles ne remplace l'autre : si une règle utilisateur et une règle de projet se contredisent, Claude peut suivre l'une ou l'autre, alors gardez-les cohérentes.

### Gérer CLAUDE.md pour de grandes équipes

Pour les organisations qui déploient Claude Code dans plusieurs équipes, vous pouvez centraliser les instructions et contrôler quels fichiers CLAUDE.md sont chargés.

#### Déployer un CLAUDE.md à l'échelle de l'organisation

Les organisations peuvent déployer un CLAUDE.md géré centralement qui s'applique à tous les utilisateurs d'une machine. Ce fichier ne peut pas être exclu par des réglages individuels.

1. **Créer le fichier à l'emplacement de la politique gérée** :
   * macOS : `/Library/Application Support/ClaudeCode/CLAUDE.md`
   * Linux et WSL : `/etc/claude-code/CLAUDE.md`
   * Windows : `C:\Program Files\ClaudeCode\CLAUDE.md`
2. **Le déployer avec votre système de gestion de configuration** : utilisez MDM, Group Policy, Ansible ou des outils similaires pour distribuer le fichier sur les machines des développeurs. Voir les [réglages gérés](https://code.claude.com/docs/en/managed-settings) pour les autres options de configuration à l'échelle de l'organisation.

La clé `claudeMd` permet de placer le contenu du CLAUDE.md géré directement dans `managed-settings.json` au lieu de déployer un fichier séparé.

**Portée** : chaque session Claude Code de la machine, dans chaque dépôt. Pour des consignes propres à un dépôt, committez plutôt un CLAUDE.md de projet.

**Priorité** : identique à celle d'un fichier CLAUDE.md géré. Se charge avant les CLAUDE.md utilisateur et projet.

**Où elle est prise en compte** : uniquement dans les réglages gérés (managed) et de politique (policy). Définir `claudeMd` dans les réglages utilisateur, projet ou locaux n'a aucun effet.

L'exemple ci-dessous ajoute des instructions comportementales directement dans un fichier de réglages gérés :

```json theme={null}
{
  "claudeMd": "Always run `make lint` before committing.\nNever push directly to main."
}
```

Un CLAUDE.md géré et les [réglages gérés](https://code.claude.com/docs/en/managed-settings) n'ont pas le même rôle. Utilisez les réglages pour l'application technique de contraintes et CLAUDE.md pour les consignes comportementales :

| Préoccupation | À configurer dans |
| :- | :- |
| Bloquer des outils, commandes ou chemins de fichiers précis | Réglages gérés : `permissions.deny` |
| Imposer l'isolation par sandbox | Réglages gérés : `sandbox.enabled` |
| Variables d'environnement et routage du fournisseur d'API | Réglages gérés : `env` |
| Méthode de connexion et restrictions d'organisation | Réglages gérés : `forceLoginMethod`, `forceLoginOrgUUID` |
| Consignes de style et de qualité de code | CLAUDE.md géré |
| Rappels sur le traitement des données et la conformité | CLAUDE.md géré |
| Instructions comportementales pour Claude | CLAUDE.md géré |

Les règles des réglages sont appliquées par le client quoi que Claude décide de faire. Les instructions CLAUDE.md orientent le comportement de Claude mais ne constituent pas une couche d'application stricte.

#### Exclure certains fichiers CLAUDE.md

Dans les gros monorepos, les fichiers CLAUDE.md des répertoires ancêtres peuvent contenir des instructions sans rapport avec votre travail. Le réglage `claudeMdExcludes` permet d'ignorer des fichiers précis par chemin ou par motif glob.

Cet exemple exclut un CLAUDE.md de premier niveau et un répertoire de règles d'un dossier parent. Ajoutez-le à `.claude/settings.local.json` pour que l'exclusion reste locale à votre machine :

```json theme={null}
{
  "claudeMdExcludes": [
    "**/monorepo/CLAUDE.md",
    "/home/user/monorepo/other-team/.claude/rules/**"
  ]
}
```

Les motifs sont comparés aux chemins absolus des fichiers avec la syntaxe glob. Vous pouvez configurer `claudeMdExcludes` à n'importe quelle [couche de réglages](https://code.claude.com/docs/en/settings#where-settings-live) : utilisateur, projet, local ou politique gérée. Les tableaux fusionnent entre les couches.

Pour exclure un fichier de règles atteint via un [lien symbolique](#partager-des-règles-entre-projets-avec-des-liens-symboliques), que le lien soit le fichier ou son répertoire, écrivez le motif sur l'un ou l'autre chemin : le chemin du fichier sous `.claude/rules/` ou la cible du lien. Un motif qui correspond à l'un des deux chemins exclut le fichier. Avant la v2.1.239, seul un motif correspondant à la cible du lien excluait le fichier.

Les fichiers CLAUDE.md de politique gérée ne peuvent pas être exclus. Cela garantit que les instructions de l'organisation s'appliquent toujours, quels que soient les réglages individuels.

## AGENTS.md

Claude Code peut lire [`AGENTS.md`](https://code.claude.com/docs/en/glossary#agents-md) comme instructions de projet : un dépôt déjà configuré pour d'autres agents de code fonctionne donc sans ajouter de `CLAUDE.md`, d'import ou de réglage. Ce tableau indique ce que Claude lit par défaut pour chaque combinaison de fichiers d'instructions dans votre dépôt :

| Votre dépôt contient | Claude lit |
| :- | :- |
| Un `AGENTS.md`, et aucun `CLAUDE.md` ni `CLAUDE.local.md` dans votre répertoire de travail ou au-dessus | Votre `AGENTS.md` |
| Un `AGENTS.md` et un `CLAUDE.md` ou `CLAUDE.local.md` dans votre répertoire de travail ou au-dessus | Vos fichiers `CLAUDE.md` uniquement |
| Un `CLAUDE.md` qui [importe déjà `AGENTS.md`](#partager-un-fichier-avec-dautres-outils-de-code) | Votre `CLAUDE.md`, avec `AGENTS.md` inclus via l'import |

Pour modifier ce comportement par défaut, par exemple pour que Claude lise toujours les deux fichiers, uniquement `CLAUDE.md`, ou seulement les instructions gérées de votre organisation, [modifiez le réglage **Project instructions**](#choisir-quels-fichiers-dinstructions-se-chargent).

> **Note** : la lecture directe d'`AGENTS.md` nécessite Claude Code v2.1.277+. Dans certaines sessions, Claude [ne peut pas lire `AGENTS.md`](#quand-la-prise-en-charge-dagentsmd-est-indisponible) ; [importez-le alors depuis un `CLAUDE.md`](#partager-un-fichier-avec-dautres-outils-de-code).

### Quand Claude Code lit AGENTS.md

Par défaut, Claude lit `AGENTS.md` uniquement lorsque vous n'avez aucun `CLAUDE.md` dans votre répertoire de travail ou au-dessus. Voici quels fichiers comptent pour cette vérification :

* **Comptent, donc Claude les lit à la place d'`AGENTS.md`** : un `CLAUDE.md`, `.claude/CLAUDE.md` ou `CLAUDE.local.md` dans votre répertoire de travail ou dans n'importe quel répertoire au-dessus
* **Ne comptent pas, et continuent de se charger en plus d'`AGENTS.md`** : votre `~/.claude/CLAUDE.md`, le `CLAUDE.md` géré de votre organisation et les fichiers `.claude/rules/`

Quand aucun ne compte, voici ce que Claude lit et comment le constater :

* **Au démarrage de la session** : chaque `AGENTS.md` et `.claude/AGENTS.md` de votre répertoire de travail et des répertoires au-dessus. Dans une session interactive, vous voyez dans la conversation une ligne telle que `no CLAUDE.md found; AGENTS.md loaded: /home/you/repo/AGENTS.md`
* **Quand Claude travaille dans des sous-répertoires** : l'`AGENTS.md` d'un sous-répertoire, lorsque Claude y ouvre un fichier avec l'outil Read et que ce sous-répertoire n'a aucun des trois fichiers `CLAUDE.md` en propre
* **À l'intérieur de chaque `AGENTS.md`** : les [imports `@path`](#importer-des-fichiers-supplémentaires) sont développés, les motifs [`claudeMdExcludes`](#exclure-certains-fichiers-claudemd) s'appliquent, et les subagents qui [démarrent sans instructions de projet](https://code.claude.com/docs/en/sub-agents#what-loads-at-startup) démarrent aussi sans ces fichiers
* **Non lus** : `AGENTS.local.md`, `AGENTS.override.md`, ni rien sous un répertoire `.agents/`

> **Note** : comme `CLAUDE.local.md` compte, en ajouter un pour garder vos propres instructions non committées dans un projet qui repose sur `AGENTS.md` empêche Claude de lire `AGENTS.md` pour vous. Pour garder votre `CLAUDE.local.md` tout en faisant lire `AGENTS.md` à Claude, réglez **Project instructions** sur [`claude-md-and-agents-md`](#choisir-quels-fichiers-dinstructions-se-chargent).

### Choisir quels fichiers d'instructions se chargent

Pour changer les fichiers que lit Claude, tapez `/config` dans une session Claude Code pour ouvrir le panneau de réglages, puis réglez **Project instructions** sur l'une de ces valeurs :

| Valeur | Ce que Claude lit |
| :- | :- |
| `claude-md-or-agents-md` | Vos fichiers `CLAUDE.md`, ou vos fichiers `AGENTS.md` lorsque vous n'avez aucun `CLAUDE.md` ni `CLAUDE.local.md` dans votre répertoire de travail ou au-dessus. C'est la valeur par défaut |
| `claude-md-and-agents-md` | Vos fichiers `CLAUDE.md` et `AGENTS.md` ensemble, les fichiers `CLAUDE.md` de chaque répertoire d'abord, puis son `AGENTS.md`. Claude Code ignore un `AGENTS.md` déjà chargé : celui que votre `CLAUDE.md` importe ou vers lequel il pointe en lien symbolique n'est donc pas lu deux fois |
| `claude-md` | Vos fichiers `CLAUDE.md` uniquement |
| `managed-only` | Au lancement, uniquement le `CLAUDE.md` géré de votre organisation et l'[auto memory](#auto-memory). Vos fichiers `CLAUDE.md` de projet, locaux et utilisateur, vos fichiers `.claude/rules/` et tous les `AGENTS.md` sont exclus. Le `CLAUDE.md` et les fichiers `.claude/rules/` d'un sous-répertoire, ainsi que les [règles restreintes à des chemins](#règles-propres-à-des-chemins), se chargent toujours à la demande |

Vous pouvez aussi définir la valeur dans un fichier de réglages au lieu de `/config`. Ajoutez-la à [`pluginConfigs`](https://code.claude.com/docs/en/settings-reference#pluginconfigs) sous `cc-plugin-agents-md@builtin`, l'ID du plugin intégré qui lit `AGENTS.md`. Claude Code lit cette entrée depuis `~/.claude/settings.json`, un fichier `--settings` ou les [réglages gérés](https://code.claude.com/docs/en/managed-settings), et l'ignore dans les fichiers de réglages de projet et locaux. Cet exemple fait lire les deux fichiers à Claude :

```json settings.json theme={null}
{
  "pluginConfigs": {
    "cc-plugin-agents-md@builtin": {
      "options": { "instructionFiles": "claude-md-and-agents-md" }
    }
  }
}
```

Avant la v2.1.285, l'ID du plugin était `agents-md@builtin`, et Claude Code ignorait une entrée sous `cc-plugin-agents-md@builtin`. Si des versions antérieures lisent aussi votre fichier de réglages, utilisez-y `agents-md@builtin`. Claude Code v2.1.285+ lit une entrée sous l'un ou l'autre ID.

Votre modification s'applique dès le prochain message que vous envoyez et dans chaque nouvelle session.

### Quand la prise en charge d'AGENTS.md est indisponible

Dans ces sessions, Claude ne lit que les fichiers `CLAUDE.md`, et **Project instructions** n'apparaît pas dans le panneau de réglages `/config` :

* Vous utilisez une version de Claude Code antérieure à la v2.1.277
* Vous avez utilisé `/plugin` pour désactiver le plugin intégré qui lit `AGENTS.md`
* Dans certains cas, c'est votre [première session après une mise à jour](https://code.claude.com/docs/en/env-vars#first-session-after-an-install-or-upgrade) depuis la v2.1.276 ou antérieure. Claude lit `AGENTS.md` à partir de la session suivante

Avant la v2.1.281, certaines sessions, comme celles sur Amazon Bedrock ou avec la télémétrie désactivée, ne lisaient que les fichiers `CLAUDE.md`. Sur ces versions, mettez Claude Code à jour. Pour fournir votre `AGENTS.md` à Claude dans n'importe laquelle de ces sessions, [importez-le depuis un `CLAUDE.md`](#partager-un-fichier-avec-dautres-outils-de-code).

### Différences entre AGENTS.md et CLAUDE.md

Un `AGENTS.md` que Claude lit via le réglage **Project instructions** diffère d'un `CLAUDE.md` sur ces points :

| | `CLAUDE.md` | `AGENTS.md` lu via le réglage |
| :- | :- | :- |
| [Hooks `InstructionsLoaded`](https://code.claude.com/docs/en/hooks#instructionsloaded) | Se déclenchent | Ne se déclenchent pas. Ils se déclenchent normalement pour un `AGENTS.md` qu'un `CLAUDE.md` importe ou vers lequel il pointe en lien symbolique |
| Répertoires ajoutés avec `--add-dir` lorsque [`CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD`](#charger-depuis-des-répertoires-supplémentaires) est défini | Leur `CLAUDE.md` se charge | Leur `AGENTS.md` ne se charge pas |
| Un import `@path` d'un fichier hors de votre répertoire de travail | Claude Code vous demande d'approuver les [imports externes](#importer-des-fichiers-supplémentaires) | Se charge seulement si vous avez déjà approuvé les imports externes pour ce projet, sans demande |

### Retirer un ancien contournement pour AGENTS.md

Si vous aviez configuré Claude Code pour lire `AGENTS.md` avant qu'il ne le fasse de lui-même, voici quoi faire pour chaque configuration courante :

* **Un `CLAUDE.md` contenant `@AGENTS.md`** : vous pouvez le laisser. Conserver l'import ne fait jamais lire `AGENTS.md` deux fois à Claude, quelle que soit la valeur de **Project instructions**. Supprimez le `CLAUDE.md` s'il ne contient rien d'autre, ou gardez-le si certaines de vos sessions [ne peuvent pas charger `AGENTS.md` directement](#quand-la-prise-en-charge-dagentsmd-est-indisponible).
* **Un `CLAUDE.md` qui demande en toutes lettres à Claude de lire `AGENTS.md`** : Claude ne voit `AGENTS.md` que s'il décide d'ouvrir le fichier. Supprimez le `CLAUDE.md` pour que Claude lise `AGENTS.md` directement, ou remplacez la phrase par un import `@AGENTS.md`.
* **Un `CLAUDE.md` en lien symbolique vers `AGENTS.md`** : rien à faire, ou supprimez le lien. Dans les deux cas, Claude lit le contenu une seule fois.
* **Un hook `SessionStart` qui affiche `AGENTS.md`** : retirez-le. Dès que Claude lit `AGENTS.md` directement, le hook ajoute une seconde copie au contexte.

### Partager un fichier avec d'autres outils de code

Lorsque Claude ne lit pas directement votre `AGENTS.md`, vous pouvez quand même en faire le fichier unique partagé par tous les outils en plaçant un import `@AGENTS.md` dans un `CLAUDE.md` à côté. Faites-le lorsque votre projet a aussi un `CLAUDE.md`, lorsque vous avez réglé **Project instructions** sur `claude-md`, ou dans les sessions qui [ne peuvent pas charger `AGENTS.md`](#quand-la-prise-en-charge-dagentsmd-est-indisponible). Ajoutez les instructions propres à Claude sous l'import ; Claude lit d'abord le fichier importé, puis le reste :

```markdown CLAUDE.md theme={null}
@AGENTS.md

## Claude Code

Use plan mode for changes under `src/billing/`.
```

(Exemple : utiliser le plan mode pour les modifications sous `src/billing/`.)

Si vous n'avez pas besoin de contenu propre à Claude, un lien symbolique fonctionne aussi :

```bash theme={null}
ln -s AGENTS.md CLAUDE.md
```

La commande n'affiche rien en cas de succès. Avant de préférer le lien symbolique à l'import, vérifiez ces contraintes :

* **Modification** : Claude lit `CLAUDE.md` via le lien, mais les outils Edit et Write [refusent d'écrire à travers un lien symbolique](https://code.claude.com/docs/en/errors#refusing-after-a-symlink-changed), et le refus invite Claude à modifier la cible du lien, `AGENTS.md`, à la place
* **Windows** : si vous ou quiconque clone le dépôt travaillez sous Windows, utilisez plutôt l'import `@AGENTS.md`. Créer un lien symbolique y nécessite des privilèges Administrateur ou le mode Développeur, et Git extrait un lien symbolique committé sous forme de simple fichier texte sauf si `core.symlinks` est activé, ce qui laisse ce clone avec un `CLAUDE.md` d'une ligne au lieu de vos instructions

Quelle que soit l'approche, lancez `/context` dans votre session suivante et vérifiez que `CLAUDE.md` apparaît sous **Memory files**.

### Migrer les instructions d'autres outils

Lancer [`/init`](https://code.claude.com/docs/en/commands) lit les fichiers d'instructions d'autres outils et intègre les parties pertinentes dans le `CLAUDE.md` généré :

* Règles Cursor dans `.cursor/rules/` ou `.cursorrules`
* Règles Copilot dans `.github/copilot-instructions.md`
* Avec `CLAUDE_CODE_NEW_INIT=1` défini : `AGENTS.md`, `.devin/rules/`, `.windsurf/rules/` ou `.windsurfrules`, et `.clinerules`

Vous pouvez aussi lancer [`/import`](https://code.claude.com/docs/en/commands) pour importer dans Claude Code la configuration d'un agent de code pris en charge : il ajoute une copie ponctuelle des fichiers d'instructions tels qu'`AGENTS.md` au `CLAUDE.md` correspondant et reprend les serveurs MCP, commandes, subagents et skills. Nécessite Claude Code v2.1.213+.

## Auto memory

L'auto memory permet à Claude d'accumuler des connaissances d'une session à l'autre sans que vous écriviez quoi que ce soit. Au fil de son travail, Claude enregistre pour lui-même quatre types de notes. Claude consigne le type dans un champ `type` du frontmatter du fichier mémoire :

* `user` : votre rôle, votre expertise et vos préférences de travail
* `feedback` : les corrections que vous donnez à Claude et les approches que vous validez
* `project` : le travail en cours, les échéances et les décisions que Claude ne peut pas déduire du code ou de l'historique git
* `reference` : où trouver des informations hors du projet, comme un outil de suivi des tickets ou un tableau de bord

Claude ignore tout ce qu'il peut déduire de la base de code, comme l'architecture, les chemins de fichiers ou les correctifs de débogage. Il ignore aussi tout ce que vos fichiers CLAUDE.md disent déjà.

Claude n'enregistre pas quelque chose à chaque session. Il décide de ce qui mérite d'être retenu selon l'utilité que l'information aurait dans une conversation future.

### Activer ou désactiver l'auto memory

L'auto memory est activée par défaut dans les sessions locales. En dehors des sessions [Claude Tag](https://claude.com/docs/claude-tag/overview), une session dans un [environnement auto-hébergé](https://code.claude.com/docs/en/self-hosted-environments-configuration#how-each-session’s-config-is-assembled) tourne avec l'auto memory désactivée par défaut.

Pour la basculer, ouvrez `/memory` dans une session et utilisez l'interrupteur d'auto memory, qui enregistre `autoMemoryEnabled` dans vos réglages utilisateur à `~/.claude/settings.json`.

L'interrupteur désactive l'auto memory mais ne la réactive pas dans ces sessions :

* Une [session en arrière-plan](https://code.claude.com/docs/en/agent-view)
* Une session lancée par une autre session Claude Code, par exemple lorsque Claude exécute `claude` via son outil Bash

Lorsque l'auto memory y est désactivée, l'interrupteur affiche `off · can't be turned on here; use a session started outside Claude Code`. Pour la réactiver, lancez `claude` directement dans votre terminal et utilisez l'interrupteur de `/memory` dans cette session.

Pour la désactiver pour un seul projet, définissez `autoMemoryEnabled` dans les réglages de ce projet :

```json theme={null}
{
  "autoMemoryEnabled": false
}
```

Pour désactiver l'auto memory par variable d'environnement, définissez `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`.

### Emplacement de stockage

Chaque projet dispose de son propre répertoire mémoire à `~/.claude/projects/<project>/memory/`. Le chemin `<project>` est dérivé du dépôt git : tous les worktrees et sous-répertoires d'un même dépôt partagent donc un seul répertoire d'auto memory. Hors d'un dépôt git, c'est la racine du projet qui est utilisée.

Si vous définissez [`CLAUDE_CODE_PROJECT_DIR_NAME`](https://code.claude.com/docs/en/sessions#name-the-project-directory-yourself) en plus de `CLAUDE_CONFIG_DIR`, Claude Code utilise ce nom comme répertoire `<project>` sous `<config dir>/projects/`, quel que soit le dépôt dans lequel vous le lancez : les projets lancés avec ce répertoire de configuration partagent donc un seul répertoire d'auto memory. Nécessite Claude Code v2.1.234+.

Pour stocker l'auto memory ailleurs, définissez `autoMemoryDirectory` dans votre `settings.json`. Il est lu depuis n'importe quelle [portée de réglages](https://code.claude.com/docs/en/settings#settings-precedence) : utilisateur, projet, local, politique ou `--settings`.

```json theme={null}
{
  "autoMemoryDirectory": "~/my-custom-memory-dir"
}
```

La valeur doit être un chemin absolu ou commencer par `~/`.

Lorsque vous le définissez dans le `.claude/settings.json` ou le `.claude/settings.local.json` d'un projet, Claude Code l'honore selon la même [règle de confiance de l'espace de travail que les hooks des fichiers de réglages](https://code.claude.com/docs/en/permissions#what-runs-before-you-trust-a-folder). Tant que [`permissions.blockReadsOutsideWorkingDirectories`](https://code.claude.com/docs/en/settings-reference#permissions-blockreadsoutsideworkingdirectories) est activé, Claude Code ne charge aucune auto memory depuis un répertoire choisi par un [fichier de réglages fourni par le dépôt](https://code.claude.com/docs/en/permissions#when-your-local-settings-file-needs-trust) et n'y enregistre rien, où que se trouve ce répertoire.

Le répertoire contient un index `MEMORY.md` et un fichier thématique par mémoire :

```text theme={null}
~/.claude/projects/<project>/memory/
├── MEMORY.md           # Index, une ligne par mémoire, chargé dans chaque session
├── user_role.md        # Une mémoire
├── feedback_testing.md # Une mémoire
└── ...                 # Tous les autres fichiers thématiques créés par Claude
```

`MEMORY.md` sert d'index au répertoire mémoire. Claude lit et écrit des fichiers dans ce répertoire tout au long de votre session, en utilisant `MEMORY.md` pour savoir ce qui est stocké où.

L'auto memory est locale à la machine. Tous les worktrees et sous-répertoires d'un même dépôt git partagent un seul répertoire d'auto memory. Les fichiers ne sont pas partagés entre machines ni avec les environnements cloud.

Claude Code supprime les anciens transcripts de session après la période de rétention [`cleanupPeriodDays`](https://code.claude.com/docs/en/settings-reference#cleanupperioddays), mais exclut de ce [nettoyage de rétention](https://code.claude.com/docs/en/claude-directory#cleaned-up-automatically) les fichiers mémoire du répertoire mémoire. `MEMORY.md` et les fichiers thématiques restent jusqu'à ce que vous ou Claude les modifiiez ou les supprimiez.

### Fonctionnement

Les 200 premières lignes de `MEMORY.md`, ou ses 25 premiers Ko, selon la limite atteinte en premier, sont chargées au début de chaque conversation. Le contenu au-delà de ce seuil n'est pas chargé au démarrage de la session. Claude garde `MEMORY.md` concis en déplaçant les notes détaillées dans des fichiers thématiques séparés.

Après que Claude a écrit dans `MEMORY.md`, Claude Code mesure le fichier par rapport aux limites de lecture de 200 lignes et 25 Ko. Si le fichier approche d'une limite, Claude Code rappelle à Claude de le raccourcir : garder une ligne par entrée, déplacer le détail dans des fichiers thématiques, fusionner ou supprimer les entrées périmées. Si le fichier dépasse une limite, l'écriture réussit quand même, mais Claude Code renvoie une [erreur demandant à Claude de réécrire l'index](https://code.claude.com/docs/en/errors#memory-index-is-over-its-read-limit), car tout ce qui dépasse la limite est perdu au chargement suivant.

Cette limite ne s'applique qu'à `MEMORY.md`. Claude Code charge en entier un fichier CLAUDE.md jusqu'à 4 Mio et ignore un fichier plus gros. Les fichiers plus courts produisent une meilleure adhésion.

Claude Code ne charge pas au démarrage les fichiers thématiques comme `user_role.md` ou `feedback_testing.md`. Claude les lit à la demande avec ses outils de fichiers standard lorsqu'il a besoin de l'information.

L'auto memory de la conversation principale n'est pas chargée dans les [subagents](https://code.claude.com/docs/en/sub-agents#what-loads-at-startup) ; l'exception est un [fork](https://code.claude.com/docs/en/sub-agents#fork-the-current-conversation), qui hérite de la conversation parente et du prompt système. L'auto memory propre à un subagent, activée avec le champ `memory` du subagent, est un répertoire distinct.

Claude lit et écrit des fichiers mémoire pendant votre session. Lorsque vous voyez des messages comme « Saved 2 memories » ou « Recalled 2 memories » dans l'interface de Claude Code, Claude est en train de mettre à jour ou de lire `~/.claude/projects/<project>/memory/`.

Lorsque Claude écrit un fichier mémoire qui commence par un frontmatter YAML, Claude Code enregistre l'heure d'écriture dans un champ de frontmatter `modified` sous forme d'horodatage ISO 8601. L'horodatage indique à quel point le fait est récent, à vous comme à Claude lorsqu'il relit la mémoire. Tout fichier doté d'un frontmatter reçoit ce champ la prochaine fois que Claude l'écrit, y compris les fichiers créés avec des versions antérieures ; Claude Code n'ajoute jamais de frontmatter à un fichier qui n'en a pas. Le champ `modified` nécessite Claude Code v2.1.214+.

### Auditer et modifier votre mémoire

Les fichiers d'auto memory sont du markdown brut que vous pouvez modifier ou supprimer à tout moment. Lancez [`/memory`](#consulter-et-modifier-avec-memory) pour parcourir et ouvrir les fichiers mémoire depuis une session.

## Consulter et modifier avec `/memory`

La commande `/memory` liste vos fichiers CLAUDE.md, CLAUDE.local.md et les autres emplacements de fichiers mémoire des portées utilisateur et projet, y compris des entrées CLAUDE.md utilisateur et projet pour des fichiers qui n'existent pas encore. Elle permet aussi d'activer ou de désactiver l'auto memory et propose une option pour ouvrir le dossier d'auto memory. Sélectionnez un fichier pour l'ouvrir dans votre éditeur ; en sélectionner un qui n'existe pas encore le crée d'abord. Pour vérifier quels fichiers `CLAUDE.md` et de règles se sont chargés au lancement, lancez `/context`.

Les éditeurs graphiques comme VS Code ouvrent le fichier dans une fenêtre séparée, et vous pouvez continuer à utiliser la session pendant qu'il est ouvert. Avant la v2.1.216, `/memory` attendait que vous fermiez le fichier pour répondre. Les éditeurs de terminal comme Vim prennent le contrôle du terminal jusqu'à ce que vous les quittiez.

Lorsque vous demandez à Claude de retenir quelque chose, comme « utilise toujours pnpm, pas npm » ou « retiens que les tests d'API nécessitent une instance Redis locale », Claude l'enregistre dans l'auto memory. Pour ajouter des instructions à CLAUDE.md à la place, demandez-le directement à Claude, par exemple « ajoute ceci à CLAUDE.md », ou modifiez le fichier vous-même via `/memory`.

## Dépanner les problèmes de mémoire

Voici les problèmes les plus courants avec CLAUDE.md et l'auto memory, ainsi que les étapes pour les déboguer.

### Claude ne suit pas mon CLAUDE.md

Le contenu de CLAUDE.md est transmis comme un message utilisateur après le prompt système, et non comme une partie du prompt système lui-même. Claude le lit et essaie de le suivre, mais un respect strict n'est pas garanti, surtout pour des instructions vagues ou contradictoires.

Pour déboguer :

* Lancez `/context` et regardez la liste sous **Memory files** pour vérifier les fichiers CLAUDE.md et CLAUDE.local.md qui devraient se charger au lancement. S'il en manque un, Claude ne peut pas le voir. Utilisez `/memory` pour ouvrir et modifier les fichiers.
* Un `CLAUDE.md` situé dans un sous-répertoire de votre répertoire de travail n'apparaît pas sous **Memory files**, car il se charge à la demande et non au lancement. Lorsqu'il se charge, une ligne `Loaded` avec son chemin apparaît dans le terminal. Pour en tester un nouveau, créez-le depuis votre shell plutôt que de demander à Claude de l'écrire, puis demandez à Claude de lire un fichier de ce sous-répertoire.
* Vérifiez que le CLAUDE.md concerné se trouve à un emplacement chargé pour votre session (voir [Choisir où placer les fichiers CLAUDE.md](#choisir-où-placer-les-fichiers-claudemd)).
* Rendez les instructions plus précises. « Utiliser une indentation de 2 espaces » fonctionne mieux que « bien formater le code ».
* Cherchez des instructions contradictoires entre fichiers CLAUDE.md. Si deux fichiers donnent des consignes différentes pour le même comportement, Claude peut en choisir une arbitrairement.
* Vérifiez si votre instruction entre en concurrence avec les consignes que Claude Code ajoute de lui-même. Si votre CLAUDE.md fixe des règles de commit ou de pull request, désactivez les consignes intégrées avec [`includeGitInstructions`](https://code.claude.com/docs/en/settings-reference#includegitinstructions) et définissez le texte d'attribution avec [`attribution`](https://code.claude.com/docs/en/settings-reference#attribution).

Si l'instruction doit s'exécuter à un moment précis, par exemple avant chaque commit ou après chaque modification de fichier, écrivez-la plutôt sous forme de [hook](https://code.claude.com/docs/en/hooks-guide). Les hooks s'exécutent comme des commandes shell à des événements fixes du cycle de vie et s'appliquent quoi que Claude décide de faire.

Pour des instructions au niveau du prompt système, utilisez [`--append-system-prompt`](https://code.claude.com/docs/en/cli-reference#system-prompt-flags). Il se passe au lancement, il convient donc mieux aux scripts et à l'automatisation qu'à l'usage interactif. Pour son comportement lors de la reprise d'une conversation, voir [Flags de prompt système dans les conversations reprises](https://code.claude.com/docs/en/cli-reference#system-prompt-flags-in-resumed-conversations).

> **Astuce** : utilisez le [hook `InstructionsLoaded`](https://code.claude.com/docs/en/hooks#instructionsloaded) pour journaliser quels fichiers `CLAUDE.md` et de règles se chargent, quand ils se chargent et pourquoi. C'est utile pour déboguer les règles propres à des chemins ou les fichiers chargés paresseusement dans les sous-répertoires.

### Mon AGENTS.md ne se charge pas

Si votre dépôt contient un `AGENTS.md` et que Claude ne semble pas connaître son contenu, la cause habituelle est un `CLAUDE.md` quelque part sur le chemin du projet. Par défaut, Claude ne lit `AGENTS.md` que lorsque vous n'avez aucun `CLAUDE.md` ni `CLAUDE.local.md` dans votre répertoire de travail ou au-dessus. Vérifiez ces points dans l'ordre :

1. Cherchez un `CLAUDE.md`, `.claude/CLAUDE.md` ou `CLAUDE.local.md` dans votre répertoire de travail ou dans n'importe quel répertoire au-dessus, hormis votre `~/.claude/CLAUDE.md`. Si vous en trouvez un, Claude le lit à la place d'`AGENTS.md`, sauf si vous réglez **Project instructions** sur `claude-md-and-agents-md`.
2. Lancez `claude --version` et vérifiez que vous êtes en v2.1.277 ou plus. Avant la v2.1.281, certaines sessions, comme celles sur Amazon Bedrock ou avec la télémétrie désactivée, [ne pouvaient pas non plus charger `AGENTS.md`](#quand-la-prise-en-charge-dagentsmd-est-indisponible) ; sur ces versions, passez à la v2.1.281 ou plus.
3. Tapez `/config` dans votre session pour ouvrir le panneau de réglages et vérifiez que **Project instructions** n'est pas réglé sur `claude-md` ou `managed-only`. Si le réglage n'y apparaît pas du tout, votre session fait partie de celles qui [ne peuvent pas charger `AGENTS.md`](#quand-la-prise-en-charge-dagentsmd-est-indisponible).

Pour vérifier si Claude a lu votre `AGENTS.md`, lancez `/memory` et cherchez son chemin dans la liste.

Avant la v2.1.280, `/memory` et `/context` ne listaient pas un `AGENTS.md` lu directement par Claude. Sur ces versions, demandez plutôt à Claude ce que disent ses instructions de projet.

Si vous voulez garder le `CLAUDE.md` trouvé, ou si votre session ne peut pas charger `AGENTS.md`, [ajoutez à côté de votre `AGENTS.md` un `CLAUDE.md` qui l'importe](#partager-un-fichier-avec-dautres-outils-de-code).

### Je ne sais pas ce que l'auto memory a enregistré

Lancez `/memory` et sélectionnez le dossier d'auto memory pour parcourir ce que Claude a enregistré. Tout est en markdown brut que vous pouvez lire, modifier ou supprimer.

### Mon CLAUDE.md est trop gros

Les fichiers de plus de 200 lignes consomment davantage de contexte et peuvent réduire l'adhésion. Claude Code ignore un fichier de plus de 4 Mio. Utilisez des [règles restreintes à des chemins](#règles-propres-à-des-chemins) pour ne charger des instructions que lorsque Claude travaille sur des fichiers correspondants, ou retirez le contenu qui n'est pas utile à chaque session. Découper en [imports `@path`](#importer-des-fichiers-supplémentaires) aide à l'organisation mais ne réduit pas le contexte, puisque les fichiers importés se chargent au lancement.

Si l'un de vos fichiers d'instructions dépasse la longueur recommandée, vous voyez un avertissement au démarrage et lorsque vous lancez `/status`. Vous voyez aussi un avertissement lorsque des fichiers qui respectent chacun cette longueur dépassent ensemble une limite cumulée au démarrage de la session. Chaque CLAUDE.md, fichier de règles et import `@path` compte comme un fichier distinct.

Le bilan [`/doctor`](https://code.claude.com/docs/en/commands#all-commands) propose des coupes pour un CLAUDE.md versionné : il retire le contenu que Claude peut déduire de la base de code, comme l'organisation des répertoires, les listes de dépendances et les vues d'ensemble de l'architecture, et conserve les pièges, les justifications et les conventions qui diffèrent des comportements par défaut des outils.

### Les instructions semblent perdues après `/compact`

Le CLAUDE.md à la racine du projet survit au compactage : après `/compact`, Claude le relit depuis le disque et le réinjecte dans la session. Les CLAUDE.md imbriqués dans les sous-répertoires et les règles avec un [frontmatter `paths:`](#règles-propres-à-des-chemins) se rechargent à la demande.

Si une instruction a disparu après le compactage, c'est qu'elle n'avait été donnée que dans la conversation, qu'elle se trouve dans un CLAUDE.md imbriqué qui ne s'est pas encore rechargé, ou qu'il s'agit d'une règle restreinte à des chemins qui n'a correspondu à aucun fichier depuis. Ajoutez à CLAUDE.md les instructions données uniquement en conversation pour les rendre persistantes. Voir [Ce qui survit au compactage](https://code.claude.com/docs/en/context-window#what-survives-compaction) pour le détail complet.

Voir [Écrire des instructions efficaces](#écrire-des-instructions-efficaces) pour des conseils sur la taille, la structure et la précision.

## Ressources associées

* [Déboguer votre configuration](https://code.claude.com/docs/en/debug-your-config) : diagnostiquer pourquoi CLAUDE.md ou les réglages ne prennent pas effet
* [Skills](https://code.claude.com/docs/en/skills) : empaqueter des workflows répétables qui se chargent à la demande
* [Réglages](https://code.claude.com/docs/en/settings) : configurer le comportement de Claude Code avec des fichiers de réglages
* [Mémoire des subagents](https://code.claude.com/docs/en/sub-agents#enable-persistent-memory) : permettre aux subagents de tenir leur propre auto memory
