# Étendre les agents avec des skills

> Source : https://code.claude.com/docs/en/agent-sdk/skills · [← Index](../31-index-documentation-officielle.md)

*Contrôler quels skills Claude peut invoquer dans les sessions du Claude Agent SDK, dispatcher des commandes par leur nom, et rédiger des skills que vos sessions découvrent.*

Les Agent Skills étendent Claude avec des capacités spécialisées que Claude invoque lorsqu'elles sont pertinentes. Les skills sont empaquetés sous forme de fichiers `SKILL.md` contenant des instructions, des descriptions et des ressources complémentaires optionnelles. Cette page couvre aussi les [commandes dans les sessions de l'Agent SDK](#commandes-dans-les-sessions-de-lagent-sdk).

Pour en savoir plus sur les skills (avantages, architecture, recommandations de rédaction), voir l'[aperçu des Agent Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview).

## Fonctionnement des skills avec l'Agent SDK

Avec le Claude Agent SDK, les skills sont :

* **Définis comme des artefacts du système de fichiers** : vous créez chaque skill sous forme de fichier `SKILL.md` dans son propre répertoire, par exemple `.claude/skills/<name>/SKILL.md`
* **Chargés depuis le système de fichiers** : le SDK charge les skills depuis les emplacements gouvernés par `settingSources` (TypeScript) ou `setting_sources` (Python)
* **Découverts automatiquement** : une fois les réglages du système de fichiers chargés, le SDK découvre les métadonnées des skills au démarrage dans les répertoires utilisateur et projet, et charge le contenu complet lorsque Claude invoque le skill
* **Invoqués par le modèle** : Claude choisit de façon autonome quand les utiliser selon le contexte
* **Invoqués par l'utilisateur** : vous dispatchez un skill directement en envoyant `/<name>` dans un prompt. Voir [Commandes dans les sessions de l'Agent SDK](#commandes-dans-les-sessions-de-lagent-sdk)
* **Délimités via l'option `skills`** : les skills découverts sont activés par défaut. Passez une liste de noms de skills, `"all"` ou `[]` pour contrôler quels skills Claude peut invoquer

Contrairement aux sous-agents, que vous pouvez définir dans l'[option `agents`](https://code.claude.com/docs/en/agent-sdk/subagents#programmatic-definition-recommended), les skills se créent sous forme de fichiers sur disque. Le SDK ne fournit pas d'API programmatique pour les enregistrer.

> **Note** : les skills sont découverts via les sources de réglages du système de fichiers. Avec les options `query()` par défaut, le SDK charge les sources utilisateur et projet, donc les skills de `~/.claude/skills/`, de `<cwd>/.claude/skills/` et de `.claude/skills/` dans tout répertoire parent de `<cwd>` jusqu'à la racine du dépôt sont disponibles. La source projet couvre aussi `<dir>/.claude/skills/` dans chaque répertoire passé via `additionalDirectories` (TypeScript) ou `add_dirs` (Python), car le SDK transmet ces répertoires à Claude Code en tant que [`--add-dir`](https://code.claude.com/docs/en/skills#skills-from-additional-directories). Si vous définissez `settingSources` explicitement, incluez `'project'` pour conserver les skills du projet et des répertoires ajoutés, et `'user'` pour conserver vos skills personnels, ou utilisez l'[option `plugins`](https://code.claude.com/docs/en/agent-sdk/plugins) pour charger des skills depuis un chemin précis.

## Utiliser les skills avec l'Agent SDK

Définissez l'option `skills` sur `query()` pour contrôler quels skills Claude peut invoquer dans la session. Si elle est omise, les skills découverts sont activés et l'outil Skill est disponible, comme dans le CLI. Valeurs possibles :

* `"all"` : Claude peut invoquer tous les skills découverts ;
* une liste de noms de skills : seuls ceux-là sont autorisés ;
* `[]` : Claude ne peut en invoquer aucun.

Par exemple, pour n'autoriser que deux skills nommés :

#### Python

```python Python theme={null}
options = ClaudeAgentOptions(skills=["pdf", "docx"])
```

#### TypeScript

```typescript TypeScript theme={null}
const options = { skills: ["pdf", "docx"] };
```

### Configurer les skills dans une session

Lorsque vous définissez `skills`, le SDK ajoute automatiquement l'outil Skill à `allowedTools`. Si vous passez aussi une liste `tools` explicite, incluez-y `"Skill"` pour que Claude puisse invoquer des skills.

Une fois configuré, Claude découvre automatiquement les skills sur le système de fichiers et les invoque lorsqu'ils sont pertinents pour la demande de l'utilisateur.

L'exemple suivant active tous les skills découverts dans une session et pré-approuve les outils dont les skills ont couramment besoin. L'exemple définit `cwd` sur le répertoire de travail courant du processus : lancez-le donc depuis un projet qui possède un répertoire `.claude/skills/` dans le répertoire courant ou dans un parent jusqu'à la racine du dépôt :

#### Python

```python Python theme={null}
import asyncio
import os

from claude_agent_sdk import query, ClaudeAgentOptions


async def main():
    options = ClaudeAgentOptions(
        cwd=os.getcwd(),  # .claude/skills/ ici ou dans un répertoire parent
        setting_sources=["user", "project"],  # Charge les skills depuis le système de fichiers
        skills="all",  # Laisse Claude invoquer tous les skills découverts
        allowed_tools=["Read", "Write", "Bash"],
    )

    async for message in query(
        prompt="Help me process this PDF document", options=options
    ):
        print(message)


asyncio.run(main())
```

#### TypeScript

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Help me process this PDF document",
  options: {
    cwd: process.cwd(), // .claude/skills/ ici ou dans un répertoire parent
    settingSources: ["user", "project"], // Charge les skills depuis le système de fichiers
    skills: "all", // Laisse Claude invoquer tous les skills découverts
    allowedTools: ["Read", "Write", "Bash"]
  }
})) {
  console.log(message);
}
```

### Confirmer que les skills sont chargés

Vers le début du flux, le SDK produit un message système de sous-type `init`. Vérifiez son tableau `skills` pour confirmer que vos skills sont chargés avant que Claude ne commence à travailler. Le tableau inclut les skills invocables par l'utilisateur que vous avez définis avec un champ de frontmatter `description` ou `when_to_use`, ainsi que les [skills intégrés (bundled) fournis avec Claude Code](https://code.claude.com/docs/en/skills#bundled-skills).

Le tableau ne liste que les skills invocables par l'utilisateur. Un skill avec [`user-invocable: false`](https://code.claude.com/docs/en/skills#control-who-invokes-a-skill) dans son frontmatter se charge et reste disponible pour Claude, mais n'apparaît pas dans le tableau. Le tableau liste les mêmes skills, qu'ils figurent ou non dans votre liste `skills`.

### N'autoriser que certains skills

Pour ne laisser Claude invoquer que certains skills, passez leurs noms dans la liste `skills`. Les noms correspondent au champ `name` de `SKILL.md` ou au nom du répertoire du skill. Utilisez `plugin:skill` pour les skills fournis par un plugin.

La liste n'accepte que des noms de skills exacts. Si une entrée ne peut pas fonctionner comme nom exact, `query()` rejette la liste avant le démarrage de la session. Voir [Erreur de nom de skill invalide](#erreur-de-nom-de-skill-invalide) pour les règles de nommage et l'erreur levée par chaque SDK.

Le modèle ne voit pas les skills non listés et l'outil Skill les refuse, mais leurs fichiers restent sur disque et restent accessibles via Read et Bash. Restreindre la liste ne restreint pas le [dispatch par nom](#dispatcher-des-commandes-par-leur-nom).

Pour laisser Claude invoquer tous les skills découverts, passez `skills: "all"` plutôt qu'un joker (wildcard).

## Commandes dans les sessions de l'Agent SDK

Cette section constitue la documentation des commandes du SDK. Une commande est tout ce que vous exécutez en envoyant `/<name>` dans un prompt. Les entrées de cette surface de commandes diffèrent par ce qui les sous-tend :

* **Commandes intégrées (built-in)** : exécutent une logique codée dans le processus Claude Code que le SDK lance, par exemple `/compact`
* **Skills intégrés (bundled)** : artefacts de prompt fournis avec Claude Code, par exemple `/code-review`
* **Vos skills** : artefacts de prompt que vous rédigez, chacun étant un répertoire contenant un fichier `SKILL.md`. Le nom d'un skill invocable par l'utilisateur rejoint automatiquement la surface de commandes, si bien que dispatcher votre propre `/security-check` fonctionne comme l'exécution d'une commande intégrée
* **Fichiers de commandes personnalisées** : une forme d'artefact plus ancienne au comportement identique, des fichiers Markdown plats dans `.claude/commands/` dont les noms de fichiers deviennent des noms de commandes. Les skills en sont le successeur recommandé

Par défaut, vous comme Claude pouvez invoquer n'importe quel skill. Vous pouvez restreindre l'une ou l'autre voie via le [frontmatter](https://code.claude.com/docs/en/skills#control-who-invokes-a-skill) du skill. Pour les définitions de commande et de skill, voir les entrées [Command](https://code.claude.com/docs/en/glossary#command) et [Skill](https://code.claude.com/docs/en/glossary#skill) du glossaire. Voir [Commands in Claude Code](https://code.claude.com/docs/en/commands) pour toutes les commandes intégrées et [Extend Claude with skills](https://code.claude.com/docs/en/skills) pour le guide complet des deux formes d'artefacts.

### Découvrir les commandes disponibles

Vous pouvez dispatcher via le SDK les commandes qui fonctionnent sans terminal interactif. Le message `system/init` liste celles disponibles dans votre session dans son champ `slash_commands`. Les commandes qui nécessitent un terminal interactif, comme `/theme` et `/terminal-setup`, n'apparaissent pas dans la liste. Accédez au champ au démarrage de la session :

#### TypeScript

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Hello Claude",
  options: { maxTurns: 1 }
})) {
  if (message.type === "system" && message.subtype === "init") {
    console.log("Available commands:", message.slash_commands);
  }
}
```

#### Python

```python Python theme={null}
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, SystemMessage


async def main():
    async for message in query(prompt="Hello Claude", options=ClaudeAgentOptions(max_turns=1)):
        if isinstance(message, SystemMessage) and message.subtype == "init":
            print("Available commands:", message.data["slash_commands"])


asyncio.run(main())
```

La liste affichée mélange commandes intégrées, skills intégrés, vos skills invocables par l'utilisateur et fichiers `.claude/commands/` :

```text theme={null}
Available commands: ["clear", "compact", "context", "usage", "code-review", "verify", "security-check", ...]
```

Un skill avec [`user-invocable: false`](https://code.claude.com/docs/en/skills#control-who-invokes-a-skill) dans son frontmatter n'apparaît ni dans cette liste ni dans le tableau `skills` de [Confirmer que les skills sont chargés](#confirmer-que-les-skills-sont-chargés). Les sessions qui configurent des [serveurs MCP](https://code.claude.com/docs/en/agent-sdk/mcp) peuvent aussi exposer des [prompts MCP sous forme de commandes](https://code.claude.com/docs/en/mcp#use-mcp-prompts-as-commands).

### Dispatcher des commandes par leur nom

Envoyez une commande en l'incluant dans votre chaîne de prompt, comme du texte ordinaire. Le dispatch ne dépend pas de l'option `skills` : envoyer `/<name>` exécute un skill invocable par l'utilisateur même si votre liste `skills` l'omet. Les commandes qui agissent sur l'historique de conversation, comme `/compact`, ont besoin de messages antérieurs sur lesquels travailler.

Un `/<name>` qui ne correspond ni à une commande de la session ni à une commande intégrée de Claude Code ne fait pas échouer la requête. Claude Code envoie le prompt à Claude comme un message ordinaire, avec une note indiquant que la commande ne s'est pas exécutée ; la requête consomme donc un tour de modèle et renvoie la réponse de Claude. Avant la v2.1.274, un `/<name>` sans correspondance renvoyait `Unknown command: /<name>` comme résultat, sans tour de modèle.

Un `/<name>` qui correspond à une commande intégrée de Claude Code non disponible dans la session, comme `/theme`, renvoie `/theme isn't available in this environment.` comme résultat, sans tour de modèle.

> **Note** : une commande peut atteindre la limite `maxTurns` / `max_turns` comme n'importe quel autre prompt, terminant la requête par un résultat d'erreur au lieu de `success`. Pour le contrat des résultats d'erreur, voir [Handle the result](https://code.claude.com/docs/en/agent-sdk/agent-loop#handle-the-result). Si votre commande risque d'atteindre la limite, enveloppez la boucle dans un `try`/`catch` en TypeScript ou un `try`/`except` en Python, comme dans [Single Message Input](https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode#single-message-input), ou fixez `maxTurns` assez haut pour que le travail puisse se terminer.

### Compacter l'historique avec `/compact`

La commande `/compact` réduit la taille de votre historique de conversation en résumant les messages anciens tout en préservant le contexte important. La compaction nécessite une conversation existante avec suffisamment de messages antérieurs à résumer. Cet exemple mène d'abord une conversation, puis la compacte et lit le message système `compact_boundary` qui rapporte le résultat :

#### TypeScript

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

// La compaction nécessite un historique existant : on mène d'abord une conversation
try {
  for await (const message of query({
    prompt: "Explain what this project does",
    options: { maxTurns: 2 }
  })) {
    if (message.type === "result" && message.subtype === "success") {
      console.log(message.result);
    }
  }
} catch (error) {
  // Un query() ponctuel lève une exception après avoir produit un résultat d'erreur,
  // donc la requête de suivi ci-dessous s'exécute quand même.
  console.error(`Session ended with an error: ${error}`);
}

// Compacte la même conversation
for await (const message of query({
  prompt: "/compact",
  options: { continue: true, maxTurns: 1 }
})) {
  if (message.type === "system" && message.subtype === "compact_boundary") {
    console.log("Compaction completed");
    console.log("Pre-compaction tokens:", message.compact_metadata.pre_tokens);
    console.log("Trigger:", message.compact_metadata.trigger);
    // Exemple de sortie :
    // Compaction completed
    // Pre-compaction tokens: 1842
    // Trigger: manual
  }
}
```

#### Python

```python Python theme={null}
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage, SystemMessage


async def main():
    # La compaction nécessite un historique existant : on mène d'abord une conversation
    try:
        async for message in query(
            prompt="Explain what this project does",
            options=ClaudeAgentOptions(max_turns=2),
        ):
            if isinstance(message, ResultMessage) and message.subtype == "success":
                print(message.result)
    except Exception as error:
        # Un query() ponctuel lève une exception après avoir produit un résultat d'erreur,
        # donc la requête de suivi ci-dessous s'exécute quand même.
        print(f"Session ended with an error: {error}")

    # Compacte la même conversation
    async for message in query(
        prompt="/compact",
        options=ClaudeAgentOptions(continue_conversation=True, max_turns=1),
    ):
        if isinstance(message, SystemMessage) and message.subtype == "compact_boundary":
            print("Compaction completed")
            print("Pre-compaction tokens:", message.data["compact_metadata"]["pre_tokens"])
            print("Trigger:", message.data["compact_metadata"]["trigger"])
            # Exemple de sortie :
            # Compaction completed
            # Pre-compaction tokens: 1842
            # Trigger: manual


asyncio.run(main())
```

> **Note** : un message `compact_boundary` n'arrive que si la compaction s'est exécutée. Lorsqu'une session continuée contient des messages mais rien que `/compact` puisse résumer, l'exécution se termine quand même par un résultat `success` plutôt que par une erreur, et aucun message `compact_boundary` n'arrive. Le texte du résultat donne alors la raison, par exemple `Not enough messages to compact.` lorsque la session contient un prompt mais encore aucune réponse de Claude. Un appel `query()` ponctuel neuf démarre avec un contexte vide : utilisez donc ce schéma dans une session ayant des tours antérieurs, par exemple en [mode d'entrée streaming](https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode) ou lors de la reprise d'une session.

### Réinitialiser le contexte avec `/clear`

La commande `/clear` réinitialise la conversation à un contexte vide, de sorte que les prompts suivants démarrent sans historique de conversation antérieur. La conversation précédente reste sur disque. Vous pouvez y revenir en passant son ID de session à l'[option `resume`](https://code.claude.com/docs/en/agent-sdk/sessions#resume-by-id).

`/clear` est utile en [mode d'entrée streaming](https://code.claude.com/docs/en/agent-sdk/streaming-vs-single-mode), où vous envoyez plusieurs prompts sur une seule connexion. Pour les appels `query()` ponctuels, chaque appel démarre déjà avec un contexte vide, donc envoyer `/clear` n'a pas d'effet pratique. Lancez plutôt un nouveau `query()`.

## Créer des skills

Créez chaque skill sous forme de répertoire contenant un fichier `SKILL.md` avec un frontmatter YAML et un contenu Markdown. Le champ `description` détermine quand Claude invoque votre skill.

**Exemple de structure de répertoire** :

```text theme={null}
.claude/skills/security-check/
└── SKILL.md
```

### Choisir un niveau de découverte

Enregistrez les skills à l'un des deux [niveaux de découverte](https://code.claude.com/docs/en/skills#where-skills-live) les plus courants :

* **Skills de projet** : `.claude/skills/`, disponibles uniquement dans le projet courant
* **Skills personnels** : `~/.claude/skills/`, disponibles dans tous vos projets

Si vous avez des fichiers de commandes personnalisées existants dans `.claude/commands/`, ils continuent de fonctionner. Un fichier de commande `.claude/commands/deploy.md` crée `/deploy` et fonctionne de la même façon qu'un skill `.claude/skills/deploy/SKILL.md`. Si un fichier de commande et un skill portent le même nom, voir [Resolve skills that share a name](https://code.claude.com/docs/en/skills#resolve-skills-that-share-a-name) pour savoir lequel s'exécute. Le SDK charge les fichiers de `.claude/commands/` et `~/.claude/commands/` depuis les deux mêmes portées que les skills. Voir [Extend Claude with skills](https://code.claude.com/docs/en/skills) pour le guide complet des deux formes d'artefacts.

### Créer et dispatcher votre premier skill

Pour voir le flux complet, créez `.claude/skills/security-check/SKILL.md` :

```markdown theme={null}
---
name: security-check
description: Run a security vulnerability scan
---

Analyze the codebase for security vulnerabilities including:
- SQL injection risks
- XSS vulnerabilities
- Exposed credentials
- Insecure configurations
```

(Le skill demande d'analyser le code à la recherche de vulnérabilités : risques d'injection SQL, failles XSS, identifiants exposés, configurations non sécurisées.)

Une fois le fichier créé, le skill est disponible via le SDK. Claude l'invoque lorsqu'une demande correspond à sa description, et vous pouvez le dispatcher directement :

#### TypeScript

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "/security-check",
  options: { maxTurns: 10 }
})) {
  if (message.type === "result" && message.subtype === "success") {
    console.log(message.result);
  }
}
```

#### Python

```python Python theme={null}
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


async def main():
    async for message in query(
        prompt="/security-check", options=ClaudeAgentOptions(max_turns=10)
    ):
        if isinstance(message, ResultMessage) and message.subtype == "success":
            print(message.result)


asyncio.run(main())
```

Une exécution réussie se termine par un résultat `success` dont le texte contient les constats de l'analyse. Sur une petite application Express contenant des problèmes volontairement introduits, le texte du résultat commence ainsi :

```text theme={null}
**Security scan of `app.js` — 4 findings (most severe first):**

1. **SQL Injection** (line 8) — `req.query.name` is concatenated directly into the SQL string. Trivially exploitable (`' OR '1'='1`, `'; DROP TABLE users;--`). **Fix:** use parameterized queries, e.g. `db.query("SELECT * FROM users WHERE name = ?", [req.query.name], cb)`.
...
```

Le nom du skill apparaît aussi dans le tableau `slash_commands` du message init.

> **Note** : Claude Code inclut les skills intégrés `code-review` et `verify`. Si vous nommez un fichier `.claude/commands/` d'après l'un d'eux, par exemple `.claude/commands/code-review.md`, la commande du fichier masque le skill intégré et `slash_commands` ne liste le nom qu'une fois.

## Pré-approuver des outils pour les skills

> **Note** : dans les sessions SDK, vous pouvez pré-approuver des outils pour un skill de projet ou personnel avec le frontmatter [`allowed-tools`](https://code.claude.com/docs/en/skills#pre-approve-tools-for-a-skill) du skill, ou avec l'option `allowedTools` (`allowed_tools` en Python) de la configuration de votre requête. Si votre organisation définit [`allowManagedPermissionRulesOnly`](https://code.claude.com/docs/en/settings-reference#allowmanagedpermissionrulesonly) dans les réglages gérés, Claude Code ignore les deux. Les skills [synchronisés depuis claude.ai](https://code.claude.com/docs/en/skills#how-claude-code-handles-the-frontmatter-of-a-synced-skill) suivent leurs propres règles de frontmatter.

Les skills s'exécutent avec les outils de la session. L'exemple ci-dessous pré-approuve `Read`, `Grep` et `Glob` avec `allowedTools` (`allowed_tools` en Python), pour que Claude puisse inspecter les fichiers en exécutant le [skill security-check](#créer-et-dispatcher-votre-premier-skill) sans s'arrêter pour demander une approbation :

#### Python

```python Python theme={null}
import asyncio

from claude_agent_sdk import query, ClaudeAgentOptions

options = ClaudeAgentOptions(
    setting_sources=["user", "project"],  # Charge les skills depuis le système de fichiers
    skills="all",
    allowed_tools=["Read", "Grep", "Glob"],
)


async def main():
    async for message in query(prompt="Check this project for security issues", options=options):
        print(message)


asyncio.run(main())
```

#### TypeScript

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Check this project for security issues",
  options: {
    settingSources: ["user", "project"], // Charge les skills depuis le système de fichiers
    skills: "all",
    allowedTools: ["Read", "Grep", "Glob"]
  }
})) {
  console.log(message);
}
```

Dans le flux, l'invocation du skill apparaît comme une utilisation de l'outil Skill, suivie d'appels Read sur les fichiers du projet. L'exécution se termine par un résultat `success` dont le texte contient les constats.

La liste pré-approuve les outils nommés sans restreindre les autres. Pour le flux de permissions complet, y compris les modes de permission et le callback `canUseTool`, voir [Permissions](https://code.claude.com/docs/en/agent-sdk/permissions).

## Dépannage

### Skills introuvables

**Vérifier la configuration de settingSources** : le SDK découvre les skills via les sources de réglages `user` et `project`. Si vous définissez `settingSources`/`setting_sources` explicitement en omettant ces sources, le SDK ne charge pas les skills :

#### Python

```python Python theme={null}
# Skills non chargés : setting_sources exclut user et project
options = ClaudeAgentOptions(setting_sources=[], skills="all")

# Skills chargés : sources user et project incluses
options = ClaudeAgentOptions(
    setting_sources=["user", "project"],
    skills="all",
)
```

#### TypeScript

```typescript TypeScript theme={null}
// Skills non chargés : settingSources exclut user et project
const optionsWithoutSkills = {
  settingSources: [],
  skills: "all"
};

// Skills chargés : sources user et project incluses
const optionsWithSkills = {
  settingSources: ["user", "project"],
  skills: "all"
};
```

Pour savoir quels répertoires de skills chaque source charge, voir le [tableau des sources du système de fichiers](https://code.claude.com/docs/en/agent-sdk/claude-code-features#control-filesystem-settings-with-settingsources). Pour plus de détails sur `settingSources`/`setting_sources`, voir la [référence du SDK TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript#settingsource) ou la [référence du SDK Python](https://code.claude.com/docs/en/agent-sdk/python#settingsource).

**Vérifier le répertoire de travail** : le SDK charge les skills depuis `.claude/skills/` dans l'option `cwd` et dans chaque répertoire parent jusqu'à la racine du dépôt. Assurez-vous que `cwd` pointe sur le répertoire contenant `.claude/skills/` ou en dessous, au sein du même dépôt :

#### Python

```python Python theme={null}
# Assurez-vous que cwd pointe vers le répertoire contenant .claude/skills/
options = ClaudeAgentOptions(
    cwd="/path/to/project",  # .claude/skills/ ici ou dans un répertoire parent
    setting_sources=["user", "project"],  # Charge les skills depuis ces sources
    skills="all",
)
```

#### TypeScript

```typescript TypeScript theme={null}
// Assurez-vous que cwd pointe vers le répertoire contenant .claude/skills/
const options = {
  cwd: "/path/to/project", // .claude/skills/ ici ou dans un répertoire parent
  settingSources: ["user", "project"], // Charge les skills depuis ces sources
  skills: "all"
};
```

Voir [Utiliser les skills avec l'Agent SDK](#utiliser-les-skills-avec-lagent-sdk) pour le schéma complet.

**Vérifier l'emplacement sur le système de fichiers** :

```bash theme={null}
# Vérifier les skills de projet
ls .claude/skills/*/SKILL.md

# Vérifier les skills personnels
ls ~/.claude/skills/*/SKILL.md
```

### Skill non utilisé

**Vérifier l'option `skills`** : si vous avez passé une liste `skills`, confirmez que le nom du skill y figure. Lorsque Claude tente d'invoquer un skill non listé, l'outil Skill renvoie `Skill <name> is not in this session's skills allowlist`. Ajoutez le nom à votre liste, ou dispatchez le skill directement en envoyant `/<name>` dans un prompt, ce qui fonctionne sans le lister.

**Vérifier la description** : assurez-vous qu'elle est précise et contient des mots-clés pertinents. Voir les [bonnes pratiques des Agent Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices#writing-effective-descriptions) pour rédiger des descriptions efficaces.

### Erreur de nom de skill invalide

Lorsqu'un nom de votre liste `skills` ne peut pas fonctionner comme nom de skill exact, `query()` rejette la liste avant de démarrer le processus Claude Code. Les noms qui déclenchent ce rejet incluent :

* Un nom vide
* Un nom contenant des parenthèses, des virgules ou des caractères de contrôle
* Un nom entouré d'espaces
* Une forme joker, comme un `*` seul ou un suffixe `:*`

Chaque SDK signale ce rejet différemment :

#### TypeScript

Le SDK TypeScript lève une `Error` indiquant la règle enfreinte par l'entrée. Par exemple, `skills: ["docs:*"]` lève :

```text theme={null}
Invalid skill name "docs:*": wildcard-suffix names are not allowed; list each skill by its exact name.
```

Un nom vide renvoie `Skill names must be non-empty strings.`

Avant le TypeScript Agent SDK 0.3.221, le SDK n'effectuait pas cette vérification.

#### Python

Le SDK Python lève `ValueError` indiquant la règle enfreinte par l'entrée. Par exemple, `skills=["docs:*"]` lève :

```text theme={null}
ValueError: Invalid skill name 'docs:*': wildcard-suffix names are not allowed; list each skill by its exact name.
```

Un nom vide renvoie `Skill names must be non-empty strings`.

Avant le Python Agent SDK 0.2.129, le SDK n'effectuait pas cette vérification.

### Dépannage complémentaire

Pour le dépannage général des skills, comme les erreurs de syntaxe YAML et le débogage, voir la [section de dépannage des skills de Claude Code](https://code.claude.com/docs/en/skills#troubleshooting).

## Étapes suivantes

Le [guide des skills de Claude Code](https://code.claude.com/docs/en/skills) couvre la rédaction en profondeur. Ses recommandations s'appliquent aux sessions SDK. Commencez par ces sections :

* [Frontmatter reference](https://code.claude.com/docs/en/skills#frontmatter-reference) : tous les champs pris en charge
* [Pass arguments to skills](https://code.claude.com/docs/en/skills#pass-arguments-to-skills) : `$ARGUMENTS`, `$0`, `$1` et l'empilement de skills. Le [tableau complet des substitutions](https://code.claude.com/docs/en/skills#available-string-substitutions) ajoute les arguments nommés et les variables `${CLAUDE_*}`
* [Inject dynamic context](https://code.claude.com/docs/en/skills#inject-dynamic-context) : lignes `` !`command` `` exécutées avant que Claude ne voie le contenu du skill
* [Choose where skills load](https://code.claude.com/docs/en/skills#where-skills-live) : tous les emplacements de skills, l'espace de noms des plugins, et quel skill s'exécute lorsque deux portent le même nom

## Ressources associées

* [Commands in Claude Code](https://code.claude.com/docs/en/commands) : toute la surface de commandes, y compris chaque commande intégrée
* [Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) : aperçu conceptuel, avantages et architecture
* [Agent Skills best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) : recommandations de rédaction pour des skills efficaces
* [Agent Skills cookbook](https://platform.claude.com/cookbook/skills-notebooks-01-skills-introduction) : exemples de skills et modèles
* [Subagents in the SDK](https://code.claude.com/docs/en/agent-sdk/subagents) : agents similaires basés sur le système de fichiers, avec options programmatiques
* [SDK overview](https://code.claude.com/docs/en/agent-sdk/overview) : concepts généraux du SDK
* [TypeScript SDK reference](https://code.claude.com/docs/en/agent-sdk/typescript) : documentation complète de l'API
* [Python SDK reference](https://code.claude.com/docs/en/agent-sdk/python) : documentation complète de l'API
