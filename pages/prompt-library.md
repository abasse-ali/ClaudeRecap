# Bibliothèque de prompts

> Source : https://code.claude.com/docs/en/prompt-library · [← Index](../31-index-documentation-officielle.md)

*Des prompts à copier-coller dans Claude Code, étiquetés par tâche et par rôle.*

Cette page est une bibliothèque de prompts à copier dans Claude Code. Servez-vous-en pour explorer des façons de travailler que vous n'avez pas encore essayées, ou quand vous ne savez pas par où commencer.

Les prompts sont tirés de plusieurs guides d'Anthropic, notamment [Common workflows](https://code.claude.com/docs/en/common-workflows), [Best practices](https://code.claude.com/docs/en/best-practices) et [How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code). Ce sont des points de départ, pas des scripts. Ouvrez **Why this works** (« Pourquoi ça marche ») sous n'importe quel prompt pour voir le principe qui le sous-tend, afin de pouvoir écrire les vôtres.

> **Note** : sur le site, la bibliothèque est un composant interactif (`PromptLibrary`, écrit en React) qui contient l'essentiel des données de la page. Cette fiche en extrait tout le contenu : le fonctionnement de l'interface, les étiquettes, phases, catégories, sources, prérequis et consignes, puis les **52 prompts** avec leur texte exact (à copier tel quel, en anglais), leurs champs à remplir et leurs valeurs d'exemple, leurs étiquettes, l'explication « Why this works », le lien « Make it stick » et la source.

## Utiliser la bibliothèque interactive

### Recherche

- Le champ **« Search prompts… »** (« Rechercher des prompts… ») filtre les cartes sans tenir compte de la casse. Il cherche dans le titre de la carte et dans son texte : le prompt (avec les valeurs d'exemple insérées), l'explication « Why this works » (sans les liens) et le texte « Make it stick ».
- Quand la correspondance n'est pas dans le titre, la carte affiche un extrait (environ 30 caractères avant et 50 après) où l'occurrence est surlignée, à la place de l'aperçu du prompt.
- Pendant une recherche, les boutons d'étiquette sont grisés (ils redeviennent nets au survol), le mode « Start here » est désactivé et les résultats gardent l'ordre du catalogue. Cliquer une étiquette vide la recherche.

### Bouton « Start here » et filtres par étiquette

- Le bouton **★ Start here** (« Commencer ici ») est actif à l'ouverture de la page. Il affiche, sous l'en-tête **« Five prompts to try first »** (« Cinq prompts à essayer en premier »), les cinq prompts de départ triés par numéro, puis un bouton **« Show all {n} prompts »** (« Afficher les {n} prompts », soit 52) qui réinitialise les filtres. Recliquer sur Start here le désactive ; l'activer désélectionne l'étiquette en cours.
- Viennent ensuite 19 boutons d'étiquette (voir le tableau plus bas). Un clic filtre sur cette étiquette ; un second clic sur la même étiquette retire le filtre. Une seule étiquette est active à la fois.
- Le bouton **Clear** (« Effacer ») apparaît dès qu'un filtre, le mode Start here ou une recherche est actif ; il réinitialise tout et affiche l'ensemble des prompts.
- Un compteur à droite indique le nombre de résultats : « 1 prompt » ou « N prompts ».
- Sans résultat, la page affiche **« No prompts match »** (« Aucun prompt ne correspond »), suivi du terme recherché et d'un bouton Clear.

### Tri et regroupement

- Hors mode Start here, les résultats sont regroupés sous des intertitres « phase · catégorie » (par exemple « Build · Test »), dans l'ordre du catalogue.
- Avec une étiquette sélectionnée, les prompts sont triés d'abord par nombre croissant de rôles associés (les prompts généraux, sans rôle, en premier), puis, à égalité, ceux de la phase **Operate** passent devant.

### Cartes de prompt

- **Carte fermée** : titre, pastille « Start here · N » pour les cinq prompts de départ, et aperçu du prompt sur une ligne (valeurs d'exemple insérées).
- **Carte ouverte** (clic sur le titre), dans l'ordre :
  1. le libellé **« Fill in and copy »** (« Remplir et copier ») si le prompt a des champs, sinon **« Copy this prompt »** (« Copier ce prompt ») ;
  2. un encadré **Needs** (« Prérequis ») si le prompt exige un outil connecté ;
  3. une consigne de collage si le prompt attend une image, une sortie ou un fichier ;
  4. pour les prompts à champs, l'indication **« Type in the highlighted fields to customize, then copy. »** (« Tapez dans les champs surlignés pour personnaliser, puis copiez. ») ;
  5. la boîte du prompt, précédée du caractère `❯`, avec des champs éditables préremplis par la valeur d'exemple (leur largeur s'adapte au texte, leur contenu est sélectionné au focus) et un bouton **Copy** (« Copier ») qui affiche **Copied** (« Copié ») pendant 1,6 seconde ;
  6. la rubrique **« Why this works »** (« Pourquoi ça marche ») ;
  7. le cas échéant, un lien **« Make it stick »** (« Pour pérenniser ») vers la page de documentation qui permet de rendre la pratique durable ;
  8. la mention **« From »** (« Source ») suivie du guide d'origine, avec lien.
- La copie passe par le presse-papiers ; si l'API n'est pas disponible, une zone de texte masquée et `document.execCommand('copy')` servent de repli. Un champ laissé vide est remplacé à la copie par sa valeur d'exemple (ou, à défaut, par le nom du champ).
- Les liens internes `/en/...` sont réécrits vers la langue de la page consultée.

### Étiquettes (tags)

Chaque prompt porte l'étiquette de tâche déduite de sa catégorie, plus zéro ou plusieurs étiquettes de rôle.

| Clé | Libellé affiché | Traduction | Type | Catégories associées |
| --- | --- | --- | --- | --- |
| `understand` | Understand | Comprendre | tâche | Onboard, Understand |
| `plan` | Plan | Planifier | tâche | Plan |
| `prototype` | Prototype | Prototyper | tâche | Prototype |
| `build` | Build | Construire | tâche | Implement |
| `test` | Test | Tester | tâche | Test |
| `refactor` | Refactor | Refactoriser | tâche | Refactor |
| `review` | Review | Relire | tâche | Review |
| `steer` | Steer | Piloter, réorienter | tâche | Steer |
| `debug` | Debug | Déboguer | tâche | Debug, Incident |
| `git` | Git | Git | tâche | Git |
| `release` | Release | Publier une version | tâche | Release |
| `data` | Data | Données | tâche et rôle | Data (et prompts au rôle `data`) |
| `automate` | Automate | Automatiser | tâche | Automate |
| `pm` | Product | Produit | rôle | — |
| `design` | Design | Design | rôle | — |
| `docs` | Docs | Documentation | rôle | — |
| `marketing` | Marketing | Marketing | rôle | — |
| `security` | Security | Sécurité | rôle | — |
| `ops` | On-call | Astreinte (opérations) | rôle | — |

### Phases du cycle de développement

| Clé | Libellé affiché | Traduction |
| --- | --- | --- |
| `discover` | Discover | Découvrir |
| `design` | Design | Concevoir |
| `build` | Build | Construire |
| `ship` | Ship | Livrer |
| `operate` | Operate | Exploiter |

### Catégories

| Clé | Libellé affiché | Traduction | Phase | Étiquette de tâche | Nombre de prompts |
| --- | --- | --- | --- | --- | --- |
| `Onboard` | Onboard | Prise en main | discover | `understand` | 1 |
| `Understand` | Understand | Comprendre | discover | `understand` | 6 |
| `Plan` | Plan | Planifier | design | `plan` | 4 |
| `Prototype` | Prototype | Prototyper | design | `prototype` | 2 |
| `Implement` | Implement | Implémenter | build | `build` | 7 |
| `Test` | Test | Tester | build | `test` | 3 |
| `Refactor` | Refactor | Refactoriser | build | `refactor` | 4 |
| `Review` | Review | Relire | build | `review` | 5 |
| `Steer` | Steer | Piloter | build | `steer` | 3 |
| `Git` | Git | Git | ship | `git` | 3 |
| `Release` | Release | Publier une version | ship | `release` | 2 |
| `Debug` | Debug | Déboguer | operate | `debug` | 3 |
| `Incident` | Incident | Incident | operate | `debug` | 3 |
| `Data` | Data | Données | operate | `data` | 2 |
| `Automate` | Automate | Automatiser | operate | `automate` | 4 |

### Sources des prompts

| Clé | Libellé affiché (« From … ») | Lien |
| --- | --- | --- |
| `workflows` | Common workflows | https://code.claude.com/docs/en/common-workflows |
| `teams` | How Anthropic teams use Claude Code | https://claude.com/blog/how-anthropic-teams-use-claude-code |
| `legal` | How Anthropic uses Claude in Legal | https://claude.com/blog/how-anthropic-uses-claude-legal |
| `cybersecurity` | How Anthropic uses Claude in Cybersecurity | https://claude.com/blog/how-anthropic-uses-claude-cybersecurity |
| `best-practices` | Best practices | https://code.claude.com/docs/en/best-practices |
| `ebook` | Scaling agentic coding guide | https://resources.anthropic.com/hubfs/Scaling%20agentic%20coding%20across%20your%20organization.pdf |

### Prérequis (« Needs »)

Certains prompts affichent un encadré **Needs** indiquant ce qui doit être disponible pour qu'ils fonctionnent.

| Clé | Prérequis affiché (traduit) | Prompts concernés |
| --- | --- | --- |
| `tracker` | Votre outil de suivi de tickets ajouté comme [connecteur claude.ai](https://code.claude.com/docs/en/mcp#use-mcp-servers-from-claude-ai) ou comme [serveur MCP](https://code.claude.com/docs/en/mcp). | Transformer une réunion en tickets ; Ouvrir une pull request à partir d'un ticket |
| `gh` | La [CLI gh](https://cli.github.com) authentifiée, ou GitHub ajouté comme [connecteur claude.ai](https://code.claude.com/docs/en/mcp#use-mcp-servers-from-claude-ai). | Traiter un ticket de bout en bout ; Relire une pull request |
| `browser` | Un moyen pour Claude d'afficher le résultat et d'en faire une capture d'écran. L'[application Desktop](https://code.claude.com/docs/en/desktop#preview-your-app) l'intègre. Dans le terminal, installez l'[extension Chrome](https://code.claude.com/docs/en/chrome) ou un serveur [MCP](https://code.claude.com/docs/en/mcp) Playwright. | Implémenter à partir d'une capture d'écran et s'auto-vérifier |
| `db` | Votre entrepôt de données ou votre stockage de logs ajouté comme [connecteur claude.ai](https://code.claude.com/docs/en/mcp#use-mcp-servers-from-claude-ai) ou comme [serveur MCP](https://code.claude.com/docs/en/mcp). | Interroger les logs en langage courant |

### Consignes de collage (« paste »)

Les prompts qui s'appuient sur une image, une sortie de commande ou un fichier affichent une consigne avant la boîte du prompt.

| Clé | Consigne affichée (traduite) | Prompts concernés |
| --- | --- | --- |
| `mockup` | Collez, glissez ou mentionnez avec `@` votre image de maquette, puis envoyez ceci : | Transformer une maquette en prototype fonctionnel |
| `design` | Collez, glissez ou mentionnez avec `@` votre image de design, puis envoyez ceci : | Implémenter à partir d'une capture d'écran et s'auto-vérifier |
| `screenshot` | Collez, glissez ou mentionnez avec `@` votre capture d'écran, puis envoyez ceci : | Diagnostiquer à partir d'une capture de console |
| `plan` | Collez d'abord la sortie de votre plan dans le prompt, puis envoyez ceci : | Relire des changements d'infrastructure avant de les appliquer |
| `error` | Collez d'abord la sortie d'erreur dans le prompt, puis envoyez ceci : | Corriger une erreur de build à la racine |
| `csv` | Glissez votre fichier dans le prompt, ou remplacez le chemin ci-dessous par une mention `@` de votre propre fichier : | Analyser un fichier de données ; Générer des variantes à partir de données de performance |

### Libellés de l'interface

| Clé | Texte anglais | Traduction |
| --- | --- | --- |
| `startHere` | Start here | Commencer ici |
| `startHereHeader` | Five prompts to try first | Cinq prompts à essayer en premier |
| `showAll` | Show all {n} prompts | Afficher les {n} prompts |
| `search` | Search prompts… | Rechercher des prompts… |
| `clear` | Clear | Effacer |
| `prompt` | prompt | prompt |
| `prompts` | prompts | prompts |
| `noMatch` | No prompts match | Aucun prompt ne correspond |
| `fillAndCopy` | Fill in and copy | Remplir et copier |
| `copyThis` | Copy this prompt | Copier ce prompt |
| `hintBefore` / `hintChip` / `hintAfter` | Type in the *highlighted* fields to customize, then copy. | Tapez dans les champs *surlignés* pour personnaliser, puis copiez. |
| `copy` | Copy | Copier |
| `copied` | Copied | Copié |
| `whyWorks` | Why this works | Pourquoi ça marche |
| `makeItStick` | Make it stick | Pour pérenniser |
| `from` | From | Source |
| `needsLabel` | Needs | Prérequis |

## Les cinq prompts pour commencer

| N° | Titre | Prompt | Phase · catégorie |
| --- | --- | --- | --- |
| 1 | S'orienter dans un nouveau dépôt (*Get oriented in a new repository*) | `give me an overview of this codebase: architecture, key directories, and how the pieces connect` | Discover · Onboard |
| 2 | Trouver où quelque chose se produit (*Find where something happens*) | `where do we {behavior}?` | Discover · Understand |
| 3 | Trouver et corriger un test en échec (*Find and fix a failing test*) | `the {test} test is failing, find out why and fix it` | Operate · Debug |
| 4 | Écrire des tests, les exécuter, corriger les échecs (*Write tests, run them, fix failures*) | `write tests for {path}, run them, and fix any failures` | Build · Test |
| 5 | Relire vos modifications avant de committer (*Review your changes before you commit*) | `review my uncommitted changes and flag anything that looks risky before I commit` | Build · Review |

## Catalogue complet des prompts

Les 52 prompts apparaissent ci-dessous dans l'ordre et les groupes « phase · catégorie » de la page. Pour chacun : le titre (traduit, puis original), l'identifiant, les étiquettes, le texte exact à copier (les `{champs}` sont à remplacer), une traduction indicative, les valeurs d'exemple des champs, les éventuels prérequis et consignes, l'explication « Why this works », le lien « Make it stick » et la source.

### Discover (Découvrir) · Onboard (Prise en main)

#### 1. S'orienter dans un nouveau dépôt — *Get oriented in a new repository*

- **Identifiant** : `get-oriented-in-a` · **Étiquettes** : Understand · **Start here** : n° 1

```text
give me an overview of this codebase: architecture, key directories, and how the pieces connect
```

- **Traduction** : donne-moi une vue d'ensemble de cette base de code : architecture, répertoires clés et façon dont les différentes parties s'articulent.
- **Pourquoi ça marche** : décrivez ce que vous voulez savoir, pas les fichiers à lire. Claude explore le projet de lui-même et renvoie un résumé de la façon dont l'ensemble s'articule.
- **Pour pérenniser** : lancez `/init` pour mettre en place `CLAUDE.md`, afin que Claude s'en souvienne à chaque session → [Memory](https://code.claude.com/docs/en/memory).
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

### Discover (Découvrir) · Understand (Comprendre)

#### 2. Expliquer du code inconnu — *Explain unfamiliar code*

- **Identifiant** : `explain-unfamiliar-code` · **Étiquettes** : Understand

```text
explain what {path} does and how data flows through it. write it up as {format}
```

- **Traduction** : explique ce que fait {path} et comment les données y circulent. rédige le résultat sous la forme {format}.
- **Valeurs d'exemple** : `{path}` = `src/scheduler/queue.ts` ; `{format}` = `an HTML page with a diagram, then open it in my browser` (une page HTML avec un diagramme, puis ouvre-la dans mon navigateur).
- **Pourquoi ça marche** : nommez le fichier et dites sous quel format vous voulez la réponse. Remplacez la page HTML par un diagramme, des puces ou tout autre format qui correspond à votre façon d'apprendre.
- **Pour pérenniser** : définissez un style de sortie (*output style*) pour que Claude explique toujours dans votre format préféré → [Output styles](https://code.claude.com/docs/en/output-styles).
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 3. Trouver où quelque chose se produit — *Find where something happens*

- **Identifiant** : `find-where-something-happens` · **Étiquettes** : Understand · **Start here** : n° 2

```text
where do we {behavior}?
```

- **Traduction** : où est-ce qu'on {behavior} ?
- **Valeur d'exemple** : `{behavior}` = `validate uploaded file types` (valide les types des fichiers téléversés).
- **Pourquoi ça marche** : cherchez par comportement plutôt que par nom de fichier. La recherche fonctionne même quand vous ne savez pas comment s'appelle le fichier ni dans quel répertoire il se trouve.
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 4. Vérifier ce qui casse avant de supprimer — *Check what breaks before you delete*

- **Identifiant** : `see-what-depends-on` · **Étiquettes** : Understand

```text
what would break if I deleted {target}?
```

- **Traduction** : qu'est-ce qui casserait si je supprimais {target} ?
- **Valeur d'exemple** : `{target}` = `the retryWithBackoff helper` (la fonction utilitaire `retryWithBackoff`).
- **Pourquoi ça marche** : demandez avant de supprimer quoi que ce soit. La liste des appelants et des effets en aval vous dit si vous avez affaire à un nettoyage d'une ligne ou à un changement qu'il faudra coordonner.
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 5. Retracer l'évolution du code — *Trace how code evolved*

- **Identifiant** : `trace-how-code-evolved` · **Étiquettes** : Understand

```text
look through the commit history of {path} and summarize how it evolved and why
```

- **Traduction** : parcours l'historique des commits de {path} et résume comment il a évolué et pourquoi.
- **Valeur d'exemple** : `{path}` = `internal/auth/session.go`.
- **Pourquoi ça marche** : orientez Claude vers l'historique des commits quand la question est « pourquoi » et non « quoi ». Claude lit le journal (*log*) et le *blame* de votre gestionnaire de versions, quel qu'il soit, et explique les décisions derrière l'implémentation actuelle.
- **Source** : [Best practices](https://code.claude.com/docs/en/best-practices).

#### 6. Délimiter un changement avant de commencer — *Scope a change before you start*

- **Identifiant** : `scope-a-change-before` · **Étiquettes** : Understand, Product, Design

```text
which files would I need to touch to {change}?
```

- **Traduction** : quels fichiers devrais-je modifier pour {change} ?
- **Valeur d'exemple** : `{change}` = `add a dark mode toggle to settings` (ajouter un interrupteur de mode sombre dans les réglages).
- **Pourquoi ça marche** : évaluez l'ampleur du travail avant de l'inscrire dans une feuille de route. La liste des fichiers vous dit si vous avez affaire à un seul composant ou à un changement transversal.
- **Source** : [How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code).

#### 7. Poser une question produit à la base de code — *Ask the codebase a product question*

- **Identifiant** : `ask-the-codebase-a` · **Étiquettes** : Understand, Product

```text
I am a {role}. walk me through what happens when a user {action}, from the UI down to the result
```

- **Traduction** : je suis {role}. explique-moi pas à pas ce qui se passe quand un utilisateur {action}, depuis l'interface jusqu'au résultat.
- **Valeurs d'exemple** : `{role}` = `PM` (chef de produit) ; `{action}` = `clicks Export to PDF` (clique sur Export to PDF).
- **Pourquoi ça marche** : indiquez votre rôle pour que la réponse soit formulée au bon niveau. Claude explique ce que fait réellement le produit à partir du code source, sans que vous ayez besoin de le lire.
- **Pour pérenniser** : définissez un style de sortie pour que Claude formule toujours ses réponses à ce niveau → [Output styles](https://code.claude.com/docs/en/output-styles).
- **Source** : [How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code).

### Design (Concevoir) · Plan (Planifier)

#### 8. Planifier un changement sur plusieurs fichiers avant de toucher au code — *Plan a multi-file change before touching code*

- **Identifiant** : `plan-a-multi-file` · **Étiquettes** : Plan, Product, Design

```text
plan how to refactor the {target} to {goal}. list the files you would change, but don't edit anything yet
```

- **Traduction** : planifie comment refactoriser le {target} pour {goal}. liste les fichiers que tu modifierais, mais ne modifie rien pour l'instant.
- **Valeurs d'exemple** : `{target}` = `payment module` (module de paiement) ; `{goal}` = `support multiple currencies` (prendre en charge plusieurs devises).
- **Pourquoi ça marche** : ajouter « don't edit yet » (ne modifie rien pour l'instant) sépare l'exploration des modifications : vous voyez l'approche avant que la moindre ligne de code ne bouge. Pour faire de « planifier d'abord » le comportement par défaut à chaque prompt, appuyez sur `Shift+Tab` pour passer en [mode plan](https://code.claude.com/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode).
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 9. Rédiger une spécification par interview — *Draft a spec by interview*

- **Identifiant** : `draft-a-spec-by` · **Étiquettes** : Plan, Product

```text
I want to build {feature}. interview me about implementation, UX, edge cases, and tradeoffs until we have covered everything, then write the spec to SPEC.md
```

- **Traduction** : je veux construire {feature}. interroge-moi sur l'implémentation, l'UX, les cas limites et les compromis jusqu'à ce que nous ayons tout couvert, puis écris la spécification dans SPEC.md.
- **Valeur d'exemple** : `{feature}` = `per-workspace rate limits` (des limites de débit par espace de travail).
- **Pourquoi ça marche** : demandez à être interviewé au lieu d'écrire la spécification vous-même. Claude vous pose des questions structurées jusqu'à ce que les exigences soient complètes, puis écrit le résultat dans un fichier.
- **Pour pérenniser** : enregistrez vos questions d'interview sous forme de skill `/spec` pour que chaque spécification démarre de la même façon → [Skills](https://code.claude.com/docs/en/skills).
- **Source** : [Best practices](https://code.claude.com/docs/en/best-practices).

#### 10. Transformer une réunion en tickets — *Turn a meeting into tickets*

- **Identifiant** : `turn-a-meeting-into` · **Étiquettes** : Plan, Product
- **Prérequis** : votre outil de suivi de tickets ajouté comme [connecteur claude.ai](https://code.claude.com/docs/en/mcp#use-mcp-servers-from-claude-ai) ou [serveur MCP](https://code.claude.com/docs/en/mcp).

```text
read {input} and write up the action items, then create a ticket in {tracker} for each one, with acceptance criteria
```

- **Traduction** : lis {input} et rédige la liste des actions à mener, puis crée un ticket dans {tracker} pour chacune, avec des critères d'acceptation.
- **Valeurs d'exemple** : `{input}` = `@meeting-notes.md` ; `{tracker}` = `our issue tracker` (notre outil de suivi de tickets).
- **Pourquoi ça marche** : sautez l'étape de transcription. Claude extrait les actions à mener de l'entrée non structurée et les écrit directement dans votre outil de suivi via [MCP](https://code.claude.com/docs/en/mcp) : vous relisez les tickets, pas la transcription.
- **Pour pérenniser** : enregistrez ceci comme skill `/tickets` → [Skills](https://code.claude.com/docs/en/skills).
- **Source** : [How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code).

#### 11. Recenser les cas limites avant de construire — *Map edge cases before building*

- **Identifiant** : `map-edge-cases-before` · **Étiquettes** : Plan, Design, Product

```text
list the error states, empty states, and edge cases for {feature} that the design needs to cover
```

- **Traduction** : liste les états d'erreur, les états vides et les cas limites de {feature} que le design doit couvrir.
- **Valeur d'exemple** : `{feature}` = `the file upload flow` (le parcours de téléversement de fichiers).
- **Pourquoi ça marche** : demandez ce qui manque, pas ce qui est là. Claude liste les états d'erreur, les états vides et les cas limites qu'un design centré sur le cas nominal (*happy path*) a tendance à oublier.
- **Source** : [How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code).

### Design (Concevoir) · Prototype (Prototyper)

#### 12. Transformer une maquette en prototype fonctionnel — *Turn a mockup into a working prototype*

- **Identifiant** : `turn-a-mockup-into` · **Étiquettes** : Prototype, Design, Product, Marketing
- **Consigne** : collez, glissez ou mentionnez avec `@` votre image de maquette, puis envoyez ceci :

```text
here is a mockup. build a working prototype I can click through, matching the layout and states shown
```

- **Traduction** : voici une maquette. construis un prototype fonctionnel dans lequel je peux cliquer, fidèle à la mise en page et aux états représentés.
- **Pourquoi ça marche** : un prototype cliquable répond à des questions qu'une maquette statique ne peut pas trancher. Remettez le code fonctionnel à l'équipe d'ingénierie au lieu d'expliquer les interactions dans un document.
- **Source** : [How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code).

#### 13. Implémenter à partir d'une capture d'écran et s'auto-vérifier — *Implement from a screenshot and self-check*

- **Identifiant** : `implement-from-a-screenshot` · **Étiquettes** : Prototype, Design
- **Prérequis** : un moyen pour Claude d'afficher le résultat et d'en faire une capture d'écran. L'[application Desktop](https://code.claude.com/docs/en/desktop#preview-your-app) l'intègre ; dans le terminal, installez l'[extension Chrome](https://code.claude.com/docs/en/chrome) ou un serveur [MCP](https://code.claude.com/docs/en/mcp) Playwright.
- **Consigne** : collez, glissez ou mentionnez avec `@` votre image de design, puis envoyez ceci :

```text
implement this design, then take a screenshot of the result, compare it to the original, and fix any differences
```

- **Traduction** : implémente ce design, puis fais une capture d'écran du résultat, compare-la à l'original et corrige toutes les différences.
- **Pourquoi ça marche** : cela donne à Claude une boucle de vérification : il affiche le rendu, le compare à l'image source et itère sans que vous ayez à signaler chaque écart.
- **Pour pérenniser** : utilisez `/goal` pour que Claude continue d'itérer jusqu'à ce que les captures correspondent → [Goal](https://code.claude.com/docs/en/goal).
- **Source** : [Best practices](https://code.claude.com/docs/en/best-practices).

### Build (Construire) · Implement (Implémenter)

#### 14. Suivre un modèle existant — *Follow an existing pattern*

- **Identifiant** : `follow-an-existing-pattern` · **Étiquettes** : Build

```text
look at how {example} is implemented to understand the pattern, then build {new} the same way
```

- **Traduction** : regarde comment {example} est implémenté pour comprendre le modèle, puis construis {new} de la même façon.
- **Valeurs d'exemple** : `{example}` = `the existing webhook handler` (le gestionnaire de webhooks existant) ; `{new}` = `a payments webhook handler` (un gestionnaire de webhooks de paiement).
- **Pourquoi ça marche** : montrez du code que vous appréciez déjà. Sans référence, Claude s'en tient aux bonnes pratiques générales. Avec une référence, il reproduit les conventions que votre base de code utilise réellement.
- **Pour pérenniser** : demandez à Claude d'écrire dans `CLAUDE.md` le modèle qu'il a suivi, pour que les sessions futures s'y conforment sans avoir besoin de la référence → [Memory](https://code.claude.com/docs/en/memory).
- **Source** : [Best practices](https://code.claude.com/docs/en/best-practices).

#### 15. Générer la documentation du code non documenté — *Generate docs for undocumented code*

- **Identifiant** : `generate-docs-for-code` · **Étiquettes** : Build, Docs

```text
find {scope} without {format} comments and add them, matching the style already used in the file
```

- **Traduction** : trouve {scope} sans commentaires {format} et ajoute-les, en reprenant le style déjà utilisé dans le fichier.
- **Valeurs d'exemple** : `{scope}` = `the public functions in src/auth/` (les fonctions publiques de `src/auth/`) ; `{format}` = `JSDoc`.
- **Pourquoi ça marche** : nommez le périmètre et le format. Claude trouve ce qui manque et reprend le style de commentaire déjà présent dans le fichier, si bien que la nouvelle documentation se lit comme le reste.
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 16. Ajouter une petite fonctionnalité bien définie — *Add a small, well-defined feature*

- **Identifiant** : `add-a-small-well` · **Étiquettes** : Build

```text
add a {endpoint} endpoint that returns {payload}
```

- **Traduction** : ajoute un endpoint {endpoint} qui renvoie {payload}.
- **Valeurs d'exemple** : `{endpoint}` = `/health` ; `{payload}` = `the app version and uptime` (la version de l'application et sa durée de fonctionnement).
- **Pourquoi ça marche** : indiquez les entrées et les sorties, pas la façon de construire. Claude trouve où se trouve le code similaire et ajoute le vôtre à côté.
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 17. Construire un petit outil interne à partir de zéro — *Build a small internal tool from scratch*

- **Identifiant** : `build-a-small-internal` · **Étiquettes** : Build, Product, Design, Marketing, Docs

```text
create a {tool} using HTML, CSS, and vanilla JavaScript, then open it in my browser
```

- **Traduction** : crée un {tool} en HTML, CSS et JavaScript pur (*vanilla*), puis ouvre-le dans mon navigateur.
- **Valeur d'exemple** : `{tool}` = `drag-and-drop Kanban board with three columns` (tableau Kanban à glisser-déposer avec trois colonnes).
- **Pourquoi ça marche** : vous n'avez besoin ni d'un projet, ni d'un framework, ni d'une étape de build. Décrivez l'outil et demandez à Claude de l'ouvrir pour le voir fonctionner immédiatement.
- **Source** : [How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code).

#### 18. Traiter un ticket de bout en bout — *Work an issue end to end*

- **Identifiant** : `work-an-issue-end` · **Étiquettes** : Build
- **Prérequis** : la [CLI gh](https://cli.github.com) authentifiée, ou GitHub ajouté comme [connecteur claude.ai](https://code.claude.com/docs/en/mcp#use-mcp-servers-from-claude-ai).

```text
read issue #{issue}, implement the fix, and run the tests
```

- **Traduction** : lis l'issue #{issue}, implémente le correctif et lance les tests.
- **Valeur d'exemple** : `{issue}` = `312`.
- **Pourquoi ça marche** : donnez le numéro du ticket, pas un résumé. Claude lit lui-même le ticket complet, donc les exigences que vous oublieriez de mentionner sont prises en compte, et il valide le changement avant de vous faire son retour.
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 19. Trouver et mettre à jour un texte dans toute la base de code — *Find and update copy across the codebase*

- **Identifiant** : `find-and-update-copy` · **Étiquettes** : Build, Design, Docs, Marketing

```text
find every place we say "{copy}" or a close variant, show me each one in context, then update them all to "{new}". leave tests and the changelog alone
```

- **Traduction** : trouve tous les endroits où nous disons « {copy} » ou une variante proche, montre-moi chacun dans son contexte, puis remplace-les tous par « {new} ». ne touche ni aux tests ni au changelog.
- **Valeurs d'exemple** : `{copy}` = `Sign up free` ; `{new}` = `Start free trial`.
- **Pourquoi ça marche** : demandez les variantes et dites quoi ignorer. Claude trouve des formulations qu'une recherche littérale manquerait et laisse intacts les fixtures de test et l'historique : vous ne relisez que le texte que les utilisateurs voient réellement.
- **Source** : [How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code).

#### 20. Rédiger un document à partir d'exemples passés — *Draft a document from past examples*

- **Identifiant** : `draft-from-past-examples` · **Étiquettes** : Build, Docs, Marketing, Product

```text
read the {examples} in {folder} to learn the structure and voice, then draft a new one for {topic}
```

- **Traduction** : lis les {examples} de {folder} pour en apprendre la structure et le ton, puis rédige-en un nouveau pour {topic}.
- **Valeurs d'exemple** : `{examples}` = `privacy impact assessments` (analyses d'impact relatives à la protection des données) ; `{folder}` = `legal/pia/` ; `{topic}` = `the new analytics integration` (la nouvelle intégration analytique).
- **Pourquoi ça marche** : montrez un dossier de travaux terminés au lieu de décrire votre style. Claude apprend la structure et le ton à partir de ce que vous avez déjà publié, si bien que le premier jet se lit comme l'un des vôtres.
- **Pour pérenniser** : enregistrez ce ton (*voice*) sous forme de skill pour que chaque brouillon parte de là → [Skills](https://code.claude.com/docs/en/skills).
- **Source** : [How Anthropic uses Claude in Legal](https://claude.com/blog/how-anthropic-uses-claude-legal).

### Build (Construire) · Test (Tester)

#### 21. Écrire des tests, les exécuter, corriger les échecs — *Write tests, run them, fix failures*

- **Identifiant** : `write-tests-run-them` · **Étiquettes** : Test · **Start here** : n° 4

```text
write tests for {path}, run them, and fix any failures
```

- **Traduction** : écris des tests pour {path}, exécute-les et corrige les éventuels échecs.
- **Valeur d'exemple** : `{path}` = `app/parsers/feed.py`.
- **Pourquoi ça marche** : demandez d'écrire, d'exécuter et de corriger en une seule fois, pour que Claude itère sans s'arrêter pour attendre vos instructions.
- **Pour pérenniser** : lancez `/init` pour que Claude apprenne automatiquement votre commande de test → [Memory](https://code.claude.com/docs/en/memory).
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 22. Piloter l'implémentation par les tests — *Drive implementation from tests*

- **Identifiant** : `drive-implementation-from-tests` · **Étiquettes** : Test

```text
write tests for {feature} first, then implement it until they pass
```

- **Traduction** : écris d'abord les tests de {feature}, puis implémente la fonctionnalité jusqu'à ce qu'ils passent.
- **Valeur d'exemple** : `{feature}` = `the password reset flow` (le parcours de réinitialisation du mot de passe).
- **Pourquoi ça marche** : c'est le développement piloté par les tests (TDD) : les tests définissent quand le travail est terminé, et Claude itère sur l'implémentation jusqu'à ce qu'ils passent.
- **Source** : [Scaling agentic coding guide](https://resources.anthropic.com/hubfs/Scaling%20agentic%20coding%20across%20your%20organization.pdf).

#### 23. Combler les lacunes d'après un rapport de couverture — *Fill gaps from a coverage report*

- **Identifiant** : `fill-gaps-from-a` · **Étiquettes** : Test

```text
read {report} and add tests for the lowest-covered files until each is above {target}%
```

- **Traduction** : lis {report} et ajoute des tests pour les fichiers les moins couverts jusqu'à ce que chacun dépasse {target} %.
- **Valeurs d'exemple** : `{report}` = `coverage/coverage-summary.json` ; `{target}` = `80`.
- **Pourquoi ça marche** : montrez le rapport de couverture au lieu de deviner ce qui n'est pas testé. Claude lit les chiffres réels et écrit des tests pour les fichiers qui en ont le plus besoin.
- **Pour pérenniser** : définissez ceci comme `/goal` pour que Claude continue d'écrire des tests jusqu'à atteindre l'objectif de couverture → [Goal](https://code.claude.com/docs/en/goal).
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

### Build (Construire) · Refactor (Refactoriser)

#### 24. Migrer un modèle dans toute la base de code — *Migrate a pattern across the codebase*

- **Identifiant** : `migrate-a-pattern-across` · **Étiquettes** : Refactor

```text
migrate everything from {from} to {to}: identify every place that needs to change, then make the changes
```

- **Traduction** : migre tout de {from} vers {to} : identifie chaque endroit à modifier, puis effectue les modifications.
- **Valeurs d'exemple** : `{from}` = `the old logging API` (l'ancienne API de journalisation) ; `{to}` = `the structured logger` (le logger structuré).
- **Pourquoi ça marche** : décrivez l'ancien modèle et le nouveau. Demander d'abord à Claude d'identifier chaque endroit fait figurer la liste des sites d'appel dans la réponse, ce qui vous permet de vérifier qu'aucun n'a été oublié. Pour une migration portant sur de nombreux fichiers, lancez [/batch](https://code.claude.com/docs/en/commands) : Claude découpe le travail en unités que vous approuvez, puis des sous-agents en arrière-plan effectuent les modifications.
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 25. Porter du code vers un autre langage — *Port code to another language*

- **Identifiant** : `port-code-between-languages` · **Étiquettes** : Refactor

```text
port {source} to {target}, keeping the same {keep}
```

- **Traduction** : porte {source} en {target}, en conservant les mêmes {keep}.
- **Valeurs d'exemple** : `{source}` = `this Python module` (ce module Python) ; `{target}` = `Rust` ; `{keep}` = `public API and test behavior` (API publique et comportement aux tests).
- **Pourquoi ça marche** : dites ce qui doit être préservé, pas seulement le langage cible. Nommer l'API ou le comportement qui doit rester identique donne à Claude un contrat par rapport auquel vérifier le portage.
- **Source** : [How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code).

#### 26. Optimiser par rapport à un objectif mesurable — *Optimize against a measurable target*

- **Identifiant** : `optimize-against-a-measurable` · **Étiquettes** : Refactor, Data

```text
optimize {target} to bring {metric} from {current} down to under {goal}
```

- **Traduction** : optimise {target} pour faire passer {metric} de {current} à moins de {goal}.
- **Valeurs d'exemple** : `{target}` = `the search query` (la requête de recherche) ; `{metric}` = `p95 latency` (la latence au 95e centile) ; `{current}` = `2s` ; `{goal}` = `500ms`.
- **Pourquoi ça marche** : indiquer la métrique et la cible donne à Claude une définition claire de « terminé ».
- **Pour pérenniser** : définissez ceci comme `/goal` pour que Claude continue de mesurer et d'itérer jusqu'à atteindre le chiffre → [Goal](https://code.claude.com/docs/en/goal).
- **Source** : [Scaling agentic coding guide](https://resources.anthropic.com/hubfs/Scaling%20agentic%20coding%20across%20your%20organization.pdf).

#### 27. Corriger un bug visuel précis — *Fix a precise visual bug*

- **Identifiant** : `fix-a-precise-visual` · **Étiquettes** : Refactor, Design

```text
the {element} extends {amount} beyond the {container} on {viewport}. fix it.
```

- **Traduction** : le {element} dépasse de {amount} au-delà de la {container} sur {viewport}. corrige-le.
- **Valeurs d'exemple** : `{element}` = `login button` (bouton de connexion) ; `{amount}` = `20px` ; `{container}` = `card border` (bordure de la carte) ; `{viewport}` = `mobile`.
- **Pourquoi ça marche** : un retour visuel précis donne un correctif précis. Indiquez l'élément exact, la mesure et la taille d'écran (*viewport*).
- **Pour pérenniser** : ajoutez un outil d'aperçu pour que Claude fasse lui-même la capture d'écran et vérifie le correctif → [Desktop : prévisualiser votre app](https://code.claude.com/docs/en/desktop#preview-your-app).
- **Source** : [Scaling agentic coding guide](https://resources.anthropic.com/hubfs/Scaling%20agentic%20coding%20across%20your%20organization.pdf).

### Build (Construire) · Review (Relire)

#### 28. Relire vos modifications avant de committer — *Review your changes before you commit*

- **Identifiant** : `review-your-changes-before` · **Étiquettes** : Review · **Start here** : n° 5

```text
review my uncommitted changes and flag anything that looks risky before I commit
```

- **Traduction** : relis mes modifications non committées et signale tout ce qui paraît risqué avant que je committe.
- **Pourquoi ça marche** : repérez les problèmes tant qu'ils demandent peu de travail à corriger. Claude lit les fichiers modifiés en entier, pas seulement les lignes du diff, et repère donc des problèmes qu'une relecture rapide par vous-même laisserait passer.
- **Pour pérenniser** : lancez `/code-review` pour obtenir la même vérification en une seule commande → [Commands](https://code.claude.com/docs/en/commands).
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 29. Relire une pull request — *Review a pull request*

- **Identifiant** : `review-a-pull-request` · **Étiquettes** : Review
- **Prérequis** : la [CLI gh](https://cli.github.com) authentifiée, ou GitHub ajouté comme [connecteur claude.ai](https://code.claude.com/docs/en/mcp#use-mcp-servers-from-claude-ai).

```text
review PR #{pr} and summarize what changed, then list any concerns
```

- **Traduction** : relis la PR #{pr} et résume ce qui a changé, puis liste tes éventuelles réserves.
- **Valeur d'exemple** : `{pr}` = `247`.
- **Pourquoi ça marche** : Claude relit avec toute la base de code en contexte, pas seulement le diff. Il lit le code modifié et ce qu'il appelle, et repère donc des problèmes qu'une revue limitée au diff manquerait.
- **Pour pérenniser** : lancez `/code-review <pr#>` en une seule commande, ou activez Code Review pour chaque PR → [Code Review](https://code.claude.com/docs/en/code-review).
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 30. Relire des changements d'infrastructure avant de les appliquer — *Review infrastructure changes before applying*

- **Identifiant** : `review-infrastructure-changes-before` · **Étiquettes** : Review, Security, On-call
- **Consigne** : collez d'abord la sortie de votre plan dans le prompt, puis envoyez ceci :

```text
here is my Terraform plan output. what is this going to do, and is anything here going to cause problems?
```

- **Traduction** : voici la sortie de mon plan Terraform. qu'est-ce que cela va faire, et y a-t-il quelque chose ici qui va poser problème ?
- **Pourquoi ça marche** : la sortie d'un plan est dense et difficile à parcourir. En la collant, vous obtenez un résumé en langage clair de ce qui va réellement changer, avant de l'appliquer.
- **Source** : [How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code).

#### 31. Lancer une revue de sécurité avec un sous-agent — *Run a security review with a subagent*

- **Identifiant** : `run-a-security-review` · **Étiquettes** : Review, Security

```text
use a subagent to review {path} for security issues and report what it finds
```

- **Traduction** : utilise un sous-agent pour examiner {path} à la recherche de problèmes de sécurité et rapporte ce qu'il trouve.
- **Valeur d'exemple** : `{path}` = `src/api/`.
- **Pourquoi ça marche** : un [sous-agent](https://code.claude.com/docs/en/sub-agents) mène l'audit dans sa propre fenêtre de contexte et renvoie un résumé, si bien qu'une longue revue de sécurité ne remplit pas votre session principale. Le sous-agent généraliste intégré (*general-purpose*) s'en charge sans configuration supplémentaire.
- **Pour pérenniser** : configurez un sous-agent dédié à la revue de sécurité que toute votre équipe peut utiliser → [Subagents](https://code.claude.com/docs/en/sub-agents).
- **Source** : [Best practices](https://code.claude.com/docs/en/best-practices).

#### 32. Repérer les problèmes avant la relecture formelle — *Catch issues before formal review*

- **Identifiant** : `review-content-before-sending` · **Étiquettes** : Review, Marketing, Docs

```text
review {file} for {concerns} and list anything I should fix before it goes to {reviewer}
```

- **Traduction** : examine {file} pour repérer {concerns} et liste tout ce que je devrais corriger avant de l'envoyer à {reviewer}.
- **Valeurs d'exemple** : `{file}` = `launch-post.md` ; `{concerns}` = `unsupported claims, missing attributions, and brand-guideline issues` (affirmations non étayées, attributions manquantes et écarts par rapport à la charte de marque) ; `{reviewer}` = `legal` (le service juridique).
- **Pourquoi ça marche** : obtenez une première passe avant qu'un humain n'y consacre du temps. Nommez les points à vérifier pour que la relecture soit ciblée, puis corrigez ce qu'elle trouve et envoyez un brouillon plus propre.
- **Pour pérenniser** : formalisez votre checklist de relecture sous forme de skill que toute votre équipe peut exécuter → [Skills](https://code.claude.com/docs/en/skills).
- **Source** : [How Anthropic uses Claude in Legal](https://claude.com/blog/how-anthropic-uses-claude-legal).

### Build (Construire) · Steer (Piloter)

#### 33. Rectifier une mauvaise approche — *Course-correct a wrong approach*

- **Identifiant** : `course-correct-a-wrong` · **Étiquettes** : Steer

```text
that is not right: {feedback}. try a different approach
```

- **Traduction** : ce n'est pas correct : {feedback}. essaie une autre approche.
- **Valeur d'exemple** : `{feedback}` = `the function signature needs to stay backward-compatible` (la signature de la fonction doit rester rétrocompatible).
- **Pourquoi ça marche** : nommez la contrainte que Claude a manquée, pas seulement le fait que c'est faux. Une raison précise donne à Claude une contrainte concrète à respecter lors du nouvel essai, au lieu de le laisser deviner à nouveau.
- **Pour pérenniser** : appuyez deux fois sur `Esc` pour ouvrir le menu de retour arrière (*rewind*) et restaurer le code et la conversation, afin que le nouvel essai reparte d'un état propre → [Checkpointing](https://code.claude.com/docs/en/checkpointing).
- **Source** : [Best practices](https://code.claude.com/docs/en/best-practices).

#### 34. Réduire la portée d'un changement — *Narrow the scope of a change*

- **Identifiant** : `narrow-the-scope-of` · **Étiquettes** : Steer

```text
that is too much. keep only the changes to {scope} and undo your other edits
```

- **Traduction** : c'est trop. garde uniquement les modifications de {scope} et annule tes autres changements.
- **Valeur d'exemple** : `{scope}` = `the validation logic in src/forms/` (la logique de validation de `src/forms/`).
- **Pourquoi ça marche** : quand la direction est bonne mais que le changement est allé trop loin, demandez à Claude d'en garder une partie plutôt que de tout annuler. Une limite explicite empêche un petit correctif de se transformer en refactorisation.
- **Source** : [Best practices](https://code.claude.com/docs/en/best-practices).

#### 35. Transformer une correction en règle — *Turn a correction into a rule*

- **Identifiant** : `turn-a-correction-into` · **Étiquettes** : Steer

```text
you keep {mistake}. add a rule to CLAUDE.md so this stops happening
```

- **Traduction** : tu n'arrêtes pas de {mistake}. ajoute une règle dans CLAUDE.md pour que cela cesse.
- **Valeur d'exemple** : `{mistake}` = `using default exports when this project uses named exports` (utiliser des exports par défaut alors que ce projet utilise des exports nommés).
- **Pourquoi ça marche** : une correction faite dans le chat n'est pas partagée avec votre équipe. Une règle dans le [CLAUDE.md](https://code.claude.com/docs/en/memory) du projet l'est dès que vous la committez, et Claude la lit au début de chaque session.
- **Pour pérenniser** : ouvrez `/memory` pour relire ce que Claude a écrit → [Memory](https://code.claude.com/docs/en/memory).
- **Source** : [Best practices](https://code.claude.com/docs/en/best-practices).

### Ship (Livrer) · Git

#### 36. Résoudre des conflits de fusion — *Resolve merge conflicts*

- **Identifiant** : `resolve-merge-conflicts` · **Étiquettes** : Git

```text
resolve the merge conflicts in this branch and explain what you kept from each side
```

- **Traduction** : résous les conflits de fusion de cette branche et explique ce que tu as gardé de chaque côté.
- **Pourquoi ça marche** : dites quel état vous voulez obtenir, pas quels marqueurs garder. Demander le raisonnement rend la fusion vérifiable au lieu d'en faire une boîte noire.
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 37. Committer avec un message généré — *Commit with a generated message*

- **Identifiant** : `commit-with-a-generated` · **Étiquettes** : Git

```text
commit these changes with a message that summarizes what I did
```

- **Traduction** : committe ces modifications avec un message qui résume ce que j'ai fait.
- **Pourquoi ça marche** : laissez Claude déduire le message à partir du diff. Il reprend le style de commit déjà en usage dans votre dépôt.
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 38. Ouvrir une pull request à partir d'un ticket — *Open a pull request from a ticket*

- **Identifiant** : `open-a-pull-request` · **Étiquettes** : Git
- **Prérequis** : votre outil de suivi de tickets ajouté comme [connecteur claude.ai](https://code.claude.com/docs/en/mcp#use-mcp-servers-from-claude-ai) ou [serveur MCP](https://code.claude.com/docs/en/mcp).

```text
find the ticket about {topic} in {tracker} and open a PR that implements it
```

- **Traduction** : trouve le ticket concernant {topic} dans {tracker} et ouvre une PR qui l'implémente.
- **Valeurs d'exemple** : `{topic}` = `the login timeout` (le délai d'expiration de la connexion) ; `{tracker}` = `our issue tracker` (notre outil de suivi de tickets).
- **Pourquoi ça marche** : évitez de jongler entre l'outil de suivi, l'éditeur et GitHub. Un seul prompt lit la spécification, effectue la modification et ouvre la PR.
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

### Ship (Livrer) · Release (Publier une version)

#### 39. Rédiger des notes de version à partir de l'historique git — *Draft release notes from git history*

- **Identifiant** : `draft-release-notes-from` · **Étiquettes** : Release, Product, Docs, Marketing

```text
compare {from} to {to} and draft release notes grouped by feature, fix, and breaking change
```

- **Traduction** : compare {from} à {to} et rédige des notes de version regroupées par fonctionnalité, correctif et changement cassant.
- **Valeurs d'exemple** : `{from}` = `v2.3.0` ; `{to}` = `v2.4.0`.
- **Pourquoi ça marche** : donnez deux points de référence et la structure voulue. Claude lit le journal des commits entre les deux et rédige un changelog que vous pouvez retoucher.
- **Pour pérenniser** : enregistrez ceci comme skill `/changelog` → [Skills](https://code.claude.com/docs/en/skills).
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 40. Écrire un workflow de CI — *Write a CI workflow*

- **Identifiant** : `write-a-ci-workflow` · **Étiquettes** : Release, On-call

```text
write a GitHub Actions workflow that {steps} on every push to {branch}
```

- **Traduction** : écris un workflow GitHub Actions qui {steps} à chaque push sur {branch}.
- **Valeurs d'exemple** : `{steps}` = `runs the tests and deploys to staging` (lance les tests et déploie en préproduction) ; `{branch}` = `main`.
- **Pourquoi ça marche** : décrivez quand il doit s'exécuter et ce qu'il doit faire ; le YAML est généré pour vous, adapté aux commandes de build et de test de votre projet.
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

### Operate (Exploiter) · Debug (Déboguer)

#### 41. Trouver et corriger un test en échec — *Find and fix a failing test*

- **Identifiant** : `find-and-fix-a` · **Étiquettes** : Debug · **Start here** : n° 3

```text
the {test} test is failing, find out why and fix it
```

- **Traduction** : le test {test} échoue, trouve pourquoi et corrige-le.
- **Valeur d'exemple** : `{test}` = `UserAuth`.
- **Pourquoi ça marche** : décrivez le symptôme ; inutile de savoir quel fichier est cassé. Claude exécute le test pour voir l'échec, remonte jusqu'au code source en cause et le corrige.
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 42. Enquêter sur une erreur signalée — *Investigate a reported error*

- **Identifiant** : `investigate-a-reported-error` · **Étiquettes** : Debug, On-call

```text
users are seeing {symptom} on {where}. investigate and tell me what is going on
```

- **Traduction** : des utilisateurs voient {symptom} sur {where}. enquête et dis-moi ce qui se passe.
- **Valeurs d'exemple** : `{symptom}` = `500 errors` (des erreurs 500) ; `{where}` = `/api/settings`.
- **Pourquoi ça marche** : décrivez le symptôme et l'endroit ; Claude lit le chemin de code concerné et remonte les causes probables. Collez les traces de pile (*stack traces*) ou les logs si vous en avez.
- **Pour pérenniser** : placez dans votre runbook un deeplink qui ouvre Claude avec ce prompt prérempli → [Pré-remplir des sessions](https://code.claude.com/docs/en/web-quickstart#pre-fill-sessions).
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 43. Corriger une erreur de build à la racine — *Fix a build error at the root*

- **Identifiant** : `fix-a-build-error` · **Étiquettes** : Debug, On-call
- **Consigne** : collez d'abord la sortie d'erreur dans le prompt, puis envoyez ceci :

```text
here is a build error. fix the root cause and verify the build succeeds
```

- **Traduction** : voici une erreur de build. corrige la cause racine et vérifie que le build réussit.
- **Pourquoi ça marche** : demander la cause racine et une vérification évite les correctifs superficiels qui masquent l'erreur sans la corriger.
- **Source** : [Best practices](https://code.claude.com/docs/en/best-practices).

### Operate (Exploiter) · Incident

#### 44. Enquêter sur un incident de production — *Investigate a production incident*

- **Identifiant** : `investigate-a-production-incident` · **Étiquettes** : Debug, On-call, Security

```text
{symptom}. check the logs, recent deploys, and config changes, then tell me the most likely cause
```

- **Traduction** : {symptom}. vérifie les logs, les déploiements récents et les changements de configuration, puis dis-moi la cause la plus probable.
- **Valeur d'exemple** : `{symptom}` = `the checkout endpoint started returning 500s an hour ago` (l'endpoint de paiement a commencé à renvoyer des erreurs 500 il y a une heure).
- **Pourquoi ça marche** : listez les sources d'indices à recouper, pas les étapes à suivre. Claude lit ensemble les logs, l'historique git et la configuration pour cerner la cause.
- **Pour pérenniser** : connectez votre outil de suivi des erreurs ou votre stockage de logs via MCP → [MCP](https://code.claude.com/docs/en/mcp).
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 45. Diagnostiquer à partir d'une capture de console — *Diagnose from a console screenshot*

- **Identifiant** : `diagnose-from-a-console` · **Étiquettes** : Debug, On-call, Data
- **Consigne** : collez, glissez ou mentionnez avec `@` votre capture d'écran, puis envoyez ceci :

```text
here is a screenshot of {console}. walk me through why {resource} is failing and give me the exact commands to fix it
```

- **Traduction** : voici une capture d'écran de {console}. explique-moi pas à pas pourquoi {resource} est en échec et donne-moi les commandes exactes pour le corriger.
- **Valeurs d'exemple** : `{console}` = `our Kubernetes dashboard` (notre tableau de bord Kubernetes) ; `{resource}` = `this pod` (ce pod).
- **Pourquoi ça marche** : les consoles cloud vous montrent le problème, mais pas les commandes pour le corriger. Claude lit la capture d'écran et traduit le tableau de bord en commandes `kubectl`, `gcloud` ou `aws` à exécuter.
- **Source** : [How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code).

#### 46. Interroger les logs en langage courant — *Query logs in plain English*

- **Identifiant** : `query-logs-in-plain` · **Étiquettes** : Debug, Security, On-call, Data
- **Prérequis** : votre entrepôt de données ou votre stockage de logs ajouté comme [connecteur claude.ai](https://code.claude.com/docs/en/mcp#use-mcp-servers-from-claude-ai) ou [serveur MCP](https://code.claude.com/docs/en/mcp).

```text
show me all {events} for {scope} over {timeframe}. write the query, run it, and tell me what stands out
```

- **Traduction** : montre-moi tous les {events} de {scope} sur {timeframe}. écris la requête, exécute-la et dis-moi ce qui ressort.
- **Valeurs d'exemple** : `{events}` = `failed logins` (échecs de connexion) ; `{scope}` = `the auth service` (le service d'authentification) ; `{timeframe}` = `the past 24 hours` (les dernières 24 heures).
- **Pourquoi ça marche** : posez la question au lieu d'écrire le SQL. Claude construit la requête, l'exécute sur vos logs connectés et affiche à la fois la requête et le résultat, pour que vous puissiez vérifier ce qui a été exécuté.
- **Source** : [How Anthropic uses Claude in Cybersecurity](https://claude.com/blog/how-anthropic-uses-claude-cybersecurity).

### Operate (Exploiter) · Data (Données)

#### 47. Analyser un fichier de données — *Analyze a data file*

- **Identifiant** : `analyze-a-data-file` · **Étiquettes** : Data, Product, Marketing (rôles `data`, `pm`, `marketing`)
- **Consigne** : glissez votre fichier dans le prompt, ou remplacez le chemin ci-dessous par une mention `@` de votre propre fichier :

```text
read {file}, summarize the key patterns, and write the results to {output}
```

- **Traduction** : lis {file}, résume les principales tendances et écris les résultats dans {output}.
- **Valeurs d'exemple** : `{file}` = `@reports/q1-signups.csv` ; `{output}` = `an HTML page with charts, then open it in my browser` (une page HTML avec des graphiques, puis ouvre-la dans mon navigateur).
- **Pourquoi ça marche** : une question ponctuelle ne nécessite pas un script ponctuel. Indiquez un fichier de votre dossier de projet : Claude le lit directement, trouve les tendances et écrit le résultat là où vous le demandez.
- **Pour pérenniser** : connectez la source de données via MCP au lieu d'exporter des fichiers → [MCP](https://code.claude.com/docs/en/mcp).
- **Source** : [How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code).

#### 48. Générer des variantes à partir de données de performance — *Generate variations from performance data*

- **Identifiant** : `generate-variations-from-performance` · **Étiquettes** : Data, Marketing (rôles `marketing`, `data`)
- **Consigne** : glissez votre fichier dans le prompt, ou remplacez le chemin ci-dessous par une mention `@` de votre propre fichier :

```text
read {file}, find the underperforming {items}, and generate {n} new variations that stay under {limit} characters
```

- **Traduction** : lis {file}, trouve les {items} les moins performants et génère {n} nouvelles variantes qui restent sous {limit} caractères.
- **Valeurs d'exemple** : `{file}` = `@ads-performance.csv` ; `{items}` = `headlines` (titres d'annonces) ; `{n}` = `20` ; `{limit}` = `90`.
- **Pourquoi ça marche** : énoncez la contrainte dès le début pour que la génération reste dans la limite. Claude lit les métriques, choisit ce qu'il faut remplacer et produit des alternatives conformes.
- **Pour pérenniser** : connectez la plateforme publicitaire via MCP au lieu d'exporter un fichier → [MCP](https://code.claude.com/docs/en/mcp).
- **Source** : [How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code).

### Operate (Exploiter) · Automate (Automatiser)

#### 49. Transformer une tâche récurrente en skill — *Turn a recurring task into a skill*

- **Identifiant** : `turn-a-recurring-task` · **Étiquettes** : Automate

```text
create a /{name} skill for this project that {steps}
```

- **Traduction** : crée pour ce projet une skill /{name} qui {steps}.
- **Valeurs d'exemple** : `{name}` = `ship` ; `{steps}` = `runs the linter and tests, then drafts a commit message` (lance le linter et les tests, puis rédige un message de commit).
- **Pourquoi ça marche** : nommez les étapes une fois, puis réutilisez-les comme une commande. Claude écrit une [skill](https://code.claude.com/docs/en/skills) que n'importe qui dans votre équipe peut exécuter.
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 50. Ajouter un hook pour un comportement répété — *Add a hook for repeat behavior*

- **Identifiant** : `add-a-hook-for` · **Étiquettes** : Automate

```text
write a hook that {action} after every {event}
```

- **Traduction** : écris un hook qui {action} après chaque {event}.
- **Valeurs d'exemple** : `{action}` = `runs prettier` (lance prettier) ; `{event}` = `edit to a .ts or .tsx file` (modification d'un fichier `.ts` ou `.tsx`).
- **Pourquoi ça marche** : les hooks rendent un comportement automatique, au lieu d'en faire quelque chose que vous devez penser à demander. Décrivez le déclencheur et l'action, et Claude écrit la configuration du [hook](https://code.claude.com/docs/en/hooks).
- **Source** : [Best practices](https://code.claude.com/docs/en/best-practices).

#### 51. Connecter un outil avec MCP — *Connect a tool with MCP*

- **Identifiant** : `connect-a-tool-with` · **Étiquettes** : Automate

```text
connect {server} via MCP so you can read its {data} directly
```

- **Traduction** : connecte {server} via MCP pour que tu puisses lire directement ses {data}.
- **Valeurs d'exemple** : `{server}` = `our error tracker` (notre outil de suivi des erreurs) ; `{data}` = `stack traces` (traces de pile).
- **Pourquoi ça marche** : connectez la source une fois au lieu de coller les données à chaque session. Une fois [MCP](https://code.claude.com/docs/en/mcp) configuré, Claude lit directement depuis l'outil quand vous l'interrogez à son sujet.
- **Source** : [Common workflows](https://code.claude.com/docs/en/common-workflows).

#### 52. Noter ce qu'il faut retenir pour la prochaine fois — *Capture what to remember for next time*

- **Identifiant** : `capture-what-to-remember` · **Étiquettes** : Automate, Product, Docs

```text
summarize what we did this session and suggest what to add to CLAUDE.md
```

- **Traduction** : résume ce que nous avons fait pendant cette session et suggère ce qu'il faudrait ajouter à CLAUDE.md.
- **Pourquoi ça marche** : demandez avant d'oublier. Claude sait ce qu'il a dû découvrir pendant cette session et propose des entrées pour [CLAUDE.md](https://code.claude.com/docs/en/memory), afin que la session suivante démarre avec ce contexte.
- **Source** : [How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code).

## Ce qui fait fonctionner ces prompts

Les prompts ci-dessus partagent quelques principes. Savoir les reconnaître vous aide à adapter n'importe lequel d'entre eux à votre propre tâche.

**Décrivez le résultat, pas les étapes.** Dites ce que vous voulez et laissez Claude trouver les fichiers. Le prompt ci-dessous fonctionne sans nommer un seul chemin de fichier.

```text
add rate limiting to the public API and make sure existing tests still pass
```

*(Ajoute une limitation de débit à l'API publique et assure-toi que les tests existants passent toujours.)*

**Donnez-lui un moyen de vérifier son propre travail.** Demandez dans le même prompt d'exécuter, de tester, de comparer ou de vérifier, pour que Claude itère au lieu de s'arrêter après une seule tentative. Pour vérifier le changement terminé sur l'application en cours d'exécution, lancez [`/verify`](https://code.claude.com/docs/en/skills#run-and-verify-your-app).

```text
write the migration, run it against the dev database, and confirm the schema matches
```

*(Écris la migration, exécute-la sur la base de données de développement et confirme que le schéma correspond.)*

**Montrez une référence.** Nommez un fichier, un test ou un modèle existant à imiter, pour que le nouveau code soit cohérent avec l'existant.

```text
add a settings page that follows the same layout as the profile page
```

*(Ajoute une page de réglages qui suit la même mise en page que la page de profil.)*

**Énoncez l'objectif mesurable.** Quand l'objectif porte sur la performance ou la couverture, donnez la métrique et le seuil pour que la fin du travail soit sans ambiguïté.

```text
get the bundle size under 200KB and show me what you removed
```

*(Fais passer la taille du bundle sous 200 Ko et montre-moi ce que tu as retiré.)*

**Donnez-lui l'artefact.** Collez directement dans le prompt les erreurs, logs, captures d'écran et sorties de plan, ou tapez `@` pour référencer un fichier. Claude lit la source elle-même plutôt que votre description de celle-ci.

```text
why is the build failing? @build.log
```

*(Pourquoi le build échoue-t-il ? `@build.log`)*

**Dites comment vous voulez la réponse.** Nommez le format, la longueur ou le public visé pour que l'explication corresponde à l'usage que vous en ferez. Pour faire d'un format le comportement par défaut de toutes les réponses, définissez un [style de sortie](https://code.claude.com/docs/en/output-styles).

```text
explain how the payment retry logic works as an HTML page with a diagram, then open it in my browser
```

*(Explique comment fonctionne la logique de nouvelle tentative de paiement sous forme de page HTML avec un diagramme, puis ouvre-la dans mon navigateur.)*

Pour en savoir plus sur chacun de ces principes, voir les [bonnes pratiques](https://code.claude.com/docs/en/best-practices).

## D'où viennent ces prompts

Ces prompts s'appuient sur des principes tirés de ressources publiées par Anthropic. Chaque carte renvoie vers sa source :

* [Common workflows](https://code.claude.com/docs/en/common-workflows) : guides pas à pas pour les tâches essentielles.
* [Best practices](https://code.claude.com/docs/en/best-practices) : principes de rédaction des prompts et configuration de projet.
* [How Anthropic teams use Claude Code](https://claude.com/blog/how-anthropic-teams-use-claude-code) : workflows réels des équipes ingénierie, produit, design et données, avec des études approfondies sur le [juridique](https://claude.com/blog/how-anthropic-uses-claude-legal), le [marketing](https://claude.com/blog/how-anthropic-uses-claude-marketing) et la [cybersécurité](https://claude.com/blog/how-anthropic-uses-claude-cybersecurity).
* [Scaling agentic coding guide](https://resources.anthropic.com/hubfs/Scaling%20agentic%20coding%20across%20your%20organization.pdf) : le guide d'adoption en entreprise.

Pour des démonstrations vidéo de ces principes, voir le cours gratuit [Claude Code in Action](https://academy.claude.com/courses/claude-code-in-action) sur [Claude Academy](https://academy.claude.com/).

## Ressources associées

Les prompts de cette page sont des points de départ. Une fois qu'un prompt fonctionne pour votre projet, l'étape suivante consiste à le rendre reproductible : enregistrez-le comme [skill](https://code.claude.com/docs/en/skills) pour que n'importe qui dans votre équipe puisse l'exécuter comme une `/commande`, et consignez dans [CLAUDE.md](https://code.claude.com/docs/en/memory) les conventions que Claude a apprises, pour que chaque session démarre avec ce contexte au lieu que Claude doive les réapprendre. Pour les changements plus importants ou plus risqués, le [mode plan](https://code.claude.com/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode) vous montre la liste des fichiers avant toute modification.

Si vous déployez Claude Code au sein d'une équipe, consultez la page [administration](https://code.claude.com/docs/en/admin-setup) pour les réglages gérés (*managed settings*) et les politiques, et la page [coûts et utilisation](https://code.claude.com/docs/en/costs) pour savoir comment ce travail est facturé selon votre forfait.
