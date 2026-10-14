# Obtenir une sortie structurée des agents

> Source : https://code.claude.com/docs/en/agent-sdk/structured-outputs · [← Index](../31-index-documentation-officielle.md)

*Renvoyer du JSON validé depuis des workflows d'agent grâce à JSON Schema, Zod ou Pydantic. Obtenir des données structurées et typées après une utilisation d'outils sur plusieurs tours.*

Les sorties structurées vous permettent de définir la forme exacte des données que l'agent doit vous renvoyer. L'agent peut utiliser tous les outils dont il a besoin pour accomplir la tâche, et vous obtenez quand même, à la fin, du JSON validé conforme à votre schéma. Définissez un [JSON Schema](https://json-schema.org/understanding-json-schema/about) pour la structure voulue : le SDK valide la sortie par rapport à ce schéma et **relance le modèle** (re-prompt) en cas de non-conformité. Si la validation n'aboutit pas dans la limite de tentatives, le résultat est une erreur au lieu de données structurées ; voir [Gestion des erreurs](#gestion-des-erreurs).

Pour une sûreté de typage complète, utilisez [Zod](#schémas-typés-avec-zod-et-pydantic) (TypeScript) ou [Pydantic](#schémas-typés-avec-zod-et-pydantic) (Python) pour définir votre schéma et récupérer des objets fortement typés.

## Pourquoi des sorties structurées ?

Par défaut, les agents renvoient du texte libre, ce qui convient pour un chat mais pas quand vous devez exploiter la sortie par programme. Les sorties structurées vous fournissent des données typées que vous pouvez passer directement à la logique de votre application, à votre base de données ou à vos composants d'interface.

Prenons une application de recettes où un agent cherche sur le web et rapporte des recettes. Sans sorties structurées, vous obtenez du texte libre qu'il vous faudrait analyser vous-même. Avec les sorties structurées, vous définissez la forme voulue et obtenez des données typées utilisables directement dans votre application.

### Sans sorties structurées

```text
Here's a classic chocolate chip cookie recipe!

**Chocolate Chip Cookies**
Prep time: 15 minutes | Cook time: 10 minutes

Ingredients:
- 2 1/4 cups all-purpose flour
- 1 cup butter, softened
...
```

Pour exploiter cela dans votre application, il faudrait extraire le titre, convertir « 15 minutes » en nombre, séparer les ingrédients des instructions et gérer un formatage incohérent d'une réponse à l'autre.

### Avec sorties structurées

```jsonc
{
  "name": "Chocolate Chip Cookies",
  "prep_time_minutes": 15,
  "cook_time_minutes": 10,
  "ingredients": [
    { "item": "all-purpose flour", "amount": 2.25, "unit": "cups" },
    { "item": "butter, softened", "amount": 1, "unit": "cup" }
    // ...
  ],
  "steps": ["Preheat oven to 375°F", "Cream butter and sugar" /* ... */]
}
```

Des données typées, utilisables directement dans votre interface.

## Démarrage rapide

Pour utiliser les sorties structurées, définissez un [JSON Schema](https://json-schema.org/understanding-json-schema/about) décrivant la forme des données voulues, puis passez-le à `query()` via l'option `outputFormat` (TypeScript) ou `output_format` (Python). Quand l'agent a terminé, le message de résultat comporte un champ `structured_output` contenant des données validées conformes à votre schéma.

Avant d'exécuter les exemples de cette page, installez le Claude Agent SDK en suivant le [quickstart](https://code.claude.com/docs/en/agent-sdk/quickstart#setup). L'exemple ci-dessous demande à l'agent de se renseigner sur Anthropic et de renvoyer, sous forme de sortie structurée, le nom de l'entreprise, l'année de fondation et le siège.

### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

// Définir la forme des données attendues
const schema = {
  type: "object",
  properties: {
    company_name: { type: "string" },
    founded_year: { type: "number" },
    headquarters: { type: "string" }
  },
  required: ["company_name"]
};

try {
  for await (const message of query({
    prompt: "Research Anthropic and provide key company information",
    options: {
      outputFormat: {
        type: "json_schema",
        schema: schema
      }
    }
  })) {
    // Le message de résultat contient structured_output avec les données validées
    if (message.type === "result" && message.subtype === "success" && message.structured_output) {
      console.log(message.structured_output);
      // { company_name: "Anthropic", founded_year: 2021, headquarters: "San Francisco, CA" }
    }
  }
} catch (error) {
  // Un query() ponctuel lève une erreur après avoir produit un résultat d'erreur, comme
  // error_max_structured_output_retries ; voir la section Gestion des erreurs.
  console.error(`Session ended with an error: ${error}`);
}
```

### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

# Définir la forme des données attendues
schema = {
    "type": "object",
    "properties": {
        "company_name": {"type": "string"},
        "founded_year": {"type": "number"},
        "headquarters": {"type": "string"},
    },
    "required": ["company_name"],
}


async def main():
    try:
        async for message in query(
            prompt="Research Anthropic and provide key company information",
            options=ClaudeAgentOptions(
                output_format={"type": "json_schema", "schema": schema}
            ),
        ):
            # Le message de résultat contient structured_output avec les données validées
            if isinstance(message, ResultMessage) and message.structured_output:
                print(message.structured_output)
                # {'company_name': 'Anthropic', 'founded_year': 2021, 'headquarters': 'San Francisco, CA'}
    except Exception as error:
        # Un query() ponctuel lève une erreur après avoir produit un résultat d'erreur, comme
        # error_max_structured_output_retries ; voir la section Gestion des erreurs.
        print(f"Session ended with an error: {error}")


asyncio.run(main())
```

## Schémas typés avec Zod et Pydantic

Plutôt que d'écrire le JSON Schema à la main, vous pouvez utiliser [Zod](https://zod.dev/) (TypeScript) ou [Pydantic](https://docs.pydantic.dev/latest/) (Python) pour définir votre schéma. Ces bibliothèques génèrent le JSON Schema à votre place et vous permettent d'analyser la réponse en un objet entièrement typé, utilisable dans tout votre code avec autocomplétion et vérification de types.

L'exemple ci-dessous définit un schéma de plan d'implémentation d'une fonctionnalité, avec un résumé, une liste d'étapes (chacune avec un niveau de complexité) et les risques potentiels. L'agent planifie la fonctionnalité et renvoie un objet `FeaturePlan` typé. Vous pouvez ensuite accéder à des propriétés comme `plan.summary` et parcourir `plan.steps` avec une sûreté de typage complète.

Le SDK valide les schémas avec **JSON Schema draft-07** : les schémas qui déclarent une version plus récente sont rejetés. Zod cible par défaut le draft 2020-12 ; passez donc `target: "draft-7"` lors de la conversion de votre schéma.

### TypeScript

```typescript
import { z } from "zod";
import { query } from "@anthropic-ai/claude-agent-sdk";

// Définir le schéma avec Zod
const FeaturePlan = z.object({
  feature_name: z.string(),
  summary: z.string(),
  steps: z.array(
    z.object({
      step_number: z.number(),
      description: z.string(),
      estimated_complexity: z.enum(["low", "medium", "high"])
    })
  ),
  risks: z.array(z.string())
});

type FeaturePlan = z.infer<typeof FeaturePlan>;

// Convertir en JSON Schema avec la cible draft-07 attendue par le SDK
const schema = z.toJSONSchema(FeaturePlan, { target: "draft-7" });

// Utiliser dans la requête
try {
  for await (const message of query({
    prompt:
      "Plan how to add dark mode support to a React app. Break it into implementation steps.",
    options: {
      outputFormat: {
        type: "json_schema",
        schema: schema
      }
    }
  })) {
    if (message.type === "result" && message.subtype === "success" && message.structured_output) {
      // Valider et obtenir un résultat entièrement typé
      const parsed = FeaturePlan.safeParse(message.structured_output);
      if (parsed.success) {
        const plan: FeaturePlan = parsed.data;
        console.log(`Feature: ${plan.feature_name}`);
        console.log(`Summary: ${plan.summary}`);
        plan.steps.forEach((step) => {
          console.log(`${step.step_number}. [${step.estimated_complexity}] ${step.description}`);
        });
      }
    }
  }
} catch (error) {
  // Un query() ponctuel lève une erreur après avoir produit un résultat d'erreur, comme
  // error_max_structured_output_retries ; voir la section Gestion des erreurs.
  console.error(`Session ended with an error: ${error}`);
}
```

### Python

```python
import asyncio
from pydantic import BaseModel
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage


class Step(BaseModel):
    step_number: int
    description: str
    estimated_complexity: str  # 'low', 'medium', 'high'


class FeaturePlan(BaseModel):
    feature_name: str
    summary: str
    steps: list[Step]
    risks: list[str]


async def main():
    try:
        async for message in query(
            prompt="Plan how to add dark mode support to a React app. Break it into implementation steps.",
            options=ClaudeAgentOptions(
                output_format={
                    "type": "json_schema",
                    "schema": FeaturePlan.model_json_schema(),
                }
            ),
        ):
            if isinstance(message, ResultMessage) and message.structured_output:
                # Valider et obtenir un résultat entièrement typé
                plan = FeaturePlan.model_validate(message.structured_output)
                print(f"Feature: {plan.feature_name}")
                print(f"Summary: {plan.summary}")
                for step in plan.steps:
                    print(
                        f"{step.step_number}. [{step.estimated_complexity}] {step.description}"
                    )
    except Exception as error:
        # Un query() ponctuel lève une erreur après avoir produit un résultat d'erreur, comme
        # error_max_structured_output_retries ; voir la section Gestion des erreurs.
        print(f"Session ended with an error: {error}")


asyncio.run(main())
```

## Configuration du format de sortie

L'option `outputFormat` (TypeScript) ou `output_format` (Python) accepte un objet comportant :

* `type` : à régler sur `"json_schema"` pour les sorties structurées ;
* `schema` : un objet [JSON Schema](https://json-schema.org/understanding-json-schema/about) définissant la structure de votre sortie. Vous pouvez le générer à partir d'un schéma Zod avec `z.toJSONSchema(schema, { target: "draft-7" })` ou d'un modèle Pydantic avec `.model_json_schema()`.

Le SDK prend en charge les fonctionnalités standard de JSON Schema, notamment tous les types de base (object, array, string, number, boolean, null), `enum`, `const`, `required`, les objets imbriqués et les définitions `$ref`. Pour la liste complète des fonctionnalités prises en charge et des limitations, voir [Limitations de JSON Schema](https://platform.claude.com/docs/en/build-with-claude/structured-outputs#json-schema-limitations).

Un schéma qui n'est pas du JSON Schema valide fait **échouer l'exécution dès le démarrage**, avec une erreur nommant le problème. Avant la v2.1.205, un schéma invalide était ignoré silencieusement et l'agent renvoyait du texte non structuré.

Le mot-clé `format`, par exemple `"format": "email"`, est accepté comme simple annotation et n'est **pas** appliqué par le validateur du SDK. Avant la v2.1.205, tout schéma contenant `format` était considéré comme invalide.

## Exemple : agent de suivi des TODO

Cet exemple montre comment les sorties structurées fonctionnent avec une utilisation d'outils en plusieurs étapes. L'agent doit trouver les commentaires TODO dans la base de code, puis récupérer les informations de `git blame` pour chacun. Il décide de façon autonome des outils à utiliser (Grep pour chercher, Bash pour exécuter les commandes git) et combine les résultats en une seule réponse structurée.

Le schéma comporte des champs facultatifs (`author` et `date`), car les informations de `git blame` peuvent ne pas être disponibles pour tous les fichiers. L'agent remplit ce qu'il trouve et omet le reste.

### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

// Définir la structure d'extraction des TODO
const todoSchema = {
  type: "object",
  properties: {
    todos: {
      type: "array",
      items: {
        type: "object",
        properties: {
          text: { type: "string" },
          file: { type: "string" },
          line: { type: "number" },
          author: { type: "string" },
          date: { type: "string" }
        },
        required: ["text", "file", "line"]
      }
    },
    total_count: { type: "number" }
  },
  required: ["todos", "total_count"]
};

// L'agent utilise Grep pour trouver les TODO, Bash pour obtenir les infos git blame
try {
  for await (const message of query({
    prompt: "Find all TODO comments in this codebase and identify who added them",
    options: {
      outputFormat: {
        type: "json_schema",
        schema: todoSchema
      }
    }
  })) {
    if (message.type === "result" && message.subtype === "success" && message.structured_output) {
      const data = message.structured_output as { total_count: number; todos: Array<{ file: string; line: number; text: string; author?: string; date?: string }> };
      console.log(`Found ${data.total_count} TODOs`);
      data.todos.forEach((todo) => {
        console.log(`${todo.file}:${todo.line} - ${todo.text}`);
        if (todo.author) {
          console.log(`  Added by ${todo.author} on ${todo.date}`);
        }
      });
    }
  }
} catch (error) {
  // Un query() ponctuel lève une erreur après avoir produit un résultat d'erreur, comme
  // error_max_structured_output_retries ; voir la section Gestion des erreurs.
  console.error(`Session ended with an error: ${error}`);
}
```

### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

# Définir la structure d'extraction des TODO
todo_schema = {
    "type": "object",
    "properties": {
        "todos": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "text": {"type": "string"},
                    "file": {"type": "string"},
                    "line": {"type": "number"},
                    "author": {"type": "string"},
                    "date": {"type": "string"},
                },
                "required": ["text", "file", "line"],
            },
        },
        "total_count": {"type": "number"},
    },
    "required": ["todos", "total_count"],
}


async def main():
    # L'agent utilise Grep pour trouver les TODO, Bash pour obtenir les infos git blame
    try:
        async for message in query(
            prompt="Find all TODO comments in this codebase and identify who added them",
            options=ClaudeAgentOptions(
                output_format={"type": "json_schema", "schema": todo_schema}
            ),
        ):
            if isinstance(message, ResultMessage) and message.structured_output:
                data = message.structured_output
                print(f"Found {data['total_count']} TODOs")
                for todo in data["todos"]:
                    print(f"{todo['file']}:{todo['line']} - {todo['text']}")
                    if "author" in todo:
                        print(f"  Added by {todo['author']} on {todo['date']}")
    except Exception as error:
        # Un query() ponctuel lève une erreur après avoir produit un résultat d'erreur, comme
        # error_max_structured_output_retries ; voir la section Gestion des erreurs.
        print(f"Session ended with an error: {error}")


asyncio.run(main())
```

## Gestion des erreurs

La génération d'une sortie structurée peut échouer quand l'agent ne parvient pas à produire un JSON valide conforme à votre schéma. Cela arrive typiquement quand le schéma est trop complexe pour la tâche, quand la tâche elle-même est ambiguë, ou quand l'agent atteint sa limite de tentatives en essayant de corriger les erreurs de validation. Cela peut aussi se produire **sans aucun échec de validation** : un [repli de modèle (model fallback)](https://code.claude.com/docs/en/model-config#automatic-model-fallback) peut rétracter en cours de stream une sortie déjà terminée, et si aucune nouvelle tentative ne la remplace, l'exécution se termine avec la même erreur. Consultez la liste `errors` du message de résultat d'erreur pour distinguer ces deux causes avant de déboguer votre schéma.

Quand une erreur survient, le message de résultat porte un `subtype` qui indique ce qui s'est mal passé :

| Sous-type | Signification |
| - | - |
| `success` | La sortie a été générée et validée avec succès |
| `error_max_structured_output_retries` | Aucune sortie valide ne subsiste après plusieurs tentatives (échecs de validation, ou rétractation due à un repli de modèle sans nouvelle tentative réussie) |

Un résultat peut aussi se terminer avec le sous-type `success` mais **sans** valeur `structured_output`, par exemple quand l'exécution se termine sans que l'agent ait produit de sortie structurée. Traitez ce cas comme un échec lui aussi. L'entrée de dépannage [structured_output is None but the result says success](https://code.claude.com/docs/en/agent-sdk/troubleshooting#structured_output-is-none-but-the-result-says-success) couvre ce cas. L'exemple ci-dessous ne considère un résultat comme réussi que si le `subtype` vaut `success` **et** que `structured_output` est présent, et traite tout autre résultat comme un échec.

### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

const contactSchema = {
  type: "object",
  properties: {
    name: { type: "string" },
    email: { type: "string" }
  },
  required: ["name"]
};

try {
  for await (const msg of query({
    prompt: "Extract contact info from the document",
    options: {
      outputFormat: {
        type: "json_schema",
        schema: contactSchema
      }
    }
  })) {
    if (msg.type === "result") {
      if (msg.subtype === "success" && msg.structured_output) {
        // Utiliser la sortie validée
        console.log(msg.structured_output);
      } else if (msg.subtype === "error_max_structured_output_retries") {
        console.error("Could not produce valid output");
      } else {
        console.error("Run ended without a structured output");
      }
    }
  }
} catch (error) {
  // Un query() ponctuel lève une erreur après avoir produit un résultat d'erreur. Si
  // l'échec était un résultat d'erreur, les branches de sous-type ci-dessus se sont
  // déjà exécutées ; les échecs de connexion ou de processus ne produisent aucun message de résultat.
  console.log(`Session ended with an error: ${error}`);
}
```

### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

contact_schema = {
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "email": {"type": "string"},
    },
    "required": ["name"],
}


async def main():
    try:
        async for message in query(
            prompt="Extract contact info from the document",
            options=ClaudeAgentOptions(
                output_format={"type": "json_schema", "schema": contact_schema}
            ),
        ):
            if isinstance(message, ResultMessage):
                if message.subtype == "success" and message.structured_output:
                    # Utiliser la sortie validée
                    print(message.structured_output)
                elif message.subtype == "error_max_structured_output_retries":
                    print("Could not produce valid output")
                else:
                    print("Run ended without a structured output")
    except Exception as error:
        # Un query() ponctuel lève une erreur après avoir produit un résultat d'erreur. Si
        # l'échec était un résultat d'erreur, les branches de sous-type ci-dessus se sont
        # déjà exécutées ; les échecs de connexion ou de processus ne produisent aucun message de résultat.
        print(f"Session ended with an error: {error}")


asyncio.run(main())
```

**Conseils pour éviter les erreurs :**

* **Gardez des schémas ciblés.** Les schémas profondément imbriqués avec de nombreux champs obligatoires sont plus difficiles à satisfaire. Commencez simple et ajoutez de la complexité au besoin.
* **Adaptez le schéma à la tâche.** Si la tâche risque de ne pas fournir toutes les informations exigées par votre schéma, rendez ces champs facultatifs.
* **Rédigez des prompts clairs.** Des prompts ambigus compliquent, pour l'agent, la détermination de la sortie à produire.

## Ressources associées

* [Documentation JSON Schema](https://json-schema.org/) : apprendre la syntaxe JSON Schema pour définir des schémas complexes avec objets imbriqués, tableaux, enums et contraintes de validation ;
* [Structured Outputs de l'API](https://platform.claude.com/docs/en/build-with-claude/structured-outputs) : utiliser les sorties structurées directement avec l'API Claude, pour des requêtes à un seul tour sans utilisation d'outils ;
* [Outils personnalisés](https://code.claude.com/docs/en/agent-sdk/custom-tools) : donner à votre agent des outils personnalisés à appeler pendant l'exécution, avant de renvoyer la sortie structurée.
