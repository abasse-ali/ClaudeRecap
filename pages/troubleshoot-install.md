# Dépanner l'installation et la connexion

> Source : https://code.claude.com/docs/en/troubleshoot-install · [← Index](../31-index-documentation-officielle.md)

*Corriger les erreurs « command not found », de PATH, de permissions, de réseau et d'authentification lors de l'installation de Claude Code ou de la connexion.*

Si l'installation échoue ou si vous n'arrivez pas à vous connecter, trouvez votre erreur ci-dessous. Pour les problèmes à l'exécution une fois Claude Code fonctionnel, voir [Troubleshooting](https://code.claude.com/docs/en/troubleshooting). Pour les problèmes de configuration (réglages qui ne s'appliquent pas, hooks qui ne se déclenchent pas…), voir [Debug your configuration](https://code.claude.com/docs/en/debug-your-config).

## Trouver votre erreur

Faites correspondre le message d'erreur ou le symptôme observé à une solution :

| Ce que vous voyez | Solution |
| :- | :- |
| `command not found: claude` ou `'claude' is not recognized` | [Corriger votre PATH](#command-not-found-claude-après-linstallation) |
| `Native installation exists but ... is not in your PATH` | [Ajouter le répertoire d'installation à votre PATH](#vérifier-votre-path) |
| `INFO: Could not find files for the given pattern(s).` renvoyé par `where.exe claude` | [Vérifier si Claude Code est installé](#rechercher-des-installations-en-conflit) |
| `zsh: permission denied: /Users/you/.zshrc` ou `bash: /home/you/.bashrc: Permission denied` | [Rendre votre fichier de configuration du shell modifiable](#permission-denied-en-ajoutant-au-path) |
| `syntax error near unexpected token '<'` | [Le script d'installation renvoie du HTML](#le-script-dinstallation-renvoie-du-html-au-lieu-dun-script-shell) |
| `< was unexpected at this time` dans CMD | [Le script d'installation renvoie du HTML](#le-script-dinstallation-renvoie-du-html-au-lieu-dun-script-shell) |
| `The term 'System.Xml.XmlDocument' is not recognized` | [Le script d'installation renvoie du HTML](#le-script-dinstallation-renvoie-du-html-au-lieu-dun-script-shell) |
| `curl: (22) The requested URL returned error: 403` | [Le script d'installation a renvoyé 403](#le-script-dinstallation-renvoie-du-html-au-lieu-dun-script-shell) |
| `curl: (23)` ou `curl: (56) Failure writing output to destination` | [Vérifier la connectivité ou utiliser un autre installateur](#curl-56-failure-writing-output-to-destination) |
| `Killed` pendant l'installation sous Linux | [Libérer de la mémoire ou ajouter du swap](#installation-tuée-sur-des-serveurs-linux-à-faible-mémoire) |
| `Installation was killed before it could finish` | [Libérer de la mémoire puis relancer l'installateur](#installation-was-killed-before-it-could-finish) |
| `Raw mode is not supported` pendant l'installation | [Relancer l'installateur](#raw-mode-is-not-supported-pendant-linstallation) |
| `EACCES: permission denied` pendant l'installation | [Corriger les permissions du répertoire d'installation](#erreurs-de-permission-pendant-linstallation) |
| `TLS connect error` ou `SSL/TLS secure channel` | [Mettre à jour les certificats d'autorité (CA)](#erreurs-de-connexion-tls-ou-ssl) |
| `CRYPT_E_NO_REVOCATION_CHECK` ou `CRYPT_E_REVOCATION_OFFLINE` | [Contourner les vérifications de révocation bloquées](#erreurs-de-connexion-tls-ou-ssl) |
| `Failed to fetch version` ou serveur de téléchargement injoignable | [Vérifier les réglages réseau et proxy](#vérifier-la-connectivité-réseau) |
| `The connection dropped while downloading the update` ou `Download timed out: exceeded the total deadline` | [Relancer la mise à jour ou configurer votre proxy](#la-connexion-sest-interrompue-pendant-le-téléchargement-de-la-mise-à-jour) |
| `irm is not recognized` ou `The token '&&' is not a valid statement separator` | [Utiliser la bonne commande pour votre shell](#mauvaise-commande-dinstallation-sous-windows) |
| `Cask 'claude-code' is unavailable: No Cask with this name exists` | [Mettre à jour Homebrew](#cask-homebrew-indisponible-ou-obsolète) |
| `Cask 'claude-code@latest' is not installed` | [Mettre à niveau le cask que vous avez installé](#cask-claude-codelatest-is-not-installed) |
| `'bash' is not recognized as the name of a cmdlet` | [Utiliser la commande d'installation Windows](#mauvaise-commande-dinstallation-sous-windows) |
| `A parameter cannot be found that matches parameter name 'fsSL'` | [Utiliser la commande d'installation Windows](#mauvaise-commande-dinstallation-sous-windows) |
| `Claude Code on Windows requires either Git for Windows (for bash) or PowerShell` | [Installer un shell](#claude-code-on-windows-requires-either-git-for-windows-for-bash-or-powershell) |
| `Claude Code does not support 32-bit Windows` | [Ouvrir Windows PowerShell, pas l'entrée x86](#claude-code-does-not-support-32-bit-windows) |
| `The process cannot access the file ... because it is being used by another process` | [Vider le dossier de téléchargements et réessayer](#the-process-cannot-access-the-file-pendant-linstallation-windows) |
| `Error loading shared library` | [Mauvaise variante de binaire pour votre système](#incompatibilité-de-binaire-musl-ou-glibc-sous-linux) |
| `Illegal instruction` | [Incompatibilité d'architecture ou de jeu d'instructions CPU](#illegal-instruction) |
| `cannot execute binary file: Exec format error` dans WSL | [Régression du binaire natif sous WSL1](#exec-format-error-sous-wsl1) |
| `Bus error` ou `oh no: Bun has crashed` pendant qu'une session tourne | [Garder l'exécutable lisible](#bus-error-pendant-quune-session-tourne) |
| L'installateur PowerShell se termine mais `claude` est introuvable ou affiche une ancienne version | [Ajouter le répertoire d'installation à votre PATH](#vérifier-votre-path), puis ouvrir un nouveau terminal |
| `dyld: Symbol not found`, `dyld: cannot load` ou `Abort trap` sur macOS | [Incompatibilité du binaire](#dyld-cannot-load-sur-macos) |
| `claude update` se bloque après `Checking for updates`, ou `claude doctor` se bloque sans rien afficher | [Déplacer le répertoire situé à un chemin de configuration du shell](#claude-update-ou-claude-doctor-se-bloque) |
| Erreurs d'analyse `Invoke-Expression` ou `iex` citant des balises HTML ou du CSS, ou `ParserError` avec `ParseException` | [Le script d'installation renvoie du HTML](#le-script-dinstallation-renvoie-du-html-au-lieu-dun-script-shell) |
| `running scripts is disabled on this system` ou `PSSecurityException` | [Autoriser l'exécution des shims npm](#running-scripts-is-disabled-on-this-system) |
| `Error: claude native binary not installed` | [Terminer l'installation npm](#binaire-natif-introuvable-après-une-installation-npm) |
| `npm error code ENOTEMPTY` pendant une mise à jour ou une réinstallation | [Supprimer le répertoire de paquet résiduel](#erreur-npm-enotempty-pendant-une-mise-à-jour-ou-une-réinstallation) |
| `'claude' is not recognized` juste après une mise à jour sous Windows | [Restaurer `claude.exe` depuis sa sauvegarde](#claudeexe-manquant-après-une-mise-à-jour-sous-windows) |
| Sous Windows, la commande d'installation affiche le texte du script et rien ne s'installe | [Exécuter la commande d'installation complète](#mauvaise-commande-dinstallation-sous-windows) |
| `App unavailable in region` | Claude Code n'est pas disponible dans votre pays. Voir les [pays pris en charge](https://www.anthropic.com/supported-countries). |
| `unable to get local issuer certificate` | [Configurer les certificats CA de l'entreprise](#erreurs-de-connexion-tls-ou-ssl) |
| `OAuth error` ou `403 Forbidden` | [Corriger l'authentification](#connexion-et-authentification) |
| `Claude Code access has not been granted for this account` | [Obtenir un rôle qui inclut Claude Code](#claude-code-access-has-not-been-granted-for-this-account) |
| `Unable to connect to Anthropic services` pendant la configuration | Voir [Unable to connect to Anthropic services](https://code.claude.com/docs/en/errors#unable-to-connect-to-anthropic-services) dans la référence des erreurs |
| `Could not load the default credentials` ou `Could not load credentials from any providers` | [Identifiants Amazon Bedrock, Google Cloud's Agent Platform ou Microsoft Foundry](#les-identifiants-bedrock-agent-platform-ou-foundry-ne-se-chargent-pas) |
| `ChainedTokenCredential authentication failed` ou `CredentialUnavailableError` | [Identifiants Amazon Bedrock, Google Cloud's Agent Platform ou Microsoft Foundry](#les-identifiants-bedrock-agent-platform-ou-foundry-ne-se-chargent-pas) |
| `API Error: 500`, `529 Overloaded`, `429` ou toute autre erreur 4xx/5xx non listée ci-dessus | Voir la [référence des erreurs](https://code.claude.com/docs/en/errors) |

Si votre problème n'est pas listé, suivez les vérifications de diagnostic ci-dessous pour cerner la cause.

> **Astuce** : si vous préférez vous passer complètement du terminal, l'[application Claude Code Desktop](https://code.claude.com/docs/en/desktop-quickstart) permet d'installer et d'utiliser Claude Code via une interface graphique. Téléchargez-la pour [macOS](https://claude.ai/api/desktop/darwin/universal/dmg/latest/redirect?utm_source=claude_code&utm_medium=docs) ou [Windows](https://claude.com/download?utm_source=claude_code&utm_medium=docs) et commencez à coder sans aucune configuration en ligne de commande. Sous Linux, installez l'application avec apt en suivant les [instructions d'installation Linux](https://code.claude.com/docs/en/desktop-linux).

## Lancer les vérifications de diagnostic

### Vérifier la connectivité réseau

L'installateur télécharge depuis `downloads.claude.ai`. Vérifiez que vous pouvez l'atteindre :

#### macOS/Linux

```bash theme={null}
curl -sI https://downloads.claude.ai/claude-code-releases/latest
```

#### Windows PowerShell

```powershell theme={null}
curl.exe -sI https://downloads.claude.ai/claude-code-releases/latest
```

PowerShell fait de `curl` un alias de `Invoke-WebRequest`, qui rejette les flags `-sI` : appelez donc explicitement `curl.exe`.

#### Interpréter le résultat

Vous avez atteint le serveur si la première ligne affiche un statut `200`. Vous voyez `HTTP/2 200` sur macOS et Linux, et `HTTP/1.1 200 OK` avec le `curl.exe` fourni avec Windows. Les autres résultats orientent vers la cause :

* `403` : généralement un proxy ou un filtre réseau qui bloque l'hôte, ou bien Claude Code n'est [pas disponible dans votre région](https://www.anthropic.com/supported-countries)
* `5xx` : généralement un problème temporaire du service ; attendez quelques minutes et réessayez

Si rien ne s'affiche, ou si vous obtenez `Could not resolve host` ou un délai de connexion dépassé, votre réseau bloque la connexion. Causes fréquentes :

* Pare-feux ou proxys d'entreprise qui bloquent `downloads.claude.ai`
* Restrictions réseau régionales : essayez un VPN ou un autre réseau
* Problèmes TLS/SSL : mettez à jour les certificats CA de votre système, ou vérifiez si `HTTPS_PROXY` est configuré

Si vous êtes derrière un proxy d'entreprise, définissez `HTTPS_PROXY` et `HTTP_PROXY` avec l'adresse de votre proxy avant d'installer. Demandez l'URL du proxy à votre équipe informatique si vous ne la connaissez pas, ou consultez les réglages de proxy de votre navigateur.

Cet exemple définit les deux variables de proxy, puis lance l'installateur à travers votre proxy :

#### macOS/Linux (proxy)

```bash theme={null}
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=http://proxy.example.com:8080
curl -fsSL https://claude.ai/install.sh | bash
```

#### Windows PowerShell (proxy)

```powershell theme={null}
$env:HTTP_PROXY = 'http://proxy.example.com:8080'
$env:HTTPS_PROXY = 'http://proxy.example.com:8080'
irm https://claude.ai/install.ps1 | iex
```

### Vérifier votre PATH

Si l'installation a réussi mais que vous obtenez une erreur `command not found` ou `not recognized` en lançant `claude`, le répertoire d'installation n'est pas dans votre PATH. Votre shell cherche les programmes dans les répertoires listés dans PATH, et l'installateur place `claude` dans `~/.local/bin/claude` sur macOS/Linux ou `%USERPROFILE%\.local\bin\claude.exe` sous Windows.

L'installateur détecte ce cas et le signale sous `Setup notes:` dans sa sortie : `Native installation exists but ~/.local/bin is not in your PATH.` sur macOS et Linux, ou `Native installation exists but C:\Users\you\.local\bin is not in your PATH.` sous Windows. Il affiche le correctif avec cette note, mais ne modifie pas lui-même le PATH.

> **Note** : l'[extension VS Code](https://code.claude.com/docs/en/vs-code) ne place pas `claude` à cet emplacement. Elle embarque une copie privée de la CLI dans le répertoire de l'extension, pour son propre panneau de chat, et ne l'ajoute pas au PATH. Si vous avez seulement installé l'extension, `~/.local/bin/claude` n'existe pas. Lancez l'[installation autonome](https://code.claude.com/docs/en/setup) pour utiliser `claude` depuis un terminal, puis continuez ci-dessous.

Vérifiez d'abord que le programme est bien présent, puis que son dossier est dans votre PATH. Le correctif du PATH est permanent : vous ne l'appliquez qu'une fois. Choisissez l'onglet de votre plateforme et exécutez ses commandes : dans votre terminal sur macOS et Linux, ou dans PowerShell ou l'invite de commandes (Command Prompt) sous Windows.

#### macOS/Linux

Vérifiez que l'installateur a mis le programme en place :

```bash theme={null}
ls -la ~/.local/bin/claude
```

* **`No such file or directory`** : il n'y a pas d'installation native. Si vous n'avez pas installé Claude Code autrement (npm, Homebrew ou gestionnaire de paquets Linux, par exemple), [installez Claude Code](https://code.claude.com/docs/en/setup#install-claude-code). Si vous l'avez installé autrement, voir [Rechercher des installations en conflit](#rechercher-des-installations-en-conflit).
* **Une ligne de listing pour le fichier** : le programme est là. Vérifiez ensuite votre PATH.

Listez les entrées de votre PATH en filtrant sur le dossier d'installation :

```bash theme={null}
echo $PATH | tr ':' '\n' | grep -Fx "$HOME/.local/bin"
```

Si cela affiche `/Users/you/.local/bin` ou `/home/you/.local/bin`, le répertoire est dans votre PATH et vous pouvez passer directement à [Rechercher des installations en conflit](#rechercher-des-installations-en-conflit). S'il n'y a aucune sortie, ajoutez-le à la configuration de votre shell avec les deux commandes correspondant à votre shell. La commande `echo` enregistre le réglage pour chaque nouveau terminal, et `source` l'applique à la fenêtre courante. La commande `echo` n'affiche rien quand elle réussit.

Pour Zsh, le shell par défaut sur macOS :

```bash theme={null}
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

Pour Bash sous Linux, shell par défaut sur la plupart des distributions :

```bash theme={null}
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Pour Bash sur macOS, ajoutez plutôt la ligne à `~/.bash_profile`. Le Terminal de macOS lance Bash comme shell de connexion (*login shell*), qui ignore `~/.bashrc` et ne lit que le premier fichier existant parmi `~/.bash_profile`, `~/.bash_login` ou `~/.profile`. Si vous avez déjà un `~/.bash_login` ou un `~/.profile` et pas de `~/.bash_profile`, mettez la ligne dans ce fichier plutôt que de créer `~/.bash_profile` :

```bash theme={null}
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bash_profile
source ~/.bash_profile
```

Vous pouvez aussi fermer puis rouvrir votre terminal.

Si la commande `echo` affiche `permission denied`, voir [`permission denied` en ajoutant au PATH](#permission-denied-en-ajoutant-au-path).

Pour d'autres shells comme fish ou Nushell, ajoutez `~/.local/bin` à votre PATH avec la syntaxe de configuration propre au shell, puis redémarrez votre terminal.

Vérifiez que le correctif a fonctionné :

```bash theme={null}
claude --version
```

Si `claude` reste introuvable, vérifiez ces causes :

* **Le terminal est antérieur au changement** : une fenêtre déjà ouverte garde son ancien PATH, et un terminal intégré à un éditeur prend son PATH de l'éditeur. Ouvrez une nouvelle fenêtre, ou quittez puis rouvrez l'éditeur.
* **La ligne n'a pas été enregistrée** : lancez `grep -n '.local/bin' ~/.zshrc`, en utilisant le nom de fichier de votre shell. La commande affiche la ligne avec son numéro si elle est présente. Si elle n'affiche rien, relancez les deux commandes PATH.
* **La ligne est partie dans le fichier d'un autre shell** : lancez `echo $0` pour voir votre shell, puis lancez les deux commandes PATH correspondant à ce shell.

#### Windows PowerShell

Vérifiez que l'installateur a mis le programme en place :

```powershell theme={null}
Test-Path "$env:USERPROFILE\.local\bin\claude.exe"
```

* **`False`** : il n'y a pas d'installation native. Si vous n'avez pas installé Claude Code autrement (npm ou WinGet, par exemple), [installez Claude Code](https://code.claude.com/docs/en/setup#install-claude-code). Si vous l'avez installé autrement, voir [Rechercher des installations en conflit](#rechercher-des-installations-en-conflit).
* **`True`** : le programme est là. Vérifiez ensuite votre PATH.

Listez les entrées de votre PATH en filtrant sur le dossier d'installation :

```powershell theme={null}
$env:PATH -split ';' | Select-String '\.local\\bin'
```

Si cela affiche `C:\Users\you\.local\bin`, le répertoire est dans votre PATH et vous pouvez passer à [Rechercher des installations en conflit](#rechercher-des-installations-en-conflit). S'il n'y a aucune sortie, ajoutez le répertoire d'installation à votre PATH utilisateur (*User PATH*) :

```powershell theme={null}
$currentPath = [Environment]::GetEnvironmentVariable('PATH', 'User')
[Environment]::SetEnvironmentVariable('PATH', "$currentPath;$env:USERPROFILE\.local\bin", 'User')
```

Redémarrez votre terminal pour que le changement prenne effet.

Vérifiez que le correctif a fonctionné :

```powershell theme={null}
claude --version
```

Si `claude` reste introuvable dans un nouveau terminal, vérifiez ces causes :

* **Le terminal tourne dans un éditeur** : il prend son PATH de l'éditeur ; quittez puis rouvrez l'éditeur.
* **Le changement n'a pas été enregistré** : lancez `[Environment]::GetEnvironmentVariable('PATH', 'User')` et cherchez `.local\bin` dans le PATH affiché. S'il manque, relancez les deux commandes.

#### Windows CMD

Vérifiez que l'installateur a mis le programme en place :

```batch theme={null}
dir "%USERPROFILE%\.local\bin\claude.exe"
```

* **`File Not Found` ou `The system cannot find the path specified.`** : il n'y a pas d'installation native. Si vous n'avez pas installé Claude Code autrement (npm ou WinGet, par exemple), [installez Claude Code](https://code.claude.com/docs/en/setup#install-claude-code). Si vous l'avez installé autrement, voir [Rechercher des installations en conflit](#rechercher-des-installations-en-conflit).
* **Une ligne de listing pour `claude.exe`** : le programme est là. Vérifiez ensuite votre PATH.

Listez les entrées de votre PATH en filtrant sur le dossier d'installation :

```batch theme={null}
echo %PATH% | findstr /i "local\bin"
```

S'il n'y a aucune sortie, ouvrez les Paramètres système, allez dans Variables d'environnement, et ajoutez `%USERPROFILE%\.local\bin` à votre variable PATH utilisateur. Redémarrez votre terminal.

Vérifiez que le correctif a fonctionné :

```batch theme={null}
claude --version
```

Si `claude` reste introuvable dans un nouveau terminal : un terminal intégré à un éditeur prend son PATH de l'éditeur, donc quittez puis rouvrez aussi l'éditeur.

### Rechercher des installations en conflit

Plusieurs installations de Claude Code peuvent provoquer des versions incohérentes ou des comportements inattendus. Vérifiez ce qui est installé :

#### macOS/Linux

Listez tous les binaires `claude` trouvés dans votre PATH :

```bash theme={null}
which -a claude
```

Si cela affiche `claude not found`, une ligne `no claude in`, ou rien du tout, aucun `claude` n'est dans votre PATH. Les vérifications suivantes montrent s'il y en a un d'installé quelque part.

Vérifiez les trois emplacements d'où peut venir un binaire `claude`. `~/.local/bin/claude` correspond à l'installateur natif, `~/.claude/local/` à une ancienne installation npm locale (*legacy*) créée par d'anciennes versions de Claude Code, et la liste globale npm montre une installation `-g` :

```bash theme={null}
ls -la ~/.local/bin/claude
```

Une installation native se présente comme un lien symbolique vers `~/.local/share/claude/versions/`. Un script ou un lien symbolique que vous avez créé vous-même à ce chemin est un lanceur personnalisé, que [la mise à jour automatique laisse en place](https://code.claude.com/docs/en/setup#auto-updates).

Si l'une des commandes `ls` affiche `No such file or directory`, ce n'est pas une erreur : rien n'est installé à cet emplacement, passez à la vérification suivante.

```bash theme={null}
ls -la ~/.claude/local/
```

```bash theme={null}
npm -g ls @anthropic-ai/claude-code 2>/dev/null
```

Si `ls -la ~/.local/bin/claude` a affiché `No such file or directory`, il n'y a pas d'installation native. Si vous n'avez pas installé Claude Code autrement (npm, Homebrew ou gestionnaire de paquets Linux, par exemple), [installez Claude Code](https://code.claude.com/docs/en/setup#install-claude-code). Si `~/.local/bin/claude` existe mais que `which -a claude` ne l'a pas listé, le dossier n'est pas dans votre PATH : voir [Vérifier votre PATH](#vérifier-votre-path).

#### Windows PowerShell

Listez tous les binaires `claude` trouvés dans votre PATH :

```powershell theme={null}
where.exe claude
```

Si cela affiche `INFO: Could not find files for the given pattern(s).`, aucun `claude` n'est dans votre PATH.

Vérifiez si l'installateur natif a placé un binaire :

```powershell theme={null}
Test-Path "$env:USERPROFILE\.local\bin\claude.exe"
```

* **`True`** : l'installation native est présente. Si `where.exe` n'a rien trouvé, son dossier n'est pas dans votre PATH : voir [Vérifier votre PATH](#vérifier-votre-path).
* **`False`** : il n'y a pas d'installation native. Si vous n'avez pas installé Claude Code autrement (npm ou WinGet, par exemple), [installez Claude Code](https://code.claude.com/docs/en/setup#install-claude-code).

#### Supprimer les installations en trop

Si vous trouvez plusieurs installations, n'en gardez qu'une. L'installation native, dans `~/.local/bin/claude` sur macOS/Linux ou `%USERPROFILE%\.local\bin\claude.exe` sous Windows, est recommandée. Supprimez les autres :

Désinstaller une installation npm globale :

```bash theme={null}
npm uninstall -g @anthropic-ai/claude-code
```

Supprimer l'ancienne installation npm locale (*legacy*) :

##### macOS/Linux

```bash theme={null}
rm -rf ~/.claude/local
```

##### Windows PowerShell

```powershell theme={null}
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\local"
```

Supprimer une installation Homebrew sur macOS. Si vous avez installé le cask `claude-code@latest`, remplacez par ce nom :

```bash theme={null}
brew uninstall --cask claude-code
```

Supprimer une installation WinGet sous Windows :

```powershell theme={null}
winget uninstall Anthropic.ClaudeCode
```

### Vérifier les permissions des répertoires

Une installation qui échoue pour une question de permissions nomme le chemin qu'elle n'a pas pu créer ou écrire. Sous Windows, l'installation écrit sous `%USERPROFILE%`, accessible en écriture à votre utilisateur par défaut, donc cette section s'y applique rarement.

Sur macOS et Linux, l'installation écrit dans ces emplacements :

* `~/.claude/downloads/` : là où la commande d'installation dépose le binaire téléchargé
* `~/.local/bin/` : le lanceur `claude`
* `~/.local/share/claude/` : chaque version qu'elle télécharge
* `~/.local/state/claude/` : ses fichiers de verrou (*lock files*)
* `~/.cache/claude/` : les téléchargements en cours de préparation (*staged*)
* [`~/.claude.json`](https://code.claude.com/docs/en/claude-directory) : votre fichier de configuration global, où l'installateur enregistre la méthode d'installation

Si vous définissez `XDG_DATA_HOME`, `XDG_STATE_HOME` ou `XDG_CACHE_HOME`, l'installation les utilise à la place de `~/.local/share`, `~/.local/state` et `~/.cache`. Si vous définissez [`CLAUDE_CONFIG_DIR`](https://code.claude.com/docs/en/env-vars), le fichier de configuration global se trouve sous ce répertoire au lieu de votre répertoire personnel.

Vérifiez si les répertoires sont accessibles en écriture :

```bash theme={null}
test -w ~/.local/bin && echo "writable" || echo "not writable"
test -w ~/.claude && echo "writable" || echo "not writable"
```

Si l'un des deux n'est pas accessible en écriture, créez le répertoire d'installation et faites de votre utilisateur son propriétaire :

```bash theme={null}
sudo mkdir -p ~/.local/bin
sudo chown -R $(whoami) ~/.local
```

### Vérifier que le binaire fonctionne

Si `claude --version` affiche une version mais que `claude` plante ou se bloque au démarrage, lancez ces vérifications pour cerner la cause. Si `claude --version` répond « command not found », allez d'abord à [Vérifier votre PATH](#vérifier-votre-path) ; les commandes ci-dessous supposent que `claude` est dans votre PATH.

Confirmez que le binaire existe et est exécutable :

#### macOS/Linux

```bash theme={null}
ls -la "$(command -v claude)"
```

#### Windows PowerShell

```powershell theme={null}
Get-Command claude | Select-Object Source
```

#### Bibliothèques partagées et exécution

Sous Linux, recherchez les bibliothèques partagées manquantes. Si `ldd` signale des bibliothèques manquantes, vous devrez peut-être installer des paquets système. Sur Alpine Linux et les autres distributions basées sur musl, voir [Alpine Linux setup](https://code.claude.com/docs/en/setup#alpine-linux-and-musl-based-distributions).

```bash theme={null}
ldd "$(command -v claude)" | grep "not found"
```

Confirmez que le binaire peut s'exécuter :

```bash theme={null}
claude --version
```

## Problèmes d'installation courants

Voici les problèmes d'installation les plus fréquents et leurs solutions.

### Le script d'installation renvoie du HTML au lieu d'un script shell

La commande d'installation échoue avec l'une de ces erreurs lorsque ce qu'elle a téléchargé n'est pas le script d'installation.

**Bash ou Zsh** : l'erreur cite la première ligne de la page renvoyée.

```text theme={null}
bash: line 1: syntax error near unexpected token `<'
bash: line 1: `<!DOCTYPE html>'
```

**PowerShell, erreurs d'analyse** : les erreurs pointent dans la page renvoyée, `iex` essayant d'exécuter du HTML et du CSS comme du PowerShell.

```text theme={null}
iex : At line:1 char:2310
+ ... igin="anonymous"/><script type="text/javascript">!function(o,c){var n ...
Missing argument in parameter list.
...
```

Le libellé varie selon la version de PowerShell et la langue du système : vous pouvez voir à la place `Missing expression after unary operator '--'` ou une `ParserError` avec `ParseException`. Ce sont les balises HTML ou le CSS dans le texte cité qui identifient cette panne. Si vous téléchargez plutôt avec `-OutFile install.ps1`, le fichier enregistré est la même page web : cela n'aide donc pas non plus.

**PowerShell, `System.Xml.XmlDocument`** : l'erreur nomme ce type au lieu de citer la page.

```text theme={null}
System.Xml.XmlDocument : The term 'System.Xml.XmlDocument' is not recognized as the name of a cmdlet, function, script
file, or operable program.
```

Quand `irm` peut analyser la réponse comme du XML, il renvoie un objet XML au lieu de texte, et `iex` tente alors d'exécuter le nom de type de cet objet comme une commande. Le script d'installation est du code PowerShell qui ne s'analyse pas comme du XML : cette erreur signifie donc aussi que la réponse n'était pas le script. Le libellé autour du nom de type varie selon la version de PowerShell et la langue du système, mais `System.Xml.XmlDocument` reste identique : repérez donc le nom de type.

**CMD** : vous voyez cette erreur, suivie du HTML de la page renvoyée.

```text theme={null}
< was unexpected at this time.

C:\Users\you><!DOCTYPE html>...
```

La première ligne s'affiche dans la langue de votre système : repérez donc le HTML qui la suit.

**Un 403 sans page** : selon le routage de la requête, curl signale un statut 403 sans corps HTML.

```text theme={null}
curl: (22) The requested URL returned error: 403
```

Tous ces cas signifient que l'URL d'installation a renvoyé une page web, un document XML ou un statut d'erreur au lieu du script d'installation. Si la sortie d'erreur cite « App unavailable in region », Claude Code n'est pas disponible dans votre pays. Voir les [pays pris en charge](https://www.anthropic.com/supported-countries).

Un 403 nu, sans corps, a souvent la même cause, mais il peut aussi venir d'un proxy ou d'un pare-feu d'entreprise qui bloque le téléchargement. Si vous êtes dans un pays pris en charge et voyez toujours le 403, suivez [Vérifier la connectivité réseau](#vérifier-la-connectivité-réseau) avant d'essayer les installateurs alternatifs ci-dessous, puisque ceux-ci contactent les mêmes hôtes.

Sinon, cela peut venir de problèmes réseau, d'un routage régional ou d'une interruption temporaire du service.

**Solutions :**

1. **Réessayer après quelques minutes** : le problème est souvent temporaire. Attendez puis relancez la commande d'origine.

2. **Utiliser une autre méthode d'installation** : contrairement à une installation native, une installation Homebrew ou WinGet [ne se met pas à jour toute seule par défaut](https://code.claude.com/docs/en/setup#auto-updates).

   Sur macOS, installer via Homebrew :

   ```bash theme={null}
   brew install --cask claude-code
   ```

   Sous Windows, installer via WinGet :

   ```powershell theme={null}
   winget install Anthropic.ClaudeCode
   ```

   Lancez ensuite `claude --version` pour confirmer : la commande affiche un numéro de version tel que `2.1.211 (Claude Code)`. Si le shell indique que `claude` est introuvable, ouvrez une nouvelle fenêtre de terminal et réessayez : la session depuis laquelle vous avez installé garde son ancien `PATH`.

### `command not found: claude` après l'installation

L'installation s'est terminée mais `claude` ne fonctionne pas. L'erreur exacte varie selon la plateforme :

| Plateforme | Message d'erreur |
| :- | :- |
| macOS | `zsh: command not found: claude` |
| Linux | `bash: claude: command not found` |
| Windows CMD | `'claude' is not recognized as an internal or external command` |
| PowerShell | `claude : The term 'claude' is not recognized as the name of a cmdlet` |

Sous Windows, si l'erreur est apparue juste après une mise à jour de Claude Code, voir [restaurer `claude.exe` depuis sa sauvegarde](#claudeexe-manquant-après-une-mise-à-jour-sous-windows).

Sinon, voir [Vérifier votre PATH](#vérifier-votre-path) pour le correctif propre à chaque plateforme.

### `permission denied` en ajoutant au PATH

Si la commande `echo` qui ajoute `~/.local/bin` à votre PATH affiche `zsh: permission denied: /Users/you/.zshrc` ou `bash: /home/you/.bashrc: Permission denied`, votre utilisateur ne peut pas écrire dans ce fichier et rien n'a été enregistré. Dans votre terminal, vérifiez à qui appartient le fichier, en remplaçant `~/.zshrc` par le nom de fichier de votre shell :

```bash theme={null}
ls -l ~/.zshrc
```

Le troisième champ de la sortie est le propriétaire.

* **Le propriétaire est un autre utilisateur, comme `root`** : prenez-en la propriété avec `sudo chown $(whoami) ~/.zshrc`, ce qui nécessite des droits d'administrateur.
* **Le propriétaire est vous** : le fichier est en lecture seule. Rendez-le modifiable avec `chmod u+w ~/.zshrc`.

Relancez ensuite les deux commandes PATH correspondant à votre shell, dans [Vérifier votre PATH](#vérifier-votre-path).

### `curl: (56) Failure writing output to destination`

La commande `curl ... | bash` télécharge le script et le transmet par un pipe à Bash pour exécution. Cette erreur, comme l'erreur apparentée `curl: (23) Failure writing output to destination`, signifie que Bash n'a pas reçu le script complet. Le code de sortie 56 indique que le téléchargement lui-même a été interrompu, et le code 23 que curl n'a pas pu écrire ce qu'il recevait dans le pipe, généralement parce que Bash s'est arrêté prématurément.

Testez que vous pouvez atteindre `downloads.claude.ai` avec la vérification de [Vérifier la connectivité réseau](#vérifier-la-connectivité-réseau). Si vous avez atteint le serveur, la panne d'origine était probablement intermittente ; relancez la commande d'installation. Vous pouvez aussi [essayer une autre méthode d'installation](https://code.claude.com/docs/en/setup#install-claude-code).

### Cask Homebrew indisponible ou obsolète

Homebrew signale `Error: Cask 'claude-code' is unavailable: No Cask with this name exists` lorsque votre copie locale de l'index des casks Homebrew est antérieure à la publication du cask. Rafraîchissez l'index et réessayez :

```bash theme={null}
brew update
brew install --cask claude-code
```

Si Homebrew installe une version de Claude Code plus ancienne que prévu, c'est généralement le même index périmé qui en est la cause. Le cask `claude-code` suit le canal stable et a en général environ une semaine de retard sur la dernière version ; pour la version la plus récente, lancez plutôt `brew install --cask claude-code@latest`. Voir [Configure release channel](https://code.claude.com/docs/en/setup#configure-release-channel) pour la différence entre les deux casks.

### `Cask 'claude-code@latest' is not installed`

Homebrew propose deux casks, `claude-code` et `claude-code@latest`. Lancer `brew upgrade --cask claude-code@latest` alors que ce cask n'est pas celui installé affiche `Error: Cask 'claude-code@latest' is not installed.` Pour voir quel cask vous avez, lancez ceci dans votre terminal :

```bash theme={null}
brew list --cask | grep claude-code
```

Mettez à niveau le cask affiché. Si rien ne s'affiche, aucun des deux casks n'est installé.

### Erreurs de connexion TLS ou SSL

Des erreurs comme celles-ci signifient que la négociation TLS (*handshake*) a échoué :

* `curl: (35) TLS connect error`
* `schannel: next InitializeSecurityContext failed`
* `Could not create SSL/TLS secure channel` (PowerShell)
* `Could not establish trust relationship for the SSL/TLS secure channel` (PowerShell)

Pour `CRYPT_E_NO_REVOCATION_CHECK` ou `CRYPT_E_REVOCATION_OFFLINE`, allez à l'étape 4.

**Solutions :**

1. **Mettre à jour les certificats CA du système** :

   Sur Ubuntu/Debian :

   ```bash theme={null}
   sudo apt-get update && sudo apt-get install ca-certificates
   ```

   Sur macOS, le curl système utilise le magasin de confiance du Trousseau (Keychain) ; mettre à jour macOS lui-même met à jour les certificats racine.

2. **Dans Windows PowerShell 5.1, activer TLS 1.2** :

   ```powershell theme={null}
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   ```

   Puis lancez l'installateur dans la même fenêtre :

   ```powershell theme={null}
   irm https://claude.ai/install.ps1 | iex
   ```

3. **Rechercher une interférence d'un proxy ou d'un pare-feu** : les proxys d'entreprise qui font de l'inspection TLS peuvent provoquer ces erreurs, y compris `unable to get local issuer certificate` et `SELF_SIGNED_CERT_IN_CHAIN`. Pour l'étape d'installation, faites en sorte que le téléchargement fasse confiance à la CA du proxy de votre entreprise :

   **macOS/Linux :**

   ```bash theme={null}
   curl --cacert /path/to/corporate-ca.pem -fsSL https://claude.ai/install.sh | bash
   ```

   **Windows PowerShell :** l'installateur PowerShell télécharge via .NET, qui valide TLS avec le magasin de certificats Windows. Demandez à votre équipe informatique d'ajouter le certificat CA du proxy au magasin Windows s'il n'y est pas déjà, puis lancez l'installateur :

   ```powershell theme={null}
   irm https://claude.ai/install.ps1 | iex
   ```

   Pour Claude Code lui-même une fois installé, définissez `NODE_EXTRA_CA_CERTS` pour que les requêtes API fassent confiance au même bundle :

   **macOS/Linux :**

   ```bash theme={null}
   export NODE_EXTRA_CA_CERTS=/path/to/corporate-ca.pem
   ```

   **Windows PowerShell :**

   ```powershell theme={null}
   $env:NODE_EXTRA_CA_CERTS = 'C:\path\to\corporate-ca.pem'
   ```

   Demandez le fichier de certificat à votre équipe informatique si vous ne l'avez pas. Vous pouvez aussi essayer sur une connexion directe pour confirmer que le proxy est en cause.

4. **Sous Windows, contourner les vérifications de révocation bloquées**. Les erreurs `CRYPT_E_NO_REVOCATION_CHECK (0x80092012)` et `CRYPT_E_REVOCATION_OFFLINE (0x80092013)` signifient que curl a atteint le serveur mais que votre réseau bloque la consultation de révocation du certificat, ce qui est courant derrière les pare-feux d'entreprise. Si la commande en échec est le `curl` qui télécharge `install.cmd`, relancez-la depuis une invite de commandes en ajoutant `--ssl-revoke-best-effort` :

   ```batch theme={null}
   curl --ssl-revoke-best-effort -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
   ```

   Quand les propres téléchargements du script rencontrent les mêmes erreurs, il les relance automatiquement avec une vérification de révocation « best-effort » : le flag n'est donc nécessaire que sur la commande que vous lancez vous-même. La vérification best-effort tolère un serveur de révocation injoignable, mais rejette toujours un certificat connu comme révoqué, comme le font les navigateurs. Vous pouvez aussi éviter totalement la vérification de révocation de curl en lançant l'installateur PowerShell depuis PowerShell : il télécharge via .NET et n'échoue pas quand le serveur de révocation est injoignable :

   ```powershell theme={null}
   irm https://claude.ai/install.ps1 | iex
   ```

   Vous pouvez aussi installer avec `winget install Anthropic.ClaudeCode`, qui se passe complètement de curl.

### `Failed to fetch version from downloads.claude.ai`

L'installateur n'a pas pu joindre le serveur de téléchargement. Cela signifie généralement que `downloads.claude.ai` est bloqué sur votre réseau. Voir [Vérifier la connectivité réseau](#vérifier-la-connectivité-réseau).

### La connexion s'est interrompue pendant le téléchargement de la mise à jour

La connexion au serveur de téléchargement s'est fermée pendant que `claude install` ou `claude update` récupérait le binaire Claude Code, et les nouvelles tentatives n'ont pas suffi. Claude Code relance le téléchargement quand la connexion tombe, que le transfert se bloque ou que le fichier téléchargé échoue à la vérification de somme de contrôle (*checksum*), jusqu'à trois tentatives au total. Une erreur HTTP complète, comme un 404, n'est pas relancée puisque le serveur a déjà répondu. Avant la v2.1.202, une seule coupure de connexion faisait échouer immédiatement le téléchargement avec l'erreur brute `aborted`, sans nouvelle tentative.

```text theme={null}
The connection dropped while downloading the update (attempt 3/3: aborted). Check your network — proxies sometimes cut off large downloads.
```

Le texte entre parenthèses indique quelle tentative a échoué et l'erreur réseau sous-jacente. `claude update` fait précéder le message de `Error: Failed to install native update` sur stderr.

Un téléchargement qui reste connecté mais ne se termine pas en 10 minutes échoue à la place avec `Download timed out: exceeded the total deadline`. Claude Code ne relance pas un téléchargement expiré, car une connexion trop lente pour finir dans le délai ne finira pas non plus sur une relance immédiate. Les étapes ci-dessous s'appliquent aux deux messages.

Un proxy ou une passerelle peut fermer un long transfert avant la fin, et le binaire Claude Code est un téléchargement volumineux.

**Que faire :**

* Relancez `claude update`. Sur un réseau par ailleurs sain, le téléchargement réussit généralement au passage suivant. Pour le message d'expiration, relancez-le depuis un réseau plus rapide ou moins bridé.
* Si votre réseau exige un proxy, définissez `HTTPS_PROXY` avant de lancer l'installateur ou `claude update`. Voir [Vérifier la connectivité réseau](#vérifier-la-connectivité-réseau).
* Si un proxy d'entreprise continue de fermer le transfert, demandez à votre équipe réseau d'autoriser le téléchargement complet depuis `downloads.claude.ai`. Voir [Network access requirements](https://code.claude.com/docs/en/network-config#network-access-requirements).
* Lancez `claude doctor` depuis votre shell pour obtenir un diagnostic de l'installation.

### Mauvaise commande d'installation sous Windows

Si vous voyez `'irm' is not recognized`, `The token '&&' is not a valid statement separator`, `A parameter cannot be found that matches parameter name 'fsSL'` ou `'bash' is not recognized as the name of a cmdlet`, vous avez copié la commande d'installation d'un autre shell ou d'un autre système d'exploitation. Si la commande affiche le texte du script au lieu d'installer quoi que ce soit, vous n'en avez exécuté qu'une partie.

* **`irm` non reconnu** : vous êtes dans CMD, pas dans PowerShell. Deux options :

  Ouvrez PowerShell en cherchant « PowerShell » dans le menu Démarrer, puis lancez la commande d'installation d'origine :

  ```powershell theme={null}
  irm https://claude.ai/install.ps1 | iex
  ```

  Ou restez dans CMD et utilisez plutôt l'installateur CMD :

  ```batch theme={null}
  curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
  ```

* **`&&` n'est pas un séparateur d'instructions valide** : vous êtes dans PowerShell mais avez lancé la commande de l'installateur CMD. Utilisez l'installateur PowerShell :

  ```powershell theme={null}
  irm https://claude.ai/install.ps1 | iex
  ```

* **`A parameter cannot be found that matches parameter name 'fsSL'`** : vous avez lancé l'installateur macOS/Linux `curl -fsSL ... | bash` dans Windows PowerShell, où `curl` est un alias de `Invoke-WebRequest` qui rejette les flags `-fsSL`. Utilisez plutôt l'installateur PowerShell :

  ```powershell theme={null}
  irm https://claude.ai/install.ps1 | iex
  ```

* **`bash` non reconnu** : vous avez lancé l'installateur macOS/Linux sous Windows. Utilisez plutôt l'installateur PowerShell :

  ```powershell theme={null}
  irm https://claude.ai/install.ps1 | iex
  ```

* **La commande affiche le texte du script au lieu d'installer** : vous avez lancé la moitié « téléchargement » de la commande sans la partie qui l'exécute. `irm https://claude.ai/install.ps1` seul affiche le script téléchargé dans le terminal. Passez-le par un pipe à `iex` pour l'exécuter :

  ```powershell theme={null}
  irm https://claude.ai/install.ps1 | iex
  ```

  Dans CMD, `curl -fsSL https://claude.ai/install.cmd` sans `-o` affiche le script batch au lieu de l'enregistrer. Lancez la commande complète :

  ```batch theme={null}
  curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
  ```

Quel que soit l'installateur utilisé, confirmez qu'il a fonctionné : ouvrez un nouveau terminal et lancez `claude --version`, qui affiche un numéro de version tel que `2.1.211 (Claude Code)`.

### `running scripts is disabled on this system`

Installer ou lancer Claude Code via npm sous Windows peut échouer avec une `SecurityError` :

```text theme={null}
npm : File C:\Program Files\nodejs\npm.ps1 cannot be loaded because running scripts is disabled on this system. For more information, see about_Execution_Policies at https:/go.microsoft.com/fwlink/?LinkID=135170.
...
    + CategoryInfo          : SecurityError: (:) [], PSSecurityException
```

La même erreur nomme `claude.ps1` quand vous lancez `claude` après une installation npm. La stratégie d'exécution (*execution policy*) de PowerShell bloque les scripts lanceurs `.ps1` que npm crée pour ses commandes. Cette stratégie s'applique aux fichiers de script : elle n'affecte donc pas l'installateur PowerShell `irm https://claude.ai/install.ps1 | iex`, qui exécute directement le texte téléchargé.

**Solutions :**

1. **Autoriser les scripts créés localement pour votre utilisateur**, puis réessayer :

   ```powershell theme={null}
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
   ```

2. **Appeler plutôt le lanceur `.cmd`** : `npm.cmd` et `claude.cmd` font le même travail, et la stratégie ne les couvre pas.
3. **Utiliser l'[installateur PowerShell](https://code.claude.com/docs/en/setup#install-claude-code)** au lieu de npm. Il installe un binaire plutôt qu'un script `.ps1`.

### `The process cannot access the file` pendant l'installation Windows

Si l'installateur PowerShell échoue avec `Failed to download binary: The process cannot access the file ... because it is being used by another process`, l'installateur n'a pas pu écrire dans `%USERPROFILE%\.claude\downloads`. Cela signifie généralement qu'une tentative d'installation précédente est encore en cours, ou qu'un antivirus analyse un binaire partiellement téléchargé dans ce dossier.

Fermez les autres fenêtres PowerShell qui exécutent l'installateur et attendez que les analyses antivirus libèrent le fichier. Supprimez ensuite le dossier de téléchargements et relancez l'installateur :

```powershell theme={null}
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\downloads"
irm https://claude.ai/install.ps1 | iex
```

### `claude.exe` manquant après une mise à jour sous Windows

Si votre terminal signale `'claude' is not recognized` juste après une mise à jour de Claude Code sous Windows, vérifiez si `%USERPROFILE%\.local\bin` contient toujours `claude.exe`. Si ce répertoire n'est pas du tout dans votre PATH, voir plutôt [Vérifier votre PATH](#vérifier-votre-path). Pour se mettre à jour sous Windows, Claude Code renomme le `claude.exe` existant en une sauvegarde mise de côté, puis place la nouvelle version à sa place. Si ce déplacement échoue et que Claude Code ne parvient pas non plus à renommer la sauvegarde en sens inverse, le répertoire conserve la sauvegarde mais n'a plus de `claude.exe`.

La sauvegarde est un fichier du même répertoire dont le nom commence par `claude.exe.old.` suivi d'un horodatage numérique. Lancez ceci dans PowerShell pour renommer la sauvegarde la plus récente en `claude.exe` :

```powershell theme={null}
Get-ChildItem "$env:USERPROFILE\.local\bin\claude.exe.old.*" | Sort-Object Name | Select-Object -Last 1 | Rename-Item -NewName claude.exe
```

Lancez ensuite `claude --version` pour confirmer la correction. Un `claude.exe` restauré affiche un numéro de version.

S'il n'y a aucun fichier `claude.exe.old.*`, ou si `claude` échoue toujours après le renommage, réinstallez :

```powershell theme={null}
irm https://claude.ai/install.ps1 | iex
```

Avant la v2.1.281, Claude Code pouvait supprimer la sauvegarde alors que `claude.exe` manquait encore.

### Installation tuée sur des serveurs Linux à faible mémoire

Un message `Killed` pendant l'installation signifie généralement que le tueur de processus en manque de mémoire de Linux (*OOM killer*) a terminé l'étape `claude install` parce que le système n'avait plus de mémoire libre. C'est courant sur les petits VPS et instances cloud. Le script d'installation indique la cause et se termine avec le code 137. Dans cet exemple, le numéro de ligne et l'identifiant de processus varient selon la version et l'exécution :

```text theme={null}
Setting up Claude Code...
bash: line 183: 34803 Killed    "$binary_path" install ${TARGET:+"$TARGET"}
Installation was killed before it could finish (exit code 137). This usually means the system ran out of memory.
Claude Code needs roughly 512MB of free memory to install. Free up memory, then run this script again.
```

L'installation nécessite environ 512 Mo de mémoire libre, et l'exécution de Claude Code davantage. Voir la [configuration requise](https://code.claude.com/docs/en/setup#system-requirements).

**Solutions :**

1. **Ajouter de l'espace d'échange (swap)** si votre serveur a peu de RAM. Le swap utilise de l'espace disque comme mémoire de débordement, ce qui permet à l'installation d'aboutir même avec peu de RAM physique.

   Créez un fichier de swap de 2 Go et activez-le :

   ```bash theme={null}
   sudo fallocate -l 2G /swapfile
   sudo chmod 600 /swapfile
   sudo mkswap /swapfile
   sudo swapon /swapfile
   ```

   Puis relancez l'installation :

   ```bash theme={null}
   curl -fsSL https://claude.ai/install.sh | bash
   ```

2. **Fermer d'autres processus** pour libérer de la mémoire avant d'installer.

3. **Utiliser une instance plus grande** si possible. Claude Code nécessite au moins 4 Go de RAM.

### Installation was killed before it could finish

Le script d'installation signale quand l'étape `claude install` est terminée par un signal. Sous Linux, le code de sortie 137 signifie que le processus a reçu SIGKILL, et sur un hôte à faible mémoire, c'est généralement le tueur OOM du noyau. Le script affiche cette explication et se termine avec le code 137 :

```text theme={null}
Installation was killed before it could finish (exit code 137). This usually means the system ran out of memory.
Claude Code needs roughly 512MB of free memory to install. Free up memory, then run this script again.
```

Pour tout autre signal fatal, ainsi que pour le code 137 sur macOS, le script affiche `Installation was killed before it could finish (exit code <N>)` avec le code de sortie réel, sans l'explication sur le manque de mémoire. Le message vient du script d'installation utilisé par macOS et Linux, qui couvre aussi les installations dans WSL ; les scripts d'installation Windows natifs ne l'affichent jamais. Avant la v2.1.200, le script se terminait avec seulement la ligne brute `Killed` du shell.

**Que faire :**

* Arrêtez d'autres processus pour libérer de la mémoire, puis relancez l'installateur
* Ajoutez du swap ou passez à une instance plus grande. Voir [Installation tuée sur des serveurs Linux à faible mémoire](#installation-tuée-sur-des-serveurs-linux-à-faible-mémoire) pour les commandes de fichier de swap.

### L'installation se bloque dans Docker

Lors de l'installation de Claude Code dans un conteneur Docker, installer en tant que root dans `/` peut provoquer des blocages.

**Solutions :**

1. **Définir un répertoire de travail** avant de lancer l'installateur. Lancé depuis `/`, l'installateur parcourt tout le système de fichiers, ce qui entraîne une consommation de mémoire excessive. Définir `WORKDIR` limite ce parcours à un petit répertoire :

   ```dockerfile theme={null}
   WORKDIR /tmp
   RUN curl -fsSL https://claude.ai/install.sh | bash
   ```

2. **Donner plus de mémoire à Docker** si vous utilisez Docker Desktop. Les conteneurs de build partagent la mémoire allouée à la machine virtuelle de Docker Desktop : ouvrez **Settings > Resources** dans Docker Desktop, augmentez la limite de mémoire, puis relancez le build.

### `Raw mode is not supported` pendant l'installation

Lorsque les [réglages gérés par le serveur](https://code.claude.com/docs/en/server-managed-settings) de votre organisation contiennent des changements qui nécessitent une [approbation de sécurité](https://code.claude.com/docs/en/server-managed-settings#security-approval-dialogs), les versions de Claude Code antérieures à 2.1.246 tentent d'afficher la boîte de dialogue d'approbation pendant `claude install`. Cette boîte de dialogue a besoin d'un terminal sur stdin. Quand l'installateur lance `claude install` depuis un pipe, comme le fait `curl -fsSL https://claude.ai/install.sh | bash`, stdin est le pipe et non un terminal : l'installation échoue alors avec une erreur contenant `Raw mode is not supported`.

Claude Code v2.1.246 et ultérieures n'affichent pas la boîte de dialogue pendant `claude install` ou `claude update`. La commande s'exécute avec les réglages que vous avez approuvés en dernier, et Claude Code affiche la boîte de dialogue lors de votre prochaine session interactive. Si la configuration de démarrage de votre organisation [attend la récupération des réglages](https://code.claude.com/docs/en/server-managed-settings#enforce-fail-closed-startup), par exemple quand elle définit `forceRemoteSettingsRefresh`, la boîte de dialogue apparaît toujours pendant ces commandes, et une installation lancée depuis un pipe échoue toujours.

Dans toute autre configuration, relancer l'installateur permet de dépasser cette erreur, car le script exécute la commande `install` de la dernière version même si vous demandez l'installation d'une version plus ancienne. Relancez la commande de votre plateforme :

#### macOS/Linux

```bash theme={null}
curl -fsSL https://claude.ai/install.sh | bash
```

#### Windows PowerShell

```powershell theme={null}
irm https://claude.ai/install.ps1 | iex
```

`claude --version` affiche la version installée par la relance.

### `claude update` ou `claude doctor` se bloque

`claude update` et `claude doctor` parcourent vos fichiers de configuration du shell à la recherche d'un alias `claude` obsolète : `~/.zshrc`, `~/.bashrc` et `~/.config/fish/config.fish`, plus, sur macOS, le premier fichier existant parmi `~/.bash_profile`, `~/.bash_login` ou `~/.profile`. Si vous définissez `ZDOTDIR`, le fichier Zsh est `$ZDOTDIR/.zshrc`. Quand l'un de ces chemins est un répertoire, Claude Code l'ignore et les deux commandes se terminent normalement. Avant la v2.1.214, un répertoire à l'un de ces chemins bloquait les deux commandes et laissait vide la section « System diagnostics » de `/status`. `claude doctor` se bloquait sans aucune sortie ; `claude update` se bloquait juste après avoir affiché `Checking for updates`.

Si vous rencontrez ce blocage sur une version antérieure, trouvez le répertoire. Dans la sortie de cette commande, une ligne commençant par `d` indique que ce chemin est un répertoire. Une ligne `No such file or directory` signifie que rien n'existe à ce chemin et n'est pas la cause :

```bash theme={null}
ls -ld ~/.zshrc ~/.bashrc ~/.bash_profile ~/.bash_login ~/.profile ~/.config/fish/config.fish
```

Déplacez le répertoire ailleurs, ou mettez à jour vers la v2.1.214 ou ultérieure. Comme `claude update` se bloque sur les versions concernées, faites la mise à jour en relançant le [script d'installation](https://code.claude.com/docs/en/setup#install-claude-code).

### Claude Desktop remplace la commande `claude` sous Windows

Si vous avez installé une ancienne version de Claude Desktop, elle peut avoir enregistré un `Claude.exe` dans le répertoire `WindowsApps` qui a priorité dans le PATH sur la CLI Claude Code. Lancer `claude` ouvre alors l'application Desktop au lieu de la CLI.

Mettez à jour Claude Desktop vers la dernière version pour corriger ce problème.

### Claude Code on Windows requires either Git for Windows (for bash) or PowerShell

(« Claude Code sous Windows nécessite soit Git for Windows (pour bash), soit PowerShell ».) Git for Windows est facultatif. Claude Code utilise l'[outil PowerShell](https://code.claude.com/docs/en/tools-reference#powershell-tool) quand Git Bash est absent : cette erreur signifie donc qu'aucun des deux shells n'a été trouvé.

**Si PowerShell manque dans votre PATH**, son emplacement par défaut est `C:\Windows\System32\WindowsPowerShell\v1.0\`. Ajoutez ce répertoire à votre `PATH`, ou installez [PowerShell 7](https://aka.ms/powershell), qui fournit `pwsh`.

**Pour installer plutôt Git for Windows**, téléchargez-le depuis [git-scm.com/downloads/win](https://git-scm.com/downloads/win). Pendant l'installation, sélectionnez « Add to PATH ». Redémarrez votre terminal après l'installation. L'installer active l'outil Bash, utile pour travailler avec des scripts et outils basés sur Bash.

**Si Git est déjà installé** mais que Claude Code ne le trouve pas, comparez son emplacement aux endroits que Claude Code vérifie. Quand `CLAUDE_CODE_GIT_BASH_PATH` n'est pas défini, Claude Code cherche `bash.exe` dans cet ordre :

1. Les emplacements d'installation par défaut `C:\Program Files\Git` et `C:\Program Files (x86)\Git`.
2. Le `git` de votre `PATH`, en utilisant le `bin\bash.exe` de cette installation de Git.

À l'étape 2, Claude Code ignore un `git` situé dans le dossier depuis lequel vous avez lancé Claude Code, ou en dessous dans un chemin contenant `node_modules` ou un dossier d'environnement virtuel comme `.venv` ou `env` — par exemple `C:\dev\env\myproject\Git` si vous avez lancé depuis `C:\dev\env\myproject`. Cela évite que Claude Code exécute un exécutable placé là par un projet. Si votre Git se trouve à un tel emplacement, faites pointer `CLAUDE_CODE_GIT_BASH_PATH` vers lui.

**Pour indiquer à Claude Code une installation de Git précise**, trouvez-la en lançant `where.exe git` dans PowerShell, puis définissez le chemin `bin\bash.exe` de cette installation comme `CLAUDE_CODE_GIT_BASH_PATH` dans votre [fichier settings.json](https://code.claude.com/docs/en/settings) :

```json theme={null}
{
  "env": {
    "CLAUDE_CODE_GIT_BASH_PATH": "C:\\Program Files\\Git\\bin\\bash.exe"
  }
}
```

**Si `CLAUDE_CODE_GIT_BASH_PATH` est défini avec le bon chemin et que le fichier existe** mais que Claude Code ne l'utilise toujours pas, vérifiez d'abord le nom du fichier. Claude Code n'accepte qu'un fichier nommé `bash.exe`, `sh.exe`, `bash` ou `sh` ; avec tout autre nom, comme le lanceur `git-bash.exe` de Git for Windows, il ignore la variable et détecte automatiquement Git Bash comme si elle n'était pas définie, en journalisant un avertissement visible avec `--debug`. Un chemin inexistant donne lieu au même repli et au même avertissement. Avant la v2.1.219, Claude Code utilisait comme shell n'importe quel fichier existant sans vérifier son nom, et s'arrêtait au démarrage avec `Claude Code was unable to find CLAUDE_CODE_GIT_BASH_PATH path` quand le chemin n'existait pas.

Si le nom du fichier est correct, un logiciel de sécurité des postes de travail (AppLocker, stratégies de restriction logicielle de stratégie de groupe, agents EDR) interfère peut-être. Demandez à votre équipe informatique d'autoriser (*allowlist*) `claude.exe` et les processus qu'il lance, y compris `cmd.exe` et `bash.exe`, dans votre politique de protection des postes de travail.

### Claude Code does not support 32-bit Windows

(« Claude Code ne prend pas en charge Windows 32 bits ».) Windows propose deux entrées PowerShell dans le menu Démarrer : `Windows PowerShell` et `Windows PowerShell (x86)`. L'entrée x86 s'exécute en processus 32 bits et déclenche cette erreur même sur une machine 64 bits. Pour savoir dans quel cas vous êtes, lancez ceci dans la fenêtre même qui a produit l'erreur :

```powershell theme={null}
[Environment]::Is64BitOperatingSystem
```

Si cela affiche `True`, votre système d'exploitation convient. Fermez la fenêtre, ouvrez `Windows PowerShell` sans le suffixe x86, et relancez la commande d'installation.

Si cela affiche `False`, vous êtes sur une édition 32 bits de Windows. Claude Code nécessite un système d'exploitation 64 bits. Voir la [configuration requise](https://code.claude.com/docs/en/setup#system-requirements).

### Incompatibilité de binaire musl ou glibc sous Linux

Si vous voyez après l'installation des erreurs de bibliothèques partagées manquantes comme `libstdc++.so.6` ou `libgcc_s.so.1`, l'installateur a peut-être téléchargé la mauvaise variante de binaire pour votre système.

```text theme={null}
Error loading shared library libstdc++.so.6: No such file or directory
```

Cela peut arriver sur des systèmes basés sur glibc où des paquets de compilation croisée musl sont installés, ce qui conduit l'installateur à détecter à tort un système musl.

**Solutions :**

1. **Vérifier quelle libc votre système utilise** :

   ```bash theme={null}
   ldd --version 2>&1 | head -1
   ```

   Une sortie mentionnant `GNU libc` ou `GLIBC` signifie glibc. Une sortie mentionnant `musl` signifie musl.

2. **Si vous êtes sur glibc mais avez reçu le binaire musl**, supprimez l'installation et réinstallez. Vous pouvez aussi télécharger manuellement le bon binaire à l'aide du manifeste `https://downloads.claude.ai/claude-code-releases/{VERSION}/manifest.json`. Ouvrez une [issue GitHub](https://github.com/anthropics/claude-code/issues) avec la sortie de `ldd --version` et de `ls /lib/libc.musl*`.

3. **Si vous êtes réellement sur musl**, comme Alpine Linux, installez les paquets requis :

   ```bash theme={null}
   apk add libgcc libstdc++ ripgrep
   ```

   Sur Alpine, `ripgrep` se trouve dans le dépôt community. Si `apk` signale que le paquet est absent, voir [Alpine Linux setup](https://code.claude.com/docs/en/setup#alpine-linux-and-musl-based-distributions).

### `Illegal instruction`

Si lancer `claude` ou l'installateur affiche `Illegal instruction`, le binaire natif utilise des instructions CPU que votre processeur ne prend pas en charge. Il y a deux causes distinctes.

**Incompatibilité d'architecture.** L'installateur a téléchargé le mauvais binaire, par exemple x86 sur un serveur ARM. Vérifiez avec `uname -m` sur macOS ou Linux, ou `$env:PROCESSOR_ARCHITECTURE` dans PowerShell. Si le résultat ne correspond pas au binaire reçu, [ouvrez une issue GitHub](https://github.com/anthropics/claude-code/issues) avec la sortie.

**Jeu d'instructions AVX manquant.** Si votre architecture est correcte mais que vous voyez toujours `Illegal instruction`, votre CPU n'a probablement pas AVX ou une autre instruction requise par le binaire. Cela concerne en gros les processeurs Intel et AMD antérieurs à 2013, ainsi que les machines virtuelles dont l'hyperviseur ne transmet pas AVX à l'invité.

Sur un VPS ou une VM, lancez `grep -m1 -ow avx /proc/cpuinfo` ; un résultat vide signifie qu'AVX n'est pas disponible pour l'invité.

Il n'existe pas de contournement pour le binaire natif ; suivez l'[issue #50384](https://github.com/anthropics/claude-code/issues/50384) pour l'état d'avancement, et indiquez votre modèle de CPU lors d'un signalement, obtenu avec `grep -m1 "model name" /proc/cpuinfo` sous Linux ou `sysctl -n machdep.cpu.brand_string` sur macOS.

Les méthodes d'installation alternatives téléchargent le même binaire natif et ne résolvent aucune des deux causes.

### `dyld: cannot load` sur macOS

Si vous voyez `dyld: Symbol not found`, `dyld: cannot load` ou `Abort trap: 6` pendant l'installation, le binaire est incompatible avec votre version de macOS ou votre matériel.

Une erreur `Symbol not found` qui fait référence à `libicucore` signifie que votre version de macOS est plus ancienne que ce que le binaire prend en charge :

```text theme={null}
dyld: Symbol not found: _ubrk_clone
  Referenced from: claude-darwin-x64 (which was built for Mac OS X 13.0)
  Expected in: /usr/lib/libicucore.A.dylib
```

Le chargeur peut aussi rejeter les commandes de chargement (*load commands*) du binaire, ce qui signifie également que votre version de macOS est trop ancienne :

```text theme={null}
dyld: cannot load 'claude-2.1.42-darwin-x64' (load command 0x80000034 is unknown)
Abort trap: 6
```

**Solutions :**

1. **Vérifier votre version de macOS** : Claude Code nécessite macOS 13.0 ou ultérieur. Ouvrez le menu Pomme et choisissez « À propos de ce Mac » pour connaître votre version.

2. **Mettre à jour macOS** si vous êtes sur une version plus ancienne. Le binaire utilise des commandes de chargement et des bibliothèques système que les anciennes versions de macOS ne prennent pas en charge. Les méthodes d'installation alternatives comme Homebrew téléchargent le même binaire et ne résoudront pas cette erreur.

### `Bus error` pendant qu'une session tourne

Si une session en cours se termine et que votre shell affiche `Bus error`, une cause possible est que Claude Code ne pouvait plus lire son propre fichier exécutable sur le disque. Par exemple, le fichier a été tronqué, ou supprimé sur un stockage réseau, pendant que la session tournait.

Avant le message du shell, l'environnement d'exécution de Claude Code peut afficher un rapport de plantage contenant `panic(main thread): Bus error at address` et `oh no: Bun has crashed. This indicates a bug in Bun, not your code.` Quand l'exécutable est devenu illisible, le plantage vient du fichier illisible, pas d'un bug de Bun. Le rapport peut aussi être absent, si l'environnement d'exécution n'a pas pu lire non plus le code qui l'affiche.

Démarrez une nouvelle session pour continuer. Si Claude Code est installé sur un stockage réseau, suivez [Install on network storage](https://code.claude.com/docs/en/setup#install-on-network-storage) pour que les mises à niveau ne suppriment pas un binaire dont des sessions en cours ont encore besoin.

### `Exec format error` sous WSL1

Si lancer `claude` dans WSL affiche `cannot execute binary file: Exec format error`, vous êtes sous WSL1 et rencontrez une régression connue du binaire natif, suivie dans l'[issue #38788](https://github.com/anthropics/claude-code/issues/38788). Les en-têtes de programme du binaire ont changé d'une manière que le chargeur de WSL1 ne sait pas gérer.

Le correctif le plus propre consiste à convertir votre distribution en WSL2, depuis PowerShell :

```powershell theme={null}
wsl --set-version <DistroName> 2
```

Si vous devez rester sous WSL1, invoquez le binaire via l'éditeur de liens dynamique. Ajoutez cette fonction à `~/.bashrc` dans WSL, en adaptant le chemin si votre répertoire personnel diffère :

```bash theme={null}
claude() {
  /lib64/ld-linux-x86-64.so.2 "$(readlink -f "$HOME/.local/bin/claude")" "$@"
}
```

Lancez ensuite `source ~/.bashrc` et réessayez `claude`.

### Erreurs d'installation npm dans WSL

Ces problèmes s'appliquent si vous avez installé Claude Code avec `npm install -g` dans WSL. Si vous avez utilisé l'[installateur natif](https://code.claude.com/docs/en/setup), sautez cette section.

**Problèmes de détection du système ou de la plateforme.** Si npm signale une incompatibilité de plateforme pendant l'installation, WSL utilise probablement le `npm` de Windows. Lancez d'abord `npm config set os linux`, puis installez avec `npm install -g @anthropic-ai/claude-code --force`. N'utilisez pas `sudo`.

**`exec: node: not found` en lançant `claude`.** Votre environnement WSL utilise probablement l'installation Windows de Node.js. Confirmez avec `which npm` et `which node` : les chemins commençant par `/mnt/c/` sont des binaires Windows, alors que les chemins Linux commencent par `/usr/`. Pour corriger, installez Node via le gestionnaire de paquets de votre distribution Linux ou via [`nvm`](https://github.com/nvm-sh/nvm).

**Conflits de version nvm.** Si nvm est installé à la fois dans WSL et dans Windows, changer de version de Node dans WSL peut échouer, car WSL importe par défaut le PATH Windows et le nvm de Windows a priorité. La cause la plus fréquente est que nvm n'est pas chargé dans votre shell. Ajoutez le chargeur nvm à `~/.bashrc` ou `~/.zshrc` :

```bash theme={null}
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

Ou chargez-le dans votre session courante :

```bash theme={null}
source ~/.nvm/nvm.sh
```

Si nvm est chargé mais que les chemins Windows ont toujours priorité, placez explicitement votre chemin Node Linux en tête :

```bash theme={null}
export PATH="$HOME/.nvm/versions/node/$(node -v)/bin:$PATH"
```

> **Avertissement** : évitez de désactiver l'import du PATH Windows via `appendWindowsPath = false`, car cela empêche d'appeler des exécutables Windows depuis WSL. De même, évitez de désinstaller Node.js de Windows si vous l'utilisez pour du développement Windows.

### Erreurs de permission pendant l'installation

Si l'installateur natif échoue avec des erreurs de permission, le répertoire cible n'est peut-être pas accessible en écriture. Voir [Vérifier les permissions des répertoires](#vérifier-les-permissions-des-répertoires).

Si vous aviez installé avec npm et rencontrez des erreurs de permission propres à npm, passez à l'installateur natif :

```bash theme={null}
curl -fsSL https://claude.ai/install.sh | bash
```

### Binaire natif introuvable après une installation npm

Le paquet npm `@anthropic-ai/claude-code` télécharge le binaire natif sous forme de dépendance optionnelle propre à chaque plateforme, comme `@anthropic-ai/claude-code-darwin-arm64`. npm lance ensuite le script postinstall du paquet, qui copie ce binaire en place comme commande `claude` ; tant qu'il n'a pas tourné, `claude` est un script provisoire (*placeholder*). Si le téléchargement ou l'étape postinstall est sautée, le placeholder reste en place, et lancer `claude` sur macOS et Linux affiche :

```text theme={null}
Error: claude native binary not installed.

Either postinstall did not run (--ignore-scripts, some pnpm configs)
or the platform-native optional dependency was not downloaded
(--omit=optional).

Run the postinstall manually (adjust path for local vs global install):
  node node_modules/@anthropic-ai/claude-code/install.cjs

Or reinstall without --ignore-scripts / --omit=optional.
```

Sous Windows, `bin/claude.exe` est ce même placeholder (un script shell) et non un véritable exécutable : PowerShell et CMD signalent donc qu'ils ne peuvent pas exécuter le fichier au lieu d'afficher ce message.

Vérifiez les causes suivantes :

* **Les dépendances optionnelles sont désactivées.** Retirez `--omit=optional` de votre commande npm install, `--no-optional` de pnpm ou `--ignore-optional` de yarn, et vérifiez que `.npmrc` ne définit pas `optional=false`. Puis réinstallez. Le binaire natif n'est livré que sous forme de dépendance optionnelle : il n'y a donc aucun repli JavaScript s'il est sauté, et relancer `install.cjs` ne peut pas mettre en place un binaire qui n'a jamais été téléchargé.
* **Les scripts d'installation sont désactivés.** `--ignore-scripts` et certaines configurations pnpm sautent l'étape postinstall tout en téléchargeant le paquet de la plateforme. Lancez `node node_modules/@anthropic-ai/claude-code/install.cjs` comme le suggère le message, ou réinstallez sans le flag. Si postinstall ne peut pas du tout s'exécuter dans votre environnement, `node node_modules/@anthropic-ai/claude-code/cli-wrapper.cjs` trouve le paquet téléchargé et le lance, au prix d'un processus Node supplémentaire à chaque démarrage. Si le wrapper affiche plutôt `Could not find native binary package`, le paquet de la plateforme n'a jamais été téléchargé : corrigez d'abord la cause « dépendances optionnelles » ci-dessus.
* **Plateforme non prise en charge.** Des binaires précompilés sont publiés pour `darwin-arm64`, `darwin-x64`, `linux-x64`, `linux-arm64`, `linux-x64-musl`, `linux-arm64-musl`, `win32-x64` et `win32-arm64`. Claude Code ne fournit pas de binaire pour d'autres plateformes ; voir la [configuration requise](https://code.claude.com/docs/en/setup#system-requirements). Sous FreeBSD, l'installateur signale la plateforme comme non prise en charge. Avant la v2.1.205, il traitait FreeBSD comme Linux et téléchargeait un binaire incapable de s'exécuter.
* **Le miroir npm de l'entreprise n'a pas les paquets de plateforme.** Assurez-vous que votre registre réplique les huit paquets de plateforme `@anthropic-ai/claude-code-*` en plus du méta-paquet.

### Erreur npm `ENOTEMPTY` pendant une mise à jour ou une réinstallation

Quand vous lancez `npm install -g @anthropic-ai/claude-code` par-dessus une installation existante, npm peut échouer en déplaçant l'ancien répertoire du paquet :

```text theme={null}
npm error code ENOTEMPTY
npm error syscall rename
npm error path /home/you/.nvm/versions/node/v22.13.1/lib/node_modules/@anthropic-ai/claude-code
npm error dest /home/you/.nvm/versions/node/v22.13.1/lib/node_modules/@anthropic-ai/.claude-code-tVWAnUUt
npm error errno -39
npm error ENOTEMPTY: directory not empty, rename '...'
```

La ligne `npm error path` nomme le répertoire que npm n'a pas pu déplacer. Supprimez ce répertoire ainsi que les éventuels répertoires `.claude-code-*` résiduels à côté, que des exécutions interrompues antérieures peuvent laisser. Les commandes ci-dessous trouvent votre répertoire global de paquets avec `npm root -g` ; si le répertoire nommé par la ligne `npm error path` n'est pas sous celui qu'affiche `npm root -g` (par exemple parce que vous avez changé de version de Node avec nvm), supprimez plutôt les répertoires nommés par l'erreur :

#### macOS/Linux

```bash theme={null}
rm -rf "$(npm root -g)/@anthropic-ai/claude-code"
```

Supprimez ensuite les éventuels répertoires temporaires résiduels. Si Zsh affiche `no matches found`, il n'y en avait aucun :

```bash theme={null}
rm -rf "$(npm root -g)/@anthropic-ai/.claude-code-"*
```

#### Windows PowerShell

```powershell theme={null}
Remove-Item -Recurse -Force "$(npm root -g)/@anthropic-ai/claude-code", "$(npm root -g)/@anthropic-ai/.claude-code-*"
```

#### Réinstaller

Puis réinstallez :

```bash theme={null}
npm install -g @anthropic-ai/claude-code
```

Confirmez avec `claude --version`, qui affiche un numéro de version tel que `2.1.211 (Claude Code)`.

## Connexion et authentification

Ces sections traitent des échecs de connexion, des erreurs OAuth et des problèmes de jetons (*tokens*).

### Réinitialiser votre connexion

Quand la connexion échoue sans cause évidente, une ré-authentification propre résout la plupart des cas :

1. Lancez `/logout` pour vous déconnecter complètement
2. Fermez Claude Code
3. Redémarrez avec `claude` et refaites tout le processus d'authentification

Si le navigateur ne s'ouvre pas automatiquement pendant la connexion, appuyez sur `c` pour copier l'URL OAuth dans votre presse-papiers, puis collez-la manuellement dans un navigateur. Cela fonctionne aussi quand l'URL est coupée sur plusieurs lignes dans un terminal étroit ou SSH et ne peut pas être cliquée directement.

### OAuth error: Invalid code

Si vous voyez `OAuth error: Invalid code. Please make sure the full code was copied`, le code de connexion a expiré ou a été tronqué lors du copier-coller.

**Solutions :**

* Appuyez sur Entrée pour réessayer, et terminez la connexion rapidement après l'ouverture du navigateur
* Tapez `c` pour copier l'URL complète si le navigateur ne s'ouvre pas automatiquement
* En session distante/SSH, le navigateur peut s'ouvrir sur la mauvaise machine. Copiez l'URL affichée dans le terminal et ouvrez-la plutôt dans votre navigateur local.

### 403 Forbidden après la connexion

Si vous voyez `API Error: 403 Request not allowed` après vous être connecté :

* **Utilisateurs Claude Pro/Max** : vérifiez que votre abonnement est actif sur [claude.ai/settings](https://claude.ai/settings)
* **Utilisateurs de l'Anthropic Console** : vérifiez que votre compte a le rôle « Claude Code » ou « Developer ». Les administrateurs l'attribuent sur la page Members de la Console, à [platform.claude.com/settings/members](https://platform.claude.com/settings/members).
* **Derrière un proxy** : les proxys d'entreprise peuvent perturber les requêtes API. Voir [network configuration](https://code.claude.com/docs/en/network-config) pour configurer le proxy.

### Claude Code access has not been granted for this account

(« L'accès à Claude Code n'a pas été accordé à ce compte ».) Si la page de connexion affiche `Authorization failed` avec le message `Claude Code access has not been granted for this account. Contact your administrator.` après que vous vous êtes connecté depuis Claude Code, votre organisation Claude Enterprise a défini votre rôle sur Custom et aucun des [rôles personnalisés](https://support.claude.com/en/articles/13930452) attribués à vos groupes n'accorde Claude Code. Avec le rôle Custom, vous n'obtenez l'accès que par ces rôles personnalisés : rien de ce que vous changez dans Claude Code ne résout cette erreur.

Pour obtenir l'accès :

1. Demandez à un Owner de votre organisation Claude d'attribuer à l'un de vos groupes un rôle personnalisé qui accorde l'accès à Claude Code, ou de faire passer votre rôle de Custom à un rôle standard comme User. Les Owners gèrent les rôles dans les [réglages des rôles](https://claude.ai/admin-settings/roles) de l'organisation.
2. Une fois la modification faite par l'Owner, lancez `claude` et reconnectez-vous.

### Cette organisation a été désactivée malgré un abonnement actif

Si vous voyez `API Error: 400 ... "This organization has been disabled"` alors que vous avez un abonnement Claude actif, une variable d'environnement `ANTHROPIC_API_KEY` prend le pas sur votre abonnement. Cela arrive souvent quand une ancienne clé API d'un employeur ou d'un projet précédent est encore définie dans votre profil de shell.

Quand `ANTHROPIC_API_KEY` est présente et que vous l'avez approuvée, Claude Code utilise cette clé au lieu des identifiants OAuth de votre abonnement. En mode non interactif avec le flag `-p`, la clé est toujours utilisée si elle est présente. Voir [authentication precedence](https://code.claude.com/docs/en/authentication#authentication-precedence) pour l'ordre de résolution complet.

Pour utiliser plutôt votre abonnement, supprimez la variable d'environnement et retirez-la de votre profil de shell :

#### macOS/Linux

```bash theme={null}
unset ANTHROPIC_API_KEY
claude
```

#### Windows PowerShell

```powershell theme={null}
Remove-Item Env:ANTHROPIC_API_KEY
claude
```

#### Rendre le changement permanent

Cherchez des lignes `export ANTHROPIC_API_KEY=...` dans `~/.zshrc`, `~/.bashrc` ou `~/.profile` et supprimez-les pour rendre le changement permanent. Sous Windows, vérifiez votre profil PowerShell (`$PROFILE`) et vos variables d'environnement utilisateur à la recherche de `ANTHROPIC_API_KEY`. Lancez `/status` dans Claude Code pour confirmer quelle méthode d'authentification est active.

### La connexion OAuth échoue dans WSL2, en SSH ou dans des conteneurs

Quand Claude Code tourne dans WSL2, sur une machine distante via SSH ou dans un conteneur, le navigateur s'ouvre généralement sur un autre hôte et sa redirection ne peut pas atteindre le serveur de rappel (*callback*) local de Claude Code. Après la connexion, le navigateur affiche un code de connexion au lieu de rediriger automatiquement. Collez ce code dans le terminal à l'invite `Paste code here if prompted` pour terminer la connexion.

Si le navigateur ne s'ouvre pas du tout depuis WSL2, définissez la variable d'environnement `BROWSER` avec le chemin de votre navigateur Windows :

```bash theme={null}
export BROWSER="/mnt/c/Program Files/Google/Chrome/Application/chrome.exe"
claude
```

Vous pouvez aussi appuyer sur `c` à l'invite de connexion interactive pour copier l'URL OAuth, ou copier l'URL qu'affiche `claude auth login`, et l'ouvrir dans un navigateur sur votre machine locale.

Si coller le code dans l'invite interactive n'a aucun effet, le raccourci de collage de votre terminal n'atteint probablement pas le champ de saisie. Essayez le raccourci de collage alternatif de votre terminal (souvent clic droit ou Maj+Inser dans Windows Terminal), ou utilisez plutôt `claude auth login`, qui lit le code collé sur l'entrée standard :

```bash theme={null}
claude auth login
```

Ce repli s'applique aussi sous Windows natif et dans tout terminal où coller dans l'invite interactive échoue.

### Non connecté ou jeton expiré

Si Claude Code vous redemande de vous connecter après une session, votre jeton OAuth a peut-être expiré.

Lancez `/login` pour vous ré-authentifier. Si cela arrive souvent, vérifiez que l'horloge de votre système est exacte, car la validation des jetons dépend d'horodatages corrects.

Les sessions parallèles sur une même machine partagent une connexion enregistrée et coordonnent son renouvellement, de sorte qu'un seul processus rafraîchit le jeton à la fois. Pour ce que font les autres sessions après que vous vous êtes reconnecté dans l'une d'elles, voir [Not logged in](https://code.claude.com/docs/en/errors#not-logged-in).

Avant la v2.1.211, réveiller la machine après une mise en veille pouvait amener deux sessions à renouveler avec le même jeton, ce qui révoquait la connexion enregistrée et faisait demander une reconnexion à toutes les sessions ouvertes en même temps.

Sur macOS, Claude Code enregistre les identifiants dans le trousseau de session (*login Keychain*). Quand le trousseau refuse l'écriture (par exemple s'il est verrouillé dans une session SSH, ou si son mot de passe est désynchronisé de celui de votre compte), Claude Code enregistre votre connexion dans le fichier en clair `~/.claude/.credentials.json`. Une connexion Console qui crée une clé API échoue tant que le trousseau n'est pas de nouveau accessible en écriture.

Pour rendre le trousseau de nouveau accessible en écriture et y replacer votre connexion chiffrée :

#### Étape 1 — Vérifier l'accès au trousseau

Lancez `claude doctor` pour vérifier l'accès au trousseau. Quand le trousseau refuse les écritures, le rapport liste un avertissement commençant par `macOS Keychain is not writable`, suivi d'un correctif suggéré. Quand le rapport ne liste aucun avertissement sur le trousseau, celui-ci est accessible en écriture et vous pouvez passer directement à la dernière étape.

#### Étape 2 — Déverrouiller le trousseau

```bash theme={null}
security unlock-keychain ~/Library/Keychains/login.keychain-db
```

Saisissez le mot de passe du trousseau quand la commande le demande, puis relancez `claude doctor`. Si le déverrouillage a fonctionné, le rapport ne liste plus l'avertissement sur le trousseau.

#### Étape 3 — Resynchroniser le mot de passe du trousseau si le déverrouillage ne suffit pas

Ouvrez Trousseau d'accès (*Keychain Access*), sélectionnez le trousseau `login`, et choisissez **Edit > Change Password for Keychain "login"** pour le resynchroniser avec le mot de passe de votre compte. Relancez ensuite `claude doctor`. Passez à l'étape suivante une fois que le rapport ne liste plus l'avertissement sur le trousseau.

#### Étape 4 — Se déconnecter puis se reconnecter

Une fois le trousseau de nouveau accessible en écriture, Claude Code y replace les identifiants la prochaine fois qu'il écrit un identifiant. Pour le forcer maintenant, lancez `/logout` puis `/login`. La déconnexion supprime tous les identifiants stockés, y compris le contenu du fichier en clair, les connexions enregistrées aux serveurs MCP et les valeurs sensibles des plugins : attendez-vous donc à devoir réautoriser les serveurs MCP et ressaisir les secrets des plugins ensuite. Se reconnecter stocke votre connexion dans le trousseau.

### Les identifiants Bedrock, Agent Platform ou Foundry ne se chargent pas

Si vous avez configuré Claude Code pour utiliser un fournisseur cloud et voyez `Could not load credentials from any providers` sur Amazon Bedrock, `Could not load the default credentials` sur Google Cloud's Agent Platform, ou `ChainedTokenCredential authentication failed` sur Microsoft Foundry, la CLI de votre fournisseur cloud n'est probablement pas authentifiée dans le shell courant.

Pour Amazon Bedrock, vérifiez que vos identifiants AWS sont valides :

```bash theme={null}
aws sts get-caller-identity
```

Pour Google Cloud's Agent Platform, vérifiez que `ANTHROPIC_VERTEX_PROJECT_ID` et `CLOUD_ML_REGION` sont définies dans votre shell, puis configurez les identifiants par défaut de l'application (*application default credentials*) :

```bash theme={null}
gcloud auth application-default login
```

Pour Microsoft Foundry, vérifiez que `ANTHROPIC_FOUNDRY_API_KEY` est définie, ou connectez-vous avec l'Azure CLI pour que la chaîne d'identifiants par défaut puisse trouver votre compte :

```bash theme={null}
az login
```

Si les identifiants fonctionnent dans votre terminal mais pas dans l'extension VS Code ou JetBrains, le processus de l'IDE n'a probablement pas hérité de l'environnement de votre shell. Définissez les variables d'environnement du fournisseur dans les réglages propres à l'IDE, ou lancez l'IDE depuis un terminal où elles sont déjà exportées.

Voir [Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock), [Google Cloud's Agent Platform](https://code.claude.com/docs/en/google-vertex-ai) ou [Microsoft Foundry](https://code.claude.com/docs/en/microsoft-foundry) pour la configuration complète du fournisseur.

## Toujours bloqué

Si rien de ce qui précède ne résout votre problème :

1. Consultez le [dépôt GitHub](https://github.com/anthropics/claude-code/issues) pour les problèmes connus, ou ouvrez une nouvelle issue avec votre système d'exploitation, la commande d'installation lancée et la sortie d'erreur complète
2. Si `claude --version` fonctionne mais que quelque chose d'autre ne va pas, lancez `claude doctor` pour obtenir un rapport de diagnostic automatisé
3. Si vous pouvez démarrer une session, utilisez `/feedback` dans Claude Code pour signaler le problème
4. Si le problème concerne votre compte plutôt que l'installation (boucle de connexion, abonnement non reconnu, organisation désactivée…), contactez le support Anthropic : connectez-vous sur [claude.ai](https://claude.ai) (utilisateurs de la Console : [platform.claude.com](https://platform.claude.com)), cliquez sur vos initiales en bas à gauche, puis sélectionnez **Get help**. Voir [How to get support](https://support.claude.com/en/articles/9015913-how-to-get-support) pour la démarche complète.
