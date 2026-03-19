# Guide d'installation de Arch-Linux UEFI (Minimal)


## Pré-requis pour le bon déroulement de l'installation

**Permet de passer le clavier en FR (temporairement)**
___
```shell
loadkeys fr
```
**Pour agrandir la police en fonction de la taille de l'écran**
```shell
setfont -d
```

**Pour savoir si une adresse IP a été récupérer et voir le nom des cartes réseaux**
___

```shell
ip a
``` 
**Si vous récupérer bien une adresse IP, passer au ping sinon suivre les indications suivantes**

Si Ethernet :  
Branche ton câble  
Si y a une ip cohérante on continue  
Sinon bah check ton câble ou si c'est bien branché man  
Et si ta toujours un problème skill issue chef  

Si wifi :
```shell
iwctl  
station <nom de carte wifi> scan  
station <nom de carte wifi> connect <SSID>
Saisir la passphrase  
Si wifi caché  : station <nom de carte wifi> connect-hidden <SSID> 
exit  
```
### Pour tester la connectivité (IP & DNS)
___
```shell
ping www.google.com 
```

## Début de l'installation
A partir de maintenant, il va falloir vous poser 3 questions :  
Est ce que je dois mettre de la SWAP ?  
Est ce que mon disque doit être chiffré ?  
Est ce que je dois séparer /home ou non ?  

Avant de commencer voici un petit brief sur ces 3 options.

La SWAP permet de déplacer des données de la RAM vers le disque pour éviter une surconsommation de la RAM en cas de pic, utile aussi en cas de machine qui peuvent être mis en hibernation.  
ATTENTION : La swap est assez pratique sur cet aspect mais garder en tête que si un virus ou une quelquonque attaque touche votre systeme et charge quelque chose en mémoire cela pourra être garder dans la swap (sur votre disque) je vais surment trop loin mais garder cela en tête.

Le chiffrement disque intervient comme une mesure de protection de vos données, elle sert souvent en cas de vol, c'est une mesure qui est fortement recommandé sur un laptop.  
ATTENTION : C'est une bonne mesure de sécurité mais si vous ordinateur, cela n'est peut être pas pertinant, parter du principe que cela rajoute un mot de passe à taper à chaque fois et ralenti le démarrage du PC car déchiffrement obligatoire au début de celui-ci

La partition /home est assez importante, il s'agit de celle qui va contenir toutes nos données l'avantages vu qu'elle est modulaire il possible de garder en cas de problème avec l'OS mais elle permet de bien distinguée la partie systèmes et fichier personnel, de plus la configuration si dessous sera fait avec LVM qui permet de réallouer de la taille de disque en cas d'un /root trop gourmand.

### Lister les partitions créer afin de les formater correctement
___
Afin de continuer,  il faut correctement identifier les partitions de notre disque dur repérer en fonction de la taille 
```shell
lsblk
```  
**Dans notre cas UEFI :**   
sda1 est la partition EFI  
sda2 est la partition de swap   
sda3 est la partition de EXT4 de tout notre OS (Linux Filesystem)  
sda4 est la partition de EXT4 de /home (Linux Filesystem)   



### Mettre en place le partitionnement du disques dur en UEFI
Il est très important de préciser le disque par défaut pour être sur de ne pas ce tromper de périphérique d'installation, les disque en sd? sont fait par lettre de l'alphabet et les NVME et Carte SD en numéro
- /sd? = disque HDD & SSD 
- /nvme0n? = disque NVME
- /mmcblk? = carte SD (qui fait ça sérieux ?)
___
Dans notre cas ça sera /dev/sda n'hésiter pas à adapter en fonction de votre disque
```shell
cfdisk /dev/sda 
```  

Sélectionner GPT (C'est marrant hein ?)

**En cas de disque non chiffré et /home séparer** 

| Type de Partition | Taille de Partition | Type de Formatage |
| :--------------------- | :--------------- | :---------------|
| EFI System| 512 MB ou 1G (en cas de dual-boot ou multi kernel) | FAT32
| Linux swap | En fonction de la RAM (Si 4G de RAM mettre 4G de SWAP ect..) | Linux SWAP
|Linux Filesystem | +-30G | EXT4 
|Linux Filesystem | Tout le reste du disque| EXT4 


**SPAMMER PAS ENTRÉE PAR IMPATIENCE CELA VOUS FERRAIS QUITTER LE MENU DE CFDISK ET TOUT RECOMMENCER**  
- Sélectionner le disque vide avec Entrée
- Saisir la taille de la partition souhaité ( Format : NombreUnité ex: 512MB) puis Entrée
- Naviguer avec les flèches pour changer le type de partition via Type
- Sélectionner la bonne partition (EFI System, Linux Filesystem, Linux swap)
- Répéter le processus pour chaque partition
- Sélectionner Write pour Sauvegarder puis écrire yes
- Quit pour Quitter


### Formater les partitions sous le bon format (UEFI)
___
**Pour formater notre partition/dev/sda3 en EXT4**
```shell 
mkfs.ext4 /dev/sda3
```
**Pour formater notre /dev/sda2 en swap**
```shell
mkswap /dev/sda2
```
**Pour formater le /dev/sda1 en FAT32**
```shell
mkfs.fat -F 32 /dev/sda1
```
**Pour formater /home** 
```shell
mkfs.ext4 /dev/sda4
```

### Monter les lecteurs afin de commencer l'installation (UEFI) 
___
Pour monter les lecteur nous allons utliser la commande `mount`

**Pour monter la partition EXT4**
```shell
mount /dev/sda3 /mnt
```
**Pour monter la partition EFI**
```shell 
mount --mkdir /dev/sda1 /mnt/boot
```
**Pour activer le swap**
```shell
swapon /dev/sda2
```
**Pour monter la partition /home**
```shell 
mount --mkdir /dev/sda4 /mnt/home
```


### Vérification que l'intégrité des paquets lors de l'installations 
___
Il faut désormais installer tout les paquets qui permet de faire fonctionner correctement l'OS mais avant nous allons vérifié que nous avons les bonnes paire de clé pour pouvoir installer les paquets correctement avec le gestionnaire de paquet `pacman`
```shell
pacman-key --init && pacman-key --populate
```
### Modification sur le fichier de configuration de Pacman
___
nano est installer par défaut si vous voulez utiliser vim ou autre faites vous plaisir.
```shell
nano /etc/pacman.conf
``` 
- Décommenter Color (un peu de couleurs ça mets bien quand même)
- Décommenter les autres reposititory en fonction des besoins
core & extra sont présent par défaut, je conseille de décommenter multilib aussi car gère les application 32 bits tel que `steam` et `wine` donc aucune raison de ne pas le prendre 
- Augmenter le nombre de ParallelDownloads en fonction de votre débit

Puis sauvegarder et quitter

### Installation du système de base 
___
Afin d'initialiser le système d'exploitation une bonne fois pour toute lancer la commande, elle permet de lacner le minimun requis pour avoir un arch fonctionnel : 
```shell
pacstrap -K /mnt base linux linux-firmware base-devel networkmanager git curl man fastfetch grub efibootmgr nano sudo
``` 

**Optionnel mais peut être bien `intel-ucode` ou `amd-ucode` en fonction du processeur afin d'éviter de potentiel bug à ce niveau** 

En cas d'erreur à cause d'un réseau trop lent, relancer la commande seul les paquets non installé s'installeront 

### Initialisation des partitions au démarrage
___
Cette commande permettra de monté les bon lecteurs au démmarage.
```shell
genfstab -U /mnt >> /mnt/etc/fstab
```
### Configuration Post-Installation 
___
Afin de terminé l'installation, nous allons directment rentrée dans le système fraichement installé 

**La différence de terminal indique qu'on est rentrée à l'intréieur du système fraichement installé**
```txt
root@archiso ~ # arch-chroot /mnt
[root@archiso /] #
```
### Configuration de la timezone
___
**Lien symbolique vers notre timezone afin qu'elle sois pris par défaut**  
**Permet de synchroniser la clock**
```shell
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtimedate
hwclock --systohc
``` 
### Configuration du clavier
___
**Décommenter la ligne associer à la langue dans notre cas "fr_FR.UTF-8", enregistrer puis quitter**  
**`locale-gen` Permet d'appliquer la modification**

```shell 
nano /etc/locale.gen 
locale-gen 
```
**Permet de changer la langue du login screen**
```shell
echo "KEYMAP=fr" > /etc/vconsole.conf
```
### Configuration de la langue système
___
**Ici, les messages d'erreur seront en Anglais (plus facile pour dépanner)
```shell
nano /etc/locale.conf
LANG=fr_FR.UTF-8
LC_MESSAGES=en_US.UTF-8
```
### Configuration du nom d'hôte 
___
```shell
echo "hostname souhaité" > /etc/hostname
```
### Création d'un utilisateur et définition du mot de passe user & root 
___
Créer un utilisateur avec les droit sudo 
(Bon vous savez qu'en terme de sécurité c'est une faille donc à vos risques et péril)  

**Création d'un utilisateur test affecter au groupe wheel**
```shell 
useradd -m -G wheel test
```
Définition du mot de passe de test
```shell
passwd test
``` 
**Donne accès au fichier sudoers qui va nos permettre de nous faire éxécuter toutes les commandes** 
```shell
EDITOR=nano visudo
Décommenter la ligne    %wheel ALL=(ALL:ALL) ALL
``` 
Permet de définir le mot de passe de root
```shell
passwd
```

## Initialisation du bootloader (GRUB)

Dans le cas d'une installation UEFI :  
**Si vous n'êtes pas sur du efi directory faire un `lsblk` et prendre le chemin qui correspond à la partition efi**
```shell
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB 
```
**Commande qui va permettre générer la configuration de GRUB et qui va rendre le système bootable**
```shell
grub-mkconfig -o /boot/grub/grub.cfg
```
## Redémarrage
**Il ne nous reste plus qu'a activer NetworkManager, sortir de chroot et redémarrer**
```shell
systemctl enable NetworkManager
exit
reboot
```
Et voila une fois le reboot terminer enlever votre clé ou votre ISO et voici une configuration Arch minimal toute prête !  
Si vous trouvez cela trop minimaliste n'hésiter à voir le tuto d'installation d'un environnment graphique type GNOME ou KDE Plasma.

