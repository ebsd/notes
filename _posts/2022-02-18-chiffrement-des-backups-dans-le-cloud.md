---
layout: post
title:  "Chiffrement des backups dans le cloud"
date:   2022-02-18 20:14:48 +0100
---

[Rclone](https://rclone.org/) permet de synchroniser vos fichiers avec le cloud (Dropbox, Google drive, pcloud, Mega et bien d'autres). C'est idéal pour un backup en ligne. Pour assurer la confidentialité de nos données, utilisons en complément [gocryptfs](https://nuetzlich.net/gocryptfs/).

Gocryptfs propose un mode "reverse" qui "présente" une vue chiffrée de vos données (et elles restent non chiffrées). Rclone peut ensuite être utilisé pour sauvegarder cette vue chiffrée de vos données, dans un serveur en ligne de votre choix.

## Initialiser la configuration "reverse" gocryptfs

`-plaintextnames` : option pour ne pas chiffrer les noms de fichiers, c'est moins sécurisé mais plus simple pour restaurer un seul fichier !

	$ umask 0077
	$ mkdir ~/.nobackup
	$ gocryptfs -reverse -aessiv -plaintextnames \
    	-config ~/.nobackup/gocryptfs.reverse.rootfs.conf \
	    -init ~/.nobackup/
	$ mkdir -p /tmp/remotebackup/rootfs


## "Monter" la vue chiffrée dans un dossier

`-extpass`: option qui permet d'obtenir la clé de chiffrement stockée dans votre gestionnaire de mots de passe favoris, [pass](https://www.passwordstore.org/).

	$ gocryptfs -reverse -config ~/.nobackup/gocryptfs.reverse.rootfs.conf -exclude .nobackup/ -exclude /remotebackup/rootfs -fsname gcryptfs-reverse /home/ebsd /tmp/remotebackup/rootfs -extpass pass -extpass mypass/gocrypt

Maintenant les données visibles dans /tmp/remotebackup/rootfs sont indéchiffrables. C'est la fameuse vue chiffrées de vos données.

## Full Backup de la vue chiffrée vers pcloud

`--backup-dir` permet de conserver la dernière version de chaque fichier remplacé, une assurance complémentaire.

	# sync vers pcloud
	/usr/bin/rclone sync --update --verbose --transfers 30 --checkers 8 --contimeout 60s --timeout 300s --retries 3 --low-level-retries 10 --stats 2s "/tmp/remotebackup/rootfs" "rclone_pcloud:/ebsdhome/" --filter-from ~/rclone-filter

`--filter-from` permet de filtrer ce qui est uploadé, contenu du filtre `~/rclone-filter`

	$ cat rclone-filter 
	- /Documents/no_backup/**
	+ /Documents/**
	+ /sync/**
	- /.bashrc.d/**
	+ .bashrc
	+ .pandoc
	+ /scripts/**
	- *
	- .*
	- .*/

Tags: backup, cloud, rclone, gocryptfs
