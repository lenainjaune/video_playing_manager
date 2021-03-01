# video_playing_manager
Gérer la lecture de vidéos (fichiers aux formats vidéo, CD/DVD physique, images au format ISO, etc.)

Ce script (SOUS Ubuntu) permet de créer des actionneurs vidéo (ici principalement 
avec l'application [vlc](http://doc.ubuntu-fr.org/vlc)), 
des liens Internet, etc. tout ce qui nous 
passe par la tête. Depuis une fenêtre graphique [zenity](http://doc.ubuntu-fr.org/zenity) 
composée d'une liste d'actions, on peut : exécuter ces actions, éditer 
directement le script (ici avec l'application [geany](http://doc.ubuntu-fr.org/geany)), 
rafraichir la liste et autres.
```sh
#!/bin/bash

# basé sur : https://wiki.visionduweb.fr/index.php/Programmation_GNU_Linux_Zenity
# man CLI vlc : https://wiki.videolan.org/VLC_command-line_help/

# Créer un lanceur :
# user@host:~$ cat <do_actions_file_name>.desktop
# [Desktop Entry]
# Version=1.0
# Type=Application
# Name=<do_actions_displayed_name>
# Comment=
# Exec=</path/to/do_actions_script.sh>
# Icon=vlc
# Path=
# Terminal=false
# StartupNotify=false
#
# Nota : pour tester ce lanceur en CLI et avoir la sortie (Ubuntu)
# xdg-open <do_actions_launcher_file>.desktop

# remarques :
# "exec" remplace l'instance bash actuelle 
#  => on n'est plus dans l'exécution du script
# si on veut exécuter et revenir dans le script, 
#  il faut utiliser plutôt "bash -c" (ou autres) en englobant avec \" 
# source : https://superuser.com/questions/344478/bash-execute-command-given-in-commandline-and-dont-exit/344481#344481

# Nota
# Dans les commandes VLC, si on remplace --repeat par --play-and-exit 
#  => on quitte VLC juste après la lecture 
# ex : 
#  for i in $( seq 1 2 ) ; do vlc --fullscreen \
#    --play-and-exit dvdsimple://$VIDEO#21 --start-time=224 \
#    --stop-time=229 ; done
# permet de répéter 2 fois seulement


# TODO faire en sorte que le code intégré de zenity soit PLUS lisible


# source : https://stackoverflow.com/questions/2181712/simple-way-to-convert-hhmmss-hoursminutesseconds-split-seconds-to-seconds/48032950#48032950
# référence perso : https://github.com/lenainjaune/bash/blob/main/README.md
# cette solution gère les parties non renseignées (00:01:01 = 01:01)
#  et semble assez portable
function hms2s () {
	echo $1 | awk -F\: '{ for(k=NF;k>0;k--) sum+=($k*(60^(NF-k))); print sum }'
}
#echo $( hms2s "05:10" )
#exit 0

VIDEO=/path/to/dvd/or/iso


# Pour avoir les numéros de pistes ou les positions de lecture
# il faut ouvrir VLC normalement et récupérer les infos

# Par défaut, on relance le gestionnaire
while true ; do

 code=$( zenity \
   --width=600 --height=600 --list \
   --text= --title="<title_of_action_window>" \
   --column="Action" --column="code" --hide-column=2 --print-column=2 \
  \
  \
  "Run DVD normally in fullscreen" \
   "bash -c \"vlc --fullscreen dvd://$VIDEO\"" \
  \
  "---------------------- section separator ---------------------" "-" \
  \
  "run a specific vlc title (behind #)" \
   "bash -c \"vlc --fullscreen dvdsimple://$VIDEO#20\"" \
  \
  "run from a position to another and quit" \
    "bash -c \"vlc --fullscreen dvdsimple://$VIDEO#20 \
    --start-time=$( hms2s "00:10" ) --stop-time=$( hms2s "08:45" ) \
    vlc://quit\"" \
  \
  "----------------------------- Web ----------------------------" "-" \
  \
  "<web_link_description_to_open_with_default_internet_browser>" \
    "xdg-open <url_link>" \
  \
  \
  "---------------------------- Tools ---------------------------" "-" \
  \
  "Edit the script..." \
    "geany \"$0\" &" \
  \
  "Reload script..." \
    "-" \
  \
 )

 if [ "$code" == "" ] ; then
  exit
 fi

 if [ "$code" != "-" ] ; then
  eval "$code"
 fi
 
 # On remplace le code actuel pour actualiser dynamiquement grace à exec
 exec "$0"

done
```
