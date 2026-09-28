# Tests, débogage et compatibilité
Sources consultées le 28 septembre 2026. Recontrôle les versions locales avant d'appliquer une recette dépendante d'Epsilon ou de CEdev.


## Sommaire
- [1. Séparer les niveaux de test](#1-séparer-les-niveaux-de-test)
- [2. Tester le noyau](#2-tester-le-noyau)
- [3. Tester chaque backend](#3-tester-chaque-backend)
- [4. Tester les sauvegardes](#4-tester-les-sauvegardes)
- [5. Tester sur simulateur et appareil](#5-tester-sur-simulateur-et-appareil)
- [6. Diagnostiquer une panne](#6-diagnostiquer-une-panne)
- [7. Matrice de compatibilité](#7-matrice-de-compatibilité)
- [8. Sources](#8-sources)

## 1. Séparer les niveaux de test

Un test C hôte valide logique ou codec mais pas API calculatrice.
Un build croisé valide compilation/linkage mais pas le comportement du système en exécution.
Un simulateur aide à répéter des scénarios et inspecter état, mais peut diverger du matériel.
Un essai sur calculatrice vérifie la cible réelle dans son OS et son environnement.
Une installation et un lancement vérifient le chemin utilisateur de livraison.
Une save qui passe un round-trip hôte n'est pas encore persistante sur l'appareil.
Distingue les niveaux dans le rapport et les conclusions.
Ne publie pas « testé sur TI/NumWorks » si seul le core a été exécuté sur PC.
Construis un scénario minimal pour chaque assertion importante.
Établis le comportement attendu avant le test.
Enregistre aussi les résultats inattendus.
Évite d'élargir un test sans risque concret identifié.
Reproduis les régressions avec même build et mêmes données.
Garde les fixtures de save et d'images source versionnées.
N'utilise pas de données personnelles pour les cas de panne.

## 2. Tester le noyau

Compile les sources core dans un test host sans headers des calculatrices.
Ajoute warnings et sanitizers disponibles sur hôte si compatibles avec le projet.
Teste transitions normales, erreur d'entrée et valeur hors domaine.
Teste état de jeu neuf, pause, reprise et checkpoint.
Teste feuille vide, cellule extrême, valeur invalide et valeurs négatives si permises.
Teste le code de conversion d'unité avec bornes explicites si l'app manipule unités.
Teste hasard avec graine constante.
Teste boucle logicielle avec temps simulé plutôt qu'attente réelle.
Teste l'invariant avant et après chaque transition métier majeure.
Teste sérialisation avec bytes attendus calculés indépendamment.
Teste la migration de chaque version prise en charge.
Teste que le core ne dépend pas de fonctions de temps ou hasard globales.
Teste échec d'allocation ou backend si la logique doit continuer en mode dégradé.
Teste récupération depuis un ancien état si la sauvegarde possède deux slots.
Ne mesure pas performance hôte comme performance cible.
Si des unités utilisent des flottants, teste les valeurs extrêmes et l'arrondi.
Le test hôte doit être rapide afin de l'exécuter après chaque modification core.
Ajoute un cas par défaut et un cas limite à chaque fonctionnalité non triviale.

## 3. Tester chaque backend

### NumWorks

- Compile à partir du template EADK courant.
- Vérifie les symboles du header et les bibliothèques de linkage.
- Lance l'app dans la cible simulateur réellement documentée.
- Teste transfert et lancement sur le modèle Epsilon visé.
- Teste clavier, saisie, sortie, rafraîchissement et temporisation.
- Teste les appels EADK qui ont été marqués comme expérimentaux ou dépendants de versions.
- Teste chargement external_data et sa taille si le projet embarque des ressources.
- Teste le backend de stockage seulement avec une révision épinglée de la bibliothèque communautaire.
- Note le modèle exact, firmware et méthode de transfert.

### TI CE

- Compile avec la release CEdev et libraries déclarées dans le projet.
- Vérifie la sortie 8XP et le linker report.
- Teste GraphX Begin/End, buffer d'écran et retour à l'OS.
- Teste scan direct keypadc et méthode d'attente de touches choisie.
- Teste entrée multiple si le gameplay s'y attend.
- Teste chaque dépendance de bibliothèque et sa procédure d'installation.
- Teste lancement sur l'OS exact, y compris arTIfiCE si le chemin utilisateur l'exige.
- Teste la sauvegarde AppVar en RAM et archive selon le besoin.
- Note TI-83 Premium CE ou TI-84 Plus CE, OS, CEdev et bibliothèques.

## 4. Tester les sauvegardes

Prépare deux slots valides avec générations différentes.
Prépare un seul slot valide et l'autre absent.
Prépare un slot récent corrompu et un slot ancien valide.
Prépare deux slots corrompus.
Prépare une version inconnue et une version trop ancienne.
Prépare une taille zéro, taille en-tête moins un et payload max.
Prépare un fichier tronqué après chaque champ critique.
Prépare un CRC invalide et un CRC valide mais des données incohérentes.
Prépare une longueur qui dépasse le buffer.
Prépare deux générations égales et bytes différents.
Prépare un wrap uint32 et le cas ambigu moitié de plage.
Simule un échec d'ouverture de slot.
Simule une écriture partielle.
Simule un stockage plein.
Simule un échec de réouverture ou readback.
Simule un échec d'archivage TI et un dialogue GC annulé.
Simule absence du backend NumWorks ou erreur de parse.
Après chaque scénario, confirme que le slot précédent n'a pas été effacé.
Sur NumWorks, teste la capacité partagée avec données jetables.
Sur TI, confirme statut archive après write et après lecture.
Teste relance et chaque événement de reset annoncé séparément.
Ne transforme pas « fermeture/réouverture app » en garantie « mise à jour Epsilon ».
Lis save-format-and-recovery.md pour règles du codec.
Lis storage-numworks.md ou storage-ti-ce.md pour les tests dépendants du backend.

## 5. Tester sur simulateur et appareil

### Simulateur NumWorks

Epsilon contient des méthodes de compilation/test pour applications externes, mais les cibles et formats dépendent du template.
Lis le README et les outils external_apps pour commit choisi.
Certains tickets signalent des différences d'API et de rafraîchissement entre simulateur et appareil.
Ne déduis pas que toutes fonctions EADK sont valides parce que l'app simule.
Ne déduis pas que tous bugs simulateur sont bugs de logique.
Teste au moins un scénario sur appareil lorsque l'API ou firmware intervient.

### CEmu

CEdev documente des builds de debug utilisables avec CEmu.
Utilise CEmu pour isoler logique, état d'écran et transitions reproductibles.
Utilise des OS/ROM que l'utilisateur est autorisé à utiliser ; ne redistribue pas de ROM.
Vérifie si fichier ou fonctionnalité vient de l'émulateur et non d'un programme réellement installé.
Teste archives et dialogs sur hardware si l'émulation ne reproduit pas exactement le comportement.
Fournis des symboles debug uniquement dans les builds où ils sont supportés.

### Calculatrice physique

Commence par une app de test sans save destructrice.
Teste modèle, OS, clavier, graphismes et lancement.
Puis effectue un write dans un slot temporaire et relis-le.
N'utilise jamais un reset usine comme premier test.
Conserve les backups de variables existantes avant toute manipulation.
Vérifie les différences de batterie, fréquence ou firmware si le résultat varie.
Rejoue après mise à jour uniquement si l'utilisateur cible cette version.
Distingue appareil de laboratoire et appareil utilisateur.

## 6. Diagnostiquer une panne

Classe la panne :
- Configuration locale.
- Pré-requis manquant.
- Compilation C/C++.
- Assemblage.
- Linkage.
- Génération ou conversion d'assets.
- Format NWA/8XP.
- Transfert.
- Lancement.
- Runtime.
- Affichage/clavier.
- Sauvegarde ou archivage.
- Reprise/migration.

Capture sortie de commande complète et première erreur.
Vérifie versions outils et source du symbole.
Réduis le cas sans supprimer le défaut.
Compare au template minimal.
Désactive une seule intégration à la fois.
Vérifie tailles de sections, pile, heap, stack frames et buffers.
Vérifie si la panne dépend du modèle ou OS.
Recherche les issues avec signature d'erreur + version exacte.
Lis la discussion complète et l'état de résolution.
Construis une reproduction indépendante de la codebase si le problème vient du firmware.
Ne contourne pas la mémoire par un cast ou appel à une API interne sans preuve.
Après correction, réexécute le test original et le test de non-régression.
Consigne la limite restant non résolue.
N'ajoute pas une issue publique si elle expose les fichiers privés du projet.

## 7. Matrice de compatibilité

Maintiens une ligne par combinaison réellement visée :

| Matériel | OS / firmware | Toolchain | Build | Installation/lancement | Save | Résultat |
| --- | --- | --- | --- | --- | --- | --- |
| NumWorks modèle exact | version Epsilon | ARM + nwlink | NWA hash/commit | transfert noté | événements testés | pass/fail/non testé |
| TI-83 Premium CE | OS exact | CEdev release | 8XP hash/commit | OS ou launcher | AppVar archive | pass/fail/non testé |
| TI-84 Plus CE | OS exact | CEdev release | 8XP hash/commit | OS ou launcher | AppVar archive | pass/fail/non testé |

Ne regroupe pas des OS différents dans une ligne si leur lancement change.
Utilise non testé au lieu de vide ambigu.
Ajoute la date de test.
Réteste une ligne seulement quand une mise à jour affecte une API, un ABI, un OS ou la procédure.
Garde une trace des résultats négatifs.
Ne présente pas le nombre de builds comme nombre d'appareils testés.
Un simulateur et un vrai calculateur sont deux environnements distincts.

## 8. Sources

- [CEdev Getting Started](https://ce-programming.github.io/toolchain/static/getting-started.html) — build, OS TI et cible.
- [CEdev debugging](https://ce-programming.github.io/toolchain/static/debugging.html) — CEmu et symboles.
- [CEdev FAQ](https://ce-programming.github.io/toolchain/static/faq.html) — mémoire runtime.
- [NumWorks external apps](https://github.com/numworks/epsilon/tree/master/external_apps) — cibles externes.
- [NumWorks issue #2387](https://github.com/numworks/epsilon/issues/2387) — divergence Linux simulator signalée.
- [NumWorks issue #2393](https://github.com/numworks/epsilon/issues/2393) — divergence ponctuelle d'affichage signalée.
- [NumWorks issue #2357](https://github.com/numworks/epsilon/issues/2357) — incident mémoire rapporté sur N0120 et environnement décrit.
