# What to download?

Each board is fully supported with three basic system options: Debian Wheezy, Jessie and Ubuntu Trusty desktop build per board.

# Which kernel to use?

All stable kernels are production ready, but you should use them for different purpuses:

 * for headless server or light desktop operations use vanilla kernel (linux-image-next-[kernelfamily](http://forum.armbian.com/index.php/topic/211-kernel-update-procedure-has-been-changed/))  
 * for using video acceleration, audio, IR, NAND, ... you should stick to legacy (linux-image-[kernelfamily](http://forum.armbian.com/index.php/topic/211-kernel-update-procedure-has-been-changed/))

# How to prepare SD card?

Unzipped .raw images can be written with supplied imagewriter.exe on Windows XP/2003/Win7, with [Win32Diskimager](http://sourceforge.net/projects/win32diskimager/) on Windows 8.x or with DD command in Linux/Mac:

	dd bs=1M if=filename.raw of=/dev/sdx
	# /dev/sdx is your sd card device

Image writting takes around 3 minutes on a slow, class 6 SD card.

# How to boot?

Insert SD card into a slot and power the board. First boot takes around 3 minutes than it reboots and you will need to wait another one minute to login. This delay is because system updates package list and creates 128Mb emergency SWAP on the SD card.

Normal boot (with DHCP) takes up to 35 seconds with a class 6 SD CARD and cheapest board.

# How to login? 

Login as **root** on console or via SSH and use password **1234**. You will be prompted to change this password at first login. This is the only pre-installed user.

Desktop images starts into desktop without asking for password. To change this add some display manager:

	apt-get install lightdm

... or edit the contents of file:

	/etc/default/nodm

and change the autologin user.

# How to update kernel?

For Armbian 4.2 and newer.

	apt-get update
	apt-get upgrade
	reboot
	
For all other cases. Note that this procedure upgrades only kernel with hardware definitions (bin, dtb, firmware and headers. Operating system and modifications remain as is.

	wget -q -O - http://upgrade.armbian.com | bash

You will be prompted to select and confirm some actions. It's possible to upgrade **from any other distribution**.

# How to add users?

To create a normal user do this:

    adduser MyNewUsername

Put user to sudo group:

    usermod -aG sudo MyNewUsername

# How to customize keyboard, time zone?

keyboard: 

	dpkg-reconfigure keyboard-configuration
	
system language: 

	dpkg-reconfigure locales

time zone: 

	dpkg-reconfigure tzdata
	
screen resolution: 

	nano /boot/boot.cmd 

	# example:
	# change example from 
	# disp.screen0_output_mode=1920x1080p60 
	# to 
	# disp.screen0_output_mode=1280x720p60

	mkimage -C none -A arm -T script -d /boot/boot.cmd /boot/boot.scr	

screen resolution interactive - only Allwinner boards with A10 and A20 with legacy kernel:
	
	# Example to set console framebuffer resolution to 1280 x 720
	a10disp changehdmimodeforce 4

Other modes:	

	0 480i
	1 576i
	2 480p
	3 576p
	4 720p 50Hz
	5 720p 60Hz
	6 1080i 50 Hz
	7 1080i 60 Hz
	8 1080p 24 Hz
	9 1080p 50 Hz
	10 1080p 60 Hz
	
# How to alter CPU frequency?

Some boards allow to adjust CPU speed.

	nano /etc/init.d/cpufrequtils

Alter **min_speed** or **max_speed** variable.

	service cpufrequtils restart

# How to upgrade into simple desktop environment?

	apt-get -y install xorg lightdm xfce4 tango-icon-theme gnome-icon-theme
	reboot


Check [this site](http://namhuy.net/1085/install-gui-on-debian-7-wheezy.html) for others.

# How to upgrade Debian from Wheezy to Jessie?

	dpkg -r ramlog	
	cp /etc/apt/sources.list{,.bak}
    sed -i -e 's/ \(old-stable\|wheezy\)/ jessie/ig' /etc/apt/sources.list
    apt-get update
    apt-get --download-only dist-upgrade
    apt-get dist-upgrade


# How to upgrade from Ubuntu Trusty to next LTS?

... when available.
	
	apt-get install update-manager-core
    do-release-upgrade -d
  	# further to vivid
	apt-get dist-upgrade

# How to toggle boot output?
Edit and change [boot parameters](http://redsymbol.net/linux-kernel-boot-parameters/) in /boot/boot.cmd:

    - console=ttyS0,115200
    + console=tty1

and convert it to boot.scr with this command:

	mkimage -C none -A arm -T script -d /boot/boot.cmd /boot/boot.scr

Reboot.

Serial console on imx6 boards are ttymxc0 (Hummingboard, Cubox-i) or ttymxc1 (Udoo).

# How to install to NAND, SATA & USB?

![Installer](http://www.igorpecovnik.com/wp-content/uploads/2015/05/sata-installer.png)

Required condition:

NAND:

 * kernel 3.4.x and NAND storage
 * pre-installed system on NAND (stock Android or other Linux)

SATA/USB:

 * any kernel
 * pre-partitioned SATA or USB storage

Start the install script: 

    cd /root
    ./nand-sata-install

and follow the guide. You can create up to three scenarios:

 * boot from SD, system on SATA / USB
 * boot from NAND, system on NAND
 * boot from NAND, system on SATA / USB

# How to change network configuration?

There are five predefined configurations, you can find them in those files:

	/etc/network/interfaces.default
	/etc/network/interfaces.hostapd	
	/etc/network/interfaces.bonding
	/etc/network/interfaces.r1
	/etc/network/interfaces.r1switch

By default **/etc/network/interfaces** is symlinked to **/etc/network/interfaces.default**

1. DEFAULT: your network adapters are connected classical way. 
2. HOSTAPD: your network adapters are bridged together and bridge is connected to the network. This allows you to have your AP connected directly to your router.
3. BONDING: your network adapters are bonded in fail safe / "notebook" way.
4. Router configuration for Lamobo R1 / Banana R1.
5. Switch configuration for Lamobo R1 / Banana R1.

You can switch configuration with re-linking.

	cd /etc/network
	ln -sf interfaces.x interfaces
(x = default,hostapd,bonding,r1)

Than check / alter your interfaces:

	nano /etc/network/interfaces

# How to set fixed IP?

By default your main network adapter's IP is assigned by your router DHCP server.

	iface eth0 inet dhcp 

change to - for example:

	iface eth0 inet static
	 	address 192.168.1.100
        netmask 255.255.255.0
		gateway 192.168.1.1

# How to set wireless access point?

There are two different hostap daemons. One is **default** and the other one is for some **Realtek** wifi cards. Both have their own basic configurations and both are patched to gain maximum performances.

Sources: [https://github.com/igorpecovnik/hostapd](https://github.com/igorpecovnik/hostapd "https://github.com/igorpecovnik/hostapd")

Default binary and configuration location:

	/usr/sbin/hostapd
	/etc/hostapd.conf
	
Realtek binary and configuration location:

	/usr/sbin/hostapd-rt
	/etc/hostapd.conf-rt

Since its hard to define when to use which you always try both combinations in case of troubles. To start AP automatically:

1. Edit /etc/init.d/hostapd and add/alter location of your conf file **DAEMON_CONF=/etc/hostapd.conf** and binary **DAEMON_SBIN=/usr/sbin/hostapd**
2. Link **/etc/network/interfaces.hostapd** to **/etc/network/interfaces**
3. Reboot
4. Predefined network name: "BOARD NAME" password: 12345678
5. To change parameters, edit /etc/hostapd.conf BTW: You can get WPAPSK the long blob from wpa_passphrase YOURNAME YOURPASS

# How to connect IR remote?

Required conditions: 

- IR hardware
- loaded driver

Get your [remote configuration](http://lirc.sourceforge.net/remotes/) (lircd.conf) or [learn](http://kodi.wiki/view/HOW-TO:Setup_Lirc#Learning_Commands). You are going to need the list of all possible commands which you can map to your IR remote keys:
	
	irrecord --list-namespace

To start with learning process you need to delete old config:
		
	rm /etc/lircd.conf 

Than start the process with:

	irrecord --driver=default --device=/dev/lirc0 /etc/lircd.conf

And finally start your service when done with learning:

	service lirc start

Test your remote:

	irw /dev/lircd
