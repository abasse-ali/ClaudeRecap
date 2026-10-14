# Comment Claude Code utilise la mise en cache des prompts (prompt caching)

> Source : https://code.claude.com/docs/en/prompt-caching · [← Index](../31-index-documentation-officielle.md)

> Claude Code gère automatiquement la mise en cache des prompts. Cette page explique pourquoi un changement de modèle déclenche un tour lent non mis en cache, ce que coûte `/compact`, pourquoi les modifications de CLAUDE.md ne s'appliquent pas en cours de session, et comment vérifier votre taux de réussite du cache (cache hit rate).

La mise en cache des prompts rend Claude Code plus rapide et plus économique. Sans cache, l'API retraiterait tout votre historique à chaque tour. Avec le cache, elle réutilise ce qu'elle a déjà traité, facture cette relecture au [tarif des tokens en cache](https://platform.claude.com/docs/en/about-claude/pricing), et ne traite entièrement que ce qui a changé.

Claude Code gère la mise en cache pour vous, sauf si vous la [désactivez](https://code.claude.com/docs/en/prompt-caching#disable-prompt-caching). Il reste utile de comprendre son fonctionnement, car certaines actions invalident le cache et rendent la réponse suivante plus lente et plus chère le temps qu'il se reconstruise. Cette page indique quelles actions sont concernées, pourquoi certains réglages attendent un redémarrage pour s'appliquer, et comment vérifier les performances du cache quand la consommation semble élevée.

---

## Comment le cache est organisé

À chaque message envoyé dans Claude Code, une nouvelle requête API est émise. Le modèle ne se souvient de rien entre deux requêtes : Claude Code renvoie donc tout le contexte — le prompt système, le contexte du projet, chaque message et résultat d'outil précédent, et votre nouveau message. Le nouveau contenu est ajouté à la fin, si bien que la majeure partie de chaque requête est identique à la précédente. La mise en cache des prompts permet à l'API d'éviter de retraiter la partie inchangée.

L'API met en cache en comparant le début de chaque requête, appelé **préfixe**, avec du contenu traité récemment. Lors d'un tour normal, le préfixe correspond à la totalité de la requête précédente et seul le dernier échange est nouveau. La correspondance est **exacte** : une modification n'importe où dans le préfixe force le recalcul de tout ce qui suit. Il n'existe **pas de cache par fichier ni par segment**. Voir [comment fonctionne la mise en cache des prompts](https://platform.claude.com/docs/en/build-with-claude/prompt-caching#how-prompt-caching-works) dans la référence API pour le mécanisme sous-jacent.

*(Schéma de la source : quatre tours représentés par des barres horizontales croissantes. Chaque requête contient tout le tour précédent plus le dernier échange ajouté à la fin. Aux tours 2 et 3, le préfixe inchangé est lu depuis le cache et seul le nouvel échange est traité. Au tour 4, le prompt système a changé : le préfixe ne correspond plus et toute la requête est retraitée et réécrite.)*

Pour tirer le meilleur parti de cette correspondance de préfixe, Claude Code ordonne chaque requête de façon que le contenu qui change rarement d'un tour à l'autre vienne en premier :

| Couche | Contenu | Change quand |
| - | - | - |
| Prompt système | Instructions de base, définitions d'outils | L'ensemble des définitions d'outils chargées change |
| Contexte du projet | CLAUDE.md, mémoire automatique (auto memory), règles non ciblées (unscoped rules) | Au démarrage de la session, ou après `/clear` ou `/compact` |
| Conversation | Vos messages, les réponses de Claude, les résultats d'outils | À chaque tour |

Une modification de la couche conversation laisse en cache le prompt système et le contexte du projet. Une modification du prompt système invalide **tout**, car tout le contenu suivant se trouve désormais derrière un préfixe différent. La troisième colonne donne des déclencheurs courants, pas une liste exhaustive ; les sections ci-dessous couvrent l'ensemble des cas.

La règle de correspondance de préfixe explique la plupart des comportements décrits sur cette page. Par exemple, le [mode plan](https://code.claude.com/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode) et le [chargement de skills](https://code.claude.com/docs/en/skills) ajoutent leurs instructions sous forme de messages de conversation, si bien que le préfixe en cache reste intact.

Deux réglages n'apparaissent pas dans le tableau des couches mais influencent ce qui reste en cache :

* **Modèle** : chaque modèle a son propre cache. Changer de modèle recalcule toute la requête, même si le contenu est identique. Voir [Changer de modèle](#changer-de-modèle) ci-dessous.
* **Niveau d'effort** : sur la plupart des modèles, chaque niveau d'effort a son propre cache ; changer d'effort en cours de session recalcule donc toute la requête. Sur Opus 5.5, Sonnet 5.5, Haiku 5.5 et Fable 5.1 avec une clé API ou un abonnement Claude, le cache reste intact par défaut. Voir [Changer le niveau d'effort](#changer-le-niveau-deffort) ci-dessous.

> **Astuce** : choisissez votre modèle et votre niveau d'effort en début de session, et réservez `/compact` aux pauses naturelles entre deux tâches. Moins vous faites de changements en pleine tâche, plus votre taux de réussite du cache est élevé.

### Où se trouve le cache

La mise en cache a lieu côté serveur, dans l'infrastructure qui sert votre modèle. Son emplacement dépend de votre mode d'authentification :

* **Clé API, abonnement Claude ou [Claude Platform on AWS](https://code.claude.com/docs/en/claude-platform-on-aws)** : le cache se trouve dans l'infrastructure d'Anthropic, accessible via l'[API Claude](https://platform.claude.com/docs).
* **Amazon Bedrock ou Google Cloud's Agent Platform** : le cache se trouve dans l'infrastructure de service de votre fournisseur cloud.
* **Microsoft Foundry** : dépend de l'[option d'hébergement](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry#hosting-options) du déploiement. Les déploiements « Hosted on Azure » sont servis sur l'infrastructure Azure ; les déploiements « Hosted on Anthropic » sont servis sur l'infrastructure d'Anthropic.
* **`ANTHROPIC_BASE_URL` personnalisée ou [passerelle LLM](https://code.claude.com/docs/en/llm-gateway)** : le cache se trouve là où vos requêtes sont transférées, et le bon fonctionnement du cache dépend de la passerelle.

Claude Code ajoute aussi du contexte système en cours de conversation, comme des notifications de modification de fichiers, et marque ce bloc pour la mise en cache sur tous les fournisseurs et toutes les connexions, **sauf** si vous définissez [`CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS`](https://code.claude.com/docs/en/llm-gateway-protocol#disable-pre-release-capabilities) : dans ce cas, ce bloc est envoyé sans mise en cache.

Sur le point de terminaison propre au fournisseur, Amazon Bedrock et son [point de terminaison Mantle](https://code.claude.com/docs/en/amazon-bedrock#use-the-mantle-endpoint), Google Cloud's Agent Platform et Microsoft Foundry mettent ce bloc en cache de la même manière que l'API Claude.

Quand vos requêtes passent par une [passerelle LLM](https://code.claude.com/docs/en/llm-gateway), une `ANTHROPIC_BASE_URL` personnalisée ou une surcharge d'URL de base d'un fournisseur cloud comme [`ANTHROPIC_BEDROCK_BASE_URL`](https://code.claude.com/docs/en/env-vars), ce qui reste en cache dépend de la façon dont la passerelle traite les [marqueurs `cache_control`](https://platform.claude.com/docs/en/build-with-claude/prompt-caching#explicit-cache-breakpoints) envoyés par Claude Code :

* **Elle les transmet sans modification** : le bloc et votre conversation sont mis en cache comme sur le point de terminaison propre au fournisseur.
* **Elle rejette la requête marquée avec une erreur `400` mentionnant `cache_control`** : Claude Code renvoie la requête en déplaçant le marqueur du bloc vers votre dernier message de conversation, et l'y laisse pour le reste de la conversation. Le bloc est facturé comme entrée non mise en cache ; votre conversation reste en cache.
* **Elle supprime les marqueurs tout en renvoyant un succès** : tout votre historique de conversation est facturé comme entrée non mise en cache à **chaque tour**. Une passerelle qui convertit le contenu système sous forme de blocs en simple chaîne de caractères supprime le marqueur de la même façon.

Pour savoir ce que chaque fournisseur stocke et traite, voir [utilisation des données](https://code.claude.com/docs/en/data-usage). Où que se trouve le cache, les entrées expirent après une période d'inactivité ; la section [Durée de vie du cache](#durée-de-vie-du-cache) ci-dessous décrit le TTL et comment le prolonger.

---

## Actions qui invalident le cache

Ces actions peuvent faire manquer tout ou partie du cache à la requête suivante. Vous constatez alors un tour ponctuellement plus lent et plus cher, après quoi le nouveau préfixe est mis en cache. La plupart sont évitables en pleine tâche une fois que l'on sait qu'elles ont un coût. Un changement de modèle peut sembler gratuit jusqu'à ce que l'on remarque le tour plus lent qui suit.

* [Changer de modèle](#changer-de-modèle)
* [Changer le niveau d'effort](#changer-le-niveau-deffort)
* [Activer le mode rapide](#activer-le-mode-rapide-fast-mode)
* [Connecter ou retirer un serveur MCP](#connecter-ou-retirer-un-serveur-mcp)
* [Activer ou désactiver un plugin](#activer-ou-désactiver-un-plugin)
* [Refuser un outil entier](#refuser-un-outil-entier-deny)
* [Compacter la conversation](#compacter-la-conversation)
* [Accumuler de nombreuses images](#accumuler-de-nombreuses-images)
* [Mettre à jour Claude Code](#mettre-à-jour-claude-code)

### Changer de modèle

Chaque modèle a son propre cache. Changer de modèle avec [`/model`](https://code.claude.com/docs/en/model-config#setting-your-model) signifie que la requête suivante relit tout l'historique de la conversation sans aucun accès au cache, même si le contenu est identique.

Quand vous lancez `/model` dans le terminal, Claude Code vous demande de confirmer le changement **uniquement** tant que le cache est encore « chaud » (warm) et que le nouveau modèle n'est pas celui qui a produit la dernière réponse. Le cache reste chaud pendant un [TTL de cache](#durée-de-vie-du-cache) après le dernier envoi d'une requête par Claude Code dans cette conversation ou la dernière réponse de Claude. Une fois ce délai écoulé, le cache a expiré et Claude Code change de modèle sans demander.

Avant la v2.1.238, Claude Code ne vérifiait pas le TTL du cache et demandait confirmation même après expiration du cache.

Vous pouvez aussi exiger cette confirmation ou la sauter avec un [hook PreModelSwitch](https://code.claude.com/docs/en/hooks#premodelswitch-decision-control).

Le réglage de modèle [`opusplan`](https://code.claude.com/docs/en/model-config#opusplan-model-setting) se résout en Opus pendant le mode plan et en Sonnet pendant l'exécution : chaque bascule du mode plan est donc un changement de modèle et démarre un cache neuf.

Le [repli automatique de modèle](https://code.claude.com/docs/en/model-config#automatic-model-fallback) (automatic model fallback) sur les modèles Fable, Opus 5.5, Sonnet 5.5 et Opus 5 est aussi un changement de modèle. Quand un classifieur de sécurité signale une requête dans une catégorie qui possède un modèle de repli et que Claude Code relance la requête sur ce modèle, la session continue sur celui-ci.

Quand le frontmatter d'une skill ou d'une commande indique un [`model`](https://code.claude.com/docs/en/skills#frontmatter-reference) différent du modèle courant de la session, ce tour est aussi un changement de modèle : la requête suivante relit tout l'historique sans accès au cache. Le modèle de la session reprend à votre prompt suivant. Une skill `context: fork` définit plutôt le [modèle du sous-agent forké](https://code.claude.com/docs/en/skills#run-skills-in-a-subagent).

### Changer le niveau d'effort

Sur la plupart des modèles, changer le [niveau d'effort](https://code.claude.com/docs/en/model-config#adjust-effort-level) en cours de session signifie que la requête suivante relit tout l'historique sans accès au cache. Tant que le cache est chaud, Claude Code vous demande d'abord de confirmer le changement.

Sur Opus 5.5, Sonnet 5.5, Haiku 5.5 et Fable 5.1 avec une clé API ou un abonnement Claude, changer d'effort **conserve** le cache, et Claude Code applique le nouveau niveau sans demander. Cela **ne s'applique pas** :

* sur Amazon Bedrock ;
* sur Google Cloud's Agent Platform ;
* via une [passerelle Claude apps (Claude apps gateway)](https://code.claude.com/docs/en/claude-apps-gateway) ;
* quand vous définissez [`CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS`](https://code.claude.com/docs/en/llm-gateway-protocol#disable-pre-release-capabilities) ;
* ou quand votre organisation a une configuration HIPAA.

Avant la v2.1.260, changer d'effort sur Fable 5.1 avec une clé API ou un abonnement Claude invalidait aussi le cache.

### Activer le mode rapide (fast mode)

Activer le [mode rapide](https://code.claude.com/docs/en/fast-mode) ajoute un en-tête de requête qui fait partie de la clé de cache : la première requête envoyée par Claude Code avec le mode rapide activé relit donc tout l'historique sans accès au cache. Claude Code définit cet en-tête une seule fois au début d'un tour et le conserve pendant tout le tour ; si vous activez le mode rapide pendant que Claude travaille, l'échec de cache dû à l'en-tête se produit à la première requête de votre tour **suivant**. Ces tokens d'entrée non mis en cache sont facturés au [tarif du mode rapide](https://code.claude.com/docs/en/fast-mode#understand-the-cost-tradeoff), ce qui explique pourquoi l'activer en début de session coûte moins cher que l'activer au milieu d'une longue session. Si votre modèle actuel ne prend pas en charge le mode rapide, l'activer [change aussi votre modèle](#changer-de-modèle), et ce changement démarre à lui seul un cache neuf dès la requête suivante du tour en cours.

Ce coût ne s'applique **qu'une fois par conversation**. Après le premier tour en mode rapide, Claude Code continue d'envoyer l'en-tête et ne fait varier que le paramètre de vitesse de la requête, qui ne fait pas partie de la clé de cache. Désactiver le mode rapide, le [repli automatique vers la vitesse standard](https://code.claude.com/docs/en/fast-mode#handle-rate-limits) après une limite de débit, et le réactiver plus tard conservent tous le cache. Si vous [épuisez vos crédits d'usage](https://code.claude.com/docs/en/fast-mode#handle-rate-limits) en cours de session, Claude Code relance de la même façon chaque requête en mode rapide rejetée à vitesse standard, donc ce repli conserve aussi le cache. `/clear` et `/compact` réinitialisent ce comportement, puisqu'ils reconstruisent de toute façon le cache à ce moment-là.

### Connecter ou retirer un serveur MCP

Les définitions d'outils se trouvent dans la couche prompt système : le cache est donc invalidé quand l'ensemble des définitions d'outils de la requête change d'un tour à l'autre. Basculer l'[outil advisor](https://code.claude.com/docs/en/advisor) est une exception : sa définition se trouve après le point d'arrêt du cache (cache breakpoint), donc activer ou désactiver `/advisor` laisse le préfixe en cache intact. Qu'un changement de [serveur MCP](https://code.claude.com/docs/en/mcp) invalide ou non le cache dépend du fait que la [recherche d'outils (tool search)](https://code.claude.com/docs/en/mcp#scale-with-mcp-tool-search) diffère ou non les outils MCP de la session — comportement par défaut sur les modèles compatibles :

* **Outils différés (deferred)** : Claude Code conserve la liste d'outils de la première requête de la conversation pendant toute la conversation ; un serveur qui se connecte ou se déconnecte en cours de session ne perturbe donc rien de ce qui est déjà en cache. Un serveur qui termine sa connexion après la première requête fournit ses outils sous forme de définitions différées que Claude charge à la demande.
* **Outils chargés d'emblée (upfront)** : ajouter une définition invalide le cache, tout comme en retirer une volontairement. C'est le cas lorsque la recherche d'outils est [sous son seuil `auto`, désactivée ou indisponible](https://code.claude.com/docs/en/mcp#configure-tool-search), par exemple :
  * sur les modèles Google Cloud's Agent Platform antérieurs à Opus 4.5, Sonnet 4.5 et Haiku 4.5 ;
  * avec une passerelle `ANTHROPIC_BASE_URL` personnalisée ;
  * sur un [déploiement Microsoft Foundry hébergé sur Azure](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry#hosting-options), une fois que Claude Code détecte que le déploiement rejette la recherche d'outils.

Sans recherche d'outils, l'invalidation du cache par un changement de serveur en cours de session dépend de ce qui a changé. Pour chaque changement, ce tableau indique si le cache est conservé et ce qu'il advient des définitions d'outils dans la requête suivante.

| Changement en cours de session | Cache | Définitions d'outils dans la requête suivante |
| - | - | - |
| Un serveur se connecte, ou une [mise à jour dynamique d'outils](https://code.claude.com/docs/en/mcp#dynamic-tool-updates) ajoute des outils | Invalidé | Les nouvelles définitions sont ajoutées |
| Un serveur disparaît sans action de votre part, par exemple quand le processus d'un serveur stdio se termine | Conservé | Les définitions du serveur restent inchangées. Un appel à l'un de ses outils renvoie une erreur au lieu de s'exécuter |
| Un serveur distant [se reconnecte automatiquement](https://code.claude.com/docs/en/mcp#automatic-reconnection) après une coupure de connexion | Conservé, sauf si une requête envoyée pendant la reconnexion ajoute l'outil `WaitForMcpServers`, ce qui invalide le cache une fois | Les définitions du serveur restent inchangées. Une requête envoyée pendant la reconnexion peut ajouter `WaitForMcpServers` si la conversation ne l'a pas encore listé ; l'outil reste alors listé pour le reste de la conversation |
| Vous retirez un outil volontairement, par exemple avec une [règle deny](#refuser-un-outil-entier-deny) ou en désactivant son serveur dans `/mcp` | Invalidé | La définition est retirée |

Quand vous reprenez une conversation dont les outils sont chargés dans le préfixe, l'un de ses serveurs MCP peut encore être en cours de connexion au moment où part la première requête. Si la transcription a enregistré les définitions d'outils de ce serveur, cette requête les inclut telles qu'enregistrées ; elle ne change donc pas lorsque le serveur termine sa connexion avec les mêmes outils.

Modifier votre configuration MCP ne change pas le cache à lui seul. La nouvelle configuration ne prend effet qu'après un redémarrage, moment où le serveur se connecte ou se déconnecte.

### Activer ou désactiver un plugin

Quand vous activez ou désactivez un [plugin](https://code.claude.com/docs/en/plugins/overview), le coût du changement dépend des types de composants fournis par le plugin. Les cas ci-dessous couvrent chaque type de composant, le moment où Claude Code applique le changement, et ce qui se passe si vous désactivez à nouveau un plugin dans la même session.

#### Composants de plugin qui conservent le cache

Claude Code n'invalide **jamais** le cache pour les skills, commandes, agents, hooks, moniteurs (monitors) ou thèmes d'un plugin. Il ajoute leur contenu après la conversation existante : la requête suivante paie pour ce contenu et lit toujours tout ce qui précède depuis le cache.

#### Plugins qui fournissent des serveurs MCP

Quand vous activez ou désactivez un plugin qui fournit des [serveurs MCP](https://code.claude.com/docs/en/plugins/components#mcp-servers), Claude Code applique les mêmes règles que lorsque vous [connectez ou retirez un serveur MCP](#connecter-ou-retirer-un-serveur-mcp).

#### Plugins d'intelligence de code

Quand vous activez un [plugin d'intelligence de code](https://code.claude.com/docs/en/plugins/code-intelligence), Claude obtient l'[outil LSP](https://code.claude.com/docs/en/tools-reference#lsp-tool-behavior).

#### Quand les changements de plugins s'appliquent

Un changement effectué dans le menu `/plugin` passe par [`/reload-plugins`](https://code.claude.com/docs/en/plugins/cli-reference#reload-plugins), que Claude Code exécute pour vous à la fermeture du menu. Vous payez le coût — annonces ajoutées ou relecture complète — au premier tour suivant l'application du changement. Claude Code peut aussi appliquer un changement de lui-même :

* Pour un plugin avec une source `command`, Claude Code [peut recharger le plugin lui-même](https://code.claude.com/docs/en/plugins/loading#when-a-command-source-re-runs).
* Quand vous [installez un plugin depuis l'interface `/plugin`](https://code.claude.com/docs/en/plugins/install#install-a-plugin), Claude Code peut l'activer pendant l'installation. Le résumé d'installation indique s'il l'a fait.
* Quand vous [déplacez la session avec `/cd`](https://code.claude.com/docs/en/permissions#move-the-session-to-another-directory) (v2.1.246+), Claude Code applique les plugins activés par les réglages du nouveau répertoire dans le cadre du déplacement, sans l'avertissement de relecture complète qui bloque un `/reload-plugins`.
* Dans les sessions interactives, quand vous ajoutez ou retirez un plugin dans un [dossier de plugins](https://code.claude.com/docs/en/plugins/create#load-a-directory-or-archive-for-one-session) passé avec `--plugin-dir`, le changement s'applique immédiatement. Si l'appliquer déclenchait une relecture complète, Claude Code met plutôt le changement en attente et affiche une notification invitant à lancer `/reload-plugins`. (v2.1.265+)

Quand `/reload-plugins` s'exécute et que le rechargement déclencherait une relecture complète, Claude Code affiche un avertissement et **n'applique pas** le rechargement. Lancez `/reload-plugins --force` pour l'appliquer malgré tout.

`/reload-plugins` fonctionne aussi dans les sessions sans terminal interactif — application de bureau, Agent SDK et [mode non interactif](https://code.claude.com/docs/en/headless) avec `-p` — quand vous le tapez directement dans la session. (v2.1.260+)

Dans ces sessions, le rechargement applique tout **sauf** les changements de serveurs MCP de plugins, qui [prennent effet à la session suivante](https://code.claude.com/docs/en/plugins/cli-reference#reload-plugins) et ne coûtent donc jamais de relecture complète en cours de session.

#### Plugins activés puis désactivés dans la même session

Quand vous désactivez un plugin activé plus tôt dans la session, Claude Code restaure la forme de requête précédente. Si ce préfixe est encore dans sa [durée de vie de cache](#durée-de-vie-du-cache), la requête suivante lit l'ancienne entrée de cache au lieu de reconstruire.

### Refuser un outil entier (deny)

Si vous ajoutez un nom d'outil seul comme `Bash` ou `WebFetch` en tant que [règle deny](https://code.claude.com/docs/en/permissions#manage-permissions), Claude ne peut plus appeler cet outil à partir de votre requête suivante, que vous ajoutiez la règle via `/permissions` ou en [modifiant directement un fichier de réglages](https://code.claude.com/docs/en/settings#when-edits-take-effect). Cela inclut une règle ajoutée via `/permissions` au milieu d'un tour.

Quand la [recherche d'outils](https://code.claude.com/docs/en/mcp#scale-with-mcp-tool-search) est active — comportement par défaut sur les modèles compatibles —, les définitions d'outils de la requête ne changent pas et le préfixe en cache survit. Quand la recherche d'outils est indisponible ou désactivée, Claude Code retire la définition de la requête suivante, ce qui invalide le cache ; retirer la règle plus tard l'invalide aussi.

Seule une règle deny qui correspond **en position de nom d'outil** bloque un outil de cette façon :

* un nom d'outil seul ;
* la forme équivalente `Bash(*)` ;
* un [glob de nom d'outil](https://code.claude.com/docs/en/permissions#tool-name-wildcards) comme `"*"`.

Un glob qui ne correspond qu'aux outils MCP, comme `"mcp__*"`, bloque ces outils de la même manière. Les règles deny ciblées comme `Bash(rm *)`, ainsi que toutes les règles allow et ask, ne changent pas les outils visibles par Claude : Claude Code les vérifie quand Claude tente un appel, ce qui laisse le préfixe intact.

### Compacter la conversation

Le [compactage](https://code.claude.com/docs/en/context-window#what-survives-compaction) remplace votre historique de messages par un résumé. Par conception, il invalide la couche conversation, puisque la requête suivante a un historique nouveau et plus court qui ne partage pas de préfixe avec l'ancien. Claude Code réutilise la couche prompt système, sauf si la conversation a été [reprise en conservant un prompt système qui aurait sinon changé](#reprendre-une-session) ; dans ce cas, le premier compactage bascule vers le prompt actuel et cette couche se reconstruit une fois. Il recharge le contexte du projet depuis le disque, ce qui ne touche le cache que si CLAUDE.md et la mémoire n'ont pas changé depuis le début de la session.

Pour produire le résumé, Claude Code envoie une requête distincte avec le même prompt système, les mêmes outils et le même historique que votre conversation, plus une instruction de résumé ajoutée comme dernier message utilisateur. Tant que le cache est chaud, cette requête lit votre préfixe depuis le cache : un `/compact` en cours de session coûte donc une fraction de ce que suggère la taille du contexte, et passe l'essentiel de son temps à générer le résumé.

Après une pause plus longue que la [durée de vie du cache](#durée-de-vie-du-cache), il n'y a plus de cache à lire : la requête de résumé retraite tout l'historique comme entrée non mise en cache. C'est pourquoi `/compact` coûte le plus cher quand vous [reprenez une ancienne session](https://code.claude.com/docs/en/sessions#resume-from-a-summary). Dans les deux cas (cache chaud ou froid), le tour qui suit le compactage ne reconstruit le cache de conversation que pour le résumé, bien plus court : ce tour-là n'est donc pas la partie lente.

> **Astuce** : le compactage joue en votre faveur quand le contexte abandonné est du contenu dont vous n'avez plus besoin. Pour choisir le moment où son surcoût intervient, lancez `/compact` à une pause naturelle de votre travail, par exemple entre deux tâches, plutôt que d'attendre que l'auto-compactage se déclenche en pleine tâche. Si vous vous êtes engagé dans une voie que vous voulez abandonner entièrement, utilisez plutôt [`/rewind`](#revenir-en-arrière-dans-la-conversation-rewind) pour revenir à un tour antérieur. Le retour en arrière tronque jusqu'à un préfixe déjà en cache, au lieu d'en construire un nouveau comme le fait le compactage.

Si vous laissez une longue conversation inactive dans une session terminal alors qu'elle bénéficie du [TTL d'une heure](#quel-ttl-reçoit-chaque-requête), Claude Code peut la compacter peu avant l'expiration du cache. Pour désactiver ce compactage en inactivité, définissez [`idleCompaction`](https://code.claude.com/docs/en/settings-reference#idlecompaction) à `false` (v2.1.290+).

### Accumuler de nombreuses images

L'API limite le nombre d'images et de PDF que chaque requête peut transporter. Pour les valeurs actuelles, voir [Request limits](https://platform.claude.com/docs/en/build-with-claude/vision#request-limits) dans la documentation de l'API. Claude Code plafonne aussi la **taille totale** des images et PDF d'une requête : de grandes captures d'écran atteignent donc la limite avec moins d'images que de petites.

Quand la requête suivante dépasserait l'une ou l'autre limite, Claude Code retire **un lot** des images et PDF les plus anciens de ce qu'il envoie, ce qui laisse de la place pour d'autres avant de devoir en retirer à nouveau. Claude ne peut plus voir les images retirées. Si Claude a de nouveau besoin de l'une d'elles, partagez-la à nouveau.

Retirer des images modifie les messages qui les contenaient : la requête suivante retraite donc la conversation à partir du plus ancien de ces messages. Comme Claude Code retire un lot à la fois, vous constatez un tour plus lent par lot plutôt qu'à chaque nouvelle capture d'écran.

### Mettre à jour Claude Code

Une nouvelle version de Claude Code met généralement à jour le prompt système ou les définitions d'outils : la première conversation démarrée après une mise à jour construit donc son cache depuis le début. La [mise à jour automatique](https://code.claude.com/docs/en/setup#auto-updates) télécharge les nouvelles versions en arrière-plan mais ne les applique qu'au lancement suivant, **jamais** en cours de session : vous le constatez donc comme un premier tour non mis en cache après un redémarrage, et non comme une surprise en cours de session. Définissez `DISABLE_AUTOUPDATER=1` pour contrôler le moment où les mises à jour s'appliquent.

> **Note** : pour le coût de la reprise d'une conversation démarrée avant la mise à jour, voir [Reprendre une session](#reprendre-une-session).

---

## Actions qui conservent le cache

Ces actions soit ajoutent du contenu à la fin de la conversation, soit ne touchent pas du tout à la requête. Certaines, comme la modification de CLAUDE.md, conservent le cache pour la même raison que le changement n'atteint pas la session en cours avant `/clear`, `/compact` ou un redémarrage.

* [Modifier des fichiers de votre dépôt](#modifier-des-fichiers-de-votre-dépôt)
* [Modifier CLAUDE.md en cours de session](#modifier-claudemd-en-cours-de-session)
* [Changer de mode de permission](#changer-de-mode-de-permission)
* [Changer de style de sortie](#changer-de-style-de-sortie)
* [Invoquer des skills et des commandes](#invoquer-des-skills-et-des-commandes)
* [Lancer `/recap`](#lancer-recap)
* [Revenir en arrière dans la conversation](#revenir-en-arrière-dans-la-conversation-rewind)
* [Lancer un sous-agent](#sous-agents-et-cache)

### Modifier des fichiers de votre dépôt

Le contenu des fichiers n'entre dans le contexte que lorsque Claude les lit, et ces lectures sont ajoutées à la conversation. Modifier un fichier que Claude a lu auparavant ne change pas rétroactivement la lecture antérieure dans l'historique. À la place, Claude Code ajoute un [`<system-reminder>`](https://code.claude.com/docs/en/glossary#system-reminder) signalant que le fichier a changé, et Claude le relit si nécessaire.

### Modifier CLAUDE.md en cours de session

Vos fichiers CLAUDE.md à la racine du projet et au niveau utilisateur sont lus **une seule fois** au démarrage de la session et conservés en mémoire. Les modifier en cours de session n'invalide pas le cache, mais la modification **ne s'applique pas non plus** : Claude continue de travailler avec la version chargée au démarrage. Le nouveau contenu est chargé au prochain `/clear`, `/compact` ou redémarrage.

Les [fichiers CLAUDE.md imbriqués dans des sous-répertoires](https://code.claude.com/docs/en/memory) et les [règles avec frontmatter `paths:`](https://code.claude.com/docs/en/memory#path-specific-rules) se chargent plus tard, à la demande. Les modifier vous-même **avant** leur chargement prend bien effet. Une fois chargés, leur contenu fait partie de l'historique de conversation : une modification en cours de session ne le change donc pas rétroactivement.

### Changer de mode de permission

Basculer entre [modes de permission](https://code.claude.com/docs/en/permission-modes), par exemple de Manual à accept edits, ne change ni le prompt système ni les définitions d'outils : les changements de mode sont donc sans danger pour le cache. L'exception est le mode plan avec le réglage de modèle [`opusplan`](https://code.claude.com/docs/en/model-config#opusplan-model-setting), qui bascule le modèle entre Opus et Sonnet quand vous entrez dans le mode plan ou en sortez. La bascule de mode devient alors un [changement de modèle](#changer-de-modèle).

### Changer de style de sortie

Quand vous changez de [style de sortie](https://code.claude.com/docs/en/output-styles) en cours de session avec [`/output-style`](https://code.claude.com/docs/en/output-styles#change-your-output-style), `/config` ou le réglage `outputStyle`, Claude utilise le nouveau style à partir de votre message suivant. Claude Code transmet les instructions du nouveau style sous forme de message dans la conversation : cette requête lit donc toujours le prompt système et la conversation antérieure depuis le cache.

Avant la v2.1.251, un changement de style en cours de session conservait le cache mais ne s'appliquait pas avant `/clear` ou une nouvelle session.

### Invoquer des skills et des commandes

Les [skills](https://code.claude.com/docs/en/skills) et les [commandes](https://code.claude.com/docs/en/commands) injectent leurs instructions sous forme de messages utilisateur au moment de l'invocation. Rien de ce qui précède dans la conversation ne change. Une skill ou une commande dont le frontmatter indique un `model` peut constituer un [changement de modèle](#changer-de-modèle) pour ce tour.

### Lancer `/recap`

[`/recap`](https://code.claude.com/docs/en/interactive-mode#session-recap) génère un résumé affiché dans votre terminal. Contrairement à `/compact`, il ajoute le résumé comme sortie de commande au lieu de remplacer votre historique de messages : le préfixe en cache reste donc intact.

### Revenir en arrière dans la conversation (rewind)

[`/rewind`](https://code.claude.com/docs/en/checkpointing) tronque votre conversation jusqu'à un tour antérieur. L'historique restant est le même contenu que celui à partir duquel le cache avait été construit à ce moment-là, et les couches prompt système et contexte du projet sont inchangées : la requête suivante touche donc l'entrée de cache antérieure. Chaque tour depuis lors a relu ce préfixe, ce qui a maintenu l'entrée chaude même si le tour d'origine date de plus longtemps que le TTL.

Restaurer les points de contrôle de fichiers (file checkpoints) en même temps que la conversation n'a pas d'effet séparé sur le cache. Le contenu des fichiers n'entre dans le contexte que lorsque Claude les lit, comme pour la [modification de fichiers de votre dépôt](#modifier-des-fichiers-de-votre-dépôt).

---

## Reprendre une session

Quand vous [reprenez une session](https://code.claude.com/docs/en/sessions#resume-a-session), Claude Code renvoie toute la conversation, et la requête lit depuis le cache la partie de son préfixe qui est inchangée et encore dans la [durée de vie du cache](#durée-de-vie-du-cache). Le tableau des couches en haut de cette page indique ce qui change chaque couche.

Le prompt système changerait après une [mise à jour de Claude Code](#mettre-à-jour-claude-code) ou avec un texte [`--append-system-prompt`](https://code.claude.com/docs/en/cli-reference#system-prompt-flags) différent à la reprise. Par défaut, la conversation reprise **conserve le prompt système avec lequel elle a démarré**, de sorte que son historique reste derrière le même prompt ; le changement prend effet une fois la conversation compactée, ou dans une nouvelle conversation. La section [System prompt flags in resumed conversations](https://code.claude.com/docs/en/cli-reference#system-prompt-flags-in-resumed-conversations) couvre les cas où Claude Code reconstruit plutôt le prompt à chaque requête.

---

## Durée de vie du cache

Les préfixes en cache expirent après une période d'inactivité. Chaque requête qui touche le cache réinitialise le minuteur : le cache reste chaud tant que vous continuez à travailler. Après une interruption suffisamment longue, la requête suivante recalcule toute l'entrée et rétablit le cache, ce qui explique pourquoi le premier tour après une absence peut être nettement plus lent.

Avec un forfait **Pro ou Max**, quand vous reprenez une grosse session après une longue pause, Claude Code [propose de reprendre à partir d'un résumé](https://code.claude.com/docs/en/sessions#resume-from-a-summary) afin que les requêtes suivantes ne transportent pas tout l'historique.

La durée de vie (TTL, *time to live*) détermine la durée d'interruption que le cache supporte. L'API en propose deux :

* un TTL de **cinq minutes** ;
* un [TTL d'**une heure**](https://platform.claude.com/docs/en/build-with-claude/prompt-caching#1-hour-cache-duration), qui garde le cache chaud pendant des pauses plus longues mais [facture un tarif plus élevé pour les écritures en cache](https://platform.claude.com/docs/en/build-with-claude/prompt-caching#pricing).

Le TTL plus long est utile quand vous laissez une session inactive puis y revenez, car vous évitez le retraitement qu'entraîne un préfixe expiré. Il coûte plus cher sur de courtes rafales de travail qui ne restent jamais inactives plus de cinq minutes : le tarif d'écriture plus élevé s'applique et la durée de vie plus longue n'est pas exploitée.

### Quel TTL reçoit chaque requête

Claude Code décide du TTL requête par requête, et chaque requête tombe dans l'un de deux « compartiments » (buckets) fixes :

* **Conversation principale** : vos tours interactifs, les exécutions non interactives `-p` et les tours Agent SDK, ainsi que les requêtes d'assistance que Claude Code exécute en ligne avec eux.
* **Tout le reste** : les requêtes que Claude Code émet en dehors de cette conversation, comme les [sous-agents](https://code.claude.com/docs/en/sub-agents), les [workflows](https://code.claude.com/docs/en/workflows), les [coéquipiers](https://code.claude.com/docs/en/agent-teams) en processus (in-process teammates), les forks, le compactage et les titres de session.

Sauf si vous choisissez vous-même un TTL, Claude Code ne demande le TTL d'une heure **que sur un abonnement Claude, dans la limite de l'usage inclus de votre forfait**. Il demande alors l'heure pour la conversation principale, plus un petit ensemble de requêtes d'assistance qu'Anthropic contrôle côté serveur. Ce tableau donne le TTL par défaut de chaque compartiment selon les deux modes de facturation.

| Compartiment de requêtes | Abonnement Claude, dans l'usage du forfait | Crédits d'usage, clé API ou fournisseur cloud |
| - | - | - |
| Conversation principale | Une heure | Cinq minutes |
| Tout le reste | Cinq minutes, sauf les requêtes d'assistance contrôlées par le serveur, qui reçoivent une heure | Cinq minutes |

Une fois que vous dépassez la limite d'usage de votre forfait et que Claude Code puise dans les [crédits d'usage (usage credits)](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans), cet usage vous est facturé : Claude Code ramène donc la conversation principale au TTL de cinq minutes, dont le tarif d'écriture en cache est plus bas. Pour conserver le TTL d'une heure dans ce cas, [choisissez le TTL vous-même](#choisir-le-ttl-vous-même).

### Choisir le TTL vous-même

Vous pouvez définir un TTL pour chaque compartiment. Chaque contrôle accepte `5m` ou `1h` ; Claude Code ignore toute autre valeur.

* **Conversation principale** : le réglage [`promptCacheTtl`](https://code.claude.com/docs/en/settings-reference#promptcachettl), ou la [variable d'environnement](https://code.claude.com/docs/en/env-vars) `CLAUDE_CODE_PROMPT_CACHE_TTL`.
* **Tout le reste** : le réglage [`subagentPromptCacheTtl`](https://code.claude.com/docs/en/settings-reference#subagentpromptcachettl), ou la variable d'environnement `CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL`.

Les deux réglages et les deux variables d'environnement nécessitent la v2.1.242+. Si vous vous connectez avec une clé API ou utilisez un fournisseur cloud, définissez `promptCacheTtl` à `1h` pour donner à la conversation principale un cache d'une heure. Les requêtes hors de celle-ci gardent la valeur par défaut de cinq minutes tant que vous ne choisissez pas aussi un TTL pour ce compartiment.

Quand plusieurs contrôles s'appliquent, Claude Code retient la première correspondance dans cet ordre :

1. `FORCE_PROMPT_CACHING_5M=1`, qui force cinq minutes pour les deux compartiments.
2. La variable d'environnement du compartiment.
3. Le réglage du compartiment.
4. Pour les requêtes d'un sous-agent, la valeur `cacheTtl` dans le [champ de frontmatter `experimental`](https://code.claude.com/docs/en/sub-agents#supported-frontmatter-fields) du sous-agent (v2.1.248+). Claude Code ignore une valeur `1h` à cet endroit tant que votre abonnement Claude consomme des crédits d'usage.
5. `ENABLE_PROMPT_CACHING_1H=1`, qui demande une heure pour les deux compartiments.
6. La [valeur par défaut du compartiment de la requête](#quel-ttl-reçoit-chaque-requête).

Définissez `FORCE_PROMPT_CACHING_5M=1` quand vous déboguez le comportement du cache, comparez les deux TTL, ou voulez surcharger un TTL plus long défini dans les [réglages gérés (managed settings)](https://code.claude.com/docs/en/managed-settings).

Pour vérifier quel TTL ont utilisé les écritures en cache de votre conversation principale, lancez la commande ci-dessous et lisez `usage.cache_creation` dans le résultat :

```bash
claude -p "hello" --output-format json
```

Claude Code rapporte les écritures en cache d'une heure sous `ephemeral_1h_input_tokens` et celles de cinq minutes sous `ephemeral_5m_input_tokens`.

Remarques selon le mode d'accès :

* **Passerelle LLM définie avec `ANTHROPIC_BASE_URL`** : une partie de la demande d'une heure transite dans l'en-tête `anthropic-beta` ; configurez donc la passerelle pour [transmettre cet en-tête sans modification](https://code.claude.com/docs/en/llm-gateway-protocol#request-headers).
* **[Passerelle Claude apps (Claude apps gateway)](https://code.claude.com/docs/en/claude-apps-gateway#availability-and-limitations)** : le TTL d'une heure n'y est **pas disponible**.
* **Amazon Bedrock** : la prise en charge de la mise en cache des prompts, la longueur minimale de préfixe pouvant être mise en cache et la disponibilité du TTL d'une heure varient toutes selon le modèle. Si les compteurs de tokens en cache restent à zéro, consultez [supported models, regions, and limits](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-caching.html#prompt-caching-models) dans la documentation d'Amazon Bedrock.

---

## Portée du cache

Dans Claude Code, le cache est en pratique limité **à une machine et à un répertoire**. Le prompt système intègre les chemins de votre mémoire automatique, et la conversation s'ouvre sur l'annonce du répertoire de travail, de la plateforme, du shell et de la version du système d'exploitation. Deux sessions dans des répertoires différents construisent donc des préfixes différents et ne profitent pas du cache l'une de l'autre.

Les sessions lancées **en parallèle dans le même répertoire** construisent des préfixes identiques et lisent le cache l'une de l'autre. Les sessions **successives** ne partagent le préfixe que si l'instantané du statut git pris au démarrage est identique, car chaque conversation transporte aussi la branche et les commits récents issus de cet instantané.

Le cache sous-jacent de l'API est plus large. Les caches sont isolés entre organisations et, chez certains fournisseurs, [entre espaces de travail (workspaces) d'une même organisation](https://platform.claude.com/docs/en/build-with-claude/prompt-caching#cache-storage-and-sharing). À l'intérieur de ces frontières, deux requêtes quelconques avec le même modèle et le même préfixe lisent le même cache. Pour les appelants de l'Agent SDK qui font tourner des flottes de processus automatisés, voir [improve prompt caching across users and machines](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts#improve-prompt-caching-across-users-and-machines) pour sortir l'emplacement de la mémoire automatique du prompt système et partager l'entrée de cache du prompt système entre utilisateurs et machines.

---

## Vérifier les performances du cache

Les performances du cache apparaissent sous forme de deux compteurs de tokens que l'API renvoie à chaque réponse. Le moyen le plus direct de les suivre en direct est un [script de ligne de statut (statusline)](https://code.claude.com/docs/en/statusline) qui lit l'objet `current_usage` :

| Champ | Signification |
| - | - |
| `cache_creation_input_tokens` | Tokens écrits dans le cache lors de ce tour, facturés au tarif d'écriture en cache |
| `cache_read_input_tokens` | Tokens servis depuis le cache lors de ce tour, facturés au [tarif des tokens en cache](https://platform.claude.com/docs/en/about-claude/pricing) du modèle, inférieur au tarif d'entrée standard |

Un rapport lecture/création élevé signifie que le cache fonctionne bien. Si la création reste élevée tour après tour, quelque chose change dans votre préfixe. La section [Actions qui invalident le cache](#actions-qui-invalident-le-cache) liste les causes habituelles.

Pour un résumé par session, lancez `/usage`. Après la première réponse de la conversation principale, Claude Code ajoute une [ligne `Prompt cache (main)`](https://code.claude.com/docs/en/costs#prompt-cache-statistics) au bloc Session, qui indique le taux de réussite de la session, le nombre d'échecs (misses) et si le cache est chaud à cet instant. Un script de ligne de statut peut lire les mêmes chiffres depuis l'[objet `prompt_cache`](https://code.claude.com/docs/en/statusline#prompt-cache-fields). Les deux nécessitent la v2.1.251+.

La ligne `Prompt cache (main)` indique aussi la cause probable du dernier échec quand Claude Code peut l'identifier, par exemple `likely cause: tool definitions changed`. Ce texte de cause probable nécessite la v2.1.260+.

Pour une visibilité à l'échelle d'une organisation, l'exportateur OpenTelemetry rapporte les tokens de lecture et de création du cache par utilisateur et par session. Voir [Monitor usage](https://code.claude.com/docs/en/monitoring-usage) pour la référence des métriques et des attributs d'événements.

---

## Sous-agents et cache

Un [sous-agent](https://code.claude.com/docs/en/sub-agents) démarre sa propre conversation avec son propre prompt système et son propre jeu d'outils, distincts de ceux du parent. Sa première requête ne lit pas le cache du parent, car les deux préfixes diffèrent, et il chauffe son propre cache au fil de ses tours. Les sous-agents se situent hors du [compartiment TTL](#quel-ttl-reçoit-chaque-requête) de la conversation principale : ils reçoivent donc cinq minutes, même sur un abonnement, tant que vous n'en [choisissez pas un plus long](#choisir-le-ttl-vous-même).

Le cache du parent n'est pas affecté. Du point de vue du parent, l'appel du sous-agent et son résultat s'ajoutent à la conversation, ce qui laisse le préfixe du parent intact.

Un [fork](https://code.claude.com/docs/en/sub-agents#fork-the-current-conversation), en revanche, hérite exactement du prompt système, des outils et de l'historique de conversation du parent : sa première requête lit donc le cache du parent.

D'autres requêtes peuvent aussi lire un préfixe mis en cache par une requête antérieure :

* **Copies de session** : une session que vous [copiez avec `/fork`](https://code.claude.com/docs/en/agent-view#copy-the-session-with-%2Ffork) reçoit son instruction d'isolement sous forme de message à la fin de la conversation copiée, de sorte que le cache construit par la conversation d'origine reste intact.
* **Compactage** : l'appel de résumé décrit dans [Compacter la conversation](#compacter-la-conversation) utilise la même approche de partage de préfixe.
* **Sous-agents repris** : quand Claude [reprend un sous-agent](https://code.claude.com/docs/en/sub-agents#resume-subagents), la première requête de l'exécution reprise peut lire le cache chauffé par l'exécution d'origine.
* **Fan-outs de workflow** : dans un [fan-out de workflow](https://code.claude.com/docs/en/workflows#prompt-caching-in-a-fan-out) d'agents partageant le même préfixe, Claude Code retient tous les agents sauf le premier pendant **jusqu'à 5 secondes** par défaut, afin que leurs premières requêtes puissent lire le préfixe mis en cache par le premier agent.

---

## Désactiver la mise en cache des prompts

Désactiver le cache est parfois utile pour déboguer le comportement du cache avec un modèle ou un fournisseur précis. Pour le désactiver, définissez l'une de ces variables d'environnement à `1` :

| Variable | Effet |
| - | - |
| `DISABLE_PROMPT_CACHING` | Désactive pour tous les modèles |
| `DISABLE_PROMPT_CACHING_HAIKU` | Désactive pour le modèle Haiku par défaut |
| `DISABLE_PROMPT_CACHING_SONNET` | Désactive pour le modèle Sonnet par défaut |
| `DISABLE_PROMPT_CACHING_OPUS` | Désactive pour le modèle Opus par défaut |
| `DISABLE_PROMPT_CACHING_FABLE` | Désactive pour Fable uniquement |

`DISABLE_PROMPT_CACHING_HAIKU` s'applique au modèle Haiku par défaut, c'est-à-dire le modèle vers lequel se résout l'alias `haiku`. Elle désactive le cache partout où ce modèle s'exécute, y compris la conversation principale lorsqu'il est votre modèle principal. La couverture de la conversation principale nécessite la v2.1.283+.

Cette variable couvre aussi un modèle d'arrière-plan défini avec la variable obsolète (deprecated) `ANTHROPIC_SMALL_FAST_MODEL`, lorsque ce modèle diffère de votre modèle principal.

Une autre version de Haiku que vous épinglez comme modèle principal **conserve** le cache ; définissez `DISABLE_PROMPT_CACHING` pour le désactiver pour elle.

`DISABLE_PROMPT_CACHING_SONNET` et `DISABLE_PROMPT_CACHING_OPUS` s'appliquent chacune au modèle vers lequel se résout l'alias `sonnet` ou `opus`. Si vous définissez un autre identifiant de modèle Sonnet ou Opus comme modèle principal, ce modèle conserve le cache. Par exemple, une session sur `claude-sonnet-5` conserve le cache alors que `sonnet` se résout en `claude-sonnet-5-5`. Pour désactiver le cache pour ce modèle, définissez `DISABLE_PROMPT_CACHING`.

Pour définir une politique de cache à l'échelle d'une organisation, placez l'une de ces variables ou les [variables de TTL](#durée-de-vie-du-cache) dans le bloc `env` des [réglages gérés (managed settings)](https://code.claude.com/docs/en/managed-settings). Pour un usage normal, laissez le cache activé.

---

## Ressources associées

* [Lessons from building Claude Code: Prompt caching is everything](https://claude.com/blog/lessons-from-building-claude-code-prompt-caching-is-everything) : la justification de conception du mode plan, du chargement différé des outils et du compactage.
* [Explore the context window](https://code.claude.com/docs/en/context-window) : ce qui est chargé dans le contexte, et quand.
* [Reduce token usage](https://code.claude.com/docs/en/costs#reduce-token-usage) : stratégies au-delà du cache pour gérer la taille du contexte.
* [Track and reduce costs](https://code.claude.com/docs/en/agent-sdk/cost-tracking) : suivi des tokens en cache et configuration du TTL pour les appelants de l'Agent SDK.
* [Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) : le mécanisme API sous-jacent, les points d'arrêt (breakpoints) et la tarification.
