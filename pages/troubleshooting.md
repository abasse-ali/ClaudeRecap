# Dépannage

> Source : https://code.claude.com/docs/en/troubleshooting · [← Index](../31-index-documentation-officielle.md)

*Corriger une utilisation élevée du CPU ou de la mémoire, les blocages, l'emballement de l'auto-compaction et les problèmes de recherche dans Claude Code, et trouver la bonne page pour les autres problèmes.*

Cette page couvre les problèmes de performances, de stabilité et de recherche une fois Claude Code lancé. Pour les autres problèmes, commencez par la page qui correspond à l'endroit où vous êtes bloqué :

| Symptôme | Aller à |
| :- | :- |
| `command not found`, échec d'installation, problèmes de PATH, `EACCES`, erreurs TLS | [Troubleshoot installation and login](https://code.claude.com/docs/en/troubleshoot-install) |
| Le téléchargement d'une mise à jour ou d'une installation échoue avec `The connection dropped while downloading the update` ou `aborted` | [Troubleshoot installation and login](https://code.claude.com/docs/en/troubleshoot-install#the-connection-dropped-while-downloading-the-update) |
| Boucles de connexion, erreurs OAuth, `403 Forbidden`, « organization disabled », identifiants Amazon Bedrock, Google Cloud's Agent Platform ou Microsoft Foundry | [Troubleshoot installation and login](https://code.claude.com/docs/en/troubleshoot-install#login-and-authentication) |
| Réglages qui ne s'appliquent pas, hooks qui ne se déclenchent pas, serveurs MCP qui ne se chargent pas | [Debug your configuration](https://code.claude.com/docs/en/debug-your-config) |
| Session démarrée en mode auto, ou Claude modifie des fichiers et lance des commandes sans demander | [Which mode a session starts in](https://code.claude.com/docs/en/permission-modes#which-mode-a-session-starts-in) |
| `API Error: 5xx`, `529 Overloaded`, `429`, erreurs de validation de requête | [Error reference](https://code.claude.com/docs/en/errors) |
| `model not found` ou `you may not have access to it` | [Error reference](https://code.claude.com/docs/en/errors#theres-an-issue-with-the-selected-model) |
| Une commande lancée par Claude échoue avec `Your disk quota is full`, `is full (ENOSPC)` ou `Command output was lost` | [Error reference](https://code.claude.com/docs/en/errors#disk-quota-or-temp-filesystem-is-full) |
| L'extension VS Code ne se connecte pas ou ne détecte pas Claude | [VS Code integration](https://code.claude.com/docs/en/vs-code#fix-common-issues) |
| `Claude Code process exited with code 1` dans VS Code ou une application SDK | [Error reference](https://code.claude.com/docs/en/errors#claude-code-process-exited-with-code-n) |
| Plugin JetBrains ou IDE non détecté | [JetBrains integration](https://code.claude.com/docs/en/jetbrains#troubleshooting) |
| CPU ou mémoire élevés, réponses lentes, blocages, recherche qui ne trouve pas les fichiers | [Performances et stabilité](#performances-et-stabilité) ci-dessous |

Si vous ne savez pas lequel s'applique, lancez `/doctor` dans Claude Code pour une vérification automatisée de votre installation, de vos réglages, de vos extensions et de l'utilisation du contexte ; il propose des correctifs qu'il peut appliquer après votre confirmation. Si `claude` ne démarre pas du tout, lancez plutôt `claude doctor` depuis votre shell. Lancez `/mcp` pour vérifier l'état des serveurs MCP.

## Performances et stabilité

Ces sections couvrent les problèmes liés à l'utilisation des ressources, à la réactivité et au comportement de la recherche.

### Utilisation élevée du CPU ou de la mémoire

Claude Code est conçu pour fonctionner avec la plupart des environnements de développement, mais peut consommer beaucoup de ressources lorsqu'il traite de grandes bases de code. Si vous rencontrez des problèmes de performances :

1. Utilisez régulièrement `/compact` pour réduire la taille du contexte. S'il renvoie `Not enough messages to compact.`, la conversation compte trop peu de tours pour être résumée ; cela peut arriver même avec un contexte plein, quand un seul gros collage l'a rempli
2. Fermez et redémarrez Claude Code entre les tâches importantes
3. Envisagez d'ajouter les gros répertoires de build à votre fichier `.gitignore`
4. Redémarrez avec [`claude --safe-mode`](https://code.claude.com/docs/en/cli-reference#cli-flags) pour vérifier si un plugin, un serveur MCP ou un hook est en cause. Ce flag désactive toutes les personnalisations pour la session ; si l'utilisation baisse, voir [Debug your configuration](https://code.claude.com/docs/en/debug-your-config#test-against-a-clean-configuration) pour trouver laquelle

Si la mémoire du tas (*heap*) d'une session dépasse 2,5 Go, un avertissement d'utilisation critique de la mémoire apparaît. Pour libérer la mémoire, redémarrez Claude Code et lancez [`claude --continue`](https://code.claude.com/docs/en/cli-reference#cli-flags) pour reprendre la conversation dans un processus neuf.

En dehors du [rendu plein écran](https://code.claude.com/docs/en/fullscreen), lancer `/compact` libère aussi de la mémoire. L'avertissement disparaît une fois l'utilisation de la mémoire redescendue sous 2,5 Go.

Si l'utilisation de la mémoire reste élevée après ces étapes, lancez `/heapdump` pour écrire deux fichiers dans `~/Desktop` : un instantané du tas JavaScript nommé `<session-id>.heapsnapshot` et une ventilation de la mémoire nommée `<session-id>-diagnostics.json`. Claude Code [masque cette commande dans le menu des commandes](https://code.claude.com/docs/en/commands#how-the-command-menu-matches-what-you-type) : tapez-la en entier. Sous Linux sans dossier Desktop, les fichiers sont écrits dans votre répertoire personnel.

> **Avertissement** : le fichier `.heapsnapshot` contient toutes les chaînes du processus, y compris votre conversation complète et vos identifiants. Ne le joignez pas à une issue publique et ne le partagez pas.

La commande affiche aussi un résumé dans la conversation, montrant la mémoire totale du processus, la part qui se trouve dans le tas JS et la part hors du tas. Le résumé liste aussi les éventuels indicateurs de fuite, comme un taux de croissance mémoire élevé ou un nombre anormalement élevé de handles ouverts. Il indique si la majorité de la mémoire se trouve dans le tas JS, que l'instantané capture, ou en mémoire native, qu'il ne capture pas.

Signalez la sortie ou analysez-la vous-même :

* **La signaler** : ouvrez une [issue GitHub](https://github.com/anthropics/claude-code/issues) et joignez uniquement le fichier `-diagnostics.json`, qui contient les statistiques derrière le résumé affiché, sans contenu de conversation ni identifiants
* **L'analyser vous-même** : si le résumé indique que la majorité de la mémoire est dans le tas JS, ouvrez le fichier `.heapsnapshot` dans Chrome DevTools via Memory → Load et triez par taille retenue (*retained size*) pour voir ce qui retient la mémoire

Si le résumé indique que la majorité de la mémoire est native, l'instantané ne peut pas la montrer ; incluez plutôt dans votre signalement les indicateurs de fuite du résumé.

### Les grands tableaux sont tronqués dans le terminal

Un tableau Markdown de plus de 200 lignes affiche ses 200 premières lignes suivies d'une ligne `… N more rows not shown`. Seul l'affichage est plafonné : le tableau complet reste dans la conversation, et [`/copy`](https://code.claude.com/docs/en/commands) copie toutes les lignes. Pour un tableau trop grand pour être lu dans le terminal, demandez plutôt à Claude de l'écrire dans un fichier. Avant la v2.1.208, Claude Code affichait toutes les lignes, si bien que reprendre une session contenant un très grand tableau pouvait se bloquer pendant le nouveau rendu.

### L'auto-compaction s'arrête avec une erreur d'emballement

Si vous voyez `Autocompact is thrashing: the context refilled to the limit...`, la compaction automatique a réussi, mais un fichier ou une sortie d'outil a immédiatement rempli à nouveau la fenêtre de contexte plusieurs fois de suite. Claude Code cesse alors de réessayer pour ne pas gaspiller d'appels API dans une boucle qui ne progresse pas.

Pour vous en sortir :

1. Demandez à Claude de lire le fichier trop volumineux par petits morceaux, par exemple une plage de lignes ou une fonction précise, plutôt que le fichier entier
2. Lancez `/compact` avec une consigne qui écarte la grosse sortie, par exemple `/compact keep only the plan and the diff`
3. Déplacez le travail sur le gros fichier vers un [subagent](https://code.claude.com/docs/en/sub-agents) pour qu'il s'exécute dans une fenêtre de contexte séparée
4. Lancez `/clear` si la conversation précédente n'est plus nécessaire

Si l'erreur revient après un `/clear`, lancez [`/context`](https://code.claude.com/docs/en/debug-your-config) et comparez la ligne `Messages` aux lignes situées au-dessus :

* **`Messages` est la ligne la plus grande** : un fichier ou une sortie d'outil de la nouvelle conversation remplit la fenêtre ; reprenez les étapes 1 à 3
* **Les autres lignes cumulées sont plus grandes** : ce qui se charge au démarrage de la session laisse trop peu de place pour travailler ; [allégez ce qui se charge au démarrage](https://code.claude.com/docs/en/errors#prompt-is-too-long)

### Une commande se bloque ou se fige

Si Claude Code semble ne plus répondre :

1. Appuyez sur Ctrl+C pour tenter d'annuler l'opération en cours
2. S'il ne répond toujours pas, vous devrez peut-être fermer le terminal et redémarrer

Redémarrer ne fait pas perdre votre conversation. Lancez `claude --resume` dans le même répertoire pour reprendre la session.

### Texte brouillé ou corrompu dans le terminal intégré d'un éditeur

Si des caractères s'affichent sous forme de carrés, de traînées ou de mauvais glyphes lorsque vous utilisez Claude Code dans le terminal intégré de VS Code, Cursor ou Devin Desktop, le rendu GPU du terminal est probablement en cause. Lancez `/terminal-setup` dans Claude Code pour définir `terminal.integrated.gpuAcceleration` à `"off"`, ou réglez-le manuellement dans les paramètres de votre éditeur puis rechargez la fenêtre. Voir [Terminal configuration](https://code.claude.com/docs/en/terminal-config) pour les autres réglages qu'écrit `/terminal-setup`.

### La molette de la souris fait défiler une ligne à la fois en rendu plein écran

En [rendu plein écran](https://code.claude.com/docs/en/fullscreen), Claude Code fait défiler la conversation lui-même plutôt que de laisser faire votre terminal. Si chaque cran de molette déplace moins de lignes que vous ne le souhaitez, lancez `/scroll-speed` pour augmenter le nombre de lignes par cran et l'enregistrer, ou définissez la variable d'environnement `CLAUDE_CODE_SCROLL_SPEED` — sauf dans le terminal des IDE JetBrains, où Claude Code applique sa propre gestion du défilement et où aucun des deux n'a d'effet. Voir [Mouse wheel scrolling](https://code.claude.com/docs/en/fullscreen#mouse-wheel-scrolling) pour les valeurs acceptées par chacun.

Pour aller plus vite sans changer la vitesse, appuyez sur `PgUp` et `PgDn` pour défiler d'un demi-écran à la fois. Pour utiliser plutôt l'historique de défilement natif de votre terminal, lancez `/tui default` pour revenir au moteur de rendu classique.

### Les commandes de presse-papiers comme `pbcopy` échouent dans le sandbox

Quand le [sandboxing](https://code.claude.com/docs/en/sandboxing) est activé, les utilitaires de presse-papiers comme `pbcopy`, `xclip` et `wl-copy` peuvent ne pas atteindre le presse-papiers système depuis une commande Bash sandboxée : votre presse-papiers reste inchangé après que Claude leur a envoyé du texte par un pipe.

Pour mettre la sortie de Claude dans votre presse-papiers, demandez à Claude d'afficher le contenu dans sa réponse, puis lancez [`/copy`](https://code.claude.com/docs/en/commands). `/copy` écrit dans le presse-papiers depuis le processus Claude Code lui-même plutôt que depuis une commande sandboxée : le sandboxing ne le bloque donc pas. Il peut copier un seul bloc de code au lieu de toute la réponse, et il écrit aussi ce qu'il a copié dans un fichier dont il affiche le chemin, ce qui offre un repli quand l'écriture dans le presse-papiers n'atteint pas votre terminal, par exemple via SSH.

Quand Claude envoie du texte par un pipe à l'un de ces outils, ajouter `pbcopy *`, `wl-copy *` ou `xclip *` à [`excludedCommands`](https://code.claude.com/docs/en/settings-reference#sandbox-excludedcommands) ne suffit pas, à lui seul, à sortir cet appel du sandbox.

### Le texte copié n'atteint pas votre presse-papiers local via SSH

Quand Claude Code tourne sur une machine distante via SSH, il ne peut pas lancer d'outil de presse-papiers sur votre machine locale. Hors tmux, quand vous sélectionnez du texte en [rendu plein écran](https://code.claude.com/docs/en/fullscreen) ou lancez `/copy`, Claude Code envoie plutôt le texte à votre terminal sous forme de séquence d'échappement OSC 52. Votre terminal décide alors s'il le met dans votre presse-papiers. `/copy` indique `Copied to clipboard` que le texte soit arrivé ou non, et hors tmux la notification de sélection indique `sent N chars via OSC 52`.

Certains terminaux ne tiennent pas compte d'OSC 52. iTerm2 l'ignore tant que vous n'avez pas activé **Settings > General > Selection > Applications in terminal may access clipboard**, et le Terminal.app de macOS ne le prend pas en charge.

Pour récupérer le texte sans OSC 52 :

* Maintenez la touche de sélection native de votre terminal pendant que vous faites glisser, puis copiez avec le raccourci habituel de votre terminal, comme `Cmd+C`. La touche est `Fn` dans Terminal.app et `Option` dans iTerm2. [Keep native text selection](https://code.claude.com/docs/en/fullscreen#keep-native-text-selection) l'indique pour les autres terminaux.
* Définissez [`CLAUDE_CODE_DISABLE_MOUSE=1`](https://code.claude.com/docs/en/env-vars) sur la machine distante pour que votre terminal gère la sélection pendant toute la session.

### Problèmes de recherche et de découverte

Si l'outil Search, les mentions `@file`, les agents personnalisés ou les skills personnalisés ne trouvent pas les fichiers, le binaire `ripgrep` fourni ne fonctionne peut-être pas sur votre système. Installez le paquet `ripgrep` de votre plateforme et indiquez à Claude Code de l'utiliser à la place :

#### macOS

```bash theme={null}
brew install ripgrep
```

#### Ubuntu/Debian

```bash theme={null}
sudo apt install ripgrep
```

#### Alpine

```bash theme={null}
apk add ripgrep
```

`ripgrep` se trouve dans le dépôt community d'Alpine. Si `apk` signale que le paquet est absent, voir [Alpine Linux setup](https://code.claude.com/docs/en/setup#alpine-linux-and-musl-based-distributions).

#### Arch

```bash theme={null}
pacman -S ripgrep
```

#### Windows

```powershell theme={null}
winget install BurntSushi.ripgrep.MSVC
```

#### Utiliser le ripgrep du système

Définissez ensuite `USE_BUILTIN_RIPGREP` à `0`, soit dans l'[environnement](https://code.claude.com/docs/en/env-vars) de votre shell, soit dans le bloc `env` de votre [`settings.json`](https://code.claude.com/docs/en/settings-reference#all-settings) :

```json theme={null}
{
  "env": {
    "USE_BUILTIN_RIPGREP": "0"
  }
}
```

Pour confirmer que le changement a pris effet, lancez `claude doctor` dans votre terminal et vérifiez que la ligne Search affiche le chemin de votre ripgrep système au lieu de `OK (bundled)`.

### Résultats de recherche lents ou incomplets sous WSL

Les pénalités de performance en lecture disque lorsque l'on [travaille à travers les systèmes de fichiers sous WSL](https://learn.microsoft.com/en-us/windows/wsl/filesystems) peuvent produire moins de correspondances que prévu en utilisant Claude Code sous WSL. La recherche fonctionne toujours, mais renvoie moins de résultats que sur un système de fichiers natif.

> **Note** : dans ce cas, `claude doctor` affiche Search comme OK.

**Solutions :**

1. **Formuler des recherches plus précises** : réduisez le nombre de fichiers parcourus en précisant des répertoires ou des types de fichiers : « Search for JWT validation logic in the auth-service package » ou « Find use of md5 hash in JS files ».

2. **Déplacer le projet sur le système de fichiers Linux** : si possible, assurez-vous que votre projet se trouve sur le système de fichiers Linux (`/home/`) plutôt que sur le système de fichiers Windows (`/mnt/c/`).

3. **Utiliser plutôt Windows natif** : envisagez de lancer Claude Code nativement sous Windows plutôt qu'à travers WSL, pour de meilleures performances du système de fichiers.

## Obtenir plus d'aide

Si vous rencontrez des problèmes non couverts ici :

1. Lancez `/doctor` pour un contrôle de la configuration et `/mcp` pour vérifier l'état des serveurs MCP
2. Utilisez la commande `/feedback` dans Claude Code pour signaler les problèmes directement à Anthropic
3. Consultez le [dépôt GitHub](https://github.com/anthropics/claude-code) pour les problèmes connus
4. Interrogez directement Claude sur ses capacités et fonctionnalités : Claude a un accès intégré à sa documentation.

Pour les problèmes de compte, de facturation ou d'abonnement, contactez plutôt le support Anthropic : connectez-vous sur [claude.ai](https://claude.ai) (utilisateurs de la Console : [platform.claude.com](https://platform.claude.com)), cliquez sur vos initiales en bas à gauche, puis sélectionnez **Get help**. Voir [How to get support](https://support.claude.com/en/articles/9015913-how-to-get-support) pour la démarche complète, y compris qui peut joindre un agent humain selon le forfait.
