---
title: Memory-mapped I/O ports on CX II
permalink: /Memory-mapped_IO_ports_on_CX_II/
parent: Memory-mapped I/O ports
layout: page
---

---
1. TOC
{:toc}
---

Not all parts have been discovered and researched yet, so the
information on this page is not complete.

## 00000000 - Boot1 ROM

128kB of on-chip ROM.

## 10000000 - SDRAM

64 MiB, managed by 0x90120000.

## 90000000 - General Purpose I/O (GPIO)

See <a href="/GPIO_Pins" class="wikilink" title="GPIO Pins">GPIO Pins</a>

## 90010000 - Fast timer

The same interface as 900C0000/900D0000, see
<a href="#900d0000---second-timer" class="wikilink"
title="Second timer">Second timer</a>.

For this timer, the configurable speed register defaults to no bits set.

## 90020000 - Serial UART

[PL011](http://infocenter.arm.com/help/topic/com.arm.doc.ddi0183f/DDI0183.pdf).

## 90030000 - Fastboot RAM

4KiB of RAM, not cleared on resets/reboots.

Only the lower 12 bits of the address are used, so the content aliases
at 0x1000 and so on.

The OS uses that to store some data which is used during boot to restore
the previous state of the device.

The installer images use the area at 0x200 to store some variables for
tracking the progress.

## 90040000 - SPI controller

FTSSP010 SPI controller connected to the LCD.

## 90050000 - I2C controller

The Touchpad on the CX II is accessed through this controller. See
<a href="/Keypads#Touchpad_I²C" class="wikilink"
title="Keypads#Touchpad I²C">Keypads#Touchpad I²C</a> for protocol
details. It seems to be a Synopsys Designware I2C adapter.

- 90050000 (R/W): Control register?
- 90050004 (?): ?
- 90050010 (R/W): Data/command register
- 90050014 (R/W): Speed divider for high period (standard speed) OS:
  0x9c
- 90050018 (R/W): Speed divider for low period (standard speed) OS: 0xea
- 9005001c (R/W): Speed divider for high period (high speed) OS: 0x3b
- 90050020 (R/W): Speed divider for low period (high speed) OS: 0x2b
- 9005002c (R/W?): Interrupt status
- 90050030 (R/W): Interrupt mask
- 90050040 (R/W): Interrupt clear. Write 1 bits to clear
- 9005006c (R/W): Enable register
- 90050070 (R): Status register
- 90050074 (R?/W): TX FIFO?
- 90050078 (R?/W): RX FIFO?
- 900500f4 (?): ?
- 90050080 (?): ?

## 90060000 - Watchdog timer

Possibly an [ARM
SP805](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.ddi0270b/index.html)
or compatible. Runs at the APB clock frequency.

## 90070000 - Second Serial UART

[PL011](http://infocenter.arm.com/help/topic/com.arm.doc.ddi0183f/DDI0183.pdf).

## 90080000 - Cradle SPI Controller

An FTSSP010 for communicating with the EEPROM in the cradle.

## 90090000 - Real-Time Clock (RTC)

Similar to the [ARM PrimeCell
PL031](http://infocenter.arm.com/help/topic/com.arm.doc.ddi0224b/index.html),
but interrupt registers are different.

- 90090000 (R): Current time, increments by 1 every second.
- 90090004 (R/W): Alarm value. When the time passes this, interrupt
  becomes active.
- 90090008 (R/W): Sets the value of 90090000 (clock will not read new
  time until a couple seconds later). Reads last value written.
- 9009000C (R/W): Interrupt mask (1-bit)
- 90090010 (R/W): Masked interrupt status, reads 1 if interrupt active
  and mask bit is set. Write 1 to acknowledge.
- 90090014 (R): Status
  - Bit 0: Time setting in progress
  - Bit 1: Alarm setting in progress
  - Bit 2: Interrupt acknowledgment in progress
  - Bit 3: Interrupt mask setting in progress

## 900A0000 - Miscellaneous

Seems to be similar to CX and Classic, except for the model ID at
900A0000 which is now 0x202.

## 900B0000 - ADC

A Faraday FTADCC010.

## 900C0000 - First timer

Same port structure as
<a href="#900d0000---second-timer" class="wikilink"
title="Second timer">Second timer</a>. 

For this timer, the configurable speed register defaults to bit 0 set.

## 900D0000 - Second timer

Timer is a
[SP804](https://developer.arm.com/documentation/ddi0271/latest/) with an
additional speed control register at +0x80.

- 900D0080 (R/W): Speed control register
  - Bits 0-1: Speed mode
    - No bits set: Fastest, runs at the APB clock frequency, typically
      99MHz on the CX II.
    - Bit 0 set: Runs at 12MHz(?).
    - Bit 1 set: Runs at 32768Hz(?). This bit takes priority over bit 0.

For this timer, the configurable speed register defaults to bit 1 set.

The Ndless SDK uses this timer for its implementation of msleep. Configuring
the speed control register to something other than the default will cause
msleep to not work correctly.

## 900E0000 - Keypad controller

See also <a href="/Keypads" class="wikilink" title="Keypads">Keypads</a>
for information about the keypads themselves.

- 900E0000 (R/W):
  - Bits 0-1: Scan mode
    - Mode 0: Idle.
    - Mode 1: Indiscriminate key detection. Data registers are not
      updated, but whenever any key is pressed, interrupt bit 2 is set
      (and cannot be cleared until the key is released).
    - Mode 2: Single scan. The keypad is scanned once, and then the mode
      returns to 0.
    - Mode 3: Continuous scan. When scanning completes, it just starts
      over again after a delay.
  - Bits 2-15: Number of APB cycles to wait before scanning each row
  - Bits 16-31: Number of APB cycles to wait between scans
- 900E0004 (R/W):
  - Bits 0-7: Number of rows to read (later rows are not updated in
    900E0010-900E002F, and just read as whatever they were before being
    disabled)
  - Bits 8-15: Number of columns to read (later column bits in a row are
    set to 1 when it is updated)
- 900E0008 (R/W): Keypad interrupt status/acknowledge (3-bit). Write "1"
  bits to acknowledge.
  - Bit 0: Keypad scan complete
  - Bit 1: Keypad data register changed
  - Bit 2: Key pressed in mode 1
- 900E000C (R/W): Keypad interrupt mask (3-bit). Set each bit to 1 if
  the corresponding event in \[900E0008\] should cause an interrupt.
- 900E0010-900E002F (R): Keypad data, one halfword per row.
- 900E0030-900E003F (R/W): Keypad GPIOs. Each register is 20 bits, with
  one bit per GPIO. The role of each register is unknown.
- 900E0040 (R/W): Interrupt enable. Bits unknown but seems to be related
  to touchpad. Causes interrupt on touchpad touched.
- 900E0044 (R/W): Interrupt status. Bits unknown. Write 1s to
  acknowledge.
- 900E0048 (R/W): Unknown

## 90120000 - SDRAM Controller

An FTDDR3030.

## 90130000 - Unknown Controller for the LCD Backlight

The OS enables the LCD backlight by writing 255 to 90130018. The
brightness is controlled by 90130014, the OS writes 0 (brightest) to 225
(darkest).

## 90140000 - Power management

A new "Aladdin PMU" unit. Not much known.

- 90140000 (R/?): Reason for waking up from low-power mode.
- 90140050 (R/W): Disable bus access to peripherals. Reads will just
  return the last word read from anywhere in the address range, and
  writes will be ignored.
  - Bit 0: <a href="#900a0000---miscellaneous" class="wikilink" title="#900a0000 - Miscellaneous">#900a0000 - Miscellaneous</a>
  - Bit 9: <a href="#c8010000---triple-des-encryption" class="wikilink"
    title="#C8010000 - Triple DES encryption">#C8010000 - Triple DES
    encryption</a>
  - Bit 10:
    <a href="#cc000000---sha-256-hash-generator" class="wikilink"
    title="#CC000000 - SHA-256 hash generator">#CC000000 - SHA-256 hash
    generator</a>
  - Bit 13: <a href="#90060000---watchdog-timer" class="wikilink"
    title="#90060000 - Watchdog timer">#90060000 - Watchdog timer</a>
    (?)
  - Bit 26: <a href="#90050000---i2c-controller" class="wikilink"
    title="#90050000 - I2C controller">#90050000 - I2C controller</a>
    (?)

## A0000000 - Boot1 ROM again

Mirror of the ROM at 0.

## A4000000 - Internal SRAM

0x40000 bytes SRAM, managed by the controller at ?.

## A8000000 - Magic VRAM

0x25800 bytes SRAM for an LCD framebuffer.

It is wired up in a way that the written data is X-Y swapped and
rotated, so that writing a 320x240 image with (0/0) at the top left
results in a 320x320 image in the right orientation for the LCD. This
means that it can't be used as generic RAM. How this mechanism works
isn't known yet.

## AC000000 - SDIO Controller

Seems to be the same as in the [Zynq 7000](https://docs.amd.com/r/en-US/ug585-zynq-7000-SoC-TRM/SD-Controller-sdio), but not much is known.

Only used in Diagnostics, among other things it is used to pull a factory 
NAND image from an attached SD card. Where this SD card attaches is unknown.

## B0000000 - USB OTG/Host/Device controller (top)

An FOTG210 connected to the top USB port.

## B4000000 - USB OTG/Host/Device controller (bottom)

An FOTG210 connected to the bottom USB port (dock connector).

## B8000000 - SPI NAND controller

An FTSPI020 with a MICRON 1Gb flash at CS 1.

## BC000000 - DMA controller

An FTDMAC020 with main SDRAM and LCD RAM (everything?) connected to
AHB1. The OS uses this to copy the framebuffer into LCD RAM. It is a
derivative of the
[PL080](https://developer.arm.com/documentation/ddi0196/latest/) with
some changes

## C0000000 - LCD controller

A
[PL111](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.ddi0293c/index.html).

## C8010000 - Triple DES encryption

Implements the [Triple DES encryption
algorithm](http://en.wikipedia.org/wiki/Triple_DES).

- C8010000 (R/W): Right half of block
- C8010004 (R/W): Left half of block. Writing this causes the block to
  be encrypted/decrypted.
- C8010008 (R/W): Right 32 bits of key 1
- C801000C (R/W):
  - Bits 0-23: Left 24 bits of key 1
  - Bit 30: Set to 0 to encrypt, 1 to decrypt
- C8010010 (R/W): Right 32 bits of key 2
- C8010014 (R/W): Left 24 bits of key 2
- C8010018 (R/W): Right 32 bits of key 3
- C801001C (R/W): Left 24 bits of key 3

## CC000000 - SHA-256 hash generator

Implements the [SHA-256 hash
algorithm](http://en.wikipedia.org/wiki/SHA_hash_functions), which is
used in cryptographic signatures.

- CC000000 (R): Busy if bit 0 set
- CC000000 (W): Write 0x10 and then 0x0 to initialize. Write 0xA to
  process first block, 0xE to process subsequent blocks
- CC000008 (R/W): Some sort of bus write-allow register? If a bit is
  set, it allows R/W access to the registers of the peripheral, if
  clear, R/O access only. Don't know what it's doing here, but it's here
  anyway.
  - Bit 8: <a href="#CC000000_-_SHA-256_hash_generator" class="wikilink"
    title="#CC000000 - SHA-256 hash generator">#CC000000 - SHA-256 hash
    generator</a>
  - Bit 10: ?
- CC000010-CC00004F (R/W): 512-bit block
- CC000060-CC00007F (R): 256-bit state

## DC000000 - Interrupt controller

See
<a href="/Interrupts" class="wikilink" title="Interrupts">Interrupts</a>.
The controller is a
[PL190](https://developer.arm.com/documentation/ddi0181/e).
