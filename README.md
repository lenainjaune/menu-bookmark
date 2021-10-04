# menu-bookmark
Fuzzy bookmarks for your shell

[Source](https://dmitryfrank.com/articles/shell_shortcuts)

![Result](https://dmitryfrank.com/_media/articles/cdg_recorded.gif)
```sh
# Prerequisite : needs gawk
# install
git clone --depth 1 https://github.com/junegunn/fzf.git
fzf/install
source ~/.bashrc
# example
su -c "cat << EOF > /etc/cdg_paths
/media/lnj/VBIG # video, zik etc
/media/lnj/DATA
EOF"
cat << EOF > ~/.cdg_paths
/mnt/3tb/ # les vms
EOF
cdg(){ c=$( cat /etc/cdg_paths ~/.cdg_paths \
 | gawk '{ print gensub ( /(.+) #.+/ , "\\1" , "g" ) }' \
 | fzf ) ; [[ "$c" != "" ]] && cd "$c" ; }
```
Une fois fzf installé, quand je lance une recherche depuis une console (ctrl+r), l'interface habituelle de recherche est remplacé par un accès fzf depuis l'historique de la commande **history** (je ne sais pas où c'est paramétré) et quand on sélectionne une ligne, elle s'écrit à la suite du prompt comme si je venait de taper la commande manuellement, qu'il ne restera plus qu'à valider comme n'importe quelle commande. J'ai trouvé cette [source](https://github.com/junegunn/fzf/issues/1695) qui cherche à utiliser le même mécanisme pour le reproduire. Le fil indique une solution qui marche sur certains systèmes et pas sur d'autres. A étudier...

```sh
# Afficher/filtre bookmarks et exécuter la commande du bookmark choisi

# forme générale : cat bookmarks | awk .... | fzf | awk ... | perl ...
#	bookmarks : fichier texte qui contient les bookmarks (voir [1])
#	1er awk : sert à mettre en forme les bookmarks pour les rendre 
#		lisibles avec gestion de padding (voir [2])
#	fzf : l'outil de recherche fuzzing pour filtrer et choisir 
#		(voir [3])
#	2eme awk : sert à extraire du choix la commande prête à être 
#		exécutée (voir [4])
#	perl : sert à insérer la commande prête à être exécutée dans la 
#		ligne en cours (voir [5])
#
# [1] le fichier bookmark est composé de lignes :
#	# commentaire interne aux bookmarks
#	
#	`# commentaire de commande` ; commande
#	commande
# Notas : 
#	o un commentaire de commande doit commencer par `#' est optionnel
#		mais conseillé et ne sera pas exécuté ; voir 
# https://stackoverflow.com/questions/9522631/how-to-put-a-line-comment-for-a-multi-line-command
#		utile par exemple pour identifier un accès ssh par du port 
# 		forwarding (ex : port 10022 -> port SSH du NAS) vu qu'on ne peut pas 
#		nommer des ports
#	o la commande peut être multiple/composée ; ex : ( a | b ) 2&>1 | c
# [2] gawk 'BEGIN { ... } { ... } END { ... }' :
#	BEGIN { ... } : initialiser
#	le corps : si la ligne n'est pas un commentaire interne ou une ligne vide
#		convertir chaque ligne pour rendre dissociable le 
#		commentaire optionnel, de la commande par le caractère null 
#		(\0) et la stocker dans un tableau ; extraire le commentaire 
#		s'il existe pour déterminer sa taille qui servira au padding 
#		lors de l'affichage
#	END { ... } : afficher toutes les commandes formatées avec un 
#		right-padding (%-...s) de la plus grande chaine de commentaire 
#		(s'il y en a au moins un)
#	Nota : je ne sais plus bien pourquoi mais pour afficher dans 
#		l'ordre du fichier bookmarks, il faut faire 
#		`for ( i = taille ; i >= 0 ; i-- )'
# [3] la commande fzf : afficher le menu
# [4] gawk ... : 
#		extraire de la ligne choisie, le commentaire s'il existe et la 
#		commande, en les restituant avec un trim droit et gauche des 
#		espaces
# [5] perl ... : insère la commande (copié en l'état sans maitriser 
#		quoi que ce soit) ; voir 
#		https://github.com/junegunn/fzf/issues/1695

# TODO : corriger le bug du [5] lors de l'insertion de la commande => CMD user@host:~$ CMD
# => sur lnj-tosh je n'ai pas ce pb, ça fonctionne nickel ! Peut être le pb est uniquement sur certains (ex : Ubuntu Xenial 16.04 LTS)

choose_ssh_bookmarks(){ cat ~/ssh_bookmarks \
| gawk 'BEGIN { \
		nb_bookmarks = 0 ; \
        } \
         ! /^[[:space:]]*#/ \
        { \
		line = gensub ( /(`#.+` * ; *)*(.+)/ , \
		 "\\1\0\\2" , "g" ) ; \
		bookmarks [ nb_bookmarks++ ] = line ; \
		split ( line , commands , "\0" ) ; \
		l = length ( commands [ 1 ] ) ; \
		if ( l > length_max ) \
			length_max = l ; \
	} END { \
		for ( i = nb_bookmarks - 1 ; i >= 0 ; i-- ) { \
			split ( bookmarks [ i ] ,  \
			 commands , "\0" ) ; \
			printf "%-" length_max "s%s" , \
			 commands [ 1 ] , commands [ 2 ] "\n" \
		} \
	}' \
| fzf \
| gawk '{ \
		line = gensub ( /(`# .+` *; *)*(.+)/ , \
		 "\\1\0\\2" , "g" ) ; \
		split ( line , commands , "\0" ) ; \
		comment = commands [ 1 ] ; \
		command = commands [ 2 ] ; \
		gsub ( /^[ \t]+|[ \t]+$/ , "" , comment ) ; \
		gsub ( /^[ \t]+|[ \t]+$/ , "" , command ) ; \
		if ( comment != "" ) \
			comment = comment " " ; \
		print comment command \
	}' \
| perl -e \
 'ioctl STDOUT, 0x5412, $_ for split //, do{ chomp($_ = <>); $_ }' ; \
)

# And a sample of ~/ssh_bookmark :
`# SSH test` ; ssh $USER@localhost
`# SSH port forwarding` ; ssh -P 10022 root@server
```
