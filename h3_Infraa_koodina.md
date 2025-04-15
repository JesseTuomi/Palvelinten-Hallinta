Laitteen nimi Doombox Suoritin Intel(R) Core(TM) i7-14700KF 3.40 GHz Asennettu RAM 32,0 Gt (31,8 Gt käytettävissä) Tallennustila 466 GB SSD Samsung SSD 850 EVO 500GB, 1.82 TB SSD Samsung SSD 870 QVO 2TB, 224 GB SSD KINGSTON SV300S37A240G Näytönohjain NVIDIA GeForce RTX 3060 Ti (8 GB) Laitetunnus AC2BD1F2-D134-452F-9A63-10A23CCC3D9C Tuotetunnus 00328-00271-12521-AA158 Järjestelmätyyppi 64-bittinen käyttöjärjestelmä, x64-suoritin Kynä- ja kosketuslaitteet Tässä näytössä ei ole käytettävissä kynä- tai kosketussyötetoimintoja Versio Windows 10 Education Versio 2009 Asennuspäivä ‎22/‎06/‎2020 Käyttöjärjestelmän koontikäännös 19045.5679

# h3 Infraa Koodina

## x) lue ja tiivistä

### hello salt Infra as a code

-  Saltilla voi ohjata tuhansia tietokoneita
-  Modulet asentavat, konfiguroivat ja käynnistävät asioita
-  esim:
```debian
$ sudo mkdir -p /srv/salt/hello/
$ cd /srv/salt/hello/
```
-  Kun teet infraa koodina, varmista että olet oikeassa kansiossa
-  Sitten tehdään .sls tiedosto. joka sisältää halutut komennot sudoedit init.sls
```esim
/tmp/hellotero:
  file.managed
sitten ajetaan
$ sudo salt-call --local state.apply hello
```

### Salt overview
-  YAML on markdownin käyttämä renderöijä, se kääntää YAML datan python dataksi Saltille
-  Data rakenne on key: value pareina
-  Isot ja pienet kirjaimet ovat merkityksellisiä
-  Älä käytä tabulaattoiria, vain välilyöntiä
-  Kommenttikentät alkavat merkillä "#"
-  YAML koostuu kolmesta perus tyypistä:
```
Scalar

# key: value

vegetables: peas
fruit: apples
grains: bread
Lists

# sequence_key:
#  - value1
#  - value2

vegetables:
   - peas
   - carrots
fruits:
   - apples
   - oranges
Dictionary

dinner:
  appetizer: shrimp cocktail
  drink: sparkling water
  entree:
    - steak
    - mashed potatoes
    - dinner roll
  dessert:
    - chocolate cake
```

## a) Hei infrakoodi!

Lähdin tekemään ohjeen mukaan:

```
sudo mkdir -p /srv/salt/hello
sudoedit init.sls
/tmp/hellojesse:
  file.managed
sudo salt-call --local state.apply hello
local:
----------
          ID: /tmp/hellojesse
    Function: file.managed
      Result: True
     Comment: Empty file
     Started: 11:52:07.709052
    Duration: 5.564 ms
     Changes:   
              ----------
              new:
                  file /tmp/hellojesse created

Summary for local
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:   5.564 ms
```
Sitten tarkistin, että tiedosto löytyy kansiosta
```
jesset@jesse-virtualbox:/srv/salt/hello$ cd /tmp
jesset@jesse-virtualbox:/tmp$ ls
hellojesse
```

## b) 
Laitoin aluksi pystyyn kaksi virtuaali konetta vagrantilla ja asensin edellisen tehtävän mukaisesti salt-master salt-minion parin.
Sitten loin kansion /srv/salt/hello, johon tein tuon init.sls komennon 
sen jälkeen ajoin tehtävän:
```
vagrant@t001:/srv/salt/hello$ vagrant@t001:/srv/salt/hello$ sudo salt "*" state.apply hello
willy:
----------
          ID: /tmp/hellojesse
    Function: file.managed
      Result: True
     Comment: Empty file
     Started: 09:27:50.110830
    Duration: 3.526 ms
     Changes:
              ----------
              new:
                  file /tmp/hellojesse created

Summary for willy
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:   3.526 ms
```
Nyt orjalle pitäisi olla muodostunut tiedosto
```
vagrant@t002:~$ cd /tmp
vagrant@t002:/tmp$ ls
hellojesse  s
```
Ja siellähän se oli:

## c) Tee kahden tilafunktion SLS-tiedosto

Tein masterille uuden kansion /srv/salt/work
Tein sinne init.sls kansion johon laitoin kaksi komentoa, yksi asentaa apache2 ohjelman ja toinen luo työkansion nimeltä /workspace
```
apache2-install:
  pkg.installed:
    - name: apache2

workspace-directory:
  file.directory:
    - name: /workspace
```
sitten ajoin komennot: 
```
apache2-install:
  pkg.installed:
    - name: apache2

workspace-directory:
  file.directory:
    - name: /workspace
```
Lopuksi tarkistamme orjalta onko kaikki tehty kuten pitää.
```
vagrant@t002:~$ cd /workspace
vagrant@t002:/workspace$
```
Kansio löytyy orjalta, sitten vielä katsotaan onko apache2 orjalla:
```
vagrant@t002:/$ dpkg -l | grep apache2
ii  apache2                       2.4.62-1~deb12u2               amd64        Apache HTTP Server
ii  apache2-bin                   2.4.62-1~deb12u2               amd64        Apache HTTP Server (modules and other binary files)
ii  apache2-data                  2.4.62-1~deb12u2               all          Apache HTTP Server (common files)
ii  apache2-utils                 2.4.62-1~deb12u2               amd64        Apache HTTP Server (utility programs for web servers)
vagrant@t002:/$
```
Sekin löytyy, eli asennus onnistunut.

## LÄHTEET

https://terokarvinen.com/2024/hello-salt-infra-as-code/
https://docs.saltproject.io/salt/user-guide/en/latest/topics/overview.html#rules-of-yaml




