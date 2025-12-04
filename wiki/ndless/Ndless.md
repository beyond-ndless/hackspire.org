---
title: Ndless
permalink: /Ndless/
layout: page
nav_order: 3
---

Ndless combines a resident program and utilities to open the TI-Nspire
to third-party C and assembly development. Ndless is distributed under
the Open Source [Mozilla Public
License](https://en.wikipedia.org/wiki/Mozilla_Public_License).

The installation of Ndless on a TI-Nspire is required to be able to run
<a href="/Assembly_programs" class="wikilink"
title="assembly programs">assembly programs</a>.

Find help for any Ndless related question or issues on the [issue
tracker](https://github.com/ndless-nspire/Ndless/issues).

'''[Latest stable release](http://ndless.me/)

## Developing with Ndless

Tutorials and general information are available in the
<a href="/#Ndless" class="wikilink"
title="Development resources">Ndless</a> section.

### Version upgrade

The executable format, the conventions and the header files are
currently being defined and prone to change. Some changes are forced by
This section describes the upgrade steps between the different releases
of Ndless.

#### To v4.4 r2005

- Should work directly for most programs if they don't contain any
  OS-version specific addresses (SYSCALL_CUSTOM(), hooks, etc.)

#### To v4.2 r2004

- HW rev. W introduced a new 240x320px screen which breaks most Ndless
  programs.
- A new API to make use of the different screen is now available,
  SCREEN_BASE_ADDRESS is only available in OLD_SCREEN_API (Add the
  define to the compile options) mode. See
  <a href="/Libndls#LCD_API" class="wikilink" title="LCD API">LCD API</a>
  for information about the new API.
- Nspire I/O and nSDL support the new screen, and programs that use them
  should work after recompilation.

#### From v3.9 to v4.2

- Should work directly for most programs if they don't contain any
  OS-version specific addresses (SYSCALL_CUSTOM(), hooks, etc.)

#### From v3.6 to v3.9

- Migration to newlib and libsyscalls. os.h and common.h shouldn't be
  used anymore. Use the standard C API directly.
- Introduction of the new
  <a href="/Zehn" class="wikilink" title="Zehn">Zehn</a> format.
  Makefiles need to be updated to use
  <a href="/Zehn#genzehn" class="wikilink" title="genzehn">genzehn</a>
  and
  <a href="/Zehn#make-prg" class="wikilink" title="make-prg">make-prg</a>.

#### From v2.0 to v3.1

- Programs must be rebuilt to work on TI-Nspire CX because of several
  hardware changes. Programs with low-level accesses to hardware must
  also be slightly adapted: some hardware controllers have changed and
  their registers are laid out differently. The
  <a href="/libndls#Hardware" class="wikilink" title="IO() macro">IO()
  macro</a> can be used to select the address corresponding to the
  TI-Nspire model.
  - The LCD control register of the
    <a href="/Memory-mapped_I/O_ports#C0000000_-_LCD_controller"
    class="wikilink" title="LCD controller">LCD controller</a> has moved
    from C000001C to C0000018. Use IO_LCD_CONTROL instead.
  - The <a href="/Memory-mapped_I/O_ports#900D0000_-_Second_timer"
    class="wikilink" title="timer module">timer module</a> has changed
    and has completely different registers
  - Programs are run in color mode. Add *clrscr(); lcd_ingray();* at the
    beginning of a program ported from classic TI-Nspire to CX (the
    *clrscr()* avoids displaying color data that remains in screen
    buffer)
  - Keep the TI-Nspire classic user base in mind: make your programs
    compatible with both color and grayscale mode with the help of
    *has_color* . The color screen buffer is in
    <a href="/Memory-mapped_I/O_ports#A4000000_-_Internal_SRAM"
    class="wikilink" title="R5G6B5 mode">R5G6B5 mode</a>.
- Programs which use *show_msgbox()* must be rebuilt with the new SDK,
  even for TI-NSpire classic
- SYSCALL_CUSTOM(): the first parameter should now only contain
  <a href="/Ndless_features_and_limitations#Syscalls" class="wikilink"
  title="OS v3.1 addresses">OS v3.1 addresses</a>.

#### From v1.7 to v2.0

- Touchpad key map support: rebuild your programs with the SDK of v2.0,
  which includes an updated `isKeyPressed()` function. Make sure that
  the programs do not depend on keys specific to the Clickpad.

#### From v1.1.x to v1.7

C and assembly programs:

- The syscall convention has changed, programs built with Ndless \< v1.7
  need to be rebuilt without any change to the code. This format should
  be compatible with the new OS versions once supported by Ndless, as
  long as backward compatibility can be kept.
- Custom syscalls should be defined using SYSCALL_CUSTOM (see above)
- The "PRG\0" signature before main() isn't required anymore, you can
  remove it
- MakeTNS doesn't exist anymore. You must objcopy directly to the .tns
  file in your Makefile.
- The program format is not specific to an hardware model anymore. You
  can now build your programs only once without defining
  NSPIRE_HARDWARE.
- OS v1.1 is not supported anymore. Check that your programs still work
  on OS v1.7.
- The program path is passed to the main function by following the C
  calling convention instead of using register r9

Assembly programs:

- The way to call syscalls has changed: replace 'oscall <os_function>'
  with 'syscall(<os_function>)'

#### From v1.0 to v1.1

C and assembly programs:

- Install the toolchain as described above. Your project doesn't need to
  follow the Ndless file tree anymore.
- Adapt your Makefile based on src/arm/Makefile or src/arm/demo/Makefile
- You may upgrade to the latest version of YAGARTO. Don't forget to
  delete any remaining \*.o file before rebuilding a project.

Pure-assembly programs:

- Make sure that the file extensions is in uppercase (.S)
- Add the "main" symbol as described in the previous section
- You do not need to define the symbol "_start" anymore