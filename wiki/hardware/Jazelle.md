---
title: Jazelle
permalink: /Jazelle/
parent: Hardware
layout: page
---

Jazelle is a feature of the TI-Nspire's ARM926EJ-S CPU that implements a
subset of the Java bytecode instruction set in hardware.

The ARMv6 Architecture Reference Manual (DDI 0100I) gives some
information about Jazelle, intended for operating system programmers who
want to be able to correctly handle exceptions that occurred while in
Jazelle state, or manage the configuration registers if two Jazelle JVMs
with possibly different configurations could be running. However, most
of the details of how to actually write a Jazelle JVM are left
"<small>SUBARCHITECTURE DEFINED</small>". These details may be
elaborated in the Jazelle v1 Architecture Reference Manual (DDI 0225A),
but unfortunately this document is not available to the public.

## Configuration registers

Configuration registers may be read with the instruction
`MRC{<cond>} p14, 7, <Rd>, <CRn>, c0`, or written with
`MCR{<cond>} p14, 7, <Rd>, <CRn>, c0`. A list of known `CRn`
follows:

- c0: Jazelle Identity register (read-only)
  - Bits 0-11: Subarchitecture-defined bits (reads as 4, meaning
    unknown)
  - Bits 12-19: Subarchitecture (reads as 0, Jazelle V1 according to
    documentation)
  - Bits 20-27: Implementor (reads as 0x41, ARM Limited according to
    documentation)
  - Bits 28-31: Architecture (reads as 6, ARMv5TEJ according to
    documentation)
- c1: Operating System Control register
  - Bit 0: Configuration Disabled (CD) (documented)
  - Bit 1: Configuration Valid (CV) (documented)
- c2: Main Configuration register
  - Bit 0: Jazelle Enable (JE) (documented)
  - Bits 26-28: Unknown
  - Bit 29: If set, array object contains its elements directly,
    otherwise it contains a pointer to its elements
  - Bit 31: Disable array instructions if set?
- c3: Array object layout register
  - Bits 0-7: Unknown
  - Bits 8-11: Offset (in words) within array object of first element or
    of pointer to first element
  - Bits 12-15: Offset (in words) within array object of length
  - Bit 16: If set, offset to length is subtracted, otherwise added
  - Bits 17-19: Array length shift value (number of elements = stored
    length \>\> this)
  - Bits 20-21: Disable array instructions if set?

## Register usage

- r0-r3: Holds up to 4 words of the stack
- r4: Copy of local variable 0. Only valid when local variable 0 is a
  single word (i.e. int, float, or Object; not long or double)
- r5:
  - Bits 0-1: If any of the stack is held in registers, this specifies
    which of the registers in r0-r3 is currently the top of stack,
    otherwise this is 0
  - Bits 2-4: Number of stack words held in registers
  - Bit 5: Seems to get cleared upon entry to Jazelle state, unless a
    prefetch abort or data abort happens before any bytecode is read
  - Bits 6-9: Cleared immediately upon entry to Jazelle state
  - Bits 10-31: Pointer to software handler table (must be 1024-byte
    aligned)
- r6: Pointer to top of stack (Note that the Java stack grows **up**,
  not down)
- r7: Pointer to current method's local variables
- r8: Pointer to constant pool? (haven't checked this yet)
- r9-r11: Do not appear to be used; available for the JVM's own use
- r12: Temporary
- sp: Probably not used (the JVM's ARM code needs this for its own
  stack, after all)
- lr: The `bxj` instruction uses this as the address of the bytecode to
  jump to, and when an exception handler is called the address of the
  faulting instruction is stored here. (During actual Jazelle execution
  this is used as another temporary, but this is not visible to the JVM,
  so from the JVM's point of view this can be thought of as the Java
  program counter)

## Software handler table

This contains routines that Jazelle calls when it encounters a bytecode
it doesn't recognize, as well as in some exceptional conditions.

- +000-3FF: Unhandled bytecodes
  - The stack is flushed to memory before calling any of these handlers,
    so they may modify r0-r3 freely
- +400: Null pointer exception
- +404: Array index out of bounds exception
- +40C: Jazelle mode entered with JE = 0
- +410: Configuration invalid (Jazelle mode entered with CV = 0)
  - CV is automatically set to 1 on entering this handler
- +414: Prefetch abort occurred in middle of instruction

## Example code

The following code adds 2 and 2.

{% highlight asm %}
    .asciz  "PRG"
    push    {r4-r11, lr}

    msr cpsr_c, #0x13           @ Enable interrupts

    @ Set Configuration Valid and Jazelle Enable bits
    mov r0, #2
    mcr p14, 7, r0, c1, c0, 0
    mov r0, #1
    mcr p14, 7, r0, c2, c0, 0

    mov r5, #0x18000000         @ Handler table pointer (must be 1024-byte aligned)
    add r6, r5, #0x800          @ Stack pointer
    add r7, r5, #0x900          @ Local variables pointer

    @ Set up handler for ireturn bytecode
    adr r0, ireturn
    str r0, [r5, #0xAC * 4]

    @ Execute the bytecode
    adr r12, jazelle_unavailable
    adr lr, bytecode
    bxj r12

bytecode:
    .byte   0x05    @ iconst_2
    .byte   0x05    @ iconst_2
    .byte   0x60    @ iadd
    .byte   0xAC    @ ireturn

ireturn:
    @ Get result off the stack
    ldr r0, [r6, #-4]!

    @ Display it
    add r0, #'0'
    str r0, [sp, #-4]!
    mov r0, #0
    add r1, sp, #2
    mov r2, sp
    swi 30  @ show_dialog_box2
    add sp, #4

jazelle_unavailable:
    @ Restore configuration registers to 0
    mov r0, #0
    mcr p14, 7, r0, c1, c0, 0
    mcr p14, 7, r0, c2, c0, 0

    pop {r4-r11, pc}
{% endhighlight %}
