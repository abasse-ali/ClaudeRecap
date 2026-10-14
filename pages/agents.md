# Exécuter des agents en parallèle

> Source : https://code.claude.com/docs/en/agents · [← Index](../31-index-documentation-officielle.md)

*Comparer les différentes manières dont Claude Code peut prendre en charge plusieurs tâches à la fois : sous-agents, vue des agents (agent view), équipes d'agents (agent teams), workflows dynamiques et projets.*

Claude Code propose cinq manières de travailler sur plusieurs tâches à la fois : les [sous-agents](https://code.claude.com/docs/en/sub-agents), la [vue des agents](https://code.claude.com/docs/en/agent-view), les [équipes d'agents](https://code.claude.com/docs/en/agent-teams), les [workflows dynamiques](https://code.claude.com/docs/en/workflows) et les [projets](https://code.claude.com/docs/en/claude-projects). Elles diffèrent par votre degré d'implication — du pilotage de chaque conversation par vous-même jusqu'à laisser Claude coordonner un groupe de travailleurs — et par le lieu d'exécution du travail : sur votre machine ou dans le cloud.

| Approche | Ce qu'elle vous apporte | À utiliser quand |
| :- | :- | :- |
| [Sous-agents](https://code.claude.com/docs/en/sub-agents) | Des travailleurs délégués à l'intérieur d'une seule session, qui réalisent une tâche annexe dans leur propre contexte et renvoient un résumé | Une tâche annexe inonderait votre conversation principale de résultats de recherche, de logs ou de contenus de fichiers que vous ne consulterez plus |
| [Vue des agents (agent view)](https://code.claude.com/docs/en/agent-view) | Un seul écran pour lancer et surveiller des sessions qui tournent en arrière-plan, ouvert avec `claude agents`. Aperçu de recherche (research preview) | Vous avez plusieurs tâches indépendantes et voulez les confier, vérifier leur état d'un coup d'œil, et n'intervenir que lorsque l'une d'elles a besoin de vous |
| [Équipes d'agents (agent teams)](https://code.claude.com/docs/en/agent-teams) | Plusieurs sessions coordonnées avec une liste de tâches partagée et une messagerie entre agents, gérées par un chef d'équipe (lead). Expérimental et désactivé par défaut | Vous voulez que Claude découpe un projet en morceaux, les attribue et garde les travailleurs synchronisés |
| [Projets](https://code.claude.com/docs/en/claude-projects) | Une conversation continue sur claude.ai/code ou dans l'application de bureau. Claude lance des sessions parallèles appelées fils (threads), dans le cloud ou, si vous le demandez, sur votre ordinateur via Remote Control ; il donne à chacune les instructions du projet et vous montre celles qui ont besoin de vous. Bêta publique sur les forfaits Pro et Max | Le travail couvre de nombreuses tâches sur des jours ou des semaines, doit continuer à tourner quand votre machine est éteinte, et vous préférez le décrire une seule fois plutôt que lancer et suivre chaque session |
| [Workflows dynamiques](https://code.claude.com/docs/en/workflows) | Un script qui exécute de nombreux sous-agents et recoupe leurs résultats, pour un travail trop volumineux pour être coordonné tour par tour ou qui demande plus d'une seule passe | Une tâche dépasse ce que gère une poignée de sous-agents, ou vous voulez des conclusions vérifiées les unes contre les autres : un audit de toute la base de code, une migration de 500 fichiers, une recherche recoupée, ou un plan rédigé sous plusieurs angles |

Dans toutes ces approches, les travailleurs sont des sessions Claude. Pour impliquer un autre outil, exposez-le à Claude sous forme de [serveur MCP](https://code.claude.com/docs/en/mcp).

Trois autres outils soutiennent ce travail sans être eux-mêmes une manière d'exécuter des agents :

* Les [worktrees](https://code.claude.com/docs/en/worktrees) donnent à chaque session un checkout git séparé, de sorte que des sessions parallèles modifient chacune leur propre copie des fichiers. Utilisez-les pour les sessions que vous lancez vous-même. Une session lancée depuis la vue des agents [se déplace dans son propre worktree avant de modifier des fichiers](https://code.claude.com/docs/en/agent-view#how-file-edits-are-isolated), et les sous-agents que vous lancez peuvent aussi en recevoir un chacun.
* La [messagerie entre sessions (cross-session messaging)](https://code.claude.com/docs/en/cross-session-messaging) permet à Claude de lister vos autres sessions Claude Code et de leur envoyer des messages, sur cette machine, sur une autre machine ou [dans le cloud](https://code.claude.com/docs/en/claude-code-on-the-web), afin que les sessions que vous lancez vous-même puissent s'échanger des résultats et des états d'avancement.
* [`/batch`](https://code.claude.com/docs/en/commands) est une [skill](https://code.claude.com/docs/en/skills) qui fait découper par Claude une grosse modification en 5 à 30 sous-agents isolés chacun dans un worktree. C'est un usage packagé des sous-agents et des worktrees, pas un style de coordination distinct.

Quelques autres fonctionnalités font tourner Claude sans que vous pilotiez chaque étape, mais elles résolvent un autre problème que la répartition du travail entre agents :

* Une [commande bash en arrière-plan](https://code.claude.com/docs/en/interactive-mode#background-bash-commands) exécute une seule commande shell sans bloquer la conversation. Elle ne lance pas d'agent.
* Un [sous-agent forké (forked subagent)](https://code.claude.com/docs/en/sub-agents#fork-the-current-conversation) est un sous-agent qui hérite de tout le contexte de votre conversation au lieu de repartir de zéro. C'est une manière de lancer un sous-agent, pas une surface distincte. Lancez-en un avec `/subtask`. Claude en lance aussi lui-même lorsque le [mode fork](https://code.claude.com/docs/en/sub-agents#turn-fork-mode-on-or-off) est activé. Pour copier toute la session dans une nouvelle [session en arrière-plan](https://code.claude.com/docs/en/agent-view#from-inside-a-session) qui tourne à côté d'elle, utilisez `/fork`. Lorsque la [vue des agents est désactivée](https://code.claude.com/docs/en/agent-view#turn-off-agent-view), la commande de sous-agent forké est `/fork` à la place, et `/subtask` n'est pas disponible.
* Une [routine](https://code.claude.com/docs/en/routines) exécute une session selon un planning dans le cloud, pas en parallèle sur votre machine.

> **Note** : exécuter plusieurs sessions ou sous-agents à la fois multiplie la consommation de tokens. Voir [Coûts](https://code.claude.com/docs/en/costs) pour les détails sur l'utilisation et les limites de débit (rate limits).

## Choisir une approche

La bonne approche dépend de qui coordonne le travail, de la nécessité pour les travailleurs de communiquer entre eux, et du fait qu'ils modifient ou non les mêmes fichiers :

* **Qui coordonne le travail ?**
  * Claude délègue et collecte les résultats à l'intérieur d'une seule conversation : [sous-agents](https://code.claude.com/docs/en/sub-agents)
  * Vous confiez des tâches indépendantes et revenez plus tard vérifier : [vue des agents](https://code.claude.com/docs/en/agent-view)
  * Claude planifie, attribue et supervise un groupe de travailleurs : [équipes d'agents](https://code.claude.com/docs/en/agent-teams), expérimental et désactivé par défaut
  * Un script porte le plan au lieu du jugement de Claude tour par tour : [workflows dynamiques](https://code.claude.com/docs/en/workflows). Voir [comment les workflows se comparent aux sous-agents et aux skills](https://code.claude.com/docs/en/workflows#when-to-use-a-workflow)
* **Les travailleurs doivent-ils se parler ?** Claude peut transmettre des résultats via la [messagerie entre sessions](https://code.claude.com/docs/en/cross-session-messaging) entre les sessions que vous lancez vous-même, y compris celles que vous lancez depuis la vue des agents. Les sous-agents renvoient leurs résultats à la conversation qui les a lancés, et les sessions de la vue des agents ne rapportent leurs résultats qu'à vous. Les coéquipiers (teammates) d'une équipe d'agents s'envoient des messages directement et, lorsqu'ils [disposent des outils Task](https://code.claude.com/docs/en/tools-reference#task-tool-availability), partagent une liste de tâches.
* **Les tâches touchent-elles les mêmes fichiers ?** Isolez le travail avec des [worktrees](https://code.claude.com/docs/en/worktrees). Les sous-agents et les sessions que vous lancez vous-même peuvent chacun utiliser un worktree séparé. Les équipes d'agents n'isolent pas les coéquipiers dans des worktrees : [répartissez le travail](https://code.claude.com/docs/en/agent-teams#avoid-file-conflicts) de sorte que chaque coéquipier possède un ensemble de fichiers différent.

## Suivre le travail en cours

La commande pour suivre le travail en cours dépend de l'approche utilisée :

* Pour les sessions en arrière-plan, `claude agents` ouvre la [vue des agents](https://code.claude.com/docs/en/agent-view) : un seul écran montrant chaque session, son état et celles qui attendent votre intervention.
* Pour les sous-agents de la session en cours, les sous-agents d'arrière-plan nommés apparaissent dans l'autocomplétion des @-mentions avec leur statut. La commande `/agents` affiche un avis indiquant l'emplacement des fichiers de sous-agents. Pour [créer et modifier des sous-agents personnalisés](https://code.claude.com/docs/en/sub-agents#configure-subagents), demandez à Claude ou modifiez directement les fichiers. Malgré la ressemblance des noms, `/agents` est distinct de `claude agents`.
* Pour tout ce qui tourne en arrière-plan de la session en cours, `/tasks` liste chaque élément et vous permet de le consulter, de vous y attacher ou de l'arrêter. La liste inclut aussi les sous-agents terminés.
* Pour les workflows dynamiques, `/workflows` liste les exécutions en cours et terminées, la phase où se trouve chacune et le nombre d'agents ayant terminé.

Pour une vue de bureau de toutes vos sessions, voir [les sessions parallèles dans l'application de bureau](https://code.claude.com/docs/en/desktop#work-in-parallel-with-sessions).

## En savoir plus

Chacun des guides ci-dessous couvre la mise en place et la configuration d'une approche :

* [Créer des sous-agents personnalisés](https://code.claude.com/docs/en/sub-agents) : définir des spécialistes réutilisables et contrôler les outils qu'ils peuvent utiliser.
* [Gérer les agents avec la vue des agents](https://code.claude.com/docs/en/agent-view) : lancer des sessions, surveiller leur état et s'y attacher quand l'une a besoin de vous.
* [Orchestrer des équipes d'agents](https://code.claude.com/docs/en/agent-teams) : mettre en place un chef d'équipe et des coéquipiers, attribuer des tâches et relire leur travail.
* [Orchestrer des workflows dynamiques](https://code.claude.com/docs/en/workflows) : exécuter un workflow fourni ou demander à Claude d'en écrire un qui lance de nombreux sous-agents et vérifie leurs conclusions les unes contre les autres.
* [Exécuter des sessions parallèles avec des worktrees](https://code.claude.com/docs/en/worktrees) : démarrer Claude dans un checkout isolé, contrôler ce qui y est copié et faire le ménage ensuite.
