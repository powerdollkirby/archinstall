![arch](https://github.com/powerdollkirby/archinstall/blob/main/archlinux-anime-logo.png)
# archinstall


Quick arch linux install guide using archinstall script and manual partitioning. (without a swap partition)

__________________________________________




>if you're using a laptop or a wifi card 
>follow the [iwctl](https://wiki.archlinux.org/title/Iwd#iwctl) guide 🕮
<details>
<summary>iwctl</summary>
    

To get an interactive prompt do:

    $ iwctl

The interactive prompt is then displayed with a prefix of [iwd]#.

Tip:
In the iwctl prompt you can auto-complete commands and device names by hitting Tab.
To exit the interactive prompt, send EOF by pressing Ctrl+d.
You can use all commands as command line arguments without entering an interactive prompt. For example: iwctl device wlan0 show.

To list all available commands:

    [iwd]# help

Connect to a network

First, if you do not know your wireless device name, list all Wi-Fi devices:

    [iwd]# device list

If the device or its corresponding adapter is turned off, turn it on:

    [iwd]# device name set-property Powered on

    [iwd]# adapter adapter set-property Powered on

Then, to initiate a scan for networks (note that this command will not output anything):

    [iwd]# station name scan

You can then list all available networks:

    [iwd]# station name get-networks

Finally, to connect to a network:

    [iwd]# station name connect SSID

Note: For automatic IP and DNS configuration via DHCP, you have to manually enable the built-in DHCP client or configure a standalone DHCP client.
Tip: The user interface supports autocomplete, by typing station and Tab Tab, the available devices are displayed, type the first letters of the device and Tab to complete. The same way, type connect and Tab Tab in order to have the list of available networks displayed. Then, type the first letters of the chosen network followed by Tab in order to complete the command.

If a passphrase is required (and it is not already stored in one of the profiles that iwd automatically checks), you will be prompted to enter it. Alternatively, you can supply it as a command line argument:

    $ iwctl --passphrase passphrase station name connect SSID

Note:

    iwd automatically stores network passphrases in the /var/lib/iwd directory and uses them to auto-connect in the future. See #Network configuration.
    To connect to a network with spaces in the SSID, the network name should be double quoted when connecting.
    iwd only supports PSK pass-phrases from 8 to 63 ASCII-encoded characters. The following error message will be given if the requirements are not met: PMK generation failed. Ensure Crypto Engine is properly configured.

Connect to a network using WPS/WSC

If your network is configured such that you can connect to it by pressing a button (Wikipedia:Wi-Fi Protected Setup), check first that your network device is also capable of using this setup procedure.

    [iwd]# wsc list

Then, provided that your device appeared in the above list,

    [iwd]# wsc device push-button

and push the button on your router. The procedure works also if the button was pushed beforehand, less than 2 minutes earlier.

If your network requires to validate a PIN number to connect that way, check the help command output to see how to provide the right options to the wsc command.
Disconnect from a network

To disconnect from a network:

    [iwd]# station device disconnect

Show device and connection information

To display the details of a WiFi device, like MAC address:

    [iwd]# device device show

To display the connection state, including the connected network of a Wi-Fi device:

    [iwd]# station device show

Manage known networks

To list networks you have connected to previously:

    [iwd]# known-networks list

To forget a known network:

    [iwd]# known-networks SSID forget
</details>


__________________________________________
⋆｡ﾟ☁︎｡⋆｡ ﾟ☾ ﾟ｡⋆

1.

set the time and date:

    root@archiso ~ # timedatectl set-ntp true


2.


to speed things up we'll do

    root@archiso ~ # pacman -Sy archlinux-keyring --noconfirm

__________________________________________

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
nvme0n1p1 1G (boot) as fat32

    root@archiso ~ # mkfs.fat -F32 /dev/nvme0n1p1


nvme0n1p2 499G (root) as ext4

    root@archiso ~ # mkfs.ext4 /dev/nvme0n1p2


then mount nvme0n1p2 to /mnt

    root@archiso ~ # mount /dev/nvme0n1p2 /mnt

then make a boot directory on /mnt

    root@archiso ~ # mkdir /mnt/boot

then mount /dev/nvme0n1p1 to /mnt/boot

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
    Bootloader:               grub
    Unified kernel images:    false
    Swap:                     true
    Host name:                Default or change it, up to you
    Root password:            Set a password
    User account:             add a user and use the same password as root and make this user a superuser.
    Profile:                  here you'll select the type, desktop environment, gpu drivers and greeter
    Gpu drivers:              Open source for amd and intel, if you're using nvidia select proprietary nvidia
    greeter:                  sddm
    
    Audio:                    Pipewire
    Kernels:                  Default
    Additional packages:      firefox flatpak nano gcc clang make cmake btop htop nvtop
    Network configuration:    use networkmanager
    Timezone:                 select your timezone
    Additional repositories:  multilib

then

    install

__________________________________________

after the install:

    would you like to chroot into the newly created installation and perform past-installation configuration?
yes


then install grub efibootmgr dosfstools mtools:

    root@archiso ~ # pacman -Sy grub efibootmgr dosfstools mtools

then:

    root@archiso ~ # grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB

then make a grub config file:

    root@archiso ~ # grub-mkconfig -o /boot/grub/grub.cfg

now exit:

    root@archiso ~ # exit

then:

    root@archiso ~ # reboot

done!✿

· · ───────


## Star History

<a href="https://star-history.com/#powerdollkirby/archinstall&Date">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=powerdollkirby/archinstall&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=powerdollkirby/archinstall&type=Date" />
   <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=powerdollkirby/archinstall&type=Date" />
 </picture>
</a>


arch linux anime logo by: [@sawaratsuki1004](https://twitter.com/sawaratsuki1004) post: [x.com](https://x.com/sawaratsuki1004/status/1782373444233118036)
