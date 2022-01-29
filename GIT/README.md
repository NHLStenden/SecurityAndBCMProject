# Bestanden beschikbaar maken
In deze installatiehandleiding zitten enkele bestanden die gebruikt moeten worden om de installatie af te kunnen maken.
Om dit proces te vereenvoudigen is een GIT client geinstalleerd.



## Repository ophalen
We maken een kloon (`clone`) door middel van een `git clone` commando in een aparte directory.

```bash
root@debian:~# mkdir -p ~root/repo
root@debian:~# cd ~root/repo
root@debian:~/repo# git clone https://github.com/NHLStenden/SecurityAndBCMProject
Cloning into 'SecurityAndRiskProject'...
remote: Enumerating objects: 222, done.
remote: Counting objects: 100% (222/222), done.
remote: Compressing objects: 100% (191/191), done.
remote: Total 381 (delta 42), reused 187 (delta 24), pack-reused 159
Receiving objects: 100% (381/381), 7.61 MiB | 5.06 MiB/s, done.
Resolving deltas: 100% (82/82), done.
```

Hierna zou de directory er als volgt uit moeten zien:

```bash
root@debian:~/repo# 
root@debian:~/repo# ls -alp
total 12
drwxr-xr-x  3 root root 4096 Jan  9 16:06 ./
drwx------  3 root root 4096 Jan  9 16:06 ../
drwxr-xr-x 12 root root 4096 Jan  9 16:06 SecurityAndRiskProject/
```

En nog een niveau dieper: (merk op dat als de repo verandert de lijst anders kan zijn).

```bash
root@debian:~/repo# cd SecurityAndRiskProject
root@debian:~/repo# ls -alp
total 56
drwxr-xr-x 12 root root 4096 Jan  9 16:06 ./
drwxr-xr-x  3 root root 4096 Jan  9 16:06 ../
drwxr-xr-x  3 root root 4096 Jan  9 16:06 ApacheLDAPStudio/
drwxr-xr-x  3 root root 4096 Jan  9 16:06 ApacheWebServer/
drwxr-xr-x  3 root root 4096 Jan  9 16:06 DebianInstall/
drwxr-xr-x  2 root root 4096 Jan  9 16:06 files/
drwxr-xr-x  8 root root 4096 Jan  9 16:06 .git/
-rw-r--r--  1 root root    8 Jan  9 16:06 .gitignore
drwxr-xr-x  2 root root 4096 Jan  9 16:06 images/
drwxr-xr-x  3 root root 4096 Jan  9 16:06 Netwerk/
drwxr-xr-x  3 root root 4096 Jan  9 16:06 OpenLDAP/
drwxr-xr-x  2 root root 4096 Jan  9 16:06 php/
-rw-r--r--  1 root root 1222 Jan  9 16:06 README.md
drwxr-xr-x  2 root root 4096 Jan  9 16:06 Samba/

```

Je hebt nu de bestanden tot je beschikking voor alle handleidingen in deze repository.
