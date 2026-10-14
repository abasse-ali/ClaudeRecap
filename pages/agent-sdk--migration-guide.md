# Migrer vers le Claude Agent SDK

> Source : https://code.claude.com/docs/en/agent-sdk/migration-guide · [← Index](../31-index-documentation-officielle.md)

*Guide de migration des SDK Claude Code TypeScript et Python vers le Claude Agent SDK.*

## Vue d'ensemble

Le Claude Code SDK a été renommé **Claude Agent SDK** et sa documentation a été réorganisée. Ce changement reflète les capacités plus larges du SDK pour construire des agents IA au-delà des tâches de programmation.

Vous migrez plutôt depuis l'OpenAI Agents SDK ? La [recette de migration depuis l'OpenAI Agents SDK](https://platform.claude.com/cookbook/claude-agent-sdk-04-migrating-from-openai-agents-sdk) fait correspondre chaque primitive au Claude Agent SDK à travers un exemple complet unique.

## Ce qui a changé

| Aspect | Ancien | Nouveau |
| :- | :- | :- |
| **Nom du paquet (TS/JS)** | `@anthropic-ai/claude-code` | `@anthropic-ai/claude-agent-sdk` |
| **Paquet Python** | `claude-code-sdk` | `claude-agent-sdk` |
| **Emplacement de la documentation** | Docs Claude Code | Docs Claude Code → section dédiée [Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview) |

## Étapes de migration

### Migrer un projet TypeScript ou JavaScript

**1. Désinstaller l'ancien paquet :**

```bash
npm uninstall @anthropic-ai/claude-code
```

**2. Installer le nouveau paquet :**

```bash
npm install @anthropic-ai/claude-agent-sdk
```

**3. Mettre à jour les imports :**

Remplacez tous les imports de `@anthropic-ai/claude-code` par `@anthropic-ai/claude-agent-sdk` :

```typescript
// Avant
import { query, tool, createSdkMcpServer } from "@anthropic-ai/claude-code";

// Après
import { query, tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
```

**4. Mettre à jour package.json :**

Si `@anthropic-ai/claude-code` figure encore dans votre `package.json`, remplacez-le par `@anthropic-ai/claude-agent-sdk` et mettez aussi à jour la plage de versions, par exemple de `"^0.0.42"` à `"^0.3.0"`.

**5. Passer en revue les [changements incompatibles](#changements-incompatibles-breaking-changes)**

Apportez les modifications de code nécessaires pour terminer la migration.

### Migrer un projet Python

**1. Désinstaller l'ancien paquet :**

```bash
pip uninstall -y claude-code-sdk
```

Si l'ancien paquet n'est pas installé, pip affiche `WARNING: Skipping claude-code-sdk as it is not installed.` C'est attendu, et vous pouvez passer à l'étape suivante.

**2. Installer le nouveau paquet :**

```bash
pip install claude-agent-sdk
```

Si `claude-code-sdk` figure dans votre `requirements.txt` ou votre `pyproject.toml`, remplacez-le par `claude-agent-sdk`.

**3. Mettre à jour les imports :**

Remplacez tous les imports de `claude_code_sdk` par `claude_agent_sdk` :

```python
# Avant
from claude_code_sdk import query, ClaudeCodeOptions

# Après
from claude_agent_sdk import query, ClaudeAgentOptions
```

**4. Passer en revue les [changements incompatibles](#changements-incompatibles-breaking-changes)**

Apportez les modifications de code nécessaires pour terminer la migration.

## Changements incompatibles (breaking changes)

> **Avertissement** : pour améliorer l'isolation et rendre la configuration explicite, le Claude Agent SDK v0.1.0 introduit des changements incompatibles pour les utilisateurs qui migrent depuis le Claude Code SDK.

### Python : ClaudeCodeOptions renommé en ClaudeAgentOptions

**Ce qui a changé :** le type Python du SDK `ClaudeCodeOptions` a été renommé `ClaudeAgentOptions`.

**Migration :**

```python
# AVANT (claude-code-sdk)
from claude_code_sdk import query, ClaudeCodeOptions

options = ClaudeCodeOptions(model="claude-opus-4-7", permission_mode="acceptEdits")

# APRÈS (claude-agent-sdk)
from claude_agent_sdk import query, ClaudeAgentOptions

options = ClaudeAgentOptions(model="claude-opus-4-7", permission_mode="acceptEdits")
```

### Le prompt système n'est plus celui par défaut

**Ce qui a changé :** le SDK n'utilise plus par défaut le prompt système de Claude Code.

**Migration :**

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

// AVANT (v0.0.x) - Utilisait par défaut le prompt système de Claude Code
const before = query({ prompt: "Hello" });

// APRÈS (v0.1.0) - Utilise par défaut un prompt système minimal
// Pour retrouver l'ancien comportement, demandez explicitement le preset de Claude Code :
const presetResult = query({
  prompt: "Hello",
  options: {
    systemPrompt: { type: "preset", preset: "claude_code" }
  }
});

// Ou utilisez un prompt système personnalisé :
const customResult = query({
  prompt: "Hello",
  options: {
    systemPrompt: "You are a helpful coding assistant"
  }
});
```

#### Python

```python
from claude_agent_sdk import query, ClaudeAgentOptions
import asyncio


async def main():
    # AVANT (v0.0.x) - Utilisait par défaut le prompt système de Claude Code
    async for message in query(prompt="Hello"):
        print(message)

    # APRÈS (v0.1.0) - Utilise par défaut un prompt système minimal
    # Pour retrouver l'ancien comportement, demandez explicitement le preset de Claude Code :
    async for message in query(
        prompt="Hello",
        options=ClaudeAgentOptions(
            system_prompt={"type": "preset", "preset": "claude_code"}  # Utiliser le preset
        ),
    ):
        print(message)

    # Ou utilisez un prompt système personnalisé :
    async for message in query(
        prompt="Hello",
        options=ClaudeAgentOptions(system_prompt="You are a helpful coding assistant"),
    ):
        print(message)


asyncio.run(main())
```

### Valeur par défaut des sources de réglages

Ce comportement par défaut a brièvement été modifié en v0.1.0 pour ne charger aucun réglage du système de fichiers, puis a été rétabli : aucune action de migration n'est donc nécessaire.

**Comportement actuel :** omettre `settingSources` dans `query()` charge les réglages utilisateur (user), projet (project) et locaux (local) du système de fichiers, comme le CLI. Cela inclut `~/.claude/settings.json`, `.claude/settings.json`, `.claude/settings.local.json`, les fichiers CLAUDE.md et les commandes personnalisées.

Pour fonctionner isolé des réglages du système de fichiers, passez `settingSources: []`, ou `setting_sources=[]` en Python. Voir [Control filesystem settings with settingSources](https://code.claude.com/docs/en/agent-sdk/claude-code-features#control-filesystem-settings-with-settingsources) pour ce que charge chaque source.

L'isolation est particulièrement importante pour les pipelines CI/CD, les applications déployées, les environnements de test et les systèmes multi-locataires (multi-tenant), où des personnalisations locales ne doivent pas s'infiltrer.

> **Note** : le SDK Python 0.1.59 et antérieur traitait une liste vide comme l'omission de l'option ; mettez donc à jour avant de vous appuyer sur `setting_sources=[]`. Voir [What settingSources does not control](https://code.claude.com/docs/en/agent-sdk/claude-code-features#what-settingsources-does-not-control) pour les entrées lues même quand `settingSources` vaut `[]`.

## Prochaines étapes

- Explorer la [vue d'ensemble de l'Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview) pour découvrir les fonctionnalités disponibles
- Consulter la [référence du SDK TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript) pour la documentation détaillée de l'API
- Parcourir la [référence du SDK Python](https://code.claude.com/docs/en/agent-sdk/python) pour la documentation spécifique à Python
- Découvrir les [outils personnalisés](https://code.claude.com/docs/en/agent-sdk/custom-tools) et l'[intégration MCP](https://code.claude.com/docs/en/agent-sdk/mcp)
