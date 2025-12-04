---
title: Keypads
permalink: /Keypads/
parent: Hardware
layout: page
---

## Key maps

Each bit in the 900E0010-900E001F registers represents a key. Only bits
0 to 10 are used in each halfword. The mapping depends on the currently
used keypad.

**Clickpad keypad map:**

If the bit is cleared, the key is being pressed.

<div class="compact_table"></div>

| offset | bit 0 | bit 1 | bit 2 | bit 3 | bit 4 | bit 5 | bit 6 | bit 7 | bit 8 | bit 9 | bit 10 |
|--------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|--------|
| 0010   | ret   | enter | space | (-)   | Z     | .     | Y     | 0     | X     | on    | theta  |
| 0012   | ,     | \+    | W     | 3     | V     | 2     | U     | 1     | T     | e^x   | pi     |
| 0014   | ?     | \-    | S     | 6     | R     | 5     | Q     | 4     | P     | 10^x  | EE     |
| 0016   | :     | \*    | O     | 9     | N     | 8     | M     | 7     | L     | x^2   | i      |
| 0018   | "     | /     | K     | tan   | J     | cos   | I     | sin   | H     | ^     | \>     |
| 001A   | '     | cat   | G     | )     | F     | (     | E     | var   | D     | shift | \<     |
| 001C   | flag  | click | C     | home  | B     | menu  | A     | esc   | \|    | tab   |        |
| 001E   | up    | u+r   | right | r+d   | down  | d+l   | left  | l+u   | del   | ctrl  | =      |

**TI-84+ keypad map:**

If the bit is cleared, the key is being pressed.

<div class="compact_table"></div>

| offset | bit 0 | bit 1 | bit 2 | bit 3 | bit 4 | bit 5 | bit 6 | bit 7 | bit 8 | bit 9 | bit 10 |
|--------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|--------|
| 0010   | down  | left  | right | up    |       |       |       |       |       |       |        |
| 0012   | enter | \+    | \-    | \*    | /     | ^     | clear |       |       |       |        |
| 0014   | (-)   | 3     | 6     | 9     | )     | tan   | vars  |       |       |       |        |
| 0016   | .     | 2     | 5     | 8     | (     | cos   | prgm  | stat  |       |       |        |
| 0018   | 0     | 1     | 4     | 7     | ,     | sin   | apps  | X     |       |       |        |
| 001A   | on    | sto   | ln    | log   | x^2   | x^-1  | math  | alpha |       |       |        |
| 001C   | graph | trace | zoom  | wind  | y=    | 2nd   | mode  | del   |       |       |        |
| 001E   |       |       |       |       |       |       |       |       |       |       |        |

**Touchpad/CX/CM/CX II keypad map:**

If the bit is set, the key is being pressed.

The On/Home button is not visible here (and can not generate a keypad
interrupt) but accessible on the Touchpad/CX/CM at bit 4 of 0x900B0028
and on the CX II at bit 8 of 0x90140810. Both are inverted, i.e. 0 =
pressed, 1 = not pressed. libndls' `on_key_pressed()` function handles
that for all models.

<div class="compact_table"></div>

| offset | bit 0 | bit 1 | bit 2 | bit 3 | bit 4 | bit 5 | bit 6 | bit 7 | bit 8 | bit 9 | bit 10  |
|--------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|---------|
| 0010   | ret   | enter |       | (-)   | space | Z     | Y     | 0     | ?!    |       |         |
| 0012   | X     | W     | V     | 3     | U     | T     | S     | 1     | pi    | trig  | 10^x    |
| 0014   | R     | Q     | P     | 6     | O     | N     | M     | 4     | EE    | x^2   |         |
| 0016   | L     | K     | J     | 9     | I     | H     | G     | 7     | /     | e^x   |         |
| 0018   | F     | E     | D     |       | C     | B     | A     | =     | \*    | ^     |         |
| 001A   |       | var   | \-    | )     | .     | (     | 5     | cat   | frac  | del   | scratch |
| 001C   | flag  |       | \+    | doc   | 2     | menu  | 8     | esc   |       | tab   |         |
| 001E   |       |       |       |       |       |       |       |       | shift | ctrl  | ,       |

## Touchpad I²C

Communication with the actual touchpad is done with the
[I²C](http://en.wikipedia.org/wiki/I%C2%B2C) protocol.

On the original Touchpad models, the I2C protocol is bitbanged through
the GPIO: GPIO 1 is the Serial Clock (SCL) and GPIO 3 is the Serial Data
Line (SDA). On the CX and CX II models, there is a dedicated
<a href="/Memory-mapped_I/O_ports#90050000_-_I2C_controller"
class="wikilink" title="I2C controller">I2C controller</a> for accessing
it.

Calculators before the CX II HW rev. AK use the Synaptics protocol, but
later versions (indicated by bit 0 of the
<a href="/NAND_Memory_Layout#Manuf_Format" class="wikilink"
title="HW flags">HW flags</a>) use a custom touchpad and protocol called
"CapTIvate".

### Synaptics protocol

The touchpad seems to be a
[TM-603](http://read.pudn.com/downloads64/doc/fileformat/229278/510-501-rev4-tm603.pdf).
[The linux
driver](http://lxr.free-electrons.com/source/drivers/input/mouse/synaptics_i2c.c)
is a good source for documentation.

it responds to messages with address 0x20. A write message consists of
the first port to write to, followed by the values to write: the port
number auto-increments, so a message of `00 de ad` writes `de` to port
`00` and `ad` to port `01`. A read message reads from whatever port was
set by the previous write message, so it generally has to be preceded by
an empty write to set the port.

A list of some of the touchpad's ports follows. All 16-bit numbers are
big-endian.

- Any page:
  - FF: Page number
- Page 04 (default):
  - 00: Contact (usually is 01 if proximity \>= 0x2F, 00 otherwise)
  - 01: Proximity
  - 02-03: X position
  - 04-05: Y position
  - 06: relative X
  - 07: relative Y (both refresh on irq)
  - 0A: 1 if touchpad pressed down, 0 if not
  - 0B: Status (reading this clears the low bits)
    - Bit 0 (0x01): Set when proximity is nonzero
    - Bit 1 (0x02): Set when velocity bytes are nonzero
    - Bit 2 (0x04): Set when unknown byte at 08 is nonzero
    - Bit 3 (0x08): Set when pressed byte has changed
    - Bit 6 (0x40): Set when the touchpad is configured
    - Bit 7 (0x80): Set when an error occured
  - E4-E7: Firmware version
- Page 10:
  - 04-05: Maximum X coordinate (0x0918)
  - 06-07: Maximum Y coordinate (0x069B)

### CapTIvate protocol

The captivate touchpad seems to use an entirely custom protocol with
less capabilities. It has a simpler structure: The first byte is a
command byte, then data is either read or written, depending on the
command. Unlike the synaptics protocol, multi-byte values are
little-endian.

#### 01: Read "WHY_BOTHER_ME"

- Byte 0: 0/Unknown
- Byte 1: Flag byte
  - Bit 0: Set if touchpad pressed
  - Bit 1: Set if touchpad touched
  - Bit 2: Clear if touchpad touched (?)
  - Bit 3: Set if something changed after the last read
- Byte 2+3: X position
- Byte 4+5: Y position

#### 03: Write unknown

The OS writes a single byte here.

#### 06: Read status

- Byte 0: 0/Unknown
- Byte 1: Whether the touchpad is configured
- Byte 2-5: Unknown

#### 07: Read size

- Byte 0: 0/Unknown
- Byte 1: 0/Unknown
- Byte 2+3: Width
- Byte 4+5: Height

#### 08: Read firmware version

`   0, 0, 1, 0, 0, 4`

#### 0A: Read unknown

Six unknown bytes.

#### 0C: Write unknown

The OS writes a single byte here.

## Keypad Connector

The Connector joining the keypads to the nspire itself has a very basic
design concept: There are a bunch of GPIO pins, VCC, and GND. The OS
memory-maps the pins to the bit chart seen above, using half as input,
half as output. The same pins the touchpad uses for its I2C connection
are used for buttons on the clickpad.

The pins are as follows: 1. Ground 2. Vcc 3-30. GPIO pins, alternating
Column-Row-Column-Row

On the 84+ Pad: Pin 23 = Link port wire 1 Pin 25 = Link port wire 2

The OS is constantly doing iskeypressed() on all the keys, so the pins
end up looking like a square wave if read by a 'scope. Input pins only
show the wave if one of their buttons is pressed, but output pins show
the whole wave regardless.

If you can't envision how the keys actually work yet, this might help:
All the buttons have 2 connectors, that get shorted when they are
pressed. The bits in the table above represent columns of buttons. It's
not layed out perfectly because it has more than 8 buttons per column,
so it gets offset strangely. The bits in the offsets (the rows)
represent rows of buttons.

The columns are each hooked to one Input pin. The rows are each hooked
to one output pin. The calculator turns on *One* output pin, and looks
for any signals on the inputs. If there are any, it records the button
that matches the input pin and that column. It then repeats this process
for all the other columns, and the iskeypressed() process is complete.

If this doesn't make any sense still, email me (willrandship at gmail
dot com) and I'll see if I can help.
--<a href="/User:Willrandship" class="wikilink"
title="William Shipley">William Shipley</a> 05:38, 16 September 2011
(CEST)