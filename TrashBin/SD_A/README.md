#Guida per la configurazione della yun per renderla un bel client per plotly

Bene, eccoci.
Questa guida raccoglie tutti i passaggi per poter configurare l'arduino YÚNN, con i 5 sensori collegati.
L'idea di base è la seguente:

 - Il processore 32U4 presente sulla YÚN verrò fleshato con apposito codice e si occuperà di raccogliere i dati ogni X secondi e scriverli all'interno di due file:
    - **datalog.txt** dove verranno registrati per backup
    - **/collector/arduinoData** che rappresenta il buffer per la comunicazione a plot.ly

 - Il sistema operativo *openWrt* gestirà i rapporti tra i due file e plot.ly.



##Aggiorna il firmware della yun
Trovi qua la guida:

https://www.arduino.cc/en/Tutorial/YunSysupgrade

##Espandi lo spazio del sistema operativo sulla SD

https://www.arduino.cc/en/Tutorial/ExpandingYunDiskSpace

Se no, puoi collegarti in ssh e poi eseguire i seguenti comandi (per avere un disco da 1Gb):


      opkg update
      opkg install e2fsprogs mkdosfs fdisk rsync

      rm -rf /mnt/sda?
      umount /dev/sda?

      dd if=/dev/zero of=/dev/sda bs=4096 count=10

      (echo n; echo p; echo 1; echo; echo 1048576; echo w) | fdisk /dev/sda

      rm -rf /mnt/sda?
      umount /dev/sda?

      (echo n; echo p; echo 2; echo; echo; echo w) | fdisk /dev/sda

      rm -rf /mnt/sda?
      umount /dev/sda?

      (echo t; echo 1; echo c; echo w) | fdisk /dev/sda

      mkfs.vfat /dev/sda1
      mkfs.ext4 /dev/sda2

      mkdir -p /mnt/sda1
      mount /dev/sda1 /mnt/sda1
      mkdir -p /mnt/sda1/arduino/www

      rm -rf /mnt/sda?
      umount /dev/sda?

      mkdir -p /mnt/sda2
      mount /dev/sda2 /mnt/sda2
      rsync -a --exclude=/mnt/ --exclude=/www/sd /overlay/ /mnt/sda2/

      rm -rf /mnt/sda?
      umount /dev/sda?

      uci add fstab mount
      uci set fstab.@mount[0].target=/overlay
      uci set fstab.@mount[0].device=/dev/sda2
      uci set fstab.@mount[0].fstype=ext4
      uci set fstab.@mount[0].enabled=1
      uci set fstab.@mount[0].enabled_fsck=0
      uci set fstab.@mount[0].options=rw,sync,noatime,nodiratime
      uci commit

se ti servono altre info puoi guardare la guida su:

https://wiki.openwrt.org/doc/howto/extroot


##Installa la libreria di python per plotly

    opkg update
    opkg install python-openssl
    opkg install distribute
    opkg install subversion-client
    easy_install pip
    pip install plotly

##Clona tutti i file che ti servono sulla SD

    svn export https://github.com/paolocavagnolo/KitMonitorScuola/trunk/arduino/ --force
    svn export https://github.com/paolocavagnolo/KitMonitorScuola/trunk/linino/ --force


##Configura plotly

    python -c "import plotly; plotly.tools.set_credentials_file(username='techlabplot', api_key='2dehs2k3sw')"

cambia il nome della ClasseA, nei seguenti file:

- PRIMOplotterA.py
- plotterA.py

    python PRIMOplotterA.py

##Configura openwrt

Fai aggiornare automaticamente il software in 2 momenti distinti della giornata, alle 12.00 e alle 19.00

  crontab -e

aggiungi le seguenti righe:

  00 12 * * * date > /mnt/sda1/SD_A/log.log; python /mnt/sda1/SD_A/plotterA.py > /mnt/sda1/SD_A/log.log
  00 19 * * * date > /mnt/sda1/SD_A/log.log; python /mnt/sda1/SD_A/plotterA.py > /mnt/sda1/SD_A/log.log

attiva crontab:

  /etc/init.d/cron start
  /etc/init.d/cron enable
