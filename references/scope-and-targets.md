# Portée et diagnostic d'une cible
Sources consultées le 28 septembre 2026. Recontrôle les versions locales avant d'appliquer une recette dépendante d'Epsilon ou de CEdev.


## Sommaire
- [1. Portée prouvée](#1-portée-prouvée)
- [2. Identifier le projet](#2-identifier-le-projet)
- [3. Choisir la plateforme](#3-choisir-la-plateforme)
- [4. Noter les preuves](#4-noter-les-preuves)
- [5. Délimiter la demande](#5-délimiter-la-demande)
- [6. Sources](#6-sources)

## 1. Portée prouvée

La V1 du skill traite deux voies natives en C :
- Application externe EADK NumWorks produite en fichier NWA.
- Programme natif CEdev destiné à TI-83 Premium CE ou TI-84 Plus CE, distribué en fichier 8XP.

Ne déduis pas que d'autres modèles TI sont compatibles parce que le fichier se termine en 8XP.
Ne déduis pas qu'un binaire NWA est du code C récupérable ou un format source.
Ne confonds pas une app externe NumWorks et une app intégrée à un firmware Epsilon recompilé.
Ne confonds pas un script Python stocké par NumWorks et une app native EADK.
Ne confonds pas un AppVar de données TI et l'exécutable 8XP.
Ne décris pas un C commun comme un binaire commun.

Garde hors de la recette standard :
- TI-83 Plus monochrome et variantes Z80.
- TI-Nspire C/C++ et systèmes CX/CX II.
- TI-BASIC sur toute famille.
- Python NumWorks et ses modules.
- Applications compilées dans un firmware NumWorks personnalisé.
- Forks et firmwares communautaires qui modifient l'API ou le stockage.
- Calculatrices Casio, HP, Sharp ou microcontrôleurs non calculatrices.

Pour une cible hors périmètre, fais une recherche ciblée avant de proposer la chaîne.
Demande le modèle écrit sur la face avant ou la boîte si le nom fourni est ambigu.
Consigne exactement l'OS installé, pas seulement le modèle commercial.
Traite les variantes Édition Python comme une question de compatibilité à vérifier.
Ne présume pas qu'une variante TI avec mémoire plus grande a les mêmes restrictions de lancement.
Si le projet exige une autre famille, déclare le nouveau backend comme un chantier indépendant.

## 2. Identifier le projet

### Fichiers de départ

- Cherche le Makefile, les scripts de build, le dossier source et le fichier d'artefact généré.
- Nomme le fichier attendu avant de compiler : NWA ou 8XP.
- Cherche la chaîne réelle de compilateur et la version liée dans les journaux.
- Recherche les dépendances EADK, nwlink, CEdev, GraphX, fontlibc, keypadc et fileioc.
- Repère les fichiers de métadonnées d'app NumWorks, dont l'icône et le nom déclarés.
- Repère les AppVars de données ou les noms de sauvegarde TI.
- Cherche les scripts Python d'import, ressources embarquées et conversions d'images.
- Vérifie si le dépôt a une cible simulateur ou une cible CEmu.
- Lis les fichiers de test, fixtures et anciens formats de sauvegarde.
- Vérifie l'état de Git avant de commencer à modifier.
- Repère des travaux non validés pour ne pas les écraser.
- Cherche les instructions AGENTS.md ou équivalentes dans les dossiers concernés.

### État du projet

Classe le contexte dans une de ces catégories :
- Nouveau projet sans code.
- Template officiel cloné.
- Projet existant qui compile.
- Projet existant avec build cassé.
- Portage d'un code de bureau ou d'une autre calculatrice.
- Audit de code ou explication sans demande de modification.
- App existante avec données utilisateurs à conserver.

Ne remplace pas un template ancien sans vérifier ses différences avec le template source.
Ne change pas de chaîne d'outils pour résoudre une erreur tant que tu ne l'as pas isolée.
Ne migre pas de format de données avant de trouver les fichiers existants.
Si une réinstallation peut effacer les données, traite les données comme une dépendance à sauvegarder.
Lis la configuration et les commandes avant d'imposer une nouvelle arborescence.
Décris le comportement actuel à l'utilisateur lorsqu'il affecte le plan.

### Diagnostic minimal à consigner

Pour chaque test ou conclusion, note :
- Le modèle matériel réel.
- Le nom et la version de l'OS ou firmware.
- La branche et le commit du template ou dépôt.
- La version du compilateur et du linker.
- La version de nwlink ou CEdev.
- La commande exacte de compilation.
- Le nom de l'artefact produit.
- Le simulateur, émulateur ou appareil qui l'a exécuté.
- Les opérations effectuées avant d'observer le résultat.
- Le résultat observé, y compris les erreurs.
- La limite de cette observation.

## 3. Choisir la plateforme

### Route NumWorks EADK

Choisis-la si :
- L'application doit apparaître comme app externe dans Epsilon.
- L'utilisateur demande un NWA ou un développement C natif.
- Le projet peut utiliser le template C EADK et son API.
- Les modèles et firmwares de test peuvent installer les apps externes.
- Le développeur accepte que la persistance nécessite une méthode communautaire.

Vérifie la route dans le [template C officiel](https://github.com/numworks/epsilon-sample-app-c).
Vérifie les types et fonctions dans le [header EADK officiel](https://github.com/numworks/epsilon/blob/master/epsilon/eadk/include/eadk/eadk.h).
Vérifie la version du paquet nwlink et les instructions de lancement actuelles.
Vérifie séparément le fonctionnement sur appareil et simulateur.
Vérifie si le projet dépend de stockage Epsilon non officiellement exposé.

### Route TI CEdev

Choisis-la si :
- La cible est une TI-83 Premium CE ou TI-84 Plus CE couverte par CEdev.
- Le programme est écrit en C ou C++ avec les bibliothèques CE.
- Un programme 8XP et ses dépendances peuvent être transférés.
- L'OS ou un chargeur permet de lancer un programme natif.
- Les AppVars peuvent stocker les données qui doivent survivre au programme.

Vérifie le [démarrage CEdev](https://ce-programming.github.io/toolchain/static/getting-started.html).
Vérifie chaque API dans la documentation de la même version de CEdev.
Vérifie les notes de version, car CEdev v15 constitue un changement majeur de chaîne.
Lis l'OS exact de la calculatrice avant d'annoncer qu'un fichier 8XP se lancera.
Sépare transfert du programme, installation de bibliothèques et création d'AppVars.

### Route multi-cible

Adopte un noyau commun seulement si :
- Le modèle métier et ses transitions sont réellement les mêmes.
- Les dépendances peuvent être confinées dans des adaptateurs.
- Les types et formats de données sont définis indépendamment des ABI.
- Le rendu peut être adapté aux palettes et aux buffers propres à chaque cible.
- Les interactions nécessaires existent sur les deux claviers.
- Les tests peuvent exécuter le noyau sur hôte et deux appareils.
- Les volumes de données tiennent dans les budgets de chaque cible.

Ne force pas la même interface si l'écran ou les touches physiques rendent l'usage inférieur.
Ne force pas la même fréquence d'affichage sur les deux machines.
Ne prétends pas que des sauvegardes sont transférables parce que les modèles de données se ressemblent.
Précise si les fichiers de sauvegarde sont binaires portables ou convertis par un export.
Ajoute une cible uniquement quand le premier chemin compile et reste testable.

## 4. Noter les preuves

Marque chaque affirmation périssable selon son type :
- Documentée officiellement : API ou procédure énoncée dans la doc primaire.
- Observée dans le code : comportement visible dans une révision source précise.
- Signalée : rapport d'utilisateur ou issue isolée à reproduire.
- Testée : expérience avec modèle, OS, version et étapes consignés.
- Recommandée : règle d'architecture ou de robustesse du skill.

Associe à un test matériel les affirmations relatives au reset, à la durée de vie mémoire, au clavier ou au lancement.
Associe à une version les affirmations relatives aux options de compilateur et à la disponibilité d'une API.
Utilise les issues pour trouver des risques, pas pour déduire que toutes les machines sont touchées.
Vérifie si une issue est ouverte, fermée, résolue ou liée à une version remplacée.
Utilise la documentation de version plutôt que le seul lien latest si la version de l'utilisateur est connue.
Vérifie les headers locaux quand une documentation en ligne est ambiguë.
Enregistre un hash ou commit des dépendances communautaires copiées dans le projet.
N'appelle pas « officiel » un paquet tiers simplement parce qu'il s'intègre à Epsilon.
Ne promets pas une disponibilité permanente d'un serveur de téléchargement.
Ne prends pas une capture d'émulateur pour preuve de fonctionnement d'un appareil.

## 5. Délimiter la demande

Si la demande manque d'un détail mais qu'une partie est indépendante, avance sur cette partie.
N'interromps pas l'utilisateur pour une préférence de couleur avant d'avoir confirmé la voie de build.
Clarifie la cible quand deux modèles ont des ABI ou modes d'installation différents.
Clarifie la durée de conservation quand une réponse oui/non masquerait plusieurs types de reset.
Clarifie les données à sauvegarder si leur définition change la taille ou la migration.
Clarifie si le portage doit garder le comportement exact ou peut adapter l'interface.
Clarifie qui utilisera le binaire si le canal de distribution dépend de son système hôte.
Signale tout fichier que le déploiement pourrait remplacer avant d'exécuter une opération destructive.
Utilise un exemple réduit si la demande est large mais l'utilisateur veut commencer rapidement.
Propose des étapes réalisables dans l'ordre au lieu d'un plan abstrait.
Réserve les tests destructifs à des données de démonstration.
Note ce qui n'a pas pu être testé au lieu de remplir les trous par des suppositions.

## 6. Sources

- [NumWorks epsilon-sample-app-c](https://github.com/numworks/epsilon-sample-app-c) — point de départ pour la compilation EADK et le NWA.
- [Header EADK dans Epsilon](https://github.com/numworks/epsilon/blob/master/epsilon/eadk/include/eadk/eadk.h) — types, fonctions exposées et sections.
- [Documentation CE C/C++ Toolchain](https://ce-programming.github.io/toolchain/) — chaîne et bibliothèques pour les modèles CE.
- [CEdev Getting Started](https://ce-programming.github.io/toolchain/static/getting-started.html) — prérequis, build et sortie 8XP.
- [CEdev releases](https://github.com/CE-Programming/toolchain/releases) — évolutions qui changent les recettes.
- [CEdev changelog](https://github.com/CE-Programming/toolchain/blob/master/changelog.md) — changements par version.
- [NumWorks Epsilon issue #1547](https://github.com/numworks/epsilon/issues/1547) — stockage shared évoqué dans Epsilon.
- [Nwagyu storage docs](https://yaya-cout.github.io/Nwagyu/reference/apps/storage.html) — méthode communautaire, pas API constructeur.
