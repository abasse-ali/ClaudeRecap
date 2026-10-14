# Kit de communication

> Source : https://code.claude.com/docs/en/communications-kit · [← Index](../31-index-documentation-officielle.md)

*Annonces de lancement, messages de campagne au compte-gouttes (« drip campaign ») et réponses FAQ pour déployer Claude Code dans votre organisation d'ingénierie.*

Cette page s'adresse aux administrateurs et aux responsables techniques qui déploient Claude Code auprès d'une équipe. Elle fournit des annonces de lancement prêtes à copier, une campagne d'astuces diffusée progressivement, et des réponses d'une ligne aux questions qu'on vous posera le plus souvent.

> **Note** : considérez tout ce qui suit comme un brouillon, pas comme un texte définitif. Réécrivez chaque message avec la voix de votre organisation, remplacez les tâches d'exemple par de vrais bugs et modules de votre propre base de code, et remplacez les `[espaces réservés entre crochets]` avant l'envoi. Les annonces qui font réellement adopter l'outil sont celles qui donnent l'impression d'avoir été écrites par quelqu'un de votre entreprise.

> Les modèles de messages ci-dessous sont reproduits tels quels (en anglais, dans des blocs de code), suivis d'un résumé en français de leur contenu.

## Communications de lancement

Une annonce en deux formats, plus deux variantes facultatives. Choisissez celle qui convient à votre déploiement et réécrivez-la à partir de là.

### Avant d'envoyer

Passez cette liste de contrôle avant que l'annonce parte. Chaque élément comble une lacune qui, sinon, se transforme en fil de support le jour du lancement.

| Élément | Pourquoi c'est important |
| - | - |
| Canal `#claude-code` créé et lié dans le message | Donne un endroit unique où les questions arrivent |
| Commande d'installation testée sur au moins une machine de votre environnement | Détecte les problèmes de proxy ou de pare-feu avant que tout le monde les rencontre en même temps |
| Lien sécurité et traitement des données prêt ([Data usage](https://code.claude.com/docs/en/data-usage) ou votre équivalent interne) | « Où va mon code ? » sera la première réponse au message |
| Une première tâche concrète choisie, un vrai bug ou fichier de votre base de code | Les exemples génériques ne convertissent pas ; « corrige le test instable dans `auth_test.go` », si |
| Un responsable nommé pour le canal pendant les 48 premières heures | Les questions sans réponse le jour du lancement tuent l'élan |
| Un sponsor de la direction (C-suite) prêt à envoyer ou cosigner l'annonce | Les lancements envoyés par un dirigeant obtiennent systématiquement une meilleure adoption la première semaine que ceux envoyés par un administrateur |

### L'annonce

À utiliser comme message standard de déploiement à l'échelle de l'organisation. Il explique ce qu'est Claude Code, donne un chemin d'installation en deux minutes, propose au lecteur une tâche concrète à essayer, et répond à « où va mon code ? » avant que quiconque ait à le demander.

#### Onglet « Email »

```text
Subject: Claude Code is live for [Engineering / your team]

Team,

As of today you have access to Claude Code, an AI coding agent that runs in
your terminal, reads your actual codebase, and works through real tasks end
to end: debugging, refactors, tests, PRs. It is not autocomplete and it is
not a chat window. It edits files and runs your commands.

Get running in two minutes:

    curl -fsSL https://claude.ai/install.sh | bash
    cd <your-repo>
    claude

Then run /init once. Claude reads your project and writes a CLAUDE.md with
your build commands and conventions, so you stop re-explaining the basics.

Then try one of these on the repo you are already in:

  - "The test in [file] is flaky. Figure out why and fix it"
  - "Walk me through how [module] handles [X]"
  - "Look at my working diff and tell me what's risky before I push"

Where your code goes: Claude Code runs in your terminal and talks directly
to Anthropic's API. On our Team or Enterprise plan, Anthropic does not use
your code or prompts to train its models.
Details: https://code.claude.com/docs/en/data-usage
         https://code.claude.com/docs/en/security

Where to go with questions: #claude-code. [Owner name] is watching it
this week.

- [Name]

P.S. Prefer your editor? There is a VS Code extension and a JetBrains
plugin. Same agent, no terminal required.
```

Contenu en français :
- **Objet** : « Claude Code est disponible pour [Engineering / votre équipe] ».
- Présentation : un agent de codage IA qui tourne dans le terminal, lit votre vraie base de code et mène des tâches réelles de bout en bout (débogage, refactorisations, tests, PR). Ce n'est ni de l'autocomplétion ni une fenêtre de chat : il modifie des fichiers et exécute vos commandes.
- Démarrage en deux minutes : `curl -fsSL https://claude.ai/install.sh | bash`, `cd <your-repo>`, `claude`.
- Puis lancer `/init` une fois : Claude lit le projet et écrit un `CLAUDE.md` avec les commandes de build et les conventions, pour ne plus réexpliquer les bases.
- Trois tâches à essayer sur le dépôt courant : corriger un test instable dans [fichier] ; expliquer comment [module] gère [X] ; relire le diff de travail et signaler ce qui est risqué avant le push.
- Où va votre code : Claude Code tourne dans votre terminal et parle directement à l'API d'Anthropic. Avec un forfait Team ou Enterprise, Anthropic n'utilise pas votre code ni vos prompts pour entraîner ses modèles (liens data-usage et security).
- Questions : `#claude-code`, surveillé cette semaine par [nom du responsable].
- P.-S. : extension VS Code et plugin JetBrains disponibles, même agent, sans terminal.

#### Onglet « Slack ou Teams »

```markdown
🚀 *Claude Code is live for [team]*

AI coding agent, runs in your terminal, reads your repo, does real work:
bugs, refactors, tests, PRs.

`curl -fsSL https://claude.ai/install.sh | bash` → `cd your-repo` → `claude`

*First thing to try* → run `/init`, then: "the test in [file] is flaky,
figure out why and fix it."

🔒 Runs in your terminal, talks directly to Anthropic's API. On our Team or
Enterprise plan, your code and prompts are not used to train models.
Data usage → https://code.claude.com/docs/en/data-usage

📚 Quickstart · VS Code · Free 1-hr course
   https://code.claude.com/docs/en/quickstart
   https://code.claude.com/docs/en/vs-code
   https://academy.claude.com/courses/claude-code-in-action

Questions → this thread. [Owner] is on point.
```

Contenu en français : version courte de l'annonce — agent de codage dans le terminal qui lit le dépôt et fait du vrai travail (bugs, refactorisations, tests, PR) ; installation en une ligne (`curl … | bash` → `cd your-repo` → `claude`) ; première chose à essayer : `/init` puis « le test dans [fichier] est instable, trouve pourquoi et corrige-le » ; rappel sécurité (API Anthropic en direct, pas d'entraînement sur le code et les prompts avec Team/Enterprise, lien data-usage) ; liens Quickstart, VS Code et cours gratuit d'une heure (Claude Code in Action sur academy.claude.com) ; questions dans le fil, [responsable] en charge.

### Variante sponsor exécutif

À envoyer par votre dirigeant sponsor (CTO, CIO, SVP Engineering…), sous son nom et depuis son compte. Les lancements envoyés au nom d'un dirigeant obtiennent systématiquement de meilleurs taux d'ouverture et une activation plus rapide la première semaine que le même message envoyé par un administrateur ou une équipe outillage. Cela signale une priorité de l'entreprise plutôt qu'une expérience facultative.

Cette version est volontairement réduite à une seule demande : l'installer et l'utiliser sur une vraie tâche. Le rôle du dirigeant est de faire passer la demande ; l'annonce standard et `#claude-code` s'occupent du « comment ».

#### Onglet « Email »

```text
Subject: One thing I'd like every engineer to try this week

Team,

We have turned on Claude Code for all of engineering. It is an AI agent
that works directly in your terminal, on your actual codebase, and the
early results from teams already using it are strong enough that I want
everyone on it this week.

I am asking for ten minutes:

    curl -fsSL https://claude.ai/install.sh | bash
    cd <your-repo>
    claude

Then hand it one real task: the bug you have been putting off, or "walk me
through how [module] works."

That is the whole ask. [Owner name] and team are in #claude-code for
anything you hit along the way.

- [Exec Name]
  [Title]
```

Contenu en français : objet « Une chose que j'aimerais que chaque ingénieur essaie cette semaine » ; Claude Code est activé pour toute l'ingénierie ; les premiers résultats des équipes qui l'utilisent sont assez solides pour que le dirigeant veuille que tout le monde s'y mette cette semaine ; demande de dix minutes (les trois commandes d'installation) puis lui confier une vraie tâche (le bug repoussé, ou « explique-moi comment fonctionne [module] ») ; c'est toute la demande ; [responsable] et l'équipe sont dans `#claude-code` ; signature [nom du dirigeant], [titre].

#### Onglet « Slack ou Teams »

```markdown
📣 *From [Exec Name]: one thing to try this week*

We have turned on *Claude Code* for all of engineering. Early results are
strong enough that I am asking everyone to give it ten minutes on real
work this week.

`curl -fsSL https://claude.ai/install.sh | bash` → `cd your-repo` →
`claude` → hand it one real task.

That's it. Questions → #claude-code.
```

Contenu en français : « De la part de [dirigeant] : une chose à essayer cette semaine » ; Claude Code activé pour toute l'ingénierie ; dix minutes sur du vrai travail demandées à chacun ; installation → `cd` → `claude` → une vraie tâche ; questions dans `#claude-code`.

### Variante groupe pilote

À utiliser pour un déploiement progressif. À envoyer uniquement à la cohorte pilote.

```text
Subject: You're in the Claude Code pilot

[Name / team],

You are in the first wave of Claude Code at [company]. We picked this group
because you will put it on real problems and tell us the truth about it.

The ask: use it on at least one real task this week, then drop a note in
#claude-code-pilot covering what worked, what was annoying, and what
surprised you. That feedback decides how we roll it out to everyone else.

[Continue with "Get running in two minutes" from the standard announcement]

One extra thing for pilots: on your first multi-file change, press Shift+Tab
until you see "plan". Claude will lay out what it intends to do
without editing your source. It is the fastest way to calibrate how much to
trust it.
```

Contenu en français : objet « Vous faites partie du pilote Claude Code » ; vous êtes dans la première vague chez [entreprise], choisis parce que vous l'utiliserez sur de vrais problèmes et direz la vérité ; demande : l'utiliser sur au moins une vraie tâche cette semaine puis poster dans `#claude-code-pilot` ce qui a marché, ce qui a agacé et ce qui a surpris — ce retour décide du déploiement général ; [reprendre la partie « Get running in two minutes » de l'annonce standard] ; conseil pilote : lors du premier changement multi-fichiers, appuyer sur **Shift+Tab** jusqu'à voir « plan » — Claude expose ce qu'il compte faire sans modifier le code source ; c'est le moyen le plus rapide de calibrer la confiance à lui accorder.

### Message privé de recrutement de champions

Après le lancement, envoyez un message privé aux deux ou trois personnes les plus actives dans `#claude-code`.

```text
Hey [name], your #claude-code posts are doing more for adoption than my
announcement did. A couple of people told me your [thread / screenshot]
was why they actually tried it.

Want to make that semi-official? Low lift: mostly keep posting what you
are posting, plus first crack at new features and a direct line to the
Anthropic team. I can share a short playbook if you're in.
```

Contenu en français : vos messages dans `#claude-code` font plus pour l'adoption que l'annonce ; plusieurs personnes disent avoir essayé grâce à votre [fil / capture d'écran] ; proposition de rendre cela semi-officiel, avec peu d'effort : continuer à poster, plus un accès en avant-première aux nouvelles fonctionnalités et une ligne directe avec l'équipe Anthropic ; un court guide (« playbook ») peut être partagé en cas d'accord.

## Campagne d'astuces

Messages Slack ou Teams prêts à coller, conçus pour stimuler l'activation des fonctionnalités après le lancement. Chacun suit le même schéma : une accroche, le bénéfice, une invitation « essayez maintenant » et un lien vers la doc. Diffusez-les au compte-gouttes, un ou deux par semaine dans `#claude-code`, ou choisissez ceux qui correspondent aux lacunes de votre équipe. Ils sont autonomes, sans ordre imposé.

### Démarrer

**Choisir le bon modèle**

```markdown
🎯 *Tip: Match the model to the moment*

Using Opus to fix a typo burns compute. Using Haiku for a 12-file refactor
is asking for a re-do.

Claude Code runs on the same models as the Claude app, and you can switch
mid-session. *Sonnet* is the workhorse default for everyday feature work,
bugs, tests, and reviews. Reach for *Opus* on large refactors, gnarly
debugging, or anything high-stakes. Drop to *Haiku* for quick questions,
formatting, and mechanical edits where speed wins.

*Fable* is the most
capable model for your hardest, longest-running tasks; it is not the
default, so select it with `/model fable`, and note that cybersecurity and
biology content falls back to Opus automatically. Opus 5.5, Sonnet 5.5, and
Opus 5 run their own checks too: flagged content switches to an earlier model
in the same family, except that flagged biology content on Opus 5 or Sonnet
5.5 is refused.

*Try it now:* type `/model` and pick Sonnet if you haven't already. It is
the right default for most tasks.

📖 Model configuration → https://code.claude.com/docs/en/model-config
```

En bref : utiliser Opus pour une coquille gaspille du calcul, Haiku pour une refactorisation de 12 fichiers mène à refaire le travail. Claude Code utilise les mêmes modèles que l'app Claude et on peut changer en cours de session. **Sonnet** = modèle par défaut polyvalent (fonctionnalités, bugs, tests, revues) ; **Opus** = grosses refactorisations, débogage ardu, enjeux élevés ; **Haiku** = questions rapides, formatage, modifications mécaniques. **Fable** = le modèle le plus capable pour les tâches les plus dures et les plus longues, non activé par défaut (`/model fable`) ; le contenu cybersécurité et biologie bascule automatiquement vers Opus. Opus 5.5, Sonnet 5.5 et Opus 5 font aussi leurs propres contrôles : un contenu signalé bascule vers un modèle antérieur de la même famille, sauf le contenu biologique signalé sur Opus 5 ou Sonnet 5.5, qui est refusé. Essayez : `/model` puis choisir Sonnet. Lien : [Model configuration](https://code.claude.com/docs/en/model-config).

| Modèle | Idéal pour |
| - | - |
| Fable | Les tâches les plus difficiles et les plus longues. Sur activation uniquement : sélectionnez-le avec `/model fable`. Un contenu de cybersécurité ou de biologie déclenche un [basculement automatique vers Opus](https://code.claude.com/docs/en/model-config#automatic-model-fallback) |
| Opus | Refactorisations à grande échelle, débogage complexe, décisions d'architecture, changements à fort enjeu. Sur Opus 5.5 et Opus 5, un contenu de cybersécurité ou de biologie déclenche un [basculement automatique de modèle ou un refus](https://code.claude.com/docs/en/model-config#automatic-model-fallback) |
| Sonnet | Développement de fonctionnalités au quotidien, corrections de bugs, tests, documentation, revue de code. Défaut recommandé. Sur Sonnet 5.5, un contenu de cybersécurité ou de biologie déclenche un [basculement automatique de modèle ou un refus](https://code.claude.com/docs/en/model-config#automatic-model-fallback) |
| Haiku | Questions rapides, formatage, modifications mécaniques, itérations rapides |

**Gains rapides à essayer en premier**

```markdown
🚀 *Tip: Three things to try in your first 10 minutes*

Installed Claude Code but not sure what to actually ask it? Start with the
stuff that has been bugging you all week.

  - Fix something annoying: "the test in [file] is flaky, figure out why"
  - Get oriented in code you didn't write: "walk me through how [module] works"
  - Sanity-check before you push: "look at my working diff and tell me what
    looks risky"

None of these need setup. Just `cd` into your repo and run `claude`.

*Try it now:* pick the bug you have been avoiding and paste the error
message in.

📖 Quickstart → https://code.claude.com/docs/en/quickstart
```

En bref : trois choses à essayer dans les 10 premières minutes, sans configuration — corriger un agacement (test instable), s'orienter dans du code écrit par d'autres (« explique-moi comment fonctionne [module] »), vérifier avant de pousser (« regarde mon diff et dis-moi ce qui semble risqué »). Il suffit de `cd` dans le dépôt et lancer `claude`. Essayez : prendre le bug évité et coller le message d'erreur. Lien : [Quickstart](https://code.claude.com/docs/en/quickstart).

### Mémoire de projet

**`/init` et CLAUDE.md**

```markdown
📁 *Tip: Stop re-explaining your repo every session*

Telling Claude "we use pnpm, not npm" for the fifth time? There is a
one-time fix.

Run `/init` once per repo. Claude reads your project structure and writes a
CLAUDE.md file with your build commands, architecture, and conventions.
Every future session in that repo starts from this file automatically. Keep
it under two screens. It is a cheat sheet, not documentation.

*Try it now:* open your main repo, run `claude`, type `/init`. Thirty
seconds, pays off every session after.

📖 CLAUDE.md and project memory → https://code.claude.com/docs/en/memory
```

En bref : lancer `/init` une fois par dépôt ; Claude lit la structure du projet et écrit un `CLAUDE.md` (commandes de build, architecture, conventions) chargé automatiquement à chaque session future. Le garder sous deux écrans : c'est une antisèche, pas de la documentation. Essayez : ouvrir le dépôt principal, `claude`, `/init` (trente secondes). Lien : [CLAUDE.md et mémoire de projet](https://code.claude.com/docs/en/memory).

**Références @**

```markdown
📎 *Tip: Stop pasting file contents into the chat*

Copying 200 lines of a component into your prompt so Claude can "see" it?
You don't have to.

Type `@` then a file path. Claude pulls the file directly into context.
Works for whole directories too.

> the styles in @src/components/Button.tsx look off, check against
> @docs/design-system.md

*Try it now:* type `@` then Tab. Autocomplete shows you every file in reach.

📖 Referencing files → https://code.claude.com/docs/en/common-workflows
```

En bref : inutile de coller le contenu des fichiers ; taper `@` suivi d'un chemin pour que Claude charge directement le fichier (fonctionne aussi pour des répertoires entiers), ex. `@src/components/Button.tsx` comparé à `@docs/design-system.md`. Essayez : `@` puis **Tab**, l'autocomplétion montre tous les fichiers accessibles. Lien : [Referencing files](https://code.claude.com/docs/en/common-workflows).

### Contrôle et sécurité

**Modes de permission**

```markdown
🛡️ *Tip: One keystroke between "look but don't touch" and "just do it"*

Sometimes you want Claude to ask before every edit. Sometimes you just want
it to ship. You shouldn't have to pick one forever.

*Shift+Tab* cycles through how much Claude can do without asking: *Manual* (the
`default` setting value) asks before file edits and most shell commands, *acceptEdits* lets file
edits and common filesystem commands
flow through while still checking before other shell commands, and *plan*
researches and proposes changes without editing your source. Plan mode is
the trust-builder, so start there for anything touching multiple files.

*Try it now:* on your next refactor, hit Shift+Tab until you see "plan",
then describe the change. You'll get a full proposal to review.

📖 Permission modes → https://code.claude.com/docs/en/permissions
```

En bref : **Shift+Tab** fait défiler les niveaux d'autonomie — *Manual* (valeur de réglage `default`) demande avant les modifications de fichiers et la plupart des commandes shell ; *acceptEdits* laisse passer les modifications de fichiers et les commandes courantes du système de fichiers, tout en demandant avant les autres commandes shell ; *plan* recherche et propose des changements sans modifier le code source. Le mode plan bâtit la confiance : commencer par lui pour tout ce qui touche plusieurs fichiers. Essayez : à la prochaine refactorisation, Shift+Tab jusqu'à « plan », puis décrire le changement. Lien : [Permission modes](https://code.claude.com/docs/en/permissions).

**Checkpointing et `/rewind`**

```markdown
⏪ *Tip: There is an undo button for the whole conversation*

Claude went down the wrong path three turns ago and now you're untangling
it? You don't have to fix forward.

`/rewind` rolls back to an earlier point in the conversation, including the
file changes Claude made along the way. Checkpointing is automatic; you
don't set anything up.

*Try it now:* press *Esc* twice to open the rewind menu, or type `/rewind`.
Pick the point before things went sideways.

📖 Checkpointing → https://code.claude.com/docs/en/checkpointing
```

En bref : `/rewind` revient à un point antérieur de la conversation, y compris les modifications de fichiers faites par Claude ; le checkpointing est automatique, rien à configurer. Essayez : **Esc** deux fois pour ouvrir le menu de retour arrière, ou `/rewind`, puis choisir le point d'avant le dérapage. Lien : [Checkpointing](https://code.claude.com/docs/en/checkpointing).

### Connecter vos outils

**Connecteurs MCP**

```markdown
🔌 *Tip: Let Claude read your issue tracker so you don't have to paste tickets*

Copy-pasting Jira tickets into the terminal feels like a step backward.
It is.

One config file (`.mcp.json` at your project root) wires Claude into GitHub,
Jira, Linear, or whatever tracker you use. Then "what's the top-priority
issue assigned to me?" and "go ahead and fix it" happen in the same
conversation.

*Try it now:* ask Claude "set up an MCP connector for [GitHub/Jira/Linear]
in this repo". It will write the config for you.

📖 MCP connectors → https://code.claude.com/docs/en/mcp
```

En bref : un fichier de configuration (`.mcp.json` à la racine du projet) connecte Claude à GitHub, Jira, Linear ou tout autre gestionnaire de tickets ; « quel est le ticket le plus prioritaire qui m'est assigné ? » et « corrige-le » se font dans la même conversation. Essayez : demander à Claude « configure un connecteur MCP pour [GitHub/Jira/Linear] dans ce dépôt », il écrit la config. Lien : [MCP](https://code.claude.com/docs/en/mcp).

### Automatiser vos workflows

**Skills**

```markdown
⚡ *Tip: Turn that prompt you keep retyping into a command*

Typed "summarize what I worked on today from git log, format it for standup"
three times this week? That's a slash command waiting to happen.

A SKILL.md file in `.claude/skills/<name>/` becomes a reusable prompt; type
`/name` to run it. Make one the second time you type a multi-step prompt
you've typed before. Easiest path: ask Claude to make it for you.

*Try it now:* type "make me a /standup skill that summarizes what I worked
on today from git log", then run `/standup` tomorrow morning.

📖 Skills → https://code.claude.com/docs/en/skills
```

En bref : un fichier `SKILL.md` dans `.claude/skills/<name>/` devient un prompt réutilisable, lancé avec `/name`. En créer un dès la deuxième fois qu'on retape un prompt en plusieurs étapes ; le plus simple est de demander à Claude de le faire. Essayez : « fais-moi un skill /standup qui résume ce que j'ai fait aujourd'hui depuis git log », puis `/standup` le lendemain matin. Lien : [Skills](https://code.claude.com/docs/en/skills).

**Hooks**

```markdown
🔔 *Tip: Get pinged when your refactor finishes*

Sitting at your desk watching Claude work through a long task? You've got
better things to do for those eight minutes.

Hooks are shell commands that fire on Claude Code events. A Stop hook that
sends a desktop notification means you can kick off a long refactor, walk
away, and get pinged the moment it's done.

*Try it now:* ask Claude "add a Stop hook that sends a desktop notification
when you finish". It will write the script and wire it up.

📖 Hooks guide → https://code.claude.com/docs/en/hooks-guide
```

En bref : les hooks sont des commandes shell déclenchées sur des événements Claude Code ; un hook `Stop` qui envoie une notification de bureau permet de lancer une longue refactorisation, s'éloigner et être prévenu dès la fin. Essayez : demander « ajoute un hook Stop qui envoie une notification de bureau quand tu as fini » ; Claude écrit le script et le branche. Lien : [Hooks guide](https://code.claude.com/docs/en/hooks-guide).

### Développement au quotidien

**Captures d'écran et images**

```markdown
📸 *Tip: Stop describing the error dialog. Just show it.*

Typing out "there's a red box that says something about a null reference
and it's pointing at line 47-ish"? Screenshot it.

Drag a screenshot straight into the terminal and Claude sees it: error
dialogs, UI mockups, whiteboard photos, Figma exports. *Ctrl+V* pastes from
the clipboard, or *Alt+V* on Windows and WSL.

*Try it now:* next time something visual breaks, screenshot it and paste it
right into the prompt. Then just type "what's wrong here?"

📖 Working with images → https://code.claude.com/docs/en/common-workflows
```

En bref : glisser une capture d'écran directement dans le terminal pour que Claude la voie (boîtes de dialogue d'erreur, maquettes d'interface, photos de tableau blanc, exports Figma). **Ctrl+V** colle depuis le presse-papiers, **Alt+V** sous Windows et WSL. Essayez : à la prochaine casse visuelle, coller la capture puis demander « qu'est-ce qui ne va pas ? ». Lien : [Working with images](https://code.claude.com/docs/en/common-workflows).

**Workflows Git**

```markdown
🌿 *Tip: Hand off the whole git ceremony*

The fix took 5 minutes. The commit message, branch, and PR description
took 15. That ratio is wrong.

Claude handles the full git flow: commits with conventional messages,
branches, PRs with proper summaries. One ask: "fix the off-by-one, commit
with a conventional commit message, and open a PR." Reviewing someone
else's work? Paste the PR URL and ask Claude to walk you through the diff.

*Try it now:* after your next fix, instead of switching to your git client,
just type "commit this with a good message and open a PR".

📖 Creating pull requests → https://code.claude.com/docs/en/common-workflows
```

En bref : Claude gère tout le flux git — commits avec messages conventionnels, branches, PR avec résumés propres — en une seule demande (« corrige l'erreur de décalage d'un, commit avec un message conventionnel et ouvre une PR »). Pour relire le travail d'un autre : coller l'URL de la PR et demander une explication du diff. Essayez : après la prochaine correction, « commit ceci avec un bon message et ouvre une PR ». Lien : [Creating pull requests](https://code.claude.com/docs/en/common-workflows).

### Partager et passer à l'échelle

**Plugins**

```markdown
📦 *Tip: Someone probably already built that skill*

About to spend an hour building a `/deploy` command? Check if it
already exists.

Skills get bundled and shared as plugins. `/plugin` browses what's
available and installs in one step. Five minutes of browsing can save an
hour of building.

*Try it now:* type `/plugin` and scroll through. You'll find at least one
thing you didn't know you wanted.

📖 Plugins → https://code.claude.com/docs/en/plugins
```

En bref : les skills sont regroupés et partagés sous forme de plugins ; `/plugin` parcourt ce qui est disponible et installe en une étape. Cinq minutes de recherche peuvent économiser une heure de développement (ex. avant de créer une commande `/deploy`). Essayez : `/plugin` et faire défiler. Lien : [Plugins](https://code.claude.com/docs/en/plugins).

### Sécurité et administration

**Architecture de sécurité**

```markdown
🔐 *Tip: The answer to "is this safe?" for the next time you're asked*

Someone on your team is going to ask "wait, where does my code go?"
Here's the short version you can paste.

A permission mode sets which actions Claude can take without asking you
first. The CLI runs in your terminal, talks directly to Anthropic's API,
and supports optional OS-level sandboxing for shell commands. On a Team or Enterprise plan,
Anthropic does not use your code or prompts to train its models.

*Try it now:* save these two links for the next time the question comes up.
They answer most security-review questions.

📖 https://code.claude.com/docs/en/security
📖 https://code.claude.com/docs/en/data-usage
```

En bref : réponse courte à « où va mon code ? » — un mode de permission définit quelles actions Claude peut faire sans demander ; la CLI tourne dans le terminal, parle directement à l'API d'Anthropic et prend en charge un sandboxing optionnel au niveau du système d'exploitation pour les commandes shell ; avec un forfait Team ou Enterprise, Anthropic n'utilise pas votre code ni vos prompts pour entraîner ses modèles. Essayez : garder les liens [security](https://code.claude.com/docs/en/security) et [data-usage](https://code.claude.com/docs/en/data-usage), qui répondent à la plupart des questions de revue de sécurité.

**Bonnes pratiques**

```markdown
✅ *Tip: The 4 habits that separate "tried it once" from "use it daily"*

Most people who bounce off Claude Code skipped one of these. Most people
who stick did all four in week one.

  - Start in plan mode for anything touching multiple files
  - Run /init early; context compounds
  - Review diffs before committing; Claude can be confidently wrong
  - Verify changes that touch critical paths; treat it like a sharp
    junior, not an oracle

*Try it now:* if you've only done one or two of these, pick the one you're
missing and do it on your next task. Post what changed in #claude-code.

📖 Best practices → https://code.claude.com/docs/en/best-practices
```

En bref : les 4 habitudes qui distinguent « essayé une fois » de « utilisé tous les jours » — commencer en mode plan pour tout ce qui touche plusieurs fichiers ; lancer `/init` tôt (le contexte se cumule) ; relire les diffs avant de committer (Claude peut se tromper avec assurance) ; vérifier les changements qui touchent des chemins critiques (le traiter comme un junior brillant, pas comme un oracle). Essayez : adopter l'habitude manquante sur la prochaine tâche et partager le résultat dans `#claude-code`. Lien : [Best practices](https://code.claude.com/docs/en/best-practices).

## Référence rapide

### Réponses FAQ

Réponses d'une ligne aux questions qu'on vous posera le plus.

| Question | Réponse |
| - | - |
| « Ça marche dans VS Code ? » | Oui. Il existe une extension VS Code et un plugin JetBrains avec les mêmes fonctionnalités, intégrés à votre éditeur. [VS Code →](https://code.claude.com/docs/en/vs-code) |
| « Dois-je configurer quelque chose d'abord ? » | Non. Installez, puis lancez `claude` dans n'importe quel dépôt. Lancez `/init` une fois et c'est prêt. [Quickstart →](https://code.claude.com/docs/en/quickstart) |
| « Où va mon code ? » | La CLI tourne dans votre terminal et envoie le contexte à l'API d'Anthropic pour l'inférence. Avec un forfait Team ou Enterprise, votre code et vos prompts ne servent pas à entraîner les modèles. [Data usage →](https://code.claude.com/docs/en/data-usage) |
| « Peut-il voir tout mon dépôt ? » | Il lit ce à quoi vous lui donnez accès. Les lectures de fichiers dans votre répertoire de travail ne déclenchent pas de demande d'autorisation. [Permissions →](https://code.claude.com/docs/en/permissions) |
| « En quoi est-ce différent de Copilot ? » | Copilot complète des lignes. Claude Code est un agent qui lit des fichiers, exécute des commandes et fait des modifications sur plusieurs fichiers. [Overview →](https://code.claude.com/docs/en/overview) |
| « Que dois-je essayer en premier ? » | Un bug que vous repoussez parce qu'il est fastidieux. « The test in \[file] is flaky, figure out why. » [Quickstart →](https://code.claude.com/docs/en/quickstart) |

### Modèles de prompts

Partagez ces prompts de démarrage avec les ingénieurs qui ont installé l'outil mais ne savent pas quoi demander. Chacun est formulé comme il serait tapé dans une vraie session ; remplacez les éléments entre crochets par des fichiers de votre propre dépôt.

| Tâche | Prompt (original) | Traduction |
| - | - | - |
| Corriger un bug | "the tests in \[file] are failing, figure out why and fix it" | « les tests dans [fichier] échouent, trouve pourquoi et corrige » |
| Comprendre du code | "walk me through how \[module] works, then tell me where the entry point is" | « explique-moi comment fonctionne [module], puis dis-moi où est le point d'entrée » |
| Refactorisation sûre | "refactor \[module] to \[goal], use plan mode so I can review first" | « refactorise [module] pour [objectif], utilise le mode plan pour que je relise d'abord » |
| Écrire des tests | "write tests for \[file] that cover the edge cases around \[scenario]" | « écris des tests pour [fichier] couvrant les cas limites autour de [scénario] » |
| Relire avant commit | "look at my working diff and tell me what looks risky" | « regarde mon diff de travail et dis-moi ce qui semble risqué » |
| Ouvrir une PR | "fix \[issue], write a conventional commit, and open a PR with a summary" | « corrige [ticket], écris un commit conventionnel et ouvre une PR avec un résumé » |
| Créer un skill | "make me a /ship skill that runs tests and lint before commit" | « crée-moi un skill /ship qui lance les tests et le lint avant le commit » |
| Déboguer une stack trace | "here's the stack trace, find the root cause, don't just paper over it" | « voici la stack trace, trouve la cause racine, ne te contente pas de masquer le problème » |

> **Astuce** : Claude Code sort de nouvelles versions fréquemment. Vérifiez les détails propres à une version sur la [page d'accueil de la documentation](https://code.claude.com/docs/en/overview) avant de diffuser en interne.
