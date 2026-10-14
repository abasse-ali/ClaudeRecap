# 09 — Sous-agents et travail en parallèle

[← Sommaire](README.md)

Claude Code peut répartir le travail entre plusieurs « cerveaux ». Vue d'ensemble :

| Approche | Ce que c'est | Qui décide de la suite | Échelle | Idéal pour |
|----------|--------------|------------------------|---------|------------|
| **Sous-agent** | Un assistant que Claude lance, avec son propre contexte | Claude, tour par tour | Quelques tâches | Recherche, exploration, tâches isolées |
| **Session en arrière-plan** (agent view) | Une session complète qui tourne seule | Vous | Autant que vous voulez | Plusieurs tâches indépendantes en parallèle |
| **Équipe d'agents** | Un chef + des coéquipiers qui se parlent | Le chef | Une poignée | Travail collaboratif complexe |
| **Workflow dynamique** | Un script qui orchestre des dizaines/centaines d'agents | Le script | Des dizaines à des centaines | Audits, migrations massives, recherches recoupées |
| **`/batch`** | Changement découpé en unités, un agent + un worktree par unité | Claude (plan validé par vous) | 5 à 30 unités | Migrations à l'échelle du code |
| **Worktrees** | Copies isolées du dépôt git | — | — | Éviter les conflits entre sessions parallèles |

## Les sous-agents

Un sous-agent a **sa propre fenêtre de contexte**, son prompt système, ses outils et son modèle. Il travaille puis renvoie un **résumé** : votre conversation principale reste légère.

### Sous-agents intégrés

| Agent | Rôle | Outils |
|-------|------|--------|
| **Explore** | Recherche rapide dans le code (niveaux *quick*, *medium*, *very thorough*) | Lecture seule |
| **Plan** | Recherche pendant le mode plan | Lecture seule |
| **general-purpose** | Tâches complexes en plusieurs étapes, exploration + modification | Tous |
| **claude** | Agent « fourre-tout », agent par défaut des sessions d'arrière-plan | Tous |
| **statusline-setup** | Configure la barre d'état (`/statusline`) | Lecture/édition |
| **claude-code-guide** | Répond aux questions sur Claude Code, l'Agent SDK et l'API | Lecture, web |

### Créer un sous-agent

Le plus simple : demandez à Claude (« crée un sous-agent code-reviewer qui… »). Ou écrivez un fichier Markdown :

```markdown
---
name: code-reviewer
description: Relit le code après chaque modification importante. À utiliser de façon proactive.
tools: Read, Grep, Glob, Bash
model: sonnet
permissionMode: default
memory: project
color: purple
---

Tu es un relecteur de code senior. Quand on t'appelle :
1. Lance `git diff` pour voir les changements récents.
2. Vérifie lisibilité, gestion d'erreurs, sécurité, tests.
3. Classe tes remarques : critique / à corriger / suggestion.
```

| Emplacement | Portée | Priorité |
|-------------|--------|:--------:|
| Réglages gérés | Organisation | 1 |
| Flag `--agents '{…}'` | Session | 2 |
| `.claude/agents/` | Projet | 3 |
| `~/.claude/agents/` | Tous vos projets | 4 |
| `agents/` d'un plugin | Où le plugin est actif | 5 |

**Champs du frontmatter** : `name`, `description` (obligatoires), `tools`, `disallowedTools`, `model` (`sonnet`, `opus`, `haiku`, `fable`, ID complet, `inherit`), `permissionMode`, `maxTurns`, `skills` (préchargés), `mcpServers` (serveurs MCP propres à l'agent), `hooks`, `memory` (`user`/`project`/`local` → mémoire persistante dans `.claude/agent-memory/…`), `background`, `omitClaudeMd`, `effort`, `isolation: worktree`, `color`, `initialPrompt`, `autoCompactWindow`.

### Utiliser un sous-agent

| Méthode | Exemple |
|---------|---------|
| Laisser Claude déléguer | Claude choisit d'après la `description` |
| Le nommer en langage naturel | « Utilise le sous-agent test-runner pour corriger les tests » |
| **@-mention** (garantit l'agent) | `@"code-reviewer (agent)" regarde les changements d'auth` ou `@agent-code-reviewer` |
| Toute la session comme cet agent | `claude --agent code-reviewer` ou `"agent": "code-reviewer"` dans les réglages |
| Forker la conversation | `/subtask <tâche>` : un sous-agent qui hérite de toute la conversation |

- **Premier plan / arrière-plan** : en session interactive, les sous-agents tournent par défaut **en arrière-plan** (vous continuez à travailler) ; leurs demandes de permission remontent dans la session principale. `Ctrl+B` envoie un sous-agent en arrière-plan.
- **Suivre** : le panneau sous le prompt liste les agents en cours (flèches + `Entrée` pour ouvrir, `x` pour arrêter) ; `/tasks` ; `Ctrl+X Ctrl+K` arrête tous les agents d'arrière-plan.
- **Reprendre** un sous-agent terminé pour continuer son travail (par son ID ou son nom).
- Les sous-agents peuvent lancer leurs propres sous-agents (avec une limite de profondeur).
- Interdire un type : `"deny": ["Agent(Explore)"]` ; tous : `"deny": ["Agent"]`.

## Sessions en arrière-plan et agent view

L'**agent view** affiche toutes vos sessions Claude Code d'arrière-plan sur un seul écran.

- **Ouvrir** : `claude agents` (ou `←` sur un prompt vide depuis une session).
- **Lancer une session d'arrière-plan** : taper la tâche dans l'agent view, `claude --bg "enquête sur le test instable"`, `/background` (détache la session actuelle, alias `/bg`), `/fork` (copie la conversation dans une nouvelle session de fond).
- **États** : en cours (animé), **attend une réponse** (jaune), inactif (grisé), terminé (vert), échec (rouge), arrêté (gris). Chaque ligne a un résumé d'une ligne et l'état de la PR éventuelle (jaune/vert/violet fusionnée/gris).
- **Isolation** : chaque session modifie le code dans son propre worktree.

| Touche (agent view) | Action |
|---------------------|--------|
| `Entrée` / `→` | S'attacher à la session sélectionnée |
| `Espace` | Aperçu (*peek*) et réponse rapide |
| `Ctrl+Entrée` | Lancer et s'attacher immédiatement |
| `Tab` | Parcourir les sous-agents (input vide) ou appliquer une suggestion |
| `Ctrl+S` | Grouper par état / par dossier |
| `Ctrl+T` | Épingler |
| `Ctrl+F` | Chercher par nom |
| `Ctrl+R` | Renommer |
| `Ctrl+G` | Éditer le prompt dans l'éditeur externe |
| `Ctrl+X` | Arrêter (2× pour supprimer) |
| `Esc` / `Ctrl+C` | Fermer / quitter |
| `?` | Aide |

Filtres dans la saisie : `a:<agent>`, `s:<état>` (`s:blocked` = tout ce qui vous attend), `n:<texte>`, `o:<texte>`. Préfixes de lancement : `<agent> <prompt>`, `@<agent>`, `@<dépôt>`, `/<commande>`, `! <commande shell>`.

Depuis le shell : `claude attach`, `claude logs`, `claude stop`, `claude respawn`, `claude rm`, `claude agents --json`.

## Équipes d'agents (agent teams)

Un **chef d'équipe** (votre session) lance des **coéquipiers**, chacun étant une session Claude Code complète et indépendante. Ils partagent une **liste de tâches**, se répartissent le travail et **s'envoient des messages**.

- **Activer** (expérimental) : `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`.
- **Démarrer** : décrivez l'équipe en langage naturel (« crée une équipe de 3 : un pour le front, un pour l'API, un pour les tests »).
- **Affichage** : `--teammate-mode in-process` (défaut, dans le même terminal), `tmux`, `iterm2` (panneaux séparés) ou `auto`.
- Vous pouvez écrire directement à un coéquipier, lui demander de planifier avant d'agir, choisir son modèle.
- Plus coûteux que les sous-agents (chaque coéquipier est une instance complète) ; disponible dans le CLI (pas dans l'app Desktop).

## Workflows dynamiques

Un **workflow** est un script que Claude écrit et que le moteur exécute : il orchestre **des dizaines à des centaines de sous-agents** en arrière-plan, par phases, et renvoie un résultat consolidé. Reprenable et réutilisable.

- Essayer : `/deep-research <question>` (workflow intégré).
- Demander un workflow : écrire **`ultracode`** dans le prompt, ou « utilise un workflow » ; ou activer ultracode pour la session (`/effort ultracode`).
- Suivre : `/workflows` → `↑/↓` choisir, `Entrée`/`→` détailler, `Esc`/`←` revenir, `f` filtrer, `p` pause/reprise, `x` arrêter, `r` relancer un agent, `s` **sauvegarder** le script comme commande réutilisable.
- Référence pour écrire : `/workflow-authoring`.

## Worktrees git

Un worktree est une copie de travail séparée du dépôt (sa propre branche, ses propres fichiers) : plusieurs sessions peuvent modifier le code sans se marcher dessus.

- `claude -w feature-auth` → crée `.claude/worktrees/feature-auth/` sur la branche `worktree-feature-auth` et y démarre. `claude -w '#123'` part d'une PR.
- En session : « travaille dans un worktree » (outil `EnterWorktree`).
- Sous-agents isolés : « utilise des worktrees pour tes agents », ou `isolation: worktree` dans la définition.
- `.worktreeinclude` : copier des fichiers ignorés par git (ex. `.env`) dans les nouveaux worktrees.
- Réglages `worktree.baseRef`, `worktree.location`, `worktree.sparsePaths`… ; à la sortie, Claude vérifie s'il y a du travail non sauvegardé avant de supprimer le worktree.
- Autres gestionnaires de versions : hooks `WorktreeCreate` / `WorktreeRemove`.

## Messagerie entre sessions

Claude peut **lister et écrire à vos autres sessions** Claude Code (même machine, autres machines via Remote Control, sessions cloud).

- « Dis à la session paiements que le schéma a changé » ; « préviens-moi quand la session migration a fini ».
- `/list-agents` (alias `/peers`) liste les sessions joignables ; `@` + lettres propose vos sessions ouvertes.
- Les messages arrivent entre deux appels d'outils (jamais d'interruption) ; sur la même machine ils passent par un socket local, pas par les serveurs d'Anthropic.
- Contrôle des messages entrants : réglage `crossSessionInbound` ; `isolatePeerMachines`.
