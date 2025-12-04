---
title: TI-84 Plus Emulation
permalink: /TI-84_Plus_Emulation/
parent: Operating System
layout: page
---

## TI-84 Plus Silver Edition OS

### Image file

The archive memory of the TI-84 Plus Silver Edition is integrated to the
TI-Nspire OS as 64KB files in a PK-Zip file (see the
<a href="/OS_upgrade_files#Compressed_file_system" class="wikilink"
title="OS upgrade file format">OS upgrade file format</a>). The OS code
is encrypted and perhaps compressed with the rest of the TI-Nspire OS
code in certificate field `8070`. Fortunately a ROM image of the
emulated TI-84 Plus can be dumped through the DBUS with standard tools
such as [TiLP](http://lpg.ticalc.org/prj_tilp/).

### OS Emulation

Do not expect the emulated TI-84 Plus Silver Edition to offer more
features than the real one. The speed is roughly the same, and the RAM
and archive memory have equivalent sizes (the whole memory of the
TI-Nspire is not visible from the emulated TI-84 Plus Silver Edition).

Starting with version 2.42 (and continued with 2.44 and 2.46),
even-numbered versions of the TI-84 Plus Silver Edition OS are used
while the latest version available for the real TI-84 Plus Silver
Edition is
[2.43](http://education.ti.com/educationportal/sites/US/productDetail/us_os_84plus.html).
The version of the boot code is 1.02, according to 0x400F of ROM page 7F
(see
[WikiTI](http://wikiti.denglend.net/index.php?title=83Plus:OS:ByROMPage)).

Some features are disabled in the Nspire versions: the key combo for the
self-test doesn't work, and the TI-84 Plus Silver Edition OS can't be
upgraded (an error is returned by the computer link software).

The TI-84 Plus Silver Edition ROM archive memory is set up with
preinstalled Flash applications, but they can be deleted from the
current memory image. These applications are restored on a full reset
from the back of the case.

## Hardware Emulation

### Z80

The emulated Z80 has several
<a href="/Special_Z80_Instructions" class="wikilink"
title="special opcodes">special opcodes</a>, which control functions
such as linking, Flash reading and writing, and lock/unlock the TI-84
Plus Silver Edition keypad.

### LCD

The TI-Nspire displays the TI-84+SE screen in a 288x192 pixel area, each
3x3 block of Nspire pixels corresponding to one TI-84+SE pixel. The
unused pixels outside this area are filled with gray. On Nspires with
the version 2 operating system (2.53MP in 84+ emulation mode) the border
is not filled with gray and remains white.

One glitch known to happen with the LCD is that after the processor
halts in low-power mode (turning the LCD off) and the LCD is turned back
on, the contrast is reset to a particular (most likely incorrect) level.

### USB

The USB ports are not emulated, and events on the USB bus are not
propagated to the memory-mapped port according to Calcsys and usb8x's
USBTools and PortMon. The TI-84 Plus/SE (and 89 Titanium) USB controller
has a quite complicated port interface, and the TI-Nspire probably uses
a different controller. Texas Instruments has probably preferred to
integrate a DBUS to the snap-in TI-84 Plus Silver Edition keypad and
emulate it rather than to waste time on USB emulation.

When connected to a computer linking software such as TI-Connect or
[TiLP](http://lpg.ticalc.org/prj_tilp/), a TI-89 Titanium is strangely
advertised by the <a href="/USB#USB_Descriptors" class="wikilink"
title="USB descriptors">USB descriptors</a> (fixed in versions 2.44+).
Commands can be sent by the computer to this fake Titanium through the
OUT endpoint, but the TI-Nspire never responds. Forcing *TI-84 Plus* as
device type in TiLP doesn't change anything. Since the USB descriptors
of the TI-84 Plus Silver Edition ROM image in the TI-Nspire OS have not
been changed, the USB attachment is entirely handled by the TI-Nspire,
and not by the emulated TI-84 Plus Silver Edition which never sees the
USB events as described above.

Connecting a USB optical mouse with an adapter in TI-84 Plus Silver
Edition mode makes the mouse flicker as in the TI-Nspire standard mode,
but much slower (~1 second instead of ~0.2s).

### Flash Writing

The Flash protection through port 14h works as it does in a real TI-84
Plus Silver Edition. However, there were some changes to communicating
with the Flash chip.

Instead of emulating the odd method of communicating with the Flash chip
through memory-mapped commands, these commands have been replaced with
<a href="/Special_Z80_Instructions" class="wikilink"
title="invalid instructions">invalid instructions</a>, which the
emulator will catch and intercept. When it comes across one of these
instructions, it will check the destination Flash page and will IGNORE
writes/erases if it does not fall within the user archive (pages
08h-69h). This makes OS updates through the TI-OS impossible, although
it is possible to work around this.

Writes to the user archive will be stored in the Nspire file system and
preserved if the keypad is removed. However, if you do manage to write
to OS space, writes here are NOT preserved and are restored to the
original OS contents when re-inserting the keypad. This implies that the
entire OS as well as the user archive is copied to RAM, and only written
back to the Flash chip when removing the keypad or turning off the
calculator.

If the certificate sector is erased and then the calculator is fully
reset, the certificate is rebuilt using the same calculator ID and
validation number. It is unknown whether this is stored somewhere or
calculated, perhaps based on the Nspire's unique ID.

## Working and Non-working Programs

Programs known not to work with the Nspire emulator include programs
which use undocumented instructions such as "in 0,(c)" and anything
involving IXH or IXL, and also anything that relies on direct USB
communication or the USB activity hook.

Please mention any TI-84 Plus Silver Edition programs which do not work
with the TI-Nspire integrated emulator, with enough details for the
authors or other hackers to fix them. Mention only *noteworthy* programs
when they work, which wouldn't obviously be compatible for example
because of special use of the TI-84 Plus Silver Edition hardware.

### Working

- **[Doors CS](http://www.dcs.cemetech.net/index.php?title=Main_Page)**:
  fully compatible
- **Noshell** : All features work except when running an incompatible
  assembly program.
- **MirageOS**: [Brandon Wilson](mailto:brandonlw_at_gmail.com) has
  released a [patch for MirageOS
  1.2](http://www.brandonw.net/calcstuff/mospatch.zip). Without it the
  TI-Nspire reboots when run with at least one assembly program in
  memory (because it uses IXH/IXL once it finds a program).
- **CalcUtil 2.021b**: All features work except when running an
  incompatible assembly program.
- **[Virtual
  Calc](http://www.ticalc.org/archives/files/fileinfo/196/19625.html)**

### Not Working

- [TI-Boy
  SE](http://www.ticalc.org/archives/files/fileinfo/419/41990.html):
  depends on undocumented functions not available in the emulator
- [RealSound](http://www.ticalc.org/archives/files/fileinfo/385/38513.html)
  Has very bad sound quality. In the mario example, [I hear tons of
  static and the word
  "Mario"](http://www.supload.com/sound_confirm.php?get=916292354.wav)
- [DCS 6.1](http://cemetech.net/)
- [SimCity
  83](http://www.ticalc.org/archives/files/fileinfo/324/32432.html)
- [Emu8x](http://www.ticalc.org/archives/files/fileinfo/377/37796.html):
  freezes on the "Mem cleared" screen of the emulated calculator.
- [msd8x](http://msd8x.denglend.net/): System hangs, forcing cold boot.
  Both msd8x and usb8x cannot work because they rely on the OS passing
  USB events to the USB activity hook, which does not fire in
  even-numbered OS versions. The complicated port interface is also not
  emulated, making most of the code in these applications useless.
  Making it work would require a massive rewrite of all the routines to
  use the <a href="/Special_Z80_Instructions" class="wikilink"
  title="special opcodes">special opcodes</a> from the emulator, and
  there is no guarantee there are or will be matching instructions for
  the necessary operations.

## We Need Your Help

- Try as many assembly programs and Flash applications for the TI-84
  Plus Silver Edition as possible on the TI-Nspire to find which ones do
  not work. We will try to make them compatible with the TI-Nspire, and
  may be TI-Nspire hackers may use the underlying emulation flaw. You
  may also try the non-tested and noteworthy programs mentioned
  <a href="#Working_and_non-working_programs" class="wikilink"
  title="above">above</a>.
- Calculate the difference between the CPU and the DBUS speed of the
  emulated TI-84 Plus Silver Edition and a real one.
- Test intensely and in detail the accuracy of the emulation (CPU,
  memory mapping, ports, ...) and report your tests.