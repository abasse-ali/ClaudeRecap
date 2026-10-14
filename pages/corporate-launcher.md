# Exécuter Claude Code derrière un lanceur d'entreprise

> Source : https://code.claude.com/docs/en/corporate-launcher · [← Index](../31-index-documentation-officielle.md)

> Faire passer par un lanceur obligatoire les processus que Claude Code démarre depuis son propre binaire — y compris le service d'arrière-plan et chaque session de l'agent view — avec `CLAUDE_CODE_PROCESS_WRAPPER` ou le réglage `processWrapper`.

Certaines organisations exigent que chaque processus d'un poste de travail démarre via un lanceur obligatoire. Ce lanceur applique le sandbox, les contrôles réseau ou l'injection d'identifiants dont dépend la posture de sécurité de l'entreprise ; un binaire qui démarre sans lui constitue une violation de la politique.

`CLAUDE_CODE_PROCESS_WRAPPER` fait démarrer via votre lanceur chaque processus que Claude Code lance depuis son propre binaire : le [service d'arrière-plan](https://code.claude.com/docs/en/agent-view#the-supervisor-process), chaque session qu'il héberge dans l'[agent view](https://code.claude.com/docs/en/agent-view), et les relances de Claude Code après une mise à jour. Donnez-lui comme valeur le chemin absolu de votre lanceur : Claude Code exécute alors le lanceur en lui passant la commande Claude Code comme arguments.

Un lanceur qui enveloppe la commande `claude` présente sur votre `PATH` ne peut pas atteindre le service d'arrière-plan ni les sessions qu'il héberge, car ceux-ci démarrent depuis le chemin direct du binaire, sans rechercher `claude` dans le `PATH`.

> **Note** : `CLAUDE_CODE_PROCESS_WRAPPER` nécessite Claude Code v2.1.208 ou ultérieure. Les versions antérieures ignorent la variable et démarrent chaque processus sans lanceur. Le [réglage `processWrapper`](https://code.claude.com/docs/en/settings-reference#processwrapper) équivalent nécessite v2.1.210 ou ultérieure. Les versions antérieures l'ignorent comme une clé inconnue, n'appliquent aucun lanceur et ne signalent aucune erreur.
>
> Après avoir déployé l'une ou l'autre forme, utilisez l'[étape Vérifier](#mettre-en-place-le-lanceur) pour confirmer que la version en cours d'exécution l'applique bien.

## Ce que couvre le lanceur

Avec `CLAUDE_CODE_PROCESS_WRAPPER` défini, Claude Code démarre chacun des processus suivants via votre lanceur :

* Le service d'arrière-plan que `claude agents` et les sessions d'arrière-plan démarrent à la demande.
* L'hôte de terminal et la session Claude Code à l'intérieur de chaque ligne de l'agent view, y compris les sessions de réserve « chaudes » (warm standby) que le service garde prêtes.
* Les sessions que le service relance après une mise à jour ou un plantage.
* La relance que Claude Code effectue de lui-même pour terminer l'installation d'une mise à jour, y compris l'action de redémarrage-pour-mise-à-jour de l'agent view.
* Les processus de session que démarre [Remote Control](https://code.claude.com/docs/en/remote-control). Nécessite Claude Code v2.1.210+.
* Les sessions de coéquipiers en panneaux divisés que les [agent teams](https://code.claude.com/docs/en/agent-teams) démarrent dans tmux ou iTerm2. Les panneaux de coéquipiers sont interactifs et non des processus d'arrière-plan, mais Claude Code les démarre depuis son propre binaire, donc le lanceur les couvre. Nécessite Claude Code v2.1.210+.

**Windows** : la variable y est ignorée, car le contrat du lanceur repose sur `exec`, que Windows ne prend pas en charge. Une machine Windows où la variable est définie exécute chaque processus sans lanceur et continue de fonctionner ; le seul signal est un avertissement dans le [journal de débogage](https://code.claude.com/docs/en/troubleshooting). Si votre politique de lanceur couvre Windows, la variable n'y satisfait pas : comptez les machines Windows comme non enveloppées lorsque vous planifiez le déploiement.

### Processus qui démarrent hors du lanceur

Les processus suivants ne démarrent pas via le lanceur :

* **Une session que vous démarrez vous-même dans un terminal**, qui s'exécute selon la façon dont vous l'avez invoquée. Pour couvrir ces sessions, placez un script nommé `claude` dans un répertoire situé plus tôt dans le `PATH`, qui exécute votre lanceur avec le vrai binaire ; ne remplacez pas le lien symbolique géré. Le service d'arrière-plan et ses sessions démarrent sans recherche dans le `PATH`, donc les deux lanceurs ne s'empilent pas à cet endroit.
* **Le premier processus d'un lien profond `claude-cli://`**, que le gestionnaire de protocole du système d'exploitation démarre directement. Tout ce que cette session démarre ensuite en arrière-plan passe par le lanceur. Pour fermer complètement ce chemin, [empêchez l'enregistrement du gestionnaire](https://code.claude.com/docs/en/deep-links#registration-and-supported-platforms) avec le réglage `disableDeepLinkRegistration`.
* **La relance effectuée par `--worktree` combiné à `--tmux`** : c'est le multiplexeur de terminal qui démarre ce panneau, pas le binaire de Claude Code.
* **L'hôte de messagerie native (native-messaging host) qu'enregistre [Claude in Chrome](https://code.claude.com/docs/en/chrome)** : c'est le navigateur qui le démarre, pas le binaire de Claude Code.

### Noms des processus auxiliaires dans les moniteurs de processus

Avec un lanceur configuré, `ps` et le Moniteur d'activité (Activity Monitor) n'affichent plus les libellés `claude bg-pty-host` et `claude bg-spare` de Claude Code pour les processus auxiliaires d'arrière-plan, car l'`exec` du lanceur reconstruit la liste d'arguments. La perte de ces libellés est un effet de bord, pas une dissimulation : les processus sont par ailleurs inchangés, et Claude Code identifie ses propres processus par le chemin du binaire, jamais par le nom affiché.

## Mettre en place le lanceur

### Étape 1 — Écrire le script du lanceur

Créez un script exécutable à un chemin absolu, par exemple `/opt/corp/launcher`. Claude Code l'exécute avec la commande Claude Code complète comme arguments, et le script doit se terminer par l'appel `exec "$@"` afin de se remplacer lui-même par Claude Code :

```bash theme={null}
#!/bin/sh
# Configuration propre à votre organisation : entrer dans le sandbox, appliquer
# les contrôles réseau ou injecter des identifiants.
exec "$@"
```

Rendez-le exécutable avec `chmod +x`. La partie de configuration correspond à tout ce que votre lanceur doit faire avant l'exécution de Claude Code ; [le contrat du lanceur](#le-contrat-du-lanceur) ci-dessous liste les règles que le script doit respecter.

> **Note** : si vous aviez auparavant remplacé le lien symbolique `~/.local/bin/claude` par votre lanceur, restaurez le lien symbolique d'origine dans le même changement. Un lien symbolique remplacé fait démarrer le service d'arrière-plan, dès la première session enveloppée, à travers les deux lanceurs à la fois, et place l'installation dans un état « géré de l'extérieur » (externally managed) : `/doctor` le signale, la mise à jour automatique laisse le fichier en place, et le nettoyage des anciennes versions reste désactivé tant que l'installeur ne gère pas de nouveau ce chemin.

### Étape 2 — Définir CLAUDE_CODE_PROCESS_WRAPPER dans les réglages

Définissez la variable dans le bloc `env` d'un fichier de réglages afin que le service d'arrière-plan détaché en hérite. Un `export` dans le shell ne suffit pas : le service d'arrière-plan démarre à la demande, survit à votre shell et ne relit jamais les profils du shell.

Pour une seule machine, ajoutez-la à `~/.claude/settings.json`. Pour la déployer sur toutes les machines de votre organisation, placez le même bloc dans les [réglages gérés (managed settings)](https://code.claude.com/docs/en/managed-settings) :

```json theme={null}
{
  "env": {
    "CLAUDE_CODE_PROCESS_WRAPPER": "/opt/corp/launcher"
  }
}
```

Lorsque plusieurs sources définissent la variable, la valeur des réglages gérés l'emporte à la fois sur `~/.claude/settings.json` et sur une valeur exportée dans le shell, de sorte que les utilisateurs ne peuvent pas faire pointer les auto-lancements (self-spawns) vers un autre lanceur.

Le [réglage `processWrapper`](https://code.claude.com/docs/en/settings-reference#processwrapper) porte la même valeur sous forme de clé de réglage nommée, de premier niveau. Utilisez-le lorsque votre organisation pousse les réglages sous forme de clés individuelles plutôt que d'un bloc `env`. Le réglage `processWrapper` nécessite Claude Code v2.1.210+. Le fichier de réglages suivant définit le même lanceur via cette clé :

```json theme={null}
{
  "processWrapper": "/opt/corp/launcher"
}
```

`CLAUDE_CODE_PROCESS_WRAPPER` est prioritaire lorsque les deux sont définis.

Comme `processWrapper` est un réglage nommé, une organisation qui le distribue via les [réglages gérés à distance (remote managed settings)](https://code.claude.com/docs/en/managed-settings#delivery-mechanisms) le voit listé dans la [boîte de dialogue d'approbation de sécurité](https://code.claude.com/docs/en/server-managed-settings#security-approval-dialogs), aux côtés des autres réglages qui exécutent des exécutables fournis par l'administrateur.

**Les réglages de projet et locaux ne peuvent pas configurer le lanceur.** Un fichier commité dans un dépôt ne doit pas pouvoir placer un binaire devant chaque processus Claude Code de la machine : Claude Code ignore donc `CLAUDE_CODE_PROCESS_WRAPPER` dans `.claude/settings.json` ou `.claude/settings.local.json` (avec un avertissement dans le [journal de débogage](https://code.claude.com/docs/en/troubleshooting)), et ne lit jamais la clé `processWrapper` dans ces fichiers.

### Étape 3 — Redémarrer le service d'arrière-plan et vos sessions

Un service d'arrière-plan en cours d'exécution et toutes les sessions `claude` ouvertes lisent la variable une seule fois au démarrage ; ils continuent donc de lancer des processus non enveloppés jusqu'à leur redémarrage. Exécutez `claude daemon stop --any` pour arrêter le service à la demande. La prochaine commande qui en a besoin, comme `claude agents`, en démarre un enveloppé. Redémarrez ensuite vos sessions `claude` ouvertes.

Sur les machines que vous ne pouvez pas redémarrer vous-même, la première session démarrée après la diffusion des réglages retire automatiquement un service à la demande non enveloppé restant. Une machine sur laquelle aucune nouvelle session ne démarre conserve son service d'arrière-plan non enveloppé jusqu'à ce qu'une session démarre.

### Étape 4 — Vérifier

Exécutez `/status` dans une session : l'entrée **Self-exec** affiche la commande de lancement résolue et avertit lorsque le service d'arrière-plan en cours d'exécution ne lui correspond pas. `claude daemon status` affiche les mêmes informations depuis le shell, y compris après que vous avez supprimé la variable, lorsque `/status` n'affiche plus l'entrée.

## Le contrat du lanceur

Lorsque le lanceur ne peut pas s'exécuter, Claude Code refuse de démarrer le processus au lieu de le démarrer sans lanceur. Sous Windows, [la variable est ignorée](#ce-que-couvre-le-lanceur) et les processus démarrent sans lanceur. Claude Code impose au script les règles suivantes :

* **Se terminer par `exec "$@"`.** Un lanceur qui crée un processus enfant (fork) puis se termine laisse un processus Claude Code orphelin que le service d'arrière-plan ne peut pas suivre. L'agent view marque une telle session comme échouée avec un message nommant le lanceur, et le service récupère (reap) ce que le lanceur a laissé derrière lui.
* **Ne pas réordonner, absorber ni préfixer d'arguments.** Le premier argument est le binaire Claude Code, et tout ce qui suit constitue son argv.
* **Transmettre chaque variable d'environnement héritée jusqu'à `exec`.** Ajouter des variables (par exemple des identifiants injectés) est acceptable ; supprimer des variables héritées ne l'est pas.
  * Les jetons d'authentification par session, la sélection du modèle et du fournisseur, et `CLAUDE_CODE_PROCESS_WRAPPER` lui-même transitent tous par l'environnement hérité : un lanceur qui reconstruit l'environnement à partir d'une liste autorisée (allow list) casse les sessions qu'il démarre, et `/status` signale une incohérence de lanceur.
  * Si le lanceur doit entrer dans un namespace ou un sandbox qui réinitialise l'environnement, réexportez-y l'environnement hérité à l'identique.
* **Atteindre `exec` en environ trois secondes à chaque exécution du lanceur.** Une distribution d'arrière-plan à froid (cold background dispatch) exécute le lanceur deux fois en série avant le premier octet de sortie : effectuez les traitements lents, comme un échange d'authentification unique (single sign-on), de manière paresseuse ou depuis un cache.
* **Tolérer d'être invoqué depuis lui-même.** Claude Code applique le lanceur à chaque auto-lancement imbriqué ; un lanceur qui acquiert une ressource exclusive doit donc détecter qu'il la détient déjà.
* **Ne rien écrire dans le terminal avant le démarrage de Claude Code.** Tout ce qui est affiché avant l'`exec` est signalé comme cause du plantage si la session meurt avant son initialisation.
* **Ne pas dépendre de la façon dont les arguments sont écrits.** La valeur d'un flag peut arriver comme argument séparé, `--flag value`, ou accolée au flag, `--flag=value`. La forme utilisée par un flag peut changer d'une version à l'autre.
* **Exécuter `exec` depuis le répertoire dans lequel le lanceur a démarré.** Claude Code travaille dans le répertoire où il démarre ; un lanceur qui change de répertoire peut donc démarrer une session dans le mauvais projet. Si le lanceur doit entrer dans un namespace ou un sandbox qui réinitialise le répertoire de travail, revenez au répertoire de départ à l'intérieur de celui-ci, avant l'`exec`.

### Format de la valeur du lanceur

`CLAUDE_CODE_PROCESS_WRAPPER` et le réglage `processWrapper` acceptent le même format. Pour la plupart des lanceurs, la valeur est le chemin absolu du script, comme `/opt/corp/launcher`.

Pour passer à votre lanceur des arguments qui lui sont propres, écrivez-les après le chemin. Claude Code analyse la valeur comme une liste d'arguments, pas comme une commande shell :

* Les espaces séparent les jetons, et les guillemets doubles regroupent un jeton contenant des espaces.
* Une valeur qui commence par `[` est lue comme un tableau JSON de chaînes, par exemple `["/opt/corp/launcher", "--profile", "cc"]`.
* La syntaxe shell ne fonctionne pas : il n'y a ni expansion de variables ni globbing, et un opérateur non entre guillemets tel que `;`, `|`, `&` ou `$(` est rejeté comme erreur de configuration au lieu d'être réinterprété.

Lorsque la valeur est inutilisable, Claude Code refuse de démarrer le processus concerné et [en indique la raison](https://code.claude.com/docs/en/errors#claude_code_process_wrapper-launcher-errors).

## Relation avec `CLAUDE_CODE_SHELL_PREFIX`

`CLAUDE_CODE_PROCESS_WRAPPER` enveloppe les propres processus de Claude Code et transmet la commande sous forme de jetons argv séparés, que le lanceur passe à `exec`. [`CLAUDE_CODE_SHELL_PREFIX`](https://code.claude.com/docs/en/env-vars) enveloppe les commandes shell que Claude exécute pour vous — appels de l'outil Bash, hooks, et commandes qui démarrent les serveurs MCP stdio — et transmet chacune sous forme d'une seule chaîne échappée pour le shell dans `$1`, que l'enveloppe doit réévaluer. Un lanceur écrit pour l'un ne fonctionne pas comme l'autre.

| Variable | Ce qu'elle enveloppe | Forme de la commande transmise |
| :- | :- | :- |
| `CLAUDE_CODE_PROCESS_WRAPPER` | Les propres processus de Claude Code (service d'arrière-plan, sessions, relances) | Jetons argv séparés, à passer à `exec "$@"` |
| `CLAUDE_CODE_SHELL_PREFIX` | Les commandes shell exécutées par Claude (outil Bash, hooks, démarrage des serveurs MCP stdio) | Une seule chaîne échappée pour le shell dans `$1`, à réévaluer |

## Ressources associées

* [Agent view](https://code.claude.com/docs/en/agent-view) : les sessions d'arrière-plan et le processus superviseur couverts par le lanceur
* [Variables d'environnement](https://code.claude.com/docs/en/env-vars) : l'entrée de référence de `CLAUDE_CODE_PROCESS_WRAPPER`
* [Réglages gérés](https://code.claude.com/docs/en/permissions#managed-settings) : distribuer le bloc `env` sur l'ensemble d'un parc
* [Référence des erreurs du lanceur](https://code.claude.com/docs/en/errors#claude_code_process_wrapper-launcher-errors) : les messages de refus et la manière de rétablir la situation
