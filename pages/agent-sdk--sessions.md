# Travailler avec les sessions

> Source : https://code.claude.com/docs/en/agent-sdk/sessions · [← Index](../31-index-documentation-officielle.md)

*Comment les sessions conservent l'historique de conversation de l'agent, et quand utiliser continue, resume et fork pour revenir à une exécution antérieure.*

Une **session** est l'historique de conversation que le SDK accumule pendant que votre agent travaille. Elle contient votre prompt, chaque appel d'outil effectué par l'agent, chaque résultat d'outil et chaque réponse. Le SDK l'écrit automatiquement sur disque pour que vous puissiez y revenir plus tard.

Revenir à une session signifie que l'agent dispose de tout le contexte antérieur : fichiers déjà lus, analyses déjà effectuées, décisions déjà prises. Vous pouvez poser une question de suivi, reprendre après une interruption, ou bifurquer pour essayer une autre approche.

> **Note** : les sessions conservent la **conversation**, pas le système de fichiers. Pour prendre un instantané des modifications de fichiers faites par l'agent et les annuler, utilisez le [file checkpointing](https://code.claude.com/docs/en/agent-sdk/file-checkpointing).

Ce guide explique comment choisir la bonne approche pour votre application, les interfaces du SDK qui suivent les sessions automatiquement, comment capturer les identifiants de session et utiliser `resume` et `fork` manuellement, et ce qu'il faut savoir pour reprendre des sessions d'un hôte à l'autre.

## Choisir une approche

Le niveau de gestion de session nécessaire dépend de la forme de votre application. La gestion de session entre en jeu lorsque vous envoyez plusieurs prompts qui doivent partager le même contexte. Au sein d'un seul appel `query()`, l'agent effectue déjà autant de tours que nécessaire, et les demandes de permission ainsi que `AskUserQuestion` sont [gérées dans la boucle](https://code.claude.com/docs/en/agent-sdk/user-input) (elles ne terminent pas l'appel).

| Ce que vous construisez | Quoi utiliser |
| :- | :- |
| Tâche ponctuelle : un seul prompt, sans suivi | Rien de plus. Un seul appel `query()` suffit. |
| Chat multi-tours dans un seul processus | [`ClaudeSDKClient` (Python) ou `continue: true` (TypeScript)](#gestion-automatique-des-sessions). Le SDK suit la session pour vous, sans manipuler d'ID. |
| Reprendre là où vous en étiez après un redémarrage du processus | `continue_conversation=True` (Python) / `continue: true` (TypeScript). Reprend la session la plus récente du répertoire, sans ID. |
| Reprendre une session passée précise (pas la plus récente) | Capturer l'ID de session et le passer à `resume`. |
| Essayer une approche alternative sans perdre l'originale | Forker (bifurquer) la session. |
| Tâche sans état, sans rien écrire sur disque | Définir [`persistSession: false`](https://code.claude.com/docs/en/agent-sdk/typescript#options) (TypeScript uniquement). La session n'existe qu'en mémoire pendant la durée de l'appel. En Python, définissez plutôt [`CLAUDE_CODE_SKIP_PROMPT_HISTORY`](https://code.claude.com/docs/en/env-vars) dans l'option `env` pour supprimer l'écriture des transcriptions. |

### Continue, resume et fork

Continue, resume et fork sont des champs d'options que vous définissez sur `query()` ([`ClaudeAgentOptions`](https://code.claude.com/docs/en/agent-sdk/python#claudeagentoptions) en Python, [`Options`](https://code.claude.com/docs/en/agent-sdk/typescript#options) en TypeScript).

**Continue** et **resume** reprennent tous deux une session existante et la prolongent. La différence tient à la façon dont ils trouvent cette session :

* **Continue** trouve la session la plus récente dans le répertoire courant. Vous n'avez rien à suivre. Convient bien lorsque votre application ne mène qu'une conversation à la fois.
* **Resume** prend un ID de session précis. C'est vous qui suivez l'ID. Nécessaire lorsque vous avez plusieurs sessions (par exemple une par utilisateur dans une application multi-utilisateurs) ou que vous voulez revenir à une session qui n'est pas la plus récente.

**Fork** est différent : il crée une nouvelle session qui démarre avec une copie de l'historique de l'originale. L'originale reste inchangée. Utilisez fork pour essayer une autre direction tout en gardant la possibilité de revenir en arrière.

## Gestion automatique des sessions

Les deux SDK offrent une interface qui suit l'état de session pour vous d'un appel à l'autre, sans avoir à transmettre les ID manuellement. Utilisez-les pour les conversations multi-tours au sein d'un seul processus.

### Python : `ClaudeSDKClient`

[`ClaudeSDKClient`](https://code.claude.com/docs/en/agent-sdk/python#claudesdkclient) gère les ID de session en interne. Chaque appel à `client.query()` continue automatiquement la même session. Appelez [`client.receive_response()`](https://code.claude.com/docs/en/agent-sdk/python#claudesdkclient) pour itérer sur les messages de la requête en cours. Utilisez le client comme gestionnaire de contexte asynchrone pour que l'ouverture et la fermeture de la connexion soient gérées pour vous, ou appelez `connect()` et `disconnect()` manuellement.

Cet exemple exécute deux requêtes sur le même `client`. La première demande à l'agent d'analyser un module ; la seconde lui demande de le refactoriser. Comme les deux appels passent par la même instance de client, la seconde requête dispose de tout le contexte de la première, sans `resume` explicite ni ID de session :

```python Python theme={null}
import asyncio
from claude_agent_sdk import (
    ClaudeSDKClient,
    ClaudeAgentOptions,
    AssistantMessage,
    ResultMessage,
    TextBlock,
)


def print_response(message):
    """Affiche uniquement les parties lisibles d'un message."""
    if isinstance(message, AssistantMessage):
        for block in message.content:
            if isinstance(block, TextBlock):
                print(block.text)
    elif isinstance(message, ResultMessage):
        cost = (
            f"${message.total_cost_usd:.4f}"
            if message.total_cost_usd is not None
            else "N/A"
        )
        print(f"[done: {message.subtype}, cost: {cost}]")


async def main():
    options = ClaudeAgentOptions(
        allowed_tools=["Read", "Edit", "Glob", "Grep"],
    )

    async with ClaudeSDKClient(options=options) as client:
        # Première requête : le client capture l'ID de session en interne
        await client.query("Analyze the auth module")
        async for message in client.receive_response():
            print_response(message)

        # Seconde requête : continue automatiquement la même session
        await client.query("Now refactor it to use JWT")
        async for message in client.receive_response():
            print_response(message)


asyncio.run(main())
```

Chaque requête affiche la réponse textuelle de l'agent suivie d'une ligne d'état issue du message de résultat, par exemple `[done: success, cost: $0.0042]`.

Voir la [référence du SDK Python](https://code.claude.com/docs/en/agent-sdk/python#choosing-between-query-and-claudesdkclient) pour savoir quand utiliser `ClaudeSDKClient` plutôt que la fonction autonome `query()`.

### TypeScript : `continue: true`

Le SDK TypeScript n'a pas d'objet client qui conserve la session comme le `ClaudeSDKClient` de Python. À la place, passez `continue: true` à chaque appel `query()` suivant, et le SDK reprend la session la plus récente du répertoire courant. Aucun suivi d'ID nécessaire.

Cet exemple effectue deux appels `query()` distincts. Le premier crée une nouvelle session ; le second définit `continue: true`, ce qui indique au SDK de trouver et reprendre la session la plus récente sur disque. L'agent dispose de tout le contexte du premier appel :

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

// Première requête : crée une nouvelle session
try {
  for await (const message of query({
    prompt: "Analyze the auth module",
    options: { allowedTools: ["Read", "Glob", "Grep"] }
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

// Seconde requête : continue: true reprend la session la plus récente
for await (const message of query({
  prompt: "Now refactor it to use JWT",
  options: {
    continue: true,
    allowedTools: ["Read", "Edit", "Write", "Glob", "Grep"]
  }
})) {
  if (message.type === "result" && message.subtype === "success") {
    console.log(message.result);
  }
}
```

> **Note** : l'[API de session V2](https://code.claude.com/docs/en/agent-sdk/typescript-v2-preview) expérimentale, qui fournissait `createSession()` avec un modèle `send` / `stream`, a été supprimée dans le TypeScript Agent SDK 0.3.142. Utilisez à la place la fonction `query()` et les options de session décrites sur cette page.

## Utiliser les options de session avec `query()`

### Capturer l'ID de session

Resume et fork exigent un ID de session. Lisez-le dans le champ `session_id` du message de résultat ([`ResultMessage`](https://code.claude.com/docs/en/agent-sdk/python#resultmessage) en Python, [`SDKResultMessage`](https://code.claude.com/docs/en/agent-sdk/typescript#sdkresultmessage) en TypeScript), présent sur chaque résultat, qu'il s'agisse d'un succès ou d'une erreur. En TypeScript, l'ID est aussi disponible plus tôt, comme champ direct du `SystemMessage` d'initialisation (`init`) ; en Python, il est imbriqué dans `SystemMessage.data`.

#### Python

```python Python theme={null}
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


async def main():
    session_id = None

    try:
        async for message in query(
            prompt="Analyze the auth module and suggest improvements",
            options=ClaudeAgentOptions(
                allowed_tools=["Read", "Glob", "Grep"],
            ),
        ):
            if isinstance(message, ResultMessage):
                session_id = message.session_id
                if message.subtype == "success":
                    print(message.result)
    except Exception as error:
        # Un query() ponctuel lève une exception après avoir produit un résultat d'erreur.
        # Si l'échec était un résultat d'erreur, la boucle ci-dessus a déjà capturé session_id ;
        # les échecs de connexion ou de processus ne produisent aucun message de résultat,
        # donc session_id reste None.
        print(f"Session ended with an error: {error}")

    print(f"Session ID: {session_id}")
    return session_id


session_id = asyncio.run(main())
```

#### TypeScript

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

let sessionId: string | undefined;

try {
  for await (const message of query({
    prompt: "Analyze the auth module and suggest improvements",
    options: { allowedTools: ["Read", "Glob", "Grep"] }
  })) {
    if (message.type === "result") {
      sessionId = message.session_id;
      if (message.subtype === "success") {
        console.log(message.result);
      }
    }
  }
} catch (error) {
  // Un query() ponctuel lève une exception après avoir produit un résultat d'erreur.
  // Si l'échec était un résultat d'erreur, la boucle ci-dessus a déjà capturé sessionId ;
  // les échecs de connexion ou de processus ne produisent aucun message de résultat,
  // donc sessionId reste undefined.
  console.error(`Session ended with an error: ${error}`);
}

console.log(`Session ID: ${sessionId}`);
```

Une fois la requête terminée, le script affiche la réponse de l'agent suivie d'une ligne telle que `Session ID: 5b3f2c1a-8d4e-4f6b-9a7c-2e1d0f9b8a6c`. Dans les sections suivantes, vous passez cet ID à `resume`.

### Reprendre par ID

Passez un ID de session à `resume` pour revenir à cette session précise. L'agent reprend avec tout le contexte, là où la session s'était arrêtée. Raisons courantes de reprendre :

* **Donner suite à une tâche terminée.** L'agent a déjà analysé quelque chose ; vous voulez maintenant qu'il agisse sur cette analyse sans relire les fichiers.
* **Récupérer après une limite.** La première exécution s'est terminée par `error_max_turns` ou `error_max_budget_usd` (voir [Handle the result](https://code.claude.com/docs/en/agent-sdk/agent-loop#handle-the-result)) ; reprenez avec une limite plus élevée. Dans un appel `query()` ponctuel, le SDK lève une exception après avoir produit ce résultat d'erreur : interceptez l'erreur avant de reprendre.
* **Redémarrer votre processus.** Vous avez capturé l'ID avant l'arrêt et voulez restaurer la conversation.

Cet exemple reprend la session de [Capturer l'ID de session](#capturer-lid-de-session) avec un prompt de suivi. Comme vous reprenez, l'agent a déjà l'analyse précédente en contexte :

#### Python

```python Python theme={null}
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

session_id = "..."  # L'ID capturé dans l'exemple précédent


async def main():
    # La session précédente a analysé le code ; on s'appuie maintenant sur cette analyse
    async for message in query(
        prompt="Now implement the refactoring you suggested",
        options=ClaudeAgentOptions(
            resume=session_id,
            allowed_tools=["Read", "Edit", "Write", "Glob", "Grep"],
        ),
    ):
        if isinstance(message, ResultMessage) and message.subtype == "success":
            print(message.result)


asyncio.run(main())
```

#### TypeScript

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

const sessionId = "..."; // L'ID capturé dans l'exemple précédent

// La session précédente a analysé le code ; on s'appuie maintenant sur cette analyse
for await (const message of query({
  prompt: "Now implement the refactoring you suggested",
  options: {
    resume: sessionId,
    allowedTools: ["Read", "Edit", "Write", "Glob", "Grep"]
  }
})) {
  if (message.type === "result" && message.subtype === "success") {
    console.log(message.result);
  }
}
```

Vous devriez obtenir une réponse qui s'appuie sur l'analyse précédente au lieu de repartir de zéro. Cela confirme que l'agent a repris la session avec son contexte antérieur intact.

> **Astuce** : Claude Code stocke les sessions sous `~/.claude/projects/<encoded-cwd>/*.jsonl`. Si vous définissez la variable d'environnement `CLAUDE_CONFIG_DIR`, cherchez plutôt sous `$CLAUDE_CONFIG_DIR/projects/`.
>
> Pour trouver le répertoire de votre session, remplacez chaque caractère non alphanumérique du répertoire de travail absolu par `-` : `/Users/me/proj` devient `-Users-me-proj`. Pour un répertoire de travail dont le nom converti dépasse 200 caractères, Claude Code [tronque le nom et ajoute un hash](https://code.claude.com/docs/en/sessions#where-transcripts-are-stored) ; faites donc correspondre les 200 premiers caractères du nom converti lorsque vous listez `projects/`.
>
> Si vous définissez [`CLAUDE_CODE_PROJECT_DIR_NAME`](https://code.claude.com/docs/en/sessions#name-the-project-directory-yourself) à côté de `CLAUDE_CONFIG_DIR`, cherchez sous ce nom dans `projects/`. Nécessite le TypeScript Agent SDK v0.3.234+ ou le Python Agent SDK v0.2.140+.
>
> Vous pouvez reprendre depuis n'importe quel répertoire de travail :
>
> * **Recherche inter-répertoires** : Claude Code cherche l'ID au-delà du répertoire de projet courant ; voir [Resume a session](https://code.claude.com/docs/en/sessions#where-the-session-picker-looks) pour l'ordre exact de recherche et le traitement des copies en double.
> * **Même machine uniquement** : le fichier de session doit toujours exister sur la machine courante.
>
> Avant la v2.1.223, la recherche se limitait au répertoire de projet courant et à ses worktrees git ; les versions du SDK qui embarquent un CLI plus ancien se comportent toujours ainsi.

Pour reprendre des sessions entre machines ou dans des environnements serverless, répliquez les transcriptions vers un stockage partagé avec un [adaptateur `SessionStore`](https://code.claude.com/docs/en/agent-sdk/session-storage).

### Forker pour explorer des alternatives

Forker crée une nouvelle session qui démarre avec une copie de l'historique de l'originale, puis diverge à partir de ce point. Le fork reçoit son propre ID de session ; l'ID et l'historique de l'originale restent inchangés. Vous obtenez deux sessions indépendantes que vous pouvez reprendre séparément.

> **Note** : forker bifurque l'historique de conversation, pas le système de fichiers. Si un agent forké modifie des fichiers, ces modifications sont réelles et visibles par toute session travaillant dans le même répertoire. Pour bifurquer et annuler des modifications de fichiers, utilisez le [file checkpointing](https://code.claude.com/docs/en/agent-sdk/file-checkpointing).

Cet exemple s'appuie sur [Capturer l'ID de session](#capturer-lid-de-session) : vous avez déjà analysé un module d'authentification dans `session_id` et voulez explorer OAuth2 sans perdre le fil centré sur JWT. Le premier bloc forke la session et capture l'ID du fork (`forked_id`) ; le second bloc reprend la `session_id` originale pour poursuivre la piste JWT. Vous avez désormais deux ID de session pointant vers deux historiques séparés :

#### Python

```python Python theme={null}
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

session_id = "..."  # L'ID capturé dans l'exemple précédent


async def main():
    # Fork : bifurque depuis session_id vers une nouvelle session
    forked_id = None
    try:
        async for message in query(
            prompt="Instead of JWT, outline how OAuth2 would work for the auth module",
            options=ClaudeAgentOptions(
                resume=session_id,
                fork_session=True,
                max_turns=5,
            ),
        ):
            if isinstance(message, ResultMessage):
                forked_id = message.session_id  # L'ID du fork, distinct de session_id
                if message.subtype == "success":
                    print(message.result)
    except Exception as error:
        # Un query() ponctuel lève une exception après avoir produit un résultat d'erreur.
        # Si l'échec était un résultat d'erreur, forked_id a déjà été capturé par la
        # boucle ci-dessus ; les échecs de connexion ou de processus ne produisent aucun message de résultat.
        print(f"Session ended with an error: {error}")

    print(f"Forked session: {forked_id}")

    # La session originale est intacte ; la reprendre poursuit le fil JWT
    try:
        async for message in query(
            prompt="Continue with the JWT approach",
            options=ClaudeAgentOptions(resume=session_id),
        ):
            if isinstance(message, ResultMessage) and message.subtype == "success":
                print(message.result)
    except Exception as error:
        # Un query() ponctuel lève une exception après avoir produit un résultat d'erreur.
        print(f"Session ended with an error: {error}")


asyncio.run(main())
```

#### TypeScript

```typescript TypeScript theme={null}
import { query } from "@anthropic-ai/claude-agent-sdk";

const sessionId = "..."; // L'ID capturé dans l'exemple précédent

// Fork : bifurque depuis sessionId vers une nouvelle session
let forkedId: string | undefined;

try {
  for await (const message of query({
    prompt: "Instead of JWT, outline how OAuth2 would work for the auth module",
    options: {
      resume: sessionId,
      forkSession: true,
      maxTurns: 5
    }
  })) {
    if (message.type === "system" && message.subtype === "init") {
      forkedId = message.session_id; // L'ID du fork, distinct de sessionId
    }
    if (message.type === "result" && message.subtype === "success") {
      console.log(message.result);
    }
  }
} catch (error) {
  // Un query() ponctuel lève une exception après avoir produit un résultat d'erreur.
  // Si l'échec était un résultat d'erreur, forkedId a déjà été capturé par la boucle
  // ci-dessus ; les échecs de connexion ou de processus ne produisent aucun message de résultat.
  console.error(`Session ended with an error: ${error}`);
}

console.log(`Forked session: ${forkedId}`);

// La session originale est intacte ; la reprendre poursuit le fil JWT
try {
  for await (const message of query({
    prompt: "Continue with the JWT approach",
    options: { resume: sessionId }
  })) {
    if (message.type === "result" && message.subtype === "success") {
      console.log(message.result);
    }
  }
} catch (error) {
  // Un query() ponctuel lève une exception après avoir produit un résultat d'erreur.
  console.error(`Session ended with an error: ${error}`);
}
```

Vous devriez constater que `forkedId` diffère de l'ID de session original. Reprendre la session originale poursuit toujours le fil JWT, ce qui confirme que le fork n'a pas modifié l'historique d'origine.

Récapitulatif des options de fork : `fork_session=True` (Python) / `forkSession: true` (TypeScript), toujours combiné avec `resume`.

## Reprendre d'un hôte à l'autre

Les fichiers de session sont locaux à la machine qui les a créés. Pour reprendre une session sur un autre hôte (workers de CI, conteneurs éphémères, serverless), choisissez l'approche adaptée :

* **Passer un session store.** Attachez un [adaptateur `sessionStore` / `session_store`](https://code.claude.com/docs/en/agent-sdk/session-storage) pour que le SDK réplique les transcriptions vers votre propre backend et qu'un autre hôte puisse les reprendre. La clé de recherche du store dérive du répertoire de travail : reprenez donc depuis un `cwd` correspondant à celui de l'exécution d'origine.

* **Déplacer le fichier de session.** Conservez `~/.claude/projects/<encoded-cwd>/<session-id>.jsonl` issu de la première exécution et restaurez-le dans n'importe quel répertoire sous `~/.claude/projects/` sur le nouvel hôte avant d'appeler `resume`.

  Claude Code cherche l'ID au-delà du répertoire de projet courant ; voir [Resume a session](https://code.claude.com/docs/en/sessions#where-the-session-picker-looks) pour l'ordre exact de recherche et le traitement des copies en double. Avant la v2.1.223, la recherche se limitait au répertoire de projet courant et à ses worktrees git ; les versions du SDK qui embarquent un CLI plus ancien se comportent toujours ainsi.

* **Ne pas dépendre de la reprise de session.** Capturez les résultats dont vous avez besoin (sortie d'analyse, décisions, diffs de fichiers) comme état applicatif et injectez-les dans le prompt d'une nouvelle session. C'est souvent plus robuste que de transporter des fichiers de transcription.

Les deux SDK exposent des fonctions pour énumérer les sessions sur disque et lire leurs messages : [`listSessions()`](https://code.claude.com/docs/en/agent-sdk/typescript#listsessions) et [`getSessionMessages()`](https://code.claude.com/docs/en/agent-sdk/typescript#getsessionmessages) en TypeScript, [`list_sessions()`](https://code.claude.com/docs/en/agent-sdk/python#list_sessions) et [`get_session_messages()`](https://code.claude.com/docs/en/agent-sdk/python#get_session_messages) en Python. Utilisez-les pour construire des sélecteurs de session personnalisés, une logique de nettoyage ou des visionneuses de transcriptions.

Les deux SDK exposent aussi des fonctions pour consulter et modifier des sessions individuelles : [`get_session_info()`](https://code.claude.com/docs/en/agent-sdk/python#get_session_info), [`rename_session()`](https://code.claude.com/docs/en/agent-sdk/python#rename_session) et [`tag_session()`](https://code.claude.com/docs/en/agent-sdk/python#tag_session) en Python, et [`getSessionInfo()`](https://code.claude.com/docs/en/agent-sdk/typescript#getsessioninfo), [`renameSession()`](https://code.claude.com/docs/en/agent-sdk/typescript#renamesession) et [`tagSession()`](https://code.claude.com/docs/en/agent-sdk/typescript#tagsession) en TypeScript. Utilisez-les pour organiser les sessions par tag ou leur donner des titres lisibles.

| Usage | TypeScript | Python |
| :- | :- | :- |
| Lister les sessions sur disque | `listSessions()` | `list_sessions()` |
| Lire les messages d'une session | `getSessionMessages()` | `get_session_messages()` |
| Consulter les infos d'une session | `getSessionInfo()` | `get_session_info()` |
| Renommer une session | `renameSession()` | `rename_session()` |
| Taguer une session | `tagSession()` | `tag_session()` |

## Ressources associées

* [How the agent loop works](https://code.claude.com/docs/en/agent-sdk/agent-loop) : comprendre les tours, les messages et l'accumulation du contexte au sein d'une session
* [File checkpointing](https://code.claude.com/docs/en/agent-sdk/file-checkpointing) : prendre des instantanés et annuler les modifications de fichiers faites par l'agent au sein d'une session
* [`ClaudeAgentOptions` Python](https://code.claude.com/docs/en/agent-sdk/python#claudeagentoptions) : référence complète des options de session pour Python
* [`Options` TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript#options) : référence complète des options de session pour TypeScript
