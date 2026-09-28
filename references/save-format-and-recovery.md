# Format portable, versions et récupération
Sources consultées le 28 septembre 2026. Recontrôle les versions locales avant d'appliquer une recette dépendante d'Epsilon ou de CEdev.


## Sommaire
- [1. Définir la portée d'une save](#1-définir-la-portée-dune-save)
- [2. Enveloppe binaire proposée](#2-enveloppe-binaire-proposée)
- [3. Encodeur et décodeur](#3-encodeur-et-décodeur)
- [4. Deux slots et commit](#4-deux-slots-et-commit)
- [5. Migration](#5-migration)
- [6. Erreurs et interface](#6-erreurs-et-interface)
- [7. Vecteurs et tests](#7-vecteurs-et-tests)
- [8. Sources](#8-sources)

## 1. Définir la portée d'une save

Ce format est une recommandation applicative, pas une capacité native d'EADK ou fileioc.
Définis d'abord les données métier à restaurer.
N'enregistre pas des pixels ou handles de fonctions.
N'enregistre pas des valeurs qui peuvent être recalculées sans perte.
Sépare état de partie, réglages et caches afin de ne pas écrire des caches lourds avec chaque save.
Décide si la position d'affichage, curseur ou filtre est utile à conserver.
Décide ce qui arrive à une valeur invalide, absente ou hors domaine.
Mesure la taille maximale du payload.
Ajoute l'en-tête, checksum, deux copies, espace interne et marge.
Vérifie que les données tiennent dans les budgets de chaque plateforme.
Établis si les bytes doivent être identiques NumWorks/TI ou si un import/export traduit le modèle.
Un format portable n'implique pas qu'il existe un outil pour transférer le fichier entre machines.
Ne confonds pas sauvegarde binaire et texte lisible par l'utilisateur.
Si l'app a déjà un format livré, ajoute un décodeur historique ou une procédure explicite de conversion.
Garde les fixtures historiques avec tests.
Évite d'écrire l'ancien format à nouveau après migration.

## 2. Enveloppe binaire proposée

Voici une enveloppe de départ que le projet peut adopter ; elle n'est pas un standard constructeur.
Adapte magic et nommage à l'application, sans changer la taille sans versionner.
Utilise une séquence fixe de bytes décrite par offsets.
Exemple version 1 :
- Octets 0–3 : magic ASCII, par exemple NSAV.
- Octet 4 : version majeure du schéma, valeur 1 pour cet exemple.
- Octet 5 : flags réservés ; initialiser à 0 et refuser les bits inconnus dans la V1.
- Octets 6–7 : longueur de payload en uint16 little-endian.
- Octets 8–11 : génération uint32 little-endian.
- Octets 12..12+payload_length-1 : payload défini par schéma.
- Les quatre derniers octets : CRC32 little-endian.
- CRC calculé sur les octets 0 jusqu'à la fin du payload, sans inclure le champ CRC lui-même.
- Taille totale attendue : 16 + payload_length octets.

La longueur est un nombre d'octets exact, pas une taille de struct.
Le décodeur exige que la taille reçue soit exactement 16 + payload_length.
Si des extensions trailing sont voulues, crée un protocole explicite avant de les accepter.
Utilise les types de stockage uint8_t, uint16_t et uint32_t uniquement dans le codec.
Écris chaque champ par décalage et masque dans l'ordre défini.
Ne memcpy pas un struct directement vers le disque.
Ne dépends pas de l'endianness de l'ARM, eZ80 ou PC hôte.
N'enregistre pas int, long, size_t, bool, enum ou pointeurs directement.
Ne dépends pas du padding ou de l'alignement.
Vérifie que le payload tient dans uint16 avant l'encode.
Fixe une limite SAVE_MAX_PAYLOAD adaptée à l'app et aux deux supports.
Ne prends pas 2048 octets comme quota universel ; c'est seulement une valeur possible.
Ne dépasse pas les tailles accessibles du backend en raison d'un champ de longueur plus grand.
L'en-tête, les slots et les AppVars ont eux aussi un coût.
Le magic doit être rare dans les fichiers non-save pour limiter une reconnaissance accidentelle.
Le magic ne prouve ni version correcte ni intégrité.

### CRC

CRC-32/ISO-HDLC est un contrôle d'erreurs accidentelles, pas une authentification.
Variante usuelle : polynomial reflété 0xEDB88320, initial value 0xFFFFFFFF, XOR final 0xFFFFFFFF.
Stocke le résultat final en little-endian selon le champ CRC défini.
Vérifie le vecteur canonique des neuf bytes ASCII 123456789, résultat 0xCBF43926.
Teste l'implémentation NumWorks, TI CE et hôte contre ce vecteur.
Teste le CRC d'un fichier complet avec un golden construit indépendamment de l'encodeur.
N'ajoute pas un CRC différent sous le même nom de schéma.
Pour corriger une variante, change la version ou le format et garde le lecteur historique.
Un CRC ne protège pas contre l'utilisateur qui modifie volontairement les données.
Ne le décris pas comme chiffrement, signature ou anti-triche.

## 3. Encodeur et décodeur

Construis un buffer borné de taille maximale fixe ou allouée après validation.
Initialise tous les bytes de l'en-tête, y compris les flags réservés.
Vérifie chaque calcul de taille avant addition ou multiplication.
Pour n éléments de taille s, refuse si n > limite/s avant de calculer n*s.
N'ajoute pas un terminateur nul dans un blob binaire sauf si le payload le spécifie.
Retourne taille exacte écrite avec le status.
Refuse payload NULL si sa longueur est supérieure à zéro.
Accepte payload vide seulement si le modèle le permet.
Écris les champs dans l'ordre de la spécification.
Calcule CRC après que le header et payload soient entièrement définis.
Au décodage, vérifie le pointeur et la capacité reçus.
Vérifie longueur minimum avant lire magic ou champs.
Vérifie magic et version avant utiliser la taille déclarée comme borne.
Vérifie flags et réserves avant d'interpréter le payload.
Vérifie payload_length <= SAVE_MAX_PAYLOAD.
Vérifie la somme totale sans overflow.
Vérifie que total == taille du fichier.
Vérifie CRC avant d'allouer la taille déclarée ou parcourir des tableaux.
Décode vers état candidat.
Vérifie les invariants du domaine après CRC.
Applique l'état candidat uniquement quand tout passe.
Retourne des erreurs distinctes pour absent, taille, magic, CRC, version, domaine et mémoire.
N'utilise pas un booléen unique qui empêche l'interface de différencier corruption et absence.
Ne change pas la save sur une simple lecture.

### Représenter le payload métier

Attribue un champ et une largeur pour chaque donnée.
Spécifie si les nombres sont signés, non signés ou encodés autrement.
Pour une longueur de texte, stocke longueur explicite puis bytes autorisés.
Borne nombre de cellules, chaînes, items d'inventaire et niveaux.
Ne stocke pas des caractères UTF-8 sans avoir choisi et borné l'encodage.
Si tu codes un float IEEE, spécifie largeur, endianness et gestion NaN/infini.
Sur une calculatrice, une représentation décimale métier ou fixed-point peut être plus stable qu'un float natif.
Ajoute un champ à un payload par version de schéma, pas par recompilation implicite.
Valide références entre champs : index de cellule, niveau, item et parent.
Vérifie que les valeurs sont cohérentes avec les règles de l'application.
Ignore un cache corrompu et recalculable sans ignorer un état indispensable.
Garde l'ordre des colonnes et unités explicites pour NumSheet.
Ne sérialise pas un pointeur vers une formule ou un token TI.
Établis une représentation commune avant de coder les deux backends.

## 4. Deux slots et commit

Deux AppVars ou deux fichiers réduisent le risque qu'une écriture interrompue détruise la seule copie connue.
Ils ne garantissent pas qu'un système de fichiers assure atomicité.
Ils n'empêchent pas une panne qui endommage plusieurs zones.
Ils ne sauvent pas des données si les deux emplacements sont effacés.
Considère le double slot comme une stratégie d'application à vérifier sur chaque backend.

Au chargement :
1. Lis A et B sans les ouvrir en mode destructif.
2. Vérifie chaque fichier indépendamment.
3. Sélectionne un candidat seulement si son schéma, CRC et domaine sont valides.
4. Si un seul est valide, charge celui-là.
5. Si aucun n'est valide, signale absent, versions inconnues ou corruption.
6. Si deux valeurs ont même génération et même payload, une copie est redondante.
7. Si deux valeurs ont même génération et payload différent, signale une ambiguïté.
8. Si deux valeurs diffèrent exactement de la moitié du range uint32, leur ordre est ambigu.
9. Ne supprime ni ne réécris automatiquement les fichiers ambigus.

Pour enregistrer :
1. Construis l'état logique à sauvegarder.
2. Choisis la génération suivante selon une règle déterministe.
3. Encode et vérifie en mémoire.
4. Sélectionne le slot non choisi comme slot valide actif.
5. Écris ce slot seulement.
6. Vérifie toutes les erreurs et quantités écrites.
7. Ferme le handle si le backend le demande.
8. Relis le candidat sauvegardé.
9. Vérifie à nouveau longueur, version, CRC, domaine et génération.
10. Sur TI, archive la variable si cette durabilité est requise.
11. Relis après archivage si l'API et le backend le permettent.
12. Ne marque la save courante comme confirmée qu'après validation.
13. Garde l'ancien slot valide comme récupération.

Pour des générations uint32 qui peuvent wrap, utilise une comparaison circulaire clairement définie :
- delta = generation_a - generation_b en uint32.
- A est plus récente si delta != 0 et delta < 0x80000000.
- Si delta == 0, compare bytes et gère l'égalité ou le conflit.
- Si delta == 0x80000000, traite comme ambigu ; ne choisis pas arbitrairement.
- Teste le wrap avec les générations près de 0xFFFFFFFF.
- Ne compare pas par a > b, qui échoue au wrap.

N'incrémente pas la génération après un encode raté.
N'annonce pas réussite avant l'archivage TI si c'est une exigence.
Pour Nwagyu, ne promets pas qu'extapp_fileWrite est atomique ; teste un refus de write.

## 5. Migration

Versionne le schéma, pas seulement l'application.
Décide si une version mineure est compatible en ignorant des champs ou si chaque changement modifie major.
Un décodeur connaît exactement les versions qu'il accepte.
Pour une version plus récente que le lecteur, refuse et préserve le fichier.
Ne tente pas de « réparer » une version future en la réécrivant.
Pour une ancienne version, lis-la en lecteur dédié et valide tous ses domaines.
Convertis vers un modèle métier courant en RAM.
Crée le nouveau buffer avec le nouvel encodeur.
Écris-le dans un slot distinct de l'ancien.
Relis et valide le nouveau fichier.
Seulement après validation, marque la nouvelle version comme active.
Garde l'ancien slot jusqu'au prochain cycle d'écriture.
En cas d'échec de migration, charge l'ancien format si le jeu peut encore l'utiliser ou informe l'utilisateur.
Ne détruis pas un ancien payload en ouvrant le même nom en mode w.
Garde des fixtures binaires réelles de chaque schéma supporté.
Teste migration réussie, CRC invalide, champs inconnus et payload tronqué.
Vérifie que le même ancien fixture donne le même modèle sur les builds hôte, TI et NumWorks.
Ajoute un test qui valide que le lecteur courant refuse la version future sans modification.
Ne fais pas de migration automatique à chaque lecture si l'ancienne version reste utilisable.
Utilise un message qui explique quand une save ne peut pas être lue par cette version.

## 6. Erreurs et interface

Une sauvegarde absente au premier lancement est un état normal.
Une corruption n'est pas équivalente à absence.
Une version future n'est pas nécessairement corrompue.
Un backend indisponible n'est pas équivalent à fichier vide.
L'espace insuffisant doit laisser l'état en cours modifiable mais non confirmé sur disque.
Le succès d'écriture seule n'est pas une preuve de relisibilité.
Le succès de CRC seul n'est pas une preuve de domaine valide.
Montre un message lorsque l'app charge un slot de secours.
Indique le dernier checkpoint perdu si l'app le sait.
Propose nouvelle partie séparément de récupérer ancienne save.
Ne supprime pas les deux slots automatiquement pour réinitialiser l'application.
Donne une commande ou un menu pour effacer les données, avec confirmation explicite.
Si la persistance NumWorks n'est pas supportée, indique que la session peut être perdue.
Si la variable TI reste en RAM, indique la portée de conservation.
Ne promets pas plus que les opérations testées.
Ne présente pas CRC comme protection contre falsification.
Évite d'écrire les bytes bruts de la save dans un log utilisateur.
Ajoute une option export seulement si le transfert et le retour du format ont été testés.

## 7. Vecteurs et tests

Crée une fixture golden manuellement ou avec un outil indépendant.
Vérifie le header octet par octet.
Vérifie endianess et longueur pour les valeurs 0, 1, 255, 256, 65535 et bornes du modèle.
Vérifie le CRC avec le vecteur standard.
Vérifie un payload contenant zéro octet au milieu.
Vérifie payload de longueur nulle si admis.
Vérifie payload maximal.
Vérifie un octet de plus que la limite.
Vérifie fichier plus court que 16 octets.
Vérifie fichier dont longueur déclarée dépasse réellement la taille.
Vérifie longueur qui provoque overflow d'addition.
Vérifie mauvaise magic, flags inconnus et version future.
Vérifie CRC incorrect avec fichier par ailleurs valide.
Vérifie CRC correct mais champ métier hors limites.
Vérifie deux slots et générations identiques.
Vérifie deux slots avec un slot récent corrompu.
Vérifie générations avant et après wrap.
Vérifie l'ambiguïté de la différence 0x80000000.
Injecte open failed, partial read, partial write, full storage, archive failure et readback failure.
Après chaque échec, confirme qu'au moins l'ancien slot prévu reste valide.
Utilise CTest ou le système hôte déjà présent ; n'introduis pas un framework sans besoin.
Teste le codec sans inclure les headers NumWorks ou TI.
Compile les mêmes vecteurs sur les trois cibles.
Compare les bytes encodés entre cibles si le format se dit portable.
Teste aussi les backends réels et les opérations de reset annoncées.
Lis testing-and-debugging.md pour les appareils et événements à inclure.

## 8. Sources

- [CEdev fileioc](https://ce-programming.github.io/toolchain/libraries/fileioc.html) — sémantique d'ouverture, I/O et archivage TI.
- [Nwagyu storage](https://yaya-cout.github.io/Nwagyu/reference/apps/storage.html) — lecture/écriture communautaire NumWorks.
- [Epsilon issue #1547](https://github.com/numworks/epsilon/issues/1547) — contexte de stockage partagé.
- CRC-32/ISO-HDLC est une convention documentée largement utilisée ; ce fichier donne les paramètres et le vecteur requis pour une implémentation testable.
