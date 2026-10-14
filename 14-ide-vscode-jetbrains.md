# 14 — VS Code et JetBrains

[← Sommaire](README.md)

## Extension VS Code (et forks : Cursor, etc.)

### Ouvrir Claude Code

| Où cliquer | Effet |
|------------|-------|
| **Icône Spark** dans la barre d'outils de l'éditeur (en haut à droite, fichier ouvert) | Ouvre le panneau Claude |
| **Icône Spark** dans la barre d'activité (à gauche) | Liste des sessions |
| **Palette de commandes** (`Cmd/Ctrl+Shift+P` → « Claude Code ») | Toutes les commandes de l'extension |
| **✻ Claude Code** dans la barre d'état (en bas à droite) | Ouvre Claude même sans fichier ouvert |

Le panneau se **déplace** par glisser-déposer : barre latérale secondaire (droite), barre latérale principale (gauche) ou zone d'édition (comme un onglet). À la première ouverture : écran **Sign in**, puis une checklist **Learn Claude Code** (bouton **Show me**). « Claude Code: Open Walkthrough » pour une visite guidée.

### Relire les modifications

- En **Manual**, chaque modification s'affiche en **diff côte à côte** : accepter, refuser, ou dire à Claude quoi faire à la place ; on peut éditer le contenu proposé directement.
- Boutons **Accept this change** / **Reject this change** sous chaque changement (et commandes « Accept/Reject Change at Cursor »).
- En **Auto** ou **Edit automatically**, Claude modifie sans demander.

### La zone de saisie

| Élément | Rôle |
|---------|------|
| **Indicateur de mode** (bas de la zone) | Auto, Manual, Plan, Edit automatically (et Bypass si autorisé) ; en mode Plan, le plan s'ouvre en document Markdown commentable ; `/plan`, `/plan open` |
| **Nom du modèle** (bas de la zone) | Sélecteur de modèle, ligne **Effort**, interrupteur **Ultracode** |
| **Menu `/`** | Joindre des fichiers, changer de modèle, réflexion étendue, et section **Customize** : serveurs MCP, commandes, styles de sortie, hooks, mémoire, instructions (`CLAUDE.md`), permissions, plugins, Slash commands, Status, Sandbox, Claude in Chrome ; section **Context** : Export conversation, Bookmarks ; section **Settings** : Enable Remote Control for all sessions, Focus view, Sign out ; **Report a problem** |
| **Indicateur de contexte** | Part de la fenêtre de contexte utilisée |
| **Horloge du cache** | Temps restant avant expiration du cache de prompt (rouge = expiré, prochaine réponse plus lente et plus chère) |
| **Compteur d'agents** (« 2 agents ») | Ouvre la **carte des agents** : arbre des sous-agents avec statut, durée, jetons ; voir leur transcription ou les arrêter ; tâches de fond (aussi via `/tasks`) |
| **Run in background** | Sous un appel d'outil long, l'envoyer en arrière-plan |

Autres actions : **Copy response** et **Bookmark response** au survol d'une réponse (`/bookmarks` pour les revoir), `/btw` (panneau de questions annexes), `Shift+Entrée` pour une nouvelle ligne, `Ctrl+O` pour déplier la réflexion, fichiers envoyés par Claude affichés dans la conversation.

### Donner du contexte

- `@fichier` ou `@dossier/` avec correspondance approximative (`@auth` trouve `auth.js`, `AuthService.ts`…).
- **Sélection** dans l'éditeur : Claude la voit automatiquement ; `Option+K` / `Alt+K` insère une référence `@app.ts#5-10`.
- Le **fichier ouvert** est joint automatiquement (réglage `attachOpenFile`).
- `@terminal:nom` : inclure la sortie d'un terminal.
- **Images** : coller dans la zone ; **fichiers** : `Shift`+glisser.
- PDF volumineux : demander des pages précises.

### Sessions

- Bouton **Session history** : recherche, reprise ; onglets **Local** et **Web** (reprendre des sessions cloud de claude.ai ou des sessions Remote Control).
- Titres générés automatiquement ; **renommer**, **archiver** au survol ; archivage automatique après 14 jours d'inactivité (réglable).
- **Groupes** de sessions (clic droit), filtres **Active** et par statut (Needs input, Working, Completed, Open, Closed).
- Plusieurs conversations : **Open in New Tab** / **Open in New Window** ; point coloré sur l'icône Spark : **bleu** = permission en attente, **orange** = Claude a fini pendant que l'onglet était caché.
- Après rechargement de la fenêtre, la conversation revient et Claude reprend l'étape interrompue.
- **Points de contrôle** : survoler un message → Fork conversation from here / Rewind code to here / Fork conversation and rewind code.
- Mode **terminal** : réglage **Use Terminal** pour l'interface CLI dans VS Code.

### Raccourcis et commandes VS Code

| Commande | Raccourci | Rôle |
|----------|-----------|------|
| Focus Input | `Cmd+Esc` / `Ctrl+Esc` | Basculer le focus entre l'éditeur et Claude |
| Open in New Tab | `Cmd+Shift+Esc` / `Ctrl+Shift+Esc` | Nouvelle conversation en onglet |
| New Conversation | `Cmd+N` / `Ctrl+N` | Nouvelle conversation (si activé) |
| Reopen Closed Session | `Cmd+Shift+T` / `Ctrl+Shift+T` | Rouvrir le dernier onglet Claude fermé |
| Insert @-Mention Reference | `Option+K` / `Alt+K` | Référence au fichier et à la sélection |
| Toggle Focus view | `Ctrl+Option+F` / `Ctrl+Alt+F` | Masquer/afficher l'activité des outils |
| Open in Side Bar / Terminal / New Window | — | Choisir où ouvrir Claude |
| Accept / Reject Change at Cursor | — | Revue changement par changement |
| Rename Session Tab, Add Session Tab to Group, Mark Session as Unread | — | Organisation |
| Focus last message, Show Logs, Logout | — | Divers |

Lien d'ouverture depuis d'autres outils : `vscode://anthropic.claude-code/open`.

### Réglages de l'extension (Extensions › Claude Code)

`useTerminal`, `initialPermissionMode`, `preferredLocation` (`sidebar`/`panel`), `lockEditorGroups`, `autosave`, `attachOpenFile`, `useCtrlEnterToSend`, `scrollToBottomOnSend`, `showMessageTimestamps`, `spinnerVerbs`, `enableNewConversationShortcut`, `archiveInactiveSessions`, `continueAfterReload`, `hideOnboarding`, `focusView`, `respectGitIgnore`, `usePythonEnvironment`, `environmentVariables`, `disableLoginPrompt`…

### Extension vs CLI

| Fonction | CLI | Extension VS Code |
|----------|-----|-------------------|
| Commandes et skills | Toutes | Une partie (tapez `/`) |
| Serveurs MCP | Oui | Oui (`/mcp` dans le panneau) |
| Points de contrôle | Oui | Oui |
| Raccourci `!` (shell) | Oui | Non |
| Complétion `Tab` | Oui | Non |

Pour tout avoir : lancer `claude` dans le terminal intégré de VS Code (il s'intègre automatiquement : diffs dans l'éditeur, diagnostics) ; depuis un terminal externe, `/ide` pour se connecter. L'extension et le CLI partagent l'historique (`claude --resume`). L'extension embarque son propre CLI : installez le CLI séparément pour avoir `claude` dans le shell.

### Serveur MCP intégré de l'IDE

L'extension expose à Claude des outils de l'IDE (diagnostics, diff, sélection…), ce qui lui permet par exemple de lire les erreurs de lint et de compilation affichées dans VS Code.

## Plugin JetBrains

IDE pris en charge : IntelliJ IDEA, PyCharm, Android Studio, WebStorm, PhpStorm, GoLand, etc. Le plugin lance `claude` dans le **terminal intégré** de l'IDE (installez le plugin **et** le CLI).

| Fonction | Détail |
|----------|--------|
| **Lancement rapide** | `Cmd+Esc` / `Ctrl+Esc`, ou bouton Claude Code |
| **Diffs dans l'IDE** | Les modifications s'ouvrent dans la visionneuse de diff de l'IDE (`/config` › Diff tool : `auto` ou `terminal`) |
| **Contexte de sélection** | La sélection et l'onglet actif sont partagés avec Claude |
| **Références de fichier** | `Cmd+Option+K` / `Alt+Ctrl+K` insère `@src/auth.ts#L1-99` |
| **Diagnostics** | Claude lit les inspections de l'IDE (lint, erreurs de syntaxe) |
| **Terminal externe** | `/ide` pour se connecter à l'IDE |
