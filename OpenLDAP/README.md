# OpenLDAP installatie en configuratie.
Het product `OpenLDAP` geeft ons de mogelijkheid om gebruikers veilig te administreren. Deze gebruikers kunnen we dan 
autoriseren voor bijvoorbeeld onze webserver. We gebruiken hier (minimaal) versie 2.4.

Zorg dat je bent ingelogd via `root` of een gebruiker met `sudo`-rechten. Onderstaande gaat uit van de laatste (een normale)
gebruiker die via `sudo` tijdelijk verhoogde rechten krijgt. 

De software die daadwerkelijk geinstalleerd staat is [slapd](https://www.openldap.org/doc/admin24/runningslapd.html).

## Testen OpenLDAP
Om wat testen te kunnen doen kunnen we zowel bash-commando's gebruiken, óf een GUI. 
In het eerste geval gebruik je de command-line van je VM met onderstaande commando's: 

```bash
$ ldapsearch -s sub -x -b "" -s base supportedfeatures
# extended LDIF
#
# LDAPv3
# base <> with scope baseObject
# filter: (objectclass=*)
# requesting: supportedfeatures 
#

#
dn:
supportedFeatures: 1.3.6.1.1.14
supportedFeatures: 1.3.6.1.4.1.4203.1.5.1
supportedFeatures: 1.3.6.1.4.1.4203.1.5.2
supportedFeatures: 1.3.6.1.4.1.4203.1.5.3
supportedFeatures: 1.3.6.1.4.1.4203.1.5.4
supportedFeatures: 1.3.6.1.4.1.4203.1.5.5

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1

```
