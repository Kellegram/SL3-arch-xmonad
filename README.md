# SL3-arch-xmonad

## Connect to wifi
    iwctl
      device list
      station stationname scan
      station stationname get-networks
      station stationname connect networkname
  
 ## Sync time
    timedatectl set-ntp true

## Partitioning
    fdisk -l   (lists out the partitions)
    fdisk /dev/nvme0n1
    ## partition as usual
    "m" for help
    "w" (write table to disk)

## Make filesystem:
    mkfs.fat -F32 /dev/nvme0n1p1
    mkswap /dev/nvme0n1p2
    swapon /dev/nvme0n1p2
    mkfs.ext4 /dev/nvme0n1p3

## Base Install:
    mount /dev/nvme0n1p3 /mnt
    pacstrap /mnt base linux linux-firmware
    genfstab -U /mnt >> /mnt/etc/fstab

## Chroot
    arch-chroot /mnt
    ln -sf /usr/share/zoneinfo/EUROPE/London /etc/localtime
    hwclock --systohc
    pacman -S nano
    nano /etc/locale.gen
    locale-gen
    echo "LANG=en_GB.UTF-8" >> /etc/locale.conf
    
## Hostname and Hosts
    echo "kellelap" >> /etc/hostname
    nano /etc/hosts
    VVVVV
    127.0.0.1 localhost
    ::1       localhost
    127.0.1.1 kellelap.localdomain kellelap (replace with your hostname)

## Root pass and new user
    passwd 
    useradd -m kellegram
    passwd kellegram (set that user's password)
    usermod -aG wheel,audio,video,optical,storage kellegram
    
## Grab some tools
    pacman -S network-manager-applet dialog base-devel linux-headers reflector bluez bluez-utils xdg-utils xdg-user-dirs pulseaudio-bluetooth openssh

## Get and configure sudo
    pacman -S sudo
    EDITOR=nano visudo

## Refind
    pacman -S refind 
    pacman -S  efibootmgr dosfstools os-prober mtools
    mkdir /boot/EFI
    mount /dev/nvme0n1p1 /boot/EFI
    refind-install --usedefault /dev/nvme0n1p1 --alldrivers
    
    mkrlconf
    nano /boot/EFI/refind_linux.conf
    ## Delete the first two lines, they are there because of the archiso LE
    
    nano /boot/EFI/BOOT/refind.conf
    ## ctrl+w and search for "arch linux"
    ## Remove the existing PARTUUID and replace with /dev/nvmpe0n1p1
    
## Or Grub
    pacman -S grub
    pacman -S  efibootmgr dosfstools os-prober mtools
    mkdir /boot/EFI
    mount /dev/nvme0n1p1 /boot/EFI
    grub-install --target=x86_64-efi  --bootloader-id=grub_uefi --recheck
    grub-mkconfig -o /boot/grub/grub.cfg

## Networking:
    pacman -S networkmanager
    systemctl enable NetworkManager

## Exit the installer and reboot
    exit
    umount -a
    reboot
    
# Surface patches

## Import the keys used to sign packages, then verify them and sign
    $ wget -qO - https://raw.githubusercontent.com/linux-surface/linux-surface/master/pkg/keys/surface.asc | sudo pacman-key --add -
    $ sudo pacman-key --finger 56C464BAAC421453
    $ sudo pacman-key --lsign-key 56C464BAAC421453

## Add the repo at the end of /etc/pacman.conf
    [linux-surface]
    Server = https://pkg.surfacelinux.com/arch/

## Refresh the repos then grab the packages
    $ sudo pacman -Sy
    $ sudo pacman -S linux-surface linux-surface-headers iptsd libwacom-surface intel-ucode
    $ sudo systemctl enable iptsd

## Enable early KMS to fix certain xorg issues on surface devices
        || /etc/mkinitcpio.conf  ||

        MODULES=(... i915 ...)

## If using grub
    $ sudo grub-mkconfig -o /boot/grub/grub.cfg

## Verify if the right kernel is used
    uname -a
