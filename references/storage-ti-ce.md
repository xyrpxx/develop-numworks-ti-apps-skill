# Sauvegardes TI CE avec AppVars et fileioc
Sources consultées le 28 septembre 2026. Recontrôle les versions locales avant d'appliquer une recette dépendante d'Epsilon ou de CEdev.


## Sommaire
- [1. Stockage TI](#1-stockage-ti)
- [2. API fileioc](#2-api-fileioc)
- [3. Écrire sans perdre la sauvegarde active](#3-écrire-sans-perdre-la-sauvegarde-active)
- [4. Archiver](#4-archiver)
- [5. Charger et valider](#5-charger-et-valider)
- [6. Erreurs et pointeurs](#6-erreurs-et-pointeurs)
- [7. Tests de persistance](#7-tests-de-persistance)
- [8. Sources](#8-sources)

## 1. Stockage TI

Un programme CEdev transféré comme 8XP est distinct de ses AppVars.
fileioc permet de stocker les bytes d'une sauvegarde dans une AppVar séparée.
L'OS place ses variables en RAM ou dans l'archive.
La documentation CEdev décrit RAM comme volatile et effacée au reset.
La zone archive est plus durable pour les variables, mais elle reste modifiable et effaçable par l'utilisateur.
Les variables archivées occupent un espace commun avec les autres données et programmes.
Ne dis pas qu'une AppVar archive résiste à tout reset, tout clear ou toute suppression.
Définis l'action précise que l'utilisateur veut survivre.
Teste effacement RAM séparément de l'effacement archive et réinitialisation.
Ne confonds pas import du 8XP avec restauration de son AppVar.
Le transfert du programme ne garantit pas qu'une variable auxiliaire est copiée.
La variable de sauvegarde doit avoir son propre nom et sa procédure de migration.

## 2. API fileioc

Lis la [documentation fileioc versionnée](https://ce-programming.github.io/toolchain/libraries/fileioc.html) correspondant au CEdev installé.
Les signatures exactes peuvent changer ; vérifie aussi le header fileioc.h local.
ti_Open(name, mode) retourne un handle opaque ou zéro à l'échec.
ti_Open en mode r ouvre une AppVar en lecture et ne la déplace pas de son emplacement.
ti_Open en mode w supprime l'AppVar existante portant ce nom et crée une nouvelle variable en RAM.
ti_Open en mode a crée si nécessaire et peut déplacer une AppVar archivée en RAM.
ti_Open en mode r+ lit et écrit, et peut déplacer l'AppVar archivée en RAM.
ti_Open en mode w+ détruit l'ancienne AppVar et en crée une nouvelle en RAM.
Traite w et w+ comme opérations destructives sur le nom passé.
Ne mets jamais le nom de la seule sauvegarde active dans w.
ti_Close doit être appelé pour chaque ti_Open et ti_OpenVar.
ti_GetSize retourne la taille de l'AppVar dans les limites du type de la version installée.
ti_Read reçoit buffer, taille d'élément, nombre d'éléments et handle.
ti_Write reçoit bytes, taille de chaque élément, nombre d'éléments et handle.
ti_Read et ti_Write retournent le nombre d'éléments effectivement traités.
Pour traiter un buffer d'octets, l'appel size=1,count=length permet de comparer le résultat au nombre d'octets, après validation de la signature utilisée.
ti_IsArchived vérifie si l'AppVar est dans l'archive selon la convention documentée.
ti_SetArchiveStatus archive ou désarchive l'AppVar liée au handle.
La documentation indique que ti_SetArchiveStatus retourne zéro en cas d'échec.
L'archivage peut provoquer un garbage collect et un dialogue OS.
ti_SetGCBehavior configure les fonctions avant/après ce dialogue ou sa collecte.
Si une fonction retourne un handle, taille ou nombre d'éléments, contrôle la valeur avant l'usage suivant.
Ne suppose pas qu'un appel réussi signifie que le nombre demandé d'octets est disponible.
Ne ferme pas le même handle deux fois.
Ne laisse pas de branche erreur sans cleanup.

### Appels à éviter pour une sauvegarde ordinaire

ti_GetDataPtr expose un accès direct et la doc le présente comme risqué.
Évite-le pour le format de sauvegarde courant ; fileioc read/write a des retours mesurables.
La documentation avertit qu'écrire par pointeur direct dans une variable archivée peut provoquer un reset système.
Elle avertit aussi qu'un pointeur peut devenir invalide si une variable est créée, supprimée, redimensionnée ou déplacée.
Ne conserve pas un pointeur interne après modification du stockage.
ti_Resize ne conserve pas les bytes lorsque l'AppVar est étendue ou réduite selon la doc.
Ne l'utilise pas comme une fonction realloc préservant les données.
Si un code ancien s'en sert, vérifie précisément le comportement et crée d'abord un slot séparé.
Pour tout AppVar existant, relis-la avant une opération susceptible de la déplacer.

## 3. Écrire sans perdre la sauvegarde active

Utilise deux slots aux noms conformes aux limites TI de la version visée.
Réserve un préfixe propre au projet afin de limiter les collisions.
Exemple conceptuel : APPDATAA et APPDATAB ; vérifie la longueur autorisée par le système.
Ne reprends pas ces noms si une autre app les utilise déjà.
Lis les deux AppVars en mode non destructif.
Vérifie magic, version, longueur, CRC et invariants métier.
Sélectionne le slot valide le plus récent selon une règle de génération documentée.
Si un seul slot est valide, garde-le comme source active.
Si aucun slot n'est valide, traite état neuf, corruption et version inconnue séparément.
Choisis l'autre slot pour le prochain commit.
Ouvre uniquement cette destination en mode w.
Contrôle le handle avant tout appel qui le consomme.
Écris le header et payload sérialisés avec un appel borné.
Vérifie que ti_Write a écrit le nombre exact d'éléments.
Ferme le handle sur succès comme sur chaque échec.
Si écriture partielle, ne lis pas ce slot comme nouveau commit.
L'ancien slot actif doit rester inchangé dans le processus.
Ne supprime pas l'ancien slot après succès, il sert de secours lors du commit suivant.
Ne considère pas le double slot comme transaction native de TI OS ; c'est un protocole applicatif à tester.
Si l'ouverture de destination échoue, retourne un code d'espace/IO au core.
Ne lance pas un autre mode d'écriture en espérant qu'il crée plus d'espace.

## 4. Archiver

Une AppVar en RAM peut survivre au simple retour OS selon le flux, mais elle ne résiste pas à l'effacement RAM.
Une AppVar archivée est le choix habituel pour conserver des saves à travers un clear de RAM.
Vérifie les retours d'archivage ; n'annonce pas durable si l'AppVar reste en RAM.
Utilise le handle valide selon la séquence que les docs CEdev de la version installée décrivent.
L'archivage peut lancer un garbage collection.
Le système peut demander confirmation à l'utilisateur ; annulation et manque de place sont des résultats normaux à gérer.
Si GraphX ou un mode écran personnalisé est actif, la documentation CEdev indique de restaurer le mode LCD standard avant le dialogue GC.
Configure ti_SetGCBehavior avec callback before qui ferme proprement GraphX si nécessaire.
Configure callback after pour restaurer l'affichage, buffers et pointeurs qui doivent être relus.
Les callbacks de GC ne doivent pas supposer que la collection a été acceptée.
Teste la branche d'annulation du dialogue.
Contrôle le résultat de ti_SetArchiveStatus.
Ferme ensuite le handle dans le chemin de cleanup.
Rouvre en mode r et confirme taille, status archive si l'API l'expose, bytes, CRC et version.
Ne sauvegarde pas l'écran GraphX depuis callback si le callback est appelé en contexte non sûr ; vérifie le contrat de l'API.
Ne relance pas l'archivage si le premier appel a renvoyé une erreur sans avoir relu l'état de l'AppVar.
Ne laisse pas le callback changer les données métier ou générer un second write.

## 5. Charger et valider

Ouvre les AppVars candidates en lecture seule.
Vérifie absence sans créer une AppVar vide.
Lis la taille avant de copier.
Rejette une AppVar plus courte que l'en-tête.
Rejette une AppVar plus grande que la limite du modèle.
Copie uniquement après avoir borné la taille contre le tampon de destination.
Lis la quantité exacte de bytes attendue.
Vérifie le retour de ti_Read avant de décoder.
Ferme toujours l'AppVar.
Vérifie CRC avant de parser des champs métier.
Vérifie ensuite dimensions, index, score, cellules et enum métier.
Décode dans une structure candidate distincte de l'état courant.
Applique l'état uniquement après succès de toutes validations.
Si la première génération échoue, essaie le second slot valide.
Rends à l'interface le slot choisi et la raison de récupération.
Ne charge pas une save de version future comme si c'était l'ancienne.
Garde l'ancien slot intact si le code propose une migration.
Charge un fichier 8XP ne doit pas déclencher automatiquement une écriture d'AppVar.
Ne crée pas de save par défaut à chaque lancement si aucune modification n'a eu lieu.

## 6. Erreurs et pointeurs

Traite handle 0 comme une ouverture échouée conformément à fileioc.
Traite retour de ti_Read plus petit que count comme lecture incomplète.
Traite retour de ti_Write plus petit que count comme écriture incomplète.
Traite ti_SetArchiveStatus égal à zéro comme un échec documenté.
Fais remonter une erreur d'archive jusqu'à l'UI.
Signale si les données existent mais sont uniquement en RAM.
N'associe pas erreur d'ouverture et AppVar absente si l'API ne les différencie pas.
Utilise une erreur générale de backend si la cause ne peut pas être diagnostiquée.
Ferme tout handle avant d'ouvrir un autre nom si la ressource ou la bibliothèque le requiert.
N'utilise pas un pointeur ti_GetDataPtr après write, resize, archive ou création d'un autre variable.
Ne fais pas un memcpy depuis une AppVar vers le modèle sans contrôler longueur.
Ne conserve pas le handle dans le fichier de sauvegarde.
Ne transforme pas un échec de fermeture en succès silencieux.
Teste le cleanup par injection d'échec à chaque étape.
Garde le slot actif et l'état en RAM après une écriture manquée.
Montre « sauvegarde non confirmée » tant que read-back et archive attendus ne sont pas validés.

## 7. Tests de persistance

Sur une machine de test, crée des slots avec contenus reconnaissables et différents.
Sauvegarde, quitte au menu TI puis relance le programme.
Relis les données et compare les valeurs métier.
Vérifie séparément que le slot est archivé dans l'écran de gestion mémoire.
Efface la RAM avec une procédure connue, puis relance et vérifie l'AppVar.
Ne confonds pas effacer RAM et effacer archive.
Teste redémarrage normal séparément de RAM clear.
Ne teste pas archive clear sur une calculatrice personnelle sans sauvegarde des données utilisateur.
Teste mode w avec le nom inactif pour confirmer l'effet destructif attendu.
Teste une écriture tronquée simulée ou un backend fautif.
Teste archive pleine avec annulation du dialogue.
Teste que les callbacks GraphX restaurent correctement l'écran.
Teste programme mis à jour tout en gardant les AppVars.
Teste une nouvelle version du schéma sans supprimer la save précédente.
Enregistre TI model, OS, CEdev, libraries, arTIfiCE si nécessaire, handle state et steps.
Ne prétends pas que l'archive résiste à un effacement explicite des variables.
Ne prétends pas qu'un test CEmu prouve la même réaction au GC physique.
Utilise des AppVars de test, pas celles d'une autre app.
Restaure le banc de test après les scénarios destructifs.

## 8. Sources

- [CEdev fileioc v15 docs](https://ce-programming.github.io/toolchain/libraries/fileioc.html) — modes d'ouverture, read/write, handles, tailles, archive, GC et pointeurs directs.
- [CEdev GraphX](https://ce-programming.github.io/toolchain/libraries/graphx.html) — modes d'affichage.
- [CEdev keypadc](https://ce-programming.github.io/toolchain/libraries/keypadc.html) — entrée directe.
- [CEdev Getting Started](https://ce-programming.github.io/toolchain/static/getting-started.html) — build et OS 5.5+.
- [CEdev FAQ](https://ce-programming.github.io/toolchain/static/faq.html) — contexte mémoire.
- [Toolchain changelog](https://github.com/CE-Programming/toolchain/blob/master/changelog.md) — versions CEdev.
