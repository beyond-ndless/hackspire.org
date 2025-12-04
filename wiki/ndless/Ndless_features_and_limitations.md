---
title: Ndless features and limitations
permalink: /Ndless_features_and_limitations/
parent: Ndless
layout: page
---

<a href="/Ndless" class="wikilink" title="Ndless">Ndless</a> combines an
executable loader and utilities to open the TI-Nspire to third-party C
and assembly development.

This article covers advanced features of Ndless for developers. If you
want to try C and assembly development on the TI-Nspire, start with
<a href="/C_and_assembly_development_introduction" class="wikilink"
title="the tutorial">the tutorial</a>.

---
1. TOC
{:toc}
---

## nspire-tools utility

Since v3.1 r541, `nspire-tools` can be used to execute various actions
on your build environment.

- `nspire-tools new `<program>: create in the current directory a
  Makefile to build a program called *<program>.tns*

## Entry point

You can optionally define the standard [argc and
argv](http://publications.gbdirect.co.uk/c_book/chapter10/arguments_to_main.html)
parameters in your main function definition to get the current program
path in `argv[0]`:

`int main(int argc, char *argv[]) {`
`  printf("Run from '%s'\n", argv[0]);`
`}`

If the program is registered to open files with a specific extension (in
ndless/ndless.cfg.tns) and such a file has been opened, the absolute
path to the file is in argv\[1\].

## Program loader

3 types of executable files are currently supported:

- "PRG" loader: No relocation, executable begins after the signature
  "PRG\0"
- [bFLT](http://retired.beyondlogic.org/uClinux/bflt.htm): Basic
  relocation, commonly used by embedded Linux distributions, and
  supported since Ndless v3.1 r806 (thanks tangrs). Due to it's
  limitations and awful bugs in the "elf2flt" tool it has been
  superseded by the
  <a href="/Zehn" class="wikilink" title="Zehn">Zehn</a> format and the
  bFLT tools have been removed from the SDK. bFLT programs will still
  work and the bFLT loader will not be removed in the near future.
- <a href="/Zehn" class="wikilink" title="Zehn">Zehn</a>: Complete
  relocation, support for flags and c++ exceptions, backwards-compatible
  with the "make-prg" tool

## Using static libraries

Since v3.1 r604, Ndless's `nspire-gcc` and `nspire-ld` compilation tools
create and use the directory `.ndless` for static libraries. This
directory is stored in `${USERPROFILE}`, which should be something like
`C:\Users\`<Your Account> on Windows and `/home/`<username> on unix
based machines. This is the recommended directory to drop third-party
libraries your program uses into, as it is untouched during Ndless SDK
version upgrades.

To use a static library:

- Drop the include files (.h) into `.ndless/include`
- Drop the library files (lib\*.a) into `.ndless/lib`
- In your program's Makefile, Add the option `-l`<libname> to `LDFLAGS`,
  where `libname` is the name of the library without its `lib` prefix
  (for example for libnspireio2.a it should be <i>-lnspireio2</i>)

## Checking Ndless's version

Ndless is frequently updated with new syscalls. Calling a syscall on a
previous version of Ndless will make the calculator crash.

Protect your Ndless version-dependent program with libndls's function
<a href="/Libndls#Platform" class="wikilink"
title="assert_ndless_rev(unsigned required_rev)"><code>assert_ndless_rev(unsigned required_rev)</code></a>
at the beginning of it. The revision is the ZZZ part of "vX.Y rZZZ".
libndls functions and syscalls which won't work if Ndless is not
up-to-date have the revision indicated in their documentation.

## Syscalls

<a href="/Syscalls" class="wikilink" title="Syscalls">Syscalls</a> are OS
functions exposed by Ndless to C and assembly programs. Some syscalls
are part of standard libraries and may be used to port programs to the
TI-Nspire more easily.

To call a syscall in assembly, use `syscall(`<symbol name>`)`.

If you have found an interesting syscall not defined in the latest
version of Ndless, you can temporary define it locally to your program
with `SYSCALL_CUSTOM()`. For example to call the syscall
`int internal_func(char *s)` you have found, define in one of your
program's header files:

`static const unsigned internal_func_addrs[] = {0x10123456, 0x10654321, 0x10234567, 0x11234567, 0x11234567, 0x11234567}; // non-CAS 3.1, CAS 3.1, non-CAS CX 3.1, CAS CX 3.1, CM-C 3.1, CAS CM-C 3.1 addresses`
`#define internal_func SYSCALL_CUSTOM(internal_func_addrs, int, char *s)`

This is a temporary workaround which will force a re-build of the
program each time a new OS version supported by Ndless comes out. You
should suggest the syscall to the Ndless development team to make it
standard, publicly available and maintained.

## OS variables

Ndless also abstracts access to some OS variables.

To read from or write to these variables in assembly, use
`osvar(`<symbol name>`)`. The address of the variable is returned in
`r0`.

## Assembly source code

Pure-assembly programs must define the global symbol "main" as their
entry point.

Always make sure that the assembly files extensions are in uppercase
(`.S`) to let them be preprocessed by the C preprocessor on which Ndless
include files depend.

## Thumb state

The TI-Nspire ARM9 processor features a Thumb instruction set state
which improves compiled code density with 16-bit instructions instead of
32-bit.

Only ARM state `main()` entry points is currently supported, but other
source files may be built in Thumb state. Switching from one state to
the other is transparent to C programs as long as the
`-mthumb-interwork` GCC switch is used. To build a `.c` file in Thumb
state, use a GCC command similar to:

`test_thumb.o: test_thumb.c`
` $(GCC) $(GCCFLAGS) --mthumb-interwork -mthumb -c $< -o $@`

Calling syscalls in thumb state is supported by Ndless.

Assembly source files must declare the instruction set to use with the
`.arm` and `.thumb` directives (ARM is the default). Switching from one
mode to the other requires [specific
tricks](http://www.eetimes.com/discussion/other/4024632/Introduction-to-ARM-thumb).

## Newlib

[Newlib](http://sourceware.org/newlib/) is an implementation of the C
library intended for use on embedded systems. Newlib uses the OS's
functions for IO through an abstraction layer "libsyscalls".

## File association

File association is configured through the configuration file
*ndless.cfg.tns* (see the [User
Guide](http://ndlessly.wordpress.com/ndless-user-guide/#file-association)).
Once the file extensions supported by your program are declared in the
file, your program is run with the *argv\[1\]* parameter set to the full
path of the file open from the OS document browser. Check that *argc \>=
2* to know if the program is being run through file association. Always
check the format of file open this way.

You may ask Ndless's maintainers to add your file assocation to the
default *ndless.cfg.tns* file, if the extensions aren't already defined.

## Resident programs

You can ask Ndless not to free the program's memory block after it exits
by calling `void nl_set_resident(void)`. This can be use for resident
programs such as OS patches.

Resident programs shouldn't use `argv[]` after returning.

This is currently no way to free a resident program memory block.

## Builtin functions

Ndless exposes internal features and states throw the nl_\*()
functions.

- `BOOL nl_isstartup(void)`: (since v3.1 r540) returns TRUE if the
  program is currently being run at OS startup. See the [User
  Guide](http://ndlessly.wordpress.com/ndless-user-guide/#startup).
- `int nl_osvalue(const int values[], unsigned size)`: returns the value
  of `values` corresponding to the OS version. `size` is the number of
  values. values\[0\] corresponds to non-CAS 3.1, values\[1\] to CAS
  3.1, values\[2\] to non-CAS CX 3.1, values\[3\] to CAS CX 3.1,
  values\[4\] to CM-C 3.1, values\[5\] to CAS CM-C 3.1, values\[6\] to
  non-CAS 3.6, values\[7\] to CAS 3.6, values\[8\] to non-CAS CX 3.6,
  values\[9\] to CAS CX 3.6.
- `void nl_set_resident(void)`: (since v3.1 r553) see
  <a href="#Resident_programs" class="wikilink"
  title="Resident programs">Resident programs</a>
- `void nl_no_scr_redraw(void)`: (since v3.1 r756) don't restore the
  screen on program exit
- `BOOL nl_loaded_by_3rd_party_loader(void)`: (since v3.1 r791) return
  TRUE if a third-party Launcher was used to boot the OS, such as
  nLaunch/nLaunchy
- `int nl_exec(const char *prgm_path, int argsn, char *args[])`: (since
  v3.1 r877) run a program. `prgm_path` is its full path with .tns
  extension, `argsn` is `args[]` size, `args[]` sets the program
  additional arguments, passed through the main function's `argv[]`
  parameter. `args[]` must not include the program name (`argv[0]`) nor
  the terminating NULL argument. If the the program is run through a
  file association, `argv[2]` will be set to `args[0]`. `argsn` and
  `args[]` may be respectively 0 and NULL.