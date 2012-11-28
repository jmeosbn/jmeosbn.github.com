---

layout: post
title: "Compiling the Stellaris toolchain"
date: 2012-11-15 20:02
comments: true
categories: [arm, hacking]
permalink: stellaris-toolchain
sidebar: false

---

If you have Mac OS X and a Stellaris Launchpad, you can mostly follow the instructions at [y3xz] to build your own toolchain for it - the only issues I ran into are listed below.  Additionally, ARM is maintaining a GCC [toolchain] targeting embedded ARM processors. 

[toolchain]: https://launchpad.net/gcc-arm-embedded/+download

[y3xz]: http://blog.y3xz.com/blog/2012/10/29/an-open-toolchain-for-the-ti-stellaris/

## toolchain

The makefile for the [ARM EABI Toolchain Builder][EABI] failed to download the source archive from [mentor], you can [download] it manually into the root of the toolchain repo and make will continue so long as the checksum matches.
  
[EABI]: https://github.com/jsnyder/arm-eabi-toolchain

[mentor]: http://www.mentor.com/embedded-software/sourcery-tools/sourcery-codebench/editions/lite-edition/arm-eabi

[download]: https://sourcery.mentor.com/GNUToolchain/package10384/public/arm-none-eabi/arm-2012.03-56-arm-none-eabi.src.tar.bz2

[mylink]: https://sourcery.mentor.com/GNUToolchain/subscription3053?lite=arm&lite=ARM&signature=4-1352914385-0-81777d693584c1d30acc48c7abaf41235a1766c3

## lm4tools

The [lm4tools] binary couldn't read the serial number of the device using my MacBook running OS X 10.8 so the binary wasn't flashed.

	$ ./lm4flash/lm4flash project0.bin
	Unable to get device serial number: LIBUSB_ERROR_OTHER
	Unable to find any ICDI devices

The simplest workaround is to checkout and build commit [cc49426], which doesn't check for the serial number.  To use newer versions of lm4tools, a [kernel extension] needs to be installed.

For more info, see the [issue] on github.

[lm4tools]: https://github.com/utzig/lm4tools
[issue]: https://github.com/utzig/lm4tools/issues/8
[cc49426]: https://github.com/utzig/lm4tools/commit/cc49426081
[kernel extension]: http://utzig.net/files/lm4f120_icdi.tgz
