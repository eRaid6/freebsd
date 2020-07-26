# Installing FreeBSD 12.1 as a KVM guest
First and foremost, when in doubt visit the [FreeBSD Handbook](https://www.freebsd.org/doc/handbook/), it is an amazing source of knowledge.

## VM config
* Add evtouch tablet input device, this uses absolute mouse positioning and makes it so you do not need to hit ctrl + alt to release the mouse from the remote-viewer/virt-manager window.

## Install
* Select ports and source to be installed
* Select AutoZFS
* Select Encrypt Disk and Encrypt Swap

## Basic config
1. Install nice to have packages
```
# pkg install sudo vim bash bash-completion tcpdump
```
2. Configure sudo
```
visudo
```
3. Switch shell to bash
```
chsh -s /usr/local/bin/bash
```

## Configure Desktop
1. Install XFCE4 packages
```
sudo pkg install xorg xfce xfce4-power-manager xfce4-pulseaudio-plugin xfce4-screensaver xfce4-screenshooter-plugin xfce4-whiskermenu-plugin xfce4-xkb-plugin lightdm-gtk-greeter lightdm vlc gimp firefox utouch-kmod xf86-video-qxl xf86-input-evdev
```
2. Enable dbus and lightdm for XFCE4 in /etc/rc.conf
```
#
# XFCE4 config
dbus_enable="YES"
lightdm_enable="YES"
```
3. Enable utouch driver for better mouse support
```
echo '#
# evtouch driver for mouse
utouch_load="YES"' >> /boot/loader.conf
```
4. Configure qxl video output
/usr/local/etc/X11/xorg.conf.d/spiceqxl.xorg.conf
```
#
# Configure spiceqxl per furybsd example
# https://github.com/furybsd/furybsd-livecd/blob/master/xorg.conf.d/driver-qxl.conf
Section "Device"
    Identifier "Card0"
    Driver "qxl"
EndSection
```
5. Configure evtouch input
/usr/local/etc/X11/xorg.conf.d/99-qemu-input.conf
```
#
# Overide libinput
# https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=244079
Section "InputClass"
        Identifier "evdev touchscreen catchall"
        MatchIsTouchscreen "on"
        MatchDevicePath "/dev/input/event*"
        Driver "evdev"
EndSection
```

## Configure Security
### IPFW
1. Enable IPFW firewall and configure basic config
/etc/rc.conf
```
#
# IPFW configuration
firewall_enable="YES"
firewall_quiet="YES"
firewall_type="workstation"
firewall_myservices="22"
firewall_allowservices="192.168.1.0/24"
firewall_logdeny="YES"
```
See [FreeBSD HandBook - IPFW](https://www.freebsd.org/doc/handbook/firewalls-ipfw.html) for more configuration options.
2. Reboot
```
sudo shutdown -r now
```

### Veracrypt
1. Install VeraCrypt package
```
sudo pkg install veracrypt fusefs-libs fuse-zip
```
2. Start fuse on boot
/boot/loader.conf
```
#
# Start Fuse for VeraCrypt
fuse_load="YES"
```

### Wireguard
1. Install WireGuard
```
sudo pkg install wireguard
```
2. Configure WireGuard
/usr/local/etc/wireguard/wg0.conf
```
#
# Example WireGuard Config
[Interface]
PrivateKey = sdgsdgshfdshSDHGSDHShfdshfsdhfds66sd6g6dsg6ds6gs6==
ListenPort = 50123
DNS = 1.1.1.1, 1.0.0.1
Address = 192.168.52.1/24

[Peer]
PublicKey = asdfhdfshdfs9h7SDHGSDHfdshsdfASDgaghfdshsfdhfsdasdh==
AllowedIPs = 0.0.0.0/0
Endpoint = 192.168.1.1:443' > /usr/local/etc/wireguard/wg0.conf
```
3. Enable WireGuard on boot
/etc/rc.conf
```
#
# WireGuard config
wireguard_interfaces="wg0"
wireguard_enable="YES"
```
4. Start WireGuard and/or enable WireGuard to start on boot
```
# Start WireGuard on boot
sudo service wireguard enable
# Start WireGuard once
sudo service wireguard onestart
```

### Tor
1. Install tor package
```
sudo pkg install security/tor
```
2. Enable Tor to start on boot
/etc/rc.conf
```
#
# Tor config
tor_enable="YES"
```
3. Prevent traffic analysis that exploits sequential IP IDs
/etc/sysctl.conf
```
# 
# Tor
# prevent traffic analysis that exploits sequential IP IDs
net.inet.ip.random_id=1
```
4. Start Tor
```
sudo service tor onestart
```
4. Configure a Firefox profile for Tor (using default SOCKS port 9050) and make it secure and use tor only after wireguard is started.  Install ublock-origin, no-script and https everywhere addons.

# Additional info
[FreeBSD Handbook](https://www.freebsd.org/doc/handbook/)  
[FuryBSD LiveCD](https://github.com/furybsd/furybsd-livecd)  
[FreeBSD Forums](https://forums.freebsd.org/)  
