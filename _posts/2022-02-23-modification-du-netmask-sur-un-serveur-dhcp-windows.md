---
layout: post
title: "Modification du netmask sur un serveur dhcp windows"
date: 2022-02-23 22:54:32 +0100
---
Via l'interface graphique, on peut changer le scope IP en revanche impossible de modifier le masque. Pour ce faire, il faut exporter la config en XML, supprimer le scope, modifier le netmask dans le fichier XML et enfin importer cette config. Cette opération sauvegarde / conserve les leases / baux en cours.

1. Export 

	`Export-DhcpServer -Leases -File C:\dhcp.xml -Verbose -ScopeId 192.168.128.0`

2. Puis modifier SubnetMask et EndRange dans C:\dhcp.xml

changer :

	<SubnetMask>255.255.240.0</SubnetMask>
	<StartRange>192.168.128.50</StartRange>
	<EndRange>192.168.143.254</EndRange>

en :

	<SubnetMask>255.255.248.0</SubnetMask>
	<StartRange>192.168.128.50</StartRange>
	<EndRange>192.168.135.254</EndRange>

3. Supprimer le scope dans dhcpmgmt.msc

4. Importer la nouvelle étendue

	`Import-DhcpServer -File C:\dhcp.xml -Verbose -ScopeId 192.168.128.0 -Lease -BackupPath C:\BkDHCP`

Lors de l'import, les baux qui ne peuvent pas être recréés car ils sont hors du nouveaux scope / subnet, vont être affichés en erreur. Il faut poursuivre l'import jusqu'à la fin. Vérifier dans la console dhcp, on a bien les leases en cours et toute la config précédante.

Tags: windows, dhcp
