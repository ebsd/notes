---
layout: post
title: "La sante de ton hdd est primordiale"
date: 2020-05-01 18:24:49 +0100
---
Comment tester l'état de santé de ton précieux disque ? Les smartmontools sont là pour ça !

#### Où est ton disque ?
	$ lsblk
	NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
	sda      8:0    0 223.6G  0 disk 
	├─sda1   8:1    0   300M  0 part /boot/efi
	└─sda2   8:2    0 223.3G  0 part /
	sdb      8:16   0 465.8G  0 disk 
	└─sdb1   8:17   0 465.8G  0 part /run/media/ebsd/8e054110-80aa-4857-8344-84b3c460051e

#### Vérifie que le disque supporte SMART et qu'il est actif
	$ smartctl -i /dev/sdb
	=== START OF INFORMATION SECTION ===
	Device Model:     TOSHIBA MQ01ACF050
	Serial Number:    Y64IT95UT
	LU WWN Device Id: 5 000039 761881438
	Firmware Version: AV002D
	User Capacity:    500,107,862,016 bytes [500 GB]
	Sector Sizes:     512 bytes logical, 4096 bytes physical
	Rotation Rate:    7200 rpm
	Form Factor:      2.5 inches
	Device is:        Not in smartctl database [for details use: -P showall]
	ATA Version is:   ATA8-ACS (minor revision not indicated)
	SATA Version is:  SATA 3.0, 6.0 Gb/s (current: 6.0 Gb/s)
	Local Time is:    Fri May  1 15:25:10 2020 CEST
	SMART support is: Available - device has SMART capability.
	SMART support is: Enabled

Si tu utilises un adapter usb, il se peut que :

	dev/sdb: Unknown USB bridge [0x152d:0x1562 (0x204)]
	Please specify device type with the -d option.

Donc précise le type de device ainsi :

	$  smartctl -d sat -i /dev/sdb

#### Si tu dois activer
	$ smartctl --smart=on --offlineauto=on --saveauto=on /dev/sdX

#### Estime le temps des checks et ceux possibles
	$ smartctl -c /dev/sdb
	Short self-test routine 
	recommended polling time: 	 (   2) minutes.
	Extended self-test routine
	recommended polling time: 	 (  97) minutes.

Checks possibles : ici par exemple Conveyance n'est pas supporté.

	Offline data collection capabilities: (0x5b) SMART execute Offline immediate.
	Auto Offline data collection on/off support.
	Suspend Offline collection upon new command.
	Offline surface scan supported.
	Self-test supported.
	No Conveyance Self-test supported.
	Selective Self-test supported.
	SMART capabilities: (0x0003) 

#### Lance les tests en fonction de ceux possibles 
	$ smartctl -t short /dev/sdb
	$ smartctl -t long /dev/sdb
	$ smartctl -t conveyance /dev/sdb

#### Où en est mon test ?
	$ smartctl -a /dev/sdb | grep -A1 "Self-test execution status"
	Self-test execution status:      ( 248)	Self-test routine in progress...
						80% of test remaining.


#### Affiche les résultats
	$ smartctl -H /dev/sdb

#### Affiche les tests récents
	$ smartctl -l selftest /dev/sd
	=== START OF READ SMART DATA SECTION ===
	SMART Self-test log structure revision number 1
	Num  Test_Description    Status                  Remaining  LifeTime(hours)  LBA_of_first_error
	# 1  Extended offline    Completed without error       00%       739         -
	# 2  Short offline       Completed without error       00%       738         -
	# 3  Short offline       Completed without error       00%         1         -

#### Plus de précisions (toutes)
	$ smartctl -s on -a /dev/sdb

#### N'afficher que les erreurs
	$ smartctl -q errorsonly -H -l selftest /dev/sdb

#### Test sur des grandes capacités

Un test long peut durer des heures. S'il y a un arrêt inopiné du test, on repart à zéro. Préfère lancer un test sur les 500 premiers Gio par exemple.

	# pour 500 Go
	$ smartctl  -t select,0-999999999  /dev/sdX 
	# pour 1 To
	$ smartctl  -t select,0-1999999999  /dev/sdX    ### pour 1 Tio
	# pour 250 Go
	$ smartctl  -t select,0-499999999  /dev/sdX     ### pour 250 Gio

#### Un disque potentiellement en mauvaise santé

Ici read failure.

	$ smartctl -l selftest /dev/sdb
	SMART Self-test log structure revision number 1
	Num  Test_Description    Status                  Remaining  LifeTime(hours)  LBA_of_first_error
	# 1  Extended offline    Completed: read failure       00%      3257         63488
	# 2  Extended offline    Aborted by host               90%      3247         -
	# 3  Short offline       Completed without error       00%         0         -

et...

	$ smartctl -a /dev/sdb | grep -A1 "Self-test execution status"
	Self-test execution status:      ( 112)	The previous self-test completed having
					the read element of the test failed.

#### Badblocks

Après les tests SMART, tu peux rechercher les secteurs défectueux.

Première vérification en lecture seule.

	# badblocks -v -s /dev/sda
	Checking blocks 0 to 58615703
	Checking for bad blocks (read-only test): done
	Pass completed, 0 bad blocks found. (0/0/0 errors)

Plusieurs passes possibles avec -p.

	# badblocks -v -s -p 2 /dev/sda

L’option -w permet d’effectuer des tests en écriture. Ne pas utiliser sur un système en production les données seront perdues.

	# badblocks -v -w -s -p 2 /dev/sda

Tags: linux
