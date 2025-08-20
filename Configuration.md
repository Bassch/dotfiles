# My setup
## Desktop
Hardware: 
Applications: 
- Windows
	- p
- Arch
	- yay
	- steam
	- fIrefox
	- obsidian
	- hyprland
	- wayland
	- xorg
	- kitty
	- sddm
	- piper
	- spotify
	- discord
	- webcord
	- gimp
	- Krita
	- Inkscape
	- libreoffice
	- vlc
	- keepass
	- git
	- neofetch
	- neovim
	- reflector
	- anki
	- nvidia
	- qalculate-qt
	- telegram-desktop
	- xorg-xwayland

## Laptop
Hardware: 
Applications: 

## Peripherals
Phone:
Router?
Raspberry Pi's?

---
# Configuring Arch/Hyprland
As with every installation, use pacman -Syu

## Storage
![[Disk partitioning layout.png]]
>sudo parted /dev/nvme0n1p1 print
>sudo parted /dev/nvme0n1p1
>udevadm settle
>mkfs.ext4 /dev/nvme0n1p1
>mkfs.ext4 /dev/nvme0n1p2
>sudo mkdir /mnt/data
>sudo mkdir /mnt/games
>sudo lsblk --fs | grep nvme0n1 >> /etc/fstab
>sudo chown bas /mnt/data
>sudo chown bas /mnt/games

I created two 1GB partitions for the /efi and /boot mount point, 4GB for swap and the rest as lvm.

## Hyde
git clone --depth 1 https://github.com/HyDE-Project/HyDE
sudo ./install.sh

## Monitors
> ~/.config/hypr/monitors.cfg
>monitor = DVI-D-1,1920x1080@60,2560x600,auto
monitor = DP-2,2560x1440@143.856,0x0,auto
workspace = 1,monitor:DP-2
workspace - 2,monitor:DVI-D-1

## Applications
In general for all packages to install through pacman use:
>sudo pacman -Sy yay

### Kitty
.config/kitty/kitty.conf
font_size 11

.config/kitty/hyde.conf
font-size 9.0
window_padding_width 25

### Steam
>Try different compatibility layers of proton while booting games

### Firefox

### Obsidian


---

## Arch installation

### Download & creating boot drive
Just download the magnet-link. There's less chance of a faulty .iso

From Windows: Use Rufus
From Linux: Use dd

### Performing installation
1. Check network availability by looking up the ip `ip link` and pinging `ping archlinux.org -c 4`
2. Set the local system time `timedatectl`  and `timedatectl set-timezone Europe/Amsterdam`
3. Identify and partition the installation disk `lsblk` and `# fdisk _/dev/the_disk_to_be_partitioned_`
	1. `g` to create a new empty GPT partition table
	2. Setup the partition according to the Storage layout above
	3. Also format any 2nd disk you have as lvm so it can be added later
4. Format the partitions using `mkfs.fat -F32 /dev/firstpartition` and then `mkfs.ext4 /dev/secondpartition` then `mkswap /dev/thirdpartition` , `swapon /dev/thirdpartition`
5. -- Skipping encryption on desktop for now -- (On a laptop the installation would effect the lvm configuration so that should be different)
6. Create lvm drives using `pvcreate /dev/lastpartition` , `vgcreate volgroup0 /dev/lastpartition /dev/otherpartitions`, `lvcreate -L 30GB volgroup0 -n lv_root` `lvcreate -L 1500GB volgroup0 -n lv_home` (leave some lv storage empty for later projects)
7. `modproble dm_mod` & `vgscan` & `vgchange -ay`
8. format both lvs like so `mkfs.ext4 /dev/volgroup0/lv_x
9. mount lvs to /mnt `mount /dev/volgroup0/lv_root /mnt` , `mkdir /mnt/boot` , `mount /dev/secondpartition /mnt/boot` , `mkdir /mnt/home` `mount /dev/volgroup0/lv_home /mnt/home`
10. `reflector --country COUNTRY --latest 20 --save /etc/pacman.d/mirrorlist --sort rate --verbose`
11. initialize and populate the keyring `pacman-key --init` , `pacman-key --populate`
12. Install essential packages `pacstrap -i /mnt base linux linux-firmware base-devel grub efibootmgr vim networkmanager`
13. Generate the fstab file `genfstab -U -p /mnt >> /mnt/etc/fstab`
14. Change the root location into the installation `arch-chroot /mnt`
15. Edit /etc/locale.gen so it contains `en_US.UTF-8`, `de_DE.UTF-8 UTF-8`,  `ja_JP.UTF-8 UTF-8` and create `/etc/locale.conf` containing the main LANG `LANG=en_US.UTF-8`
	1. execute `locale-gen`
16. create `/etc/hostname` containing the hostname
	1. `archdesktop` or `archlaptop`
17. Set root password `passwd`
18. Create the to-be-user `useradd -m -g users -G wheel username` and `passwd username`
19. Execute `EDITOR=vim visudo` and uncomment `%wheel ALL=(ALL) ALL`
20. Set time again `ln -sf /usr/share/zoneinfo/Europe/Amsterdam /etc/localtime`, `hwclock --systohc`
21. Populate the keyring again `pacman-key --init` , `pacman-key --populate`
22. Install packages `pacman -S sddm dosfstools lvm2 zsh mtools openssh os-prober sudo pipewire wireplumber pipewire-audio pipewire-jack pipewire-alsa pipewire-pulse pavucontrol git btop wget zip unzip bash-completion python reflector man-db man-pages noto-fonts noto-fonts-emoji noto-fonts-cjk`
23. Install GPU packages 
	1. On my nvidia PC: `pacman -S nvidia nvidia-utils nvidia-settings`
24. Edit mkinitcpio `/etc/mkinitcpio.conf` and append to the HOOKS line `lvm2`
	1. Also add encrypt when using encryption
	2. Generate stuff for the kernel(s) `mkinitcpio -P
		1. For encryption: edit `/etc/default/grub` and append to `GRUB_CMDLIN_LINUX_DEFAULT` and add the cryptdevice
25. Setup the efi partition `mkdir /boot/EFI` and `mount /dev/firstpartition /boot/EFI`
26. Install the bootloader
	1. Install grub `grub-install --efi-directory=/boot/EFI --target=x86_64-efi --bootloader-id=grub_uefi --recheck`
	2. Copy into boot directory `cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo`
	3. Generate grub config file `grub-mkconfig -o /boot/grub/grub.cfg`
27. systemctl enable
	1. `NetworkManager`
	2. `sddm.service`
	3. `sshd.service`
28. Exit, use `umount -R /mnt` and `reboot`
29. Optional: Pray that the installation succeeded
30. Install hyprland packages `pacman -S hyprland hyprpaper xdg-desktop-portal-hyprland waybar wofi wayland wayland-protocols xorg-xwayland kitty dunst hyprpolkitagent qt5-wayland qt6-wayland rofi-wayland hyprpicker hyprlock dolphin grim polkit-kde-agent slurp uwsm iwd smartmontools xdg-desktop-portal-hyprland xdg-utils wpa-supplicant`
31. edit /etc/pacman.conf to include `[multilib]` and run `pacman -Sy`
32. Install personal packages `pacman -S steam proton-steam discord spotify-launcher qbittorrent firefox obs-studio obsidian neovim kdenlive audacity anki telegram-desktop keepass libreoffice gimp vlc krita 
33. git clone yay `git clone https://aur/archlinux.org/yay.git`
	1. 
34. From the Arch User Repository you can install `anki` , `proton-steam`, `yay`

