---
title: Zehn
permalink: /Zehn/
parent: Ndless
layout: page
---

Zehn is a new executable format, which supports:

- Full relocation (No "-fpic" or similiar needed)
- Every kind of section in an ELF file (E.g. ARM unwind information for
  C++ exceptions)
- Executable files up to 16MB in size (can be larger if no relocations
  after 0xFFFFFF)
- Various flags and extra information about the executable:
  - Maximum and minimum version and revision of ndless needed to run
    (https://cloud.githubusercontent.com/assets/1622084/3762493/a7ed3a22-1897-11e4-8d94-77f82ecfa239.png)
  - HW types (Clickpad/Touchpad/CX/CM) supported
  - Application name, version, author, description (these can be viewed
    by holding down the "catalog" key when opening the executable:
    <https://cloud.githubusercontent.com/assets/1622084/3762440/0d70af1a-1897-11e4-860b-cf07b6521634.png>)

## Name

"Zehn" is German for 10 while ELF is German for 11.

## Format

The main header file
(https://github.com/ndless-nspire/Ndless/blob/master/ndless-sdk/include/zehn.h)
contains everything needed to parse a Zehn file. The Zehn header
(identified by ZEHN_SIGNATURE) may be everywhere (4-byte aligned) in the
first 4KiB if the file starts with "PRG\0" to support
backwards-compatibility.

## genzehn tool

The "genzehn" tool
(https://github.com/ndless-nspire/Ndless/blob/master/ndless-sdk/toolchain/genzehn/genzehn.cpp)
converts ELF files with embedded relocation info (needs to be linked
with "--emit-relocs") to relocatable Zehn files. Run "genzehn --help" to
get version and usage information. For basic conversion it's sufficient
to only specify "--input input.elf" and "--output output.tns". The
resulting executable will open on every version of OS, ndless and type
of hardware and the version will be 1.

## make-prg tool

The "make-prg" tool
(https://github.com/ndless-nspire/Ndless/blob/master/ndless-sdk/bin/make-prg)
adds zehn_loader.tns at the top of the file, so that older versions of
ndless without native support for Zehn. "zehn_loader.tns" is also used
to boostrap ndless_resources, which is a Zehn file itself. It doesn't
support any of the flags, so your executable may be executed on
unsupported hardware or significantly too old OS versions! To use it,
execute "make-prg <input>

<output>

". It compiles zehn_loader.tns if needed.