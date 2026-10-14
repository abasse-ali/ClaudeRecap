# Points de contrôle (checkpointing)

> Source : https://code.claude.com/docs/en/checkpointing · [← Index](../31-index-documentation-officielle.md)

*Suivre, rembobiner (rewind) et résumer les modifications et la conversation de Claude pour gérer l'état de la session.*

Claude Code suit automatiquement les modifications de fichiers effectuées par Claude pendant que vous travaillez, ce qui vous permet d'annuler rapidement des changements et de revenir à des états antérieurs si quelque chose dérape.

## Fonctionnement des checkpoints

Pendant que vous travaillez avec Claude, le checkpointing capture automatiquement l'état de votre code avant chaque prompt que vous envoyez et qui démarre un tour (turn).

### Suivi automatique

Claude Code suit toutes les modifications effectuées par ses outils d'édition de fichiers :

* chaque prompt que vous envoyez et qui démarre un tour crée un nouveau checkpoint ;
* Claude Code conserve les instantanés (snapshots) de fichiers pour les **100 checkpoints les plus récents** d'une session. L'abandon d'un checkpoint plus ancien supprime les fichiers d'instantané qu'aucun checkpoint restant ne référence, à l'exception du premier instantané de chaque fichier, que l'extension VS Code utilise comme base de référence pour ses diffs de session ;
* Claude Code enregistre les checkpoints avec la conversation : vous pouvez donc toujours lancer `/rewind` après avoir repris une session ;
* Claude Code supprime les instantanés de fichiers d'une session lors du [balayage de rétention](https://code.claude.com/docs/en/claude-directory#cleaned-up-automatically), par défaut environ 30 jours après le dernier instantané enregistré par la session. Revenir à un checkpoint dont les instantanés ont disparu peut échouer avec [`No files were restored`](https://code.claude.com/docs/en/errors#no-files-were-restored). Pour conserver les instantanés plus longtemps, définissez [`cleanupPeriodDays`](https://code.claude.com/docs/en/settings-reference#cleanupperioddays).

### Rembobiner et résumer

Lancez `/rewind`, ou appuyez deux fois sur `Esc` quand la zone de saisie du prompt est vide, pour ouvrir le menu de rewind.

> **Note** : si la zone de saisie contient du texte, un double `Esc` l'efface au lieu d'ouvrir le menu. Le texte effacé est enregistré dans votre historique de saisie : appuyez sur `Up` (flèche haut) pour le rappeler une fois que vous en avez terminé avec le menu de rewind.

Le menu de rewind liste les prompts envoyés pendant la session. Sélectionnez le point sur lequel vous voulez agir, puis choisissez une action :

* **Restore code and conversation** : rétablit à la fois le code et la conversation à ce point ;
* **Restore conversation** : rembobine jusqu'à ce message en conservant le code actuel ;
* **Restore code** : annule les modifications de fichiers en conservant la conversation ;
* **Summarize from here** : compresse la conversation à partir de ce point en un résumé, libérant de l'espace dans la fenêtre de contexte ;
* **Summarize up to here** : compresse la conversation antérieure à ce point en un résumé, en gardant intacts les messages suivants ;
* **Never mind** : revient à la liste des messages sans rien modifier.

Les deux options de restauration du code n'apparaissent que si le checkpoint sélectionné comporte des modifications de fichiers suivies à annuler. Si aucune édition de fichier n'a été capturée après ce point, le menu ne propose que **Restore conversation**, les options de résumé et **Never mind**.

Après une restauration de la conversation ou le choix de Summarize from here, le prompt d'origine du message sélectionné est replacé dans le champ de saisie pour que vous puissiez le renvoyer ou le modifier.

Choisir Summarize up to here vous laisse à la fin de la conversation, avec la saisie vide. Avec l'une ou l'autre option de résumé, un marqueur **Summarized conversation** apparaît dans la conversation à l'endroit où se trouvaient les messages compressés.

#### Rembobiner au-delà d'une conversation effacée

Si vous avez lancé `/clear` plus tôt dans le même processus Claude Code, le menu de rewind affiche une entrée supplémentaire en haut de la liste, intitulée `/resume <session-id> (previous session)`. Sélectionnez-la pour reprendre la conversation qui était active avant l'exécution de `/clear`. L'entrée reste disponible jusqu'à ce que vous quittiez Claude Code ou repreniez une autre session.

#### Orienter un résumé

Résumer ne modifie pas les fichiers sur disque, et les messages d'origine restent dans la transcription de la session : Claude peut donc toujours s'y référer pour les détails. Pour orienter ce sur quoi le résumé se concentre, mettez en surbrillance une option **Summarize** avec les touches fléchées et tapez des instructions là où la ligne indique **add context (optional)**, puis appuyez sur `Enter`. Sélectionner l'option avec sa touche numérique résume immédiatement, sans instructions.

> **Note** : résumer vous maintient dans la même session et compresse le contexte, comme un `/compact` ciblé. Pour bifurquer et essayer une approche différente tout en préservant intacte la session d'origine, utilisez plutôt [`/branch`](https://code.claude.com/docs/en/sessions#branch-a-session) ou `claude --continue --fork-session`.

## Cas d'usage courants

Les checkpoints sont particulièrement utiles pour :

* **Explorer des alternatives** : essayer différentes approches d'implémentation sans perdre votre point de départ ;
* **Se remettre d'erreurs** : annuler rapidement des modifications qui ont introduit des bugs ou cassé des fonctionnalités ;
* **Itérer sur des fonctionnalités** : expérimenter des variantes en sachant que vous pouvez revenir à des états fonctionnels ;
* **Libérer de l'espace de contexte** : résumer une session de débogage verbeuse à partir de son milieu, en gardant intactes vos instructions initiales.

## Limitations

### Modifications faites par des commandes Bash non suivies

Le checkpointing ne suit pas les fichiers modifiés par des commandes Bash. Par exemple, si Claude Code exécute :

```bash theme={null}
rm file.txt
mv old.txt new.txt
cp source.txt dest.txt
```

Ces modifications de fichiers ne peuvent pas être annulées via le rewind. Seules les éditions directes de fichiers faites via les outils d'édition de fichiers de Claude sont suivies.

### Éditions des sous-agents non restaurées

Un [sous-agent](https://code.claude.com/docs/en/sub-agents) effectue ses éditions avec les outils d'édition de fichiers de Claude, mais Claude Code ne capture généralement pas ces éditions dans les checkpoints de votre session. Le fait que le rewind les restaure dépend de la façon dont le sous-agent s'exécute :

* **Skill forké au premier plan** : un [skill avec `context: fork`](https://code.claude.com/docs/en/skills#run-skills-in-a-subagent) qui s'exécute au premier plan (foreground) modifie votre arbre de travail pendant votre propre tour ; le rewind restaure donc ses éditions comme d'habitude. Définissez `background: false` pour exécuter un fork au premier plan ; quelques situations, [listées sur la page des skills](https://code.claude.com/docs/en/skills#run-skills-in-a-subagent), l'y exécutent quel que soit ce réglage.
* **Tout autre sous-agent** : le rewind ne restaure pas les éditions. Utilisez git pour les annuler. Cela inclut un skill forké exécuté en arrière-plan (le comportement par défaut) et une exécution en arrière-plan de [`/code-review --fix`](https://code.claude.com/docs/en/code-review).

### Modifications externes non suivies

Le checkpointing ne suit que les fichiers édités au sein de la session en cours. Les modifications manuelles que vous faites aux fichiers en dehors de Claude Code et les éditions provenant d'autres sessions concurrentes ne sont normalement pas capturées, sauf si elles touchent les mêmes fichiers que la session en cours.

### Messages envoyés en cours de tour sans checkpoint

Dans le menu de rewind, un message que vous avez [tapé pendant que Claude travaillait encore](https://code.claude.com/docs/en/interactive-mode#queue-messages-while-claude-works) peut être marqué **No code restore**. Claude a lu ce message avant la fin de son tour. [Les checkpoints sont pris pour les prompts qui démarrent un tour](#fonctionnement-des-checkpoints) : ce message n'a donc pas de checkpoint propre. Les éditions faites par Claude après l'avoir lu sont comptées dans le prompt qui a démarré le tour.

Vous n'avez rien à faire concernant le message lui-même. Pour annuler les modifications de fichiers de cette partie de la session, sélectionnez le prompt qui a démarré le tour et choisissez **Restore code** ou **Restore code and conversation**. Cela annule les éditions de fichiers de Claude pour tout le tour, y compris celles antérieures à l'arrivée de votre message. Sélectionner le message marqué propose toujours **Restore conversation**, qui rembobine la conversation jusqu'à lui et laisse vos fichiers tels quels.

### Chemins en lien symbolique ou en lien physique non restaurés

Le checkpointing ne rembobine pas les fichiers en lien symbolique (symlink) ou en lien physique (hard link). Quand vous choisissez **Restore code** ou **Restore code and conversation** dans le menu `/rewind`, Claude Code ignore tout chemin suivi qui est un lien symbolique ou un lien physique et affiche un avertissement `Restored the code, but skipped N files`. Les fichiers ignorés conservent leur contenu actuel. Pour annuler les modifications de la session sur l'un d'eux, demandez à Claude d'inverser l'édition ou modifiez le fichier vous-même. Les fichiers de configuration qu'un gestionnaire de dotfiles lie symboliquement dans votre projet et les fichiers que pnpm met en place par liens physiques entrent tous deux dans cette catégorie.

Pour voir quels chemins une restauration ignore, activez la journalisation de débogage avec `/debug` avant de restaurer : le journal de débogage dans `~/.claude/debug/<session-id>.txt` nomme chaque chemin ignoré. Pour toutes les raisons d'ignorance et les étapes de récupération, voir [l'entrée sur les fichiers ignorés dans la référence des erreurs](https://code.claude.com/docs/en/errors#restored-the-code-but-skipped-files).

### Pas un substitut au contrôle de version

Les checkpoints sont conçus pour une récupération rapide au niveau de la session. Pour un historique de versions permanent et la collaboration, continuez à utiliser un contrôle de version comme Git pour les commits, les branches et l'historique à long terme.

## Voir aussi

* [Mode interactif](https://code.claude.com/docs/en/interactive-mode) — raccourcis clavier et contrôles de session ;
* [Commandes](https://code.claude.com/docs/en/commands) — accès aux checkpoints via `/rewind` ;
* [Référence CLI](https://code.claude.com/docs/en/cli-reference) — options de ligne de commande.
