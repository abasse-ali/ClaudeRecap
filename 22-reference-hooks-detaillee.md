# 22 — Référence détaillée des hooks

[← Sommaire](README.md) · Introduction : [10 — Hooks et automatisation](10-hooks-et-automatisation.md)

## Structure : événement → groupe de matcher → handlers

```json
{
  "hooks": {
    "<Événement>": [
      {
        "matcher": "<filtre>",
        "hooks": [ { "type": "command", "command": "…" } ]
      }
    ]
  }
}
```

## Où définir des hooks

| Emplacement | Portée | Partageable |
|-------------|--------|-------------|
| `~/.claude/settings.json` | Tous vos projets | Non |
| `.claude/settings.json` | Le projet | Oui (git) |
| `.claude/settings.local.json` | Le projet, pour vous | Non |
| Réglages gérés | Organisation | Oui (admin) |
| `hooks/hooks.json` d'un plugin | Quand le plugin est actif | Oui |
| Frontmatter d'un **skill** | Reste de la session après invocation du skill | Oui |
| Frontmatter d'un **sous-agent** | Pendant l'exécution du sous-agent | Oui |

- Les hooks des différents niveaux **s'additionnent** (ils ne se remplacent pas).
- Les hooks des réglages et plugins s'exécutent aussi dans les sous-agents.
- Les sessions cloud ne lisent pas votre `~/.claude/settings.json`.
- Aucun hook d'un fichier de réglages ne s'exécute tant que l'espace de travail n'est pas approuvé.
- `allowManagedHooksOnly` (géré) bloque vos hooks utilisateur/projet/plugins ; `allowedHttpHookUrls` et `httpHookAllowedEnvVars` encadrent les hooks HTTP.

## Les matchers

| Valeur | Interprétation |
|--------|----------------|
| `"*"`, `""` ou absent | Tout |
| Lettres, chiffres, `_`, `-`, espaces, `,`, `\|` | Nom exact ou liste (`Edit\|Write`, `Edit, Write`) |
| Autres caractères | Expression régulière JavaScript non ancrée (`^Notebook`, `mcp__memory__.*` ; `^Edit$` pour un nom exact) |

### Ce que filtre le matcher selon l'événement

| Événement | Champ filtré | Valeurs possibles |
|-----------|--------------|-------------------|
| `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`, `PermissionDenied` | Nom de l'outil | `Bash`, `Edit\|Write`, `mcp__.*`, `mcp__github__.*` |
| `SessionStart` | Façon dont la session démarre | `startup`, `resume`, `clear`, `compact`, `fork` |
| `Setup` | Flag déclencheur | `init`, `maintenance` |
| `SessionEnd` | Raison de la fin | `clear`, `resume`, `logout`, `prompt_input_exit`, `other` |
| `Notification` | Type de notification | `permission_prompt`, `idle_prompt`, `auth_success`, `elicitation_dialog`, `elicitation_url_dialog`, `elicitation_complete`, `elicitation_response`, `agent_needs_input`, `agent_completed`, `quota_auto_resume_fired`, `quota_auto_resume_stale`, `quota_auto_resume_disabled` |
| `SubagentStart`, `SubagentStop` | Type d'agent | `general-purpose`, `Explore`, `Plan`, vos agents, `^mon-plugin:reviewer$` |
| `PreCompact`, `PostCompact` | Déclencheur | `manual`, `auto` |
| `PreModelSwitch`, `PostModelSwitch` | Modèle cible | `claude-opus-5`, `.*opus.*` |
| `ConfigChange` | Source | `user_settings`, `project_settings`, `local_settings`, `policy_settings`, `skills` |
| `DirectoryAdded` | Origine | `slash_command`, `register_repo_root` |
| `FileChanged` | Noms de fichiers à surveiller | `.envrc\|.env` |
| `StopFailure` | Type d'erreur | `rate_limit`, `overloaded`, `authentication_failed`, `oauth_org_not_allowed`, `account_on_hold`, `billing_error`, `invalid_request`, `model_not_found`, `server_error`, `max_output_tokens`, `cloud_credential_error`, `unknown` |
| `InstructionsLoaded` | Raison du chargement | `session_start`, `nested_traversal`, `path_glob_match`, `include`, `compact` |
| `UserPromptExpansion` | Nom de la commande | Vos skills/commandes |
| `Elicitation`, `ElicitationResult` | Serveur MCP | Vos serveurs |
| `UserPromptSubmit`, `PostToolBatch`, `Stop`, `TeammateIdle`, `TaskCreated`, `TaskCompleted`, `WorktreeCreate`, `WorktreeRemove`, `MessageDisplay`, `CwdChanged` | (pas de matcher) | Toujours déclenchés |

Filtre plus fin pour les outils : le champ **`if`** d'un handler (syntaxe des règles de permission : `"Bash(git *)"`, `"Edit(*.ts)"`, `"Edit(**/src/**)"`). Il vérifie chaque sous-commande (`npm test && git push`), les commandes dans `$()`, et ignore les affectations `VAR=x` en tête.

## Les handlers

### Champs communs

| Champ | Rôle |
|-------|------|
| `type` | `command`, `http`, `mcp_tool`, `prompt`, `agent` |
| `if` | Règle de permission qui conditionne l'exécution |
| `timeout` | Secondes (600 pour command/http/mcp_tool, 30 pour prompt, 60 pour agent) |
| `statusMessage` | Texte du spinner pendant l'exécution |
| `once` | Retirer le hook après sa première exécution réussie (skills) |

### `command`

| Champ | Rôle |
|-------|------|
| `command` | Commande shell (ou exécutable si `args` est donné) |
| `args` | Arguments : lance l'exécutable directement, sans shell |
| `async` | Exécuter en arrière-plan sans bloquer |
| `asyncRewake` | En arrière-plan, et réveiller Claude si le hook sort en code 2 (sa sortie lui est montrée) |
| `shell` | `bash` ou `powershell` |
| `onFailure` | `continue` (défaut) ou `block` (bloquer l'action si le hook échoue) |

Le JSON de l'événement arrive sur **stdin**. Utilisez `$CLAUDE_PROJECT_DIR` pour référencer vos scripts (`"$CLAUDE_PROJECT_DIR"/.claude/hooks/check.sh`), et `${CLAUDE_PLUGIN_ROOT}` dans un plugin.

### `http`

`url`, `headers` (avec `$VAR` / `${VAR}`), `allowedEnvVars` (variables autorisées dans les en-têtes), `onFailure`. Le JSON est envoyé en POST ; la réponse suit le même format de sortie JSON.

### `mcp_tool`

`server` (nom du serveur, `plugin:<plugin>:<serveur>` pour un plugin), `tool`, `input` (avec substitution `${tool_input.file_path}`). Le texte renvoyé est traité comme la sortie d'une commande.

### `prompt` et `agent`

`prompt` (avec `$ARGUMENTS` = JSON de l'événement), `model` (modèle rapide par défaut). Le modèle répond :

```json
{ "ok": true, "reason": "…", "impossible": false }
```

`ok: false` sur `Stop` fait continuer Claude avec la raison ; `impossible: true` laisse le tour se terminer. Un hook **agent** peut utiliser des outils (Read, Grep, Glob…) pour vérifier avant de répondre (expérimental). `/goal` est construit sur un hook prompt de type Stop.

## Entrées communes (JSON sur stdin)

| Champ | Contenu |
|-------|---------|
| `session_id` | ID de la session |
| `prompt_id` | ID du prompt en cours |
| `transcript_path` | Chemin de la transcription |
| `cwd` | Dossier courant |
| `scratchpad_dir` | Dossier de brouillon de la session |
| `permission_mode` | `default`, `plan`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions` |
| `effort` | `{ "level": "medium" }` |
| `hook_event_name` | Nom de l'événement |
| `agent_id`, `agent_type` | Si le hook se déclenche dans un sous-agent ou avec `--agent` |

## Sorties

### Codes de sortie

| Sortie standard | Code 0 | Code 2 | Autre code |
|-----------------|--------|--------|------------|
| JSON valide | Succès, champs appliqués | Blocage (champs lus mais ne lèvent pas le blocage) | Succès, champs appliqués |
| JSON invalide | Erreur non bloquante | Blocage, stderr = raison | Erreur non bloquante |
| Texte ou rien | Succès | **Blocage, stderr = raison donnée à Claude** | Erreur non bloquante |

### Champs JSON communs

| Champ | Rôle |
|-------|------|
| `continue` | `false` = Claude arrête tout traitement |
| `stopReason` | Message affiché quand `continue` est `false` |
| `systemMessage` | Avertissement affiché à l'utilisateur |
| `terminalSequence` | Séquence d'échappement à émettre (notification de bureau OSC 9/99/777, titre, cloche) |
| `suppressOutput` | Accepté, sans effet |
| `hookSpecificOutput.additionalContext` | Texte ajouté au contexte de Claude (sous forme de rappel système) |

### Décisions par événement

| Événement | Ce que le hook peut faire |
|-----------|---------------------------|
| `SessionStart` | `additionalContext`, `initialUserMessage` (premier message en `-p`), `sessionTitle`, `watchPaths`, `reloadSkills` ; écrire des `export` dans `$CLAUDE_ENV_FILE` pour persister des variables |
| `Setup` | Rien (ne bloque jamais) — préparation en CI |
| `InstructionsLoaded` | Rien (audit, conformité) |
| `UserPromptSubmit` | `decision: "block"` + `reason` (le prompt n'atteint pas Claude), `additionalContext` |
| `UserPromptExpansion` | Bloquer une commande (`/deploy` sans fichier d'approbation…), ajouter du contexte |
| `MessageDisplay` | `displayContent` : remplacer le texte affiché pendant le streaming |
| `PreToolUse` | `permissionDecision` : `allow` (sauter la demande), `deny`, `ask`, `defer` (mettre en pause pour un hôte `-p`/SDK) ; `permissionDecisionReason` ; **`updatedInput`** (modifier les arguments de l'outil) ; `additionalContext` |
| `PermissionRequest` | `decision.behavior` : `allow`/`deny` ; `updatedPermissions` (ajouter des règles, changer de mode…) |
| `PermissionDenied` | `retry: true` : Claude peut réessayer |
| `PostToolUse` | `decision: "block"` + `reason`, `additionalContext`, `classifierContext` (note pour le classifieur auto), **`updatedToolOutput`** (remplacer la sortie de l'outil) |
| `PostToolUseFailure` | Contexte correctif pour Claude |
| `PostToolBatch` | Contexte après un lot d'appels parallèles |
| `Notification` | Effets de bord (notifier, sonner) |
| `SubagentStart` | Contexte supplémentaire pour le sous-agent |
| `Stop`, `SubagentStop` | `decision: "block"` + `reason` **force Claude à continuer** (vérifier les tests, le lint…) ; limité à 8 blocages consécutifs (`CLAUDE_CODE_STOP_HOOK_BLOCK_CAP`) ; entrée `stop_hook_active` pour éviter les boucles |
| `StopFailure` | Journaliser l'erreur d'API (sortie ignorée sauf `terminalSequence`) |
| `TaskCreated`, `TaskCompleted` | Code 2 ou `decision: "block"` : refuser la création / la complétion (critères qualité) |
| `TeammateIdle` | Code 2 : renvoyer le coéquipier au travail ; `continue: false` l'arrête |
| `ConfigChange` | Bloquer l'application d'un changement de configuration |
| `CwdChanged`, `FileChanged` | `watchPaths` : fichiers à surveiller (ex. recharger `.envrc` avec direnv) |
| `DirectoryAdded` | Préparer le nouveau dossier (installer des dépendances) |
| `WorktreeCreate` | Imprimer le chemin du worktree créé (remplace git : SVN, Perforce, Jujutsu…) |
| `WorktreeRemove` | Nettoyer le worktree |
| `PreCompact` | Bloquer la compaction ; reçoit les consignes de `/compact` |
| `PostCompact` | Reçoit le résumé généré (`compact_summary`) |
| `PreModelSwitch` | Bloquer, demander confirmation (`ask`) ou laisser passer un changement de modèle ; reçoit le coût estimé du changement |
| `PostModelSwitch` | Donner des consignes propres au nouveau modèle |
| `SessionEnd` | Nettoyage, journalisation |
| `Elicitation` | Répondre à la place de l'utilisateur à un formulaire MCP (`action`: `accept`/`decline`/`cancel`, `content`) |
| `ElicitationResult` | Modifier ou bloquer la réponse avant envoi au serveur |

## Hooks asynchrones

`"async": true` lance le hook en arrière-plan sans bloquer Claude (seulement pour `command`). `asyncRewake` permet de réveiller Claude quand le résultat arrive (ex. lancer les tests après chaque écriture et lui signaler les échecs).

## Recettes courantes (guide officiel)

| Besoin | Événement | Idée |
|--------|-----------|------|
| Être notifié quand Claude attend | `Notification` | `notify-send`, `osascript`, ou `terminalSequence` |
| Formater après modification | `PostToolUse` + `Edit\|Write` | Prettier, Black, gofmt |
| Protéger des fichiers | `PreToolUse` + `Edit\|Write` | Code 2 si le chemin est `.env`, `package-lock.json`, `.git/` |
| Réinjecter du contexte après compaction | `SessionStart` + `compact` | Rappeler les règles critiques |
| Auditer les changements de configuration | `ConfigChange` | Journal |
| Recharger l'environnement | `CwdChanged` / `FileChanged` | direnv |
| Approuver automatiquement certaines demandes | `PermissionRequest` | `allow` pour `ExitPlanMode`, etc. |
| Vérifier avant de s'arrêter | `Stop` (hook `prompt` ou `agent`) | « Les tests passent-ils ? » |
| Journaliser chaque commande | `PreToolUse` + `Bash` | Ajouter à un fichier |

## Dépannage

- `/hooks` : voir les hooks chargés (par événement) ; `claude --debug` ou `/debug` pour le détail des exécutions.
- Hook qui ne se déclenche pas : vérifier le matcher (sensible à la casse), la confiance de l'espace de travail, `disableAllHooks`.
- JSON sans effet : un message imprimé par le profil du shell avant le JSON le rend invalide ; champs au mauvais niveau.
- Hooks et modes : un hook `allow` ne contourne pas les actions qu'aucun mode n'approuve ; une règle `deny` l'emporte.
- **Attention** : les hooks de commande s'exécutent avec **tous vos droits**. Relisez-les avant de les ajouter.
