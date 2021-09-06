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
Non creusé mais une fois fzf installé, quand je lance une recherche depuis une console (ctrl+r), l'interface habituelle de recherche est remplacé par un accès fzf depuis l'historique de la commande **history** (je ne sais pas où c'est paramétré) et quand on sélectionne une ligne, elle s'écrit à la suite du prompt comme si je venait de taper la commande manuellement, qu'il ne restera plus qu'à valider comme n'importe quelle commande. J'ai trouvé cette [source](https://github.com/junegunn/fzf/issues/1695) qui cherche à utiliser le même mécanisme pour le reproduire mais apparemment le fil n'indique pas de solution finale. Je cherche exactement la même chose, donc à étudier.
