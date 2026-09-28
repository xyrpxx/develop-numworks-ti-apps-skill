---
name: develop-numworks-ti-apps
description: "Guide les agents qui créent, portent, corrigent, compilent ou distribuent des applications et jeux C pour NumWorks EADK (.nwa) et TI-83 Premium CE / TI-84 Plus CE avec CEdev (.8xp). Couvre le choix de la cible, le noyau portable, l'affichage, le clavier, la mémoire et les sauvegardes. À utiliser pour développer une app de calculatrice, porter du C entre NumWorks et TI CE, déboguer un build ou rendre une sauvegarde persistante. Ne pas extrapoler les recettes CE aux TI non CE ou à TI-Nspire."
---

# Développer des apps C NumWorks et TI CE

## Instruction immédiate

Commence par identifier la calculatrice, son système, le fichier à produire, l'état du projet et la tâche exacte. Lis seulement les références nécessaires à ce cas, puis inspecte les sources réelles du dépôt avant de modifier le code. Garde la logique utile au projet, choisis une méthode de plateforme documentée, compile, teste et rapporte ce qui a été vérifié.

Pars du principe que l'agent sait déjà coder en C. Utilise ce skill pour orienter ses décisions, lui faire repérer les différences matérielles et API, et encadrer les opérations fragiles. Ne transforme pas la réponse en cours général sur les variables, les boucles, les fonctions ou les algorithmes courants.

## Ordre de priorité

1. Respecte la demande explicite et le modèle réellement visé.
2. Lis les versions présentes dans le dépôt, les fichiers de configuration et les en-têtes réellement compilés.
3. Privilégie les documents officiels de la chaîne et le code source de l'API utilisée.
4. Utilise les projets communautaires pour les fonctions que l'API officielle ne fournit pas ; signale leur statut.
5. Ne présente pas une issue isolée comme une règle universelle.
6. Ne transforme pas une valeur issue d'une documentation en garantie de mémoire libre ou de compatibilité matérielle.
7. Mesure ou teste sur appareil tout comportement dépendant d'un firmware, de l'OS ou d'un modèle.
8. Si la preuve manque, indique le test nécessaire et rédige le code pour échouer sans perdre les données.
9. Ne remplace pas une architecture existante avant de l'avoir comprise.
10. N'invente jamais un nom de fonction, un flag de compilation, un format de fichier ou un comportement du système.

## Démarrer par un triage bref

Recueille les faits connus dans le prompt, le dépôt et les fichiers de build. Demande uniquement les informations qui empêchent réellement de choisir une route ; continue les vérifications indépendantes pendant ce temps.

| Question | Réponse qui change le plan |
| --- | --- |
| Quel modèle exact ? | NumWorks N0110/N0115/N0120, TI-83 Premium CE, TI-84 Plus CE ou autre |
| Quel système ou firmware ? | Epsilon/version ou version TI OS ; nécessaire si l'installation ou les APIs varient |
| Quel langage et format ? | EADK C vers NWA, CEdev C/C++ vers 8XP, Python, TI-BASIC, firmware intégré |
| Quel est le point de départ ? | Dépôt existant, source d'un jeu, prototype ou projet vide |
| Quel résultat est demandé ? | Audit, architecture, correctif, build, portage, test, stockage ou livraison |
| Quelles données doivent survivre ? | État, progression, réglages ; relance, extinction, effacement RAM, mise à jour |
| Quelle chaîne est installée ? | Version et commande du compilateur, nwlink, CEdev, make, Node |
| Quel est le canal de distribution ? | Site NumWorks, câble, TI Connect CE, CEmu ou méthode communautaire |

Quand l'utilisateur ne connaît pas le modèle ou l'OS, explique où relever l'information ou propose le diagnostic minimal. Ne demande pas un détail déjà fourni par les fichiers du dépôt.

## Choisir la voie de développement

- Pour une app externe NumWorks en C, pars du template C EADK officiel et du NWA produit par sa chaîne.
- Pour une TI-83 Premium CE ou TI-84 Plus CE en C/C++, pars du CE C/C++ Toolchain et du 8XP produit par CEdev.
- Le suffixe NWA ou 8XP ne donne pas à lui seul le modèle, l'OS ou la méthode de lancement.
- Un binaire NumWorks ne s'exécute pas sur TI ; un binaire CE ne s'exécute pas sur NumWorks.
- Le cœur C peut être partagé si son ABI, ses données, son entrée et ses dépendances restent portables.
- Chaque cible garde son propre entry point, build, rendu, clavier, horloge et stockage.
- Pour NumWorks Python, TI-BASIC, TI-Nspire, TI non CE ou firmware Epsilon intégré, arrête d'appliquer la recette native décrite ici ; vérifie une voie spécifique.
- Si le besoin porte surtout sur apprentissage rapide ou script scolaire, compare aussi NumWorks Python, sans confondre modules et API C.
- Vérifie l'effet de l'OS TI sur le lancement avant d'annoncer qu'un 8XP s'ouvrira directement.
- Pour un projet multi-cible, exige un artefact et un test propres à chaque calculatrice.

## Séquence de travail par demande

### A. Examiner le projet

- Liste le dépôt et repère les consignes locales, scripts, tests, CI et fichiers de build.
- Vérifie les modifications déjà présentes et ne les écrase pas.
- Identifie le vrai point d'entrée par cible.
- Repère où le projet stocke son état métier, ses assets et ses dépendances.
- Cherche les symboles et macros du template au lieu de recopier un exemple ancien.
- Lis les versions consignées et compare-les à celles réellement installées.
- Lances un build existant avant le changement si l'environnement le permet.
- Résume le problème initial en un cas reproductible.
- Note les tests que le projet a déjà, et ceux qui manquent pour la demande.
- Lis [scope-and-targets.md](references/scope-and-targets.md) si la cible ou la portée est ambiguë.

### B. Établir le plan minimal

- Découpe le besoin en comportement commun et besoins propres à la plateforme.
- Place règles et modèle d'état dans le noyau C quand ils sont véritablement partagés.
- Place appels EADK ou CEdev derrière des adaptateurs dédiés.
- Détermine les budgets mémoire, taille des assets, cadence, données à conserver et espace de save.
- Décris la migration de données avant de changer le format d'une sauvegarde existante.
- Précise les tests du cas nominal et les échecs à injecter.
- Pour une demande de revue ou d'explication, livre l'analyse demandée sans modifier arbitrairement le projet.
- Pour une correction simple, réutilise l'architecture plutôt que de créer un framework général.
- Lis [portable-architecture.md](references/portable-architecture.md) pour définir les frontières du code.
- Lis [display-input-performance.md](references/display-input-performance.md) si le travail touche au rendu, clavier, temps ou fluidité.

### C. Vérifier les sources avant les appels de plateforme

- Ouvre le header réellement inclus au build et relève signature, types et macros associés.
- Ouvre la documentation de même version que l'outil ; si seule la version latest est disponible, compare au header installé.
- Utilise les exemples officiels comme preuve de structure, pas comme preuve que chaque périphérique ou firmware se comporte pareil.
- Pour NumWorks, lis la référence EADK et le Makefile du template courant.
- Pour TI, lis la page CEdev correspondant à la bibliothèque et à la version CEdev.
- Pour une bibliothèque communautaire, épingle dépôt, commit, fichier source, licence, modèle et OS testés.
- Marque les affirmations de code source comme « API », « implémentation observée » ou « comportement mesuré ».
- Si deux pages se contredisent, relève leurs versions et contrôle l'exécutable installé.
- Exemple de conflit à surveiller : la FAQ CEdev actuelle peut mentionner Clang 17 quand le changelog CEdev v15 annonce LLVM/Clang 19.
- Lance la commande de version locale avant de choisir des options de compilateur.
- Consulte [numworks-eadk.md](references/numworks-eadk.md) pour le NWA et EADK.
- Consulte [ti-ce-toolchain.md](references/ti-ce-toolchain.md) pour le CEdev et 8XP.
- Ne prétends pas que cette recherche confirme une fonctionnalité si tu as seulement trouvé son prototype.

### D. Implémenter sans casser le portage

- Garde les types de données du cœur explicites et assez étroits pour la cible.
- Ne lie pas le cœur à des dimensions, couleurs ou keycodes physiques.
- Convertis chaque touche physique en une action logique au backend.
- Dessine en petites régions tant que le profil ne justifie pas un buffer plus grand.
- Sépare assets source et données générées par cible.
- Évite une allocation sur la pile TI si sa taille dépasse clairement son budget documenté.
- N'alloue pas tout l'écran NumWorks RGB565 sans prouver que ce buffer peut tenir avec le reste de l'application.
- Contrôle tous les retours des fonctions d'allocation, lecture, écriture, archivage et fermeture.
- Ne garde pas un pointeur vers des données de stockage au-delà de sa durée de vie documentée.
- N'exécute pas un stockage depuis une boucle de rendu ou à chaque image.
- Ne modifie pas un fichier de save en place si un échec pourrait détruire la seule copie valide.
- Ne centralise pas artificiellement toutes les différences dans une API gigantesque.
- Lis [save-format-and-recovery.md](references/save-format-and-recovery.md) avant de créer ou migrer le format persistant.

### E. Traiter les sauvegardes prudemment

- Écris d'abord ce que « survivre » veut dire pour cette app : relance, sortie vers l'OS, extinction, redémarrage, effacement RAM, mise à jour ou désinstallation.
- Distingue support mutable, données intégrées en lecture seule et RAM de travail.
- Sur NumWorks, l'EADK public ne fournit pas d'API officielle générale d'écriture de fichiers. L'accès étudié par Nwagyu est communautaire, dépend d'internals Epsilon et doit rester une option explicitement non officielle.
- Ne présente pas les 32 Kio du stockage partagé Epsilon comme libres ou réservés à l'app.
- Sur TI CE, un AppVar est séparé du programme compilé ; l'archive et la RAM ont des garanties différentes.
- N'ouvre pas le slot actif en mode TI `w` pour le remplacer : CEdev documente que ce mode supprime la variable existante.
- Utilise deux slots ou une méthode transactionnelle prouvée ; vérifie l'ancien et le nouveau fichier avant de choisir.
- Contrôle magic, version, longueur, checksum, bornes et invariants métier avant d'appliquer un état lu.
- Rends tout échec d'écriture et toute récupération visibles à l'utilisateur.
- N'annonce une résistance au reset qu'après l'essai correspondant sur une calculatrice réelle.

### F. Compiler puis tester

- Lance les tests hôte du noyau quand il existe.
- Compile chaque cible séparément avec son Makefile et sa toolchain réels.
- N'interprète pas un succès de compilation comme une validation sur calculatrice.
- Utilise simulateur ou CEmu pour accélérer les essais, puis valide sur chaque famille réelle ciblée.
- Pour les saves, teste fichier absent, version courante, version future, corruption, troncature, plein, write partiel et mauvais archivage.
- Pour la saisie, teste appui bref, maintien, relâchement, touches simultanées et touche de sortie.
- Pour le rendu, vérifie coins écran, clipping, palette, texte français et limites de frame.
- Pour une mesure de performance, indique appareil, build, scène, cadence et méthode.
- Pour un résultat non testé, marque-le explicitement non testé.
- Lis [testing-and-debugging.md](references/testing-and-debugging.md) pour la matrice complète.
- Si un test physique destructif est requis, utilise des données de test et explique l'impact avant toute manipulation.
- Ne perds pas de temps à refaire des tests déjà concluants sauf si la modification touche leur risque.

## Choisir le parcours de travail correspondant

### Créer une application sur une seule cible

- Confirme d'abord que l'utilisateur veut une application native et identifie son modèle exact.
- Choisis la chaîne officielle documentée pour cette famille.
- Contrôle les prérequis installés et leurs versions.
- Crée le projet à partir du template officiel actuel, s'il existe.
- Compile un exemple minimal sans logique métier supplémentaire.
- Vérifie le chargement et les contrôles du clavier sur la cible.
- Ajoute la logique demandée après ce contrôle de base.
- Ajoute les assets seulement après avoir mesuré leur taille et leur format cible.
- Ajoute une sauvegarde uniquement si l'application doit reprendre un état modifiable.
- Livre l'artefact avec la méthode d'installation adaptée au modèle et à l'OS.
- Garde un journal des appareils et versions testés.
- N'ajoute pas le second backend si l'utilisateur ne demande pas le portage.

### Porter une logique existante sur l'autre famille

- Liste les dépendances directes du noyau actuel.
- Repère les appels graphiques, clavier, horloge, stockage et macros ABI liés à la première cible.
- Sépare le modèle et les transitions des appels de plateforme avant de dupliquer la logique.
- Choisis les types portables en mesurant sizeof uniquement si la valeur influe sur le code.
- Compare les tailles écran et palettes, mais garde la conversion dans chaque renderer.
- Compare l'état des touches et le mode d'entrée avant de traduire les contrôles.
- Implémente une seule interface de backend à la fois.
- Compile d'abord le noyau sur cible hôte puis les deux toolchains.
- Rends les divergences visuelles et de performance explicites au lieu de promettre le même rendu.
- Vérifie séparément le fichier de sauvegarde et son éventuelle compatibilité binaire.
- Ne suppose pas qu'un nom de touche identique implique un même code clavier.
- Garde des builds distincts et sélectionne la cible dans le système de build.

### Ajouter une sauvegarde à un projet sans stockage

- Définis les champs métier nécessaires à la reprise et ceux qui peuvent être recalculés.
- Définis les événements exacts que l'application doit survivre.
- Mesure l'empreinte courante et le pire cas de payload.
- Choisis un codec à champs explicites indépendant de l'ABI.
- Crée un backend par plateforme, avec un résultat d'erreur portable.
- Ajoute le chargement avant l'écran d'accueil, mais après initialisation minimale des services requis.
- Fournis un état neuf valide quand aucun fichier n'existe.
- Ajoute un indicateur d'état modifié et des points de sauvegarde appropriés.
- Ajoute corruption et écriture refusée aux tests avant d'exposer la fonctionnalité.
- Lis les références des deux backends si l'application cible les deux familles.
- Pour tout format déjà livré à des utilisateurs, ajoute un chemin de migration sans toucher à l'ancienne save.

### Dépanner un build ou un crash

- Identifie l'étape exacte qui échoue : configuration, compilation, édition de liens, empaquetage, transfert, lancement ou runtime.
- Reproduis le cas avec les mêmes versions et la même cible.
- Lis la première erreur complète, puis confirme son symbole dans le header ou la documentation.
- Vérifie si l'erreur apparaît uniquement dans le simulateur ou uniquement sur matériel.
- Réduis le projet à la plus petite action qui reproduit la panne.
- Vérifie pile, globals, heap et taille des données avant d'accuser l'API.
- Désactive temporairement une dépendance à la fois, sans supprimer sa configuration permanente.
- Consigne modèle, OS, toolchain, options de build et étapes exactes.
- Cherche une issue pertinente après avoir obtenu le cas reproductible.
- Ne transforme pas la correction d'un contournement en garantie générale.
- Ajoute un test de non-régression pour chaque cause confirmée.
- Rapporte les hypothèses restantes séparément des faits observés.

### Réaliser une revue d'architecture ou de stockage

- Commence par lire les fichiers existants sans les modifier.
- Cherche les données persistantes et le moment où elles sont écrites.
- Vérifie si une écriture destructive peut atteindre le seul slot valide.
- Vérifie que chaque erreur de backend remonte au modèle et à l'interface.
- Repère les accès hors limites et les conversions de largeur non contrôlées.
- Repère les appels de save dans le chemin de rendu ou d'entrée rapide.
- Compare les claims de compatibilité aux versions prouvées.
- Donne une liste classée par risque avec chemins, fonctions et test proposé.
- N'invente pas une réécriture complète quand un défaut isolé suffit.
- Si la revue est terminée sans modification de code, le dis clairement.

## Règles de persistance NumWorks

- Commence par le header EADK officiel pour vérifier que l'app ne confond pas external_data avec une API mutable.
- Examine la révision exacte de storage.c/storage.h de NumWorks Extapp Storage avant intégration.
- Isole ses fonctions extapp_* derrière storage_numworks.c ; n'inclus pas storage.h dans le noyau.
- Confirme que l'application écrit un fichier de données qui n'est pas destiné à être un script Python.
- Vérifie retours, limites de taille, existence, effacement, espace utilisé et comportement des pointeurs.
- Copie les données lues dans un tampon détenu par l'application si la mutation suivante peut invalider leur adresse.
- Interroge extapp_used() et extapp_size() comme estimation de capacité partagée ; contrôle le retour réel de write.
- Teste fermeture et relance de l'app.
- Teste extinction/reset/mise à jour uniquement sur appareil de test et rapporte chaque résultat séparément.
- Ne déduis pas la tenue au redémarrage matériel d'un test de relance d'app.
- Conserve l'état courant si une écriture échoue.
- Pour les détails, lis [storage-numworks.md](references/storage-numworks.md).

## Règles de persistance TI CE

- Confirme que la cible est un modèle CE pris en charge et que le projet lie fileioc.
- Distingue le 8XP de l'AppVar, qui conserve les données.
- Lis le statut d'archive avant de décider qu'une save survivra à l'effacement de RAM.
- Lis les deux slots sans modifier les variables.
- Valide chaque payload avant de calculer le slot cible.
- Ouvre uniquement le slot inactif en mode d'écriture destructif.
- Vérifie le nombre d'éléments écrits avec ti_Write ; size et count déterminent les unités.
- Contrôle le résultat de ti_SetArchiveStatus et ferme tout handle sur toute branche.
- Prends en charge l'éventuel prompt de garbage collection sans casser le mode vidéo GraphX.
- Relis la variable archivée et son contenu avant d'afficher « sauvegarde réussie ».
- Lis [storage-ti-ce.md](references/storage-ti-ce.md) pour le séquencement complet et les retours.
- Ne conserve pas un pointeur ti_GetDataPtr au-delà des mutations de variables ou mouvements d'archive.

## Contrat du format portable

Applique ces règles à toute sauvegarde commune NumWorks/TI, en adaptant le modèle si l'application impose un autre format.

- Sérialise champ par champ avec des largeurs et une endianness documentées.
- Ne sérialise jamais un struct C brut, un enum sans largeur fixée, un pointeur ou un type dont le padding est implicite.
- Bornes toutes les longueurs avant les copies et avant les accès au support.
- Conserve le magic et la version de schéma séparés de la version de l'app.
- Valide le CRC avant de parcourir le payload.
- Valide ensuite les invariants métier avant de rendre l'état actif.
- Refuse une version future sans interpréter ses octets.
- Migre une ancienne version vers un nouveau tampon ; garde la source intacte jusqu'à validation de la nouvelle copie.
- Préfère deux slots et une génération si le backend l'autorise.
- Traite CRC comme détection d'erreur accidentelle, jamais comme chiffrement ou signature.
- Utilise les vecteurs golden et tests d'injection dans [save-format-and-recovery.md](references/save-format-and-recovery.md).

## Contrôle de qualité minimal

Avant de conclure une tâche, vérifie les points applicables :

- La famille de matériel et son OS sont identifiés ou annoncés inconnus.
- Les appels d'API cités existent dans le header et la version déclarés.
- Le résultat cible est bien un NWA ou un 8XP suivant la route demandée.
- Les fichiers de build existants et les changements utilisateur sont préservés.
- Les différences EADK, CEdev, ABI, rendu et stockage sont isolées par backend.
- Les limites mémoire viennent de la documentation ciblée et gardent une marge.
- Les buffers et allocations sont mesurés ou bornés.
- Les erreurs de persistance ont un résultat utilisateur et ne détruisent pas la dernière copie valide.
- Le format est versionné, borné et validé avant utilisation.
- Les options d'archive TI et la disponibilité du stockage NumWorks ont été traitées.
- Les tests nommés ont réellement été exécutés sur l'environnement signalé.
- Les simulateurs ne sont pas présentés comme preuve d'un comportement matériel non mesuré.
- Les commandes dépendent d'une version donnée ou demandent explicitement un contrôle de version.
- Les licences des fichiers réutilisés ont été vérifiées avant redistribution.
- Le résumé final distingue code modifié, build réussi, simulateur réussi et essai matériel réussi.

## Orienter l'agent vers les références

Ouvre seulement les documents utiles à la demande ; chaque lien ci-dessous est directement accessible depuis ce SKILL.md.

| Sujet observé | Référence à lire |
| --- | --- |
| Modèle incertain, NWA/8XP, OS et périmètre | [scope-and-targets.md](references/scope-and-targets.md) |
| Template NumWorks, EADK, compilation et API | [numworks-eadk.md](references/numworks-eadk.md) |
| CEdev, AppVars de bibliothèques, 8XP et OS TI | [ti-ce-toolchain.md](references/ti-ce-toolchain.md) |
| Noyau C commun, ABI, données et interfaces | [portable-architecture.md](references/portable-architecture.md) |
| Clavier, écran, assets, boucle et temps | [display-input-performance.md](references/display-input-performance.md) |
| Stockage Epsilon accédé par la bibliothèque communautaire | [storage-numworks.md](references/storage-numworks.md) |
| AppVars, fileioc, RAM, archive et garbage collection | [storage-ti-ce.md](references/storage-ti-ce.md) |
| Format, génération, CRC, migration et récupération | [save-format-and-recovery.md](references/save-format-and-recovery.md) |
| Hôte, simulateurs, appareil physique et cas de panne | [testing-and-debugging.md](references/testing-and-debugging.md) |
| Installation, artefacts, debug et livraison | [release-and-troubleshooting.md](references/release-and-troubleshooting.md) |

## Rapporter le résultat

- Indique la plateforme, le modèle et la version réellement vérifiés.
- Résume la modification au niveau du comportement, pas seulement des fichiers.
- Cite les commandes exécutées et leur résultat utile.
- Distingue compile, simulateur, émulateur et calculatrice physique.
- Pour une save, indique les événements de conservation effectivement éprouvés.
- Pour une API communautaire, indique le dépôt, la révision et le risque de compatibilité.
- Pour une limite, indique l'observation, le seuil ou l'erreur au lieu d'une promesse.
- Si la recette ne peut pas être vérifiée, fournis le test exact restant à faire.
- Ne prétends pas que le skill couvre toutes les calculatrices TI.
- Ne prétends pas qu'une sauvegarde NumWorks survivra à une mise à jour ou à un reset sans preuve ciblée.

## Sources de base à revalider

Les références détaillées doivent maintenir une date de contrôle et une source pour chaque recette fragile.

- [EADK header officiel](https://github.com/numworks/epsilon/blob/master/epsilon/eadk/include/eadk/eadk.h)
- [Template C officiel NumWorks](https://github.com/numworks/epsilon-sample-app-c)
- [Code source Epsilon](https://github.com/numworks/epsilon)
- [Issue Epsilon #1547 sur le stockage partagé](https://github.com/numworks/epsilon/issues/1547)
- [Documentation Nwagyu Extapp Storage](https://yaya-cout.github.io/Nwagyu/reference/apps/storage.html)
- [Documentation CE C/C++ Toolchain](https://ce-programming.github.io/toolchain/)
- [fileioc et AppVars CEdev](https://ce-programming.github.io/toolchain/libraries/fileioc.html)
- [GraphX CEdev](https://ce-programming.github.io/toolchain/libraries/graphx.html)
- [keypadc CEdev](https://ce-programming.github.io/toolchain/libraries/keypadc.html)
- [FAQ mémoire CEdev](https://ce-programming.github.io/toolchain/static/faq.html)
- [Versions CEdev](https://github.com/CE-Programming/toolchain/releases)
