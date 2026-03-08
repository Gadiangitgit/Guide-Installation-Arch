# Guide d'installation de Arch-Linux UEFI (Minimal)


## Pré-requis pour le bon déroulement de l'installation

**Permet de passer le clavier en FR (temporairement)**
___
```shell
loadkeys fr
```
Pour agrandir la police en fonction de la taille de l'écran
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
> (Problème avec bridge ? Skill issue ?)
> J'ai fait la configuration en NAT je n'ai pas réussi à récupérer une IP sinon

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
Afin de continuer il faut correctement identifier les partitions de notre disque dur repérer en fonction de la taille 
```shell
lsblk
```  
Dans notre cas UEFI : 
sda1 est la partition EFI  
sda2 est la partition de swap   
sda3 est la partition de EXT4 (Linux Filesystem) 

Dans notre cas UEFI Chiffré :
sda1 est la partition EFI
sda2 est la partition EXT4 (Linux Filesystem)
(La swap sera créer plus tard et chiffré)



### Mettre en place le partitionnement du disques dur en UEFI
Il est très important de préciser le disque par défaut pour être sur de ne pas ce tromper de périphérique d'installation  
- /sda = disque HDD & SSD
- /nvme0n? = disque NVME
- /mmcblk? = carte SD (qui fait ça sérieux ?)
___
Dans notre cas ça sera /dev/sda n'hésiter pas à adapter en fonction de votre disque
```shell
cfdisk /dev/sda 
```  

Sélectionner GPT (C'est marrant hein ?)

 En cas de disque non chiffré sans /home séparer 
| Type de Partition | Taille de Partition | Type de Formatage |
| :--------------------- | :--------------- | :---------------|
| EFI System| 512 MB ou 1G (en cas de dual-boot ou multi kernel) | FAT32
| Linux swap | En fonction de la RAM (Si 4G de RAM mettre 4G de SWAP ect..) | Linux SWAP
|Linux Filesystem | Tout le reste du disque| EXT4 

En cas de disque non chiffré et /home séparer 
| Type de Partition | Taille de Partition | Type de Formatage |
| :--------------------- | :--------------- | :---------------|
| EFI System| 512 MB ou 1G (en cas de dual-boot ou multi kernel) | FAT32
| Linux swap | En fonction de la RAM (Si 4G de RAM mettre 4G de SWAP ect..) | Linux SWAP
|Linux Filesystem | +-30G | EXT4 
|Linux Filesystem | Tout le reste du disque| EXT4 

 En cas de disque chiffré et /home séparer ou non 
| Type de Partition | Taille de Partition | Type de Formatage |
| :--------------------- | :--------------- | :---------------|
| EFI System| 512 MB ou 1G (en cas de dual-boot ou multi kernel) | FAT32
|Linux Filesystem | Tout le reste du disque| EXT4 


**SPAMMER PAS ENTREE PAR IMPATIENCE CELA VOUS FERRAIS QUITTER LE MENU DE CFDISK ET TOUT RECOMMENCER**  
- Sélectionner le disque vide avec Entrée
- Saisir la taille de la partition souhaité ( Format : NombreUnité ex: 512MB) puis Entrée
- Naviguer avec les flèches pour changer le type de partition via Type
- Sélectionner la bonne partition (EFI System, Linux Filesystem, Linux swap)
- Répéter le processus pour chaque partition
- Sélectionner Write pour Sauvegarder puis écrire yes
- Quit pour Quitter


### Configuration supplémentaire pour le volume Chiffré
___
La manière de procéder est totalement différente
```shell
#Pour formater notre partition/dev/sda2  avec LUKS2 
cryptsetup luksFormat --type luks2 --pbkdf pbkdf2 /dev/sda2
```
--pbkdf pbkdf2 obligatoire pour la compatibilté avec GRUB
Ecrire `YES`
Rentrer le mot de passe qui va être demander lors de chaque connexion

```shell
# Rentrer dans le volume chiffré
cryptsetup luksOpen /dev/sda2 cryptlvm
```
```shell
# Créer le volume LVM
pvcreate /dev/mapper/cryptlvm
```
```shell
# Créer le groupe de volume
vgcreate vg0 /dev/mapper/cryptlvm
```
Une fois terminée nous allons créer nos chiffré en focntion du besoin (ne faites pas la commande pour le swap ou le home si vous n'en voulez pas)
```shell
# ?G à adapter selon la RAM, la deuxième commande va prendre toutce qui reste, la dernière permet de vérifier que les paramètres ont bien été appliquée 
lvcreate -L ?G vg0 -n swap  
lvcreate -L 30G vg0 -n root    
lvcreate -l 100%FREE vg0 -n home 
lvs
```

### Formater les partitions sous le bon format (UEFI)
___
```shell
#Pour formater notre partition/dev/sda3 en EXT4 
mkfs.ext4 /dev/sda3
```
```shell
# Pour formater notre /dev/sda2 en swap
mkswap /dev/sda2
```
```shell
# Pour formater le /dev/sda1 en FAT32
mkfs.fat -F 32 /dev/sda1
```
Si /home séparer 
```shell
mkfs.ext4 /dev/sda4
```
### Formater les partitions sous le bon format (UEFI Chiffré)
___
```shell
#Pour formater notre partition root en EXT4 
mkfs.ext4 /dev/vg0/root
```
```shell
# Pour formater la partition swap en swap
mkswap /dev/vg0/swap
```
```shell
# Pour formater le /dev/sda1 en FAT32
mkfs.fat -F 32 /dev/sda1
```
En cas de /home séparée 
```shell
#Pour formater notre partition home en EXT4 
mkfs.ext4 /dev/vg0/home
```

### Monter les lecteurs afin de commencer l'installation (UEFI) 
___
Pour monter les lecteur nous allons utliser la commande `mount`

```shell
# Pour monter la partition EXT4
mount /dev/sda3 /mnt
```
```shell 
# Pour monter la partition EFI
mount --mkdir /dev/sda1 /mnt/boot
```
```shell
# Pour activer le swap
swapon /dev/sda2
```
```shell 
# Pour monter la partition /home
mount --mkdir /dev/sda4 /mnt/home
```
Notes : Dans certains tuto pour la partition EFI, il créer le dossier /mnt/boot appart j'ai décidé de le mettre dans une seule et unique commande par facilité

### Monter les lecteurs afin de commencer l'installation (UEFI Chiffré) 
___
Pour monter les lecteur nous allons utliser la commande `mount`

```shell
# Pour monter la partition EXT4
mount /dev/vg0/root /mnt
```
```shell 
# Pour monter la partition EFI
mount --mkdir /dev/sda1 /mnt/boot
```
```shell
# Pour activer le swap
swapon /dev/vg0/swap
```
```shell 
# Pour monter la partition EFI
mount --mkdir /dev/vg0/home /mnt/home

```shell
# Pour vérifier que tout est en place les 2 partitions doivent apparaitre
lvs 
pvs
```
Notes : Dans certains tuto pour la partition EFI, il créer le dossier /mnt/boot appart j'ai décidé de le mettre dans une seule et unique commande par facilité 

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
core & extra sont présent par défaut, je conseille de décommenter multilib aussi car gère les application 32 bits tel que `steam` et `wine` et stable donc aucune raison de ne pas le prendre 
- Augmenter le nombre de ParallelDownloads en fonction de votre débit

Puis sauvegarder et quitter

### Installation du système de base 
___
Afin d'initialiser le système d'exploitation une bonne fois pour toute lancer la commande, elle permet de lacner le minimun requis pour avoir un arch fonctionnel : 
```shell
pacstrap -K /mnt base linux linux-firmware base-devel networkmanager git curl man fastfetch grub efibootmgr nano sudo
``` 
**Si on chiffre le système les paquets `lvm2 cryptsetup` sont à rajouter**
**Si on chiffre le système avec le TPM les paquets `tpm2-tools tpm2-tss` sont à rajouter en plus de `lvm2 cryptsetup`**

**Optionnel mais peut être bien `intel-ucode` ou `amd-ucode` en fonction du processeur afin d'éviter de potentiel bug à ce niveau** 

En cas d'erreur à cause d'un réseau trop lent, relancer la commande seul les paquets non installé s'installeront 

### Initialisation des partitions au démarrage
___
Cette commande permettra au système d'exploitation quelle lecteur monté au démmarage.
```shell
genfstab -U /mnt >> /mnt/etc/fstab
```
### Configuration Post-Installation 
___
Afin de terminé l'installation, nous allons directment rentrée dans le système fraichement installé via 
```shell
root@archiso ~ # arch-chroot /mnt
# La différence de terminal indique qu'on est rentrée à l'intréieur du système fraichement installé
[root@archiso /] #
```
#### Configuration de la timezone
___
```shell
# Lien symbolique vers notre timezone afin qu'elle sois pris par défaut
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtimedate
# Permet de synchroniser la clock 
hwclock --systohc
``` 
#### Configuration du clavier
___
```shell 
nano /etc/locale.gen 
# Décommenter la ligne associer à la langue dans notre cas "fr_FR.UTF-8", enregistrer puis quitter 
# Permet d'appliquer la modification
locale-gen 
```
```shell
echo "KEYMAP=fr" > /etc/vconsole.conf
```
#### Configuration de la langue système
___
```shell
nano /etc/locale.conf
LANG=fr_FR.UTF-8
# Pour laisser les messages d'erreur en Anglais (plus facile pour dépanner)
LC_MESSAGES=en_US.UTF-8
```
#### Configuration du nom d'hôte 
___
```shell
echo "hostname souhaité" > /etc/hostname
```
### Création d'un utilisateur et définition du mot de passe user & root 
___
Créer un utilisateur avec les droit sudo 
(Bon vous savez qu'en terme de sécurité c'est une faille donc à vos risques et péril)
```shell
# Création d'un utilisateur test affecter au groupe wheel 
useradd -m -G wheel test
# Définition du mot de passe de test
passwd test 
# Donne accès au fichier sudoers qui va nos permettre de nous faire éxécuter toutes les commandes
EDITOR=nano visudo
Décommenter la ligne    %wheel ALL=(ALL:ALL) ALL
``` 
```shell
# Permet de définir le mot de passe de root 
passwd
```
## Configuration de mkinitpio
Editer le hook de initramfs pour pouvoir le faire supporter le LUKS et LVM
```shell
nano /etc/mkinitcpio.conf
```
Modifier la ligne `HOOKS` pour que cela ressemble EXCATEMENNT A CECI (l'ordre est important)
```shell
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block sd-encrypt lvm2 filesystems fsck)
```
Plus qu'a regénérer la configuration via 
```shell
mkinitcpio -P
```
## Préconfiguration de GRUB (UEFI chiffré)
Pour faire en sorte que grub boot correctement sur la partition chiffré nous allons devoir extraire l'UUID qui nous allons rentrer dans la configuration de GRUB et pour ceci nous allons utiliser la commande 
```shell
blkid /dev/sda2
UUUID-SDA2
```
Modifier la configuration de GRUB
```shell
nano /etc/default/grub
```
Modifier la ligne `GRUB_CMDLINE_LINUX` :
```shell
GRUB_CMDLINE_LINUX="rd.luks.name=UUID-SDA2=cryptlvm rd.luks.options=UUID-SDA2=tpm2-device=auto root=/dev/vg0/root"
```
Toujours dans le fichier grub décommentée la ligne 
```shell
GRUB_ENABLE_CRYPTODISK=y
```
Si jamais vous voulez juste un disque chiffré c'était la dernière étape Bravo !
Pour l'implémentation de la TPM2 on continue !

## Initialisation du bootloader (GRUB)

Dans le cas d'une installation UEFI :
```shell
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB 
# Si vous n'êtes pas sur du efi directory faire un lsblk et prendre le chemin qui correspond à la partition efi
```
```shell
#Commande qui va permettre générer la configuration de GRUB et qui va rendre le système bootable
grub-mkconfig -o /boot/grub/grub.cfg
```
## Redémarrage 
```shell
# Afin d'avoir du réseau quand on va redémarrer !
systemctl enable NetworkManager
# Qui va nous faire sortir de Chroot 
exit
reboot
```
## Post-Installation 
Lors du redémarrage, saisir le mot de passe défini lors de la section [cette partie](#configuration-supplémentaire-pour-le-volume-chiffré) et les credentials de votre utilisateurs nous allons bien vérifier que la TPM est toujours détecter
```shell
sudo systemd-cryptenroll --tpm2-device=list
```
Nous allons générer la clé de récuparation **GARDER LA EN LIEUX SUR ELLE VOUS SERA UTILE EN CAS DE PROBLEME !!!**
```shell
sudo systemd-cryptenroll --recovery-key /dev/sda2
```
Mettre en place la TPM2 avec un code PIN **(QUI PRENDS CHIFFRE ET LETTRES !!)**
Sur votre PC éxécuter la première commande car cela rajoute l'état du secure boot
La seconde ne passera pas par le secure boot, on perd donc l'intérêt mais pratique sur VM
>VMware caprisieux avec le secure boot 
```shell
sudo systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+7 --tpm2-with-pin=yes /dev/sda2

sudo systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0 --tpm2-with-pin=yes /dev/sda2
```
Va permettre de vérifier que le TPM est bien mis en place
```shell
cryptsetup luksDump /dev/sda2
```
Plus qu'a reboot afin de voir si cela est corrctement appliqué !  

NE PAS OUBLIER DE REACTIVER LE SECURE BOOT DANS LE BIOS 

Au prochain démarrage, le TPM demandera le PIN définit plutôt.
## En cas de problème via une mise à jour firmware ou bios 
```shell
Utilisé la clé récupération générer plus haut
systemd-cryptenroll --wipe-slot=tpm2 /dev/sda2
systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+7 --tpm2-with-pin=yes /dev/sda2
```
Et voila une fois le reboot terminer enlever votre clé ou votre ISO et voici une configuration Arch minimal toute prête !  
Si vous trouvez cela trop minimaliste n'hésiter à voir le tuto d'installation d'un environnment graphique type GNOME ou KDE Plasma.

