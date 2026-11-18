# TP — Mise en place du protocole SSH

> **Trame à compléter** : les commandes ci-dessous sont des exemples à adapter à ton environnement. Les résultats et captures doivent provenir de tes propres manipulations. Le sujet ne précise pas qu'un rapport, un format Markdown ou un dépôt GitHub soit obligatoire.

**Membre 1 :** Ali Bechir Mahamat Khalifa  
**Membre 2 :** Aboubacar Ousman Ali Haggar  
**Classe :** [à compléter]  
**Date :** [à compléter]

## Objectif et environnement

Mettre en place un accès distant SSH, tester l'authentification par clé et utiliser des tunnels pour accéder à un serveur web et à une base MariaDB.

| Élément | Informations à compléter |
| --- | --- |
| Serveur : système et version | [à compléter] |
| Client : système et version | [à compléter] |
| Adresse IP du serveur | [à compléter] |
| Compte sur le serveur | [à compléter] |
| Réseau et machines physiques ou virtuelles | [à compléter] |

Dans les exemples ci-dessous, remplacer `UTILISATEUR` et `IP_SERVEUR` par les valeurs du serveur. Les commandes d'installation supposent un serveur Ubuntu ou Debian disposant des droits administrateur. Les adresses du sujet sont propres à son environnement.

## 1. Installation du serveur SSH

Sur le serveur :

```bash
sudo apt update
sudo apt install openssh-server
sudo systemctl enable --now ssh
systemctl status ssh --no-pager
sudo ss -ltnp | grep ':22'
```

Vérifier également que le pare-feu et la configuration réseau autorisent l'accès au port TCP 22 depuis le client. L'écoute sur le serveur seule ne prouve pas l'accessibilité depuis le réseau.

**Résultats observés :** [à compléter : état du service, port en écoute et éventuelle vérification du pare-feu]

**Capture :** [à insérer : état du service et écoute sur le port 22]

## 2. Connexion SSH depuis le client

Sur le client Windows (PowerShell) ou Linux :

```bash
ssh UTILISATEUR@IP_SERVEUR
```

Lors de la première connexion, comparer l'empreinte affichée à celle du serveur par un canal fiable avant de l'accepter. Sur le serveur, la commande `ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub` permet d'afficher son empreinte Ed25519, si cette clé d'hôte existe. Saisir ensuite le mot de passe du compte si demandé. Dans la session distante, `hostname` et `whoami` permettent de vérifier la machine et le compte utilisés.

**Résultats observés :** [à compléter : connexion, compte et machine obtenus]

**Capture :** [à insérer : commande et session distante, sans données sensibles]

## 3. Authentification par clé

Pour se connecter **du client vers le serveur** sans mot de passe du compte distant, la clé privée reste sur le client et la clé publique du client doit être ajoutée au fichier `~/.ssh/authorized_keys` du compte distant. Le sujet propose de générer et copier une clé dans le sens serveur vers client : cela correspond à une connexion dans l'autre sens si la machine cliente reçoit aussi un serveur SSH et si la clé est installée dans le compte cible. Le simple dépôt d'un fichier public dans `/home` ne suffit pas à autoriser une connexion.

Sur le client, créer une clé si nécessaire ; ne pas écraser une clé existante :

```bash
ssh-keygen -t ed25519
```

Sur un client Linux avec `ssh-copy-id` installé :

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub UTILISATEUR@IP_SERVEUR
ssh UTILISATEUR@IP_SERVEUR
```

Sur un client Windows, dans PowerShell, une autre possibilité est :

```powershell
Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub" | ssh UTILISATEUR@IP_SERVEUR "umask 077; mkdir -p ~/.ssh; cat >> ~/.ssh/authorized_keys; chmod 700 ~/.ssh; chmod 600 ~/.ssh/authorized_keys"
ssh UTILISATEUR@IP_SERVEUR
```

L'ajout initial demande généralement le mot de passe du compte distant. Lors du test suivant, observer si ce mot de passe est demandé. Une phrase secrète de clé peut toutefois être demandée si elle n'est pas déjà chargée dans un agent SSH. Ne jamais publier la clé privée.

**Résultats observés :** [à compléter : emplacement de la clé publique, installation et résultat du test]

**Capture :** [à insérer : test, sans exposer la clé privée]

## 4. Tunnels SSH : web et base de données

### 4.1 Accès au serveur web

Sur le serveur Ubuntu/Debian, installer un serveur web si nécessaire et vérifier son état :

```bash
sudo apt install apache2
sudo systemctl enable --now apache2
systemctl status apache2 --no-pager
```

Sur le client, ouvrir le tunnel et laisser ce terminal ouvert :

```bash
ssh -N -L 127.0.0.1:1024:127.0.0.1:80 UTILISATEUR@IP_SERVEUR
```

Dans un navigateur sur le client, ouvrir `http://localhost:1024`. Le premier `127.0.0.1` limite l'écoute au client local ; le second désigne le serveur vu depuis la session SSH. Arrêter le tunnel avec `Ctrl+C` après le test. Si le port 1024 est occupé ou interdit, choisir un autre port local et le signaler dans les résultats.

**Résultats observés :** [à compléter : page obtenue et port utilisé]

**Capture :** [à insérer : tunnel et page affichée]

### 4.2 Accès à MariaDB

Sur le serveur Ubuntu/Debian :

```bash
sudo apt install mariadb-server
sudo systemctl enable --now mariadb
sudo mariadb
```

Dans le client SQL administrateur, créer une base et un compte de test avec un mot de passe personnel, puis quitter :

```sql
CREATE DATABASE tp_ssh;
CREATE USER 'tpuser'@'localhost' IDENTIFIED BY 'MOT_DE_PASSE_A_REMPLACER';
GRANT ALL PRIVILEGES ON tp_ssh.* TO 'tpuser'@'localhost';
EXIT;
```

Ne pas inclure de vrai mot de passe dans le rapport. Si la base ou le compte existe déjà, adapter les noms ou utiliser un compte prévu pour le TP.

Sur le client, laisser ce tunnel ouvert dans un terminal :

```bash
ssh -N -L 127.0.0.1:3307:127.0.0.1:3306 UTILISATEUR@IP_SERVEUR
```

Dans un second terminal **sur le client**, avec un client MariaDB ou MySQL installé :

```bash
mysql -h 127.0.0.1 -P 3307 -u tpuser -p tp_ssh
```

Le mot de passe est demandé à l'invite. Une fois connecté, exécuter `SELECT DATABASE();` pour vérifier l'accès à la base, puis `exit` pour quitter. Le port 3307 est local au client ; le test applicatif confirme plus que la seule ouverture du port. Selon la configuration du serveur MariaDB, vérifier qu'il écoute sur `127.0.0.1:3306` et que le compte est autorisé à s'y connecter.

**Résultats observés :** [à compléter : connexion SQL et résultat de la requête]

**Capture :** [à insérer : tunnel et requête SQL, sans mot de passe]

## Conclusion

[À rédiger après les essais : opérations réellement réussies, difficultés rencontrées, solutions et limites éventuelles.]
