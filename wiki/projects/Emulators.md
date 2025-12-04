---
title: Emulators
permalink: /Emulators/
parent: Projects, programs and tools
layout: page
---

1. TOC
{:toc}
---

## Current programs

### Firebird

**Firebird** is a fork of nspire_emu. It runs natively on Windows,
Linux, Mac, Android, and iOS, thanks to a Qt-based GUI. It is still in
development, but is available [on
GitHub](https://github.com/nspire-emus/firebird/releases/latest).
Added/Changed features compared to nspire_emu are mainly improvements to
the emulator core to make it more powerful as well as a fully-featured
GUI to go along more user-friendly additions.
Like nspire_emu, use
[PolyDumper](https://tiplanet.org/forum/archives_voir.php?id=3829) to
dump the *boot1.img.tns* and *boot2.img.tns* from your TI-Nspire. Diags
and Manuf can also be dumped and used in Firebird (see the flash
creation feature)

## Obsolete programs

### nspire_emu

**nspire_emu** is a TI-Nspire Clickpad/TouchPad/LabStation/CX (CAS)
emulator, and was the first one made. It only supports Windows, but is
compatible with [Wine](https://www.winehq.org/) on other OSes. You can
download it
[here](https://tiplanet.org/modules/archives/download.php?id=10068) or
[here](https://www.omnimaga.org/ti-nspire-projects/ti-nspire-emulator/?action=dlattach;attach=14364).
Its official discussion thread is on
[Omnimaga](https://www.omnimaga.org/ti-nspire-projects/ti-nspire-emulator/).
Most TI-Nspire emulators derive from *nspire_emu*. Note that these
custom versions are not always kept up-to-date with the latest version
of *nspire_emu*. **Setup:**

- Use [PolyDumper](https://tiplanet.org/forum/archives_voir.php?id=3829)
  to dump the *boot1.img.tns* and *boot2.img.tns* from your TI-Nspire.
- Open a DOS console
- Follow [these
  instructions](https://www.omnimaga.org/ti-nspire-projects/ti-nspire-emulator/msg279905/#msg279905)

### Ncubate

**[Ncubate](https://www.omnimaga.org/other-calculator-discussion-and-news/ncubate-enhanced-ti-nspire-emulator/)**
is a derived version of *nspire_emu* enhanced with features useful to
developers, such as state saving/reloading, additional debugger commands
and
<a href="/Debugging_programs#Ncubate&#39;s_GDB_support" class="wikilink"
title="support for the GDB debugger">support for the GDB debugger</a>.
It is not compatible with OS v3.x.

### Xspire

**Xspire** is a port of *nspire_emu* for the GTK+ GUI library,
compatible with Windows and Linux. This port also supports skins. You
can download it from the [United-TI
thread](http://www.unitedti.org/forum/index.php?showtopic=8191&st=720)
(account required). Note that a more recent version may be available
further in the thread. The GDK/GTK+ libraries are required for it to
work (Windows users should use the [Windows
port](http://sourceforge.net/projects/gtk-win/)).

### Nspire Memory Editor

The **Nspire Memory Editor** plugs itself into *nspire_emu* to offer
advanced memory-related features, such as hexadecimal memory edition,
string and binary search, memory chunks read and write, and string or
instruction-based breakpoints. Download it from the [United-TI
thread](http://www.unitedti.org/forum/index.php?showtopic=8191&st=720)
(account required). Note that a more recent version may be available
further in the thread.