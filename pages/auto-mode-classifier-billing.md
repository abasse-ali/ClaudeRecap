# Facturation des requêtes du classifieur du mode auto

> Source : https://code.claude.com/docs/en/auto-mode-classifier-billing · [← Index](../31-index-documentation-officielle.md)

Résoudre l'avis de Claude Code indiquant que cette session n'est pas éligible aux requêtes de classifieur gratuites du mode auto : ce qu'il signifie, pourquoi il apparaît et que faire.

En [mode auto](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode), un classifieur vérifie les actions comme les commandes shell et les requêtes réseau avant leur exécution. Partout où les [vérifications côté serveur sont actives](https://code.claude.com/docs/en/permission-modes#server-side-classifier-review), le serveur effectue ces vérifications dans le cadre des propres requêtes au modèle de la session, sans frais. Cet avis signifie que les vérifications du serveur n'atteignent pas votre session : Claude Code effectue donc ses propres requêtes de classifieur, et sur votre compte ces requêtes comptent dans votre consommation de jetons :

```text theme={null}
We're changing auto mode to no longer charge for classifier requests in Claude Code. However, this session isn't eligible.
```

(Traduction : « Nous modifions le mode auto pour ne plus facturer les requêtes de classifieur dans Claude Code. Cependant, cette session n'est pas éligible. »)

À l'invite, Claude Code met en attente la première action qu'il vérifierait de cette façon jusqu'à ce que vous répondiez. Rien n'est cassé : le mode auto continue de fonctionner, et ses requêtes de classifieur sont facturées comme avant. La cause la plus fréquente est une passerelle (gateway) ou un proxy LLM entre Claude Code et l'API ; quand Claude Code peut l'identifier, l'avis la nomme. Appuyez sur **Enter** pour continuer, ou voir [Rendre la session éligible](#rendre-la-session-éligible) pour qu'il n'apparaisse plus dans les nouvelles sessions.

## Répondre à l'avis

L'avis met l'action en attente jusqu'à votre réponse :

* **Enter** continue : l'action en attente et le reste de la session utilisent les propres requêtes de classifieur de Claude Code, facturées en consommation de jetons comme avant, et l'avis ne réapparaît pas dans cette session. Si l'avis nommait une passerelle, l'acquitter l'empêche de réapparaître sur cette machine pendant 24 heures. Sinon, l'avis revient la prochaine fois qu'une session bascule en repli (fallback).
* **Esc** ou **Ctrl+C** annule : l'action en attente n'est pas exécutée et le tour en cours s'arrête, la session restant en mode auto. Rien n'est mémorisé : l'avis réapparaît avant la prochaine action vérifiée.

Pour cesser d'utiliser le mode auto, changez de mode de permission avec `Shift+Tab` après avoir répondu.

Là où l'avis ne peut pas attendre de réponse, Claude Code signale le même texte et la session continue en mode auto, sauf si un acquittement de passerelle sur cette machine au cours des dernières 24 heures l'a écarté. En [mode non interactif](https://code.claude.com/docs/en/headless) avec `-p`, il écrit le texte sur stderr ; en sortie `stream-json`, il émet un message d'avertissement `system`, que les applications Agent SDK peuvent lire dans le flux de messages.

## Rendre la session éligible

Si une passerelle est en cause, demandez à l'administrateur de votre entreprise ou à votre fournisseur de passerelle de laisser passer les requêtes et réponses sans modification. Cela signifie transmettre les en-têtes et les champs du corps de requête tels quels, y compris ceux que la passerelle ne reconnaît pas comme le champ de requête `safeguards`, et renvoyer les réponses et événements de streaming sans supprimer de clés comme le champ `safeguard_results` ni réécrire les identifiants tool-use, comme le décrit le [guide de compatibilité des passerelles](https://code.claude.com/docs/en/llm-gateway-protocol#feature-pass-through). Une passerelle qui laisse passer le trafic de cette façon continue de fonctionner avec cette fonctionnalité et les suivantes. Les nouvelles sessions utilisent alors à nouveau les vérifications du serveur.

Si vous savez déjà que votre passerelle ne peut pas fournir les vérifications du serveur, dites à Claude Code de ne pas les demander en définissant `CLAUDE_CODE_AUTO_MODE_SERVER` à `0` avant de démarrer la session, dans votre shell ou dans la [clé de réglages `env`](https://code.claude.com/docs/en/settings-reference#env) :

```bash theme={null}
export CLAUDE_CODE_AUTO_MODE_SERVER=0
```

Les requêtes de classifieur sont alors toujours celles de Claude Code, facturées de la même façon, et l'avis n'apparaît pas. Sur une connexion directe à l'API Anthropic, la variable nécessite Claude Code v2.1.281+. Définir `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1` alors que `CLAUDE_CODE_AUTO_MODE_SERVER` n'est pas défini désactive aussi les vérifications du serveur, sauf dans les cas décrits par [Disable pre-release capabilities](https://code.claude.com/docs/en/llm-gateway-protocol#disable-pre-release-capabilities).

`CLAUDE_CODE_AUTO_MODE_SERVER` est un réglage temporaire et pourra être supprimé dans une version ultérieure.

## Pourquoi l'avis apparaît

[Server-side classifier review](https://code.claude.com/docs/en/permission-modes#server-side-classifier-review) liste les sessions qui demandent au serveur les vérifications du classifieur. Les forfaits Pro, Max et Team n'affichent jamais l'avis. Quand il apparaît, les causes habituelles sont :

* **Une passerelle ou un proxy LLM est sur le chemin** : une passerelle qui supprime ou réécrit les en-têtes de requête, abandonne les champs de requête qu'elle ne reconnaît pas, ou modifie les réponses. Le serveur ne reçoit alors jamais la demande de vérification, ou Claude Code ne reçoit jamais les résultats. Quand votre configuration ou les réponses identifient la passerelle, l'avis la nomme.
* **Les vérifications côté serveur n'ont pas encore atteint votre plateforme, votre région ou votre type d'identifiant** : le fait qu'une plateforme ou une région les effectue dépend du déploiement progressif de cette plateforme. Si vous voyez l'avis sans passerelle ni proxy sur le chemin et qu'il continue d'apparaître, c'est la cause probable. Pour confirmer, contactez le support ou l'administrateur de votre entreprise, ou signalez-le avec `/feedback`.

Pour vérifier une session en mode auto, lancez `/status` à l'invite de Claude Code : sa ligne **Auto mode server** indique `Enabled` tant que les vérifications du serveur décident des actions de la session, et `Disabled` une fois que la session est passée en repli.

Quand une passerelle tronque les réponses ou réécrit les résultats sous une forme que Claude Code ne peut pas lire, vous obtenez des refus sans verdict au lieu de cet avis ; voir [Server-side classifier review](https://code.claude.com/docs/en/permission-modes#server-side-classifier-review).

## Ressources associées

* [Mode auto](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode) : ce qu'est le mode auto et ce qu'il bloque par défaut
* [Server-side classifier review](https://code.claude.com/docs/en/permission-modes#server-side-classifier-review) : quelles sessions demandent au serveur de vérifier les actions, et la version de Claude Code requise pour chacune
* [Guide de compatibilité des passerelles](https://code.claude.com/docs/en/llm-gateway-protocol#feature-pass-through) : ce qui casse quand une passerelle supprime des en-têtes ou des champs du corps
* [The server returned no safety verdict](https://code.claude.com/docs/en/errors#the-server-returned-no-safety-verdict) : le refus affiché quand le serveur ne donne aucun verdict pour une action
* [Gérer efficacement les coûts](https://code.claude.com/docs/en/costs) : suivre la consommation de jetons et réduire les coûts de Claude Code
