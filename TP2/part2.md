
# Part II : Observe

**Il est possible d'observer en temps réel ce que fait un programme. On dit qu'on peut *tracer* un programme.**

Plusieurs techniques pour faire ça, suivant ce qu'on veut voir ; dans ce TP on va se concentrer sur les *syscalls*.

L'outil le plus élémentaire à connaître est `strace`. Il s'utilise en terminal et affiche tous les *syscalls*  que réalisent un processus.

On va aussi utiliser `sysdig` plus moderne et plus puissant.

## Sommaire

- [Part II : Observe](#part-ii--observe)
  - [Sommaire](#sommaire)
  - [1. strace](#1-strace)
  - [2. sysdig](#2-sysdig)
    - [A. Intro](#a-intro)
    - [B. Use it](#b-use-it)
  - [3. Bonus : Stratoshark](#3-bonus--stratoshark)

## 1. strace

Si on veut tracer un processus avec `strace`, c'est comme ça :

```bash
# pour tracer l'exécution d'un echo par exemple
$ strace echo yo
```

🌞 **Utiliser `strace` pour tracer l'exécution de la commande `ls`**

```sh
[gwuill@vbox ~]$ strace ls 2>&1 | grep "write"
write(1, "oui\nsrm_buss_ii.mp3\n", 20)  = -1 EPIPE (Broken pipe)
```

🌞 **Utiliser `strace` pour tracer l'exécution de la commande `cat`**

```sh
[gwuill@vbox ~]$ strace cat oui 2>&1 | grep "open"
openat(AT_FDCWD, "oui", O_RDONLY)       = 3
[gwuill@vbox ~]$ strace cat oui 2>&1 | grep "write"
write(1, "coucou\n", 7)                 = -1 EPIPE (Broken pipe)
```

🌞 **Utiliser `strace` pour tracer l'exécution de `curl example.org`**

```sh
[gwuill@vbox ~]$ strace -c curl google.com
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 35.16    0.000704           4       141           mmap
 11.14    0.000223           6        35           poll
 10.04    0.000201           3        60        14 openat
  8.19    0.000164           1       107           rt_sigaction
  7.09    0.000142         142         1           execve
  6.99    0.000140           4        35           mprotect
  5.19    0.000104           2        36           read
  3.95    0.000079           1        54           close
  3.30    0.000066          11         6           write
  2.75    0.000055           1        46           fstat
  2.00    0.000040          40         1           recvfrom
  0.75    0.000015           0        24           futex
  0.50    0.000010           5         2           statfs
  0.50    0.000010           5         2           newfstatat
  0.40    0.000008           2         4           brk
  0.40    0.000008           4         2         1 access
  0.35    0.000007           7         1           munmap
  0.35    0.000007           3         2           getdents64
  0.30    0.000006           6         1           pipe
  0.20    0.000004           1         4           pread64
  0.10    0.000002           1         2         1 arch_prctl
  0.10    0.000002           2         1           set_robust_list
  0.10    0.000002           2         1           getrandom
  0.05    0.000001           1         1           set_tid_address
  0.05    0.000001           1         1           prlimit64
  0.05    0.000001           1         1           rseq
  0.00    0.000000           0         3           rt_sigprocmask
  0.00    0.000000           0         2           ioctl
  0.00    0.000000           0         2           socket
  0.00    0.000000           0         1         1 connect
  0.00    0.000000           0         1           sendto
  0.00    0.000000           0         1           getsockname
  0.00    0.000000           0         1           getpeername
  0.00    0.000000           0         2           socketpair
  0.00    0.000000           0         4           setsockopt
  0.00    0.000000           0         1           getsockopt
  0.00    0.000000           0         6           fcntl
  0.00    0.000000           0         1           sysinfo
  0.00    0.000000           0         1           clone3
------ ----------- ----------- --------- --------- ----------------
100.00    0.002002           3       597        17 total
```
## 2. sysdig

### A. Intro


### B. Use it

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par `ls`**

```sh
[gwuill@vbox ~]$ sudo sysdig proc.name=ls and evt.type=write
1250 11:55:30.199964051 0 ls (6650.6650) < write res=352 data=.[0m.[01;34mafs.[0m  .[01;36mbin.[0m  .[01;34mboot.[0m  .[01;34mdev.[0m  .[01;34
```

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par `cat`**

```sh
[gwuill@vbox ~]$ sudo sysdig proc.name=cat and evt.type=openat
1481 11:58:30.884869981 0 cat (6667.6667) > openat dirfd=-100(AT_FDCWD) name=oui(/home/gwuill/oui) flags=1(O_RDONLY) mode=0
[gwuill@vbox ~]$ sudo sysdig proc.name=cat and evt.type=write
1196 11:59:28.178268799 0 cat (6671.6671) < write res=7 data=coucou.
```

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par votre utilisateur**

```sh
[gwuill@vbox ~]$ sudo sysdig user.name=gwuill
```


🌞 **Livrez le fichier `curl.scap` dans le dépôt git de rendu**

[curl.scap](curl.scap)