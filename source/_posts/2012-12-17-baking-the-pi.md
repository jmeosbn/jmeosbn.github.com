---

layout: post
title: "Home Theatre Pi (HTPi)"
date: 2012-12-17 19:13
comments: true
sidebar: false
categories: [arm, pi, htpc]

---


Just a collection of notes for setting up a clean image on the Raspberry Pi.


Notes are current for [XBian](http://www.xbian.org) 1.0 Alpha 3.

[^Todo]: Install fsck.ntfs (used to fix unclean unmount)

<!-- more -->

## Basic Pi Tips

* Guide for navigating XBMC with the [keyboard]
* Login using hostname with zeroconfig: `ssh user@host.local`

[keyboard]: http://wiki.xbmc.org/index.php?title=Keyboard


## Pre-setup, on PC, laptop, etc.

Before setting up the Pi remotely, there are some things to do locally to ease logging in.  This will obviate the need to enter the password, or specify the full host name each time we access the Pi.


### Copy public key to Pi

```sh
	# remove any old, conflicting host entries
	ssh-keygen -R xbian.local
	
	# generate key pair if none already exists
	test -f ~/.ssh/id_rsa.pub || ssh-keygen

	# password required until key installed
	ssh-copy-id xbian@xbian.local
```

The easiest way to append a key to the remote user's `~/.ssh/authorized_keys` file is to use `ssh-copy-id` as shown above.  Download [ssh-copy-id] from source if you don't have it ([installation instructions]), this [alternative method] should also work in most situations.

[ssh-copy-id]: http://hg.mindrot.org/openssh/raw-file/tip/contrib/ssh-copy-id
[installation instructions]: http://www.commandlinefu.com/commands/view/10228/...if-you-have-sudo-access-you-could-just-install-ssh-copy-id-mac-users-take-note.-this-is-how-you-install-ssh-copy-id-
[alternative method]: http://www.commandlinefu.com/commands/view/188/copy-your-ssh-public-key-to-a-server-from-a-machine-that-doesnt-have-ssh-copy-id
	
	
### Add an alias to .ssh/config

Locally define the alias `xb` to be used in place of `xbian@xbian.local`.

```sh
	echo '
	Host xb
	  User  xbian
	  Hostname  xbian.local' >> ~/.ssh/config
```

With this alias defined, `ssh xb` and `sftp xb` can be used to login.


## First login tasks

If you have any files stored locally, you can transfer them using `sftp`.

```sh
	# connect to the pi, using the `xb` alias defined above
	sftp xb
	sftp> put .profile
	sftp> exit
	
	# you can also use ctlr+d to logout
```

Most of the commands below need root privileges on the Pi, as they configure the system.  For a single command you can prepend `sudo`, for many commands it's easier to run `sudo -s` to start a new shell using sudo.

```sh
	# Login to the pi, using the `xb` alias
	ssh xb
	
	# xbian-config may run here, set it up as you like then exit

	# Prevent future display of login message
	touch ~/.hushlogin

	# Login as root for the remainder of the setup
	sudo -s

	# Allow full use of sudo without needing password
	echo '%sudo  ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers
	
	# Prevent autorun of xbian-config.sh,
	# moving it to /usr/bin to be executable
	mv /etc/profile.d/xbian-config.sh /usr/bin
	chmod +x /usr/bin/xbian-config.sh
```
	
## Update packages

```sh
	apt-get update && apt-get upgrade
```	

## Fake a hw clock based on recent timestamp

The Pi doesn't have any hardware to track time while powered off; using fake-hwclock, the time can be set during startup to the last known date and time.

```sh
	touch /etc/init.d/hwclock.sh
	/etc/init.d/ntp restart
	apt-get install ntpdate fake-hwclock
	ntpdate-debian
	dpkg-reconfigure tzdata
	sed -i 's/^exit 0/ntpdate-debian\nexit 0/g' /etc/rc.local
```

## Generate new RSA host keys

These keys confirm the identity of the Pi when logging in remotely, to prevent a malicious host from intercepting the process.  Not so important for your HTPi, but good standard security practice.  Also recommended if you have more than one Pi.

```sh
	ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
	ssh-keygen -t rsa1 -f /etc/ssh/ssh_host_key
```

## Install Mono

Basic mono setup to run and compile command line tools

```sh
	apt-get install mono-devel mono-gmcs mono-csharp-shell
```

## Packages from Raspbian

Some standard packages that are usually excluded from the xbian distro, as they are not required for use of xbmc only.

```sh
	apt-get install psmisc usbutils omxplayer
```

[^omx]: see <http://omxplayer.sconde.net/> for recent builds



<br />
# Useful extras, not always used

Stuff used infrequently, or currently being tested



## Installing iPlayer (and 4od, ITV, 5)

Download the latest versions from the links below, then open the zip files directly from xbmc's addons page.

<http://code.google.com/p/xbmc-iplayerv2/downloads/list>  
<http://code.google.com/p/mossy-xbmc-repo/downloads/list>  
<http://code.google.com/p/xbmc-itv-player/downloads/list>  


## Useful packages

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
	nano /etc/default/motion /etc/motion/motion.conf
```


<br />
# Troubleshooting / backup

Some useful commands and procedures



## Backup XBMC settings

- Settings, addons etc. are in ~/.xbmc
- .xbmc/userdata - preferences etc
- .xbmc/addons - binaries, themes
- .xbmc/addons/packages - original downloads, can use with "install from zip"


## Clear cached network adapter

(needed for switching cards between devices)

```sh
    echo | sudo tee /etc/udev/rules.d/70-persistent-net.rules
```

When the file opens, just erase the lines and save it.


## Quick Tips

* You can detect hdmi audio modes: `/opt/vc/bin/tvservice -a`
* Setup CEC remote over hdmi from console: `cec-config`



<br />
# Not used/needed with recent versions

[^Reaper]: http://f.cl.ly/items/0S1S1Y0B3Q241Z2F0z1j/Untitled.png

Previously useful functionality or workarounds



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
