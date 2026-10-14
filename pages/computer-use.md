# Laisser Claude utiliser votre ordinateur depuis la CLI

> Source : https://code.claude.com/docs/en/computer-use · [← Index](../31-index-documentation-officielle.md)

*Activez computer use (utilisation de l'ordinateur) dans la CLI Claude Code pour que Claude puisse ouvrir des applications, cliquer, taper et voir votre écran sur macOS. Testez des applications natives, déboguez des problèmes visuels et automatisez des outils uniquement graphiques sans quitter votre terminal.*

> **Note** : computer use est un aperçu de recherche (research preview) sur **macOS** qui nécessite un forfait **Pro ou Max**. Il n'est **pas disponible** sur les forfaits Team ou Enterprise. Il exige une session interactive : il n'est donc pas disponible en mode non interactif avec le flag `-p`.

Computer use permet à Claude d'ouvrir des applications, de contrôler votre écran et de travailler sur votre machine comme vous le feriez. Depuis la CLI, Claude peut compiler une app Swift, la lancer, cliquer sur chaque bouton et faire une capture du résultat, le tout dans la même conversation que celle où il a écrit le code.

Cette page couvre le fonctionnement de computer use dans la CLI. Pour l'application Desktop sur macOS ou Windows, voir [computer use dans Desktop](https://code.claude.com/docs/en/desktop#let-claude-use-your-computer).

## Ce que vous pouvez faire avec computer use

Computer use gère les tâches qui nécessitent une interface graphique : tout ce que vous devriez normalement faire à la main en quittant le terminal.

- **Construire et valider des applications natives** : demandez à Claude de créer une app de barre de menus macOS. Claude écrit le Swift, le compile, lance l'app et clique sur chaque contrôle pour vérifier qu'elle fonctionne avant même que vous ne l'ouvriez.
- **Tests d'interface de bout en bout** : dirigez Claude vers une app Electron locale et dites « teste le parcours d'onboarding ». Claude ouvre l'app, parcourt l'inscription en cliquant et fait une capture de chaque étape. Pas de configuration Playwright, pas de harnais de test.
- **Déboguer des problèmes visuels et de mise en page** : dites à Claude « la modale est rognée sur les petites fenêtres ». Claude redimensionne la fenêtre, reproduit le bug, fait une capture, corrige le CSS et vérifie le correctif. Claude voit ce que vous voyez.
- **Piloter des outils uniquement graphiques** : interagir avec des outils de design, des panneaux de contrôle matériel, l'iOS Simulator ou des applications propriétaires qui n'ont ni CLI ni API.

## Quand computer use s'applique

Claude dispose de plusieurs moyens d'interagir avec une application ou un service. Computer use est le plus large et le plus lent ; Claude essaie donc d'abord l'outil le plus précis :

- Si vous avez un [serveur MCP](https://code.claude.com/docs/en/mcp) pour le service, Claude l'utilise.
- Si la tâche est une commande shell, Claude utilise Bash.
- Si la tâche relève du navigateur et que [Claude in Chrome](https://code.claude.com/docs/en/chrome) est configuré, Claude l'utilise.
- Si rien de tout cela ne s'applique, Claude utilise computer use.

Le contrôle de l'écran est réservé à ce que rien d'autre ne peut atteindre : applications natives, simulateurs comme l'iOS Simulator, et outils sans API.

## Activer computer use

Computer use est disponible sous la forme d'un serveur MCP intégré nommé `computer-use`. Il est désactivé par défaut tant que vous ne l'activez pas.

1. **Ouvrir le menu MCP** — dans une session Claude Code interactive, lancez :

   ```text
   /mcp
   ```

   Trouvez `computer-use` dans la liste des serveurs. Il apparaît comme désactivé.

2. **Activer le serveur** — sélectionnez `computer-use` et choisissez **Enable**. Le réglage persiste **par projet** : vous ne le faites qu'une fois pour chaque projet où vous voulez computer use.

3. **Accorder les permissions macOS** — la première fois que Claude essaie d'utiliser votre ordinateur, une invite vous demande d'accorder deux permissions macOS :
   - **Accessibility** (Accessibilité) : permet à Claude de cliquer, taper et faire défiler ;
   - **Screen Recording** (Enregistrement de l'écran) : permet à Claude de voir ce qui est à l'écran.

   L'invite contient des liens pour ouvrir le panneau Réglages Système correspondant. Accordez les deux, puis sélectionnez **Try again** dans l'invite. macOS peut exiger de redémarrer Claude Code après l'octroi de Screen Recording.

Une fois configuré, demandez à Claude quelque chose qui nécessite l'interface graphique :

```text
Build the app target, launch it, and click through each tab to make
sure nothing crashes. Screenshot any error states you find.
```

(« Compile la cible de l'app, lance-la et clique sur chaque onglet pour vérifier que rien ne plante. Fais une capture de tout état d'erreur trouvé. »)

## Approuver les applications par session

Activer le serveur `computer-use` ne donne pas à Claude l'accès à toutes les applications de votre machine. La première fois que Claude a besoin d'une application précise dans une session, une invite apparaît dans votre terminal indiquant :

- quelles applications Claude veut contrôler ;
- les éventuelles permissions supplémentaires demandées, comme l'accès au presse-papiers ;
- combien d'autres applications seront masquées pendant que Claude travaille.

Choisissez **Allow for this session** (Autoriser pour cette session) ou **Deny** (Refuser). Les approbations durent le temps de la session en cours. Vous pouvez approuver plusieurs applications à la fois lorsque Claude les demande ensemble.

Les applications à large portée affichent un avertissement supplémentaire dans l'invite pour que vous sachiez ce que vous accordez :

| Avertissement | S'applique à |
| :- | :- |
| Équivalent à un accès shell (« Equivalent to shell access ») | Terminal, iTerm, VS Code, Warp et autres terminaux et IDE |
| Peut lire ou écrire n'importe quel fichier (« Can read or write any file ») | Finder |
| Peut modifier les réglages système (« Can change system settings ») | System Settings (Réglages Système) |

Ces applications ne sont pas bloquées. L'avertissement vous permet de décider si la tâche justifie ce niveau d'accès.

Approuvez Finder pour permettre à Claude de cliquer sur le bureau, le Dock ou une fenêtre Finder.

Le niveau de contrôle de Claude varie aussi selon la catégorie d'application : les navigateurs et plateformes de trading sont en **lecture seule** (affichage uniquement), les terminaux et IDE en **clic uniquement**, et tout le reste en **contrôle complet**. Voir [les permissions d'applications dans Desktop](https://code.claude.com/docs/en/desktop#app-permissions) pour le détail complet des niveaux.

## Comment Claude travaille sur votre écran

Comprendre le déroulement aide à anticiper ce que Claude va faire et comment intervenir.

### Une session à la fois

Une seule session à la fois peut utiliser votre ordinateur. Une session prend un verrou (lock) lors de sa première action computer use et le libère lorsque la session se termine, et non lorsque la tâche se termine. L'utilisation de computer use par une deuxième session échoue avec une erreur nommant la session qui détient le verrou. Quittez d'abord cette session.

### Les applications sont masquées pendant que Claude travaille

Quand Claude commence à contrôler votre écran, les autres applications visibles sont masquées pour que Claude n'interagisse qu'avec les applications approuvées. Votre fenêtre de terminal reste visible et est exclue des captures d'écran : vous pouvez suivre la session et Claude ne voit jamais sa propre sortie.

Quand Claude termine son tour, les applications masquées sont restaurées automatiquement.

### Les captures d'écran sont réduites automatiquement

Claude Code réduit chaque capture d'écran avant de l'envoyer au modèle. Inutile de baisser la résolution de votre écran ou de redimensionner les fenêtres sur un écran Retina ou à haute résolution. Un MacBook Pro 16 pouces en résolution Retina native capture en 3456×2234 et réduit à environ 1372×887, en conservant le rapport d'aspect.

Il n'existe **aucun réglage** pour changer la taille cible. Si le texte ou les contrôles à l'écran sont trop petits pour que Claude les lise après réduction, augmentez leur taille dans l'application plutôt que de changer la résolution de l'écran.

### Arrêter à tout moment

La première fois que Claude utilise votre ordinateur à chaque tour, une notification macOS apparaît : « Claude is using your computer · press Esc to stop ». Appuyez sur `Esc` n'importe où pour interrompre immédiatement l'action en cours, ou sur `Ctrl+C` dans le terminal. Dans les deux cas, Claude s'arrête, réaffiche vos applications et vous rend la main. La session conserve le verrou computer use (voir « Une session à la fois ») jusqu'à ce qu'elle se termine.

Une seconde notification apparaît lorsque Claude a terminé.

## Sécurité et frontière de confiance

> **Avertissement** : contrairement à l'[outil Bash sandboxé](https://code.claude.com/docs/en/sandboxing), computer use s'exécute sur votre véritable bureau, avec accès aux applications que vous approuvez. Claude vérifie chaque action et signale les injections de prompt potentielles provenant du contenu affiché à l'écran, mais la frontière de confiance est différente. Voir le [guide de sécurité computer use](https://support.claude.com/en/articles/14128542) pour les bonnes pratiques.

Les garde-fous intégrés réduisent les risques sans nécessiter de configuration :

- **Approbation par application** : Claude ne peut contrôler que les applications que vous avez approuvées dans la session en cours.
- **Avertissements « sentinelles »** : les applications donnant accès au shell, au système de fichiers ou aux réglages système sont signalées avant votre approbation.
- **Terminal exclu des captures** : Claude ne voit jamais votre fenêtre de terminal, donc les invites affichées dans votre session ne peuvent pas être réinjectées dans le modèle.
- **Échappement global** : la touche `Esc` interrompt computer use depuis n'importe où, et l'appui est consommé, de sorte qu'une injection de prompt ne peut pas l'utiliser pour fermer des boîtes de dialogue.
- **Fichier de verrou** : une seule session peut contrôler votre machine à la fois.

## Exemples de workflows

Ces exemples montrent des façons courantes de combiner computer use et tâches de code.

### Valider une build native

Après des modifications d'une app macOS ou iOS, faites compiler et vérifier par Claude en une passe :

```text
Build the MenuBarStats target, launch it, open the preferences window,
and verify the interval slider updates the label. Screenshot the
preferences window when you're done.
```

(« Compile la cible MenuBarStats, lance-la, ouvre la fenêtre des préférences et vérifie que le curseur d'intervalle met à jour le libellé. Fais une capture de la fenêtre des préférences à la fin. »)

Claude exécute `xcodebuild`, lance l'app, interagit avec l'interface et rapporte ce qu'il trouve.

### Reproduire un bug de mise en page

Quand un bug visuel n'apparaît qu'à certaines tailles de fenêtre, laissez Claude le trouver :

```text
The settings modal clips its footer on narrow windows. Resize the app
window down until you can reproduce it, screenshot the clipped state,
then check the CSS for the modal container.
```

(« La modale des réglages rogne son pied de page sur les fenêtres étroites. Réduis la fenêtre jusqu'à reproduire le problème, capture l'état rogné, puis vérifie le CSS du conteneur de la modale. »)

Claude redimensionne la fenêtre, capture l'état cassé et lit les feuilles de style concernées.

### Tester un parcours dans le simulateur

Pilotez l'iOS Simulator sans écrire de XCTest :

```text
Open the iOS Simulator, launch the app, tap through the onboarding
screens, and tell me if any screen takes more than a second to load.
```

(« Ouvre l'iOS Simulator, lance l'app, parcours les écrans d'onboarding et dis-moi si un écran met plus d'une seconde à charger. »)

Claude contrôle le simulateur comme vous le feriez avec une souris. Ce parcours s'applique à la CLI ; dans l'application Desktop, la même demande ouvre le [panneau iOS Simulator](https://code.claude.com/docs/en/desktop-ios-simulator) au lieu du contrôle de l'écran.

## Différences avec l'application Desktop

La CLI et Desktop partagent le même moteur computer use, avec quelques différences :

| Fonctionnalité | Desktop | CLI |
| :- | :- | :- |
| Plateformes | macOS et Windows | macOS uniquement |
| Activation | Interrupteur dans **Settings > This computer > System** | Activer `computer-use` dans `/mcp` |
| Liste des applications refusées | Configurable dans Settings | Pas encore disponible |
| Bascule de réaffichage automatique (auto-unhide) | Facultative | Toujours activée |
| Intégration Dispatch | Les sessions lancées par Dispatch peuvent utiliser computer use | Sans objet |

## Dépannage

### « Computer use is in use by another Claude session »

Une autre session Claude Code détient le verrou, qu'elle conserve jusqu'à sa fermeture. Quittez cette session. Si l'autre session a planté, le verrou est libéré automatiquement lorsque Claude détecte que le processus ne tourne plus.

### L'invite de permissions macOS réapparaît sans cesse

macOS exige parfois le redémarrage du processus demandeur après l'octroi de Screen Recording. Quittez complètement Claude Code et démarrez une nouvelle session. Si l'invite persiste, ouvrez **System Settings > Privacy & Security > Screen Recording** et vérifiez que votre application de terminal y figure et est activée.

### `computer-use` n'apparaît pas dans `/mcp`

Le serveur n'apparaît que sur les configurations éligibles. Vérifiez que :

- vous êtes sur **macOS**. Computer use dans la CLI n'est pas disponible sur Linux ni Windows. Sous Windows, utilisez plutôt [computer use dans Desktop](https://code.claude.com/docs/en/desktop#let-claude-use-your-computer) ;
- vous avez un forfait **Pro ou Max**. Lancez `/status` pour confirmer votre abonnement ;
- vous êtes **authentifié via claude.ai**. Computer use n'est pas disponible avec des fournisseurs tiers comme Amazon Bedrock, Google Cloud's Agent Platform ou Microsoft Foundry. Si vous accédez à Claude exclusivement via un fournisseur tiers, il vous faut un compte claude.ai séparé pour utiliser cette fonctionnalité ;
- vous êtes dans une **session interactive**. Computer use n'est pas disponible en mode non interactif avec le flag `-p`.

## Voir aussi

- [Computer use dans Desktop](https://code.claude.com/docs/en/desktop#let-claude-use-your-computer) : la même capacité avec une page de réglages graphique
- [Claude in Chrome](https://code.claude.com/docs/en/chrome) : automatisation du navigateur pour les tâches web
- [MCP](https://code.claude.com/docs/en/mcp) : connecter Claude à des outils et API structurés
- [Sandboxing](https://code.claude.com/docs/en/sandboxing) : comment l'outil Bash de Claude isole l'accès au système de fichiers et au réseau
- [Guide de sécurité computer use](https://support.claude.com/en/articles/14128542) : bonnes pratiques pour un usage sûr de computer use
