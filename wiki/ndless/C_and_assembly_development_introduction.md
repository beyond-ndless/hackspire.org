---
title: C and assembly development introduction
permalink: /C_and_assembly_development_introduction/
parent: Ndless
layout: page
---

This tutorial describes how to set up an environment and use the Ndless
SDK to write native Ndless-compatible programs for the TI-Nspire.

---
1. TOC
{:toc}
---

## Install dependencies

### On Linux distros

- **Make sure your system has the following dependencies**: git, GCC
  (with c++ support), binutils, GMP (libgmp-dev), MPFR (libmpfr-dev),
  MPC (libmpc-dev), zlib, boost-program-options, wget. Install them with
  your system's package manager if not.

### On Mac OS X

- TODO
- **Make sure your system has the following dependencies**: git, GCC
  (with c++ support), binutils, GMP, MPFR, MPC, boost-program-options,
  zlib, wget. Install them with your system's package manager if not,
  for example brew.

### On Windows

MinGW and MSYS do not work correctly, so use WSL or install Cygwin
(32bit, x86). WSL is closer to a real Linux environment and likely
easier to set up.

#### Cygwin

Install the following dependencies: php (5.6+), libboost-devel,
libboost_program_options\*, binutils, gcc-core, gcc-g++, git, mpfr,
mpfr-devel, gmp, libgmp-devel, libmpc3, libmpc-devel, make, zlib-devel,
wget

#### Windows Subsystem for Linux (WSL)

Install a distro (which one should not matter much) and continue in the
Linux section.

## Build and install toolchain and SDK

- **Get the latest code from
  [GitHub](https://github.com/ndless-nspire/Ndless)**

`git clone --recursive `[`https://github.com/ndless-nspire/Ndless.git`](https://github.com/ndless-nspire/Ndless.git)

- On Windows, fix the few symlinks, for instance zehn.h in the
  ndless-sdk/tools/genzehn folder, which has to be deleted then copied
  there from ndless-sdk/include (and if you intend to rebuild Ndless,
  utils.c from the resources folder into the different installers
  folders)

<!-- -->

- **Run the SDK's *build_toolchain.sh***script that will download and
  build a complete ARM toolchain compatible with Ndless, and install it
  (edit the `PREFIX` variable at the beginning of the script to change
  the install location). You don't need to be root for this.

`cd ndless-sdk/toolchain/`
`./build_toolchain.sh`

Running the script again will continue from the last successful step
(not redownloading everything for instance). At the end of a successful
build you should see `Done!`. Alternatively you can verify the build
using `echo $?`. `0` indicates success.

- Now **add the following folders to your PATH environment variable**.
  On linux, `~/.bash_profile` should be a good place for this, just add
  something like this to it:

`export PATH="[path_to_ndless]/ndless-sdk/toolchain/install/bin:[path_to_ndless]/ndless-sdk/bin:${PATH}"`

- **Build Ndless and the SDK**, in the top level of the repository, run:

`make`

### Verifying the installation

- Open a console, and run:

`$ nspire-gcc`

If everything has been set up correctly you should see something similar
to:

`arm-none-eabi-gcc: fatal error: no input files`
`compilation terminated.`

## 2-minute tutorial

As a convention for the next chapters, lines starting with `$` are
commands you should type in a console. Other lines are the command's
output.

### Your first build

Ndless comes with sample programs in the *samples/* directory of the
Ndless SDK. We will try to build the C Hello World. Change the current
directory of the console:

`$ cd "`<my_ndless_sdk_copy>`/ndless-sdk/samples/helloworld-sdl"`

Ndless programs are built with *GNU Make*, which is run with the command
`make`. So let's *make* the program:

`$ make`
`nspire-gcc -Wall -W -marm -Os -c hello-sdl.c`
`mkdir -p .`
`nspire-ld hello-sdl.o -o ./helloworld-sdl.elf `
`genzehn --input ./helloworld-sdl.elf --output ./helloworld-sdl.tns --name "helloworld-sdl"`
`make-prg ./helloworld-sdl.tns ./helloworld-sdl.prg.tns`

`nspire-gcc` is Ndless's wrapper for the GNU C Compiler *GCC*, which
compiles C and assembly source files to object files (here *hello.o*).

`nspire-ld` is the wrapper for *GCC*, which redirects gcc with the
option "-fuse-ld=gold" to use another wrapper "arm-none-eabi-ld.gold" as
linker. "arm-none-eabi-ld.gold" adds some necessary libraries to the
final program.

`genzehn` converts the executable created by "nspire-ld" to a format,
which ndless supports.

`make-prg` adds a simple loader on top so the executable works on older
versions of ndless.

### Your first program

If you want to create a program from scratch:

- Create a new directory for the program
- Type in a console:

`cd "`<your directory path>`"`
`nspire-tools new `<name>


where <name> is your program name. This will create a Makefile to build
*<program>.tns*

- Create a new .c file and edit your program
- Run `make` to build it