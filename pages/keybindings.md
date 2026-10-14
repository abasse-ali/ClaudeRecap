# Personnaliser les raccourcis clavier

> Source : https://code.claude.com/docs/en/keybindings · [← Index](../31-index-documentation-officielle.md)

*Personnalisez les raccourcis clavier de Claude Code avec un fichier de configuration des keybindings.*

Claude Code prend en charge des raccourcis clavier personnalisables. Lancez `/keybindings` pour créer ou ouvrir votre fichier de configuration `~/.claude/keybindings.json`.

## Fichier de configuration

Le fichier de configuration des keybindings est un objet contenant un tableau `bindings`. Chaque bloc indique un contexte et une table qui associe des frappes (keystrokes) à des actions.

> **Note** : les modifications du fichier de keybindings sont détectées et appliquées automatiquement, sans redémarrer Claude Code.

| Champ | Description |
| :- | :- |
| `$schema` | URL de JSON Schema facultative, pour l'autocomplétion dans l'éditeur |
| `$docs` | URL de documentation facultative |
| `bindings` | Tableau de blocs de raccourcis, par contexte |

Cet exemple associe `Ctrl+E` à l'ouverture d'un éditeur externe dans le contexte de chat, et supprime le raccourci `Ctrl+S` :

```json theme={null}
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "$docs": "https://code.claude.com/docs/en/keybindings",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+s": null
      }
    }
  ]
}
```

## Contextes

Chaque bloc de raccourcis indique un **contexte** dans lequel les raccourcis s'appliquent :

| Contexte | Description |
| :- | :- |
| `Global` | S'applique partout dans l'application |
| `Chat` | Zone principale de saisie du chat |
| `Autocomplete` | Le menu d'autocomplétion est ouvert |
| `Settings` | Menu des réglages |
| `Confirmation` | Boîtes de dialogue d'autorisation et de confirmation |
| `Tabs` | Composants de navigation par onglets |
| `Help` | Le menu d'aide est visible |
| `Transcript` | Visionneuse de transcript |
| `HistorySearch` | Mode de recherche dans l'historique (Ctrl+R) |
| `Task` | Une tâche s'exécute au premier plan |
| `ThemePicker` | Boîte de dialogue de choix du thème |
| `Attachments` | Navigation entre les images jointes dans les dialogues de sélection |
| `Footer` | Navigation entre les indicateurs du pied de page (tâches, équipes, diff, artifacts) |
| `MessageSelector` | Sélection de message dans les dialogues de retour en arrière (rewind) et de résumé |
| `DiffDialog` | Navigation dans la [visionneuse de diff](#actions-de-diff) |
| `DiffPanel` | Le [panneau de diff](https://code.claude.com/docs/en/interactive-mode#diff-panel) est ouvert |
| `ModelPicker` | Niveau d'effort dans le sélecteur de modèle |
| `EffortSlider` | Curseur d'effort ouvert par `/effort` |
| `Select` | Composants génériques de sélection / liste |
| `Plugin` | Dialogue des plugins (parcourir, découvrir, gérer) |
| `AbovePrompt` | La [bande au-dessus du prompt](#actions-au-dessus-du-prompt), ou un bouton qu'elle contient, a le focus clavier |
| `AbovePromptInput` | Un champ de saisie de la bande au-dessus du prompt ou du panneau (pane) d'un mod a le focus clavier |
| `AbovePromptSelect` | Une liste de sélection de la bande au-dessus du prompt ou du panneau d'un mod a le focus clavier |
| `Pane` | Un panneau dessiné par un [mod](https://code.claude.com/docs/en/plugins/mods/interface#know-which-keys-your-mod-can-receive) a le focus clavier |
| `PaneField` | Un champ de saisie ou une liste de sélection du panneau d'un mod a le focus clavier |
| `Agents` | [Vue des agents](https://code.claude.com/docs/en/agent-view) (`claude agents`) |
| `Scroll` | Défilement de la conversation et sélection de texte en mode plein écran |

Avant la v2.1.205, il existait un contexte `Doctor` et une action `doctor:fix` pour l'écran de diagnostic `/doctor`.

## Actions disponibles

Les actions suivent le format `namespace:action`, par exemple `chat:submit` pour envoyer un message ou `app:toggleTodos` pour afficher la liste des tâches. Chaque contexte dispose d'actions spécifiques.

### Actions de l'application

Actions disponibles dans le contexte `Global` :

| Action | Par défaut | Description |
| :- | :- | :- |
| `app:interrupt` | Ctrl+C | Annuler l'opération en cours |
| `app:exit` | Ctrl+D | Quitter Claude Code. Appuyer deux fois en moins de 800 ms pour confirmer |
| `app:redraw` | (non associé) | Forcer le réaffichage du terminal |
| `app:toggleTodos` | Ctrl+T | Afficher/masquer la checklist de to-do de Claude. Ce n'est pas la vue des tâches d'arrière-plan [`/tasks`](https://code.claude.com/docs/en/commands) |
| `app:toggleTranscript` | Ctrl+O | Afficher/masquer le transcript détaillé (verbose) |

### Actions de l'historique

Actions de navigation dans l'historique des commandes :

| Action | Par défaut | Description |
| :- | :- | :- |
| `history:search` | Ctrl+R | Ouvrir la recherche dans l'historique |
| `history:previous` | Up | Élément précédent de l'historique |
| `history:next` | Down | Élément suivant de l'historique |

### Actions du chat

Actions disponibles dans le contexte `Chat` :

| Action | Par défaut | Description |
| :- | :- | :- |
| `chat:cancel` | Escape | Annuler la saisie en cours |
| `chat:clearInput` | Ctrl+L | Forcer un réaffichage complet de l'écran, en conservant la saisie et la conversation |
| `chat:clearScreen` | Cmd+K | Identique à `chat:clearInput`. Voir [Effacer la conversation](https://code.claude.com/docs/en/fullscreen#clear-the-conversation) pour le comportement de Cmd+K dans iTerm2 et Terminal.app |
| `chat:killAgents` | Ctrl+X Ctrl+K | Arrêter tous les [sous-agents d'arrière-plan](https://code.claude.com/docs/en/sub-agents#run-subagents-in-foreground-or-background) en cours dans cette session et désactiver les [réponses automatiques aux artifacts](https://code.claude.com/docs/en/artifacts#let-claude-reply-to-comments-on-its-own) pour le reste de la session. Appuyer deux fois sur le raccourci en moins de 3 secondes pour confirmer. Utilisable même lorsqu'une invite d'autorisation d'un sous-agent d'arrière-plan est ouverte |
| `chat:cycleMode` | Shift+Tab\* | Parcourir les modes d'autorisation |
| `chat:modelPicker` | Meta+P | Ouvrir le sélecteur de modèle |
| `chat:fastMode` | Meta+O | Activer/désactiver le fast mode |
| `chat:thinkingToggle` | Meta+T | Activer/désactiver la réflexion étendue (extended thinking) |
| `chat:submit` | Enter | Envoyer le message |
| `chat:queueSubmit` | Ctrl+X Enter | Envoyer le message en le marquant comme devant attendre son tour : pendant que Claude travaille, Claude Code [le met en file d'attente](https://code.claude.com/docs/en/interactive-mode#queue-messages-while-claude-works) et n'interrompt jamais le tour. Contrairement à `chat:submit`, envoie le brouillon même lorsqu'une suggestion d'autocomplétion est surlignée. (v2.1.247+) |
| `chat:sendNow` | Ctrl+Enter, Ctrl+X Ctrl+S | Envoyer immédiatement vos [messages en file d'attente](https://code.claude.com/docs/en/interactive-mode#queue-messages-while-claude-works), avec votre brouillon. [Quand Claude Code envoie ce que vous avez mis en file](https://code.claude.com/docs/en/interactive-mode#when-claude-code-sends-what-you-queued) explique ce qu'il advient du tour en cours de Claude. Quand rien ne s'exécute, la touche envoie le brouillon ; en [mode shell](https://code.claude.com/docs/en/interactive-mode#shell-mode-with-prefix), elle ne fait que mettre la commande en file. Les terminaux qui ne signalent pas les touches étendues transmettent `Ctrl+Enter` comme un simple `Enter` : `Ctrl+X Ctrl+S` est donc le raccourci qui fonctionne dans tous les terminaux. (v2.1.275+) |
| `chat:newline` | Ctrl+J | Insérer un saut de ligne sans envoyer |
| `chat:undo` | Ctrl+\_, Ctrl+Shift+- | Annuler la dernière action |
| `chat:externalEditor` | Ctrl+G, Ctrl+X Ctrl+E | Ouvrir dans l'éditeur externe. Le [champ de dispatch de la vue des agents](https://code.claude.com/docs/en/agent-view#keyboard-shortcuts) suit aussi les raccourcis à frappe unique de cette action |
| `chat:stash` | Ctrl+S | Mettre de côté (stash) le prompt en cours |
| `chat:imagePaste` | Ctrl+V (Alt+V sous Windows et WSL) | Coller une image depuis le presse-papiers. Sous WSL, les deux raccourcis sont associés par défaut |

\*Sous Windows sans mode VT (Node \<24.2.0/\<22.17.0, Bun \<1.2.23), la valeur par défaut est Meta+M.

### Actions d'autocomplétion

Actions disponibles dans le contexte `Autocomplete` :

| Action | Par défaut | Description |
| :- | :- | :- |
| `autocomplete:accept` | Tab | Accepter la suggestion |
| `autocomplete:dismiss` | Escape | Fermer le menu |
| `autocomplete:previous` | Up | Suggestion précédente |
| `autocomplete:next` | Down | Suggestion suivante |

### Actions de confirmation

Actions disponibles dans le contexte `Confirmation` :

| Action | Par défaut | Description |
| :- | :- | :- |
| `confirm:yes` | Enter | Confirmer l'action |
| `confirm:no` | Escape | Refuser l'action |
| `confirm:previous` | Up | Option précédente |
| `confirm:next` | Down | Option suivante |
| `confirm:nextField` | Tab | Dans le dialogue `/fast`, activer ou désactiver le fast mode |
| `confirm:previousField` | (non associé) | Claude Code ne réagit pas à cette action, et un `keybindings.json` qui la nomme reste valide |
| `confirm:toggle` | Space | Basculer la sélection |
| `confirm:cycleMode` | Shift+Tab\* | Sur une invite d'autorisation de fichier, ferme un [champ de commentaire](https://code.claude.com/docs/en/permissions#add-a-comment-when-you-answer-a-permission-prompt) ouvert ; si aucun champ n'est ouvert, sélectionne l'option qui autorise l'action pour le reste de la session, lorsque l'invite propose cette option |

\*Sous Windows sans mode VT (Node \<24.2.0/\<22.17.0, Bun \<1.2.23), la valeur par défaut est Meta+M.

Avant la v2.1.257, une action `confirm:toggleExplanation`, associée par défaut à `Ctrl+E`, affichait une explication de la commande générée par le modèle sur les invites d'autorisation Bash et PowerShell.

Les dialogues utilisent `confirm:yes` et `confirm:no` pour accepter et annuler, même lorsqu'ils ne posent pas une question oui/non. Si vous associez une lettre seule comme `y` ou `n` dans ce contexte, la lettre agit aussi sur des dialogues qui ne l'affichent jamais comme touche. Un dialogue qui affiche `y` et `n` comme touches lit lui-même ces lettres et n'a besoin d'aucun raccourci.

Dans la plupart des dialogues, appuyer deux fois sur `Ctrl+C` ou `Ctrl+D` ferme le dialogue au lieu de quitter Claude Code. L'indication affichée après le premier appui précise si le second appui ferme le dialogue ou quitte l'application. Ces deux touches sont [réservées](#raccourcis-réservés) et ne peuvent pas être réassociées.

Cet exemple associe `y` à `confirm:yes` et `n` à `confirm:no` :

```json theme={null}
{
  "bindings": [
    {
      "context": "Confirmation",
      "bindings": {
        "y": "confirm:yes",
        "n": "confirm:no"
      }
    }
  ]
}
```

Avec ces raccourcis, `y` et `n` continuent à s'écrire comme des lettres lorsqu'un [champ de texte](#champs-de-texte) a le focus.

Avant la v2.1.280, `y` était aussi associé par défaut à `confirm:yes` et `n` à `confirm:no`. Si vous avez créé votre `keybindings.json` avec `/keybindings` avant la v2.1.280, le fichier contient ces deux raccourcis, qui restent actifs tant que vous ne supprimez pas ces deux lignes.

### Actions d'autorisation

Actions disponibles dans le contexte `Confirmation` pour les dialogues d'autorisation :

| Action | Par défaut | Description |
| :- | :- | :- |
| `permission:toggleDebug` | (non associé) | Claude Code ne réagit pas à cette action, et un `keybindings.json` qui la nomme reste valide |

### Actions du transcript

Actions disponibles dans le contexte `Transcript` :

| Action | Par défaut | Description |
| :- | :- | :- |
| `transcript:toggleShowAll` | Ctrl+E | Afficher/masquer tout le contenu |
| `transcript:exit` | q, Ctrl+C, Escape | Quitter la vue transcript |

`transcript:toggleShowAll` ne s'applique qu'avec le moteur de rendu classique ; en [rendu plein écran](https://code.claude.com/docs/en/fullscreen), la visionneuse de transcript ne propose pas de bascule « tout afficher ».

### Actions de recherche dans l'historique

Actions disponibles dans le contexte `HistorySearch` :

| Action | Par défaut | Description |
| :- | :- | :- |
| `historySearch:next` | Ctrl+R | Correspondance suivante |
| `historySearch:accept` | Escape, Tab | Accepter la sélection |
| `historySearch:cancel` | Ctrl+C | Annuler la recherche |
| `historySearch:execute` | Enter | Exécuter la commande sélectionnée |
| `historySearch:cycleScope` | Ctrl+S | Parcourir la portée : session, projet, partout |

Les valeurs par défaut de `historySearch:next`, `historySearch:accept`, `historySearch:cancel` et `historySearch:execute` s'appliquent à la recherche d'historique en ligne du moteur de rendu classique, qui cherche toujours dans les prompts de tous les projets. `historySearch:cycleScope` n'a d'effet qu'en [rendu plein écran](https://code.claude.com/docs/en/fullscreen), où `Ctrl+R` ouvre à la place un dialogue de recherche et où `Ctrl+S` change sa portée. Les autres touches de ce dialogue sont fixes et ne peuvent pas être réassociées : `Enter` ou `Tab` place la correspondance surlignée dans le champ du prompt et `Esc` annule.

### Actions des tâches

Actions disponibles dans le contexte `Task` :

| Action | Par défaut | Description |
| :- | :- | :- |
| `task:background` | Ctrl+B, Ctrl+X Ctrl+B | Passer la tâche en cours en arrière-plan |

### Actions du thème

Actions disponibles dans le contexte `ThemePicker` :

| Action | Par défaut | Description |
| :- | :- | :- |
| `theme:toggleSyntaxHighlighting` | Ctrl+T | Activer/désactiver la coloration syntaxique |

### Actions de l'aide

Actions disponibles dans le contexte `Help` :

| Action | Par défaut | Description |
| :- | :- | :- |
| `help:dismiss` | Escape | Fermer le menu d'aide |

### Actions des onglets

Actions disponibles dans le contexte `Tabs` :

| Action | Par défaut | Description |
| :- | :- | :- |
| `tabs:next` | Tab, Right | Onglet suivant |
| `tabs:previous` | Shift+Tab, Left | Onglet précédent |

Dans un dialogue à onglets, `tabs:next` et `tabs:previous` changent d'onglet tant que la rangée d'onglets a le focus. Dans certains dialogues, comme `/help` et `/sandbox`, les touches de changement d'onglet fonctionnent aussi depuis l'intérieur du contenu de l'onglet.

`Up` et `Down` déplacent le focus entre la rangée d'onglets et le contenu de l'onglet ; une liste dans le contenu ne réagit aux touches que lorsqu'elle a le focus.

### Actions des pièces jointes

Actions disponibles dans le contexte `Attachments` :

| Action | Par défaut | Description |
| :- | :- | :- |
| `attachments:next` | Right | Pièce jointe suivante |
| `attachments:previous` | Left | Pièce jointe précédente |
| `attachments:remove` | Backspace, Delete | Retirer la pièce jointe sélectionnée |
| `attachments:exit` | Down, Escape | Quitter la navigation entre pièces jointes |

### Actions du pied de page

Actions disponibles dans le contexte `Footer` :

| Action | Par défaut | Description |
| :- | :- | :- |
| `footer:next` | Right | Élément suivant du pied de page |
| `footer:previous` | Left | Élément précédent du pied de page |
| `footer:up` | Up, Ctrl+P | Monter dans le pied de page (désélectionne en haut) |
| `footer:down` | Down, Ctrl+N | Descendre dans le pied de page |
| `footer:openSelected` | Enter | Ouvrir l'élément sélectionné du pied de page |
| `footer:clearSelection` | Escape | Effacer la sélection du pied de page |
| `footer:close` | x | Arrêter l'[agent](https://code.claude.com/docs/en/sub-agents#observe-and-steer-running-forks) ou le [workflow](https://code.claude.com/docs/en/workflows#manage-runs) sélectionné, ou retirer sa ligne s'il ne s'exécute plus |
| `footer:dismiss` | (non associé) | Associer une touche à cette action n'a aucun effet, et un `keybindings.json` qui la nomme reste valide. Avant la v2.1.281, Backspace et Delete y étaient associés et retiraient du pied de page le lien d'artifact sélectionné. |

Lorsqu'un élément du pied de page est sélectionné, par exemple une ligne du panneau des agents sous le prompt, `Enter` l'ouvre même si vous avez réassocié `Enter` dans le contexte `Chat` à `chat:queueSubmit` ou `chat:newline`.

Les raccourcis `Chat` posés sur des touches que le contexte `Footer` n'associe pas, comme `Shift+Tab` pour `chat:cycleMode`, continuent de fonctionner pendant qu'un élément est sélectionné.

### Actions du sélecteur de messages

Dans la liste des messages du [menu de retour en arrière (rewind)](https://code.claude.com/docs/en/checkpointing), vous parcourez les messages et en choisissez un avec les [actions Select](#actions-de-sélection) et leurs touches par défaut. Vos raccourcis `Select` pour ces actions s'y appliquent aussi. Le contexte `MessageSelector` n'a ni actions ni raccourcis par défaut propres. Utilisez-le pour changer une touche pour cette seule liste, en associant une action Select comme `select:accept` dans un bloc `MessageSelector`.

Cet exemple associe `o` au choix du message surligné dans le menu rewind, sans modifier aucune autre liste :

```json theme={null}
{
  "bindings": [
    {
      "context": "MessageSelector",
      "bindings": {
        "o": "select:accept"
      }
    }
  ]
}
```

Avant la v2.1.283, cette liste ignorait les raccourcis `Select` et avait ses propres actions : `messageSelector:up`, `messageSelector:down`, `messageSelector:top`, `messageSelector:bottom` et `messageSelector:select`. Si votre `keybindings.json` associe l'un de ces noms, le raccourci continue de fonctionner dans cette liste comme l'action Select qui fait la même chose. `Home` et `End` sautent à l'une ou l'autre extrémité de la liste ; avant la v2.1.283, des touches comme `Shift+K` et `Shift+J` le faisaient par défaut.

### Actions de diff

Ces actions n'atteignent que l'ancienne visionneuse de diff de Claude Code, que `/diff` ouvre hors [rendu plein écran](https://code.claude.com/docs/en/fullscreen) après que vous avez désactivé le [mod `cc-plugin-diff`](https://code.claude.com/docs/en/plugins/mods/overview#mods-built-into-claude-code) dans `/plugin`. Tant que ce mod est activé, `/diff` ouvre à la place le [dialogue de diff](https://code.claude.com/docs/en/interactive-mode#diff-dialog). Un `keybindings.json` qui nomme ces actions se charge sans erreur dans les deux cas.

Actions disponibles dans le contexte `DiffDialog` :

| Action | Par défaut | Description |
| :- | :- | :- |
| `diff:dismiss` | Escape | Fermer la visionneuse de diff ; depuis la vue détaillée, revient plutôt à la liste des fichiers |
| `diff:previousSource` | Left | Source de diff précédente |
| `diff:nextSource` | Right | Source de diff suivante |
| `diff:previousFile` | Up, K | Fichier précédent dans la liste ; défilement d'une ligne vers le haut dans la vue détaillée |
| `diff:nextFile` | Down, J | Fichier suivant dans la liste ; défilement d'une ligne vers le bas dans la vue détaillée |
| `diff:back` | (non associé) | Revenir en arrière dans la visionneuse de diff. Escape effectue le retour via `diff:dismiss`. L'ancienne valeur par défaut Left dans la vue détaillée a été retirée en v2.1.203 |

La liste des fichiers réagit aussi aux [actions Select](#actions-de-sélection), via leurs touches par défaut et vos raccourcis `Select`. `select:previous` et `select:next` passent au fichier précédent et suivant, et `Enter` ouvre le diff du fichier sélectionné via `select:accept`. Pour changer l'une de ces touches pour la seule liste des fichiers, associez l'action Select dans un bloc `DiffDialog`.

Avant la v2.1.283, la liste des fichiers ignorait les raccourcis `Select`, et `Enter` ouvrait le diff du fichier sélectionné via une action distincte `diff:viewDetails`. Si votre `keybindings.json` associe `diff:viewDetails`, le raccourci continue de fonctionner dans la liste des fichiers comme `select:accept`.

La vue détaillée du diff associe aussi des touches de type pager aux [actions de défilement](#actions-de-défilement) standard. Ces raccourcis font partie du contexte `DiffDialog` et ne s'appliquent que dans la vue détaillée ; les valeurs par défaut du contexte `Scroll` listées dans [Actions de défilement](#actions-de-défilement) ne changent pas.

| Action | Par défaut | Description |
| :- | :- | :- |
| `scroll:pageUp` | PageUp | Remonter d'une demi-hauteur de fenêtre |
| `scroll:pageDown` | PageDown | Descendre d'une demi-hauteur de fenêtre |
| `scroll:fullPageUp` | Shift+Space, B | Remonter d'une hauteur de fenêtre complète |
| `scroll:fullPageDown` | Space | Descendre d'une hauteur de fenêtre complète |
| `scroll:top` | G, Home | Aller tout en haut |
| `scroll:bottom` | Shift+G, End | Aller tout en bas |

### Actions du panneau de diff

Actions du [panneau de diff](https://code.claude.com/docs/en/interactive-mode#diff-panel) que `/diff` ouvre en rendu plein écran. `app:cycleDiffBase` appartient au contexte `DiffPanel`, actif tant que le panneau est ouvert ; les autres sont dans `Global`.

Le [mod intégré `cc-plugin-diff`](https://code.claude.com/docs/en/plugins/mods/overview#mods-built-into-claude-code) dessine ce panneau et gère `app:cycleDiffBase`, `app:diffFileListUp` et `app:diffFileListDown`. `app:toggleReplTab`, `app:toggleDiffNoiseFilter` et `app:toggleDiffPreSession` n'atteignent que l'ancien panneau de Claude Code, que `/diff` ouvre après que vous avez désactivé `cc-plugin-diff` dans `/plugin`.

| Action | Par défaut | Description |
| :- | :- | :- |
| `app:toggleReplTab` | (non associé) | Ouvrir ou fermer le panneau de diff |
| `app:cycleDiffBase` | Ctrl+X B | Faire tourner la base de comparaison du panneau : cette session, non commité, puis branche |
| `app:diffFileListUp` | Ctrl+Up, Meta+Up | Faire défiler vers le haut la liste des fichiers du panneau lorsqu'elle déborde |
| `app:diffFileListDown` | Ctrl+Down, Meta+Down | Faire défiler vers le bas la liste des fichiers du panneau lorsqu'elle déborde |
| `app:toggleDiffNoiseFilter` | (non associé) | Afficher ou masquer les fichiers de test et générés dans le panneau |
| `app:toggleDiffPreSession` | (non associé) | Déplier ou replier les modifications antérieures à cette session |

### Actions du sélecteur de modèle

Actions disponibles dans le contexte `ModelPicker` :

| Action | Par défaut | Description |
| :- | :- | :- |
| `modelPicker:decreaseEffort` | Left | Diminuer le niveau d'effort |
| `modelPicker:increaseEffort` | Right | Augmenter le niveau d'effort |
| `modelPicker:thisSessionOnly` | s | Appliquer le modèle surligné à cette session uniquement |

### Actions du curseur d'effort

Actions disponibles dans le contexte `EffortSlider`, le curseur qui s'ouvre quand vous lancez `/effort` sans argument. Les touches Enter et Escape du curseur ne peuvent pas être réassociées.

| Action | Par défaut | Description |
| :- | :- | :- |
| `effortSlider:decreaseEffort` | Left | Déplacer le curseur au niveau d'effort inférieur suivant. (v2.1.284+) |
| `effortSlider:increaseEffort` | Right | Déplacer le curseur au niveau d'effort supérieur suivant. (v2.1.284+) |
| `effortSlider:toggleUltracode` | Tab | Activer ou désactiver l'[ultracode](https://code.claude.com/docs/en/workflows#let-claude-decide-with-ultracode) pour cette session, lorsque le curseur [le propose](https://code.claude.com/docs/en/model-config#when-ultracode-is-available). (v2.1.284+) |
| `effortSlider:thisSessionOnly` | s | Appliquer le [niveau d'effort](https://code.claude.com/docs/en/model-config#adjust-effort-level) ciblé à cette session uniquement. (v2.1.257+) |

### Actions de sélection

Actions disponibles dans le contexte `Select` :

| Action | Par défaut | Description |
| :- | :- | :- |
| `select:next` | Down, J, Ctrl+N | Option suivante |
| `select:previous` | Up, K, Ctrl+P | Option précédente |
| `select:pageUp` | PageUp | Remonter d'une page d'options |
| `select:pageDown` | PageDown | Descendre d'une page d'options |
| `select:first` | Home | Première option |
| `select:last` | End | Dernière option |
| `select:accept` | Enter | Valider la sélection |
| `select:cancel` | Escape | Annuler la sélection |

Dans les panneaux de liste comme `/skills`, `/mcp` et `/tasks`, Claude Code applique vos raccourcis `select:pageUp`, `select:pageDown`, `select:first` et `select:last`. Dans la plupart des autres listes, comme le sélecteur `/model`, vos raccourcis `select:first` et `select:last` s'appliquent. PageUp et PageDown font défiler les options de ces listes page par page quels que soient vos raccourcis.

Avant la v2.1.280, ces autres listes ignoraient Home, End et vos raccourcis `select:first` et `select:last`.

Avant la v2.1.283, la liste des outils de `/mcp` défilait par page avec des touches PageUp et PageDown fixes, quels que soient vos raccourcis.

### Actions des plugins

Actions disponibles dans le contexte `Plugin` :

| Action | Par défaut | Description |
| :- | :- | :- |
| `plugin:toggle` | Space | Basculer la sélection du plugin |
| `plugin:install` | I | Installer les plugins sélectionnés |
| `plugin:favorite` | F | Mettre en favori le plugin sélectionné pour qu'il soit trié vers le haut de l'onglet Installed |

### Actions au-dessus du prompt

Actions de la bande au-dessus du prompt, la zone partagée où les [mods](https://code.claude.com/docs/en/plugins/mods/interface#pick-where-to-draw) dessinent des boutons, des champs de saisie et des listes de sélection. `abovePrompt:toggle` et `abovePrompt:focus` s'appliquent dans le contexte `Chat`. Les autres actions s'appliquent dans le [contexte](#contextes) de l'élément qui a le focus clavier dans la bande ou dans un panneau.

| Action | Par défaut | Description |
| :- | :- | :- |
| `abovePrompt:toggle` | Ctrl+X Ctrl+A | Réduire la bande à une indication d'une ligne, ou la déplier à nouveau |
| `abovePrompt:focus` | Ctrl+X Tab | Déplacer le focus clavier dans la bande, puis vers chaque [panneau](#actions-des-panneaux) ouvert, et du dernier panneau de retour au prompt |
| `abovePrompt:next` | Tab | Donner le focus au contrôle suivant |
| `abovePrompt:previous` | Shift+Tab | Donner le focus au contrôle précédent |
| `abovePrompt:press` | Enter | Appuyer sur le bouton ciblé, envoyer le champ de saisie ciblé, ou choisir l'option surlignée d'une liste de sélection |
| `abovePrompt:leave` | Escape | Rendre le focus clavier au prompt |
| `abovePrompt:highlightNext` | Down | Surligner l'option suivante d'une liste de sélection ciblée |
| `abovePrompt:highlightPrevious` | Up | Surligner l'option précédente d'une liste de sélection ciblée |

Deux contextes associent par défaut davantage de touches à ces actions :

* **`AbovePrompt`** : Right et Left exécutent aussi `abovePrompt:next` et `abovePrompt:previous`, et Space exécute aussi `abovePrompt:press`
* **`AbovePromptInput`** : Down et Up exécutent aussi `abovePrompt:next` et `abovePrompt:previous`

Le contexte `AbovePrompt` associe aussi Up, Down, PageUp, PageDown, Home et End aux [actions de défilement de panneau](#actions-des-panneaux) `pane:scrollUp` à `pane:bottom` ; pour changer l'une de ces touches pour la bande, associez l'action de défilement dans un bloc `AbovePrompt`.

### Actions des panneaux

Actions d'un panneau (pane) dessiné par un [mod](https://code.claude.com/docs/en/plugins/mods/interface#know-which-keys-your-mod-can-receive). Les actions de défilement, de redimensionnement et de fermeture s'appliquent dans le [contexte](#contextes) `Pane`. `pane:close` s'applique aussi dans le contexte `PaneField`, donc fonctionne pendant que l'un des champs du panneau a le focus. `pane:next` et `pane:previous` s'appliquent dans le contexte `Global` lorsque plus d'un panneau est ouvert.

| Action | Par défaut | Description |
| :- | :- | :- |
| `pane:scrollUp` | Up | Faire défiler le panneau vers le haut lorsqu'il a plus de lignes qu'il ne peut en afficher |
| `pane:scrollDown` | Down | Faire défiler le panneau vers le bas lorsqu'il a plus de lignes qu'il ne peut en afficher |
| `pane:pageUp` | PageUp | Remonter le panneau d'une page |
| `pane:pageDown` | PageDown | Descendre le panneau d'une page |
| `pane:top` | Home | Aller en haut du panneau |
| `pane:bottom` | End | Aller en bas du panneau |
| `pane:grow` | Ctrl+X Left, Ctrl+X Up | Donner plus de place au panneau : en largeur lorsqu'il est à côté du transcript, en hauteur lorsqu'il est au-dessus du prompt |
| `pane:shrink` | Ctrl+X Right, Ctrl+X Down | Donner moins de place au panneau : en largeur lorsqu'il est à côté du transcript, en hauteur lorsqu'il est au-dessus du prompt |
| `pane:close` | Ctrl+X X | Fermer le panneau |
| `pane:next` | (non associé) | Afficher le panneau ouvert suivant |
| `pane:previous` | (non associé) | Afficher le panneau ouvert précédent |

Le contexte `Pane` associe aussi Tab, Shift+Tab, Enter et Escape aux mêmes [actions au-dessus du prompt](#actions-au-dessus-du-prompt) que la bande, et les champs de saisie et listes de sélection d'un panneau utilisent les contextes `AbovePromptInput` et `AbovePromptSelect`. [Focus clavier et raccourcis](https://code.claude.com/docs/en/plugins/mods/interface#know-which-keys-your-mod-can-receive) liste ce que fait chaque touche dans un panneau.

### Actions des réglages

Actions disponibles dans le contexte `Settings`. Les actions `select:accept` et `confirm:no` sont reprises des contextes [Select](#actions-de-sélection) et [Confirmation](#actions-de-confirmation) avec un comportement propre aux réglages : chaque modification s'applique au réglage dès que vous le changez, donc Escape ferme le panneau en conservant vos changements au lieu de les refuser.

| Action | Par défaut | Description |
| :- | :- | :- |
| `settings:search` | / | Entrer en mode recherche |
| `settings:retry` | R | Réessayer de charger les données d'utilisation en cas d'erreur |
| `select:accept` | Enter, Space | Modifier le réglage sélectionné ou ouvrir son sous-menu |
| `confirm:no` | Escape | Fermer le panneau. Les modifications sont déjà enregistrées |

### Actions des agents

Actions disponibles dans le contexte `Agents`, qui s'applique dans la [vue des agents](https://code.claude.com/docs/en/agent-view), ouverte avec `claude agents`. (v2.1.257+)

| Action | Par défaut | Description |
| :- | :- | :- |
| `agents:switchView` | Ctrl+S | Basculer le [regroupement des sessions](https://code.claude.com/docs/en/agent-view#organize-the-list) entre état et répertoire |
| `agents:togglePin` | Ctrl+T | [Épingler ou désépingler](https://code.claude.com/docs/en/agent-view#organize-the-list) la session sélectionnée |
| `agents:find` | Ctrl+F | Trouver des sessions par nom, avec le [filtre `n:`](https://code.claude.com/docs/en/agent-view#filter-sessions). (v2.1.288+) |
| `agents:rename` | Ctrl+R | [Renommer](https://code.claude.com/docs/en/agent-view#organize-the-list) la session sélectionnée. (v2.1.288+) |
| `agents:previousGroup` | Ctrl+Up, Meta+Up | Sauter à l'[en-tête de groupe](https://code.claude.com/docs/en/agent-view#organize-the-list) précédent. (v2.1.288+) |
| `agents:nextGroup` | Ctrl+Down, Meta+Down | Sauter à l'en-tête de groupe suivant. (v2.1.288+) |

Lorsque la vue des agents est ouverte, Claude Code utilise le raccourci `Agents` pour toute touche que le contexte `Agents` associe, et ignore un raccourci `Chat` ou `Global` sur la même touche. Par exemple, Ctrl+S dans la vue des agents change le regroupement des sessions au lieu de déclencher le `chat:stash` par défaut.

Le raccourci d'éditeur externe du champ de dispatch n'est pas une action `Agents`. La vue des agents suit le raccourci `chat:externalEditor` du contexte `Chat`, Ctrl+G par défaut.

Dans la vue des agents, les raccourcis se déclenchent sur des frappes uniques : le chord Ctrl+X Ctrl+E associé à `chat:externalEditor` n'y ouvre donc pas l'éditeur.

### Actions vocales

Actions disponibles dans le contexte `Chat` lorsque la [dictée vocale](https://code.claude.com/docs/en/voice-dictation) est activée :

| Action | Par défaut | Description |
| :- | :- | :- |
| `voice:pushToTalk` | Space | Dicter un prompt. Maintenir ou appuyer brièvement selon le mode `/voice` |

### Actions de défilement

Actions disponibles dans le contexte `Scroll` lorsque le [rendu plein écran](https://code.claude.com/docs/en/fullscreen) est activé :

| Action | Par défaut | Description |
| :- | :- | :- |
| `scroll:lineUp` | `wheelup` | Remonter d'une ligne. Le défilement à la molette de la souris déclenche cette action |
| `scroll:lineDown` | `wheeldown` | Descendre d'une ligne. Le défilement à la molette de la souris déclenche cette action |
| `scroll:pageUp` | PageUp | Remonter d'une demi-hauteur de fenêtre |
| `scroll:pageDown` | PageDown | Descendre d'une demi-hauteur de fenêtre |
| `scroll:top` | Ctrl+Home | Aller au début de la conversation |
| `scroll:bottom` | Ctrl+End | Aller au dernier message et réactiver le suivi automatique (auto-follow) |
| `scroll:halfPageUp` | (non associé) | Remonter d'une demi-hauteur de fenêtre. Même comportement que `scroll:pageUp`, fourni pour les réassociations de style vi |
| `scroll:halfPageDown` | (non associé) | Descendre d'une demi-hauteur de fenêtre. Même comportement que `scroll:pageDown`, fourni pour les réassociations de style vi |
| `scroll:fullPageUp` | (non associé) | Remonter d'une hauteur de fenêtre complète |
| `scroll:fullPageDown` | (non associé) | Descendre d'une hauteur de fenêtre complète |
| `selection:copy` | Ctrl+Shift+C / Cmd+C | Copier le texte sélectionné dans le presse-papiers |
| `selection:clear` | (non associé) | Effacer la sélection de texte active. (v2.1.234+) |
| `selection:extendLeft` | Shift+Left | Étendre la sélection active d'une colonne vers la gauche |
| `selection:extendRight` | Shift+Right | Étendre la sélection active d'une colonne vers la droite |
| `selection:extendUp` | Shift+Up | Étendre la sélection active d'une ligne vers le haut. Fait défiler la vue quand la sélection atteint le bord supérieur |
| `selection:extendDown` | Shift+Down | Étendre la sélection active d'une ligne vers le bas. Fait défiler la vue quand la sélection atteint le bord inférieur |
| `selection:extendLineStart` | Shift+Home | Étendre la sélection active jusqu'au début de la ligne |
| `selection:extendLineEnd` | Shift+End | Étendre la sélection active jusqu'à la fin de la ligne |

## Syntaxe des frappes

### Modificateurs

Utilisez les touches de modification avec le séparateur `+` :

* `ctrl` ou `control` - touche Ctrl
* `shift` - touche Shift (Maj)
* `alt`, `opt`, `option` ou `meta` - touche Alt sous Windows et Linux, touche Option sous macOS
* `cmd`, `command`, `super` ou `win` - touche Commande sous macOS, touche Windows sous Windows, touche Super sous Linux

Le groupe `cmd` n'est détecté que dans les terminaux qui signalent le modificateur Super, comme ceux qui prennent en charge le Kitty keyboard protocol ou le mode `modifyOtherKeys` de xterm. La plupart des terminaux ne l'envoient pas : utilisez `ctrl` ou `meta` pour les raccourcis qui doivent fonctionner partout.

Par exemple :

```text theme={null}
ctrl+k          Ctrl + K
shift+tab       Shift + Tab
meta+p          Option + P sur macOS, Alt + P ailleurs
ctrl+shift+c    Plusieurs modificateurs
```

### Lettres majuscules

Claude Code analyse les noms de touches sans tenir compte de la casse : `K` est le même raccourci que `k`, et `ctrl+K` est identique à `ctrl+k`. Pour associer Shift et une lettre, écrivez `shift+k`.

### Dispositions de clavier non US

Écrivez les noms de touches des raccourcis Ctrl en caractères latins, même lorsque votre disposition de clavier active produit d'autres caractères.

La façon dont Claude Code fait correspondre la touche pressée à un raccourci dépend du type de disposition :

* Avec une disposition non latine comme le cyrillique, Claude Code fait correspondre les raccourcis Ctrl selon la position de la touche sur une disposition US, lorsque le terminal utilise le Kitty keyboard protocol et signale cette position. Dans un tel terminal, avec une disposition russe active, appuyer sur Ctrl et la touche physique W déclenche `ctrl+w`. Dans un terminal qui ne signale pas la position, Claude Code utilise ce que le terminal envoie pour la frappe : un code de contrôle ASCII déclenche le raccourci latin, et une frappe qui arrive sous forme de caractère cyrillique ne correspond à aucun raccourci
* Avec des dispositions qui réorganisent les lettres latines, comme l'AZERTY, Claude Code utilise la lettre que la touche produit : appuyer sur Ctrl et la touche marquée A déclenche `ctrl+a`

### Chords

Les chords sont des séquences de frappes séparées par des espaces :

```text theme={null}
ctrl+k ctrl+s   Appuyer sur Ctrl+K, relâcher, puis Ctrl+S
```

Appuyez sur chaque frappe moins de 3 secondes après la précédente. Si vous attendez plus longtemps, Claude Code annule le chord et affiche un bref avis l'indiquant.

### Touches spéciales

* `escape` ou `esc` - touche Échap
* `enter` ou `return` - touche Entrée
* `tab` - touche Tab
* `space` - barre d'espace
* `up`, `down`, `left`, `right` - touches fléchées
* `pageup`, `pagedown` - touches Page Up et Page Down
* `home`, `end` - touches Début (Home) et Fin (End)
* `backspace`, `delete` - touches d'effacement
* `wheelup`, `wheeldown` - événements de défilement de la molette de la souris

## Supprimer des raccourcis par défaut

Affectez `null` à une action pour supprimer un raccourci par défaut :

```json theme={null}
{
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+s": null
      }
    }
  ]
}
```

Cela fonctionne aussi pour les chords. Supprimer tous les chords qui partagent un préfixe libère ce préfixe pour un raccourci à touche unique. Un chord dans n'importe quel contexte actif garde son préfixe réservé : vous devez donc supprimer chaque chord dans le contexte qui le définit.

Claude Code définit ces chords par défaut sur le préfixe `ctrl+x`, par contexte :

* `Chat` : `ctrl+x ctrl+k`, `ctrl+x ctrl+e`, `ctrl+x enter`, `ctrl+x ctrl+a`, `ctrl+x ctrl+s` et `ctrl+x tab`
* `Task` : `ctrl+x ctrl+b`
* `DiffPanel` : `ctrl+x b`
* `Pane` : `ctrl+x left`, `ctrl+x right`, `ctrl+x up`, `ctrl+x down` et `ctrl+x x`
* `PaneField` : `ctrl+x x`

Le chord `ctrl+x enter` nécessite la v2.1.247+, `ctrl+x b`, `ctrl+x ctrl+a` et `ctrl+x tab` nécessitent la v2.1.260+, et `ctrl+x ctrl+s` nécessite la v2.1.275+.

Pour récupérer `ctrl+x` lui-même comme raccourci à touche unique, supprimez-les tous :

```json theme={null}
{
  "bindings": [
    {
      "context": "Task",
      "bindings": {
        "ctrl+x ctrl+b": null
      }
    },
    {
      "context": "DiffPanel",
      "bindings": {
        "ctrl+x b": null
      }
    },
    {
      "context": "Pane",
      "bindings": {
        "ctrl+x left": null,
        "ctrl+x right": null,
        "ctrl+x up": null,
        "ctrl+x down": null,
        "ctrl+x x": null
      }
    },
    {
      "context": "PaneField",
      "bindings": {
        "ctrl+x x": null
      }
    },
    {
      "context": "Chat",
      "bindings": {
        "ctrl+x ctrl+k": null,
        "ctrl+x ctrl+e": null,
        "ctrl+x enter": null,
        "ctrl+x ctrl+a": null,
        "ctrl+x ctrl+s": null,
        "ctrl+x tab": null,
        "ctrl+x": "chat:newline"
      }
    }
  ]
}
```

Si vous supprimez certains chords d'un préfixe mais pas tous, appuyer sur le préfixe fait toujours entrer en mode d'attente de chord pour les raccourcis restants.

## Raccourcis réservés

Ces raccourcis ne peuvent pas être réassociés :

| Raccourci | Raison |
| :- | :- |
| Ctrl+C | Interruption/annulation codée en dur |
| Ctrl+D | Sortie codée en dur |
| Ctrl+M | Claude Code le reçoit toujours comme Enter |
| Ctrl+\[ | Claude Code le reçoit toujours comme Escape. Dans les terminaux qui utilisent le Kitty keyboard protocol, nécessite la v2.1.242+ |
| Ctrl+I | Claude Code le reçoit toujours comme Tab |
| Ctrl+H | Envoie l'octet ASCII backspace. [La façon dont Claude Code le lit sous Windows](https://code.claude.com/docs/en/terminal-config#fix-backspace-deleting-a-whole-word-on-windows) dépend de votre terminal et de la variable d'environnement [`CLAUDE_CODE_BS_AS_CTRL_BACKSPACE`](https://code.claude.com/docs/en/env-vars) |
| Caps Lock | N'est pas transmis aux applications de terminal |

## Conflits avec le terminal

Certains raccourcis peuvent entrer en conflit avec les multiplexeurs de terminal :

| Raccourci | Conflit |
| :- | :- |
| Ctrl+B | Préfixe de tmux (appuyer deux fois pour l'envoyer) |
| Ctrl+A | Préfixe de GNU screen |
| Ctrl+Z | Suspension de processus Unix (SIGTSTP) |

## Champs de texte

Si vous associez une lettre, un chiffre ou Space seuls, vous pouvez toujours taper ce caractère dans un champ de texte à l'intérieur d'un dialogue ou d'un panneau. Un tel champ est par exemple la réponse `Other` à une question que Claude vous pose. Tant que le champ a le focus, une touche imprimable pressée sans Ctrl, Alt ni Cmd va dans le champ, et Claude Code ne la compare pas à vos raccourcis.

Ces touches exécutent toujours leurs raccourcis pendant que le champ a le focus :

* Les touches qui ne produisent pas de caractère, comme Enter, Escape, Tab et les flèches
* Toute touche pressée avec Ctrl, Alt ou Cmd
* La seconde frappe d'un [chord](#chords) déjà en cours

Au prompt principal, Claude Code compare chaque touche aux contextes actifs, comme `Chat`, et ne tape la touche que si aucun raccourci ne la prend.

## Interaction avec le mode vim

Lorsque le mode vim est activé via `/config` → Editor mode, les keybindings et le mode vim fonctionnent indépendamment :

* **Le mode vim** gère la saisie au niveau du champ de texte (déplacement du curseur, modes, mouvements)
* **Les keybindings** gèrent les actions au niveau des composants (afficher les to-do, envoyer, etc.)
* La touche Escape en mode vim fait passer du mode INSERT au mode NORMAL ; elle ne déclenche pas `chat:cancel`
* La plupart des raccourcis Ctrl+touche traversent le mode vim jusqu'au système de keybindings
* Les touches vim ne sont pas réassociables via le fichier de keybindings. Pour associer une séquence de deux touches en mode INSERT comme `jj` à Escape, utilisez le réglage [`vimInsertModeRemaps`](https://code.claude.com/docs/en/interactive-mode#remap-insert-mode-key-sequences)
* En mode NORMAL de vim, `?` affiche le menu d'aide (comportement vim)
* En mode NORMAL de vim, `/` ouvre la recherche dans l'historique, comme Ctrl+R en mode standard

## Validation

Claude Code valide vos keybindings et écrit un avertissement dans le journal de débogage pour :

* Les erreurs d'analyse (JSON ou structure invalide)
* Les modificateurs mal orthographiés, comme `ctl+k`. Claude Code supprime la partie qu'il ne reconnaît pas et applique le raccourci à la frappe restante, `k` dans cet exemple.
* Les noms de contexte invalides
* Les valeurs d'action invalides, comme une action qui n'est ni une chaîne ni `null`
* Les noms d'action inconnus, comme une faute de frappe dans une action enregistrée. Claude Code ignore le raccourci et conserve tout raccourci par défaut de cette touche.
* Les conflits avec des raccourcis réservés
* Les raccourcis en double dans un même contexte

Lancez Claude Code avec [`--debug`](https://code.claude.com/docs/en/cli-reference#cli-flags) pour voir les détails.
