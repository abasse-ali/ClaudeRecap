# Exemples de fichiers de réglages

> Source : https://code.claude.com/docs/en/settings-example · [← Index](../31-index-documentation-officielle.md)

*Des fichiers settings.json réalistes pour un développeur, une équipe et une organisation : copiez-en un, gardez les clés voulues et changez les valeurs.*

Cette page contient trois exemples de fichiers `settings.json`, un pour chaque endroit où l'on enregistre un réglage :

* Le `~/.claude/settings.json` d'un développeur
* Le `.claude/settings.json` d'une équipe, committé dans le dépôt
* Le `managed-settings.json` d'une organisation

Chacun est un fichier plausible pour ce lecteur, afin que vous voyiez sa forme et copiiez les parties voulues. Aucun n'est une base recommandée. Chaque valeur provient de l'entrée de la clé dans la [référence des réglages](https://code.claude.com/docs/en/settings-reference), qui donne son type, sa valeur par défaut et où elle peut être définie.

Chaque exemple a deux onglets. **Copyable settings file** (fichier copiable) est le fichier tel que vous l'enregistreriez. **What each key does** (ce que fait chaque clé) est le même fichier avec un commentaire au-dessus de chaque clé ; Claude Code n'accepte pas de commentaires dans un fichier de réglages, copiez donc depuis le premier onglet.

## Vos propres réglages

Les réglages personnels d'un développeur. Le fichier choisit un modèle et un effort, ajuste le terminal, et pré-approuve une commande en lecture seule et la lecture d'un fichier. Tout ce qui n'est pas listé garde sa valeur par défaut. Un fichier comme celui-ci se place dans `~/.claude/settings.json`, où il s'applique à tous les projets que vous ouvrez.

### Onglet « Copyable settings file »

Enregistrez ceci sous `~/.claude/settings.json`. C'est du JSON valide sans commentaires : vous pouvez le coller tel quel et supprimer les clés dont vous ne voulez pas.

```json ~/.claude/settings.json
{
  "model": "claude-sonnet-5",
  "modelSettings": {
    "claude-sonnet-5": { "effortLevel": "xhigh" }
  },
  "editorMode": "vim",
  "theme": "light-daltonized",
  "statusLine": {
    "type": "command",
    "command": "jq -r '\"[\\(.model.display_name)] \\(.context_window.used_percentage // 0)% context\"'",
    "padding": 2
  },
  "spinnerTipsEnabled": false,
  "preferredNotifChannel": "terminal_bell",
  "permissions": {
    "allow": [
      "Bash(git diff *)",
      "Read(~/.zshrc)"
    ]
  },
  "autoUpdatesChannel": "stable",
  "cleanupPeriodDays": 20
}
```

### Onglet « What each key does »

Le même fichier avec un commentaire au-dessus de chaque clé. Lisez-le ici ; copiez depuis l'autre onglet, car Claude Code n'accepte pas de commentaires dans un fichier de réglages.

```jsonc ~/.claude/settings.json
{
  // Démarrer chaque session sur Sonnet 5
  "model": "claude-sonnet-5",
  // Faire tourner Sonnet 5 au-dessus de son niveau par défaut high ; /effort enregistre un niveau par modèle, et --effort en définit un pour une seule session
  "modelSettings": {
    "claude-sonnet-5": { "effortLevel": "xhigh" }
  },
  // Raccourcis clavier Vim dans le prompt
  "editorMode": "vim",
  // Le thème clair adapté aux daltoniens
  "theme": "light-daltonized",
  // Une ligne de statut sous le prompt : nom du modèle et contexte utilisé
  "statusLine": {
    "type": "command",
    "command": "jq -r '\"[\\(.model.display_name)] \\(.context_window.used_percentage // 0)% context\"'",
    "padding": 2
  },
  // Masquer les astuces qui défilent sous le spinner
  "spinnerTipsEnabled": false,
  // Faire sonner la cloche du terminal pour les notifications, comme une tâche terminée ou une demande de permission en attente
  "preferredNotifChannel": "terminal_bell",
  // Laisser Claude Code exécuter git diff et lire votre .zshrc sans demander
  "permissions": {
    "allow": [
      "Bash(git diff *)",
      "Read(~/.zshrc)"
    ]
  },
  // Recevoir les mises à jour depuis le canal stable
  "autoUpdatesChannel": "stable",
  // Supprimer les transcriptions de session et autres données de session locales de plus de 20 jours
  "cleanupPeriodDays": 20
}
```

Récapitulatif des clés utilisées :

| Clé | Valeur | Effet |
| :- | :- | :- |
| `model` | `"claude-sonnet-5"` | Chaque session démarre sur Sonnet 5 |
| `modelSettings` | `{"claude-sonnet-5": {"effortLevel": "xhigh"}}` | Effort `xhigh` pour Sonnet 5 (au-dessus du défaut `high`) |
| `editorMode` | `"vim"` | Raccourcis Vim dans le prompt |
| `theme` | `"light-daltonized"` | Thème clair adapté aux daltoniens |
| `statusLine` | `type: "command"`, commande `jq`, `padding: 2` | Ligne de statut : nom du modèle et % de contexte utilisé |
| `spinnerTipsEnabled` | `false` | Masque les astuces sous le spinner |
| `preferredNotifChannel` | `"terminal_bell"` | Notifications par la cloche du terminal |
| `permissions.allow` | `Bash(git diff *)`, `Read(~/.zshrc)` | Pré-approbations sans demande |
| `autoUpdatesChannel` | `"stable"` | Mises à jour du canal stable |
| `cleanupPeriodDays` | `20` | Suppression des données de session locales de plus de 20 jours |

## Les réglages partagés d'une équipe

Les réglages partagés d'une équipe, committés dans le dépôt pour que toute personne qui le clone obtienne les mêmes permissions, hooks et marketplace de plugins. Enregistrez un fichier comme celui-ci dans `.claude/settings.json` à la racine du dépôt. À savoir avant d'en committer un :

* **Les sessions cloud le lisent aussi.** Une [session cloud](https://code.claude.com/docs/en/settings#settings-in-cloud-sessions) démarre depuis un clone du dépôt, donc le fichier committé s'y applique également.
* **La télémétrie va dans les réglages gérés ou personnels.** Claude Code ignore les [variables d'exportateur OpenTelemetry](https://code.claude.com/docs/en/settings-reference#variables-claude-code-ignores-in-env) dans les fichiers de réglages d'un dépôt, à part certaines valeurs qui désactivent la télémétrie. Définissez-les dans les [réglages gérés](https://code.claude.com/docs/en/monitoring-usage#administrator-configuration) de votre organisation, ou dans le `~/.claude/settings.json` de chaque personne.
* **Les règles allow attendent la confiance.** Les règles allow et les entrées `extraKnownMarketplaces` prennent effet après que chaque personne a [fait confiance à ce dossier lui-même](https://code.claude.com/docs/en/permissions#project-allow-rules-and-workspace-trust), et pas seulement à un dossier parent ; les règles deny et ask s'appliquent dans chaque session, de confiance ou non.
* **Le hook est un script dans le dépôt.** Le hook de ce fichier exécute `.claude/hooks/block-rm.sh` ; [How a hook resolves](https://code.claude.com/docs/en/hooks#how-a-hook-resolves) explique comment l'écrire.
* **Les règles correspondent à la commande et au chemin tels qu'écrits.** `Bash(git push *)` ne correspond pas à [`git -C . push`](https://code.claude.com/docs/en/permissions#bash-rule-limits). `Read(./.env)` seul bloque les outils de fichiers et les commandes qui nomment le fichier, comme `cat .env`, mais pas un [`grep -r` lancé sur le répertoire](https://code.claude.com/docs/en/permissions#read-and-edit) ; le bloc `sandbox` de ce fichier comble cette lacune, car la sandbox [ajoute vos chemins `Read` refusés](https://code.claude.com/docs/en/settings-reference#sandbox-filesystem-denyread) à ce qu'aucune commande sandboxée ne peut lire.

### Onglet « Copyable settings file »

Enregistrez ceci sous `.claude/settings.json` à la racine du dépôt et committez-le. C'est du JSON valide sans commentaires : vous pouvez le coller tel quel et supprimer les clés dont vous ne voulez pas.

```json .claude/settings.json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)"
    ],
    "ask": [
      "Bash(git push *)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/block-rm.sh"
          }
        ]
      }
    ]
  },
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "code-formatter@acme-tools": true
  },
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "allowWrite": [
        "/tmp/build"
      ]
    },
    "network": {
      "allowedDomains": [
        "registry.npmjs.org",
        "*.example.com"
      ]
    }
  },
  "plansDirectory": "./plans"
}
```

### Onglet « What each key does »

Le même fichier avec un commentaire au-dessus de chaque clé. Lisez-le ici ; copiez depuis l'autre onglet, car Claude Code n'accepte pas de commentaires dans un fichier de réglages.

```jsonc .claude/settings.json
{
  "permissions": {
    // Exécuter les scripts npm sans demander
    "allow": [
      "Bash(npm run *)"
    ],
    // Demander confirmation avant les commandes git push
    "ask": [
      "Bash(git push *)"
    ],
    // Refuser la lecture des fichiers env et du dossier secrets par les outils de fichiers et les commandes qui lisent des fichiers
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  },
  // Avant chaque commande Bash, exécuter un script du dépôt qui peut la bloquer
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/block-rm.sh"
          }
        ]
      }
    ]
  },
  // Enregistrer la marketplace de plugins de l'équipe sur chaque clone
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    }
  },
  // Activer un plugin de cette marketplace ; un plugin d'une source externe comme un dépôt GitHub doit quand même être installé une fois par chaque personne
  "enabledPlugins": {
    "code-formatter@acme-tools": true
  },
  // Sandboxer les commandes : répertoire de build inscriptible ; npm et example.com pré-autorisés
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "allowWrite": [
        "/tmp/build"
      ]
    },
    "network": {
      "allowedDomains": [
        "registry.npmjs.org",
        "*.example.com"
      ]
    }
  },
  // Garder les fichiers de plan dans le dépôt
  "plansDirectory": "./plans"
}
```

Récapitulatif des clés utilisées :

| Clé | Valeur | Effet |
| :- | :- | :- |
| `permissions.allow` | `Bash(npm run *)` | Scripts npm sans demande |
| `permissions.ask` | `Bash(git push *)` | Confirmation avant `git push` |
| `permissions.deny` | `Read(./.env)`, `Read(./.env.*)`, `Read(./secrets/**)` | Lecture des fichiers env et du dossier secrets refusée |
| `hooks.PreToolUse` | `matcher: "Bash"`, commande `${CLAUDE_PROJECT_DIR}/.claude/hooks/block-rm.sh` | Script du dépôt pouvant bloquer chaque commande Bash |
| `extraKnownMarketplaces` | `acme-tools` → GitHub `acme-corp/claude-plugins` | Marketplace d'équipe enregistrée sur chaque clone |
| `enabledPlugins` | `code-formatter@acme-tools: true` | Active un plugin de cette marketplace |
| `sandbox.enabled` | `true` | Commandes sandboxées |
| `sandbox.filesystem.allowWrite` | `/tmp/build` | Répertoire de build inscriptible |
| `sandbox.network.allowedDomains` | `registry.npmjs.org`, `*.example.com` | Domaines réseau pré-autorisés |
| `plansDirectory` | `"./plans"` | Fichiers de plan gardés dans le dépôt |

## Les réglages gérés d'une organisation

Un fichier `managed-settings.json` qui montre la forme des clés gérées, avec une valeur plausible pour chacune. Ce n'est pas une politique recommandée : choisissez les clés qui correspondent à vos propres exigences et définissez vos propres valeurs. L'exemple définit ces clés :

* `forceLoginMethod` et `forceLoginOrgUUID` fixent la méthode de connexion et l'organisation
* `availableModels` et `enforceAvailableModels` restreignent les modèles utilisables par les sessions
* `permissions.deny` refuse deux lectures de fichiers et les commandes `curl` [telles que Claude les écrit](https://code.claude.com/docs/en/permissions#bash-rule-limits), et `disableBypassPermissionsMode` supprime le mode de permission bypass
* [`allowManagedPermissionRulesOnly`](https://code.claude.com/docs/en/settings-reference#allowmanagedpermissionrulesonly) et [`allowManagedMcpServersOnly`](https://code.claude.com/docs/en/settings-reference#allowmanagedmcpserversonly) font des listes d'autorisation gérées de permissions et de MCP les seules qui s'appliquent
* `allowedMcpServers` épingle le serveur MCP par URL
* `strictKnownMarketplaces` autorise une seule marketplace de plugins
* `sandbox` sandboxe les commandes avec une liste d'autorisation réseau fixe et sans nouvelle tentative hors sandbox. Sa clé `failIfUnavailable` [empêche Claude Code de démarrer là où la sandbox ne peut pas fonctionner](https://code.claude.com/docs/en/sandboxing#enforce-sandboxing-with-managed-settings)
* `requiredMinimumVersion` fixe une version minimale de Claude Code
* `cleanupPeriodDays` raccourcit la conservation des transcriptions de session et autres données locales à sept jours
* `companyAnnouncements` affiche un message au démarrage

Les administrateurs déploient un fichier comme celui-ci en tant que `managed-settings.json`, ou le même JSON via MDM ou les [server-managed settings](https://code.claude.com/docs/en/server-managed-settings). Un fichier déployé s'applique à chaque machine ou compte qu'il atteint. Pour donner des valeurs différentes à un groupe, déployez un autre fichier ou profil à ce groupe, car [les server-managed settings ne prennent pas encore en charge de politique par groupe](https://code.claude.com/docs/en/server-managed-settings#current-limitations).

### Onglet « Copyable settings file »

Déployez ceci en tant que `managed-settings.json`, ou le même JSON via MDM ou la console claude.ai. C'est du JSON valide sans commentaires ; remplacez l'UUID d'organisation, l'URL de serveur et la marketplace d'exemple par les vôtres, et supprimez les clés dont vous ne voulez pas.

```json managed-settings.json
{
  "forceLoginMethod": "claudeai",
  "forceLoginOrgUUID": [
    "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  ],
  "availableModels": [
    "opus",
    "sonnet"
  ],
  "enforceAvailableModels": true,
  "permissions": {
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./secrets/**)"
    ],
    "disableBypassPermissionsMode": "disable"
  },
  "allowManagedPermissionRulesOnly": true,
  "allowedMcpServers": [
    {
      "serverUrl": "https://api.githubcopilot.com/*"
    }
  ],
  "allowManagedMcpServersOnly": true,
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "acme-corp/approved-plugins"
    }
  ],
  "sandbox": {
    "enabled": true,
    "failIfUnavailable": true,
    "allowUnsandboxedCommands": false,
    "network": {
      "allowedDomains": [
        "registry.npmjs.org",
        "github.com"
      ],
      "allowManagedDomainsOnly": true
    }
  },
  "requiredMinimumVersion": "2.1.150",
  "cleanupPeriodDays": 7,
  "companyAnnouncements": [
    "Welcome to Acme Corp! Review our code guidelines at docs.example.com"
  ]
}
```

### Onglet « What each key does »

Le même fichier avec un commentaire au-dessus de chaque clé. Lisez-le ici ; copiez depuis l'autre onglet, car Claude Code n'accepte pas de commentaires dans un fichier de réglages.

```jsonc managed-settings.json
{
  // Uniquement des connexions claude.ai, et uniquement dans cette organisation
  "forceLoginMethod": "claudeai",
  "forceLoginOrgUUID": [
    "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  ],
  // Uniquement les modèles Opus et Sonnet ; avec enforceAvailableModels, l'option Default respecte aussi la liste
  "availableModels": [
    "opus",
    "sonnet"
  ],
  "enforceAvailableModels": true,
  "permissions": {
    // Refuser les commandes curl et la lecture du fichier .env et du dossier secrets du projet sur toutes les machines
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./secrets/**)"
    ],
    // Supprimer le mode bypass-permissions de toutes les sessions
    "disableBypassPermissionsMode": "disable"
  },
  // Ignorer les règles de permission des réglages user, project et local
  "allowManagedPermissionRulesOnly": true,
  // Uniquement le serveur MCP GitHub, identifié par URL plutôt que par nom, puisqu'un utilisateur peut
  // nommer n'importe quel serveur « github ». Les serveurs ajoutés par l'utilisateur qui ne correspondent pas
  // ne se chargent pas, y compris tout serveur stdio quand la liste ne contient que des entrées URL. La clé
  // allowManagedMcpServersOnly ci-dessous fait de cette liste gérée la seule liste d'autorisation qui s'applique
  "allowedMcpServers": [
    {
      "serverUrl": "https://api.githubcopilot.com/*"
    }
  ],
  "allowManagedMcpServersOnly": true,
  // Les plugins ne peuvent venir que de cette marketplace
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "acme-corp/approved-plugins"
    }
  ],
  // Sandboxer chaque commande exécutée par Claude, refuser de démarrer si la sandbox ne peut pas être
  // mise en place, et ne jamais laisser une commande bloquée réessayer hors sandbox ; réseau
  // limité à npm et GitHub, et les utilisateurs ne peuvent pas ajouter de domaines
  "sandbox": {
    "enabled": true,
    "failIfUnavailable": true,
    "allowUnsandboxedCommands": false,
    "network": {
      "allowedDomains": [
        "registry.npmjs.org",
        "github.com"
      ],
      "allowManagedDomainsOnly": true
    }
  },
  // Refuser de démarrer sur les versions antérieures à 2.1.150
  "requiredMinimumVersion": "2.1.150",
  // Supprimer les transcriptions de session et autres données de session locales après 7 jours
  "cleanupPeriodDays": 7,
  // Un message que chaque utilisateur voit au démarrage
  "companyAnnouncements": [
    "Welcome to Acme Corp! Review our code guidelines at docs.example.com"
  ]
}
```

Récapitulatif des clés utilisées :

| Clé | Valeur | Effet |
| :- | :- | :- |
| `forceLoginMethod` | `"claudeai"` | Connexions claude.ai uniquement |
| `forceLoginOrgUUID` | `["xxxxxxxx-…"]` | Uniquement dans cette organisation |
| `availableModels` | `["opus", "sonnet"]` | Seuls Opus et Sonnet |
| `enforceAvailableModels` | `true` | L'option Default respecte aussi la liste |
| `permissions.deny` | `Bash(curl *)`, `Read(./.env)`, `Read(./secrets/**)` | curl et lectures sensibles refusés sur toutes les machines |
| `permissions.disableBypassPermissionsMode` | `"disable"` | Mode bypass-permissions supprimé |
| `allowManagedPermissionRulesOnly` | `true` | Règles de permission user/project/local ignorées |
| `allowedMcpServers` | `serverUrl: "https://api.githubcopilot.com/*"` | Seul le serveur MCP GitHub, identifié par URL |
| `allowManagedMcpServersOnly` | `true` | Seule la liste MCP gérée s'applique |
| `strictKnownMarketplaces` | GitHub `acme-corp/approved-plugins` | Une seule marketplace de plugins autorisée |
| `sandbox.enabled` | `true` | Toutes les commandes sandboxées |
| `sandbox.failIfUnavailable` | `true` | Refus de démarrer si la sandbox est indisponible |
| `sandbox.allowUnsandboxedCommands` | `false` | Pas de nouvelle tentative hors sandbox |
| `sandbox.network.allowedDomains` | `registry.npmjs.org`, `github.com` | Réseau limité à npm et GitHub |
| `sandbox.network.allowManagedDomainsOnly` | `true` | Les utilisateurs ne peuvent pas ajouter de domaines |
| `requiredMinimumVersion` | `"2.1.150"` | Refus de démarrer en dessous de 2.1.150 |
| `cleanupPeriodDays` | `7` | Données de session locales supprimées après 7 jours |
| `companyAnnouncements` | `["Welcome to Acme Corp! …"]` | Message affiché au démarrage |
