# Installatie Samba - delen van bestanden
Als je bestanden die in je VM zitten wilt gebruiken op je laptop/PC dan moet je Samba gebruiken. 
Samba is een naam die gebaseerd is het SMB-protocol: *Server Message Block*. Dit protocol loopt over poort
445 en is eenvoudig te installeren via de command-line in Linux.

Als de software eenmaal geinstalleerd is moet je nog wel wat kleine zaken regelen in de welbekende .conf-bestanden.

## Installatie

```bash
  root@risksec:~# apt install samba
```

## Configuratie
De configuratie voor Samba is opgeslagen in het bestand `/etc/samba/smb.conf`. Je moet de volgende aanpassingen doen
om je HOME-drive (bijv `/home/student`) te kunnen gebruiken met lezen+schrijven.

 1. Zoek de sectie  `[homes]`
 1. blader door totdat je ziet staan `read only = yes`
 1. verander deze naar `read only = no`
 1. blader eventueel door naar de entry `valid users`
 1. verander de standaard instelling eventueel naar de namen van gebruikers, gescheiden door een spatie.
 
Verlaat je editor met opslaan van de wijzigingen en herstart de service voor Samba (zorg dat je root bent). Controleer
vervolgens de instellingen met `testparm -L`. Je kunt hiermee alle instellingen en gedeelde mappen zien.
```bash
  root@risksec:~# service smbd restart
  root@risksec:~# testparm -L
.........snipped............
[homes]
	comment = Home Directories
	create mask = 0775
	read only = No
	valid users = martin 
.........snipped............

```

## Gebruikers toestaan in te loggen
Als een gebruiker toegang wil hebben tot Samba, dan moet er voor de gebruiker een apart wachtwoord opgegeven worden.
Dat doe je met het commando `smbpassword`. In onderstaande commando wordt voor de gebruiker `martin` een nieuw 
wachtwoord voor Samba aangemaakt. De optie `-a` zorgt voor deze toevoeging ('add'). Je kunt later hiermee ook 
wachtwoorden aanpassen. Zie manual page voor dit commando voor meer informatie (`man smbpasswd`).

```bash
  root@risksec:~# smbpasswd -a martin
```

## Eigen share toevoegen voor website
Als je website niet in je home-directory staat, dan kan het nuttig zijn om een aparte share te maken. Voeg eenvoudig
aan het einde van het bestand `/etc/samba/smb.conf` een nieuwe sectie toe. Bijvoorbeeld:

```text
  [website]
  browseable = yes
  path = /var/www/energy
  valid users = martin
  force group = www-data
  force user = www-data
  create mask = 660
  directory mask = 770
```

Eén van de problemen waar je tegen aan loopt bij het op afstand maken van bestanden, is dat deze zonder extra maatregelen 
waarschijnlijk niet leesbaar zijn voor de webserver Apache2. Dit proces maakt namelijk gebruik van een user met naam
`www-data`. Daarom zie je in bovenstaande daarvoor een drietal maatregelen:

  1. `force group = www-data` : doe net alsof de ingelogd gebruiker lid is van de groep `www-data`. Nieuwe bestanden
  worden dan aangemaakt met als groep `www-data`. Op deze manier kun je zorgen dat de webserver de bestanden 
  wel mag gebruiken, mits je de rechten goed instelt. Dat regelen de volgende twee items.
  1. `create mask` = 660. Dit zorgt er voor dat aangemaakte 'normale' bestanden de file-toegang goed heeft staan: `-rw-rw----`
      1. eerste set `rw-` de eigenaar mag lezen, schrijven 
      1. tweede set `rw-` de groep `www-data` mag lezen, schrijven 
      1. derde set `---` anderen (zijnde niet de eigenaar en groep) hebben geen rechten
  1. `directory mask = 770` als een directory aangemaakt wordt, dan krijgt deze de rechten `drwxwrx---`.
      1. eerste set `rwx` de eigenaar mag lezen, schrijven en de map betreden (inhoud zien)
      1. tweede set `rwx` de groep `www-data` mag lezen, schrijven en de map betreden (inhoud zien)
      1. derde set `---` anderen (zijnde niet de eigenaar en groep) hebben geen rechten


Vergeet niet je instellingen te activeren & controleren!
```bash
  root@risksec:~# service smbd restart
  root@risksec:~# testparm -L
```

## Inloggen op de gedeelde mappen
In Windows is het openen van bestanden op een andere netwerklocatie eenvoudig te regelen via de verkenner. Aangezien
jouw VM zich nu waarschijnlijk als een server gedraagt (zie ook [hier](../Netwerk/README.md)) kun je eenvoudig via
het IP-adres van je server deze bestanden openen. 

Typ in de adresbalk van je Windows Verkenner bijvoorbeeld onderstaande:

`\\192.168.233.203\martin` 

om de gebruikers HOME-directory van gebruiker `martin` te openen op IP-adres `192.168.233.203`. Je kunt eventueel
een netwerkdrive mappen naar zo´n locatie, zodat je bijvoorbeeld via `H:` deze bestanden makkelijk kunt gebruiken.

Je kunt dit ook met een command-line commando doen:

```
NET USE w: \\energy.org\website /user:student wachtwoord
```
Het laatste woord is overigens het wachtwoord.

Op Linux kun je vaak verbinding maken via onderstaande URL:

`smb://192.168.233.203/martin/`

of door deze op te nemen in het bestand `/etc/fstab`. Zie de referenties onderaan voor meer informatie hierover. 

## Gebruik van PHPStorm en gedeelde mappen
Let op dat PHPStorm geen project accepteert dat in de root van een driveletter (bijv. w:\) staan. 
In dat geval dien je de gedeelde map in Samba mogelijk één map hoger instellen.

Dus niet 
```text
  [website]
  browseable = yes
  path = /var/www/energy
  valid users = martin
  force group = www-data
  force user = www-data
  create mask = 660
  directory mask = 770
```

maar wel:
```text
  [website]
  browseable = yes
  path = /var/www
  valid users = martin
  force group = www-data
  force user = www-data
  create mask = 660
  directory mask = 770
```



## Referenties
  * [Samba op Wikipedia](https://nl.wikipedia.org/wiki/Samba_(software))
  * [SMB / CIFS Protocol](https://nl.wikipedia.org/wiki/Server_Message_Block)
  * [Ubuntu Samba Tutorial](https://ubuntu.com/tutorials/install-and-configure-samba#1-overview)
  * [Samba organisation](https://www.samba.org/)
  * [Linux Mount Windows share permanently](https://wiki.ubuntu.com/MountWindowsSharesPermanently)
