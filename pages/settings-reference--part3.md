# Tous les réglages — partie 3/4

> Source : https://code.claude.com/docs/en/settings-reference · [← Index](../31-index-documentation-officielle.md)

> Les liens « Portée » renvoient à la section [Scopes](https://code.claude.com/docs/en/settings-reference#scopes) de la page (partie 1). « Any file » = n'importe quel fichier de réglages ; « User or managed » = réglages utilisateur ou gérés ; « Managed » = réglages gérés uniquement.

## Interface et terminal

Modifier l'apparence et le comportement de Claude Code dans votre terminal : thème, mode d'édition, ligne de statut, spinner, notifications dans la session et accessibilité. Voir [Terminal configuration](https://code.claude.com/docs/en/terminal-config).

### `askUserQuestionTimeout`

Permet à une boîte de dialogue [`AskUserQuestion`](https://code.claude.com/docs/en/tools-reference) restée sans réponse de continuer automatiquement après une période d'inactivité, en soumettant les options que vous aviez déjà sélectionnées. À utiliser quand vous vous absentez et voulez que Claude continue sans vous. Avec la valeur par défaut, les questions attendent votre réponse. Pour savoir quand le minuteur se met en pause ou ne démarre jamais, voir [Question auto-continue timeout](https://code.claude.com/docs/en/tools-reference#question-auto-continue-timeout).

* **Portée** : [`User or managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, parmi `"60s"`, `"5m"`, `"10m"` ou `"never"`
* **Défaut** : `"never"`
* **Surcharges par session** : [`CLAUDE_AFK_TIMEOUT_MS`](https://code.claude.com/docs/en/env-vars) prime sur cette clé pour une session

```json settings.json theme={null}
{
  "askUserQuestionTimeout": "5m"
}
```

Apparaît dans `/config` sous **Question auto-continue timeout**, qui écrit cette clé dans les réglages utilisateur ; Claude Code masque la ligne lorsque les réglages gérés ou le flag `--settings` définissent la clé.

### `autoContinueAtUsageLimit`

Après qu'une limite d'utilisation claude.ai a interrompu votre session, attendre dans la session ouverte et poursuivre la tâche automatiquement après la réinitialisation. Voir [Turn automatic continue off](https://code.claude.com/docs/en/interactive-mode#turn-automatic-continue-off). (v2.1.234+)

* **Portée** : [`User or managed`](https://code.claude.com/docs/en/settings-reference#scopes). Lue uniquement depuis les réglages utilisateur, `--settings` et les réglages gérés. Si aucun d'eux ne définit la clé, un fichier de réglages projet ou local qui la définit **désactive** la fonction au lieu d'être ignoré.
* **Type** : booléen
  * `true` : après qu'une limite d'utilisation claude.ai a interrompu la session, Claude Code attend dans la session ouverte et reprend la tâche automatiquement après la réinitialisation
  * `false` : Claude Code ne lance pas l'attente de lui-même. Vous pouvez toujours [lancer une attente vous-même](https://code.claude.com/docs/en/interactive-mode#start-a-wait-yourself) depuis le menu d'options de limite d'utilisation
* **Défaut** : `true`

```json settings.json theme={null}
{
  "autoContinueAtUsageLimit": false
}
```

Apparaît dans `/config` sous **Continue automatically at usage limit**, qui écrit cette clé dans les réglages utilisateur ; Claude Code masque la ligne lorsque les réglages gérés ou le flag `--settings` définissent la clé.

### `autoScrollEnabled`

Suivre la nouvelle sortie jusqu'en bas de la conversation en [rendu plein écran](https://code.claude.com/docs/en/fullscreen). Désactivez-le pour rester là où vous avez défilé pendant que Claude continue de travailler ; les demandes de permission défilent tout de même jusqu'à être visibles.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : la conversation suit la nouvelle sortie jusqu'en bas
  * `false` : vous restez là où vous avez défilé pendant que Claude travaille ; les demandes de permission apparaissent toujours sous la transcription
* **Défaut** : `true`

```json settings.json theme={null}
{
  "autoScrollEnabled": false
}
```

Apparaît dans `/config` sous **Auto-scroll** lorsque le rendu plein écran est actif ; écrit cette clé dans les réglages utilisateur.

### `axScreenReader`

Produire une sortie adaptée aux lecteurs d'écran : texte brut sans bordures décoratives ni animations. Le mode lecteur d'écran utilise le moteur de rendu classique, donc le réglage `tui` n'a aucun effet tant qu'il est actif ; les [sessions en arrière-plan](https://code.claude.com/docs/en/agent-view) attachées restent rendues en plein écran.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code affiche du texte brut sans bordures décoratives ni animations, avec le moteur de rendu classique
  * `false` : Claude Code s'affiche normalement
* **Défaut** : non défini, donc le mode lecteur d'écran est désactivé
* **Surcharges par session** : [`--ax-screen-reader`](https://code.claude.com/docs/en/cli-reference#cli-flags) prime sur [`CLAUDE_AX_SCREEN_READER`](https://code.claude.com/docs/en/env-vars), et les deux priment sur cette clé pour une session

```json settings.json theme={null}
{
  "axScreenReader": true
}
```

### `bashEditDiffEnabled`

Choisir si Claude Code enregistre quels fichiers ont changé dans un dépôt Git pendant l'exécution d'une commande Bash. Quand c'est le cas, vous voyez leur diff dans le terminal après la commande, et vos [hooks PostToolUse Bash](https://code.claude.com/docs/en/hooks#bash) reçoivent la liste des fichiers modifiés.

Un fichier listé n'est pas toujours un fichier que la commande a modifié. Une modification faite par un autre programme ou un autre appel Bash pendant l'exécution de la commande peut aussi y apparaître.

Mettez la clé à `true` pour les enregistrer dans tous les modes de permission. (v2.1.269+)

* **Portée** : [`User or managed`](https://code.claude.com/docs/en/settings-reference#scopes). Un `true` ne compte que depuis vos réglages utilisateur, le JSON passé avec `--settings` ou les [réglages gérés](https://code.claude.com/docs/en/managed-settings) : un `true` dans le `.claude/settings.json` ou le `.claude/settings.local.json` d'un dépôt ne peut pas activer l'enregistrement. Un `false` dans l'un de ces fichiers de dépôt le désactive tout de même, sauf si un fichier de [précédence supérieure](https://code.claude.com/docs/en/settings#settings-precedence) définit `true`.
* **Type** : booléen
* **Défaut** : non défini, donc Claude Code enregistre les changements en mode auto et en mode `bypassPermissions` lorsqu'il demande à Claude de modifier des fichiers via Bash
* **Surcharges par session** : [`CLAUDE_CODE_BASH_EDIT_DIFF`](https://code.claude.com/docs/en/env-vars) prime sur cette clé pour une session

```json settings.json theme={null}
{
  "bashEditDiffEnabled": true
}
```

### `companyAnnouncements`

Afficher les annonces de votre organisation aux utilisateurs au démarrage. Si vous en listez plusieurs, Claude Code en choisit une au hasard à chaque session ; lors du tout premier lancement d'une personne, il affiche la première entrée.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : tableau de chaînes
* **Défaut** : non défini, donc aucune annonce ne s'affiche

```json settings.json theme={null}
{
  "companyAnnouncements": [
    "Welcome to Acme Corp! Review our code guidelines at docs.example.com"
  ]
}
```

### `defaultShell`

Choisir si Bash ou PowerShell exécute les commandes shell que vous tapez avec le [préfixe `!`](https://code.claude.com/docs/en/interactive-mode#shell-mode-with-prefix) dans la zone de saisie, celles que Claude Code exécute directement et ajoute à la session.

`"powershell"` ne fonctionne que si l'[outil PowerShell](https://code.claude.com/docs/en/tools-reference#powershell-tool) est actif. L'outil est actif par défaut sous Windows sans Git Bash, et sous Windows avec Git Bash pour les comptes claude.ai et Console. Dans les sessions Amazon Bedrock, Google Cloud's Agent Platform et Microsoft Foundry, ainsi que sur macOS, Linux et WSL, définissez `CLAUDE_CODE_USE_POWERSHELL_TOOL=1` pour activer l'outil. Mettez cette variable à `0` pour le désactiver.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, parmi :
  * `"bash"` : Claude Code exécute vos commandes `!` dans Bash
  * `"powershell"` : Claude Code exécute vos commandes `!` dans PowerShell
* **Défaut** : `"bash"`, ou `"powershell"` sous Windows quand Bash n'est pas disponible

```json settings.json theme={null}
{
  "defaultShell": "powershell"
}
```

Si le shell nommé n'est pas disponible, Claude Code utilise l'autre : `"powershell"` se replie sur Bash lorsque l'outil PowerShell est désactivé, et `"bash"` se replie sur PowerShell lorsque Bash n'est pas installé.

### `dialogExpiry`

Définir le délai des boîtes de dialogue que Claude Code [transmet à un client distant](https://code.claude.com/docs/en/remote-control#limitations), comme un hôte Remote Control ou SDK, ainsi que de la boîte d'approbation d'un [message inter-sessions retenu](https://code.claude.com/docs/en/cross-session-messaging#control-inbound-messages). Sur Claude Code v2.1.236+, le même délai borne la demande de consentement en cours de session pour les [crédits d'utilisation Fable](https://code.claude.com/docs/en/model-config#fable-and-usage-credits) dans une session où il se peut que personne ne soit devant le terminal. Si aucune réponse n'arrive avant l'échéance, Claude Code annule la boîte de dialogue et continue avec son comportement par défaut « aucune action ». (v2.1.224+)

* **Portée** : [`User or managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, parmi `"60s"`, `"5m"`, `"10m"` ou `"never"` (qui désactive le délai)
* **Défaut** : `"5m"`
* **Surcharges par session** : [`CLAUDE_CODE_USER_DIALOG_TIMEOUT_MS`](https://code.claude.com/docs/en/env-vars) prime sur cette clé pour une session

```json settings.json theme={null}
{
  "dialogExpiry": "10m"
}
```

Les demandes de permission et les questions [`AskUserQuestion`](https://code.claude.com/docs/en/tools-reference#askuserquestion-tool-behavior) suivent leurs propres flux et ne sont pas régies par ce délai. Apparaît dans `/config` sous **Dialog expiry**, qui écrit cette clé dans les réglages utilisateur ; la ligne exige v2.1.232+, et Claude Code la masque lorsque les réglages gérés ou le flag `--settings` définissent la clé.

### `editorMode`

Choisir le mode de raccourcis clavier de la zone de saisie.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, parmi :
  * `"normal"` : raccourcis standard dans la zone de saisie
  * `"vim"` : édition façon vim avec les modes NORMAL, INSERT et VISUAL
* **Défaut** : `"normal"`

```json settings.json theme={null}
{
  "editorMode": "vim"
}
```

Apparaît dans `/config` sous **Editor mode**, qui écrit cette clé dans les réglages utilisateur.

### `emojiCompletionEnabled`

Afficher des suggestions d'emoji quand vous tapez `:` suivi d'un code court dans la zone de saisie, et remplacer un code court complet comme `:heart:` par son emoji. Mettez `false` pour désactiver les deux.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code propose des emoji après `:` et remplace un code court complet par son emoji
  * `false` : Claude Code ne suggère pas d'emoji et ne remplace pas les codes courts
* **Défaut** : `true`

```json settings.json theme={null}
{
  "emojiCompletionEnabled": false
}
```

Voir [Emoji shortcodes](https://code.claude.com/docs/en/interactive-mode#emoji-shortcodes). (v2.1.217+)

### `fileSuggestion`

Exécuter votre propre commande pour fournir l'autocomplétion des chemins de fichiers `@` à la place de la suggestion de fichiers intégrée. La suggestion intégrée utilise un parcours rapide du système de fichiers ; un gros monorepo peut obtenir de meilleurs résultats avec une indexation propre au projet, comme un index de fichiers pré-construit.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Sous les [verrous de la ligne de statut et de la suggestion de fichiers](#verrous-de-la-ligne-de-statut-et-de-la-suggestion-de-fichiers), Claude Code désactive la commande ou n'exécute qu'une valeur gérée, et ignore la vôtre sans avertissement.
* **Type** : objet avec `type`, toujours `"command"`, et `command`, la commande shell à exécuter
* **Défaut** : non défini, donc Claude Code utilise la suggestion de fichiers intégrée

```json settings.json theme={null}
{
  "fileSuggestion": {
    "type": "command",
    "command": "~/.claude/file-suggestion.sh"
  }
}
```

Après l'enregistrement, tapez `@` suivi d'une partie d'un chemin dans la saisie : les suggestions proviennent de la sortie de votre commande.

#### Entrée et sortie de la commande

Claude Code exécute la commande avec les mêmes variables d'environnement que les [hooks](https://code.claude.com/docs/en/hooks), dont `CLAUDE_PROJECT_DIR`, et cesse d'attendre au bout de cinq secondes. La commande reçoit du JSON sur stdin avec un champ `query` contenant ce que vous avez tapé jusque-là :

```json theme={null}
{"query": "src/comp"}
```

Écrivez sur stdout des chemins de fichiers séparés par des retours à la ligne. Claude Code en affiche au maximum 15 :

```text theme={null}
src/components/Button.tsx
src/components/Modal.tsx
src/components/Form.tsx
```

Le script suivant lit la requête et la transmet à un index de fichiers du dépôt :

```bash theme={null}
#!/bin/bash
query=$(cat | jq -r '.query')
# Remplacez your-repo-file-index par votre propre commande de recherche de fichiers
your-repo-file-index --query "$query" | head -20
```

### `footerLinksRegexes`

Afficher des badges cliquables supplémentaires dans le pied de page sous la zone de saisie lorsqu'une regex correspond à la sortie d'un tour : résultats d'outils (y compris contenus de fichiers et pages récupérées) et réponses de Claude lui-même. Sert à transformer les identifiants affichés par les CLI du projet, comme les outils de revue et les gestionnaires de tickets, en liens dans la session.

* **Portée** : [`User or managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : tableau d'objets, chacun avec `type` à `"regex"`, une regex `pattern`, un modèle `url` et un `label` facultatif ; les espaces réservés `{name}` dans `url` et `label` sont remplis à partir des groupes de capture nommés de `pattern`
* **Défaut** : non défini, donc aucun badge ne s'affiche

Cet exemple reconnaît les clés de ticket comme `PROJ-1234` et construit chaque lien à partir de la clé capturée :

```json settings.json theme={null}
{
  "footerLinksRegexes": [
    {
      "type": "regex",
      "pattern": "\\b(?<key>PROJ-\\d+)\\b",
      "url": "https://issues.example.com/browse/{key}",
      "label": "{key}"
    }
  ]
}
```

Avec cette configuration, quand `PROJ-1234` apparaît dans un résultat d'outil ou dans la réponse de Claude, un badge `PROJ-1234` apparaît dans le pied de page avec un lien vers `https://issues.example.com/browse/PROJ-1234`.

#### Contraintes des badges

L'URL, le libellé et le nombre de badges de chaque entrée sont bornés comme suit :

| Contrainte | Comportement |
| :- | :- |
| Origine de l'URL | Les valeurs capturées sont encodées pour l'URL et l'URL construite doit partager l'origine littérale du modèle. Une capture peut remplir un segment de chemin ou une valeur de requête mais ne peut pas changer la destination du lien |
| Longueur de l'URL | Les URL construites de plus de 2048 caractères sont abandonnées |
| Schéma de l'URL | Doit être `https`, `http` ou un schéma de lien profond d'éditeur ou d'espace de travail reconnu : `vscode`, `vscode-insiders`, `cursor`, `windsurf`, `zed`, `jetbrains`, `idea`, `slack`, `linear`, `notion`, `figma` |
| Libellé | Par défaut le texte reconnu, tronqué à 28 colonnes d'affichage |
| Nombre de badges | Au plus 5 badges s'affichent. Le plus ancien est remplacé par les nouvelles correspondances et `/clear` les supprime |

Quand un tour se termine, Claude Code applique la regex `pattern` de chaque entrée à la sortie du tour sur le thread principal : une regex lente bloque donc l'interface jusqu'à ce qu'elle finisse.

> **Avertissement** : des quantificateurs imbriqués comme `(a+)+$` peuvent prendre un temps exponentiel sur certaines entrées et figer la session ; gardez chaque `pattern` linéaire et évitez d'imbriquer `+` ou `*`.

Les badges du pied de page s'affichent à côté d'une [ligne de statut personnalisée](https://code.claude.com/docs/en/statusline) si elle est configurée ; aucun ne remplace l'autre. Utilisez une ligne de statut pour une rangée pilotée par script qui calcule son contenu à partir des données de session, et les badges pour transformer les identifiants de la conversation en liens sans script.

### `keybindingFlavor`

> **Avertissement** : obsolète depuis v2.1.261 et sans effet. Les touches d'édition de mots de la saisie [suivent toujours les conventions readline](https://code.claude.com/docs/en/interactive-mode#make-ctrl-w-delete-back-to-whitespace), comme dans Bash. Claude Code accepte encore `keybindingFlavor`, de sorte qu'un fichier de réglages qui le définit reste valide.

De v2.1.238 à v2.1.260, la valeur `"readline"` faisait supprimer à `Ctrl+W` jusqu'à l'espace précédent au lieu du seul mot précédent.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, `"classic"` ou `"readline"`
* **Défaut** : non défini

### `maxProseWidth`

Limiter la largeur de la prose des réponses de Claude pour que les lignes restent lisibles dans un terminal large. Paragraphes, titres, listes et citations sont repliés dans ce nombre de colonnes, tandis que tableaux et blocs de code gardent toute la largeur du terminal. (v2.1.282+)

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : nombre de colonnes du terminal, entier, minimum `40`. Claude Code ignore toute autre valeur
* **Défaut** : non défini, donc la prose se replie au bord du terminal

```json settings.json theme={null}
{
  "maxProseWidth": 80
}
```

### `prefersReducedMotion`

Réduire ou désactiver les animations de l'interface comme le spinner, l'effet de scintillement (shimmer) et les flashs. Apparaît dans `/config` sous **Reduce motion**.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code réduit ou désactive les animations (spinner, shimmer, flashs)
  * `false` : identique à non défini ; Claude Code affiche ses animations
* **Défaut** : `false`

```json settings.json theme={null}
{
  "prefersReducedMotion": true
}
```

### `promptSuggestionEnabled`

Afficher ou masquer les [suggestions de prompt](https://code.claude.com/docs/en/interactive-mode#prompt-suggestions), les prédictions grisées qui apparaissent dans votre zone de saisie. Mettez `false`, ou désactivez **Prompt suggestions** dans `/config`, pour les masquer.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : vous voyez les suggestions de prompt dans la zone de saisie
  * `false` : Claude Code masque les suggestions de prompt
* **Défaut** : `true`
* **Surcharges par session** : [`CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION`](https://code.claude.com/docs/en/env-vars) prime sur cette clé pour une session

```json settings.json theme={null}
{
  "promptSuggestionEnabled": false
}
```

Les suggestions de prompt exigent un compte claude.ai ou Console avec la télémétrie activée. Sur Amazon Bedrock, Google Cloud's Agent Platform et Microsoft Foundry, ou avec la télémétrie désactivée (par exemple via [`DISABLE_TELEMETRY`](https://code.claude.com/docs/en/env-vars)), cette clé est sans effet et seul `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION=1` les active.

### `respectGitignore`

Contrôler si le sélecteur de fichiers `@` exclut les fichiers correspondant aux motifs `.gitignore`. Apparaît dans `/config` sous **Respect .gitignore in file picker**.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Si aucun fichier de réglages ne le définit, Claude Code se rabat sur `respectGitignore` dans `~/.claude.json`, que la bascule de `/config` écrit.
* **Type** : booléen
  * `true` : le sélecteur `@` exclut les fichiers correspondant aux motifs `.gitignore`
  * `false` : le sélecteur `@` inclut les fichiers correspondant aux motifs `.gitignore`
* **Défaut** : `true`

```json settings.json theme={null}
{
  "respectGitignore": false
}
```

### `respondToBashCommands`

Choisir si Claude répond après que vous avez exécuté une commande shell avec le [préfixe `!`](https://code.claude.com/docs/en/interactive-mode#shell-mode-with-prefix) dans la zone de saisie. Par défaut, Claude Code ajoute la sortie de la commande à la conversation et Claude y répond. Mettez cette clé à `false` pour ajouter la sortie au contexte sans réponse, afin de lancer plusieurs commandes et de poser ensuite une question sur l'ensemble.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code ajoute la sortie à la conversation et Claude y répond
  * `false` : Claude Code ajoute la sortie au contexte sans réponse
* **Défaut** : `true`

```json settings.json theme={null}
{
  "respondToBashCommands": false
}
```

Voir [Shell mode with `!` prefix](https://code.claude.com/docs/en/interactive-mode#shell-mode-with-prefix).

### `showClearContextOnPlanAccept`

Quand Claude termine un plan en [mode plan](https://code.claude.com/docs/en/permission-modes#review-and-approve-a-plan), il affiche un menu d'approbation. La planification pouvant consommer beaucoup de contexte, cette clé ajoute à ce menu une première option, **Yes, clear context and …**, qui approuve le plan, efface le contexte de la conversation et commence l'implémentation à partir du seul plan. La suite du libellé nomme le mode de permission dans lequel la session continue et indique la part de votre contexte consommée par la planification.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : le menu d'approbation du plan reçoit une première option, **Yes, clear context and …**, qui approuve le plan et efface le contexte de la conversation
  * `false` : le menu d'approbation n'affiche aucune option d'effacement du contexte
* **Défaut** : `false`

```json settings.json theme={null}
{
  "showClearContextOnPlanAccept": true
}
```

### `showTurnDuration`

Afficher ou masquer le message de durée du tour après chaque réponse, par exemple « Cooked for 1m 6s · done 6:05 PM ». L'heure après « done » indique la fin du tour ; [`timeFormat`](#timeformat) et [`timeZone`](#timezone) contrôlent son format et son fuseau. Apparaît dans `/config` sous **Show turn duration**.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Une valeur dans `~/.claude.json` provenant d'une ancienne version s'applique si aucun fichier de réglages ne la définit.
* **Type** : booléen
  * `true` : vous voyez le message de durée du tour après chaque réponse
  * `false` : Claude Code masque le message de durée du tour
* **Défaut** : `true`

```json settings.json theme={null}
{
  "showTurnDuration": false
}
```

### `spellcheck`

Souligner les mots mal orthographiés dans la zone de saisie pendant la frappe, à l'aide d'un correcteur orthographique que vous installez. Claude Code ne vérifie que le texte de la zone de saisie. [Check spelling as you type](https://code.claude.com/docs/en/interactive-mode#check-spelling-as-you-type) explique comment installer aspell, hunspell ou ispell et ce que couvre le correcteur. (v2.1.235+)

* **Portée** : [`User or managed`](https://code.claude.com/docs/en/settings-reference#scopes). Le bloc du niveau le plus élevé qui le définit s'applique en entier.
* **Type** : objet avec `enabled` (booléen), `checker` (`"aspell"`, `"hunspell"`, `"ispell"` ou `"auto"`), `language` (chaîne, transmise au correcteur comme nom de dictionnaire) et `color` (chaîne : nom de couleur de terminal, `#rrggbb`, `rgb(r,g,b)`, `ansi256(n)` ou `ansi:<name>`)
* **Défaut** : non défini, donc la correction orthographique est désactivée ; `checker` vaut par défaut `"auto"`, le premier des trois trouvé dans le `PATH` ; `language` vaut par défaut le dictionnaire propre du correcteur ; `color` vaut par défaut la couleur d'erreur du thème

```json settings.json theme={null}
{
  "spellcheck": { "enabled": true, "language": "en_GB" }
}
```

### `spinnerTipsEnabled`

Pendant que Claude travaille, la ligne du spinner fait défiler de courtes astuces sur les fonctionnalités de Claude Code, comme « Use Plan Mode to prepare for a complex request before making changes. Press Shift+Tab twice to enable. ». Mettez cette clé à `false` pour les masquer. Apparaît dans `/config` sous **Show tips**.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : vous voyez des astuces dans le spinner pendant que Claude travaille
  * `false` : Claude Code masque les astuces du spinner
* **Défaut** : `true`

```json settings.json theme={null}
{
  "spinnerTipsEnabled": false
}
```

### `spinnerTipsOverride`

Ajouter vos propres astuces aux [astuces du spinner](#spinnertipsenabled) affichées par Claude Code pendant que Claude travaille, ou remplacer les astuces intégrées par les vôtres. Claude Code place vos astuces dans la même rotation que les astuces intégrées.

Si vous mettez [`spinnerTipsEnabled`](#spinnertipsenabled) à `false`, Claude Code masque toutes les astuces, y compris les vôtres.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code respecte les objets astuce, `tipsFile`, `label` et `excludeDefault` depuis les réglages utilisateur, le flag `--settings` et les réglages gérés ; depuis les réglages projet et locaux, il ne lit que les astuces sous forme de chaînes simples.
* **Type** : objet avec les champs `tips`, `tipsFile`, `label` et `excludeDefault`, tous facultatifs
* **Défaut** : non défini, donc Claude Code n'affiche que les astuces intégrées

Les objets astuce, `tipsFile`, `label` et la règle de la Portée selon laquelle les réglages projet et locaux ne fournissent que des chaînes simples exigent v2.1.247+.

Chaque entrée de `tips` est une chaîne simple ou un objet avec ces champs :

| Champ | Requis | Description |
| :- | :- | :- |
| `id` | Oui | Jusqu'à 64 lettres, chiffres, `.`, `_` ou `-`. Claude Code y rattache l'historique d'affichage de l'astuce, de sorte que son délai de réapparition survit à un réordonnancement de la liste. Entre deux entrées de même id, Claude Code utilise la première |
| `text` | Oui | L'astuce, une ligne de 500 caractères au plus. Claude Code retire les séquences d'échappement ANSI et les caractères de contrôle et réduit les espaces |
| `cooldownSessions` | Non | Nombre de sessions que Claude Code attend avant de réafficher l'astuce, de `0` à `1000`, défaut `0` |
| `priority` | Non | Ordre parmi les astuces non affichées depuis aussi longtemps, la plus haute d'abord, de `-10` à `10`, défaut `0` |

Claude Code lit une chaîne simple comme une astuce avec ces valeurs par défaut et un id fondé sur sa position, donc son historique d'affichage se réinitialise si vous réordonnez la liste. Donnez un `id` à une astuce pour conserver son historique au fil des modifications.

Claude Code lit au plus 200 astuces au total entre `tips` et `tipsFile`, et écarte une entrée invalide avec un avertissement de débogage au lieu de rejeter le fichier de réglages.

Les autres champs permettent de nommer un fichier d'astuces, de définir le préfixe et de masquer les astuces intégrées :

* `tipsFile` : un chemin absolu ou en `~/` vers un fichier JSON local contenant un tableau des mêmes entrées, ou un objet avec un tableau `tips`, jusqu'à 256 Ko. Claude Code lit le fichier une fois par processus, donc vos modifications sont prises en compte au démarrage suivant. Il ne peut pas être défini via les [réglages gérés côté serveur](https://code.claude.com/docs/en/server-managed-settings) ; déployez-y des `tips` en ligne, ou déployez le chemin dans un `managed-settings.json` sur disque.
* `label` : le préfixe affiché par Claude Code avant les astuces provenant des réglages utilisateur, de `--settings` et des réglages gérés, jusqu'à 40 caractères. Par défaut `Tip`, le même préfixe que les astuces intégrées ; les astuces des réglages projet et locaux l'utilisent toujours.
* `excludeDefault` : mettez `true` pour masquer les astuces intégrées et n'afficher que les vôtres. Si Claude Code ne parvient à charger aucune de vos astuces, par exemple parce que `tipsFile` n'existe pas ou que toutes les entrées sont invalides, il conserve la rotation intégrée plutôt qu'un spinner vide.

Lorsque plusieurs fichiers de réglages définissent la clé, Claude Code affiche les astuces de tous et prend `tipsFile`, `label` et `excludeDefault` dans celui des réglages gérés, du flag `--settings` et des réglages utilisateur qui a la plus haute précédence parmi ceux qui définissent chaque champ.

Cet exemple, dans vos réglages utilisateur, ajoute une astuce chaîne simple et une astuce objet à la rotation, sous le préfixe `Acme tip` :

```json settings.json theme={null}
{
  "spinnerTipsOverride": {
    "label": "Acme tip",
    "tips": [
      "Run /review before opening a PR",
      {
        "id": "gateway-errors",
        "text": "Seeing 5xx errors? Check the gateway status page first",
        "cooldownSessions": 5,
        "priority": 2
      }
    ]
  }
}
```

Chaque champ de l'exemple modifie un aspect de l'affichage des astuces :

* `label` : Claude Code affiche les deux astuces sous la forme `Acme tip: ...` au lieu de `Tip: ...`.
* La chaîne simple : Claude Code lui applique les valeurs par défaut, elle peut donc réapparaître dès la session suivante.
* `id` : Claude Code rattache l'historique d'affichage de la seconde astuce à `gateway-errors`, donc son délai s'applique encore après l'ajout ou le réordonnancement d'astuces.
* `cooldownSessions` : après avoir affiché l'astuce `gateway-errors`, Claude Code ne la réaffiche pas avant cinq sessions.
* `priority` : quand l'astuce `gateway-errors` et une autre n'ont pas été affichées depuis le même nombre de sessions (par exemple quand aucune ne l'a encore été), Claude Code affiche `gateway-errors` en premier. La chaîne simple a la priorité par défaut, `0`.

Pendant que Claude travaille, Claude Code affiche vos astuces dans le spinner avec votre préfixe, par exemple `Acme tip: Run /review before opening a PR`.

### `spinnerVerbs`

Pendant qu'un tour est en cours, le spinner affiche un verbe tournant comme « Accomplishing », « Architecting » ou « Baking ». Cette clé permet d'ajouter vos propres verbes à cette rotation ou de remplacer la liste intégrée par la vôtre.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : objet avec un tableau de chaînes `verbs` et `mode`, parmi :
  * `"append"` : Claude Code ajoute vos verbes à l'ensemble intégré
  * `"replace"` : Claude Code n'affiche que vos verbes
* **Défaut** : non défini, donc Claude Code utilise les verbes intégrés

Cet exemple ajoute deux verbes à l'ensemble intégré :

```json settings.json theme={null}
{
  "spinnerVerbs": {
    "mode": "append",
    "verbs": ["Pondering", "Crafting"]
  }
}
```

En mode `"replace"` avec un tableau `verbs` vide, Claude Code conserve les verbes intégrés.

### `statusLine`

Exécuter votre propre commande pour afficher une [ligne de statut](https://code.claude.com/docs/en/statusline) sous la saisie, avec un contexte comme le modèle, le coût ou la branche git. Des champs facultatifs ajustent l'espacement, ajoutent des réexécutions périodiques et masquent l'indicateur de mode vim intégré lorsque votre script affiche lui-même `vim.mode`.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Quand [`allowManagedHooksOnly`](#allowmanagedhooksonly) est actif, ou que [`disableAllHooks`](#disableallhooks) est défini hors des réglages gérés, seule la valeur des réglages gérés s'exécute.
* **Type** : objet avec `type` à `"command"` et une chaîne `command`, plus `padding` facultatif (nombre de caractères), `refreshInterval` (nombre de secondes, minimum `1`) et `hideVimModeIndicator` (booléen)
* **Défaut** : non défini, donc pas de ligne de statut

Cet exemple affiche le nom du modèle et l'utilisation du contexte, et ajoute deux caractères d'espacement horizontal :

```json settings.json theme={null}
{
  "statusLine": {
    "type": "command",
    "command": "jq -r '\"[\\(.model.display_name)] \\(.context_window.used_percentage // 0)% context\"'",
    "padding": 2
  }
}
```

L'exemple nécessite [`jq`](https://jqlang.org/) installé et s'exécute dans un shell. Pour les équivalents PowerShell et Git Bash, voir [Windows configuration](https://code.claude.com/docs/en/statusline#windows-configuration) ; pour la configuration complète, voir [Manually configure a status line](https://code.claude.com/docs/en/statusline#manually-configure-a-status-line).

### `subagentStatusLine`

Quand Claude exécute des [sous-agents](https://code.claude.com/docs/en/sub-agents), Claude Code les liste dans un affichage de tâches sous la saisie, une ligne par sous-agent au format `name · description · token count`. Cette clé permet d'exécuter votre propre commande pour réécrire ces lignes, par exemple pour afficher l'utilisation du contexte de chaque sous-agent en pourcentage. À chaque rafraîchissement, Claude Code envoie les lignes visibles sous forme d'un objet JSON sur stdin, avec un tableau `tasks` portant pour chaque sous-agent `id`, `name`, `status`, `model`, `tokenCount`, etc., et remplace la ligne de chaque `id` que vous renvoyez sous forme de ligne `{"id", "content"}`. Les lignes que vous ne renvoyez pas gardent le rendu par défaut.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Quand [`allowManagedHooksOnly`](#allowmanagedhooksonly) est actif, ou que [`disableAllHooks`](#disableallhooks) est défini hors des réglages gérés, seule la valeur des réglages gérés s'exécute.
* **Type** : objet avec `type` à `"command"` et une chaîne `command`
* **Défaut** : non défini, donc Claude Code affiche les lignes par défaut

```json settings.json theme={null}
{
  "subagentStatusLine": {
    "type": "command",
    "command": "jq -c '.tasks[] | {id, content: \"\\(.name): \\(.tokenCount) tokens\"}'"
  }
}
```

Voir [Subagent status lines](https://code.claude.com/docs/en/statusline#subagent-status-lines).

### `syntaxHighlightingDisabled`

Claude Code colore le code selon le langage dans les diffs, blocs de code et aperçus de fichiers qu'il affiche dans le terminal, avec son surligneur intégré ; aucun plugin ni serveur de langage n'intervient. Mettez cette clé à `true` pour les afficher en texte brut, par exemple si les couleurs jurent avec le thème de votre terminal ou ralentissent un lecteur d'écran.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code désactive la coloration syntaxique dans les diffs, blocs de code et aperçus de fichiers
  * `false` : Claude Code colore la syntaxe
* **Défaut** : `false`

```json settings.json theme={null}
{
  "syntaxHighlightingDisabled": true
}
```

### `terminalProgressBarEnabled`

Certains terminaux peuvent afficher un indicateur de progression sur l'onglet ou dans la barre des tâches pour le programme qui y tourne. Pendant que Claude travaille, Claude Code signale un état « en cours » au terminal, pour que vous voyiez depuis un autre onglet ou une autre fenêtre si la session est encore occupée. L'indicateur reste visible après la fin du tour tant que des [sous-agents en arrière-plan](https://code.claude.com/docs/en/sub-agents#run-subagents-in-foreground-or-background) ou des [workflows dynamiques](https://code.claude.com/docs/en/workflows) tournent encore, et disparaît une fois la session inactive.

Claude Code ne le signale que dans les terminaux qui prennent en charge l'indicateur : ConEmu, Ghostty 1.2.0+ et iTerm2 3.6.6+. Mettez cette clé à `false` pour que Claude Code cesse de le signaler. Apparaît dans `/config` sous **Terminal progress bar**.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Une valeur dans `~/.claude.json` provenant d'une ancienne version s'applique si aucun fichier de réglages ne la définit.
* **Type** : booléen
  * `true` : vous voyez la barre de progression du terminal dans les terminaux compatibles
  * `false` : Claude Code masque la barre de progression du terminal
* **Défaut** : `true`

```json settings.json theme={null}
{
  "terminalProgressBarEnabled": false
}
```

### `terminalTitleFromRename`

Claude Code définit le titre de l'onglet de votre terminal. Par défaut, il utilise un titre qu'il génère à partir de la conversation, et dès que vous donnez un [nom](https://code.claude.com/docs/en/sessions#name-your-sessions) à la session avec `/rename` ou `--name`, l'onglet affiche ce nom à la place. Mettez cette clé à `false` pour garder le titre généré sur l'onglet même après avoir nommé la session. Le nom lui-même s'applique quand même, donc `/resume <name>` et le sélecteur de sessions le retrouvent.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : le titre de l'onglet affiche le nom de session que vous avez défini
  * `false` : l'onglet garde le titre que Claude Code génère à partir de votre conversation
* **Défaut** : `true`

```json settings.json theme={null}
{
  "terminalTitleFromRename": false
}
```

Pour que Claude Code ne mette plus du tout à jour le titre du terminal, définissez plutôt [`CLAUDE_CODE_DISABLE_TERMINAL_TITLE`](https://code.claude.com/docs/en/env-vars) à `1`.

### `theme`

Choisir le thème de couleurs de l'interface. Apparaît dans `/config` sous **Theme**.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Une valeur dans `~/.claude.json` provenant d'une ancienne version s'applique si aucun fichier de réglages ne la définit.
* **Type** : chaîne, parmi :
  * `"auto"` : s'aligne sur le fond clair ou sombre de votre terminal
  * `"dark"` : le thème sombre
  * `"light"` : le thème clair
  * `"dark-daltonized"` : le thème sombre avec des couleurs adaptées au daltonisme
  * `"light-daltonized"` : le thème clair avec des couleurs adaptées au daltonisme
  * `"dark-ansi"` : le thème sombre n'utilisant que la palette ANSI de votre terminal
  * `"light-ansi"` : le thème clair n'utilisant que la palette ANSI de votre terminal
  * `"custom:<slug>"` ou `"custom:<plugin-name>:<slug>"` : un thème personnalisé de `~/.claude/themes/` ou d'un plugin
* **Défaut** : `"dark"`

```json settings.json theme={null}
{
  "theme": "light-daltonized"
}
```

Voir [Create a custom theme](https://code.claude.com/docs/en/terminal-config#create-a-custom-theme).

### `timeFormat`

Choisir comment Claude Code écrit les heures qu'il affiche dans l'interface, comme le `done 6:05 PM` à la fin de chaque message de durée de tour et les horodatages du [visualiseur de transcription](https://code.claude.com/docs/en/interactive-mode#transcript-viewer). Pour choisir un préréglage, lancez `/config` et réglez **Time format**. (v2.1.257+)

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, parmi :
  * `"auto"` : identique à non défini ; chaque heure garde son format intégré, qui suit votre locale dans le message de durée du tour
  * `"12-hour"` : horloge sur 12 heures
  * `"24-hour"` : horloge sur 24 heures
  * `"24-hour-utc"` : horloge sur 24 heures en UTC avec `Z` après les minutes, par exemple `18:05Z` ; Claude Code ignore [`timeZone`](#timezone) pour ce préréglage
  * Un motif strftime comme `"%H:%M"` : Claude Code écrit chaque heure avec ce motif. Toute valeur contenant un `%` est un motif, et toute autre valeur hors préréglages compte comme `"auto"`
* **Défaut** : `"auto"`

```json settings.json theme={null}
{
  "timeFormat": "24-hour"
}
```

`/config` ne propose que les préréglages ; pour utiliser un motif strftime, ajoutez la clé dans un fichier de réglages. Cet exemple affiche chaque heure sur 24 heures à deux chiffres :

```json settings.json theme={null}
{
  "timeFormat": "%H:%M"
}
```

Le message de durée du tour et le visualiseur de transcription affichent alors des heures comme `18:05`. Dans le visualiseur de transcription, le motif constitue tout l'horodatage : ajoutez des directives de date si vous voulez y voir la date. Cet exemple place la date devant l'heure :

```json settings.json theme={null}
{
  "timeFormat": "%Y-%m-%d %H:%M"
}
```

Les mêmes surfaces affichent alors des heures comme `2026-09-01 18:05`.

### `timeZone`

Afficher les heures de l'interface dans un autre fuseau horaire que celui du système. Indiquez un [nom de fuseau IANA](https://www.iana.org/time-zones), comme `"UTC"` ou `"Europe/Dublin"`. Les heures contrôlées par [`timeFormat`](#timeformat) s'affichent alors dans ce fuseau. Si `timeFormat` vaut `"24-hour-utc"`, les heures restent en UTC et Claude Code ignore cette clé. `/config` n'a pas de ligne pour cette clé : définissez-la dans un fichier de réglages. (v2.1.257+)

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, un nom de fuseau IANA. Si Claude Code ne reconnaît pas le nom, il utilise le fuseau du système
* **Défaut** : non défini, donc les heures s'affichent dans le fuseau du système

```json settings.json theme={null}
{
  "timeZone": "Europe/Dublin"
}
```

### `tui`

Choisir le moteur de rendu de l'interface terminal. Utilisez `"fullscreen"` pour le [moteur alt-screen](https://code.claude.com/docs/en/fullscreen) sans scintillement avec historique de défilement virtualisé, ou `"default"` pour le moteur classique sur l'écran principal. Lancer `/tui fullscreen` ou `/tui default` écrit cette clé pour vous.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, parmi :
  * `"default"` : le moteur classique sur l'écran principal
  * `"fullscreen"` : le moteur alt-screen sans scintillement avec défilement virtualisé
* **Défaut** : non défini, donc Claude Code [choisit le moteur pour vous](https://code.claude.com/docs/en/fullscreen#fullscreen-by-default)
* **Surcharges par session** : [`CLAUDE_CODE_NO_FLICKER`](https://code.claude.com/docs/en/env-vars) et [`CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN`](https://code.claude.com/docs/en/env-vars) priment sur cette clé pour une session : `CLAUDE_CODE_NO_FLICKER=1` active le plein écran, et `CLAUDE_CODE_NO_FLICKER=0` ou `CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN=1` le désactive ; si les deux sont définies, Claude Code le désactive

```json settings.json theme={null}
{
  "tui": "fullscreen"
}
```

Sous tmux `-CC` ou en SSH vers Windows, Claude Code garde le moteur classique sauf si vous définissez `CLAUDE_CODE_NO_FLICKER=1`. Les sessions en arrière-plan ouvertes depuis l'[agent view](https://code.claude.com/docs/en/agent-view) utilisent toujours le moteur plein écran, quel que soit ce réglage.

### `verbose`

Par défaut, la transcription réduit chaque appel d'outil à un court résumé, comme la commande lancée par Claude et le nombre de lignes de sa sortie, et vous appuyez sur `Ctrl+O` pour basculer toute la transcription en vue détaillée quand vous voulez les détails. Mettez cette clé à `true` pour afficher en ligne l'entrée et la sortie complètes de chaque appel d'outil au fil de l'eau, utile pour déboguer un hook, un serveur MCP ou une longue commande shell. Apparaît dans `/config` sous **Verbose output**.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Une valeur dans `~/.claude.json` provenant d'une ancienne version s'applique si aucun fichier de réglages ne la définit.
* **Type** : booléen
  * `true` : vous voyez la sortie complète des outils
  * `false` : vous voyez des résumés tronqués de la sortie des outils
* **Défaut** : `false`
* **Surcharges par session** : [`--verbose`](https://code.claude.com/docs/en/cli-reference#cli-flags) prime sur cette clé pour une session

```json settings.json theme={null}
{
  "verbose": true
}
```

Une valeur [`viewMode`](#viewmode) ou une sélection `/focus` persistante remplace cette clé à chaque session.

### `viewMode`

Définir la vue de transcription dans laquelle Claude Code démarre : `"default"`, `"verbose"` ou `"focus"`. Lorsqu'elle est définie, elle prime à la fois sur la sélection `/focus` persistante et sur le réglage [`verbose`](#verbose).

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, parmi :
  * `"default"` : la transcription normale avec sortie d'outils tronquée
  * `"verbose"` : la transcription avec sortie d'outils complète
  * `"focus"` : uniquement votre dernier prompt, un résumé d'une ligne des appels d'outils avec les statistiques de diff des modifications, et la réponse finale. La vue focus nécessite le [moteur plein écran](#tui)
* **Défaut** : non défini, donc le réglage `verbose` et votre dernier choix `/focus` s'appliquent
* **Surcharges par session** : [`--verbose`](https://code.claude.com/docs/en/cli-reference#cli-flags) prime sur cette clé pour une session

```json settings.json theme={null}
{
  "viewMode": "focus"
}
```

### `vimInsertModeRemaps`

Associer des séquences de deux touches en mode INSERT à Échap dans le [mode d'édition vim](https://code.claude.com/docs/en/interactive-mode#vim-editor-mode). Chaque clé est exactement deux caractères imprimables tapés à la suite, et `"<Esc>"` est la seule cible prise en charge ; Claude Code ignore les autres entrées. (v2.1.208+)

* **Portée** : [`User or managed`](https://code.claude.com/docs/en/settings-reference#scopes). Un dépôt ne peut pas remapper vos frappes.
* **Type** : objet associant une séquence de deux caractères à `"<Esc>"`
* **Défaut** : non défini

```json settings.json theme={null}
{
  "vimInsertModeRemaps": {
    "jj": "<Esc>"
  }
}
```

Sans effet si `editorMode` ne vaut pas `"vim"`. Voir [Remap INSERT-mode key sequences](https://code.claude.com/docs/en/interactive-mode#remap-insert-mode-key-sequences). (v2.1.208+)

### `voice`

Activer la [dictée vocale](https://code.claude.com/docs/en/voice-dictation) et choisir le comportement de la touche de dictée. Claude Code écrit cet objet pour vous quand vous lancez `/voice`.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : objet avec `enabled` (booléen), `autoSubmit` (booléen, ne s'applique qu'en mode maintien) et `mode`, parmi :
  * `"hold"` : vous maintenez la touche de dictée pendant que vous parlez et la relâchez pour arrêter
  * `"tap"` : vous appuyez une fois sur la touche pour démarrer l'enregistrement et une seconde fois pour envoyer
* **Défaut** : non défini, donc la dictée est désactivée ; quand `enabled` vaut `true` et que `mode` n'est pas défini, Claude Code utilise `"hold"`

Cet exemple active la dictée et configure la touche pour un appui de démarrage puis un appui d'envoi :

```json settings.json theme={null}
{
  "voice": {
    "enabled": true,
    "mode": "tap"
  }
}
```

`autoSubmit` envoie le prompt quand vous relâchez la touche en mode maintien. La dictée vocale exige un compte claude.ai.

### `voiceEnabled`

> **Avertissement** : obsolète depuis v2.1.92, quand l'objet [`voice`](#voice) l'a remplacé. Claude Code le lit encore pour que les anciens fichiers de réglages continuent de fonctionner, mais les nouvelles configurations doivent définir `voice.enabled`.

Active la dictée vocale avec la forme booléenne unique antérieure à l'objet `voice`. Si les deux sont définis, `voice.enabled` s'applique.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : la dictée vocale est active lorsque vous êtes connecté avec un compte claude.ai et que la politique de votre organisation autorise la voix, sauf si `voice.enabled` est défini
  * `false` : la dictée vocale est désactivée, sauf si `voice.enabled` est défini
* **Défaut** : non défini

```json settings.json theme={null}
{
  "voiceEnabled": true
}
```

### `wheelScrollAccelerationEnabled`

Accélérer la vitesse de défilement à la molette lors des défilements rapides en [rendu plein écran](https://code.claude.com/docs/en/fullscreen#mouse-wheel-scrolling). Mettez `false` pour un pas de défilement constant par cran de molette.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code accélère le défilement à la molette lors des défilements rapides
  * `false` : Claude Code défile à vitesse constante par cran de molette
* **Défaut** : `true`

```json settings.json theme={null}
{
  "wheelScrollAccelerationEnabled": false
}
```

## Git et attribution

Contrôler l'attribution que Claude Code ajoute aux commits et aux pull requests, et sa façon de travailler avec git.

### `attribution`

Personnaliser l'attribution que Claude Code ajoute aux commits git et aux pull requests. Par défaut, les commits reçoivent un [trailer git](https://git-scm.com/docs/git-interpret-trailers) comme `Co-Authored-By` ; les descriptions de pull request reçoivent du texte brut. Réglez chaque partie séparément avec les sous-clés ci-dessous.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : objet avec les chaînes `commit` et `pr` et un booléen `sessionUrl`, ou `false` pour masquer toute attribution. La valeur `false` exige v2.1.281+ ; les versions antérieures la rejettent et [ignorent tout le fichier de réglages utilisateur, projet ou local](https://code.claude.com/docs/en/settings#fix-a-broken-settings-file) qui la contient
* **Défaut** : non défini, donc Claude Code utilise l'attribution standard indiquée sous chaque sous-clé

Pour masquer toute attribution, mettez `attribution` à `false`. Dans un fichier de réglages également lu par des versions antérieures, mettez plutôt [`commit`](#attributioncommit) et [`pr`](#attributionpr) à des chaînes vides et [`sessionUrl`](#attributionsessionurl) à `false`.

Cet exemple remplace l'attribution des commits, supprime celle des pull requests et retire le lien de session :

```json settings.json theme={null}
{
  "attribution": {
    "commit": "Generated with AI\n\nCo-Authored-By: AI <ai@example.com>",
    "pr": "",
    "sessionUrl": false
  }
}
```

Dès que vous définissez `commit` ou `pr`, Claude Code ignore le réglage obsolète `includeCoAuthoredBy` et utilise son texte par défaut pour celui des deux que vous n'avez pas défini.

Claude Code indique à Claude que vos propres instructions sur l'attribution, comme une règle de CLAUDE.md ou de [mémoire](https://code.claude.com/docs/en/memory), priment sur ces lignes de commit et de PR, sauf si la ligne est définie dans les [réglages gérés](https://code.claude.com/docs/en/managed-settings).

### `includeCoAuthoredBy`

> **Avertissement** : obsolète depuis v2.0.62, quand [`attribution`](#attribution) l'a remplacé. Claude Code le lit encore, mais les nouvelles configurations doivent définir `attribution`.

Utilisez plutôt [`attribution`](#attribution), qui remplace cette clé et permet de modifier ou masquer séparément le trailer de commit, le texte de pull request et le lien de session. Claude Code respecte encore `includeCoAuthoredBy: false` dans les fichiers de réglages antérieurs à `attribution`, mais l'ignore dès que vous définissez `attribution.commit` ou `attribution.pr`.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : identique à non défini ; Claude Code ajoute le trailer de commit et le texte d'attribution de pull request
  * `false` : Claude Code omet à la fois le trailer de commit et le texte d'attribution de pull request, sauf si `attribution` définit `commit` ou `pr`, auquel cas les règles d'[`attribution`](#attribution) s'appliquent
* **Défaut** : `true`

```json settings.json theme={null}
{
  "includeCoAuthoredBy": false
}
```

Pour masquer toute attribution, voir [`attribution`](#attribution).

### `includeGitInstructions`

Claude Code fournit à Claude deux éléments de contexte liés à git : ses instructions intégrées sur la rédaction des commits et des pull requests, dans la description de l'outil Bash, et un instantané du statut git de votre dépôt. L'instantané contient la branche courante, la branche principale, la sortie de `git status` et les commits récents. Claude Code le lit au début d'une conversation.

Mettez cette clé à `false` pour omettre les deux, par exemple quand vous utilisez vos propres skills de workflow git.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code inclut ses instructions intégrées de workflow de commit et de pull request ainsi que l'instantané du statut git. Les sessions cloud n'incluent jamais l'instantané
  * `false` : Claude Code omet les deux
* **Défaut** : `true`
* **Surcharges par session** : [`CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS`](https://code.claude.com/docs/en/env-vars) prime sur cette clé pour une session

```json settings.json theme={null}
{
  "includeGitInstructions": false
}
```

### `prUrlTemplate`

Faire pointer les liens de PR que Claude Code affiche, dans le badge du pied de page et dans les résumés de résultats d'outils, vers un outil interne de revue de code au lieu de `github.com`. Claude Code substitue `{host}`, `{owner}`, `{repo}`, `{number}` et `{url}` à partir de l'URL de la PR. Les liens de [merge request GitLab](https://code.claude.com/docs/en/interactive-mode#gitlab-merge-requests) sur ces deux surfaces gardent leur URL GitLab.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne, un modèle d'URL utilisant n'importe lequel des cinq espaces réservés
* **Défaut** : non défini

```json settings.json theme={null}
{
  "prUrlTemplate": "https://reviews.example.com/{owner}/{repo}/pull/{number}"
}
```

Claude Code n'applique le modèle qu'aux liens qu'il affiche lui-même ; un numéro de PR que Claude écrit dans un message, comme `#123`, reste tel que Claude l'a écrit. Une URL qui n'a pas la forme `/pull/<number>` reste inchangée.

### `attribution.commit`

Définir le texte d'attribution que Claude Code ajoute aux commits git, trailers compris. Mettez une chaîne vide pour masquer l'attribution des commits.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne
* **Défaut** : non défini, donc Claude Code ajoute `Co-Authored-By: <name> <noreply@anthropic.com>`. Le nom est le modèle utilisé au moment du commit, par exemple `Claude Sonnet 5`. Quand un [sous-agent](https://code.claude.com/docs/en/sub-agents) fait le commit, le trailer nomme le modèle du sous-agent.
  * Quand Claude Code reconnaît le modèle comme un modèle Claude mais ne peut pas confirmer sa version exacte, il écrit simplement `Claude`.
  * Quand il ne peut associer l'ID du modèle à aucun modèle Claude, par exemple un modèle tiers servi via un [`ANTHROPIC_BASE_URL`](https://code.claude.com/docs/en/env-vars) personnalisé, il écrit `Claude Code`.

Cet exemple remplace le trailer par défaut par une ligne personnalisée et un trailer `Co-Authored-By` personnalisé :

```json settings.json theme={null}
{
  "attribution": {
    "commit": "Generated with AI\n\nCo-Authored-By: AI <ai@example.com>"
  }
}
```

### `attribution.pr`

Définir le texte d'attribution que Claude Code ajoute aux descriptions de pull request. Mettez une chaîne vide pour masquer l'attribution des pull requests.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne
* **Défaut** : non défini, donc Claude Code ajoute `🤖 Generated with [Claude Code](https://claude.com/claude-code)`

```json settings.json theme={null}
{
  "attribution": {
    "pr": ""
  }
}
```

### `attribution.sessionUrl`

Choisir si Claude Code ajoute le lien de session claude.ai lorsqu'il commite ou ouvre une pull request depuis une session [cloud](https://code.claude.com/docs/en/claude-code-on-the-web) ou [Remote Control](https://code.claude.com/docs/en/remote-control). Claude Code ajoute le lien sous forme de trailer `Claude-Session` sur les commits et sous forme de lien dans les descriptions de pull request. Mettez `false` pour omettre le lien.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code ajoute le lien de session claude.ai lorsqu'il commite ou ouvre une pull request depuis une session cloud ou Remote Control
  * `false` : Claude Code omet le lien
* **Défaut** : `true`

```json settings.json theme={null}
{
  "attribution": {
    "sessionUrl": false
  }
}
```

## Hooks et automatisation

Enregistrer des hooks, restreindre les hooks qui s'exécutent et contrôler les workflows. Pour les événements de hook et leurs charges utiles, voir la [référence des hooks](https://code.claude.com/docs/en/hooks).

### `allowedHttpHookUrls`

Limiter les URL que les [hooks HTTP](https://code.claude.com/docs/en/hooks#http-hook-fields) peuvent cibler. Lorsque cette clé est définie, Claude Code n'exécute un hook HTTP que si son URL correspond à l'un des motifs et bloque les autres sans les exécuter ; un tableau vide bloque tous les hooks HTTP.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Les tableaux fusionnent entre fichiers de réglages.
* **Type** : tableau de motifs d'URL, avec `*` comme joker
* **Défaut** : non défini, donc toute URL est autorisée

Cet exemple autorise toute URL sous `https://hooks.example.com/` et toute URL `http://localhost` :

```json settings.json theme={null}
{
  "allowedHttpHookUrls": ["https://hooks.example.com/*", "http://localhost:*"]
}
```

La correspondance des noms d'hôte est insensible à la casse et traite `hooks.example.com.` (avec le point final qui marque un nom de domaine pleinement qualifié) comme `hooks.example.com`, à la manière du DNS. La liste d'autorisation s'applique aux hooks de toutes les sources, réglages gérés compris.

### `allowManagedHooksOnly`

Restreindre l'exécution des hooks à ceux que votre organisation déploie.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : seuls les hooks gérés s'exécutent, plus les hooks de l'Agent SDK et ceux des plugins que vos réglages gérés forcent à l'activation. Voir [Ce qui s'exécute sous `allowManagedHooksOnly`](#ce-qui-sexécute-sous-allowmanagedhooksonly)
  * `false` : les hooks de toutes les portées de réglages et de tous les plugins s'exécutent
* **Défaut** : non défini, donc les hooks de toutes les portées et de tous les plugins s'exécutent

```json managed-settings.json theme={null}
{
  "allowManagedHooksOnly": true
}
```

#### Ce qui s'exécute sous `allowManagedHooksOnly`

Quand vous le mettez à `true`, Claude Code modifie les hooks et commandes de type hook qui se chargent :

* **Les hooks gérés et SDK s'exécutent** : hooks des réglages gérés et hooks que l'[Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview) enregistre dans le processus
* **Les hooks des plugins forcés s'exécutent** : hooks des plugins que vos réglages gérés forcent à l'activation via [`enabledPlugins`](#enabledplugins). Claude Code compare l'ID complet `plugin@marketplace`, donc un plugin de même nom provenant d'une autre marketplace reste bloqué. Cela permet de distribuer des hooks validés via une marketplace d'organisation tout en bloquant le reste. Un [mod](https://code.claude.com/docs/en/plugins/mods/overview) d'un tel plugin ne se charge que s'il [compte comme appartenant à votre organisation](https://code.claude.com/docs/en/plugins/mods/admin#install-your-organizations-mods)
* **Tout le reste est bloqué** : hooks utilisateur, projet et locaux, hooks et mods des autres plugins installés, et hooks déclarés dans le frontmatter des agents. Les [mods intégrés à Claude Code](https://code.claude.com/docs/en/plugins/mods/overview#mods-built-into-claude-code) continuent de s'exécuter. Pour ne bloquer que les mods des utilisateurs, définissez plutôt [`allowManagedModsOnly`](https://code.claude.com/docs/en/plugins/mods/admin#set-options-on-the-built-in-guard)
* **Les plugins à source commande sont désactivés** : Claude Code désactive aussi les plugins à [source `command`](https://code.claude.com/docs/en/plugins/marketplace-reference#command-plugin-source), y compris ceux forcés dans le `enabledPlugins` géré, sauf si vous mettez explicitement [`disableCommandPluginSources`](#disablecommandpluginsources) à `false`
* **Les commandes `headersHelper` des marketplaces sont bloquées** : Claude Code bloque aussi les [commandes `headersHelper`](https://code.claude.com/docs/en/plugins/host-marketplace#authenticate-archive-downloads) des marketplaces sauf si [`disableCommandPluginSources`](#disablecommandpluginsources) est explicitement à `false`, à l'exception d'une marketplace déclarée par les réglages gérés eux-mêmes. (v2.1.238+)
* **La ligne de statut et la suggestion de fichiers se limitent aux réglages gérés** : Claude Code lit [`statusLine`](https://code.claude.com/docs/en/statusline), [`fileSuggestion`](#filesuggestion) et [`subagentStatusLine`](https://code.claude.com/docs/en/statusline#subagent-status-lines) uniquement depuis les réglages gérés, selon les [verrous de la ligne de statut et de la suggestion de fichiers](#verrous-de-la-ligne-de-statut-et-de-la-suggestion-de-fichiers)

La commande [`/goal`](https://code.claude.com/docs/en/goal) ne peut pas s'exécuter tant que cette clé est définie, car elle dépend des hooks.

### `disableAllHooks`

Désactiver les [hooks](https://code.claude.com/docs/en/hooks#disable-or-remove-hooks), toute [ligne de statut](https://code.claude.com/docs/en/statusline) personnalisée et toute commande personnalisée de [suggestion de fichiers](#filesuggestion). Permet de tout désactiver temporairement sans les supprimer de vos réglages.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Seuls les réglages gérés peuvent désactiver les hooks gérés.
* **Type** : booléen
  * `true` : Claude Code désactive les hooks, toute ligne de statut personnalisée et toute commande personnalisée de suggestion de fichiers
  * `false` : les hooks, la ligne de statut et la commande de suggestion de fichiers s'exécutent
* **Défaut** : non défini, donc les hooks s'exécutent

```json settings.json theme={null}
{
  "disableAllHooks": true
}
```

La portée de l'effet dépend du fichier qui porte la clé :

* **Dans les réglages gérés** : Claude Code désactive tous les hooks configurés, y compris les hooks gérés, et continue d'exécuter les hooks que l'[Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview) enregistre dans le processus
* **Dans tout autre fichier de réglages** : Claude Code désactive les hooks utilisateur, projet, locaux et de plugins ; les hooks gérés, les hooks de l'Agent SDK et ceux des plugins forcés dans le [`enabledPlugins`](#enabledplugins) géré continuent de s'exécuter

La clé arrête aussi les [mods](https://code.claude.com/docs/en/plugins/mods/overview), qui sont des plugins dont le code enregistre des hooks :

* **Dans les réglages gérés** : les mods de tous les plugins installés s'arrêtent, y compris ceux de votre organisation
* **Dans tout autre fichier de réglages** : les mods que vous avez installés s'arrêtent, et [les mods de votre organisation](https://code.claude.com/docs/en/plugins/mods/admin#install-your-organizations-mods) continuent de s'exécuter

Les mods intégrés à Claude Code continuent de s'exécuter dans les deux cas. Chacun a [son propre interrupteur](https://code.claude.com/docs/en/plugins/mods/overview#mods-built-into-claude-code).

Le maintien des hooks de l'Agent SDK lorsque les réglages gérés définissent cette clé exige v2.1.242+.

La commande [`/goal`](https://code.claude.com/docs/en/goal) ne peut pas s'exécuter tant que les hooks sont désactivés, et le menu `/hooks` affiche un avis au lieu de vos hooks.

#### Verrous de la ligne de statut et de la suggestion de fichiers

Claude Code prend deux décisions pour `statusLine`, `fileSuggestion` et `subagentStatusLine`, dans cet ordre :

* **Désactivation complète** : quand les réglages gérés définissent `disableAllHooks`, ou quand le dossier n'est pas approuvé selon la même [règle de confiance de l'espace de travail que les hooks des fichiers de réglages](https://code.claude.com/docs/en/permissions#what-runs-before-you-trust-a-folder)
* **Restriction aux réglages gérés** : quand [`allowManagedHooksOnly`](#allowmanagedhooksonly) est défini, quand `disableAllHooks` vaut `true` hors des réglages gérés après application de la [précédence des réglages](https://code.claude.com/docs/en/hooks#disable-or-remove-hooks), ou quand vous lancez Claude Code avec `--safe-mode`

En cas de restriction, Claude Code exécute une valeur gérée si elle est déployée. Sinon, il ignore votre valeur sans avertissement : la ligne de statut est désactivée et l'autocomplétion `@` se rabat sur la suggestion de fichiers intégrée.

### `disableWorkflows`

Désactiver les [workflows dynamiques](https://code.claude.com/docs/en/workflows#turn-workflows-off) et les commandes de workflow fournies pour toutes les personnes que vos réglages atteignent, par exemple une organisation via les réglages gérés. Pour activer ou désactiver les workflows pour vous seul, utilisez plutôt [`enableWorkflows`](#enableworkflows), que la bascule **Dynamic workflows** de `/config` écrit dans vos réglages utilisateur.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code désactive les workflows dynamiques et les commandes de workflow fournies pour toutes les personnes que vos réglages atteignent
  * `false` : identique à non défini ; l'activation des workflows suit alors [`enableWorkflows`](#enableworkflows) et le défaut de votre forfait
* **Défaut** : `false`
* **Surcharges par session** : [`CLAUDE_CODE_DISABLE_WORKFLOWS`](https://code.claude.com/docs/en/env-vars) désactive les workflows pour une session ; quel que soit celui des deux qui les désactive, l'autre ne peut pas les réactiver

```json settings.json theme={null}
{
  "disableWorkflows": true
}
```

### `enableWorkflows`

Activer ou désactiver les [workflows dynamiques](https://code.claude.com/docs/en/workflows) pour vous-même lorsque le défaut de votre forfait ne vous convient pas. Apparaît dans `/config` sous **Dynamic workflows**, qui écrit cette clé dans vos réglages utilisateur et la retire quand vous revenez au défaut de votre forfait. Pour désactiver les workflows pour tout le monde depuis les réglages gérés, utilisez plutôt [`disableWorkflows`](#disableworkflows).

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code active les workflows dynamiques pour vous
  * `false` : Claude Code désactive les workflows dynamiques pour vous
* **Défaut** : non défini, donc les workflows sont actifs, sauf sur le forfait Pro où ils sont désactivés
* **Surcharges par session** : [`CLAUDE_CODE_DISABLE_WORKFLOWS`](https://code.claude.com/docs/en/env-vars) désactive les workflows pour une session, et `true` ici ne peut pas les réactiver tant qu'elle est définie

```json settings.json theme={null}
{
  "enableWorkflows": true
}
```

[`disableWorkflows`](#disableworkflows) et la politique de workflows de votre organisation priment aussi : `enableWorkflows: true` ne peut pas réactiver les workflows tant qu'une source les désactive. Claude Code masque la ligne de `/config` lorsqu'une source autre que vos réglages utilisateur définit `enableWorkflows`, ou met `disableWorkflows` à `true`.

### `hooks`

Exécuter vos propres commandes, prompts, agents, requêtes HTTP ou outils MCP comme [hooks](https://code.claude.com/docs/en/hooks) à certains points du cycle de vie de Claude Code, par exemple avant un appel d'outil ou au démarrage d'une session ; la [référence des hooks](https://code.claude.com/docs/en/hooks#hook-events) liste chaque événement, sa charge utile et ses codes de sortie. Chaque événement correspond à une liste de groupes de matchers, et chaque groupe liste les gestionnaires à exécuter quand le matcher s'applique.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Les hooks fusionnent entre fichiers au lieu de se remplacer, et les hooks des réglages gérés ne peuvent pas être supprimés depuis d'autres fichiers.
* **Type** : objet indexé par [événement de hook](https://code.claude.com/docs/en/hooks#hook-events) ; chaque valeur est un tableau de groupes `{ "matcher", "hooks" }` dont les entrées `hooks` ont un `type` parmi `"command"`, `"prompt"`, `"agent"`, `"http"` ou `"mcp_tool"`
* **Défaut** : non défini, donc aucun hook ne s'exécute

Cet exemple exécute un script avant chaque appel de l'outil Bash :

```json settings.json theme={null}
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "~/.claude/hooks/check-bash.sh" }
        ]
      }
    ]
  }
}
```

Pour chaque événement, motif de matcher et champ de gestionnaire, voir la [référence des hooks](https://code.claude.com/docs/en/hooks#configuration). Pour désactiver les hooks, voir [`disableAllHooks`](#disableallhooks) ; pour limiter les hooks à ceux que votre organisation déploie, voir [`allowManagedHooksOnly`](#allowmanagedhooksonly).

### `httpHookAllowedEnvVars`

Un [hook HTTP](https://code.claude.com/docs/en/hooks#http-hook-fields) peut placer la valeur d'une variable d'environnement dans un en-tête de requête, par exemple un en-tête `Authorization: Bearer $HOOK_TOKEN`, mais uniquement pour les variables que le hook liste dans son propre `allowedEnvVars`. Cette clé fixe une limite externe à cette liste pour tous les hooks HTTP : un hook ne peut utiliser une variable que si son propre `allowedEnvVars` **et** cette clé la nomment. Sert à empêcher un hook de lire un secret qu'il ne devrait pas lire, même si sa définition le demande.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Les tableaux fusionnent entre fichiers de réglages.
* **Type** : tableau de noms de variables d'environnement
* **Défaut** : non défini, donc la liste `allowedEnvVars` propre à chaque hook s'applique

Cet exemple limite l'interpolation d'en-têtes à `MY_TOKEN` et `HOOK_SECRET` :

```json settings.json theme={null}
{
  "httpHookAllowedEnvVars": ["MY_TOKEN", "HOOK_SECRET"]
}
```

La liste d'autorisation s'applique aux hooks de toutes les sources, réglages gérés compris.

### `workflowKeywordTriggerEnabled`

Choisir si taper le mot-clé `ultracode` dans un prompt déclenche un [workflow dynamique](https://code.claude.com/docs/en/workflows#ask-for-a-workflow-in-your-prompt). Mettez `false` pour pouvoir taper le mot sans en déclencher.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Apparaît dans `/config` sous **Ultracode keyword trigger**.
* **Type** : booléen
  * `true` : taper `ultracode` dans un prompt déclenche un workflow dynamique
  * `false` : vous pouvez taper le mot sans en déclencher
* **Défaut** : `true`

```json settings.json theme={null}
{
  "workflowKeywordTriggerEnabled": false
}
```

Le réglage d'effort `ultracode`, `/workflows` et les commandes de workflow enregistrées ne sont pas affectés.

### `workflowSizeGuideline`

Définir le [nombre d'agents visé par Claude](https://code.claude.com/docs/en/workflows#set-a-size-guideline) dans les workflows dynamiques qu'il écrit. Claude Code transmet la valeur à Claude comme un conseil, pas comme un plafond imposé : `"small"` demande moins de 5 agents, `"medium"` moins de 10 et `"large"` moins de 50. Choisissez `"small"` pour borner ce qu'un workflow dépense. (v2.1.219+)

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Une valeur dans un fichier de réglages prime sur le choix **Dynamic workflow size** de `/config`, que Claude Code stocke dans `~/.claude.json`, et Claude Code masque cette ligne lorsqu'un fichier de réglages définit la clé.
* **Type** : chaîne, parmi :
  * `"unrestricted"` : aucune consigne, Claude dimensionne le workflow selon la tâche
  * `"small"` : Claude vise moins de 5 agents
  * `"medium"` : Claude vise moins de 10 agents
  * `"large"` : Claude vise moins de 50 agents
* **Défaut** : `"medium"`, ou `"small"` si vous êtes connecté avec un forfait Pro sur v2.1.271+

```json settings.json theme={null}
{
  "workflowSizeGuideline": "small"
}
```

Exige v2.1.219+ ; de v2.1.202 à v2.1.218, réglez plutôt la consigne dans `/config`.

## Plugins et skills

Activer des plugins, enregistrer des marketplaces, restreindre les sources de plugins autorisées par une organisation et contrôler quelles skills se chargent. Pour installer et créer des plugins, voir [Plugins](https://code.claude.com/docs/en/plugins/overview).

### `disableBundledSkills`

Désactiver les [skills](https://code.claude.com/docs/en/skills) et workflows fournis avec Claude Code. Claude Code supprime entièrement les skills et workflows fournis, tandis que les commandes intégrées comme `/init` restent tapables mais sont masquées au modèle.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code supprime les skills et workflows fournis et masque au modèle les commandes intégrées comme `/init`
  * `false` : les skills fournies se chargent
* **Défaut** : non défini, donc les skills fournies se chargent
* **Surcharges par session** : [`CLAUDE_CODE_DISABLE_BUNDLED_SKILLS`](https://code.claude.com/docs/en/env-vars) à `1` désactive les skills fournies pour une session ; quel que soit celui des deux qui les désactive, l'autre ne peut pas les réactiver

```json settings.json theme={null}
{
  "disableBundledSkills": true
}
```

Les skills provenant de plugins, de `.claude/skills/` et de `.claude/commands/` ne sont pas affectées. `/doctor` reste tapable comme les commandes intégrées ; pour le masquer, définissez plutôt [`DISABLE_DOCTOR_COMMAND`](https://code.claude.com/docs/en/env-vars).

### `disableSkillShellExecution`

Désactiver l'exécution shell en ligne pour les blocs `` !`...` `` et ` ```! ` dans les [skills](https://code.claude.com/docs/en/skills) et commandes personnalisées provenant de sources utilisateur, projet, plugin ou répertoire additionnel. Claude Code remplace chaque commande par `[shell command execution disabled by policy]` au lieu de l'exécuter.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Un `true` dans les réglages gérés ne peut pas être annulé par un `false` ailleurs.
* **Type** : booléen
  * `true` : Claude Code remplace chaque commande shell en ligne par `[shell command execution disabled by policy]` au lieu de l'exécuter
  * `false` : le shell en ligne s'exécute
* **Défaut** : non défini, donc le shell en ligne s'exécute

```json settings.json theme={null}
{
  "disableSkillShellExecution": true
}
```

Les skills fournies et les skills déployées via les réglages gérés ne sont pas affectées.

### `skillOverrides`

Masquer ou réduire une [skill](https://code.claude.com/docs/en/skills#override-skill-visibility-from-settings) sans modifier son `SKILL.md`. Claude Code applique la valeur associée au nom de chaque skill à la liste des skills que voit Claude et à votre autocomplétion `/`.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Le menu `/skills` écrit dans `.claude/settings.local.json`.
* **Type** : objet associant un nom de skill à l'une des valeurs :
  * `"on"` : Claude voit la skill et vous pouvez taper `/name`
  * `"name-only"` : Claude voit la skill par son nom, sans sa description
  * `"user-invocable-only"` : Claude ne voit pas la skill, mais vous pouvez toujours taper `/name`
  * `"off"` : Claude ne voit pas la skill et `/name` est masqué de l'autocomplétion
* **Défaut** : non défini, donc chaque skill est à `"on"`

Cet exemple présente `legacy-context` à Claude par son seul nom et masque `deploy` à Claude et à l'autocomplétion `/` :

```json settings.json theme={null}
{
  "skillOverrides": {
    "legacy-context": "name-only",
    "deploy": "off"
  }
}
```

Les surcharges ne s'appliquent pas aux skills de plugins, que vous gérez via `/plugin`.

Dans les réglages gérés et les fichiers passés avec `--settings`, une clé portant sur l'alias d'une skill fournie, comme `checkup` pour `/doctor`, s'applique aussi à la skill ; voir [comment les clés d'alias se combinent avec celles du nom propre de la skill](https://code.claude.com/docs/en/skills#override-skill-visibility-from-settings).

### `syncClaudeAiSkills`

Désactiver le téléchargement des [skills activées pour votre compte claude.ai](https://code.claude.com/docs/en/skills#how-synced-skills-behave). Claude Code les télécharge dans `~/.claude/skills/synced/` dans les [sessions terminal où vous vous connectez avec votre compte claude.ai](https://code.claude.com/docs/en/skills#where-synced-skills-load), interactives ou non, ainsi que dans les sessions Cowork et cloud. Mettez `false` pour arrêter ce téléchargement et ne plus charger les skills déjà synchronisées. Claude Code ne tient compte que de `false` : `true` équivaut à non défini et n'active pas la synchronisation là où elle est désactivée par ailleurs.

* **Portée** : [`User, local, or managed`](https://code.claude.com/docs/en/settings-reference#scopes), plus les fichiers passés avec `--settings`. Un dépôt ne peut pas la désactiver à votre place.
* **Type** : booléen
  * `false` : Claude Code arrête de télécharger les skills synchronisées et ne charge plus celles déjà présentes dans `~/.claude/skills/synced/`. Dans les réglages utilisateur ou gérés, il les déplace aussi vers `~/.claude/skills/.trash/`
  * `true` : identique à non défini
* **Défaut** : non défini, donc les sessions connectées avec votre compte claude.ai synchronisent vos skills

Cet exemple empêche une machine de télécharger les skills du compte dans toute session :

```json settings.json theme={null}
{
  "syncClaudeAiSkills": false
}
```

### `syncClaudeAiPlugins`

Désactiver le téléchargement des [plugins activés pour votre compte claude.ai](https://code.claude.com/docs/en/plugins/loading#synced-plugins). Claude Code les télécharge dans `~/.claude/plugins/synced/` au démarrage des sessions terminal où vous vous connectez avec votre compte claude.ai et dans les sessions Cowork, et charge chacun sous le nom `<name>@synced`. Mettez `false` pour arrêter ce téléchargement et ne plus charger les plugins déjà synchronisés. Claude Code ne tient compte que de `false` : `true` équivaut à non défini et n'active pas la synchronisation là où elle est désactivée par ailleurs. (v2.1.273+)

* **Portée** : [`User, local, or managed`](https://code.claude.com/docs/en/settings-reference#scopes), plus les fichiers passés avec `--settings`. Un dépôt ne peut pas la désactiver à votre place.
* **Type** : booléen
  * `false` : Claude Code arrête de télécharger les plugins synchronisés et ne charge plus ceux déjà présents dans `~/.claude/plugins/synced/`. Dans les réglages utilisateur ou gérés, il les déplace aussi vers `~/.claude/plugins/.trash/`
  * `true` : identique à non défini
* **Défaut** : non défini, donc les sessions connectées avec votre compte claude.ai synchronisent vos plugins

Pour désactiver un seul plugin synchronisé plutôt que tous, mettez `"<name>@synced": false` dans [`enabledPlugins`](#enabledplugins).

Cet exemple empêche une machine de télécharger les plugins du compte dans toute session :

```json settings.json theme={null}
{
  "syncClaudeAiPlugins": false
}
```

### `allowedChannelPlugins`

Choisir quels plugins de [canal](https://code.claude.com/docs/en/channels) peuvent pousser des messages dans les sessions de votre organisation. Lorsqu'elle est définie, Claude Code utilise votre liste à la place de la liste d'autorisation Anthropic par défaut ; chaque entrée nomme un plugin et la marketplace dont il provient.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : tableau d'objets, chacun avec les chaînes `marketplace` et `plugin`. Une entrée peut aussi être une chaîne `"plugin@marketplace"` comme `"telegram@claude-plugins-official"`, que Claude Code traite comme l'objet équivalent. La forme chaîne exige v2.1.267+ ; les versions antérieures rejettent toute la valeur `allowedChannelPlugins` si elle en contient une
* **Défaut** : non défini, donc Claude Code utilise la liste d'autorisation Anthropic par défaut

Cet exemple active les canaux et n'autorise que le plugin Telegram de la marketplace officielle Anthropic :

```json managed-settings.json theme={null}
{
  "channelsEnabled": true,
  "allowedChannelPlugins": [
    { "marketplace": "claude-plugins-official", "plugin": "telegram" }
  ]
}
```

Un tableau vide bloque tous les plugins de canal.

Cette clé prend effet une fois que les canaux ont franchi le verrou [`channelsEnabled`](#channelsenabled) pour le compte : sur les forfaits Team et Enterprise, et sur les comptes Console avec réglages gérés, cela signifie `channelsEnabled: true`. Voir [Restrict which channel plugins can run](https://code.claude.com/docs/en/channels#restrict-which-channel-plugins-can-run).

### `blockedMarketplaces`

Bloquer des sources de marketplaces de plugins pour votre organisation. Claude Code vérifie la liste de blocage à l'ajout d'une marketplace et lors de l'installation, la mise à jour, le rafraîchissement et la mise à jour automatique d'un plugin, de sorte qu'une marketplace ajoutée avant la politique ne peut pas non plus servir à récupérer des plugins. Les sources bloquées sont vérifiées avant téléchargement : elles ne touchent jamais le système de fichiers.

Si vous définissez cette clé dans la [console d'administration claude.ai](https://code.claude.com/docs/en/server-managed-settings), claude.ai l'applique aussi lorsque quelqu'un de votre organisation ajoute une marketplace depuis un dépôt git sur claude.ai, comme le décrit [How restrictions work](https://code.claude.com/docs/en/plugins/org#restrict-what-users-can-install).

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : tableau d'objets source de marketplace, sous les mêmes formes que [`strictKnownMarketplaces`](#types-de-sources-autorisés)
* **Défaut** : non défini, donc aucune marketplace n'est bloquée

Cet exemple bloque un dépôt GitHub comme source de marketplace :

```json managed-settings.json theme={null}
{
  "blockedMarketplaces": [
    { "source": "github", "repo": "untrusted/plugins" }
  ]
}
```

Une entrée `github` peut utiliser la [forme joker de propriétaire](#jokers-de-propriétaire) `"owner/*"` pour bloquer tous les dépôts de ce propriétaire GitHub (v2.1.223+). Ajoutez `{ "source": "skills-dir" }` pour empêcher Claude Code de charger les [plugins `@skills-dir`](https://code.claude.com/docs/en/plugins/loading#plugins-shared-through-a-repository) depuis `~/.claude/skills/` sans restreindre aucune marketplace. Voir [Managed marketplace restrictions](https://code.claude.com/docs/en/plugins/org#restrict-what-users-can-install).

### `channelsEnabled`

Autoriser les [canaux](https://code.claude.com/docs/en/channels) pour votre organisation. Sur les forfaits claude.ai Team et Enterprise, Claude Code bloque les canaux tant que vous ne mettez pas cette clé à `true`. Pour les comptes [Anthropic Console](https://code.claude.com/docs/en/authentication#claude-console-authentication) qui s'authentifient par clé API, les canaux sont autorisés par défaut. Si votre organisation déploie des réglages gérés, Claude Code bloque aussi les canaux sur ces comptes tant que vous ne mettez pas cette clé à `true`.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code autorise les canaux pour votre organisation
  * `false` : identique à non défini ; le blocage des canaux dépend de votre forfait, comme indiqué dans Défaut
* **Défaut** : non défini ; les canaux sont bloqués sur les forfaits Team et Enterprise et sur les comptes Console avec réglages gérés, et autorisés sur les forfaits Pro et Max et sur les comptes Console sans réglages gérés

```json managed-settings.json theme={null}
{
  "channelsEnabled": true
}
```

Pour restreindre quels plugins peuvent s'enregistrer comme canaux une fois ceux-ci activés, définissez [`allowedChannelPlugins`](#allowedchannelplugins). Voir [Enterprise controls](https://code.claude.com/docs/en/channels#enterprise-controls).

### `disableCommandPluginSources`

Bloquer la [source de plugin `command`](https://code.claude.com/docs/en/plugins/marketplace-reference#command-plugin-source), qui installe un plugin en exécutant sur la machine de l'utilisateur une commande déclarée par la marketplace. Avec `true`, Claude Code n'exécute jamais la commande, n'installe ni ne met à jour les plugins à source commande, et cesse de charger ceux déjà installés. Mettez `false` pour les autoriser explicitement. Chaque fois qu'il bloque les sources commande, que vous l'ayez mis à `true` ou laissé non défini sous [`allowManagedHooksOnly`](#allowmanagedhooksonly), il bloque aussi les [commandes `headersHelper`](https://code.claude.com/docs/en/plugins/host-marketplace#authenticate-archive-downloads) des marketplaces, sauf pour une marketplace déclarée par les réglages gérés eux-mêmes. (v2.1.229+ ; le blocage de `headersHelper` exige v2.1.238+)

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : booléen
  * `true` : Claude Code n'exécute jamais la commande déclarée par la marketplace, n'installe ni ne met à jour les plugins à source commande et cesse de charger ceux déjà installés
  * `false` : Claude Code autorise explicitement les plugins à source commande
* **Défaut** : non défini, donc Claude Code suit [`allowManagedHooksOnly`](#allowmanagedhooksonly) : une organisation qui restreint l'exécution des hooks aux réglages gérés voit aussi les sources commande désactivées

```json managed-settings.json theme={null}
{
  "disableCommandPluginSources": true
}
```

Exige v2.1.229+.

### `pluginSuggestionMarketplaces`

Nommer les marketplaces dont les plugins peuvent apparaître comme suggestions d'installation contextuelles, dans les astuces du spinner et épinglés en haut de l'onglet **Discover** de `/plugin`. L'astuce intégrée first-party frontend-design n'est pas affectée. Les suggestions proviennent de la déclaration `relevance` de chaque plugin dans son entrée de marketplace.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : tableau de noms de marketplaces
* **Défaut** : non défini, donc aucune suggestion déclarée par une marketplace n'apparaît

```json managed-settings.json theme={null}
{
  "pluginSuggestionMarketplaces": ["acme-corp-plugins"]
}
```

Un nom ne prend effet que si la marketplace est enregistrée sur la machine et que sa source enregistrée est aussi déclarée dans les mêmes réglages gérés, soit comme entrée [`extraKnownMarketplaces`](#extraknownmarketplaces) pour ce nom, soit comme entrée de [`strictKnownMarketplaces`](#strictknownmarketplaces). Claude Code ignore une marketplace enregistrée depuis une autre source sous un nom autorisé. La marketplace officielle est exemptée de l'exigence de source : autoriser son nom suffit, puisque ce nom ne peut s'enregistrer que depuis la source officielle Anthropic. Voir [Suggest plugins by context](https://code.claude.com/docs/en/plugins/relevance).

### `pluginTrustMessage`

Ajouter un texte propre à votre organisation à l'avertissement de confiance des plugins que Claude Code affiche avant l'installation, par exemple pour confirmer que les plugins de votre marketplace interne sont validés.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : chaîne
* **Défaut** : non défini, donc Claude Code affiche l'avertissement standard seul

```json managed-settings.json theme={null}
{
  "pluginTrustMessage": "All plugins from our marketplace are approved by IT"
}
```

### `strictKnownMarketplaces`

Restreindre les sources de marketplaces de plugins depuis lesquelles les membres de votre organisation peuvent ajouter et installer des plugins. Claude Code applique la liste d'autorisation à l'ajout d'une marketplace et lors de l'installation, la mise à jour, le rafraîchissement et la mise à jour automatique d'un plugin, avant toute opération réseau ou système de fichiers : une marketplace ajoutée avant la politique ne peut donc plus servir à récupérer des plugins dès que sa source ne correspond plus. Les utilisateurs bloqués voient une erreur citant la politique gérée.

Si vous définissez cette clé dans la [console d'administration claude.ai](https://code.claude.com/docs/en/server-managed-settings), claude.ai l'applique aussi lorsque quelqu'un de votre organisation ajoute une marketplace depuis un dépôt git sur claude.ai, comme le décrit [How restrictions work](https://code.claude.com/docs/en/plugins/org#restrict-what-users-can-install).

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : tableau d'objets source de marketplace ; voir [Types de sources autorisés](#types-de-sources-autorisés)
* **Défaut** : non défini, donc les utilisateurs peuvent ajouter n'importe quelle marketplace. Un tableau vide est un verrouillage complet qui bloque toutes les sources de marketplaces, y compris la marketplace officielle Anthropic

Cet exemple autorise deux dépôts GitHub, dont l'un épinglé sur la ref `v2.0`, et une URL `marketplace.json` hébergée :

```json managed-settings.json theme={null}
{
  "strictKnownMarketplaces": [
    { "source": "github", "repo": "acme-corp/approved-plugins" },
    { "source": "github", "repo": "acme-corp/security-tools", "ref": "v2.0" },
    { "source": "url", "url": "https://plugins.example.com/marketplace.json" }
  ]
}
```

Vous pouvez aussi écrire cette clé `allowedMarketplaces` ; [Alias des clés de marketplace](#alias-des-clés-de-marketplace) décrit comment Claude Code traite l'alias et quelle version l'accepte. Cette clé est un verrou de politique : elle contrôle ce que les utilisateurs peuvent ajouter mais n'enregistre rien. Pour restreindre et pré-enregistrer dans un même fichier, voir [Combiner avec `extraKnownMarketplaces`](#combiner-avec-extraknownmarketplaces). Pour la vue côté utilisateur, voir [Managed marketplace restrictions](https://code.claude.com/docs/en/plugins/org#restrict-what-users-can-install).

#### Types de sources autorisés

Chaque ligne ci-dessous montre une entrée de liste d'autorisation par type de source et les champs qu'elle accepte. La plupart des types correspondent exactement ; `hostPattern` et `pathPattern` correspondent par regex, et les entrées `github` peuvent utiliser un [joker de propriétaire](#jokers-de-propriétaire).

| Source | Exemple d'entrée | Champs |
| :- | :- | :- |
| `github` | `{ "source": "github", "repo": "acme-corp/plugins", "ref": "main", "path": "marketplace" }` | `repo` requis ; `ref` est une branche ou un tag ; `path` est un sous-répertoire |
| `git` | `{ "source": "git", "url": "https://gitlab.example.com/tools/plugins.git", "ref": "production" }` | `url` requis ; `ref` et `path` comme pour `github` |
| `url` | `{ "source": "url", "url": "https://plugins.example.com/marketplace.json", "headers": { "Authorization": "Bearer ${TOKEN}" } }` | `url` requis ; `headers` ajoute des en-têtes HTTP pour un accès authentifié |
| `file` | `{ "source": "file", "path": "/opt/acme-corp/plugins/marketplace.json" }` | `path` requis, le chemin absolu d'un fichier `marketplace.json` |
| `directory` | `{ "source": "directory", "path": "/opt/acme-corp/approved-marketplaces" }` | `path` requis, le chemin absolu d'un répertoire contenant `.claude-plugin/marketplace.json` |
| `hostPattern` | `{ "source": "hostPattern", "hostPattern": "^github\\.example\\.com$" }` | `hostPattern` requis, une regex recherchée n'importe où dans l'hôte de la marketplace ; ancrez-la avec `^` et `$` pour correspondre à l'hôte entier |
| `pathPattern` | `{ "source": "pathPattern", "pathPattern": "^/opt/approved/" }` | `pathPattern` requis, une regex recherchée n'importe où dans le `path` des sources `file` et `directory` ; commencez-la par `^` pour fixer un préfixe |
| `skills-dir` | `{ "source": "skills-dir" }` | Aucun champ. Réactive l'analyse des plugins de `~/.claude/skills/` |

Trois types de sources ont des règles allant au-delà du tableau :

* **`url`** : une marketplace URL ne télécharge que le fichier `marketplace.json`, et Claude Code ne récupère pas de fichiers de plugins par chemin relatif depuis ce serveur ; ses plugins doivent donc utiliser une [source de plugin](https://code.claude.com/docs/en/plugins/marketplace-reference#plugin-sources) autre qu'un chemin relatif, comme une URL d'archive, qui peut être sur le même hôte. Pour des plugins à chemins relatifs, utilisez plutôt une marketplace basée sur Git. Voir [Plugins with relative paths fail in URL-based marketplaces](https://code.claude.com/docs/en/plugins/troubleshooting#plugins-with-relative-paths-fail-in-url-based-marketplaces).
* **`hostPattern`** : permet d'autoriser toutes les marketplaces d'un serveur interne GitHub Enterprise ou GitLab sans lister chaque dépôt. Claude Code compare les sources `github` à `github.com`, prend le nom d'hôte des sources `url` et le prend des sources `git` selon la forme de l'[URL git](https://git-scm.com/docs/git-clone#_git_urls) :

  * Une URL avec schéma, comme `https://` ou `ssh://` : le nom d'hôte de l'URL.
  * Une adresse SSH sans schéma, de la forme git `user@host:path`, comme `git@git.example.com:tools/plugins.git` : l'hôte entre `@` et `:`, c'est-à-dire l'hôte auquel git se connecte.
  * Toute autre forme sans schéma : pas d'hôte, donc aucune entrée `hostPattern` de `strictKnownMarketplaces` ne correspond. Pour un `hostPattern` de `blockedMarketplaces`, Claude Code extrait un hôte d'un ensemble plus large de formes, si bien qu'une entrée de blocage peut tout de même correspondre à une telle forme. Avant v2.1.234, un `hostPattern` de `strictKnownMarketplaces` correspondait aussi à certaines formes que git ne traite pas comme des adresses SSH.

  Les sources `file` et `directory` n'ont pas d'hôte et ne correspondent jamais à une entrée `hostPattern`.
* **`pathPattern`** : permet d'autoriser des marketplaces du système de fichiers à côté d'entrées `hostPattern` pour les sources réseau. `".*"` autorise tout chemin local ; un motif plus étroit comme `"^/opt/approved/"` restreint à un répertoire.

Toute liste d'autorisation, même vide, empêche aussi Claude Code de charger les [plugins `@skills-dir`](https://code.claude.com/docs/en/plugins/loading#plugins-shared-through-a-repository) depuis `~/.claude/skills/`. Ajoutez l'entrée `{ "source": "skills-dir" }` pour continuer à les charger ; cette entrée n'a de sens que dans cette clé et dans `blockedMarketplaces`.

#### Jokers de propriétaire

Une entrée `github` dont la valeur `repo` est `"<owner>/*"` correspond à tous les dépôts de ce propriétaire GitHub. Les jokers de propriétaire exigent v2.1.223+ et ne fonctionnent que dans `strictKnownMarketplaces` et `blockedMarketplaces`. Partout ailleurs où une source `github` apparaît, comme `extraKnownMarketplaces` ou `/plugin marketplace add`, la valeur `repo` doit nommer un seul dépôt. Avant v2.1.223, Claude Code comparait l'entrée littéralement : une entrée d'autorisation ne correspondait à aucun dépôt et une entrée de blocage ne bloquait rien ; les entrées à dépôt unique sont appliquées dans toutes les versions.

Cette entrée autorise tout dépôt de marketplace de l'organisation `acme-corp` :

```json managed-settings.json theme={null}
{
  "strictKnownMarketplaces": [
    { "source": "github", "repo": "acme-corp/*" }
  ]
}
```

Seule la position entière du nom de dépôt peut être un joker. Claude Code ignore comme invalides des entrées comme `*`, `*/plugins` ou `acme-corp/tools-*`, qui ne correspondent donc à aucun dépôt.

Les règles de correspondance diffèrent entre les deux réglages :

| Règle | `strictKnownMarketplaces` | `blockedMarketplaces` |
| - | - | - |
| Graphies de source reconnues | Forme `owner/repo` uniquement. Une URL git qui clone le même dépôt ne correspond pas | Toute graphie, y compris les URL git qui pointent vers le même dépôt github.com |
| Casse du propriétaire | Sensible à la casse | Insensible à la casse |
| `ref` | Suit les règles d'entrée exacte : une entrée avec `ref` ne correspond qu'aux sources ayant exactement cette ref, et une entrée sans `ref` ne correspond qu'aux sources qui n'en précisent pas | Une entrée sans `ref` bloque toutes les refs des dépôts qu'elle vise |
| `path` | Plus souple que les règles d'entrée exacte : une entrée avec `path` exige cette valeur exacte, tandis qu'une entrée sans `path` correspond à n'importe quel chemin dans le dépôt | Une entrée sans `path` bloque tous les chemins des dépôts qu'elle vise |

#### Correspondance exacte

Pour tous les types de sources sauf les entrées `github` à joker de propriétaire et les entrées `hostPattern` et `pathPattern` à regex, Claude Code n'autorise l'ajout d'un utilisateur que si la source de la marketplace correspond exactement à une entrée. Pour les sources basées sur git, `github` et `git`, la correspondance exacte inclut les champs facultatifs :

* Le `repo` ou l'`url` doit correspondre exactement
* Le champ `ref` doit correspondre exactement, ou être absent des deux côtés
* Le champ `path` doit correspondre exactement, ou être absent des deux côtés

Par exemple, Claude Code traite chaque paire ci-dessous comme deux sources différentes :

* `{ "source": "github", "repo": "acme-corp/plugins" }` et `{ "source": "github", "repo": "acme-corp/plugins", "ref": "main" }`
* `{ "source": "github", "repo": "acme-corp/plugins", "path": "marketplace" }` et `{ "source": "github", "repo": "acme-corp/plugins" }`

#### N'autoriser que la marketplace officielle

Pour autoriser la marketplace officielle Anthropic et rien d'autre, listez son dépôt :

```json managed-settings.json theme={null}
{
  "strictKnownMarketplaces": [
    { "source": "github", "repo": "anthropics/claude-plugins-official" }
  ]
}
```

Avec cette entrée, Claude Code garde disponible une marketplace officielle déjà enregistrée et, sur une machine neuve, enregistre automatiquement la marketplace au premier lancement d'une session terminal interactive. L'enregistrement automatique échoue le plus souvent sur :

* Les environnements non interactifs qui s'exécutent avant la première session terminal interactive de la machine.
* Les machines où Claude Code n'a tourné que via l'extension VS Code.
* Les machines où Claude Code a déjà lancé une session terminal interactive sous une politique qui bloquait la marketplace, comme le verrouillage par tableau vide. Claude Code enregistre la tentative bloquée et ne réessaie pas après le changement de politique.

Sur ces machines, ajoutez la marketplace à [`extraKnownMarketplaces`](#extraknownmarketplaces) dans le même `managed-settings.json` pour que Claude Code l'enregistre automatiquement, ou lancez `claude plugin marketplace add anthropics/claude-plugins-official`.

#### Combiner avec `extraKnownMarketplaces`

Les deux clés remplissent des rôles différents. Ce tableau les compare :

| Aspect | `strictKnownMarketplaces` | `extraKnownMarketplaces` |
| - | - | - |
| Objectif | Application d'une politique d'organisation | Commodité pour l'équipe |
| Fichier de réglages | Réglages gérés uniquement | N'importe quel fichier de réglages |
| Comportement | Bloque les ajouts hors liste d'autorisation | Enregistre les marketplaces manquantes |
| Moment d'application | Avant les opérations réseau et système de fichiers | Immédiatement depuis les réglages utilisateur ou gérés ; après la boîte de dialogue de confiance de l'espace de travail pour les fichiers d'un dépôt |
| Peut être surchargé | Non, précédence la plus haute | Oui, par des réglages de précédence supérieure |
| Format de source | Objet source direct | Marketplace nommée avec un objet `source` imbriqué |

Pour à la fois restreindre et pré-enregistrer une marketplace pour tous les utilisateurs, définissez les deux dans `managed-settings.json` :

```json managed-settings.json theme={null}
{
  "strictKnownMarketplaces": [
    { "source": "github", "repo": "acme-corp/plugins" }
  ],
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": { "source": "github", "repo": "acme-corp/plugins" }
    }
  }
}
```

Avec seulement `strictKnownMarketplaces`, les utilisateurs peuvent toujours ajouter eux-mêmes une marketplace autorisée avec `/plugin marketplace add`. La marketplace officielle Anthropic est la seule que Claude Code enregistre automatiquement, et seulement si la liste d'autorisation la permet. [N'autoriser que la marketplace officielle](#nautoriser-que-la-marketplace-officielle) liste les machines où cela échoue.

### `strictPluginOnlyCustomization`

Bloquer les skills, agents, hooks et serveurs MCP provenant des sources utilisateur et projet, pour qu'ils ne puissent venir que de plugins ou des réglages gérés. Combinez-le avec [`strictKnownMarketplaces`](#strictknownmarketplaces) pour contrôler toute la chaîne d'approvisionnement des personnalisations : la liste d'autorisation des marketplaces contrôle quels plugins les utilisateurs peuvent installer.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : `true` pour verrouiller les quatre types de personnalisation, ou un tableau nommant les types à verrouiller parmi `"skills"`, `"agents"`, `"hooks"` et `"mcp"`
* **Défaut** : non défini, donc rien n'est verrouillé

Cet exemple verrouille les skills et les hooks et laisse les agents et serveurs MCP déverrouillés :

```json managed-settings.json theme={null}
{
  "strictPluginOnlyCustomization": ["skills", "hooks"]
}
```

Les quatre entrées de sous-clés ci-dessous indiquent ce que chaque surface bloque et ce qui se charge encore. Claude Code ignore les noms de surface qu'il ne reconnaît pas au lieu de faire échouer le fichier de réglages, ce qui permet d'ajouter de nouveaux noms de surface avant que tous les clients soient mis à jour.

### `strictPluginOnlyCustomization.skills`

Verrouiller la surface `skills`. Claude Code cesse de charger les skills de `~/.claude/skills/` et `.claude/skills/`, les commandes personnalisées de `~/.claude/commands/` et `.claude/commands/`, les skills et commandes des répertoires `--add-dir`, et les skills synchronisées depuis votre compte claude.ai. Il continue de charger les skills de plugins, les skills fournies et les skills du répertoire de politique gérée.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : la chaîne `"skills"` dans le tableau [`strictPluginOnlyCustomization`](#strictpluginonlycustomization)
* **Défaut** : non verrouillé

```json managed-settings.json theme={null}
{
  "strictPluginOnlyCustomization": ["skills"]
}
```

### `strictPluginOnlyCustomization.agents`

Verrouiller la surface `agents`. Claude Code cesse de charger les agents de `~/.claude/agents/`, `.claude/agents/` et des répertoires `--add-dir`. Il continue de charger les agents de plugins, les agents intégrés et les agents du répertoire de politique gérée.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : la chaîne `"agents"` dans le tableau [`strictPluginOnlyCustomization`](#strictpluginonlycustomization)
* **Défaut** : non verrouillé

```json managed-settings.json theme={null}
{
  "strictPluginOnlyCustomization": ["agents"]
}
```

### `strictPluginOnlyCustomization.hooks`

Verrouiller la surface `hooks`. Claude Code cesse d'exécuter les hooks des `settings.json` utilisateur, projet et locaux, et continue d'exécuter les hooks de plugins et ceux des réglages gérés.

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : la chaîne `"hooks"` dans le tableau [`strictPluginOnlyCustomization`](#strictpluginonlycustomization)
* **Défaut** : non verrouillé

```json managed-settings.json theme={null}
{
  "strictPluginOnlyCustomization": ["hooks"]
}
```

### `strictPluginOnlyCustomization.mcp`

Verrouiller la surface `mcp`. Claude Code cesse de charger les serveurs MCP de `~/.claude.json` et `.mcp.json`, et continue de charger les serveurs MCP de plugins, les serveurs de [`managed-mcp.json`](https://code.claude.com/docs/en/managed-mcp) et ceux de [`managedMcpServers`](https://code.claude.com/docs/en/settings-reference#managedmcpservers).

* **Portée** : [`Managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : la chaîne `"mcp"` dans le tableau [`strictPluginOnlyCustomization`](#strictpluginonlycustomization)
* **Défaut** : non verrouillé

```json managed-settings.json theme={null}
{
  "strictPluginOnlyCustomization": ["mcp"]
}
```

### `enabledPlugins`

Activer ou désactiver individuellement des [plugins](https://code.claude.com/docs/en/plugins/overview), indexés par `plugin-name@marketplace-name`. Un plugin sans entrée à aucune portée se rabat sur sa valeur [`defaultEnabled`](https://code.claude.com/docs/en/plugins/manifest-reference#fields). Quand vous activez ou désactivez un plugin avec `/plugin` ou `claude plugin enable`, Claude Code écrit cette clé pour vous.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : objet associant `plugin-name@marketplace-name` à un booléen
* **Défaut** : non défini, donc chaque plugin suit sa valeur `defaultEnabled`

Cet exemple active deux plugins de la marketplace `team-tools` et en désactive un de `personal` :

```json settings.json theme={null}
{
  "enabledPlugins": {
    "code-formatter@team-tools": true,
    "deployment-tools@team-tools": true,
    "experimental-features@personal": false
  }
}
```

Chaque portée a un rôle différent :

* **Réglages utilisateur** : vos préférences personnelles de plugins
* **Réglages projet** : plugins partagés avec tout le monde dans le dépôt
* **Réglages locaux** : surcharges propres à la machine, ignorées par git (gitignored) quand Claude Code y enregistre un réglage
* **Réglages gérés** : politique à l'échelle de l'organisation. Un plugin mis à `false` ici est bloqué à l'installation à toutes les portées et masqué de la marketplace

Les réglages projet priment sur les réglages utilisateur : mettre un plugin à `false` dans `~/.claude/settings.json` ne désactive donc pas un plugin que le `.claude/settings.json` du projet active. Pour refuser sur votre machine un plugin activé par le projet, mettez-le plutôt à `false` dans `.claude/settings.local.json`. Les plugins forcés par les réglages gérés ne peuvent pas être désactivés ainsi, puisque les réglages gérés priment sur les réglages locaux.

Activer dans le `.claude/settings.json` d'un projet un plugin provenant d'une source externe, comme un dépôt GitHub ou un paquet npm, ne l'installe pas pour les autres. Sur tous les chemins qui chargent des plugins, Claude Code signale le plugin comme non installé tant que chaque utilisateur ne l'a pas [installé lui-même](https://code.claude.com/docs/en/plugins/org#require-plugins-per-repository).

### `extraKnownMarketplaces`

Enregistrer des marketplaces de plugins supplémentaires par nom, pour que les personnes qui ouvrent le dépôt, ou toutes celles que vos réglages gérés atteignent, obtiennent la marketplace sans l'ajouter elles-mêmes. Claude Code enregistre chaque marketplace qu'il ne connaît pas encore. L'installation d'un plugin de cette marketplace nommé par [`enabledPlugins`](#enabledplugins) dépend de la source du plugin et du fichier qui l'active ; cette entrée en donne les règles.

* **Portée** : [`Any file`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code ne respecte les entrées du `.claude/settings.json` ou du `.claude/settings.local.json` d'un dépôt qu'après acceptation de la boîte de dialogue de confiance de l'espace de travail pour ce dossier ; dans un dossier non approuvé, y compris lors d'une exécution `-p` dans ce dossier, il les ignore sans message.
* **Type** : objet associant un nom de marketplace à un objet avec un objet `source` et un booléen `autoUpdate` facultatif
* **Défaut** : non défini

Cet exemple enregistre une marketplace GitHub et une marketplace depuis une URL git auto-hébergée :

```json settings.json theme={null}
{
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    },
    "security-plugins": {
      "source": {
        "source": "git",
        "url": "https://git.example.com/security/plugins.git"
      }
    }
  }
}
```

[What runs before you trust a folder](https://code.claude.com/docs/en/permissions#what-runs-before-you-trust-a-folder) compare ce verrou de confiance aux autres contenus qu'un dépôt peut fournir. Vous pouvez aussi écrire cette clé `additionalMarketplaces` ; voir [Alias des clés de marketplace](#alias-des-clés-de-marketplace).

Ajoutez `"autoUpdate": true` à côté de `source` pour que Claude Code rafraîchisse cette marketplace et mette à jour ses plugins installés en arrière-plan après le démarrage. S'il est omis, `claude-plugins-official` et la plupart des autres marketplaces officielles Anthropic valent `true` par défaut, et les marketplaces tierces `false`. Voir [Configure auto-updates](https://code.claude.com/docs/en/plugins/install#keep-plugins-updated).

Lorsque plusieurs fichiers de réglages définissent une entrée de marketplace sous le même nom, Claude Code utilise en entier l'entrée du [fichier de plus haute précédence](https://code.claude.com/docs/en/settings#settings-precedence). Cette entrée remplace celle de précédence inférieure et n'hérite d'aucun de ses champs, de sorte qu'une redéfinition ne peut pas combiner l'identifiant `source.headers` d'un fichier avec une URL contrôlée par un autre. Avant v2.1.228, Claude Code fusionnait les entrées de même nom champ par champ, si bien qu'une entrée de précédence supérieure pouvait hériter de champs qu'elle ne définissait pas, y compris les `headers` d'un autre fichier.

#### Types de sources de marketplace

L'objet `source` prend l'une de ces formes :

* **`github`** : un dépôt GitHub, avec `repo`
* **`git`** : toute URL git, avec `url`
* **`url`** : une URL directe vers un fichier `marketplace.json`, avec `url` et, facultativement, `headers` et `headersHelper` pour un accès authentifié. `headersHelper` nomme une commande qui affiche des en-têtes dont les valeurs ont une durée de vie trop courte pour être listées dans `headers` (v2.1.238+)
* **`file`** : un chemin local vers un fichier `marketplace.json`, avec `path`
* **`directory`** : un chemin du système de fichiers local, avec `path`. Pour le développement, ou pour une marketplace que votre organisation [déploie sur chaque machine](https://code.claude.com/docs/en/plugins/mods/admin#install-your-organizations-mods).
* **`settings`** : une marketplace en ligne déclarée directement dans le fichier de réglages sans dépôt hébergé, avec `name` et `plugins`

Le type de source `git` fonctionne avec tout service d'hébergement git, y compris GitLab et Bitbucket auto-hébergés. Claude Code clone le dépôt avec la même authentification que `git clone` utiliserait sur cette machine : assistants d'identification (credential helpers) configurés ou clés SSH. Un jeton de fournisseur comme `GITHUB_TOKEN` prend effet via un credential helper qui le lit. Voir [Private repositories](https://code.claude.com/docs/en/plugins/host-marketplace#grant-access-to-a-private-marketplace) pour la configuration.

Pour les sources `github` et `git`, Claude Code ne télécharge jamais le contenu [Git LFS](https://git-lfs.com) lorsqu'il clone le dépôt de la marketplace pour l'ajouter ou la mettre à jour. Les fichiers suivis par LFS sont extraits sous forme de fichiers pointeurs, et la sortie de l'ajout ou de la mise à jour indique leur nombre.

Le champ `skipLfs` dans l'objet `source` est accepté et sans effet. Avant v2.1.274, Claude Code téléchargeait le contenu LFS sauf si vous définissiez `"skipLfs": true`.

Pour une source `url`, définissez `headersHelper` dans l'objet `source` quand l'identifiant de `headers` expire et qu'une commande doit en produire un nouveau. (v2.1.238+) Pour ce que la commande doit afficher et où Claude Code l'exécute, voir [Write the headersHelper command](https://code.claude.com/docs/en/plugins/host-marketplace#write-the-headershelper-command), et pour les cas où Claude Code ne l'exécute pas, voir [When Claude Code skips a headersHelper command](https://code.claude.com/docs/en/plugins/host-marketplace#when-claude-code-skips-a-headershelper-command-or-drops-its-output). Une fois `headersHelper` défini sur une URL de marketplace `https://`, Claude Code exécute la commande à deux moments, en réutilisant la sortie d'une exécution pendant 60 secondes au plus :

* Avant chaque récupération du `marketplace.json` de cette marketplace, y compris lors d'un rafraîchissement ultérieur. Claude Code envoie les en-têtes affichés avec cette récupération.
* Avant chaque téléchargement d'archive de plugin sur l'origine de l'URL de la marketplace, c'est-à-dire même schéma, même hôte et même port. Claude Code envoie la sortie avec ce téléchargement, et aucun autre téléchargement ne reçoit les en-têtes.

Claude Code ignore tout `headersHelper` défini dans le `.claude/settings.json` ou le `.claude/settings.local.json` d'un répertoire ajouté avec [`--add-dir`](https://code.claude.com/docs/en/permissions#what-runs-before-you-trust-a-folder), que ce soit sur une source `url` ou sur une entrée de plugin en ligne, et n'envoie que les `headers` fixes définis dans ce fichier. [How users accept a headersHelper command](https://code.claude.com/docs/en/plugins/host-marketplace#how-users-accept-a-headershelper-command) couvre les autres fichiers de réglages.

Les plugins listés dans une source `settings` doivent référencer des sources externes comme GitHub ou npm, et le `name` doit correspondre à la clé de la marketplace. Vous devez toujours activer chaque plugin séparément dans `enabledPlugins`. Cet exemple déclare un plugin en ligne :

```json settings.json theme={null}
{
  "extraKnownMarketplaces": {
    "team-tools": {
      "source": {
        "source": "settings",
        "name": "team-tools",
        "plugins": [
          {
            "name": "code-formatter",
            "source": {
              "source": "github",
              "repo": "acme-corp/code-formatter"
            }
          }
        ]
      }
    }
  }
}
```

Une entrée de plugin sous `source: 'settings'` dont la propre `source` est une [`archive`](https://code.claude.com/docs/en/plugins/marketplace-reference#archive-plugin-source) peut définir `headers` pour le téléchargement de l'archive. Si la valeur que vous mettriez dans `headers` a une durée de vie courte, comme un jeton que votre registre émet à la demande, définissez plutôt une commande `headersHelper`. Une entrée peut définir les deux. Les deux champs exigent v2.1.238+.

Claude Code envoie les `headers` de l'entrée, et ce qu'affiche la commande, avec le téléchargement de l'archive de ce plugin et avec aucun autre téléchargement. Claude Code n'exécute la commande que lorsqu'un utilisateur [installe ou met à jour ce seul plugin isolément](https://code.claude.com/docs/en/plugins/host-marketplace#how-users-accept-a-headershelper-command). Trois règles supplémentaires dépendent du fichier qui contient l'entrée :

* **`strict`** : contrairement à une entrée dans le `marketplace.json` d'une marketplace, une entrée dans les réglages n'a pas besoin de `"strict": false`, car un fichier de réglages ne porte aucun champ de manifeste à intégrer. Voir [Strict mode](https://code.claude.com/docs/en/plugins/marketplace-reference#strict-mode).
* **Confiance du dossier** : pour une entrée dans le `.claude/settings.json` ou le `.claude/settings.local.json` d'un projet, Claude Code n'exécute la commande qu'après que l'utilisateur a aussi [approuvé ce dossier](https://code.claude.com/docs/en/permissions#what-runs-before-you-trust-a-folder).
* **Filtre d'en-têtes** : Claude Code retire les [noms d'en-têtes de routage de requête et d'identité client](https://code.claude.com/docs/en/plugins/host-marketplace#when-claude-code-skips-a-headershelper-command-or-drops-its-output) d'une entrée située dans le `.claude/settings.json` ou le `.claude/settings.local.json` d'un projet, car un dépôt peut fournir ces fichiers. Claude Code applique le même filtre à une entrée de catalogue et à une entrée des réglages d'un répertoire `--add-dir`, et aucun filtre à une entrée de vos réglages utilisateur, d'un fichier `--settings` ou des réglages gérés.

#### Alias des clés de marketplace

Sur Claude Code v2.1.232+, vous pouvez écrire `extraKnownMarketplaces` sous la forme `additionalMarketplaces` et `strictKnownMarketplaces` sous la forme `allowedMarketplaces`. Claude Code traite chaque alias ainsi :

* Les versions antérieures ignorent l'alias : gardez la graphie canonique dans un fichier également lu par d'anciennes versions, comme un fichier de réglages gérés pour un parc aux versions de Claude Code hétérogènes.
* Dans tout fichier de réglages qui accepte la clé canonique, Claude Code lit l'alias exactement comme la clé canonique.
* Claude Code peut réécrire `additionalMarketplaces` en `extraKnownMarketplaces` lorsqu'il met à jour le fichier.
* Si vous définissez les deux graphies dans un même fichier, Claude Code utilise la valeur canonique et ignore l'alias.

### `pluginConfigs`

Stocker les réponses non sensibles que vous donnez à la boîte de dialogue de configuration [`userConfig`](https://code.claude.com/docs/en/plugins/manifest-reference#user-configuration) d'un plugin, indexées par ID de plugin. Claude Code écrit cette clé dans vos réglages utilisateur quand vous remplissez la boîte de dialogue, vous n'avez donc pas à la modifier à la main. Claude Code stocke les options sensibles dans le trousseau macOS (Keychain), avec repli sur `~/.claude/.credentials.json` si le trousseau refuse l'écriture ; sur les plateformes sans trousseau pris en charge, il les stocke dans `~/.claude/.credentials.json`.

* **Portée** : [`User or managed`](https://code.claude.com/docs/en/settings-reference#scopes)
* **Type** : objet associant un ID de plugin à un objet avec un champ `options`, qui associe chaque nom d'option à une chaîne, un nombre, un booléen ou un tableau de chaînes, et un champ facultatif `mcpServers` contenant les valeurs de configuration utilisateur par serveur sous la même forme
* **Défaut** : non défini

Cet exemple stocke l'option `api_endpoint` du plugin `deployer` de `acme-tools` :

```json settings.json theme={null}
{
  "pluginConfigs": {
    "deployer@acme-tools": {
      "options": {
        "api_endpoint": "https://api.example.com"
      }
    }
  }
}
```

Les plugins intégrés stockent leurs options sous la même clé avec un suffixe `@builtin`. Par exemple, le réglage [**Project instructions**](https://code.claude.com/docs/en/memory#choose-which-instruction-files-load), qui contrôle si Claude Code lit les fichiers `AGENTS.md`, est `pluginConfigs["cc-plugin-agents-md@builtin"].options.instructionFiles`. Avant v2.1.285, l'ID du plugin était `agents-md@builtin`. Les versions ultérieures lisent une entrée sous l'un ou l'autre ID.

Claude Code ignore les entrées projet et locales parce qu'il substitue ces valeurs dans les configurations de hooks, MCP et LSP des plugins, et qu'un dépôt cloné ne doit pas pouvoir les fournir.

### `prependPlugins`

Lister les plugins gérés dont les [mods](https://code.claude.com/docs/en/plugins/mods/overview) s'exécutent avant chaque mod installé par un utilisateur, dans l'ordre listé. Quand vous définissez cette clé dans les réglages gérés, nommez `sec-default@builtin` dans la liste pour conserver la protection intégrée. Dans les réglages gérés, Claude Code ignore un id dont le plugin ne compte pas comme appartenant à votre organisation. Voir [Install your organization's mods and set the order](https://code.claude.com/docs/en/plugins/mods/admin#install-your-organizations-mods) pour ces conditions et pour le fonctionnement conjoint des deux clés d'ordre.

* **Portée** : [`User or managed`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code lit la clé depuis les réglages gérés. Il ne la lit depuis les réglages utilisateur que sur une machine sans réglages gérés, pour un utilisateur non connecté avec un forfait Team ou Enterprise. Il l'ignore dans les réglages projet et locaux et dans un fichier `--settings`.
* **Type** : tableau de chaînes `plugin-name@marketplace-name`
* **Défaut** : non défini

```json managed-settings.json theme={null}
{
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": { "source": "directory", "path": "/opt/acme/claude-plugins" }
    }
  },
  "enabledPlugins": { "acme-guard@acme-tools": true },
  "prependPlugins": ["acme-guard@acme-tools", "sec-default@builtin"]
}
```

### `appendPlugins`

Lister les plugins gérés dont les [mods](https://code.claude.com/docs/en/plugins/mods/overview) s'exécutent après chaque mod installé par un utilisateur, dans l'ordre listé. Un id listé à la fois dans `prependPlugins` et `appendPlugins` est placé en tête (prepend). Dans les réglages gérés, Claude Code ignore un id dont le plugin ne [compte pas comme appartenant à votre organisation](https://code.claude.com/docs/en/plugins/mods/admin#install-your-organizations-mods).

* **Portée** : [`User or managed`](https://code.claude.com/docs/en/settings-reference#scopes). Claude Code lit la clé depuis les réglages gérés. Il ne la lit depuis les réglages utilisateur que sur une machine sans réglages gérés, pour un utilisateur non connecté avec un forfait Team ou Enterprise. Il l'ignore dans les réglages projet et locaux et dans un fichier `--settings`.
* **Type** : tableau de chaînes `plugin-name@marketplace-name`
* **Défaut** : non défini

```json managed-settings.json theme={null}
{
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": { "source": "directory", "path": "/opt/acme/claude-plugins" }
    }
  },
  "enabledPlugins": { "acme-audit@acme-tools": true },
  "appendPlugins": ["acme-audit@acme-tools"]
}
```

## MCP

Contrôler à quels serveurs MCP Claude Code se connecte et lesquels une organisation autorise. Voir [Connect to external tools with MCP](https://code.claude.com/docs/en/mcp) et [Managed MCP configuration](https://code.claude.com/docs/en/managed-mcp).

> Les réglages détaillés de cette section figurent dans la partie 4/4.
