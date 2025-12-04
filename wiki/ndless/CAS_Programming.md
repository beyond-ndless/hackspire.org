---
title: CAS Programming
permalink: /CAS_Programming/
parent: Ndless
layout: page
---

The Nspire's standard OS ("Phoenix") contains CAS programming
capabilities *similar enough* to those of the TI-68k's standard OS
("AMS").

- On Phoenix, `Quantum` / `ESQ` are 2 bytes, while they're 1 byte on
  AMS.
- **Many entry points** described in the documentation of GCC4TI or TIFS
  **exist on Phoenix**; however, there's no equivalent of the AMS jump
  table on Phoenix.
- The higher-level functions (or their wrappers for execution as part of
  a BASIC program) are accessible through `primary_tag_list`, fairly
  similar to that of AMS. The addresses of lower-level functions and
  variables (e.g. `next_expression_index`, `push_quantum`, `top_estack`,
  `NG_control`, and many others) need to be found from those, on a
  per-OS basis...
- **We don't yet have any documentation about OS integration** (reading
  from variables and storing to variables; being accessible from a
  Calculator screen, which probably means exporting BASIC functions and
  embedding a Ndless program into a regular document).

A rough proof of concept for native EStack programming (on Phoenix
1.7.2741 CAS only, as of 2011/05/01) can be downloaded from
<http://www.ticalc.org/archives/files/fileinfo/437/43727.html>.