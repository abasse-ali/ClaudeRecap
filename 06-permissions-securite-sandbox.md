# 06 — Permissions, sécurité et sandbox

[← Sommaire](README.md)

Claude Code demande la permission avant d'agir, selon le **mode de permission** et vos **règles**. Ces règles sont appliquées par Claude Code lui-même, pas par le modèle : une consigne dans `CLAUDE.md` influence ce que Claude *essaie* de faire, mais seules les règles et les hooks déterminent ce qui est *autorisé*.

## Les modes de permission

| Mode (libellé dans l'interface) | Valeur de config | Ce qui passe sans demander | Usage |
|---|---|---|---|
| **Manual** | `default` (alias `manual`) | Lectures uniquement | Tout valider soi-même, travail sensible |
| **Accept edits** / Edit automatically | `acceptEdits` | Lectures, modifications de fichiers, commandes de fichiers courantes (`mkdir`, `touch`, `mv`, `cp`…) dans le dossier de travail | Itérer sur du code qu'on relit |
| **Plan** | `plan` | Lectures (+ commandes approuvées par le classifieur si le mode auto est disponible) ; **aucune modification du code** | Explorer et concevoir avant de changer |
| **Auto** | `auto` | Tout, avec un **classifieur de sécurité** en arrière-plan | Longues tâches, moins de demandes |
| *(CLI seulement)* | `dontAsk` | Lectures dans le dossier de travail et outils pré-autorisés ; **tout le reste est refusé** | CI et scripts verrouillés |
| **Bypass permissions** | `bypassPermissions` | Tout | **Uniquement** conteneurs/VM isolés |

### Changer de mode

- **Terminal** : `Shift+Tab` fait défiler Manual → Accept edits → Plan → (Bypass si activé) → Auto. `dontAsk` ne s'obtient qu'avec `--permission-mode dontAsk`.
- **VS Code** : indicateur de mode en bas de la zone de saisie.
- **Desktop** : sélecteur à côté du bouton Envoyer (`Cmd+Shift+M`).
- **Au démarrage** : `claude --permission-mode plan`, ou `permissions.defaultMode` dans les réglages.
- Mode de départ par défaut (v2.1.283+) : **auto** dans le terminal et VS Code quand il est disponible ; `default` en mode `-p`.

### Le mode plan

Claude lit, explore, lance des commandes d'exploration et rédige un **plan**, sans modifier le code. Une fois le plan prêt, vous choisissez :

- **Yes, and use auto mode** — approuver et exécuter en mode auto ;
- **Yes, manually approve edits** — approuver et valider chaque modification ;
- **No, keep planning** — rester en mode plan et dire ce qu'il faut changer.

Entrer en mode plan : `Shift+Tab`, `/plan [tâche]`, ou `claude --permission-mode plan`. Le modèle `opusplan` utilise Opus en mode plan puis Sonnet pour l'exécution.

### Le mode auto et son classifieur

Un classifieur vérifie chaque action risquée avant exécution. Disponible avec les modèles récents (Opus 4.6+, Sonnet 4.6+, Haiku 5.5, Fable) ; les administrateurs peuvent le désactiver (`permissions.disableAutoMode`).

**Ce qu'il bloque par défaut** (extraits) : `curl | bash`, envoi de données sensibles vers l'extérieur, déploiements et migrations en production, suppressions massives sur le stockage cloud, octroi de permissions IAM/dépôt, `git push --force`, `git reset --hard` / `git clean -fd` / `git stash drop`, `terraform destroy`, écriture de secrets, modifications DNS/TLS, fusion d'une PR non approuvée par un humain, désactivation de la CI, affichage d'un identifiant en clair, tunnels et reverse shells, flags qui désarment une sécurité (`--insecure`), suppression de tests de sécurité, push vers un dépôt public de données privées, PR vers un autre dépôt non demandé…

**Ce qu'il autorise** : opérations locales dans le dossier de travail, installation des dépendances déclarées dans les lockfiles, etc.

Personnaliser : `/auto-mode-setup`, onglet **Auto mode** de `/permissions`, réglage `autoMode` (infrastructure de confiance), `claude auto-mode defaults|config|reset`. Les refus récents sont visibles dans `/permissions`.

### Ce qu'aucun mode n'approuve automatiquement

- les outils visés par une règle **ask** explicite ;
- `AskUserQuestion` et les outils MCP marqués « interaction requise » ;
- `rm`/`rmdir` visant un **chemin critique** (ex. `rm -rf /`, `rm -rf ~`) ;
- certaines garde-fous de messagerie entre sessions.

**Chemins protégés** (écritures jamais auto-approuvées hors bypass) : `.git`, `.config/git`, `.vscode`, `.idea`, `.husky`, `.cargo`, `.devcontainer`, `.yarn`, `.mvn`, `.claude` (sauf exceptions comme les worktrees de Claude et la mémoire auto).

## Répondre à une demande de permission

Une demande affiche l'action exacte (commande, diff du fichier, URL…) et des options :

- **Yes** — autoriser une fois ;
- **Yes, and don't ask again** — autoriser durablement (règle enregistrée dans `.claude/settings.local.json` pour Bash/WebFetch) ou pour la session (modifications de fichiers) ;
- **Yes, and switch to auto mode** — sur certaines demandes ;
- **No** — refuser ; `Tab` sur Oui/Non pour **ajouter un commentaire** (« non, utilise plutôt pnpm ») ; `Esc` = refuser.

Par type d'outil (mode Manual) : lectures sans demande ; Bash avec demande (sauf commandes en lecture seule : `ls`, `cat`, `git status`, `git log`…), mémorisable par dépôt et commande ; modifications de fichiers avec demande, mémorisable pour la session ; WebFetch avec demande (sauf domaines de documentation pré-approuvés), mémorisable par domaine ; WebSearch avec demande.

## Règles de permission (allow / ask / deny)

Gérées avec `/permissions` ou dans `settings.json`. **Ordre d'évaluation : deny, puis ask, puis allow** — la première correspondance gagne ; une règle allow ne peut pas créer d'exception à une règle deny.

Syntaxe : `Outil` ou `Outil(spécificateur)`.

| Règle | Effet |
|-------|-------|
| `Bash` | Toutes les commandes Bash |
| `Bash(npm run build)` | Exactement cette commande |
| `Bash(git log *)` | Toute commande commençant par `git log` |
| `Read(./.env)` | Lire le fichier `.env` |
| `Edit(src/**)` | Modifier tout fichier sous `src/` |
| `WebFetch(domain:example.com)` | Requêtes vers example.com |
| `mcp__github` / `mcp__github__create_issue` | Tout un serveur MCP / un outil précis |
| `Agent(Explore)` | Un type de sous-agent |
| `Skill(deploy)` | Un skill |

Exemple de `.claude/settings.json` :

```json
{
  "permissions": {
    "allow": ["Bash(npm test)", "Bash(npm run lint)", "Bash(git diff *)"],
    "ask": ["Bash(git push *)"],
    "deny": ["Read(./.env)", "Read(./secrets/**)", "Bash(curl *)", "WebFetch"],
    "additionalDirectories": ["../shared-lib"],
    "defaultMode": "acceptEdits"
  }
}
```

- Une règle **deny avec un nom d'outil seul** retire l'outil du contexte de Claude.
- Les commandes composées (`a && b`, `$(…)`) sont vérifiées sous-commande par sous-commande.
- `permissions.blockReadsOutsideWorkingDirectories` empêche les lectures hors des dossiers de travail.

## Dossiers de travail

Claude lit et modifie les fichiers du dossier de lancement. Pour en ajouter : `--add-dir`, `/add-dir`, ou `permissions.additionalDirectories`. `/cd` déplace toute la session. Un dossier ajouté donne accès aux fichiers, mais la plupart de sa configuration `.claude/` n'est pas chargée.

## Confiance de l'espace de travail

À la première ouverture d'un dossier, Claude Code demande si vous lui faites confiance. Tant que ce n'est pas le cas, les éléments du dépôt qui exécutent du code (hooks du projet, serveurs MCP de `.mcp.json`, règles allow du projet…) ne s'appliquent pas.

## La sandbox Bash

Activez-la avec `/sandbox`. Les commandes de Claude (et leurs sous-processus) tournent alors dans une enceinte :

| Accès | Par défaut |
|-------|-----------|
| Écriture | Dossier de travail, dossier temporaire, dossiers ajoutés |
| Lecture | La majeure partie de la machine (restreignable avec `sandbox.filesystem.denyRead`) |
| Réseau | Pas d'accès direct : un proxy local vérifie chaque hôte contre `sandbox.network.allowedDomains` |
| Variables d'environnement | Héritées (option pour protéger les identifiants) |

Deux modes : **auto-allow** (les commandes sandboxées passent sans demande) et **régulier** (demande normale). macOS utilise le sandbox natif ; Linux/WSL2 utilisent bubblewrap (`bwrap`) et `socat`. Commandes exclues : `sandbox.excludedCommands`. Tous les réglages `sandbox.*` sont dans [19](19-reference-tous-les-reglages.md#sandbox-bash-isolé).

### Choisir son niveau d'isolation

| Approche | Ce qui est isolé | Docker ? | Mise en place |
|----------|------------------|:--------:|---------------|
| Bash en sandbox (`/sandbox`) | Commandes Bash, PowerShell, Monitor et leurs enfants | Non | Minimale (macOS), faible (Linux/WSL2) |
| Sandbox runtime | Tout le processus Claude Code (outils de fichiers, MCP, hooks) | Non | Faible |
| Dev container | Tout l'environnement de développement | Oui | Moyenne |
| Conteneur personnalisé | Tout l'environnement | Oui | Moyenne à élevée |
| Machine virtuelle | Tout le système | Non | Élevée |
| Session cloud | Tout le système, hébergé par Anthropic | Non | Aucune (abonnement Claude) |

## Autres protections de sécurité

- **Points de contrôle** : chaque prompt crée un checkpoint, `/rewind` ou `Esc` `Esc` pour annuler (voir [07](07-memoire-contexte-sessions.md)).
- **Suppression des caractères invisibles** dans les textes collés.
- **Détection d'injection de prompt** dans le contenu web et à l'écran (computer use, navigateur).
- **Hooks `PreToolUse`** pour bloquer ce que vous voulez de façon déterministe (voir [10](10-hooks-et-automatisation.md)).
- **`/security-review`** et plugins de sécurité (voir [16](16-integrations-revue-ci.md)).
- **Réglages gérés** par l'entreprise, que l'utilisateur ne peut pas contourner (voir [17](17-sdk-entreprise-couts.md)).
- **Mode restreint** `--restricted` et **mode sans échec** `--safe-mode`.

## Configurations types

| Objectif | Commande |
|----------|----------|
| Tout valider soi-même | `claude --permission-mode default` |
| Moins de demandes sans classifieur | Manual + `/sandbox` en mode auto-allow |
| Explorer d'abord | `claude --permission-mode plan` |
| Travailler en autonomie | `claude --permission-mode auto` |
| CI avec liste blanche exacte | `claude -p "…" --permission-mode dontAsk --allowedTools "Bash(npm test)" "Read"` |
| Totalement autonome dans un conteneur | `claude -p "…" --dangerously-skip-permissions` (utilisateur non-root, conteneur/VM) |
