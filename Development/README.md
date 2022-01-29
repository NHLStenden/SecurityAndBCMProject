# Hoe te ontwikkelen
Nu de VM klaar is voor het ontwikkelen van een website is er nog wel wat uitleg nodig hoe je dit efficiënt kunt doen.

Hiervoor doorlopen we een aantal stappen:
  1. Aanpassen hosts-file voor het beschikbaar maken van de URL van de website
  2. verbinding maken met de bestanden op de VM via Samba
  3. PHP Storm IDE configureren
  4. Debugging configureren
  5. Browser-plugin installeren voor debugging

Deze stappen zijn uitgelegd in [deze](https://web.microsoftstream.com/video/e751ac72-d9ff-412a-9ea4-b1f1e34c6f78)
video.

## Aanpassen hosts-file
De hosts-file van een systeem wordt gebruikt om een leesbare naam te koppelen aan een IP-adres. Dit kan vervolgens
gebruikt worden voor bijvoorbeeld inloggen via SSH en het vertalen van een naam naar een URL van een website.

Op Windows staat deze hosts-file op `c:\windows\system32\drivers\etc\hosts`. Bij Linux/Debian staat deze op `/etc/hosts`.
Open een editor en voeg daar een nieuwe regel aan toe. Let op dat je zowel bij Windows als Linux verhoogde rechten
nodig hebt om deze bestanden aan te passen. 

Tip: als je op Windows het programma Notepad++ gebruikt, dan zal deze bij het opslaan vragen om (tijdelijk) de rechten
van het programma te verhogen via UAC.

De regel die je toe moet voegen heeft het volgende formaat:
```
<ip-adres><tab><leesbare naam>
```

### IP Adres vinden
Het IP-adres van je machine kun je vinden met

```shell
  root@secbcm:~# ip addr
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
  2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:b8:88:cd brd ff:ff:ff:ff:ff:ff
    inet 192.168.123.44/24 brd 192.168.123.255 scope global dynamic enp0s3
       valid_lft 85048sec preferred_lft 85048sec
    inet6 2a02:a460:f4dd:1:a00:27ff:feb8:88cd/64 scope global dynamic mngtmpaddr 
       valid_lft 190524sec preferred_lft 104124sec
    inet6 fe80::a00:27ff:feb8:88cd/64 scope link 
       valid_lft forever preferred_lft forever
```

Kijk bij de tweede netwerkkaart (`2: enp0s3`) en dan de regel `inet` (voor IPv4) of `inet6` (voor IPv6). Daar staat

```shell
    inet 192.168.123.44/24 brd 192.168.123.255 scope global dynamic enp0s3
```
In dit geval is het ip adres dus `192.168.123.44`. 

### Leesbare naam vaststellen
De leesbare naam die we gaan gebruiken moet overeenkomen met de configuratie die in Apache2 op je VM geconfigureerd is.
Als je een andere naam kiest zal de website in de VM vanaf jouw PC niet bruikbaar zijn.

We kunnen de naam van deze website opvragen door in de configuratie van de VM te kijken.
```shell
student@secbcm:~$ grep ServerName /etc/apache2/sites-available/energy.conf
```
We krijgen dan de volgende output
```apacheconf 
	# The ServerName directive sets the request scheme, hostname and port that
	# redirection URLs. In the context of virtual hosts, the ServerName
	#ServerName www.example.com
	ServerName energy.org
student@secbcm:~$ 
```
De regels met een `#` zijn commentaar en kunnen we dus negeren. De servernaam is in dit geval dus `energy.org`. 
Dat betekent dat Apache elke URL die opgevraagd wordt checkt tegen de naam `energy.org`. Bijvoorbeeld
`http://energy.org/index.php`.

Als de URL als hostnaam dus `energy.org` heeft dan zal Apache de betreffende pagina opzoeken in de map en terugsturen.

### Uiteindelijke resultaat
Uiteindelijk staat er dus het volgende in jouw hosts-file:

```shell
192.168.123.44  energy.org
```

Je kunt dit testen met SSH: gebruik niet het IP-adres maar de naam energy.org. Bijvoorbeeld vanaf de command-line in 
Linux of Windows:
```shell
ssh student@energy.org
```

## Verbinding maken met SAMBA voor het delen van bestanden
Om de bestanden van je website te kunnen gebruiken is samba geconfigureerd. Zie voor meer een uitgebreide uitleg 
de [Samba](../Samba/README.md) uitleg.

Start nu bijvoorbeeld PHPStorm en open de bestanden in je IDE zodat je kunt ontwikkelen. Gebruik hiervoor de share
`website`.


## PHP & LDAP
Jullie gaan met LDAP en PHP aan de slag. Daarom moet ook in PHP de LDAP modules gebruikt worden.
[LDAP module](http://php.net/manual/en/book.ldap.php).

* Maak eerst verbinding met [ldap_connect](http://php.net/manual/en/function.ldap-connect.php)
* Doe een [ldap-bind](http://php.net/manual/en/function.ldap-bind.php) met de juiste
    * Username
    * Password
* Voer een LDAP-query uit met bijv. [ldap_search](http://php.net/manual/en/function.ldap-search.php)
* Haal de gevonden items op met [ldap_get_entries](http://php.net/manual/en/function.ldap-get-entries.php)

In de voorbeelden van de hiervoor genoemde URL's van de PHP.net handleidingen staan verder uitgewerkte voorbeelden.

**Let op**: Zorg dat (als dat nodig is) je de LDAP-protocol versie op 3 zet! Immers, bij de installatie van de OpenLDAP
server heb je V2-protocol niet geactiveerd. Dergelijke aanpassingen doe je na de `ldap_connect` en vóór de `ldap_bind`.

Een voorbeeld:
```php
    $ldapconn = ldap_connect("ldap.example.com")
                or die("Could not connect to LDAP server.");
    
    if ($ldapconn) {
        // set protocol version to 3
        ldap_set_option($ldapconn, LDAP_OPT_PROTOCOL_VERSION, 3);
        
        // binding to ldap server
        $ldapbind = ldap_bind($ldapconn, $ldaprdn, $ldappass);
        
        // verify binding
        if ($ldapbind) {
            echo "LDAP bind successful...";
            
        } else {
            echo "LDAP bind failed...";
        }
        
    }
  
```

**Let op**: ALs je heel veel gebruikers hebt, of gebruikers in een groep, dan kan het zijn dat resultaten gepagineerd
opgeleverd worden. Kijk in dat geval eens bij [LDAP Paged Results Response](http://php.net/manual/en/function.ldap-control-paged-result-response.php).


## Debugging en PHPStorm & LDAP
Om te kunnen debuggen met PHP is het handig om een debugger te installeren naast PHP. In dit geval kies ik voor [XDebug]
(https://www.xdebug.org).

```bash
  sudo apt-get install php-xdebug
```
Daarnaast dien je een aantal aanpassingen te doen in je configuratie van PHP. De [documentatie](https://xdebug.org/docs/install#configure-php)
van XDebug kan goed helpen, maar hier alvast wat tips voor de VM met Debian .

De configuratie van PHP staat voor het grootste deel in de `php.ini` maar ook in extra configuratie bestanden in de map

`/etc/php/7.3/apache2/conf.d`

Door het aanmaken van een nieuw bestand in deze map kun je eenvoudig configuratie informatie toevoegen aan de PHP-configuratie
als onderdeel van Apache. Als je via de package manager (`apt`) de installatie doet zoals hierboven beschreven, wordt
dit configuratiebestand meteen toegevoegd.

Met onderstaande commando's kun je vervolgens deze configuratie toepassen. Het herstarten van Apache2 is nodig
omdat je de onderliggende configuratie hebt aangepast.

```bash
  $ cd /etc/php/7.3/apache2/conf.d
  $ nano 20-xdebug.ini
  $ service apache2 restart 
```

Zet onderstaande informatie in je configuratie:
```conf
  [xdebug]
  zend_extension=xdebug.so
  xdebug.mode=debug
  xdebug.remote_enable=1
```

## Controleren juiste configuratie
Uiteindelijk kun je via het command `php -v` zien of je extensie werkt. Dit zou onderstaande informatie moeten geven (
afhankelijk van de versie van PHP en het platform uiteraard).

```
PHP 7.3.19-1~deb10u1 (cli) (built: Jul  5 2020 06:46:45) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.3.19, Copyright (c) 1998-2018 Zend Technologies
with Zend OPcache v7.3.19-1~deb10u1, Copyright (c) 1999-2018, by Zend Technologies
with Xdebug v2.7.0RC2, Copyright (c) 2002-2019, by Derick Rethans
```

### Debug en XAMPP
Als je gebruik maakt van XAMPP (omdat bijvoorbeeld je VM nog niet af is), dan kun je ook daar XDebug installeren.
[Tutorial](gist.github.com/odan/1abe76d373a9cbb15bed)

Wat erg **belangrijk** is, is dat je na het downloaden van de XDebug de betreffende DLL (.dll-bestand) in de juiste map
plaatst. Het gaat om de subfolder `C:\xampp\php\ext`. Ook als je in je configuratiebestand aangeeft dat ie op een bepaalde
locatie staat die anders is, dan werkt het niet. Hij **moet** gewoon in die subfolder **php\ext** staan.

Maak aanpassingen in de `php.ini`  door aan het einde de volgende regels toe te voegen:

```conf
  [xdebug]
  zend_extension=php_xdebug.dll
  xdebug.mode=debug
  xdebug.remote_enable=1
```

Er zijn nog veel meer mogelijkheden


## Debugging en PHP Storm
Debuggen is nogal complex en is daarom al voor geconfigureerd in de VM. Je moet mogelijk nog wel extra stappen configureren
in je IDE. Als je in PHPStorm werkt moet je de debugger 'aanzetten'. Dat kan op verschillende manieren die netjes zijn beschreven
in een tutorial van JetBrains. De informatie is toepasbaar voor meerdere platformen, vooral het gedeelte waarbij je
gebruik maakt van de browser-plugin om voor een specifieke pagina de debugger aan of uit te zetten.

De tutorial is [hier](jetbrains.com/help/phpstorm/configuring-xdebug.html) te vinden.

## Debuggen
Debuggen is nogal complex en is daarom al voor geconfigureerd in de VM. Je moet mogelijk nog wel extra stappen configureren
in je IDE. Deze staan beschreven in de video en op de website van 
[JetBrains](https://www.jetbrains.com/help/phpstorm/configuring-xdebug.html).

