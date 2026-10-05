# App iOS Omi pour SideStore

Recette de compilation de l'app iOS **officielle** d'Omi ([BasedHardware/omi](https://github.com/BasedHardware/omi)),
branchée sur un serveur Omi personnel (mode « harness local » prévu par Omi) et installée avec
[SideStore](https://sidestore.io) : ni Mac, ni compte développeur payant.

Ce dépôt ne contient que la recette (`.github/workflows/build-ios.yml`) : le code est celui d'Omi, à la même
version que le serveur ; aucune donnée personnelle, aucun secret. Le serveur n'est joignable que par les appareils de
son propriétaire sur Tailscale.

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
