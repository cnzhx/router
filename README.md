# openSUSE Router for classrooms
This desribes how to build a router from scratch on a openSUSE Leap 42.2 system.

Meer Nederlandstalige informatie bij [openSUSE Router op PindaNet.be](https://linux.pindanet.be/faq/tips16/router.html).
## Hardware
### Systeem
* HP EliteDesk 800 G1 SFF 
* VMWare virtuele computers (testsysteem)
  VMware Player in vmx configuratiebestand:

        ethernet1.connectionType = "pvn"
        ethernet1.pvnID = "52 dd bc d5 36 19 1b 6b-0f f1 fb 1c 4c ac 44 f7"
        firmware = "efi"

### Netwerk
| Verbinding | Netwerkkaart                                              | Apparaat | Functie |
|------------|-----------------------------------------------------------|----------|---------|
| Onboard 	 | Ethernet Connection I217-LM					             | eth0     | WAN
| PCIE x1 	 | RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller | eth1		| LAN
| Wireless	 |							                                 | wlan0    | Access Point
### Schijfbeheer
| Partitie | Grootte | Bestandssyteem | Koppelpunt |
|----------|--------:|:--------------:|------------|
|/dev/sda1 |  1 GB |	          FAT |	/boot/efi  |
|/dev/sda2 |	8 GB |	         swap ||
|/dev/sda3 |  111 GB |	         Ext4 |	/  | 
|||||
|/dev/sdb1 |  100 GB |	         Ext4 |	/var/backup
|/dev/sdb2 |  832 GB |           Ext4 |	/usr/home/Documents
## Software
### OpenSuSE Leap 15.0
* Opstarten met installatie x86_64 DVD  
* Esc  
* Installation  
* Language: Dutch - Nederlands  
  * Toetsenbordindeling > Belgisch  
* Gebruikersinterface: Server
* Partitionering door expert...
  * Begin met bestaande partities
  * Apparaten opnieuw scannen  
  * Vaste schijven > sda > sda1 > Bewerken  
    * Aankoppelpunt: /boot/efi > Voltooien  
  * Vaste schijven > sda > sda2 > Bewerken  
    * Formatteer de partitie.  
    * Swap  
    * Aankoppelpunt: swap > Voltooien  
  * Vaste schijven > sda > sda3 > Bewerken
    * Formatteer de partitie.  
    * Ext4  
    * Aankoppelpunt: / > Voltooien  
  * Vaste schijven > sdb > sdb1 > Bewerken  
    * Formatteer de partitie.  
    * Ext4  
    * Aankoppelpunt: /var/backup > Voltooien  
  * Vaste schijven > sdb > sdb2 > Bewerken  
    * Formatteer de partitie.  
    * Ext4  
    * Aankoppelpunt: /usr/home/Documents > Voltooien  
  * Accepteren > Volgende  
* Tijdzone:
  * Hardwareklok instellen op UTC inschakelen
  * Tijdzone: België
  * Andere instellingen...
    * Met NTP-server synchroniseren
    * Adres van de NTP-server: be.pool.ntp.org > Accepteren > Volgende
* Nieuwe gebruiker aanmaken
  * SNT Beheerder
  * sntbeheerder
  * snt+4567
  * snt+4567
  * Volgende
* Installatie instellingen:
  * Opstarten > Bootloader: Niet beheerd > Doorgaan > OK
  * Firewall inshakelen, SSH-poort openen en SSH-service inschakelen
  * Installeren > Installeren
* Computer laten herstarten zonder installatie DVD
* Aanmelden als root

          zypper up  
          shutdown -r now

### Backupsysteem
* Opstarten met SystemRescueCD vanaf CD

        mount /dev/sda1 /mnt/custom
        mount -o remount,rw /sys/firmware/efi/efivars
        grub-install --target=x86_64-efi --efi-directory=/mnt/custom --boot-directory=/mnt/custom --bootloader-id=grub --recheck /dev/sda
        mkdir /mnt/custom/sysrcd
        cp /livemnt/boot/{sysrcd.dat,sysrcd.md5} /mnt/custom/sysrcd/
        cp /livemnt/boot/???linux/{initram.igz,rescue64} /mnt/custom/sysrcd/
        wget -P /mnt/custom/grub/locale/ https://raw.githubusercontent.com/pindanet/router/master/clients/nl.mo
        wget -P /mnt/custom/grub/ https://raw.githubusercontent.com/pindanet/router/master/grub.cfg
        wget -P /mnt/custom https://raw.githubusercontent.com/pindanet/router/master/autorun

### Installatie
Zorg voor een actieve LAN verbinding (kabel naar ingeschakelde switch of computer)

    wget https://raw.githubusercontent.com/pindanet/router/master/router.setup.sh
    sh router.setup.sh router
    sh router.setup.sh services

### Updaten enz.
* Raspian repository

      rsync -avHxh --numeric-ids --progress --delete archive.raspbian.org::archive/raspbian/ /usr/home/Documents/raspbian/raspbian.org
* Info

        cat /etc/resolv.conf
        sudo route
* Backuppartitie koppelen vanaf SystemRescueCD/USB:

        joe /mnt/backup/autorun
* Backuppartitie koppelen vanaf geïnstalleerd backupsysteem:

        mount /livemnt/boot -o rw,remount
* Backup maken

        partclone.extfs --clone --source /dev/sda3 | gzip -c > /mnt/backup/child.linux.partclone.gz
* Update router via ssh als root

        zypper update
* Update backupsysteem via linux systeem

        sudo mount Downloads/systemrescuecd-x86-4.9.0.iso /mnt/
  * Update backupsysteem via fish van linux systeem naar router als root
    * kopieer /mnt/isolinux/{initram.igz,rescue64} /var/backup/sysrcd/
    * kopieer /mnt/{sysrcd.dat,sysrcd.md5} /var/backup/sysrcd/
    * kopieer /mnt/isolinux/{initram.igz,rescue64} /srv/www/htdocs/tftpboot/
    * kopieer /mnt/{sysrcd.dat,sysrcd.md5} /srv/www/htdocs/tftpboot/

            sudo umount /mnt/
* Update Windows installatiebestanden via fish van linux systeem naar router als root
  * kopieer Documenten/Software/SNT/Windows.iso /usr/home/Documents/
  * kopieer Documenten/SNT/Thuisnetwerken/WinPE_amd64.iso /usr/home/Documents/
  * kopieer Downloads/HP Support Assistant.exe /usr/home/Documents/
# ToDo
* Firewall: Pas de Interface zone aan naar Interne zone
* Netdata
* Realtec hostapd
  * https://nikunjlahoti.wordpress.com/2016/09/07/run-your-wifi-dongle-as-access-point-soft-ap-8188eu-on-linux/
  * http://billauer.co.il/blog/2014/06/linux-realtek-hostapd/
* https://opensource.com/business/16/2/creating-disk-images-with-fog
