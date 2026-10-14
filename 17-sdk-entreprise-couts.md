# 17 — Agent SDK, entreprise et coûts

[← Sommaire](README.md)

## Agent SDK : Claude Code comme bibliothèque

L'**Agent SDK** permet d'intégrer la boucle agentique de Claude Code dans vos propres applications **Python** ou **TypeScript**, avec les mêmes outils, hooks, sous-agents, MCP, permissions, sessions, skills et plugins.

```bash
pip install claude-agent-sdk                 # Python
npm install @anthropic-ai/claude-agent-sdk   # TypeScript
```

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions

async def main():
    async for message in query(
        prompt="Trouve et corrige le bug dans auth.py",
        options=ClaudeAgentOptions(allowed_tools=["Read", "Edit", "Bash"]),
    ):
        print(message)

asyncio.run(main())
```

| Choisir… | Pour |
|----------|------|
| **Agent SDK** | Embarquer l'agent de Claude Code dans votre application, sur votre infrastructure |
| **Claude Code CLI** | Travail interactif, tâches ponctuelles en terminal |
| **Client SDK** (API Claude) | Appeler l'API directement |
| **Managed Agents** | Agent hébergé par Anthropic, configuré via l'API |

Fonctions du SDK : outils personnalisés (serveurs MCP en processus), sorties structurées (JSON Schema), streaming, gestion des approbations et des questions, sessions (reprise, fork, stockage externe), checkpoints de fichiers, suivi des coûts, OpenTelemetry, suivi des tâches, prompts système modifiables, plugins par chemin local.

Le mode `claude -p` (voir [04](04-cli-flags-et-mode-script.md)) est la version ligne de commande de la même chose.

## Fournisseurs et déploiement en entreprise

| Option | Description |
|--------|-------------|
| **Claude for Teams / Enterprise** | Recommandé : SSO, réglages gérés depuis la console, analytique, administration ; Enterprise ajoute capture de domaine, rôles, API de conformité |
| **Claude Console** | Facturation API à l'usage, invitations par l'admin |
| **Amazon Bedrock** | `CLAUDE_CODE_USE_BEDROCK=1`, assistant `/setup-bedrock` |
| **Google Cloud Agent Platform (Vertex AI)** | `CLAUDE_CODE_USE_VERTEX=1`, assistant `/setup-vertex` |
| **Microsoft Foundry** | Variables dédiées |
| **Claude Platform on AWS** | Plateforme Claude via AWS |
| **Claude apps gateway** | Passerelle auto-hébergée (`claude gateway`) : SSO avec votre fournisseur d'identité, politiques, plafonds de dépense, routage vers Bedrock/Google Cloud/Foundry |
| **Autres passerelles LLM** | `ANTHROPIC_BASE_URL` vers votre proxy (LiteLLM, etc.) |
| **Environnements auto-hébergés** | Les sessions « cloud » tournent sur vos machines (`claude self-hosted-runner`) |
| **Dev containers, launcher d'entreprise, proxy réseau, ZDR, HIPAA** | Pages dédiées de la doc d'administration |

Certaines fonctions dépendent de claude.ai et ne sont pas disponibles chez les fournisseurs tiers (artifacts, Remote Control, routines, `/import`, skills synchronisés…).

## Réglages gérés (managed settings)

L'organisation impose des réglages **que l'utilisateur ne peut pas modifier** :

- Livraison : fichier `managed-settings.json` dans un dossier système, **MDM** / politiques OS, ou **réglages gérés par le serveur** depuis la console d'administration claude.ai.
- Exemples : règles de permission, `permissions.disableBypassPermissionsMode`, `disableAutoMode`, modèles autorisés (`availableModels`), plafond d'effort, serveurs MCP autorisés/interdits (`managed-mcp.json`, `allowedMcpServers`), marketplaces et plugins imposés ou bloqués, `allowManagedHooksOnly`, `allowManagedPermissionRulesOnly`, `channelsEnabled`, `forceLoginMethod`, `forceLoginOrgUUID`, `minimumVersion`, `CLAUDE.md` d'organisation, skills d'entreprise, variables `env` (télémétrie)…
- `/status` montre la source gérée appliquée.

## Suivre la consommation et les coûts

| Outil | Ce qu'il montre |
|-------|-----------------|
| `/usage` (alias `/cost`, `/stats`) | Coût de la session (API), limites du forfait avec temps avant réinitialisation, **ventilation de ce qui consomme** (cache manqué, long contexte, sous-agents…), statistiques du cache de prompt, onglet Stats |
| Spinner | Temps écoulé et jetons générés pendant le tour |
| `/context` | Ce qui occupe la fenêtre de contexte |
| `/insights` | Rapport HTML sur vos habitudes et points de friction |
| `/skill-doctor` | Coût en contexte de chaque skill |
| `/usage-credits` | Activer ou demander des **crédits d'usage** pour dépasser les limites du forfait |
| `/rate-limit-options` | Que faire quand une limite est atteinte |
| Barre d'état | Afficher le coût ou le contexte en permanence |
| **Analytics** | Tableau de bord d'équipe : claude.ai/analytics/claude-code (Team/Enterprise : utilisateurs actifs, lignes acceptées, PR et lignes livrées avec Claude, classement, export CSV) ; platform.claude.com/claude-code (Console) |
| **OpenTelemetry** | Export de métriques, logs et traces (`CLAUDE_CODE_ENABLE_TELEMETRY`, `OTEL_*`) vers votre outil d'observabilité |

### Réduire les coûts

- `/clear` entre deux tâches sans rapport ; `/compact` avec des consignes.
- Choisir le bon modèle (`haiku`/`sonnet` pour les tâches simples) et le bon **effort**.
- Confier l'exploration verbeuse à des **sous-agents** (on peut les faire tourner sur un modèle moins cher).
- Garder `CLAUDE.md` court ; déplacer les procédures dans des **skills** (chargées à la demande) ; désactiver les serveurs MCP et plugins inutiles.
- Hooks de pré-traitement (filtrer les logs avant de les donner à Claude).
- En équipe : plafonds de dépense (Console, gateway), limites par utilisateur.

## Données et confidentialité

- `/privacy-settings` (Pro/Max) pour vos préférences d'utilisation des données.
- Télémétrie désactivable (`DISABLE_TELEMETRY`, `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`).
- Rapports de bug (`/bug`) : vous choisissez ce qui est envoyé après un écran de consentement.
- Rétention des transcriptions locales : `cleanupPeriodDays` (30 jours par défaut) ; `claude purge` pour tout effacer.
- Zero Data Retention possible pour les organisations éligibles.
