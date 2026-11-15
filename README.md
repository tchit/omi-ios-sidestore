# App iOS Omi pour SideStore

Recette de compilation de l'app iOS **officielle** d'Omi ([BasedHardware/omi](https://github.com/BasedHardware/omi)),
branchée sur un serveur Omi personnel (mode « harness local » prévu par Omi) et installée avec
[SideStore](https://sidestore.io) : ni Mac, ni compte développeur payant.

Ce dépôt ne contient que la recette (`.github/workflows/build-ios.yml`) et quelques correctifs (`patches/app/`) :
le code est celui d'Omi, à la même version que le serveur ; aucune donnée personnelle, aucun secret. Le serveur n'est
joignable que par les appareils de son propriétaire sur Tailscale.

## Correctifs du clone (`patches/app/`, 2026-10-08)

Appliqués par le flux au code d'Omi intact, un par un dans l'ordre des numéros (`git apply --check`, puis
`git apply`), avant la préparation. S'ils ne s'appliquent plus (nouvel `OMI_COMMIT`), la compilation s'arrête : les
refaire sur la nouvelle version.

- `0001` : plus jamais la commande 30 vers un Plaud (d'après le SDK Plaud décompilé, elle efface un
  enregistrement) ; un **Plaud NotePin S** (lié à l'app Plaud, Bluetooth chiffré) est reconnu par son nom, sinon par
  la lecture seule de son modèle (6AA50003), et Omi refuse de s'y connecter. Échec en mode fermé : un Plaud que ni
  son nom ni 6AA50003 n'identifient (caractéristique absente, vide, illisible, ou autre modèle) est refusé aussi.
  Par défaut, Omi n'écrit donc rien sur aucun Plaud et ne s'abonne à rien : quand le nom ne suffit pas, il se
  connecte le temps de lire 6AA50003, puis refuse.
  Le direct en clair du NotePin de 1re génération (série 880) n'existe plus que derrière la constante
  `PlaudDeviceConnection.directNotePinOrigineParDefaut`, à `false` : Mike n'a pas ce modèle. La passer à `true`
  rouvre le risque. Un NotePin S au nom muet (« NotePin », nom de repli d'Omi) dont 6AA50003 est absente ou illisible
  recevrait alors l'abonnement à 2BB0 puis les commandes 9, 23, 20 et 28 : Omi rend la même réponse vide pour une
  caractéristique absente et pour une lecture en erreur.
- `0002` : pas de mise à jour de firmware d'Omi pour un Plaud.
- `0003` : message « lié à l'app Plaud » au choix d'un NotePin S, carte correspondante sur la page de l'appareil,
  modèle « Plaud NotePin S » et firmware inconnu au lieu de « PLAUD NotePin » / « 1.0.0 ».
- `0004` : tests `test/plaud/` (transport simulé), lancés par le flux avant la compilation.
- `0005` (2026-10-09) : champ du nom d'un locuteur et recherche de personnes lisibles en thème clair (fond et texte
  suivent la palette du thème au lieu d'un gris très foncé codé en dur).
- `0006` (2026-10-09) : chat en discussions séparées (« Nouvelle discussion », « Discussions passées ») ; le chat
  rouvre la discussion qui était ouverte ; « Demander à Omi » depuis une conversation reprend la discussion de cette
  conversation au lieu de la discussion ouverte.
- `0007` (2026-10-09) : résumés gardés (champ `summary_history` du serveur du clone) ; flèche de l'onglet Résumé,
  menu « Résumés » pour lire une ancienne version, qui se lit sans s'éditer.
- `0008` (2026-10-10) : troisième onglet Insights (Accueil, Tâches, Insights) ; dossiers tenus par Omi à partir du
  chat, avec citation et sources de chaque élément ; l'app lit et ne peut que retirer.
- `0009` (2026-10-10) : « Corriger le résumé » (POST `/v1/conversations/{id}/corrections` du serveur du clone) ;
  l'ancienne version reste dans le menu de 0007.
- `0010` (2026-10-11) : carte « Maintenant » et bilans de la semaine en haut de l'onglet Insights ; nature de chaque
  ligne par icône et libellé, jamais par la seule couleur.
- `0011` (2026-10-20, repris d'Omi officiel) : sources de chaque section des notes d'Omi (passages, m:ss, saut dans
  la transcription et l'audio), versions gardées comprises.
- `0012` (2026-10-20, repris d'Omi officiel) : nommer plusieurs locuteurs d'un coup ; résumé régénéré une seule fois,
  à « Terminé ».
- `0013` (2026-10-20, repris d'Omi officiel) : régénération du résumé envoyée une seule fois, délai de 320 s, message
  clair en cas d'échec ; noms MagicDNS `*.ts.net` acceptés en `local_dev` (la recette garde l'adresse 100.x dans
  `OMI_DEV_HOST`).
- `0014` (2026-10-20) : « À vérifier » (propositions de la relecture de nuit : Accepter / Ignorer) et « Changements
  récents » (Annuler) dans l'onglet Insights.
- `0015` (2026-10-21) : tâches qu'Omi pense faites, marquées sans être cachées (« Omi : faite ? ») et validées en un
  geste.
- `0016` (2026-10-21) : Omi coche seul les tâches prouvées ; « Cochées par Omi » montre la raison et la preuve, et
  décocher remet la tâche à faire.
- `0017` (2026-11-11, repris d'Omi officiel) : les listes sauvegardées par l'app (appareils, vocabulaire, mémoires
  en attente, recherches récentes) ne sont plus vidées à chaque lancement sur iOS (amont #23548, lecture seulement :
  ce qui a déjà été mis de côté sous `<clé>.corrupt-…` n'est ni restauré ni effacé) ; l'ouverture d'une longue
  conversation ne compare plus chaque ligne à toutes les autres (amont #23431, cette seule fonction) ; filtre des
  mémoires : rien d'enregistré = « Tout » (amont #22792).

Après installation, un NotePin S resté appairé n'est plus connecté ; « Oublier l'appareil » dans Omi le retire.
Le bouton d'enregistrement utilise alors le micro de l'iPhone. Un Plaud non identifié n'est pas connecté non plus :
choisi dans la liste des appareils, il en disparaît sans message (il revient au scan suivant).

Refaire les correctifs : appliquer `patches/app/*.patch` un par un, dans l'ordre, sur `app/` d'Omi au nouveau commit
(un commit par correctif), corriger, puis `git format-patch` (chemins `app/…`). Les correctifs s'empilent :
27 fichiers sont touchés par plusieurs d'entre eux (les traductions `app_en.arb` et `app_fr.arb` par dix), et 0009
modifie des fichiers créés par 0007 ; un seul `git apply --check` de toute la série la refuse. Un nouveau fichier de
test doit aussi être ajouté à l'étape de tests de `build-ios.yml`, qui les nomme un par un.

Dépôt public : les tests et les messages des correctifs n'utilisent que des noms et des situations fictifs, jamais
ceux des conversations réelles.

## Compiler

Actions → **App iOS Omi pour SideStore** → *Run workflow* → Team ID Apple (10 caractères, lu dans SideStore :
Réglages → App IDs, fin de l'identifiant de SideStore). Vide pour un premier essai. Résultat : artefact
`Omi-sidestore` (fichier `Omi-sidestore.ipa`), compilé sans signature, sans app Apple Watch ni widget (limites d'un
identifiant Apple gratuit).

## Installer et renouveler

1. SideStore installé sur l'iPhone (une seule fois, depuis un ordinateur : guide docs.sidestore.io).
2. Copier `Omi-sidestore.ipa` dans Fichiers, puis SideStore → My Apps → **+** → choisir le fichier.
3. Ouvrir Omi avec **Tailscale actif** → *Sign in (local dev)*.
4. Tous les 7 jours : activer LocalDevVPN, toucher le compteur de jours dans SideStore, puis revenir sur Tailscale
   (iOS n'accepte qu'un VPN à la fois). Pas besoin de recompiler.
