# 16 — Intégrations, revue de code et CI/CD

[← Sommaire](README.md)

## Relire du code

| Outil | Où | Ce qu'il fait |
|-------|----|---------------|
| `/code-review [niveau] [cible]` (alias `/review`) | Session locale | Cherche les **bugs** du diff courant, d'une PR (`/code-review high 1234`), d'une branche ou d'un chemin ; niveaux `low` → `max` ; `--fix` applique ; `--comment` publie sur la PR GitHub / MR GitLab ; `--max-findings n\|all` |
| `/simplify` | Session locale | Qualité : réutilisation, simplification, efficacité, abstraction (applique les corrections) |
| `/security-review` | Session locale | Failles du diff de la branche vs la branche par défaut d'`origin` (injection, authentification, exposition de données…) |
| **Ultrareview** (`/code-review ultra`, `/ultrareview`, `claude ultrareview`) | Cloud | Revue **multi-agents approfondie** qui trouve et **vérifie** les bugs (15–25 min, en arrière-plan) ; `--post` publie les résultats sur la PR ; 3 essais gratuits (Pro/Max) puis crédits d'usage |
| **Code Review** (GitHub) | Automatique sur chaque PR | Revue multi-agents à l'ouverture de la PR, à chaque push ou à la demande |
| Desktop | Carte **Code review** | **Walk through in diff**, **Fix this one**, **Apply fixes** |

### Code Review automatique (Team/Enterprise)

Activé par un Owner pour l'organisation et les dépôts choisis. Chaque remarque a une sévérité :

| Marqueur | Sévérité | Sens |
|:--------:|----------|------|
| 🔴 | Important | Bug à corriger avant de fusionner |
| 🟡 | Nit | Problème mineur, non bloquant |
| 🟣 | Pre-existing | Bug déjà présent, pas introduit par la PR |

- Check run **Claude Code Review** avec un résumé de toutes les remarques ; 👍/👎 sur chaque commentaire.
- Commandes en commentaire de PR : `@claude review` (une fois), `@claude review always` (s'abonner aux pushs suivants), `@claude review once`.
- Personnalisation : `CLAUDE.md` (violations relevées en *nit*) et **`REVIEW.md`** à la racine (consignes propres à la revue : quoi signaler, à quelle sévérité).

## Sécurité

| Outil | Rôle |
|-------|------|
| `/security-review` | Revue de sécurité du diff de la branche |
| Plugin **security-guidance** | Claude vérifie son propre code pendant qu'il l'écrit : motifs risqués à chaque modification, revue du diff à chaque fin de tour, revue au commit |
| Plugin **claude-security** | Commande `/claude-security` : scanner toute la base, une branche, une PR ou un commit, puis proposer des **correctifs** à relire |

## GitHub Actions

Claude Code tourne dans vos workflows GitHub : il répond aux **mentions `@claude`** dans les issues et PR, transforme une issue en PR, applique des revues, fait de la maintenance planifiée.

- **Installation rapide** : `/install-github-app` (installe l'app GitHub Claude, le secret et le fichier de workflow).
- **Manuel** : installer l'app, ajouter le secret `ANTHROPIC_API_KEY` (ou un jeton OAuth via `claude setup-token`), copier le workflow utilisant l'action `anthropics/claude-code-action@v1`.
- **Exemples** : « `@claude implémente cette fonctionnalité d'après la description de l'issue` », « `@claude corrige le TypeError dans le composant dashboard` », exécution d'un skill, tâche planifiée (cron).
- Paramètres de l'action : `prompt` (texte ou invocation de skill), `claude_args` (ex. `--max-turns`, `--model`, `--allowedTools`), `anthropic_api_key` ou `claude_code_oauth_token`, `github_token`, `trigger_phrase` (défaut `@claude`), `plugins` et `plugin_marketplaces`, `settings`, `use_bedrock` / `use_vertex` / `use_foundry`.
- Les normes du projet viennent du `CLAUDE.md`.

## GitLab CI/CD

Même principe dans GitLab : un job dans `.gitlab-ci.yml` (clé API en variable masquée), déclenché par exemple par une mention `@claude` dans une issue ou une MR ; Claude travaille dans un conteneur isolé et propose ses changements via une MR. Fournisseurs : API Claude, Bedrock, Google Cloud.

Dans le terminal, Claude Code affiche aussi un badge **MR !N** pour les merge requests GitLab (avec `glab`), et `/code-review --comment` peut publier sur une MR.

## Slack

| | Claude Code dans Slack | Claude Tag |
|---|---|---|
| Forfaits | Pro, Max | Team, Enterprise |
| Identité | Votre compte | Identité partagée de l'organisation, accès gérés par l'admin |
| Fonctionnement | `@Claude` dans un canal → session cloud Claude Code, mises à jour dans le fil, boutons **View Session** et **Create PR** | `@Claude` dans un canal, un fil ou un DM ; bac à sable éphémère ; connecteurs, plugins et skills configurés par canal |
| Routage | Mode **Code only** ou **Code + Chat** (détection automatique des tâches de code) | Toute tâche d'équipe (bugs → PR, triage, alertes de monitoring…) |

Installation : `/install-slack-app` ou la marketplace Slack, puis liaison du compte Claude dans l'onglet App Home.

## Navigateur : Claude in Chrome

Claude pilote **votre Chrome** (avec vos sessions connectées) grâce à l'extension Claude in Chrome :

- tester une application web locale, lire les **logs de la console**, remplir des formulaires, extraire des données, rédiger dans Google Docs, téléverser des fichiers, enregistrer des GIF ;
- lancer avec `claude --chrome`, ou `/chrome` › **Enabled by default** ; skill `/claude-in-chrome [tâche]` ;
- permissions par site gérées dans l'extension ; les actions qui pourraient faire sortir des données sont vérifiées.

L'app Desktop a en plus un **navigateur intégré** (profil vierge, sans vos identifiants) pour l'aperçu de l'app et la navigation.

## Computer use

- **CLI (macOS)** : serveur MCP intégré `computer-use`, à activer via `/mcp` ; Claude ouvre des apps, clique, tape, voit l'écran (tester une app native, déboguer un problème visuel, piloter un simulateur, automatiser un outil sans CLI). Approbation par app et par session, avec avertissements pour les terminaux, le Finder, les Réglages système.
- **Desktop (macOS et Windows)** : voir [13](13-application-desktop.md).
- **Simulateur iOS** (Desktop, macOS) : volet dédié, un simulateur par session.

## IDE

VS Code et JetBrains : voir [14](14-ide-vscode-jetbrains.md). `/ide` connecte une session du terminal à l'IDE ouvert.

## Importer depuis d'autres outils

`/import` (ou `claude import`) récupère la configuration d'**OpenAI Codex**, **Gemini CLI** ou **Cursor** : fichiers d'instructions, serveurs MCP, commandes, sous-agents, skills.

## Dev containers et environnements isolés

Claude Code s'installe dans n'importe quel **dev container** (Dev Container Feature officielle). Le dépôt `anthropics/claude-code` fournit un conteneur de référence avec **pare-feu de sortie** (`init-firewall.sh`), volumes persistants et utilisateur non-root : on peut y lancer Claude en autonomie (`--dangerously-skip-permissions`, refusé en root). Voir aussi la sandbox ([06](06-permissions-securite-sandbox.md)) et les environnements cloud auto-hébergés.
