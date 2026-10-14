# 08 — Les skills

[← Sommaire](README.md)

Un **skill** est un dossier contenant un fichier `SKILL.md` : des instructions (et éventuellement des scripts, modèles, docs de référence) que Claude charge **quand c'est pertinent** ou quand vous tapez **`/nom-du-skill`**. C'est le moyen principal d'ajouter vos propres commandes et workflows réutilisables.

- Seules la **description** du skill est en permanence dans le contexte ; le contenu complet se charge à l'invocation.
- Vous pouvez l'invoquer (`/deploy staging`), et Claude peut l'invoquer seul quand la demande correspond à sa description.
- `/skills` liste les skills (`t` pour trier par coût en jetons, `Espace`/`Entrée` pour changer leur visibilité) ; `/skill-doctor` montre leur coût et leur fréquence d'utilisation ; `/reload-skills` recharge les dossiers.

## Les skills intégrés (bundled)

| Skill | Ce qu'il fait |
|-------|---------------|
| `/code-review` (alias `/review`) | Relit le diff, une PR, une branche ou un chemin pour trouver des **bugs** ; niveaux `low`→`max`, `ultra` (cloud) ; `--fix` applique, `--comment` publie sur la PR |
| `/simplify` | 4 agents parallèles : réutilisation, simplification, efficacité, niveau d'abstraction ; applique les corrections (qualité, pas de chasse aux bugs) |
| `/batch <instruction>` | Gros changement découpé en 5–30 unités, chacune confiée à un sous-agent dans son propre worktree, qui teste et publie |
| `/debug [description]` | Active le journal de débogage et analyse les problèmes de la session |
| `/doctor` | Bilan de santé (installation, extensions, `CLAUDE.md`, permissions) avec corrections après accord ; `prompt-audit` |
| `/loop [intervalle] [prompt]` | Répète un prompt à intervalle fixe ou auto-rythmé ; sans prompt, lance un prompt de maintenance (ou votre `loop.md`) |
| `/run` | Lance et pilote l'application pour voir un changement fonctionner |
| `/verify` | Construit, lance l'app et observe le résultat pour vérifier un changement (pas seulement les tests) |
| `/run-skill-generator` | Écrit un skill de projet qui apprend à `/run` et `/verify` à lancer votre app |
| `/fewer-permission-prompts` | Ajoute une liste blanche de commandes en lecture seule à partir de vos transcriptions |
| `/update-config` | Modifie le bon `settings.json` à partir d'une demande en langage naturel |
| `/claude-api` | Référence API Claude / Managed Agents ; sous-commandes `migrate`, `upgrade`, `managed-agents-onboard`, `prompt-audit`, `cost-optimize`, `build-eval`, `hillclimb`… S'active seul quand le code importe `anthropic` |
| `/claude-in-chrome [tâche]` | Fait une tâche dans votre navigateur Chrome |
| `/dataviz` | Conception de graphiques et tableaux de bord (palette accessible validée par script) |
| `/design [brief]` | Maquettes UI sur un canevas Claude Design (artifact) |
| `/design-sync` | Importe votre design system React dans Claude Design |
| `/slides [brief]` | Présentation Claude Slides (artifact) |
| `/artifact-capabilities`, `/artifact-diagramming` | Références pour construire des artifacts |
| `/workflow-authoring` | Référence pour écrire des workflows dynamiques |
| `/plugin-authoring` | Référence pour écrire des mods (skill d'un plugin intégré) |
| `/deep-research <question>` | **Workflow** intégré : recherche web en éventail et rapport cité |

> Si une session contient un skill nommé `verify` ou `simplify`, Claude l'exécute automatiquement juste avant chaque commit (hors changements de docs/tests).

Désactiver les skills intégrés : `"disableBundledSkills": true` ; masquer un skill précis : `skillOverrides` (`"on"`, `"name-only"`, `"user-invocable-only"`, `"off"`).

## Skills synchronisés depuis claude.ai

Les skills activés sur votre compte claude.ai sont disponibles dans les sessions cloud, Cowork et les sessions du terminal connectées à ce compte, sous le nom `/anthropic-skills:<nom>` (ou `/<nom>` sans conflit). Exemples typiques d'Anthropic : création et lecture de **Word** (`docx`), **PDF** (`pdf`), **PowerPoint** (`pptx`), **Excel** (`xlsx`), **skill-creator** (créer et évaluer des skills), **deep-research**, documents Claude Docs, Google Workspace, navigateur intégré, Claude in Chrome, computer use, import de mémoire, brief matinal. La liste exacte dépend de votre compte et de votre organisation.

## Où placer ses skills

| Portée | Chemin | Disponible pour |
|--------|--------|-----------------|
| Entreprise | `.claude/skills/<nom>/SKILL.md` dans le dossier des réglages gérés | Tous les utilisateurs de l'organisation |
| Personnel | `~/.claude/skills/<nom>/SKILL.md` | Tous vos projets sur cette machine |
| Projet | `.claude/skills/<nom>/SKILL.md` | Ce dépôt (committez-le pour l'équipe) |
| Imbriqué | `<sous-dossier>/.claude/skills/<nom>/SKILL.md` | Sessions travaillant dans ce sous-dossier (monorepos) |
| Plugin | `<plugin>/skills/<nom>/SKILL.md` | Partout où le plugin est actif, en `/plugin:nom` |
| Compte claude.ai | Skills activés sur votre compte | Cloud, Cowork, terminal connecté |

Priorité en cas de nom identique : entreprise > personnel > projet. Votre skill remplace un skill intégré du même nom (mais pas ses alias). L'ancien format `.claude/commands/nom.md` fonctionne toujours (un skill l'emporte sur une commande du même nom).

## Créer un skill

```text
.claude/skills/resume-diff/
├── SKILL.md
├── reference.md        # doc chargée seulement si besoin
└── scripts/
    └── check.sh
```

```markdown
---
name: resume-diff
description: Résume les changements non commités et signale ce qui est risqué. À utiliser avant un commit ou quand on demande "qu'est-ce qui a changé ?".
argument-hint: "[focus]"
allowed-tools: Bash(git diff *) Bash(git status *)
---

## Changements actuels

!`git diff HEAD`

## Consignes

Résume les changements ci-dessus en 3 à 5 puces, puis liste les risques
(gestion d'erreurs manquante, valeurs codées en dur, tests à mettre à jour).
Si un focus est donné, concentre-toi dessus : $ARGUMENTS
```

Puis tapez `/resume-diff sécurité`. Le plus simple reste de demander à Claude : « crée-moi un skill qui… » (ou d'utiliser le skill `skill-creator`).

## Frontmatter de `SKILL.md`

| Champ | Rôle |
|-------|------|
| `name` | Nom de la commande (par défaut le nom du dossier) |
| `description` | **Recommandé** : ce que fait le skill et quand l'utiliser (Claude s'en sert pour décider de l'invoquer) |
| `when_to_use` | Phrases déclencheuses, exemples de demandes |
| `argument-hint` | Indice d'arguments affiché dans l'autocomplétion |
| `arguments` | Arguments nommés (`$issue`, `$branch`…) |
| `disable-model-invocation: true` | Seul vous pouvez le lancer (workflows avec effets de bord, ex. déploiement) |
| `user-invocable: false` | Seul Claude peut le lancer (connaissance de fond) |
| `allowed-tools` | Outils autorisés sans demande pendant le tour qui l'invoque |
| `disallowed-tools` | Outils retirés tant que le skill est actif |
| `model` | Modèle à utiliser pendant le tour |
| `effort` | Niveau d'effort pendant le skill |
| `context: fork` + `agent` | Exécuter le skill dans un **sous-agent** isolé (ex. `agent: Explore`) |
| `background` | Avec `context: fork`, attendre ou non le résultat |
| `hooks` | Hooks enregistrés à l'invocation |
| `paths` | Motifs de fichiers qui déclenchent le chargement automatique |
| `shell` | `bash` (défaut) ou `powershell` pour les commandes injectées |
| `metadata`, `license`, `compatibility` | Métadonnées (spécification Agent Skills) |

## Variables et injection dynamique

| Variable | Valeur |
|----------|--------|
| `$ARGUMENTS` | Tous les arguments |
| `$ARGUMENTS[0]`, `$0`, `$1`… | Argument par position |
| `$nom` | Argument nommé déclaré dans `arguments` |
| `${CLAUDE_SESSION_ID}` | ID de la session |
| `${CLAUDE_EFFORT}` | Niveau d'effort actuel |
| `${CLAUDE_SKILL_DIR}` | Dossier du skill (pour appeler ses scripts) |
| `${CLAUDE_PROJECT_DIR}` | Racine du projet |
| `${CLAUDE_PLUGIN_ROOT}`, `${CLAUDE_PLUGIN_DATA}` | Dossier et données persistantes du plugin |

- `` !`commande` `` (ou un bloc ` ```! `) exécute une commande **avant** d'envoyer le skill à Claude et insère sa sortie : Claude reçoit des données réelles (diff, logs, état). Si la commande échoue, l'invocation est annulée.
- Les fichiers annexes (docs, exemples, scripts) ne coûtent rien tant que Claude ne les lit pas.

## Qui peut invoquer un skill ?

| Frontmatter | Vous | Claude | Dans le contexte |
|-------------|:----:|:------:|------------------|
| (défaut) | Oui | Oui | Description toujours chargée |
| `disable-model-invocation: true` | Oui | Non | Description non chargée |
| `user-invocable: false` | Non | Oui | Description toujours chargée |

Écrire `/deploy` au milieu d'une phrase (« vas-y et /deploy en staging ») ne lance rien directement mais donne à Claude la permission de l'utiliser.

## Skills vs autres extensions

| Besoin | Outil |
|--------|-------|
| Règle toujours valable | `CLAUDE.md` |
| Procédure ou connaissance réutilisable, à la demande | **Skill** |
| Travail isolé qui renvoie un résumé | Sous-agent (ou skill avec `context: fork`) |
| Action garantie à chaque événement | Hook |
| Accès à un service externe | MCP |
| Distribuer tout ça à une équipe | Plugin |
