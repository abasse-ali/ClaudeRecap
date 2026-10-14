# Revenir en arrière sur les modifications de fichiers avec les checkpoints

> Source : https://code.claude.com/docs/en/agent-sdk/file-checkpointing · [← Index](../31-index-documentation-officielle.md)

*Suivre les modifications de fichiers pendant les sessions d'agent et restaurer les fichiers à n'importe quel état antérieur.*

Le checkpointing de fichiers (« file checkpointing ») suit les modifications de fichiers effectuées via les outils [Write](https://code.claude.com/docs/en/tools-reference#write-tool-behavior), [Edit](https://code.claude.com/docs/en/tools-reference#edit-tool-behavior) et [NotebookEdit](https://code.claude.com/docs/en/tools-reference#notebookedit-tool-behavior) pendant une session d'agent, ce qui permet de ramener (« rewind ») les fichiers à n'importe quel état antérieur. Pour voir directement un rewind restaurer un fichier, allez à l'[exemple interactif](#essayer).

> **Avertissement** : seules les modifications faites via les outils Write, Edit et NotebookEdit sont suivies. Les modifications faites via des commandes Bash (comme `echo > file.txt` ou `sed -i`) ne sont pas capturées par le système de checkpoints, pas plus que les modifications appliquées par un [sous-agent](https://code.claude.com/docs/en/agent-sdk/subagents) — à l'exception d'un [skill avec `context: fork`](https://code.claude.com/docs/en/skills#run-skills-in-a-subagent) exécuté au premier plan (foreground).

## Fonctionnement des checkpoints

Lorsque le checkpointing de fichiers est activé, le SDK crée des sauvegardes des fichiers avant de les modifier via les outils Write, Edit ou NotebookEdit. Les messages utilisateur du flux de réponse contiennent un UUID de checkpoint, utilisable comme point de restauration.

> **Note** : le rewind de fichiers restaure les fichiers sur disque à un état antérieur. Il ne rembobine **pas** la conversation elle-même. L'historique et le contexte de la conversation restent intacts après l'appel à `rewindFiles()` (TypeScript) ou `rewind_files()` (Python).

Lors d'un rewind vers un checkpoint, Claude Code :

- **supprime** les fichiers qu'il a créés ;
- **restaure** les fichiers qu'il a modifiés à leur contenu au moment du checkpoint.

Claude Code **ignore** (skip) :

- un chemin suivi qui est un lien symbolique (symlink), un lien physique (hard link) ou tout autre fichier non régulier ;
- un fichier suivi dont le répertoire parent ne se résout plus vers son emplacement au moment du checkpoint ;
- un fichier suivi dont il ne peut pas lire la sauvegarde de façon sûre.

[`RewindFilesResult`](https://code.claude.com/docs/en/agent-sdk/typescript#rewindfilesresult) compte chaque chemin ignoré dans son champ `skippedLinks`. Ce comportement d'ignorance exige Claude Code v2.1.216+ ; avant la v2.1.216, un rewind écrivait et supprimait à travers les liens situés aux chemins suivis.

## Mettre en œuvre le checkpointing

Pour utiliser le checkpointing de fichiers : activez-le dans vos options, capturez les UUID de checkpoint depuis le flux de réponse, puis appelez `rewindFiles()` (TypeScript) ou `rewind_files()` (Python) quand vous voulez restaurer.

Les exemples de cette section utilisent le prompt « Refactor the authentication module ». Exécutez-les dans un projet qui contient un module d'authentification, ou modifiez le prompt pour nommer des fichiers existant dans votre projet, afin de voir les fichiers changer puis le rewind les restaurer.

Cet exemple active le checkpointing, capture l'UUID de checkpoint et l'ID de session depuis le flux de réponse, puis reprend la session plus tard pour faire le rewind des fichiers. Les commentaires numérotés de l'exemple correspondent aux étapes qui suivent.

### Python

```python
import asyncio
from claude_agent_sdk import (
    ClaudeSDKClient,
    ClaudeAgentOptions,
    UserMessage,
    ResultMessage,
)


async def main():
    # Étape 1 : activer le checkpointing
    options = ClaudeAgentOptions(
        enable_file_checkpointing=True,
        permission_mode="acceptEdits",  # Accepte automatiquement les modifications de fichiers sans demander
        extra_args={
            "replay-user-messages": None
        },  # Requis pour recevoir les UUID de checkpoint dans le flux de réponse
    )

    checkpoint_id = None
    session_id = None

    # Exécuter la requête et capturer l'UUID de checkpoint et l'ID de session
    async with ClaudeSDKClient(options) as client:
        await client.query("Refactor the authentication module")

        # Étape 2 : capturer l'UUID de checkpoint depuis le premier message utilisateur
        async for message in client.receive_response():
            if isinstance(message, UserMessage) and message.uuid and not checkpoint_id:
                checkpoint_id = message.uuid
            if isinstance(message, ResultMessage) and not session_id:
                session_id = message.session_id

    # Étape 3 : plus tard, rewind en reprenant la session avec un prompt vide
    if checkpoint_id and session_id:
        async with ClaudeSDKClient(
            ClaudeAgentOptions(enable_file_checkpointing=True, resume=session_id)
        ) as client:
            await client.query("")  # Prompt vide pour ouvrir la connexion
            async for message in client.receive_response():
                await client.rewind_files(checkpoint_id)
                break
        print(f"Rewound to checkpoint: {checkpoint_id}")


asyncio.run(main())
```

### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

async function main() {
  // Étape 1 : activer le checkpointing
  const opts = {
    enableFileCheckpointing: true,
    permissionMode: "acceptEdits" as const, // Accepte automatiquement les modifications de fichiers sans demander
    extraArgs: { "replay-user-messages": null } // Requis pour recevoir les UUID de checkpoint dans le flux de réponse
  };

  const response = query({
    prompt: "Refactor the authentication module",
    options: opts
  });

  let checkpointId: string | undefined;
  let sessionId: string | undefined;

  // Étape 2 : capturer l'UUID de checkpoint depuis le premier message utilisateur
  try {
    for await (const message of response) {
      if (message.type === "user" && message.uuid && !checkpointId) {
        checkpointId = message.uuid;
      }
      if ("session_id" in message && !sessionId) {
        sessionId = message.session_id;
      }
    }
  } catch (error) {
    // Un query() à coup unique lève une exception après avoir émis un résultat
    // d'erreur. Si l'échec était un résultat d'erreur, sessionId et checkpointId
    // ont déjà été capturés par la boucle ci-dessus ; les échecs de connexion ou
    // de processus n'émettent aucun message de résultat.
    console.error(`Session ended with an error: ${error}`);
  }

  // Étape 3 : plus tard, rewind en reprenant la session avec un prompt vide
  if (checkpointId && sessionId) {
    const rewindQuery = query({
      prompt: "", // Prompt vide pour ouvrir la connexion
      options: { ...opts, resume: sessionId }
    });

    for await (const msg of rewindQuery) {
      await rewindQuery.rewindFiles(checkpointId);
      break;
    }
    console.log(`Rewound to checkpoint: ${checkpointId}`);
  }
}

main();
```

### Étape 1 — Activer le checkpointing

Configurez les options du SDK pour activer le checkpointing et recevoir les UUID de checkpoint :

| Option | Python | TypeScript | Description |
| - | - | - | - |
| Activer le checkpointing | `enable_file_checkpointing=True` | `enableFileCheckpointing: true` | Suit les modifications de fichiers pour permettre le rewind |
| Recevoir les UUID de checkpoint | `extra_args={"replay-user-messages": None}` | `extraArgs: { 'replay-user-messages': null }` | Requis pour obtenir les UUID des messages utilisateur dans le flux |

L'exemple principal de [Mettre en œuvre le checkpointing](#mettre-en-œuvre-le-checkpointing) règle aussi le mode de permission sur `acceptEdits`, qui approuve les modifications de fichiers de l'agent sans demande de confirmation. Pour plus d'informations, voir [Modes de permission](https://code.claude.com/docs/en/agent-sdk/permissions#permission-modes).

### Étape 2 — Capturer l'UUID de checkpoint et l'ID de session

Avec l'option `replay-user-messages` activée, chaque message utilisateur du flux de réponse possède un UUID qui sert de checkpoint.

Dans la plupart des cas, capturez l'UUID du **premier** message utilisateur (`message.uuid`) : un rewind vers celui-ci restaure les fichiers suivis à leur état d'origine. Pour stocker plusieurs checkpoints et revenir à des états intermédiaires, voir [Plusieurs points de restauration](#plusieurs-points-de-restauration).

Capturer l'ID de session (`message.session_id`) est facultatif : vous n'en avez besoin que si vous voulez faire le rewind plus tard, après la fin du flux. Si vous appelez `rewindFiles()` immédiatement pendant le traitement des messages (comme dans l'exemple de [Checkpoint avant les opérations risquées](#checkpoint-avant-les-opérations-risquées)), vous pouvez vous passer de l'ID de session.

### Étape 3 — Faire le rewind des fichiers

Pour faire le rewind après la fin du flux, reprenez la session avec un prompt vide et appelez `rewind_files()` (Python) ou `rewindFiles()` (TypeScript) avec votre UUID de checkpoint. Activez aussi le checkpointing dans les options de la session reprise, et appelez le rewind **depuis l'intérieur** de la boucle de réponse. Vous pouvez aussi faire le rewind pendant le flux : voir [Checkpoint avant les opérations risquées](#checkpoint-avant-les-opérations-risquées) pour ce schéma.

Si vous avez capturé l'ID de session et l'ID de checkpoint, vous pouvez aussi faire le rewind depuis la CLI. Cette commande nécessite l'exécutable `claude`, fourni par l'[installation de Claude Code](https://code.claude.com/docs/en/setup). Le SDK active le checkpointing pour vous, mais si vous lancez `claude -p` directement, vous devez définir la variable d'environnement `CLAUDE_CODE_ENABLE_SDK_FILE_CHECKPOINTING` :

```bash
CLAUDE_CODE_ENABLE_SDK_FILE_CHECKPOINTING=true claude -p --resume <session-id> --rewind-files <checkpoint-uuid>
```

Le flag `--rewind-files` n'apparaît pas dans la sortie de `claude --help`, mais la CLI l'accepte comme indiqué. Quand le rewind réussit, la commande affiche `Files rewound to state at message <checkpoint-uuid>` et se termine sans envoyer de prompt.

## Schémas courants

Ces schémas montrent différentes façons de capturer et d'utiliser les UUID de checkpoint selon le cas d'usage.

### Checkpoint avant les opérations risquées

Ce schéma ne conserve que l'UUID de checkpoint le plus récent, mis à jour avant chaque tour de l'agent. Si quelque chose se passe mal pendant le traitement, vous pouvez immédiatement revenir au dernier état sûr et sortir de la boucle.

Avant d'exécuter cet exemple, remplacez `your_revert_condition` (Python) ou `yourRevertCondition` (TypeScript) par votre propre vérification (détection d'erreur, échec de validation…) ; ce placeholder n'est pas défini dans l'exemple.

#### Python

```python
import asyncio
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions, UserMessage


async def main():
    options = ClaudeAgentOptions(
        enable_file_checkpointing=True,
        permission_mode="acceptEdits",
        extra_args={"replay-user-messages": None},
    )

    safe_checkpoint = None

    async with ClaudeSDKClient(options) as client:
        await client.query("Refactor the authentication module")

        async for message in client.receive_response():
            # Mettre à jour le checkpoint avant le début de chaque tour de l'agent
            # Ceci écrase le checkpoint précédent. On ne garde que le dernier
            if isinstance(message, UserMessage) and message.uuid:
                safe_checkpoint = message.uuid

            # Décider quand revenir en arrière selon votre propre logique
            # Par exemple : détection d'erreur, échec de validation ou saisie utilisateur
            if your_revert_condition and safe_checkpoint:
                await client.rewind_files(safe_checkpoint)
                # Sortir de la boucle après le rewind, les fichiers sont restaurés
                break


asyncio.run(main())
```

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

async function main() {
  const response = query({
    prompt: "Refactor the authentication module",
    options: {
      enableFileCheckpointing: true,
      permissionMode: "acceptEdits" as const,
      extraArgs: { "replay-user-messages": null }
    }
  });

  let safeCheckpoint: string | undefined;

  for await (const message of response) {
    // Mettre à jour le checkpoint avant le début de chaque tour de l'agent
    // Ceci écrase le checkpoint précédent. On ne garde que le dernier
    if (message.type === "user" && message.uuid) {
      safeCheckpoint = message.uuid;
    }

    // Décider quand revenir en arrière selon votre propre logique
    // Par exemple : détection d'erreur, échec de validation ou saisie utilisateur
    if (yourRevertCondition && safeCheckpoint) {
      await response.rewindFiles(safeCheckpoint);
      // Sortir de la boucle après le rewind, les fichiers sont restaurés
      break;
    }
  }
}

main();
```

### Plusieurs points de restauration

Si Claude effectue des modifications sur plusieurs tours, vous pouvez vouloir revenir à un point précis plutôt que tout au début. Par exemple, si Claude refactorise un fichier au tour 1 puis ajoute des tests au tour 2, vous pouvez vouloir garder la refactorisation mais annuler les tests.

Ce schéma stocke tous les UUID de checkpoint dans un tableau avec des métadonnées. Une fois la session terminée, vous pouvez revenir à n'importe quel checkpoint antérieur :

#### Python

```python
import asyncio
from dataclasses import dataclass
from datetime import datetime
from claude_agent_sdk import (
    ClaudeSDKClient,
    ClaudeAgentOptions,
    UserMessage,
    ResultMessage,
)


# Stocker les métadonnées des checkpoints pour un meilleur suivi
@dataclass
class Checkpoint:
    id: str
    description: str
    timestamp: datetime


async def main():
    options = ClaudeAgentOptions(
        enable_file_checkpointing=True,
        permission_mode="acceptEdits",
        extra_args={"replay-user-messages": None},
    )

    checkpoints = []
    session_id = None

    async with ClaudeSDKClient(options) as client:
        await client.query("Refactor the authentication module")

        async for message in client.receive_response():
            if isinstance(message, UserMessage) and message.uuid:
                checkpoints.append(
                    Checkpoint(
                        id=message.uuid,
                        description=f"After turn {len(checkpoints) + 1}",
                        timestamp=datetime.now(),
                    )
                )
            if isinstance(message, ResultMessage) and not session_id:
                session_id = message.session_id

    # Plus tard : rewind vers n'importe quel checkpoint en reprenant la session
    if checkpoints and session_id:
        target = checkpoints[0]  # Choisir n'importe quel checkpoint
        async with ClaudeSDKClient(
            ClaudeAgentOptions(enable_file_checkpointing=True, resume=session_id)
        ) as client:
            await client.query("")  # Prompt vide pour ouvrir la connexion
            async for message in client.receive_response():
                await client.rewind_files(target.id)
                break
        print(f"Rewound to: {target.description}")


asyncio.run(main())
```

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

// Stocker les métadonnées des checkpoints pour un meilleur suivi
interface Checkpoint {
  id: string;
  description: string;
  timestamp: Date;
}

async function main() {
  const opts = {
    enableFileCheckpointing: true,
    permissionMode: "acceptEdits" as const,
    extraArgs: { "replay-user-messages": null }
  };

  const response = query({
    prompt: "Refactor the authentication module",
    options: opts
  });

  const checkpoints: Checkpoint[] = [];
  let sessionId: string | undefined;

  try {
    for await (const message of response) {
      if (message.type === "user" && message.uuid) {
        checkpoints.push({
          id: message.uuid,
          description: `After turn ${checkpoints.length + 1}`,
          timestamp: new Date()
        });
      }
      if ("session_id" in message && !sessionId) {
        sessionId = message.session_id;
      }
    }
  } catch (error) {
    // Un query() à coup unique lève une exception après avoir émis un résultat
    // d'erreur. Si l'échec était un résultat d'erreur, sessionId et le tableau
    // checkpoints ont déjà été remplis par la boucle ci-dessus ; les échecs de
    // connexion ou de processus n'émettent aucun message de résultat.
    console.error(`Session ended with an error: ${error}`);
  }

  // Plus tard : rewind vers n'importe quel checkpoint en reprenant la session
  if (checkpoints.length > 0 && sessionId) {
    const target = checkpoints[0]; // Choisir n'importe quel checkpoint
    const rewindQuery = query({
      prompt: "", // Prompt vide pour ouvrir la connexion
      options: { ...opts, resume: sessionId }
    });

    for await (const msg of rewindQuery) {
      await rewindQuery.rewindFiles(target.id);
      break;
    }
    console.log(`Rewound to: ${target.description}`);
  }
}

main();
```

## Essayer

Cet exemple complet crée un petit fichier utilitaire, demande à l'agent d'y ajouter des commentaires de documentation, vous montre les modifications, puis vous demande si vous voulez faire un rewind.

Avant de commencer, assurez-vous d'avoir [installé le Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/quickstart).

### Étape 1 — Créer un fichier de test

Créez un nouveau fichier nommé `utils.py` (Python) ou `utils.ts` (TypeScript) et collez-y le code suivant :

#### utils.py

```python
def add(a, b):
    return a + b


def subtract(a, b):
    return a - b


def multiply(a, b):
    return a * b


def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
```

#### utils.ts

```typescript
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}

export function divide(a: number, b: number): number {
  if (b === 0) {
    throw new Error("Cannot divide by zero");
  }
  return a / b;
}
```

### Étape 2 — Créer l'exemple interactif

Créez un nouveau fichier nommé `try_checkpointing.py` (Python) ou `try_checkpointing.ts` (TypeScript) dans le même répertoire que votre fichier utilitaire, et collez-y le code suivant.

Ce script demande à Claude d'ajouter des commentaires de documentation à votre fichier utilitaire, puis vous propose de faire un rewind pour restaurer l'original.

#### try_checkpointing.py

```python
import asyncio
from claude_agent_sdk import (
    ClaudeSDKClient,
    ClaudeAgentOptions,
    UserMessage,
    ResultMessage,
)


async def main():
    # Configurer le SDK avec le checkpointing activé
    # - enable_file_checkpointing : suit les modifications de fichiers pour le rewind
    # - permission_mode : accepte automatiquement les modifications sans demander
    # - extra_args : requis pour recevoir les UUID des messages utilisateur dans le flux
    options = ClaudeAgentOptions(
        enable_file_checkpointing=True,
        permission_mode="acceptEdits",
        extra_args={"replay-user-messages": None},
    )

    checkpoint_id = None  # Stocke l'UUID du message utilisateur pour le rewind
    session_id = None  # Stocke l'ID de session pour la reprise

    print("Running agent to add doc comments to utils.py...\n")

    # Exécuter l'agent et capturer les données de checkpoint depuis le flux de réponse
    async with ClaudeSDKClient(options) as client:
        await client.query("Add doc comments to utils.py")

        async for message in client.receive_response():
            # Capturer l'UUID du premier message utilisateur : c'est notre point de restauration
            if isinstance(message, UserMessage) and message.uuid and not checkpoint_id:
                checkpoint_id = message.uuid
            # Capturer l'ID de session pour pouvoir reprendre plus tard
            if isinstance(message, ResultMessage):
                session_id = message.session_id

    print("Done! Open utils.py to see the added doc comments.\n")

    # Demander à l'utilisateur s'il veut annuler les modifications
    if checkpoint_id and session_id:
        response = input("Rewind to remove the doc comments? (y/n): ")

        if response.lower() == "y":
            # Reprendre la session avec un prompt vide, puis faire le rewind
            async with ClaudeSDKClient(
                ClaudeAgentOptions(enable_file_checkpointing=True, resume=session_id)
            ) as client:
                await client.query("")  # Un prompt vide ouvre la connexion
                async for message in client.receive_response():
                    await client.rewind_files(checkpoint_id)  # Restaurer les fichiers
                    break

            print(
                "\n✓ File restored! Open utils.py to verify the doc comments are gone."
            )
        else:
            print("\nKept the modified file.")


asyncio.run(main())
```

#### try_checkpointing.ts

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";
import * as readline from "readline";

async function main() {
  // Configurer le SDK avec le checkpointing activé
  // - enableFileCheckpointing : suit les modifications de fichiers pour le rewind
  // - permissionMode : accepte automatiquement les modifications sans demander
  // - extraArgs : requis pour recevoir les UUID des messages utilisateur dans le flux
  const opts = {
    enableFileCheckpointing: true,
    permissionMode: "acceptEdits" as const,
    extraArgs: { "replay-user-messages": null }
  };

  let sessionId: string | undefined; // Stocke l'ID de session pour la reprise
  let checkpointId: string | undefined; // Stocke l'UUID du message utilisateur pour le rewind

  console.log("Running agent to add doc comments to utils.ts...\n");

  // Exécuter l'agent et capturer les données de checkpoint depuis le flux de réponse
  const response = query({
    prompt: "Add doc comments to utils.ts",
    options: opts
  });

  try {
    for await (const message of response) {
      // Capturer l'UUID du premier message utilisateur : c'est notre point de restauration
      if (message.type === "user" && message.uuid && !checkpointId) {
        checkpointId = message.uuid;
      }
      // Capturer l'ID de session pour pouvoir reprendre plus tard
      if ("session_id" in message) {
        sessionId = message.session_id;
      }
    }
  } catch (error) {
    // Un query() à coup unique lève une exception après avoir émis un résultat
    // d'erreur. Si l'échec était un résultat d'erreur, checkpointId et sessionId
    // ont déjà été capturés par la boucle ci-dessus ; les échecs de connexion ou
    // de processus n'émettent aucun message de résultat.
    console.error(`Session ended with an error: ${error}`);
  }

  console.log("Done! Open utils.ts to see the added doc comments.\n");

  // Demander à l'utilisateur s'il veut annuler les modifications
  if (checkpointId && sessionId) {
    const rl = readline.createInterface({
      input: process.stdin,
      output: process.stdout
    });

    const answer = await new Promise<string>((resolve) => {
      rl.question("Rewind to remove the doc comments? (y/n): ", resolve);
    });
    rl.close();

    if (answer.toLowerCase() === "y") {
      // Reprendre la session avec un prompt vide, puis faire le rewind
      const rewindQuery = query({
        prompt: "", // Un prompt vide ouvre la connexion
        options: { ...opts, resume: sessionId }
      });

      for await (const msg of rewindQuery) {
        await rewindQuery.rewindFiles(checkpointId); // Restaurer les fichiers
        break;
      }

      console.log("\n✓ File restored! Open utils.ts to verify the doc comments are gone.");
    } else {
      console.log("\nKept the modified file.");
    }
  }
}

main();
```

### Étape 3 — Lancer l'exemple

Lancez le script depuis le même répertoire que votre fichier utilitaire.

> **Astuce** : ouvrez votre fichier utilitaire (`utils.py` ou `utils.ts`) dans votre IDE ou éditeur avant de lancer le script. Vous verrez le fichier se mettre à jour en temps réel pendant que l'agent ajoute les commentaires, puis revenir à l'original si vous choisissez le rewind.

#### Python

```bash
python try_checkpointing.py
```

#### TypeScript

```bash
npx tsx try_checkpointing.ts
```

Vous verrez l'agent ajouter les commentaires de documentation, puis une question vous demandant si vous voulez faire un rewind. Si vous répondez oui, le fichier est restauré à son état d'origine.

## Limitations

Le checkpointing de fichiers présente les limitations suivantes :

| Limitation | Description |
| - | - |
| Outils Write/Edit/NotebookEdit uniquement | Les modifications faites via des commandes Bash ne sont pas suivies |
| Modifications des sous-agents | Les modifications appliquées par un [sous-agent](https://code.claude.com/docs/en/agent-sdk/subagents) ne sont ni suivies ni restaurées, sauf pour un skill avec `context: fork` exécuté au premier plan ; utilisez git pour annuler les modifications non suivies |
| Même session | Les checkpoints sont liés à la session qui les a créés |
| Contenu des fichiers uniquement | La création, le déplacement ou la suppression de répertoires ne sont pas annulés par le rewind |
| Fichiers locaux | Les fichiers distants ou réseau ne sont pas suivis |

## Dépannage

### Options de checkpointing non reconnues

Si `enableFileCheckpointing` ou `rewindFiles()` n'est pas disponible, vous utilisez peut-être une ancienne version du SDK.

**Solution** : mettez à jour vers la dernière version du SDK :

- **Python** : `pip install --upgrade claude-agent-sdk`
- **TypeScript** : `npm install @anthropic-ai/claude-agent-sdk@latest`

### Les messages utilisateur n'ont pas d'UUID

Si `message.uuid` est `undefined` ou absent, vous ne recevez pas les UUID de checkpoint.

**Cause** : l'option `replay-user-messages` n'est pas définie.

**Solution** : ajoutez `extra_args={"replay-user-messages": None}` (Python) ou `extraArgs: { 'replay-user-messages': null }` (TypeScript) à vos options.

### Erreur « No file checkpoint found for this message »

Cette erreur survient lorsque les données de checkpoint n'existent pas pour l'UUID de message utilisateur indiqué.

**Causes fréquentes** :

- Le checkpointing de fichiers n'était pas activé sur la session d'origine (`enable_file_checkpointing` ou `enableFileCheckpointing` n'était pas réglé sur `true`).
- La session n'était pas correctement terminée avant la tentative de reprise et de rewind.

**Solution** : assurez-vous que `enable_file_checkpointing=True` (Python) ou `enableFileCheckpointing: true` (TypeScript) était défini sur la session d'origine, puis suivez le schéma des exemples : capturer l'UUID du premier message utilisateur, terminer complètement la session, puis la reprendre avec un prompt vide et appeler `rewindFiles()` une seule fois.

### Erreur « File rewinding is not enabled »

Cette erreur survient lorsque vous tentez un rewind non interactif sans que le checkpointing soit activé :

- en lançant `claude -p` « nu » avec `--rewind-files` ;
- ou en lançant une session SDK, y compris une session reprise, dont les options n'activent pas le checkpointing.

Le SDK définit en interne la variable d'environnement `CLAUDE_CODE_ENABLE_SDK_FILE_CHECKPOINTING` uniquement lorsque `enable_file_checkpointing` (Python) ou `enableFileCheckpointing` (TypeScript) est activé sur la session qui effectue le rewind ; la CLI seule ne la définit jamais.

**Solution** : pour la CLI seule, définissez la variable d'environnement en lançant la commande :

```bash
CLAUDE_CODE_ENABLE_SDK_FILE_CHECKPOINTING=true claude -p --resume <session-id> --rewind-files <checkpoint-uuid>
```

Pour le SDK, définissez `enable_file_checkpointing=True` (Python) ou `enableFileCheckpointing: true` (TypeScript) sur la session reprise, comme le font les exemples de cette page.

### Erreur « ProcessTransport is not ready for writing »

Cette erreur survient lorsque vous appelez `rewindFiles()` ou `rewind_files()` **après** avoir fini d'itérer sur la réponse. La connexion au processus CLI se ferme quand la boucle se termine.

**Solution** : reprenez la session avec un prompt vide, puis appelez le rewind sur la nouvelle requête. L'exemple principal de [Mettre en œuvre le checkpointing](#mettre-en-œuvre-le-checkpointing) montre l'appel de reprise + rewind dans les deux langages, au commentaire « Étape 3 ».

## Étapes suivantes

- **[Sessions](https://code.claude.com/docs/en/agent-sdk/sessions)** : apprendre à reprendre des sessions, ce qui est nécessaire pour faire un rewind après la fin du flux. Couvre les ID de session, la reprise de conversations et le fork de sessions.
- **[Permissions](https://code.claude.com/docs/en/agent-sdk/permissions)** : configurer les outils que Claude peut utiliser et la manière dont les modifications de fichiers sont approuvées. Utile pour mieux contrôler le moment où les modifications ont lieu.
- **[Référence du SDK TypeScript](https://code.claude.com/docs/en/agent-sdk/typescript)** : référence complète de l'API, y compris toutes les options de `query()` et la méthode `rewindFiles()`.
- **[Référence du SDK Python](https://code.claude.com/docs/en/agent-sdk/python)** : référence complète de l'API, y compris toutes les options de `ClaudeAgentOptions` et la méthode `rewind_files()`.
