# Configuratie Apache Directory Studio for LDAP

Installatie volgens [deze](https://directory.apache.org/studio/) handleiding.

Je hebt minimaal versie 11 nodig van Java. Tegenwoordig kun je deze downloaden via
[deze](https://www.oracle.com/java/) link. Gebruik bijvoorbeeld de .exe-file of een .zip-file om deze 
op je systeem te installeren.

Na het installeren van Apache Directory Studio kan het zijn dat je de .ini-file moet aanpassen om te zorgen dat
de juiste JAVA-versie gebruikt wordt. Het gaat om het pad naar de `bin` directory waar de `java.exe` 
gevonden kan worden. 

Ga op zoek naar de regel
`-vm` en een padnaam (Linux-style) op de volgende regel. Pas deze regel aan zodat je bijvoorbeeld onderstaande 
krijgt. In dit voorbeeld is de JAVA versie 15 te vinden op het pad `C:\PortableApps\CommonFiles\jdk-15.0.2\bin`. 

```ini
-startup
plugins/org.eclipse.equinox.launcher_1.6.0.v20200915-1508.jar
--launcher.library
plugins/org.eclipse.equinox.launcher.win32.win32.x86_64_1.2.0.v20200915-1442
/studio-rcp/resources/icons/linux/studio.xpm
###
#Uncomment_to_configure_the_language
#https://directory.apache.org/studio/faqs.html#how-to-set-the-language-of-studio
#-nl
#en
###
#Uncomment_to_configure_Java_version_to_use
#https://directory.apache.org/studio/faqs.html#how-to-set-the-java-vm-to-use
-vm
C:\PortableApps\CommonFiles\jdk-15.0.2\bin
-vmargs
-Dosgi.requiredJavaVersion=11
###
#Uncomment_to_configure_heap_memory
#https://directory.apache.org/studio/faqs.html#how-to-increase-the-heap-memory
#-Xms1g
#-Xmx2g

```

## Stap 1: Aanmaken nieuwe verbinding
Als je Apache Directory Studio hebt gestart krijg je na het welkom scherm onderstaande afbeelding: 
![img1](images/config-apache-directory00.png)
Ga naar het menu, en klik op `LDAP` en dan `New Connection`. Er wordt een dialoogvenster geopend:

![img2](images/config-apache-directory01.png)
Vul de gegevens in zoals weergegeven:
  * Connection name mag je zelf weten
  * Hostname = localhost óf het IP-adres van je Virtuele Machine (bijv. 10.0.0.2 of 192.168.1.45)
  * Port = 389
  
Druk vervolgens op de knop 'Check Network Parameter' om te kijken of de LDAP-service bereikt kan worden. Als dat lukt krijg je onderstaande bevestiging: 

![img3](images/config-apache-directory02.png)

Druk op `Next` om naar de volgende stap te gaan. 


In de volgende stap gaan we inloggen op de LDAP-service met het **Admin**-account. Let op: als je hier het wachtwoord opslaat kan iedereen dus 
als admin inloggen zonder wachtwoord op te geven!

Echter, voor het testen van je configuratie is het wel nuttig om nu even het wachtwoord opslaan aan te zetten, zodat we in dit dialoogvenster kunnen testen.

![img4](images/config-apache-directory04a.png)

Voer de gegevens in:
  * Bind DN or User: `cn=admin, dc=samenfit, dc=local`
  * Bind password : je wachtwoord dat je bij installatie gebruikt hebt.
  
Let op: bij de **Bind DN**  moet je de domein-naam die je gekozen hebt tijdens de installatie gebruiken. Stel, je kiest voor de domeinnaam 'bedienopafstand.local' dan wordt de Bind DN:
  * `cn=admin, dc=bedienopafstand, dc=local`

Druk vervolgens op de knop 'Check authentication'. Je krijgt onderstaande bevestiging:
![img5](images/config-apache-directory04b.png)

Druk op `Next` om naar de volgende stap te gaan. 

We gaan nu de DIT (Directory Information Tree) uitvragen: welke basis DN's zitten er in de root van de tree? 

![img6](images/config-apache-directory05.png)

Klik in bovenstaande afbeelding op de knop `Fetch BaseDN's`. De dropdown-lijst links van deze button wordt nu ververst. 

![img7](images/config-apache-directory06.png)

Zet daarna het vinkje uit bij `Get Base DN's from Root DSE` en kies in de dropdown-lijst je domeinnaam. 

Ga naar het volgende stap in de wizard door op `Next` te drukken.

Vervolgens moet je de edit-options instellen (*nieuw* sinds versie 2021 van Apache LDAP Studio in combinatie met OpenLDAP op Debian):

Zet de `Modify mode` op `Always Replace`. zie onderstaande afbeelding. 

![img8](images/config-apache-directory08.png)

Druk `Finish` om de wizard af te ronden. Je komt dan in onderstaande scherm uit:

![img8](images/config-apache-directory07.png)


## Check op geinstalleerde schema's
Om later gebruikers op te kunnen voeren, maken we gebruik van onderstaande schema's:
  * ObjectClass = inetOrgPerson
  * ObjectClass = organizationalPerson
  * ObjectClass = person
  * ObjectClass = top
  
Als het goed is zijn deze al geinstalleerd. Je kunt dit checken in de Schema Browser: 
`LDAP Directory Studio --> Menu-->LDAP-->Open Schema Browser`

![schema browser](images/config-ldap-schema01.png)

Type bij de zoekbalk `inetOrgPerson` in en je ziet als het goed is het schema in de zoekresultaten verschijnen. 

Voor het maken van groepen maken we gebruik van een `ObjectClass = groupOfUniqueNames`.  Check dat deze bestaat.
