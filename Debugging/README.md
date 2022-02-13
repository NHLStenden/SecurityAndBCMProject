## Debugging en PHPStorm
Om te kunnen debuggen met PHP is het handig om een debugger te installeren naast PHP. In dit geval kies ik voor [XDebug]
(https://www.xdebug.org).

```bash
  sudo apt-get install php-xdebug
```
Daarnaast dien je een aantal aanpassingen te doen in je configuratie van PHP. De [documentatie](https://xdebug.org/docs/install#configure-php)
van XDebug kan goed helpen, maar hier alvast wat tips voor de VM met Debian .

De configuratie van PHP staat voor het grootste deel in de `php.ini` maar ook in extra configuratie bestanden in de map

`/etc/php/7.4/apache2/conf.d`

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

# Debugging aan de praat krijgen
De Debugger in de VM wil verbinding maken met jouw lokale machine. Dat moet via een IP-adres. Dat IP-adres van jouw machine
wil natuurlijk nog wel eens veranderen, afhankelijk of je op school zit of thuis. Daarom probeert XDebug
dat adres slim te ontdekken (*discovery*). Hiervoor is in XDebug versie 3 een speciale configuratie nodig.

In de VM zit een bestand voor de configuratie van de debugger, en deze is te vinden op

`/etc/php/7.4/apache2/conf.d/20-xdebug.ini`

De inhoud is als volgt:

```apacheconf
# Settings for XDebug 3
zend_extension=xdebug.so
xdebug.idekey=PHPSTORM
xdebug.mode=debug
xdebug.default_enable=1
xdebug.client_port=9003
xdebug.remote_handler=dbgp
xdebug.discover_client_host=on
xdebug.log=/tmp/xdebug.log
xdebug.log_level=1
xdebug.start_with_request=yes
```

De volgende items zijn van belang:
* `discover_client_host=on` : dit zorgt er voor dat XDebug probeert verbinding terug te maken naar jouw laptop .
  Hiervoor wordt de PHP-variabele `$SERVER_['REMOTE_ADDR]` gebruikt.
* `client_port=9003` : dit is de poort waarop XDebug informatie uitwisselt. Deze staat normaliter ook goed in
  PHPStorm.
* `log_level=1`: dit is het niveau van informatie (log) die XDebug neer zal zetten in de genoemde logfile
  (`log=/tmp/xdebug`). Deze kan na goede werking op `0` worden gezet.

**LET OP**: de logfile wordt in verband met beveiliging niet rechtstreeks in `/tmp/` neergezet worden,maar
in een subdirectory. Bijvoorbeeld:
```
root@secbcm:/# ls /tmp
systemd-private-fcd15963223043aa910e9cdf91342b37-apache2.service-2vSVoj
systemd-private-fcd15963223043aa910e9cdf91342b37-systemd-logind.service-s5K04f
systemd-private-fcd15963223043aa910e9cdf91342b37-systemd-timesyncd.service-dKzXkj
xdebug.log
root@secbcm:/# ls /tmp/systemd-private-fcd15963223043aa910e9cdf91342b37-apache2.service-2vSVoj/tmp/
xdebug.log
root@secbcm:/#
```

Je moet dus die xdebug.log hebben in de map die begint met `systemd-private` en eindigt met `apache2.service`.
Elke keer dat je `Apache2` opnieuw opstart wordt er een nieuwe map aangemaakt!

Zie deze links om het debuggen aan de praat te krijgen mocht dat niet lukken

Jetbrains PHPStorm
* [Validate the Configuration of a Debugging Engine](https://www.jetbrains.com/help/phpstorm/validating-the-configuration-of-the-debugging-engine.html)
* [Configure a debugging engine](https://www.jetbrains.com/help/phpstorm/configuring-a-debugging-engine.html)
* [Configure Xdebug](https://www.jetbrains.com/help/phpstorm/configuring-xdebug.html#updatingPhpIni)
* [Zero config tutorial](https://www.jetbrains.com/help/phpstorm/2021.3/zero-configuration-debugging.html?utm_source=product&utm_medium=link&utm_campaign=PS&utm_content=2021.3)
* [Configure Remote PHP Interpreters](https://www.jetbrains.com/help/phpstorm/configuring-remote-interpreters.html)

XDebug
* [XDebug wizard](https://xdebug.org/wizard)
* [Setting the client host](https://xdebug.org/docs/all_settings#client_host)
* [Start with request](https://xdebug.org/docs/all_settings#start_with_request)

## Controleren juiste configuratie
Uiteindelijk kun je via het command `php --version` zien of je extensie werkt. Dit zou onderstaande informatie moeten geven (
afhankelijk van de versie van PHP en het platform uiteraard).

```
root@secbcm:~# php --version
PHP 7.4.25 (cli) (built: Oct 23 2021 21:53:50) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.25, Copyright (c), by Zend Technologies
    with Xdebug v3.0.2, Copyright (c) 2002-2021, by Derick Rethans
```
