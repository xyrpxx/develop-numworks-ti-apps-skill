# Stockage persistant communautaire sur NumWorks
Sources consultées le 28 septembre 2026. Recontrôle les versions locales avant d'appliquer une recette dépendante d'Epsilon ou de CEdev.


## Sommaire
- [1. Statut technique](#1-statut-technique)
- [2. Espace disponible](#2-espace-disponible)
- [3. Intégrer la bibliothèque](#3-intégrer-la-bibliothèque)
- [4. API documentée](#4-api-documentée)
- [5. Protéger les données](#5-protéger-les-données)
- [6. Essais de conservation](#6-essais-de-conservation)
- [7. Limites connues](#7-limites-connues)
- [8. Sources](#8-sources)

## 1. Statut technique

L'EADK public documente affichage, clavier, temps et données externes en lecture seule.
Le header EADK consulté ne fournit pas d'API générale officielle de fichier mutable.
La documentation Nwagyu le confirme et décrit la bibliothèque communautaire NumWorks Extapp Storage.
Ne nomme pas cette bibliothèque « API officielle NumWorks ».
La bibliothèque vise les applications externes Epsilon et se distribue comme fichiers source C à ajouter au projet.
Elle expose des fonctions extapp_* pour accéder à des fichiers du stockage utilisateur.
La documentation indique que l'implémentation inspecte directement la structure de stockage présente en RAM.
Elle détermine le modèle à partir d'informations userland et choisit des adresses internes.
Cette dépendance aux structures Epsilon internes la rend plus fragile qu'une API publique documentée par le fabricant.
Épingle le commit exact et garde la licence de la bibliothèque dans le projet.
Vérifie son code source et son header avant intégration ; les signatures peuvent évoluer.
N'ajoute pas de pointeurs ou d'adresses internes NumWorks dans le core.
Garde l'adaptateur storage_numworks.c isolé.
Lis les notes de release et issues Epsilon qui concernent le modèle et firmware visés.
Une réussite en simulateur ne garantit pas que les mêmes emplacements ou fonctions existent sur l'appareil.
La bibliothèque décrit des cas d'usage de save pour jeux et applications ; cela ne remplace pas les essais de survie requis.

## 2. Espace disponible

L'issue Epsilon #1547 décrit une zone de stockage de 32 Kio utilisée à l'époque par scripts Python et données Poincaré.
Traite ce chiffre comme une description du stockage partagé dans le contexte de cette issue, pas comme un quota actuel garanti à chaque application.
Ne promets pas 32 Kio disponibles par NWA.
Les scripts Python, variables, séquences, fonctions et autres données utilisateur peuvent déjà occuper une partie de cet espace.
La bibliothèque documente extapp_size() pour la capacité et extapp_used() pour l'espace utilisé.
Ces fonctions mesurent le pool partagé selon l'implémentation et sa vue du stockage.
Leur résultat peut être obsolète au moment où l'écriture commence.
Calcule ta taille théorique avant l'écriture et contrôle ensuite son résultat.
Ne tente pas une allocation complète de la capacité annoncée.
Réserve un budget raisonnable à l'utilisateur et aux données système.
Limite la taille des fichiers de sauvegarde selon le modèle métier.
Ne stocke pas des caches régénérables si cela peut saturer les scripts scolaires.
Prévois l'application sans backend si la bibliothèque est incompatible.
Affiche un état non sauvegardé au lieu d'une valeur inventée de capacité libre.
Si la place manque, conserve le jeu ou document en cours en RAM et montre un message clair.
Ne supprime aucune donnée Python ou Poincaré pour faire de la place sans consentement explicite.
Ne recommande pas un reset pour résoudre « storage full » sans explication de perte de données.

### Calculer les coûts de deux slots

Pour un payload maximal de P octets et un en-tête de H octets, deux slots prennent au moins :
- Slot A : H + P octets.
- Slot B : H + P octets.
- Total minimal : 2 × (H + P).
- Ajouter les noms de fichiers et le coût interne du filesystem si la bibliothèque en a un.
- Comparer ce total à l'espace partagé, pas à l'espace total annoncé seul.
- Prendre en compte les sauvegardes de plusieurs fichiers ou parties si l'application en permet plusieurs.
- Inclure une marge pour migration avant d'adopter un format plus grand.
- Mesurer le cas de données maximales, pas une sauvegarde vide.

N'affirme pas que le système peut remplacer un fichier de façon atomique.
La documentation de l'API indique les retours et données, mais pas une garantie transactionnelle contre coupure.
Teste ce que chaque écriture fait lorsqu'elle échoue ou que le stockage est plein.

## 3. Intégrer la bibliothèque

Pars de la documentation [Nwagyu Accessing storage](https://yaya-cout.github.io/Nwagyu/reference/apps/storage.html).
Copie storage.c et storage.h depuis un commit choisi, si la licence et la compatibilité conviennent.
Ajoute storage.c à la liste de sources compilées.
Vérifie que le Makefile traite le fichier C si le projet principal compile du C++.
Incorpore le header local de la révision intégrée ; ne prédis pas la signature depuis un blog.
Documente dépôt d'origine, commit, empreinte et licence.
Vérifie l'intégration sur le modèle et Epsilon réels.
Ne pars pas du principe que la méthode marche sur tous les forks de firmware.
Évite d'éditer les sources copiées en silence ; garde un patch lisible si tu les modifies.
Utilise une abstraction du core qui copie les bytes entre état et backend.
Passe explicitement longueur de fichier à toute lecture.
Ne traite pas les fichiers .sav comme des chaînes terminées par NUL.
Ne transmets pas de pointeur de save à une fonction texte.
Ne conserve pas un pointeur vers storage interne après un appel de mutation, sauf si le code épinglé garantit sa durée de vie.
En l'absence de garantie claire, copie vers un tampon application puis relis après mutation.
Vérifie l'effet des écritures qui remplacent un nom de fichier existant.
Ne supprime pas le slot actif avant d'avoir un autre slot validé.
Ne considère pas extapp_fileExists comme preuve que le fichier est entier.
Ne considère pas extapp_fileRead non-NULL comme preuve que la version ou checksum est valide.

## 4. API documentée

Utilise uniquement les signatures déclarées dans storage.h de la révision intégrée.
La documentation Nwagyu décrit les appels suivants :
- extapp_fileRead(name, &length) retourne un pointeur de contenu ou NULL ; length est une taille binaire.
- extapp_fileWrite(name, data, length) retourne un résultat booléen de réussite selon le header documenté.
- extapp_fileErase(name) efface un nom et retourne le statut indiqué par le header.
- extapp_fileExists(name) teste si une entrée existe.
- extapp_fileList(buffer, max, prefix) remplit une liste bornée dans sa forme documentée.
- extapp_used() fournit l'espace utilisé.
- extapp_size() fournit la capacité du stockage vue par cette bibliothèque.
- extapp_calculatorModel() expose un code modèle défini par le header communautaire.

Ne code pas ces signatures de mémoire : vérifie chaque type et paramètre dans storage.h.
Ne confonds pas NULL « absent » avec toutes les erreurs possibles si l'API ne les distingue pas.
Si la bibliothèque ne distingue pas espace plein et autre échec, indique un échec général.
Vérifie le statut retour pour tous les writes et erase.
Si l'API de read retourne un pointeur interne, copie les bytes avant le prochain changement du stockage.
Passe un buffer de taille suffisante ou traite la longueur avant d'accéder au payload.
Vérifie l'overflow d'addition entre l'en-tête et longueur.
Utilise des noms propres à l'app et limite les collisions avec les scripts utilisateur.
Le format .py a un comportement particulier d'auto-import Python documenté par la bibliothèque.
Pour des bytes arbitraires, préfère un nom de fichier de données non .py, par exemple un suffixe .sav, après vérification des règles internes.
Ne préfixe pas un fichier de sauvegarde binaire comme un script Python.
Ne suppose pas que toutes les extensions sont traitées pareil par Epsilon.
Ne mélange pas les fonctions de fichiers de la bibliothèque avec eadk_external_data.
Ne considère pas external_data comme modifiable ; le header EADK la déclare constante.

### Séquence type d'accès

1. Construire le nom de slot à partir d'un préfixe réservé au projet.
2. Vérifier le nombre maximal d'octets autorisé par le modèle.
3. Vérifier existence si l'API en a besoin.
4. Lire taille et bytes ; traiter une entrée absente séparément.
5. Copier dans un tampon maîtrisé si la durée de vie du pointeur est inconnue.
6. Vérifier le format et les bornes.
7. Écrire uniquement le slot cible inactif.
8. Contrôler le booléen retour de write.
9. Relire le fichier et exécuter la validation du codec.
10. Ne confirmer la sauvegarde qu'après relecture valide.

Cette liste est une procédure prudente, pas une garantie atomique fournie par la bibliothèque.
Teste son interaction exacte sur la version épinglée.

## 5. Protéger les données

Donne un préfixe distinct à chaque application et un nom de slot stable.
Évite les noms réservés d'extensions Python ou d'apps existantes.
Ne supprime jamais des fichiers à préfixe inconnu.
Lis A et B avant de décider lequel écraser.
Écris dans le slot non sélectionné comme valide le plus récent.
Vérifie la capacité de destination avant l'écriture, mais traite cette mesure comme estimation.
Contrôle longueur, checksum, magic, version et invariants après lecture.
Si A est valide et B corrompu, garde A et propose une nouvelle écriture dans B.
Si les deux slots sont invalides, ne présente pas l'état vide comme une sauvegarde restaurée.
Lors d'une migration, conserve au moins un slot lisible dans l'ancien format jusqu'à validation.
Ne compresse pas puis écris directement dans le seul slot valide.
Ne lance pas une sauvegarde à chaque frame.
Sauvegarde après modification logique, checkpoint, menu quitter ou demande explicite.
Coalesce les modifications rapprochées pour limiter les écritures répétées.
Pense au coût CPU de calcul CRC si le payload est grand.
Ne stocke pas de copies identiques de ressources statiques déjà dans le NWA.
N'enregistre que les valeurs nécessaires pour reprendre.
Rends l'état sauvegardé visible avec heure ou marqueur uniquement si l'app peut connaître l'heure.
Quand write échoue, garde l'état de travail et indique qu'il n'est pas sur disque.
Ne masque pas erreur de stockage par « sauvegarde OK ».

## 6. Essais de conservation

Définis les mots avant de tester :
- Relancer signifie quitter l'application puis l'ouvrir à nouveau.
- Éteindre signifie opération normale d'extinction puis rallumer.
- Redémarrer signifie cycle complet d'alimentation s'il est possible sans perte de matériel.
- Réinitialiser peut signifier redémarrage logiciel ou effacement de données ; demande lequel.
- Mettre à jour signifie le chemin précis utilisé pour mettre à jour Epsilon.
- Réinstaller signifie la procédure d'installation exacte du NWA.
- Effacer application signifie suppression par le moyen qui existe dans le firmware testé.

Teste des données jetables, jamais des données utilisateur irremplaçables.
Pour chaque essai, consigne modèle, région, firmware, commit Epsilon, révision storage, date et résultat.
Écris une valeur sentinelle unique dans A et une autre dans B avant l'action testée.
Relance l'app et compare chaque octet décodé, pas uniquement un message de succès.
Vérifie aussi si l'installation du NWA a remplacé ou effacé les apps externes.
Teste d'abord la relance normale.
Teste ensuite extinction et démarrage normal.
Teste une mise à jour et une réinstallation sur un profil qui peut être restauré.
Teste la saturation du stockage partagé avec données temporaires uniquement.
Confirme que l'ancienne copie reste chargeable après write refusé.
Ne teste pas un reset d'usine sur calculatrice personnelle sans plan de restauration.
Ne déduis pas la conservation après mise à jour de la conservation après relance.
Ne déduis pas que toutes versions Epsilon ont la même structure mémoire.

### Tableau de preuve à tenir

| Opération | Résultat | Appareil/version | Preuve |
| --- | --- | --- | --- |
| Fermer/réouvrir l'app | À tester | modèle + Epsilon | valeur exacte relue |
| Extinction normale | À tester | modèle + Epsilon | valeur exacte relue |
| Redémarrage | À tester | modèle + Epsilon | valeur exacte relue |
| Mise à jour Epsilon | À tester | ancienne → nouvelle version | valeur exacte relue |
| Réinstallation NWA | À tester | procédure notée | valeur exacte relue |
| Espace partagé saturé | À tester | quantité mesurée | ancien slot valide |
| Suppression app | À tester | procédure notée | présence/absence des saves |

Tant qu'une case est à tester, ne promets pas que l'événement conserve la save.

## 7. Limites connues

La méthode lit et modifie une structure d'Epsilon non promise comme API stable.
Un changement de layout interne peut rendre la lecture fausse ou invalider les écritures.
Le code choisit modèle et adresses depuis des informations du firmware ; audite cette décision.
Une custom userland ou firmware alternatif peut avoir une structure incompatible.
L'espace disponible est partagé avec d'autres données utilisateur.
L'écriture peut échouer ; toujours vérifier retour.
Un fichier portant l'extension .py peut avoir des métadonnées ou règles d'import spécifiques.
Un pointeur de lecture peut référer à une zone gérée par la bibliothèque, pas à une copie indépendante.
Le simulateur peut être configuré différemment du matériel.
La présence de l'app dans un installeur tiers ne transforme pas cette méthode en API officielle.
La bibliothèque libre peut être copiée dans une app ; respecte sa licence et conserve sa provenance.
Le skill doit recommander ce backend seulement si l'utilisateur comprend cette dépendance communautaire.
Si l'application ne peut pas supporter ce risque, propose une autre route ou un fonctionnement non persistant.
Ne propose pas d'écrire dans un emplacement mémoire brut choisi par adresse depuis l'app.
Ne recommande pas des syscalls non documentés pour contourner EADK.
Ne promets pas une méthode alternative stable sans sources et tests.

## 8. Sources

- [Nwagyu: Accessing storage](https://yaya-cout.github.io/Nwagyu/reference/apps/storage.html) — statut non officiel, intégration, méthodes de fichier et fonctionnement interne.
- [NumWorks Extapp Storage sur Framagit](https://framagit.org/Yaya.Cout/numworks-extapp-storage) — source C ; Framagit peut exiger un navigateur pour lire les fichiers.
- [Epsilon issue #1547](https://github.com/numworks/epsilon/issues/1547) — zone de 32 KiB partagée évoquée, datée et contextualisée.
- [Header EADK](https://github.com/numworks/epsilon/blob/master/epsilon/eadk/include/eadk/eadk.h) — EADK public et external_data constante.
- [NumWorks external apps sample](https://github.com/numworks/epsilon-sample-app-c) — construction de l'application externe.
