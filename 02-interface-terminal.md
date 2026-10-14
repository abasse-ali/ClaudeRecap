# 02 — L'interface du terminal (CLI)

[← Sommaire](README.md)

On lance l'interface avec `claude`. Tout se pilote au clavier ; en **rendu plein écran** la souris fonctionne aussi.

## Ce qu'on voit à l'écran

| Zone | Ce qu'elle montre |
|------|-------------------|
| **En-tête de démarrage** | Version, modèle, dossier de travail, compte ; nom de l'agent (`@nom`) si la session tourne avec `--agent` |
| **Transcription** | Vos messages, les réponses de Claude, les appels d'outils repliés (« Called slack 3 times »), les diffs des modifications |
| **Spinner** | Verbe animé + temps écoulé + jetons générés, ex. `Deciphering… (10m 27s · ↓ 5.5k tokens)` |
| **Zone de saisie (prompt)** | Votre message ; suggestion grisée du prochain prompt ; couleur personnalisable (`/color`) ; nom de la session (`/rename`) |
| **Indicateur de mode** | Sous le prompt : mode de permission actuel, ex. `⏵⏵ accept edits on`, plan mode, auto mode, `Manual` |
| **Pied de page (footer)** | Badge de PR GitHub `PR #446` (souligné vert = approuvée, jaune = en attente, rouge = changements demandés, gris = brouillon) ou `MR !N` GitLab ; tâches de fond ; équipe ; diff ; artifacts |
| **Panneau des sous-agents** | Sous le prompt : agents et workflows en cours, navigables avec les flèches |
| **Liste de tâches** | Checklist de Claude (`Ctrl+T`) : en attente / en cours / terminé |
| **Barre d'état (status line)** | Ligne personnalisable (modèle, dossier, git, contexte, coût…) via `/statusline` |
| **Indicateurs** | `↯` = mode rapide actif ; `/rc active` = Remote Control connecté ; notes « You should know » au-dessus du prompt (mod optionnel) |
| **Récapitulatif** | En revenant après une absence (≥ 3 min, terminal hors focus), une ligne résume la session ; `/recap` à la demande |

Les messages envoyés ou en file d'attente apparaissent **en gris** tant que Claude n'a pas commencé à y répondre.

## Les préfixes et caractères spéciaux du prompt

| Saisie | Effet |
|--------|-------|
| `/` en début de message | Menu des **commandes et skills** (filtrage en tapant ; `Tab` pour compléter) |
| `/` au milieu du texte | Complétion d'une commande en cours de phrase (ex. `lance les tests, puis /com`) |
| `!` en début de message | **Mode shell** : exécute la commande directement, ajoute la sortie au contexte, Claude réagit à la sortie |
| `@` | **Mention de fichier/dossier** avec autocomplétion (`@src/auth.ts`, `@src/components/`) ; mentionne aussi un sous-agent (`@"code-reviewer (agent)"` ou `@agent-nom`), une ressource MCP (`@serveur:ressource`), ou une autre session ouverte |
| `:` | **Emoji** : `:heart:` → ❤️, `:hea` ouvre des suggestions |
| `?` sur prompt vide | Affiche/masque l'**aide des raccourcis** |
| `\` + `Entrée` | Nouvelle ligne (fonctionne partout) |
| `←` sur prompt vide | Passe la session en arrière-plan et ouvre l'**agent view** |
| Coller une image | `Ctrl+V` (ou `Cmd+V` sous iTerm2, `Alt+V` sous Windows/WSL) insère une puce `[Image #N]` |
| Coller un long texte | Replié en `[Pasted text #N]` |
| Glisser-déposer un fichier | Ajoute son chemin |

### Mode shell `!`

```
! npm test
! git status
```

- La sortie entre dans le contexte et Claude y répond automatiquement (désactivable avec `respondToBashCommands: false`).
- Complétion `Tab` depuis l'historique des commandes `!` du projet, et complétion de chemins dès qu'on tape un `/`.
- `Ctrl+B` pour envoyer une commande longue en arrière-plan.
- Sortir du mode : `Esc`, `Retour arrière` ou `Ctrl+U` sur prompt vide.
- Les commandes tapées vous-même s'exécutent hors de la sandbox (la sandbox s'applique aux commandes de Claude).

## Tous les raccourcis clavier

> Sur macOS, les raccourcis `Alt+…`/`Option+…` demandent de régler la touche Option comme « Meta » dans le terminal (`/terminal-setup` aide à configurer). Tous les raccourcis sont personnalisables (voir [12](12-modeles-et-personnalisation.md#raccourcis-clavier-personnalisés)).

### Contrôle général

| Raccourci | Action |
|-----------|--------|
| `Ctrl+C` | Interrompt l'opération ; si rien ne tourne : 1er appui efface le prompt, 2e appui quitte. `↑` sur prompt vide récupère le brouillon effacé |
| `Esc` | **Interrompt Claude** (garde le travail fait) ou ferme une boîte de dialogue ; sur une demande de permission = « Non » |
| `Esc` `Esc` | Prompt rempli : efface le brouillon (récupérable avec `↑`). Prompt vide : ouvre le menu **rewind** (retour en arrière) |
| `Ctrl+D` | Quitter (2 appuis en moins de 800 ms) ; supprime le caractère suivant si le prompt contient du texte |
| `Shift+Tab` (ou `Alt+M` sur certains Windows) | **Change de mode de permission** : Manual → Accept edits → Plan → (Bypass) → Auto |
| `Option+P` / `Alt+P` | Changer de **modèle** sans effacer le prompt |
| `Option+T` / `Alt+T` | Activer/désactiver la **réflexion étendue** (sans effet sur les modèles qui réfléchissent toujours) |
| `Option+O` / `Alt+O` | Activer/désactiver le **mode rapide** |
| `Ctrl+O` | **Visionneuse de transcription** détaillée (outils, horodatage, modèle par message) |
| `Ctrl+R` | **Recherche inversée** dans l'historique des prompts |
| `Ctrl+T` | Afficher/masquer la **liste de tâches** de Claude |
| `Ctrl+B` | Envoyer une commande Bash ou un agent en **arrière-plan** (2× sous tmux) |
| `Ctrl+X Ctrl+K` | **Arrêter tous les sous-agents** en arrière-plan (2 appuis en 3 s) |
| `Ctrl+S` | **Mettre de côté** le prompt (stash) ; re-appui sur prompt vide = le restaurer |
| `Ctrl+G` ou `Ctrl+X Ctrl+E` | Éditer le prompt dans votre **éditeur externe** (`$EDITOR`) |
| `Ctrl+L` | Redessiner l'écran (si l'affichage est corrompu) |
| `Ctrl+Z` | Suspendre Claude Code (Unix ; `fg` pour reprendre) |
| `Ctrl+Entrée` ou `Ctrl+X Ctrl+S` | **Envoyer tout de suite** les messages en file d'attente |
| `Ctrl+X Entrée` | Envoyer le message « en file », sans jamais interrompre le tour en cours |
| `Ctrl+V` / `Cmd+V` / `Alt+V` | Coller une image |
| `Tab` | Accepter une suggestion d'autocomplétion ; sur une demande de permission, ajouter un **commentaire** à la réponse Oui/Non |
| `Tab` ou `→` | Accepter la **suggestion de prompt** grisée |
| `↑` / `↓` (ou `Ctrl+P`/`Ctrl+N`) | Déplacer le curseur dans un prompt multiligne, puis naviguer dans l'historique ; `↑` reprend les messages en file d'attente |
| `←` / `→` | Changer d'onglet dans les boîtes de dialogue |
| Maintenir ou taper `Espace` | **Dictée vocale** (si activée avec `/voice`) |

### Édition de texte (style readline)

| Raccourci | Action |
|-----------|--------|
| `Ctrl+A` / `Ctrl+E` | Début / fin de ligne |
| `Ctrl+K` | Supprimer jusqu'à la fin de la ligne |
| `Ctrl+U` | Supprimer jusqu'au début de la ligne (`Cmd+Backspace` sur macOS) |
| `Ctrl+W` | Supprimer jusqu'à l'espace précédent (un chemin entier d'un coup) |
| `Option+Delete` / `Ctrl+Backspace` | Supprimer le mot précédent |
| `Ctrl+Y` | Recoller le texte supprimé ; puis `Alt+Y` pour parcourir l'historique des suppressions |
| `Alt+B` / `Alt+F` | Mot précédent / suivant |
| `Alt+D` | Supprimer jusqu'à la fin du mot |
| `Ctrl+_` ou `Ctrl+Shift+-` | Annuler la dernière modification du prompt |

### Saisie multiligne

| Méthode | Raccourci |
|---------|-----------|
| Universel | `\` puis `Entrée` |
| Sans configuration | `Ctrl+J` |
| Natif dans iTerm2, WezTerm, Ghostty, Kitty, Warp, Terminal Apple, Windows Terminal | `Shift+Entrée` |
| macOS avec Option = Meta | `Option+Entrée` |
| VS Code, Cursor, Alacritty, Zed… | Lancer `/terminal-setup` pour installer `Shift+Entrée` |

### Visionneuse de transcription (`Ctrl+O`)

| Touche | Action |
|--------|--------|
| `?` | Aide des raccourcis (plein écran) |
| `{` / `}` | Prompt précédent / suivant (plein écran) |
| `/` | Rechercher dans la conversation (plein écran) |
| `[` | Écrire toute la conversation dans le défilement natif du terminal (pour `Cmd+F`, tmux) |
| `v` | Ouvrir la conversation dans `$VISUAL`/`$EDITOR` |
| `Ctrl+E` | Tout afficher (rendu classique) |
| `q`, `Ctrl+C`, `Esc` | Quitter la visionneuse |

## Mode Vim

Activable dans `/config` → **Editor mode** (ou `"editorMode": "vim"`).

- **Changer de mode** : `Esc`/`Ctrl+[` → NORMAL ; `i`, `I`, `a`, `A`, `o`, `O` → INSERT ; `v` / `V` → VISUAL.
- **Déplacements** : `h j k l`, `w e b`, `0 $ ^`, `gg G`, `f F t T` + `;` `,`, `/` (recherche dans l'historique).
- **Édition** : `x`, `r`, `dd`, `D`, `dw/de/db`, `df/dt`, `dj/dk`, `dgg/dG`, `d0/c0/y0`, `cc`, `C`, `cw/ce/cb`, `s`, `S`, `yy/Y`, `yw/ye/yb`, `p/P`, `>>`, `<<`, `J`, `u`, `.`.
- **Objets texte** : `iw/aw`, `iW/aW`, `i"/a"`, `i'/a'`, `i(/a(`, `i[/a[`, `i{/a{`.
- **Visuel** : `d/x`, `y`, `c/s`, `p`, `r`, `~ u U`, `> <`, `J`, `o`.
- Remapper une séquence vers Échap : `"vimInsertModeRemaps": { "jj": "<Esc>" }`.

## Historique et recherche

- L'historique est **par dossier de travail**, et `↑` remonte aussi aux sessions précédentes du projet.
- `Ctrl+R` : recherche inversée (`Ctrl+R` à nouveau = résultat plus ancien ; `Tab`/`Esc` = accepter et éditer ; `Entrée` = exécuter). En plein écran, une boîte de recherche avec `Ctrl+S` pour changer de portée (session / projet / tous projets).

## Envoyer des messages pendant que Claude travaille (file d'attente)

- Tapez et faites `Entrée` pendant un tour : le message est **mis en file**, sans interrompre.
- Un message en file est lu par Claude **dès la fin des appels d'outils en cours**, dans le même tour.
- Les commandes `/` et `!` en file s'exécutent à la fin du tour, dans l'ordre.
- `Ctrl+Entrée` envoie la file immédiatement ; `↑` reprend les messages en file pour les modifier ; `Esc` interrompt le tour et envoie la file.
- Certaines commandes s'appliquent tout de suite sans attendre (ex. `/status`, `/usage`, `/tasks`, `/model`, `/effort`, `/fast`, `/permissions`, `/add-dir`).

## Questions annexes avec `/btw`

`/btw quel était le nom du fichier de config ?` pose une question **sans l'ajouter à l'historique** de la conversation. Claude répond à partir de ce qu'il sait déjà (pas d'outils), même pendant qu'il travaille.

Touches dans la fenêtre de réponse : `Espace`/`Entrée`/`Esc` fermer, `↑`/`↓` défiler, `Shift+←/→` (ou `[` `]`) réponses précédentes, `c` copier en Markdown, `f` **forker** la question dans un sous-agent qui a accès aux outils, `x` effacer l'historique des `/btw`.

## Revue des changements avec `/diff`

- **Plein écran** : un **panneau de diff** s'ouvre à côté de la conversation et se met à jour en direct (s'ouvre tout seul si le terminal fait ≥ 144 colonnes).
  - Cliquer un fichier pour y aller ; bouton **`ask`** pour joindre le diff d'un fichier au prochain prompt ; sélecteur **`source`** pour voir les modifications d'un tour précis (`T1`, `T2`…) ou `Current` ; `Ctrl+X B` change la base de comparaison (session / non commité / depuis la branche par défaut).
- **Rendu classique** : boîte de dialogue au-dessus du prompt (`↑/↓` choisir, `Entrée` ouvrir, `Tab` aller au sélecteur `source` ou au bouton `ask`, `Esc` fermer).

## Rendu plein écran

`/tui fullscreen` (ou `/tui default` pour revenir) : rendu sans scintillement, prompt fixe en bas, mémoire stable.

- **Souris** : cliquer dans le prompt pour placer le curseur ; cliquer une suggestion, une option de menu, une valeur de `/config` ; cliquer un résultat d'outil replié pour le déplier ; `Cmd`/`Ctrl`+clic sur une URL ou un chemin pour l'ouvrir ; glisser pour sélectionner (copie automatique au relâchement, désactivable) ; molette pour défiler (`/scroll-speed` pour la vitesse).
- `Ctrl+O` puis `/` pour chercher ; `[` pour renvoyer dans le défilement natif.
- `/focus` : vue « focus » (seulement votre dernier prompt, un résumé des outils et la réponse finale).

## Dictée vocale

- `/voice` (bascule), `/voice hold` (maintenir `Espace` pour parler, par défaut), `/voice tap` (taper pour démarrer/envoyer), `/voice off`.
- `Esc` ou `Ctrl+C` annule l'enregistrement. La langue suit le réglage `language`.
- L'audio est transcrit sur les serveurs d'Anthropic ; nécessite un compte Claude.ai.

## Autres éléments d'interface

- **Suggestions de prompt** : après chaque réponse, une suggestion grisée du prochain message (`Tab` pour l'accepter) ; désactivable dans `/config`.
- **Correcteur orthographique** du prompt (optionnel, via aspell/hunspell/ispell : réglage `spellcheck`).
- **Caractères invisibles** : supprimés automatiquement des textes collés (protection contre les instructions cachées).
- **Liens cliquables** : références `owner/repo#123` vers les issues ; chemins de fichiers.
- **Attente de limite d'usage** : si une limite claude.ai est atteinte, Claude attend et **reprend tout seul** à la réinitialisation (`Esc` pour annuler, `/rate-limit-options` pour les options).
- **Thèmes** : `/theme` (auto, clair, sombre, daltoniens, ANSI, personnalisés) ; `Ctrl+T` dans le sélecteur active/désactive la coloration syntaxique.
- **Accessibilité** : `--ax-screen-reader` ou réglage `axScreenReader` pour une sortie adaptée aux lecteurs d'écran.
- **Copier une réponse** : `/copy` (ou `/copy 2` pour l'avant-dernière ; `w` dans le sélecteur pour écrire dans un fichier).
- **Exporter** la conversation : `/export [fichier]`.
