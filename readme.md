# Alpine on X1 Carbon Gen 6
This is how I setup my x1c6 with alpine (3.23) linux + the gnome desktop.

## Install
- download [alpine std x64](https://alpinelinux.org/downloads/) iso
- write the iso to a usb (usb drive is `/dev/sdb`): `dd if=alpine.iso of=/dev/sdb bs=1M`
- reboot and press F12 to enter the boot menu
- select the usb drive and press enter

## Initial Setup
Once alpine has booted run `setup-alpine`. Here are some of the settings I used:
```
setup-alpine
# keymap: us
# key loayout: us
# hostname: HOSTNAME
# interface: wlan0/SSID/PASSWORD/done
# root pass: 
# timezone: Canada/Eastern
# HTTP/FTP Proxy: none
# NTP: busybox
# apk mirror: dl-cdn.alpinelinux.org
# user: USER
# pass: ...
# ssh key: none
# ssh srv: none
# disk: ...
# how use: sys
# reboot
```

### Setup doas (as root)
Post reboot I setup doas for my `USER` and updated the system:
- add `permit persist :wheel as root` to `/etc/doas.conf`
- add my user to wheel `adduser USER wheel`
- reboot

### Configured repos and updated
After setting up `doas` I logged in as my USER and did the following:
- uncommented the  `community` line in `/etc/apk/repositories`
- ran `doas apk update && apk upgrade`
- reboot

### Add some software
Before installing the deskop I added some software:
```
apk add vim
apk add lsblk
apk add htop
apk add fastfetch
apk add plocate && updatedb
apk add git
apk add build-base
```

## Install the Desktop
I chose to setup the deskop with [gnome](https://wiki.alpinelinux.org/wiki/GNOME). In the future going to try sway but for now this suits me well.
- login as root
- run `setup-desktop gnome`
- run `apk add dconf-editor`
- reboot

## fix: network unavailable in settings and no wifi icon in top bar
Right off the bat almost everything worked except showing and setting wifi. I found instructions to replace the networking service with [network manager](https://wiki.alpinelinux.org/wiki/NetworkManager) which resolved it.

Open console, `su -` to change to root, and run the following:
```
apk add networkmanager
apk add networkmanager-wifi
adduser USER plugdev
# copy the config and "wpa_supplicant backend" config from the wiki to `/etc/NetworkManager/NetworkManager.conf`
rc-service networking stop
rc-service wpa_supplicant stop
rc-service networkmanager restart
rc-update add networkmanager default
rc-update del networking boot
rc-update del wpa_supplicant boot
reboot
```
## Extras

### Install flatpak and sublime text
```
apk add flatpak
apk add gnome-software-plugin-flatpak
flatpak --user remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak --user search sublime
flatpak --user install com.sublimehq.SublimeText
reboot
```

### Undervolt the cpu for longer battery life
```
git clone https://github.com/kitsunyan/intel-undervolt.git
cd intel-undervolt && ./configure && make
doas make install
# edit CPU/GPU/CPU Cache to -100 in /etc/intel-undervolt.conf
doas intel-undervolt apply
```
### Have nautilus always show the full file path
```
gsettings set org.gnome.nautilus.preferences always-use-location-entry true
```
