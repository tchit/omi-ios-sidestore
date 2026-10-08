# App iOS Omi pour SideStore

Recette de compilation de l'app iOS **officielle** d'Omi ([BasedHardware/omi](https://github.com/BasedHardware/omi)),
branchée sur un serveur Omi personnel (mode « harness local » prévu par Omi) et installée avec
[SideStore](https://sidestore.io) : ni Mac, ni compte développeur payant.

Ce dépôt ne contient que la recette (`.github/workflows/build-ios.yml`) et quelques correctifs (`patches/app/`) :
le code est celui d'Omi, à la même version que le serveur ; aucune donnée personnelle, aucun secret. Le serveur n'est
joignable que par les appareils de son propriétaire sur Tailscale.

## Correctifs du clone (`patches/app/`, 2026-10-08)

Appliqués par le flux au code d'Omi intact (`git apply --check`, puis `git apply`), avant la préparation. S'ils ne
s'appliquent plus (nouvel `OMI_COMMIT`), la compilation s'arrête : les refaire sur la nouvelle version.

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

Après installation, un NotePin S resté appairé n'est plus connecté ; « Oublier l'appareil » dans Omi le retire.
Le bouton d'enregistrement utilise alors le micro de l'iPhone. Un Plaud non identifié n'est pas connecté non plus :
choisi dans la liste des appareils, il en disparaît sans message (il revient au scan suivant).

Refaire les correctifs : appliquer `patches/app/*.patch` sur `app/` d'Omi au nouveau commit, corriger, puis
`git format-patch` (chemins `app/…`, chaque fichier touché par un seul correctif).

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
