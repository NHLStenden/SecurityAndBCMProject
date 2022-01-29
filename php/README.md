# Installatie van PHP als onderdeel van Apache2
Op de VM is inmiddels PHP geinstalleerd zodat deze gebruikt kan worden met
* Command line interface
* Apache2
* XDebug

In dit voorbeeld werken we met Apache 2.4.51, PHP versie 7.4.25 met XDebug 3.0.2.

Start een interactieve shell als root (`sudo -i`) en installeer de juiste modules.
```bash
  root@risksec: apt install php7.3 php7.3-ldap libapache2-mod-php
```

Na deze installatie heb je twee versies van PHP:
  1. PHP command line interface (CLI) waarmee je vanaf een terminal sessie PHP-scripts kan starten
  1. PHP integratie in Apache2 om aanvragen die Apache2 krijgt ook via PHP af te kunnen handelen.
  
Deze twee PHP-varianten hebben allebei hun eigen .ini-bestand. Deze kun je vinden op

  1. `/etc/php/7.3/cli/php.ini` voor de Command Line Interface
  1. `/etc/php/7.3/apache2` voor de PHP die in Apache geintegreerd is.
  
## PHP Instellingen
Wijzig de instellingen van PHP.ini. Deze kun je vinden op `/etc/php/7.3/apache2`

```bash
  root@risksec: nano /etc/php/7.3/apache2/php.ini
```

Als je in Developer modus werkt zijn onderstaande instellingen handig.
 1. `short_open_tag=On`
 1. `display_errors=On`

De eerste instelling (`short_open_tag=On`) zorgt er voor dat je verkorte PHP-tags kunt gebruiken: `<?` in plaats 
van `<?PHP`. Hoewel het laatste wordt aangeraden zijn er veel voorbeelden die gebruik maken
van de eerste instelling.

De tweede instelling (`display_errors=On`) zorgt er voor dat fouten niet alleen in de Apache/PHP-errorlog
(`/var/log/apache/.....`) komen te staan, maar ook in de browser. Dat is in productie echt niet de bedoeling
omdat je zo vaak teveel informatie weggeeft aan een hacker. 

Als je eenmaal klaar bent met je ontwikkelwerk, dan ga je je server zo instellen dat deze 
klaar is voor productie. We noemen dat "Server Hardening". Je kunt op internet veel
informatie vinden over Debian & PHP Server hardening. Doe hier je voordeel mee.

Voor een productie omgeving zijn onderstaande aan te raden:
 1. `short_open_tag=On`
 1. `display_errors=Off`

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
Als je in PHPStorm werkt moet je de debugger 'aanzetten'. Dat kan op verschillende manieren die netjes zijn beschreven
in een tutorial van JetBrains. De informatie van toepasbaar voor meerdere platformen, vooral het gedeelte waarbij je 
gebruik maakt van de browser-plugin om voor een specifieke pagina de debugger aan of uit te zetten.

De tutorial is [hier](jetbrains.com/help/phpstorm/configuring-xdebug.html) te vinden.

