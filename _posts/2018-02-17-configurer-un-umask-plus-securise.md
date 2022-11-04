---
layout: post
title:  "Confirgurer un umask plus sécurisé"
date:   2018-02-17 19:18:11 +0100
---

Je fais le point sur les différentes options de configuration du umask. Pour rendre les permissions de votre système plus restrictives, sans pour aurant bloquer tout son fonctionnement, le umask idéal est à mon sens le 027 ; l'utilisateur peut lire et écrire, et le groupe peut seulement lire. Les autres utilisateurs ne possèdent pas de permissions.

## Quel est le Umask par defaut ?

Pour cet article, nous utilisons une distribution redhat like.

Il existe de nombreux shell comme bash, ksh, zsh, et tcsh. Ces shells peuvent se comporter en tant que login shells et non login-shells. Le login shell est généralement appelé en ouvrant un terminal natif ou depuis une GUI.

### Login shell ou non-login shell ?

Pour déterminer quel type de shell on exécute, utilisons la commande `echo $0`.

Si la sortie de `echo $0` retourne bash, alors c'est un non-login shell qui est exécuté. La commande `sudo -s` ne créé pas un login shell.

	$ echo $0
	bash
Le umask par défaut pour un non-login shell est configuré dans /etc/bashrc.

Si la sortie de `echo $0` retourne -bash, alors c'est un login shell qui est utilisé. La commande `sudo su -` créé un login shell par exemple.

	# echo $0
	-bash
Le umask par défaut pour un login shell est configuré dans /etc/profile.

Pour afficher le umask configuré par défaut pour un non-login shell :

	$ grep umask /etc/bashrc
	# By default, we want umask to get set. This sets it for non-login shell.
       umask 002
       umask 022
Pour afficher le umask configuré par défaut pour un login shell :

	$ grep umask /etc/profile
	# By default, we want umask to get set. This sets it for login shell
       umask 002
       umask 022
       
## Configurer un umask plus restrictif

Nous allons nous attacher aux login shells plus particulièrement.

Utilisons une valeur du umask plus sécurisée dans /etc/profile : umask 022 => umask 027

	if [ $UID -gt 199 ] && [ "/usr/bin/id -gn" = "/usr/bin/id -un" ]; then
		umask 002
	else
		umask 027
	fi

En complément, on souhaite conserver un umask à 022 pour un utilisateur particulier :

	$ echo 'umask 022' >> /home/username/.bashrc

## Job Ansible

Et voici une tâche ansible qui fait le job :

	- name: Set default umask 027 in /etc/profile for login shells
	  replace:
	    path: /etc/profile
	    regexp: '(^\s+umask) (0[012][0-6])'
	    replace: '\1 027'

	- name: Set default umask 027 in /etc/login.defs for newly created home directories
	  replace:
	    path: /etc/login.defs
	    regexp: '(umask|UMASK).+(0[012][0-6])'
	    replace: '\1 027'

	- name: Set mask 022 or set back to 022 if other umask already set for specific account
	  lineinfile:
	    dest: /home/username/.bashrc
	    state: present
	    regexp: '^(umask|UMASK)'
	    line: 'umask 022'

Tags: linux, security, umask, ansible
