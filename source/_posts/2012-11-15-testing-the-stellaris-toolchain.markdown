---

layout: post
title: "Testing the Stellaris toolchain"
date: 2012-11-15 20:40
comments: true
categories: arm
sidebar: false

---

With the [toolchain] installed, here's the basic steps to compile an example and get the code onto the launchpad.  A mirror for the Stellaris example code and other pieces is available on [github].

[toolchain]: https://github.com/jsnyder/arm-eabi-toolchain
[macosx]: /blog/2012/11/15/compiling-the-stellaris-toolchain/
[github]: https://github.com/yuvadm/stellaris

	# Add the toolchain to your profile's path
	# Ideally this should go into your bash profile
	$ export PATH=$HOME/arm-cs-tools/bin:$PATH
 
	# Test the compiler
	$ git clone https://github.com/yuvadm/stellaris
	$ cd stellaris/boards/ek-lm4f120xl/project0/
	$ make clean; make
	
	# Flash binary to the board
	$ lm4flash gcc/project0.bin
	
	# Try making a source modification
	$ nano project0.c
	$ make && lm4flash gcc/project0.bin


## debugging

[lm4tools] has a bridge to enable TCP over USB, so code running on the device can be debugged using gdb from the compiled toolchain.

[lm4tools]: https://github.com/utzig/lm4toolsn
	
	# build with debug symbols
	$ make clean && DEBUG=1 make
	$ lm4flash gcc/project0.bin
	
	# start the tcp/usb bridge (in the background)
	$ lmicdi &
	
	# start gdb and connect to device
	$ arm-none-eabi-gdb gcc/project0.axf
	Reading symbols from ./gcc/project0.axf...done.
	(gdb) target remote :7777
	Remote debugging using :7777
	0x00000494 in SysCtlDelay ()
	(gdb) c
	Continuing.
	
Note: As well as quitting lm4flash, I also found ``detach`` was needed in gdb (even if it hadn't yet been ran), else the board would require a power cycle before flashing was possible again.