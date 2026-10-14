# 19 — Référence : tous les réglages (`settings.json`)

[← Sommaire](README.md)

Liste complète des **245 clés de réglages** de Claude Code (v2.1.299), classées par thème. Où les mettre :

| Abréviation | Fichiers acceptés |
|-------------|-------------------|
| **Tous** | `~/.claude/settings.json` (utilisateur), `.claude/settings.json` (projet), `.claude/settings.local.json` (local), réglages gérés |
| **U/L/G** | Utilisateur, local ou géré |
| **U/G** | Utilisateur ou géré |
| **U** | Utilisateur seulement |
| **Géré** | Réglages gérés par l'organisation seulement |
| **Global** | `~/.claude.json` (configuration globale) |

Modifier : `/config`, `/config clé=valeur`, `/update-config`, ou à la main (JSON strict). Voir aussi [12](12-modeles-et-personnalisation.md#fichiers-de-réglages) pour la priorité entre fichiers.

## Modèle et réponses

| Clé | Rôle | Portée |
|-----|------|--------|
| `model` | Modèle de démarrage | Tous |
| `availableModels` | Restreindre les modèles sélectionnables | Tous |
| `availableModelsMatch` | Chaque ID de `availableModels` n'autorise que la version nommée | Géré |
| `deniedModels` | Bloquer des modèles, même autorisés par `availableModels` | Géré |
| `enforceAvailableModels` | Garder le choix « Default » de `/model` dans la liste autorisée | Tous |
| `modelPicker` | Modèles listés par `/model`, ordre et libellés personnalisés | U/G |
| `modelOverrides` | Correspondance d'IDs de modèles vers ceux du fournisseur (ARN Bedrock…) | Tous |
| `modelPricing` | Afficher les coûts aux tarifs négociés de l'organisation | Géré |
| `modelSettings` | Effort et seuil d'auto-compaction enregistrés par modèle, ou plafond d'effort par modèle | Tous |
| `effortLevel` | Effort par défaut des modèles sans niveau enregistré | Tous |
| `maxEffortLevel` | Plafonner l'effort (global ou par modèle) | Tous |
| `ultracode` | Claude planifie un workflow pour chaque tâche importante | Tous |
| `fallbackModel` | Modèles de secours si le principal est surchargé | Tous |
| `switchModelsOnFlag` | Changer de modèle ou faire une pause quand un classifieur de sécurité signale une requête | Tous |
| `fastMode` | Activer le mode rapide quand il est disponible | Tous |
| `fastModePerSessionOptIn` | Obliger à réactiver le mode rapide à chaque session | Tous |
| `advisorModel` | Modèle de l'advisor | Tous |
| `alwaysThinkingEnabled` | Désactiver la réflexion étendue pour toutes les sessions | Tous |
| `showThinkingSummaries` | Voir des résumés de la réflexion au lieu d'un bloc replié | Tous |
| `outputStyle` | Style de sortie (rôle, ton, format) | Tous |
| `language` | Langue des réponses (ex. `"french"`) | Tous |
| `promptCacheTtl` | Durée du cache de prompt de la conversation principale | Tous |
| `subagentPromptCacheTtl` | Durée du cache pour les sous-agents et requêtes annexes | Tous |

## Permissions

| Clé | Rôle | Portée |
|-----|------|--------|
| `permissions` | Règles allow/ask/deny et mode de départ | Tous |
| `permissions.allow` | Utilisations d'outils approuvées sans demande | Tous |
| `permissions.ask` | Toujours demander avant ces utilisations | Tous |
| `permissions.deny` | Bloquer ces utilisations (dont la lecture de fichiers secrets) | Tous |
| `permissions.additionalDirectories` | Dossiers supplémentaires accessibles | Tous |
| `permissions.defaultMode` | Mode de permission des nouvelles sessions | Tous |
| `permissions.disableBypassPermissionsMode` | Interdire le mode bypassPermissions | Tous |
| `permissions.blockReadsOutsideWorkingDirectories` | Refuser les lectures hors des dossiers de travail dans tous les modes | Tous |
| `allowManagedPermissionRulesOnly` | Seules les règles des réglages gérés s'appliquent | Géré |
| `disableAutoMode` | Retirer le mode auto du cycle | Tous |
| `autoMode` | Ajouter vos règles allow/deny au classifieur du mode auto | U/G |
| `autoMode.classifyAllShell` | Faire passer toutes les commandes shell par le classifieur | U/G |
| `useAutoModeDuringPlan` | Le classifieur examine les commandes en mode plan (`false` = demandes) | U/L/G |
| `skipAutoPermissionPrompt` | Masquer l'avis affiché la première fois qu'on entre en mode auto | U/G |
| `skipDangerousModePermissionPrompt` | Masquer la confirmation avant le mode bypassPermissions | U/L/G |

## Sandbox (Bash isolé)

| Clé | Rôle | Portée |
|-----|------|--------|
| `sandbox` | Isoler les commandes Bash (fichiers et réseau) sur macOS, Linux, WSL2 | Tous |
| `sandbox.enabled` | Activer la sandbox | Tous |
| `sandbox.autoAllowBashIfSandboxed` | Commandes sandboxées sans demande de permission | Tous |
| `sandbox.excludedCommands` | Commandes pouvant tourner hors de la sandbox | Tous |
| `sandbox.allowUnsandboxedCommands` | Autoriser ou interdire de réessayer hors sandbox | Tous |
| `sandbox.failIfUnavailable` | Refuser de démarrer si la sandbox est impossible | Tous |
| `sandbox.ignoreViolations` | Taire les violations attendues | Tous |
| `sandbox.enableWeakerNestedSandbox` | Sandbox Linux dans un conteneur non privilégié | Tous |
| `sandbox.enableWeakerNetworkIsolation` | Laisser `gh`, `gcloud`, `terraform` vérifier TLS derrière un proxy MITM (macOS) | Tous |
| `sandbox.allowAppleEvents` | Autoriser les Apple Events (macOS) | U/G |
| `sandbox.ripgrep` | Utiliser votre propre binaire ripgrep | U/G |
| `sandbox.bwrapPath` | Chemin de bubblewrap | Géré |
| `sandbox.socatPath` | Chemin de `socat` | Géré |
| `sandbox.filesystem` | Chemins lisibles/écrivables | Tous |
| `sandbox.filesystem.allowWrite` | Ajouter des chemins en écriture | Tous |
| `sandbox.filesystem.denyWrite` | Interdire l'écriture | Tous |
| `sandbox.filesystem.denyRead` | Interdire la lecture | Tous |
| `sandbox.filesystem.allowRead` | Rouvrir la lecture dans une zone bloquée | Tous |
| `sandbox.filesystem.allowManagedReadPathsOnly` | Empêcher de rouvrir ce que l'organisation bloque | Géré |
| `sandbox.filesystem.disabled` | Désactiver l'isolation des fichiers en gardant celle du réseau | U/G |
| `sandbox.network` | Hôtes, ports et sockets accessibles | Tous |
| `sandbox.network.allowedDomains` | Domaines pré-autorisés | Tous |
| `sandbox.network.deniedDomains` | Domaines bloqués (même sous un joker autorisé) | Tous |
| `sandbox.network.strictAllowlist` | Refuser au lieu de demander hors liste | U/G |
| `sandbox.network.allowManagedDomainsOnly` | Verrouiller la liste sur les réglages gérés | Géré |
| `sandbox.network.allowLocalBinding` | Écouter sur des ports et joindre localhost (macOS) | Tous |
| `sandbox.network.allowUnixSockets` | Sockets Unix autorisés (macOS) | Tous |
| `sandbox.network.allowAllUnixSockets` | Tous les sockets Unix | Tous |
| `sandbox.network.allowMachLookup` | Services XPC (simulateur iOS, Playwright) sur macOS | Tous |
| `sandbox.network.httpProxyPort` / `socksProxyPort` | Passer par votre propre proxy HTTP / SOCKS | Tous |
| `sandbox.network.tlsTerminate` | Le proxy termine TLS pour lire les requêtes HTTPS | U/G |
| `sandbox.credentials` | Masquer ou cacher les identifiants dans la sandbox | Tous |
| `sandbox.credentials.files` | Bloquer ou masquer la lecture d'un fichier d'identifiants | Tous |
| `sandbox.credentials.envVars` | Supprimer ou masquer une variable d'environnement | Tous |
| `sandbox.credentials.awsPairs` | Associer des variables de clés AWS pour la re-signature | U/G |
| `sandbox.credentials.sigv4` | Comportement des requêtes AWS en streaming, pré-signées ou SigV4A | U/G |
| `sandbox.credentials.allowPlaintextInject` | Laisser des identifiants masqués atteindre des services HTTP non chiffrés de test | U/G |

## Mémoire et contexte

| Clé | Rôle | Portée |
|-----|------|--------|
| `autoCompactEnabled` | Activer/désactiver la compaction automatique | Tous |
| `autoCompactWindow` | Taux de remplissage avant compaction | Tous |
| `idleCompaction` | Compacter une longue conversation pendant l'inactivité | Tous |
| `autoMemoryEnabled` | Activer/désactiver la mémoire automatique | Tous |
| `autoMemoryDirectory` | Dossier de la mémoire automatique | Tous |
| `claudeMd` | Instructions `CLAUDE.md` d'organisation injectées | Géré |
| `claudeMdExcludes` | Ignorer certains `CLAUDE.md` | Tous |
| `env` | Variables d'environnement pour chaque session et ses sous-processus | Tous |
| `fileCheckpointingEnabled` | Instantanés de fichiers utilisés par `/rewind` | Tous |
| `plansDirectory` | Dossier des fichiers de plan | Tous |
| `bashOutputMaxChars` | Taille de sortie de commande reçue directement | Tous |
| `skillListingBudgetFraction` | Part du contexte réservée à la liste des skills | Tous |
| `skillListingMaxDescChars` | Longueur max de chaque description de skill | Tous |
| `taskOutputMaxChars` | Retiré (v2.1.277) | — |

## Interface et terminal

| Clé | Rôle | Portée |
|-----|------|--------|
| `theme` | Thème de couleurs | Tous |
| `tui` | Rendu plein écran ou classique | Tous |
| `viewMode` | Vue par défaut : default, verbose ou focus | Tous |
| `verbose` | Sortie complète des outils | Tous |
| `editorMode` | Mode Vim dans le prompt | Tous |
| `vimInsertModeRemaps` | Séquence comme `jj` → Échap | U/G |
| `statusLine` | Commande qui produit la barre d'état | Tous |
| `subagentStatusLine` | Réécrire les lignes des sous-agents avec votre commande | Tous |
| `spinnerVerbs` | Verbes du spinner | Tous |
| `spinnerTipsEnabled` | Astuces dans le spinner | Tous |
| `spinnerTipsOverride` | Vos propres astuces | Tous |
| `showTurnDuration` | Masquer « Cooked for … » après chaque réponse | Tous |
| `prefersReducedMotion` | Réduire les animations | Tous |
| `syntaxHighlightingDisabled` | Désactiver la coloration syntaxique | Tous |
| `maxProseWidth` | Largeur max du texte dans un terminal large | Tous |
| `timeFormat` | Heures en 12 h / 24 h / UTC / motif strftime | Tous |
| `timeZone` | Fuseau horaire affiché | Tous |
| `terminalProgressBarEnabled` | Barre de progression du terminal | Tous |
| `terminalTitleFromRename` | `/rename` et `--name` changent le titre de l'onglet | Tous |
| `autoScrollEnabled` | Suivre la sortie en bas (plein écran) | Tous |
| `wheelScrollAccelerationEnabled` | Accélération de la molette (plein écran) | Tous |
| `axScreenReader` | Sortie pour lecteurs d'écran | Tous |
| `companyAnnouncements` | Annonces de l'organisation au démarrage | Tous |
| `promptSuggestionEnabled` | Suggestions de prompt grisées | Tous |
| `emojiCompletionEnabled` | Suggestions d'emoji `:code:` | Tous |
| `spellcheck` | Correcteur orthographique du prompt | U/G |
| `fileSuggestion` | Votre propre commande d'autocomplétion `@` | Tous |
| `respectGitignore` | Exclure les fichiers ignorés par git du sélecteur `@` | Tous |
| `footerLinksRegexes` | Rendre cliquables des identifiants (tickets, revues) sous le prompt | U/G |
| `defaultShell` | Bash ou PowerShell pour les commandes `!` | Tous |
| `respondToBashCommands` | Claude répond (ou non) après une commande `!` | Tous |
| `showClearContextOnPlanAccept` | Option « effacer le contexte » à l'acceptation d'un plan | Tous |
| `askUserQuestionTimeout` | Une question sans réponse continue après un délai | U/G |
| `autoContinueAtUsageLimit` | Reprendre seul après la réinitialisation d'une limite | U/G |
| `bashEditDiffEnabled` | Enregistrer les fichiers modifiés par une commande Bash | U/G |
| `dialogExpiry` | Délai avant d'annuler une boîte transmise à Remote Control ou au SDK | U/G |
| `voice` | Dictée vocale et mode (maintenir/taper) | Tous |
| `voiceEnabled` | Ancienne forme d'activation de la dictée | Tous |
| `keybindingFlavor` | Obsolète, sans effet | — |

## Git et attribution

| Clé | Rôle | Portée |
|-----|------|--------|
| `attribution` | Personnaliser l'attribution des commits et PR | Tous |
| `attribution.commit` | Modifier/masquer la mention ajoutée aux commits | Tous |
| `attribution.pr` | Modifier/masquer la ligne dans la description des PR | Tous |
| `attribution.sessionUrl` | Retirer le lien de session claude.ai des commits | Tous |
| `includeCoAuthoredBy` | Obsolète (→ `attribution`) | Tous |
| `includeGitInstructions` | Retirer les instructions de commit/PR intégrées | Tous |
| `prUrlTemplate` | Liens de PR vers votre outil de revue interne | Tous |

## Hooks et automatisation

| Clé | Rôle | Portée |
|-----|------|--------|
| `hooks` | Vos hooks | Tous |
| `disableAllHooks` | Désactiver hooks, barre d'état et suggestion `@` personnalisées | Tous |
| `allowManagedHooksOnly` | Seuls les hooks de l'organisation s'exécutent | Géré |
| `allowedHttpHookUrls` | URL autorisées pour les hooks HTTP | Tous |
| `httpHookAllowedEnvVars` | Variables utilisables dans les en-têtes des hooks HTTP | Tous |
| `enableWorkflows` / `disableWorkflows` | Activer/désactiver les workflows dynamiques | Tous |
| `workflowKeywordTriggerEnabled` | Le mot `ultracode` déclenche un workflow | Tous |
| `workflowSizeGuideline` | Nombre d'agents visé par les workflows | Tous |

## Plugins et skills

| Clé | Rôle | Portée |
|-----|------|--------|
| `enabledPlugins` | Activer/désactiver des plugins par portée | Tous |
| `extraKnownMarketplaces` | Déclarer des marketplaces pour un dépôt ou une organisation | Tous |
| `pluginConfigs` | Réponses données aux dialogues de configuration des plugins | U/G |
| `disableBundledSkills` | Désactiver les skills et workflows intégrés | Tous |
| `disableSkillShellExecution` | Empêcher les skills d'exécuter du shell en ligne | Tous |
| `skillOverrides` | Masquer ou réduire un skill sans modifier son `SKILL.md` | Tous |
| `syncClaudeAiSkills` / `syncClaudeAiPlugins` | Arrêter de charger les skills / plugins de votre compte claude.ai | U/L/G |
| `prependPlugins` / `appendPlugins` | Mods de l'organisation exécutés avant / après ceux de l'utilisateur | U/G |
| `blockedMarketplaces` | Bloquer des sources de marketplaces | Géré |
| `strictKnownMarketplaces` | Liste blanche des marketplaces | Géré |
| `disableCommandPluginSources` | Bloquer les plugins installés via une commande | Géré |
| `pluginSuggestionMarketplaces` | Marketplaces pouvant suggérer des plugins | Géré |
| `pluginTrustMessage` | Texte ajouté à l'avertissement de confiance | Géré |
| `strictPluginOnlyCustomization` (+ `.skills`, `.agents`, `.hooks`, `.mcp`) | N'accepter skills/agents/hooks/MCP que depuis les plugins et réglages gérés | Géré |
| `channelsEnabled` | Autoriser les channels | Géré |
| `allowedChannelPlugins` | Liste des plugins de channel autorisés | Géré |

## MCP

| Clé | Rôle | Portée |
|-----|------|--------|
| `enableAllProjectMcpServers` | Approuver tous les serveurs de `.mcp.json` | Tous |
| `enabledMcpjsonServers` / `disabledMcpjsonServers` | Approuver / refuser certains serveurs de `.mcp.json` | Tous |
| `allowedMcpServers` / `deniedMcpServers` | Liste blanche / noire des serveurs MCP | Tous |
| `allowManagedMcpServersOnly` | Seule la liste gérée s'applique | Géré |
| `managedMcpServers` | Serveurs distants fournis à tous les utilisateurs | Géré |
| `disableClaudeAiConnectors` | Ne pas récupérer les connecteurs claude.ai | Tous |
| `allowAllClaudeAiMcps` | Charger les connecteurs claude.ai avec un `managed-mcp.json` | Géré |
| `allowClaudeInChromeWithManagedMcp` | Laisser Claude in Chrome tourner avec un `managed-mcp.json` | Géré |

## Agents, sessions et worktrees

| Clé | Rôle | Portée |
|-----|------|--------|
| `agent` | Démarrer chaque session comme un sous-agent nommé | Tous |
| `teammateMode` | Affichage des coéquipiers d'équipe d'agents | Tous |
| `disableAgentView` | Désactiver les agents d'arrière-plan et l'agent view | Tous |
| `crossSessionInbound` | Accepter, signaler ou refuser les messages de vos autres sessions | Tous |
| `isolatePeerMachines` | Demander avant d'écrire à une session sur une autre machine | Tous |
| `processWrapper` | Lancer les processus de fond via un launcher d'entreprise | U/G |
| `worktree` | Création des worktrees | Tous |
| `worktree.baseRef` | Partir de la branche par défaut distante ou du HEAD local | Tous |
| `worktree.bgIsolation` | Sessions de fond sans worktree | Tous |
| `worktree.location` | Emplacement des worktrees des sessions SSH | U |
| `worktree.sparsePaths` | N'extraire que certains dossiers | Tous |
| `worktree.symlinkDirectories` | Lier (symlink) de gros dossiers au lieu de les dupliquer | Tous |

## Distant, Desktop et notifications

| Clé | Rôle | Portée |
|-----|------|--------|
| `remoteControlAtStartup` | Connecter Remote Control au démarrage | Tous |
| `disableRemoteControl` | Désactiver Remote Control partout | Tous |
| `remote.defaultEnvironmentId` | Environnement cloud par défaut de `claude --cloud` | Tous |
| `agentPushNotifEnabled` | Claude peut envoyer une notification push sur le téléphone | Tous |
| `inputNeededNotifEnabled` | Notification push quand Claude vous attend | Tous |
| `preferredNotifChannel` | Cloche du terminal ou notification de bureau | Tous |
| `awaySummaryEnabled` | Récapitulatif au retour devant le terminal | Tous |
| `enableArtifact` / `disableArtifact` | Désactiver l'outil Artifact (`disableArtifact` obsolète) | Tous |
| `disableDeepLinkRegistration` | Ne pas enregistrer le gestionnaire `claude-cli://` | Tous |
| `sshConfigs` | Connexions SSH dans le menu d'environnement de Desktop | U/G |
| `sshHostAllowlist` | Hôtes SSH autorisés dans Desktop | Géré |
| `disableDesktopLocalSessions` | Désactiver les sessions locales de Desktop (garder SSH et cloud) | Géré |

## Authentification et fournisseurs

| Clé | Rôle | Portée |
|-----|------|--------|
| `apiKeyHelper` | Générer la clé API avec votre propre commande | Tous |
| `awsAuthRefresh` / `awsCredentialExport` | Rafraîchir / fournir les identifiants Bedrock | Tous |
| `gcpAuthRefresh` | Rafraîchir les identifiants Google Cloud | Tous |
| `otelHeadersHelper` | En-têtes OpenTelemetry tournants | Tous |
| `forceLoginMethod` | Limiter la connexion à claude.ai, Console ou une gateway | Tous |
| `forceLoginOrgUUID` | Restreindre les connexions claude.ai à votre organisation | Tous |
| `forceLoginGatewayUrl` | URL de gateway de l'écran de connexion | U/G |
| `allowedProviders` | Fournisseurs d'API autorisés | Géré |
| `gatewayInternalNetworks` | Joindre une gateway sur des IP publiques internes | Géré |

## Mises à jour

| Clé | Rôle | Portée |
|-----|------|--------|
| `autoUpdatesChannel` | Canal `stable` au lieu de `latest` | Tous |
| `minimumVersion` | Ne pas installer en dessous d'une version | Tous |
| `requiredMinimumVersion` / `requiredMaximumVersion` | Refuser de démarrer hors de la plage de versions | Géré |

## Outils (navigateur et simulateur de Desktop)

| Clé | Rôle | Portée |
|-----|------|--------|
| `browserExternalPageTools` | Interdire les outils de Claude sur les pages externes du navigateur intégré | Géré |
| `disableBrowserExternalNavigation` | Limiter le navigateur intégré à localhost | Géré |
| `disableMobileSimulatorTools` | Bloquer les outils de Claude dans le simulateur iOS | Géré |

## Confidentialité et télémétrie

| Clé | Rôle | Portée |
|-----|------|--------|
| `cleanupPeriodDays` | Durée de conservation des transcriptions (30 jours par défaut) | Tous |
| `desktopSessionCleanupPeriodDays` | Idem pour Desktop et Cowork | U/G |
| `feedbackDrafts` | Brouillons de retours rédigés par Claude | U/G |
| `feedbackSurveyRate` | Fréquence du sondage de qualité | Tous |
| `skipWebFetchPreflight` | Sauter la vérification d'hôte de WebFetch si Anthropic est injoignable | Tous |

## Entreprise et réglages gérés

| Clé | Rôle | Portée |
|-----|------|--------|
| `managedSourcesBehavior` | Combiner toutes les sources gérées au lieu de la plus prioritaire | Géré |
| `parentSettingsBehavior` | Appliquer ou ignorer les restrictions d'un hôte SDK/IDE | Géré |
| `policyHelper` (+ `.path`, `.refreshIntervalMs`, `.timeoutMs`) | Exécutable qui calcule les réglages gérés au démarrage | Géré |
| `forceRemoteSettingsRefresh` | Attendre des réglages serveur à jour avant de démarrer | Géré |
| `disableSideloadFlags` | Refuser les flags qui chargent plugins, agents, MCP à la volée | Géré |
| `wslInheritsWindowsSettings` | WSL lit les réglages gérés de Windows | Géré |

## Configuration globale (`~/.claude.json`)

| Clé | Rôle |
|-----|------|
| `autoConnectIde` | Se connecter automatiquement à VS Code / JetBrains depuis un terminal externe |
| `autoInstallIdeExtension` | Désactiver l'installation automatique de l'extension IDE |
| `claudeInChromeDefaultEnabled` | Activer Chrome au démarrage (CLI et VS Code) |
| `copyFullResponse` | `/copy` copie toute la réponse sans sélecteur |
| `copyOnSelect` | Copie automatique de la sélection à la souris |
| `defaultToAgentsView` | `claude` sans argument ouvre l'agent view |
| `diffTool` | Diffs dans l'IDE ou dans le terminal |
| `externalEditorContext` | Afficher la dernière réponse en commentaire avec `Ctrl+G` |
| `leftArrowOpensAgents` | Désactiver le raccourci `←` vers l'agent view |
| `prStatusFooterEnabled` | Désactiver le badge de PR du pied de page |
| `permissionExplainerEnabled` | Retiré (v2.1.257) |
| `teammateDefaultModel` | Retiré (v2.1.234) |

## Exemple complet

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "opus",
  "language": "french",
  "outputStyle": "Concise",
  "permissions": {
    "defaultMode": "acceptEdits",
    "allow": ["Bash(npm run test *)", "Bash(git diff *)"],
    "deny": ["Read(./.env)", "Read(./secrets/**)"]
  },
  "env": { "NODE_ENV": "development" },
  "hooks": {
    "PostToolUse": [{ "matcher": "Edit|Write", "hooks": [{ "type": "command", "command": "npx prettier --write ." }] }]
  },
  "statusLine": { "type": "command", "command": "~/.claude/statusline.sh" },
  "attribution": { "commit": "", "pr": "" },
  "cleanupPeriodDays": 60,
  "spinnerTipsEnabled": false
}
```
