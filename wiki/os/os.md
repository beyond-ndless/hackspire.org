---
title: Operating System
permalink: /os/
layout: page
nav_order: 5
---

This article currently contains notes and ideas jotted down, but may be
splitted and reorganized in the future as knowledge becomes more
structured and comprehensive.

## Booting

*Booting* happens when:

- a battery is removed and put back after a few seconds
- after a crash of the TI-Nspire OS
- when the keypad is changed
- after an OS update
- after the maintenance menu (see below)

There is no backup battery as we can find on previous calculator models,
so removing at least one AA battery for a few seconds make the RAM lose
its content. In TI-Nspire mode removing, and putting back the same
keypad doesn't make it boot. In TI-84 Plus mode, removing the keypad
automatically stores the RAM image to the flash memory (we can see it
process for one second in TI-84 Plus mode when removing the keypad while
the calculator is still on). If the TI-84 Plus keypad is put back within
a few seconds, the TI-84 Plus will immediately start as after a
shutdown. If the keypad is removed for more than a few seconds, the
TI-Nspire OS will boot, and the RAM and archive memory of the TI-84 Plus
will be restored at its last state. It seems that the exception handler
of the TI-Nspire OS systematically forces a reboot: you won't see
anything like *illegal instruction* in a black bar on the screen as
there is on TI-68k.

In TI-Nspire mode, the About screen indicates two boots, *boot 1* in
version 1.1.8916 and *boot 2* in version 1.8981. The versions are the
same for TI-Nspire CAS. Whether booting in TI-Nspire mode or in TI-84
Plus mode, a progress bar is displayed. At half-way, the screen flashes
as if it was reinitialized, then the progress bar continue to be filled
up and 2 messages are displayed sequencially: *"Preparing file system,
please wait..."* and *"Loading Operating System..."*. The supposed
memory setup during boot up is described in
<a href="/Memory_layout" class="wikilink" title="Memory layout">Memory
layout</a>. We expect "boot 1" to be what is run during the first half
of the progress bar, and "boot 2" during the second half. The boot code
exists even without OS, and seems to use 4224KB of storage capacity on a
TI-Nspire (see
<a href="#Upgrading_the_OS" class="wikilink" title="further">further</a>).

## Startup key combinations

The boot 1 and boot 2 both accept key combos as startup options. When
the TI-Nspire *is ready to boot* (for example after a battery has been
removed and reinstalled), it can be turned on by holding the following
keys while pressing ON (making sure ON is the last key pressed):

|  |  |  |  |
|----|----|----|----|
|  | Clickpad | TI-84+ Keypad | Touchpad |
| Boot 1: | Home+Enter+B | Graph+Enter+Apps | Doc+Enter+2 |
|  | Home+Enter+G | Graph+Enter+Tan | Doc+Enter+Minus |
|  | Esc+Menu+G \| y=+Clear+Tan | Esc+Menu+Minus | toggle diagnostics - if boot data is configured to try to run diagnostics first, this will skip them, and vice versa. |
| Boot 2: | Home+Enter+X | (not possible) | Doc+Enter+?! |
|  | Home+Enter+P | Graph+Enter+8 | Doc+Enter+EE |

## Upgrading the boot 2

The boot 2 can be upgraded:

- During an OS upgrade, if a *boot2.img* is present in the OS upgrade
  file (see
  <a href="/OS_upgrade_files" class="wikilink" title="OS upgrade files">OS
  upgrade files</a>)
- Through
  <a href="/Hardware#RS232" class="wikilink" title="RS232">RS232</a>,
  with the X-Modem protocol, by sending the file *boot2.img*.

The boot image is not check before being written to flash memory but
checked after reboot. The signature of the file prevents corrupted or
modified boot images from being sent.

A boot 2 image greater than 1358976 bytes (~1327kB, i.e. 10617 128-bit
X-Modem packets) cannot be sent: the warning sign is displayed on the
Nspire screen, and lsz hangs with "Got 44 for sector ACK". The size of
*boot2.img* v1.4.1571 is 1216KB. (caution, this also could be the result
of a streamed image validation by boot 1).

## Third-party components

The About screen reports that the following third-party components are
integrated to the TI-Nspire OS:

- *Graph & Geometry* developed in conjunction with Cabrilog SAS
- "Data & Statistics" design and developed in conjunction with KCP
  Technologies
- Portions of the software are based on Fathom(TM) Dynamic Data and
  copyright ©2007 KCP Technologies.
- *[Nucleus Real-Time Operating
  System](http://www.mentor.com/products/embedded_software/nucleus_rtos/index.cfm)*
  by [Mentor Graphics Corp.](http://www.mentor.com/). Note that the
  source code of the OS is provided to licensees, so it could have been
  customized by Texas Instruments. Nucleus RTOS is also a modular OS, it
  is currently not clear which modules are included.
- The "Flash Media Manager" [FlashFX
  Pro](http://datalight.com/products/flashfx/) and the "High-Integrity
  Transactional File System"
  [Reliance](http://datalight.com/products/reliance/) by [Datalight
  Inc.](http://www.datalight.com/) (both compatible with Nucleus RTOS),
  for a fault-tolerant file system, presented in this [success
  story](http://www.datalight.com/companyinfo/success.php?successid=11).
  Only the NAND flash memory of the TI-Nspire is supported by FlashFX,
  and not the NOR memory.
- The compression library *[zlib](http://www.zlib.net/)*
- The XML parser *[eXpat](http://expat.sourceforge.net/)*
- The *[OpenSSL](http://www.openssl.org/)* library

## Weblinks

[TI-Nspire OS
Archive](http://tiplanet.org/forum/archives_list.php?id=OS+Nspire)
almost all OS releases