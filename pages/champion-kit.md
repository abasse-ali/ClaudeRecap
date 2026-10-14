# Kit du champion (« Champion kit »)

> Source : https://code.claude.com/docs/en/champion-kit · [← Index](../31-index-documentation-officielle.md)

*Un guide pratique pour les ingénieurs qui promeuvent Claude Code en interne : quoi partager, comment répondre aux questions et comment faire grandir l'adoption dans son équipe.*

Cette page s'adresse aux ingénieurs qui utilisent déjà Claude Code à titre individuel et veulent aider leur équipe à l'adopter. Elle couvre ce qu'il faut partager, comment répondre aux questions qu'on vous posera, un plan d'action sur trente jours et des réponses aux objections courantes.

L'adoption d'un outil de développement se produit rarement grâce à une annonce de déploiement. Elle se produit parce que quelqu'un dans l'équipe commence à bien utiliser l'outil, en parle ouvertement et permet aux autres de suivre facilement. Le travail que vous faites en tant que champion a un effet disproportionné : chaque exemple partagé raccourcit la courbe d'apprentissage des ingénieurs qui viennent après vous, et chaque question à laquelle vous répondez en public transforme l'expérience d'une personne en quelque chose sur quoi toute l'équipe peut construire.

## Le rôle de champion

Le rôle repose sur trois comportements qui se renforcent mutuellement.

| Comportement | À quoi cela ressemble en pratique | Pourquoi c'est important |
| - | - | - |
| Partager ce que vous découvrez | Publier les prompts, captures d'écran et petites victoires tirés de votre propre travail là où votre équipe lit déjà : un canal d'ingénierie, un fil de standup ou une description de pull request. | Les exemples tirés de votre propre base de code sont plus convaincants que toute documentation externe, car vos collègues voient exactement comment l'outil s'applique aux problèmes qu'ils partagent avec vous. |
| Être la personne qu'on sollicite | Quand un collègue demande comment vous avez obtenu un résultat, répondre avec le prompt réellement utilisé pour qu'il puisse l'appliquer directement à sa propre tâche. | Un exemple concret et exécutable comble l'écart entre la curiosité et une première utilisation réussie, là où la plupart des efforts d'adoption s'enlisent. |
| Élargir le cercle | Mettre en place un petit nombre d'habitudes légères et récurrentes, comme un canal dédié ou un fil hebdomadaire, pour que l'élan continue même quand votre attention est ailleurs. | Une adoption qui dépend d'une seule personne est fragile. Une adoption portée par des habitudes partagées continue de se renforcer d'elle-même. |

### Ce que cela devrait vous coûter

Fixez les attentes avec vous-même et avec votre responsable. Les activités ci-dessous sont prévues pour tenir dans une semaine de travail normale ; le rôle doit rester un multiplicateur de votre travail existant et non une responsabilité de support supplémentaire.

| Activité | Temps par semaine | Conseil |
| - | - | - |
| Publier des victoires et des prompts | Environ 15 minutes | Les capturer sur le moment avec une capture d'écran et une ou deux phrases ; éviter d'en faire des comptes rendus formels. |
| Répondre aux questions dans un canal partagé | Environ 20 minutes | Répondre une fois publiquement, puis renvoyer vers cette réponse quand la question revient. |
| Animer un fil hebdomadaire « show-and-tell » (montrer et raconter) | Environ 5 minutes | Vous publiez le message d'ouverture ; l'équipe fournit le contenu. |
| Binômage ou démonstrations guidées (facultatif) | 0 à 30 minutes | Réserver cela aux collègues bloqués, et proposer le lien vers le [Quickstart](https://code.claude.com/docs/en/quickstart) avant de planifier un créneau. |

## Partager ce que vous découvrez

Votre propre expérience est le matériau le plus convaincant que vos collègues rencontreront, car elle est spécifique à la base de code, aux workflows et aux problèmes que vous partagez. La documentation dit aux gens ce qui est possible ; vos publications leur montrent ce qui fonctionne réellement dans votre environnement.

### Ce qui vaut la peine d'être partagé

Les publications les plus utiles décrivent une technique qu'un collègue pourra réutiliser demain, plutôt qu'un résultat déjà terminé. Les techniques se démultiplient en se diffusant dans une équipe ; les points d'avancement, non.

Exemples de techniques réutilisables :

* « J'ai appris que mentionner un répertoire avec @ fonctionne. En le pointant sur `@src/components/` et en demandant lesquels manquaient de tests, il en a fait ressortir deux que j'avais oubliés. »
* « Le mode plan (`Shift+Tab`) expose d'abord les modifications proposées, c'est pourquoi je suis à l'aise de l'utiliser sur du code partagé. »
* « J'ai configuré un hook Stop pour recevoir une notification de bureau quand une longue tâche se termine. La configuration est dans le fil. »
* « Lancer `/init` génère un `CLAUDE.md` à partir du dépôt, ce qui évite que l'assistant redemande sans cesse nos conventions. »

### Où le partager

Publiez là où votre équipe lit déjà. Le but est de placer les exemples sur le chemin du travail normal plutôt que de créer une destination.

| Emplacement | Idéal pour | Format recommandé |
| - | - | - |
| Un canal `#claude-code` ou un canal d'ingénierie général | Découvertes, prompts et moments « aujourd'hui j'ai appris » | Une capture d'écran accompagnée d'une ou deux phrases de contexte |
| Descriptions de pull requests | Montrer l'approche sur du vrai code que les relecteurs lisent déjà | Une seule ligne, par exemple « Claude et moi avons fait ce refactoring ; je peux vous expliquer l'approche. » |
| Standups ou points écrits hebdomadaires | Normaliser l'usage auprès des responsables et des managers N+2 | Une phrase décrivant un résultat concret |
| Wiki d'équipe ou documentation interne | Patterns durables, skills personnalisés et exemples de `CLAUDE.md` | Une page courte, liée depuis le sujet du canal pour rester découvrable |

### Le format qui fonctionne

Une capture d'écran accompagnée d'une seule ligne de contexte, ou une brève description avant/après, est généralement le bon niveau de détail. Gardez chaque publication assez courte pour que quelqu'un qui fait défiler en retienne quand même l'essentiel. Un long compte rendu a tendance à être mis de côté pour plus tard puis oublié, alors qu'une publication courte avec capture d'écran a tendance à être copiée et essayée.

Les exemples de publications ci-dessous illustrent le ton et la longueur ; adaptez-les plutôt que de les copier mot pour mot.

```text theme={null}
Learned today that @-mentioning a directory works. I pointed it at
@src/components/ and asked which components were missing tests, and it
surfaced two I had forgotten about.
```

*(Traduction : « Appris aujourd'hui que mentionner un répertoire avec @ fonctionne. Je l'ai pointé sur @src/components/ en demandant quels composants manquaient de tests, et il en a fait ressortir deux que j'avais oubliés. »)*

```text theme={null}
I configured a Stop hook so I receive a desktop notification when a long
task completes. I started a refactor, stepped away, and was notified when
it finished. Configuration is in the thread.
```

*(Traduction : « J'ai configuré un hook Stop pour recevoir une notification de bureau quand une longue tâche se termine. J'ai lancé un refactoring, je me suis absenté, et j'ai été prévenu à la fin. La configuration est dans le fil. »)*

```text theme={null}
Plan mode is the reason I am comfortable using this on code that matters.
Press Shift+Tab until you see "plan"; it lays out the changes it proposes
without editing your source.
```

*(Traduction : « Le mode plan est la raison pour laquelle je suis à l'aise de l'utiliser sur du code important. Appuyez sur Shift+Tab jusqu'à voir « plan » ; il expose les modifications qu'il propose sans éditer vos sources. »)*

## Être la personne qu'on sollicite

Une fois que vous avez partagé quelques exemples, les questions suivront. C'est là qu'un champion aide le plus, car une bonne réponse à une personne débloque souvent plusieurs autres qui suivent le même canal.

### Répondre avec un prompt plutôt qu'avec une explication

Quand un collègue demande comment vous avez obtenu un résultat, la réponse la plus utile est le prompt que vous avez réellement utilisé. Il apprendra davantage en exécutant ce prompt sur son propre problème qu'avec n'importe quelle description que vous pourriez écrire, et cela lui donne quelque chose sur quoi agir immédiatement.

```text theme={null}
Colleague: How did you get it to find that race condition?

Champion: I asked, "The test in @tests/scheduler.test.ts is flaky, figure
out why," and it traced two unjoined promises in the scheduler. Try the
same phrasing on your test.
```

*(Traduction : « Collègue : Comment lui as-tu fait trouver cette race condition ? — Champion : J'ai demandé « Le test dans @tests/scheduler.test.ts est instable (flaky), trouve pourquoi », et il a remonté deux promesses non jointes dans le scheduler. Essaie la même formulation sur ton test. »)*

### Pointer vers la fonctionnalité plutôt que vers la documentation

Une réponse comme « Essaie le mode plan, appuie sur `Shift+Tab` jusqu'à le voir » est plus utile sur le moment qu'un lien vers la documentation. Si la personne a besoin d'approfondir plus tard, elle trouvera par elle-même ; pour l'instant, elle a besoin de l'unique chose qui la débloque.

### Questions que vous entendrez probablement

| Question | Réponse suggérée | Ressource de suivi |
| - | - | - |
| « Sur quoi devrais-je l'essayer en premier ? » | Recommander une tâche réelle mais circonscrite, idéalement un bug ou une corvée que la personne repousse parce qu'elle est fastidieuse plutôt que difficile. | [Common workflows](https://code.claude.com/docs/en/common-workflows) |
| « Comment lui faire confiance avec mon code ? » | Présenter le mode plan : appuyer sur `Shift+Tab` permet d'y entrer en faisant défiler les modes, et Claude analyse et propose des modifications sans éditer vos sources. | [Permissions](https://code.claude.com/docs/en/permissions) |
| « L'installation en vaut-elle la peine ? » | L'installation prend environ deux minutes, s'exécute dans le terminal et ne nécessite aucune extension d'IDE. Lancer `/init` une fois suffit pour commencer à travailler. | [Quickstart](https://code.claude.com/docs/en/quickstart) |
| « Il a produit un résultat incorrect. » | L'encourager à renvoyer l'échec à Claude. Coller le message d'erreur ou le test en échec est bien plus efficace que reformuler la demande initiale. | [Common workflows](https://code.claude.com/docs/en/common-workflows) |
| « Il ne comprend pas les conventions de notre base de code. » | Suggérer de lancer `/init` pour générer un fichier `CLAUDE.md`, puis d'y ajouter les conventions de l'équipe, les commandes de test et les répertoires à éviter. | [Memory](https://code.claude.com/docs/en/memory) |
| « Ce n'est que de l'autocomplétion ? » | Proposer une brève démonstration où Claude explique un fichier inconnu, trace un bug à travers plusieurs services ou rédige un plan de migration. Ces tâches demandent de raisonner sur l'ensemble du dépôt plutôt que de compléter une seule ligne. | Une démonstration en direct de deux minutes |
| « Et la sécurité et le traitement des données ? » | Renvoyer cette question à votre administrateur. La politique de déploiement et de traitement des données de votre organisation est déjà configurée, et les champions ne doivent pas improviser cette réponse. | [Security](https://code.claude.com/docs/en/security) · [Data usage](https://code.claude.com/docs/en/data-usage) |

## Élargir le cercle

L'objectif est d'établir un petit nombre d'habitudes légères qui permettent à l'élan de continuer une fois que vous avez cessé de le porter activement. Vous n'avez pas besoin de bâtir un programme ni de piloter un déploiement. Quand les questions du canal reçoivent des réponses d'autres personnes que vous, le rôle a rempli sa mission.

### Patterns qui fonctionnent généralement

| Pattern | Comment le mettre en œuvre | Effort requis |
| - | - | - |
| Un canal dédié | Créer un canal `#claude-code` (ou un fil récurrent dans un canal existant), épingler le lien vers le [Quickstart](https://code.claude.com/docs/en/quickstart) et un exemple marquant, et répondre aux questions publiquement pour que chaque réponse profite à tous ceux qui suivent. | Environ cinq minutes de mise en place, puis en arrière-plan |
| Un fil hebdomadaire « show-and-tell » | Chaque vendredi, publier « Sur quoi Claude vous a-t-il aidé cette semaine ? ». Aucune préparation, aucune diapositive ni réunion n'est nécessaire ; des captures d'écran et de courtes descriptions suffisent. | Environ deux minutes par semaine |
| Partager un skill personnalisé | Publier votre fichier `.claude/skills/<name>/SKILL.md` le plus utile, par exemple un skill `/ship` qui lance les tests et le lint avant de committer, avec une description d'une ligne. Les skills étant du simple Markdown, les collègues peuvent les adopter immédiatement. | Environ cinq minutes par skill |
| Générer un guide d'installation à partir de votre propre usage | Lancer `/team-onboarding` dans un projet sur lequel vous avez réellement passé du temps. Claude analyse vos sessions récentes, vos commandes et vos serveurs MCP, puis produit un guide qu'un nouveau coéquipier peut coller comme premier message pour reproduire votre configuration. L'épingler dans le canal. | Environ deux minutes |
| Faire binôme sur une première tâche | Proposer une seule session de binômage de quinze minutes à toute personne qui débute. Un succès sur son propre code est plus convaincant que n'importe quelle présentation. | Environ quinze minutes par personne |
| Identifier le prochain champion | Le collègue qui vous pose le plus de questions est généralement prêt à endosser ce rôle. Transférez-lui cette page et répartissez-vous les responsabilités du canal. | Négligeable |

### Plan d'action sur trente jours

Si un plan souple vous aide, la séquence ci-dessous reflète ce qui fonctionne dans la plupart des équipes. Adaptez-la librement à votre contexte.

1. **Semaine 1 : amorcer le canal**
   Créez le canal, épinglez le [Quickstart](https://code.claude.com/docs/en/quickstart) et publiez deux ou trois de vos propres exemples en incluant les prompts.

   **Signe que cela fonctionne :** quelques collègues réagissent ou répondent, et au moins une question est posée dans le canal.

2. **Semaine 2 : lancer le rythme**
   Lancez le fil hebdomadaire « show-and-tell », répondez publiquement à chaque question et partagez un skill personnalisé ou un extrait de `CLAUDE.md`.

   **Signe que cela fonctionne :** quelqu'un d'autre que vous publie un exemple qui lui est propre.

3. **Semaine 3 : binômer et consolider**
   Proposez deux ou trois courtes sessions de binômage et regroupez les questions et réponses les plus fréquentes dans un message FAQ épinglé.

   **Signe que cela fonctionne :** vous constatez un usage répété, les mêmes collègues revenant plutôt que d'essayer une fois puis d'arrêter.

4. **Semaine 4 : passer le relais**
   Identifiez un second champion et partagez avec votre responsable ou votre administrateur un bref résumé de ce qui fonctionne et de ce qui ne fonctionne pas.

   **Signe que cela fonctionne :** les questions du canal reçoivent des réponses d'autres personnes que vous.

### Quand quelqu'un veut aller plus loin

Vous êtes l'introduction chaleureuse, pas le programme d'intégration. Quand un collègue passe de « devrais-je essayer ? » à « comment devenir efficace avec ? », orientez-le vers les pages [Quickstart](https://code.claude.com/docs/en/quickstart) et [Common workflows](https://code.claude.com/docs/en/common-workflows). Elles contiennent de courtes sections couvrant les fonctionnalités utiles mais difficiles à découvrir seul.

## Répondre aux objections courantes

Un scepticisme sain est normal ; les ingénieurs doivent être prudents avec les outils qui touchent à leur code. La réponse la plus efficace consiste rarement à argumenter sur le cas général. Reconnaissez plutôt l'inquiétude, proposez un bref recadrage et suggérez une démonstration concrète sur le code de la personne. La plupart des objections sont levées par une seule expérience réussie.

| Objection | Réponse suggérée | Preuve à proposer |
| - | - | - |
| « Je vais plus vite sans. » | C'est probablement vrai pour le code que la personne écrit couramment. Suggérer de l'essayer sur le travail qu'elle a tendance à éviter : fichiers legacy, services peu familiers ou échafaudage de tests, là où il aide le plus. | Chronométrer une tâche fastidieuse des deux façons et comparer. |
| « Je ne fais pas confiance à l'IA pour toucher au code de production. » | Convenir qu'aucune modification ne doit être intégrée sans avoir été lue. Suggérer le mode plan pour voir d'abord les modifications proposées, puis relire le diff avec la même exigence que n'importe quelle pull request. | Démontrer le mode plan sur un vrai fichier. |
| « Cela va affaiblir les ingénieurs juniors. » | Bien utilisé, c'est un excellent explicateur. Encourager les juniors à demander à Claude d'expliquer un fichier et ses points d'appel avant de lui demander de modifier quoi que ce soit. | Lancer ensemble « Explain @file and where it is called from » (« Explique @file et d'où il est appelé »). |
| « Je l'ai essayé une fois et il a halluciné. » | C'est généralement un problème de contexte plutôt qu'un problème de modèle. Mentionner les fichiers pertinents avec @, lancer `/init` et fournir la sortie d'erreur réelle résout généralement le problème. | Relancer leur prompt d'origine avec un contexte `@` approprié. |
| « Nous n'avons pas le temps d'apprendre un nouvel outil. » | Claude Code est une commande de terminal, pas une plateforme. S'il n'apporte pas de valeur dès la première session, il est raisonnable de le mettre de côté. | Une installation de deux minutes suivie d'un vrai bug. |

## Aide-mémoire

Les techniques ci-dessous sont celles qui font le plus sûrement passer quelqu'un d'un premier essai à un usage quotidien. Épinglez ce tableau dans un canal ou partagez-le séparément.

| Technique | Comment l'appliquer |
| - | - |
| Fournir le bon contexte | Utiliser des références `@file` ou `@directory/`, ou coller directement la sortie d'erreur ou de log. Fournir un contexte pertinent est plus efficace qu'un prompt élaboré. |
| Relire le plan avant la modification | Appuyer sur `Shift+Tab` pour entrer en mode plan. Claude analyse et propose des modifications sans éditer vos sources. |
| Lui apprendre votre dépôt | Lancer `/init` pour générer un fichier `CLAUDE.md`, puis y ajouter vos conventions, vos commandes de test et les répertoires à ne pas modifier. Voir [Memory](https://code.claude.com/docs/en/memory). |
| Réutiliser un workflow | Enregistrer un fichier `SKILL.md` dans `.claude/skills/<name>/` pour créer un skill `/name` utilisable par toute l'équipe. Voir [Skills](https://code.claude.com/docs/en/skills). |
| Rester informé pendant les longues tâches | Configurer un hook Stop pour recevoir une notification de bureau quand une tâche longue se termine. Voir [Hooks](https://code.claude.com/docs/en/hooks-guide). |
| Se remettre d'un résultat incorrect | Plutôt que reformuler la demande, recoller le test en échec ou la stack trace à Claude et lui demander de traiter cet échec précis. |
| Garder des modifications chirurgicales | Demander un diff, ou préciser « ne modifie que X ». Claude respecte le périmètre quand il est explicité. |

> **Astuce** : Claude Code est mis à jour fréquemment. Vérifiez les détails propres à une version sur la [page d'accueil de la documentation](https://code.claude.com/docs/en/overview) avant de diffuser ce matériel en interne.
