---
title: dnsmasq pour ton environnement de développement
date: 2014-05-20 21:34 UTC
tags: dnsmasq, dns, développement, réseau
---

C'est pas le tout de monter un bel environnement de développement à grand coup de Vagrant, Docker, et VirtualBox, mais encore faut-il pouvoir y accéder.READMORE

Pour cela, il existe la solution classique de modification du fichier `/etc/hosts`. Toutefois, cela est assez pénible car il faut le modifier à chaque fois qu'on créé un nouveau projet et cela nécessite obligatoirement les droits root sur la machine. Pas cool !

Une autre approche (plus belle et plus pratique !) consiste à utiliser un serveur DNS en local. Contrairement à ce que l'on peut penser, cette approche n'est pas très compliqué et offre une plus grande souplesse de configuration.

Dans cet article, je vais utiliser [Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) sous un environnement Mac OSX (10.9.2) disposant de [Homebrew](http://brew.sh/).

![DNS Cat!](images/dnsmasq/dns-cat.jpg)

Pourquoi Dnsmasq ? Tout simplement car :

> Dnsmasq est un serveur léger pour fournir les services DNS, DHCP, Bootstrap Protocol et TFTP pour un petit réseau, voire pour un poste de travail. **[Wikipédia](http://fr.wikipedia.org/wiki/Dnsmasq)**

Tout d'abord, installons le serveur.

```sh
$ brew install dnsmasq
==> Downloading https://downloads.sf.net/project/machomebrew/Bottles/dnsmasq-2.71.mavericks.bottle.tar.gz
######################################################################## 100,0%
==> Pouring dnsmasq-2.71.mavericks.bottle.tar.gz
==> Caveats
To configure dnsmasq, copy the example configuration to /usr/local/etc/dnsmasq.conf
and edit to taste.

  cp /usr/local/opt/dnsmasq/dnsmasq.conf.example /usr/local/etc/dnsmasq.conf

To have launchd start dnsmasq at startup:
    sudo cp -fv /usr/local/opt/dnsmasq/*.plist /Library/LaunchDaemons
Then to load dnsmasq now:
    sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist
==> Summary
   /usr/local/Cellar/dnsmasq/2.71: 7 files, 488K
```

Et là, Homebrew est sympa car il nous explique comment configurer notre serveur.

On commence donc par mettre en place la configuration locale.

```sh
$ cp /usr/local/opt/dnsmasq/dnsmasq.conf.example /usr/local/etc/dnsmasq.conf
```

On fait en sorte que Dnsmasq se lance automatiquement au démarrage de la session.

```sh
$ sudo cp -fv /usr/local/opt/dnsmasq/*.plist /Library/LaunchDaemons
```

Et enfin, on démarre le serveur.

```sh
$ sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist
```

Passons maintenant au plus intéressant : la configuration.  

Prenons un exemple concret.  
Nous souhaitons que toutes les requêtes, qui se terminent par `.lxc`, retournent `127.0.0.1`.

Comme nous l'avons vu plus haut, le fichier par défaut de configuration se trouve dans `/usr/local/etc/dnsmasq.conf`. Nous allons donc l'éditer pour ajouter la ligne suivante à la fin.

```
address=/lxc/127.0.0.1
```

Nous devons recharger le serveur pour prendre en compte cette nouvelle configuration.

```sh
$ sudo launchctl stop homebrew.mxcl.dnsmasq
$ sudo launchctl start homebrew.mxcl.dnsmasq
```

Faisons un test en utilisant notre résolveur.

```sh
$ dig foobar.lxc @127.0.0.1
```

Normalement, vous devriez obtenir la réponse suivante.

```sh
;; ANSWER SECTION:
foobar.lxc.     0   IN  A   127.0.0.1
```

Tout cela est très bien mais pour l'instant notre résolveur DNS n'est pas reconnu par notre système.  
Pas de panique, c'est super simple grâce aux fichiers `resolvers`.

On commence par préparer le système si ce n'est pas déjà fait.

```sh
$ sudo mkdir -p /etc/resolver
```

Il suffit ensuite de créer un fichier pour chaque résolveur que nous souhaitons ajouter à notre système.  
**Attention**, le fichier doit porter le même nom que le domaine principal.

Dans notre exemple, nous cherchons à rediriger les requêtes se terminant par `.lxc` à notre serveur DNS local. Nous créons donc le fichier `/etc/resolver/lxc` contenant l'instruction suivante.

```
nameserver 127.0.0.1
```

Regardons si nous n'avons rien cassé.

```sh
$ ping -c1 www.google.com
PING www.google.com (173.194.40.177): 56 data bytes
64 bytes from 173.194.40.177: icmp_seq=0 ttl=54 time=39.117 ms

--- www.google.com ping statistics ---
1 packets transmitted, 1 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 39.117/39.117/39.117/0.000 ms
```

Et enfin, regardons si notre requête à `foobar.lxc` répond favorablement.

```sh
$ ping -c1 foobar.lxc
PING foobar.lxc (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.034 ms

--- foobar.lxc ping statistics ---
1 packets transmitted, 1 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.034/0.034/0.034/0.000 ms
```

Victory! \o/
