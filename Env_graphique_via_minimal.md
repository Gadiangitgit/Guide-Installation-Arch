# Installation de l'interface graphique GNOME
- la 1er ligne installe l'environnement graphique et les dossier users
- la 2 eme ligne installe tout l'enviromment gnome (parametres, gestionnaire de fichier, terminal) et des élément de personalisation bonus (je trouve GNOME pas pratique donc vaut mieux le custom selon moi)
- la 5ème ligne passe le clavier en fr
- la 6 ème lance pour de bon l'interface graphique 
```shell
sudo pacman -S xorg xdg-user-dirs
sudo pacman -S gnome gnome-tweaks          
# Si pas déja fait active le réseau à chaque démarrage
sudo systemctl enable --now NetworkManager
localectl --no-convert set-x11-keymap fr
sudo systemctl enable --now gdm.service
```
**Ne pas hésiter à vérifier si le clavier est qwerty ou azerty via le petit oeuil**  
Une fois connecté, pour avoir clavier fr partout aller dans Paramètres -> Clavier -> Choisir Français (il faut le modifier sur l'interface graphique malgré tout ce qui à été défini auparavant c'est un peu tordu mais bon)

Et c'est fini !


# Installation de l'interface graphique KDE Plasma
- la 1ère ligne gère toute la partie graphique et création de répertoires pour le user
- la 2 ème ligne install l'environnment plasma mais il existe plusieurs variantes en fonction du besoin
- la 3 ème ligne contient des applications nécessaire (terminal, gestionnaires de fichiers, éditeur de texte, gestionnaire d'archive)
- la 4 ème ligne contient les pilotes audio car sous KDE-Plasma il ne sont pas installer nativement
- la 5ème ligne contient le gestionnaire de connexion qui une fois activer lancera le login screen à l'interface graphique.
- la 8ème ligne passe le clavier en fr
- la 9 ème lance pour de bon l'interface graphique  

```shell 
sudo pacman -S xorg xdg-user-dirs
sudo pacman -S plasma 
sudo pacman -S kitty dolphin ark kate firefox 
sudo pacman -S pipewire pipewire-pulse wireplumber
sudo pacman -S sddm
# Si pas déja fait active le réseau à chaque démarrage
sudo systemctl enable --now NetworkManager
localectl --no-convert set-x11-keymap fr
sudo systemctl enable --now sddm
``` 
Beurk sddm à démarrer c'est moche hein ?

Pour avoir clavier fr partout aller les modifier dans Paramètres -> Clavier -> Choisir Français (a vrai dire je sais plus si on dois aussi le faire sur kde mais dans le doute)

Et c'est fini !

# Installation de l'interface graphique Hyprland

A venir...

