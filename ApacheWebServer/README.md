# Configuratie Apache WebServer

LET OP: de instructies die hier gegeven worden, zijn op de aangeleverde VM al uitgevoerd.

Wel interessant is de koppeling met LDAP en de geavanceerde configuratie in het geval zaken niet werken. Zie 
[hier](#advanced-configuration-en-ldap).

# Meer informatie over het opstellen van een Apache Virtuele Configuratie

Hier gaan we aan de slag met het configureren van de Apache WebServer. Het doel is 
  * Maak een virtuele host
  * Zorg voor een leesbare URL
  * Koppel de URL aan de virtuele host
  
In de taal van de Apache WebServer is een `virtuele host` niks meer dan een configuratie-bestand (`.conf`) 
die er voor zorgt dat je allerlei aspecten van een website kunt instellen. Denk daarbij aan:
  * naar welke URL moet ik 'luisteren'?
  * waar staan de bestanden?
  * wat moet ik allemaal toestaan of weigeren ?
    * gebruikers
    * aanpassingen via .htaccess
  * wat moet ik loggen ?
  * waar moet ik logbestanden opslaan?
  * custom error pagina's 
  * ....
  
We installeren de webserver via een bash-commando:
```bash
  $ sudo apt install apache2
```

Accepteer alle gesuggereerde pakketten en start de installatie. 

## Basis configuratie
Na de installatie is er slechts een basis-site. Hier gaan we wat aan doen. Open als `root` een editor, zoals 
nano (Terminal), Scite (GUI) of gedit (GUI). In deze editor gaan we een nieuw bestand maken genaamd `energy.conf`:

```bash
  $ sudo gedit /etc/apache2/sites-available/energy.conf
```

De meest basis configuratie ziet er ongeveer zo uit:
```apacheconf
 <VirtualHost  *:80>
   DocumentRoot /var/www/energy
   ServerName energy.org
   
   ErrorLog "/var/log/apache2/error.energy.log"
   CustomLog "/var/log/apache2/access.energy.log" common
   loglevel error

   <Directory "/var/www/energy/">
      Require all granted
   </Directory>
  </VirtualHost>
```

Van boven naar beneden enige uitleg. De configuratie is opgeslagen tussen `<VirtualHost>` en `</VirtualHost>`.
De stijl van schrijven lijkt een beetje op XML, maar is verder nogal losjes. 

Bij de `<VirtualHost>` staat een poort waar naar geluisterd wordt. Poort nummer 80 is de standaard. Het `*`
betekent dat er naar alle IP-adressen wordt geluisterd. Zie bij de [Referenties](#referenties) onderaan deze pagina voor meer informatie 
rondom configuratie van virtuele hosts.

De `DocumentRoot` geeft aan op welk pad (absoluut) de bestanden van de webserver gevonden kunnen worden. 

De `ServerName` is belangrijk! De waarde van deze instelling is de URL die de gebruiker in moet typen. 

**Let op**: Deze URL is niet magisch beschikbaar als je deze configuratie activeert! Je moet zelf de URL via
DNS of hosts (`/etc/hosts` ) inregelen!! Zie onderaan deze instructie bij de [Referenties](#referenties) en
[hoe te ontwikkelen](../Development/README.md).

De instelling `ErrorLog` geeft de locatie van de Error-log: in dit bestand worden de fouten gelogd. Wat er wel
of niet gelogd moet worden kun je instellen met `loglevel` . Tijdens ontwikkelen van een website kun je deze 
bijvoorbeld op `debug` zetten voor zeer uitgebreide informatie (let op je disk ruimte!).

De instelling `CustomLog` geeft aan waar de toegangsverzoeken worden gelogd. Zo kun je elke URL zien die
de webserver behandeld, wat het resultaat is en hoe snel het verzoek is afgehandeld.

Vervolgens worden van een specifieke directory (map) van het bestandssysteem aangegeven wat de instellingen
voor die map zijn. Dit doe je met het directive `<Directory>`. In deze simpele variant wordt alle verzoeken / gebruikers
toegestaan. Zie [Referenties](#referenties) (Apache toegangsbeveiliging) voor meer informatie.

## Configuratie activeren
Nu we een bestand hebben gemaakt, moeten we deze configuratie nog activeren. Dat doe je met het commando `a2ensite` (
Apache2 enable site). Je kunt het bestand dat je wilt activeren opgeven, of interactief werken:

```bash
  $ sudo a2ensite energy.conf
  Enabling site energy.
  To activate the new configuration, you need to run:
    service apache2 reload
  $ 
```
of interactief:
```bash
  $ sudo a2ensite 
  Your choices are: 000-default default-ssl energy.conf 
  Which site(s) do you want to enable (wildcards ok)?
  *energy*
  Enabling site energy.
  To activate the new configuration, you need to run:
    service apache2 reload
  $
```
**let op**: de sterretjes (*) rondom `*energy*` moet je **wel** invoeren! Dit zijn wildcards zodat je niet exact de naam
hoeft in te voeren.

Na afloop moet je nog de webserver herladen (`reload`) of herstarten (`restart`):
```bash
  $ sudo service apache2 restart
```
## Default Configuratie uitzetten
De kans is groot dat jouw installatie van Apache een default configuratie heeft geinstalleerd én geactiveerd. Deze moeten
we nog uitzetten omdat deze standaardconfiguratie *al het webverkeer* afvangt. Jouw eigen configuratie komt zo dus nooit aan bod.

Dit uitzetten doe je met het commando `a2dissite`: Apache2 Disable Site. Om te kijken welke configuraties al actief zijn kijk 
je in de map `/etc/apache2/sites-enabled`. Bijvoorbeeld:

```bash
	$ sudo ls -la /etc/apache2/sites-available
	total 24
	drwxr-xr-x 2 root root 4096 Feb 12 17:13 .
	drwxr-xr-x 8 root root 4096 Feb 12 15:19 ..
	-rw-r--r-- 1 root root 1332 Nov  3  2018 000-default.conf
	-rw-r--r-- 1 root root 6338 Jun 16  2019 default-ssl.conf
	-rw-r--r-- 1 root root  587 Feb 12 17:13 energy.conf
	
	$ ls -la /etc/apache2/sites-enabled/
	total 8
	drwxr-xr-x 2 root root 4096 Feb 13 07:25 .
	drwxr-xr-x 8 root root 4096 Feb 12 15:19 ..
	lrwxrwxrwx 1 root root   35 Feb 13 07:25 000-default.conf -> ../sites-available/000-default.conf
	lrwxrwxrwx 1 root root   32 Feb 12 17:14 energy.conf -> ../sites-available/energy.conf

```
Kijk wat de inhoud is van het configuratiebestand '000-default.conf'. Waarschijnlijk bevat het *geen* ServerName (er staat een # voor)
zodat deze als commentaar wordt gezien. Het effect is dat deze configuratie al het verkeer van poort 80 (zie eerste regel van het configuratie
bestand) afvangt en een standaard pagina serveert.

## Fouten bij herstarten webserver
<a name="fouten" />
 
Soms krijg je fouten bij het starten/herladen van de configuratie van de webserver. Een typische foutmelding is dan:
```bash
Job for apache2.service failed. See 'systemctl status apache2.service' and 'journalctl -xn' for details.
```
Voer in dat geval het gesuggeerde commando uit: (in dit geval zit er een fout in de configuratie van LDAP die je pas in de volgende stap maakt)
```bash
 $ systemctl status apache2.service
 ● apache2.service - LSB: Apache2 web server
    Loaded: loaded (/etc/init.d/apache2)
   Drop-In: /lib/systemd/system/apache2.service.d
            └─forking.conf
    Active: failed (Result: exit-code) since Fri 2019-02-08 15:17:05 CET; 1min 30s ago
   Process: 6657 ExecStop=/etc/init.d/apache2 stop (code=exited, status=0/SUCCESS)
   Process: 6442 ExecReload=/etc/init.d/apache2 reload (code=exited, status=0/SUCCESS)
   Process: 6686 ExecStart=/etc/init.d/apache2 start (code=exited, status=1/FAILURE)
 
 Feb 08 15:17:05 risksec apache2[6686]: Starting web server: apache2 failed!
 Feb 08 15:17:05 risksec apache2[6686]: The apache2 configtest failed. ... (warning).
 Feb 08 15:17:05 risksec apache2[6686]: Output of config test was:
 Feb 08 15:17:05 risksec apache2[6686]: AH00526: Syntax error on line 24 of /etc/apache2/sites-enabled/energy.conf:
 Feb 08 15:17:05 risksec apache2[6686]: AuthLDAPBindDN takes one argument, DN to use to bind to LDAP server. If not provided, will do an anonymous bind.
 Feb 08 15:17:05 risksec apache2[6686]: Action 'configtest' failed.
 Feb 08 15:17:05 risksec apache2[6686]: The Apache error log may have more information.
 Feb 08 15:17:05 risksec systemd[1]: apache2.service: control process exited, code=exited status=1
 Feb 08 15:17:05 risksec systemd[1]: Failed to start LSB: Apache2 web server.
 Feb 08 15:17:05 risksec systemd[1]: Unit apache2.service entered failed state.
```

## Rechten goed zetten
Het Apache proces draait als een achtergrond proces ("Daemon") en is hiervoor ingelogd met de gebruiker `www-data`. Deze gebruiker
moet de juiste rechten krijgen op de map waar jouw website opgeslagen is. Er zijn twee varianten:
1. Gebruik `/var/www/ ` om je website op de hosten
2. Gebruik je home-drive als locatie (bijv `/home/martin/website/`)

Beide hebben hun voor en nadelen. Als je voor de eerste optie gaat, dan heeft de webserver waarschijnlijk al de juiste rechten
maar jij als gebruiker niet. 
Als je voor de tweede optie gaat, dan heb jij wel rechten maar de webserver niet. 

Om dit te repareren maak je gebruik van de commando's `chgrp`, `chown` en `chmod`. Tip: er is niet alleen een gebruiker `www-data` 
maar ook een group `www-data`.  Op deze manier kun je de eigenaar en de groep van een map / submappen eenvoudiger besturen.

## Advanced configuration en LDAP
De volgende stap is het opnemen van beperkingen in de toegang: we willen niet dat alle gebruikers zomaar
op bepaalde beveiligde mappen mogen kijken. We gebruiken hiervoor de LDAP-identity store die we eerder 
hebben geinstalleerd. 

Het komt er op neer dat een gebruiker 
  * geauthentiseerd moet zijn (bewijs van wie je bent)
  * geautoriseerd zijn (wat mag je?)

Dat betekent dat we toegang tot een submap `intranet` in onze webserver omgeving gaan beveiligen. Een gebruiker
moet eerst inloggen zodat we na het inloggen kunnen controleren of die gebruiker ook in een bepaalde 
autorisatiegroep zit. Zo niet, dan krijgt de webserver domweg geen toegang tot de bestanden in die map.

**Merk op dat** je hiervoor dus geen enkele regel in je website-code voor hoeft op te nemen! Als de webserver de pagina
laadt, dan zijn de identiteit & autorisatieniveau reeds vastgesteld.


Een paar opvallende zaken:
  * Om een gebruiker te kunnen zoeken is toegang nodig tot de LDAP-service. Hiervoor zijn een gebruikersnaam
  en wachtwoord nodig. Om te voorkomen dat je een plain-tekst wachtwoord moet opnemen in configuratie 
  gebruik je onderstaande instellingen:

  * AuthLDAPBindDN "cn=webuserldap,ou=application,o=Energy,dc=energy,dc=org"
  * AuthLDAPBindPassword "exec:/bin/cat /root/website/password"
  


```apacheconfig
LDAPCacheTTL 10
LDAPOpCacheTTL 10

<VirtualHost *:80>
	# The ServerName directive sets the request scheme, hostname and port that
	# the server uses to identify itself. This is used when creating
	# redirection URLs. In the context of virtual hosts, the ServerName
	# specifies what hostname must appear in the request's Host: header to
	# match this virtual host. For the default virtual host (this file) this
	# value is not decisive as it is used as a last resort host regardless.
	# However, you must set it for any further virtual host explicitly.
	#ServerName www.example.com

	ServerAdmin webmaster@localhost
	ServerName energy.org

	DocumentRoot /var/www/energy

	<Directory "/var/www/energy/">
	   Require all granted
	</Directory>

<Directory "/var/www/energy/intranet">
    # Basic Authentication config
	
    # Indicate Basic Authentication (this will trigger a popup in the browser)
    AuthType Basic
    
    # The popup will indicate an environment . The AuthName will provide the description of the environment
    AuthName "Beheeromgeving van Energy.org"
    
    # which provider will support the BasicAuth protocol? ==> LDAP
    AuthBasicProvider ldap

    # What search query is used : domain / baseDN ? attribute = uid ? sub = search substree from baseDB ? filter (only objects of class iNetOrgPerson)
    AuthLDAPURL "ldap://localhost:389/o=energy,dc=energy,dc=org?uid?sub?(objectClass=iNetOrgPerson)" NONE

    # Which user is used to do an initial BIND to the LDAP Provider (hardly ever anonymous access is granted)
    AuthLDAPBindDN "cn=webuserldap,ou=application,o=Energy,dc=energy,dc=org"
    AuthLDAPBindPassword "exec:/bin/cat /root/website/password"

    # Which user-attribute contains the group-memberlist?
    AuthLDAPGroupAttribute uniqueMember   
    
    # are the groups in the group-member list DistinguishedNames (=DN)? 
    AuthLDAPGroupAttributeIsDN on

    # Finally, now everything is in place, now we can use the REQUIRE instruction to demand that a certain group must be used. this implies a valid user!
    # give the correct group where a user should be a member of
    # If AuthLDAPGroupAttributeIsDN=ON, then a full distinguishedName should be supplied
    Require ldap-group cn=allwebsiteusers,ou=application,o=Energy,dc=energy,dc=org

  </Directory>

    # Following setting will create a default web-page containing information about the server. This page can be obtained
    # using e.g. http://energy.org/server/cache-info
    <Location "/server/cache-info">
      SetHandler ldap-status
    </Location>



	ErrorLog ${APACHE_LOG_DIR}/error.energy.log
	CustomLog ${APACHE_LOG_DIR}/access.energy.log combined
        loglevel error authnz_ldap:debug

	# For most configuration files from conf-available/, which are
	# enabled or disabled at a global level, it is possible to
	# include a line for only one particular virtual host. For example the
	# following line enables the CGI configuration for this host only
	# after it has been globally disabled with "a2disconf".
	#Include conf-available/serve-cgi-bin.conf
</VirtualHost>


```
Je zet het wachtwoord dus in een aparte file `/root/website/passwd`, die je goed beveiligt. Er zijn ook 
andere veiligere methodes om zo'n wachtwoord kenbaar te maken op deze manier. Kijk 
[hier](https://httpd.apache.org/docs/2.4/mod/mod_authnz_ldap.html#authldapurl) voor meer informatie.   

**Let op**: er wordt bij elke inlogpoging van een gebruiker 2x toegang gevraagd richting de LDAP-service:
  1. De webserver heeft toegang nodig om de gebruiker te zoeken (in je config staat deze bij `AuthLDAPBindDN`)
  2. nadat de gebruiker is gevonden, wordt gepoogd een `ldap bind` te doen met de gevonden gebruiker en het wachtwoord
  dat de gebruiker heeft ingevoerd.
  
In de logfiles is dat verschil niet altijd duidelijk. Als je een foutmelding krijgt dat de gebruiker niet gevonden kan
worden, heeft dat vaak ook met toegang van **stap 1** te maken! Je kunt dan onderstaande foutmelding krijgen:
```log
  AH01695: auth_ldap authenticate: user martin authentication failed; URI /intranet/ [LDAP: ldap_simple_bind() failed][Server is unwilling to perform], referer: http://secrisk-dev.local/
```

## Laden van modules voor Apache LDAP
Om te zorgen dat LDAP ook daadwerkelijk bruikbaar is voor Apache2 is een aantal modules toegevoegd aan Apache. 
  * ldap
  * authnz_ldap

Dit is gedaan met onderstaande commando's. Deze hoef je dus niet meer uit te voeren!

```bash
root@NHLStendenSecurityRisk:~# a2enmod ldap authnz_ldap
Enabling module ldap.
Considering dependency ldap for authnz_ldap:
Module ldap already enabled
Enabling module authnz_ldap.
To activate the new configuration, you need to run:
  systemctl restart apache2
```

Let op dat de volgende modules nodig zijn:
  
Herstart na afloop van deze commando's de Apache2 Webserver 
  

## Testen en testen en testen en...
Het werkend krijgen van deze configuratie is niet eenvoudig. Een paar tips:
  * Zet het loglevel van de LDAP-module op DEBUG: `loglevel error authnz_ldap:debug`
  * monitor de logfile met `tail -f /var/log/apache2/error.energy.log` (de naam uit de configuratie file)
  * als je een configuratie wijzigt, herstart dan de webserver : `sudo service apache2 restart`.
  dit zorgt er voor dat de cache van LDAP geleegd wordt en je weer met schone wachtwoorden begint.
  * als je Apache2 herstart, en je krijgt een foutmelding dan staat er een commando dat je kunt uitvoeren
  om problemen te onderzoeken. Maak daar gebruik van en lees zorgvuldig de output van dat commando! (zie ook [hier](./#fouten))
  * test eventueel je configuratie van Apache met `apache2 -S`. Zie verderop 
  
  
  
Veel voorkomende problemen:
  * encryptie van wachtwoorden is onjuist
  * wachtwoord fouten
  * locaties van bestanden / mappen deugen niet (DocumentRoot)


# Testen van je Apache Configuratie
Het kan handig zijn om de configuratie van je Apache Virtual Config te kunnen laten controleren door Apache. Daarvoor kun je
het commando `apache2 -S`  gebruiken. Echter, het komt vaak voor dat je dan foutmeldingen krijgt die niks met jouw 
configuratie te maken hebben, maar met de globale configuratie van Apache2: bepaalde globale variabelen zijn niet 
gezet als je dit commando vanaf de commando-regel opstart. 

```bash
[Sat Feb 15 16:26:30.733272 2020] [core:warn] [pid 14639] AH00111: Config variable ${APACHE_RUN_DIR} is not defined
apache2: Syntax error on line 80 of /etc/apache2/apache2.conf: DefaultRuntimeDir must be a valid directory, 
absolute or relative to ServerRoot
```

Dit kun je fixen met het runnen van een script die deze
variabelen wel goed zet: `source envvars`:
```bash
root@risksec:/root# cd /etc/apache2/
root@risksec:/root# source envvars
root@risksec:/root# apache2 -S
root@risksec:/root# apache2 -S
AH00112: Warning: DocumentRoot [/var/www/energy/] does not exist
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
VirtualHost configuration:
*:80                   energy.org (/etc/apache2/sites-enabled/energy.conf:1)
ServerRoot: "/etc/apache2"
Main DocumentRoot: "/var/www/html"
Main ErrorLog: "/var/log/apache2/error.log"
Mutex default: dir="/var/run/apache2/" mechanism=default 
Mutex watchdog-callback: using_defaults
PidFile: "/var/run/apache2/apache2.pid"
Define: DUMP_VHOSTS
Define: DUMP_RUN_CFG
User: name="www-data" id=33
Group: name="www-data" id=33
root@risksec:/root# 
```

## Werken met poorten
Je kunt je webserver op een andere poort laten werken, bijvoorbeeld poort 8080 of poort 8443. Je moet dit dan aangeven 
bovenin je configuratie van je virtuele host. Bijvoorbeeld

```
<VirtualHost  *:8080>
```

Om te zorgen dat Apache2 überhaupt naar deze port *kan* luisteren moet je dat ook aangeven in de Apache2 http-configuratie. Deze is te vinden op

```conf
/etc/apache2/ports.conf
```

Voeg eenvoudig een regel toe:
```bash
Listen 80
listen 8080

<IfModule ssl_module>
        Listen 443
</IfModule>

<IfModule mod_gnutls.c>
        Listen 443
</IfModule>
```

Nadat je de configuratie bestanden hebt aangepast moet je Apache2 wel herstarten of herladen:
```bash
 service apache2 restart
```


# Referenties <a name="referenties"/>
  * [Apache Virtual Hosts config](https://httpd.apache.org/docs/2.4/vhosts/examples.html)
  * [Linux: How to edit hosts file](https://manpages.ubuntu.com/manpages/bionic/en/man5/hosts.5.html)
  * [Apache 2.4 Core](https://httpd.apache.org/docs/2.4/mod/core.html)
  * [Apache 2.4 logging](http://httpd.apache.org/docs/current/mod/mod_log_config.html)
  * [Apache toegangsbeveiliging](https://httpd.apache.org/docs/2.4/howto/access.html)
