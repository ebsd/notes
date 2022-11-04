---
layout: post
title:  "Comment déterminer les priorités des SI de santé dans l'établissement d'un PRA"
date:   2015-01-18 19:15:00 +0100
---

Pour de nombreux hôpitaux, les sauvegardes et restaurations de données sont des tâches lourdes à planifier et à gérer au quotidien. Les volumes de données se multiplient tellement rapidement que la réalisation d'une sauvegarde dans la fenêtre temporelle initialement définie devient complexe, voire impossible. De plus, l'explosion du numérique et des moyens de collectes vont engendrer une évolution du volume mondial de données de santé, on parle de multiplier ce volume de données de santé par 50 d'ici 2020...

Développer de bonnes pratiques de protection des données de santé, et passer d'un mode de sauvegarde difficile à une stratégie globale de plan de reprise d'activité ; tels sont les enjeux de la sécurité.

L'étape essentielle dans l'établissement d'un plan de reprise d'activité est la priorisation de chaque composant des systèmes d'information de santé. Pour ce faire, le RSSI (Responsable Sécurité des Systèmes d'Information) doit prendre en considération deux notions : le RPO (Recovery Time Objective) et le RTO (Recovery Point Objective).

- Le RTO correspond à la durée maximale d'interruption admissible d'une activité. Il s'agit du temps maximal dont disposent les équipes infrastructures pour rendre fonctionnel un service sinistré. Le RTO est défini est répondant à la question : "combien de temps pouvons nous nous permettre de nous passer d'un système ?"

- Le RPO détermine la durée maximale d'enregistrement de données qu'on peut se permettre de perdre lors d'un sinistre. Il s'agit du point à partir duquel il faut absolument être capable de disposer de sauvegardes valides. En somme il faut se poser la question "combien de données nous permettons nous d'avoir à récréer, réintégrer, resaisir..?"

Un hôpital ne possède pas un seul RPO ou RTO pour toutes ces activités. Un niveau de criticité doit être affecté à tous les systèmes pour déterminer dans quel ordre ils seront restaurés et selon quelle planification ils seront sauvegardés. Cette priorisation en fonction de la criticité est la clé du PRA. Par exemple, un établissement de santé peut juger que son DPI (Dossier Patient Informatisé) est l'application la plus critique. Par conséquent il va lui assigner un niveau de criticité maximal et donc les RPO et RTO les plus courts. En revanche, la messagerie, même si elle est nécessaire à la communication, peut être jugée moins critique car elle n'intervient pas dans le processus de prise en charge du patient. Son niveau de RTO peut être positionné à un niveau court et son RPO à un niveau long.

Le volume de données ainsi que leur type, influencent le RPO et le RTO, et la stratégie de protection des données qui sera adoptée.

## RPO/RTO des systèmes fréquemment actualisés

Les systèmes dont les données sont mises à jour fréquemment au cours de la journée, comme les dossiers patients, posséderons un RPO et un RTO court. Il est possible qu'un RPO d'une à quatre heures soit nécessaire. Avec un RPO de 4 heures, une sauvegarde doit avoir lieu 6 fois par jour. Les technologies actuelles de réplication et de snapshot permettent un RPO d'une heure. Elles fournissent également des possibilité de RTO très court. Les copies répliquées étant "démarrables" sans avoir à être restaurées. Toutefois, un second datacenter doit permettre d'accueillir les données complètes des systèmes répliqués.

## RPO/RTO des systèmes PACS (Picture Archiving and Communication Systems)

Ces systèmes nécessitent également des RPO et RTO courts même si les données de ce type de système représentent un défi pour la sauvegarde, en considérant leur volume. Comment réaliser un snapshot de plusieurs TB de donneés ? C'est impossible, si on veut être rapide. Le RPO sera donc plus long, tout comme le RTO : la durée de restauration d'un tel volume de données est très élevée.

Toutefois les bases de données qui constituent les PACS ne sont pas si volumineuses et peuvent bénéficier d'un couple RPO/RTO identique à celui d'un DPI. Je le précise, ce qui représente un défi dans la sauvegarde du PACS, c'est le volume des données (les images). Heureusement elles sont statiques. Les métadonnées associées aux images peuvent changer, mais l'image elle même ne change pas. Effectuer une duplication fréquente de ces images constituerait une mauvaise tactique. Pourquoi sauvegarder des données qui ne changent pas ?

Une duplication de données est utile pour restaurer un système complet. En ce qui concerne les données volumineuses et statiques, l'archivage est une meilleure approche. La mise en place de copies géographiquement réparties des données, sur des supports différents permet d'atteindre un bon RPO et RTO pour les systèmes de type PACS. Par exemple, en cas de sinistre, votre base de données de PACS peut être restaurée aussi rapidement que votre DPI, et pointer sur une copie des images.

La combinaison de sauvegarde et d'archivage fournit une stratégie de PRA optimale en permettant des valeurs de RPO/TRO acceptables sur les technologies de l'information dans la santé.

Tags: santé
