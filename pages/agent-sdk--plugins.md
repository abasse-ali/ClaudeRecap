# Les plugins dans le SDK

> Source : https://code.claude.com/docs/en/agent-sdk/plugins · [← Index](../31-index-documentation-officielle.md)

> Charger des plugins personnalisés pour étendre Claude Code avec des skills, des agents, des hooks et des serveurs MCP via l'Agent SDK.

Les plugins permettent d'étendre Claude Code avec des fonctionnalités personnalisées, partageables entre projets. Via l'Agent SDK, vous pouvez charger par programme des plugins depuis des répertoires locaux pour ajouter des capacités à vos sessions d'agent. Un plugin peut contenir :

- **Skills** : capacités que Claude invoque de manière autonome quand c'est pertinent. Vous pouvez aussi invoquer directement un skill de plugin avec `/plugin-name:skill-name`.
- **Agents** : sous-agents spécialisés pour des tâches précises.
- **Hooks** : gestionnaires d'événements qui réagissent à l'utilisation d'outils et à d'autres événements.
- **Serveurs MCP** : intégrations d'outils externes via le Model Context Protocol.

Pour tout savoir sur la structure d'un plugin et sur sa création, voir [Plugins](https://code.claude.com/docs/en/plugins/overview).

## Charger des plugins

Chargez les plugins en fournissant leurs chemins sur le système de fichiers local dans la configuration des options.
- Le champ `type` doit valoir `"local"`, seule valeur acceptée par le SDK.
- Le SDK prend en charge le chargement de plusieurs plugins depuis des emplacements différents.

Pour utiliser un plugin distribué via une [marketplace](https://code.claude.com/docs/en/plugins/overview) ou un dépôt distant, téléchargez-le d'abord puis fournissez le chemin du répertoire local. Pour l'arborescence attendue d'un plugin, voir la [Référence de structure d'un plugin](#référence-de-structure-dun-plugin) ci-dessous.

**TypeScript**

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Hello",
  options: {
    plugins: [
      { type: "local", path: "./my-plugin" },
      { type: "local", path: "/absolute/path/to/another-plugin" }
    ]
  }
})) {
  // Les commandes, agents et autres fonctionnalités du plugin sont maintenant disponibles
}
```

**Python**

```python Python theme={null}
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions


async def main():
    async for message in query(
        prompt="Hello",
        options=ClaudeAgentOptions(
            plugins=[
                {"type": "local", "path": "./my-plugin"},
                {"type": "local", "path": "/absolute/path/to/another-plugin"},
            ]
        ),
    ):
        # Les commandes, agents et autres fonctionnalités du plugin sont maintenant disponibles
        pass


asyncio.run(main())
```

### Spécification des chemins

Les chemins de plugin peuvent être :

- **Relatifs** : résolus par rapport à l'option `cwd` (par exemple `"./plugins/my-plugin"`).
- **Absolus** : chemins complets du système de fichiers (par exemple `"/home/user/plugins/my-plugin"`).

> **Note** : le chemin doit pointer vers le répertoire racine du plugin, c'est-à-dire le parent de `skills/`, `agents/`, `hooks/`, `commands/` ou `.claude-plugin/`.

## Vérifier l'installation des plugins

Quand les plugins se chargent correctement, ils apparaissent dans le message système d'initialisation (`system` / `init`). Vous pouvez vérifier que vos plugins sont disponibles :

**TypeScript**

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Hello",
  options: {
    plugins: [{ type: "local", path: "./my-plugin" }]
  }
})) {
  if (message.type === "system" && message.subtype === "init") {
    // Vérifier les plugins chargés
    console.log("Plugins:", message.plugins);
    // Exemple : [{ name: "my-plugin", path: "/absolute/path/to/my-plugin" }]

    // Les skills de plugin apparaissent préfixés par le nom du plugin
    console.log("Skills:", message.skills);
    // Exemple : ["my-plugin:greet"]

    // Les commandes de plugin utilisent le même préfixe, et les skills apparaissent ici aussi
    console.log("Commands:", message.slash_commands);
    // Exemple : ["compact", "context", "my-plugin:custom-command", "my-plugin:greet"]
  }
}
```

**Python**

```python Python theme={null}
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, SystemMessage


async def main():
    async for message in query(
        prompt="Hello",
        options=ClaudeAgentOptions(
            plugins=[{"type": "local", "path": "./my-plugin"}]
        ),
    ):
        if isinstance(message, SystemMessage) and message.subtype == "init":
            # Vérifier les plugins chargés
            print("Plugins:", message.data.get("plugins"))
            # Exemple : [{"name": "my-plugin", "path": "/absolute/path/to/my-plugin"}]

            # Les skills de plugin apparaissent préfixés par le nom du plugin
            print("Skills:", message.data.get("skills"))
            # Exemple : ["my-plugin:greet"]

            # Les commandes de plugin utilisent le même préfixe, et les skills apparaissent ici aussi
            print("Commands:", message.data.get("slash_commands"))
            # Exemple : ["compact", "context", "my-plugin:custom-command", "my-plugin:greet"]


asyncio.run(main())
```

Champs du message d'init utiles ici :
- `plugins` : liste des plugins chargés (`name`, `path`).
- `skills` : skills disponibles, ceux des plugins préfixés `nom-du-plugin:`.
- `slash_commands` : commandes disponibles (commandes intégrées, commandes de plugin préfixées, et skills).

## Utiliser les skills de plugin

Les skills issus de plugins reçoivent automatiquement un espace de noms correspondant au nom du plugin, pour éviter les conflits. Pour en invoquer un directement, envoyez `/plugin-name:skill-name` comme prompt.

**TypeScript**

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

// Charger un plugin doté d'un skill /greet personnalisé
for await (const message of query({
  prompt: "/my-plugin:greet", // Utiliser le skill du plugin avec son espace de noms
  options: {
    plugins: [{ type: "local", path: "./my-plugin" }]
  }
})) {
  // Claude exécute le skill de salutation personnalisé du plugin
  if (message.type === "assistant") {
    console.log(message.message.content);
  }
}
```

**Python**

```python Python theme={null}
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage, TextBlock


async def main():
    # Charger un plugin doté d'un skill /greet personnalisé
    async for message in query(
        prompt="/my-plugin:greet",  # Utiliser le skill du plugin avec son espace de noms
        options=ClaudeAgentOptions(
            plugins=[{"type": "local", "path": "./my-plugin"}]
        ),
    ):
        # Claude exécute le skill de salutation personnalisé du plugin
        if isinstance(message, AssistantMessage):
            for block in message.content:
                if isinstance(block, TextBlock):
                    print(f"Claude: {block.text}")


asyncio.run(main())
```

> **Note** : si vous avez installé un plugin via la CLI (par exemple `/plugin install my-plugin@marketplace`), vous pouvez quand même l'utiliser dans le SDK en fournissant son chemin d'installation. Les plugins installés via la CLI se trouvent dans `~/.claude/plugins/`.

## Exemple complet

Voici un exemple complet de chargement et d'utilisation d'un plugin :

**TypeScript**

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";
import { fileURLToPath } from "node:url";

async function runWithPlugin() {
  const pluginPath = fileURLToPath(new URL("./plugins/my-plugin", import.meta.url));

  console.log("Loading plugin from:", pluginPath);

  for await (const message of query({
    prompt: "What custom commands do you have available?",
    options: {
      plugins: [{ type: "local", path: pluginPath }],
      maxTurns: 3
    }
  })) {
    if (message.type === "system" && message.subtype === "init") {
      console.log("Loaded plugins:", message.plugins);
      console.log("Available skills:", message.skills);
      console.log("Available commands:", message.slash_commands);
    }

    if (message.type === "assistant") {
      console.log("Assistant:", message.message.content);
    }
  }
}

runWithPlugin().catch(console.error);
```

**Python**

```python Python theme={null}
#!/usr/bin/env python3
"""Exemple montrant comment utiliser des plugins avec l'Agent SDK."""

import asyncio
from pathlib import Path

from claude_agent_sdk import (
    AssistantMessage,
    ClaudeAgentOptions,
    SystemMessage,
    TextBlock,
    query,
)


async def run_with_plugin():
    """Exemple utilisant un plugin personnalisé."""
    plugin_path = Path(__file__).parent / "plugins" / "my-plugin"

    print(f"Loading plugin from: {plugin_path}")

    options = ClaudeAgentOptions(
        plugins=[{"type": "local", "path": str(plugin_path)}],
        max_turns=3,
    )

    async for message in query(
        prompt="What custom commands do you have available?", options=options
    ):
        if isinstance(message, SystemMessage) and message.subtype == "init":
            print(f"Loaded plugins: {message.data.get('plugins')}")
            print(f"Available skills: {message.data.get('skills')}")
            print(f"Available commands: {message.data.get('slash_commands')}")

        if isinstance(message, AssistantMessage):
            for block in message.content:
                if isinstance(block, TextBlock):
                    print(f"Assistant: {block.text}")


if __name__ == "__main__":
    asyncio.run(run_with_plugin())
```

## Référence de structure d'un plugin

Un répertoire de plugin contient généralement un fichier manifeste `.claude-plugin/plugin.json`. Le manifeste est **optionnel** : s'il est absent, Claude Code découvre automatiquement les composants à partir de l'arborescence. Le répertoire peut contenir :

```text theme={null}
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # Manifeste du plugin (optionnel, composants auto-découverts sans lui)
├── skills/                   # Agent Skills (invoqués de façon autonome ou via /plugin-name:skill-name)
│   └── my-skill/
│       └── SKILL.md
├── commands/                 # Skills sous forme de fichiers .md à plat
│   └── custom-cmd.md
├── agents/                   # Agents personnalisés
│   └── specialist.md
├── hooks/                    # Gestionnaires d'événements
│   └── hooks.json
└── .mcp.json                # Définitions de serveurs MCP
```

> **Note** : le répertoire `commands/` contient des skills sous forme de fichiers Markdown à plat. Utilisez `skills/` pour les nouveaux plugins. Claude Code prend en charge les deux emplacements.

## Sources de plugins multiples

Combinez des plugins provenant d'emplacements différents :

```typescript theme={null}
import * as os from "node:os";
import * as path from "node:path";

plugins: [
  { type: "local", path: "./local-plugin" },
  {
    type: "local",
    path: path.join(os.homedir(), ".claude", "custom-plugins", "shared-plugin")
  }
];
```

> **Note** : le SDK ne développe pas les chemins avec tilde comme `~/plugins` (d'où `os.homedir()` ci-dessus). Si un chemin de plugin n'existe pas, le SDK ignore ce plugin et la session continue : vérifiez donc la liste `plugins` du message d'init pour confirmer que chaque plugin a bien été chargé.

## Dépannage

### Le plugin ne se charge pas

Si votre plugin n'apparaît pas dans la liste `plugins` du message d'init, consultez son champ [`plugin_errors`](https://code.claude.com/docs/en/agent-sdk/typescript#sdksystemmessage) pour connaître la raison, puis passez en revue ces vérifications :

1. **Vérifier le chemin** : assurez-vous qu'il pointe vers le répertoire racine du plugin, le parent de `skills/`, `agents/`, `hooks/`, `commands/` ou `.claude-plugin/`.
2. **Valider plugin.json** : si votre plugin inclut un manifeste, vérifiez que sa syntaxe JSON est valide.
3. **Vérifier les permissions de fichiers** : assurez-vous que le répertoire du plugin est lisible.
4. **Confirmer que le répertoire existe** : le SDK ignore un chemin inexistant, et le plugin n'apparaît alors pas dans la liste `plugins` du message d'init.

### Les skills n'apparaissent pas

Si les skills du plugin ne fonctionnent pas :

1. **Utiliser l'espace de noms** : invoquez les skills de plugin sous la forme `/plugin-name:skill-name`.
2. **Vérifier le message d'init** : vérifiez que le skill apparaît dans la liste `skills` avec le bon espace de noms.
3. **Valider les fichiers de skill** : assurez-vous que chaque skill possède un fichier `SKILL.md` dans son propre sous-répertoire sous `skills/`, par exemple `skills/my-skill/SKILL.md`.

## Voir aussi

- [Plugins](https://code.claude.com/docs/en/plugins/overview) — guide complet de développement de plugins.
- [Plugins reference](https://code.claude.com/docs/en/plugins/manifest-reference) — spécifications techniques.
- [Commands](https://code.claude.com/docs/en/agent-sdk/skills#dispatch-commands-by-name) — distribuer des commandes dans le SDK.
- [Subagents](https://code.claude.com/docs/en/agent-sdk/subagents) — travailler avec des agents spécialisés.
- [Skills](https://code.claude.com/docs/en/agent-sdk/skills) — utiliser les Agent Skills.
