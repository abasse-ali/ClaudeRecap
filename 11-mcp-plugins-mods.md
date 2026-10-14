# 11 — MCP, plugins, mods et intelligence de code

[← Sommaire](README.md)

## MCP (Model Context Protocol)

MCP est un standard ouvert qui connecte Claude à des **outils et services externes** : GitHub, Jira, Linear, Slack, Notion, Sentry, Figma, bases de données (PostgreSQL…), navigateurs, API internes… Une fois un serveur connecté, on demande simplement : « implémente la fonctionnalité du ticket JIRA ENG-4521 et ouvre une PR », « vérifie Sentry pour les erreurs de cette fonction », « interroge la base pour les 10 derniers clients ».

On trouve des connecteurs vérifiés dans l'**Anthropic Directory** (claude.ai/directory).

### Ajouter un serveur

```bash
# Serveur distant HTTP (recommandé)
claude mcp add --transport http github https://api.githubcopilot.com/mcp/ \
  --header "Authorization: Bearer VOTRE_TOKEN"

# Serveur local (stdio) : tout ce qui suit -- est la commande de lancement
claude mcp add --env API_KEY=xxx mon-serveur -- npx -y @exemple/mcp-server

# Serveur SSE (déprécié) : --transport sse (HTTP bascule seul vers SSE si besoin)

# Depuis un bloc JSON écrit pour un autre client (ou pour un serveur WebSocket "type":"ws")
claude mcp add-json mon-serveur '{"type":"http","url":"https://…"}'
claude mcp add-json events-server '{"type":"ws","url":"wss://…"}'

# Importer les serveurs de Claude Desktop
claude mcp add-from-claude-desktop
```

### Gérer les serveurs

| Commande | Rôle |
|----------|------|
| `claude mcp list` / `get <nom>` / `remove <nom>` | Lister / détailler / supprimer |
| `claude mcp login <nom>` / `logout <nom>` | Authentification OAuth depuis le shell |
| `claude mcp reset-project-choices` | Réinitialiser les approbations des serveurs de `.mcp.json` |
| `claude mcp serve` | Utiliser **Claude Code lui-même comme serveur MCP** pour d'autres applications |
| `/mcp` | Panneau interactif : état, outils par serveur, authentification, activer/désactiver, reconnecter |
| `/mcp reconnect all` | Réessayer tous les serveurs en échec |

### Portées

| Portée | Chargé dans | Partagé | Stocké dans |
|--------|-------------|---------|-------------|
| **local** (défaut) | Ce projet | Non | `~/.claude.json` |
| **project** | Ce projet | Oui, via git | `.mcp.json` à la racine |
| **user** | Tous vos projets | Non | `~/.claude.json` |

Choisir avec `--scope local|project|user`. Priorité : local > project > user > plugins > connecteurs claude.ai. `.mcp.json` accepte les variables d'environnement `${VAR}` et `${VAR:-défaut}`.

### Fonctions MCP

- **Authentification OAuth 2.0** pour les serveurs distants (via `/mcp`), en-têtes dynamiques (`headersHelper`).
- **Connecteurs claude.ai** : les connecteurs ajoutés sur claude.ai/customize/connectors sont disponibles automatiquement si vous êtes connecté avec ce compte.
- **Ressources** : les mentionner avec `@serveur:protocole://ressource`, ex. `analyse @github:issue://123`.
- **Prompts MCP** : apparaissent comme commandes `/`.
- **Tool search** : avec beaucoup d'outils, ils sont chargés à la demande (outil `ToolSearch`) pour économiser le contexte.
- **Élicitation** : un serveur peut vous poser une question (formulaire) pendant un appel.
- **Mises à jour dynamiques** des outils, **reconnexion automatique**, appels longs passés en arrière-plan après 2 minutes.
- **Channels** : un serveur peut pousser des événements dans la session (voir [10](10-hooks-et-automatisation.md)).
- **Sous-agents** : chaque sous-agent peut avoir ses propres serveurs (`mcpServers`).
- Contrôle d'entreprise : `managed-mcp.json`, `allowedMcpServers` / `deniedMcpServers`.

## Plugins

Un **plugin** est un paquet qui regroupe des **skills**, des **sous-agents**, des **hooks**, des **serveurs MCP** et **LSP**, et éventuellement un **module de hooks JavaScript** (un mod). Manifeste : `.claude-plugin/plugin.json`. Ses skills s'appellent `/nom-plugin:nom-skill`.

### Le gestionnaire `/plugin`

| Onglet | Contenu |
|--------|---------|
| **Discover** | Plugins des marketplaces ajoutées (avec estimation du **coût en contexte** pour la marketplace officielle) |
| **Installed** | Plugins installés : `Espace` activer/désactiver, `f` favori, `Entrée` détails, configurer les options, mettre à jour, désinstaller ; section **Built-in** pour les mods intégrés |
| **Marketplaces** | Ajouter, mettre à jour, retirer des marketplaces |
| **Errors** | Erreurs de chargement |

Commandes directes : `/plugin install <plugin>@<marketplace>`, `/plugin enable|disable|uninstall …`, `/plugin marketplace add <source>`, `/reload-plugins`. Depuis le shell : `claude plugin install … [--scope project|local] [--marketplace <source>]`, `claude plugin list`, `claude plugin validate <dossier>`, `claude plugin eval` (tests d'évaluation de plugins).

### Portées d'installation

- **User** : pour vous, dans tous les projets (`~/.claude/settings.json`) ;
- **Project** : pour tous les collaborateurs du dépôt (`.claude/settings.json`, à committer) ;
- **Local** : pour vous, dans ce dépôt seulement (`.claude/settings.local.json`).

### Les marketplaces

| | Officielle | Communautaire | Démo |
|---|---|---|---|
| Dépôt | `anthropics/claude-plugins-official` | `anthropics/claude-plugins-community` | `anthropics/claude-code` |
| Nom | `claude-plugins-official` | `claude-community` | `claude-code-plugins` |
| Contenu | Plugins Anthropic + partenaires | Plugins soumis par des tiers | Exemples |
| Ajout | **Automatique** au premier lancement | `/plugin marketplace add anthropics/claude-plugins-community` | `/plugin marketplace add anthropics/claude-code` |

Une marketplace peut être un dépôt GitHub (`owner/repo#tag`), une URL git, un dossier local ou un `marketplace.json` hébergé. Catalogue web : claude.com/marketplace/plugins. Exemples de plugins officiels : `commit-commands`, `code-review`, `security-guidance`, `claude-security`, les plugins LSP, les channels Telegram/Discord/iMessage…

> Un plugin s'exécute **avec vos droits** : n'installez que depuis des sources de confiance.

### Créer un plugin

```text
mon-plugin/
├── .claude-plugin/
│   └── plugin.json        # { "name": "mon-plugin", "version": "1.0.0", "description": "…" }
├── skills/revue/SKILL.md
├── agents/securite.md
├── hooks/hooks.json
└── .mcp.json
```

Tester sans installer : `claude --plugin-dir ./mon-plugin`. Valider : `claude plugin validate`. Distribuer : créer une marketplace (`.claude-plugin/marketplace.json`) ou publier dans l'annuaire d'Anthropic. Les entreprises peuvent imposer ou bloquer des marketplaces et plugins.

## Mods : modifier l'interface de Claude Code

Un **mod** est un plugin dont le code JavaScript (« module de hooks », `register.js`) s'exécute **à l'intérieur** de Claude Code. Il peut :

- **dessiner une interface** : un volet à côté de la transcription ou une bande au-dessus du prompt, avec onglets, boutons, champs ;
- **redessiner l'interface existante** : ligne d'un appel d'outil, spinner, boîte de questions ;
- **intervenir sur un appel d'outil ou une requête** (le retenir, y répondre, l'approuver) ;
- **ajouter une commande `/`** qui exécute votre code immédiatement, sans tour de Claude ;
- partager des données entre ses hooks, envoyer des notifications, appeler un modèle.

Créer : décrivez le mod à Claude (il utilise le skill `/plugin-authoring`). Exemples d'Anthropic : `token-weather` (prévision du remplissage du contexte), `blast-radius` (retient une commande risquée comme `rm -rf`), `replay-theater` (commande `/replay`).

**Mods intégrés** (visibles dans `/plugin` › Installed › Built-in) :

| Mod | Rôle |
|-----|------|
| `cc-plugin-agents-md` | Charge les `AGENTS.md` comme instructions |
| `cc-plugin-diff` | Le panneau `/diff` |
| `cc-plugin-plugin-authoring` | Le skill d'écriture de mods |
| `cc-plugin-you-should-know` | Notes « You should know » (désactivé par défaut : `/plugin enable cc-plugin-you-should-know@builtin`) |
| `cc-plugin-telemetry` | Analytique |
| `cc-plugin-sec-default` | Garde-fou des réglages gérés de l'organisation |

Désactiver les mods installés : `--safe-mode` (une session) ou `"disableAllHooks": true`.

## Intelligence de code (LSP)

Les plugins **LSP** connectent Claude à un serveur de langage : après chaque modification, il voit les **erreurs de type et avertissements**, et il peut **aller à la définition**, **trouver les références**, obtenir des types (outil `LSP`). Plugins officiels : `typescript-lsp`, `pyright-lsp`, `gopls-lsp`, `rust-analyzer-lsp`, `jdtls-lsp` (Java), `kotlin-lsp`, `csharp-lsp`, `clangd-lsp` (C/C++), `php-lsp`, `ruby-lsp`, `swift-lsp`, `sourcekit-lsp`, `lua-lsp`, `liquid-lsp`. Claude Code propose automatiquement le plugin adapté à votre projet. Fonctionne dans les sessions du terminal (pas dans les sessions cloud).
