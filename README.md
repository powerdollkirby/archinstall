![arch](https://github.com/powerdollkirby/archinstall/blob/main/archlinux-anime-logo.png)
# archinstall
Quick arch linux install guide using archinstall script



this is how i do it and it works for me



if you're using a laptop or wifi follow the [iwctl](https://wiki.archlinux.org/title/Iwd) guide to connect to your wifi


now, the install

set the time and date:

    root@archiso ~ # timedatectl set-ntp true


to speed things up we'll do

    root@archiso ~ # pacman -Sy archlinux-keyring --noconfirm

listing all block devices:

    root@archiso ~ # lsblk


    nvme0n1            500G  0 disk
    nvme0n2            120G  0 disk
    ⌙nvme0n2p1         120G  0 part


in this case i'll use nvme0n1

    root@archiso ~ # cfdisk /dev/nvme0n1


eg:

    Device                   Start          End      Sectors      Size Type
    >>Free Space              1234        43210     10000000      500G


here you create a 1G EFI partition and a Linux filesystem partition (will be your root partition)

like this:

    nvme0n1                500G 0 disk
    ⊢ nvme0n1p1 0            1G 0 part (EFI system)
    ⊢ nvme0n1p2 0          499G 0 part (Linux file system)

partitioning without a swap eg:

formatting:
nvme0n1p1 1G as fat32

and

nvme0n1p2 499G as ext4

    root@archiso ~ # mkfs.fat -F32 /dev/nvme0n1p1
    root@archiso ~ # mkfs.ext4 /dev/nvme0n1p2


then mount nvme0n1p2 to /mnt

    root@archiso ~ # mount /dev/nvme0n1p2 /mnt

then

    root@archiso ~ # mkdir /mnt/boot

then

    root@archiso ~ # mount /dev/nvme0n1p1 /mnt/boot

it should be like this:

    nvme0n1                500G 0 disk
    ⊢ nvme0n1p1 0            1G 0 part /mnt/boot
    ⊢ nvme0n1p2 0          499G 0 part /mnt

then use archinstall script

    root@archiso ~ # archinstall

then 

    Mirrors:                  here you have to select the mirror that is closer to you
    Locales:                  select the defined by your region and keyboard layout
    Disk configuration:       select pre-mounted configuration
                              here enter the root directory of mounted devices: /mnt

    Disk encryption:          up to you, i generally don't use it
    Bootloader:               select grub
    Unified kernel images:    false
    Swap:                     true
    Host name:                default or change it, up to you
    Root password:            set a password
    User account:             add a user and use the same password as root and make this user a superuser.
    Profile:                  here you'll select the type, desktop environment, gpu drivers and greeter
    Gpu drivers:              select open source for amd or intel, if you're using nvidia select proprietary nvidia
    greeter:                  sddm
    Audio:                    Pipewire
    Kernels:                  use the default
    Additional packages:      firefox flatpak nano gcc clang make cmake btop htop nvtop
    Network configuration:    select use networkmanager
    Timezone:                 select your timezone
    Additional repositories:  select multilib

then

    install

after the install:

    would you like to chroot into the newly created installation and perform past-installation configuration?
yes


then

    root@archiso ~ # pacman -Sy grub efibootmgr dosfstools mtools

then

    root@archiso ~ # grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB

then

    root@archiso ~ # grub-mkconfig -o /boot/grub/grub.cfg

now exit

    root@archiso ~ # exit

then

    root@archiso ~ # reboot

done!

why grub? because i'm used to it :D


[![Archinstall Guide](https://starchart.cc/powerdollkirby/archinstall.svg)](https://starchart.cc/powerdollkirby/archinstall)


arch linux anime logo by: [@sawaratsuki1004](https://twitter.com/sawaratsuki1004) post: [x.com](https://x.com/sawaratsuki1004/status/1782373444233118036)
