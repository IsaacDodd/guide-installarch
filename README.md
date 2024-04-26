

# Guide: Installing Arch Linux

For the sake of learning, this guide will walk you through a plain/bare installation of Arch Linux on a system without Windows (or where you will delete the Windows installation). This can be done in a [virtual machine/container](https://www.redhat.com/en/topics/containers/containers-vs-vms) for practice.

This closely follows the current official [Arch Wiki Installation Guide](https://wiki.archlinux.org/title/Installation_guide) at the time of this writing but with some additional precautionary steps. However, it should not be used as a replacement for the official guide as the official guide could get updated and also contains many helpful pages to read.

**Why?**: The advantage of learning this is so that one can quickly and efficiently install Arch Linux but also have a mental model with 'modification points'. This will allow one to deviate from the canonical means of installation in order to install differences (e.g., to install an EFI stub instead of Grub, etc.).

**Mnemonic**: Remember the steps to install 'vanilla' Arch Linux with a mnemonic.

> **FFIT-K ParF MoP, FSCh-TZL, PHIR-US MB DE-D SURE**

> **F**ont
> **F**irmware
> **I**nternet Connection
> **T**ime Synchronization
> **K**eymaps
> 
> **Par**tition
> **F**ormat
> **Mo**unt
> **P**acstrap

> **FS**Tab/File System Table
> **Ch**root
> **TZ** Time Zone
> **L**ocalization/Language

> **P**ersistant Keymap
> **H**ostname
> **I**nitramfs
> **R**oot Account
> **U**ser Account
> **S**udo Setup

> **M**icrocode for CPU
> **B**oot Manager

> **DE** Desktop Environment
> **D**efault Aplications
> **S**ystem Services
> **U**nmount
> **R**eboot/Restart
> **E**nd


This mnemonic serves as a way to remember the bare "skeleton" of an Arch Installation in order to remember the steps.

As you learn more, each step can be varied (e.g., changing the file system installed, adding encryption to the partition step, or adding additional packages to the `pacstrap` step, etc.), but the general order and goals derived from the Arch Wiki Installation Guide will remain the same.


**[F]ont**:

The following sets the Arch installer to a more readable font. 

```bash
setfont sun12x22
```

**Verify [F]irmware/Boot Mode**:

Verify that you are in an EFI system. Type: `ls /sys/firmware/efi/efivars`

If you see an error, and it's not due to being improperly typed, it means your firmware is non-UEFI. Please follows steps to convert from an MBR to UEFI firmware before continuing.

**[I]nternet Connectivity**:

Preferably internet should be established with the `iwctl` command, or use `nmtui`. For iwctl, `device list` > `station device scan` > `station device get-networks` > `station device connect <SSID>`. For nmtui, use the TUI to connect to a selected SSID and enter the username and password. If neither of these commands work, you can connect a USB-C adapter with an ethernet connector and connect to your router directly as a last resort.

Enter the following:

```bash
ip link
ip addr
ping archlinux.org -c 5 
```

The first two commands should produce an itemized list. If you only see '1: lo:' it means you are not connected to the internet. The third command attempts to ping the Arch Linux website 5 times. If the pings don"t complete (i.e., it should say "5 packets transmitted, 5 received") it also means you are not connected to the internet. Please follow steps to start an internet connection.

**[T]ime Synchronization**: You should then set the time to synchronize properly. This is important for encrypted connections.

```bash
timedatectl set-ntp true
```


**[K]eymaps**:

You can set the console keymaps for the country to which your keyboard belongs, such as us for the USA.

```bash
localectl list-keymaps
loadkeys us
```

Later on this will be set to persist into the Arch Linux installation via `/etc/vconsole.conf`.

**[Par]tition the Disk**:

Now partition the disk.

```bash
fdisk -l
lsblk
```

Both of these commands will spit out relatively the same list of disks available with their sizes but in different ways (for fdisk -l it displays the partition table). Choose the disk you want to partition.

> 🗒️**Note**: This example assumes the chosen partition is `/dev/sda`, but update the following commands accordingly if your disk address is otherwise. 
> 🗒️**Single/Non-Dual Boot**: If you are dual-booting, do not run these steps as it will erase the drive.

Option 1: cfdisk
The easiest way to partition the disk is by typing in `cfdisk`.

Option 2: fdisk
`fdisk /dev/sda` This will start fdisk in its interactive mode. 
`g` This will create a new GPT disklabel.
`F` This is similar to `lsblk`, listing 
`w` This will allow fdisk to commit the label to the disk.
Type `m` if you want to see the available list of single-letter commands. This will let you use `n` to setup new partitions.


**[F]ormat the Partitions**: If dual-booting, **do not** format the Windows partitions.

```bash
mkfs.ext4 /dev/sda3
mkswap /dev/sda2
mkfs.fat -F 32 /dev/sda1
```

**[Mo]unt the Partitions**:

```bash
mount /dev/sda3 /mnt
swapon /dev/sda2
mkdir -p /mnt/boot
mount /dev/sda1 /mnt/boot
```


[P]acstrap: **Install the Base Packages** via the pacstrap command.

Get the latest lists for the repositories from the mirror lists first:

```bash
pacman -Syy
```

Use the `pacstrap` command to install the initial packages on the basic arch system.

```bash
pacstrap /mnt base base-devel linux linux-firmware sudo grub efibootmgr os-prober sof-firmware gvim nano man-db man-pages texinfo git gnupg networkmanager dosfstools curl wget mtools
```

> 🗒️**Note**: It is sometimes recommended to install GRUB above to have a bootloader as a backup, even if you only plan on installing a boot manager such as systemd-boot, rEFInd, or just using EFISTUB. However, if Grub is installed above, the system will always update Grub when booting up when it has been updated on the last system update, slowing down the boot process. Therefore, if you are installing a different bootloader/manager, either leave grub out of the pacstrap line or remove grub with `sudo pacman -R grub` later on.

**Generate the [FS]Tab**

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

---

**[Ch]root: Switch to the Installed Linux**: 

```bash
arch-chroot /mnt
```

Welcome to your new Arch Linux installation!

**[TZ] Time Zone: Set the Hardware Clock to the Local Time**

```bash
ln -sf /usr/share/zoneinfo/<Region>/<City> /etc/localtime
hwclock --systohc
```

**[L]ocalization/Language**

Edit /etc/locale.gen and uncomment en_US.UTF-8 UTF-8 or other needed locale.

```bash
locale-gen
```

Or alternatively, just run this command:

```bash
echo 'LANG=en_US.UTF-8' >> /etc/locale.conf
```

**[P]ersistant Keymap (Persist the Keymap)**

Edit the file /etc/vconsole.conf and enter: `KEYMAP=us`

Or alternatively, just run this command:

```bash
echo 'KEYMAP=us' >> /etc/vconsole.conf
```

**[H]ostname**
```bash
echo '<computer-name>' > /etc/hostname
```

```bash
127.0.0.1        localhost
::1              localhost
127.0.1.1        <computer-name>.localdomain <computer-name>
```
If the system has a permanent IP address, it should be used in place of `127.0.1.1` above.

**[I]nitramfs**

Make the initial RAM file system that creates the early environment. This is called by the kernel in the Pacstrap command already, but remembering this step becomes important when encryption or logical volumes are introduced to these basic steps.

```bash
mkinitcpio -P
```

**[R]oot Account**
Set the root user password:

```bash
passwd
```

Type in a new password for the root account. Retype it to confirm.

**[U]ser Account**
Create your first regular, non-root, user account. This will be the first account into which you will log in when your device boots up.

```bash
useradd -m -G wheel <username>
passwd <username>
```

Set a password for this account, which can be different from the root password. 

**[S]udo Setup**

```bash
EDITOR=nano visudo
```
Option 1: If you want to by default invoke 'sudo' for sudo privileged commands (with the root user's password; recommended), scroll down the file and find this line: 
> # %wheel ALL=(ALL) ALL
Uncomment it by removing the # sign. Save the file.

Option 2: If you want to enable the regular user to have sudo privileges (not recommended), scroll down the file and find this line:

> # %sudo ALL=(ALL) ALL

Uncomment it by removing the # sign. Save the file.


**[M]icrocode for CPU**
1. Install CPU microcode update package

```bash
cat /proc/cpuinfo | grep 'model name'
# Based on the type of CPU above, install one of the two below:
# Option 1: Intel CPU
sudo pacman -S intel-ucode
# Option 2: AMD CPU
sudo pacman -S amd-ucode
```

**[B]oot Manager / Bootloader Installation**

If installing in a virtual machine, you may want to change from boot to bios_grub if you did not explicitly start the installation in an EFI mode.

> parted /dev/sda
> set 1 boot off
> set 1 bios_grub on
> q

Then, run the following:

```bash
# Option 1: For UEFI systems with a GPT partition table.
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB 
#change the directory to /boot/efi if you mounted the EFI partition at /boot/efi

# Option 2: Just for reference: MBR/non-efi (or a virtual machine without EFI) - Replace the below <X> with the letter on which your partitions reside, 'a' in this example.
grub-install --bootloader-id=GRUB --boot-directory=/boot --target=i386-pc /dev/sd<X>

# Generate configuration file
grub-mkconfig -o /boot/grub/grub.cfg
```

**[DE] Desktop Environment**

Install a Desktop Environment (DE). The default/suggested DE by this guide is KDE Plasma, but if you know how to install other DE's, such as Gnome, do those instead. Others choose also to install a Window manager or change the default window manager of the desktop environment at this step.

```bash
pacman -S --noconfirm xorg sddm plasma kde-applications-meta plasma-wayland-session
# Optional packages
pacman -S --noconfirm materia-kde papirus-icon-theme
pacman -S --noconfirm xdg-user-dirs packagekit-qt5
```

If you would rather install the Gnome desktop environment, instead of kde plasma choose `gdm gnome gnome-extra`.

By default, sddm uses its own failsafe login screen and that is not the typical KDE Plasma “breeze” login screen. A small tweak is needed for a KDE Plasma login screen.

Edit the sddm config file as below:

```bash
sudo nano /usr/lib/sddm/sddm.conf.d/default.conf
```

Edit it to:

> [Theme]
> # Current theme name:
> Current=breeze

> 🔖**References**:
> - [How to Install and Configure KDE Plasma Desktop in Arch Linux [Complete Guide]](https://www.debugpoint.com/2021/01/kde-plasma-arch-linux-install/)

**[D]efault Applications: Install some important packages**

This step takes the format of:

```bash
# Important Packages
pacman -S <Packages>
```

Install just a few simple applications so that they're available on the next reboot. Choose the ones you would like based on the [Arch Package Search](https://archlinux.org/packages/).

```bash
# Browser & Video tools
pacman -S firefox
pacman -S mpv
# Network tools
pacman -S network-manager-applet dialog wpa_supplicant 
# For Laptops/Tablets: power optimization to save battery life. If using on a desktop or virtual machine, this can be commented out.
pacman -S tlp
# Other tools: you can string them together
pacman -S avahi xdg-user-dirs xdg-utils gvfs gvfs-smb nfs-utils inetutils dnsutils bluez bluez-utils cups hplip alsa-utils pipewire pipewire-alsa pipewire-pulse pipewire-jack bash-completion openssh rsync reflector acpi acpi_call virt-manager qemu qemu-arch-extra edk2-ovmf bridge-utils dnsmasq vde2 openbsd-netcat iptables-nft ipset firewalld flatpak sof-firmware nss-mdns acpid ntfs-3g terminus-font
```

This is a step where you can install an AUR helper which is a wrapper program to Pacman that also accesses the AUR (since Pacman only installs from the official Arch repositories)

```bash
# AUR installer (paru, pikaur, yay, or pamac are some good ones)
pacman -S --noconfirm rustup
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si --noconfirm
cd .. 
rm -rf paru
```

**[S] System Services / systemctl Commands**


```bash
# Enable the display server and network manager.
systemctl enable sddm
systemctl enable NetworkManager

# Turn on other services
systemctl enable bluetooth
systemctl enable cups.service
systemctl enable sshd
systemctl enable avahi-daemon
systemctl enable reflector.timer
systemctl enable fstrim.timer
systemctl enable libvirtd
systemctl enable firewalld
systemctl enable acpid

# If tlp was installed above
systemctl enable tlp # You can comment this command out if you didn't install tlp, see above
```

**[U]nmount**

```bash
exit
umount -R /mnt
```

**[R]eboot/Restart**

```bash
exit
systemctl reboot
```

**[E]nd**

Now you can log into the regular user account that you created and use your new installation of Arch Linux.


