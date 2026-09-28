# Livrer, installer et dépanner
Sources consultées le 28 septembre 2026. Recontrôle les versions locales avant d'appliquer une recette dépendante d'Epsilon ou de CEdev.


## Sommaire
- [1. Préparer la livraison](#1-préparer-la-livraison)
- [2. NumWorks](#2-numworks)
- [3. TI CE](#3-ti-ce)
- [4. Rapporter les versions](#4-rapporter-les-versions)
- [5. Dépanner par symptôme](#5-dépanner-par-symptôme)
- [6. Propriété et licences](#6-propriété-et-licences)
- [7. Sources](#7-sources)

## 1. Préparer la livraison

Construis depuis un checkout propre après avoir exécuté les tests requis.
Identifie le fichier final et sa plateforme dans le nom du paquet.
Inclue uniquement les artefacts destinés à l'utilisateur.
N'inclus pas les fichiers temporaires, clés, logs privés ou ROM.
Ajoute un bref fichier d'instructions dans le dépôt utilisateur si l'installation a des prérequis.
Ne transfère pas le guide du skill comme notice utilisateur.
Inclue le programme et ses dépendances distinctement.
Pour des AppVars, indique qu'elles sont des données séparées du programme.
Vérifie le nom, l'icône, la taille et les droits des ressources.
Garde un checksum du binaire si une reproduction exacte est utile.
Lis les règles du canal de distribution actuel.
Ne dis pas que l'app est officielle parce qu'elle passe par un uploader officiel.
Explique quel modèle ou OS a été validé.
Explique clairement si un launcher communautaire est requis.
Préviens l'utilisateur avant de remplacer des fichiers de sauvegarde ou de calculatrice.
Fournis un chemin de retour ou copie de secours si l'opération peut effacer un état.
Ne promets pas la conservation des données si l'installation ne la garantit pas.

## 2. NumWorks

NumWorks fournit un site et un flux d'installation pour apps externes ; vérifie la page active et la compatibilité navigateur/appareil.
Utilise l'artefact NWA du build courant.
Vérifie nom et icône après transfert.
Décris câble, autorisation USB et étape de confirmation si elles s'appliquent.
Vérifie si le navigateur sur ordinateur est nécessaire.
Avant d'installer, note les apps déjà présentes et sauvegardes que l'utilisateur doit conserver.
Ne dis pas que réinstaller un NWA preserve ses fichiers utilisateur sans avoir testé le même cas.
Une app externe peut s'appuyer sur la bibliothèque Nwagyu pour écrire dans le stockage partagé ; signale explicitement cette dépendance.
Indique l'espace utilisé mesuré avant le test et pas le total théorique seulement.
Teste une écriture et relecture après installation.
Confirme version Epsilon et modèle qui ont exécuté le build.
Si le navigateur ne détecte pas USB, teste un câble data et la permission du site.
Si installation échoue, compare la version de l'app et le format NWA.
Le mode simulateur et l'upload matériel sont des chemins distincts.
Ne fournis pas de lien à un binaire tiers comme s'il était le build de l'utilisateur.

## 3. TI CE

La compilation CEdev produit un programme 8XP selon le template.
Transfère le 8XP avec TI Connect CE ou le canal actuel choisi pour ce modèle.
Transfère les AppVars séparément si l'app en dépend.
Précise noms d'AppVars et rôles pour que l'utilisateur ne les supprime pas par erreur.
Vérifie le statut archive après installation et après save.
CEdev Getting Started dit que TI OS 5.5.0 et plus supprime l'exécution native directe de programmes C/ASM.
La même documentation suggère arTIfiCE comme voie communautaire pour rétablir leur exécution sur ces versions.
Vérifie si cette recommandation est toujours à jour avant de la donner.
Explique ce qu'il faut installer et comment lancer l'app sur OS visé.
Ne suppose pas qu'un transfert réussi équivaut à un lancement réussi.
N'explique pas comment downgrader l'OS sans vérifier les conséquences sur modèle, données et règles locales.
Ne propose pas un launcher communautaire comme fonctionnalité de TI.
Si l'utilisateur veut juste distribuer une app à des amis, teste le trajet complet sur un deuxième appareil.
Teste un transfert propre sans environnement de développeur.
Vérifie si les bibliothèques dynamiques requises sont déjà installées ou doivent être distribuées.
Ne repackage pas de bibliothèques CE sous une forme modifiée sans respecter leur licence.
Teste l'app avec AppVar absente et avec AppVar préexistante.

## 4. Rapporter les versions

Distingue :
- Modèle commercial.
- Version OS ou firmware.
- Variante hardware ou clavier pertinente.
- Version du compilateur.
- Version du linker et outils d'empaquetage.
- Version des bibliothèques requises.
- Révision du template.
- Format d'artefact.
- Simulateur ou calculatrice physique.
- Procédure d'installation.

CEdev v15.0 est une évolution majeure avec LLVM/Clang 19 et binutils/GAS selon sa release.
La FAQ CEdev observée peut encore annoncer LLVM/Clang 17.
Lorsque ces pages divergent, rapporte le conflit et utilise la sortie de l'outil installé.
Ne masque pas les numéros de versions derrière « récent ».
NumWorks nwlink peut être mis à jour indépendamment du template ; consigne les deux.
Ne copie pas un chiffre de mémoire depuis un ancien guide sans préciser source/version.
Ajoute une date de vérification aux recettes qui changent fréquemment.
Si la version ne peut pas être confirmée, dis-le.

## 5. Dépanner par symptôme

### App NumWorks compile mais crash au démarrage

- Vérifie la compatibilité de nwlink avec la version Epsilon.
- Compare sections, symboles et métadonnées au sample officiel.
- Lance un build minimal puis réintroduis modules un par un.
- Vérifie si l'erreur existe sur matériel ou uniquement un simulateur.
- Consulte les issues Epsilon du même modèle et firmware.
- Confirme toute API suspecte sur calculatrice réelle.

### NumWorks dit app non installable

- Vérifie que le fichier est bien NWA de l'artefact courant.
- Vérifie le format accepté par la page d'installation.
- Vérifie le modèle, firmware et version de l'uploader.
- Compare avec le NWA officiel minimal.
- Garde logs et étapes exactes sans exposer les données privées.

### Save NumWorks perdue ou vide

- Vérifie backend exact, nom et extension du fichier.
- Vérifie si le test incluait relance ou reset réellement demandé.
- Contrôle extapp_fileWrite et extapp_fileRead au niveau bytes.
- Vérifie la structure/firmware avec storage library épinglée.
- Vérifie que tu n'as pas confondu external_data et fichier mutable.
- Récupère l'autre slot valide si disponible.
- Ne réécris pas tout de suite le fichier potentiellement récupérable.

### Build TI échoue après mise à jour CEdev

- Confirme version ez80-clang et libraries.
- Lis les notes v15 pour changements de compiler et assembleur.
- Vérifie options Makefile et messages linker.
- Résous warning/error comme problème réel jusqu'à preuve contraire.
- Compare un build propre de l'exemple officiel.
- Si l'ASM est ancien, lis la doc GAS et les règles ADL.

### TI transfère mais renvoie Invalid Error

- Vérifie modèle, format du fichier et OS.
- Sur OS 5.5.0+, relis le guide CEdev relatif au lancement direct C/ASM.
- Distingue fichier incomplet d'absence de lanceur natif.
- Vérifie procédure arTIfiCE si l'utilisateur a choisi cette voie.
- Ne prétends pas qu'un changement de code de l'app corrigera une restriction OS.

### Save TI ne survit pas à un clear

- Ouvre l'AppVar sans mode destructif et vérifie son statut archive.
- Vérifie que ti_SetArchiveStatus a réussi.
- Vérifie que l'utilisateur a effacé RAM et pas archive.
- Consulte l'autre slot avant toute nouvelle écriture.
- Simule le scénario sur une calculatrice de test avec données jetables.
- Vérifie que handle était fermé et relis le fichier.

### Écran TI devient corrompu après archive

- Vérifie callbacks ti_SetGCBehavior et traitement de GraphX.
- Restaure mode LCD standard avant prompt selon docs CEdev.
- Réinitialise GraphX après retour si nécessaire.
- Teste dialogue accepté et annulé.
- Revalide sauvegarde après le cycle.

### Touches répétées ou ignorées

- Vérifie méthode keypadc ou OS choisie.
- Sépare état maintenu de fronts d'appui.
- Vérifie cadence, scan mode et gestion touche ON.
- Teste une seule touche, plusieurs touches, maintien et sortie.
- Vérifie les différences de légende TI-83 Premium CE.

## 6. Propriété et licences

Utilise les templates suivant leurs licences.
Préserve les notices des bibliothèques vendored.
Vérifie licence de sprites, sons et polices.
Ne copie pas des dumps ROM propriétaires.
Ne redistribue pas d'exécutable tiers sans autorisation.
Si le dépôt embarque NumWorks Extapp Storage, conserve sa licence et son attribution.
Si tu modifies une dépendance, identifie clairement tes changements.
Ne confonds pas open-source et absence de conditions.
Ne publie pas de secrets ou data utilisateur dans un rapport de crash.
L'utilisateur reste responsable des règles de son contexte d'installation ou examen.

## 7. Sources

- [NumWorks app installer](https://my.numworks.com/) — point d'entrée NumWorks à vérifier au moment de livrer.
- [NumWorks C sample](https://github.com/numworks/epsilon-sample-app-c) — build et sortie NWA.
- [CEdev Getting Started](https://ce-programming.github.io/toolchain/static/getting-started.html) — sortie 8XP et restrictions OS 5.5.0+.
- [CEdev releases](https://github.com/CE-Programming/toolchain/releases) — versions et toolchain.
- [CEdev fileioc](https://ce-programming.github.io/toolchain/libraries/fileioc.html) — transfert et AppVars.
- [Nwagyu storage docs](https://yaya-cout.github.io/Nwagyu/reference/apps/storage.html) — accès communautaire.
