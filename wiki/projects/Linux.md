---
title: Linux
permalink: /Linux/
parent: Projects, programs and tools
layout: page
---

This page documents the Linux port to the Nspire calculator. There are
currently two versions available on the TI-Nspire. All new development
will occur on the new kernel however, the legacy kernel is easier to get
going.

---
1. TOC
{:toc}
---

## Booting

To have something useful running using Linux on the calculator, a
bootloader, kernel and rootfs is needed.

### Bootloader

Source code: [On github](https://github.com/tangrs/nspire-linux-loader2)

Nightly builds: None.

Binary:
[TI-Planet](http://tiplanet.org/nspire-linux-builds/linuxloader2.tns)

The bootloader is run from the Nspire OS to load everything into memory
and execute the kernel.

Usage instructions can be found in the
[readme](https://github.com/tangrs/nspire-linux-loader2/blob/master/README.md)

```
 Copy linuxloader.tns to your calculator and run it.
 
 Valid commands are:
 
     kernel <filename>: Loads a kernel image into memory
     initrd <filename>: Loads a ramdisk into memory
     dtb <filename>: Loads a DTB image into memory
     dump: Prints out the current internal state of the bootloader. Useful for
 debugging.
     free: Prints out the total amount of memory provided to the bootloader by
 the Nspire OS and amounts used by the kernel and ramdisks.
     cmdline [str]: Get/set the kernel command line parameters.
     mach [id]: Get/set the machine ID that will be provided to Linux upon
 booting. Useful for overriding the builtin default value without having to
 recompile.
     phys [<start> <size>]: Get/set the address and size of physical memory.
 Useful for overriding the builtin default value without having to recompile.
     rdsize [size]: Get/set the size of the ramdisk that Linux should create on
 boot. Leave at 0x0 for the kernel default.
     probemem: If this is run on an calculator model that isn't directly
 supported by the bootloader, you can use this to try and guess how much memory
 the system has.
     poke <addr> <value>: Write a word to an arbitrary location in the memory
 address space.
     peek <addr>: Read a word from an arbitrary location in the memory address
 space.
     boot: Boot kernel.
 
 The bootloader is also scriptable. Create a text file containing a list of
 commands to be executed and change the extension to .ll2.tns, and add the
 following line to your /documents/ndless/ndless.cfg.tns file.
 
 ext.ll2=linuxloader2
 
 Then simply open your script file and the loader will execute all the commands
 in it. A sample one could look like this:
 
 kernel linux/zImage.tns
 initrd linux/initrd.tns
 cmdline root=/dev/ram
 boot
 
 This should save a lot of typing everytime you need to boot Linux.
```

To boot the device tree (newer) kernel, ensure you add a dtb command to
load the correct device tree prior to booting.

```
 kernel linux/zImage.tns
 initrd linux/initrd.tns
 dtb linux/dtb.tns
 cmdline root=/dev/ram
 boot
```

#### UART

You can get serial output on the UART. Remember to add `,115200n8` to
the end of your `console=` command line options to keep the baud rate
the same as the Nspire OS.

### Kernel

Source code: see sections below

Nightly builds:
[TI-Planet](http://tiplanet.org/nspire-linux-builds/kernel.html)

Please see below sections for the differences between the legacy and
newer kernels.

To compile the legacy kernel with default options, first clone the
github repo then run:

```
  export ARCH=arm 
  make nspire_defconfig 
  make -j4 
  cp -v arch/arm/boot/zImage /path/to/folder/zImage.tns 
```

Unfortunately, the newer kernel does not have a defconfig at the moment.
You have to enable options manually. Check the nightly builds for
example configs you can use.

```
  export ARCH=arm 
  make menuconfig 
  make -j4 
  make dtbs 
  cp -v arch/arm/boot/zImage /path/to/folder/zImage.tns 
  cp -v arch/arm/boot/dts/nspire-*.dtb /path/to/folder/ 
```

### Rootfs

Nightly builds:
[TI-Planet](http://tiplanet.org/nspire-linux-builds/buildroot.html)
(WIP)

To have programs actually run on the kernel, you need a root filesystem
of some sort containing the userspace programs.

For testing and mucking around, a initrd should be enough. It is
possible to build one using [Buildroot](http://buildroot.uclibc.org/).
Otherwise, there are a few on the nightly builds page and the Omnimaga
thread.

For a larger root filesystem, you could put a filesystem on a USB drive
and add `root=/dev/sdaX` to your kernel command line. You may also need
to add `rootdelay=10` to give the USB drive time to initialize. There
have also been successes running Arch Linux for ARM and Debian.

## Device tree kernel

The newer version is a total rewrite of the original code to switch to
device trees and has slightly less hardware support (but better USB
support). All development in this kernel will go directly to the
mainland kernel. The source code can be found at
[kernel.org](http://kernel.org).

People interested in using this branch also have to compile the device
tree blobs in addition to the kernel by running:

` make dtbs`

and include it in their bootscripts with:

` dtb linux/devicetree.tns`

Ensure you are using a version of linuxloader2 with a build date after
May 11 2013 (or commits later than 0503ca6) for stable device tree
support. If your linuxloader2 does not report a build date on startup,
you need to upgrade your copy. You can get an updated copy from the
TI-Planet nightly builds site.

### Hardware support

| Hardware | Classic | CX | In mainline | Notes |
|----|----|----|----|----|
| CPU | Yes | Yes | Yes |  |
| SRAM | Yes | Yes | No | Matter of adding a few lines to the device tree |
| GPIO | Yes | Yes | Yes |  |
| UART1 | Yes | Yes | Yes |  |
| I2C | Yes | Yes | WIP | Waiting to be accepted into mainline. Have not received extensive testing. |
| Watchdog timer | Yes |  | Yes | Have not tested. Just enable support for SP805 in the config |
| RTC | No | No | No | Should be a matter of adding a few lines to the device tree |
| Power management | No | No | No | Difficult to cleanly implement |
| Timer 1 | Yes | Yes | Yes |  |
| Timer 2 | Yes | Yes | Yes |  |
| Keypad | Yes |  | Yes |  |
| Touchpad | Yes |  | WIP | Waiting to be accepted into mainline. Have not received extensive testing. |
| LCD Contrast/Backlight | No | No | No | Should be a matter of adding a few lines to the device tree |
| LED | WIP |  | No | Unlocking the LED is already possible |
| USB OTG | Yes |  | Yes | Everything working include seamless switching between USB host and USB device mode. |
| NAND | WIP | WIP | No | Dammit, why does the TI-Nspire have such weird NAND controllers to work with? |
| LCD | Yes |  | Yes |  |
| ADC | No | No | No |  |
| DES encryption/SHA generator | No | No | No |  |
| Interrupt controller | Yes | Yes | Yes |  |

### Notes

#### Enabling USB OTG support

USB OTG with seamless switching relies on a few different options in the
kernel. Ensure you’ve enabled the following:

- Zevio GPIO driver - CONFIG_GPIO_ZEVIO
- USB NOP transceiver - CONFIG_NOP_USB_XCEIV
- Fixed voltage regulator driver - CONFIG_REGULATOR_FIXED_VOLTAGE

The NOP transceiver is passed onto the Chipidea driver because the
TI-Nspire doesn’t need/have a PHY.

The fixed voltage regulator driver is the bridge between the GPIO and
Chipidea. It controls 5V output to the USB port.

## Legacy kernel

This kernel is not being maintained any more. All new development will
occur on the new kernel. This kernel is the initial port of Linux.

### Hardware support

#### Classic

| Hardware | Possible? | Implemented? | Notes |
|----|----|----|----|
| CPU | Yes | Yes |  |
| SDRAM | Yes | Yes |  |
| SRAM | Yes | Yes |  |
| GPIO | Yes | Yes |  |
| UART1 | Yes | Yes |  |
| I2C | Yes | Yes |  |
| Watchdog timer | Yes | Yes |  |
| RTC | Yes | Yes |  |
| Power management | Yes | WIP | Basic CPU frequency scaling has been implemented |
| Timer 1 | Yes | No |  |
| Timer 2 | Yes | Yes |  |
| Keypad | Yes | Yes |  |
| Touchpad | Yes | Yes |  |
| LCD Contrast/Backlight | Yes | Basic |  |
| TI-84 Link port | Unknown | No | Do we really need this? |
| LED | Yes | No |  |
| SPI | Yes | No |  |
| USB OTG | Yes | Yes |  |
| USB Host | Yes | Yes |  |
| NAND | Unknown | No |  |
| LCD | Yes | Yes |  |
| ADC | Yes | Yes |  |
| DES encryption/SHA generator | Yes | No |  |
| Interrupt controller | Yes | Yes |  |

#### CX

| Hardware | Possible? | Implemented? | Notes |
|----|----|----|----|
| CPU | Yes | Yes |  |
| SDRAM | Yes | Yes |  |
| SRAM | Yes | Yes |  |
| GPIO | Yes | Yes |  |
| UART1 | Yes | Yes |  |
| I2C | Yes | Yes |  |
| Watchdog timer | Yes | Yes |  |
| RTC | Yes | Yes |  |
| Power management | Yes | WIP | Basic CPU frequency scaling has been implemented |
| Timer 1 | Yes | No |  |
| Timer 2 | Yes | Yes |  |
| Keypad | Yes | Yes |  |
| Touchpad | Yes | Yes |  |
| LCD Contrast/Backlight | Yes | Yes |  |
| TI-84 Link port | Unknown | No | Do we really need this? |
| LED | Yes | No |  |
| SPI | Yes | No |  |
| USB OTG | Yes | Yes | Current implementation breaks USB Host (see notes) |
| USB Host | Yes | USB 1.1 | Uses a workaround but limits speed to USB 1.1 |
| NAND | Yes | WIP | Needs a lot of testing. Unstable. Note - driver for a similar chip is available [here](https://github.com/Xilinx/linux-xlnx/blob/master/drivers/mtd/nand/zynq_nand.c) |
| LCD | Yes | Yes |  |
| ADC | Yes | Yes |  |
| DES encryption/SHA generator | Yes | No |  |
| Interrupt controller | Yes | Yes |  |

### Quirks and technical details

#### SRAM

When needing to allocate memory from SRAM when developing for the
kernel, use the following functions:

```
void *sram_alloc(unsigned int size, dma_addr_t *dma_addr)
void sram_free(dma_addr_t addr, unsigned int size)
```

#### Keypad

The keymaps can be found in `arch/arm/mach-nspire/keypad.c`. Each array
element represents one bit in
<a href="/Keypads" class="wikilink" title="Keypads">Keypads</a>.

#### Touchpad

There are some weird issues with the Touchpad. After booting Linux, the
Touchpad doesn't function correctly under Nspire OS and requires
entering diags or a few hard resets to get it back to normal. The exact
cause is unknown but it is possible that this behavior is caused by the
bootloader not correctly resetting Touchpad controller when the device
boots up again so the Nspire OS is still using a Linux-configured
Touchpad controller.

It has been half solved by having the Touchpad driver reset the
controller on a soft reboot. However, this doesn't happen on hard
reboots and will still cause the Nspire OS to not function correctly.
Whenever possible, perform a soft reboot in Nspire Linux by using
Ctrl+Alt(Var)+Delete(Scratchpad) or using the reboot command.

#### LCD contrast/backlight

At the moment, contrast settings can be found at `/proc/contrast` for
classic calculators though this will be deprecated soon. The CX already
has a working driver for backlight control.

#### USB

USB support is mostly working. The only thing missing is a USB PHY
driver. The USB controller driver in this kernel is currently unable to
switch seamlessly between USB host and USB device mode because of this.

To work around this, we get the Nspire OS to set the right modes before
booting Linux.

For USB Host mode: insert a USB OTG cable's A end or a USB device into
the calculator while still inside the Nspire OS before running the
bootloader.

For USB Device mode: connect the Nspire to a computer while in Nspire OS
before running the bootloader.

The USB hardware on the calculators also provide very little power
(something in the double digit milliamps). For anything other than a
basic USB drive or a keyboard, a powered USB hub might be required to
supply enough power.

#### Extra notes for CX

Both OTG and USB host work on the CX but since it needs a workaround
that can't be integrated into the OTG driver, only one will work at a
time. To maintain compatibility, the default is to use the USB host only
driver. If you wish to use USB OTG on the CX, simply add `cx_usb_otg` to
your kernel command line arguments.

#### ADC

ADC values can be read at `/sys/bus/iio/devices/iio:device0` (310 = 1V).

## More information

- \[<http://www.omnimaga.org/ti-nspire-projects/calling-all-linux-kernel-developers>!
  Omnimaga thread\]
- [FAQ](http://www.omnimaga.org/ti-nspire-projects/calling-all-linux-kernel-developers!/msg323459/#msg323459)
- [TI-Planet nightly builds](http://tiplanet.org/nspire-linux-builds/)
- [Linux configs](https://github.com/Vogtinator/nspire-linux-configs)