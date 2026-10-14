# 15 — Web (cloud), mobile, Remote Control, Projects et artifacts

[← Sommaire](README.md)

## Claude Code dans le cloud (claude.ai/code)

Les **sessions cloud** tournent sur une infrastructure gérée par Anthropic (ou sur l'infrastructure de votre organisation avec un **environnement auto-hébergé**). Elles **continuent même si vous fermez le navigateur ou éteignez l'ordinateur**.

### Démarrer

1. Aller sur **claude.ai/code** (ou l'onglet Code de l'app Desktop/mobile, choix **Cloud**).
2. Connecter GitHub : **app GitHub Claude** (dépôts où l'app est installée + dépôts publics) ou **`/web-setup`** depuis le terminal (utilise votre jeton `gh`).
3. Choisir un **dépôt** (plusieurs possibles), une **branche**, un **environnement**, décrire la tâche.

Les **environnements cloud** définissent l'accès réseau (**None**, **Trusted** — par défaut : registres de paquets, GitHub, SDK cloud —, **Full**, ou **Custom** avec votre propre liste de domaines), les variables d'environnement, les secrets réseau et un **script d'installation** (dépendances, outils). Choisir l'environnement par défaut depuis le CLI : `/remote-env`.

### Dans une session cloud

- **Diff** `+42 -18` : vue des changements, **commentaires en ligne** envoyés avec le prochain message.
- **Modes de permission** : Accept edits, Plan, Auto (pas de Bypass).
- Commandes disponibles : celles qui produisent du texte (`/compact`, `/context`…) ; pas celles du terminal comme `/plugin` ou `/resume` (`/clear` → nouvelle session dans la barre latérale).
- **Retirer un message en file** : ✕ sur le message.
- **Partager** une session (visibilité selon le forfait : organisation pour Team/Enterprise, lien pour Pro/Max), **archiver**, **supprimer**.
- **Auto-fix des PR** : Claude surveille la PR, corrige les échecs de CI et répond aux commentaires de revue (aussi via `/autofix-pr` depuis le CLI).
- Les skills, plugins et réglages viennent du dépôt (`.claude/`) et de votre compte claude.ai (skills et connecteurs synchronisés).

### Passer du terminal au cloud et inversement

| Action | Commande |
|--------|----------|
| Créer une session cloud depuis le terminal | `claude --cloud "Corrige le bug de connexion"` |
| Envoyer un message de suivi à une session cloud | `claude --cloud <id-session> -p "ajoute des tests"` |
| Ramener une session cloud en local | `/teleport` (alias `/tp`) ou `claude --teleport` |
| Dépôt local sans GitHub | `claude --cloud` empaquette et téléverse le dépôt |

Conseil : **planifier en local, exécuter dans le cloud** (mode plan, puis `--cloud`). Le téléport exige un état git propre, le même dépôt, une branche poussée et le même compte.

## Application mobile (iOS / Android / iPad)

Depuis l'app Claude (onglet Code) :

| Fonction | Connexion à | Quand |
|----------|-------------|-------|
| **Sessions cloud** | Le cloud | Tâche sur un dépôt GitHub qui doit continuer seule |
| **Projects** | Une conversation qui coordonne plusieurs fils de travail | Un flux de tâches liées |
| **Remote Control** | Une session Claude Code sur votre ordinateur | Besoin de vos fichiers, outils, serveurs MCP locaux |
| **Dispatch** | L'app Desktop de votre ordinateur | Envoyer une tâche et laisser Dispatch décider (Pro/Max) |

**Notifications push** quand une tâche longue se termine ou qu'une décision est nécessaire (« préviens-moi quand les tests passent »). `/mobile` affiche un QR code pour installer l'app.

## Remote Control

Piloter **depuis le téléphone, une tablette ou n'importe quel navigateur** une session Claude Code qui tourne **sur votre machine** (le code et les fichiers restent en local).

| Lancement | Effet |
|-----------|-------|
| `claude --remote-control "Mon projet"` (ou `--rc`) | Session interactive aussi pilotable à distance |
| `/remote-control` (alias `/rc`) | Rendre la session actuelle pilotable |
| `claude remote-control [--name] [--spawn same-dir\|worktree\|session] [--capacity N]` | Mode serveur : crée des sessions à la demande depuis claude.ai/l'app |
| `/config` › activer pour toutes les sessions (`remoteControlAtStartup`) | Connexion automatique |

- Indicateur `/rc active` dans le terminal (lien vers la session sur claude.ai).
- Depuis l'appareil connecté : envoyer des messages, répondre aux permissions, changer le mode, l'effort, le modèle, recevoir des fichiers (`SendUserFile`) et des notifications push.
- Sécurité : connexions **sortantes HTTPS uniquement**, aucun port entrant ouvert.
- Désactiver : `"disableRemoteControl": true`.

## Projects

Un **projet** est une conversation de coordination avec Claude : vous lui envoyez un flux de tâches liées, il lance des **fils** (threads) — des sessions cloud parallèles qui partagent dépôts, instructions et mémoire — et suit leur avancement.

- Création sur claude.ai/code, dans l'onglet Code de Desktop ou dans l'app mobile ; ou **Continue as project** / **Move to project** depuis une session cloud.
- **Overview** : tous les fils et ceux qui ont besoin de vous ; onglets **Library** (fichiers ajoutés et produits), pull requests…
- Réglages : instructions du projet, **Thread model**, **Thread effort**, nombre de fils en parallèle, proposition des fils avant lancement.

## Artifacts : publier une page web à partir de la session

Un **artifact** transforme le travail de Claude en **page web interactive privée** sur claude.ai : rapport, tableau de bord, explication d'un changement, comparaison d'options, présentation, maquette.

- **Créer** : demandez-le (« fais-moi une page qui compare les trois options ») ; Claude peut aussi en publier de lui-même quand c'est plus clair qu'un texte.
- **Mettre à jour** : Claude republie à la même URL (un suivi de tâche longue peut se rafraîchir).
- **Retrouver** : `/artifacts` (ou la galerie claude.ai/code/artifacts).
- **Partager** : bouton **Share** de la page (privé par défaut ; organisation ; lien public si autorisé) ; **éditeurs** possibles sur Team/Enterprise.
- **Commentaires** : les lecteurs commentent la page ; Claude peut y répondre (y compris automatiquement).
- **Capacités** : données en direct via vos **connecteurs MCP**, **téléchargement de fichiers**, état partagé entre lecteurs… (`/artifact-capabilities`).
- **Modèles** : **Slides** (`/slides`), **Design** (`/design`, canevas d'artboards exportables en PNG/PDF), **Docs** (documents Claude Docs).
- Une page autonome, sans backend. Disponible avec les forfaits Pro, Max, Team, Enterprise et une connexion `/login` (pas sur Bedrock/Google Cloud/Foundry). Désactivable par l'organisation.
