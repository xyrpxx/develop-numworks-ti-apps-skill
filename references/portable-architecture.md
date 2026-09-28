# Architecture C portable entre NumWorks et TI CE
Sources consultées le 28 septembre 2026. Recontrôle les versions locales avant d'appliquer une recette dépendante d'Epsilon ou de CEdev.


## Sommaire
- [1. Découper le projet](#1-découper-le-projet)
- [2. Définir les frontières](#2-définir-les-frontières)
- [3. Garder un noyau testable](#3-garder-un-noyau-testable)
- [4. Maîtriser types et ABI](#4-maîtriser-types-et-abi)
- [5. Construire et intégrer](#5-construire-et-intégrer)
- [6. Exemples de portage](#6-exemples-de-portage)
- [7. Sources](#7-sources)

## 1. Découper le projet

Sépare le modèle d'application et les services matériels.
Une organisation possible, à adapter à un dépôt existant, est :
- src/core pour les règles, l'état, les transitions et le modèle sérialisé.
- src/platform/numworks pour EADK, NWA, clavier, affichage, horloge et accès storage communautaire.
- src/platform/ti_ce pour CEdev, GraphX, keypadc, clock et fileioc.
- src/app pour écrans, navigation et coordination.
- assets/source pour images et données sources.
- assets/generated pour les données produites par chaque toolchain.
- tests pour logique et codec exécutables sur hôte.
- tools pour conversion reproductible de ressources.
Ne crée pas chaque dossier si le dépôt possède déjà une organisation saine.
Ne duplique pas tout le core dans deux branches avec quelques macros différentes.
Garde les noms propres à la plateforme au bord du programme.
N'inclus pas eadk.h, graphx.h, fileioc.h ou keypadc.h dans les headers du core.
Empêche les sources métier d'appeler directement malloc spécifique, OS, USB ou timers matériels.
Garde les appels de dessin hors des fonctions de sauvegarde.
Garde l'encodage/décodage hors des appels d'API de plateforme.
N'abstrais pas des services qui ne sont employés par aucune cible ou feature.
Ne crée pas une interface commune plus large que les comportements réels des deux produits.

## 2. Définir les frontières

Le core devrait recevoir des actions logiques telles que confirmer, retour, déplacer curseur et saisir un caractère.
Le core devrait modifier un modèle d'état qui peut être testé sans écran.
Le renderer traduit les primitives logiques en EADK ou GraphX.
Le backend d'entrée convertit les keycodes physiques en actions.
Le backend d'horloge fournit un temps exprimé dans une unité déclarée.
Le backend storage fournit des octets et des erreurs, sans interpréter le modèle.
Le codec transforme l'état en octets portables et le valide au chargement.
L'interface utilisateur décide quand sauvegarder et comment présenter l'échec.
Le système de build sélectionne une seule implémentation de chaque backend.
Ne compile pas simultanément les deux fichiers d'entrée plateforme dans un même exécutable.
Ne laisse pas l'API de sauvegarde dépendre du nom d'AppVar ou de fichier NumWorks.
Définis des codes de résultat distincts pour fichier absent, données corrompues, espace insuffisant et backend indisponible.
Utilise des noms tels que storage_read_slot pour exprimer les capacités, sans promettre d'atomicité.
Évite les callbacks génériques qui cachent l'ordre des appels.
Vérifie les retours d'allocation et d'I/O à la frontière.
Établis la durée de vie des buffers retournés par un backend.
Copie vers un buffer d'app si la mutation de stockage peut invalider le pointeur.
N'utilise pas la même structure pour modèle métier, rendu écran et payload binaire.

### Contrat minimal d'interface

Décris l'interface en langage C et garde les signatures simples.
Fais passer des buffers explicites avec capacité et longueur.
Indique si une fonction accepte un buffer vide.
Indique si les données sont copiées ou si le backend garde le pointeur.
Indique si l'échec peut avoir modifié la destination.
Ne retourne pas un pointeur de stockage si l'appel suivant le rend invalide.
N'expose pas directement un handle fileioc au core.
N'expose pas des uint16 couleurs comme identifiants d'action.
Garde un adaptateur de plus haut niveau si un format est déjà présent dans l'application.
Ne crée pas une ABI stable publique si l'interface reste interne au seul projet.

Un contrat de stockage peut distinguer :
- Recherche de slot.
- Lecture avec taille connue et capacité destination.
- Écriture d'une séquence d'octets dans le slot choisi.
- Effacement explicite.
- Estimation de capacité si le backend la connaît.
- Résultat qui indique non supporté plutôt que succès simulé.
- Documentation du coût et de l'effet possible sur l'ancien fichier.
- Contrôle après écriture lorsque le backend le permet.

## 3. Garder un noyau testable

Écris des fonctions de transition qui reçoivent état, action et paramètres explicites.
Évite l'état global mutable partagé entre écrans.
Injecte la graine aléatoire dans l'état ou dans la transition qui en a besoin.
Injecte le delta temporel ou un nombre de ticks défini.
Limite le delta avant la simulation après une longue pause.
Garde les règles du jeu indépendantes de la cadence du renderer.
Garde le calcul de feuille de calcul indépendant de l'édition de cellule.
Vérifie dimensions et index avant d'accéder à un tableau.
Établis les invariants d'un état valide : bornes, références, état de partie, sélection.
Valide l'état après lecture du codec et avant de l'afficher.
N'écrase pas un état valide par une structure partiellement décodée.
Décode vers un candidat local, valide, puis remplace l'état courant.
Donne un cas de test déterministe à chaque règle importante.
Permets de comparer résultat du noyau entre compilation hôte et calculatrices.
Évite des flottants si la cohérence binaire ou la performance ne le justifie pas.
Si des calculs flottants sont requis, encode-les dans un format clairement spécifié ou en texte normalisé.
Ne sérialise pas directement l'objet mathématique natif.
Gère les valeurs NaN, infinis, zéro négatif et arrondis si float encodé.
Préfère enregistrer une valeur décimale canonique si le sens métier est décimal.
Calcule les tailles mémoire maximales à partir du nombre de cellules et de la longueur des valeurs.
Refuse une dimension trop grande avant allocation.

## 4. Maîtriser types et ABI

Les architectures NumWorks ARM et TI CE eZ80 n'ont pas la même ABI.
Sur TI CEdev, int a une largeur particulière de 24 bits selon la documentation CEdev.
Sur d'autres toolchains, int est souvent de 32 bits ; ne traite pas ce comportement comme C universel.
Utilise les types stdint.h lorsque l'intervalle doit être exact.
Choisis le type assez large pour le calcul intermédiaire, pas seulement pour la valeur finale.
Vérifie si la multiplication peut déborder avant la division ou le cast.
Convertis signed/unsigned explicitement et compare les bornes.
Ne décale pas une valeur signée négative.
Ne fais pas d'arithmétique de pointeurs à partir d'une taille non validée.
Utilise size_t pour tailles mémoire en API locale ; sérialise un entier explicitement borné.
Ne stocke pas size_t dans la save car sa largeur dépend de l'ABI.
Ne stocke pas enum sans convertir en valeur normalisée dont le domaine est contrôlé.
Ne stocke pas des pointeurs, handles, adresses d'écran, keycodes ou références à heap.
Ne suppose pas que bool, padding de struct ou alignement ont la même représentation partout.
N'utilise pas `#pragma pack` pour rendre un struct brut un format portable.
Sérialise les octets explicitement, selon le contrat de save-format-and-recovery.md.
N'utilise pas une déclaration prototype qui dépend de l'ordre de définition implicite.
Compile le core avec warnings élevés sur hôte et avec les flags recommandés de chaque toolchain.
Résous les warnings réels plutôt que de masquer les erreurs par casts.
Place attributs de section, asm, macros de target et intrinsics dans le backend.
Garde les headers partagés valides en C99 ou version plus petite réellement commune.
Ne suppose pas une libc de bureau : évite system(), fichiers stdio ou locale si non disponibles sur cible.
Vérifie les fonctions libc avec les toolchain docs.
Remplace les fonctions non disponibles par une abstraction seulement si elle est nécessaire.

## 5. Construire et intégrer

La compilation hôte sert à tester le core et ne construit pas l'app livrable.
La compilation NumWorks utilise ARM/SDK/nwlink décrits dans son template.
La compilation CE utilise CEdev et les bibliothèques correspondant à sa release.
Donne une cible distincte par environnement.
N'utilise pas une seule variable de flags qui mélange les deux toolchains.
Fixe explicitement standards C/C++ communs et évite les extensions non portables.
Sépare symboles de debug et optimisation de release si la toolchain le permet.
Vérifie que les assets générés sont à jour avant l'édition de liens.
Rends le build reproductible depuis un checkout propre.
Écris dans le README du projet les versions qui sont indispensables au build, pas une documentation redondante dans le skill.
Conserve une cible test-host sans inclure les headers de calculatrice.
Compile chaque backend même si son simulateur n'est pas configuré.
N'ajoute pas une troisième famille dans une interface pensée uniquement pour deux.
Lis scope-and-targets.md pour savoir si l'extension est dans le périmètre.
Lis testing-and-debugging.md avant de présenter le niveau de compatibilité.
Garde une sortie binaire séparée par modèle cible si la famille distingue des formats.
Ne remplace pas le fichier utilisateur final avant d'avoir terminé les tests et préservé l'artefact précédent.

### Contrôle de dépendance de plateforme

Pour chaque include du core, demande si l'autre toolchain peut le compiler.
Pour chaque appel de fonction, détermine son header, sa bibliothèque et sa garantie.
Pour chaque type public, note l'origine et la durée de vie.
Pour chaque constante, vérifie si elle est sémantique ou matérielle.
Pour chaque global, évalue taille RAM et accessibilité aux deux cibles.
Pour chaque branche conditionnelle, demande si elle correspond à une capacité réellement différente.
Pour chaque fonction backend, décide si elle doit être testée sur hôte avec un faux backend.
Pour chaque data path, confirme la taille avant allocation et avant copie.
Pour chaque macro de build, cherche tous les endroits où elle modifie l'ABI ou l'API.
Supprime les #ifdef de cible lorsqu'ils ne font que contourner une architecture mal séparée.

## 6. Exemples de portage

### Porter un jeu C de bureau

- Garde le modèle de jeu et la logique de collision en C commun.
- Isole SDL, raylib, stdio, threads et fonctions système derrière les adapters.
- Convertis événements souris/manette en actions clavier logiques.
- Remplace audio par une capacité optionnelle si la cible ne le fournit pas.
- Adapte le renderer avant de compresser les assets.
- Retire les allocations qui excèdent le budget de calculatrice.
- Remplace les fichiers de bureau par un backend persistence adapté.
- Fixe les valeurs de graine et du temps dans les tests de logique.
- Vérifie le comportement sans les fonctionnalités optionnelles.

### Porter une feuille de calcul comme NumSheet

- Définis cellule, dimensions, préférences et vue indépendamment des pixels.
- Stocke les valeurs selon une représentation portable et bornée.
- Distingue donnée persistante de curseur et position visuelle.
- Recalcule l'affichage à partir du modèle.
- Limite les dimensions maximales par calcul de mémoire.
- Établis une politique claire pour valeurs vides, invalides ou trop longues.
- Sauvegarde les données métier et préférences utiles, pas les buffers de dessin.
- Teste une cellule négative, une limite de ligne, une table vide et la migration.
- Garde les raccourcis clavier propres à chaque modèle.

### Porter un format binaire existant

- Identifie l'endianess et largeur historique par version.
- Cherche si le format a enregistré des structs bruts.
- Garde un décodeur historique en lecture seule si des fichiers utilisateurs existent.
- Convertis vers un modèle interne validé.
- Écris la nouvelle version dans l'autre slot.
- Vérifie que l'ancienne sauvegarde reste disponible si migration échoue.
- Ne modifie pas une save utilisateur à des fins de test.
- Produis un fichier golden de chaque version prise en charge.

## 7. Sources

- [CE C/C++ Toolchain hardware overview](https://ce-programming.github.io/toolchain/static/hardware.html) — détails eZ80, int 24-bit, short/long, pointeurs et clavier.
- [CEdev FAQ](https://ce-programming.github.io/toolchain/static/faq.html) — piles, sections programme, BSS et heap ; la version du compilateur peut être en retard sur les notes de release.
- [CEdev General Coding Guidelines](https://ce-programming.github.io/toolchain/static/coding-guidelines.html) — pratiques de mémoire et d'analyse.
- [NumWorks EADK header](https://github.com/numworks/epsilon/blob/master/epsilon/eadk/include/eadk/eadk.h) — types API de l'autre backend.
- [CEdev GraphX](https://ce-programming.github.io/toolchain/libraries/graphx.html) — API de rendu TI.
- [CEdev fileioc](https://ce-programming.github.io/toolchain/libraries/fileioc.html) — stockage isolé du core.
