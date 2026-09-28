# Développer une application externe NumWorks en C avec EADK
Sources consultées le 28 septembre 2026. Recontrôle les versions locales avant d'appliquer une recette dépendante d'Epsilon ou de CEdev.


## Sommaire
- [1. Portée et sources](#1-portée-et-sources)
- [2. Partir du template](#2-partir-du-template)
- [3. Comprendre l'artefact NWA](#3-comprendre-lartefact-nwa)
- [4. API EADK](#4-api-eadk)
- [5. Mémoire et ressources](#5-mémoire-et-ressources)
- [6. Installation et test](#6-installation-et-test)
- [7. Pannes fréquentes](#7-pannes-fréquentes)
- [8. Sources](#8-sources)

## 1. Portée et sources

Cette référence vise les apps externes NumWorks utilisant l'External Application Development Kit.
Le fichier NWA est un artefact construit, pas le code source C d'origine.
L'EADK fourni par Epsilon et le template C officiel sont le point de départ de chaque nouveau build.
Ne transpose pas le code d'une app interne Epsilon ou d'un fork de firmware sans vérifier ses symboles.
Le template évolue ; relève commit, Makefile et dépendances avant de réutiliser une commande.
L'en-tête actuel expose des types et fonctions, mais une déclaration ne prouve pas une implémentation sur tout matériel.
Pour les sauvegardes modifiables, l'EADK public n'expose pas d'API générale officielle d'écriture de fichiers.
La bibliothèque Extapp Storage de Nwagyu est une solution communautaire basée sur la structure interne Epsilon.
Traite le modèle, la version d'Epsilon et le canal d'installation comme des entrées obligatoires du diagnostic.
Pour toute nouvelle recette, recontrôle les liens listés en fin de page.

## 2. Partir du template

Utilise le dépôt [numworks/epsilon-sample-app-c](https://github.com/numworks/epsilon-sample-app-c).
Lis le README avant de copier son Makefile.
Vérifie quelle version de nwlink est référencée par le Makefile du commit utilisé.
Ne remplace pas arbitrairement une version épinglée par latest.
L'issue Epsilon [#2430](https://github.com/numworks/epsilon/issues/2430) signale une incompatibilité d'anciens nwlink avec certaines versions Epsilon récentes.
Vérifie la résolution de cette issue dans le dépôt et la release de nwlink avant d'en faire une règle.
Identifie séparément le compilateur ARM et le paquet npm.
Exécute les commandes de version présentes dans le template ou le README.
Vérifie les conditions d'installation Node/npm et du compilateur ARM.
Sur macOS, la commande Homebrew du README est un exemple ; adapte-la au système réellement utilisé.
Sur Linux et Windows, relève la méthode de distribution actuelle des toolchains.
N'assume pas que npx peut accéder au réseau dans un environnement hors ligne.
Si les dépendances sont téléchargées automatiquement, consigne leurs versions exactes.
Ne construis pas un nouveau Makefile de zéro tant qu'une cible officielle convient.
Vérifie les variables de plateforme dans le Makefile avant de changer les flags.
Lis les règles de génération de l'icône et des données externes.
Garde la séparation des sources de C, des objets, du simulateur et de l'artefact de livraison.
Supprime les répertoires générés par la commande prévue, pas avec un motif qui pourrait effacer les sources.

### Build reproductible

1. Clone le template ou crée une branche dérivée identifiable.
2. Relève le commit du template.
3. Exécute le build initial avant toute modification.
4. Conserve la sortie complète du build.
5. Note les versions ARM, Node, npm et nwlink.
6. Modifie une fonctionnalité à la fois.
7. Reproduis le build avec la même commande.
8. Compare la taille de l'artefact et les sections importantes.
9. Exécute le binaire dans le simulateur si la cible courante le permet.
10. Teste ensuite sur la calculatrice physique concernée.

Le README officiel décrit la chaîne ARM embarquée, Node.js et nwlink.
Le Makefile officiel montre lui-même les options C et l'édition de liens du projet.
La source la plus fiable pour une option de ce template est le Makefile du commit retenu.
N'invente pas de flag qui n'apparaît ni dans le README, ni dans le Makefile, ni dans l'aide de nwlink.

## 3. Comprendre l'artefact NWA

Le NWA se construit depuis des objets C/C++ et des métadonnées d'application selon les règles du template.
Utilise l'artefact produit par le build courant ; ne renomme pas manuellement un fichier d'une autre cible.
Inspecte la cible exacte de sortie dans Makefile et README.
Ne présente pas un fichier NWA comme archive de sources.
Ne promets pas une récupération du C original par décompilation.
Une analyse binaire peut révéler symboles, chaînes ou instructions, mais pas nécessairement noms et structures d'origine.
N'utilise que des sources et binaires que l'utilisateur a le droit d'analyser.
Lis les sections spéciales liées au nom, à l'icône et au niveau d'API.
Ne réutilise pas à l'aveugle des attributs de section tirés d'un exemple C++.
Vérifie le point d'entrée exact du template courant.
Certains exemples utilisent eadk_main ; d'autres anciens projets peuvent exposer main ou un wrapper.
Ne renomme jamais le point d'entrée sans vérifier ce que le linker ou le runtime attend.
L'exemple C officiel est le guide pour l'application C ; l'exemple C++ n'est pas une preuve de compilation C.
Ne mélange pas fichier .nwa pour installation avec fichier .nwb de simulateur s'il existe dans la version utilisée.
Ne suppose pas que chaque modèle ou firmware accepte chaque version d'API.
Contrôle l'API level déclaré quand le template le demande.
Garde le nom et l'icône à une taille et forme compatibles avec les règles actuelles de l'uploader.
Vérifie le nom affiché après installation, pas seulement le nom du fichier.

## 4. API EADK

Source principale : [eadk.h du dépôt Epsilon](https://github.com/numworks/epsilon/blob/master/epsilon/eadk/include/eadk/eadk.h).
Vérifie la copie locale réellement incluse dans l'environnement de compilation.
Le header définit EADK_SCREEN_WIDTH à 320 et EADK_SCREEN_HEIGHT à 240 dans la révision consultée.
Le type eadk_color_t est un uint16_t dans le header.
Les valeurs couleurs standard du header sont encodées comme RGB565.
eadk_rect_t et eadk_point_t utilisent des coordonnées et dimensions uint16_t.
L'API clavier fournit eadk_keyboard_scan et des helpers pour tester un état de touche.
Un bitmap clavier peut représenter plusieurs touches simultanées.
Pour un jeu en maintien, échantillonne l'état et calcule les fronts d'appui dans l'adaptateur.
Pour un menu à événements, vérifie le comportement du délai passé à eadk_event_get dans la révision du header.
Ne bloque pas la boucle si l'interface a besoin de répéter un maintien ou de sauvegarder.
L'API dessin expose push_rect et push_rect_uniform.
draw_string prend un texte nul-terminé, un point, un indicateur de taille et couleurs.
Vérifie la gestion des caractères accentués dans la police avant d'afficher du français.
pull_rect lit une région écran ; évite de le confondre avec un accès direct à la mémoire vidéo.
wait_for_vblank existe dans le header consulté ; ne présume pas son effet matériel sans le tester.
Les fonctions battery et usb déclarées ne sont pas nécessaires à la plupart des apps.
Des issues ont signalé des fonctions déclarées mais indisponibles sur certaines versions matérielles.
Vérifie l'existence de l'implémentation avant d'ajouter un appel à une API litigieuse.
eadk_timing_millis fournit une valeur temporelle, et msleep/usleep attendent des durées selon les signatures.
Garde les unités dans le nom de tes wrappers : now_ms, sleep_ms, sleep_us.
Le header expose eadk_random ; utilise un wrapper si tu veux pouvoir injecter une graine dans le core.
Ne traite pas l'en-tête master comme contrat immuable.
L'API des écrans et événements peut différer entre révisions Epsilon.
Place les appels EADK uniquement dans le backend NumWorks.
Pour un changement de comportement, ajoute un essai minimal ciblant l'API concernée.

### Déclarations API contre preuve matérielle

- Une déclaration compile-time confirme seulement que le symbole est déclaré.
- Une bibliothèque de liens doit aussi fournir le symbole pour la cible réelle.
- Le simulateur peut exporter des fonctions différentes de la calculatrice.
- Un appareil physique est la preuve adaptée pour les appels à dépendances matérielles.
- Un rapport Github prouve que le problème a été rencontré dans son environnement indiqué.
- Vérifie si un ticket a été corrigé ou si un nwlink ultérieur a modifié le comportement.
- Ne déduis pas un bug global d'une seule version.
- Ne déduis pas que l'absence d'un ticket garantit un appel valide.

## 5. Mémoire et ressources

Les limites de mémoire disponible dépendent de l'application, du runtime et du modèle.
Ne cite pas un quota universel d'heap EADK sans source versionnée et test associée.
Un tableau RGB565 pleine surface 320×240 consomme 320×240×2 = 153 600 octets.
Cette seule surface peut dépasser le budget raisonnable d'une app externe.
Commence par pousser de petits rectangles ou un buffer de tuiles si le jeu le permet.
Mesure taille du code, globals, heap, pile et assets séparément.
Évite les grands tableaux automatiques sur la pile.
Évite d'initialiser globalement un gros bitmap si le linker le copie en RAM.
Sors les ressources constantes dans external_data si le template et l'API le permettent.
eadk_external_data est déclaré comme pointeur vers const char.
eadk_external_data_size fournit une longueur ; n'attends pas un terminateur nul dans un blob arbitraire.
External data est livrée avec le programme et ne constitue pas une sauvegarde modifiable.
Ne confonds pas la taille de l'artefact installable et l'espace mémoire disponible à l'exécution.
Compresse les images si le coût de décompression tient dans le temps et la RAM du modèle.
Choisis un format de pixels compatible avec la fonction EADK utilisée.
Vérifie le sens des couleurs RGB565 avec au moins noir, blanc, rouge, vert et bleu.
Borne les rectangles avant de les pousser à l'écran.
N'utilise pas un buffer hors écran qui contient les dimensions de la calculatrice sans vérifier le budget.
Alloue au démarrage et vérifie chaque échec si heap dynamique nécessaire.
Garde un fallback graphique avec primitives si une ressource échoue à charger.
Ne laisse pas la logique du jeu dépendre du succès d'une police décorative.

### Exemple de budget à relever

| Élément | Mesure demandée | Décision à prendre |
| --- | --- | --- |
| Code et constantes | Taille du binaire et sections linker disponibles | Réduire code ou déplacer ressources constantes |
| Données globales | Map file et globals lourdes | Remplacer grosse copie en RAM ou compresser |
| Tas | Maximum simultané pendant l'app | Prévoir échec et supprimer allocation inutile |
| Pile | Plus grand objet local et profondeur d'appel | Déplacer gros buffers vers stockage statique maîtrisé |
| Ressources | Somme des sprites, polices et données externes | Réduire, paginer ou décoder progressivement |
| Écran | Taille du buffer selon le type pixel | Dessiner par région si buffer trop coûteux |

Ces entrées ne donnent pas elles-mêmes un quota : mesure la valeur sur le template, les flags et le matériel ciblés.

## 6. Installation et test

Utilise la méthode actuelle de l'uploader NumWorks ou le mécanisme USB documenté par le template.
Vérifie que le navigateur et la calculatrice autorisent le transfert USB si la page en a besoin.
Avant d'installer, consigne la liste d'apps externes et la procédure de restauration pertinente.
Ne promets pas que la conservation des apps externes après une mise à jour du système est garantie.
Installe le build produit après le changement courant, pas un ancien fichier à même nom.
Vérifie lancement, nom, icône, clavier, écran, sortie et re-lancement.
Teste sur les modèles et versions Epsilon que le projet annonce.
Enregistre le résultat séparément pour chaque appareil.
Teste la sauvegarde sur matériel réel avec des fichiers jetables avant les données de production.
Utilise le simulateur pour détecter vite une panne logique ou un souci de boucle.
N'utilise pas le simulateur seul pour valider les appels directs mémoire ou la persistance.
Les issues Epsilon indiquent des différences entre simulateur Linux, navigateur et matériel.
Vérifie si le binaire de simulation et le NWA ont des symboles d'API communs.
Une fonction présente dans eadk.h peut ne pas être exportée par la version du simulateur.
Garde un programme minimal pour isoler une panne de transfert ou d'affichage.
Quand le binaire plante, réduis-le et teste un appel EADK à la fois.
Consigne modèle, firmware, navigateur, nwlink et commit pour toute régression.

## 7. Pannes fréquentes

### Build réussit, appareil plante

- Compare la version de nwlink au template actuel.
- Vérifie les sections d'app et les métadonnées générées.
- Teste le sample officiel propre au lieu de diagnostiquer ton app entière.
- Enlève temporairement les appels non essentiels et les allocations.
- Compare version simulateur et Epsilon matériel.
- Consulte les tickets Epsilon pour le même modèle et la même version.
- Ne désactive pas une vérification de sécurité du système sans demande et analyse.

### Simulateur ne montre pas l'image

- Vérifie que le problème est reproductible sur l'appareil.
- Vérifie l'ordre des appels de rectangle et de texte.
- Vérifie si un appel attend vblank ou un événement.
- Vérifie le support de l'API dans le simulator build.
- Cherche le ticket Epsilon #2393 avant d'attribuer le souci à la logique du jeu.

### Transfert USB ne fonctionne pas

- Vérifie navigateur, câble, permission USB et page d'installation.
- Vérifie que le fichier est bien le NWA produit récemment.
- Vérifie le modèle de calculatrice et l'état de l'uploader.
- Reproduis le transfert avec le sample officiel.
- Note si une mise à jour de firmware ou de navigateur précède l'échec.

### Sauvegarde absente

- Vérifie d'abord si le NWA possède un backend de stockage intégré.
- Vérifie le nom de fichier lu et écrit.
- Vérifie le suffixe de données utilisateur prévu par la bibliothèque.
- Lis le retour booléen de write et l'espace avant/après.
- Ne suppose pas que external_data est modifiable.
- Consulte [storage-numworks.md](storage-numworks.md).

## 8. Sources

- [Template C officiel](https://github.com/numworks/epsilon-sample-app-c) — build, Makefile, nwlink et artefact.
- [Header EADK officiel](https://github.com/numworks/epsilon/blob/master/epsilon/eadk/include/eadk/eadk.h) — types et déclarations consultés.
- [External apps dans Epsilon](https://github.com/numworks/epsilon/tree/master/external_apps) — source et outils de test externes.
- [Nwagyu: accéder au stockage](https://yaya-cout.github.io/Nwagyu/reference/apps/storage.html) — méthode communautaire.
- [Epsilon issue #2430](https://github.com/numworks/epsilon/issues/2430) — signalement de crash lié à nwlink ancien, à revalider.
- [Epsilon issue #2326](https://github.com/numworks/epsilon/issues/2326) — API déclarée mais signalée absente sur matériel dans le cas décrit.
- [Epsilon issue #2393](https://github.com/numworks/epsilon/issues/2393) — divergence ponctuelle de rafraîchissement simulateur.
