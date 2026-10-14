# Modifier les prompts système (Agent SDK)

> Source : https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts · [← Index](../31-index-documentation-officielle.md)

*Choisir entre le preset `claude_code` et un prompt système personnalisé, et personnaliser le comportement avec CLAUDE.md, les output styles, `append` ou un prompt entièrement personnalisé.*

Les prompts système définissent le comportement, les capacités et le style de réponse de Claude. Partez du preset `claude_code` pour des outils de programmation de type CLI ou IDE où un humain observe et oriente le travail. Écrivez votre propre prompt pour des agents ayant une autre surface, une autre identité ou un autre modèle de permissions.

---

## Fonctionnement des prompts système

Un prompt système est l'ensemble initial d'instructions qui façonne le comportement de Claude tout au long d'une conversation. L'Agent SDK propose trois points de départ :

- **Défaut minimal** : quand vous ne définissez pas `systemPrompt` en TypeScript ou `system_prompt` en Python, le SDK utilise un prompt minimal qui couvre l'appel d'outils mais omet le reste du contenu du preset `claude_code`, y compris ses instructions de sécurité. Cela diffère de `claude -p`, qui utilise par défaut le prompt système de Claude Code. Si vous migrez depuis le CLI et voulez un comportement identique, définissez le preset `claude_code`.
- **Preset `claude_code`** : le prompt système utilisé par le CLI Claude Code, avec les instructions d'utilisation des outils et les instructions de sécurité. Définissez `systemPrompt: { type: "preset", preset: "claude_code" }` en TypeScript ou `system_prompt={"type": "preset", "preset": "claude_code"}` en Python, éventuellement avec `append` pour ajouter vos propres instructions à la fin.
- **Chaîne personnalisée** : un prompt que vous écrivez vous-même. Le SDK n'envoie que ce que vous fournissez.

### Choisir un point de départ

Le critère décisif est la ressemblance de votre agent avec Claude Code : un agent de programmation opérant dans un dépôt, avec un humain qui regarde la sortie en streaming et oriente le travail. Plus votre produit s'en éloigne, plus vous voudrez écrire votre propre prompt.

| Vous construisez | Utilisez | Ce que vous obtenez |
| :- | :- | :- |
| Un outil de programmation de type CLI ou IDE où un humain observe et oriente, et les comportements par défaut de Claude Code vous conviennent | Preset `claude_code` | Le prompt de Claude Code, y compris les consignes d'outils et les règles de sécurité |
| Le même type d'outil, plus des règles propres au produit comme des standards de code, un format de sortie ou un contexte métier | Preset `claude_code` avec `append` | Tout ce qui précède, avec vos instructions ajoutées après le preset. Rien n'est retiré : c'est la personnalisation la moins risquée |
| Un agent avec une autre surface, identité ou modèle de permissions, ou un agent non dédié à la programmation | Chaîne de prompt personnalisée | Uniquement ce que vous écrivez. Vous êtes responsable de remplacer les consignes d'outils et les instructions de sécurité dont votre agent a encore besoin |
| Une boucle d'appel d'outils minimale sans persona d'agent, où vous fournissez tout le comportement dans le prompt utilisateur | Pas d'option `systemPrompt` | Le défaut minimal : la prise en charge de l'appel d'outils et rien d'autre |

« Différent de Claude Code » signifie généralement l'un des cas suivants :

- **Surface différente** : la sortie n'est pas lue dans un terminal par la personne qui l'a déclenchée. Les interfaces de chat, les consommateurs de sortie structurée et l'automatisation hors programmation ont chacun besoin d'un prompt adapté à la façon dont leur sortie est rendue et relue. L'automatisation de programmation sans surveillance, comme un job CI qui corrige des erreurs de lint ou relit des diffs, correspond toujours au preset, car c'est précisément ce travail pour lequel le preset est écrit.
- **Identité différente** : l'agent ne doit pas se présenter comme Claude Code. Un bot de support, un assistant d'analyse de données ou tout agent spécialisé a besoin de son propre nom, de son périmètre et de sa persona.
- **Modèle de permissions différent** : l'agent fonctionne de façon autonome sans qu'un humain approuve chaque étape, ou opère sur un ensemble restreint de ressources. Le prompt de Claude Code suppose un humain dans la boucle avec accès à un jeu d'outils complet.
- **Tâches hors programmation** : l'essentiel du prompt de Claude Code est constitué de consignes de programmation. Pour des agents de recherche, de contenu ou d'opérations, ces consignes concurrencent les instructions dont vous avez réellement besoin.

Le [tableau comparatif](#comparer-les-quatre-approches) montre ce que préserve chaque méthode de personnalisation.

---

## Personnaliser le comportement de l'agent

`append` et une chaîne de prompt personnalisée modifient chacun directement le prompt système, et un output style modifie les instructions que Claude Code donne à Claude pour chaque réponse. CLAUDE.md suit un autre chemin : le SDK le lit et injecte son contenu dans la conversation comme contexte de projet, si bien qu'il façonne le comportement en complément du prompt système choisi, quel qu'il soit. Les [skills](https://code.claude.com/docs/en/agent-sdk/skills), les [hooks](https://code.claude.com/docs/en/agent-sdk/hooks) et les [permissions](https://code.claude.com/docs/en/agent-sdk/permissions) façonnent aussi le comportement en dehors du prompt système et sont traités sur leurs propres pages.

### Fichiers CLAUDE.md pour les instructions au niveau projet

Les fichiers CLAUDE.md donnent à Claude un contexte et des instructions de projet persistants. Le SDK injecte leur contenu dans la conversation et laisse le prompt système intact : ils fonctionnent donc avec n'importe quelle configuration de prompt système. Pour savoir quoi mettre dans CLAUDE.md, où le placer et comment écrire des instructions efficaces, voir [When to add to CLAUDE.md](https://code.claude.com/docs/en/memory#when-to-add-to-claude-md) et le reste de [How Claude remembers your project](https://code.claude.com/docs/en/memory). Cette section couvre ce qui est propre au SDK : le chargement de CLAUDE.md.

Le SDK lit CLAUDE.md quand la source de réglages correspondante est activée :

- `'project'` charge `CLAUDE.md` ou `.claude/CLAUDE.md` depuis le répertoire de travail ;
- `'user'` charge `~/.claude/CLAUDE.md`.

Les options par défaut de `query()` activent les deux sources, donc CLAUDE.md se charge automatiquement. Si vous définissez explicitement `settingSources` en TypeScript ou `setting_sources` en Python, incluez les sources nécessaires. Le chargement de CLAUDE.md est contrôlé par les sources de réglages, pas par le preset `claude_code`.

#### Charger CLAUDE.md avec le SDK

Pour charger CLAUDE.md, définissez `settingSources` de façon à inclure le niveau où se trouve votre CLAUDE.md. L'exemple ci-dessous charge un CLAUDE.md de niveau projet en plus du preset `claude_code`, pour que Claude dispose à la fois du prompt d'agent de programmation et des conventions de votre projet.

##### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

const messages = [];

for await (const message of query({
  prompt: "Add a new React component for user profiles",
  options: {
    systemPrompt: {
      type: "preset",
      preset: "claude_code" // Utiliser le prompt système de Claude Code
    },
    settingSources: ["project"] // Charge CLAUDE.md depuis le projet
  }
})) {
  messages.push(message);
}

// Claude a maintenant accès aux consignes de votre projet issues de CLAUDE.md
```

##### Python

```python
import asyncio

from claude_agent_sdk import query, ClaudeAgentOptions

messages = []


async def main():
    async for message in query(
        prompt="Add a new React component for user profiles",
        options=ClaudeAgentOptions(
            system_prompt={
                "type": "preset",
                "preset": "claude_code",  # Utiliser le prompt système de Claude Code
            },
            setting_sources=["project"],  # Charge CLAUDE.md depuis le projet
        ),
    ):
        messages.append(message)


asyncio.run(main())

# Claude a maintenant accès aux consignes de votre projet issues de CLAUDE.md
```

Quand vous exécutez l'un ou l'autre exemple, le SDK diffuse les messages au fil du travail de Claude : un message system init, des messages assistant, des messages user portant les résultats d'outils, et un message de résultat final avec l'issue de la session.

CLAUDE.md est persistant sur toutes les sessions d'un projet, partagé avec votre équipe via git et découvert automatiquement sans changement de code. Il n'est pas chargé si vous passez un tableau `settingSources` vide.

### Output styles pour des configurations persistantes

Les output styles sont des ensembles d'instructions enregistrés qui changent le rôle, le ton et le format de sortie de Claude. Ils sont stockés sous forme de fichiers markdown et peuvent être réutilisés d'une session et d'un projet à l'autre.

#### Créer un output style

Un output style est un fichier markdown avec un [frontmatter](https://code.claude.com/docs/en/output-styles#frontmatter) pour les métadonnées, suivi du contenu du prompt. Enregistrez-le :

- dans `~/.claude/output-styles/` pour un style de niveau utilisateur disponible dans tous les projets ;
- ou dans `.claude/output-styles/` de votre dépôt pour un style de niveau projet que vous pouvez committer et partager avec votre équipe.

Un output style personnalisé écarte les instructions d'ingénierie logicielle du preset `claude_code` et utilise les vôtres. Pour les conserver et superposer vos instructions, définissez `keep-coding-instructions: true` dans le frontmatter. Ces instructions n'existent que dans le prompt système complet de Claude Code : le réglage n'a donc aucun effet dans une session utilisant le prompt système raccourci ; réglez [`CLAUDE_CODE_SIMPLE_SYSTEM_PROMPT`](https://code.claude.com/docs/en/env-vars#variables) à `0` pour sélectionner le prompt complet sur n'importe quel modèle. Conservez-les quand votre agent fait encore du travail d'ingénierie logicielle. Écartez-les quand vous remplacez complètement le rôle.

L'exemple ci-dessous définit une persona de relecture de code qui conserve les instructions de programmation, car relire du code bénéficie toujours des consignes de sécurité et de qualité de code de Claude Code. Enregistrez-le sous `~/.claude/output-styles/code-reviewer.md` pour le rendre disponible dans tous les projets :

```markdown
---
name: Code Reviewer
description: Thorough code review assistant
keep-coding-instructions: true
---

You are an expert code reviewer.

For every code submission:
1. Check for bugs and security issues
2. Evaluate performance
3. Suggest improvements
4. Rate code quality (1-10)
```

#### Activer un output style

Une fois créés, les output styles s'activent via :

- **CLI** : exécutez `/output-style <style>`, par exemple `/output-style concise`, ou exécutez `/config` et sélectionnez-en un. La commande `/output-style` nécessite Claude Code (v2.1.269+).
- **Réglages** : définissez `outputStyle` dans `.claude/settings.local.json`.
- **SDK TypeScript** : définissez `outputStyle` dans l'objet `settings` en ligne passé à `query()`, ou faites pointer `settings` vers un fichier de réglages qui le définit. `outputStyle` n'est pas un champ de premier niveau de `Options` :

  ```typescript
  const options = { settings: { outputStyle: "Explanatory" } };
  ```

Dans le SDK Python, définissez `outputStyle` via l'option `settings`, qui accepte une chaîne JSON comme `'{"outputStyle": "Explanatory"}'` ou le chemin d'un fichier de réglages qui le définit.

**Note pour les utilisateurs du SDK :** les output styles sont chargés quand vous incluez `settingSources: ['user']` ou `settingSources: ['project']` (TypeScript) / `setting_sources=["user"]` ou `setting_sources=["project"]` (Python) dans vos options.

### Ajouter du texte au preset `claude_code` (append)

Vous pouvez utiliser le preset Claude Code avec une propriété `append` pour ajouter vos instructions personnalisées tout en préservant toutes les fonctionnalités intégrées.

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

const messages = [];

for await (const message of query({
  prompt: "Help me write a Python function to calculate fibonacci numbers",
  options: {
    systemPrompt: {
      type: "preset",
      preset: "claude_code",
      append: "Always include detailed docstrings and type hints in Python code."
    }
  }
})) {
  messages.push(message);
  if (message.type === "assistant") {
    console.log(message.message.content);
  }
}
```

#### Python

```python
import asyncio

from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage

messages = []


async def main():
    async for message in query(
        prompt="Help me write a Python function to calculate fibonacci numbers",
        options=ClaudeAgentOptions(
            system_prompt={
                "type": "preset",
                "preset": "claude_code",
                "append": "Always include detailed docstrings and type hints in Python code.",
            }
        ),
    ):
        messages.append(message)
        if isinstance(message, AssistantMessage):
            print(message.content)


asyncio.run(main())
```

#### Améliorer le cache de prompt entre utilisateurs et machines

Par défaut, deux sessions qui utilisent le même preset `claude_code` et le même texte `append` ne peuvent pas partager une entrée de cache de prompt si leurs emplacements d'auto memory diffèrent. Le preset intègre cet emplacement dans le prompt système avant votre texte `append`. L'emplacement est par défaut un chemin absolu sous `~/.claude/projects/`, nommé d'après le chemin du dépôt sur le disque : il diffère donc entre utilisateurs, machines et checkouts.

Le contenu de CLAUDE.md et les détails d'environnement comme le répertoire de travail, la plateforme, le shell et la version de l'OS n'affectent pas le cache du prompt système, car Claude Code les transmet dans la conversation, pas dans le prompt système.

Pour rendre le prompt système identique d'une session à l'autre, définissez `excludeDynamicSections: true` en TypeScript ou `"exclude_dynamic_sections": True` en Python. Le contexte propre à l'utilisateur passe dans le premier message utilisateur, ne laissant dans le prompt système que le preset statique et votre texte `append`, de sorte que des configurations identiques partagent une entrée de cache entre utilisateurs et machines.

> **Note** : `excludeDynamicSections` nécessite `@anthropic-ai/claude-agent-sdk` v0.2.98+ ou, pour Python, `claude-agent-sdk` v0.1.58+. Définissez-le uniquement sur la forme objet du preset. Le SDK l'ignore quand vous passez un prompt personnalisé au lieu du preset ; pour garder en cache les instructions d'un prompt personnalisé dans le SDK TypeScript, voir [Mettre en cache la partie statique d'un prompt personnalisé](#mettre-en-cache-la-partie-statique-dun-prompt-personnalisé).

L'exemple suivant associe un bloc `append` partagé à `excludeDynamicSections`, afin qu'une flotte d'agents puisse réutiliser le même prompt système en cache.

##### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Triage the open issues in this repo",
  options: {
    systemPrompt: {
      type: "preset",
      preset: "claude_code",
      append: "You operate Acme's internal triage workflow. Label issues by component and severity.",
      excludeDynamicSections: true
    }
  }
})) {
  // ...
}
```

##### Python

```python
import asyncio

from claude_agent_sdk import query, ClaudeAgentOptions


async def main():
    async for message in query(
        prompt="Triage the open issues in this repo",
        options=ClaudeAgentOptions(
            system_prompt={
                "type": "preset",
                "preset": "claude_code",
                "append": "You operate Acme's internal triage workflow. Label issues by component and severity.",
                "exclude_dynamic_sections": True,
            },
        ),
    ):
        ...


asyncio.run(main())
```

**Compromis :** le texte qui sort du prompt système atteint toujours Claude, mais dans un message utilisateur. Ce texte comprend au minimum l'emplacement du répertoire d'auto memory, et souvent toute la section d'auto memory. Des instructions placées dans un message utilisateur pèsent un peu moins que le même texte dans le prompt système : Claude risque donc de suivre ses consignes d'auto memory de façon moins constante. Activez cette option quand la réutilisation du cache entre sessions compte davantage que cela.

Pour le flag équivalent en mode CLI non interactif, voir [`--exclude-dynamic-system-prompt-sections`](https://code.claude.com/docs/en/cli-reference).

### Prompts système personnalisés

Vous pouvez fournir une chaîne personnalisée comme `systemPrompt` pour remplacer entièrement le prompt par défaut par vos propres instructions.

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

const customPrompt = `You are a Python coding specialist.
Follow these guidelines:
- Write clean, well-documented code
- Use type hints for all functions
- Include comprehensive docstrings
- Prefer functional programming patterns when appropriate
- Always explain your code choices`;

const messages = [];

for await (const message of query({
  prompt: "Create a data processing pipeline",
  options: {
    systemPrompt: customPrompt
  }
})) {
  messages.push(message);
  if (message.type === "assistant") {
    console.log(message.message.content);
  }
}
```

#### Python

```python
import asyncio

from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage

custom_prompt = """You are a Python coding specialist.
Follow these guidelines:
- Write clean, well-documented code
- Use type hints for all functions
- Include comprehensive docstrings
- Prefer functional programming patterns when appropriate
- Always explain your code choices"""

messages = []


async def main():
    async for message in query(
        prompt="Create a data processing pipeline",
        options=ClaudeAgentOptions(system_prompt=custom_prompt),
    ):
        messages.append(message)
        if isinstance(message, AssistantMessage):
            print(message.content)


asyncio.run(main())
```

En Python, chargez un grand prompt personnalisé depuis un fichier avec `system_prompt={"type": "file", "path": "..."}` au lieu de le passer comme chaîne. Le SDK Python transmet un prompt chaîne comme un unique argument de ligne de commande au sous-processus CLI : un prompt qui dépasse la limite de longueur d'arguments de l'OS échoue au lancement du processus, avant tout envoi de requête API. Sous Linux, l'erreur est `Argument list too long`. Voir [`SystemPromptFile`](https://code.claude.com/docs/en/agent-sdk/python#systempromptfile) pour les seuils par plateforme et le comportement sous Windows.

#### Mettre en cache la partie statique d'un prompt personnalisé

Dans le SDK TypeScript, vous pouvez passer un prompt personnalisé sous forme de tableau de chaînes au lieu d'une seule chaîne, avec le marqueur `SYSTEM_PROMPT_DYNAMIC_BOUNDARY` entre la partie statique et le reste. Utilisez cela quand votre prompt combine des instructions identiques à chaque requête et un contexte qui change à chaque requête, comme le client ou le ticket que traite l'agent. Si vous passez les deux parties en une seule chaîne, un changement de la partie variable modifie tout le prompt système, et les instructions statiques ratent aussi le cache. La forme tableau n'est pas disponible dans le SDK Python ; [`ClaudeAgentOptions`](https://code.claude.com/docs/en/agent-sdk/python#claudeagentoptions) liste les formes acceptées par `system_prompt`.

> **Note** : Claude Code ne découpe le prompt que lorsqu'il appelle directement l'API Claude ou s'exécute sur [Claude Platform on AWS](https://code.claude.com/docs/en/claude-platform-on-aws). Dans toute autre configuration, comme Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry ou une [passerelle LLM](https://code.claude.com/docs/en/llm-gateway-connect), il envoie le prompt entier en un seul bloc, comme si vous passiez une seule chaîne. Il en va de même chaque fois que vous définissez [`CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1`](https://code.claude.com/docs/en/llm-gateway-protocol#disable-pre-release-capabilities).

Pour découper le prompt, importez `SYSTEM_PROMPT_DYNAMIC_BOUNDARY` depuis `@anthropic-ai/claude-agent-sdk` et passez-le comme élément distinct du tableau, entre les deux parties. Le SDK envoie les chaînes situées avant le marqueur en un bloc de texte, et celles situées après en un second bloc, chacun avec son propre point d'arrêt de cache (cache breakpoint). Dans l'exemple ci-dessous, un agent de support charge ses instructions de tri depuis un fichier et reçoit à chaque requête les détails d'un ticket : les instructions restent en cache tandis que les détails du ticket changent.

```typescript
import { readFile } from "node:fs/promises";
import { query, SYSTEM_PROMPT_DYNAMIC_BOUNDARY } from "@anthropic-ai/claude-agent-sdk";

// Identique à chaque requête
const instructions = await readFile("triage-instructions.md", "utf8");
// Différent à chaque requête
const ticketContext = "Customer plan: Enterprise. Other open tickets from this customer: 3.";

for await (const message of query({
  prompt: "Triage ticket 4821",
  options: {
    systemPrompt: [instructions, SYSTEM_PROMPT_DYNAMIC_BOUNDARY, ticketContext]
  }
})) {
  // ...
}
```

[Track cache tokens](https://code.claude.com/docs/en/agent-sdk/cost-tracking#track-cache-tokens) décrit les champs `cache_creation_input_tokens` et `cache_read_input_tokens` de chaque message de résultat.

Le SDK assemble les blocs à partir du tableau de la façon suivante :

- Le SDK joint les chaînes de chaque côté du marqueur avec une ligne vide entre elles et supprime le marqueur lui-même, de sorte que le texte du marqueur n'atteint pas Claude.
- Si vous incluez le marqueur plusieurs fois, le premier sert de point de découpe et le SDK supprime les autres.
- Si vous omettez le marqueur, le SDK joint toutes les chaînes en un seul bloc, comme si vous passiez une seule chaîne.

Avec les [flags `--system-prompt` ou `--system-prompt-file`](https://code.claude.com/docs/en/cli-reference#system-prompt-flags) du CLI, le prompt est une seule chaîne : il n'y a donc pas de tableau pour porter le marqueur. Incluez à la place une ligne contenant uniquement `__SYSTEM_PROMPT_DYNAMIC_BOUNDARY__` entre la partie statique et la partie propre à chaque requête. Claude Code découpe le prompt à la première ligne de ce type, en produisant les deux mêmes blocs, et supprime cette ligne. Nécessite Claude Code (v2.1.275+).

Dans le SDK, préférez la forme tableau, qui porte la frontière sans ligne marqueur.

### Changer le prompt d'une session existante

Par défaut, si vous passez un `append` ou un prompt personnalisé différent en revenant à une session avec `resume` ou `continue`, Claude ne le voit pas au tour suivant. Claude Code enregistre le prompt système à la première requête d'une session et réutilise cet enregistrement jusqu'à la compaction de la session. Le nouveau texte prend effet après cette compaction, ou dans une nouvelle session.

#### Mettre à jour les instructions de Claude en cours de session

Si les instructions placées dans le prompt système doivent changer pendant qu'une session tourne, par exemple parce que votre utilisateur a basculé l'agent en mode lecture seule ou modifié sa configuration dans votre application, envoyez les nouvelles instructions dans la conversation au lieu de changer `systemPrompt` :

- **Dans votre prochain message** : incluez les nouvelles instructions dans le prochain message utilisateur que vous envoyez.
- **Depuis un hook** : renvoyez [`additionalContext`](https://code.claude.com/docs/en/hooks#add-context-for-claude) depuis un [callback de hook](https://code.claude.com/docs/en/agent-sdk/hooks#outputs) `UserPromptSubmit` ou `PostToolUse`, formulé comme un constat factuel tel que « The workspace is now read-only ». Le SDK insère le texte dans la conversation à l'endroit où le hook s'est déclenché, et le prompt enregistré reste inchangé.

#### Désactiver l'enregistrement pendant que vous itérez sur la formulation

Pendant que vous itérez sur la formulation du prompt et voulez que chaque modification atteigne une session que vous reprenez, définissez `snapshot` à false sur la forme objet du prompt système. Claude Code reconstruit alors le prompt à chaque requête. Le champ est disponible sur les formes preset et personnalisée de [`systemPrompt`](https://code.claude.com/docs/en/agent-sdk/typescript#options) en TypeScript et de [`system_prompt`](https://code.claude.com/docs/en/agent-sdk/python#systempromptpreset) en Python, et nécessite `@anthropic-ai/claude-agent-sdk` v0.3.257+ ou `claude-agent-sdk` v0.2.153+.

Gardez l'enregistrement activé en production. Avec l'enregistrement désactivé, un `append` ou un prompt personnalisé différent sur une session reprise atteint Claude au tour suivant, et cette requête ne peut pas réutiliser le [cache de prompt](https://code.claude.com/docs/en/prompt-caching#how-the-cache-is-organized) de la session. Là où l'API impose la [conservation de la réflexion (preserved thinking)](https://platform.claude.com/docs/en/build-with-claude/preserved-thinking), Claude perd aussi sa réflexion des tours précédents.

En dehors des [sessions cloud](https://code.claude.com/docs/en/cloud-environments), si vous démarrez Claude Code en [mode bare](https://code.claude.com/docs/en/headless#start-faster-with-bare-mode) en passant `--bare` via `extraArgs` ou en définissant `CLAUDE_CODE_SIMPLE=1`, l'enregistrement reste désactivé sauf si vous définissez `snapshot: true`.

L'enregistrement par défaut d'un `append` ou d'un prompt personnalisé nécessite Claude Code (v2.1.265+), que le SDK Agent TypeScript embarque à partir de v0.3.265 et le SDK Agent Python à partir de v0.2.153. Avant Claude Code v2.1.268, les sessions qui ne [récupèrent pas les feature flags](https://code.claude.com/docs/en/env-vars#features-that-need-feature-flag-fetching), y compris les sessions sur Amazon Bedrock, Google Cloud's Agent Platform et Microsoft Foundry, reconstruisaient le prompt à chaque requête et `snapshot` n'avait aucun effet.

---

## Contexte que Claude Code ajoute hors du prompt système

Les rappels système (system reminders) sont des messages que Claude Code ajoute à la conversation pendant une session pour donner du contexte à Claude, comme le contenu de vos fichiers CLAUDE.md ou une note indiquant qu'un fichier a changé sur le disque. Claude Code les envoie dans la conversation, pas dans le prompt système : ils atteignent donc Claude que vous utilisiez le preset `claude_code` ou que vous passiez votre propre chaîne comme `systemPrompt`.

Cette section couvre les [rappels les plus susceptibles de modifier le comportement de votre agent](#rappels-que-claude-code-ajoute-à-la-conversation), comment [désactiver ceux que votre agent remplace](#désactiver-le-contexte-que-votre-agent-remplace), et comment [voir ce que Claude a reçu](#voir-ce-que-claude-a-reçu) dans une requête donnée.

### Rappels que Claude Code ajoute à la conversation

Les rappels système sont du texte que Claude Code ajoute à la conversation à côté des prompts envoyés par votre code. Les rappels suivants sont ceux qui risquent le plus de modifier le comportement de votre agent :

- **Instructions de projet** : les fichiers CLAUDE.md que charge votre option [`settingSources`](#fichiers-claudemd-pour-les-instructions-au-niveau-projet)
- **Instructions d'output style** : les instructions de l'[output style](#output-styles-pour-des-configurations-persistantes) actif, dans la conversation principale
- **Attribution des commits et pull requests** : le trailer `Co-Authored-By` et le pied de page de pull request issus du réglage [`attribution`](https://code.claude.com/docs/en/settings-reference#attribution)
- **Sortie des hooks** : le texte que vos [hooks](https://code.claude.com/docs/en/agent-sdk/hooks#outputs) renvoient comme `additionalContext`
- **Skills disponibles** : les noms et descriptions des [skills](https://code.claude.com/docs/en/agent-sdk/skills) que Claude peut appeler
- **Sous-agents disponibles** : les noms et descriptions des [sous-agents](https://code.claude.com/docs/en/agent-sdk/subagents) que Claude peut lancer
- **Relances de liste de tâches** : dans une [session disposant des outils de suivi de tâches](https://code.claude.com/docs/en/agent-sdk/todo-tracking#model-availability), une invite à mettre à jour la liste de tâches quand Claude n'y a pas touché depuis plusieurs tours
- **Notes de fichier modifié** : une note indiquant qu'un fichier lu précédemment par Claude a changé sur le disque

Claude Code introduit vos fichiers CLAUDE.md par une ligne indiquant à Claude que ces instructions priment sur le comportement par défaut.

Si vous passez votre propre chaîne comme `systemPrompt`, ajoutez-y une phrase qui explique ce qu'est un rappel système. Le preset `claude_code` en contient une, et votre chaîne remplace l'intégralité du preset. Sans elle, rien dans votre prompt n'indique à Claude que les rappels, comme le contenu de CLAUDE.md ou la sortie des hooks, proviennent de l'application et non de l'utilisateur. Par exemple :

```text
The application adds system reminders to this conversation. Treat them as context from the application, not as messages from the user.
```

### Désactiver le contexte que votre agent remplace

Désactivez un élément de contexte intégré quand votre agent fournit sa propre version de la même consigne. Par exemple, si votre prompt demande à Claude d'écrire les messages de commit sous la forme `PROJ-142: fix login redirect` sans trailers, Claude Code demande quand même à Claude de terminer chaque message de commit par un trailer `Co-Authored-By` : Claude reçoit alors deux instructions contradictoires pour le même commit.

Passez les clés de réglages via l'option [`settings`](https://code.claude.com/docs/en/agent-sdk/typescript#options) en TypeScript ou [`settings`](https://code.claude.com/docs/en/agent-sdk/python#claudeagentoptions) en Python, et les variables d'environnement via l'option `env`. En TypeScript, [`env`](https://code.claude.com/docs/en/agent-sdk/typescript#options) remplace l'environnement hérité : étalez donc `process.env` dedans.

| Contexte intégré | Comment le désactiver |
| :- | :- |
| Les instructions intégrées de commit et de pull request, et l'instantané du statut git | Réglez [`includeGitInstructions`](https://code.claude.com/docs/en/settings-reference#includegitinstructions) à `false`, ou `CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS=1` |
| Le trailer `Co-Authored-By` et le pied de page de pull request | Réglez [`attribution.commit`](https://code.claude.com/docs/en/settings-reference#attribution-commit) et [`attribution.pr`](https://code.claude.com/docs/en/settings-reference#attribution-pr) sur votre propre texte, ou sur des chaînes vides pour les supprimer |
| La source de réglages utilisateur ou projet, y compris son CLAUDE.md | Omettez `'user'` ou `'project'` de [`settingSources`](https://code.claude.com/docs/en/agent-sdk/claude-code-features#control-filesystem-settings-with-settingsources) |
| Tous les fichiers CLAUDE.md | Définissez `CLAUDE_CODE_DISABLE_CLAUDE_MDS=1` |
| Relances de liste de tâches, notes de fichier modifié et liste des skills | Définissez `CLAUDE_CODE_DISABLE_ATTACHMENTS=1` |

Les instructions intégrées de commit et de pull request de Claude Code ne sont pas un rappel. Elles font partie de la description de l'outil Bash : elles atteignent donc aussi Claude quand vous passez un `systemPrompt` personnalisé.

Si vous définissez `CLAUDE_CODE_DISABLE_ATTACHMENTS`, Claude Code envoie aussi les mentions de fichiers `@` en texte brut au lieu de les développer en contenu de fichier. La liste des sous-agents disponibles et les notifications de tâches en arrière-plan continuent d'arriver.

L'exemple suivant concerne un agent qui porte ses propres règles de commit dans `append`. Il règle les deux clés `attribution` sur des chaînes vides pour supprimer le trailer et le pied de page, et désactive `includeGitInstructions` pour que les instructions de workflow de commit de Claude Code ne concurrencent pas les vôtres.

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Commit the staged changes for ticket PROJ-142",
  options: {
    systemPrompt: {
      type: "preset",
      preset: "claude_code",
      append: "Write commit messages as: <ticket id>: <summary>. Add no trailers."
    },
    settings: {
      includeGitInstructions: false,
      attribution: { commit: "", pr: "" }
    },
    allowedTools: ["Bash(git *)"]
  }
})) {
  if (message.type === "result") console.log(message.subtype);
}
```

#### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions


async def main():
    async for message in query(
        prompt="Commit the staged changes for ticket PROJ-142",
        options=ClaudeAgentOptions(
            system_prompt={
                "type": "preset",
                "preset": "claude_code",
                "append": "Write commit messages as: <ticket id>: <summary>. Add no trailers.",
            },
            settings='{"includeGitInstructions": false, "attribution": {"commit": "", "pr": ""}}',
            allowed_tools=["Bash(git *)"],
        ),
    ):
        print(message)


asyncio.run(main())
```

Pour confirmer le changement, exécutez l'exemple dans un dépôt contenant des modifications indexées (staged) et vérifiez le nouveau commit avec `git log -1`. Le message se termine sans trailer `Co-Authored-By`.

### Voir ce que Claude a reçu

Le flux de messages du SDK n'inclut pas les rappels système : lire les messages que reçoit votre code ne vous montrera donc pas ce que Claude a vu. Pour les voir, journalisez les requêtes envoyées par Claude Code :

- **Journalisation brute des requêtes** : définissez [`OTEL_LOG_RAW_API_BODIES`](https://code.claude.com/docs/en/monitoring-usage#api-request-body-event) à `file:<dir>`. Claude Code écrit le corps de chaque requête dans ce répertoire.
- **Une passerelle que vous contrôlez** : faites pointer [`ANTHROPIC_BASE_URL`](https://code.claude.com/docs/en/llm-gateway) vers un proxy qui journalise les corps de requête.

Dans une requête journalisée, regardez le tableau `messages`. Un rappel apparaît dans un message utilisateur, entouré de balises `<system-reminder>`, ou, sur certains modèles, comme un message séparé avec le rôle `system`.

---

## Comparer les quatre approches

Les quatre méthodes de personnalisation diffèrent par leur emplacement, leur mode de partage et ce qu'elles préservent du preset `claude_code`.

| Caractéristique | CLAUDE.md | Output styles | `systemPrompt` avec append | `systemPrompt` personnalisé |
| - | - | - | - | - |
| **Persistance** | Fichier par projet | Enregistrés en fichiers | Session uniquement | Session uniquement |
| **Réutilisabilité** | Par projet | Entre projets | Duplication de code | Duplication de code |
| **Gestion** | Sur le système de fichiers | CLI + fichiers | Dans le code | Dans le code |
| **Outils par défaut** | Préservés | Préservés | Préservés | Perdus (sauf s'ils sont inclus) |
| **Sécurité intégrée** | Maintenue | Maintenue | Maintenue | Doit être ajoutée |
| **Niveau de personnalisation** | Ajouts uniquement | Ajouts ; peut omettre les instructions de programmation | Ajouts uniquement | Contrôle complet |
| **Contrôle de version** | Avec le projet | Oui | Avec le code | Avec le code |
| **Portée** | Spécifique au projet | Utilisateur ou projet | Session de code | Session de code |

« Avec append » signifie utiliser `systemPrompt: { type: "preset", preset: "claude_code", append: "..." }` en TypeScript ou `system_prompt={"type": "preset", "preset": "claude_code", "append": "..."}` en Python. CLAUDE.md ne modifie pas le prompt système lui-même : le SDK injecte son contenu dans la conversation comme contexte de projet.

---

## Combiner les approches

Les approches se combinent. Un output style persistant ou CLAUDE.md définit le comportement durable, et `append` superpose des instructions propres à la session sans toucher à la configuration enregistrée.

### Combiner un output style avec des ajouts propres à la session

L'exemple ci-dessous suppose qu'un output style « Code Reviewer » est déjà actif. Le bloc `append` superpose des axes prioritaires propres à la session à la persona, si bien qu'une seule session de relecture peut privilégier OAuth et le stockage des jetons sans modifier l'output style enregistré.

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

// En supposant que l'output style "Code Reviewer" est actif (via /config ou les réglages)
// Ajouter des axes prioritaires propres à la session
const messages = [];

for await (const message of query({
  prompt: "Review this authentication module",
  options: {
    systemPrompt: {
      type: "preset",
      preset: "claude_code",
      append: `
        For this review, prioritize:
        - OAuth 2.0 compliance
        - Token storage security
        - Session management
      `
    }
  }
})) {
  messages.push(message);
}
```

#### Python

```python
import asyncio

from claude_agent_sdk import query, ClaudeAgentOptions

# En supposant que l'output style "Code Reviewer" est actif (via /config ou les réglages)
# Ajouter des axes prioritaires propres à la session
messages = []


async def main():
    async for message in query(
        prompt="Review this authentication module",
        options=ClaudeAgentOptions(
            system_prompt={
                "type": "preset",
                "preset": "claude_code",
                "append": """
                For this review, prioritize:
                - OAuth 2.0 compliance
                - Token storage security
                - Session management
                """,
            }
        ),
    ):
        messages.append(message)


asyncio.run(main())
```

---

## Voir aussi

- [Output styles](https://code.claude.com/docs/en/output-styles) : créer, gérer et partager des output styles pour le CLI, y compris le format de fichier et les emplacements de stockage
- [How Claude remembers your project](https://code.claude.com/docs/en/memory) : quoi mettre dans CLAUDE.md, où le placer et comment écrire des instructions de projet efficaces
- [Référence du SDK TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript) : le type `Options` complet, y compris `systemPrompt`, `settingSources` et `settings`
- [Référence du SDK Python](https://code.claude.com/docs/en/agent-sdk/python) : le type `ClaudeAgentOptions` complet, y compris `system_prompt` et `setting_sources`
- [Settings](https://code.claude.com/docs/en/settings) : la référence de `settings.json`, y compris l'emplacement de stockage des output styles et des autres configurations
