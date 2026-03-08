```bash
loadkeys fr
ip a
ping archlinux.org 
cfdisk
lsblk
mkfs.ext4 /dev/sda?
mkswap /dev/sda?
mkfs.fat -F 32 /dev/sda?
mount /dev/sda? /mnt
mount --mkdir /dev/sda? /mnt/boot
swapon /dev/sda?
pacman-key --init && pacman-key --populate
nano /etc/pacman.conf
pacstrap -K /mnt base linux linux-firmware networkmanager git curl man fastfetch grub efibootmgr nano sudo
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtimedate
hwclock --systohc
nano /etc/locale.gen
locale-gen 
nano /etc/vconsole.conf
nano /etc/locale.conf
echo "hostname souhaité" > /etc/hostname
useradd -m -G wheel test
passwd test 
EDITOR=nano visudo
passwd
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB 
grub-mkconfig -o /boot/grub/grub.cfg
systemctl enable NetworkManager
exit
reboot
```