# Démarrage rapide (Agent SDK)

> Source : https://code.claude.com/docs/en/agent-sdk/quickstart · [← Index](../31-index-documentation-officielle.md)

> Prendre en main l'Agent SDK Python ou TypeScript pour construire des agents IA qui travaillent de manière autonome.

Utilisez l'Agent SDK pour construire un agent IA qui lit votre code, trouve les bugs et les corrige, le tout sans intervention manuelle.

**Ce que vous allez faire :**

1. Mettre en place un projet avec l'Agent SDK.
2. Créer un fichier contenant du code bogué.
3. Exécuter un agent qui trouve et corrige automatiquement les bugs.

## Prérequis

- **Node.js 18+** ou **Python 3.10+**.
- Un **compte Anthropic**. Si vous n'en avez pas, [inscrivez-vous ici](https://platform.claude.com/).

## Mise en place

### Étape 1 — Créer un dossier de projet

Créez un nouveau répertoire pour ce démarrage rapide :

```bash theme={null}
mkdir my-agent
cd my-agent
```

Pour vos propres projets, vous pouvez exécuter le SDK depuis n'importe quel dossier ; par défaut, il aura accès aux fichiers de ce répertoire et de ses sous-répertoires.

### Étape 2 — Installer le SDK

Installez le paquet Agent SDK pour votre langage :

#### Onglet « TypeScript (nouveau projet) »

```bash theme={null}
npm init -y
npm pkg set type=module
npm install @anthropic-ai/claude-agent-sdk
npm install --save-dev tsx
```

Définir `"type": "module"` dans `package.json` permet à votre script d'agent d'utiliser `await` au niveau supérieur, et [tsx](https://tsx.hirok.io) exécute directement les fichiers TypeScript. npm affiche `added N packages` quand l'installation réussit.

#### Onglet « TypeScript (projet existant) »

```bash theme={null}
npm install @anthropic-ai/claude-agent-sdk
npm install --save-dev tsx
```

[tsx](https://tsx.hirok.io) exécute directement les fichiers TypeScript. Si votre projet utilise CommonJS, nommez votre script d'agent `agent.mts` au lieu de `agent.ts`. L'extension `.mts` fait traiter le fichier comme un module ES par tsx : le `await` au niveau supérieur fonctionne alors sans convertir tout votre projet en modules ES. Utilisez `agent.mts` à la place de `agent.ts` dans les étapes de création et d'exécution plus loin.

#### Onglet « Python (uv) »

[Installez uv](https://docs.astral.sh/uv/), un gestionnaire de paquets Python rapide qui gère automatiquement les environnements virtuels. Puis initialisez un projet et ajoutez le SDK :

```bash theme={null}
uv init
uv add claude-agent-sdk
```

#### Onglet « Python (pip) »

Créez et activez un environnement virtuel, puis installez le paquet.

Sous macOS ou Linux :

```bash theme={null}
python3 -m venv .venv
source .venv/bin/activate
pip install claude-agent-sdk
```

Sous Windows :

```powershell theme={null}
py -m venv .venv
.venv\Scripts\Activate.ps1
pip install claude-agent-sdk
```

Si PowerShell bloque `Activate.ps1` avec une erreur de stratégie d'exécution, lancez d'abord `Set-ExecutionPolicy -Scope Process RemoteSigned`.

> **Note** : les SDK TypeScript et Python embarquent tous deux un binaire natif de Claude Code ; la plupart des installations n'ont donc pas besoin d'installer Claude Code séparément. Certaines installations n'ont pas de binaire embarqué :
>
> - Si pip installe la distribution source du SDK Python au lieu d'une wheel propre à la plateforme (par exemple sous Windows ARM64), aucun binaire n'est embarqué. [Installez Claude Code nativement](https://code.claude.com/docs/en/setup#install-claude-code) ; le SDK Python le trouve dans votre `PATH`.
> - Le SDK TypeScript installe son binaire via les dépendances optionnelles npm : une installation qui les ignore (par exemple `npm ci --omit=optional`) n'obtient aucun binaire, même sur une plateforme prise en charge. Réinstallez sans ignorer les dépendances optionnelles, ou [installez Claude Code nativement](https://code.claude.com/docs/en/setup#install-claude-code) et définissez `pathToClaudeCodeExecutable` sur son chemin.

### Étape 3 — Définir votre clé API

Obtenez une clé API depuis la [Claude Console](https://platform.claude.com/), puis définissez-la comme variable d'environnement dans le shell où vous exécuterez votre agent :

#### Onglet « macOS / Linux »

```bash theme={null}
export ANTHROPIC_API_KEY=your-api-key
```

#### Onglet « Windows (PowerShell) »

```powershell theme={null}
$env:ANTHROPIC_API_KEY = "your-api-key"
```

Le SDK lit la clé dans l'environnement du processus qui exécute votre agent ; il **ne charge pas automatiquement les fichiers `.env`**. Si vous gardez la clé dans un fichier `.env`, chargez-le vous-même (par exemple avec le paquet `dotenv`) avant d'appeler le SDK.

Le SDK prend aussi en charge l'authentification via des fournisseurs d'API tiers :

- **Amazon Bedrock** : définir la variable d'environnement `CLAUDE_CODE_USE_BEDROCK=1` et configurer les identifiants AWS.
- **Claude Platform on AWS** : définir `CLAUDE_CODE_USE_ANTHROPIC_AWS=1` et `ANTHROPIC_AWS_WORKSPACE_ID`, puis configurer les identifiants AWS.
- **Google Cloud's Agent Platform** : définir la variable d'environnement `CLAUDE_CODE_USE_VERTEX=1` et configurer les identifiants Google Cloud.
- **Microsoft Foundry** : définir la variable d'environnement `CLAUDE_CODE_USE_FOUNDRY=1` et configurer les identifiants Azure.

Voir les guides de configuration pour [Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock), [Claude Platform on AWS](https://code.claude.com/docs/en/claude-platform-on-aws), [Google Cloud's Agent Platform](https://code.claude.com/docs/en/google-vertex-ai) ou [Microsoft Foundry](https://code.claude.com/docs/en/microsoft-foundry) pour les détails.

> **Note** : sauf approbation préalable, Anthropic n'autorise pas les développeurs tiers à proposer la connexion claude.ai ou les limites de débit claude.ai pour leurs produits, y compris les agents construits avec le Claude Agent SDK. Utilisez plutôt les méthodes d'authentification par clé API décrites dans ce document.

## Créer un fichier bogué

Ce démarrage rapide vous guide dans la construction d'un agent capable de trouver et de corriger des bugs dans du code. Il vous faut d'abord un fichier contenant des bugs intentionnels que l'agent corrigera. Créez `utils.py` dans le répertoire `my-agent` et collez-y le code suivant :

```python theme={null}
def calculate_average(numbers):
    total = 0
    for num in numbers:
        total += num
    return total / len(numbers)


def get_user_name(user):
    return user["name"].upper()
```

Ce code contient deux bugs :

1. `calculate_average([])` plante avec une division par zéro.
2. `get_user_name(None)` plante avec une `TypeError`.

## Construire un agent qui trouve et corrige les bugs

Créez `agent.py` si vous utilisez le SDK Python, ou `agent.ts` pour TypeScript. Utilisez plutôt `agent.mts` si votre projet existant utilise CommonJS :

**Python**

```python Python theme={null}
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage, ResultMessage


async def main():
    # Boucle agentique : diffuse les messages pendant que Claude travaille
    async for message in query(
        prompt="Review utils.py for bugs that would cause crashes. Fix any issues you find.",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Edit", "Glob"],  # Auto-approuver ces outils
            permission_mode="acceptEdits",  # Auto-approuver les modifications de fichiers
        ),
    ):
        # Afficher une sortie lisible par un humain
        if isinstance(message, AssistantMessage):
            for block in message.content:
                if hasattr(block, "text"):
                    print(block.text)  # Raisonnement de Claude
                elif hasattr(block, "name"):
                    print(f"Tool: {block.name}")  # Outil appelé
        elif isinstance(message, ResultMessage):
            print(f"Done: {message.subtype}")  # Résultat final


asyncio.run(main())
```

**TypeScript**

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

// Boucle agentique : diffuse les messages pendant que Claude travaille
for await (const message of query({
  prompt: "Review utils.py for bugs that would cause crashes. Fix any issues you find.",
  options: {
    allowedTools: ["Read", "Edit", "Glob"], // Auto-approuver ces outils
    permissionMode: "acceptEdits" // Auto-approuver les modifications de fichiers
  }
})) {
  // Afficher une sortie lisible par un humain
  if (message.type === "assistant" && message.message?.content) {
    for (const block of message.message.content) {
      if ("text" in block) {
        console.log(block.text); // Raisonnement de Claude
      } else if ("name" in block) {
        console.log(`Tool: ${block.name}`); // Outil appelé
      }
    }
  } else if (message.type === "result") {
    console.log(`Done: ${message.subtype}`); // Résultat final
  }
}
```

Ce code comporte trois parties principales :

1. **`query`** : le point d'entrée principal qui crée la boucle agentique. Il renvoie un itérateur asynchrone : on utilise donc `async for` pour recevoir les messages en flux pendant que Claude travaille. Voir l'API complète dans la référence du SDK [Python](https://code.claude.com/docs/en/agent-sdk/python#query) ou [TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript#query).
2. **`prompt`** : ce que vous voulez que Claude fasse. Claude détermine quels outils utiliser selon la tâche.
3. **`options`** : la configuration de l'agent. Cet exemple utilise `allowedTools` pour pré-approuver `Read`, `Edit` et `Glob`, et `permissionMode: "acceptEdits"` pour auto-approuver les modifications de fichiers. D'autres options existent, dont `systemPrompt`, `mcpServers`, etc. Voir toutes les options pour [Python](https://code.claude.com/docs/en/agent-sdk/python#claudeagentoptions) ou [TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript#options).

La boucle `async for` continue de tourner pendant que Claude réfléchit, appelle des outils, observe les résultats et décide de la suite. Chaque itération produit un message : le raisonnement de Claude, un appel d'outil, un résultat d'outil ou l'issue finale. Le SDK gère l'orchestration, l'exécution des outils, la gestion du contexte et les nouvelles tentatives ; vous n'avez qu'à consommer le flux. La boucle se termine quand Claude a fini la tâche ou rencontre une erreur.

Le traitement des messages dans la boucle ne garde que la sortie lisible par un humain. Sans ce filtrage, vous verriez les objets message bruts, y compris l'initialisation système et l'état interne : utile pour le débogage, mais bruyant sinon.

> **Note** : cet exemple utilise le streaming pour afficher la progression en temps réel. Si vous n'avez pas besoin de sortie en direct (par exemple pour des tâches en arrière-plan ou des pipelines CI), vous pouvez collecter tous les messages d'un coup. Voir [Streaming vs. single-turn mode](https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode) pour les détails.

### Exécuter votre agent

Votre agent est prêt. Lancez-le avec la commande suivante :

#### Onglet « TypeScript »

```bash theme={null}
npx tsx agent.ts
```

Si vous avez nommé votre script `agent.mts`, lancez plutôt `npx tsx agent.mts`.

#### Onglet « Python (uv) »

```bash theme={null}
uv run agent.py
```

#### Onglet « Python (pip) »

Avec votre environnement virtuel toujours activé :

```bash theme={null}
python agent.py
```

Pendant son travail, l'agent affiche son raisonnement et chaque outil qu'il appelle, et se termine par `Done: success`. Après l'exécution, consultez `utils.py` : vous y verrez du code défensif gérant les listes vides et les utilisateurs nuls. Votre agent a, de manière autonome :

1. **Lu** `utils.py` pour comprendre le code ;
2. **Analysé** la logique et identifié les cas limites qui provoqueraient un plantage ;
3. **Modifié** le fichier pour ajouter une gestion d'erreurs appropriée.

C'est ce qui distingue l'Agent SDK : Claude exécute directement les outils au lieu de vous demander de les implémenter.

> **Note** : si vous voyez une erreur d'authentification comme `Not logged in` ou `Invalid API key`, vérifiez que vous avez défini la variable d'environnement `ANTHROPIC_API_KEY` dans le shell où vous exécutez votre agent. Le SDK ne charge pas automatiquement les fichiers `.env`.
>
> Pour les causes et correctifs de ces erreurs d'authentification et d'autres, voir [Authentication errors](https://code.claude.com/docs/en/errors#authentication-errors) dans la référence des erreurs.

### Essayer d'autres prompts

Maintenant que votre agent est configuré, essayez des prompts différents :

- `"Add docstrings to all functions in utils.py"` (ajouter des docstrings à toutes les fonctions)
- `"Add type hints to all functions in utils.py"` (ajouter des annotations de type)
- `"Create a README.md documenting the functions in utils.py"` (créer un README documentant les fonctions)

### Personnaliser votre agent

Vous pouvez modifier le comportement de votre agent en changeant les options. Quelques exemples :

**Ajouter la recherche web :**

```python Python theme={null}
options = ClaudeAgentOptions(
    allowed_tools=["Read", "Edit", "Glob", "WebSearch"], permission_mode="acceptEdits"
)
```

```typescript TypeScript hidelines={1,-1} theme={null}
const _ = {
  options: {
    allowedTools: ["Read", "Edit", "Glob", "WebSearch"],
    permissionMode: "acceptEdits"
  }
};
```

**Donner à Claude un prompt système personnalisé :**

```python Python theme={null}
options = ClaudeAgentOptions(
    allowed_tools=["Read", "Edit", "Glob"],
    permission_mode="acceptEdits",
    system_prompt="You are a senior Python developer. Always follow PEP 8 style guidelines.",
)
```

```typescript TypeScript hidelines={1,-1} theme={null}
const _ = {
  options: {
    allowedTools: ["Read", "Edit", "Glob"],
    permissionMode: "acceptEdits",
    systemPrompt: "You are a senior Python developer. Always follow PEP 8 style guidelines."
  }
};
```

**Exécuter des commandes dans le terminal :**

```python Python theme={null}
options = ClaudeAgentOptions(
    allowed_tools=["Read", "Edit", "Glob", "Bash"], permission_mode="acceptEdits"
)
```

```typescript TypeScript hidelines={1,-1} theme={null}
const _ = {
  options: {
    allowedTools: ["Read", "Edit", "Glob", "Bash"],
    permissionMode: "acceptEdits"
  }
};
```

Avec `Bash` activé, essayez : `"Write unit tests for utils.py, run them, and fix any failures"` (écrire des tests unitaires, les exécuter et corriger les échecs).

Chacun de ces extraits définit des champs sur le même objet d'options. Pour plus d'informations, voir [Configure your agent](https://code.claude.com/docs/en/agent-sdk/configuration).

## Concepts clés

Les **outils** déterminent ce que votre agent peut faire :

| Outils | Ce que l'agent peut faire |
| - | - |
| `Read`, `Glob`, `Grep` | Analyse en lecture seule |
| `Read`, `Edit`, `Glob` | Analyser et modifier du code |
| `Read`, `Edit`, `Bash`, `Glob`, `Grep` | Automatisation complète |

Les **modes de permission** déterminent le niveau de supervision humaine souhaité. Le SDK évalue le mode actif avec vos règles allow et deny dans un ordre fixe, décrit dans [How permissions are evaluated](https://code.claude.com/docs/en/agent-sdk/permissions#how-permissions-are-evaluated). Pour la liste complète des modes, leur comportement et quand utiliser chacun, voir [Permission mode dans How the agent loop works](https://code.claude.com/docs/en/agent-sdk/agent-loop#permission-mode).

## Étapes suivantes

Maintenant que vous avez créé votre premier agent, apprenez à étendre ses capacités et à l'adapter à votre cas d'usage :

- **[Configure your agent](https://code.claude.com/docs/en/agent-sdk/configuration)** : composer l'objet d'options et trouver la page qui couvre chaque réglage.
- **[Permissions](https://code.claude.com/docs/en/agent-sdk/permissions)** : contrôler ce que votre agent peut faire et quand il a besoin d'une approbation.
- **[Hooks](https://code.claude.com/docs/en/agent-sdk/hooks)** : exécuter du code personnalisé avant ou après les appels d'outils.
- **[Sessions](https://code.claude.com/docs/en/agent-sdk/sessions)** : construire des agents multi-tours qui conservent le contexte.
- **[MCP servers](https://code.claude.com/docs/en/agent-sdk/mcp)** : se connecter à des bases de données, navigateurs, API et autres systèmes externes.
- **[Hosting](https://code.claude.com/docs/en/agent-sdk/hosting)** : déployer des agents sur Docker, dans le cloud et en CI/CD.
- **[Example agents](https://github.com/anthropics/claude-agent-sdk-demos)** : exemples complets (assistant e-mail, agent de recherche, etc.).
- **[Troubleshooting](https://code.claude.com/docs/en/agent-sdk/troubleshooting)** : corriger les erreurs quand la CLI ne démarre pas ou se termine, ou quand un résultat arrive sans sortie structurée.
