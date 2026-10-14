# 29 — Bibliothèque de prompts (prêts à copier)

[← Sommaire](README.md)

Traduction de la bibliothèque officielle de prompts, classée par étape du cycle de développement. Remplacez les éléments `{entre accolades}`. Claude Code comprend aussi bien le français que l'anglais.

## Découvrir un code

| Objectif | Prompt |
|----------|--------|
| S'orienter dans un nouveau dépôt | « Donne-moi une vue d'ensemble de ce code : architecture, dossiers clés et comment les parties s'articulent » |
| Expliquer du code inconnu | « Explique ce que fait `{chemin}` et comment les données y circulent. Rédige-le sous forme de {page HTML avec un diagramme, puis ouvre-la dans mon navigateur} » |
| Trouver où se passe quelque chose | « Où est-ce qu'on {valide les types de fichiers téléversés} ? » |
| Voir ce qui dépend d'un élément | « Qu'est-ce qui casserait si je supprimais {le helper retryWithBackoff} ? » |
| Retracer l'évolution du code | « Parcours l'historique des commits de `{chemin}` et résume comment il a évolué et pourquoi » |
| Délimiter un changement | « Quels fichiers devrais-je modifier pour {ajouter un mode sombre aux réglages} ? » |
| Question produit au code | « Je suis {chef de produit}. Explique-moi ce qui se passe quand un utilisateur {clique sur Exporter en PDF}, de l'interface jusqu'au résultat » |

## Concevoir

| Objectif | Prompt |
|----------|--------|
| Planifier un changement sur plusieurs fichiers | « Planifie comment refactoriser {la cible} pour {objectif}. Liste les fichiers que tu modifierais, mais ne modifie rien pour l'instant » |
| Rédiger une spec par interview | « Je veux construire {fonctionnalité}. Interroge-moi sur l'implémentation, l'UX, les cas limites et les compromis jusqu'à avoir tout couvert, puis écris la spec dans SPEC.md » |
| Transformer une réunion en tickets | « Lis {notes de réunion} et rédige les actions à mener, puis crée un ticket dans {Linear/Jira} pour chacune, avec des critères d'acceptation » |
| Cartographier les cas limites | « Liste les états d'erreur, les états vides et les cas limites de {fonctionnalité} que le design doit couvrir » |
| Maquette → prototype | « Voici une maquette. Construis un prototype cliquable qui respecte la mise en page et les états montrés » |
| Implémenter depuis une capture et s'auto-vérifier | « Implémente ce design, puis fais une capture d'écran du résultat, compare-la à l'original et corrige les différences » |

## Construire

| Objectif | Prompt |
|----------|--------|
| Suivre un modèle existant | « Regarde comment {exemple} est implémenté pour comprendre le schéma, puis construis {nouveau} de la même façon » |
| Documenter du code | « Trouve {les fonctions exportées} sans commentaires {JSDoc} et ajoute-les, dans le style déjà utilisé dans le fichier » |
| Ajouter un petit endpoint | « Ajoute un endpoint {GET /health} qui renvoie {statut et version} » |
| Petit outil interne | « Crée un {convertisseur de fuseaux horaires} en HTML, CSS et JavaScript, puis ouvre-le dans mon navigateur » |
| Traiter une issue de bout en bout | « Lis l'issue #{123}, implémente la correction et lance les tests » |
| Trouver et mettre à jour un texte | « Trouve chaque endroit où l'on écrit « {ancien texte} » ou une variante, montre-moi chaque occurrence en contexte, puis remplace-les toutes par « {nouveau} ». Ne touche pas aux tests ni au changelog » |
| Rédiger d'après des exemples | « Lis les {articles} de {dossier} pour apprendre leur structure et leur ton, puis rédiges-en un nouveau sur {sujet} » |

### Tester

| Objectif | Prompt |
|----------|--------|
| Écrire et lancer des tests | « Écris des tests pour `{chemin}`, lance-les et corrige les échecs » |
| Développement piloté par les tests | « Écris d'abord les tests de {fonctionnalité}, puis implémente-la jusqu'à ce qu'ils passent » |
| Combler la couverture | « Lis {rapport de couverture} et ajoute des tests aux fichiers les moins couverts jusqu'à dépasser {80} % » |

### Refactoriser

| Objectif | Prompt |
|----------|--------|
| Migrer un schéma partout | « Migre tout de {ancienne API} vers {nouvelle API} : identifie chaque endroit à changer, puis fais les modifications » (ou `/batch …`) |
| Porter du code dans un autre langage | « Porte {source} en {langage cible} en gardant {la même API publique} » |
| Optimiser selon une mesure | « Optimise {la requête} pour faire passer {le temps de réponse} de {800 ms} à moins de {200 ms} » |
| Corriger un défaut visuel précis | « Le {bouton} dépasse de {8 px} du {conteneur} sur {mobile}. Corrige-le. » |

### Relire

| Objectif | Prompt |
|----------|--------|
| Relire avant de committer | « Relis mes changements non commités et signale ce qui semble risqué avant que je committe » (ou `/code-review`) |
| Relire une PR | « Relis la PR #{42}, résume ce qui a changé, puis liste tes inquiétudes » |
| Relire de l'infrastructure | « Voici la sortie de mon plan Terraform. Qu'est-ce que ça va faire, et est-ce que quelque chose va poser problème ? » |
| Revue de sécurité | « Utilise un sous-agent pour vérifier la sécurité de `{chemin}` et rapporte ce qu'il trouve » (ou `/security-review`) |
| Relire un contenu | « Relis {fichier} pour {le ton et l'exactitude} et liste ce que je dois corriger avant de l'envoyer à {mon responsable} » |

### Réorienter Claude

| Objectif | Prompt |
|----------|--------|
| Corriger une mauvaise direction | « Ce n'est pas ça : {retour}. Essaie une autre approche » |
| Réduire le périmètre | « C'est trop. Garde seulement les changements dans {src/api} et annule tes autres modifications » |
| Transformer une correction en règle | « Tu continues à {utiliser npm au lieu de pnpm}. Ajoute une règle dans CLAUDE.md pour que ça s'arrête » |

## Livrer

| Objectif | Prompt |
|----------|--------|
| Résoudre des conflits | « Résous les conflits de fusion de cette branche et explique ce que tu as gardé de chaque côté » |
| Committer | « Committe ces changements avec un message qui résume ce que j'ai fait » |
| Ouvrir une PR depuis un ticket | « Trouve le ticket sur {sujet} dans {outil} et ouvre une PR qui l'implémente » |
| Notes de version | « Compare {v1.2.0} à {v1.3.0} et rédige les notes de version groupées par fonctionnalité, correctif et changement cassant » |
| Workflow de CI | « Écris un workflow GitHub Actions qui {lance le lint et les tests} à chaque push sur {main} » |

## Exploiter

### Déboguer

| Objectif | Prompt |
|----------|--------|
| Test qui échoue | « Le test {nom} échoue, trouve pourquoi et corrige-le » |
| Erreur signalée | « Les utilisateurs voient {une page blanche} sur {la page de paiement}. Enquête et dis-moi ce qui se passe » |
| Erreur de build | « Voici une erreur de build. Corrige la cause racine et vérifie que le build réussit » |

### Incidents

| Objectif | Prompt |
|----------|--------|
| Incident en production | « {Les temps de réponse ont triplé depuis 14 h}. Vérifie les logs, les déploiements récents et les changements de config, puis dis-moi la cause la plus probable » |
| Diagnostic depuis une console | « Voici une capture de {la console AWS}. Explique-moi pourquoi {la fonction Lambda} échoue et donne-moi les commandes exactes pour corriger » |
| Requête de logs en langage naturel | « Montre-moi tous les {erreurs 500} pour {le service API} sur {les dernières 24 h}. Écris la requête, lance-la et dis-moi ce qui ressort » |

### Données

| Objectif | Prompt |
|----------|--------|
| Analyser un fichier | « Lis {ventes.csv}, résume les tendances clés et écris les résultats dans {rapport.md} » |
| Générer des variantes | « Lis {campagnes.csv}, trouve les {annonces} les moins performantes et génère {5} nouvelles variantes de moins de {90} caractères » |

### Automatiser

| Objectif | Prompt |
|----------|--------|
| Tâche récurrente → skill | « Crée un skill /{release} pour ce projet qui {met à jour la version, le changelog et crée le tag} » |
| Ajouter un hook | « Écris un hook qui {lance prettier} après chaque {modification de fichier} » |
| Connecter un outil | « Connecte {Sentry} via MCP pour que tu puisses lire ses {erreurs} directement » |
| Capitaliser | « Résume ce qu'on a fait dans cette session et propose ce qu'il faudrait ajouter au CLAUDE.md » |

## Ce qui rend ces prompts efficaces

- **Un résultat vérifiable** : « lance les tests », « compare la capture », « vérifie que le build passe ».
- **Un périmètre clair** : fichiers, dossiers, ce qu'il ne faut pas toucher.
- **Du contexte concret** : chemins, numéros d'issue, captures, logs, exemples à imiter.
- **Planifier avant les gros changements** : « ne modifie rien pour l'instant », mode plan.
- **Corriger tôt** et transformer les corrections répétées en règles (`CLAUDE.md`) ou en hooks.

## Demander à Claude ce qu'il sait faire

« Peux-tu créer des pull requests ? », « Comment configurer un hook ? », « Quelles sont les limites des sous-agents ? », « Comment utiliser Claude Code avec Bedrock ? » — l'agent intégré **claude-code-guide** répond à partir de la documentation officielle.
