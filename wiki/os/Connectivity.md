---
title: Connectivity
permalink: /Connectivity/
parent: Operating System
layout: page
---

## Loopback transfers

When the TI-Nspire is connected to a computer, but Computer Link
Software is *not* running, sending the Operating System from the menu of
the document management screen make it act as if it was sending and
receiving the OS at the same time. The storage memory consumption
increases during the loopback transfer, as it does for a real OS
upgrade. The same behavior appears when trying to send documents from
there contextual menu. The received documents are copied with a new name
that has numerical suffix. Loopback transfers don't seem to be faster
than real ones, probably because nearly the whole connectivity stack is
used, and the physical transfer is not what limits the speed.

No data is exchanged with the computer during a loopback transfer (USB
analyzers don't report anything, and it works even when the TI-Nspire
driver is not installed).

## USB

### USB descriptors

Here is the interesting part (i.e. different from the TI-84 Plus and
Titanium) of the USB descriptors advertised by the TI-NSpire in standard
(not TI-84 Plus emulation) mode.

```
Device Descriptor:
   idProduct   E012h (reminder: E001 : Silverlink, E004: Titanium, E008: TI-84 Plus)
   bcdDevice   0100h // 1.00
   iProduct          // "TI-Nspire(tm) Handheld"
```

```
Configuration Descriptor:
   bmAttributes      80h // Bus Powered (Titanium: Self Powered Remote Wakeup)
   bMaxPower     32h // 100 mA (Titanium: 0 mA)
```

```
OTG Descriptor // As on Titanium and TI-84 Plus
```

Interface descriptor: Vendor Specific class, 1 Bulk IN endpoint, 2 Bulk OUT endpoints
(was 1 Bulk IN and 1 Bulk OUT on Titanium/TI-84 Plus)

When a TI-84 Plus is <a href="/TI-84_Plus_Emulation" class="wikilink"
title="emulated">emulated</a> by the TI-Nspire, the descriptors are the
same as a real TI-84 Plus except:

```
Device Descriptor:
   idProduct   E004h // Titanium!
   bcdDevice   0200h // 2.00 (Real TI-84 Plus: 1.10)
   iProduct          // "TI-84 Plus Silver Edition (Emulation)"
```
```
Configuration Descriptor: // Same as the real TI-84 Plus
   bmAttributes      C0h // Self Powered
   bMaxPower     00h // 0 mA
```
```
Interface and enpoint descriptors: same as the TI-84 Plus (1 Bulk IN , 1 Bulk OUT)
```