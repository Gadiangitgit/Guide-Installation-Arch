# Installation Manuel avec UEFI (Minimal)


## Pré-requis pour le bon déroulement de l'installation

### Permet de passer le clavier en FR (temporairement)
```shell
loadkeys fr
```
### Pour savoir si une adresse IP a été récupérer et voir le nom des cartes réseaux

```shell
ip a
``` 
**Si vous récupérer bien une adresse IP, passer au ping sinon suivre les indications suivantes**

Si Ethernet :  
Branche ton câble
Si y a une ip cohérante on continue  
Sinon bah check ton câble ou si c'est bien branché man  

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
```shell
lsblk
```  
Ce repérer en fonction de la taille 

Dans notre cas sda1 est la partition EFI
sda2 la partition de swap 
sda3 la partition de EXT4 (Linux Filesystem)

mkfs.ext4 /dev/sda3
mkswap /dev/sda2
mkfs.fat -F 32 /dev/sda1

Une fois ceci effectué, il ne reste plus qu'a mount tout les lecteur correctement

Pour la partition EXT4
mount /dev/sda3 /mnt
Pour la partition EFI
mount --mkdir /dev/sda1 /mnt/boot
Pour la swap
swapon /dev/sda2

Ensuite il faut désormais installer tout les paquets qui permet de faire fonctionner correctement l'OS
mais avant vérifié que nous avons les bonne piare de clé pour pouvoir installer les paquets correcctement sous pacman

pacman-key --init
pacman-key --populate
nano /etc/pacman.conf
-> Décommenter Color
-> Décommenter les autres reposititory en fonction des besoins
-> Augmenter le nombre de ParallelDownloads au besoin 
Puis sauvegarder et quitter

pacstrap -K /mnt base linux linux-firmware networkmanager git curl man fastfetch grub efibootmgr nano sudo
Optionnel mais peut être bien 
intel-ucode ou amd-ucode en fonction du processeur 

Une fois l'install terminée, il faut faire 
genfstab -U /mnt >> /mnt/etc/fstab

Et nous allons pourvoir rentrée dans la machine via
arch-chroot /mnt

Nous allons faire désormais un lien symbolic vers notre timezone afin qu'elle sois pris par défaut

ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtimedate
hwclock --systohc <- Permet de synchroniser la clock 
nano /etc/locale.gen , Décommenter la ligne associer à la langue dans notre cas fr_FR.UTF-8, enregistrer puis quitter 
locale-gen <- permet d'appliquer la modification
nano /etc/locale.conf
-> LANG=fr_FR.UTF-8
nano /etc/vconsole.conf
-> KEYMAP=fr
POur définir le hostname 
echo "hostname que tu veux" > /etc/hostname

Ensuite définir un mot de passe pour le compte root 
passwd 
Créer un utilisateur avec les droit sudo (déconseillé mais vous êtes des grands)
useradd -m -G wheel test
passwd test 
EDITOR=nano visudo
Décommenter la ligne %wheel ALL=(ALL:ALL) ALL
systemctl enable NetworkManager

Ensuite il va falloir installer le bootloader GRUB via 
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB (Si pas sur du efi directory faire un lsblk correspond à la partition efi)
grub-mkconfig -o /boot/grub/grub.cfg

Il ne reste plus qu'a quitter chroot avec exit
et reboot et on est bon 

