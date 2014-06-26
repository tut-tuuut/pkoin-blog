---
title: arrêtez d'utiliser la même clé ssh privée partout !
date: 2014-06-26 10:43 UTC
tags: ssh, sécurité
---

J'ai trop souvent vu des développeurs utiliser la même clé SSH pour se connecter à des services totalement différents.READMORE Je suis loin d'être un expert dans le domaine mais utiliser des clés différentes n'est vraiment pas compliqué.

## Pourquoi ne doit-on pas utiliser le même couple de clé SSH publique/privée de partout ?

Si vous utilisez une seule et même clé pour vous connecter à différents services (github, ovh, heroku…), le jour où quelqu'un dérobe votre clé privée, cette personne aura accès à tous les services en question. C'est plutôt gênant.

![Hackers everywhere!](images/meme-cle-ssh-privee/hackers-hackers-everywhere.jpg)

## Quelques recommandations

Afin de garantir un minimum de sécurité,

* Une clé est privée à un utilisateur et à une machine
  * Chacun à sa clé, les clés ne sont pas partagées entre plusieurs utilisateurs
  * Chaque machine d'un même utilisateur a sa propre clé
  * Ne pas stocker sa clé privée sur des machines intermédiaires
* Toutes les clés sont protégées par une passphrase (même si c'est chiant à taper !)
  * Les passphrases sont suffisamment complexes et différentes

## Comment faire ?

Dans l'exemple qui suit, on va générer une paire de clé qui sera utilisé pour github.com uniquement.

### Générer une paire de clé

```sh
$ ssh-keygen -t rsa -f ~/.ssh/id_rsa,github.com -C "mon_compte_github,mon_mba_2014"
```

Note : Il est possible de changer la passphrase ultérieurement,

```sh
$ ssh-keygen -f ~/.ssh/id_rsa,github.com -p
```

Il faut ensuite impérativement changer les droits de la clé privée pour qu'elle soit uniquement accessible par vous.

```sh
$ chmod 600 ~/.ssh/id_rsa,github.com
```

N'oubliez pas d'associer à votre compte github la clé publique générée. [Github explique ça bien mieux que moi](https://help.github.com/articles/generating-ssh-keys#step-3-add-your-ssh-key-to-github) !

### Configurer le client SSH

Il faut ensuite éditer le fichier `~/.ssh/config` pour y ajouter,

```sh
Host github.com
  Hostname github.com
  User mon_compte_github
  IdentityFile ~/.ssh/id_rsa,github.com
```

### Test !

```sh
$ ssh -vT git@github.com
```

Cette commande va tenter de se connecter en SSH au serveur de github.  
La sortie est assez verbeuse mais, normalement, vous devriez voir que votre client SSH utilise bien votre clé privée `id_rsa,github.com` pour vous authentifier auprès du serveur de github,

```
debug1: Reading configuration data /Users/pkoin/.ssh/config
debug1: /Users/pkoin/.ssh/config line 5: Applying options for github.com
debug1: Reading configuration data /etc/ssh_config
debug1: /etc/ssh_config line 20: Applying options for *
debug1: Connecting to github.com [192.30.252.128] port 22.
debug1: Connection established.
# C'est la ligne suivante qui nous intéresse plus particulièrement
debug1: identity file /Users/pkoin/.ssh/id_rsa,github.com type 1
```

Victory \o/

