# Rendu plein écran (fullscreen)

> Source : https://code.claude.com/docs/en/fullscreen · [← Index](../31-index-documentation-officielle.md)

Activer un mode de rendu plus fluide, sans scintillement, avec prise en charge de la souris et une consommation mémoire stable dans les longues conversations.

> **Note** : le rendu plein écran est un [aperçu de recherche (research preview)](#aperçu-de-recherche). Le fait que vous [démarriez en plein écran ou avec le moteur de rendu classique](#plein-écran-par-défaut) dépend de votre configuration. Lancez `/tui fullscreen` ou `/tui default` pour basculer dans la conversation en cours. Le comportement peut évoluer selon les retours.

Le rendu plein écran est un chemin de rendu alternatif pour la CLI Claude Code qui :

- élimine le scintillement ;
- maintient une consommation mémoire constante dans les longues conversations ;
- ajoute la prise en charge de la souris.

Il dessine l'interface sur le **tampon d'écran alternatif** (alternate screen buffer) du terminal, comme `vim` ou `htop`, et n'affiche que les messages actuellement visibles. Cela réduit la quantité de données envoyées au terminal à chaque mise à jour.

La différence est la plus nette dans les émulateurs de terminal où le débit de rendu est le goulot d'étranglement, comme le terminal intégré de VS Code, tmux et iTerm2. Si la position de défilement de votre terminal saute en haut pendant que Claude travaille, ou si l'écran clignote pendant que la sortie des outils s'affiche, ce mode règle ces problèmes.

> **Note** : le terme « plein écran » décrit la manière dont Claude Code prend le contrôle de la surface de dessin du terminal, comme le fait `vim`. Cela n'a rien à voir avec l'agrandissement de la fenêtre du terminal, et fonctionne avec n'importe quelle taille de fenêtre.

## Activer le rendu plein écran

Lancez `/tui fullscreen` dans n'importe quelle conversation Claude Code. La CLI enregistre le [réglage `tui`](https://code.claude.com/docs/en/settings-reference#tui) et se relance en plein écran avec votre conversation intacte : vous pouvez donc basculer en cours de session sans perdre le contexte.

- `/tui default` : revient au moteur de rendu classique.
- `/tui` (sans argument) : affiche le moteur de rendu actif.

En [mode lecteur d'écran](https://code.claude.com/docs/en/accessibility), Claude Code utilise toujours le moteur de rendu classique, sauf dans les [sessions d'arrière-plan](https://code.claude.com/docs/en/agent-view) attachées, qui restent rendues en plein écran. Si vous lancez `/tui fullscreen` dans une autre session en mode lecteur d'écran, Claude Code affiche une explication au lieu de basculer et ne modifie pas le réglage `tui` enregistré.

Claude Code reporte les éléments suivants dans la session relancée :

- **La conversation telle qu'elle apparaît à l'écran.** Après un [`/rewind`](https://code.claude.com/docs/en/checkpointing#rewind-and-summarize), cela signifie :
  - si vous êtes revenu en arrière plus tôt dans la session, Claude Code relance à partir du point de retour, et non à partir de la transcription plus longue enregistrée sur disque. Par exemple, si vous êtes revenu avant vos trois derniers messages, la session relancée s'ouvre sans eux ;
  - si vous êtes revenu avant votre premier message, Claude Code relance avec une conversation vide.
- Votre [mode de permission](https://code.claude.com/docs/en/permission-modes) et votre [niveau d'effort](https://code.claude.com/docs/en/model-config#adjust-effort-level).
- Le modèle que vous avez choisi en dernier avec [`/model`](https://code.claude.com/docs/en/model-config#setting-your-model).
- Les règles passées avec [`--allowed-tools` ou `--disallowed-tools`](https://code.claude.com/docs/en/cli-reference#cli-flags), ainsi que vos flags `--agent`, `--agents`, `--append-system-prompt` et `--system-prompt-snapshot`.

Claude Code refuse de se relancer si la session comporte une restriction qu'il ne peut pas transmettre au processus redémarré. Restrictions non transmissibles :

- les flags de lancement tels qu'un remplacement [`--system-prompt`](https://code.claude.com/docs/en/cli-reference#cli-flags), une liste autorisée [`--tools`](https://code.claude.com/docs/en/cli-reference#cli-flags) ou [`--setting-sources`](https://code.claude.com/docs/en/cli-reference#cli-flags) ;
- les règles deny ou ask ajoutées pour cette session uniquement par un [hook ou une mise à jour de permission du SDK](https://code.claude.com/docs/en/hooks#permission-update-entries).

Dans ce cas, Claude Code affiche [`Cannot switch renderers in this session`](https://code.claude.com/docs/en/errors#cannot-switch-renderers-in-this-session) avec les raisons. Il ne bascule pas et n'enregistre rien.

Vous pouvez aussi définir la variable d'environnement `CLAUDE_CODE_NO_FLICKER` avant de démarrer Claude Code :

```bash theme={null}
CLAUDE_CODE_NO_FLICKER=1 claude
```

Pour savoir comment le réglage [`tui`](https://code.claude.com/docs/en/settings-reference#tui) et la variable se combinent lorsque les deux sont définis, voir l'entrée du réglage. Après un [démarrage plein écran échoué](#claude-codes-fullscreen-renderer-didnt-finish-starting-last-time-saffiche-au-démarrage), Claude Code respecte toujours la variable, mais plus le réglage. La commande `/tui` retire `CLAUDE_CODE_NO_FLICKER` du processus relancé pour que le réglage qu'elle écrit prenne effet.

### Plein écran par défaut

Les [sessions d'arrière-plan](https://code.claude.com/docs/en/agent-view) attachées sont rendues en plein écran, et les autres sessions en [mode lecteur d'écran](https://code.claude.com/docs/en/accessibility) utilisent le moteur classique. Sinon, Claude Code vous démarre dans le moteur de rendu indiqué par la **première ligne** du tableau ci-dessous qui correspond à votre configuration :

| Votre situation | Moteur de rendu au démarrage |
| :- | :- |
| Vous avez défini [`CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN=1`](https://code.claude.com/docs/en/env-vars) ou `CLAUDE_CODE_NO_FLICKER=0` | Classique |
| Vous avez défini `CLAUDE_CODE_NO_FLICKER=1` | Plein écran |
| Claude Code [a désactivé le plein écran après un démarrage plein écran échoué](#claude-codes-fullscreen-renderer-didnt-finish-starting-last-time-saffiche-au-démarrage) sur cette machine | Classique |
| Vous êtes dans le [mode d'intégration `tmux -CC`](#utilisation-avec-tmux) d'iTerm2, ou vous êtes connecté en SSH à un Claude Code qui tourne sous Windows | Classique |
| Vous avez enregistré un [réglage `tui`](https://code.claude.com/docs/en/settings-reference#tui) | Le moteur nommé par le réglage |
| Votre session ne [récupère pas les feature flags auprès d'Anthropic](https://code.claude.com/docs/en/env-vars#features-that-need-feature-flag-fetching), et Claude Code a cessé de proposer la boîte de dialogue de démarrage sur cette machine | Classique |
| Votre session ne récupère pas les feature flags auprès d'Anthropic, et le premier lancement de Claude Code sur cette machine utilisait la v2.1.239 ou ultérieure | Plein écran |
| Votre session récupère les feature flags auprès d'Anthropic, et vous avez utilisé Claude Code pour la première fois le 6 mai 2026 ou après | Plein écran |
| Tout autre cas | Classique |

Les sessions qui ne récupèrent pas les feature flags incluent celles passant par [Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock), [Agent Platform de Google Cloud](https://code.claude.com/docs/en/google-vertex-ai) ou [Microsoft Foundry](https://code.claude.com/docs/en/microsoft-foundry), ainsi que celles dont la télémétrie est désactivée.

Si vous démarrez avec le moteur classique et n'avez pas enregistré de réglage `tui`, Claude Code peut ouvrir au démarrage une boîte de dialogue proposant la bascule :

- **Si vous acceptez**, Claude Code se relance comme avec `/tui fullscreen`, en reportant le même état de session, et enregistre le réglage une fois que la session relancée a [démarré avec succès](#claude-codes-fullscreen-renderer-didnt-finish-starting-last-time-saffiche-au-démarrage).
- **Si vous choisissez « Not now »**, Claude Code ne le propose plus sur cette machine.
- Claude Code cesse de proposer après avoir affiché la boîte de dialogue lors de **trois lancements**, qu'elle ait reçu une réponse ou non.

## Ce qui change

Le rendu plein écran modifie la façon dont la CLI dessine dans votre terminal. La zone de saisie reste fixée en bas de l'écran au lieu de se déplacer à mesure que la sortie s'affiche. **Si la saisie ne bouge pas pendant que Claude travaille, le rendu plein écran est actif.** Seuls les messages visibles sont conservés dans l'arbre de rendu, donc la mémoire reste constante quelle que soit la longueur de la conversation.

Comme la conversation vit dans le tampon d'écran alternatif et non dans l'historique de défilement (scrollback) du terminal, certaines choses fonctionnent différemment :

| Avant | Maintenant | Détails |
| :- | :- | :- |
| `Cmd+f` ou la recherche tmux pour trouver du texte | `Ctrl+o` pour le mode transcription, puis `/` pour chercher ou `[` pour écrire dans le scrollback | [Rechercher et relire la conversation](#rechercher-et-relire-la-conversation) |
| Sélection native par cliquer-glisser du terminal pour copier | Sélection dans l'application, copie automatique au relâchement de la souris | [Utiliser la souris](#utiliser-la-souris) |
| `Cmd`-clic pour ouvrir une URL | `Cmd`-clic sur macOS, `Ctrl`-clic ailleurs | [Utiliser la souris](#utiliser-la-souris) |

Si la capture de la souris gêne votre flux de travail, vous pouvez [la désactiver](#conserver-la-sélection-de-texte-native) tout en gardant le rendu sans scintillement.

## Utiliser la souris

Le rendu plein écran capture les événements souris et les gère dans Claude Code :

- **Cliquer dans la zone de saisie** pour placer le curseur n'importe où dans le texte que vous tapez.
- **Cliquer sur une suggestion dans la liste des commandes `/` ou des fichiers `@`** pour l'accepter. Le survol met en évidence la ligne sous le curseur.
- **Cliquer sur une option d'un menu de sélection** pour la choisir. Cela couvre les invites de permission, `/model`, `/config` et les autres boîtes de dialogue qui affichent une liste d'options.
- **Cliquer sur une option d'un menu à sélection multiple** pour la cocher/décocher, puis cliquer sur le bouton de validation pour confirmer vos choix. Cliquer sur une ligne de texte libre, comme la ligne `Other` d'une question à choix multiples, place le focus dans son champ de saisie pour taper une réponse. (v2.1.208+)
- **Cliquer sur la valeur d'un réglage dans le panneau `/config`** pour la modifier, et faire défiler la liste des réglages avec la molette. (v2.1.271+)
- **Faire défiler un menu de sélection simple ou multiple avec la molette** lorsqu'il contient plus d'options qu'il n'en affiche à la fois, comme la liste `/model` dans une fenêtre de terminal peu haute. La molette fait défiler la liste tant que le pointeur est au-dessus de ses options. (v2.1.280+)
- **Faire défiler une liste qui déborde avec sa barre de défilement.** Dans les panneaux de liste comme `/skills`, `/mcp` et la liste Installed de `/plugin`, une barre de défilement apparaît à côté d'une liste comportant plus de lignes que l'espace disponible, tant que le pointeur la survole. Cliquez sur la piste pour sauter à ce point, ou faites glisser le curseur (thumb). (v2.1.281+)
  - Lorsque la barre a des flèches `↑` et `↓` à ses extrémités, cliquez sur une flèche pour défiler d'une ligne, ou maintenez-la pour continuer à défiler. Les flèches nécessitent la v2.1.286+.
- **Cliquer sur la ligne `↑ N more` ou `↓ N more` au bord d'une liste** pour sauter à cette extrémité de la liste sans choisir d'option. (v2.1.286+)
- **Cliquer sur un résultat d'outil replié** pour le déplier et voir la sortie complète. Cliquer à nouveau le replie. L'appel d'outil et son résultat se déplient ensemble. Seuls les messages ayant davantage de contenu à afficher sont cliquables.
  - Le clic déplie aussi la sortie d'une commande shell `!`, qu'il s'agisse d'un ancien résultat tronqué ou de la ligne de progression en direct pendant l'exécution. (v2.1.257+)
  - Le clic déplie aussi une ligne atténuée `Message from @<sender>` lorsque l'expéditeur est un [coéquipier (teammate)](https://code.claude.com/docs/en/agent-teams) ou un autre agent tournant dans votre session.
- **Maintenir `Cmd` sur macOS, ou `Ctrl` sur Linux et Windows, et cliquer sur une URL ou un chemin de fichier** pour l'ouvrir. Les URL `http://` et `https://` simples s'ouvrent dans votre navigateur. Lorsque Claude Code gère le clic, les chemins de fichier dans la sortie des outils, comme ceux affichés après un Edit ou un Write, s'ouvrent dans votre gestionnaire de fichiers avec le fichier sélectionné. Un clic simple sans modificateur n'ouvre pas les liens, comme dans le comportement natif du terminal.
  - Sur Linux et WSL, cliquer sur un chemin de fichier nécessite un gestionnaire de fichiers fournissant l'interface D-Bus `org.freedesktop.FileManager1`. Sans lui, rien ne s'ouvre.
  - Claude Code affiche un chemin réseau (UNC), comme `\\server\share\file.ts`, en texte brut sans lien, car ouvrir un chemin réseau peut envoyer vos identifiants Windows à l'hôte qu'il désigne.
  - Certains terminaux macOS transmettent `Cmd`+clic à l'application en cours au lieu d'ouvrir le lien eux-mêmes, et le protocole souris du terminal ne peut pas encoder la touche `Cmd` : Claude Code reçoit donc un clic simple. Dans Ghostty, et dans Warp sur macOS, Claude Code détecte cette situation et laisse un clic simple sur un lien l'ouvrir ; maintenir `Cmd` fonctionne toujours.
  - Dans le terminal intégré de VS Code et les terminaux similaires basés sur xterm.js, Claude Code s'en remet au gestionnaire de liens du terminal, qui utilise le même geste.
- **Cliquer-glisser** pour sélectionner du texte n'importe où dans la conversation. Le double-clic sélectionne un mot, avec les mêmes limites de mot qu'iTerm2, de sorte qu'un chemin de fichier est sélectionné d'un seul bloc. Double-cliquer sur une URL sélectionne toute l'URL, schéma compris. Le triple-clic sélectionne la ligne.
- **Faire défiler avec la molette** pour parcourir la conversation.

Le texte sélectionné est copié automatiquement dans le presse-papiers au relâchement de la souris. Pour désactiver ce comportement, basculez **Copy on select** dans `/config`.

Avec Copy on select désactivé, appuyez sur `Ctrl+Shift+c` pour copier manuellement. Sur les terminaux qui prennent en charge le protocole clavier kitty, comme kitty, WezTerm, Ghostty et iTerm2, `Cmd+c` fonctionne aussi. Si une sélection est active, `Ctrl+c` copie au lieu d'annuler.

Avec une sélection active, maintenez `Shift` et appuyez sur les flèches pour l'étendre au clavier. `Shift+↑` et `Shift+↓` font défiler la vue lorsque la sélection atteint le bord supérieur ou inférieur. `Shift+Home` et `Shift+End` étendent jusqu'au début ou à la fin de la ligne courante.

Dans la vue normale de saisie, ce qu'il advient d'une sélection active dépend de la touche pressée :

- **`Esc`** : Claude Code exécute l'action habituelle de la touche, comme interrompre la réponse en cours ou fermer une boîte de dialogue ouverte, et la sélection reste en surbrillance.
- **`PgUp`, `PgDn`, `Ctrl+Home`, `Ctrl+End`, ou `Shift`, `Alt` ou `Option`, ou `Cmd`, `Win` ou `Super` combinés avec une flèche, `Home` ou `End`** : la sélection est conservée.
- **Toute autre touche, y compris les flèches simples, `Enter` et les caractères tapés** : Claude Code efface la sélection.
- **Une touche liée à [`selection:clear`](https://code.claude.com/docs/en/keybindings#scroll-actions)** : Claude Code efface la sélection, même si la touche est `Esc` ou une autre touche qui la conserverait normalement. Cette action n'a pas de raccourci par défaut.

En [mode transcription](#rechercher-et-relire-la-conversation), les touches de navigation listées dans cette section conservent aussi la sélection.

## Faire défiler la conversation

Le rendu plein écran gère le défilement dans l'application. Raccourcis de navigation :

| Raccourci | Action |
| :- | :- |
| `PgUp` / `PgDn` | Défiler vers le haut ou le bas d'une demi-page d'écran |
| `Ctrl+Home` | Aller au début de la conversation |
| `Ctrl+End` | Aller au dernier message et réactiver le suivi automatique (auto-follow) |
| Molette de la souris | Défiler de quelques lignes à la fois |

Vous pouvez remonter jusqu'au début de la session même après une [compaction](https://code.claude.com/docs/en/context-window#what-survives-compaction). Claude continue de travailler à partir du résumé de compaction, mais Claude Code conserve tous les messages antérieurs dans le scrollback plein écran, même après des compactions répétées.

Sur les claviers sans touches dédiées `PgUp`, `PgDn`, `Home` ou `End`, comme les claviers de MacBook, maintenez `Fn` avec les flèches :

- `Fn+↑` envoie `PgUp` ;
- `Fn+↓` envoie `PgDn` ;
- `Fn+←` envoie `Home` ;
- `Fn+→` envoie `End`.

`Ctrl+Fn+→` n'atteint pas Claude Code sur macOS : un clavier de MacBook n'a donc, par défaut, aucun raccourci fonctionnel pour sauter en bas. Utilisez à la place l'une de ces options :

- cliquer sur le [bouton « aller en bas »](#suivi-automatique-auto-follow) ;
- faire défiler jusqu'en bas avec la molette pour reprendre le suivi ;
- relier `scroll:bottom` à un raccourci que votre clavier peut envoyer.

Ces actions sont reconfigurables. Voir [Scroll actions](https://code.claude.com/docs/en/keybindings#scroll-actions) pour la liste complète des noms d'action, y compris des variantes demi-page et page entière sans raccourci par défaut.

Lorsque vous avez remonté, une ligne d'en-tête atténuée en haut de la conversation affiche la dernière invite (prompt) qui a défilé au-dessus de la vue. Cliquez sur cette ligne pour sauter à cette invite.

### Suivi automatique (auto-follow)

Remonter met en pause le suivi automatique, pour que la nouvelle sortie ne vous ramène pas en bas. Un bouton `Jump to bottom` flotte au-dessus du bord inférieur de la transcription tant que vous avez remonté, et affiche un compteur tel que `3 new messages` lorsque de la nouvelle sortie arrive. Cliquez dessus, appuyez sur `Ctrl+End` ou faites défiler jusqu'en bas pour reprendre le suivi.

Tant que le suivi automatique est en pause, la vue reste aussi à l'endroit où vous l'avez laissée lorsqu'une réponse termine son affichage.

L'indication clavier du bouton reflète ce que votre clavier peut envoyer. Sur macOS, elle suggère de cliquer, ou `Fn+↓` pour défiler, car `Ctrl+End` n'atteint pas Claude Code depuis un clavier Mac. Si vous reliez [`scroll:bottom`](https://code.claude.com/docs/en/keybindings#scroll-actions) à un raccourci, le bouton affiche votre raccourci sur toutes les plateformes.

Sur un terminal trop étroit pour le libellé complet, le bouton raccourcit l'indication au lieu de passer à la ligne sur la ligne de transcription en dessous.

Pour désactiver complètement le suivi automatique afin que la vue reste où vous la laissez, ouvrez `/config` et réglez **Auto-scroll** sur off. Avec le défilement automatique désactivé, la vue ne saute jamais d'elle-même en bas. Les invites de permission et autres boîtes de dialogue qui exigent une réponse défilent tout de même jusqu'à être visibles, quel que soit ce réglage.

### Défilement à la molette

Le défilement à la molette exige que votre terminal transmette les événements souris à Claude Code. La plupart des terminaux le font dès qu'une application le demande. iTerm2 en fait un réglage par profil : si la molette ne fait rien alors que `PgUp` et `PgDn` fonctionnent, ouvrez Settings → Profiles → Terminal et activez **Enable mouse reporting**. Ce même réglage est aussi nécessaire pour le clic-pour-déplier et la sélection de texte.

Si le défilement à la molette semble lent, votre terminal envoie peut-être un seul événement de défilement par cran physique, sans multiplicateur. Certains terminaux, comme Ghostty et iTerm2 avec le défilement rapide activé, amplifient déjà les événements de molette. D'autres, dont le terminal intégré de VS Code, envoient exactement un événement par cran. Claude Code ne peut pas détecter lequel.

Définissez `CLAUDE_CODE_SCROLL_SPEED` pour multiplier la distance de défilement de base :

```bash theme={null}
export CLAUDE_CODE_SCROLL_SPEED=3
```

Une valeur de `3` correspond à la valeur par défaut de `vim` et d'applications similaires. Le réglage accepte toute valeur positive jusqu'à 20, y compris des valeurs fractionnaires inférieures à 1, comme `0.25`, pour ralentir le défilement accéléré du trackpad et de la molette dans les terminaux qui amplifient déjà les événements.

Pour ajuster la vitesse de défilement de façon interactive, lancez `/scroll-speed`. Faites défiler pendant que la boîte de dialogue est ouverte pour ressentir le changement immédiatement :

- `←` et `→` : ajuster la vitesse ;
- `r` : revenir à la valeur par défaut détectée automatiquement ;
- `Enter` : enregistrer.

La boîte de dialogue avance par nombres entiers jusqu'à 10 et, sur les terminaux qui permettent un contrôle plus fin, propose aussi des pas d'un quart jusqu'à 0.25.

La commande écrit la même valeur que celle définie par la variable d'environnement `CLAUDE_CODE_SCROLL_SPEED`, persistée dans `~/.claude/settings.json`. Le maximum de la boîte de dialogue est 10 : si vous définissez une valeur plus élevée via la variable d'environnement, la boîte de dialogue affiche 10, et enregistrer depuis la boîte de dialogue persiste 10. La commande n'est pas disponible dans le terminal des IDE JetBrains.

Indépendamment de la vitesse de base, Claude Code accélère le défilement lorsque vous faites tourner la molette rapidement, de sorte qu'un tour rapide couvre plus de distance que le même nombre de crans lents. Pour désactiver l'accélération et garder un rythme constant par cran, réglez `wheelScrollAccelerationEnabled` sur `false` dans [`settings.json`](https://code.claude.com/docs/en/settings-reference#all-settings). (v2.1.174+)

### Défilement dans le terminal des IDE JetBrains

Dans le terminal des IDE JetBrains, Claude Code applique sa propre gestion du défilement et ignore `CLAUDE_CODE_SCROLL_SPEED`. Ce terminal envoie des événements de défilement à un rythme bien plus élevé que les autres émulateurs, si bien qu'un multiplicateur réglé ailleurs dépasserait la cible ici.

Dans la version 2025.2, ce terminal présente aussi des bogues de molette qui produisent de fausses touches fléchées et des événements dans la mauvaise direction. Claude Code les détecte à l'exécution et les atténue automatiquement : le défilement au trackpad et à la molette fonctionne donc sans configuration. Pour une meilleure expérience, passez à la version 2025.3 ou ultérieure. Claude Code affiche une indication la première fois que vous faites défiler s'il détecte le bogue.

## Rechercher et relire la conversation

`Ctrl+o` bascule entre la vue normale de saisie et le **mode transcription**.

Pour une vue plus épurée qui n'affiche que votre dernière invite, un résumé d'une ligne des appels d'outils avec les statistiques de diff des modifications, et la réponse finale, lancez `/focus`. Ce réglage persiste d'une session à l'autre. Relancez `/focus` pour le désactiver.

Le mode transcription offre une navigation et une recherche à la manière de `less` :

| Touche | Action |
| :- | :- |
| `/` | Ouvrir la recherche. Tapez pour trouver des correspondances, `Enter` pour valider, `Esc` pour annuler et restaurer votre position de défilement |
| `n` / `N` | Aller à la correspondance suivante ou précédente. Fonctionne après avoir fermé la barre de recherche |
| `j` / `k` ou `↑` / `↓` | Défiler d'une ligne |
| `g` / `G` ou `Home` / `End` | Aller en haut ou en bas |
| `{` / `}` | Aller à l'invite précédente ou suivante |
| `Ctrl+u` / `Ctrl+d` | Défiler d'une demi-page |
| `Ctrl+b` / `Ctrl+f` ou `Space` / `b` | Défiler d'une page entière |
| `Ctrl+o`, `Esc` ou `q` | Quitter le mode transcription et revenir à la saisie |

Le `Cmd+f` de votre terminal et la recherche tmux ne voient pas la conversation, car elle vit dans le tampon d'écran alternatif et non dans le scrollback natif. Pour rendre le contenu à votre terminal, appuyez d'abord sur `Ctrl+o` pour entrer en mode transcription, puis :

- **`[`** : écrit toute la conversation dans le tampon de scrollback natif de votre terminal, avec toutes les sorties d'outils dépliées. La conversation devient alors du texte ordinaire dans votre terminal : `Cmd+f`, le mode copie de tmux et tout autre outil natif peuvent la rechercher ou la sélectionner. Les longues sessions peuvent marquer une courte pause pendant l'opération. Cet état dure jusqu'à ce que vous quittiez le mode transcription avec `Esc` ou `q`, ce qui vous ramène au rendu plein écran. Le `Ctrl+o` suivant repart de zéro.
- **`v`** : écrit la conversation dans un fichier temporaire et l'ouvre dans `$VISUAL` ou `$EDITOR`.

## Suivre vos modifications dans le panneau de diff

En rendu plein écran, [`/diff`](https://code.claude.com/docs/en/interactive-mode#review-changes-with-%2Fdiff) ouvre un panneau à côté de la conversation, pour voir les modifications s'accumuler pendant que Claude travaille. La section [Diff panel](https://code.claude.com/docs/en/interactive-mode#diff-panel) décrit ce qu'il affiche, quand il s'ouvre de lui-même, comment le garder fermé et comment changer la référence de comparaison.

## Effacer la conversation

Lancez `/clear` pour démarrer une nouvelle conversation.

Si l'affichage paraît brouillé ou partiellement vide, appuyez sur `Ctrl+L` pour redessiner l'écran. Le redessin conserve la conversation et votre saisie en place.

`Cmd+K` fait la même chose que `Ctrl+L` lorsque votre terminal le transmet à Claude Code. iTerm2 et Terminal.app gèrent eux-mêmes `Cmd+K` et effacent leur propre écran ; Claude Code détecte l'écran effacé et redessine la conversation.

Historique de comportement :

- De la v2.1.260 jusqu'avant la v2.1.280, appuyer sur `Ctrl+L` (ou `Cmd+K` lorsqu'il atteint Claude Code) effaçait l'écran en rendu plein écran.
- Avant la v2.1.238, appuyer deux fois sur `Ctrl+L` en moins de deux secondes lançait `/clear`.

## Utilisation avec tmux

Le rendu plein écran fonctionne dans tmux, avec trois réserves.

**1. Mode souris de tmux.** Le défilement à la molette nécessite le mode souris de tmux. Si votre `~/.tmux.conf` ne l'active pas déjà, ajoutez cette ligne et rechargez votre configuration :

```bash theme={null}
set -g mouse on
```

Sans le mode souris, les événements de molette vont à tmux au lieu de Claude Code. Le défilement au clavier avec `PgUp` et `PgDn` fonctionne dans les deux cas. Claude Code affiche une indication unique au démarrage s'il détecte tmux avec le mode souris désactivé.

**2. Mode d'intégration tmux d'iTerm2 (`tmux -CC`).** Le rendu plein écran est incompatible avec le mode d'intégration tmux d'iTerm2, celui dans lequel on entre avec `tmux -CC`. Dans ce mode, iTerm2 affiche chaque volet tmux comme une division native au lieu de laisser tmux dessiner dans le terminal. Le tampon d'écran alternatif et le suivi de la souris n'y fonctionnent pas correctement : la molette ne fait rien, et le double-clic peut corrompre l'état du terminal. **N'activez pas le rendu plein écran dans les sessions `tmux -CC`.** tmux classique dans iTerm2, sans `-CC`, fonctionne bien.

**3. Sortie synchronisée.** Les versions de tmux jusqu'à la série 3.6 n'implémentent pas la sortie synchronisée (synchronized output) : vous pouvez donc voir davantage de scintillement lors des redessins qu'en lançant Claude Code directement dans votre terminal. Claude Code sonde au démarrage la prise en charge de la sortie synchronisée par le terminal et l'utilise lorsque le terminal la signale. Si vous constatez du scintillement sous tmux, passez à la version la plus récente de tmux ou lancez Claude Code dans son propre onglet de terminal, hors de tmux.

## Conserver la sélection de texte native

La capture de la souris est le point de friction le plus courant, surtout via SSH ou dans tmux. Lorsque Claude Code capture les événements souris, la copie-à-la-sélection native de votre terminal cesse de fonctionner. La sélection faite par cliquer-glisser existe dans Claude Code, pas dans le tampon de sélection de votre terminal : le mode copie de tmux, les Kitty hints et outils similaires ne la voient pas.

Claude Code écrit la sélection dans le presse-papiers du système, par un chemin qui dépend de votre configuration. En session locale, il lance un outil de presse-papiers natif :

- **macOS** : `pbcopy`
- **Linux** : `wl-copy` sous Wayland, ou `xclip` ou `xsel` sous X11, selon celui qui est installé. Claude Code écrit à la fois le presse-papiers et la sélection PRIMARY, pour que le collage au clic du milieu fonctionne.
- **Windows et WSL** : PowerShell `Set-Clipboard`

Dans tmux, il écrit aussi dans le tampon de collage (paste buffer) de tmux. Via SSH, il se rabat sur les séquences d'échappement OSC 52. Dans GNU screen, Claude Code copie aussi les longues sélections dans le presse-papiers. Avant la v2.1.219, copier une sélection de plus d'environ 570 caractères faisait afficher par GNU screen du texte base64 dans la fenêtre. Claude Code affiche une notification (toast) après chaque copie indiquant le chemin utilisé.

Certains terminaux bloquent OSC 52 par défaut. iTerm2 le bloque tant que vous n'activez pas Settings → General → Selection → **Applications in terminal may access clipboard** ; lancer [`/terminal-setup`](https://code.claude.com/docs/en/terminal-config) dans iTerm2 l'active pour vous.

Pour une sélection native ponctuelle, la touche à utiliser dépend de votre terminal :

- **Terminal.app** : `Fn`
- **iTerm2** : `Option`
- **VS Code, Cursor et Devin Desktop** : `Shift`, ou `Option` sur macOS avec le réglage `terminal.integrated.macOptionClickForcesSelection` activé
- **La plupart des autres terminaux** : `Shift`

Maintenez cette touche pendant le cliquer-glisser. Votre terminal gère alors lui-même la sélection au lieu de la transmettre à Claude Code, et les raccourcis de copie comme `Cmd+C` fonctionnent sur ce que vous sélectionnez. Claude Code affiche aussi la bonne touche dans son indication à l'écran.

Via SSH ou dans tmux, Claude Code ne peut pas toujours détecter le terminal depuis lequel vous vous connectez : l'indication liste alors les touches candidates.

Si vous comptez en permanence sur la sélection native, définissez `CLAUDE_CODE_DISABLE_MOUSE=1` pour renoncer à la capture de la souris tout en conservant le rendu sans scintillement et la mémoire constante :

```bash theme={null}
CLAUDE_CODE_NO_FLICKER=1 CLAUDE_CODE_DISABLE_MOUSE=1 claude
```

Avec la capture de la souris désactivée, le défilement au clavier avec `PgUp`, `PgDn`, `Ctrl+Home` et `Ctrl+End` fonctionne toujours, et votre terminal gère la sélection nativement. Vous perdez : le clic pour placer le curseur, le clic pour déplier, le clic sur les URL et le défilement à la molette dans Claude Code.

Pour garder le défilement à la molette tout en désactivant la gestion du clic, du glisser et du survol, définissez plutôt `CLAUDE_CODE_DISABLE_MOUSE_CLICKS=1`. `CLAUDE_CODE_DISABLE_MOUSE` est prioritaire lorsque les deux variables sont définies.

Avec les clics désactivés, Claude Code capture toujours la souris : la molette et le pavé tactile font défiler la conversation, mais les clics gauches ne font rien dans Claude Code. Vous devez toujours maintenir la touche de votre terminal pour une sélection native par cliquer-glisser. Le clic droit et le collage au clic du milieu continuent de fonctionner sur les terminaux qui les prennent en charge.

## Dépannage

### Texte obsolète ou mal placé à l'écran

Le rendu plein écran n'envoie que les cellules qui ont changé entre deux images. Certains terminaux, le plus souvent Windows Terminal et d'autres hôtes basés sur ConPTY, fusionnent mal ces écritures positionnées et laissent des fragments de sortie antérieure à l'écran jusqu'à ce que vous redimensionniez la fenêtre.

Définissez [`CLAUDE_CODE_ALT_SCREEN_FULL_REPAINT=1`](https://code.claude.com/docs/en/env-vars) pour redessiner chaque cellule à chaque image au lieu d'envoyer des mises à jour incrémentales.

Sous Windows PowerShell :

```powershell theme={null}
$env:CLAUDE_CODE_ALT_SCREEN_FULL_REPAINT = "1"
claude
```

Sous macOS ou Linux :

```bash theme={null}
CLAUDE_CODE_ALT_SCREEN_FULL_REPAINT=1 claude
```

Sous Windows, Claude Code active déjà automatiquement le redessin complet pour les sessions d'arrière-plan et la [vue des agents (agent view)](https://code.claude.com/docs/en/agent-view) : vous n'avez besoin de définir la variable que pour une session plein écran interactive lancée directement.

### `Claude Code's fullscreen renderer didn't finish starting last time` s'affiche au démarrage

Si une session plein écran sur cette machine plante avant d'avoir démarré avec succès, Claude Code démarre votre session suivante avec le moteur de rendu classique et affiche l'une de deux lignes. Une session a **démarré avec succès** lorsqu'elle a dessiné sa première image puis soit est restée active 10 secondes, soit a été terminée par vous avec `/exit`, Ctrl+C ou Ctrl+D. La ligne affichée indique ce que fera Claude Code après cette session :

- **Après un démarrage échoué**, vous voyez `Claude Code's fullscreen renderer didn't finish starting last time on this machine`. Claude Code retente le rendu plein écran à la prochaine session que vous démarrez.
- **Après deux démarrages échoués**, vous voyez `Claude Code's fullscreen renderer has repeatedly failed to start on this machine`. Claude Code continue d'utiliser le moteur classique jusqu'à ce que vous mettiez à jour Claude Code ou lanciez `/tui fullscreen`, et n'affiche plus rien lors de ces sessions ultérieures.

Pour confirmer qu'un démarrage échoué est la raison pour laquelle vous êtes dans le moteur classique, lancez `/tui` sans argument. Tant qu'un démarrage échoué en est la cause, la ligne `Current renderer` l'indique.

- Pour conserver le moteur classique, lancez `/tui default`, qui enregistre le réglage `tui` sans relancer.
- Pour réessayer le rendu plein écran, lancez `/tui fullscreen`. Si cette session ne finit pas non plus de démarrer, [signalez le problème](#aperçu-de-recherche).

Avant la v2.1.236, Claude Code continuait à démarrer les sessions en plein écran après un démarrage échoué.

#### Comment Claude Code compte les démarrages échoués

- **Sessions comptabilisées** : uniquement les sessions qui ont démarré en plein écran parce que votre réglage `tui` le demande, parce que vous avez accepté la [boîte de dialogue de démarrage](#plein-écran-par-défaut), ou parce que Claude Code vous démarre en plein écran par défaut.
- **`CLAUDE_CODE_NO_FLICKER=1`** : si vous la définissez, Claude Code rend cette session en plein écran même après un démarrage échoué, et ne la comptabilise pas.
- **Remise à zéro du compteur** : Claude Code compte les démarrages échoués par version de Claude Code, et un démarrage plein écran réussi remet le compteur à zéro.
- **Boîte de dialogue de démarrage** : si vous avez accepté la boîte de dialogue et que la session relancée a planté, Claude Code n'affiche aucune des deux lignes et ne montre plus la boîte de dialogue sur cette version de Claude Code.

## Aperçu de recherche

Le rendu plein écran est une fonctionnalité en aperçu de recherche (research preview). Il a été testé sur les émulateurs de terminal courants, mais vous pouvez rencontrer des problèmes de rendu sur des terminaux moins courants ou des configurations inhabituelles.

Si vous rencontrez un problème, lancez `/feedback` dans Claude Code pour le signaler, ou ouvrez une issue sur le [dépôt GitHub claude-code](https://github.com/anthropics/claude-code/issues). Indiquez le nom et la version de votre émulateur de terminal.

Pour désactiver le rendu plein écran :

- lancez `/tui default`, ou retirez `CLAUDE_CODE_NO_FLICKER` si vous l'avez activé de cette façon ;
- lorsque vous revenez en arrière avec `/tui default`, Claude Code peut d'abord afficher une invite de retour facultative demandant ce qui vous a fait changer. Tapez une raison et appuyez sur `Enter` pour l'envoyer, ou appuyez sur `Esc` pour passer. La CLI se relance dans le moteur classique dans les deux cas ;
- pour forcer le moteur classique quel que soit le réglage `tui` enregistré, définissez `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN=1`.

Le moteur classique conserve la conversation dans le scrollback natif de votre terminal, si bien que `Cmd+f` et le mode copie de tmux fonctionnent comme d'habitude.

Les sessions d'arrière-plan ouvertes depuis la [vue des agents](https://code.claude.com/docs/en/agent-view) ou `claude attach` utilisent toujours le rendu plein écran. Le terminal qui s'attache entre dans le tampon d'écran alternatif pour afficher la session, et le moteur classique n'y dispose ni de scrollback ni de gestion de la souris : le réglage `tui` et `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN` ne s'appliquent donc pas à elles.
