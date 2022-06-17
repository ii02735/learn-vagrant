Il existe différentes manières de travailler sur des environnements virtualisés, comme les containers ou les machines virtuelles.
Pour les machines virtuelles notamment, cela peut être un peu plus complexe à manipuler, car elles n'ont pas toutes le même hyperviseur par exemple (Virtualbox, VMWare, KVM, Microsoft Virtual PC...).

Vagrant est justement un logiciel qui répond à cette problématique : il permet **d'uniformiser les manipulations**.

Le principal avantage de Vagrant est de pouvoir faciliter la création **d'environnements isolés**.

## Exemples de cas d'usage de Vagrant

- Installer un environnement virtualisé avec de vieux logiciels : une VM a une version **bloquée par le temps**, ces logiciels peuvent continuer à fonctionner dans le temps

- Créer un environnement pour un usage spécifique

- Pré-charger un environnement virtualisé, et qui est prêt à utilisation 

## Les composants de Vagrant

Vagrant possède les 3 composants suivants :

1. Un **CLI**, pour démarrer des VM, créer des environnements, et gérer des VMs déjà exécutées
2. Les **VagrantFiles**, qui sont des fichiers écrits en Ruby permettant de personnaliser des VMs, et qui sont ensuite exécutés par le CLI.
3. **VagrantCloud**, qui est une plateforme où peut récupérer des VMs par une communauté, pour ensuite les exécuter (similaire à Docker Hub pour Docker)

## Les fonctionnalités de Vagrant

- **Se connecter à une VM**, à l'aide d'une connexion en SSH
- **La synchronisation des dossiers** entre l'hôte et la VM
- **Des configurations réseaux** : Vagrant supporte de diverses configurations de réseau (pour faire de l'inter-communication, du bridge, etc.)
- **Les fournisseurs / *providers***, qui sont des plugins permettant d'utiliser des configurations spécifiques, relatives à un hyperviseur
- **Les approvisionenurs / *provisioners***, qui sont des utilitaires qui peuvent alimenter une VM durant son exécution (installation d'une dépendance par exemple)
- **Les VagrantFiles multi-machines** : un VagrantFile ne configure qu'une seule VM, mais peut gérer différentes instances de machine pour des cas particuliers (environnements...)

## Notions

### Box

Le terme "Box" désigne une **machine virtuelle fonctionnant sous Vagrant**.
Il d'un ensemble de fichiers compressés nécessaires à l'exécution d'une VM.

- Pour **intialiser** une Box : 
```sh
vagrant init <constructeur>/<système>
```

Cela va créer un VagrantFile dans le dossier où l'on se trouve, avec le système associé.
Ce dossier sera donc **un environnement Vagrant**.

Exemple pour initialiser une Box sous Ubuntu 16.04: 
```sh
vagrant init bento/ubuntu-16.04
```

- Pour **lancer** la Box :
```sh
vagrant up
```
Cette commande va se positionner sur le VagrantFile qui se trouve dans le dossier (similaire à `docker-compose` pour Docker).

Elle va aussi alimenter un dossier `.vagrant.d`, sur le répertoire utilisateur, qui va contenir un cache pour la VM.

- Pour **obtenir le statut** de la Box :
```sh
vagrant status
```

- Pour **arrêter** la Box :
```sh
vagrant halt
```

- Pour **redémarrer** la Box :
```sh
vagrant reload
```

- Pour **supprimer** la Box :
```sh
vagrant destroy
```

Si cette commande est exécutée, tous les changements, et ressources liées à la Box seront **supprimées.**
Lancer la Box aura pour effet de retélécharger les ressources, et lancer une Box neuve.

Les fichiers dans le dossier d'environnement de Vagrant sont préservés.

- Pour **obtenir le statut** de **toutes** les Boxes de l'hôte :
```sh
vagrant global-status
```

Via cette commande, il est possible de récupérer **l'id** de la box. On peut l'utiliser pour pouvoir manipuler la box en **dehors de son fichier d'environnement** : 

```sh
vagrant up <id>
```

```sh
vagrant halt <id>
```

- Pour **se connecter** à la Box, Vagrant passe par SSH.
Cela est important si la VM ne possède pas de GUI initialement.
```sh
vagrant ssh
```
ou
```sh
vagrant ssh <id>
```
Par défaut, le nom d'utilisateur et le mot de passe sont "vagrant".


### VagrantFile

- Sur un `vagrant init`, le VagrantFile est associé à la box "base" :
```ruby
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "base"
```

Il ne **s'agit pas d'une Box valide.** Il faut en récupérer une depuis Vagrant Cloud.

- Par défaut, à chaque démarrage de la Box, le VagrantFile va chercher des mises à jour à appliquer pour la Box :
```ruby
  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false  <---
```
Cette configuration est par défaut activée. HashiCorp **ne recommande pas sa désactivation**.

- Il est possible personnaliser la configuration réseau de la box.
  - Pour exposer un port à l'hôte :
```ruby
# ici on souhaite exposer le port 80 de la VM au port 8080 de l'hôte
config.vm.network "forwarded_port", guest: 80, host: 8080
```
D'après les commentaires de VagrantFile, le port mappé **est utilisé par l'adresse de la machine**.
Il n'est pas isolé à la machine.

  - On peut demander à ce que le port soit mappé à une adresse spécifique de l'hôte :
  ```ruby
  config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
  ```
  Ici on indique que le port 8080 est celui de 127.0.0.1 (=localhost) : donc ce port ne sera accessible qu'à l'hôte.

  - Associer une **adresse réseau** à la VM :
  ```ruby
  config.vm.network "private_network", ip: "192.168.33.10"
  ```
  La Box sera donc accessible à l'adresse IP "192.168.33.10" sur le réseau.

  - Il est possible de demander à ce que la Box soit accessible en mode **bridge** : 
  ```ruby
  config.vm.network "public_network"
  ```

- On peut demander à la Box de spécifier des dossiers de l'hôte qui doivent être **montés** à l'intérieur (comme un bind sous docker) :
```ruby
config.vm.synced_folder "../data", "/vagrant_data"
```
Ici on dit que le dossier `.../data` de l'hôte, va correspondre au dossier `/vagrant_data` de la box.

**Remarque :** par défaut, Vagrant monte le dossier de l'environnement, dans le chemin `/vagrant` de la box.

- On peut spécifier une configuration **spécique à l'hyperviseur** :
```ruby
  config.vm.provider "virtualbox" do |vb|
     # Display the VirtualBox GUI when booting the machine
     vb.gui = true
  
     # Customize the amount of memory on the VM:
     vb.memory = "1024"
  end
```
On demande à ce que Virtualbox affiche le GUI au démarrage de la machine + que la RAM allouée est de 1024 Mo.

- Approvisionner / utiliser un *provisioner* sur la Box :
```ruby
  config.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y apache2
  SHELL
```
On demande ici, d'installer serveur Apache2, à l'exécution de la Box. Comme il s'agit de l'exécution d'une commande shell, **on indique qu'on souhaite exécuter un provisioner `shell`**.

Il est possible d'externaliser le contenu via un script à l'extérieur du VagrantFile.

Il est possible **d'exécuter le provisioner à chaque démarrage** :

```ruby
  config.vm.provision "shell", :run => 'always',inline: <<-SHELL
      apt-get update
      apt-get install -y apache2
  SHELL
```

Il existe d'autres provisioners :

- `file` pour indiquer qu'on souhaite transmettre un fichier à la Box à l'exécution

- `docker` pour demander d'exécuter des tâches relatives à Docker dans la Box, comme pour construire une image :
  

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  config.vm.provision "docker" do |d|
    d.build_image "/vagrant/app" # chemin du Dockerfile
  end
end
```

Ou lancer une image / créer un conteneur Docker :

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  config.vm.provision "docker" do |d|
    d.run "rabbitmq" # exécution d'un conteneur basé sur l'image rabbitmq
  end
end
```