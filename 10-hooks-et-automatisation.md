# 10 — Hooks et automatisation

[← Sommaire](README.md)

## Les hooks

Un **hook** est une action lancée **automatiquement** par Claude Code à un moment précis (avant un outil, après une modification, à la fin d'un tour…). Contrairement à une consigne dans `CLAUDE.md`, un hook s'exécute **à coup sûr** : idéal pour formater, valider, bloquer, notifier, journaliser.

### Les 5 types de hooks

| Type | Ce qu'il fait |
|------|---------------|
| `command` | Lance une commande shell ; reçoit le JSON de l'événement sur stdin, répond par code de sortie et stdout |
| `http` | Envoie le JSON de l'événement en POST à une URL |
| `mcp_tool` | Appelle un outil d'un serveur MCP |
| `prompt` | Demande une évaluation en un tour à un modèle Claude, qui renvoie une décision JSON |
| `agent` | Lance un sous-agent (avec Read, Grep…) pour vérifier une condition (expérimental) |

### Tous les événements

| Événement | Quand il se déclenche | Peut bloquer ? |
|-----------|----------------------|:--------------:|
| `SessionStart` | Début ou reprise d'une session | — |
| `Setup` | Avec `--init-only`, `--init` ou `--maintenance` (préparation en CI) | — |
| `InstructionsLoaded` | Chargement d'un `CLAUDE.md` ou d'un fichier `.claude/rules/` | — |
| `UserPromptSubmit` | Envoi d'un prompt, avant traitement | Oui |
| `UserPromptExpansion` | Une commande tapée se transforme en prompt | Oui |
| `MessageDisplay` | Pendant l'affichage d'un message de Claude | — |
| `PreToolUse` | **Avant un appel d'outil** | Oui (allow/deny/ask/defer) |
| `PermissionRequest` | Une décision de permission est nécessaire | Oui (allow/deny) |
| `PermissionDenied` | Le mode auto refuse un appel | (peut autoriser une nouvelle tentative) |
| `PostToolUse` | Après un appel d'outil réussi | Retour à Claude |
| `PostToolUseFailure` | Après un appel d'outil échoué | Retour à Claude |
| `PostToolBatch` | Après un lot d'appels parallèles | Retour à Claude |
| `Notification` | Claude Code envoie une notification (attente de permission, inactivité…) | — |
| `SubagentStart` / `SubagentStop` | Lancement / fin d'un sous-agent | Stop : oui |
| `TaskCreated` / `TaskCompleted` | Création / achèvement d'une tâche de la checklist | Oui |
| `Stop` | **Claude a fini de répondre** | Oui (le force à continuer) |
| `StopFailure` | Tour terminé sur une erreur d'API | — |
| `TeammateIdle` | Un coéquipier d'équipe d'agents va passer inactif | Oui |
| `ConfigChange` | Un fichier de configuration change en cours de session | Oui |
| `CwdChanged` | Le dossier de travail change (pratique avec direnv) | — |
| `DirectoryAdded` | Ajout d'un dossier avec `/add-dir` | — |
| `FileChanged` | Un fichier surveillé change sur le disque | — |
| `WorktreeCreate` / `WorktreeRemove` | Création / suppression d'un worktree (remplace git) | Oui |
| `PreCompact` / `PostCompact` | Avant / après la compaction du contexte | Pre : oui |
| `PreModelSwitch` / `PostModelSwitch` | Avant / après un changement de modèle | Pre : oui |
| `Elicitation` / `ElicitationResult` | Un serveur MCP demande une saisie / après votre réponse | Oui |
| `SessionEnd` | Fin de la session | — |

### Configurer un hook

Dans `~/.claude/settings.json` (vous), `.claude/settings.json` (projet), `.claude/settings.local.json`, un plugin, un skill ou un sous-agent :

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "npx prettier --write \"$CLAUDE_PROJECT_DIR\"/src", "timeout": 60 }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "if": "Bash(rm *)", "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/check-rm.sh" }
        ]
      }
    ],
    "Notification": [
      { "hooks": [ { "type": "command", "command": "notify-send 'Claude Code' 'Claude attend votre réponse'" } ] }
    ]
  }
}
```

- `matcher` : filtre sur le nom de l'outil (regex, ex. `Edit|Write`, `mcp__github__.*`) ou sur un champ propre à l'événement.
- `if` : une règle de permission pour filtrer plus finement (`Bash(git *)`, `Edit(*.ts)`).
- Autres champs : `timeout` (600 s par défaut pour command/http/mcp_tool), `statusMessage` (texte du spinner), `once`, `async` (en arrière-plan), `onFailure: "block"`.
- Tous les hooks correspondants s'exécutent **en parallèle**.
- Le menu **`/hooks`** affiche les hooks chargés ; le skill **`/update-config`** les écrit pour vous (« à chaque fois que Claude modifie un fichier .py, lance black »).
- Tout désactiver : `"disableAllHooks": true`.

### Répondre depuis un hook

| Code de sortie | Effet |
|----------------|-------|
| `0` | Succès ; le JSON éventuellement imprimé sur stdout est appliqué |
| `2` | **Erreur bloquante** : l'action est stoppée (sur les événements bloquants), stderr est renvoyé à Claude comme explication |
| Autre | Erreur non bloquante : l'action continue, un avertissement s'affiche |

Champs JSON de décision (exemples) :

- `PreToolUse` : `{"hookSpecificOutput": {"hookEventName": "PreToolUse", "permissionDecision": "deny", "permissionDecisionReason": "Pas de rm en dehors de /tmp"}}` (valeurs `allow`, `deny`, `ask`, `defer`).
- `Stop`, `PostToolUse`, `UserPromptSubmit`… : `{"decision": "block", "reason": "…"}`.
- `SessionStart`, `UserPromptSubmit` : ajouter du contexte pour Claude (`additionalContext`).
- `{"continue": false, "stopReason": "…"}` pour arrêter.

Exemples d'usages : formatage automatique après chaque modification, lancement des tests à la fin d'un tour, interdiction d'écrire dans certains dossiers, protection des fichiers `.env`, notification sur le bureau ou dans Slack quand Claude attend, journal de toutes les commandes exécutées, injection du contexte (ticket Jira, branche) au démarrage.

## Faire tourner Claude sans vous

### Trois façons de planifier

| | Routines (cloud) | Tâches planifiées Desktop | `/loop` |
|---|---|---|---|
| S'exécute sur | Le cloud (Anthropic par défaut) | Votre machine | Votre machine |
| Machine allumée nécessaire | Non | Oui | Oui |
| Session ouverte nécessaire | Non | Non | Oui |
| Survit aux redémarrages | Oui | Oui | Restauré avec `--resume` |
| Accès aux fichiers locaux | Non (clone frais) | Oui | Oui |
| Demandes de permission | Non (autonome) | Configurable | Héritées de la session |
| Intervalle minimum | 1 heure | 1 minute | 1 minute |

### `/loop` (dans la session)

| Saisie | Effet |
|--------|-------|
| `/loop 5m vérifie si le déploiement est fini` | Prompt répété toutes les 5 minutes (converti en cron) |
| `/loop vérifie le déploiement` | Claude choisit l'intervalle à chaque itération (1 min à 1 h) |
| `/loop` ou `/loop 15m` | Prompt de **maintenance** intégré : continuer le travail inachevé, s'occuper de la PR de la branche (commentaires, CI, conflits), puis nettoyage — ou votre fichier `loop.md` |

Vous pouvez aussi demander « rappelle-moi dans 30 minutes de… » (outils `CronCreate`/`CronList`/`CronDelete`).

### Routines (cloud)

Créées sur claude.ai/code/routines, dans l'app Desktop ou avec **`/schedule`** (« `/schedule revue des PR tous les jours à 9h` », « `/schedule nettoyer le feature flag dans une semaine` »). Déclencheurs combinables :

- **Planification** : horaire, quotidien, jours ouvrés, hebdomadaire, ou une seule fois à une date ;
- **API** : un endpoint HTTP dédié (`POST …/fire` avec jeton) pour brancher alertes, pipelines, outils internes ;
- **GitHub** : événements de pull request ou de release sur un dépôt connecté.

### Tâches planifiées de l'app Desktop

Onglet Code → **Routines** → **New routine** → **Local** : nom, description, prompt, fréquence, dossier, permissions. Tourne sur votre machine, avec vos fichiers et vos serveurs MCP.

### `/goal` : travailler jusqu'à un objectif

`/goal tous les tests de test/auth passent et le lint est propre` : Claude enchaîne les tours jusqu'à ce qu'un modèle rapide juge la condition remplie (ou impossible). `/goal` seul affiche l'état ; `/goal clear` l'annule. Fonctionne aussi en `-p`, dans Desktop et via Remote Control. Rédigez une condition que la sortie de Claude peut démontrer.

### Channels : pousser des événements dans une session

Un **channel** est un serveur MCP qui envoie des messages dans votre session en cours (résultats de CI, alertes de monitoring, messages de chat) pour que Claude réagisse pendant votre absence. Plugins pris en charge : **Telegram**, **Discord**, **iMessage**, et la démo **fakechat** (Bun requis). Lancement : `claude --channels plugin:<nom>@<marketplace>`. Liste blanche d'expéditeurs par code d'appairage. Aperçu de recherche ; activable par l'administrateur (`channelsEnabled`).

### Autres automatisations

- **Monitor** : Claude surveille un log, une PR, un dossier ou un WebSocket en arrière-plan et réagit à chaque nouvelle ligne.
- **PushNotification** : notification bureau/téléphone en fin de tâche (« préviens-moi quand c'est fini »).
- **Mode `-p`** et **Agent SDK** dans vos scripts et votre CI (voir [04](04-cli-flags-et-mode-script.md) et [17](17-sdk-entreprise-couts.md)).
- **GitHub Actions / GitLab CI** : Claude réagit aux mentions `@claude` (voir [16](16-integrations-revue-ci.md)).
- **Liens profonds** `claude-cli://open?q=…&cwd=…` dans vos runbooks et alertes.
- **Reprise automatique après limite d'usage** : Claude attend la réinitialisation puis continue seul.
