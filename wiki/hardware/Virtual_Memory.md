---
title: Virtual Memory
permalink: /Virtual_Memory/
parent: Hardware
layout: page
---

The <a href="/Hardware#cpu" class="wikilink" title="ARM926EJ-S">ARM926EJ-S</a>
<a href="/Hardware#mmu" class="wikilink" title="MMU">MMU</a> is used to map <a href="/Hardware#nor-rom" class="wikilink" title="NOR">NOR</a>,
<a href="/Hardware#sdram" class="wikilink" title="SDRAM">SDRAM</a> and <a href="/Memory-mapped_IO_ports" class="wikilink" title="peripherals">peripherals</a> in memory.

Virtual memory is split into 1MB sections.

The Translation Table Base Register (CP15 register c2) contains the
address of an array of 4096 words, each one describes a section.

The MMU allows both reads and writes on all mentioned sections, the
others will trigger an exception for any access.

---
1. TOC
{:toc}
---


## 0x00000000-0x000FFFFF (1MB) - Internal RAM

RAM, probably internal to
<a href="/Hardware#CPU" class="wikilink" title="ZEVIO ASIC">ZEVIO
ASIC</a>

80KB of RAM, the last 16KB of which is repeated 4 times for a total
address space of 128KB. This 128KB is repeated 8 times.

- Physical address : 0xA4000000-0xA40FFFFF
- not cached, not buffered

## 0x10000000-0x11EFFFFF (31MB) - <a href="/Hardware#SDRAM" class="wikilink" title="SDRAM">SDRAM</a>

First 31MB of RAM

- Physical address : 0x10000000-0x11EFFFFF
- write-back cache

## 0x11F00000-0x11FFFFFF (1MB) - <a href="/Hardware#SDRAM" class="wikilink" title="SDRAM">SDRAM</a>

Last 1MB of RAM

- Physical address : 0x11F00000-0x11FFFFFF
- not cached, not buffered

## 0x12000000-0x17FFFFFF (96MB)

Unknown

- Physical address : 0x12000000-0x17FFFFFF
- not cached, not buffered

## 0x18000000-0x180FFFFF (1MB) - alias of <a href="#0x10000000-0x11EFFFFF_(31MB)_-_SDRAM" class="wikilink" title="SDRAM">SDRAM</a>

- Physical address : 0x11E00000-0x11EFFFFF (alias of
  0x11E00000-0x11EFFFFF virtual address)
- write-back cache

## 0x8FF00000-0x901FFFFF (3MB) - <a href="/Memory-mapped_IO_ports" class="wikilink" title="Memory-mapped I/O ports">Memory-mapped I/O ports</a>

- Physical address : 0x8FF000000-0x901FFFFF
- not cached, not buffered

## 0xA0000000-0xA00FFFFF (1MB)

I/O ?

- Physical address : 0xA0000000-0xA00FFFFF
- not cached, not buffered

## 0xA4000000-0xA40FFFFF (1MB) - alias of <a href="#0x00000000-0x000FFFFF_(1MB)_-_Internal_RAM" class="wikilink" title="Internal RAM">Internal RAM</a>

- Physical address : 0xA4000000-0xA40FFFFF (alias of
  0x00000000-0x000FFFFF virtual address)
- not cached, not buffered

## 0xA9000000-0xA90FFFFF (1MB)

I/O ?

- Physical address : 0xA9000000-0xA90FFFFF
- not cached, not buffered

## 0xAC000000-0xAC0FFFFF (1MB) - <a href="/Memory-mapped_IO_ports" class="wikilink" title="Memory-mapped I/O ports">Memory-mapped I/O ports</a>

- Physical address : 0xAC000000-0xAC0FFFFF
- not cached, not buffered

## 0xB0000000-0xB00FFFFF (1MB) - <a href="/Memory-mapped_IO_ports" class="wikilink" title="Memory-mapped I/O ports">Memory-mapped I/O ports</a>

- Physical address : 0xB0000000-0xB00FFFFF
- not cached, not buffered

## 0xB4000000-0xB40FFFFF (1MB) - <a href="/Memory-mapped_IO_ports" class="wikilink" title="Memory-mapped I/O ports">Memory-mapped I/O ports</a>

- Physical address : 0xB4000000-0xB40FFFFF
- not cached, not buffered

## 0xB8000000-0xB80FFFFF (1MB) - <a href="/Memory-mapped_IO_ports" class="wikilink" title="Memory-mapped I/O ports">Memory-mapped I/O ports</a>

- Physical address : 0xB8000000-0xB80FFFFF
- not cached, not buffered

## 0xBC000000-0xBC0FFFFF (1MB) - <a href="/Memory-mapped_IO_ports" class="wikilink" title="Memory-mapped I/O ports">Memory-mapped I/O ports</a>

- Physical address : 0xBC000000-0xBC0FFFFF
- not cached, not buffered

## 0xC0000000-0xC00FFFFF (1MB) - <a href="/Memory-mapped_IO_ports" class="wikilink" title="Memory-mapped I/O ports">Memory-mapped I/O ports</a>

- Physical address : 0xC0000000-0xC00FFFFF
- not cached, not buffered

## 0xC4000000-0xC40FFFFF (1MB) - <a href="/Memory-mapped_IO_ports" class="wikilink" title="Memory-mapped I/O ports">Memory-mapped I/O ports</a>

- Physical address : 0xC4000000-0xC40FFFFF
- not cached, not buffered

## 0xC8000000-0xC80FFFFF (1MB)

I/O ?

- Physical address : 0xC8000000-0xC80FFFFF
- not cached, not buffered

## 0xCC000000-0xCC0FFFFF (1MB) - <a href="/Memory-mapped_IO_ports" class="wikilink" title="Memory-mapped I/O ports">Memory-mapped I/O ports</a>

- Physical address : 0xCC000000-0xCC0FFFFF
- not cached, not buffered

## 0xDC000000-0xDC0FFFFF (1MB) - <a href="/Memory-mapped_IO_ports" class="wikilink" title="Memory-mapped I/O ports">Memory-mapped I/O ports</a>

- Physical address : 0xDC000000-0xDC0FFFFF
- not cached, not buffered