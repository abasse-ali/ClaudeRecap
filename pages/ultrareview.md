# Trouver des bugs avec ultrareview

> Source : https://code.claude.com/docs/en/ultrareview · [← Index](../31-index-documentation-officielle.md)

*Lancer une revue de code approfondie et multi-agents dans le cloud avec `/code-review ultra` pour trouver et vérifier des bugs avant de fusionner.*

> **Note** : ultrareview est une fonctionnalité en aperçu de recherche (*research preview*). La fonctionnalité, sa tarification et sa disponibilité peuvent évoluer selon les retours. La commande est `/code-review ultra`. Quand ultrareview est disponible pour votre compte, `/ultrareview` en est un alias.

Ultrareview est une revue de code approfondie qui s'exécute comme une [session cloud](https://code.claude.com/docs/en/claude-code-on-the-web) sur l'infrastructure d'Anthropic. Quand vous lancez `/code-review ultra`, Claude Code déploie une flotte d'agents relecteurs dans un sandbox cloud pour trouver des bugs dans votre branche ou votre pull request.

Par rapport à un `/code-review` local, ultrareview offre :

* **Un signal plus fiable** : chaque problème signalé est reproduit et vérifié de façon indépendante, si bien que les résultats se concentrent sur de vrais bugs plutôt que sur des suggestions de style
* **Une couverture plus large** : une flotte plus grande d'agents relecteurs explore la modification en parallèle, ce qui fait ressortir des problèmes qu'une revue locale peut manquer
* **Aucune ressource locale utilisée** : la revue s'exécute entièrement dans un sandbox cloud, votre terminal reste donc libre pour d'autres travaux pendant ce temps

**Conditions de disponibilité :**

* Ultrareview nécessite une authentification avec un compte claude.ai, car elle s'exécute comme session cloud sur l'infrastructure d'Anthropic. Si vous êtes connecté uniquement avec une clé API, lancez `/login` et authentifiez-vous d'abord avec claude.ai.
* Ultrareview n'est pas disponible quand Claude Code est utilisé avec Amazon Bedrock, Google Cloud's Agent Platform ou Microsoft Foundry.
* Elle n'est pas disponible pour les organisations qui ont activé la Zero Data Retention ou auxquelles la [configuration HIPAA](https://code.claude.com/docs/en/hipaa-setup) est appliquée.
* Quand ultrareview n'est pas disponible, `/code-review ultra` lance à la place une revue locale dans votre session.

## Lancer ultrareview depuis la CLI

Démarrez une revue depuis n'importe quel dépôt git :

```text theme={null}
/code-review ultra
```

Sans argument, ultrareview examine le diff entre votre branche courante et la branche par défaut, y compris les modifications non commitées et indexées (*staged*).

Pour la revue d'une branche, Claude Code empaquette l'état du dépôt et le téléverse dans un sandbox cloud selon les règles de [téléversement d'un dépôt local vers une session cloud](https://code.claude.com/docs/en/claude-code-on-the-web#send-local-repositories-without-github), qui couvrent les limites de taille, les exigences sur l'extraction (*checkout*) et ce qu'il advient des modifications non commitées dans des fichiers nommés comme des identifiants ou des clés, tels que les fichiers `.env` et `*.tfvars`. Quand vous [examinez une pull request](#examiner-une-pull-request), Claude Code ne téléverse rien depuis votre machine.

Avant le lancement, Claude Code affiche une boîte de dialogue de confirmation avec le périmètre de la revue, le nombre de revues gratuites restantes et le coût estimé ; pour une revue de branche, le périmètre inclut le nombre de fichiers et de lignes. Une fois confirmée, la revue se poursuit en arrière-plan pendant que vous continuez à utiliser votre session.

La commande ne s'exécute que lorsque vous l'invoquez avec `/code-review ultra` ; Claude ne démarre jamais une ultrareview de lui-même.

### Comparer à une autre base

Pour comparer à une base autre que la branche par défaut, passez le nom de la branche. Cet exemple examine votre branche courante par rapport à `develop` :

```text theme={null}
/code-review ultra develop
```

La branche de base n'a pas besoin d'exister dans votre clone local ; Claude Code la récupère depuis `origin`. Si le nom contient une faute de frappe, Claude Code suggère dans l'erreur le nom de branche le plus proche.

Un identifiant de commit ou un tag fonctionne aussi comme base : la revue couvre alors les modifications de votre branche depuis ce commit.

### Examiner une pull request

Pour examiner une pull request sur `github.com` plutôt qu'une branche locale, passez le numéro de la PR :

```text theme={null}
/code-review ultra 1234
```

La commande accepte aussi `#1234`, `PR 1234` et les URL de PR collées ; une URL collée doit pointer vers le dépôt de votre répertoire courant.

Le mode PR nécessite un dépôt sur `github.com`. Pour un dépôt sur une instance [GitHub Enterprise Server](https://code.claude.com/docs/en/github-enterprise-server), lancez `/code-review ultra` sans numéro de PR pour examiner plutôt votre branche locale.

En mode PR, le sandbox cloud clone la pull request depuis `github.com` au lieu de téléverser votre arbre de travail. Il utilise le compte GitHub connecté à votre compte Claude : ce compte doit donc avoir un accès en lecture au dépôt.

Lancez [`/web-setup`](https://code.claude.com/docs/en/web-quickstart#connect-from-your-terminal) pour connecter votre connexion GitHub CLI à votre compte Claude.

### Publier les résultats sur la pull request

À partir de Claude Code v2.1.227, quand vous examinez une pull request sur `github.com`, vous pouvez demander à Claude de publier les résultats finaux sur la PR sous forme d'un unique commentaire simple, depuis votre propre compte GitHub. Ce commentaire n'est ni une revue ni une approbation, et il se termine par une mention « Generated by Claude Code ». Quand vous examinez une branche, Claude Code affiche les résultats uniquement dans votre session.

Claude Code ne publie jamais sans que vous l'ayez choisi pour cette exécution, et `--no-post` est le comportement par défaut. Publier est un choix que vous faites à chaque exécution :

* **En interactif** : dans la boîte de dialogue de lancement, sélectionnez **Run and post the findings to the PR as me**. Si vous ajoutez `--post` à la commande, comme dans `/code-review ultra 1234 --post`, Claude Code présélectionne ce choix et demande quand même confirmation avant le lancement.
* **En non interactif** : lancez la [sous-commande `claude ultrareview`](#lancer-ultrareview-en-non-interactif) avec `--post`. Vous consentez à la publication en lançant la sous-commande avec ce flag : Claude Code publie donc sans demander. Dans une exécution `claude -p '/code-review ultra'`, Claude Code se termine avant l'arrivée des résultats et ne publie donc rien ; utilisez plutôt la sous-commande.

Claude Code ne publie pas depuis votre machine. Il envoie l'identifiant de session de la revue à l'API Anthropic, qui publie les résultats stockés de la revue comme commentaire via le compte GitHub que vous avez connecté à Claude. La publication exige la même connexion claude.ai que la revue elle-même, et elle n'est pas disponible avec les fournisseurs tiers ni lorsque vous définissez [`CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`](https://code.claude.com/docs/en/env-vars).

Dans une session interactive, Claude Code lance la publication à l'arrivée des résultats : gardez donc la session ouverte jusqu'à la fin de la revue. Claude Code ne conserve le choix de publication que dans cette session. Si la session se termine avant la fin de la revue, Claude Code ne publie rien, même si vous reprenez la conversation plus tard.

Une fois la publication terminée, Claude vous indique le résultat :

* **Publié** (*Posted*) : Claude vous donne un lien vers le commentaire.
* **Déjà publié** (*Already posted*) : une publication antérieure de la même revue a déjà placé le commentaire sur la PR ; Claude vous renvoie alors vers la pull request au lieu de publier à nouveau.
* **Échec** (*Failed*) : Claude vous indique pourquoi, et les résultats restent dans votre terminal pour que vous puissiez les publier à la main.

### Formuler une demande en langage courant

À partir de Claude Code v2.1.218, vous pouvez aussi décrire en langage courant ce sur quoi vous travaillez :

```text theme={null}
/code-review ultra check my auth changes
```

La revue couvre toujours votre branche courante, soit le même périmètre que sans argument. Claude conserve votre texte comme une note, affichée dans la boîte de dialogue de lancement, et rattache les résultats à cette note quand ils arrivent.

Claude Code ne traite votre texte comme une note que s'il compte plus d'un mot et n'est ni un nom de branche ni une référence de PR. Il interprète un mot unique comme un nom de branche ou une référence de PR : un nom de branche mal orthographié produit donc l'erreur « branche la plus proche » de [Comparer à une autre base](#comparer-à-une-autre-base) au lieu d'un lancement avec une note. Si votre texte combine une référence de PR et d'autres mots, comme `check PR 123 again`, Claude Code ne lance rien non plus ; il vous demande de relancer avec le seul numéro de PR pour examiner cette PR, ou sans la référence pour examiner votre branche courante.

> **Astuce** : si votre dépôt est trop volumineux pour être empaqueté, Claude Code vous propose d'utiliser plutôt le mode PR. Pour un dépôt sur `github.com`, poussez votre branche et ouvrez une PR en brouillon (*draft*), puis lancez `/code-review ultra <PR-number>`.

### Limites du diff et solutions de repli

Ultrareview vérifie le diff avant tout travail de revue et vous prévient lorsqu'il ne peut pas l'examiner tel quel :

* **Diff trop volumineux** : une revue de branche peut inclure par défaut jusqu'à 500 fichiers modifiés et 8 000 lignes modifiées. Les valeurs exactes peuvent changer, et le [refus](https://code.claude.com/docs/en/errors#diff-is-too-large-for-ultrareview) indique celles en vigueur, la taille de votre diff et les fichiers ayant le plus de lignes modifiées. Claude Code refuse de la même façon une pull request trop volumineuse, en indiquant ses nombres de fichiers et de lignes mais sans la ventilation par fichier
* **Rien à examiner** : quand le diff par rapport à la base est vide, ultrareview refuse et indique la branche ou le commit de comparaison ainsi que votre situation — par exemple être sur la branche de base elle-même sans rien de non commité, ou sur une branche dont tous les commits font déjà partie de la base. Il suggère aussi la manière d'en sortir pour ce cas : basculer sur la branche contenant votre travail, indexer ou commiter les modifications locales, ou passer une autre base
* **Premier commit** : le premier commit d'un dépôt n'a rien d'antérieur avec quoi se comparer ; ultrareview examine donc tous ses fichiers après votre confirmation dans la boîte de dialogue de lancement. Si vous avez des fichiers non suivis (*untracked*), il refuse plutôt et vous demande de faire `git add` sur ceux que vous voulez faire examiner. Les mêmes limites de taille s'appliquent.

  Un premier commit n'est examiné en entier qu'après cette confirmation : la sous-commande `claude ultrareview` et `claude -p` le refusent donc et vous renvoient vers une session interactive. Nécessite Claude Code v2.1.277 ou ultérieur
* **Pas de base de fusion** (*merge base*) : quand votre branche ne partage aucun historique avec la branche de base, ou que le dépôt n'a pas de branche de base avec laquelle comparer, ultrareview examine à la place tous les fichiers suivis du dépôt. Ce repli nécessite un clone complet et applique les mêmes limites de taille. Il ne se lance que si vous confirmez dans la boîte de dialogue de lancement ou lancez vous-même la sous-commande `claude ultrareview`. Dans `claude -p`, et partout ailleurs où aucune de ces deux conditions n'est remplie, ultrareview refuse, indique que la revue couvrirait tous les fichiers et vous renvoie vers une session interactive.

  Sur une extraction sans branche ni autre référence, comme un HEAD détaché créé en extrayant `FETCH_HEAD` après avoir récupéré une URL, Claude Code [refuse la revue](https://code.claude.com/docs/en/errors#your-checkout-has-no-branches) et suggère de créer d'abord une branche

## Tarification et revues gratuites

Ultrareview est une fonctionnalité premium facturée sur les crédits d'utilisation (*usage credits*) plutôt que sur l'utilisation incluse dans votre forfait.

| Forfait | Revues gratuites incluses | Après les revues gratuites |
| - | - | - |
| Pro | 3 revues gratuites | facturé en [crédits d'utilisation](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans) |
| Max | 3 revues gratuites | facturé en [crédits d'utilisation](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans) |

* **Revues gratuites** : les trois revues Pro et Max sont une dotation unique par compte et ne se renouvellent pas.
* **Coût par revue** : une fois les revues gratuites utilisées, généralement de 5 $ à 25 $ en crédits d'utilisation selon la taille de la modification, conformément à l'estimation affichée par la boîte de dialogue de lancement avant chaque exécution.
* **Quand une exécution est décomptée** : dès que la session cloud démarre. Une revue que vous arrêtez tôt ou qui n'aboutit pas consomme quand même une revue gratuite ; une revue payante n'est facturée que pour la partie effectivement exécutée.

Comme ultrareview est toujours facturée en crédits d'utilisation en dehors des revues gratuites, votre compte ou votre organisation doit avoir activé les crédits d'utilisation avant de pouvoir lancer une revue payante. Si ce n'est pas le cas, Claude Code bloque le lancement. Si vous pouvez gérer la facturation de votre compte, Claude Code vous renvoie vers les réglages de facturation où vous pouvez activer les crédits d'utilisation.

Vous pouvez aussi lancer `/usage-credits` pour consulter ou modifier votre réglage de crédits d'utilisation.

Claude Code vous demande de confirmer la facturation en crédits d'utilisation une fois par conversation : quand vous démarrez une nouvelle conversation, par exemple avec `/clear`, Claude Code affiche à nouveau la confirmation pour la prochaine revue payante.

## Suivre une revue en cours

Une revue prend généralement de 15 à 25 minutes. Elle s'exécute comme une tâche d'arrière-plan : vous pouvez continuer à travailler dans votre session ou lancer d'autres commandes. Si vous avez choisi de [publier les résultats sur la pull request](#publier-les-résultats-sur-la-pull-request), gardez la session ouverte jusqu'à la fin de la revue ; si la session se termine avant, Claude Code ne publie rien.

Utilisez `/tasks` pour voir les revues en cours et terminées, ouvrir la vue détaillée d'une revue, ou arrêter une revue en cours. Si vous arrêtez une revue, Claude Code archive la session cloud et ne renvoie pas de résultats partiels.

Claude peut aussi vous indiquer qu'une revue a été arrêtée ou que sa session est introuvable :

* Si la session cloud de la revue est arrêtée ou [archivée](https://code.claude.com/docs/en/claude-code-on-the-web#archive-sessions) sur claude.ai avant la fin de la revue, Claude vous indique qu'elle a été arrêtée.
* Si la session cloud de la revue a été supprimée, ou si vous vous êtes connecté à un autre compte ou une autre organisation Claude depuis son lancement, Claude vous indique que la session est introuvable.
* Si vous avez changé de compte, la revue peut tout de même se terminer sous le compte qui l'a lancée. Si la revue est toujours en cours, reconnectez-vous avec ce compte et reprenez la conversation avec `claude --resume` pour la rattacher.

Quand la revue se termine, Claude Code affiche les résultats vérifiés sous forme de notification dans votre session. Chaque résultat indique l'emplacement dans le fichier et une explication du problème, pour que vous puissiez demander directement à Claude de le corriger.

## Lancer ultrareview en non interactif

Utilisez la sous-commande `claude ultrareview` pour démarrer une ultrareview depuis la CI ou un script, sans session interactive. La sous-commande lance la même revue que `/code-review ultra`, reste bloquée jusqu'à la fin de la revue distante, et affiche les résultats sur stdout.

```bash theme={null}
claude ultrareview
claude ultrareview 1234
claude ultrareview origin/main
```

Sans argument, la sous-commande examine le diff entre votre branche courante et la branche par défaut, avec le même [repli sur tout le dépôt](#limites-du-diff-et-solutions-de-repli) que `/code-review ultra` lorsqu'il n'existe pas de base de fusion. Passez un numéro de PR pour [examiner une pull request sur `github.com`](#examiner-une-pull-request), ou une branche de base pour comparer à celle-ci ; la [gestion de la branche de base](#comparer-à-une-autre-base) est identique à celle de la commande interactive.

Vous consentez au repli sur tout le dépôt ainsi qu'à l'invite de facturation et de conditions en lançant la sous-commande : l'exécution démarre donc sans attendre de saisie. C'est le fait de la lancer vous-même qui vaut consentement. Quand c'est Claude qui lance la sous-commande pour vous, par exemple via l'outil Bash, Claude Code refuse la revue de tout le dépôt.

`claude -p '/code-review ultra'` ne vous donne pas les résultats : utilisez donc `claude ultrareview` dans les scripts. L'exécution `-p` lance la revue cloud et se termine sans l'attendre. Quand la revue serait facturée en crédits d'utilisation, l'exécution `-p` s'arrête sans la lancer. Avant la v2.1.218, `/code-review ultra` dans une session non interactive lançait une revue locale.

`claude ultrareview` écrit ses messages de progression sur stderr pour que stdout reste analysable. Utilisez ces flags pour contrôler sa sortie, le délai d'expiration et la publication des résultats :

| Flag | Description |
| - | - |
| `--json` | Affiche la charge brute `bugs.json` au lieu des résultats mis en forme |
| `--timeout <minutes>` | Nombre maximal de minutes d'attente de la fin de la revue. 45 par défaut |
| `--post` | [Publie les résultats finaux](#publier-les-résultats-sur-la-pull-request) sur la pull request sous forme d'un unique commentaire simple depuis votre compte GitHub. Fonctionne pour les cibles de type pull request sur `github.com` ; pour les autres cibles, Claude Code ignore le flag et le signale. Nécessite Claude Code v2.1.227+ |
| `--no-post` | Ne publie pas les résultats. C'est le comportement par défaut, et si vous passez les deux flags, Claude Code ne publie pas. Nécessite Claude Code v2.1.227+ |

Lancer `claude ultrareview` exige la même authentification et la même configuration des crédits d'utilisation que `/code-review ultra`.

La sous-commande se termine avec l'un de ces trois codes :

* **0** : la revue s'est terminée, avec ou sans résultats
* **1** : la revue n'a pas pu se lancer ou a été arrêtée avant la fin, la session cloud a rencontré une erreur, ou le délai d'expiration est écoulé
* **130** : vous avez interrompu la sous-commande avec Ctrl-C

Si la sous-commande se termine avant l'arrivée des résultats, ceux-ci n'atteignent jamais votre terminal. La revue peut toujours être en cours dans le cloud. Relancer la sous-commande démarre une nouvelle revue au lieu de reprendre celle-là, et la nouvelle revue [consomme une revue gratuite ou est facturée en crédits d'utilisation](#tarification-et-revues-gratuites).

Avec `--post`, la sous-commande lance la publication juste après avoir affiché les résultats, et affiche le lien sur stderr.

* Si l'exécution échoue, est arrêtée ou expire, la sous-commande ne publie rien.
* Si la revue se termine mais que le commentaire n'est pas publié, Claude Code affiche la raison sur stderr, et les résultats restent sur stdout pour que vous puissiez les publier à la main.

Pour des revues automatiques des pull requests GitHub, [Code Review](https://code.claude.com/docs/en/code-review) s'intègre directement à votre dépôt et publie les résultats sous forme de commentaires de PR en ligne (*inline*), sans étape CLI.

## Comparaison entre ultrareview et /code-review

Les deux revues examinent du code, mais vous les utilisez à des étapes différentes de votre flux de travail.

| | `/code-review` | `/code-review ultra` |
| - | - | - |
| Cible | votre diff de travail, une pull request, une branche ou un chemin | votre diff de travail ou une pull request |
| Exécution | localement dans votre session | dans un sandbox cloud |
| Profondeur | proportionnelle à l'argument d'effort | flotte multi-agents avec vérification indépendante |
| Durée | de quelques secondes à quelques minutes | environ 15 à 25 minutes |
| Coût | compte dans l'utilisation normale | revues gratuites, puis environ 5 $ à 25 $ par revue en crédits d'utilisation |
| Idéal pour | un retour rapide pendant l'itération | de la confiance avant fusion sur des modifications conséquentes |

Utilisez `/code-review` pour un retour rapide pendant que vous travaillez, ou passez un numéro de PR pour examiner la pull request d'un collègue avant de l'approuver. Utilisez `/code-review ultra` avant de fusionner une modification conséquente, quand vous voulez une passe plus approfondie qui attrape les problèmes qu'une revue locale pourrait manquer.

## Ressources associées

* [Use Claude Code in the cloud](https://code.claude.com/docs/en/claude-code-on-the-web) : comprendre le fonctionnement des sessions cloud et des sandboxes cloud
* [Manage costs effectively](https://code.claude.com/docs/en/costs) : suivre l'utilisation et fixer des limites de dépenses
