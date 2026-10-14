# 13 — L'application Claude Desktop (onglet Code)

[← Sommaire](README.md)

L'application **Claude Desktop** (macOS, Windows x64/ARM64, Linux en bêta) a trois onglets : **Chat** (conversations), **Cowork** (Dispatch et travail agentique long) et **Code** (développement, c'est-à-dire Claude Code avec une interface graphique). Chaque conversation de l'onglet Code est une **session** avec son historique et son dossier de projet.

## Démarrer une session : les 4 réglages de la zone de saisie

| Contrôle | Choix |
|----------|-------|
| **Environnement** | **Local** (votre machine), **Cloud** (continue même app fermée), **SSH** (machine distante), **WSL** (Windows) |
| **Dossier du projet** | Le dossier ou dépôt de travail (plusieurs dépôts en cloud) ; option **worktree** à côté de la branche pour isoler la session |
| **Modèle** | Menu à côté du bouton Envoyer (`Cmd+Shift+I`) ; effort (`Cmd+Shift+E`) |
| **Mode de permission** | Sélecteur à côté du bouton Envoyer (`Cmd+Shift+M`) : **Manual**, **Accept edits**, **Plan**, **Auto**, **Bypass permissions** (si activé dans les réglages) |

## La zone de saisie et ses boutons

- **Entrée** pour envoyer ; **bouton Stop** (ou `Esc`) pour interrompre ; tapez une correction et `Entrée` pour réorienter sans attendre.
- **Bouton `+`** : joindre des fichiers (images, PDF…), choisir des **skills**, des **connecteurs**, des **plugins**.
- **`@`** pour mentionner un fichier (sessions locales et SSH) ; **glisser-déposer** des fichiers.
- **Suggestion de prompt** grisée : `Tab` ou `→` pour l'accepter.
- **Anneau d'usage** à côté du sélecteur de modèle : contexte utilisé et consommation du forfait.
- Commandes `/` disponibles (celles qui ouvrent un panneau de terminal répondent différemment ; `/config` ouvre Réglages › Claude Code).

## Les volets (panes)

L'onglet Code est organisé en volets que l'on **glisse** par leur en-tête, **redimensionne**, **détache dans une fenêtre** séparée puis réancre : **chat**, **diff**, **navigateur**, **terminal**, **fichier**, **plan**, **tâches**, **sous-agent**, **simulateur iOS** (macOS).

| Volet | Ce qu'on y fait |
|-------|-----------------|
| **Diff** | Indicateur `+12 -1` cliquable → liste des fichiers à gauche, changements à droite ; **cliquer une ligne pour commenter**, puis `Cmd+Entrée` pour envoyer tous les commentaires à Claude |
| **Navigateur** (Browser) | Aperçu de votre app (Claude lance le serveur de dev, prend des captures, inspecte le DOM, clique, remplit des formulaires, corrige) ; onglets vers des sites externes ; ouverture des HTML, PDF, images, vidéos ; menu **Dev servers** ; **⋮** › Keep cookies / Clear browsing data / Open links in built-in browser ; `Cmd+Shift+S` pour sélectionner un élément de la page |
| **Terminal** | Terminal intégré dans le dossier de la session (`Ctrl+\``) |
| **Fichier** | Cliquer un chemin pour l'ouvrir, faire une retouche, **Save** |
| **Tâches** | Travail de fond de la session (sous-agents, commandes, workflows) ; ouvert via **⋮** › Background tasks |
| **Simulateur iOS** | S'ouvre seul quand Claude lance votre app iOS ; interactif ; un simulateur par session |

- **Clic droit** sur un chemin : Attach as context, Open in (VS Code, Cursor, Zed…), Show in Finder/Explorer, Copy path.
- **Modes d'affichage** (`Ctrl+O` ou menu de la session › Transcript view) : **Normal** (outils résumés), **Thinking** (+ réflexion de Claude), **Verbose** (tout).
- Configuration des serveurs d'aperçu : `.claude/launch.json` (commande, port, URL, vérification automatique des changements).

## Revue de code et pull requests

- `/code-review` → carte **Code review** groupée par fichier : **Walk through in diff** (parcourir les remarques dans le diff, **Fix this one**), **Apply fixes**.
- Après ouverture d'une PR, **barre d'état CI** dans la session ; bouton **CI** pour activer :
  - **Auto-fix CI & address comments** : Claude corrige les échecs de CI et répond aux commentaires de revue ;
  - **Auto-merge when ready** : fusion (squash) quand tout est vert.
- Nécessite le CLI GitHub `gh` installé et connecté.

## Gérer plusieurs sessions

- **+ New session** (`Cmd+N`), passer d'une session à l'autre (`Ctrl+Tab` / `Ctrl+Shift+Tab`, `Cmd+Shift+]`/`[`).
- **Vue côte à côte** : `Cmd`+clic sur une session de la barre latérale.
- Filtres par statut, projet, environnement ; regroupement par projet ; renommer en cliquant le titre.
- **Archiver** (icône au survol) ; **auto-archivage** après fusion/fermeture de la PR (Réglages › Claude Code).
- Réglages des worktrees : emplacement, **préfixe de branche** ; `.worktreeinclude` pour copier `.env` etc.
- **Notifications** système quand une session termine.
- **Side chat** (`Cmd+;` ou `/btw`) : question qui utilise le contexte sans le modifier.
- **Travail entre sessions** : « quelle session a touché au refactoring d'auth ? », « dis à la session API que le schéma a changé » ; Claude peut proposer une nouvelle tâche sous forme de **puce** cliquable (nouvelle session avec son worktree).
- **Open in** (menu de la session) : continuer dans le **Cloud** ou ouvrir le dossier dans un éditeur.
- **Remote Control** : interrupteur par session (icône d'ordinateur portable à côté du titre) pour la piloter depuis le téléphone ou claude.ai ; option « Connect new sessions to Remote Control ».
- **Projects** (barre latérale) : coordonner de nombreuses sessions cloud autour d'un même objectif.
- **Routines** (barre latérale) : tâches planifiées locales ou cloud.

## Computer use (contrôle de l'ordinateur)

Claude peut **ouvrir vos applications, voir l'écran, cliquer et taper** (aperçu de recherche, macOS et Windows, forfaits Pro/Max).

- Activer : Réglages › This computer › System › **Enable computer use** (+ permissions Accessibilité et Enregistrement d'écran sur macOS).
- Claude préfère toujours l'outil le plus précis : connecteur → Bash → Chrome → simulateur iOS → computer use en dernier recours.
- Première utilisation d'une app : **Allow for this session** / **Deny**. Niveaux fixes : **View only** (navigateurs, plateformes de trading), **Click only** (terminaux, IDE), **Full control** (le reste).
- Options : **Denied apps**, **Unhide apps when Claude finishes** ; sur macOS, peut tourner en arrière-plan.

## Dispatch

**Dispatch** est une conversation persistante dans l'onglet Cowork : vous lui envoyez une tâche (y compris depuis le téléphone) et il décide comment la traiter ; le travail de développement devient une session Code marquée **Dispatch**, avec notification push à la fin ou quand une approbation est nécessaire (Pro/Max).

## Sessions SSH et WSL

- **SSH** : travailler sur une machine distante (utilise votre `~/.ssh/config`, hôtes de rebond) ; une tâche peut continuer pendant que Desktop est déconnecté ; liens pour ouvrir une session SSH ; connexions préconfigurées et liste d'hôtes autorisés pour les équipes.
- **WSL** : choisir une distribution WSL comme environnement sous Windows.

## Raccourcis de l'onglet Code

| Raccourci (Windows : `Ctrl` à la place de `Cmd`) | Action |
|----------|--------|
| `Cmd+/` | Afficher tous les raccourcis |
| `Cmd+N` / `Cmd+W` | Nouvelle session / fermer la session |
| `Ctrl+Tab`, `Ctrl+Shift+Tab`, `Cmd+Shift+]`, `Cmd+Shift+[` | Session suivante / précédente |
| `Esc` | Arrêter la réponse |
| `Tab` / `→` | Accepter la suggestion de prompt |
| `Cmd+Shift+D` | Volet diff |
| `Cmd+Shift+B` | Volet navigateur |
| `Cmd+Shift+S` | Sélectionner un élément dans le navigateur |
| `` Ctrl+` `` | Volet terminal |
| `Cmd+\` | Fermer le volet actif |
| `Cmd+;` | Side chat |
| `Ctrl+O` | Changer de mode d'affichage |
| `Cmd+Shift+M` / `Cmd+Shift+I` / `Cmd+Shift+E` | Menu mode de permission / modèle / effort |
| `1`–`9` | Choisir un élément dans un menu ouvert |

Dans Desktop, `Shift+Tab` ne fait **pas** défiler les modes de permission.

## Réglages (Settings › Claude Code)

Prompt suggestions, Browser tools, Worktree location, Branch prefix, Auto-archive after PR merge or close, Connect new sessions to Remote Control, Allow bypass permissions mode (Pro/Max), etc. L'app lit les mêmes fichiers de réglages que le CLI (`~/.claude/settings.json`, `.claude/settings.json`…).

## CLI vs Desktop

| Fonction | CLI | Desktop |
|----------|-----|---------|
| Modes de permission | Tous, dont `dontAsk` | Manual, Accept edits, Plan, Auto (+ Bypass si activé) |
| Fournisseurs tiers | Bedrock, Google Cloud, Foundry | API Anthropic (gateway possible) |
| MCP | Fichiers de réglages | Interface **Connectors** ou fichiers |
| Plugins | `/plugin` | Gestionnaire graphique |
| Pièces jointes | Non (chemins, images collées) | Images, PDF |
| Sessions multiples | Plusieurs terminaux | Onglets de la barre latérale |
| Tâches récurrentes | Cron, CI, `/loop` | Tâches planifiées |
| Computer use | macOS (via `/mcp`) | macOS et Windows |
| Équipes d'agents | Oui | Non (utiliser les workflows) |
| Scripts / SDK | `-p`, Agent SDK | Non |

Passer du CLI à Desktop : `/desktop` ou `claude --desktop [--continue]`.
