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
Käyttöjärjestelmän koontikäännös	19045.5737

# h4 Pkg file service

## x) Lue ja tiivistä
-  Voit hallinnoida suurta määrää palvelimia cofiguration management system:llä
-  Luo master koneelle sshd.sls ja kopio konfiguraation tiedostosta
``` $ cat /srv/salt/sshd.sls
openssh-server:
 pkg.installed
/etc/ssh/sshd_config:
 file.managed:
   - source: salt://sshd_config
sshd:
 service.running:
   - watch:
     - file: /etc/ssh/sshd_config
```
-  Vaihda portin numero sshd_config tiedostoon
-  Laita tila päälle verkon yli orja koneille: sudo salt '*' state.apply sshd
-  Kokeile toimiiko: nc -vz tero.example.com 8888
-  Tai vaihtoehtoisesti: ssh -p 8888 tero@tero.example.com

## a) apache easy mode
Ensin taas vagrant up ja salt asennukset koneille.
Kokeillaan ensin tehdä apache asennus käsin:
sudo salt "t002" pkg.install apache2

![image](https://github.com/user-attachments/assets/6d98f82b-015c-4232-8e26-da3ae8142729)

sitten vaihdetaan testisivun sisältö: 
sudo salt 't002' cmd.run 'echo "Tämä on testisivu!" > /var/www/html/index.html'
t002:
    Tämä on testisivu!
    kokeillaan myös url käyttäen
    ![image](https://github.com/user-attachments/assets/9615ea75-e3fa-4ab4-9d93-66ef08feb496)

sitten poistetaan asennus orjakoneelta: sudo salt 't002' pkg.remove apache2
```vagrant@t001:~$ sudo salt 't002' pkg.remove apache2
t002:
    ----------
    apache2:
        ----------
        new:
        old:
            2.4.62-1~deb12u2
```
Seuraavaksi tehdään homma sls-tiedoston avulla

luodaan masterille salt-tila tiedosto : micro /srv/salt/ssh/apache_inst.sls
```
apache_installed:
  pkg.installed:
    - name: apache2

apache_running:
  service.running:
    - name: apache2
    - enable: True
    - require:
      - pkg: apache_installed

index_file:
  file.managed:
    - name: /var/www/html/index.html
    - contents: |
        <html>
        <head><title>Testisivu</title></head>
        <body><h1>Saltin kautta luotu testisivu!</h1></body>
        </html>
    - require:
      - service: apache_running
```
Sitten suoritetaan master koneella: sudo salt 't002' state.apply ssh.apache_inst
```
vagrant@t001:/srv/salt/ssh$ sudo salt 't002' state.apply ssh.apache_inst
t002:
----------
          ID: apache_installed
    Function: pkg.installed
        Name: apache2
      Result: True
     Comment: The following packages were installed/updated: apache2
     Started: 13:32:32.674787
    Duration: 2639.016 ms
     Changes:
              ----------
              apache2:
                  ----------
                  new:
                      2.4.62-1~deb12u2
                  old:
----------
          ID: apache_running
    Function: service.running
        Name: apache2
      Result: True
     Comment: The service apache2 is already running
     Started: 13:32:35.319977
    Duration: 14.732 ms
     Changes:
----------
          ID: index_file
    Function: file.managed
        Name: /var/www/html/index.html
      Result: True
     Comment: File /var/www/html/index.html updated
     Started: 13:32:35.336034
    Duration: 4.359 ms
     Changes:
              ----------
              diff:
                  ---
                  +++
                  @@ -1 +1,4 @@
                  -Tämä on testisivu!
                  +<html>
                  +<head><title>Testisivu</title></head>
                  +<body><h1>Saltin kautta luotu testisivu!</h1></body>
                  +</html>

Summary for t002
------------
Succeeded: 3 (changed=2)
Failed:    0
------------
Total states run:     3
Total run time:   2.658 s
```
testataan urlilla
![image](https://github.com/user-attachments/assets/3ff56a5f-440f-47ba-b994-cc66e6478ad5)

## b)  SSHouto
Ensin tehdään masterille sshd.sls state file: Sudo nano /srv/salt/ssh/sshd.sls
```
openssh-server:
 pkg.installed
/etc/ssh/sshd_config:
 file.managed:
   - source: salt://sshd_config
sshd:
 service.running:
   - watch:
     - file: /etc/ssh/sshd_config
```
Sitten tehdään sshd_config file:
```
Port 8888
Port 22
Protocol 2
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_dsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key
UsePrivilegeSeparation yes
KeyRegenerationInterval 3600
ServerKeyBits 1024
SyslogFacility AUTH
LogLevel INFO
LoginGraceTime 120
PermitRootLogin prohibit-password
StrictModes yes
RSAAuthentication yes
PubkeyAuthentication yes
IgnoreRhosts yes
RhostsRSAAuthentication no
HostbasedAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no
X11Forwarding yes
X11DisplayOffset 10
PrintMotd no
PrintLastLog yes
TCPKeepAlive yes
AcceptEnv LANG LC_*
Subsystem sftp /usr/lib/openssh/sftp-server
UsePAM yes
```
Ajetaan tila minionille:  sudo salt '*' state.apply ssh.sshd
```
t002:
----------
          ID: openssh-server
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 14:21:52.914059
    Duration: 8.533 ms
     Changes:
----------
          ID: /etc/ssh/sshd_config
    Function: file.managed
      Result: True
     Comment: File /etc/ssh/sshd_config updated
     Started: 14:21:52.923297
    Duration: 11.961 ms
     Changes:
              ----------
              diff:
                  ---
                  +++
                  @@ -1,126 +1,30 @@
                  -
                  -# This is the sshd server system-wide configuration file.  See
                  -# sshd_config(5) for more information.
                  -
                  -# This sshd was compiled with PATH=/usr/local/bin:/usr/bin:/bin:/usr/games
                  -
                  -# The strategy used for options in the default sshd_config shipped with
                  -# OpenSSH is to specify options with their default value where
                  -# possible, but leave them commented.  Uncommented options override the
                  -# default value.
                  -
                  -Include /etc/ssh/sshd_config.d/*.conf
                  -
                  -#Port 22
                  -#AddressFamily any
                  -#ListenAddress 0.0.0.0
                  -#ListenAddress ::
                  -
                  -#HostKey /etc/ssh/ssh_host_rsa_key
                  -#HostKey /etc/ssh/ssh_host_ecdsa_key
                  -#HostKey /etc/ssh/ssh_host_ed25519_key
                  -
                  -# Ciphers and keying
                  -#RekeyLimit default none
                  -
                  -# Logging
                  -#SyslogFacility AUTH
                  -#LogLevel INFO
                  -
                  -# Authentication:
                  -
                  -#LoginGraceTime 2m
                  -#PermitRootLogin prohibit-password
                  -#StrictModes yes
                  -#MaxAuthTries 6
                  -#MaxSessions 10
                  -
                  -#PubkeyAuthentication yes
                  -
                  -# Expect .ssh/authorized_keys2 to be disregarded by default in future.
                  -#AuthorizedKeysFile  .ssh/authorized_keys .ssh/authorized_keys2
                  -
                  -#AuthorizedPrincipalsFile none
                  -
                  -#AuthorizedKeysCommand none
                  -#AuthorizedKeysCommandUser nobody
                  -
                  -# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
                  -#HostbasedAuthentication no
                  -# Change to yes if you don't trust ~/.ssh/known_hosts for
                  -# HostbasedAuthentication
                  -#IgnoreUserKnownHosts no
                  -# Don't read the user's ~/.rhosts and ~/.shosts files
                  -#IgnoreRhosts yes
                  -
                  -# To disable tunneled clear text passwords, change to no here!
                  -PasswordAuthentication no
                  -#PermitEmptyPasswords no
                  -
                  -# Change to yes to enable challenge-response passwords (beware issues with
                  -# some PAM modules and threads)
                  -KbdInteractiveAuthentication no
                  -
                  -# Kerberos options
                  -#KerberosAuthentication no
                  -#KerberosOrLocalPasswd yes
                  -#KerberosTicketCleanup yes
                  -#KerberosGetAFSToken no
                  -
                  -# GSSAPI options
                  -#GSSAPIAuthentication no
                  -#GSSAPICleanupCredentials yes
                  -#GSSAPIStrictAcceptorCheck yes
                  -#GSSAPIKeyExchange no
                  -
                  -# Set this to 'yes' to enable PAM authentication, account processing,
                  -# and session processing. If this is enabled, PAM authentication will
                  -# be allowed through the KbdInteractiveAuthentication and
                  -# PasswordAuthentication.  Depending on your PAM configuration,
                  -# PAM authentication via KbdInteractiveAuthentication may bypass
                  -# the setting of "PermitRootLogin prohibit-password".
                  -# If you just want the PAM account and session checks to run without
                  -# PAM authentication, then enable this but set PasswordAuthentication
                  -# and KbdInteractiveAuthentication to 'no'.
                  +Port 8888
                  +Port 22
                  +Protocol 2
                  +HostKey /etc/ssh/ssh_host_rsa_key
                  +HostKey /etc/ssh/ssh_host_dsa_key
                  +HostKey /etc/ssh/ssh_host_ecdsa_key
                  +HostKey /etc/ssh/ssh_host_ed25519_key
                  +UsePrivilegeSeparation yes
                  +KeyRegenerationInterval 3600
                  +ServerKeyBits 1024
                  +SyslogFacility AUTH
                  +LogLevel INFO
                  +LoginGraceTime 120
                  +PermitRootLogin prohibit-password
                  +StrictModes yes
                  +RSAAuthentication yes
                  +PubkeyAuthentication yes
                  +IgnoreRhosts yes
                  +RhostsRSAAuthentication no
                  +HostbasedAuthentication no
                  +PermitEmptyPasswords no
                  +ChallengeResponseAuthentication no
                  +X11Forwarding yes
                  +X11DisplayOffset 10
                  +PrintMotd no
                  +PrintLastLog yes
                  +TCPKeepAlive yes
                  +AcceptEnv LANG LC_*
                  +Subsystem sftp /usr/lib/openssh/sftp-server
                   UsePAM yes
                  -
                  -#AllowAgentForwarding yes
                  -#AllowTcpForwarding yes
                  -#GatewayPorts no
                  -X11Forwarding yes
                  -#X11DisplayOffset 10
                  -#X11UseLocalhost yes
                  -#PermitTTY yes
                  -PrintMotd no
                  -#PrintLastLog yes
                  -#TCPKeepAlive yes
                  -#PermitUserEnvironment no
                  -#Compression delayed
                  -#ClientAliveInterval 0
                  -#ClientAliveCountMax 3
                  -#UseDNS no
                  -#PidFile /run/sshd.pid
                  -#MaxStartups 10:30:100
                  -#PermitTunnel no
                  -#ChrootDirectory none
                  -#VersionAddendum none
                  -
                  -# no default banner path
                  -#Banner none
                  -
                  -# Allow client to pass locale environment variables
                  -AcceptEnv LANG LC_*
                  -
                  -# override default of no subsystems
                  -Subsystem    sftp    /usr/lib/openssh/sftp-server
                  -
                  -# Example of overriding settings on a per-user basis
                  -#Match User anoncvs
                  -#    X11Forwarding no
                  -#    AllowTcpForwarding no
                  -#    PermitTTY no
                  -#    ForceCommand cvs server
                  -ClientAliveInterval 120
                  -
                  -# debian vagrant box speedup
                  -UseDNS no
----------
          ID: sshd
    Function: service.running
      Result: True
     Comment: Service restarted
     Started: 14:21:52.955224
    Duration: 48.782 ms
     Changes:
              ----------
              sshd:
                  True

Summary for t002
------------
Succeeded: 3 (changed=2)
Failed:    0
------------
Total states run:     3
Total run time:  69.276 ms
```

Testataan toimiiko sshd: sudo systemctl status sshd
```
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; preset: enabled)
    Drop-In: /etc/systemd/system/ssh.service.d
             └─generate-ssh-keys.conf
     Active: active (running) since Mon 2025-04-21 14:09:22 UTC; 21min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 10757 (sshd)
      Tasks: 1 (limit: 496)
     Memory: 1.4M
        CPU: 20ms
     CGroup: /system.slice/ssh.service
             └─10757 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Apr 21 14:09:22 t001 systemd[1]: Starting ssh.service - OpenBSD Secure Shell server...
Apr 21 14:09:22 t001 sshd[10757]: Server listening on 0.0.0.0 port 22.
Apr 21 14:09:22 t001 sshd[10757]: Server listening on :: port 22.
Apr 21 14:09:22 t001 systemd[1]: Started ssh.service - OpenBSD Secure Shell server.
```

Sitten kokeilin yhteyttä ohjeen mukaan: nc -p t002 8888
´´´
t002: forward host lookup failed: Unknown host
´´´
ei löydä, kokeilin urlilla: nc -vz 192.168.88.102 8888
```
t002 [192.168.88.102] 8888 (?) open
´´´
Yhteys toimii mutta, jokin on pielessä.







