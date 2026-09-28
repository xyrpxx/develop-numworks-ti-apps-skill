# Développer des programmes TI CE en C/C++ avec CEdev
Sources consultées le 28 septembre 2026. Recontrôle les versions locales avant d'appliquer une recette dépendante d'Epsilon ou de CEdev.


## Sommaire
- [1. Cibles et chaîne](#1-cibles-et-chaîne)
- [2. Installer et compiler](#2-installer-et-compiler)
- [3. ABI eZ80 et mémoire](#3-abi-ez80-et-mémoire)
- [4. Bibliothèques CE](#4-bibliothèques-ce)
- [5. Lancer le programme](#5-lancer-le-programme)
- [6. Transférer et déboguer](#6-transférer-et-déboguer)
- [7. Sources](#7-sources)

## 1. Cibles et chaîne

Cette référence vise la famille CE prise en charge par le CE C/C++ Toolchain, surtout TI-83 Premium CE et TI-84 Plus CE.
Vérifie le tableau de modèles présent dans la documentation de la version installée.
Ne déduis pas qu'une TI-83 Plus ancienne accepte un exécutable CE.
Ne déduis pas qu'un programme 8XP est compatible avec chaque OS ou chaque famille.
Le fichier 8XP contient un programme TI transférable, pas une application installable comme un NWA.
Les données modifiables peuvent être stockées dans des AppVars séparées du programme.
Le modèle TI-83 Premium CE partage des bases matérielles avec les variantes de TI-84 Plus CE, mais certaines touches et présentations d'OS diffèrent.
Traite la TI-84 Plus CE-T Python comme un cas à confirmer dans la matrice CEdev.
Relève modèle, révision matérielle si accessible, OS, édition et environnement de transfert.
Les noms CEdev et CE C/C++ Toolchain désignent ici le même écosystème de développement.
Utilise les docs correspondant à la version réelle ; les pages latest évoluent.
Les sorties et l'ABI ont changé entre versions anciennes de la toolchain.
Ne copie pas les exemples ZDS ou anciens CEdev sans vérifier leur date et leur compatibilité v15.

## 2. Installer et compiler

Commence par la page officielle [Getting Started](https://ce-programming.github.io/toolchain/static/getting-started.html).
Télécharge une release stable correspondant à Windows, macOS ou Linux.
Lis les notes de release et pas seulement la page d'installation.
La v15.0 est une release majeure, avec changement vers Clang v19 et binutils/GAS selon ses notes et changelog.
La FAQ actuelle peut encore afficher Clang v17 : ne répète pas cette valeur sans vérifier l'exécutable installé.
Exécute ez80-clang --version ou la commande équivalente fournie par ton installation.
Enregistre la release CEdev, celle de CE Libraries si séparée, et la version de make.
Si l'environnement Windows exige cedev.bat, lance-le ou configure PATH comme documenté.
Sous macOS/Linux, utilise le dossier CEdev correspondant à l'architecture et au paquet réellement téléchargé.
Évite un chemin d'installation qui contredit les exigences actuelles du paquet.
Commence par l'exemple Hello World officiel.
La commande make produit un fichier dans le répertoire cible du projet ; le guide de démarrage utilise bin/DEMO.8xp dans son exemple.
Ne suppose pas que tous les projets produisent le même chemin, nom ou type de programme.
Lance make clean seulement si tu sais quels artefacts il supprime.
Vérifie le code de sortie de make.
Garde le log complet, y compris la commande du compilateur et du linker en cas d'échec.
Utilise l'option verbeuse si le build masque les commandes nécessaires au diagnostic.
Vérifie que les fichiers C/C++ sont inclus dans la cible et que la bonne variante de Makefile est sélectionnée.
N'ajoute une bibliothèque au lien qu'après avoir confirmé son nom, sa disponibilité et ses dépendances.
Vérifie la présence d'un module en ligne de commande avant de rédiger des instructions utilisateur.
Ne présume pas que make télécharge des outils par lui-même.
Épingle une release pour reproduire un build ; passe à une nouvelle version dans un changement identifiable.

### Évolution des toolchains

CEdev v15 :
- A actualisé le compilateur vers LLVM/Clang 19 d'après la release v15.0 et son changelog.
- A remplacé l'assemblage/linking historique par binutils/GAS pour le flux de programme.
- Peut demander de porter les sources assembleur de fasmg vers GAS.
- A modifié et étendu runtime, bibliothèque standard, outils et tests.

Les pages web peuvent présenter une incohérence de version entre FAQ et notes de release.
En cas d'incohérence, rapporte les deux références et donne le résultat de ez80-clang --version.
N'étiquette pas un warning comme nouveau bug avant d'avoir comparé le code et la toolchain.
Si le projet contient de l'assembleur, lis la section assembleur de la version cible.
N'utilise pas des options linker issues d'un projet v14 dans un Makefile v15 sans confirmation.
Lis le changelog au moment de porter un ancien dépôt.

## 3. ABI eZ80 et mémoire

La cible TI CE utilise un CPU eZ80 avec un espace d'adressage de 24 bits.
Le CEdev courant documente int sur 24 bits, ce qui est inhabituel sur les toolchains de bureau.
La documentation du matériel donne des tailles précises pour char, short, int, long et pointeurs ; revois-les dans la version consultée.
Le header stdint.h expose des types à largeur explicite, dont int24_t/uint24_t pour les cibles CE.
Utilise uint8_t, uint16_t, uint32_t ou int32_t quand la largeur fait partie du format ou protocole.
Ne sérialise pas un int, enum, pointeur ou struct brut.
N'emploie pas sizeof(void *) comme une constante portable.
Vérifie les conversions signées et les décalages.
Le 24-bit natif de l'eZ80 n'implique pas que toutes opérations C soient automatiquement plus rapides avec int.
Mesure les opérations arithmétiques importantes au lieu de supposer le type le plus petit le plus rapide.
Les flottants sont gérés logiciellement dans la chaîne CE ; la FAQ ou la page hardware de la version indique les détails.
N'utilise pas une division flottante dans une boucle par frame sans profil.
Considère fixed-point ou entiers si les tests montrent que les maths dominent.
Ne traite pas les tailles de type du compilateur hôte comme celles de CEdev.

### Mémoire d'exécution

La FAQ CEdev consultée documente un ordre de grandeur d'environ 4 Kio de pile.
Elle indique une limite pouvant atteindre 64 Kio pour code/data/rodata.
Elle décrit un espace partagé d'environ 60 Kio pour BSS et heap, qui peuvent croître l'un vers l'autre.
Ces valeurs caractérisent le modèle runtime documenté et ne donnent pas le nombre exact de bytes libres dans chaque programme.
Les bibliothèques liées, les globales et le runtime consomment une partie du budget.
Vérifie la map file et les messages du linker pour le programme réel.
Évite les grands tableaux automatiques, la récursion profonde et les temporaires cachés.
Évite d'allouer un framebuffer 320×240 si un buffer de tuiles suffit.
Teste les allocations au pire cas de progression et avec sauvegarde chargée.
Réduis les assets ou compresse-les si le coût mémoire est prouvé.
Ne remplace pas aveuglément malloc par un buffer statique : la taille maximale doit être connue et tenue.
Lis la page CEdev General Coding Guidelines pour les règles de pile, heap et analyse statique.
Ne confonds pas la RAM d'exécution avec RAM de stockage des AppVars.
Ne confonds pas l'archive persistante avec la RAM réservée au programme.

## 4. Bibliothèques CE

Les docs CEdev répertorient les bibliothèques et headers disponibles.
Utilise GraphX pour les surfaces, sprites, palettes et buffers graphiques.
Utilise keypadc pour scanner directement le clavier, détecter les touches multiples et gérer un jeu réactif.
Utilise les appels OS comme os_GetCSC quand une simple touche séquentielle est adaptée au menu ou formulaire.
Utilise fileioc pour lire et écrire des AppVars et autres variables OS.
Utilise fontlibc seulement si la police intégrée ne suffit pas et que le budget le permet.
Utilise libload seulement quand le projet dépend d'une bibliothèque CE externe qui le requiert.
Vérifie comment chaque bibliothèque est distribuée et si l'utilisateur doit la transférer.
Ne suppose pas que linker une bibliothèque statique et installer une bibliothèque dynamique sont la même chose.
CEdev libraries peut distribuer des modules en AppVar ; vérifie le packaging de la release correspondant au Makefile.
Conserve les bibliothèques propres à CE dans le backend ou la couche de rendu.
Évite d'importer leurs types dans le noyau commun.
Vérifie les symbols d'une bibliothèque dans les docs et avec le linker.
Ne déduis pas un support complet d'un #include réussi.
Lis les notes de version si une bibliothèque change de signature.

## 5. Lancer le programme

Le lancement doit être analysé séparément de la compilation.
CEdev Getting Started avertit que les versions TI OS 5.5.0 et supérieures retirent l'exécution native directe des programmes C et assembleur et montrent Invalid Error.
La page actuelle recommande arTIfiCE comme moyen communautaire de restaurer le lancement natif sur les OS concernés.
arTIfiCE n'est pas une fonction officielle du SDK TI ; traite-le comme une dépendance distincte.
Avant de recommander un launcher, vérifie les versions OS visées, les consignes actuelles, la provenance et les risques.
N'invite pas à downgrader l'OS sans expliquer les conséquences et vérifier le contexte.
Ne dis pas « le programme marche » sur la seule base du build si l'OS ne le lance pas.
Précise si les utilisateurs devront lancer l'app par arTIfiCE, un menu ou un programme déclencheur.
Si l'app est destinée à l'école, vérifie les règles locales concernant les lanceurs et programmes.
Fournis l'artefact et la méthode d'exécution attendue.
Teste le lancement sur la version d'OS exacte.
En cas d'Invalid Error, distingue lancement natif désactivé, programme invalide et transfert incomplet.
Confirme la cible du fichier 8XP avant d'accuser l'OS.
La compatibilité d'un programme par modèle et OS est une ligne distincte dans la matrice release.

## 6. Transférer et déboguer

Utilise la version actuelle de TI Connect CE ou l'outil TI-compatible retenu par le projet.
Sépare transfert du programme 8XP et transfert de l'AppVar.
Vérifie que les noms de variables respectent les règles TI.
Vérifie sur la calculatrice que programme et données sont présents.
Vérifie statut de l'archive dans l'écran mémoire.
N'écrase pas une AppVar utilisateur homonyme sans prévenir.
Pour CEmu, utilise un fichier OS que l'utilisateur a le droit d'utiliser ; ne redistribue pas une ROM sans licence.
Le CE Toolchain décrit des builds debug pour CEmu.
Les fonctions dbg_printf peuvent fournir des informations au debugger CEmu si elles sont présentes dans l'environnement.
Ne laisse pas de sortie debug lente dans la version de publication.
Le simulateur n'a pas tous les comportements du clavier, OS ou archivage matériel.
Passe les tests critiques sur la vraie calculatrice CE visée.
Garde un build debug et un build optimisé comparables.
Reproduis les crashs avec le même OS, bibliothèques et fichier 8XP.
Récupère le log complet de make avec la commande verbose.
Vérifie la map file et la taille du programme.
Ne corrige pas un warning v15 par un cast arbitraire sans comprendre le type eZ80.
Pour un crash de sauvegarde, relis storage-ti-ce.md avant de modifier l'ordre d'ouverture et archivage.

## 7. Sources

- [CE C/C++ Toolchain](https://ce-programming.github.io/toolchain/) — index des docs actuelles.
- [Getting Started](https://ce-programming.github.io/toolchain/static/getting-started.html) — installation, build et OS 5.5.0+.
- [Hardware Overview](https://ce-programming.github.io/toolchain/static/hardware.html) — eZ80, types et matériel.
- [FAQ](https://ce-programming.github.io/toolchain/static/faq.html) — limites mémoire ; sa mention de version de compilateur peut être en retard sur les notes de release.
- [General Coding Guidelines](https://ce-programming.github.io/toolchain/static/coding-guidelines.html) — contraintes mémoire et règles C.
- [GraphX](https://ce-programming.github.io/toolchain/libraries/graphx.html) — rendu.
- [keypadc](https://ce-programming.github.io/toolchain/libraries/keypadc.html) — lecture directe clavier.
- [fileioc](https://ce-programming.github.io/toolchain/libraries/fileioc.html) — AppVars et stockage.
- [Debugging](https://ce-programming.github.io/toolchain/static/debugging.html) — CEmu.
- [Toolchain releases](https://github.com/CE-Programming/toolchain/releases) — CEdev v15 notes.
- [Toolchain changelog](https://github.com/CE-Programming/toolchain/blob/master/changelog.md) — migration et versions.
