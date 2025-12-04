---
title: Clock speed
permalink: /Clock_speed/
parent: Hardware
layout: page
---

This document describes the format for clock speed load values. The
clock speed can be set by writing to the first two registers at
<a href="/Memory-mapped_IO_ports#900B0000_-_Power_management"
class="wikilink"
title="Memory-mapped_I/O_ports#900B0000_-_Power_management">Memory-mapped_I/O_ports#900B0000_-_Power_management</a>.
Clock speeds are in units of MHz. Most of this information is derived
from Nover and nspire_emu source code.

## CX model

<div class="compact_table"></div>

| Bit | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 | .. | 30 | 31 |
|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
|  |  | Base/CPU value |  |  |  |  |  |  | 48MHz base |  |  |  | CPU/AHB value |  |  | Base value |  |  |  |  |  |  | Unknown |  |

|  |  |
|----|----|
| Base to CPU ratio (if 48MHz base) | 1 \<\< (Unknown) |
| CPU to AHB ratio (if 48MHz base) | 2 |
| Base clock speed (if 48MHz base) | 48 |
| Base to CPU ratio | (Base/CPU value)\*2 or 2 if 0 |
| CPU to AHB ratio | (CPU/AHB value) + 1 |
| Base clock speed | 6 \* (Base value) |
| CPU clock speed | (Base clock speed) / (Base to CPU ratio) |
| AHB clock speed | (CPU clock speed) / (CPU to AHB ratio) |
| APB clock speed | (AHB clock speed) / 2 |

The unknown quantity only ever seems to be either 1 or 2. It's probably
a multiplier.

## Classic model

<div class="compact_table"></div>

| Bit | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 |
|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
|  |  | Base/CPU value |  |  |  |  |  |  | 27MHz base |  |  |  | CPU/AHB value |  |  |  | Base value |  |  |  |  |

|                                  |                                          |
|----------------------------------|------------------------------------------|
| Base to CPU ratio                | (Base/CPU value) \* 2                    |
| CPU to AHB ratio                 | (CPU/AHB value) + 1                      |
| Base clock speed (if 27MHz base) | 27                                       |
| Base clock speed                 | 300 - 6 \* (Base value)                  |
| CPU clock speed                  | (Base clock speed) / (Base to CPU ratio) |
| AHB clock speed                  | (CPU clock speed) / (CPU to AHB ratio)   |
| APB clock speed                  | (AHB clock speed) / 2                    |