# Part IV : Gestion d'utilisateurs

- [Part IV : Gestion d'utilisateurs](#part-iv--gestion-dutilisateurs)
  - [1. Gestion d'utilisateurs](#1-gestion-dutilisateurs)
  - [2. Gestion de permissions](#2-gestion-de-permissions)
    - [3. Sudo sudo sudo](#3-sudo-sudo-sudo)

## 1. Gestion d'utilisateurs

🌞 **Gestion d'utilisateurs**

```sh
sudo useradd -m -s /bin/bash suha -p $(openssl passwd -6 -salt $(openssl rand -hex 8) $(openssl rand -hex 16)) --groups managers,admins
```

## 2. Gestion de permissions

🌞 **Gestion de permissions**

```sh
[gwuill@vbox data]$ ls -la ; getfacl ./projects/
total 0
drwxr-xr-x.  4 gwuill gwuill  34 Feb 20 13:42 .
dr-xr-xr-x. 19 gwuill gwuill 247 Feb 20 13:37 ..
drwx------+  2 gwuill gwuill  23 Feb 20 13:49 conf
drwxrwx---+  6 gwuill gwuill  89 Feb 20 15:39 projects
# file: projects/
# owner: gwuill
# group: gwuill
user::rwx
group::---
group:managers:rwx
mask::rwx
other::---
default:user::r-x
default:group::---
default:other::---
[gwuill@vbox projects]$ ls -la README.docx ; lsattr README.docx ; getfacl README.docx
-rw-------+ 1 gwuill gwuill 0 Feb 20 13:52 README.docx
----i----------------- README.docx
# file: README.docx
# owner: gwuill
# group: gwuill
user::rw-
group::---
group:managers:r--              #effective:---
group:admins:r--                #effective:---
group:sysadmins:r--             #effective:---
group:artists:r--               #effective:---
group:devs:r--                  #effective:---
group:rh:r--                    #effective:---
mask::---
other::---
[gwuill@vbox projects]$ ls -la ./the_zoo/ ; lsattr ./the_zoo/ ; getfacl ./the_zoo/
total 0
drwx------+ 2 gwuill gwuill  6 Feb 20 15:39 .
drwxrwx---+ 6 gwuill gwuill 89 Feb 20 15:39 ..
# file: the_zoo/
# owner: gwuill
# group: gwuill
user::rwx
group::r-x                      #effective:---
mask::---
other::---
default:user::rwx
default:user:suha:rwx
default:group::r-x
default:group:managers:r-x
default:group:artists:rwx
default:group:devs:rwx
default:mask::rwx
default:other::---
```

### 3. Sudo sudo sudo

🌞 **Gestion de `sudo`**

[sudoers](sudoers)

🌞 **Misconf ?**

- notre conf `sudo` est ptet un peu vulnérable là non ?
- normalement, y'a que `daniel` (car il appartient au groupe `sysadmins`) qui peut exécuter des commandes en `root`
- prouvez que 
  - `alysha` peut avoir accès à un shell `root`
  - euh prouvez que `liam` aussi
  - bah dukoo `jakub`...
  - nan `lev` pas toiiii

![sudo](./img/sudo.svg)

🌞 **Proposer une meilleure conf**

- plus personne doit pouvoir avoir un shell `root` à part daniel
- vous avez le droit à tout, mais les fonctionnalités essentielles doivent être conservées
- vous avez le droit à des compromis au nom de la sécurité :d
- mais genre lev il doit toujours pouvoir générer ses ptits certificats
- vous livrerez votre configuration finale dans le compte-rendu
