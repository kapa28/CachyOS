# artix

Plasma+Dinit latest ISO
Vetoy install
sudo pacman -Syyu [y to all]

------------------------------------------------------ Audio:
 nano /etc/modprobe.d/alsa.conf
 options bt87x index=-2
 options cx88_alsa index=-2
 options saa7134-alsa index=-2
 options snd-atiixp-modem index=-2
 options snd-intel8x0m index=-2
 options snd-via82xx-modem index=-2
 options snd-usb-audio index=-2
 options snd-usb-caiaq index=-2
 options snd-usb-ua101 index=-2
 options snd-usb-us122l index=-2
 options snd-usb-usx2y index=-2
 options snd-pcsp index=-2
 options snd-usb-audio index=-2
 

sudo pacman -R pulseaudio-zeroconf
sudo pacman -S pipewire pipewire-alsa pipewire-pulse pipewire-jack

sudo nano ~/.config/plasma-workspace/env/autostart.sh
> #!/bin/sh
> /usr/bin/pipewire & /usr/bin/pipewire-pulse & /usr/bin/wireplumber

sudo pacman -S sof-firmware

------------------------------------------------------ Keymap:
 KEYMAP=si
 


----------------------------------------------------- Wayland:
sudo nano /etc/environment
QT_STYLE_OVERRIDE= theme name(breeze)
QT_QPA_PLATFORMTHEME=gtk4/qt5
sudo pacman -S plasma-wayland-session (+kde-applications)
sudo nano /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT+="nvidia nvidia-drm.modeset=1 nvidia-drm.fbdev=1 nvidia-uvm nvidia-drm"
grub-mkconfig -o /boot/grub/grub.cfg
mkinitcpio -P

/etc/pacman.d/hooks/nvidia.hook
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia
Target=linux
# Change the linux part above if a different kernel is used

[Action]
Description=Update NVIDIA module in initcpio
Depends=mkinitcpio
When=PostTransaction
NeedsTargets
Exec=/bin/sh -c 'while read -r trg; do case $trg in linux*) exit 0; esac; done; /usr/bin/mkinitcpio -P'


------------------------------------------------------ Grub:
sudo nano /etc/default/grub

GRUB_DEFAULT=0
GRUB_TIMEOUT=0
GRUB_HIDDEN_TIMEOUT=1
GRUB_RECORDFAIL_TIMEOUT=$GRUB_HIDDEN_TIMEOUT
GRUB_CMDLINE_LINUX_DEFAULT+="i915.enable_psr=1 i915.enable_fbc=1 quiet loglevel=0 nowatchdog mitigations=off ipv6.disable=1"
grub-mkconfig -o /boot/grub/grub.cfg

sudo nano /boot/grub/grub.cfg
//remove linux kernel loading echos

------------------------------------------------------ Theming:
