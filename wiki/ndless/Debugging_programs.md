---
title: Debugging programs
permalink: /Debugging_programs/
parent: Ndless
layout: page
---

## nspire_emu's native debugger

**<a href="/Emulators#nspire_emu" class="wikilink"
title="nspire_emu">nspire_emu</a>** includes a simple command-line ARM
debugger you can bring up from the *Emulation \> Enter debugger* menu.
It features ARM disassembly, instruction stepping, memory and
instruction breakpoints, backtracing and memory dumping. The debugger
commands available can be shown with the command '`?`'.

## GDB support

GDB is the [GNU Project Debugger](http://www.gnu.org/software/gdb/). The
nspire_emu emulator implements the [GDB Remote Serial
Protocol](http://sourceware.org/gdb/current/onlinedocs/gdb/Remote-Protocol.html#Remote-Protocol)
for compatibility with any [GDB front
end](http://en.wikipedia.org/wiki/Debugger_front_end) such as the
standard command-line interface,
[Insight](http://sourceware.org/insight/), [Eclipse
CDT](http://www.eclipse.org/cdt/) or the scite-debug SciTE plugin
available with the Windows NdlessEditor.

GDB allows source-level debugging, which means:

- for ARM developments, you can see you assembly symbols while debugging
- you can execute line-by-line your C programs, view the content of C
  variables during execution and set breakpoints from the source code

### With the NdlessEditor

Follow the [tutorial on
ndlessly](http://ndlessly.wordpress.com/debugging-with-gdb/).

### With the GDB command line interface

Run nspire_emu with the `/G` option:

`nspire_emu (... usual options) /G=3333`

This will make nspire_emu listen for GDB commands on the TCP port 3333.

You can now launch your favorite GDB front end and connect it to
`localhost:1000`.

On Windows, [YAGARTO](http://www.yagarto.de/) GNU ARM toolchain includes
the standard GDB client, run with `arm-none-eabi-gdb` from a command
prompt. The following GDB commands can be used to connect to Ncubate:

`file <your_program.gdb>`
`target remote localhost:3333`
`break main`

Running the *file* command before the *target* command is important for
address relocation.

You can the run the program from nspire_emu with the latest Ndless. The
program will break on the *main()* function. Then use `s` (step), `n`
(next) and [other GDB
commands](http://users.ece.utexas.edu/~adnan/gdb-refcard.pdf).

### With Eclipse CDT

[Eclipse CDT](http://www.eclipse.org/cdt/) is a GUI for C and C++
development based on GNU tools.

On Windows, make sure that MSYS's `bin` directory is [on the
PATH](http://www.computerhope.com/issues/ch000549.htm) before running
Eclipse, else you won't be able to build your program.

If running a 64-bit operating system, ensure that Yagarto is *not*
located in `C:\Program Files (x86)`. By default, Yagarto will install
here. Simply move it to `C:\Yagarto` and update your path by removing
the old reference to Yagarto and adding `C:\Yagarto\bin`

Once your source code is imported into an Eclipse project and can be
built within Eclipse, the visual debugger can be used to connect to
nspire_emu.

This [video tutorial](http://www.youtube.com/watch?v=JB-SnyZpbA4) shows
how to set up use the main features of Eclipse CDT's debugger (note: the
tutorial features the now out-of-date Ncubate emulator and an old
version of Eclipse CDT).