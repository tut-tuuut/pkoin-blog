---
title: créer une base box Debian sous Virtualbox pour Vagrant
date: 2014-01-14 20:45 UTC
tags:
---

Une liste impressionante de base box existent déjà sur [www.vagrantbox.es](http://www.vagrantbox.es/). Toutefois, je n'ai pas trouvé celle qui me convient.

J'ai donc décidé de créer la mienne et pour cela la documentation de Vagrant explique parfaitement [comment créer une base box](http://docs.vagrantup.com/v2/boxes/base.html).

<i class="fa fa-cogs"></i> Mon installation :

* Mac OSX 10.9.1
* Virtualbox 4.3.6
* Vagrant 1.3.5

Avant toute chose, téléchargeons l'image iso d'une [Debian 6 (Squeeze) (AMD64)](http://gemmei.acc.umu.se/cdimage/archive/6.0.8/amd64/iso-cd/debian-6.0.8-amd64-netinst.iso).

Ensuite, grâce à Virtualbox, créons la machine virtuelle qui servira à créer notre base box :

* Nom : my-first-base-box
* Mémoire : 512 MB
* Espace disque : 50 GB (dynamiquement alloué)
* Type : VMDK

<i class="fa fa-lightbulb-o"></i> La mémoire pourra être redéfinie dans votre `Vagrantfile` mais attribuez lui une valeur par défaut raisonnable. Concernant l'espace disque, définissez une place assez importante pour que l'utilisateur de la base box ne soit pas bridé. Le fait de l'allouer dynamiquement permettra de ne consommer réellement que ce qui est utilisé.

Démarrons notre machine virtuelle, sélectionnons l'iso précédemment téléchargée et installons Debian :

* Hostname : vagrantbox
* Domain : vagrant.com
* Mot de passe root : vagrant
* Nouvel utilisateur :
  * Nom complet : vagrant
  * Identifiant : vagrant
  * Mot de passe : vagrant
* Partionnement :
  * Utilisation du disque complet
  * Tout dans une seule partition
* Logiciels :
  * Utilitaires standard du système

<i class="fa fa-lightbulb-o"></i> Selon la documentation, le mot de passe root, le nouvel utilisateur ainsi que son mot de passe doivent être égal à "vagrant" obligatoirement.

Nous devons maintenant préparer l'environnement pour Vagrant.

Première étape, notre utilisateur `vagrant` doit être un sudoer sans mot de passe,

```sh
$ apt-get install sudo
$ visudo
# Remplacer le contenu par :
# Defaults    env_keep="SSH_AUTH_SOCK"
# %admin ALL=(ALL) ALL
# %admin ALL=NOPASSWD: ALL
$ groupadd admin
$ usermod -a -G admin vagrant
```

Nous devons ensuite mettre en place les ajouts spécifiques à Virtualbox dans la machine virtuelle,

```sh
$ apt-get autoremove virtualbox-ose-guest-dkms virtualbox-ose-guest-utils virtualbox-ose-guest-x11
$ apt-get install linux-headers-$(uname -r) build-essential
$ wget -cq http://dlc.sun.com.edgesuite.net/virtualbox/4.3.6/VBoxGuestAdditions_4.3.6.iso
$ mount -o loop,ro /home/vagrant/VBoxGuestAdditions_4.3.6.iso /mnt
$ /mnt/VBoxLinuxAdditions.run --nox11
$ umount /mnt
```

<i class="fa fa-lightbulb-o"></i> Ces ajouts servent à optimiser les performances, la stabilité du système, monter des dossiers partagés...

Un serveur SSH doit également être configuré pour que Vagrant puisse se connecter via la commande `vagrant ssh`,

```sh
$ apt-get install openssh-server openssh-client
$  echo 'UseDNS no' >> /etc/ssh/sshd_config
# On désactive les DNS au niveau de la configuration du
# serveur SSH améliorer la rapidité de connexion
$ su vagrant
$ cd ~
$ mkdir .ssh
$ wget --no-check-certificate -O .ssh/authorized_keys https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub
$ chmod 700 .ssh
$ chmod 600 .ssh/authorized_keys
```

Notre machine virtuelle est prête. Il ne nous reste plus qu'à faire du ménage et à l'éteindre,

```sh
$ sudo apt-get clean
$ sudo poweroff
```

Créons la base box,

```sh
$ vagrant package --base my-first-base-box
[my-first-base-box] Clearing any previously set forwarded ports...
[my-first-base-box] Creating temporary directory for export...
[my-first-base-box] Exporting VM...
[Debian] Compressing package to: package.box
$ mv package.box my-first-base-box.box
```

Testons maintenant cette fameuse base box,

```sh
$ vagrant box add my-first-base-box my-first-base-box.box
Downloading or copying the box...
Extracting box...te: 387M/s, Estimated time remaining: --:--:--)
Successfully added box 'my-first-base-box' with provider 'virtualbox'!
$ mkdir test
$ cd test
$ vagrant init my-first-base-box
$ vagrant up
# Bringing machine 'default' up with 'virtualbox' provider...
# [default] Importing base box 'my-first-base-box'...
# [default] Matching MAC address for NAT networking...
# [default] Clearing any previously set forwarded ports...
# [default] Creating shared folders metadata...
# [default] Clearing any previously set network interfaces...
# [default] Preparing network interfaces based on configuration...
# [default] Forwarding ports...
# [default] -- 22 => 2222 (adapter 1)
# [default] Booting VM...
# [default] Waiting for machine to boot. This may take a few minutes...
# [default] Machine booted and ready!
# [default] Mounting shared folders...
# [default] -- /vagrant
```

Victory! \o/
