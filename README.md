# menu-bookmark
Fuzzy bookmarks for your shell

[Source](https://dmitryfrank.com/articles/shell_shortcuts)
```sh
git clone --depth 1 https://github.com/junegunn/fzf.git
fzf/install
source ~/.bashrc
su -c "cat << EOF > /etc/cdg_paths
/media/lnj/VBIG # video, zik etc
/media/lnj/DATA
EOF"
cat << EOF > ~/.cdg_paths
/mnt/3tb/ # les vms
EOF
cdgg(){ c=$( cat /etc/cdg_paths ~/.cdg_paths \
 | gawk '{ print gensub ( /(.+) #.+/ , "\\1" , "g" ) }' \
 | fzf ) ; [[ "$c" != "" ]] && cd "$c" ; }
```
