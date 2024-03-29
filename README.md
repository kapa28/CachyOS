# Lil guide to my personal setup

## Install to hardware
1. download the [latest official KDE installer](https://cachyos.org/download/)
2. copy ISO to [Ventoy](https://www.ventoy.net/en/download.html) formated namebrand USB stick
3. insert USB into your computer
4. press the power button to boot PC
5. press BIOS key (usually F2,F9,F11...) to load into UEFI/BIOS
6. set an UEFI password (*+*+*+*+) and enable secure boot under boot settings if it's not alredy on
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
1. we need to seet some kernel parameters, edit /boot/loader/entries/(*****).conf (in my case linux-cachyos.conf) with a terminal text editor fo your choice (I'll use nano for simplicity)
2. add `nvidia_drm.modeset=1 nvidia_drm.fbdev=1 nvidia.NVreg_PreserveVideoMemoryAllocations=1` + other kernel parameters (like splash quiet nowatchdog...) you might want or need at the end of the last options line
3. install some invidia drivers, I'll do it by running `sudo pacman -Sy nvidia`
4. to make them work properly we also need to enable them by running: `sudo systemctl enable nvidia-resume.service;sudo systemctl enable nvidia-hibernate.service;sudo systemctl enable nvidia-suspend;sudo systemctl enable nvidia`
5. lastly we need to tell GDM that we are ready for Wayland by running: `sudo nano /etc/gdm/custom.conf`and changing `#WaylandEnable=False` to `WaylandEnable=True`
6.*it might also be necessary to run sudo `ln -s /dev/null /etc/udev/rules.d/61-gdm.rules`.
7. reboot
8. choose your profile on the GDM screen, you should see a gear icon in the bottom right corner of the password input screen, when clicked it should present an option to use Wayland instead of Xorg!
9. check if it worked by running echo $XDG_CURRENT_DESKTOP (returns x11 or wayland)

## Initialize Secure Boot
1. if you were to go into ***Settings > Privacy > Device Security*** will most likely see failed security checks, to fix this we will use a convinient signing tool
2. install it by running `sudo pacman -S sbctl`
3. run `sudo sbctl create-keys;sudo sudo sbctl enroll-keys;sudo sbctl status` to create new keys for your OS, you should get 2 check marks out of 3 on status command output, if you only get one then you most likely didn't clear the keys in UEFI/BIOS
4. you now need to sign some files with these keys, to see which files you need to sign run `sudo sbctl list-files`
5. sign all of them by running sudo `sbctl sign -s path/to/file.extension`, eg.: `sudo sbctl sign -s /boot/vmlinuz-linux-cachyos` 
6. reboot, now your OS should pass secure boot, get 3 ticks on stats check and pass tests in Privacy>Device Security
7. now we need do set up automatic signing after every big `sudo pacman -Syyu` (otherwise you fill get locked out on kernel updates), run `sudo nano /etc/pacman.d/hooks/95-systemd-boot.hook` to add:

```
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
```

8. and `sudo nano /etc/pacman.d/hooks/95-systemd-boot.hook` to add:

```
[Trigger]
Type = Package
Operation = Upgrade
Target = systemd

[Action]
Description = Gracefully upgrading systemd-boot...
When = PostTransaction
Exec = /usr/bin/systemctl restart systemd-boot-update.service
```
## Packages
Now that we got the essentials out of the way we can get our favourtire packages:
+ firmware: `sudo pacman -Sy`+ `linux-firmware`
  +  extra `ast-firmware aic94xx-firmware linux-firmware-qlogic linux-firmware-qlogic wd719x-firmware upd72020x-fw`
+ tools: `sudo pacman -Sy librewolf code git flatpak qbittorrent podman gnome-extensions yay powertop`
  + flatpaks: `flatpak install com.mattjakeman.ExtensionManager io.podman_desktop.PodmanDesktop`   
+ remote desktop: `sudo pacman -Sy remmina freerdp libvncserver`
+ social: `sudo pacman -Sy discord-screenaudio signal-desktop`
+ creative: `sudo pacman -Sy obs-studio davinci-resolve blender inkscape krita opentoonz`
+ office: `sudo pacman -Sy wps-office thunderbird`
+ games: `sudo pacman -Sy mari0 supertuxkart xonotic`
+ rust: `rust:curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`, `sudo nano /home/kapa/.profile`, `set PATH "/home/kapa/.cargo/bin:$PATH"`, `source ~/.profile`

### Mini debloat
- `sudo pacman -R neofetch cachy-browser`

## Add if missing
+ `sudo pacman -Sy malcontent gnome-shell`
+ `sudo nano /etc/vconsole.conf, `FONT=lat2a-16`

## Customization
I like to change some stuff:
+ general settings:
  + ***Tweaks > Mouse & Touchpad > Touchpad > Didable while typing > Off, Tweaks > Mouse & Touchpad > Touchpad > Mouse click emulation > Area***
  + ***Settings > Mouse & Touchpad > Touchpad > Clicking >Tap to click > On, Settings > Mouse & Touchpad > Touchpad > Scrolling > Natural***
  + ***Tweaks > Apperance > Background > Background image > Set cool vaporwave image***
+ gnome extensions(in extension manager):
  + Dash to Dock (show dock on screen),
  + Gradient Top Bar (fade out top bar),
  + User Theme (support for gnome themes),
  + AppIndicator(app tray),
  + Blur my Shell (blur effects),
  + Tactile (Window Manager*, add gapps)
  + Battery Health Charging (https://github.com/frederik-h/acer-wmi-battery + polkit install = limit to 80%)
+ set custom theme:
  + get [Fluent Dark](https://github.com/vinceliuice/Fluent-gtk-theme) or other GTK theme
  + `sudo tar -xf Nordic-darker.tar.xz -C /usr/share/themes`
  + cange it in ***Tweaks > Appearance > Legacy Applications > Fluent Dark***
  + change QT theme in Kvantum Manager to match GTK theme
+ theme GDM:
  + `sudo pacman -Sy gdm-settings`
  + set a blurred version of normal background image
+ librewolf:
  + import browser data (about:config, browser.migrate.interactions.csvpasswords),
  + add Canvas Blocker,
  + add DecentralEyes,
  + add Cookie Notices to uBlock Annoyances block list,
  + add costum DNS,
  + Limit cross-origin referrers,
  + ENable WebGL,
  + enable restore previous session,
  + add MATRIX_PURPLE theme
+ powertop:
  + enable tweaks for better battery life  
+ BTRFS assistant
  + enable and schedule 1x day, 2x week, 4x month, 1x year
+ code:
  + Even Better TOML
  + GlassIt-VSC
  + rust-analyzer
  + Thunder Client
  + Powe Mode
  + Syntax theme
+ fish:
  + `yay -S gnome-terminal-transparency`
  + `sudo nano /usr/share/cachyos-fish-config/cachyos-config.fish`, add `--logo-type small --logo-padding-top 10` to `fastfetch`in greet

 Finish by updating with `sudo pacman -Syyu` and transfering saved data onto your computer.
