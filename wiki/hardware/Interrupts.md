---
title: Interrupts
permalink: /Interrupts/
parent: Hardware
layout: page
---

## General Processor-related Info

On the ARM processor, there are two types of interrupts:

### IRQ

IRQ is the typical interrupt type (and is the one used by TI-Nspire OS).
It can be disabled by setting bit I (0x80) in the CPSR register.

When an IRQ is triggered, the following happens:

- The return address plus 4 is copied to the IRQ_LR register.
- The CPSR register is copied to the IRQ_SPSR register.
- The I bit of CPSR is set (disabling IRQ).
- The processor mode is set to IRQ (mode 0x12). This mode swaps the SP
  and LR registers for the IRQ_SP and IRQ_LR registers (so the IRQ
  handler can have its own stack and know the return address)
- The processor begins executing from 0x00000018.

After the IRQ handler is executed, the following code can be used to
return to the previously executing code and restore the CPSR:

- SUBS PC,LR,#4

### FIQ

FIQ is designed for fast response time, and has priority over IRQ (note
that FIQ is not automatically disabled upon entry to IRQ). It can be
disabled by setting bit F (0x40) in the CPSR register.

When an FIQ is triggered, the following happens:

- The return address plus 4 is copied to the FIQ_LR register.
- The CPSR register is copied to the FIQ_SPSR register.
- The I and F bits of CPSR are set (disabling IRQ and FIQ).
- The processor mode is set to FIQ (mode 0x11). This mode swaps the R8 -
  R12, SP, and LR registers for the FIQ_R8 - FIQ_R12, FIQ_SP, and FIQ_LR
  registers (so the FIQ handler can have its own stack, know the return
  address, and have multiple free registers to improve interrupt
  latency)
- The processor begins executing from 0x0000001C. This is at the end of
  the vector table, so it is possible to put the entire FIQ handler at
  this location rather than using a branch instruction.

After the FIQ handler is executed, the following code can be used to
return to the previously executing code and restore the CPSR:

- SUBS PC,LR,#4

## TI-Nspire Interrupt Controller

From here on, IRQ and FIQ can be used interchangeably, so for
convenience' sake only IRQ will be mentioned. (An FIQ I/O register will
be the IRQ register plus 0x100 unless otherwise mentioned.)

### IRQ Numbers

On the TI-Nspire, there can be up to 32 different interrupt sources,
numbered 0-31. The following is a list of the currently known IRQ
numbers:

- IRQ 1 = <a href="/Memory-mapped_I/O_ports_on_CX#90020000_-_Serial_UART"
  class="wikilink" title="Serial UART">Serial UART</a>
- IRQ 3 =
  <a href="/Memory-mapped_I/O_ports_on_CX#90060000_-_Watchdog_timer"
  class="wikilink" title="Watchdog timer">Watchdog timer</a>
- IRQ 4 =
  <a href="/Memory-mapped_I/O_ports_on_CX#90090000_-_Real-Time_Clock_(RTC)"
  class="wikilink" title="RTC">RTC</a>
- IRQ 7 = <a
  href="/Memory-mapped_I/O_ports_on_CX#90000000_-_General_Purpose_I/O_(GPIO)"
  class="wikilink" title="GPIO">GPIO</a>
- IRQ 8 =
  <a href="/Memory-mapped_I/O_ports_on_CX#B0000000_-_USB_OTG_controller"
  class="wikilink" title="USB OTG">USB OTG</a>
- IRQ 9 =
  <a href="/Memory-mapped_I/O_ports_on_CX#B4000000_-_USB_HOST_controller"
  class="wikilink" title="USB HOST">USB HOST</a>
- IRQ 11 = <a
  href="/Memory-mapped_I/O_ports_on_CX#C4000000_-_Analog-to-Digital_Converter_(ADC)"
  class="wikilink" title="ADC">ADC</a>
- IRQ 13 = <a
  href="/Memory-mapped_I/O_ports_on_Classic#AC000000_-_SD_Host_Controller"
  class="wikilink" title="SD Host Controller">SD Host Controller</a>
- IRQ 14 = <a
  href="/Memory-mapped_I/O_ports_on_CX#900F0000_-_HDQ/1-Wire_and_LCD_contrast"
  class="wikilink" title="HDQ/1-Wire and LCD Contrast">HDQ/1-Wire and LCD
  Contrast</a>
- IRQ 15 =
  <a href="/Memory-mapped_I/O_ports_on_CX#900B0000_-_Power_management"
  class="wikilink" title="Power management">Power management</a>
- IRQ 16 =
  <a href="/Memory-mapped_I/O_ports_on_CX#900E0000_-_Keypad_controller"
  class="wikilink" title="Keypad">Keypad</a>
- IRQ 17 = <a href="/Memory-mapped_I/O_ports_on_CX#90010000_-_Fast_timer"
  class="wikilink" title="Fast timer">Fast timer</a>
- IRQ 18 =
  <a href="/Memory-mapped_I/O_ports_on_CX#900C0000_-_First_timer"
  class="wikilink" title="First timer">First timer</a>
- IRQ 19 =
  <a href="/Memory-mapped_I/O_ports_on_CX#900D0000_-_Second_timer"
  class="wikilink" title="Second timer">Second timer</a>
- IRQ 20 =
  <a href="/Memory-mapped_I/O_ports_on_CX#90050000_-_I2C_controller"
  class="wikilink" title="I2C">I2C</a>
- IRQ 21 =
  <a href="/Memory-mapped_I/O_ports_on_CX#C0000000_-_LCD_controller"
  class="wikilink" title="LCD controller">LCD controller</a>
- IRQ 22 = <a
  href="/Memory-mapped_I/O_ports_on_Classic#90100000_-_TI-84_Plus_link_port"
  class="wikilink" title="TI-84 Plus link port">TI-84 Plus link port</a>
  (?)

The following documentation is for TI-Nspire classic. The CX has a
[PL190](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.ddi0181e/index.html)
interrupt controller.

### IRQ Priority

Each of the 32 IRQ sources can be assigned a priority number from 0-7.
Lower numbers indicate higher priority. The array of priority values is
located at 0xDC000300-0xDC00037F. This array is used by both IRQ and
FIQ.

### IRQ Raw Status

The register at 0xDC000004 reads the raw interrupt status, which is a
bitfield where each bit corresponds to an interrupt source. Each bit
will be set if the source is requesting an interrupt (regardless of
whether that IRQ number is enabled or disabled) or reset if not.

### IRQ Sticky Status

The register at 0xDC000004 can also be configured bit-by-bit to read a
"sticky interrupt" status instead of the raw interrupt status. When an
interrupt source's raw status changes from 0 to 1, the sticky status
becomes set. Writing a 1 to the corresponding bit in the 0xDC000004
register will reset the sticky interrupt. The register at 0xDC000204
controls whether each bit in 0xDC000004 reads the raw status (bit=0) or
the sticky status (bit=1). This register is shared by IRQ and FIQ.

### IRQ Mask

A bitmask is used to enable or disable each interrupt source. If a bit
in the mask is 1, the corresponding IRQ will be enabled. If the bit is
0, the IRQ will be disabled. Writing to register 0xDC000008 will set the
bits in the mask that are set in the written value (thus enabling IRQ
sources), and writing to 0xDC00000C will reset the bits in the mask that
are set in the written value (disabling IRQ sources). Reading either of
these registers will return the IRQ mask.

### IRQ Masked Status

The register at 0xDC000000 reads the masked interrupt status, where each
bit is set if the corresponding bit in 0xDC000004 is set and the IRQ is
"masked in" (aka enabled). In this case, the IRQ is known as "active".
But simply having bits set in this register will not necessarily cause
an IRQ to be triggered, as shown below.

### IRQ Max Priority

The register at 0xDC00002C controls which priority level IRQs can be
triggered. Only IRQs with priority number less than the value in this
register can trigger an interrupt.

### IRQ Current Number

The register at 0xDC000020 reads the current IRQ number. This number is
the active IRQ of highest priority (i.e. lowest priority value). If
there are multiple active IRQs of the same priority, the lowest IRQ
number is chosen. If there are no active IRQs, this value will be 0. The
IRQ Current Number is what really forces IRQ priority, because it tells
the interrupt handler which IRQ to handle.

### Triggering IRQs

When an IRQ is triggered (which only happens if there is an active IRQ
with priority number less than IRQ Max Priority), imagine an internal
flag that is set to 1. The only way to reset this flag is by reading the
register at 0xDC000028 when there are no IRQs triggering the flag. The
flag will be set regardless of the status of the I bit in CPSR. The
interrupt will actually be taken once the I bit is reset (IRQ is enabled
again). But this internal flag will continue to be set even if you mask
out the IRQ source that caused it. Thus, it is possible that an
interrupt can be taken even if there are no active IRQs! Thus, it is
recommended to read register 0xDC000028 after disabling any IRQ types.

## Handling Interrupts

- Save clobbered registers to the IRQ stack.
- Read 0xDC000024.
  - The value read will be the IRQ Current Number.
  - As a side effect, the value of IRQ Max Priority is copied to
    0xDC000028 and the priority of IRQ Current Number is copied to IRQ
    Max Priority.
- If using sticky interrupts, write (1 \<\< IRQ Current Number) to
  0xDC000004.
- Read 0xDC000028.
  - This will reset the interrupt trigger flag if there are no other
    active interrupts.
  - Since IRQ Max Priority is now equal to the priority of IRQ Current
    Number, the current IRQ cannot re-trigger the interrupt flag.
  - The value read is the previous value of IRQ Max Priority. This can
    be used later.
- Use the IRQ Current Number to decide which IRQ source to handle.
- Acknowledge the interrupt at the source, which should reset the IRQ
  Raw Status bit.
- Do anything else needed for this interrupt.
- Update the IRQ Max Priority. Either write the value read from
  0xDC000028 to restore the previous value, or write 8 to allow
  interrupts of any priority.
- Restore clobbered registers from the stack and exit the interrupt
  handler.