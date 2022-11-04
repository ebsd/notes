---
layout: post
title: "nas maison raspberry pi 4 quad sata hat"
date:   2020-04-11 17:26:23 +0100
---
En complément de mon serveur VPS en ligne, j'ai récemment acquis un **Raspberry Pi 4** et un **[Quad Sata Hat](https://wiki.radxa.com/Dual_Quad_SATA_HAT)**. Ce dernier permet de connecter jusqu'à 4 disques SATA 2,5p au Raspberry Pi. Parfait pour un RAID qui sécurise mes données. Celles-ci sont synchronisées entre le VPS et le Pi via [Syncthing](https://syncthing.net). Je dispose ensuite de backups incémentaux effectués sur les disques du RAID grâce à rsync. Voici quelques infos pour la mise en route du **Quad Sata Hat** et d'un RAID1 logiciel sous Raspbian. Pour l'exercice, nous devons créer ce RAID mirroir sur 2 disques **sans perdre les données déjà présentes** sur un des deux disques. Voici une modeste représentation visuelle de mon installation. 
```
    <------+  CLOUD  +----->             <---+ HOME +--->
    rclone         ___                       ___
    crypt         +===+     Syncthing       +===+
    +-----------+ |VPS|  <---------------+> |Pi |
    |             +___+                  |  +___+
    |               ^                    |
    |               | syncthing          |  ++ ++ 2 HDD    ++ ++ 2 HDD
    v               |                    +> || || RAID1    || || RAID1
    gdrive          v                       ++ ++          ++ ++
                  Laptop                      +   rsnapshot  ^
                                              +--------------+
```
Le Quad Sata Hat est disponible [ici](https://shop.allnetchina.cn/products/dual-sata-hat-open-frame-for-raspberry-pi-4).

#### Installation du Quad Sata HAT sur raspbian
Script d'installation :

	curl -sL https://rock.sh/get-rockpi-sata | sudo -E bash -
On obtient un nouveau dossier :

	/usr/bin/rockpi-sata

Et un fichier de configuration, voir détails plus bas [1] 

	/etc/rockpi-sata.conf

#### Comment créer un RAID1 dont 1 des 2 disques contient déjà mes donneés ?
Je dispose de deux disques :

- /dev/sda = disque contenant mes données
- /dev/sdb = nouveau disque vièrge

On installe les outils de gestion du RAID:
	
	apt-get install mdadm

On démonte les disques :

	umount /mnt/disk1
	umount /mnt/disk2

On utilise sfdisk pour dupliquer la configuration des partitions du disque sda vers le disque sdb.

	sfdisk -d /dev/sda | sfdisk /dev/sdb

Utilisons maintenant `mdadm` pour créer un RAID1. On marque le premier disque sda comme "manquant" (missing) pour que les données du disque ne soient pas écrasées par la construction du RAID. Le second /dev/sdb1 est présent.

	mdadm --create /dev/md0 --level 1 --raid-devices=2 missing /dev/sdb1

On formatte le nouveau RAID :

	mkfs.ext4 /dev/md0

On monte le RAID :

	mkdir /mnt/raid1
	mount /dev/md0 /mnt/raid1

Copions le contenu du disque /dev/sda1 dans /dev/md0.

	mount /dev/sda1 /mnt/disk1
	rsync -rtavz --delete /mnt/disk1/ /mnt/raid1/

Démontons le disque que nous allons ensuite activer dans le RAID1.

	umount /mnt/disk1

Inrégrer le disque 1 au RAID

	mdadm --add /dev/md0 /dev/sda1

Le RAID va se construire, ce que nous pouvons suivre ainsi :

	cat /proc/mdstat


[1]: Fichier /etc/rockpi-sata.conf

    [fan]
    # When the temperature is above lv0 (35'C), the fan at 25% power,
    # and lv1 at 50% power, lv2 at 75% power, lv3 at 100% power.
    # When the temperature is below lv0, the fan is turned off.
    # You can change these values if necessary.
    lv0 = 35
    lv1 = 40
    lv2 = 45
    lv3 = 50
    [key]
    # You can customize the function of the key, currently available functions are
    # slider: oled display next page
    # switch: fan turn on/off switch
    # reboot, poweroff
    # If you have any good suggestions for key functions, 
    # please add an issue on https://rock.sh/rockpi-sata
    click = slider
    twice = switch
    press = none
    [time]
    # twice: maximum time between double clicking (seconds)
    # press: long press time (seconds)
    twice = 0.7
    press = 1.8
    [slider]
    # Whether the oled auto display next page and the time interval (seconds)
    auto = true
    time = 10
    [oled]
    # Whether rotate the text of oled 180 degrees, whether use Fahrenheit
    rotate = false
    f-temp = false	

Tags: raspberry, linux

## Restaurer les données du raid sur une autre machine

Brancher le disque en usb sur un laptop, puis :

	$ sudo mdadm --examine /dev/sdb1 
	/dev/sdb1:
	          Magic : a92b4efc
	        Version : 1.2
	    Feature Map : 0x1
	     Array UUID : 3c2a5eff:d7455954:d77b1813:e27e8d72
	           Name : raspberrypi:0
	  Creation Time : Sat Feb  8 15:34:52 2020
	     Raid Level : raid1
	   Raid Devices : 2
	
	 Avail Dev Size : 976506928 (465.63 GiB 499.97 GB)
	     Array Size : 488253440 (465.63 GiB 499.97 GB)
	  Used Dev Size : 976506880 (465.63 GiB 499.97 GB)
	    Data Offset : 264192 sectors
	   Super Offset : 8 sectors
	   Unused Space : before=264112 sectors, after=48 sectors
	          State : clean
	    Device UUID : 32d73af4:2948dafd:90cb7742:bfaa2b58
	
	Internal Bitmap : 8 sectors from superblock
	    Update Time : Fri Dec 17 20:57:07 2021
	  Bad Block Log : 512 entries available at offset 16 sectors
	       Checksum : 61023788 - correct
	         Events : 21788
	
	
	   Device Role : Active device 0
	   Array State : AA ('A' == active, '.' == missing, 'R' == replacing)

### Assemblage

Je tente avec cette commande qui échoue.

	$ sudo mdadm --assemble /dev/md0 /dev/sdb1
	mdadm: /dev/sdb1 is busy - skipping

Puis, assemblage automatique : Il existe déjà un RAID détecté dans /dev/md127.

	$ sudo mdadm --detail --scan
	INACTIVE-ARRAY /dev/md127 metadata=1.2 name=raspberrypi:0 UUID=3c2a5eff:d7455954:d77b1813:e27e8d72

	$ cat /proc/mdstat 
	Personalities : 
	md127 : inactive sdb1[2](S)
	      488253464 blocks super 1.2
       
	unused devices: <none>

Mais impossible de monter le raid.

	$ sudo mount /dev/md127 /mnt/nas/
	mount: /mnt/nas: can't read superblock on /dev/md127.

Donc je stoppe le raid /dev/md127 qui semble avoir été créé automatiquement. Et je lance une détection auto. Un nouveau /dev/md/raspberry devient dispo.

	$ sudo mdadm --stop /dev/md127
	mdadm: stopped /dev/md127
	$ sudo  mdadm --assemble --scan
	mdadm: /dev/md/raspberrypi:0 has been started with 1 drive (out of 2).

Et cette fois-ci je peux monter le raid :

	$ sudo mount /dev/md/raspberrypi\:0 /mnt/raid1

