# Installing Arch Linux

This document covers a very simple, minimal installation of Arch Linux with the following applications:

- AMD GPU Drivers (AMDVLK and Mesa)
- AMD U-Code (CPU)
- KDE and KDE Apps
- Pipewire (Audio)
- Bluez (Bluetooth)
- Neovim (Text Editor)
- Firefox (Web)
- Steam (Gaming)
- VLC (Media Player)
- Alacritty (Terminal)
- YAY (AUR)
- XONE (XBOX Controller Drivers)
- MakeMKV (Rip DVDs and Blu-Rays)
- TimeShift (BTRFS Snapshot Utility)
- ProtonUp-QT (Download various versions of Proton for Steam)
- MangoHud (PC Gaming Stats)

Obviously, you can add and remove anything you'd like.

# Check for IP Address via DHCP
```ip a```

# Set NTP to True
timedatectl set-ntp true
timedatectl status

# Drive Prep

## List all disks - Find name of your drive
### In my case, it's /dev/nvme0n1
fdisk -l

### Extra SATA SSD = /dev/sda
### NVME = /dev/nvme0n1

## Format and partition disk using cfdisk
cfdisk /dev/nvme0n1
600MB - EFI. # Mark as EFI type
???GB - Linux. # Use remaining space

## Write to disk and confirm with 'yes'

## Format partitions
mkfs.fat -F32 /dev/nvme0n1p1. # Boot Partition
mkfs.btrfs -f /dev/nvme0n1p2 -L "arch". # OS Partition

## BTRFS Subvolumes
mount /dev/sda2 /mnt
cd /mnt
btrfs subvolume create @
btrfs subvolume create @home
cd 
umount /mnt

## Mount filesystem AFTER Subvolume Creation
### Mount @
mount -o noatime,space_cache=v2,compress=zstd,ssd,discard=async,subvol=@ /dev/sda2 /mnt

mkdir /mnt/{boot,home}

### Mount @home
mount -o noatime,space_cache=v2,compress=zstd,ssd,discard=async,subvol=@home /dev/sda2 /mnt/home

### Mount EFI
mount /dev/sda1 /mnt/boot

# OS Install

## Update mirror list - Get fastest mirror in US
pacman -Syy. # Update repo
pacman -S reflector
reflector -c "US" -f 12 -l 10 -n 12 --save /etc/pacman.d/mirrorlist

## Install Arch Base
pacstrap /mnt base base-devel linux linux-firmware linux-headers vim git amd-ucode

## Create FSTAB for Drive Mounting
genfstab -U /mnt >> /mnt/etc/fstab

## Root into new install
arch-chroot /mnt

# Customize New Install

## Set timezone - Create symlink
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
hwclock --systohc

## Set locale
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen

## Set hostname
echo "shadow" >> /etc/hostname

## Install and Enable DHCPCD
pacman -S dhcpcd
systemctl enable dhcpcd

## Set root password
passwd

## Create new user
useradd -mg users -G wheel,storage,power -s /bin/bash ascott
passwd ascott

## Add to sudo
visudo
# Add to file
%wheel ALL=(ALL) ALL

## Install GRUB Packages
pacman -S grub efibootmgr dosfstools os-prober mtools
mkdir /boot/efi

## Mount FAT32 EFI partition
mount /dev/sda1 /boot/efi

## Install GRUB to /boot/efi
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub_uefi --recheck

## Create GRUB config
grub-mkconfig -o /boot/grub/grub.cfg

## Unmount and reboot
exit
umount -a
reboot now

# First Boot and Continued Customization

## Make sure that DHCPCD is running
sudo systemctl status dhcpcd
ping -c 3 1.1.1.1

## Enable multilib repo - Uncomment multilib lines
sudo vim /etc/pacman.conf

## Install apps and drivers
sudo pacman -Syy
sudo pacman -S neovim vlc steam plasma xorg pipewire pipewire-alsa pipewire-jack pipewire-pulse mesa amdvlk kde-applications firefox alacritty git bluez bluez-utils handbrake

### If NVIDIA GPU
sudo pacman -S nvidia nvidia-utils nvidia-settings 

## Add modules
sudo nvim /etc/mkinitcpio.conf
MODULES=(btrfs amdgpu). # Change 'amdgpu' to 'nvidia' if needed

## Run mkinitcpio
sudo mkinitcpio -p linux

## Enable and start SDDM, Bluetooth and Network Manager
sudo systemctl enable sddm
sudo systemctl enable NetworkManager
sudo systemctl enable bluetooth

## Bluetooth adjustments
sudo nvim /etc/bluetooth/main.conf
## Uncomment 'FastConnectable = true'
## Uncomment AutoEnable and 2 Reconnect lines

## Install Yay for AUR support
git clone https://aur.archlinux.org/yay.git
cd yay && makepkg -si

## Install XONE for XBOX Controller Drivers
yay -S xone-dkms

## Install MakeMKV
yay -S makemkv

## Install System Fonts
yay -S ttf-ms-win10-auto

## Install Timeshift
yay -S timeshift timeshift-autosnap

## Install ProtonUp
yay -S protonup-qt

## Install Mangohud
sudo pacman -S mangohud

### Mangohud Config
mkdir .config/MangoHud
cp /usr/share/doc/mangohud/MangoHud.conf.example .config/MangoHud/MangoHud.conf

### Edit the file, use preset 4 or 5?
sudo nvim .config/MangoHud/MangoHud.conf

## Final Reboot
sudo reboot now
```
