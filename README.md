# Lil guide to my personal setup (Artix linux let me down :[ )

## Install to hardware
1. download the [latest official KDE installer](https://cachyos.org/download/)
2. copy ISO to [Ventoy](https://www.ventoy.net/en/download.html) formated USB stick
3. insert USB into your computer
4. press the power button to boot PC
5. press BIOS key (usually F2,F9,F11...) to load into UEFI/BIOS
6. set simple UEFI password (****) and enable secure boot under boot settings if it's not alredy on
7. under Security clear all previously stored keys (you usually have an option to restore them if you ever need to do so)
8. under boot order set Ventoy USB stick as #1, save settings and BOOT
9. in Ventoy menu select CaachyOS, normal BOOT and choose normal, legacy or proprietary NVIDIA as fits
10. choose a boot loader when prompted, I choose the simplistic systemd-boot
11. go through the setup (deal timezone, language, user)
12. in the file system selection choose BTRFS (sacrefices some perforamce for easily managed snapshots for ASAP restorations)
13. under packages remove some unnecesarry bloat and choose a DE of your choice (I liked KDE but it was a bit messy last time I tried it so this time I'll try Gnome)
14. after it is installed remove your USB and reboot into newly installed OS

## Setting up for Wayland (it's cooler and shiny)...
By default it will force X11 so we will have to make some adjustements to get Wayland working on Nvidia.
First we seet some kernel parameters. Edit /boot/loader/entries/(*****).conf (in my case linux-cachyos.conf) and add `nvidia_drm.modeset=1 nvidia_drm.fbdev=1 nvidia.NVreg_PreserveVideoMemoryAllocations=1` + other kernel parameters you might want at the end of the last options line.
Then we need to install some invidia drivers. We can do this by running `sudo pacman -Sy nvidia`. To make them work properly we also need to enable them by running: `sudo systemctl enable nvidia-resume.service;sudo systemctl enable nvidia-hibernate.service;sudo systemctl enable nvidia-suspend;sudo systemctl enable nvidia`.
Lastly we need to tell GDM to give us the option to use Wayland in the login screen. We do this by running
`sudo nano /etc/gdm/custom.conf`and changin `#WaylandEnable=False` to `WaylandEnable=True`.

It might also be necessary to run sudo `ln -s /dev/null /etc/udev/rules.d/61-gdm.rules`.

Reboot. Now when you choose your profile on the GDM screen you should see a gear icon in the bottom right that when clicked should present an option to use Wayland instead of Xorg! Check if it worked by running echo $XDG_CURRENT_DESKTOP.


Now we will focus on making Secure Boot work properly. If you were to go into Settings>Privacy>Device Security you would see it fails the checks. To fix this we will use a convinient signing tool. Install it by running `sudo pacman -S sbctl`. Run `sudo sudo sbctl enroll-keys;sudo sbctl create-keys; sudo sbctl status`. You should get 2 check marks out of 3. If you only get one then you most likely didn't clear the keys in UEFI/BIOS. You now need to sign some files with these keys. To see which files you need to sign run `sudo sbctl list-files`. Sign all of them by running sudo sbctl sign -s path/to/file.extension`. Eg.: sudo sbctl sign -s /boot/vmlinuz-linux-cachyos`. 

If you reboot now your OS should pass secure boot, get 3 ticks on stats check and pass tests in Privacy>Device Security. Now you need edit sudo nano /etc/pacman.d/hooks/80-secureboot.hook` and `sudo nano /etc/pacman.d/hooks/95-systemd-boot.hook` in order to sign the kernel after every sudo pacman -Syyu (otherwise you fill get locked out on kernel updates). In 80-secureboot.hook add the following (Exec for every file you get by running sudo sbctllist-files):

[Trigger]
Operation = Install
Operation = Upgrade
Type = Path
Target = usr/lib/systemd/boot/efi/systemd-boot*.efi

[Action]
Description = Signing systemd-boot EFI binary for Secure Boot
When = PostTransaction
Exec = /bin/sh -c 'sbctl sign -s /path/to/file1.extension;sbctl sign -s /path/to/file2.extension;...;'
Depends = sh
Depends = sbctl
NeedsTargets

And for the 95-systemd-boot.hook:
  GNU nano /etc/pacman.d/hooks/95-systemd-boot.hook                                      
[Trigger]
Type = Package
Operation = Upgrade
Target = systemd

[Action]
Description = Gracefully upgrading systemd-boot...
When = PostTransaction
Exec = /usr/bin/systemctl restart systemd-boot-update.service

Now that we got the essentials out of the way we can get our favourtire packages:
(firmware) sudo pacman -Sy linux-firmware ast-firmware aic94xx-firmware  linux-firmware-qlogic  linux-firmware-qlogic  wd719x-firmware  upd72020x-fw
(tools) sudo pacman -Sy librewolf discord-screenaudio signal-desktop code qtcreator github git flatpak qbittorrent flatpak qbittorrent  podman gnome-exrensions yay
(remote desktop) sudo pacman -Sy remmina freerdp libvncserver
(games) sudo pacman -Sy mari0 supertuxkart xonotic
(office) sudo pacman -Sy onlyoffice-bin wps-office thunderbird
(design/art/video) sudo pacman -Sy obs-studio davinci-resolve blender inkscape krita opentoonz

Flatpaks:
flatpak install com.mattjakeman.ExtensionManager
flatpak install flathub io.podman_desktop.PodmanDesktop

Mini debloat:
sudo pacman -R neofetch cachy-browser 

Gnome extensions:
Dash to Dock (left, smaller), Gradient Top Bar, User Theme, AppIndicator(app tray), Blur my Shell

Set costum theme:
yay -S gnome-terminal-transparency
sudo tar -xf Nordic-darker.tar.xz -C /usr/share/themes
Change it in gnome Tweaks > Appearance > Legacy Applications > Fluent Dark.

Install GDM themes:
sudo pacman -Sy gdm-settings
add background...


LibreWolf:
Import Browser Data, add Canvas Blocker, add DecentralEyes, add Cookie Notices to uBlock Annoyances block list, add costum DNS, Limit cross-origin referrers, ENable WebGL, restore previous session...

Lastly I set my preffered general settings, import other settings, set up BTRFS snapshots in Btrfs Assistant, configure my apps...

(if missing) sudo pacman -Sy malcontent gnome-shell

VS code:
Even Better TOML
GlassIt-VSC
rust-analyzer
Thunder Client

Rust:
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
sudo nano /home/kapa/.profile
set PATH "/home/kapa/.cargo/bin:$PATH"
❯ source ~/.profile

Fish:
sudo nano /usr/share/cachyos-fish-config/cachyos-config.fish
fastfetch --logo-type small --logo-padding-top 10
