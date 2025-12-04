---
title: Timers
permalink: /Timers/
parent: Hardware
layout: page
---

From my tests it seems all 3 timer modules are equal (the fast timer,
the first timer, and the second timer), but the fast timer has a special
register set to 0x0.
This register is not in a standard sp804. It's at offset 0x80 from the
timer base address.
It is some sort of clock selection register.
If the first bit is set, it uses a ~10MHz clock.
The second bit overrules the first one.
If it is set, a 32kHz clock is used.
If neither bit is set, a 33MHz clock is used.
At this time, I have only tested values from 0 to 0xff and all single
bits, for the fast timer and the second timer.
And only on my hardware, a CX CAS HW-AA.
Firebird only supports the 32kHz clock at this time.

\- nspiredev500