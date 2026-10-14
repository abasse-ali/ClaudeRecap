# 21 — Référence : toutes les actions clavier personnalisables

[← Sommaire](README.md)

Chaque raccourci de Claude Code est une **action** rattachée à un **contexte**. On les modifie dans `~/.claude/keybindings.json` (`/keybindings` l'ouvre ; rechargement à chaud). Ce fichier liste **toutes** les actions avec leur touche par défaut. « (libre) » = aucune touche par défaut, à vous d'en attribuer une.

## Format du fichier

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "$docs": "https://code.claude.com/docs/en/keybindings",
  "bindings": [
    { "context": "Chat", "bindings": { "ctrl+e": "chat:externalEditor", "ctrl+s": null } },
    { "context": "MessageSelector", "bindings": { "o": "select:accept" } }
  ]
}
```

- **Modificateurs** : `ctrl`/`control`, `shift`, `alt`/`opt`/`option`/`meta` (Option sur macOS), `cmd`/`command`/`super`/`win` (seulement dans les terminaux qui transmettent Super, ex. protocole Kitty).
- **Touches spéciales** : `escape`/`esc`, `enter`/`return`, `tab`, `space`, `up`/`down`/`left`/`right`, `pageup`/`pagedown`, `home`/`end`, `backspace`/`delete`, `wheelup`/`wheeldown`.
- **Accords** (chords) : touches séparées par un espace, ex. `ctrl+k ctrl+s` (3 s max entre chaque).
- Insensible à la casse (`shift+k` pour Maj+K). Claviers AZERTY : la lettre tapée compte.
- `null` **désactive** un raccourci par défaut. Pour récupérer `ctrl+x` seul, il faut désactiver tous les accords qui commencent par `ctrl+x`.

## Contextes

| Contexte | Actif quand… |
|----------|--------------|
| `Global` | Partout |
| `Chat` | Zone de saisie principale |
| `Autocomplete` | Menu d'autocomplétion ouvert |
| `Settings` | Menu des réglages (`/config`) |
| `Confirmation` | Demande de permission ou de confirmation |
| `Tabs` | Navigation par onglets |
| `Help` | Aide affichée |
| `Transcript` | Visionneuse de transcription |
| `HistorySearch` | Recherche dans l'historique (`Ctrl+R`) |
| `Task` | Une tâche tourne au premier plan |
| `ThemePicker` | Sélecteur de thème |
| `Attachments` | Navigation dans les pièces jointes (images) |
| `Footer` | Éléments du pied de page (tâches, équipe, diff, artifacts) |
| `MessageSelector` | Liste des messages du menu rewind |
| `DiffDialog` | Visionneuse de diff |
| `DiffPanel` | Panneau de diff ouvert |
| `ModelPicker` | Sélecteur de modèle (effort) |
| `EffortSlider` | Curseur `/effort` |
| `Select` | Listes de choix génériques |
| `Plugin` | Fenêtre `/plugin` |
| `AbovePrompt`, `AbovePromptInput`, `AbovePromptSelect` | Bande au-dessus du prompt (mods) |
| `Pane`, `PaneField` | Volet dessiné par un mod |
| `Agents` | Agent view (`claude agents`) |
| `Scroll` | Défilement et sélection en plein écran |

## Actions par contexte

### Application (`Global`)

| Action | Défaut | Effet |
|--------|--------|-------|
| `app:interrupt` | `Ctrl+C` | Annuler l'opération en cours |
| `app:exit` | `Ctrl+D` | Quitter (2× en 800 ms) |
| `app:redraw` | (libre) | Redessiner le terminal |
| `app:toggleTodos` | `Ctrl+T` | Afficher/masquer la checklist de Claude |
| `app:toggleTranscript` | `Ctrl+O` | Transcription détaillée |

### Historique

| Action | Défaut | Effet |
|--------|--------|-------|
| `history:search` | `Ctrl+R` | Ouvrir la recherche |
| `history:previous` / `history:next` | `↑` / `↓` | Élément précédent / suivant |

### Saisie (`Chat`)

| Action | Défaut | Effet |
|--------|--------|-------|
| `chat:submit` | `Entrée` | Envoyer |
| `chat:queueSubmit` | `Ctrl+X Entrée` | Envoyer en file d'attente sans interrompre |
| `chat:sendNow` | `Ctrl+Entrée`, `Ctrl+X Ctrl+S` | Envoyer immédiatement la file |
| `chat:newline` | `Ctrl+J` | Nouvelle ligne |
| `chat:cancel` | `Échap` | Annuler la saisie |
| `chat:undo` | `Ctrl+_`, `Ctrl+Shift+-` | Annuler |
| `chat:clearInput` | `Ctrl+L` | Redessiner l'écran (garde saisie et conversation) |
| `chat:clearScreen` | `Cmd+K` | Idem |
| `chat:cycleMode` | `Shift+Tab` (`Meta+M` sur certains Windows) | Changer de mode de permission |
| `chat:modelPicker` | `Meta+P` | Sélecteur de modèle |
| `chat:fastMode` | `Meta+O` | Mode rapide |
| `chat:thinkingToggle` | `Meta+T` | Réflexion étendue |
| `chat:externalEditor` | `Ctrl+G`, `Ctrl+X Ctrl+E` | Éditeur externe |
| `chat:stash` | `Ctrl+S` | Mettre le prompt de côté |
| `chat:imagePaste` | `Ctrl+V` (`Alt+V` Windows/WSL) | Coller une image |
| `chat:killAgents` | `Ctrl+X Ctrl+K` | Arrêter tous les sous-agents de fond (2× en 3 s) |

### Autocomplétion

| Action | Défaut | Effet |
|--------|--------|-------|
| `autocomplete:accept` | `Tab` | Accepter |
| `autocomplete:dismiss` | `Échap` | Fermer |
| `autocomplete:previous` / `next` | `↑` / `↓` | Naviguer |

### Confirmation (demandes de permission)

| Action | Défaut | Effet |
|--------|--------|-------|
| `confirm:yes` | `Entrée` | Confirmer |
| `confirm:no` | `Échap` | Refuser |
| `confirm:previous` / `next` | `↑` / `↓` | Option précédente / suivante |
| `confirm:toggle` | `Espace` | Cocher/décocher |
| `confirm:nextField` | `Tab` | Dans `/fast`, activer/désactiver |
| `confirm:cycleMode` | `Shift+Tab` | Fermer le champ de commentaire, ou choisir « autoriser pour la session » |
| `confirm:previousField`, `permission:toggleDebug` | (libre) | Sans effet (gardés pour compatibilité) |

### Transcription et recherche

| Action | Défaut | Effet |
|--------|--------|-------|
| `transcript:toggleShowAll` | `Ctrl+E` | Tout afficher |
| `transcript:exit` | `q`, `Ctrl+C`, `Échap` | Quitter |
| `historySearch:next` | `Ctrl+R` | Résultat suivant |
| `historySearch:accept` | `Échap`, `Tab` | Accepter |
| `historySearch:cancel` | `Ctrl+C` | Annuler |
| `historySearch:execute` | `Entrée` | Exécuter |
| `historySearch:cycleScope` | `Ctrl+S` | Portée : session → projet → partout |

### Tâches, thème, aide, onglets, pièces jointes

| Action | Défaut | Effet |
|--------|--------|-------|
| `task:background` | `Ctrl+B`, `Ctrl+X Ctrl+B` | Passer la tâche en arrière-plan |
| `theme:toggleSyntaxHighlighting` | `Ctrl+T` | Coloration syntaxique (sélecteur de thème) |
| `help:dismiss` | `Échap` | Fermer l'aide |
| `tabs:next` / `tabs:previous` | `Tab`/`→` ; `Shift+Tab`/`←` | Onglet suivant / précédent |
| `attachments:next` / `previous` | `→` / `←` | Pièce jointe suivante / précédente |
| `attachments:remove` | `Retour arrière`, `Suppr` | Retirer la pièce jointe |
| `attachments:exit` | `↓`, `Échap` | Quitter la navigation |

### Pied de page

| Action | Défaut | Effet |
|--------|--------|-------|
| `footer:next` / `previous` | `→` / `←` | Élément suivant / précédent |
| `footer:up` / `down` | `↑`/`Ctrl+P` ; `↓`/`Ctrl+N` | Monter / descendre |
| `footer:openSelected` | `Entrée` | Ouvrir |
| `footer:clearSelection` | `Échap` | Désélectionner |
| `footer:close` | `x` | Arrêter l'agent/workflow sélectionné ou retirer sa ligne |

### Menu rewind (`MessageSelector`)

Utilise les actions `select:*` (voir plus bas) ; on peut les redéfinir spécifiquement dans ce contexte.

### Visionneuse de diff (`DiffDialog`)

| Action | Défaut | Effet |
|--------|--------|-------|
| `diff:dismiss` | `Échap` | Fermer / revenir à la liste |
| `diff:previousSource` / `nextSource` | `←` / `→` | Source précédente / suivante (Current, T1, T2…) |
| `diff:previousFile` / `nextFile` | `↑`/`K` ; `↓`/`J` | Fichier précédent / suivant (ou défiler) |
| `diff:back` | (libre) | Retour |
| `scroll:pageUp` / `pageDown` | `PgUp` / `PgDn` | Demi-page |
| `scroll:fullPageUp` / `fullPageDown` | `Shift+Espace`/`B` ; `Espace` | Page entière |
| `scroll:top` / `bottom` | `G`/`Début` ; `Shift+G`/`Fin` | Haut / bas |

### Panneau de diff (`DiffPanel`)

| Action | Défaut | Effet |
|--------|--------|-------|
| `app:toggleReplTab` | (libre) | Ouvrir/fermer le panneau |
| `app:cycleDiffBase` | `Ctrl+X B` | Base : session → non commité → branche |
| `app:diffFileListUp` / `Down` | `Ctrl+↑`/`Meta+↑` ; `Ctrl+↓`/`Meta+↓` | Défiler la liste des fichiers |
| `app:toggleDiffNoiseFilter` | (libre) | Afficher/masquer tests et fichiers générés |
| `app:toggleDiffPreSession` | (libre) | Déplier les changements d'avant la session |

### Modèle et effort

| Action | Défaut | Effet |
|--------|--------|-------|
| `modelPicker:decreaseEffort` / `increaseEffort` | `←` / `→` | Effort − / + dans `/model` |
| `modelPicker:thisSessionOnly` | `s` | Modèle pour cette session seulement |
| `effortSlider:decreaseEffort` / `increaseEffort` | `←` / `→` | Effort − / + dans `/effort` |
| `effortSlider:toggleUltracode` | `Tab` | Activer/désactiver ultracode |
| `effortSlider:thisSessionOnly` | `s` | Effort pour cette session seulement |

### Listes (`Select`)

| Action | Défaut | Effet |
|--------|--------|-------|
| `select:next` / `previous` | `↓`/`J`/`Ctrl+N` ; `↑`/`K`/`Ctrl+P` | Option suivante / précédente |
| `select:pageUp` / `pageDown` | `PgUp` / `PgDn` | Page |
| `select:first` / `last` | `Début` / `Fin` | Première / dernière |
| `select:accept` | `Entrée` | Valider |
| `select:cancel` | `Échap` | Annuler |

### Plugins

| Action | Défaut | Effet |
|--------|--------|-------|
| `plugin:toggle` | `Espace` | Sélectionner/activer |
| `plugin:install` | `I` | Installer la sélection |
| `plugin:favorite` | `F` | Mettre en favori |

### Bande au-dessus du prompt et volets de mods

| Action | Défaut | Effet |
|--------|--------|-------|
| `abovePrompt:toggle` | `Ctrl+X Ctrl+A` | Réduire/déplier la bande |
| `abovePrompt:focus` | `Ctrl+X Tab` | Donner le focus à la bande puis aux volets |
| `abovePrompt:next` / `previous` | `Tab` / `Shift+Tab` | Contrôle suivant / précédent |
| `abovePrompt:press` | `Entrée` | Appuyer / valider |
| `abovePrompt:leave` | `Échap` | Revenir au prompt |
| `abovePrompt:highlightNext` / `Previous` | `↓` / `↑` | Option suivante / précédente |
| `pane:scrollUp` / `scrollDown` | `↑` / `↓` | Défiler le volet |
| `pane:pageUp` / `pageDown` / `top` / `bottom` | `PgUp` / `PgDn` / `Début` / `Fin` | Navigation |
| `pane:grow` / `shrink` | `Ctrl+X ←`/`↑` ; `Ctrl+X →`/`↓` | Agrandir / réduire |
| `pane:close` | `Ctrl+X X` | Fermer le volet |
| `pane:next` / `previous` | (libre) | Volet suivant / précédent |

### Réglages (`Settings`)

| Action | Défaut | Effet |
|--------|--------|-------|
| `settings:search` | `/` | Rechercher |
| `settings:retry` | `R` | Recharger les données d'usage |
| `select:accept` | `Entrée`, `Espace` | Changer le réglage / ouvrir le sous-menu |
| `confirm:no` | `Échap` | Fermer (déjà enregistré) |

### Agent view (`Agents`)

| Action | Défaut | Effet |
|--------|--------|-------|
| `agents:switchView` | `Ctrl+S` | Grouper par état / par dossier |
| `agents:togglePin` | `Ctrl+T` | Épingler |
| `agents:find` | `Ctrl+F` | Chercher par nom |
| `agents:rename` | `Ctrl+R` | Renommer |
| `agents:previousGroup` / `nextGroup` | `Ctrl+↑`/`Meta+↑` ; `Ctrl+↓`/`Meta+↓` | Groupe précédent / suivant |

### Voix

| Action | Défaut | Effet |
|--------|--------|-------|
| `voice:pushToTalk` | `Espace` | Dicter (maintenir ou taper selon `/voice`) |

### Défilement et sélection (plein écran, `Scroll`)

| Action | Défaut | Effet |
|--------|--------|-------|
| `scroll:lineUp` / `lineDown` | molette | Une ligne |
| `scroll:pageUp` / `pageDown` | `PgUp` / `PgDn` | Demi-écran |
| `scroll:top` / `bottom` | `Ctrl+Début` / `Ctrl+Fin` | Début / dernier message (réactive le suivi) |
| `scroll:halfPageUp/Down`, `scroll:fullPageUp/Down` | (libre) | Pour des raccourcis façon vi |
| `selection:copy` | `Ctrl+Shift+C` / `Cmd+C` | Copier la sélection |
| `selection:clear` | (libre) | Effacer la sélection |
| `selection:extendLeft/Right/Up/Down` | `Shift+←/→/↑/↓` | Étendre la sélection |
| `selection:extendLineStart/End` | `Shift+Début` / `Shift+Fin` | Jusqu'au début / fin de ligne |

## Raccourcis réservés (non modifiables)

| Touche | Raison |
|--------|--------|
| `Ctrl+C` | Interruption/annulation fixe |
| `Ctrl+D` | Sortie fixe |
| `Ctrl+M` | Toujours reçu comme `Entrée` |
| `Ctrl+[` | Toujours reçu comme `Échap` |
| `Ctrl+I` | Toujours reçu comme `Tab` |
| `Ctrl+H` | Octet de retour arrière |
| Verr. Maj | Non transmis aux applications du terminal |

Conflits possibles : `Ctrl+B` (préfixe tmux : appuyer 2×), `Ctrl+A` (GNU screen), `Ctrl+Z` (suspension Unix).

## Accords `Ctrl+X` par défaut

- `Chat` : `ctrl+x ctrl+k`, `ctrl+x ctrl+e`, `ctrl+x enter`, `ctrl+x ctrl+a`, `ctrl+x ctrl+s`, `ctrl+x tab`
- `Task` : `ctrl+x ctrl+b` ; `DiffPanel` : `ctrl+x b` ; `Pane` : `ctrl+x ←/→/↑/↓`, `ctrl+x x` ; `PaneField` : `ctrl+x x`

## Vim et raccourcis

Le mode Vim (édition du texte) et les raccourcis (actions) sont indépendants : `Échap` en Vim passe en NORMAL (pas `chat:cancel`), la plupart des `Ctrl+…` passent au système de raccourcis, les touches Vim ne se remappent pas via ce fichier (utiliser `vimInsertModeRemaps`), `?` en NORMAL affiche l'aide, `/` en NORMAL ouvre la recherche d'historique.

## Raccourcis non remappables de l'édition de texte

`Ctrl+A/E/K/U/W/Y`, `Alt+B/F/D/Y`, `Option+Delete`, `Ctrl+Backspace` suivent les conventions readline et ne sont pas dans ce fichier (voir [02](02-interface-terminal.md)).
