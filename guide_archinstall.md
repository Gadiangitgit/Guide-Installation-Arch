il y a rien cheh nullos

Bon à l'arrache 

A partir de ce moment il est possible d'installer via archinstall

Language Archinstall -> French
Paramètres régionaux
-> Disposition du clavier  fr
-> Langue locales fr_FR.UTF-8
-> Encodage des paramètres régionaux UTF-8
Miroirs et dépôts
-> Sélectionner des régions France
-> Dépôts supplémentaires multilib
Configuration du disque
-> Partitionnement
Partitionnment Manuel (On est pas des merdes)
3 Partition 
En cas de partition BIOS Et NON UEFI !!
Taille                FSType  Mountpoint
1 GIB                 fat32   /boot
EN FONCTION DE LA RAM
TOUT LE RESTE         EXT4    /
-> Chiffrement du disque 
-> Type de Chiffrement
-> Configurer la clé unifiée Linux (LUKS)
-> Mot de passe de chiffrement 
Définir un mot de passe de chiffrement
-> Iterations time
Par défaut ou + en fonction des besoin
-> Partitions 
Sélectionner celle par défaut (la EXT4 en autre) 
-> Retour au menu Principal
// A voir si LUKS sur partition ou on fait jouer la TPM aussi
-> Swap
Swap sur zram : Activé
Algorithe de compression : zstd (A le meilleur rapport temps de compression et niveau de compression)
-> Chargeur de démarrage 
Ce bon vieux grub
-> Noyaux
Linux par défaut
-> Nom d'hôte
au choix
-> Authentification
Définir le mot de passe root
Compte Utilisateurs -> Ajouter un utilisateur
Définir nom & mdp 
Utilisateur sudo : Oui
-> Profil 
Type 

Alors la tout dépend if c'est pour installé des dots file ou non
Si utilisation Clé en main prendre GNOME ou KDE (Desktop)
Si c'est pour Rice prendre Hyprland (Desktop) ou Minimal

Applications 
-> Prendre les services nécessaire
Bluethooth -> Activé
Audio -> pipewire
Print service -> Activé

Configurer le réseau 
Configuration manuelle
Ajouter une interface
Sélectionner l'interface
DHCP
Confirmer et Quitter

Paquet Supplémentaires ( / pour faire des recherches)
git wget fastfetch

Fuseau Horaire Europe/Paris
Activer NTP

Plus qu'a installé et reboot
