# Faire travailler Claude jusqu'à l'atteinte d'un objectif

> Source : https://code.claude.com/docs/en/goal · [← Index](../31-index-documentation-officielle.md)

*Définissez une condition d'achèvement avec /goal : Claude continue de travailler jusqu'à ce qu'elle soit remplie, qu'un modèle la juge impossible, ou qu'une erreur que vous devez corriger efface l'objectif.*

La commande `/goal` définit une condition d'achèvement, et Claude continue de travailler vers elle sans que vous ayez à relancer chaque étape. Après chaque tour, un modèle vérifie si la condition est remplie. Si le modèle juge qu'elle ne l'est pas encore, Claude démarre un nouveau tour au lieu de vous rendre la main. L'objectif est effacé automatiquement dès que la condition est remplie, si le modèle juge la condition impossible à satisfaire, ou si un tour échoue sur [une erreur que vous devez corriger](#les-erreurs-que-vous-devez-corriger-effacent-lobjectif).

Utilisez un objectif pour un travail conséquent avec un état final vérifiable :

* migrer un module vers une nouvelle API jusqu'à ce que chaque site d'appel compile et que les tests passent ;
* implémenter un document de conception jusqu'à ce que tous les critères d'acceptation soient satisfaits ;
* découper un gros fichier en modules ciblés jusqu'à ce que chacun soit sous un budget de taille ;
* traiter un backlog d'issues étiquetées jusqu'à ce que la file soit vide.

## Comparer les façons de maintenir une session en marche

Trois approches maintiennent la session courante en marche entre les prompts. Choisissez selon ce qui doit démarrer le tour suivant :

| Approche | Le tour suivant démarre quand | S'arrête quand |
| :- | :- | :- |
| `/goal` | Le tour précédent se termine, ou, dans une session interactive, quand un [point d'étape en inactivité](#le-travail-en-arrière-plan-diffère-lévaluation) ou une [nouvelle tentative automatique](#les-autres-erreurs-relancent-ou-mettent-en-pause-lobjectif) arrive à échéance | Un modèle confirme que la condition est remplie ou la juge impossible, ou un tour échoue sur [une erreur que vous devez corriger](#les-erreurs-que-vous-devez-corriger-effacent-lobjectif), ou vous lancez [`/goal clear`](#effacer-un-objectif) |
| [`/loop`](https://code.claude.com/docs/en/scheduled-tasks#run-a-prompt-repeatedly-with-%2Floop) | Un intervalle de temps s'est écoulé | Vous l'arrêtez, ou Claude décide que le travail est terminé |
| [Hook Stop](https://code.claude.com/docs/en/hooks-guide#prompt-based-hooks) | Le tour précédent se termine | Votre propre script ou prompt décide |

`/goal` et un hook Stop se déclenchent tous deux après chaque tour. `/goal` est un raccourci limité à la session : vous tapez une condition et elle n'est active que pour la session courante. Un hook Stop vit dans votre fichier de réglages, s'applique à toutes les sessions de sa portée, et peut exécuter un script pour des vérifications déterministes ou un prompt pour des vérifications évaluées par un modèle.

Le [mode auto](https://code.claude.com/docs/en/auto-mode-config), à lui seul, approuve les appels d'outils au sein d'un tour mais n'en démarre pas de nouveau. Claude s'arrête lorsqu'il juge le travail terminé. `/goal` ajoute un évaluateur séparé qui vérifie votre condition après chaque tour : l'achèvement est donc décidé par un modèle neuf plutôt que par celui qui fait le travail. Les deux sont complémentaires : le mode auto supprime les demandes par outil, et `/goal` supprime les relances par tour.

> **Astuce** : les approches ci-dessus maintiennent la session courante en marche. Vous pouvez aussi planifier du travail qui s'exécute indépendamment de toute session ouverte, comme des tests nocturnes ou un tri matinal. Voir les [options de planification](https://code.claude.com/docs/en/scheduled-tasks#compare-scheduling-options) pour les routines cloud et les tâches planifiées de l'application desktop.

## Utiliser `/goal`

Un seul objectif peut être actif par session. La même commande le définit, l'affiche ou l'efface selon l'argument.

### Définir un objectif

Lancez `/goal` suivi de la condition à satisfaire. Si un objectif est déjà actif, le nouveau le remplace.

```text
/goal all tests in test/auth pass and the lint step is clean
```

(« tous les tests de test/auth passent et l'étape de lint est propre »)

Définir un objectif démarre immédiatement un tour, avec la condition elle-même comme consigne. Vous n'avez pas besoin d'envoyer un prompt séparé. Tant que l'objectif est actif, un indicateur `◎ /goal active` affiche depuis combien de temps il tourne.

Un objectif ne change pas votre mode de permission. Pour laisser les tours de l'objectif s'exécuter sans surveillance, lancez `/goal` en [mode auto](https://code.claude.com/docs/en/auto-mode-config). En [mode Manual](https://code.claude.com/docs/en/permission-modes), Claude demande toujours avant les appels d'outils que vos réglages n'autorisent pas déjà, comme la commande de test ci-dessus.

Tant que l'objectif est actif, la transcription affiche chaque verdict renvoyé par l'évaluateur, et vous pouvez appuyer sur `Ctrl+O` pour voir la raison qui le motive. La vue de statut affiche aussi la raison la plus récente, pour que vous voyiez vers quoi Claude travaille ensuite.

### Rédiger une condition efficace

L'[évaluateur](#fonctionnement-de-lévaluation) juge votre condition d'après ce que Claude a fait apparaître dans la conversation. Il n'exécute pas de commandes et ne lit pas de fichiers par lui-même : rédigez donc la condition comme quelque chose que la propre sortie de Claude peut démontrer. « All tests in `test/auth` pass » fonctionne parce que Claude exécute les tests et que le résultat apparaît dans la transcription, où l'évaluateur peut le lire.

Une condition qui tient sur de nombreux tours comporte généralement :

* **Un état final mesurable** : un résultat de tests, un code de sortie de build, un nombre de fichiers, une file vide.
* **Une vérification énoncée** : comment Claude doit le prouver, par exemple « `npm test` exits 0 » ou « `git status` is clean ».
* **Les contraintes qui comptent** : tout ce qui ne doit pas changer en chemin, par exemple « no other test file is modified » (aucun autre fichier de test n'est modifié).

La condition peut faire jusqu'à **4 000 caractères**.

Pour borner la durée d'un objectif, incluez une clause de tours ou de temps dans la condition, par exemple `or stop after 20 turns`. Claude rend compte de sa progression par rapport à cette clause à chaque tour, et l'évaluateur en juge d'après la conversation.

### Consulter le statut

Lancez `/goal` sans argument pour voir l'état courant.

```text
/goal
```

Si un objectif est actif, le statut affiche :

* la condition ;
* depuis combien de temps il tourne ;
* combien de tours ont été évalués ;
* la consommation de tokens actuelle ;
* la raison la plus récente de l'évaluateur.

Le nombre de tours et la raison la plus récente apparaissent après la première évaluation.

Si aucun objectif n'est actif mais qu'un objectif a été atteint plus tôt dans la session, le statut affiche la condition atteinte avec sa durée, son nombre de tours et sa consommation de tokens.

### Effacer un objectif

Lancez `/goal clear` pour retirer un objectif actif avant qu'il ne soit résolu.

```text
/goal clear
```

Claude affiche `Goal cleared:` suivi de la condition pour confirmer, ou `No goal set` si rien n'était actif.

`stop`, `off`, `reset`, `none` et `cancel` sont acceptés comme alias de `clear`. Lancer `/clear` pour démarrer une nouvelle conversation retire aussi tout objectif actif.

### Reprendre une session avec un objectif actif

Lorsque vous reprenez une session, Claude Code restaure un objectif qui était encore actif à la fin de la session. Il le restaure par toutes les voies de reprise : `--continue`, `--resume` avec un ID de session, un nom ou un [chemin de fichier de transcription](https://code.claude.com/docs/en/sessions#resume-a-session), et le [sélecteur de sessions](https://code.claude.com/docs/en/sessions#use-the-session-picker). Avant la v2.1.239, Claude Code restaurait l'objectif par toutes les voies sauf le sélecteur `claude --resume`.

Claude Code reporte la condition mais réinitialise le nombre de tours, le chronomètre et la base de consommation de tokens. Il ne restaure pas un objectif déjà atteint ou effacé.

### Exécution non interactive

`/goal` fonctionne en [mode non interactif](https://code.claude.com/docs/en/headless), dans l'[application desktop](https://code.claude.com/docs/en/desktop) et via [Remote Control](https://code.claude.com/docs/en/remote-control). Définir un objectif avec `-p` exécute la boucle jusqu'au bout en une seule invocation :

```bash
claude -p "/goal CHANGELOG.md has an entry for every PR merged this week"
```

Avec la sortie texte par défaut, la réponse finale de Claude ne s'affiche qu'à la fin de la boucle : un objectif qui dure de nombreux tours peut donc sembler bloqué. Ajoutez `--output-format stream-json --verbose` pour émettre chaque message au fil de la boucle.

Interrompez le processus avec `Ctrl+C` pour arrêter un objectif non interactif avant sa résolution.

## Fonctionnement de l'évaluation

`/goal` est une surcouche d'un [hook Stop à base de prompt](https://code.claude.com/docs/en/hooks#prompt-based-hooks) limité à la session. Chaque fois que Claude termine un tour, Claude Code envoie la condition et la conversation jusque-là au [petit modèle rapide](https://code.claude.com/docs/en/model-config) configuré. Le modèle renvoie l'un de trois verdicts, chacun accompagné d'une courte raison :

* **Not yet met** (pas encore remplie) : Claude continue de travailler et prend la raison comme indication pour le tour suivant.
* **Met** (remplie) : Claude Code efface l'objectif et enregistre une entrée « atteint » dans la transcription.
* **Impossible** : l'évaluateur a jugé que la condition ne pourra jamais être satisfaite. Claude Code efface l'objectif et enregistre une entrée « échoué » dans la transcription avec la raison. Vous n'avez pas besoin de l'effacer vous-même.

Si Claude continue de répondre à l'évaluateur sans progresser (aucune utilisation d'outil pendant plusieurs tours d'affilée), Claude Code arrête la boucle, affiche un avertissement et vous rend la main, l'objectif restant défini. L'évaluation reprend après votre prochain prompt. Le [guide des hooks](https://code.claude.com/docs/en/hooks-guide#stop-hook-hits-the-block-cap) explique le mécanisme sous-jacent.

### Quand un tour échoue

Quand un tour échoue, Claude Code efface l'objectif si l'erreur est de celles que vous devez corriger. Après toute autre erreur, l'objectif reste défini.

#### Les erreurs que vous devez corriger effacent l'objectif

Si un tour échoue sur une erreur qui ne disparaîtra pas tant que vous ne l'aurez pas corrigée, Claude Code efface l'objectif et affiche un avertissement nommant la cause. L'avertissement commence par `Goal cleared after an unrecoverable error` et se termine par `Run /goal again to continue`. Corrigez la cause, puis [redéfinissez l'objectif](#définir-un-objectif) avec `/goal <condition>`. Quatre types d'échec effacent l'objectif :

* Un échec d'authentification, lorsque Claude Code gère ses propres identifiants. Quand un hôte les gère pour vous, comme l'application desktop ou une [session cloud](https://code.claude.com/docs/en/claude-code-on-the-web), Claude Code laisse l'objectif actif, car l'hôte rétablit l'accès de lui-même.
* Un solde de crédits épuisé.
* Un dépassement de contexte que l'[auto-compaction](https://code.claude.com/docs/en/model-config#set-the-auto-compact-window) n'a pas pu résorber.
* Un modèle indisponible.

#### Les autres erreurs relancent ou mettent en pause l'objectif

Après tout autre échec, l'objectif reste défini. Dans une session interactive sous Claude Code v2.1.269+, Claude Code affiche aussi une ligne nommant la cause, puis soit réessaie de lui-même, soit vous attend :

* **Nouvelle tentative (Retry)** : après un échec qui tend à se résorber seul, comme un serveur surchargé ou une connexion coupée, un avis commençant par `Goal still active` indique le délai avant la tentative suivante. Après trois nouvelles tentatives automatiques, l'objectif se met plutôt en pause.
* **Pause** : après un échec qu'une nouvelle tentative ne ferait que répéter, comme une limite de débit de l'API, une [limite d'utilisation](https://code.claude.com/docs/en/errors#youve-hit-your-session-limit) claude.ai, ou un hook qui a terminé le tour, un avis commençant par `Goal paused` nomme la cause. Si la session [attend de reprendre automatiquement à la réinitialisation d'une limite d'utilisation](https://code.claude.com/docs/en/interactive-mode#wait-for-a-usage-limit-to-reset), Claude reprend alors le travail vers l'objectif.

Envoyez un message à tout moment pour démarrer immédiatement le tour suivant. Pour désactiver les nouvelles tentatives automatiques, réglez [`CLAUDE_CODE_GOAL_CHECKIN_MINUTES`](https://code.claude.com/docs/en/env-vars) sur `0`, ce qui désactive aussi les [points d'étape](#le-travail-en-arrière-plan-diffère-lévaluation).

### Le travail en arrière-plan diffère l'évaluation

Si un subagent ou une commande shell en arrière-plan tourne encore à la fin d'un tour, Claude Code saute l'évaluation pour ce tour. Il évalue à la fin du prochain tour qui se termine sans travail en arrière-plan en cours. Lorsque le travail en arrière-plan se termine, Claude Code transmet le résultat à Claude sous forme de nouveau tour : vous n'avez donc pas à relancer.

Une fois que le travail en arrière-plan a fait patienter l'objectif pendant **30 minutes**, un point d'étape (check-in) est dû. Lors du point d'étape, Claude Code liste les tâches en cours et demande à Claude de lire leur sortie, de continuer à attendre si elles progressent, et de corriger ou d'arrêter celles qui sont bloquées. Après le premier point d'étape, Claude Code attend deux fois plus longtemps avant chaque point d'étape suivant, jusqu'à quatre fois le premier intervalle : avec la valeur par défaut, 1 heure après le premier point d'étape, puis toutes les 2 heures. Claude Code délivre un point d'étape dû, y compris le premier, de l'une de deux façons :

* **À la fin d'un tour** : Claude Code délivre le point d'étape à la fin du prochain tour qui se termine alors que le travail tourne encore. Dans une session non interactive, comme une session lancée avec `-p`, c'est la seule façon dont Claude Code délivre les points d'étape.
* **Pendant que la session est inactive** : dans une session interactive, Claude Code démarre aussi un tour de lui-même pour délivrer le point d'étape au lieu d'attendre votre prochain prompt. Si le travail en arrière-plan s'est arrêté sans rendre de résultat, Claude Code demande à Claude de continuer vers l'objectif. Claude Code démarre **au plus trois** points d'étape en inactivité par objectif entre vos prompts. Au troisième, Claude Code indique que les points d'étape en inactivité sont suspendus jusqu'à ce que vous envoyiez un autre prompt. Avant la v2.1.246, ces points d'étape n'étaient pas plafonnés. Les points d'étape en inactivité nécessitent Claude Code v2.1.236+.

Avant la v2.1.239, seuls les points d'étape en inactivité s'espaçaient ainsi ; un point d'étape délivré en fin de tour se répétait au premier intervalle.

Pour changer le premier intervalle, réglez [`CLAUDE_CODE_GOAL_CHECKIN_MINUTES`](https://code.claude.com/docs/en/env-vars). Claude Code utilise votre valeur à la place de l'intervalle de 30 minutes et met à l'échelle les intervalles suivants en conséquence. Réglez-la sur `0` pour désactiver les points d'étape et les [nouvelles tentatives automatiques](#les-autres-erreurs-relancent-ou-mettent-en-pause-lobjectif).

Les points d'étape nécessitent Claude Code v2.1.234+.

### Modèle d'évaluation et coût

Pour évaluer avec un autre modèle, réglez [`ANTHROPIC_DEFAULT_HAIKU_MODEL`](https://code.claude.com/docs/en/model-config#environment-variables).

> **Avertissement** : Claude Code lit `ANTHROPIC_DEFAULT_HAIKU_MODEL` partout où il utilise le petit modèle rapide, pas seulement pour l'évaluation de `/goal`. Quand vous le définissez, Claude Code résout aussi l'[alias `haiku`](https://code.claude.com/docs/en/model-config#model-aliases) vers ce modèle et y exécute les [fonctionnalités d'arrière-plan](https://code.claude.com/docs/en/costs#background-token-usage), comme le résumé de conversation.

L'évaluateur s'exécute sur le fournisseur pour lequel votre session est configurée. Il n'appelle pas d'outils : il ne peut donc juger que ce que Claude a déjà fait apparaître dans la conversation.

> **Note** : les tokens d'évaluation sont facturés sur le petit modèle rapide configuré pour votre fournisseur et sont généralement négligeables par rapport à la consommation des tours principaux.

## Prérequis

Claude Code rend `/goal` disponible selon la même [règle de confiance de l'espace de travail que les hooks des fichiers de réglages](https://code.claude.com/docs/en/permissions#what-runs-before-you-trust-a-folder), car l'évaluateur fait partie du système de hooks. `/goal` est aussi indisponible lorsque [`disableAllHooks`](https://code.claude.com/docs/en/hooks#disable-or-remove-hooks) vaut `true` après application de la priorité des réglages, ou lorsque [`allowManagedHooksOnly`](https://code.claude.com/docs/en/settings-reference#allowmanagedhooksonly) est défini dans les réglages gérés. Dans chaque cas, la commande vous en indique la raison au lieu de ne rien faire en silence.

## Voir aussi

* [Exécuter un prompt de façon répétée avec `/loop`](https://code.claude.com/docs/en/scheduled-tasks#run-a-prompt-repeatedly-with-%2Floop) : relancer selon un intervalle de temps plutôt que vers une condition.
* [Hooks à base de prompt](https://code.claude.com/docs/en/hooks-guide#prompt-based-hooks) : écrire votre propre hook Stop lorsque vous avez besoin d'une logique d'évaluation personnalisée.
* [Mode auto](https://code.claude.com/docs/en/auto-mode-config) : approuver automatiquement les appels d'outils pour que chaque tour de l'objectif s'exécute sans surveillance.
* [Comparaison des options de planification](https://code.claude.com/docs/en/scheduled-tasks#compare-scheduling-options) : exécuter du travail selon un planning, indépendamment de toute session ouverte.
