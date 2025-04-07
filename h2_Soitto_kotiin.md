Laitteen nimi	Doombox
Suoritin	Intel(R) Core(TM) i7-14700KF   3.40 GHz
Asennettu RAM	32,0 Gt (31,8 Gt käytettävissä)
Tallennustila	466 GB SSD Samsung SSD 850 EVO 500GB, 1.82 TB SSD Samsung SSD 870 QVO 2TB, 224 GB SSD KINGSTON SV300S37A240G
Näytönohjain	NVIDIA GeForce RTX 3060 Ti (8 GB)
Laitetunnus	AC2BD1F2-D134-452F-9A63-10A23CCC3D9C
Tuotetunnus	00328-00271-12521-AA158
Järjestelmätyyppi	64-bittinen käyttöjärjestelmä, x64-suoritin
Kynä- ja kosketuslaitteet	Tässä näytössä ei ole käytettävissä kynä- tai kosketussyötetoimintoja
Versio	Windows 10 Education
Versio	2009
Asennuspäivä	‎22/‎06/‎2020
Käyttöjärjestelmän koontikäännös	19045.5679

# h2 Soitto kotiin

## x) lue ja tiivistä
-  Vagrant on ohjelma joka automaattisesti luo VirtualBox koneita
-  Se automatisoi SSH loggauksen
-  Graafista käyttöliittymää ei tarvita
-  Asennetaan Debianilla käyttämällä paketinhallintaa
-  $ sudo apt-get update
-  $ sudo apt-get install vagrant virtualbox
-  Macille tai windowsille täytyy ladata asennusohjelma netistä https://www.vagrantup.com/downloads
-  Luo uusi kansio projektillesi ja tallenna Vargrantfile sille
  
  ![image](https://github.com/user-attachments/assets/75f55165-8351-4fac-92bb-0dc9daec96e9)

-  uudelle koneelle kirjautuminen ssh:n kautta:    vagrant ssh t001
-  Poistuminen koneelta tapahtuu komennolla:     vagrant@t001$ exit
-  Luodut virtuaalikoneet pystyvät ottamaan yhteyden toisiinsa ja internettiin

![image](https://github.com/user-attachments/assets/62180e24-4166-40a1-ae5e-fa7d55c8456a)

-  Luodun virtuaalikoneen voi poistaa komennolla:  vagrant destroy
-  Uusi kone saadaan pystyyn komennolla:  vagrant up 

## salt quickstart
-  Salt ohjelman avulla voi kontrolloida koneita verkon läpi
-  Asennetaan pakentinhallinnan kautta: sudo apt-get -y install salt-master(tai minion)
-  Minion konelle annetaan masterin yhteystiedot:    slave$ sudoedit /etc/salt/minion
master: 10.0.0.88
id: tero
-   Käynnistetään minion kone uudelleen: sudo systemctl restart salt-minion.service
-   Sitten hyväksytään master koneella minionin avain
-   Sen jälkeen voidaan kokeilla eri komentoja, toimiiko hallinta
-   Luo init.sls tiedosto : $ sudo mkdir -p /srv/salt/hello $ sudoedit /srv/salt/hello/init.sls
-   Kirjoita init.sls tiedostoon: $ cat /srv/salt/hello/init.sls
  /tmp/infra-as-code:
  "tähän kaksi välilyöntiä" file.managed
  $ sudo salt '*' state.apply hello
-  Seuraavasti määritellään mitä tiloja käytetään slave koneille
![image](https://github.com/user-attachments/assets/aa251508-8075-41cd-b3cd-e370cf739b87)
-  Sitten ei tarvitse enään nimetä moduuleja state.apply:ssä

## a) Vagrant asennus windows 10:lle
Menin sivustolle https://developer.hashicorp.com/vagrant/install. Valitsin windows AMD64 asennuksen

![image](https://github.com/user-attachments/assets/bac691dc-e8f9-4fa9-8654-2d4e61c7ef5b)

Hyväksyin sopimuksen ja painoin install. VirtualBox minulla jo on, joten sitä ei tarvitse asentaa.
Asennuksen jälkeen kone piti käynnistää uudelleen.




