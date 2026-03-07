# Installation Manuel avec UEFI (Minimal)


## Pré-requis pour le bon déroulement de l'installation

### Permet de passer le clavier en FR (temporairement)
___
```shell
loadkeys fr
```
### Pour savoir si une adresse IP a été récupérer et voir le nom des cartes réseaux
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
```shell
ping www.google.com 
```
> (Problème avec bridge ? Skill issue ?)
> J'ai fait la configuration en NAT je n'ai pas réussi à récupérer une IP sinon

## Début de l'installation

### Mettre en place le partitionnement du disques dur 
```shell
cfdisk
```  

| Type de Partition | Taille de Partition | Type de Formatage |
| :--------------------- | :--------------- | :---------------|
| EFI System| 512 MB ou 1G (en cas de dual-boot ou multi kernel) | FAT32
| Linux swap | En fonction de la RAM (Si 4G de RAM mettre 4G de SWAP ect..) | Linux SWAP
|Linux Filesystem | Tout le reste du disque| EXT4 

La SWAP permet de déplacer des données de la RAM vers le disque pour éviter une surconsommation de la RAM en cas de pic, utile aussi en cas de machine qui peuvent être mis en hibernation.  
ATTENTION : La swap est assez pratique sur cet aspect mais garder en tête que si un virus ou une quelquonque attaque touche votre systeme et charge quelque chose en mémoire cela pourra être garder dans la swap (sur votre disque) je vais surment trop loin mais garder cela en tête.

**SPAMMER PAS ENTREE PAR IMPATIENCE CELA VOUS FERRAIS QUITTER LE MENU DE CFDISK ET TOUT RECOMMENCER**  
- Sélectionner le disque vide avec Entrée
- Saisir la taille de la partition souhaité ( Format : NombreUnité ex: 512MB) puis Entrée
- Naviguer avec les flèches pour changer le type de partition via Type
- Sélectionner la bonne partition (EFI System, Linux Filesystem, Linux swap)
- Répéter le processus pour chaque partition
- Sélectionner Write pour Sauvegarder puis écrire yes
- Quit pour Quitter

### Lister les partitions créer afin de les formater correctement
Afin de continuer il faut correctement identifier les partition de notre disque dur repérer en fonction de la taille 
```shell
lsblk
```  
Dans notre cas:  
sda1 est la partition EFI  
sda2 la partition de swap   
sda3 la partition de EXT4 (Linux Filesystem)  

### Formater les partition sous le bon format
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
### Monter les lecteurs afin de commencer l'installation 
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
Notes : Dans certains tuto pour la partition EFI, il créer le dossier /mnt/boot appart j'ai décidé de le mettre dans une seule et unique commande par facilité 
### Vérification que l'intégrité des paquets lors de l'installations 
Il faut désormais installer tout les paquets qui permet de faire fonctionner correctement l'OS mais avant nous allons vérifié que nous avons les bonnes paire de clé pour pouvoir installer les paquets correctement avec le gestionnaire de paquet `pacman`
```shell
pacman-key --init && pacman-key --populate
```
### Modification sur le fichier de configuration de Pacman
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
Afin d'initialiser le système d'exploitation une bonne fois pour toute lancer la commande, elle permet de lacner le minimun requis pour avoir un arch fonctionnel : 
```shell
pacstrap -K /mnt base linux linux-firmware networkmanager git curl man fastfetch grub efibootmgr nano sudo
``` 
**Optionnel mais peut être bien `intel-ucode` ou `amd-ucode` en fonction du processeur afin de corriger du potentiel bug à ce niveau**  

En cas d'erreur à cause d'un réseau trop lent, relancer la commande seul les paquet non installé s'installeront 

### Initialisation des partitions au démarrage
Cette commande permettra au système d'exploitation quelle lecteur monté au démmarage.
```shell
genfstab -U /mnt >> /mnt/etc/fstab
```
### Configuration Post-Installation 
Afin de terminé l'installation, nous allons directment rentrée dans le système fraichement installé via 
```shell
arch-chroot /mnt
```
#### Configuration de la timezone
```shell
# Lien symbolique vers notre timezone afin qu'elle sois pris par défaut
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtimedate
# Permet de synchroniser la clock 
hwclock --systohc
``` 
#### Configuration du clavier
```shell 
nano /etc/locale.gen 
# Décommenter la ligne associer à la langue dans notre cas "fr_FR.UTF-8", enregistrer puis quitter 
# Permet d'appliquer la modification
locale-gen 
```
```shell
nano /etc/vconsole.conf
KEYMAP=fr
```
#### Configuration de la langue système
```shell
nano /etc/locale.conf
LANG=fr_FR.UTF-8
# Pour laisser les messages d'erreur en Anglais (plus facile pour dépanner)
LC_MESSAGES=en_US.UTF-8
```
#### Configuration du nom d'hôte 
```shell
echo "hostname que tu veux" > /etc/hostname
```
### Création d'un utilisateur et définition du mot de passe user & root 
Créer un utilisateur avec les droit sudo 
(Bon vous savez qu'en terme de sécurité c'est une faille donc à vos risques et péril)
```shell
# Création d'un utilisateur test affecter au groupe wheel 
useradd -m -G wheel test
# Définition du mot de passe de test
passwd test 
# Donne accès au fichier sudoers qui va nos permettre de nous faire éxécuter toutes les commandes
EDITOR=nano visudo
#Décommenter la ligne    %wheel ALL=(ALL:ALL) ALL
``` 
```shell
# Permet de définir le mot de passe de root 
passwd
```
## Initialisation du bootloader (GRUB)
```shell
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB 
# Si vous n'êtes pas sur du efi directory faire un lsblk et prendre le chemin qui correspond à la partition efi
```
```shell
#Commande qui va permettre générer la configuration de GRUB et qui va rendre le système bootable
grub-mkconfig -o /boot/grub/grub.cfg
```
## Pour terminer !
```shell
# Afin d'avoir du réseau quand on va redémarrer !
systemctl enable NetworkManager
# Qui va nous faire sortir de Chroot 
exit
reboot
```
Et voila une fois le reboot terminer enlever votre clé ou votre ISO et voici une configuration Arch minimal toute prête !  
Si vous trouvez cela trop minimaliste n'hésiter à voir le tuto d'installation d'un environnment graphique type GNOME ou KDE Plasma.

