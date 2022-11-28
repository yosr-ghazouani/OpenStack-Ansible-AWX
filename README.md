# Introduction to automation with Ansible AWX, GitHub and Openstack
# OpenStack

OpenStack est une plateforme open source, libre qui permet de créer et de gérer des Clouds privés ou publics à partir des pools de ressources virtuelles. Les projets qui constituent la plateforme OpenStack assurent les principaux services du Cloud Computing : le calcul, la mise en réseau, le stockage et la gestion des identités et des images.

## Installateurs d’OpenStack
Les méthodes d’installation d’OpenStack relèvent généralement de deux grandes 
catégories : le test d’un côté et le passage en production de l’autre. 

### Devstack

Devstack est un projet d’installation d’OpenStack à base de scripts bash maintenu par une communauté de développeurs. Il permet la construction des environnements complets destinés au développement et aux tests. Devstack peut être installé en machine virtuelle ainsi que sur une machine physique. Il supporte la distribution Ubuntu en versions courantes.

#### Exigences minimales
Pour installer OpenStack en utilisant l’installateur Devstack, nous avons besoin d’un système Ubuntu 18.04, 12 Go de RAM, 6 VCPUs et un espace de stockage de 20 Go.

#### Mise à jour du système

Nous avons mis à niveau les référentiels systèmes à l'aide de la commande suivante :
<br> 
  `$ sudo apt update -y && apt upgrade -y`

#### Création d’un utilisateur pile (Stack) et attribution des privilèges

Les meilleures pratiques imposent l’exécution de Devstack en tant qu’utilisateur régulier avec les droits sudo. Dans cet esprit, nous avons créé un utilisateur appelé « stack » en exécutant la commande ci-dessous :
<br> 
  `$ sudo adduser -s /bin/bash -d /opt/stack -m stack`


Ensuite, nous avons attribué les privilèges sudo à l’utilisateur « stack » grâce à cette commande :
<br> 
  `$ sudo echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack`

#### Clonage du dépôt git de Devstack
Une fois que nous avons créé avec succès l’utilisateur, nous avons cloné le dépôt git de Devstack comme indiqué :
<br>
  `$ sudo echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack`

#### Création d’un fichier de configuration Devstack
Lors de cette étape, nous avons accédé au répertoire « devstack » et créé un fichier de configuration appelé « local.conf ». Ce dernier contient les informations indiquées dans la figure ci-dessous. 
<br>

![Devstack](/images/devstack/1.PNG)

#### Exécution du script « stack » 
Pour commencer le déploiement d’OpenStack sur Ubuntu 18.04, nous avons exécuté le 
script « stack » dans le répertoire « devstack », en utilisant la commande suivante :
<br>
  `$ ./stack.sh`

Ce script permet l’installation de ces services : <br>
- Tableau de bord OpenStack - horizon <br>
- Service de calcul - nova <br>
- Service d'image - glance <br>
- Service réseau - neutron <br>
- Service d'identité - neystone <br>
- Placement - API de placement <br>

La sortie de la commande ci-dessus est la suivante :
<br>
![devstack](/images/devstack/2.PNG)
<br>
Cela confirme que tout s'est bien configuré et que nous pouvons accéder au tableau de bord horizon via l’URL : http://172.25.52.123/dashboard.

### Packstack

Packstack utilise les modules puppet pour déployer OpenStack. Il définit l’environnement avec la minimale configuration qui est préférable pour une activité pratique pas pour un environnement de production.
L’OpenStack déployé à partir de Packstack n’est pas assez stable pour gérer les exigences du centre de données. Packstack soutient Centos et Red Hat Enterprise Linux (RHEL).

#### Prérequis
 Pour cette installation, nous utilisons une machine virtuelle ESXI 7.0 dont ses 
caractéristiques sont : <br>
- 16 Go de RAM <br>
- 60 Go de disque dur<br> 
- 6 VCPUs.


#### Configuration du réseau
D’abord, nous avons désactivé le service FirewallD pour démarrer automatiquement au démarrage du système :
<br>
  `$ sudo systemctl disable firewalld` 

Suite à l’exécution de la commande ci-dessus, nous avons obtenu la sortie suivante :
<br>
![packstack](/images/packstack/1.PNG)
<br>

Ensuite, nous avons arrêté le service FirewallD avec :
<br>
  `$ sudo systemctl stop firewalld `

Puis, nous avons désactivé le gestionnaire de réseau (définitivement) pour démarrer automatiquement lors du démarrage du système en utilisant la commande suivante : 
<br>
  `$ sudo systemctl disable NetworkManager`

La sortie de cette commande est : 
<br>
![packstack](/images/packstack/2.PNG)
<br>

Ensuite, nous avons arrêté le service de gestionnaire de réseau avec :
<br>
  `$ sudo systemctl stop NetworkManager`
<br>
Lors du redémarrage du système, nous avons activé le service gestionnaire de réseau avec :
<br>
  `$ sudo systemctl enable NetworkManager`

Enfin, nous avons démarré le serveur réseau avec :
<br>
  `$ sudo systemctl start NetworkManager`

#### Mise à jour du système
Dès que nous avons redémarré notre système et rétabli la connexion Internet, nous avons exécuté la commande suivante :
<br>
  `$ sudo yum update -y`
#### Installation des référentiels de logiciels 
Nous avons installé le package Rocky pour activer le référentiel OpenStack avec la commande ci-dessous.
<br>
  `$ sudo yum install -y centos-release-openstack-rocky`

#### Installation du programme Packstack
Une fois l’installation du package rocky est terminée, nous avons installé Packstack avec :
<br>
  `$  sudo yum install -y openstack-packstack`
#### Exécution du Packstack
Pour déployer OpenStack avec un seul nœud, nous avons exécuté la commande suivante : 
<br>
  `$ sudo packstack --allinone`

Nos paramètres de connexion sont stockés dans le fichier « keystonerc_admin » comme le montre la figure ci-dessous.
<br>
![packstack](/images/packstack/3.PNG)
<br>

Une fois le processus d'installation est terminé, nous pouvons accéder
au tableau de bord OpenStack en accédant à « http://172.25.52.127/dasboard » .

# Ansible
Ansible est un moteur d'automatisation informatique Open Source qui automatise 
l'approvisionnement, la gestion des configurations, le déploiement des applications, l'orchestration et bien d'autres processus informatiques.
Ansible repose sur le protocole d’authentification réseau SSH et l’architecture Push pour pousser les configurations décrites dans les playbooks vers les clients.

###  Architecture d’Ansible 
L’architecture d’Ansible se compose :
<br>
-Des hôtes (des nodes) auxquelles Ansible se connecte et pousse les tâches 
d’automatisation via le protocole SSH. <br>
- Du nœud de contrôle sur lequel l’outil Ansible est installé.  <br>
- De l’inventaire qui présente une liste de toutes les adresses IP des hôtes. <br>
- Des playbooks qui définissent les tâches à accomplir sur les ressources que nous souhaitons les automatiser. Chaque tâche a un nom et appelle des modules. <br> 
- Des plugins qui sont des types spéciaux de modules. Nous les exécutons avant
l’exécution des modules sur les hôtes.<br>
![ansible](/images/ansible/1.PNG)
<br>

Toutefois, Ansible présente quelques lacunes. Il ne permet pas ni la délégation de l’exécution de playbook à des équipes tierces ni la définition des extras variables. De plus, il ne crée pas des pipelines de playbooks Ansible permettant d’éviter le gaspillage du temps.
Pour remédier à ces faiblesses, nous avons utilisé l’outil d’orchestration des playbooks Ansible : AWX.

AWX est la version open source derrière Red Hat Ansible Tower. Il est conçu pour 
fonctionner au-dessus du projet Ansible, améliorant ainsi le moteur d'automatisation.

#### Implémentation d’AWX 
Nous avons installé AWX sur le nœud de contrôle Ansible.
Tout d’abord, nous avons mis à jour le système Ubuntu en utilisant la commande suivante :
<br>
  `$ sudo apt update && sudo apt –y upgrade 
   $ sudo reboot`
Ensuite, les services Ansible AWX s’exécutent en tant que conteneurs Docker. C’est pour cela que nous avons besoin de Docker Engine pour alimenter l’environnement. La commande ci-dessous installe les dépendances de base permettant la configuration des packages Docker.
<br>
  `$ sudo apt -y install apt-transport-https ca-certificates curl software-properties-common`

Puis, nous avons ajouté la clé GPG du référentiel Docker à notre système et aux sources APT en tapant les commandes suivantes : 
<br>
  `$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add –
 $ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu 
 $(lsb_release –cs) stable" `

Par la suite, nous avons installé Docker CE grâce à cette commande :
<br>
  `$ sudo apt install docker-ce docker-ce-cli containerd.io`
<br>
  `$ sudo usermod -aG docker ${USER}`

De plus, pour que nous évitions de taper « sudo » chaque fois que nous exécutons une 
commande Docker, nous avons ajouté notre utilisateur au groupe Docker :
Une fois que nous avons vérifié l’installation de Docker Engine, nous avons installé Node.js, NPM, le module python docker-py et docker-compose en exécutant les commandes ci-dessous :
<br>
  `$ sudo apt install -y nodejs npm
 $ sudo npm install npm --global
 $ sudo apt install python-pip git pwgen vim
 $ sudo pip install requests==2.14.2
 $ sudo pip3 installer docker-compose == 1.29.1`

Maintenant, nous pouvons installer AWX en clonant le code source à partir de GitHub à l’aide de la commande « git ».
<br>
  `$ sudo git clone –b 17.1.0 https://github.com/ansible/awx.git`
Après, nous avons accédé au répertoire du programme d’installation AWX et nous avons modifié le fichier inventaire en indiquant les paramètres d’accès à l’interface graphique et la clé secrète générée par la commande « pwgen ». 
<br>
![ansible](/images/ansible/2.PNG)
<br>
Enfin, nous avons exécuté le palybook « install.yml » qui permet d’installer AWX et nous avons listé les conteneurs en cours d’exécution en utilisant la commande « docker ps ».
<br>
![ansible](/images/ansible/3.PNG)
<br>
Nous pouvons alors accéder au tableau de bord via l’adresse IP suivante : 
- Http:// 172.25.52.134/#/awx
Le tableau de bord présente des informations sur notre serveur AWX qui sont :
- Les hôtes qui ont exécuté avec succès les playbooks
- Les hôtes qui n'ont pas réussi à exécuter les playbooks
- Les inventaires
- Les projets et état de la synchronisation



#  Ajout des enquêtes d’AWX
Dans AWX, une enquête est un ensemble de questions que vous pouvez configurer pour poser avant d'exécuter le script et stocker ses réponses sous forme de variables. Les étapes de configuration peuvent être résumées comme suit :
#### Ajout d’une enquête pour le modèle « utilisateur »
Une enquête peut comprendre n'importe quel nombre de questions. L’enquête spécifiée au modèle « utilisateur » se compose de quatre questions. La figure ci-dessus représente la première question permettant la saisie du nom 
d’utilisateur.<br>
![ansible awx](/images/awx/1.PNG)
<br>
La deuxième question qui demande la saisie de nom du projet, est présentée dans la figure cidessous.
<br>
![ansible](/images/awx/2.PNG)
<br>
La troisième question permet de saisir le mot de passe de l’utilisateur OpenStack comme indiqué dans cette figure.
<br>
![ansible awx](/images/awx/3.PNG)
<br>
Le type de la réponse est password. Elle sera cryptée et traitée comme des informations sensibles.
<br>
La dernière question relève de l’email de l’utilisateur comme le montre la figure ci-dessous.
<br>
![ansible awx](/images/awx/4.PNG)
<br>
La réponse de l’utilisateur à la question ci-dessus sera stockée dans la variable « email ». 
Il s'agit de la variable à utiliser par le playbook « utilisateur » qui est hébergé dans le référentiel git : https://github.com/yosr-ghazouani/ansible-openstack-awx.
#### Ajout d’une enquête pour le modèle « quotas »
Cette enquête se compose de quatre questions qui permettent de spécifier les 
caractéristiques de l’environnement Cloud y compris RAM, VCPUs et l’espace de stockage comme l’indiquent la figure ci-dessous.
<br>
![ansible awx](/images/awx/5.PNG)





