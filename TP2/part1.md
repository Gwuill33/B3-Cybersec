# Part I : Learn


### A. `file`


🌞 **Utiliser `file` pour déterminer le type de :**

- la commande `ls`
```sh
[gwuill@vbox ~]$ file /usr/bin/ls
/usr/bin/ls: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=1afdd52081d4b8b631f2986e26e69e0b275e159c, for GNU/Linux 3.2.0, stripped
```

- la commande `ip`

```sh
[gwuill@vbox ~]$ file /usr/sbin/ip
/usr/sbin/ip: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=77a2f5899f0529f27d87bb29c6b84c535739e1c7, for GNU/Linux 3.2.0, stripped
```

- un fichier `.mp3` que vous aurez téléchargé sur le disque de la VM
```sh
[gwuill@vbox ~]$ file srm_buss_ii.mp3 
srm_buss_ii.mp3: MPEG ADTS, layer III, v1, 56 kbps, 44.1 kHz, Monaural
```

### B. `readelf`

🌞 **Utiliser `readelf` sur le programme `ls`**

```sh
[gwuill@vbox ~]$ readelf -h /usr/bin/ls
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Shared object file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x6b10
  Start of program headers:          64 (bytes into file)
  Start of section headers:          139032 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         30
  Section header string table index: 29
```

```sh
[gwuill@vbox ~]$ readelf -S /usr/bin/ls | grep ".text" -A 1
  [15] .text             PROGBITS         0000000000004d50  00004d50
       0000000000012532  0000000000000000  AX       0     0     16
```

### C. `ldd`

🌞 **Utiliser `ldd` sur le programme `ls`**

```sh
[gwuill@vbox ~]$ ldd /usr/bin/ls
        linux-vdso.so.1 (0x00007ffdb9ff8000)
        libselinux.so.1 => /lib64/libselinux.so.1 (0x00007f57f41b4000)
        libcap.so.2 => /lib64/libcap.so.2 (0x00007f57f41aa000)
        libc.so.6 => /lib64/libc.so.6 (0x00007f57f3e00000) -> Glibc
        libpcre2-8.so.0 => /lib64/libpcre2-8.so.0 (0x00007f57f410e000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f57f420b000)
```


## 2. Syscalls basics

### A. Syscall list

> Vous pourrez trouver une [liste des syscalls Linux sur un système x86_64 iciiii](https://filippo.io/linux-syscall-table/).

🌞 **Donner le nom ET l'identifiant unique d'un syscall qui permet à un processus de...**

- lire un fichier stocké sur disque
```
read() - 0
```
- écrire dans un fichier stocké sur disque
```
write() - 1
```
- lancer un nouveau processus
```
execve() - 59
```


> Pour la suite du TP, gardez-vous sous le coude les réponses apportées à cette question. Juste après vous allez regarder le langage machine contenu dans des exécutables à la recherche de l'appel à un *syscall*. Il faudra le repérer grâce à son identifiant !

![Fork exec](./img/forkexec.png)

### B. `objdump`

`objdump` permet de désassembler un programme, c'est à dire d'afficher le code contenu par un exécutable, sous forme de langage assembleur compréhensible par les humains (un peu, beaucoup plus qu'une purée d'octets en tout cas !)

🌞 **Utiliser `objdump`** sur la commande `ls`

```sh
[gwuill@vbox ~]$ objdump -M amd64 -j .text -d /usr/bin/ls | grep "call" 
    4d51:       e8 da f9 ff ff          callq  4730 <abort@plt>
    4d56:       e8 d5 f9 ff ff          callq  4730 <abort@plt>
    4d5b:       e8 d0 f9 ff ff          callq  4730 <abort@plt>
    4d60:       e8 cb f9 ff ff          callq  4730 <abort@plt>
    4d65:       e8 c6 f9 ff ff          callq  4730 <abort@plt>
    4d6a:       e8 c1 f9 ff ff          callq  4730 <abort@plt>
    4d6f:       e8 bc f9 ff ff          callq  4730 <abort@plt>
    4d74:       e8 b7 f9 ff ff          callq  4730 <abort@plt>
    4d79:       e8 b2 f9 ff ff          callq  4730 <abort@plt>
    4dbc:       e8 6f fb ff ff          callq  4930 <strrchr@plt>
    4dea:       e8 61 f9 ff ff          callq  4750 <strncmp@plt>
    4e05:       e8 46 f9 ff ff          callq  4750 <strncmp@plt>
    4e3c:       e8 1f fd ff ff          callq  4b60 <setlocale@plt>
    4e4b:       e8 20 fa ff ff          callq  4870 <bindtextdomain@plt>
    4e53:       e8 d8 f9 ff ff          callq  4830 <textdomain@plt>
```

```sh
gwuill@vbox ~]$ objdump -M amd64 -j .text -d /usr/bin/ls | grep "syscall" 
```

🌞 **Utiliser `objdump`** sur la librairie Glibc

```sh
[gwuill@vbox ~]$ objdump -M amd64 -j .text -d /lib64/libc.so.6 | grep "syscall" -B 5 | grep "0x3" -A 3
  127e95:       b8 03 00 00 00          mov    $0x3,%eax
  127e9a:       8b 7d a0                mov    -0x60(%rbp),%edi
  127e9d:       0f 05                   syscall 
```

> Pour exécuter un `syscall`, le programme met dans le registre `eax` l'identifiant du syscall (avec l'instruction `mov`) puis exécute l'instruction `syscall`. Vous cherchez donc une instruction `syscall` précédé d'un `mov` qui met l'identifiant de `close()` dans `eax`.

![How it works](./img/syscall_work.jpg)
