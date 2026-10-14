# Claude Code — Le récapitulatif complet (en français)

Ce dépôt rassemble, dans des fichiers Markdown, **tout ce que Claude Code sait faire** : ses interfaces (terminal, application Desktop, VS Code, JetBrains, web, mobile), ses boutons et raccourcis, ses commandes `/`, ses skills, ses outils, ses agents, ses intégrations et ses options de configuration.

> **Source et date** : rédigé à partir de la documentation officielle (code.claude.com/docs) consultée le **14 octobre 2026**, pour **Claude Code v2.1.299**. Claude Code évolue chaque semaine : certaines fonctions dépendent de la version, du forfait (Pro, Max, Team, Enterprise, API), de la plateforme (macOS, Windows, Linux) ou du fournisseur (API Anthropic, Amazon Bedrock, Google Cloud, Microsoft Foundry). Tapez `/release-notes` pour les nouveautés de votre version.

---

## Sommaire

| # | Fichier | Contenu |
|---|---------|---------|
| 01 | [Présentation et installation](01-presentation-et-installation.md) | Ce qu'est Claude Code, où il tourne, installation, connexion, forfaits, premiers pas |
| 02 | [Interface du terminal](02-interface-terminal.md) | Écran, zones, préfixes (`/` `!` `@` `?` `:`), **tous les raccourcis clavier**, mode Vim, file d'attente, `/btw`, `/diff`, plein écran, dictée vocale |
| 03 | [Commandes slash](03-commandes-slash.md) | **Les ~120 commandes `/`** classées par thème, avec leurs arguments et alias |
| 04 | [CLI, flags et mode script](04-cli-flags-et-mode-script.md) | Sous-commandes `claude …`, tous les flags, mode `-p` (headless), formats de sortie |
| 05 | [Outils intégrés](05-outils-integres.md) | Les outils que Claude utilise (Bash, Read, Edit, Agent, WebSearch, Monitor…) |
| 06 | [Permissions, sécurité et sandbox](06-permissions-securite-sandbox.md) | Modes de permission (Manual, Accept edits, Plan, Auto, Bypass, dontAsk), règles allow/ask/deny, sandbox, classifieur |
| 07 | [Mémoire, contexte et sessions](07-memoire-contexte-sessions.md) | `CLAUDE.md`, `AGENTS.md`, règles, mémoire automatique, compaction, reprise de session, points de contrôle (`/rewind`) |
| 08 | [Skills](08-skills.md) | Skills intégrés (`/code-review`, `/batch`, `/loop`, `/doctor`…), créer ses propres skills, frontmatter, variables |
| 09 | [Sous-agents et travail en parallèle](09-sous-agents-et-parallelisme.md) | Sous-agents, agents en arrière-plan (agent view), équipes d'agents, workflows dynamiques, worktrees, messagerie entre sessions |
| 10 | [Hooks et automatisation](10-hooks-et-automatisation.md) | Les 30+ événements de hooks, `/loop`, tâches planifiées, routines cloud, `/goal`, channels, liens profonds |
| 11 | [MCP, plugins et mods](11-mcp-plugins-mods.md) | Serveurs MCP, connecteurs, plugins et marketplaces, mods (modifier l'interface), intelligence de code (LSP) |
| 12 | [Modèles et personnalisation](12-modeles-et-personnalisation.md) | Modèles et alias, effort, mode rapide, advisor, styles de sortie, thèmes, barre d'état, raccourcis personnalisés, fichiers de réglages |
| 13 | [Application Desktop](13-application-desktop.md) | L'onglet Code de l'app Claude : volets, boutons, navigateur intégré, diff, computer use, Dispatch, SSH |
| 14 | [VS Code et JetBrains](14-ide-vscode-jetbrains.md) | Extension VS Code (boutons, menus, raccourcis), plugin JetBrains |
| 15 | [Web, mobile, Remote Control et artifacts](15-web-mobile-remote-artifacts.md) | Sessions cloud, `--teleport`, application mobile, Remote Control, Projects, artifacts |
| 16 | [Intégrations, revue de code et CI](16-integrations-revue-ci.md) | GitHub Actions, GitLab, Code Review, ultrareview, sécurité, Slack, Claude Tag, Chrome, computer use |
| 17 | [SDK, entreprise et coûts](17-sdk-entreprise-couts.md) | Agent SDK, fournisseurs cloud, gateways, réglages gérés, suivi de la consommation |
| 18 | [Antisèche et bonnes pratiques](18-antiseche-et-bonnes-pratiques.md) | Les essentiels sur une page, workflows types, conseils |

### Références complètes (tout, dans le détail)

| # | Fichier | Contenu |
|---|---------|---------|
| 19 | [Tous les réglages](19-reference-tous-les-reglages.md) | **Les 245 clés** de `settings.json` et `~/.claude.json`, classées par thème, avec leur portée |
| 20 | [Toutes les variables d'environnement](20-reference-variables-environnement.md) | **Les ~400 variables** (`ANTHROPIC_*`, `CLAUDE_CODE_*`, `MCP_*`, `OTEL_*`…) |
| 21 | [Toutes les actions clavier](21-reference-raccourcis-et-actions-clavier.md) | Chaque action personnalisable, par contexte, avec sa touche par défaut ; format de `keybindings.json` |
| 22 | [Hooks en détail](22-reference-hooks-detaillee.md) | Matchers de chaque événement, champs des 5 types de hooks, entrées JSON, codes de sortie, décisions possibles, recettes |
| 23 | [Plugins, marketplaces et mods](23-plugins-marketplaces-mods-detail.md) | Manifeste, composants, commandes `claude plugin`, évaluations, `marketplace.json`, administration, API et événements des mods |
| 24 | [Agent SDK](24-agent-sdk-detail.md) | Messages, options, fonctions et méthodes, sessions, outils personnalisés, hooks, hébergement |
| 25 | [Administration et déploiement](25-administration-et-deploiement.md) | Fournisseurs, fonctions par forfait, réglages gérés, MCP géré, gateways, réseau, environnements cloud et auto-hébergés, analytics, sécurité, données, conformité |
| 26 | [Fonctionnement interne](26-fonctionnement-interne.md) | Boucle agentique, ce qui survit à la compaction, cache de prompt, fichiers locaux, données de la barre d'état, terminal, accessibilité, monorepos |
| 27 | [Dépannage et erreurs](27-depannage-et-erreurs.md) | Outils de diagnostic, problèmes d'installation et de connexion, catalogue des messages d'erreur |
| 28 | [Glossaire](28-glossaire.md) | Tous les termes de Claude Code expliqués |
| 29 | [Bibliothèque de prompts](29-bibliotheque-de-prompts.md) | Les prompts officiels prêts à copier, traduits, par étape du cycle de développement |
| 30 | [Nouveautés et historique](30-nouveautes-et-historique.md) | Versions récentes, résumés hebdomadaires de 2026, fonctions retirées |
| 31 | [Index de la documentation officielle](31-index-documentation-officielle.md) | Les **222 pages** de la doc officielle, chacune reliée au fichier qui la résume |

---

## Claude Code en 30 secondes

**Claude Code** est l'outil de codage agentique d'Anthropic. On lui décrit une tâche en langage naturel ; il **lit le code, modifie des fichiers, lance des commandes, cherche sur le web, crée des commits et des pull requests**, en demandant la permission selon le mode choisi.

Il fonctionne :

- dans le **terminal** (`claude`) ;
- dans l'**application Claude Desktop** (onglet **Code**) sur macOS, Windows et Linux (bêta) ;
- dans **VS Code** (et forks comme Cursor) et les **IDE JetBrains** ;
- dans le **navigateur** (claude.ai/code) et l'**application mobile** iOS/Android, avec des **sessions cloud** ;
- dans **Slack**, **GitHub Actions**, **GitLab CI/CD**, et par programme via l'**Agent SDK** (Python / TypeScript).

On peut l'étendre avec :

| Extension | Rôle |
|-----------|------|
| `CLAUDE.md` | Instructions permanentes chargées à chaque session |
| Skills | Instructions et workflows réutilisables, lancés par `/nom` ou par Claude |
| Sous-agents | Assistants spécialisés avec leur propre contexte |
| Hooks | Scripts lancés automatiquement à des moments précis |
| MCP | Connexion à des outils et services externes (GitHub, bases de données, Slack…) |
| Plugins | Paquets regroupant skills, agents, hooks, serveurs MCP |
| Mods | Plugins qui modifient l'interface de Claude Code elle-même |
| Styles de sortie | Changent le ton et le format des réponses |

## Les 10 réflexes à connaître

1. `claude` dans le dossier du projet, puis **`/init`** pour créer le `CLAUDE.md`.
2. **`Shift+Tab`** pour changer de mode de permission (Manual → Accept edits → Plan → Auto).
3. **`/`** pour voir toutes les commandes, **`@`** pour citer un fichier, **`!`** pour lancer une commande shell.
4. **`Esc`** pour interrompre Claude ; **`Esc` `Esc`** (prompt vide) pour revenir en arrière (`/rewind`).
5. **`/model`** et **`/effort`** pour choisir le modèle et la profondeur de réflexion.
6. **`/context`** pour voir le remplissage du contexte, **`/compact`** pour le résumer, **`/clear`** pour repartir de zéro.
7. **`claude -c`** pour reprendre la dernière conversation, **`claude -r`** pour choisir.
8. **`/code-review`** avant de livrer, **`/security-review`** pour la sécurité.
9. **`/mcp`**, **`/plugin`**, **`/skills`**, **`/hooks`**, **`/permissions`**, **`/config`** pour tout configurer.
10. **`?`** sur un prompt vide pour l'aide des raccourcis, **`/help`** pour l'aide générale, **`/powerup`** pour des leçons interactives.
