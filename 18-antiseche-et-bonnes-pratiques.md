# 18 — Antisèche et bonnes pratiques

[← Sommaire](README.md)

## Antisèche : l'essentiel sur une page

### Lancer

```bash
claude                     # session interactive
claude "tâche"             # avec une première demande
claude -c                  # reprendre la dernière conversation
claude -r                  # choisir une conversation
claude -p "question"       # réponse unique (scripts, CI)
claude -w nom              # dans un worktree isolé
claude --bg "tâche"        # en arrière-plan
claude agents              # tableau de bord des sessions de fond
claude --cloud "tâche"     # session dans le cloud
claude --permission-mode plan
claude --model opus --effort high
```

### Taper dans le prompt

| Saisie | Effet |
|--------|-------|
| `/` | Commandes et skills |
| `!commande` | Exécuter une commande shell |
| `@fichier` | Citer un fichier (ou un agent, une ressource MCP) |
| `?` | Aide des raccourcis |
| `\` + `Entrée`, `Ctrl+J`, `Shift+Entrée` | Nouvelle ligne |
| `ultrathink` | Réfléchir plus fort pour ce tour |
| `ultracode` | Faire planifier un workflow multi-agents |

### Touches indispensables

| Touche | Effet |
|--------|-------|
| `Shift+Tab` | Changer de mode de permission |
| `Esc` | Interrompre Claude |
| `Esc Esc` | Revenir en arrière (`/rewind`) |
| `Ctrl+C` | Annuler / effacer / quitter |
| `Ctrl+O` | Transcription détaillée |
| `Ctrl+R` | Rechercher dans l'historique |
| `Ctrl+B` | Mettre une tâche en arrière-plan |
| `Ctrl+T` | Liste de tâches de Claude |
| `Ctrl+G` | Éditer le prompt dans votre éditeur |
| `Ctrl+V` | Coller une image |
| `Ctrl+S` | Mettre le prompt de côté |
| `Alt+P` / `Alt+T` / `Alt+O` | Modèle / réflexion / mode rapide |
| `←` (prompt vide) | Envoyer la session en arrière-plan, ouvrir l'agent view |

### Commandes les plus utiles

| Besoin | Commande |
|--------|----------|
| Démarrer un projet | `/init`, `/memory` |
| Planifier | `/plan` |
| Choisir modèle / effort | `/model`, `/effort`, `/fast` |
| Contexte | `/context`, `/compact`, `/clear`, `/btw` |
| Revenir en arrière | `/rewind` |
| Voir les changements | `/diff` |
| Relire | `/code-review`, `/simplify`, `/security-review`, `/ultrareview` |
| Vérifier en vrai | `/verify`, `/run` |
| Paralléliser | `/batch`, `/subtask`, `/fork`, `/background`, `/tasks` |
| Automatiser | `/loop`, `/schedule`, `/goal` |
| Configurer | `/config`, `/permissions`, `/hooks`, `/mcp`, `/plugin`, `/skills`, `/statusline`, `/keybindings`, `/theme` |
| Diagnostiquer | `/doctor`, `/debug`, `/status`, `/usage` |
| Continuer ailleurs | `/desktop`, `/remote-control`, `/teleport`, `/mobile` |
| Apprendre | `/help`, `/powerup`, `/release-notes` |

### Fichiers clés

| Fichier | Rôle |
|---------|------|
| `CLAUDE.md` / `AGENTS.md` | Instructions du projet |
| `.claude/settings.json` | Permissions, hooks, plugins de l'équipe |
| `.claude/settings.local.json` | Vos réglages perso pour le projet |
| `.claude/skills/<nom>/SKILL.md` | Skills du projet |
| `.claude/agents/<nom>.md` | Sous-agents du projet |
| `.claude/rules/*.md` | Règles thématiques (éventuellement par chemin) |
| `.mcp.json` | Serveurs MCP de l'équipe |
| `~/.claude/…` | Équivalents personnels |

## Bonnes pratiques (d'après la doc officielle)

### 1. Donner à Claude un moyen de vérifier son travail
C'est le conseil le plus important : tests, script, capture d'écran, sortie attendue. « Écris une fonction de validation d'e-mail ; cas de test : … ; lance les tests après l'implémentation. » Utilisez `/verify` et `/run` pour vérifier dans l'application réelle.

### 2. Explorer, puis planifier, puis coder
1. **Mode plan** : « lis `src/auth` et explique comment les sessions sont gérées ».
2. **Plan** : « je veux ajouter OAuth Google ; quels fichiers changer ? fais un plan ». (`Ctrl+G` pour éditer le plan.)
3. **Implémenter** en sortant du mode plan, en vérifiant au fur et à mesure.
4. **Committer** et ouvrir une PR.

Pour une petite correction évidente, sautez le plan.

### 3. Être précis dans les demandes
- Délimiter : quel fichier, quel scénario, quelles contraintes.
- Pointer les sources : `@fichier`, historique git, tests existants à imiter.
- Fournir du contenu riche : images (coller), URL, données via `cat fichier | claude`, logs.
- Pour une grosse fonctionnalité : **laissez Claude vous interviewer** (« interroge-moi en détail avec l'outil AskUserQuestion, puis écris une spec complète dans SPEC.md »), puis exécutez la spec dans une session neuve.

### 4. Configurer son environnement
- `CLAUDE.md` court et concret (commandes, conventions, pièges du projet) ; élaguez ce qui est inutile.
- Permissions : pré-autoriser les commandes sûres (`/permissions`, `/fewer-permission-prompts`), mode auto, ou sandbox.
- Installer les CLI utiles (`gh`, `aws`, `gcloud`…) : Claude sait s'en servir.
- MCP pour les services externes, hooks pour ce qui doit arriver **à chaque fois**, skills pour les procédures, sous-agents pour les tâches isolées, plugins pour tout partager.

### 5. Gérer la session
- **Corriger tôt** : `Esc` pour arrêter, `Esc Esc` pour revenir en arrière, « annule ça ».
- **Gérer le contexte agressivement** : `/clear` entre deux tâches sans rapport, `/compact` avec consignes, `/btw` pour les questions annexes.
- **Déléguer l'investigation** à des sous-agents.
- **Reprendre** au lieu de tout réexpliquer : `claude -c`, `/resume`, sessions nommées.

### 6. Automatiser et passer à l'échelle
- `claude -p` dans les scripts et la CI, avec `--output-format json`.
- Plusieurs sessions en parallèle (worktrees, agent view, Desktop) : par exemple une qui écrit, une qui relit.
- Répartir sur de nombreux fichiers (`/batch`, boucle de `claude -p`, workflows).
- Mode auto pour les longues tâches ; **étape de revue adversariale** (un second Claude relit avec un regard neuf, ou `/code-review`).

### 7. Éviter les pièges classiques

| Piège | Remède |
|-------|--------|
| Session « fourre-tout » (tâches sans rapport mélangées) | `/clear` entre les tâches |
| Corriger encore et encore | Après 2 échecs, `/clear` et un meilleur prompt de départ |
| `CLAUDE.md` trop long | Élaguer ; transformer les règles critiques en hooks |
| Faire confiance sans vérifier | Toujours des tests, scripts ou captures |
| Exploration infinie | Cadrer la recherche ou la confier à un sous-agent |

## Recettes de prompts

| Objectif | Exemple de prompt |
|----------|-------------------|
| Découvrir un code | « Donne-moi une vue d'ensemble de ce code », « explique les principaux modèles d'architecture », « trouve les fichiers qui gèrent l'authentification » |
| Corriger un bug | « J'ai cette erreur en lançant `npm test` : … Trouve la cause et corrige-la, puis relance les tests » |
| Refactoriser | « Trouve l'usage d'API dépréciées dans le code et propose une modernisation, en gardant le comportement » |
| Tests | « Trouve les fonctions non testées dans `NotificationsService.swift`, écris des tests pour les cas limites, lance-les » |
| Pull request | « Résume mes changements », « crée une PR » |
| Documentation | « Trouve les fonctions sans docstring dans le module auth et documente-les » |
| Images | Coller une capture : « Implémente cette maquette », « quel est ce bug visuel ? » |
| Questions sur Claude Code | « Peux-tu créer des pull requests ? », « comment configurer un hook ? » (l'agent `claude-code-guide` répond) |

## Pour aller plus loin

- Documentation officielle : https://code.claude.com/docs (index complet : https://code.claude.com/docs/llms.txt)
- Nouveautés hebdomadaires : https://code.claude.com/docs/en/whats-new — ou `/release-notes`
- Dans Claude Code : `/powerup`, `/help`, ou demandez simplement à Claude.
