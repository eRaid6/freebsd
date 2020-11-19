# Installing FreeBSD 12.1-RELEASE on a Lenovo X1 Carbon 6th Gen
First and foremost, when in doubt visit the [FreeBSD Handbook](https://www.freebsd.org/doc/handbook/), it is an amazing source of knowledge.  The configuration below does not always show how to make changes to the running system, so always reboot to make the changes live.

## Install  
[FreeBSD Handbook - 2 Installing FreeBSD](https://www.freebsd.org/doc/handbook/bsdinstall.html)  
* Select ports and source to be installed
* Select Auto ZFS partitioning and select encrypt and encrypt swap

### Patch system  
[FreeBSD Handbook - 23 Updating and Upgrading FreeBSD](https://www.freebsd.org/doc/handbook/updating-upgrading.html)  
First thing to do when the system is finished installing is to patch it (assuming you setup a network interface during the install)
```
freebsd-update fetch
freebsd-update install
shutdown -r now 'patching base system'
```

## Basic config  
1. (Optional) Change quarterly to latest in `/etc/pkg/FreeBSD.conf`  
[FreeBSD Handbook - 4.4.2 Quarterly and Latest Ports Branches](https://www.freebsd.org/doc/handbook/pkgng-intro.html)  
```
# sed -i 's/quarterly"/latest"/g' /etc/pkg/FreeBSD.conf
```
2. Install nice to have packages  
```
pkg install sudo vim bash bash-completion tcpdump rsync
```
3. Configure sudo  
```
visudo
```
4. Switch shell to bash  
```
chsh -s /usr/local/bin/bash
```
If setting up `~/.bashrc` and you expect it to be run everytime you launch bash then either symlink `~/.bash_profile` to it or source `~/.bashrc` in `~/.bash_profile`, or just use `~/.bash_profile`.
[bash(1)](https://www.freebsd.org/cgi/man.cgi?query=bash&sektion=1&manpath=freebsd-release-ports)
>        When  bash is invoked as	an interactive login shell, or as a non-inter-
>       active shell with the --login option, it	first reads and	executes  com-
>       mands  from  the	file /etc/profile, if that file	exists.	 After reading
>       that file, it looks for ~/.bash_profile,	~/.bash_login, and ~/.profile,
>       in  that	order, and reads and executes commands from the	first one that
>       exists and is readable.	The --noprofile	option may be  used  when  the
>       shell is	started	to inhibit this	behavior.
>
>       When an interactive login shell exits, or a non-interactive login shell
>       executes	the exit builtin command, bash	reads  and  executes  commands
>       from the	file ~/.bash_logout, if	it exists.
>
>       When  an	 interactive  shell that is not	a login	shell is started, bash
>       reads and executes commands from	~/.bashrc, if that file	exists.	  This
>       may  be inhibited by using the --norc option.  The --rcfile file	option
>       will force bash to read and  execute  commands  from  file  instead  of
>       ~/.bashrc.
5. Set locale to utf-8 in [/etc/login.conf](configs/etc-login.conf)  
The below config will change the locale for the entire system.  You dont need to do this system wide and folks will argue it _could_ break something for the root user.  If you do not want to do it system wide you can do it for each user in their login shell.  See [FreeBSD Handbook - 22.2.1 Setting Locale for Login Shell](https://www.freebsd.org/doc/handbook/using-localization.html) for more info on that.
```
# https://www.freebsd.org/doc/handbook/using-localization.html
# https://www.b1c1l1.com/blog/2011/05/09/using-utf-8-unicode-on-freebsd/
# Append the below in the default section after umask=022:
        :charset=UTF-8:\
        :lang=en_US.UTF-8:
```
```
sudo cap_mkdb /etc/login.conf
```
Log out and back in or reboot then run `locale` to verify the changes worked
```
locale
```
6. Disable vim mouse support in [~/.vimrc](configs/vimrc)  
```
set mouse=
set ttymouse=
```

## Configure Desktop
1. Install i3 or XFCE 4 
i3 
```
sudo pkg install xorg i3 i3status i3lock dmenu dunst slim slim-freebsd-black-theme xautolock xfce4-terminal git webcamd freerdp nextcloudclient gnome-keyring libgnome-keyring eog vlc gimp cmus pwgen firefox-esr networkmgr virt-viewer x11-fonts/droid-fonts-ttf twemoji-color-font-ttf webfonts urwfonts unicode-emoji xf86-input-synaptics colordiff
```
XFCE 4 
```
sudo pkg install xorg xfce4 xfce4-power-manager xfce4-battery-plugin xfce4-pulseaudio-plugin xfce4-systemload-plugin slim slim-freebsd-black-theme git webcamd freerdp nextcloudclient gnome-keyring libgnome-keyring eog vlc gimp cmus pwgen firefox-esr networkmgr virt-viewer x11-fonts/droid-fonts-ttf twemoji-color-font-ttf webfonts urwfonts unicode-emoji xf86-input-synaptics colordiff
```
2. Install intel video driver, need to install via port because binary was compiled for 12.0 only not 12.1  
[FreeBSD Forums - Upgrading to FreeBSD 12.1-RELEASE - resolving an issue with drm-fbsd12.0-kmod](https://forums.freebsd.org/threads/upgrading-to-freebsd-12-1-release-resolving-an-issue-with-drm-fbsd12-0-kmod.72895/)  
[FreeBSD 12.1R Errata - Open Issues](https://www.freebsd.org/releases/12.1R/errata.html#open-issues)  
[FreeBSD Handbook - 4.5 Using the Ports Collection](https://www.freebsd.org/doc/handbook/ports-using.html)  
```
portsnap fetch
portsnap extract
cd /usr/ports/graphics/drm-kmod
make install clean
```
Load intel video driver on boot via [/etc/rc.conf](configs/etc-rc.conf)
```
#
# Load intel driver on boot
kld_list="/boot/modules/i915kms.ko"
```
3. Configure slim  
Use the black theme
```
sed 's/current_theme.*/current_theme slim-freebsd-black-theme/g' /usr/local/etc/slim.conf
```
Configure slim to start on boot [/etc/rc.conf](configs/etc-rc.conf)
```
slim_enable="YES"
```
Create `.xinitrc` to lauch your desktop when slim logs you in
```
# xfce4
. /usr/local/etc/xdg/xfce4/xinitrc
# i3
# . /usr/local/bin/i3
```
4. Configure i3  
i3 config [~/.config/i3/config](configs/i3-config)  
```
mkdir ~/.config/i3
```
i3status config [~/.config/i3status/config](configs/i3status-config)
```
mkdir ~/.config/i3status
```
Lock screen on suspend [/etc/rc.suspend](configs/etc-rc.suspend)  
*Note* For sleep to work you need at least BIOS 1.30 and you need to enable the Linux Sleep State in the Bios  
> <1.30>
> UEFI: 1.30 / ECP: 1.08
> - (New) Support Optimized Sleep State for Linux in ThinkPad Setup - Config - Power.
>        (Note) "Linux" option is optimized for Linux OS, Windows user must select 
>	       "Windows 10" option.
```
# Lock screen (via i3lock) when suspending
/usr/bin/su YOURUSERNAME -c 'DISPLAY=:0.0 /usr/local/bin/i3lock --color 000000'
```

5. Configure touchpad  
[Synaptics Touchpad](https://wiki.freebsd.org/SynapticsTouchpad)  
[/boot/loader.conf](configs/boot-loader.conf)  
```
#
# Touchpad support
hw.psm.synaptics_support="1"
```
[/etc/sysctl.conf](configs/etc-sysctl.conf)
```
#
# Touchpad support
hw.psm.synaptics.vscroll_hor_area=1300
hw.psm.synaptics.min_pressure=43
hw.psm.synaptics.max_pressure=100
hw.psm.tap_timeout=0
```
[/etc/rc.conf](configs/etc-rc.conf)
```
#
# Touchpad support
moused_enable="YES"
```
6. Webcam config  
Configure webcam driver to start on boot [/etc/rc.conf](configs/etc-rc.conf)  
```
#
# Webcam support
webcamd_load="YES"
```
Configure cuse module to load on boot [/boot/loader.conf](configs/boot-loader.conf)  
```
cuse_load="YES"
```
Add your user to the webcamd group
```
pw groupmod webcamd -m <username>
```
You can test the webcam with `pwcview`
```
sudo pkg install pwcview
pwcview
```

7. Configure WiFi [/etc/wpa_supplicant.conf](configs/etc-wpa_supplicant.conf)  
[FreeBSD Handbook - 31.3 Wireless Networking](https://www.freebsd.org/doc/handbook/network-wireless.html)  
Configure your WiFi network in [/etc/wpa_supplicant.conf](configs/etc-wpa_supplicant.conf) and then restart networking `sudo service netif restart`.  Check your connection by running `ifconfig`.  
(Optional) Configure lagg [/etc/rc.conf](configs/etc-rc.conf)  
You can also configure [lagg](https://www.freebsd.org/cgi/man.cgi?query=lagg&sektion=4&manpath=freebsd-release-ports) to enable [failover Mode Between Ethernet and Wireless Interfaces](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-aggregation.html). 
```
ifconfig_em0="ether 20:57:8a:99:ff:ff"
wlans_iwm0="wlan0"
ifconfig_wlan0="WPA powersave"
# https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-aggregation.html 31.3
cloned_interfaces="lagg0"
ifconfig_lagg0="up laggproto failover laggport em0 laggport wlan0 DHCP"
```

8. Configure Pulse audio  
If you installed Firefox via `pkg` like above it is compiled against pulse audio.  On my X1 Carbon 6th gen the audio is choppy by default.  To fix the audio sounding choppy change `default-fragments` and `default-fragment-size-msec` in [/usr/local/etc/pulse/daemon.conf](configs/pulse-daemon.conf)
```
#
# Fix choppy audio
# https://www.dan.me.uk/blog/2012/07/04/fix-choppy-audio-under-pulseaudio-in-gnome-on-freebsd/
default-fragments = 8
default-fragment-size-msec = 5
```

## Configure NFS via auto mounter
[FreeBSD Handbook - 29.3 Network File System](https://www.freebsd.org/doc/handbook/network-nfs.html)  
1. Enable autofs to start on boot in [/etc/rc.conf](configs/etc-rc.conf)
```
#
# NFS config
autofs_enable="YES"
```
2. Update [/etc/auto_master](configs/etc-auto_master) to include [/etc/auto_nfs](configs/etc-auto_nfs)
```
#
# Mount tacos NFS share
/-    /etc/auto_nfs
```
3. Create [/etc/auto_nfs](configs/etc-auto_nfs)
```
#
# Mount tacos NFS share
/nfs/data -fstype=nfs,rw  tacos:/nfs/data
```

## Configure Security
### sshd config
1. Disable root login
```
sudo sed -iE 's/.*PermitRootLogin .*/PermitRootLogin no/g' /etc/ssh/sshd_config
```
2. Do not allow password logins
```
sudo sed -iE 's/.*PermitEmptyPasswords .*/PermitEmptyPasswords no/g' /etc/ssh/sshd_config
sudo sed -iE 's/.*PasswordAuthentication .*/PasswordAuthentication no/g' /etc/ssh/sshd_config
```
3. Only allow specific users to login
```
sudo echo 'AllowUsers nick' >> /etc/ssh/sshd_config
```

### IPFW
[FreeBSD Handbook - 30.4 IPFW](https://www.freebsd.org/doc/handbook/firewalls-ipfw.html)  
1. Enable IPFW firewall and configure basic config in [/etc/rc.conf](configs/etc-rc.conf)
```
#
# IPFW configuration
firewall_enable="YES"
firewall_quiet="YES"
firewall_type="workstation"
firewall_myservices="22/tcp"
firewall_allowservices="192.168.250.0/24"
firewall_logdeny="YES"
```
2. Reboot
```
sudo shutdown -r now 'enabling ipfw'
```

### Veracrypt
1. Install VeraCrypt package
```
sudo pkg install veracrypt fusefs-libs fuse-zip
```
2. Start fuse on boot in [/boot/loader.conf](configs/boot-loader.conf)
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
2. Configure WireGuard in [/usr/local/etc/wireguard/wg0.conf](configs/wireguard-wg0.conf)
```
#
# Example WireGuard Config
[Interface]
PrivateKey = sdgsdgshfdshSDHGSDHShfdshfsdhfds66sd6g6dsg6ds6gs6==
ListenPort = 50123
DNS = 1.1.1.2, 1.0.0.2
Address = 192.168.52.1/24

[Peer]
PublicKey = asdfhdfshdfs9h7SDHGSDHfdshsdfASDgaghfdshsfdhfsdasdh==
AllowedIPs = 0.0.0.0/0
Endpoint = 192.168.1.1:443' > /usr/local/etc/wireguard/wg0.conf
```
3. Enable WireGuard on boot in [/etc/rc.conf](configs/etc-rc.conf)
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

5. Yubikey tools
```
sudo pkg install yubico-piv-tool yubikey-agent yubikey-manager-qt yubikey-personalization-gui yubioath-desktop
```

### Other hardening
[FreeBSD Handbook - 13 Security](https://www.freebsd.org/doc/handbook/security-intro.html)  
Add the following to [/etc/sysctl.conf](configs/etc-sysctl.conf)
```
#
# blackhole packets destined for ports we are not listening
# https://www.freebsd.org/doc/handbook/security-intro.html
net.inet.tcp.blackhole=2
net.inet.udp.blackhole=1
#
# disable icmp redirect packets
# https://www.freebsd.org/doc/handbook/security-intro.html
net.inet.ip.redirect=0
net.inet.icmp.drop_redirect=1
#
# disable access to non-routable addresses
# https://www.freebsd.org/doc/handbook/security-intro.html
net.inet.ip.sourceroute=0
net.inet.ip.accept_sourceroute=0
#
# reject external broadcast requests
# https://www.freebsd.org/doc/handbook/security-intro.html
net.inet.icmp.bmcastecho=0
```

# Additional info
[FreeBSD Handbook](https://www.freebsd.org/doc/handbook/)  
[FreeBSD Forums](https://forums.freebsd.org/)  
[FreeBSD on a Laptop - c0ffee.net](https://www.c0ffee.net/blog/freebsd-on-a-laptop)  
[FreeBSD on X1 Carbon (Gen 6) - RUN YOUR OWN](https://things.bleu255.com/runyourown/FreeBSD_on_X1_Carbon_(Gen_6))  
