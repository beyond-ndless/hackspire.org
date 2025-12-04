---
title: Memory-mapped I/O ports on Classic
permalink: /Memory-mapped_IO_ports_on_Classic/
parent: Memory-mapped I/O ports
layout: page
---


---
1. TOC
{:toc}
---

## 00000000 - Bootable Boot1 ROM

On the older hardware model (up to hardware revision A), this is 512kB
of NOR flash; on the newer model (from hardware revision C), it is 128kB
of on-chip ROM.

*Note: TI's INT_MMU_Initialize function uses the MMU to map the boot1
ROM (physical address 000xxxxx) at virtual address D40xxxxx, and the
<a href="#A4000000_-_Internal_SRAM" class="wikilink"
title="SRAM">SRAM</a> (physical address A40xxxxx) at virtual address
000xxxxx.*

## 10000000 - SDRAM

## 8FFF0000 - SDRAM control

Write-only (reads just return the last word read from SDRAM). In the OS,
all code that uses these registers is copied to
<a href="#A4000000_-_Internal_SRAM" class="wikilink"
title="SRAM">SRAM</a> and run from there, rather than running it
directly from the SDRAM.

## 90000000 - General Purpose I/O (GPIO)

See <a href="/GPIO_Pins" class="wikilink" title="GPIO Pins">GPIO Pins</a>

## 90010000 - Fast timer

The same interface as 900C0000/900D0000, but runs at the speed of the
APB clock (22.5MHz) rather than 32kHz. See
<a href="#900D0000_-_Second_timer" class="wikilink"
title="Second timer">Second timer</a> for more info.

## 90020000 - Serial UART

Used to communicate with the <a href="/Hardware#RS232" class="wikilink"
title="RS232 serial port">RS232 serial port</a>. The register interface
is like that of the 16550 UART used in PCs.

The classic TI-Nspires has the same clock rate as the APB. The clocks
are used to calculate the divisors for the baud rates. The formula for
working out divisors on the classic TI-Nspires is: divisor = clock_rate
/ (baud_rate \* 16). Similarly, the UART clock rate can be deduced by
calculating: clock_rate = divisor \* (baud_rate \* 16). The current
divisor can be found by setting bit 7 in the Line Control Register then
(\*0x90020000) \| (\*0x90020004)\<\<8.

- 90020000 (R): Receiver Buffer Register (if bit 7 is set in the LCR,
  this reads the LSB of the divisor)
- 90020000 (W): Transmitter Holding Register (if bit 7 is set in the
  LCR, this writes the LSB of the divisor)
- 90020004 (R/W): Interrupt Enable Register (if bit 7 is set in the LCR,
  this is the MSB of the divisor)
- 90020008 (R): Interrupt Identification Register
- 90020008 (W): FIFO Control Register
- 9002000C (R/W): Line Control Register
- 90020010 (R/W): Modem Control Register
- 90020014 (R): Line Status Register
- 90020018 (R): Modem Status Register
- 9002001C (R/W): Scratch Register

## 90030000 - Second UART

Same port structure as 90020000. It is unknown whether this is connected
to anything external, and this doesn't seem to generate interrupts.

## 90060000 - Watchdog timer

Possibly an [ARM
SP805](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.ddi0270b/index.html)
or compatible.

## 90080000 - Unknown

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

- 900A0000 (R): ? 0x01000010.
- 900A0004 (R/W): Set bit 0x20 to enable TI-84+ keypad link port. Other
  bits likely control functions of peripherals as well.
- 900A0008 (W): Write a 2 to cause a hardware reset
- 900A0010 (R/W): Fast timer interrupt status/acknowledge (6-bit). Write
  "1" bits to reset the corresponding interrupt requests.
- 900A0014 (R/W): Fast timer interrupt mask (6-bit). Set bits to 1 if
  the corresponding bits in \[900A0010\] should trigger an IRQ.
- 900A0018 (R/W): Timer 1 interrupt status/acknowledge (6-bit). Write
  "1" bits to reset the corresponding interrupt requests.
- 900A001C (R/W): Timer 1 interrupt mask (6-bit). Set bits to 1 if the
  corresponding bits in \[900A0018\] should trigger an IRQ.
- 900A0020 (R/W): Timer 2 interrupt status/acknowledge (6-bit). Write
  "1" bits to reset the corresponding interrupt requests.
- 900A0024 (R/W): Timer 2 interrupt mask (6-bit). Set bits to 1 if the
  corresponding bits in \[900A0020\] should trigger an IRQ.
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
  according to the value in 900B0000. Write 3A to enter low-power mode;
  this requires various peripherals to be prepared and probably works by
  stopping the clock.
- 900B0010 (R/W): ON interrupt mask (1-bit). 1 if ON interrupt should be
  serviced or 0 if not.
- 900B0014 (R/W): Bit 0 is set if ON interrupt is requested. Bit 1 also
  causes an interrupt, but the cause is unknown (and it is not masked by
  \[900B0010\]) - it is set after writing 4 to 900B000C. Write "1" bits
  to reset the requests.
- 900B0018 (R/W): Disable bus access to peripherals. Reads will just
  return the last word read from anywhere in the address range, and
  writes will be ignored.
  - Bit 0: <a href="#A9000000_-_SPI" class="wikilink"
    title="#A9000000 - SPI">#A9000000 - SPI</a>
  - Bit 2: <a href="#AC000000_-_SD_Host_Controller" class="wikilink"
    title="#AC000000 - SD Host Controller">#AC000000 - SD Host
    Controller</a>
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
  - Bit 7: <a href="#B8000000_-_NAND_Flash" class="wikilink"
    title="#B8000000 - NAND Flash">#B8000000 - NAND Flash</a>
  - Bit 8: <a href="#BC000000_-_Unknown" class="wikilink"
    title="#BC000000 - Unknown">#BC000000 - Unknown</a>
  - Bit 9: <a href="#C8010000_-_Unknown" class="wikilink"
    title="#C8010000 - Unknown">#C8010000 - Unknown</a>
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

Configured as ~105 Hz timer by the OS (only used in the TI-84+
emulator). Same port structure as
<a href="#900D0000_-_Second_timer" class="wikilink"
title="Second timer">Second timer</a>.

## 900D0000 - Second timer

Two timers are located here, but only the first can generate IRQs. IRQ
status/mask is located at \[900A0020\]. Configured as a ~100 Hz timer by
the OS.

- 900D0000 (R/W): Current IRQ timer value (16-bit). Set to 32 by the OS.
  Increases/decreases each \[900D0004\] ticks. The value written to this
  port is saved internally and can be reloaded automatically upon timer
  completion, depending on settings in \[900D0008\].
- 900D0004 (R/W): IRQ timer divider (16-bit). Ticks per count - 1. Set
  to 9 by the OS. Ticks are 32768Hz.
- 900D0008 (R/W): IRQ timer control (5-bit).
  - Bit 4: Set to 1 to freeze timer.
  - Bit 3: Set to 1 for increasing timer, or to 0 for decreasing timer.
  - Bits 2-0: If 0, timer will count to zero and stop. If 1-6, timer
    will complete when it reaches the corresponding timer completion
    value and then reload with original timer value (see \[900D0018\] to
    \[900D002C\]). If 7, timer will never complete and runs infinitely.
- 900D000C (R/W): Current timer value (16-bit). Increases/decreases each
  \[900D0010\] ticks. The value written to this port is saved internally
  and can be reloaded automatically upon timer completion, depending on
  settings in \[900D0014\].
- 900D0010 (R/W): Timer divider (16-bit). Ticks per count - 1. Ticks are
  approximately 32 kHz.
- 900D0014 (R/W): Timer control (5-bit).
  - Bit 4: Set to 1 to freeze timer, or 0 to run timer.
  - Bit 3: Set to 1 for increasing timer, or to 0 for decreasing timer.
  - Bits 2-0: If 0, timer will count to zero and stop. If 1-6, timer
    will complete when it reaches the corresponding timer completion
    value (see \[900D0018\] to \[900D002C\]). If 7, timer will not
    complete and runs infinitely.
- 900D0018 (R/W): Timer completion value 1 (16-bit). If IRQ timer equals
  this value, bit 0 of the interrupt status becomes set.
- 900D001C (R/W): Timer completion value 2 (16-bit). If IRQ timer equals
  this value, bit 1 of the interrupt status becomes set.
- 900D0020 (R/W): Timer completion value 3 (16-bit). If IRQ timer equals
  this value, bit 2 of the interrupt status becomes set.
- 900D0024 (R/W): Timer completion value 4 (16-bit). If IRQ timer equals
  this value, bit 3 of the interrupt status becomes set.
- 900D0028 (R/W): Timer completion value 5 (16-bit). If IRQ timer equals
  this value, bit 4 of the interrupt status becomes set.
- 900D002C (R/W): Timer completion value 6 (16-bit). If IRQ timer equals
  this value, bit 5 of the interrupt status becomes set.
- 900D0030 (R/W): Unknown 6-bit value.

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
- 900F0020 (R/W): LCD contrast/backlight. Valid range for contrast: 0x6B
  to 0x95; normal value is 0x80.

## 90100000 - TI-84 Plus link port

- 90100000 (W): Write to bits 0-1 to hold I/O link lines low or let them
  go high. 1=high, 0=low.
- 90100000 (R): Bits 0-1 hold the status of the I/O link lines. 1=high,
  0=low. Bits 4-5 hold the last value outputted to bits 0-1.
- 90100004 (R/W): Unknown. The OS writes 0x80 here during
  initialization.
- 90100008 (R): Unknown. Bit 0x40 is set if any link lines are low.

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

## A0000000 - On-chip Boot1 ROM

It is 128kB of on-chip Boot1 ROM. The OS's page table has an entry to
allow this to be accessed, but no known code does so. In the newer
hardware model (from hardware revision C), this just mirrors the
0x00000000 bootable Boot1 ROM.

## A4000000 - Internal SRAM

A4000100-A40096FF is used by the OS as the LCD screen buffer. The upper
left corner is the first byte. Each grayscaled pixel is 4-bit long. 1111
is white, 0000 is black.

## A9000000 - SPI

Possibly used to communicate with the wireless cradle.

- A9000008 (R/W): Bits 0-3 = clock speed divider - 1
- A900000C (R/W): Bit 0 = set to 1 to start transfer; bits 16-21 =
  number of bits to transfer - 1
- A9000010 (R): Bit 0 set when transfer complete
- A9000014 (R/W): Unknown
- A9000018 (R/W): Unknown
- A900001C (R/W): Data register low
- A9000020 (R/W): Data register high

## AC000000 - SD Host Controller

See <http://www.sdcard.org/developers/tech/host_controller/simple_spec/>

## B0000000 - USB OTG controller

The OTG controller on all models is a ChipIdea-based dual-role USB
controller. It only supports full speed communications so the PFSC bit
(bit 24) must be set in the PORTSC register when in host mode.
Otherwise, it'll attempt to connect at high speed for devices that
support it and will never succeed in enumerating them.

Appears to be Freescale's USB OTG controller revision 0x42. Its
documentation can be found in the [MCF54455 reference
manual](http://cache.freescale.com/files/32bit/doc/ref_manual/MCF54455RM.pdf)
which happens to use the same revision of the module. The host interface
is based on EHCI 1.0.

- Module identification registers
  - B0000000 (R): ID - Identification register - default 0x0042FA05
  - B0000004 (R): HWGENERAL - General hardware parameters - default
    0x000007C5
  - B0000008 (R): HWHOST - Host hardware parameters - default 0x10020001
  - B000000C (R): HWDEVICE - Device hardware parameters - default
    0x00000009
  - B0000010 (R): HWTXBUF - TX buffer hardware parameters - default
    0x80040604
  - B0000014 (R): HWRXBUF - RX buffer hardware parameters - default
    0x00000404
- Capability registers
  - B0000100 (R): CAPLENGTH - Capability registers length - default
    0x0100
  - B0000102 (R): HCIVERSION - Host controller interface version -
    default 0x0040
  - B0000104 (R): HCSPARAMS - Host controller structural parameters -
    default 0x00010011
  - B0000108 (R): HCCPARAMS - Host controller capability parameters -
    default 0x00000006
  - B0000120 (R): DCIVERSION - Device controller interface version -
    default 0x00000001
  - B0000124 (R): DCCPARAMS - Device controller capability parameters -
    default 0x00000184 (host-capable, device-capable, 4 endpoints)
- Operational registers
  - B0000140 (R/W): USBCMD - USB command - default 0x00080000 (in device
    mode)
  - B0000144 (R/W): USBSTS - USB status - default 0x00000080
  - B0000148 (R/W): USBINTR - USB interrupt enable - default 0x00000000
  - B000014C (R/W): FRINDEX - USB frame index - default 0x00000000
  - B0000150 (R): CTRLDSSEGMENT - 4G segment selector (always 0?)
  - B0000154 (R/W): (in host mode) PERIODICLISTBASE - Frame list base
    address
  - B0000154 (R/W): (in device mode) DEVICEADDR - Device address
  - B0000158 (R/W): (in host mode) ASYNCLISTADDR - Current asynchronous
    list address
  - B0000158 (R/W): (in device mode) EPLISTADDR - Address at endpoint
    list
  - B000015C (R/W): TTCTRL - TT status and control - default 0x00000000
  - B0000160 (R/W): BURSTSIZE - Programmable DMA burst size - default
    0x00000404
  - B0000164 (R/W): TXFILLTUNING - Host TT Xmit pre-buffer packet
    tuning - default 0x00000000
  - B0000170 (R/W): ULPI_VIEWPORT - ULPI register access - default
    0x00000000
  - B0000180 (R): CONFIGFLAG - Configured flag register (always 1?)
  - B0000184 (R/W): PORTSC - Port status and control - default
    0xEC000004
  - B00001A4 (R/W): OTGSC - On-The-Go status and control - default
    0x00000120
  - B00001A8 (R/W): USBMODE - USB device mode - default 0x00000000
  - B00001AC (R/W): ENDPOINTSETUPSTAT - Endpoint setup status - default
    0x00000000
  - B00001B0 (R/W): ENDPTPRIME - Endpoint initialization - default
    0x00000000
  - B00001B4 (R/W): ENDPTFLUSH - Endpoint de-initialize - default
    0x00000000
  - B00001B8 (R): ENDPTSTATUS - Endpoint status - default 0x00000000
  - B00001BC (R/W): ENDPTCOMPLETE - Endpoint complete - default
    0x00000000
  - B00001C0 (R/W): ENDPTCTRL0 - Endpoint control 0 - default 0x00800080
  - B00001C4 (R/W): ENDPTCTRL1 - Endpoint control 1 - default 0x00000000
  - B00001C8 (R/W): ENDPTCTRL2 - Endpoint control 2 - default 0x00000000
  - B00001CC (R/W): ENDPTCTRL3 - Endpoint control 3 - default 0x00000000

### Role switching

During role switching some GPIO output registers are modified.

- GPIO5:
  - Active low.
  - Controls VBUS/pull-up (drives VBUS to 5v for host mode)

<!-- -->

- USB-B: GPIO6
  - Active low.
  - Probably controls charging from USB

## B4000000 - USB HOST controller

Same port structure as B0000000.

## B8000000 - NAND Flash

Derived from nspire_emu.

- B8000000 (R/W): Unknown.
- B8000004 (R/W): Is NAND writable?
- B8000008 (R): Busy status. Bit 0 not set means ready.
- B8000008 (W): Write 1 to begin operation
- B800000C (R/W): Operation
  - Bit 0-7: CMD1 - command
  - Bit 8-10: CMD1 - number of address bytes
  - Bit 11: (0) Read from device, (1) Write to device
  - Bit 12-19: CMD2 – command
  - Bit 20: CMD2 – is valid
  - Bit 21: Unknown (is address phase perhaps?)
  - Bit 22: (0) No data phase, (1) data phase
- B8000010 (R/W): 1st address byte
- B8000014 (R/W): 2nd address byte
- B8000018 (R/W): 3rd address byte
- B800001C (R/W): 4th address byte
- B8000024 (R/W): DMA buffer size (basically number of bytes to
  read/write)
- B8000028 (R/W): DMA address
- B800002C (R/W): AHB speed / 2500000
- B8000030 (R/W): APB speed / 250000
- B8000034 (R): Read NAND status - sends standard READ STATUS command to
  NAND
  - Bit 0: (0) successful, (1) error
  - Bit 1: (0) successful, (1) error
  - Bit 5: (0) busy, (1) ready
  - Bit 6: (0) busy, (1) ready
  - Bit 7: (0) protected, (1) not protected
- B8000040 (W): Unknown. 1 gets written here by boot1 before an
  operation (reset ECC generator perhaps?)
- B8000044 (R): Calculate ECC. First ECC byte = (reg \>\> 6) & 0xFF,
  second ECC byte = (reg \>\> 14) & 0xFF, third ECC byte = ((reg
  \>\> 22) \| (reg \<\< 2)) & 0xFF
- B8000054 (W): Unknown. Boot1 writes here during init

## BC000000 - Unknown

## C0000000 - LCD controller

Probably an [ARM PrimeCell
PL110](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.ddi0161e/index.html)
or something compatible on the classic TI-Nspire.

- C0000010 (R/W): Frame Base Address. Holds the address to read pixel
  data from. Set to A4000100
- C0000018 (R/W): Interrupt mask.
- C000001C (R/W): LCD Control
  - Bit 0: LCD controller enable.
  - Bits 1-3: LCD bits per pixel.
    - 000 = 1 bpp
    - 001 = 2 bpp
    - 010 = 4 bpp (default setting of OS)
    - 011 = 8 bpp
    - 100 = 16 bpp (holds the literal 16-bit palette value instead of a
      palette index). Only bits 1-4 are taken into account. Values are
      interpreted as they would be in 4-bits mode : 0 means white,
      0b1111 means black.
    - 101 = 24 bpp (not applicable to TI-Nspire's STN LCD)
    - 110,111 = reserved
  - Bit 4: Set to 1 if STN LCD is monochrome, or 0 if color. Should be
    set to 1 on TI-Nspire.
  - Bit 5: Set to 1 if LCD is TFT, or 0 if STN. Should be set to 0 on
    TI-Nspire.
  - Bit 6: Set to 1 if monochrome STN LCD has a 8-bit interface, or 0 if
    4-bit. Should be set to 1 on TI-Nspire.
  - Bit 7: Set to 1 if LCD is dual panel STN, or 0 if single-panel.
    Should be set to 0 on TI-Nspire.
  - Bit 8: Set to 1 if the palettes are read as BGR, or 0 if RGB. Since
    monochrome displays use only the R color, it is possible to store a
    secondary palette in the B color and switch instantly by flipping
    this bit.
  - Bit 9: Set to 1 if the bytes in each word are to be read as
    big-endian, or 0 if little-endian. Set to 0 by the OS.
  - Bit 10: Set to 1 if the pixels within each byte are to be read as
    big-endian, or 0 if little-endian. Only affects 1,2,4 bpp modes. Set
    to 1 by the OS.
  - Bit 11: LCD power enable.
  - Bits 12-13: Vertical compare interrupt region.
    - 00 = start of vertical synchronization
    - 01 = start of back porch
    - 10 = start of active video
    - 11 = start of front porch
  - Bits 14-15: Reserved.
  - Bit 16: LCD DMA FIFO Watermark Level.
  - Bits 17-31: Reserved.
- C0000020 (R): Raw interrupt status.
  - Bit 1: FIFO Underflow
  - Bit 2: LCD next address base update. Signifies that a new Frame Base
    Address value can be loaded for double-buffering.
  - Bit 3: Vertical compare. Set when one of four vertical regions
    (specified by the LCD Control register) is reached.
  - Bit 4: AHB Master bus error.
- C0000024 (R): Masked interrupt status.
- C0000028 (W): Interrupt clear. Write a 1 to each bit to clear.
- C000002C (R): Address that is currently being read from (approximate)
- C0000200-C00003FF (R/W): 256-color palette (each entry is a half-word,
  but this must be written to an entire word at a time)
  - Bits 0-4: Red palette data (bits 1-4 are the grayscale data on the
    Nspire screen)
  - Bits 5-9: Green palette data (unused on Nspire)
  - Bits 10-14: Blue palette data (bits 11-14 are the grayscale data on
    the Nspire screen if BGR mode is set)
  - Bit 15: Intensity (unused on Nspire)

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
The following documentation is for the classic TI-Nspire.

Registers that operate on sets of IRQs are bitmaps, with bit 0
corresponding to IRQ 0, and so on.

- DC000000 (R): Masked IRQ status (always equal to \[DC000004\] &
  \[DC000008\])
- DC000004 (R): Raw interrupt status or sticky interrupt status,
  depending on bitfield in DC000204
- DC000004 (W): Resets a set of sticky interrupts
- DC000008 (R): Current set of enabled IRQs
- DC000008 (W): Enable a set of IRQs
- DC00000C (R): Mirror of DC000008
- DC00000C (W): Disable a set of IRQs
- DC000020 (R): Reads current IRQ number (no side effects)
- DC000024 (R): Reads current IRQ number, copies the value in DC00002C
  to DC000028, writes the priority of the current IRQ to DC00002C
- DC000028 (R): Reading this register will reset the IRQ request. The
  value read is whatever was last copied by reading DC000024
- DC00002C (R/W): 4-bit value. IRQs with priority greater than or equal
  to this value will not be requested
- DC000100 (R): Masked FIQ status (always equal to \[DC000104\] &
  \[DC000108\])
- DC000104 (R): Raw interrupt status or sticky interrupt status,
  depending on bitfield in DC000204
- DC000104 (W): Resets a set of sticky interrupts
- DC000108 (R): Current set of enabled FIQs
- DC000108 (W): Enable a set of FIQs
- DC00010C (R): Mirror of DC000108
- DC00010C (W): Disable a set of FIQs
- DC000120 (R): Reads current FIQ number (no side effects)
- DC000124 (R): Reads current FIQ number, copies the value in DC00012C
  to DC000128, writes the priority of the current IRQ to DC00012C
- DC000128 (R): Reading this register will reset the FIQ request. The
  value read is whatever was last copied by reading DC000124
- DC00012C (R/W): 4-bit value. FIQs with priority greater than or equal
  to this value will not be requested
- DC000200 (R/W): Bits that are 0 will invert the corresponding raw
  interrupt status bit. Typically this register should hold 0xFFFFFFFF.
- DC000204 (R/W): Bits that are 1 will cause the corresponding bit in
  DC000004 and DC000104 to read the sticky interrupt status. Bits that
  are 0 will cause the corresponding bit to read the raw interrupt
  status.
- DC000208 (R/W): ?
- DC000300-DC00037F (W): IRQ priority (0-7). One register per IRQ. Lower
  values indicate higher priority.