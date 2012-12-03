---

layout: post
title: "Compiling the Stellaris toolchain"
date: 2012-11-15 20:02
comments: true
categories: [stellaris, arm, hacking]
_permalink: stellaris-toolchain
sidebar: false

---

If you have a Stellaris [Launchpad] - and don't want to use the [official] tools - you can mostly follow the instructions at [y3xz] to build your own toolchain on any Unix/Linux based system using the [ARM EABI Toolchain Builder][EABI].  This includes Mac OS X, but I ran into a couple of minor issues as listed below.

[Stellaris]: http://www.ti.com/product/lm4f120h5qr
[Launchpad]: http://www.ti.com/ww/en/launchpad/stellaris_head.html
[y3xz]: http://blog.y3xz.com/blog/2012/10/29/an-open-toolchain-for-the-ti-stellaris/
[official]: http://www.ti.com/tool/SW-EK-LM4F120XL
[EABI]: https://github.com/jsnyder/arm-eabi-toolchain

Note, the Codesourcery Lite [toolchain][mentor] used here does not support the hardware FPU of the ARM [Cortex-M4F], using software floating point code instead.  There is a [hardfloat-toolchain] builder (which I've not used yet), and ARM is maintaining a [GCC toolchain][launchpad.net] targeting embedded ARM processors, which I'll probably try building next.

For more details, T.I. have a selection of [technical documentation][Stellaris] on their site.


[Cortex-M4F]: http://en.wikipedia.org/wiki/ARM_Cortex-M#Cortex-M4
[launchpad.net]: https://launchpad.net/gcc-arm-embedded/+download
[hardfloat-toolchain]: https://github.com/prattmic/arm-cortex-m4-hardfloat-toolchain


## toolchain

The makefile failed to download the source archive, you can [download] it manually into the root of the toolchain repo and make will continue so long as the checksum matches.

[mentor]: http://www.mentor.com/embedded-software/sourcery-tools/sourcery-codebench/editions/lite-edition

[download]: https://sourcery.mentor.com/GNUToolchain/package10384/public/arm-none-eabi/arm-2012.03-56-arm-none-eabi.src.tar.bz2

[mylink]: https://sourcery.mentor.com/GNUToolchain/subscription3053?lite=arm&lite=ARM&signature=4-1352914385-0-81777d693584c1d30acc48c7abaf41235a1766c3


## lm4tools

The [lm4tools] binary couldn't read the serial number of the device using my MacBook running OS X 10.8, so I couldn't flash the binary.

``` sh
$ ./lm4flash/lm4flash project0.bin
Unable to get device serial number: LIBUSB_ERROR_OTHER
Unable to find any ICDI devices
```

The simplest workaround is to checkout and build commit [cc49426], which doesn't check for the serial number.  To use newer versions of lm4tools, a kernel extension needs to be installed, see the [issue] on github for more info.

Btw, you will get a similar error if your system requires root privileges to access the device directly over usb, try using `sudo` on linux/unix systems if you have issues.

[lm4tools]: https://github.com/utzig/lm4tools
[issue]: https://github.com/utzig/lm4tools/issues/8
[cc49426]: https://github.com/utzig/lm4tools/commit/cc49426081


## more links

[Setting up the GCC ARM Toolchain](http://hertaville.com/2012/05/28/gcc-arm-toolchain-stm32f0discovery/) - focuses on using ARM's [GCC toolchain][launchpad.net] on Windows

An older [programming tool](https://github.com/texane/stlink) with some good info in the README ([and another one here](https://github.com/utzig/icdiflasher)).

