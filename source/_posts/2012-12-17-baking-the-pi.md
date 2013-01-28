---

layout: post
title: "Home Theatre Pi (HTPi)"
date: 2012-12-17 19:13
comments: true
sidebar: false
categories: [arm, pi, htpc]

---


A collection of notes for setting up a clean image of [XBian](http://www.xbian.org) (1.0 Alpha 4) on the Raspberry Pi.

<!-- more -->

Jump to [first login tasks](#first-login-tasks) if you have already set up terminal access to the Pi.

[^Todo]: Install fsck.ntfs (used to fix unclean unmount)

## Basic Pi Tips

* Guide for navigating XBMC with the [keyboard]
* Login using hostname with zeroconfig: `ssh user@host.local`

[keyboard]: http://wiki.xbmc.org/index.php?title=Keyboard


## Pre-setup, on PC, laptop, etc.

Before setting up the Pi remotely, there are some things to do locally to ease logging in when using SSH.  This will obviate the need to enter a password, or specify the full host name each time we access the Pi.

### Copy public key to Pi

If generating a new key pair, accept the default key location as suggested by ``ssh-keygen`` below.  While a passphrase is optional, anyone can use a copy of the unencrypted private key to authenticate with your identity.  Many operating systems are preconfigured to use ``ssh-agent`` or a similar utility, to avoid having to enter a passphrase multiple times (if at all).


```sh
	# generate a key pair if none already exists
	test -f ~/.ssh/id_rsa.pub || ssh-keygen

	# remove any old, conflicting host entries
	ssh-keygen -R xbian.local

	# password is required until the key is installed
	ssh-copy-id xbian@xbian.local
```

The easiest way to append a key to the remote user's `~/.ssh/authorized_keys` file is to use `ssh-copy-id` as shown above.  Download [ssh-copy-id] from source if your system doesn't already have it ([installation instructions]), this [alternative method] should also work in most situations.

[ssh-copy-id]: http://hg.mindrot.org/openssh/raw-file/tip/contrib/ssh-copy-id
[installation instructions]: http://www.commandlinefu.com/commands/view/10228/...if-you-have-sudo-access-you-could-just-install-ssh-copy-id-mac-users-take-note.-this-is-how-you-install-ssh-copy-id-
[alternative method]: http://www.commandlinefu.com/commands/view/188/copy-your-ssh-public-key-to-a-server-from-a-machine-that-doesnt-have-ssh-copy-id


### Add an alias to .ssh/config

Locally define the alias `xb`, to be used in place of `xbian@xbian.local` with commands such as `ssh xb` and `sftp xb`.  Enter the text below as a single command, or manually paste the quoted text into ``~/.ssh/config`` using ``nano`` or similar.

```sh
	echo '
	Host xb
	  User  xbian
	  Hostname  xbian.local' >> ~/.ssh/config
```


## Transfer files

If you have previous files from your Pi stored locally, you can transfer them using `sftp`, `scp`, etc.  For easily transferring many arbitrary files , a GUI sftp client is recommended.

```sh
	# connect to the pi, using the `xb` alias defined above
	sftp xb
	sftp> put .bash_aliases
	sftp> exit

	# you can also use ctlr+d to logout
```

## First login tasks

Most of the commands below need root privileges on the Pi, as they alter the system configuration.  To run a single command with root privileges, prepend the `sudo` command to it.  To run many commands this way without typing `sudo` each time, first start a root shell with `sudo -s`; *remember to logout with 'exit' or ctrl+d when finished*.

```sh
	# Login to the pi, using the `xb` alias
	ssh xb

	# xbian-config may run here, set it up as you like then exit

	# Disable the login message
	touch ~/.hushlogin

	# Disable setting of the terminal title
	sed -i '/PS1=.*\][0-2];/s/^/##/' ~/.bashrc

	# Use a root shell for the following commands
	sudo -s

	# Allow full use of sudo without needing password
	# note: XBian 1.0b4 made this much less essential
	echo '%sudo  ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers

	# Disable autorun of configuration menu,
	# run manually with: 'sudo xbian-config'
	sed -i '1 i return' /etc/profile.d/xbian-config.sh

	# Prevent hideoutput from overwriting the prompt
	sed -i '/40;0m/s/^/##/' /etc/profile.d/hideoutput.sh

	# Update packages
	apt-get update && apt-get upgrade

	# Install some utilities
	apt-get install p7zip zip
```

## TTL serial console

Using a TTL to USB serial connection to the Pi, I'm unable to use full screen terminal programs due to screen corruption.  Changing the terminal type enables use of programs such as ``nano`` and ``xbian-config``.

```sh
	sed -i '/ttyAMA0/s/vt100$/xterm/' /etc/inittab
```

## Fake a hardware clock

The Pi doesn't have a real time clock, so it usually defaults to some point in the past until the time can be set correctly using the Internet.  To make the clock more consistent across power cycles, it can be initialised using the last recorded date and time.  *(note: previous distros required the [unabridged instructions](#fake-a-hardware-clock-unabridged).)*

```sh
	apt-get install fake-hwclock
```

## Configure WiFi adapter

XBian includes simple menu driven WiFi configuration as part of xbian-config.  See the [manual instructions](#manually-configure-wifi-adapter) for connecting to multiple networks.

## Link settings to root profile

Occasionally you'll want to use a root shell, and then be annoyed that your aliases etc. are not configured for the root user.  You can either copy the profile files into ~/root or, as shown here, link them symbolically so that any future modifications will be reflected.

```sh
	# change to the home folder you want to link from
	cd ~xbian

	# define a list of dot files to link
	dotfiles=".profile .bashrc .bash_aliases .bash_logout .nanorc .toprc"

	# link each existing file into the root user's home folder
	for i in $dotfiles; do [ -f $i ] && sudo ln -sfv ~+/$i ~root/; done

	# hush login message
	sudo touch ~root/.hushlogin
```


## Generate new RSA host keys

These keys confirm the identity of the Pi, to prevent a malicious host from intercepting the remote login process.  Not so important for your HTPi, but good standard security practice.  Also recommended if you have more than one Pi.

```sh
	rm /etc/ssh/ssh_host_*
	/usr/sbin/dpkg-reconfigure openssh-server

	# remove old host key from clients using:
	# ssh-keygen -R xbian.local
```
<!--
	ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
	ssh-keygen -t rsa1 -f /etc/ssh/ssh_host_key
-->

## Build and install WiringPi

[WiringPi] is a library to access the Pi's GPIO, SPI, and I2C headers, modelled on the Arduino Wiring system.  It also includes the ``gpio`` utility for use of the libraries from the command prompt.

[WiringPi]: https://projects.drogon.net/raspberry-pi/wiringpi

```sh
	apt-get install gcc make git-core libi2c-dev
	git clone git://git.drogon.net/wiringPi
	cd wiringPi; ./build

	# test with gpio utility
	gpio readall
```

## Install dbox (dropbox tool)

Follow the [dbox installation instructions][dbox] to set up the dropbox sdk developer keys and authorisation tokens.  To use dbox for automatic folder syncing, see my post: [Dropbox on Pi](/blog/pi-box/).

[dbox]: https://github.com/kenpratt/dbox


```sh
	apt-get install gcc make ruby ruby-dev libsqlite3-dev
	gem install dbox
```

## Install Mono

Basic mono setup to run and compile command line tools.

```sh
	apt-get install mono-devel mono-gmcs mono-csharp-shell
```

## Installing iPlayer (and 4od, ITV, 5)

Use `wget` to download the latest versions from the links below, then open the zip files directly from xbmc's addons page.

<http://code.google.com/p/xbmc-iplayerv2/downloads/list>
<http://code.google.com/p/mossy-xbmc-repo/downloads/list>
<http://code.google.com/p/xbmc-itv-player/downloads/list>


<br />

# Useful extras, not always used

Stuff used infrequently, or currently being tested


## Packages from Raspbian

Some standard packages that are usually excluded from the xbian distro, as they are not required for use of xbmc only.

```sh
	apt-get install omxplayer

	# included on xbian >= 1.0b
	apt-get install psmisc usbutils
```

## Other useful packages

```sh
	apt-get install fs2resize exfat-fuse
	apt-get install clang geany
	apt-get install sysv-rc-conf
```

## Testing PVR

```sh
	apt-get install vdr-plugin-dvbsddevice
```

## Setting up webcam

Use 'motion' or 'fswebcam', motion may need a default cfg copying

```sh
	apt-get install motion
	cp /etc/default/motion /etc/motion/motion.conf
```

## Configure bluetooth adapter

[Adapted from ctheroux](http://www.ctheroux.com/2012/08/a-step-by-step-guide-to-setup-a-bluetooth-keyboard-and-mouse-on-the-raspberry-pi/).

```sh
	# install bluetooth support and dependencies
	agi bluez python-gobject  # minimal?
	agi bluetooth bluez-utils  # full

	# for management from the desktop
	agi blueman

	# check adapter is working*
	hcitool dev

	# scan for devices
	hcitool scan

	# pair with device, using the address listed from scan
	bluez-simple-agent hci0 XX:XX:XX:XX:XX:XX

	# trust the device
	bluez-test-device trusted XX:XX:XX:XX:XX:XX yes

	# connect to input device
	bluez-test-input hci0 XX:XX:XX:XX:XX:XX

	# adapter status
	hciconfig
```

**Note: a bluetooth adapter may be listed in ``lsusb`` and ``hciconfig``, without being recognised by ``hcitool``. This is the case with the belkin dongle I have, so use ``hcitool`` to check that a device is working properly.*


<br />

# Troubleshooting / backup

Some useful commands and procedures


## Backup settings

- Settings, addons etc. are in ~/.xbmc
- .xbmc/userdata - preferences etc
- .xbmc/addons - binaries, themes
- .xbmc/addons/packages - original downloads, can use with "install from zip"

```sh
	# backup profile settings
	zip -ry xbmc .xbmc/
	zip -ry dotfiles .bash_aliases .nanorc .toprc .ssh

	# backup system config files
	sudo zip -ry basecfg /etc/wpa_supplicant/wpa_supplicant.conf
```

## Clear cached network adapter

(needed for switching cards between devices)

```sh
    echo | sudo tee /etc/udev/rules.d/70-persistent-net.rules
```


## Quick Tips

* You can detect hdmi audio modes: `/opt/vc/bin/tvservice -a`
* Setup CEC remote over hdmi from console: `cec-config`


<br />

# Not used/needed with recent versions

[^Reaper]: http://f.cl.ly/items/0S1S1Y0B3Q241Z2F0z1j/Untitled.png

Previously useful functionality or workarounds


## Manually configure WiFi adapter

[Instructions adapted from here](http://www.savagehomeautomation.com/raspi-airlink101).

Use ``lsusb`` to check that the adapter is recognised, and ``lsmod`` to check the kernel module (e.g. ``8192cu``) is loaded.

```sh
	sudo nano /etc/network/interfaces
```

Make sure the following lines exist in the interfaces file, adding them as needed:

```
    allow-hotplug wlan0
    iface wlan0 inet manual
    wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
```

You may have the line ``wireless-power off`` in this file, which relates to power ***management*** only.  I've commented it out as it resulted in errors logged during ``ifup`` and power management remained off without it.

```sh
	sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

Add your network details to wpa_supplicant.conf, using the following template:

```
    network={
      ssid="YOUR-NETWORK-SSID"
      proto=WPA2
      key_mgmt=WPA-PSK
      pairwise=CCMP TKIP
      group=CCMP TKIP
      psk="YOUR-WLAN-PASSWORD"
    }
```

Reinitialise the adapter, and check it's connected.

```sh
	sudo ifdown wlan0
	sudo ifup wlan0
	# you may get some errors here, even when successful
```

Use ``iwconfig`` to view wifi adapter info and ``ifconfig`` for general network info.

## Fake a hardware clock (unabridged)

More complicated instructions, as used on previous versions of XBian.

```sh
	touch /etc/init.d/hwclock.sh
	/etc/init.d/ntp restart
	apt-get install ntpdate fake-hwclock
	ntpdate-debian
	dpkg-reconfigure tzdata
	sed -i 's/^exit 0/ntpdate-debian\nexit 0/g' /etc/rc.local
```

## Fix ssh access using public key

```sh
	# must be owned by root
	chown root: ~ ~/.ssh
	# no write for others
	chmod a=rx,u+w  ~

	# no access for others
	chmod -R go-rwx ~/.ssh
	# public key can be readable
	chmod a+r ~/.ssh/id_rsa.pub
```

## Download OpenSSH sftp server

```sh
	apt-get -d install openssh-server
	cp /var/cache/apt/archives/openssh-server_*.deb .
	dpkg-deb -X openssh-server_*.deb sftp
	cp sftp/usr/lib/openssh/sftp-server /usr/lib/
	rm -r sftp openssh-server_*.deb
```

## Change Hostname

```sh
    sudo nano /etc/hostname # enter the desired name
    sudo nano /etc/hosts # replace the hostname
    sudo /etc/init.d/hostname.sh start # to enable the changes
```

## Update firmware without kernel


```sh
    wget http://goo.gl/1BOfJ -O /usr/bin/rpi-update && chmod +x /usr/bin/rpi-update
    SKIP_KERNEL=1 rpi-update 128
```

## Install Shairport

Instructions found [here](http://tomsolari.id.au/post/27169019561/airplay-music-streaming-on-raspberry-pi) (alt site [here](http://cheeftun.appspot.com/trouch.com/2012/08/03/airpi-airplay-audio-with-raspberry/))

A change in IOS 6 [requires Perl Net-SDP](http://jordanburgess.com/post/38986434391/raspberry-pi-airplay) module to installed.

```sh
	git clone https://github.com/njh/perl-net-sdp.git perl-net-sdp
	cd perl-net-sdp
	perl Build.PL
	sudo ./Build
	sudo ./Build test
	sudo ./Build install
	cd ..
```

```sh
	# do as root
	sudo -s
	apt-get install alsa-utils
	modprobe snd_bcm2835
	# optionally set to headphone output
	# amixer cset numid=3 1
	# optionally restore to hdmi output
	# amixer cset numid=3 2
	apt-get install build-essential libssl-dev libcrypt-openssl-rsa-perl libao-dev libio-socket-inet6-perl libwww-perl avahi-utils pkg-config
	wget https://github.com/albertz/shairport/zipball/master
	unzip master
	cd albertz-shairport-*
	make install
	cp shairport.init.sample /etc/init.d/shairport
	# add to start of shairport: modprobe snd_bcm2835
	nano /etc/init.d/shairport
	# optionally edit name of service (remove port number):
	nano /usr/local/bin/shairport.pl
	insserv shairport
	# manually start [services](http://pi-raspberry.blogspot.co.uk/2012/08/shairport-raspberry-pi.html)
	service avahi-daemon start
	/etc/init.d/shairport start
	# exit root
	exit
```

