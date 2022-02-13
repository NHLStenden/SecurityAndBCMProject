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
   
