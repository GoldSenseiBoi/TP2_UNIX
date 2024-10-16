# TP 02 : Services, Processus, Signaux

---

### 1. Secure Shell : SSH

#### 1.1 Connexion SSH root et configuration

Pour permettre la connexion SSH en tant que root, il faut modifier le fichier de configuration SSH situé dans /etc/ssh/sshd_config. La ligne à modifier est PermitRootLogin. Par défaut, elle est souvent sur no pour des raisons de sécurité. Pour autoriser la connexion root, on peut la changer en yes.

- **Commandes** :
  ```bash
  sudo apt update
  sudo apt install ssh
  sudo nano /etc/ssh/sshd_config
  ```
  - Modifier la ligne suivante :
    ```
    PermitRootLogin yes
    ```
  - Redémarrer le service SSH :
    ```bash
    sudo systemctl restart ssh
    ```

Avantage de PermitRootLogin no : Plus sécurisé, car cela oblige les administrateurs à utiliser un utilisateur non-root et des clés SSH pour se connecter, ce qui diminue les risques d'attaques directes sur le compte root.

Inconvénient de PermitRootLogin yes : Moins sécurisé, car les attaquants peuvent tenter des attaques par force brute sur le mot de passe root. C’est déconseillé en production.

#### 1.2 Génération de clés SSH

L’authentification par clés SSH est plus sécurisée que l’utilisation d’un mot de passe, car elle repose sur un couple de clés privée/publique. La clé privée reste sur l’ordinateur de l’utilisateur, tandis que la clé publique est copiée sur le serveur. Cela permet d’éviter les attaques par force brute.
- **Générer une clé publique/privée** :
  ```bash
  ssh-keygen -t rsa -b 2048
  ```
- **Copier la clé publique sur le serveur** :
  ```bash
  ssh-copy-id root@10.0.2.15
  ```
 Pourquoi ne pas utiliser de passphrase dans un cas réel ?

   Réponse : Ne pas utiliser de passphrase sur une clé privée signifie que toute personne ayant accès à la clé privée peut se connecter au serveur sans aucune autre vérification. Cela est dangereux en production, car si la clé privée est volée, l’attaquant pourra accéder au serveur sans restriction. Il est donc recommandé de toujours utiliser une passphrase dans un environnement de production pour ajouter une couche de sécurité supplémentaire.

#### 1.3 Authentification par clé sur le serveur

Question : Comment utiliser une clé publique pour se connecter à un serveur Linux via SSH ?

   Réponse : Une fois la clé publique copiée sur le serveur dans le fichier authorized_keys, on peut se connecter au serveur sans avoir à fournir de mot de passe.

- **Créer le dossier `.ssh` et ajouter la clé publique** :
  ```bash
  mkdir -p /root/.ssh
  chmod 700 /root/.ssh
  nano /root/.ssh/authorized_keys
  chmod 600 /root/.ssh/authorized_keys
  ```
- **Connexion avec la clé privée** :
  ```bash
  ssh -i ~/.ssh/id_rsa root@10.0.2.15
  ```

#### 1.4 Sécurisation SSH

Question : Comment sécuriser l’accès SSH en autorisant uniquement les connexions par clé ?

   Réponse : Pour éviter les attaques par force brute, il est recommandé de désactiver les connexions SSH avec mot de passe et de n'autoriser que les connexions par clé. Cela se fait en modifiant la configuration SSH pour désactiver les mots de passe et activer l'authentification par clé uniquement.

- **Modifier la configuration SSH** :
  ```bash
  sudo nano /etc/ssh/sshd_config
  ```
  - Ajouter/modifier les lignes :
    ```
    PermitRootLogin prohibit-password 
    PasswordAuthentication no 
    ```

 Explication des attaques de type brute-force SSH :

 Les attaques par force brute consistent à essayer automatiquement des milliers, voire des millions de combinaisons de mots de passe pour essayer de pénétrer dans un système. En désactivant les connexions par mot de passe et en n'autorisant que les clés SSH, on empêche ces attaques de réussir, car il devient pratiquement impossible de deviner une clé SSH.

---

### 2. Processus

#### 2.1 Étude des processus UNIX

Question : Comment afficher la liste de tous les processus tournant sur la machine avec des informations spécifiques ?

   Réponse : Pour afficher la liste des processus avec des informations comme l'utilisateur, le PID, l'utilisation du CPU et de la mémoire, on peut utiliser la commande ps avec l’option aux.

- **Afficher la liste des processus** :
  ```bash
  ps aux
  ```
  
- **Utiliser `top` pour surveiller en temps réel** :
  ```bash
  top
  ```
 TIME : Temps total d'utilisation du CPU par le processus.

Premier processus lancé : Le premier processus lancé après le démarrage est  systemd (PID 1).

Processus utilisant le plus de CPU : Dans cette capture, le processus kworker semble avoir consommé le plus de CPU.



- **Afficher les processus avec leur processus père (PPID)** :
  ```bash
  ps -o pid,ppid,cmd
  ```
- **Afficher un arbre des processus** :
  ```bash
  sudo apt install pstree
  pstree 
  ```

- ** tout les processus au demarrage
   ```bash
  ps aux | wc -l
  ```

- **Lancer `htop` pour une interface plus visuelle** :
  ```bash
  sudo apt install htop
  htop
  ```
  - `htop` permet de naviguer plus facilement avec les flèches et de tuer des processus directement.


Ici, le processus kworker utilise à nouveau des ressources CPU, et top est lui-même en cours d’exécution avec un PID 561.

On peut utiliser Shift + P pour trier par CPU et Shift + M pour trier par utilisation mémoire.

Trouvez les commandes interactives permettant de : passer l’affichage en couleur, mettre en
avant le colonne de trie, changer la colonne de trie : top puis ?

### 3. Exercice 2 : Arrêt d'un processus

#### 3.1 Lancer des scripts en arrière-plan

- **Script `date.sh`** :
  ```bash
  #!/bin/sh
  while true; do
    sleep 1
    echo -n 'date: '
    date +%T
  done
  ```

- **Script `date-toto.sh`** :
  ```bash
  #!/bin/sh
  while true; do
    sleep 1
    echo -n 'toto: '
    date --date '5 hour ago' +%T
  done
  ```

- **Lancer les scripts en arrière-plan** :
  ```bash
  ./date.sh &
  ./date-toto.sh &
  ```

- resultat : 
[1] 581
[2] 582


- **Ajouter les permissions d'exécution si nécessaire** :
  ```bash
  chmod +x date.sh date-toto.sh
  ```

#### 3.2 Utilisation de `jobs`, `fg`, et `kill` pour gérer les processus: 

- **Lister les tâches en arrière-plan** :
  ```bash
  jobs
  ```
- **Ramener un processus à l'avant-plan** :
  ```bash
  fg %1   # Ramène la tâche 1 à l'avant-plan
  ```
- **Arrêter un processus avec `CTRL+C`** :
  Après avoir ramené le processus à l'avant-plan avec `fg`, utiliser `CTRL+C` pour l'arrêter.
- **Arrêter un processus avec `ps` et `kill`** :
  - **Lister les processus** :
    ```bash
    ps aux | grep date.sh
    ```
  - **Arrêter le processus avec `kill`** :
    ```bash
    kill <PID>
    ```

---
### 4. Exercice 3 : Les tubes

#### 4.1 Utilisation des pipes (`|`)

Les tubes (ou pipes) permettent de rediriger la sortie d'une commande vers l'entrée d'une autre commande. Voici quelques exemples d'utilisation des pipes :

- **Afficher la liste des fichiers et rediriger vers `cat`** :
  ```bash
  ls | cat
  ```
  Cela affiche la liste des fichiers en utilisant `cat`, qui permet d'afficher le contenu d'une commande.

- **Rediriger la sortie de `ls -l` vers un fichier `liste`** :
  ```bash
  ls -l | cat > liste
  ```
  Cette commande affiche la liste détaillée des fichiers et redirige la sortie vers un fichier nommé `liste`.

- **Utiliser `tee` pour afficher et enregistrer la sortie** :
  ```bash
  ls -l | tee liste
  ```
  `tee` permet d'afficher la sortie dans le terminal tout en l'enregistrant dans un fichier.

- **Combiner `tee` avec `wc -l` pour compter les lignes** :
  ```bash
  ls -l | tee liste | wc -l
  ```
  Cette commande affiche la liste des fichiers, l'enregistre dans `liste`, et compte le nombre de lignes.

---
### 5. Journal système : rsyslog

#### 5.1 Configuration de `rsyslog`

`rsyslog` est un service qui permet de gérer les logs du système. Voici comment configurer et utiliser `rsyslog` pour collecter les journaux du système.

- **Installation de `rsyslog`** :
  Sur la plupart des distributions Linux, `rsyslog` est installé par défaut. Si ce n'est pas le cas, installez-le avec la commande suivante :
  ```bash
  sudo apt update
  sudo apt install rsyslog
  ```

- **Vérifier que `rsyslog` est actif** :
  ```bash
  sudo systemctl status rsyslog
  ```
  Cette commande vous indique si `rsyslog` est en cours d'exécution. Si ce n'est pas le cas, vous pouvez l'activer :
  ```bash
  sudo systemctl start rsyslog
  ```
 - résultat : Main PID: 993 (rsyslogd)
- **Configuration des fichiers de log** :
  Les fichiers de configuration de `rsyslog` se trouvent généralement dans le répertoire `/etc/rsyslog.d/` ou `/etc/rsyslog.conf`. Vous pouvez éditer ces fichiers pour configurer quels messages seront enregistrés et où ils seront enregistrés.
  ```bash
  sudo nano /etc/rsyslog.conf
  ```
  Par exemple, vous pouvez ajouter une ligne pour envoyer tous les messages d'erreur à un fichier personnalisé :
  ```
  *.err /var/log/errors.log
  ```

- **Redémarrer `rsyslog` après modification** :
  ```bash
  sudo systemctl restart rsyslog
  ```

- **Consulter les logs** :
  Pour visualiser les logs générés par le système et enregistrés par `rsyslog`, utilisez la commande `cat` ou `less` :
  ```bash
  cat /var/log/syslog
  ```
  Ou :
  ```bash
  less /var/log/syslog
  ```

- **Filtrer les logs** :
  Pour filtrer les logs et ne voir que certaines informations, utilisez `grep` :
  ```bash
  cat /var/log/syslog | grep ssh
  ```
  Cette commande permet de voir uniquement les logs relatifs à SSH.
- **A quoi sert le service cron ?**
  Le service `cron` est utilisé pour automatiser l'exécution de tâches répétitives à des moments spécifiés. Ces tâches sont définies dans le fichier `crontab` et peuvent être des scripts ou des commandes à exécuter périodiquement (par exemple, toutes les heures, tous les jours, etc.).
 Commande `tail -f`

- **Que fait la commande `tail -f` ?**
  La commande `tail -f` permet de suivre en temps réel l'ajout de nouvelles lignes à un fichier. Cela est particulièrement utile pour surveiller les logs du système.
  - **Utilisation** :
    ```bash
    tail -f /var/log/messages
    ```
    Cette commande place en bas de l'écran une fenêtre qui permet de visualiser en “temps réel” le contenu du fichier `/var/log/messages`.
  - **Redémarrer le service cron** :
    Depuis un autre shell, redémarrez le service cron et observez l'impact dans le fichier de log :
    ```bash
    sudo systemctl restart cron
    ```
   résultat : je vois apparaître des messages indiquant le redémarrage du service.



- **A quoi sert le fichier `/etc/logrotate.conf` ?**
  Le fichier `/etc/logrotate.conf` est utilisé pour configurer la rotation des fichiers de log. Cela permet de gérer la taille des fichiers de log en les faisant tourner régulièrement, c'est-à-dire en les archivant et en créant de nouveaux fichiers pour éviter qu'ils ne deviennent trop volumineux. Ce processus est essentiel pour maintenir la bonne performance du système.

 

- **Examen de la sortie de `dmesg`** :
  La commande `dmesg` affiche les messages du noyau, qui incluent des informations sur le matériel détecté lors du démarrage.
  - **Modèle de processeur détecté** :
    ```bash
    dmesg | grep -i "cpu"
    ```
    Cette commande permet de trouver les informations sur le modèle de processeur détecté par Linux.
donc résultat :  CPU MTRRs all blank - virtualized system.
 smpboot: Allowing 2 CPUs, 0 hotplug CPUs
  - **Modèles de cartes réseau détectées** :
    ```bash
    dmesg | grep -i "eth"
    ```
    Cette commande permet de voir les informations sur les cartes réseau détectées (resultat: `eth0`, `eth1`, etc.).

