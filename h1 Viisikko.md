# h1 Viisikko

## x) Lue ja tiivistä 
-  Salt ohjelman avulla voit käyttää tietokoneita etänä ja myös kokeilla sen komentoja paikallisesti, jolloin voit testata komennot ennen käyttöön ottoa.
-  Salt komennot toimivat sekä linuxilla, että windowsilla.
-  Tärkeimmät tila funktiot ovat pkg(paketinhallinta), file service(tiedostojen hallinta), user(käyttäjien hallinta ja cmd(komentojen ajo)
-  Saltilla voit kontrolloida tuhansia tietokoneita yhtäaikaisesti verkon yli

## Salt quickstart
-  Ensin asennetaan salt master
-  Sitten asennetaan salt minion
-  Sen jälkeen pitää tehdä koodiavaimet, jolla masterkone voi hallinnoida minion koneita
-  Sitten vain testataan, että perustoiminnot onnistuvat.

## Raportin kirjoittaminen
-  Raportissa kerrotaan täsmällisesti mitä teit, ja mitä sitten tapahtui.
-  Raporttia kannattaa kirjoittaa samalla kun tekee työtä
-  Se on oltava toistettavissa jonkun muun toimesta, jos ympäristö on sama
-  Ole täsmällinen kun kirjoitat raporttia, kirjoita ylös kaikki antamasi komennot ja mitä tapahtui
-  Kun joku menee pieleen, tutki ja raportoi ongelmat täsmällisesti
-  Tee raportista helppolukuinen, käytä väliotsikoita ja huolellista kieltä
-  Muista viitata käyttämiisi lähteisiin
-  Älä plagioi tai sepitä

## Install Salt DEBS
-  Aja seuraavat komennot asentaaksesi Salt DEBs:
# Ensure keyrings dir exists
mkdir -p /etc/apt/keyrings
# Download public key
curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp
# Create apt repo target configuration
curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | sudo tee /etc/apt/sources.list.d/salt.sources 
-  Sitten voit asentaa salt master, minion, ssh, syndic, cloud, api -ohjelmat tarpeesi mukaan
-  Sen jälkeen voit ottaa asentamasi ohjelmat käyttöön:
sudo systemctl enable salt-master && sudo systemctl start salt-master
sudo systemctl enable salt-minion && sudo systemctl start salt-minion
sudo systemctl enable salt-syndic && sudo systemctl start salt-syndic
sudo systemctl enable salt-api && sudo systemctl start salt-api

# b) Asenna Salt minion
Ensin loin kansion avaimelle: mkdir -p /etc/apt/keyrings
Seuraavaksi hain avaimet Saltin palvelimelta ja aktivoin ne, Salt install guide:linux(DEB) mukaisesti.
Sitten asensin salt-minion ohjelman komennolla sudo apt-get install salt-minion ja ohjelma asentui virtuaalikoneelleni.

# c) Viisi tärkeintä
pkg.installed
![image](https://github.com/user-attachments/assets/6e78cd58-30a2-434a-82cc-9d5131576168)

Asentaa ohjelman kohde koneisiin jos sitä ei siellä vielä ole, jos ohjelma löytyy, Salt toteaa vain että operaatio onnistunut, mutta ei tee mitään kohdekoneelle.
Muita annettuja tietoja ovat ohjelman nimi ja ajettu funktio. Kommentti mitä tehtiin, koska operaatio alkoi ja kauanko se kesti. 
Mitä muutoksia tehtiin, mikä versio ohjelmasta on nyt asennettu ja summaryssä kerrotaan onnistuiko haluttu operaatio ja tehtiinkö muutoksia kohdekoneeseen.

File.managed
Suoritin komennon sudo salt-call --local -l info state.single file.managed /tmp/moitero contents="foo"

![image](https://github.com/user-attachments/assets/86aa1d28-8ff8-4833-a7e6-aee35af94e18)

Salt teki tiedoston ja sille sisällön onnistuneesti

service.running
Seuraavaksi sammutetaan apache2 palvelu komennolla sudo salt-call --local -l info state.single service.dead apache2 enable=False

![image](https://github.com/user-attachments/assets/64469e08-b24e-4e89-bb95-2a16c8c45649)

ja näin apache2 kuoli :(

user.present
Tämä luo uuden käyttäjän: sudo salt-call --local -l info state.single user.present Urpo
![image](https://github.com/user-attachments/assets/a6308a35-49d9-4b6b-a333-afad317270f3)

![image](https://github.com/user-attachments/assets/86d86596-183d-4a61-bace-d5145c803ba4)

näin uusi käyttäjä on luotu

cmd.run
Tämän avulla voit ajaa komennon toiseen koneeseen, esim: sudo salt-call --local -l info state.single cmd.run 'touch /tmp/wtf' creates="/tmp/wtf"

![image](https://github.com/user-attachments/assets/aadf3194-7b16-435f-a26a-193cb066471c)

komennon jälkimmäinen osio, creates="/tmp/wtf" luo idempotenssin ja estää komennon toteuttamisen jos se on jo toteutettu. 

![image](https://github.com/user-attachments/assets/4780a86c-cefa-41cc-a9dc-b6ae33823cdf)

tiedosto on luotu määriteltyyn paikkaan.


# d) Idempotentti

![image](https://github.com/user-attachments/assets/6e78cd58-30a2-434a-82cc-9d5131576168)

Asensin tree ohjelman saltin avulla. kuvassa näkyy ohjelman mikä ohjelma asennettiin ja mikä versio. 
summary kentässä 1 onnistunut toiminto, jossa yksi muutos koneelle tapahtunut, kun ajan komennon uudelleen pitäisi tulla yksi onnistunut operaatio, mutta ei muutosta koneelle.

![image](https://github.com/user-attachments/assets/61e4ac1f-7f31-4ce4-b3a5-7fccc8829aab)

Näin myös kävi, eli Saltin idempotentti toimii kuten pitääkin, eli muutoksia tehdään vain jos koneen tila on halutusta tilasta poikkeava, muutoin vain todetaan operaatio onnistuneeksi.




