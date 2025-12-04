---
title: GPIO Pins
permalink: /GPIO_Pins/
parent: Hardware
layout: page
---

This section describes the GPIO interface and known GPIO pins.

## Memory Mapped I/O registers

The GPIO registers are separated into sections:

- Section 0: 90000000-9000003F
- Section 1: 90000040-9000007F
- Section 2: 90000080-900000BF
- Section 3: 900000C0-900000FF
- Section 5: 90000140-9000017F

Each register is a word, and only bits 0-7 of each are used. There can
be up to 40 devices accessed by this setup, each known as a GPIO. Each
GPIO is defined by one of the 8 bits in one of the sections. The number
of the GPIO is the section number times 8 plus the bit number. Each GPIO
has a status bit and can cause interrupts.

### Section registers

The following addresses are offsets from the beginning of the GPIO
section:

- +00 (R): Masked interrupt status (\[+04\] & \[+08\])
- +04 (R): Reads raw interrupt status (directly dependent on the GPIO
  input) or sticky interrupt status (becomes set when GPIO status
  changes) depending on bit in \[+20\]
- +04 (W): Write 1 to the bit to reset the sticky interrupt status.
- +08 (R): Reads current interrupt mask bit.
- +08 (W): Write 1 to the bit to enable interrupt (set mask bit to 1)
- +0C (W): Write 1 to the bit to disable interrupt (set mask bit to 0)

The following addresses should be changed with a read-modify-write
pattern so as to not disturb other bits in the register.

- +10 (R/W): Direction (set bit to 0 for output, 1 for input)
- +14 (R/W): GPIO output bit
- +18 (R): Reads GPIO input bit.
- +1C (R/W): Setting bit to 1 will invert raw interrupt status (?)
- +20 (R/W): Setting bit to 1 will use sticky interrupt status in
  \[+04\], or setting to 0 will use raw interrupt status.
- +24 (R/W): (?)

## Currently known pins

|           | 7   | 6   | 5   | 4   | 3   | 2   | 1   | 0   |
|-----------|-----|-----|-----|-----|-----|-----|-----|-----|
| Section 0 | 7   | 6   | 5   | 4   | 3   | 2   | 1   | 0   |
| Section 1 | 15  | 14  | 13  | 12  | 11  | 10  | 9   | 8   |
| Section 2 | 23  | 22  | 21  | 20  | 19  | 18  | 17  | 16  |
| Section 3 | 31  | 30  | 29  | 28  | 27  | 26  | 25  | 24  |

Bit/section number to GPIO pin

|  | Classic | CX |
|----|----|----|
| 0 |  |  |
| 1 | I2C clock - for communicating with the Touchpad (see <a href="/Keypads#Touchpad_I²C" class="wikilink"
title="Keypads#Touchpad I²C">Keypads#Touchpad I²C</a> for details) |  |
| 2 | Input bit is 0 if battery door open, 1 if closed | Active low - controls USB VBUS 5V output |
| 3 | I2C data - for communicating with the Touchpad (see <a href="/Keypads#Touchpad_I²C" class="wikilink"
title="Keypads#Touchpad I²C">Keypads#Touchpad I²C</a> for details) |  |
| 4 |  |  |
| 5 | Active low - controls USB VBUS 5V output |  |
| 6 | Active low - probably controls charging from USB. |  |
| 7 |  |  |
| 8 | Input bit is 0 if Reset Button is pressed |  |
| 9 |  |  |
| 10 |  |  |
| 11 |  |  |
| 12 |  |  |
| 13 |  |  |
| 14 |  |  |
| 15 |  |  |
| 16 |  |  |
| 17 |  |  |
| 18 |  |  |
| 19 | Input is 0 if WLAN Cradle attached |  |
| 20 |  | High if USB micro B plug attached |
| 21 |  |  |
| 22 |  |  |
| 23 | Called LCD_OFF in diags mode |  |
| 24 | Input bit is 1 if a keypad is not plugged in |  |
| 25 |  |  |
| 26 |  |  |
| 27 |  |  |
| 28 |  |  |
| 29 |  |  |
| 30 |  |  |
| 31 |  |  |