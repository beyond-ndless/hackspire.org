---
title: Special Z80 Instructions
permalink: /Special_Z80_Instructions/
parent: Operating System
layout: page
---

The Nspire's Z80 emulator interprets many invalid instructions as
special opcodes.

## Prefixes

It seems that there are three two-byte prefixes for every special
opcode. Here are the three found in the odd-numbered OS versions so far:

- 0EDh, 0EDh: this happens at many points in the OS. It has data after
  it (specifying a specific operation).
- 0EDh, 0EEh: this is where the Flash sector erase commands were in
  _EraseFlash.
- 0EDh, 0EFh: this is where the Flash write commands were in
  _WriteFlash.

## Locations

Below are some of the locations in OS 2.42 where these opcodes are used.

```
00:090Eh: 0EDh,0EDh,02h,10h,18h,05h
00:0B04h: 0EDh,0EDh,05h,10h,0CDh,80h,3Bh,0CDh,0ECh,3Bh,0C3h,0F2h,3Bh
7F:4BE4h: 0EDh,0EEh (Flash sector erase from _EraseFlash)
7F:4C4Dh: 0EDh,0EFh (Flash write from _WriteFlash)
7F:6016h: 0EDh,0EDh,06h,10h
7D:73AFh: 0EDh,0EFh (Flash write duplicate?)
7D:7BDCh: 0EDh,0EDh,03h,10h,0C9h
7D:7C50h: 0EDh,0EDh,04h,10h,0C9h 
```

## 0EDh, 0EDh Opcodes

Each of these opcodes has a prefix of two 0EDh bytes and a suffix of
10h. Some of these will reboot the Nspire if run from Flash. A lot of
these will reboot the Nspire if run from RAM. Get around this by putting
a small loader in Flash (like page 0). This documentation is horribly
incomplete. If you can confirm the correctness of any of this, please
do.

- 02h: this is executed before turning off (right before BCALL
  _PowerOff). Don't know what it does, might not work from RAM.
- 03h: this is executed before _SetAppRestrictions...it locks the
  Nspire so that only the 84+SE keypad can be used, not the Nspire one.
  Reboots the Nspire if executed from RAM.
- 04h: this is executed before _RemoveAppRestrictions...it unlocks the
  Nspire so that either keypad can be used, instead of just the 84+SE
  one. Reboots the Nspire if executed from RAM.
- 05h: this is executed before displaying reset text on homescreen and
  after zeroing RAM. Might initialize USB ViewScreen as host or itself
  as peripheral (unlikely). Makes the LCD go gray all over. Reboots the
  Nspire if executed from RAM.
- 06h: this is executed in the boot code, receives a byte through the
  I/O link. Returns NZ if failed. Guessing it returns in A, don't know
  if it blocks.
- 08h: this is executed somewhere on page 6 (OS 2.44). Might check if
  the I/O link is good or something. If NZ returned, USB used. Maybe
  "bit 5,(iy+41h)", might mean more USB data should be read.
- 09h: this is executed somewhere on page 74h (OS 2.44). This finishes
  an output of bytes in either host or peripheral mode.
- 0Ah: this is executed somewhere on page 74h (OS 2.44). This is called
  before receiving bulk data. Returns C if problems.
- 0Bh: this is executed somewhere on page 74h (OS 2.44). Returns NZ if
  an A cable is plugged in.
- 0Ch: this is executed somewhere on page 7Ch (OS 2.44). Called
  immediately after pressing kReceive on LINK menu. If Z returned, USB
  attempted without 0Dh trap.
- 0Dh: this is executed somewhere on page 74h (OS 2.44). Called
  immediately after pressing kReceive on LINK menu. If NZ returned, I/O
  used instead of USB. Equivalent: "in a,(4Ch) \\ and 8"
- 0Eh: this is executed somewhere on page 74h (OS 2.44). Gets a byte of
  bulk data to A from either port 0A1h or 0A2h, depending on whether
  calc is host or peripheral. Returns C if problems.
- 0Fh: this is executed somewhere on page 74h (OS 2.44). You must call
  this after using 0Eh. It finishes outputting the byte, or something.
- 10h: this is executed somewhere on page 74h (OS 2.44). This is called
  just before outputting bulk data. Use this to start it. Returns C if
  problems.
- 11h: this is executed somewhere on page 74h (OS 2.44). This outputs a
  byte (A) to either port 0A1h or 0A2h, depending on whether calc is
  host or peripheral.
- 12h: this is executed somewhere on page 7Ch (OS 2.44). Called
  immediately after pressing kReceive on LINK menu. This is some sort of
  USB init and returns NZ if failure.
- 13h: this is executed somewhere on page 74h (OS 2.44). This is
  basically BCALL 5290h, full USB init.
- 17h: this is executed somewhere on page 74h (OS 2.44). This requests
  incoming bulk data (the first time, I think).
- 18h: this is executed somewhere on page 74h (OS 2.44). This is called
  in BCALL 5260, so maybe it's some sort of USB shutdown. Might clear
  any pending input/output data.
- 19h: this is executed somewhere on page 74h (OS 2.44). Equivalent: "in
  a,(8Fh) \\ and 5 \\ cp 5" Initializes USB device if NZ returned.
- 1Ah: this is executed somewhere on page 74h (OS 2.44). Not sure, but
  used in USB init. Must return NZ if USB init is okay to do.
  Equivalent: "ld a,(9C75h) \\ bit 6,a"
- 1Bh: this is executed somewhere on page 74h (OS 2.44). Same as 1Ah,
  not really sure.
- 1Ch: this is executed somewhere on page 74h (OS 2.44). This kills the
  USB device. Destroys A, apparently.
- 1Dh: this is executed somewhere on page 74h (OS 2.44). This starts an
  output of bytes to port 0A0h.
- 1Eh: this is executed somewhere on page 74h (OS 2.44). This sends the
  byte in A out port 0A0h.
- 1Fh: this is executed somewhere on page 74h (OS 2.44). This finishes
  an output of bytes to port 0A0h.
- 20h: this is executed somewhere on page 0 (OS 2.44). I don't know, but
  it checks something USB-related. The OS will call the 9C1Ch callback
  if it returns NZ.
- 21h: this is executed somewhere on page 76h (OS 2.44). I have no idea,
  but it returns something in NZ (error)? and is link or USB related.
  Basically "bit 0,(iy+43h)". Used in boot code (6Fh).
- 22h: this is executed somewhere on page XXh (OS 2.44). Somehow related
  to 90A8h/90A9h, but I THINK that this waits a certain number of
  milliseconds (crystal timer replacement?). Uses BC as input.
- 24h: this is executed somewhere on page 0 (OS 2.44). This returns
  whether bulk data is ready to be read. Returns NZ if so. The OS will
  call the 9C1Eh callback after reading it.
- 25h: this is executed somewhere on page 74h (OS 2.44). Not sure. Might
  be some sort of USB error thing.
- 26h: this is executed somewhere on page 74h (OS 2.44). This reads a
  byte in from port 0A1h and puts it in A.
- 27h: this is executed somewhere on page 0 (OS 2.44). I don't know, but
  it checks something USB-related. The OS calls the 9C13h callback if it
  returns NZ.
- 29h: this is executed somewhere on page 6 (OS 2.44). I don't know, but
  it checks something USB-related. If it returns NZ, the OS calls the
  auto-launch stuff.
- 2Ah: this is executed somewhere on page 6 (OS 2.44). I don't know, but
  it does something USB-related. It MIGHT return vendor/product ID and
  firmware version info in HL/DE/BC.
- 2Bh: this is executed somewhere on page 74h (OS 2.44). This finishes
  reads from port 0A1h. It returns something in A...might be 0FFh if
  there's more data to get, 0 if none.
- 2Ch: this is executed somewhere on page 74h (OS 2.44). It looks like
  this requests more reads from port 0A1h.
- 2Fh: this is executed somewhere on page 79h (OS 2.44). I don't know,
  but happens just before low battery message displayed in boot code,
  and just after on page 79h.
- 30h: this is executed somewhere on page 0 (OS 2.44). No idea, but it's
  called just after the initial _getKey call on reset.

## Other Prefixes

- 0EDh, 0EEh: this replaces the Flash chip commands in _EraseFlash. If
  you specify a page in OS space, the sector erase will not take effect.
- 0EDh, 0EFh: this is where the Flash write commands were in
  _WriteFlash. It appears to set up emulator pointers so that the
  following "ldi" will write to the proper file in the Nspire file
  system. If you specify a page in OS space, the write will not take
  effect.