# TP SSH — 

Ce dossier accompagne le TP de mise en place de SSH : installation du serveur, connexion distante, authentification par clé et tunnels vers un serveur web et une base MariaDB.

**Membres du groupe :** Ali Bechir Mahamat Khalifa et Aboubacar Ousman Ali Haggar.

## Compte rendu

- [Rapport du groupe au format PDF](Rapport-SSH.pdf) : environnement, manipulations et captures d'écran.
- [Trame complémentaire au format Markdown](RAPPORT-SSH.md) : commandes explicatives et emplacements pour d'autres observations.

Les commandes de la trame sont des exemples à adapter. Les résultats et captures du TP figurent dans le rapport PDF.

## Précisions sur les essais du rapport

- Le tunnel web du rapport utilise le port local **1025**, à la place du port **1024** illustré dans le sujet. L'adresse du navigateur et le port local du tunnel doivent correspondre.
- Pour MariaDB, le test TCP seul vérifie qu'un port local répond ; la connexion SQL et la requête montrées ensuite constituent un test applicatif plus complet.
- Le rapport ne montre pas la configuration d'écoute d'Apache. On ne peut donc pas déduire de ses captures que ce serveur web est limité à `127.0.0.1`.
