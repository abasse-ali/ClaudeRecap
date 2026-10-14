# Référence des channels

> Source : https://code.claude.com/docs/en/channels-reference · [← Index](../31-index-documentation-officielle.md)

*Construire un serveur MCP qui pousse des webhooks, des alertes et des messages de chat dans une session Claude Code. Référence du contrat de channel : déclaration de capacité, événements de notification, outils de réponse, filtrage des expéditeurs et relais des permissions.*

> **Note** : les channels sont en [aperçu de recherche (research preview)](https://code.claude.com/docs/en/channels#research-preview). Les organisations Team et Enterprise doivent [les activer explicitement](https://code.claude.com/docs/en/channels#enterprise-controls).

Un channel (canal) est un serveur MCP qui pousse des événements dans une session Claude Code pour que Claude puisse réagir à ce qui se passe hors du terminal.

Vous pouvez construire un channel unidirectionnel ou bidirectionnel. Les channels unidirectionnels transfèrent des alertes, des webhooks ou des événements de supervision sur lesquels Claude doit agir. Les channels bidirectionnels, comme les passerelles de chat, [exposent aussi un outil de réponse](#exposer-un-outil-de-réponse) pour que Claude puisse renvoyer des messages. Un channel disposant d'un chemin d'expéditeur de confiance peut aussi choisir de [relayer les demandes de permission](#relayer-les-demandes-de-permission) pour que vous puissiez approuver ou refuser l'utilisation d'outils à distance.

Cette page couvre :

* [Vue d'ensemble](#vue-densemble) : comment fonctionnent les channels ;
* [Ce dont vous avez besoin](#ce-dont-vous-avez-besoin) : prérequis et étapes générales ;
* [Exemple : construire un récepteur de webhooks](#exemple--construire-un-récepteur-de-webhooks) : un parcours minimal unidirectionnel ;
* [Options du serveur](#options-du-serveur) : les champs du constructeur ;
* [Format des notifications](#format-des-notifications) : la charge utile de l'événement et le comportement de livraison ;
* [Exposer un outil de réponse](#exposer-un-outil-de-réponse) : permettre à Claude de renvoyer des messages ;
* [Filtrer les messages entrants](#filtrer-les-messages-entrants) : vérifications d'expéditeur pour prévenir l'injection de prompt ;
* [Relayer les demandes de permission](#relayer-les-demandes-de-permission) : transférer les demandes d'approbation d'outils vers des channels distants.

Pour utiliser un channel existant plutôt qu'en construire un, voir [Channels](https://code.claude.com/docs/en/channels). Telegram, Discord, iMessage et fakechat sont inclus dans l'aperçu de recherche.

## Vue d'ensemble

Un channel est un serveur [MCP](https://modelcontextprotocol.io) qui s'exécute sur la même machine que Claude Code. Claude Code le lance comme sous-processus et communique avec lui via stdio. Votre serveur de channel fait le pont entre les systèmes externes et la session Claude Code :

* **Plateformes de chat** (Telegram, Discord) : votre plugin s'exécute localement et interroge (polling) l'API de la plateforme pour récupérer les nouveaux messages. Quand quelqu'un envoie un message privé à votre bot, le plugin le reçoit et le transfère à Claude. Aucune URL à exposer.
* **Webhooks** (CI, supervision) : votre serveur écoute sur un port HTTP local. Les systèmes externes font un POST sur ce port, et votre serveur pousse la charge utile vers Claude.

*(Schéma d'architecture : les systèmes externes se connectent à votre serveur de channel local, qui communique avec Claude Code via stdio.)*

## Ce dont vous avez besoin

Le seul prérequis strict est le paquet [`@modelcontextprotocol/sdk`](https://www.npmjs.com/package/@modelcontextprotocol/sdk) et un runtime compatible Node.js. [Bun](https://bun.sh), [Node](https://nodejs.org) et [Deno](https://deno.com) fonctionnent tous. Les plugins préconstruits de l'aperçu de recherche utilisent Bun, mais votre channel n'y est pas obligé.

Votre serveur doit :

1. déclarer la capacité `claude/channel` pour que Claude Code enregistre un écouteur de notifications ;
2. émettre des événements `notifications/claude/channel` quand quelque chose se produit ;
3. se connecter via le [transport stdio](https://modelcontextprotocol.io/docs/concepts/transports#standard-io).

Les sections [Options du serveur](#options-du-serveur) et [Format des notifications](#format-des-notifications) détaillent chacun de ces points. Voir [Exemple : construire un récepteur de webhooks](#exemple--construire-un-récepteur-de-webhooks) pour un parcours complet.

Pendant l'aperçu de recherche, les channels personnalisés ne figurent pas sur la [liste d'autorisation approuvée](https://code.claude.com/docs/en/channels#supported-channels). Utilisez `--dangerously-load-development-channels` pour tester localement. Voir [Tester pendant l'aperçu de recherche](#tester-pendant-laperçu-de-recherche) pour les détails.

## Exemple : construire un récepteur de webhooks

Ce parcours construit un serveur en un seul fichier qui écoute les requêtes HTTP et les transfère dans votre session Claude Code. À la fin, tout ce qui peut envoyer un HTTP POST — un pipeline de CI, une alerte de supervision ou une commande `curl` — peut pousser des événements vers Claude.

Cet exemple utilise [Bun](https://bun.sh) comme runtime pour son serveur HTTP intégré et son support de TypeScript. Vous pouvez utiliser [Node](https://nodejs.org) ou [Deno](https://deno.com) à la place ; le seul prérequis est le [SDK MCP](https://www.npmjs.com/package/@modelcontextprotocol/sdk).

### Étape 1 : créer le projet

Les exemples de [relais des permissions](#relayer-les-demandes-de-permission) importent `zod` directement ; il s'installe donc avec le SDK MCP. Créez un nouveau répertoire et installez les deux :

```bash theme={null}
mkdir webhook-channel && cd webhook-channel
bun add @modelcontextprotocol/sdk zod
```

### Étape 2 : écrire le serveur de channel

Créez un fichier nommé `webhook.ts`. C'est l'intégralité de votre serveur de channel : il se connecte à Claude Code via stdio et écoute les HTTP POST sur le port 8788. Quand une requête arrive, il pousse le corps vers Claude sous forme d'événement de channel.

```ts title="webhook.ts" theme={null}
#!/usr/bin/env bun
import { Server } from '@modelcontextprotocol/sdk/server/index.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'

// Crée le serveur MCP et le déclare comme channel
const mcp = new Server(
  { name: 'webhook', version: '0.0.1' },
  {
    // c'est cette clé qui en fait un channel — Claude Code enregistre un écouteur pour elle
    capabilities: { experimental: { 'claude/channel': {} } },
    // Claude Code transmet ceci à Claude comme contexte à la connexion du serveur, pour qu'il sache traiter ces événements
    instructions: 'Events from the webhook channel arrive as <channel source="webhook" ...>. They are one-way: read them and act, no reply expected.',
  },
)

// Connexion à Claude Code via stdio (Claude Code lance ce processus)
await mcp.connect(new StdioServerTransport())

// Démarre un serveur HTTP qui transfère chaque POST à Claude
Bun.serve({
  port: 8788,  // n'importe quel port libre convient
  // localhost uniquement : rien en dehors de cette machine ne peut faire de POST
  hostname: '127.0.0.1',
  async fetch(req) {
    const body = await req.text()
    await mcp.notification({
      method: 'notifications/claude/channel',
      params: {
        content: body,  // devient le corps de la balise <channel>
        // chaque clé devient un attribut de la balise, ex. <channel path="/" method="POST">
        meta: { path: new URL(req.url).pathname, method: req.method },
      },
    })
    return new Response('ok')
  },
})
```

*(La chaîne `instructions` dit en substance : « Les événements du channel webhook arrivent sous la forme <channel source="webhook" ...>. Ils sont unidirectionnels : lisez-les et agissez, aucune réponse attendue. »)*

Le fichier configure le serveur, se connecte via stdio et démarre un écouteur HTTP, dans cet ordre :

* **Configuration du serveur** : crée le serveur MCP avec `claude/channel` dans ses capacités, ce qui indique à Claude Code qu'il s'agit d'un channel. Claude Code transmet la chaîne [`instructions`](#options-du-serveur) à Claude comme contexte lors de la connexion du serveur : indiquez à Claude quels événements attendre, s'il doit répondre et, le cas échéant, comment router les réponses.
* **Connexion stdio** : se connecte à Claude Code via stdin/stdout. C'est la norme pour tout [serveur MCP](https://modelcontextprotocol.io/docs/concepts/transports#standard-io).
* **Écouteur HTTP** : démarre un serveur web local sur le port 8788. Chaque corps de POST est transféré à Claude comme événement de channel via `mcp.notification()`. Le `content` devient le corps de l'événement, et chaque entrée de `meta` devient un attribut de la balise `<channel>`. L'écouteur a besoin d'accéder à l'instance `mcp` ; il s'exécute donc dans le même processus. Pour un projet plus important, vous pourriez le découper en modules séparés.

### Étape 3 : enregistrer votre serveur auprès de Claude Code

Ajoutez le serveur à votre configuration MCP pour que Claude Code sache comment le démarrer. Pour un `.mcp.json` au niveau du projet, dans le même répertoire, utilisez un chemin relatif. Pour une configuration au niveau utilisateur dans `~/.claude.json`, utilisez le chemin absolu complet pour que le serveur soit trouvable depuis n'importe quel projet :

```json title=".mcp.json" theme={null}
{
  "mcpServers": {
    "webhook": { "command": "bun", "args": ["./webhook.ts"] }
  }
}
```

Claude Code lit votre configuration MCP au démarrage et lance chaque serveur comme sous-processus.

### Étape 4 : tester

Pendant l'aperçu de recherche, les channels personnalisés ne figurent pas sur la liste d'autorisation ; démarrez donc Claude Code avec le flag de développement :

```bash theme={null}
claude --dangerously-load-development-channels server:webhook
```

Claude Code affiche d'abord une boîte de dialogue d'avertissement plein écran listant les channels de développement que vous chargez. Sélectionnez **I am using this for local development** pour continuer, ou **Exit** pour quitter.

La première fois que vous démarrez une session dans ce projet, Claude Code demande aussi votre consentement avant d'utiliser le nouveau serveur de `.mcp.json`. La boîte de dialogue indique « New MCP server found in this project: webhook ». Sélectionnez **Use this MCP server** pour continuer.

Une fois accepté, Claude Code lance votre `webhook.ts` comme sous-processus, et l'écouteur HTTP démarre automatiquement sur le port configuré (8788 dans cet exemple). Vous n'avez pas besoin de lancer le serveur vous-même.

Un avis en grisé sous la bannière de démarrage confirme que le channel est enregistré : `Channels (experimental) messages from server:webhook inject directly in this session · restart without --dangerously-load-development-channels to stop`.

Si vous voyez « blocked by org policy », l'administrateur de votre organisation doit d'abord [activer les channels](https://code.claude.com/docs/en/channels#enterprise-controls).

Dans un autre terminal, simulez un webhook en envoyant un HTTP POST contenant un message à votre serveur. Cet exemple envoie une alerte d'échec de CI au port 8788 (ou au port que vous avez configuré) :

```bash theme={null}
curl -X POST localhost:8788 -d "build failed on main: https://ci.example.com/run/1234"
```

La charge utile arrive dans le contexte de Claude sous forme de balise `<channel>` :

```text theme={null}
<channel source="webhook" path="/" method="POST">
build failed on main: https://ci.example.com/run/1234
</channel>
```

Votre terminal affiche l'événement sous la forme d'un résumé d'une ligne, `← webhook: build failed on main: https://ci.example.com/run/1234`, plutôt que la balise brute. Vous verrez ensuite Claude commencer à répondre : lire des fichiers, exécuter des commandes, ou tout ce que le message demande. C'est un channel unidirectionnel : Claude agit dans votre session mais ne renvoie rien via le webhook. Pour ajouter des réponses, voir [Exposer un outil de réponse](#exposer-un-outil-de-réponse).

Si l'événement n'arrive pas, le diagnostic dépend de ce que `curl` a renvoyé :

* **`curl` réussit mais rien n'atteint Claude** : lancez `/mcp` dans votre session pour vérifier l'état du serveur. Un statut `failed` signale généralement une erreur de dépendance ou d'import dans votre fichier serveur. Pour voir la trace stderr, redémarrez avec `claude --debug --dangerously-load-development-channels server:webhook` et consultez le journal de débogage dans `~/.claude/debug/<session-id>.txt`.
* **`curl` échoue avec « connection refused »** : soit le port n'est pas encore lié, soit un processus résiduel d'une exécution précédente l'occupe. `lsof -i :<port>` montre ce qui écoute ; tuez (`kill`) le processus résiduel avant de redémarrer votre session.

Le [serveur fakechat](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/fakechat) étend ce modèle avec une interface web, des pièces jointes et un outil de réponse pour un chat bidirectionnel.

## Tester pendant l'aperçu de recherche

Pendant l'aperçu de recherche, chaque channel doit figurer sur la [liste d'autorisation approuvée](https://code.claude.com/docs/en/channels#research-preview) pour s'enregistrer. Le flag de développement contourne la liste d'autorisation pour des entrées précises, après une demande de confirmation. Cet exemple montre les deux types d'entrée :

```bash theme={null}
# Tester un plugin que vous développez
claude --dangerously-load-development-channels plugin:yourplugin@yourmarketplace

# Tester un serveur .mcp.json brut (pas encore de plugin autour)
claude --dangerously-load-development-channels server:webhook
```

Utilisez le flag de développement dans une session interactive, où Claude Code peut afficher la demande de confirmation. Si vous le passez en mode non interactif avec `-p` ou via l'Agent SDK, Claude Code ignore le flag et le channel ne s'enregistre pas.

Le contournement s'applique entrée par entrée. Combiner ce flag avec `--channels` n'étend pas le contournement aux entrées de `--channels`. Pendant l'aperçu de recherche, votre channel ne figure pas sur la liste approuvée ; il reste donc sur le flag de développement pendant que vous le construisez et le testez.

> **Note** : ce flag ne contourne que la liste d'autorisation. La politique d'organisation `channelsEnabled` s'applique toujours. Ne l'utilisez pas pour exécuter des channels provenant de sources non fiables.

## Options du serveur

Un channel définit ces options dans le constructeur [`Server`](https://modelcontextprotocol.io/docs/learn/server-concepts). Les champs `instructions` et `capabilities.tools` relèvent du [MCP standard](https://modelcontextprotocol.io/docs/learn/server-concepts) ; `capabilities.experimental['claude/channel']` et `capabilities.experimental['claude/channel/permission']` sont les ajouts propres aux channels :

| Champ | Type | Description |
| :- | :- | :- |
| `capabilities.experimental['claude/channel']` | `object` | Obligatoire. Toujours `{}`. Sa présence enregistre l'écouteur de notifications. |
| `capabilities.experimental['claude/channel/permission']` | `object` ou `false` | Facultatif. Définissez-le à `{}` pour déclarer que ce channel peut recevoir des demandes de relais de permission. Une fois déclaré, Claude Code transfère les demandes d'approbation d'outils à votre channel pour que vous puissiez les approuver ou les refuser à distance. Pour ne pas participer, omettez la clé ou mettez-la à `false`. Avant la v2.1.234, Claude Code considérait `false` comme une déclaration. Voir [Relayer les demandes de permission](#relayer-les-demandes-de-permission). |
| `capabilities.tools` | `object` | Bidirectionnel uniquement. Toujours `{}`. Capacité d'outils MCP standard. Voir [Exposer un outil de réponse](#exposer-un-outil-de-réponse). |
| `instructions` | `string` | Recommandé. Claude Code la transmet à Claude comme contexte à la connexion du serveur. Indiquez à Claude quels événements attendre, ce que signifient les attributs de la balise `<channel>`, s'il doit répondre et, si oui, quel outil utiliser et quel attribut renvoyer (comme `chat_id`). |

Pour créer un channel unidirectionnel, omettez `capabilities.tools`. Cet exemple montre une configuration bidirectionnelle avec la capacité de channel, les outils et les instructions :

```ts theme={null}
import { Server } from '@modelcontextprotocol/sdk/server/index.js'

const mcp = new Server(
  { name: 'your-channel', version: '0.0.1' },
  {
    capabilities: {
      experimental: { 'claude/channel': {} },  // enregistre l'écouteur du channel
      tools: {},  // à omettre pour les channels unidirectionnels
    },
    // Claude Code transmet ceci à Claude comme contexte à la connexion du serveur, pour qu'il sache traiter vos événements
    instructions: 'Messages arrive as <channel source="your-channel" ...>. Reply with the reply tool.',
  },
)
```

## Format des notifications

Votre serveur émet `notifications/claude/channel` avec deux paramètres :

| Champ | Type | Description |
| :- | :- | :- |
| `content` | `string` | Le corps de l'événement. Livré comme corps de la balise `<channel>`. |
| `meta` | `Record<string, string>` | Facultatif. Chaque entrée devient un attribut de la balise `<channel>`, pour du contexte de routage comme l'ID de chat, le nom de l'expéditeur ou la gravité d'une alerte. Les clés doivent être des identifiants : lettres, chiffres et underscores uniquement. Les clés contenant des tirets ou d'autres caractères sont supprimées silencieusement. |

Votre serveur pousse des événements en appelant `mcp.notification()` sur l'instance `Server`. Cet exemple pousse une alerte d'échec de CI avec deux clés meta :

```ts theme={null}
await mcp.notification({
  method: 'notifications/claude/channel',
  params: {
    content: 'build failed on main: https://ci.example.com/run/1234',
    meta: { severity: 'high', run_id: '1234' },
  },
})
```

L'événement arrive dans le contexte de Claude enveloppé dans une balise `<channel>`. L'attribut `source` est défini automatiquement à partir du nom configuré de votre serveur :

```text theme={null}
<channel source="your-channel" severity="high" run_id="1234">
build failed on main: https://ci.example.com/run/1234
</channel>
```

Claude Code n'accuse pas réception des notifications. Le `await` sur `mcp.notification()` se résout quand le message est écrit dans le transport, pas quand Claude l'a traité. Si la session n'a pas chargé votre serveur comme channel, ou si la politique de l'organisation le bloque, Claude Code supprime les événements silencieusement et ne renvoie aucune erreur à votre serveur.

Si vous avez besoin d'une confirmation de livraison, suivez l'état des événements dans votre serveur et exposez un [outil de réponse](#exposer-un-outil-de-réponse) que Claude peut appeler pour rendre compte du statut.

Les événements sont mis en file d'attente dans la session et traités dans l'ordre. Si plusieurs notifications arrivent pendant que Claude est occupé, elles sont livrées ensemble au tour suivant et Claude les traite en groupe. Pour traiter des flux d'événements indépendants en parallèle, exécutez des sessions séparées.

## Exposer un outil de réponse

Si votre channel est bidirectionnel, comme une passerelle de chat plutôt qu'un simple transmetteur d'alertes, exposez un [outil MCP](https://modelcontextprotocol.io/docs/concepts/tools) standard que Claude peut appeler pour renvoyer des messages. L'enregistrement de l'outil n'a rien de spécifique aux channels. Un outil de réponse comporte trois composants :

1. une entrée `tools: {}` dans les capacités du constructeur `Server`, pour que Claude Code découvre l'outil ;
2. des gestionnaires (handlers) d'outil qui définissent le schéma de l'outil et implémentent la logique d'envoi ;
3. une chaîne `instructions` dans le constructeur `Server` qui indique à Claude quand et comment appeler l'outil.

Pour les ajouter au [récepteur de webhooks ci-dessus](#exemple--construire-un-récepteur-de-webhooks) :

### Étape 1 : activer la découverte des outils

Dans le constructeur `Server` de `webhook.ts`, ajoutez `tools: {}` aux capacités pour que Claude Code sache que votre serveur propose des outils :

```ts theme={null}
capabilities: {
  experimental: { 'claude/channel': {} },
  tools: {},  // active la découverte des outils
},
```

### Étape 2 : enregistrer l'outil de réponse

Ajoutez ce qui suit à `webhook.ts`. L'`import` se place en haut du fichier avec vos autres imports ; les deux gestionnaires se placent entre le constructeur `Server` et `mcp.connect()`. Cela enregistre un outil `reply` que Claude peut appeler avec un `chat_id` et un `text` :

```ts theme={null}
// Ajoutez cet import en haut de webhook.ts
import { ListToolsRequestSchema, CallToolRequestSchema } from '@modelcontextprotocol/sdk/types.js'

// Claude interroge ceci au démarrage pour découvrir les outils proposés par votre serveur
mcp.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: 'reply',
    description: 'Send a message back over this channel',
    // inputSchema indique à Claude quels arguments passer
    inputSchema: {
      type: 'object',
      properties: {
        chat_id: { type: 'string', description: 'The conversation to reply in' },
        text: { type: 'string', description: 'The message to send' },
      },
      required: ['chat_id', 'text'],
    },
  }],
}))

// Claude appelle ceci quand il veut invoquer un outil
mcp.setRequestHandler(CallToolRequestSchema, async req => {
  if (req.params.name === 'reply') {
    const { chat_id, text } = req.params.arguments as { chat_id: string; text: string }
    // send() est votre sortie : POST vers votre plateforme de chat ou, pour des
    // tests locaux, la diffusion SSE montrée dans l'exemple complet ci-dessous.
    send(`Reply to ${chat_id}: ${text}`)
    return { content: [{ type: 'text', text: 'sent' }] }
  }
  throw new Error(`unknown tool: ${req.params.name}`)
})
```

### Étape 3 : mettre à jour les instructions

Mettez à jour la chaîne `instructions` de votre constructeur `Server` pour que Claude sache router ses réponses via l'outil. Cet exemple indique à Claude de passer le `chat_id` de la balise entrante :

```ts theme={null}
instructions: 'Messages arrive as <channel source="webhook" chat_id="...">. Reply with the reply tool, passing the chat_id from the tag.'
```

*(« Les messages arrivent sous la forme <channel source="webhook" chat_id="...">. Répondez avec l'outil reply en passant le chat_id de la balise. »)*

### `webhook.ts` complet avec outil de réponse

Voici le `webhook.ts` complet avec prise en charge bidirectionnelle. Les réponses sortantes sont diffusées sur `GET /events` via [Server-Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) (SSE), si bien que `curl -N localhost:8788/events` peut les suivre en direct ; le chat entrant arrive sur `POST /` :

```ts title="Full webhook.ts with reply tool" expandable theme={null}
#!/usr/bin/env bun
import { Server } from '@modelcontextprotocol/sdk/server/index.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'
import { ListToolsRequestSchema, CallToolRequestSchema } from '@modelcontextprotocol/sdk/types.js'

// --- Sortie : écrit vers tous les écouteurs curl -N sur /events ------------
// Une vraie passerelle ferait plutôt un POST vers votre plateforme de chat.
const listeners = new Set<(chunk: string) => void>()
function send(text: string) {
  const chunk = text.split('\n').map(l => `data: ${l}\n`).join('') + '\n'
  for (const emit of listeners) emit(chunk)
}

const mcp = new Server(
  { name: 'webhook', version: '0.0.1' },
  {
    capabilities: {
      experimental: { 'claude/channel': {} },
      tools: {},
    },
    instructions: 'Messages arrive as <channel source="webhook" chat_id="...">. Reply with the reply tool, passing the chat_id from the tag.',
  },
)

mcp.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: 'reply',
    description: 'Send a message back over this channel',
    inputSchema: {
      type: 'object',
      properties: {
        chat_id: { type: 'string', description: 'The conversation to reply in' },
        text: { type: 'string', description: 'The message to send' },
      },
      required: ['chat_id', 'text'],
    },
  }],
}))

mcp.setRequestHandler(CallToolRequestSchema, async req => {
  if (req.params.name === 'reply') {
    const { chat_id, text } = req.params.arguments as { chat_id: string; text: string }
    send(`Reply to ${chat_id}: ${text}`)
    return { content: [{ type: 'text', text: 'sent' }] }
  }
  throw new Error(`unknown tool: ${req.params.name}`)
})

await mcp.connect(new StdioServerTransport())

let nextId = 1
Bun.serve({
  port: 8788,
  hostname: '127.0.0.1',
  idleTimeout: 0,  // ne ferme pas les flux SSE inactifs
  async fetch(req) {
    const url = new URL(req.url)

    // GET /events : flux SSE pour que curl -N puisse suivre les réponses de Claude en direct
    if (req.method === 'GET' && url.pathname === '/events') {
      const stream = new ReadableStream({
        start(ctrl) {
          ctrl.enqueue(': connected\n\n')  // pour que curl affiche quelque chose immédiatement
          const emit = (chunk: string) => ctrl.enqueue(chunk)
          listeners.add(emit)
          req.signal.addEventListener('abort', () => listeners.delete(emit))
        },
      })
      return new Response(stream, {
        headers: { 'Content-Type': 'text/event-stream', 'Cache-Control': 'no-cache' },
      })
    }

    // POST : transfère à Claude comme événement de channel
    const body = await req.text()
    const chat_id = String(nextId++)
    await mcp.notification({
      method: 'notifications/claude/channel',
      params: {
        content: body,
        meta: { chat_id, path: url.pathname, method: req.method },
      },
    })
    return new Response('ok')
  },
})
```

Le [serveur fakechat](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/fakechat) montre un exemple plus complet avec pièces jointes et modification de messages.

## Filtrer les messages entrants

Un channel non filtré est un vecteur d'injection de prompt. Toute personne pouvant atteindre votre point de terminaison peut placer du texte devant Claude. Un channel qui écoute une plateforme de chat ou un point de terminaison public a besoin d'une vraie vérification de l'expéditeur avant d'émettre quoi que ce soit.

Vérifiez l'expéditeur par rapport à une liste d'autorisation avant d'appeler `mcp.notification()`. Cet exemple ignore tout message d'un expéditeur absent de l'ensemble :

```ts theme={null}
const allowed = new Set(loadAllowlist())  // depuis votre access.json ou équivalent

// dans votre gestionnaire de messages, avant d'émettre :
if (!allowed.has(message.from.id)) {  // l'expéditeur, pas le salon
  return  // ignore silencieusement
}
await mcp.notification({ ... })
```

Filtrez sur l'identité de l'expéditeur, pas sur celle du chat ou du salon : `message.from.id` dans l'exemple, et non `message.chat.id`. Dans les conversations de groupe, ces valeurs diffèrent, et filtrer sur le salon permettrait à n'importe quel membre d'un groupe autorisé d'injecter des messages dans la session.

Les channels [Telegram](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/telegram) et [Discord](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/discord) filtrent de la même manière sur une liste d'autorisation d'expéditeurs. Ils initialisent la liste par [appairage](https://code.claude.com/docs/en/channels#security). Voir l'une ou l'autre implémentation pour le flux d'appairage complet. Le channel [iMessage](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/imessage) adopte une autre approche : il détecte au démarrage les propres adresses de l'utilisateur dans la base de données Messages et les laisse passer automatiquement, les autres expéditeurs étant ajoutés par handle.

## Relayer les demandes de permission

Quand Claude appelle un outil qui nécessite une approbation, la boîte de dialogue du terminal local s'ouvre et la session attend. Un channel bidirectionnel peut choisir de recevoir la même demande en parallèle et de vous la relayer sur un autre appareil. Les deux restent actives : vous pouvez répondre dans le terminal ou sur votre téléphone, et Claude Code applique la première réponse arrivée puis ferme l'autre.

Le relais couvre les approbations d'utilisation d'outils comme `Bash`, `Write` et `Edit`. Les boîtes de dialogue de confiance du projet (project trust) et de consentement aux serveurs MCP ne sont pas relayées ; elles n'apparaissent que dans le terminal local.

Claude Code v2.1.234+ n'envoie les demandes de permission qu'aux serveurs qu'il a enregistrés comme channels pour la session ; le relais est donc soumis aux mêmes [activation par session et contrôles d'organisation](https://code.claude.com/docs/en/channels#security) que la livraison des messages. Le relais exige aussi que vous activiez le serveur avec `--channels` ou le flag de développement, et que le serveur déclare la capacité de permission.

### Fonctionnement du relais

Quand une demande de permission s'ouvre, la boucle de relais comporte quatre étapes :

1. Claude Code génère un court identifiant de requête et notifie votre serveur.
2. Votre serveur transfère la demande et l'identifiant à votre application de chat.
3. L'utilisateur distant répond par oui ou non accompagné de cet identifiant.
4. Votre gestionnaire entrant analyse la réponse pour en tirer un verdict, et Claude Code ne l'applique que si l'identifiant correspond à une requête ouverte.

La boîte de dialogue du terminal local reste ouverte pendant tout ce temps. Si quelqu'un au terminal répond avant l'arrivée du verdict distant, c'est cette réponse qui est appliquée et la requête distante en attente est abandonnée.

*(Diagramme de séquence : Claude Code envoie une notification `permission_request` au serveur de channel ; le serveur met en forme la demande et l'envoie à l'application de chat ; l'humain répond avec un verdict ; le serveur analyse cette réponse et renvoie une notification `permission` à Claude Code.)*

### Champs de la demande de permission

La notification sortante émise par Claude Code est `notifications/claude/channel/permission_request`. Comme pour la [notification de channel](#format-des-notifications), le transport est du MCP standard, mais la méthode et le schéma sont des extensions de Claude Code. L'objet `params` contient quatre champs chaîne que votre serveur met en forme dans la demande sortante :

| Champ | Description |
| - | - |
| `request_id` | Cinq lettres minuscules tirées de `a`-`z` sans `l`, pour qu'il ne soit jamais confondu avec un `1` ou un `I` quand on le tape sur un téléphone. Incluez-le dans votre demande sortante pour qu'il puisse être répété dans la réponse. Claude Code n'accepte qu'un verdict portant un identifiant qu'il a lui-même émis. La boîte de dialogue du terminal local n'affiche pas cet identifiant : votre gestionnaire sortant est le seul moyen de le connaître. |
| `tool_name` | Nom de l'outil que Claude veut utiliser, par exemple `Bash` ou `Write`. |
| `description` | Résumé lisible de ce que fait cet appel d'outil précis, jamais la commande elle-même. Pour un appel Bash, c'est la description de la commande donnée par Claude ; quand le modèle ne fournit pas de description, le champ vaut la constante `Run shell command` et ne contient aucun détail de la commande. Affichez `input_preview` quand vous avez la place. |
| `input_preview` | Les arguments de l'outil sous forme de texte d'affichage au format JSON, indexé par champ de premier niveau. Pour Bash, c'est la commande ; pour Write, le chemin du fichier et le contenu. Omettez-le de votre demande si vous n'avez la place que pour un message d'une ligne. C'est votre serveur qui décide quoi afficher. |

Les clients Claude Code v2.1.211+ assainissent (sanitize) `description` et `input_preview` avant de les relayer. Attendez-vous à trois changements dans le texte reçu :

* Claude Code neutralise les caractères d'inversion de direction (direction-override), les caractères invisibles et les sosies de guillemets et de chevrons.
* Claude Code réduit chaque suite d'espaces blancs à une seule espace.
* Claude Code relaie le texte en entier jusqu'à 3 500 points de code. Pour une valeur plus longue, vous recevez son début et sa fin autour d'un marqueur comptabilisé `⋯ N code points elided ⋯`. La fin d'une longue commande parvient donc toujours à l'approbateur.

Pour `input_preview`, Claude Code applique la limite de 3 500 séparément à chaque champ de premier niveau des arguments et conserve les guillemets structurels propres au JSON. Les clients antérieurs à la v2.1.211 relaient `description` brute et tronquent `input_preview` à 200 unités UTF-16 avec des points de suspension finaux.

Les clients Claude Code v2.1.234+ relaient le marqueur `(value unserializable)` à la place d'une valeur de champ d'`input_preview` qu'ils ne peuvent pas sérialiser de façon sûre, comme une structure circulaire ou un tableau extrêmement grand. Vous recevez quand même la clé du champ, et les autres champs de l'aperçu sont inchangés.

Les clients Claude Code v2.1.234+ masquent aussi les identifiants secrets (credentials) dans `description` et `input_preview`. Vous recevez `[REDACTED]` à la place d'un jeton d'identification de fournisseur reconnaissable, comme une clé API ou un jeton d'accès personnel. Attendez-vous à trois effets du masquage lors de l'affichage des champs :

* Claude Code masque les noms de clés dans `input_preview` ainsi que leurs valeurs. Un nom de clé que vous affichez peut ne pas correspondre au nom de clé de l'entrée.
* Claude Code ne masque jamais un segment contenant de la syntaxe shell, des caractères de chemin ou des caractères d'URL. Un masque ne peut donc pas cacher la commande, le chemin de fichier ou la destination à approuver.
* Claude Code ne masque pas un secret dépourvu de préfixe reconnaissable, ni un secret qui s'étend sur des espaces, comme un bloc de clé privée. Les deux parviennent non masqués à votre serveur.

Le masquage ne change pas les destinataires des champs. Ce qui reste non masqué ne va qu'aux serveurs que vous avez activés avec `--channels` ou le flag de développement. Traitez les deux champs comme non fiables, sauf si vous contrôlez la flotte de clients.

Le verdict que votre serveur renvoie est `notifications/claude/channel/permission`, avec deux champs : `request_id`, qui répète l'identifiant ci-dessus, et `behavior`, défini à `'allow'` ou `'deny'`. Allow laisse l'appel d'outil se poursuivre ; deny le rejette. Aucun des deux verdicts n'affecte les appels futurs.

### Ajouter le relais à une passerelle de chat

Ajouter le relais des permissions à un channel bidirectionnel demande trois composants :

1. une entrée `claude/channel/permission: {}` sous les capacités `experimental` de votre constructeur `Server`, pour que Claude Code sache qu'il doit transférer les demandes ;
2. un gestionnaire de notification pour `notifications/claude/channel/permission_request` qui met en forme la demande et l'envoie via l'API de votre plateforme ;
3. une vérification dans votre gestionnaire de messages entrants qui reconnaît `yes <id>` ou `no <id>` et émet un verdict `notifications/claude/channel/permission` au lieu de transférer le texte à Claude.

> **Avertissement** : ne déclarez la capacité que si votre channel [authentifie l'expéditeur](#filtrer-les-messages-entrants), car toute personne pouvant répondre via votre channel peut approuver ou refuser l'utilisation d'outils dans votre session.

Pour les ajouter à une passerelle de chat bidirectionnelle comme celle assemblée dans [Exposer un outil de réponse](#exposer-un-outil-de-réponse) :

#### Étape 1 : déclarer la capacité de permission

Dans votre constructeur `Server`, ajoutez `claude/channel/permission: {}` à côté de `claude/channel` sous `experimental` :

```ts theme={null}
capabilities: {
  experimental: {
    'claude/channel': {},
    'claude/channel/permission': {},  // active le relais des permissions
  },
  tools: {},
},
```

#### Étape 2 : traiter la requête entrante

Enregistrez un gestionnaire de notification entre votre constructeur `Server` et `mcp.connect()`. Claude Code l'appelle avec les [quatre champs de la requête](#champs-de-la-demande-de-permission) quand une boîte de dialogue de permission s'ouvre. Votre gestionnaire met en forme la demande pour votre plateforme et inclut les instructions pour répondre avec l'identifiant :

```ts theme={null}
import { z } from 'zod'

// setNotificationHandler route selon le z.literal du champ method,
// donc ce schéma est à la fois le validateur et la clé de dispatch
const PermissionRequestSchema = z.object({
  method: z.literal('notifications/claude/channel/permission_request'),
  params: z.object({
    request_id: z.string(),     // cinq lettres minuscules, à inclure telles quelles dans votre demande
    tool_name: z.string(),      // ex. "Bash", "Write"
    description: z.string(),    // résumé de cet appel. À traiter comme non fiable.
    input_preview: z.string(),  // arguments de l'outil en texte au format JSON. À traiter comme non fiable.
  }),
})

mcp.setNotificationHandler(PermissionRequestSchema, async ({ params }) => {
  // send() est votre sortie : POST vers votre plateforme de chat ou, pour des
  // tests locaux, la diffusion SSE montrée dans l'exemple complet ci-dessous.
  send(
    `Claude wants to run ${params.tool_name}: ${params.description}\n` +
    // input_preview porte les vrais arguments ; affichez-le quand vous
    // avez la place : pour Bash, la description seule peut se réduire à
    // "Run shell command" sans aucun détail de la commande
    `${params.input_preview}\n\n` +
    // l'ID dans l'instruction est ce que votre gestionnaire entrant analyse à l'étape 3
    `Reply "yes ${params.request_id}" or "no ${params.request_id}"`,
  )
})
```

#### Étape 3 : intercepter le verdict dans votre gestionnaire entrant

Votre gestionnaire entrant est la boucle ou le callback qui reçoit les messages de votre plateforme : l'endroit même où vous [filtrez sur l'expéditeur](#filtrer-les-messages-entrants) et émettez `notifications/claude/channel` pour transférer le chat à Claude. Ajoutez, avant l'appel de transfert du chat, une vérification qui reconnaît le format de verdict et émet à la place la notification de permission.

L'expression régulière correspond au format d'identifiant généré par Claude Code : cinq lettres, jamais de `l`. Le flag `/i` tolère la mise en majuscule par l'autocorrection du téléphone ; mettez l'identifiant capturé en minuscules avant de le renvoyer.

```ts theme={null}
// correspond à "y abcde", "yes abcde", "n abcde", "no abcde"
// [a-km-z] est l'alphabet d'ID utilisé par Claude Code (minuscules, sans 'l')
// /i tolère l'autocorrection du téléphone ; mettez la capture en minuscules avant l'envoi
const PERMISSION_REPLY_RE = /^\s*(y|yes|n|no)\s+([a-km-z]{5})\s*$/i

async function onInbound(message: PlatformMessage) {
  if (!allowed.has(message.from.id)) return  // filtrer d'abord sur l'expéditeur

  const m = PERMISSION_REPLY_RE.exec(message.text)
  if (m) {
    // m[1] est le mot du verdict, m[2] l'ID de la requête
    // émet la notification de verdict vers Claude Code au lieu du chat
    await mcp.notification({
      method: 'notifications/claude/channel/permission',
      params: {
        request_id: m[2].toLowerCase(),  // normalise en cas de majuscules dues à l'autocorrection
        behavior: m[1].toLowerCase().startsWith('y') ? 'allow' : 'deny',
      },
    })
    return  // traité comme verdict, ne pas le transférer aussi comme chat
  }

  // ne correspond pas au format de verdict : on retombe sur le chemin de chat normal
  await mcp.notification({
    method: 'notifications/claude/channel',
    params: { content: message.text, meta: { chat_id: String(message.chat.id) } },
  })
}
```

Une réponse distante qui ne correspond pas exactement au format attendu échoue de l'une de deux façons, et dans les deux cas la boîte de dialogue du terminal local reste ouverte :

* **Format différent** : l'expression régulière de votre gestionnaire entrant ne correspond pas, donc un texte comme `approve it` ou `yes` sans identifiant est transmis à Claude comme un message normal.
* **Bon format, mauvais identifiant** : votre serveur émet un verdict, mais Claude Code ne trouve aucune requête ouverte avec cet identifiant et l'ignore silencieusement.

### Exemple complet

Le `webhook.ts` assemblé ci-dessous combine les trois extensions de cette page : l'outil de réponse, le filtrage des expéditeurs et le relais des permissions. Si vous commencez ici, il vous faut aussi la [mise en place du projet et l'entrée `.mcp.json`](#exemple--construire-un-récepteur-de-webhooks) du parcours initial.

Pour rendre les deux sens testables avec curl, l'écouteur HTTP sert deux chemins :

* **`GET /events`** : maintient ouvert un flux SSE et pousse chaque message sortant sous forme de ligne `data:`, pour que `curl -N` puisse voir arriver en direct les réponses de Claude et les demandes de permission.
* **`POST /`** : le côté entrant, même gestionnaire que précédemment, avec désormais la vérification du format de verdict insérée avant la branche de transfert du chat.

```ts title="Full webhook.ts with permission relay" expandable theme={null}
#!/usr/bin/env bun
import { Server } from '@modelcontextprotocol/sdk/server/index.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'
import { ListToolsRequestSchema, CallToolRequestSchema } from '@modelcontextprotocol/sdk/types.js'
import { z } from 'zod'

// --- Sortie : écrit vers tous les écouteurs curl -N sur /events ------------
// Une vraie passerelle ferait plutôt un POST vers votre plateforme de chat.
const listeners = new Set<(chunk: string) => void>()
function send(text: string) {
  const chunk = text.split('\n').map(l => `data: ${l}\n`).join('') + '\n'
  for (const emit of listeners) emit(chunk)
}

// Liste d'autorisation des expéditeurs. Pour ce parcours local, on fait confiance
// à la seule valeur d'en-tête X-Sender "dev" ; une vraie passerelle vérifierait
// l'ID utilisateur de la plateforme.
const allowed = new Set(['dev'])

const mcp = new Server(
  { name: 'webhook', version: '0.0.1' },
  {
    capabilities: {
      experimental: {
        'claude/channel': {},
        'claude/channel/permission': {},  // active le relais des permissions
      },
      tools: {},
    },
    instructions:
      'Messages arrive as <channel source="webhook" chat_id="...">. ' +
      'Reply with the reply tool, passing the chat_id from the tag.',
  },
)

// --- outil reply : Claude l'appelle pour renvoyer un message ----------------
mcp.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: 'reply',
    description: 'Send a message back over this channel',
    inputSchema: {
      type: 'object',
      properties: {
        chat_id: { type: 'string', description: 'The conversation to reply in' },
        text: { type: 'string', description: 'The message to send' },
      },
      required: ['chat_id', 'text'],
    },
  }],
}))

mcp.setRequestHandler(CallToolRequestSchema, async req => {
  if (req.params.name === 'reply') {
    const { chat_id, text } = req.params.arguments as { chat_id: string; text: string }
    send(`Reply to ${chat_id}: ${text}`)
    return { content: [{ type: 'text', text: 'sent' }] }
  }
  throw new Error(`unknown tool: ${req.params.name}`)
})

// --- relais des permissions : Claude Code (pas Claude) l'appelle quand une boîte de dialogue s'ouvre
const PermissionRequestSchema = z.object({
  method: z.literal('notifications/claude/channel/permission_request'),
  params: z.object({
    request_id: z.string(),
    tool_name: z.string(),
    description: z.string(),
    input_preview: z.string(),
  }),
})

mcp.setNotificationHandler(PermissionRequestSchema, async ({ params }) => {
  send(
    `Claude wants to run ${params.tool_name}: ${params.description}\n` +
    `${params.input_preview}\n\n` +
    `Reply "yes ${params.request_id}" or "no ${params.request_id}"`,
  )
})

await mcp.connect(new StdioServerTransport())

// --- HTTP sur :8788 : GET /events diffuse la sortie, POST route l'entrée ---
const PERMISSION_REPLY_RE = /^\s*(y|yes|n|no)\s+([a-km-z]{5})\s*$/i
let nextId = 1

Bun.serve({
  port: 8788,
  hostname: '127.0.0.1',
  idleTimeout: 0,  // ne ferme pas les flux SSE inactifs
  async fetch(req) {
    const url = new URL(req.url)

    // GET /events : flux SSE pour que curl -N suive en direct réponses et demandes
    if (req.method === 'GET' && url.pathname === '/events') {
      const stream = new ReadableStream({
        start(ctrl) {
          ctrl.enqueue(': connected\n\n')  // pour que curl affiche quelque chose immédiatement
          const emit = (chunk: string) => ctrl.enqueue(chunk)
          listeners.add(emit)
          req.signal.addEventListener('abort', () => listeners.delete(emit))
        },
      })
      return new Response(stream, {
        headers: { 'Content-Type': 'text/event-stream', 'Cache-Control': 'no-cache' },
      })
    }

    // tout le reste est entrant : filtrer d'abord sur l'expéditeur
    const body = await req.text()
    const sender = req.headers.get('X-Sender') ?? ''
    if (!allowed.has(sender)) return new Response('forbidden', { status: 403 })

    // vérifier le format de verdict avant de traiter comme du chat
    const m = PERMISSION_REPLY_RE.exec(body)
    if (m) {
      await mcp.notification({
        method: 'notifications/claude/channel/permission',
        params: {
          request_id: m[2].toLowerCase(),
          behavior: m[1].toLowerCase().startsWith('y') ? 'allow' : 'deny',
        },
      })
      return new Response('verdict recorded')
    }

    // chat normal : transférer à Claude comme événement de channel
    const chat_id = String(nextId++)
    await mcp.notification({
      method: 'notifications/claude/channel',
      params: { content: body, meta: { chat_id, path: url.pathname } },
    })
    return new Response('ok')
  },
})
```

Testez le chemin du verdict dans trois terminaux. Le premier est votre session Claude Code, démarrée avec le [flag de développement](#tester-pendant-laperçu-de-recherche) pour qu'elle lance `webhook.ts` :

```bash theme={null}
claude --dangerously-load-development-channels server:webhook
```

Ce parcours teste la boîte de dialogue de permission elle-même ; une fois la session ouverte, appuyez donc sur `Shift+Tab` jusqu'à ce que la barre d'état affiche `⏸ manual mode on`. En mode auto, c'est le classifieur qui déciderait de l'appel `reply` à votre place, et aucune boîte de dialogue ne s'ouvrirait pour que le côté distant y réponde.

Dans le deuxième, suivez le flux sortant pour voir les réponses de Claude et les éventuelles demandes de permission au fur et à mesure :

```bash theme={null}
curl -N localhost:8788/events
```

Dans le troisième, envoyez un message qui amènera Claude à tenter d'exécuter une commande :

```bash theme={null}
curl -d "list the files in this directory" -H "X-Sender: dev" localhost:8788
```

Lister des fichiers est une opération en lecture seule ; Claude l'exécute donc sans approbation. La boîte de dialogue de permission s'ouvre quand Claude appelle l'outil `reply` pour renvoyer sa réponse. La boîte de dialogue locale s'ouvre dans votre terminal Claude Code et, un instant plus tard, la demande pour `mcp__webhook__reply` apparaît dans le flux `/events`, avec l'identifiant de cinq lettres. Approuvez-la depuis le côté distant :

```bash theme={null}
curl -d "yes <id>" -H "X-Sender: dev" localhost:8788
```

La boîte de dialogue locale se ferme, l'outil `reply` s'exécute et la réponse de Claude apparaît dans le flux.

Les trois éléments propres aux channels dans ce fichier :

* **Capacités** dans le constructeur `Server` : `claude/channel` enregistre l'écouteur de notifications, `claude/channel/permission` active le relais des permissions, `tools` permet à Claude de découvrir l'outil de réponse.
* **Chemins sortants** : le gestionnaire de l'outil `reply` est ce que Claude appelle pour les réponses conversationnelles ; le gestionnaire de notification `PermissionRequestSchema` est ce que Claude Code appelle quand une boîte de dialogue de permission s'ouvre. Les deux appellent `send()` pour diffuser sur `/events`, mais ils sont déclenchés par des parties différentes du système.
* **Gestionnaire HTTP** : `GET /events` maintient un flux SSE ouvert pour que curl puisse suivre la sortie en direct ; `POST` est l'entrée, filtrée sur l'en-tête `X-Sender`. Un corps `yes <id>` ou `no <id>` part vers Claude Code comme notification de verdict et n'atteint jamais Claude ; tout le reste est transféré à Claude comme événement de channel.

## Empaqueter comme plugin

Pour rendre votre channel installable et partageable, enveloppez-le dans un [plugin](https://code.claude.com/docs/en/plugins/overview) et publiez-le sur une [marketplace](https://code.claude.com/docs/en/plugins/overview). Les utilisateurs l'installent avec `/plugin install`, puis l'activent par session avec `--channels plugin:<name>@<marketplace>`.

Un channel publié sur votre propre marketplace nécessite toujours `--dangerously-load-development-channels` pour s'exécuter, puisqu'il ne figure pas sur la [liste d'autorisation approuvée](https://code.claude.com/docs/en/channels#supported-channels). La liste d'autorisation par défaut est constituée des plugins de channel de `claude-plugins-official`. La marketplace communautaire ne fait pas partie de la liste d'autorisation des channels.

Si vous travaillez avec un contact partenaire d'Anthropic, contactez-le pour coordonner un référencement sur la marketplace officielle. Sur les forfaits Team et Enterprise, un administrateur peut à la place inclure votre plugin dans la liste [`allowedChannelPlugins`](https://code.claude.com/docs/en/channels#restrict-which-channel-plugins-can-run) propre à l'organisation, qui remplace la liste d'autorisation par défaut d'Anthropic.

## Voir aussi

* [Channels](https://code.claude.com/docs/en/channels) pour installer et utiliser Telegram, Discord, iMessage ou la démo fakechat, et pour activer les channels dans une organisation Team ou Enterprise ;
* [Implémentations de channels fonctionnelles](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins) pour du code serveur complet avec flux d'appairage, outils de réponse et pièces jointes ;
* [MCP](https://code.claude.com/docs/en/mcp) pour le protocole sous-jacent implémenté par les serveurs de channel ;
* [Plugins](https://code.claude.com/docs/en/plugins/overview) pour empaqueter votre channel afin que les utilisateurs puissent l'installer avec `/plugin install`.
