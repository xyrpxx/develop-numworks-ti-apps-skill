# Affichage, clavier et performances
Sources consultées le 28 septembre 2026. Recontrôle les versions locales avant d'appliquer une recette dépendante d'Epsilon ou de CEdev.


## Sommaire
- [1. Repère écran commun](#1-repère-écran-commun)
- [2. Rendu NumWorks](#2-rendu-numworks)
- [3. Rendu GraphX](#3-rendu-graphx)
- [4. Assets](#4-assets)
- [5. Entrées](#5-entrées)
- [6. Boucle et temps](#6-boucle-et-temps)
- [7. Mesures](#7-mesures)
- [8. Sources](#8-sources)

## 1. Repère écran commun

EADK indique une largeur de 320 et hauteur de 240 dans le header consulté.
GraphX CE est conçu pour les écrans de cette famille TI, souvent 320×240.
Confirme les dimensions dans les docs et headers de la version des cibles réelles.
Utilise des coordonnées logiques dans l'interface commune.
Pour toute primitive, décide si x,y indiquent origine, centre ou baseline.
Définis le clipping d'une primitive dans son backend.
Ne laisse pas des dimensions uint16 s'enrouler quand tu soustrais ou calcules un bord.
Valide les rectangles avant conversion de signed vers unsigned.
Teste pixels et rectangles sur les quatre coins.
Teste rectangle nul, de taille négative passée au core, et rectangle hors écran.
Ne suppose pas qu'une surface entière peut tenir en RAM.
Mesure les buffers de rendu au lieu d'inférer la RAM depuis la résolution.
Ne recalcule pas les mêmes éléments à chaque frame si l'écran peut être réutilisé.
Sépare le style sémantique des valeurs RGB565 et indices palette.
Garde le même contraste visuel, pas nécessairement les mêmes valeurs de pixels.

## 2. Rendu NumWorks

La couleur EADK est de largeur 16 bits dans le header actuel.
Le header définit les constantes couleurs standard en RGB565.
Utilise eadk_display_push_rect_uniform pour remplir une région d'une couleur quand le prototype le confirme.
Utilise eadk_display_push_rect pour envoyer un buffer de pixels compatible avec la région.
Vérifie l'ordre de pixels et la taille exacte du buffer.
N'envoie pas de pointeur de 8 bits à une API qui attend un eadk_color_t par pixel.
Distingue longueur du buffer en pixels et longueur en bytes.
Une surface 320×240 RGB565 coûte 320×240×2 = 153 600 octets, avant toute donnée du jeu.
Préfère petits buffers, sprites limités ou redraw partiel quand cela réduit le pic RAM.
N'alloue pas de framebuffer complet sans test de mémoire sur le modèle ciblé.
Le texte EADK courant expose draw_string avec choix de grande police, couleur texte et fond.
Vérifie le contraste et les dimensions des lignes sur appareil.
Vérifie accents, exposants et symboles scientifiques ; la police peut limiter les glyphes.
Ne suppose pas que le fond du texte est transparent.
Teste la coordonnée de baseline si tu construis des lignes de texte.
Vérifie l'effet de eadk_display_wait_for_vblank sur la version matérielle avant de fonder une cadence.
Un ticket Epsilon peut signaler divergence simulateur/matériel ; limite la conclusion à son contexte.
Lis le header réellement inclus, pas seulement la branche master en ligne.
Garde l'appel au backend et évite de l'exposer à l'état métier.

## 3. Rendu GraphX

GraphX utilise une surface graphique avec pixels indexés et une palette.
Vérifie la profondeur et le format de la version du header avant de créer un asset.
Le modèle courant est 8 bits par pixel, les valeurs sont des indices et non des couleurs RGB888.
Configure une palette cohérente et stable avant d'afficher les sprites.
Respecte les conventions GraphX pour clip, coordonnées et assets convertis.
GraphX fournit un draw buffer distinct dans ses routines usuelles.
Utilise gfx_Begin et gfx_End selon la documentation de la version.
gfx_SwapDraw est un point d'échange visuel documenté ; ne l'appelle pas depuis le core.
Ne suppose pas que changer draw buffer efface ou dessine l'écran.
Initialise la zone visible ou la palette au début de chaque session.
Restaure l'état LCD attendu avant de rendre le contrôle à l'OS.
La GC TI peut demander à l'OS un écran au format standard ; GraphX doit être fermé selon sa callback.
Évite gfx_*_NoClip tant que les limites de la primitive ne sont pas prouvées.
Un sprite à coordonnée limite peut écrire en dehors de l'écran avec une fonction sans clipping.
Teste l'animation avec draw buffer et affichage de texte.
Vérifie que les sprites restent lisibles avec le nombre d'indices de palette choisi.
Limite les dépendances fontlibc si la police GraphX suffit à l'interface.

## 4. Assets

Conserve les images sources avec leur licence dans le dépôt.
Génère chaque format cible à partir d'une source reproductible.
Pour TI, vérifie convimg et son format de configuration correspondant à CEdev installé.
Ne copie pas une invocation convimg d'une version antérieure sans faire tourner --help ou l'exemple officiel.
Pour NumWorks, vérifie le pipeline d'external_data dans le template courant.
N'utilise pas external_data comme destination mutable.
La source PNG devrait être conservée séparément du sprite généré.
Vérifie le nombre d'images, les dimensions, palette, transparence et limites de clipping.
Fais le compte des octets générés avant de les inclure au binaire.
Compare compression et temps de décompression sur l'appareil.
Ne décode pas une grosse image d'un coup en heap si des bandes suffisent.
Garde une procédure qui recrée tous les assets depuis un checkout propre.
N'inclus pas un fichier généré local uniquement et oublié de Git.
Utilise des identifiants logiques comme COLOR_ACCENT et non les indices matériels dans le gameplay.
N'emploie pas des sprites à palette de 256 couleurs sur une interface où 16 couleurs suffisent.
Pour un texte d'écran, préfère le rendu texte si cela conserve lisibilité et réduit le binaire.
Vérifie séparément les licences d'images, polices et sons.

## 5. Entrées

L'adaptateur clavier transforme des touches matérielles en actions du modèle.
Définis un ensemble logique minimal : directions, valider, annuler, pause et action contextuelle.
Ajoute action secondaire seulement si le jeu ou app en a besoin.
Associe les touches par cible dans une table maintenue près du backend.
Ne mets pas des keycodes TI ou EADK dans le format de sauvegarde.
Distingue l'état maintenu et l'événement d'appui.
Distingue le keydown et le keyup si l'interaction dépend du relâchement.
Calcule une seule transition par appui pour une action de menu.
Utilise une répétition paramétrable et bornée pour déplacement en grille.
Teste les touches tenues, tapées rapidement, simultanées et en conflit.
Sur EADK, eadk_keyboard_scan renvoie un état multi-touche ; inspecte le header de la version visée.
Sur EADK, un événement se lit par eadk_event_get ; confirme unités et timeout de son paramètre.
Sur TI, keypadc permet scan direct et multi-touches, ce qui convient aux jeux réactifs.
Sur TI, les routines OS comme os_GetCSC peuvent mieux convenir à une attente simple d'entrée.
Ne mélange pas un scan keypadc avec un blocage OS sans tester l'état matériel.
La touche ON ou On/Off peut avoir un comportement système spécial.
Vérifie le chemin de sortie sans interception accidentelle du système.
Lis la doc keypadc sur mode scan, répétition et état d'une touche.
La doc indique que certains réglages de scan sont globaux ; initialise et restaure ce que l'app modifie.
Fais une table des différences de légende de touche entre TI-83 Premium CE et TI-84 Plus CE.
N'affiche pas seulement des symboles qui n'existent pas sur un clavier cible.

## 6. Boucle et temps

Utilise une fonction logique appelée régulièrement ; ne fais pas dépendre l'état du nombre de frames rendues.
Mesure le temps d'une frame sans utiliser une minuterie dont tu réaffectes la ressource.
Sur TI, lis la recommandation CEdev d'utiliser clock pour une mesure simple.
Ne reconfigure pas les minuteries matérielles réservés aux OS et bibliothèques sans preuve et nécessité.
Utilise la fonction EADK millis documentée derrière un wrapper NumWorks si adaptée.
Convertis le type uint64 EADK en largeur commune avec une soustraction contrôlée.
Une durée de jeu peut être portée dans uint32 si les bornes et le wrap sont gérés.
Pour comparer des tick uint32 cycliques, documente la durée maximale entre deux observations.
Ne traite pas une horloge murale comme un temps monotone.
Borne le delta après une attente utilisateur, un menu ou un suspend.
Préfère pas fixe pour simulation déterministe qui dépend de la physique.
Utilise accumulateur de temps si la durée réelle varie mais que la simulation doit rester fixe.
Fixe une limite d'updates par frame pour ne pas boucler sans fin après une pause.
Ne laisse pas les calculs de dessin modifier l'état logique.
Ne sauvegarde pas à chaque frame.
Sauvegarde après événement métier, checkpoint ou demande utilisateur.
Réduis fréquence de rafraîchissement lors d'inactivité.
Évite busy loops sans temporisation quand la touche n'est pas appuyée.
Calibre vitesse et animation indépendamment sur NumWorks et TI.
Mesure latence perçue et consommation CPU sur appareil si le projet l'exige.

## 7. Mesures

Crée une scène de profil reproductible : écran initial, jeu chargé, menus et sauvegarde.
Note le modèle, l'OS, le commit et options de build.
Mesure temps logique, temps de rendu, chargement d'assets et I/O à part.
Compte les rectangles et pixels envoyés par frame seulement si cela éclaire le goulot.
Compare rendu complet et partiel sur chaque backend.
Mesure l'empreinte mémoire du pire cas, pas seulement l'écran vide.
Mesure la taille de code et ressources après linker.
Mesure la cadence avec le code optimisé et le même scénario.
Garde une marge, car l'état de calculatrice et bibliothèques utilisateur peuvent varier.
Réduis le travail seulement après observation d'un coût élevé.
Ne déduis pas la fluidité d'un laptop ou d'un simulateur.
Reteste les scènes qui chargent réellement les textures ou palettes.
Déclare les mesures comme estimations si aucun profiler fiable n'est disponible.
Ne compare pas une version debug et une release comme si leurs coûts étaient identiques.

## 8. Sources

- [EADK header](https://github.com/numworks/epsilon/blob/master/epsilon/eadk/include/eadk/eadk.h) — écran, couleurs, clavier, événements et temps.
- [Template C NumWorks](https://github.com/numworks/epsilon-sample-app-c) — ressources et build.
- [CEdev GraphX](https://ce-programming.github.io/toolchain/libraries/graphx.html) — palettes, draw buffer, sprites, primitives.
- [convimg pour CE](https://github.com/mateoconlechuga/convimg) — conversion d'images.
- [CEdev keypadc](https://ce-programming.github.io/toolchain/libraries/keypadc.html) — scan et transitions clavier.
- [TI OS getcsc header](https://ce-programming.github.io/toolchain/headers/ti/getcsc.html) — événements clavier OS.
- [CEdev timers](https://ce-programming.github.io/toolchain/headers/sys/timers.html) — ressources temporisées partagées.
