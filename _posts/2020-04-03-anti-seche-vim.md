---
layout: post
title:  "anti sèche vim"
date:   2020-04-03 21:14:48 +0100
---

Profitons de cette période de **confinement** numéro 1 pour réviser.

- Se déplacer au mot suivant : `w`
- Se déplacer au mot précédant : `b`
- Fin de ligne : `$`
- Début de ligne : `^`
- Aller à la ligne n : `nG`
- Aller à la 1ere ligne : `1G`
- Dernière : `G`
- Undo : `u`
- Joindre la ligne suivante à la ligne courrante : `J`
- Numéro de ligne : `Ctrl-g` ou `:set number`
- Copier la sélection : `y`
- Copier la ligne : `yy`
- Copier n lignes : `nyy`
- Copier le mot : `yw`
- Copier n mots : `nyw`
- Coller après le curseur : `p`
- Coller avant le curseur `P`
- Supprimer la ligne : `dd`
- Supprimer le reste de la ligne : `d$`
- Supprimer depuis le début de la ligne : `d^`
- Supprimer jusqu'à la fin du fichier : `dG`
- Supprimer depuis le début du fichier : `d1G`
- Supprimer n lignes : `ndd`
- Supprimer n mots : `ndw`
- Rechercher vers le bas : `/chaine`
- Rechercher vers le haut : `?chaine`
- Répéter la recherche : `n`
- Repéter la recherche dans la direction inverse : `N`
- Désactiver la sensibilité à la casse : `:set ignorecase`

#### Un peu plus loin
- Supprimer les espaces à la fin de chaque ligne : `:%s/\s\+$//e`
- Idem au début de chaque ligne : `:%s/^\s+//e`
- Afficher un guide d'indentation (tab) : `:set listchars=tab:\|\` puis  `:set list`
- Indenter (tabulation) un bloc de texte : sélectionner le texte avec `V`, puis `jj>`
- Déplacer le curseur librement : `set virtualedit=all`

Tags: linux
