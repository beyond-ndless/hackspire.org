---
title: Memory-mapped I/O ports on CX
permalink: /Memory-mapped_IO_ports_on_CX/
parent: Memory-mapped I/O ports
layout: page
---

---
1. TOC
{:toc}
---

## 00000000 - Boot1 ROM

128kB of on-chip ROM.

## 10000000 - SDRAM

32 MiB SDRAM on CM or 64 MiB on CX. Managed by 0x8FFF0000.

## 8FFF0000 - SDRAM controller

A DMC-340 r1p0.

## 8FFF1000 - NAND controller

A PL351 r1p2.

## 90000000 - General Purpose I/O (GPIO)

See <a href="/GPIO_Pins" class="wikilink" title="GPIO Pins">GPIO Pins</a>

## 90010000 - Fast timer

The same interface as 900C0000/900D0000, but runs at the speed of the
APB clock (22.5MHz) rather than 32kHz.
The speed of the timers seems to be configurable, see
<a href="/timers" class="wikilink" title="timers">timers</a>.
An [SP804](https://developer.arm.com/documentation/ddi0271/d/).

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

A PL022 for communicating with the LCD panel controller, which is
probably an ILI9341 or ILI9340. Used on CX HW-W+ only.

## 90050000 - I2C controller

The Touchpad on the CX is accessed through this controller. See
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

## 90090000 - Real-Time Clock (RTC)

Similar to the [ARM PrimeCell
PL031](http://infocenter.arm.com/help/topic/com.arm.doc.ddi0224b/index.html),
but interrupt registers are different.

At least on HW-AA it's a standard PL031 with no registers changed.

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

- 900A0000 (R): ? 0x101
- 900A0004 (R/W): Set bit 0x20 to enable TI-84+ keypad link port. Other
  bits likely control functions of peripherals as well.
- 900A0008 (W): Write a 2 to cause a hardware reset
- 900A0028-900A002C (R): These registers together give a 64-bit number
  (28 is low, 2C is high) which comprises 56 data bits and 8 parity
  checking bits, allowing any single-bit error in it to be detected and
  corrected.
  - Parity bit 0: Check of all data bits
  - Parity bits 1, 2, 4, 8, 16, and 32: Checks of the data bits whose
    positions, expressed in binary, have that respective bit set.
  - Data bits 3, 5-7, 9-15, 17-31, and 33-55: Serial number (middle part
    of the calculator's Product ID)
  - Data bits 56-57: Unknown
  - Data bits 58-62: "ASIC user flags"; must match the 80E0 field in an
    OS image. 01 = CAS, 00 = non-CAS, 03 = CM CAS, 02 = CM non-CAS.
  - Parity bit 63: Check of parity bits 1, 2, 4, 8, 16, and 32.
- 900A0F04 (R/W): Unknown; Boot1 sets this to 0x1D

## 900B0000 - Power management

- 900B0000 (R/W):
  <a href="/Clock_speed" class="wikilink" title="Clock speed">Clock
  speed</a> load value
- 900B0004 (R/W): 25-bit mask of which events may wake the hardware up
  from low-power mode.
  - Bit 10: Unknown, probably the
    <a href="#90100000_-_TI-84_Plus_link_port" class="wikilink"
    title="TI-84 Plus link port">TI-84 Plus link port</a>
  - Bit 12: <a href="#90090000_-_Real-Time_Clock_(RTC)" class="wikilink"
    title="RTC">RTC</a> interrupt
  - Bit 13: Unknown, probably ON key or USB activity
  - Bit 17: Battery door open/close?
  - Bit 23: Keypad remove/replace?
- 900B0008 (R/W): Reason for waking up from low-power mode. Write "1"
  bits to acknowledge.
- 900B000C (R/W): Clock speed control. Write 4 to set the clock speed
  according to the value in 900B0000. If interrupts are disabled the new
  clock speed will only become effective after exiting the program.
  Write 3A to enter low-power mode; this requires various peripherals to
  be prepared and probably works by stopping the clock.
- 900B0010 (R/W): ON interrupt mask (1-bit). 1 if ON interrupt should be
  serviced or 0 if not.
- 900B0014 (R/W): Bit 0 is set if ON interrupt is requested. Bit 1 also
  causes an interrupt, but the cause is unknown (and it is not masked by
  \[900B0010\]) - it is set after writing 4 to 900B000C. Write "1" bits
  to reset the requests.
- 900B0018 (R/W): Disable bus access to peripherals. Reads will just
  return the last word read from anywhere in the address range, and
  writes will be ignored.
  - Bit 4:
    <a href="#C4000000_-_Analog-to-Digital_Converter_(ADC)" class="wikilink"
    title="#C4000000 - Analog-to-Digital Converter (ADC)">#C4000000 -
    Analog-to-Digital Converter (ADC)</a>
  - Bit 5: <a href="#B0000000_-_USB_OTG_controller" class="wikilink"
    title="#B0000000 - USB OTG controller">#B0000000 - USB OTG
    controller</a>
  - Bit 6: <a href="#B4000000_-_USB_HOST_controller" class="wikilink"
    title="#B4000000 - USB HOST controller">#B4000000 - USB HOST
    controller</a>
  - Bit 7: <a href="#B8010000_-_SRAM_Controller" class="wikilink"
    title="#B8010000 - SRAM Controller">#B8010000 - SRAM Controller</a>
  - Bit 10:
    <a href="#CC000000_-_SHA-256_hash_generator" class="wikilink"
    title="#CC000000 - SHA-256 hash generator">#CC000000 - SHA-256 hash
    generator</a>
  - Bit 11: <a href="#900C0000_-_First_timer" class="wikilink"
    title="#900C0000 - First timer">#900C0000 - First timer</a>
  - Bit 12: <a href="#900D0000_-_Second_timer" class="wikilink"
    title="#900D0000 - Second timer">#900D0000 - Second timer</a>
  - Bit 13: <a href="#90060000_-_Watchdog_timer" class="wikilink"
    title="#90060000 - Watchdog timer">#90060000 - Watchdog timer</a>
  - Bit 17: <a href="#90020000_-_Serial_UART" class="wikilink"
    title="#90020000 - Serial UART">#90020000 - Serial UART</a>
  - Bit 22: <a href="#90110000_-_LED" class="wikilink"
    title="#90110000_-_LED">#90110000_-_LED</a>?
- 900B0020 (R/W): ? - Possibly another peripheral bus access disable
  register.
- 900B0024 (R): Reads current clock speed value (see 900B0000 for
  details)
- 900B0028 (R): Bit 4 (0x10) clear when ON key pressed

## 900C0000 - First timer

An [SP804](https://developer.arm.com/documentation/ddi0271/d/).
The speed of the timers seems to be configurable, see
<a href="/timers" class="wikilink" title="timers">timers</a>.

## 900D0000 - Second timer

An [SP804](https://developer.arm.com/documentation/ddi0271/d/).
The speed of the timers seems to be configurable, see
<a href="/timers" class="wikilink" title="timers">timers</a>.

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

## 900F0000 - HDQ/1-Wire and LCD contrast

The HDQ/1-Wire registers resemble those on the TI OMAP processors, and
are possibly used to communicate with the wireless cradle. There is no
conceivable reason for the LCD contrast register to be part of the same
module, but here it is. :-(

- 900F0004 (W): Transmitted data
- 900F0008 (R): Received data
- 900F000C (R/W): Control/status
- 900F0010 (R): Interrupt status (automatically acknowledged when read)
- 900F0020 (R/W): LCD contrast/backlight. Valid range for contrast:
  0x11a to 0x1ce; normal value is 0x174. However, it can range from
  0x100 (backlight off) to about 0x1d0 (about max brightness).

## 90110000 - LED

- 90110B00 (R/W): Control register
  - Bit 0: Set this bit to enable green light blink data. If green blink
    data iteration is not on, the green light state is read from bit 0
    of green blink data.
  - Bit 1: Set this bit and bit 6 to enable green blink data iteration.
  - Bit 2: Set this bit to force green light off. Overrides bit 4.
  - Bit 3: Set this bit to force red light off. Overrides bits 5 and 13.
  - Bit 4: Set this bit to force green light on.
  - Bit 5: Set this bit to force red light on.
  - Bit 6: See this bit and bit 1 to enable green blink data iteration.
    Reset before modifying green blink data or delay.
  - Bit 9: Set this bit to enable red light blink data. If red blink
    data iteration is not on, the red light state is read from bit 0 of
    red blink data.
  - Bit 10: Set this bit and bit 12 to enable red blink data iteration.
  - Bit 12: Set this bit and bit 10 to enable red blink data iteration.
    Reset before modifying red blink data or delay.
  - Bit 13: Forces red light on if bit 4 is 0, or red light off if bit 4
    is 1. (?)
- 90110B04 (R/W): Green blink data. 32 bits of on and off state,
  represented by 1 and 0. Iteration is done from bit 31 to bit 0
  repeatedly.
- 90110B08 (R/W): Green blink delay (negative). OS sets this to -2048.
- 90110B0C (R/W): Red blink data. 32 bits of on and off state,
  represented by 1 and 0. Iteration is done from bit 31 to bit 0
  repeatedly.
- 90110B10 (R/W): Red blink delay (negative). OS sets this to -2048.

Note: If red and green lights are on at the same time, the color becomes
yellow.

## A4000000 - Internal SRAM

0x20000 bytes SRAM, managed by the controller at 0xB8000000.

## B0000000 - USB OTG controller

The OTG controller on all models is a ChipIdea-based dual-role USB
controller. It only supports full speed communications so the PFSC bit
(bit 24) must be set in the PORTSC register when in host mode.
Otherwise, it'll attempt to connect at high speed for devices that
support it and will never succeed in enumerating them.

Documentation can be found in the [IMX233 reference
manual](http://www.freescale.com/files/dsp/doc/ref_manual/IMX23RM.pdf).
The host interface is, again, based on EHCI; but the register defaults
are different. The addresses have been adjusted from the ones contained
in the IMX233 reference manual.

- Module identification registers
  - B0000000: HW_USBCTRL_ID - default 0xE241FA05
  - B0000004: HW_USBCTRL_HWGENERAL - default 0x00000015
  - B0000008: HW_USBCTRL_HWHOST - default 0x10020001
  - B000000C: HW_USBCTRL_HWDEVICE - default 0x0000000B
  - B0000010: HW_USBCTRL_HWTXBUF - default 0x40060910
  - B0000014: HW_USBCTRL_HWRXBUF - default 0x00000710
- Capability registers
  - B0000100: HW_USBCTRL_CAPLENGTH - default 0x01000040
  - B0000104: HW_USBCTRL_HCSPARAMS - default 0x00010011
  - B0000108: HW_USBCTRL_HCCPARAMS - default 0x00000006
  - B0000120: HW_USBCTRL_DCIVERSION - default 0x00000001
  - B0000124: HW_USBCTRL_DCCPARAMS - default 0x00000185 (host-capable,
    device-capable, 5 endpoints)
- Operational registers
  - B0000140: HW_USBCTRL_USBCMD - default 0x00080B00 in host mode,
    0x00080000 in device mode
  - B0000144: HW_USBCTRL_USBSTS - default 0x00001000 in host mode,
    0x00000000 in device mode
  - B0000148: HW_USBCTRL_USBINTR - default 0x00000000
  - B000014C: HW_USBCTRL_FRINDEX - default 0x00000000
  - B0000154: (in host mode) HW_USBCTRL_PERIODICLISTBASE - default
    0x00000000
  - B0000154: (in device mode) HW_USBCTRL_DEVICEADDR - default
    0x00000000
  - B0000158: (in host mode) HW_USBCTRL_ASYNCLISTADDR - default
    0x00000000
  - B0000158: (in device mode) HW_USBCTRL_ENDPOINTLISTADDR - default
    0x00000000
  - B000015C: HW_USBCTRL_TTCTRL - default 0x00000000
  - B0000160: HW_USBCTRL_BURSTSIZE - default 0x00001010
  - B0000164: HW_USBCTRL_TXFILLTUNING - default 0x000000000
  - B000016C: HW_USBCTRL_IC_USB - default 0x00000000
  - B0000170: HW_USBCTRL_ULPI - default 0x00000000
  - B0000178: HW_USBCTRL_ENDPTNAK - default 0x00000000
  - B000017C: HW_USBCTRL_ENDPTNAKEN - default 0x00000000
  - B0000184: HW_USBCTRL_PORTSC1 - default 0x10000000
  - B00001A4: HW_USBCTRL_OTGSC - default 0x00000120
  - B00001A8: HW_USBCTRL_USBMODE - default 0x00000000
  - B00001AC: HW_USBCTRL_ENDPTSETUPSTAT - default 0x00000000
  - B00001B0: HW_USBCTRL_ENDPTPRIME - default 0x00000000
  - B00001B4: HW_USBCTRL_ENDPTFLUSH - default 0x00000000
  - B00001B8: HW_USBCTRL_ENDPTSTAT - default 0x00000000
  - B00001BC: HW_USBCTRL_ENDPTCOMPLETE - default 0x00000000
  - B00001C0: HW_USBCTRL_ENDPTCTRL0 - default 0x00100010
  - B00001C4: HW_USBCTRL_ENDPTCTRL1 - default 0x00000000
  - B00001C8: HW_USBCTRL_ENDPTCTRL2 - default 0x00000000
  - B00001CC: HW_USBCTRL_ENDPTCTRL3 - default 0x00000000
  - B00001D0: HW_USBCTRL_ENDPTCTRL4 - default 0x00000000

### Role switching

During role switching some GPIO output registers are modified.

- GPIO2:
  - Active low.
  - Controls VBUS/pull-up (drives VBUS to 5v for host mode)

<!-- -->

- USB-B: GPIO6
  - Active low.
  - Probably controls charging from USB

## B4000000 - USB HOST controller

Same port structure as B0000000.

## B8001000 - SRAM Controller

A
[PL352](http://infocenter.arm.com/help/topic/com.arm.doc.ddi0380g/DDI0380G_smc_pl350_series_r2p1_trm.pdf)
r1p2.

## C0000000 - LCD controller

A
[PL111](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.ddi0293c/index.html).

## C4000000 - Analog-to-Digital Converter (ADC)

Used to check various voltages. Channels 1 ("VBATT"), 2 ("VSYS"), and 4
("B12") are used to check the battery status; channel 3 is used to
determine which keypad is in use.

- C4000000 (R): Masked interrupt status (4 bits per channel: bits 0-3
  are for channel 0, etc)
- C4000004 (R/W): Raw interrupt status, write 1 bits to acknowledge
- C4000008 (R/W): Interrupt enable register
- C4000100-C40001DF: Per-channel registers (channel 0 starts at
  C4000100, channel 1 at C4000120, etc.)
  - +00 (R/W): Set bit 0 to start measurement; interrupt status bits 0
    and 1 will be set when complete and the value will be stored in +10
    register. Other commands do exist, including some that write to
    memory.
  - +04 (R/W): Unknown (28 bits)
  - +08 (R/W): Number of halfwords to write (25 bits)
  - +0C (R/W): Base address (word-aligned)
  - +10 (R): Read measured voltage. Scale for channels 1 and 2 is 155
    units = 1 volt; scale for other channels is 310 units = 1 volt
  - +14 (R/W): Speed (10 bits, set to AHB clock speed / 40000)

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
[PL190](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.ddi0181e/index.html).