---
title: NAND Memory Layout
permalink: /NAND_Memory_Layout/
parent: Operating System
layout: page
---

NAND pages are 528-bytes long (512 + 16-bytes OOB area) on TI-Nspire and
2112-bytes long (2048 + 64-bytes OOB) on TI-Nspire CX/CM/CX II.

## Layout on Classic/CX/CM

- pages 0000 to 001F (Nspire) or 0000 to 003F (CX/CM): written to
  /phoenix/manuf.dat at each boot.
  - Offset 000-003: 3C B0 6E 79
  - Offset 804-805: model ID (little-endian): 0C (Nspire CAS), 0D
    (Nspire Lab Cradle), 0E (Nspire), 0F (Nspire CX CAS), 10 (Nspire
    CX), 11 (Nspire CM CAS), 12 (Nspire CM)
  - Offset 806-807: unknown - 00 00 or 10 00
  - Offset 808-80F: optional default language (CX/CM), filled with FF if
    missing - ISO 639 supported language string padded with 00 (for
    exemple fr, en, ar, zh_CN for TI-Nspire CX-C or CM-C...)
  - Offset 818-81B: signature - 91 5F 9E 4C (CX/CM)
  - Offset 81C-81F: features (little-endian) - 0x05 (CM + CX Napoca),
    0x85 (CX CR/HW-J+), 0x185 (CX CR4/HW-W+)
  - Offset 820-823: default keypad - 4C 00 00 00 (CX/CM)
  - Offset 824-825: lcd width - 40 01 (CX/CM)
  - Offset 826-827: lcd height - F0 00 (CX/CM)
  - Offset 228-229: lcd bpp - 10 00 (CX/CM)
  - Offset 82A-82B: lcd color - 01 00 (CX/CM)
  - Offset 82C-82F: offset diags - 00 00 32 00 (CX/CM)
  - Offset 830-833: offset boot2 - 00 00 02 00 (CX/CM)
  - Offset 834-837: offset boot data - 00 00 2C 00 (CX/CM)
  - Offset 838-83B: offset file system - 00 00 40 00 (CX/CM)
  - Offset 83C-83F: config clock - 02 10 56 00 (CX/CM)
  - Offset 840-843: SDRAM config: 12 80 01 FC for 64MB (CX) or 11 80 01
    FE for 32MB (CM)
    - Offset 840: SDRAM size in MB - keep 6 lowest bytes - size is
      4\*2^((value/8)+(value%8))
  - Offset 844-847: lcd spi count - 02 00 00 00 (CX/CM)
  - Offset 848-887: lcd spi data filled with 0xFF - 06 00 00 00 5C 00 00
    00 30 00 00 00 04 00 00 00 (CX/CM)
  - Offset 888-889: lcd light min - 1A 01 (CX/CM)
  - Offset 88A-88B: lcd light max - CE 01 (CX/CM)
  - Offset 88C-88D: lcd light default - 6A 01 (CX/CM)
  - Offset 88E-88F: lcd light increment - 14 00 (CX/CM)
  - Offset 890-893: 0C 01 A2 18 (CX/CM)
  - Offset 894-923: display informations on the 12 elements of the
    splash screen (CX/CM): horizontal display offset + vertical display
    offset + width + height (2-bytes each) + data offset (4-bytes)
    - Offset 894-89F: Low Battery error icon \[diplayed unknown\]
    - Offset 8A0-8AB: Boot1 Recoverable Error icon \[displayed 8th\]
    - Offset 8AC-8B7: Send Diagnostics Software info icon \[displayed
      8th\]
    - Offset 8B8-8C3: Boot2 Recoverable Error icon \[displayed 8th\]
    - Offset 8C4-8CF: Unrecoverable Error icon \[displayed 8th\]
    - Offset 8D0-8DB: Progress Bar Background \[displayed 6th\]
    - Offset 8DC-8E7: Progress Bar \[displayed 7th\]
    - Offset 8E8-8F3: permanent element \#1 (background) \[displayed
      1st\]
    - Offset 8F4-8FF: permanent element \#2 (unused) \[displayed 2nd\]
    - Offset 900-90B: permanent element \#3 (unused) \[displayed 3th\]
    - Offset 90C-917: permanent element \#4 (unused) \[displayed 4th\]
    - Offset 918-923: permanent element \#5 (unused) \[displayed 5th\]
  - Offset 924-927: compressed splash screen data size
  - Offset 928-92B: uncompressed splash screen data size (0x0000FA40 on
    CX EVT, 0x00029CD0 on all CX/CM)
  - Offset 92C-92F: ? (0x00000756 on CX EVT, 0x000006D3 on all CX/CM)
  - Offset 930-???: compressed splash screen data (same compression
    format as the boot2)
  - Offset ???-???: TI-Certificate - fields present :
    - Production : 0x290 (0x100), 0x290 (0x100), 0x340 (0x1A4), 0x290
      (0x100), 0x340 (0x115), 0x290 (0x100), 0xFFFF0 (0)
    - Development : 0x290 (0x100), 0x290 (0x100), 0x340 (0x1A4), 0x240
      (0x80), 0x290 (0x100), 0x340 (0x115), 0x290 (0x100), 0xFFFF0 (0)
- pages 0020 to 0A7F (Nspire) or 0040 to 057F (CX/CM): boot2 image
- pages 0A80 to 0AFF (Nspire) or 0580 to 063F (CX/CM): "bootdata" (every
  time this is modified, the next available page is used; if all 128
  pages are in use, then the whole area is erased first)
  - Offset 00-03: Marker AA C6 8C 92
  - Offset 04-07: Downgrade protection: minimum OS version allowed as a
    4-bytes word (major-minor-lower1-lower2). Written during OS
    installation with the value found in the second field 8020 of the
    <a href="/OS_upgrade_files#Structure_OS_upgrade_file" class="wikilink"
    title="OS upgrade file">OS upgrade file</a>
  - Offset 08-0F: Hold the press-to-test status (word, word, long word)
    - Offset 08-09: press to test mode
      - 00 : none
      - 01 : 84+ mode (OS is going to prompt for a 84+ keypad if not
        installed on next reboot)
      - 02 : fully restricted (all listed features disabled) - LED
        flashes in green
      - 03 : partially restricted (no or some listed features
        disables) - LED flashes in orange
      - 04 : old mode not used any more, for OS 1.x/2.x - at that time
        there were only 2 features which could be disabled - meant that
        one feature had been selected but not botg - LED flashes in
        green+orange
      - 06 : for Netherlands/Europe (since OS 4.3) - no programming, and
        easily disabled through any USB transfer - LED flashes in orange
    - 0A-0B : clear PTT folders content on next reboot (1 during the 1st
      reboot after (re)enabling PTT - default 0)
    - 0C-0D : disabled features in PTT mode - default 0
      - Mode 3 :
        - bit 0 : geometry
        - bit 1 : drag&move in graphs
        - bit 2 : vectors
        - bit 3 : isPrime()
        - bit 4 : diff eq
        - bit 5 : ineq graphing
        - bit 6 : 3D graphing
        - bit 7 : rel/coniq graphing
        - bit 8 : trig
        - both bits 9+10 : logbase()
        - both bits 11+12 : poly and simult solving
      - Mode 2 : all 13 previous bits are 1
      - Mode 6 : all 13 previous bits are 0
    - 0E-0F : unkown - default 0 - sometimes 0x8000 in PTT mode
  - Offset 10-13: If nonzero, BOOT1 will attempt to run DIAGS by
    default; if zero, it will skip straight to BOOT2. (Either behavior
    can be overridden with the Esc+Menu+G key combination.)
  - Offset 14-1A: TI-84 Plus emulator 0A1 certificate field
  - Offset 1B-1E: TI-84 Plus emulator 041 certificate field
  - Offset 1F-61: TI-84 Plus emulator 0A2 certificate field
  - Offset 64-67: (OS 1.6+) Default LCD contrast (if not in range from
    0x76 to 0x8A, assumed to be 0x80)
- pages 0B00 to 0F7F (Nspire) or 0640 to 079F (CX) or 0640 to 7BF (CM):
  diags software
- pages 0F80 to 0FFF (Nspire) or 0780 to 07FF (CX): diags test results
- pages from 1000 (Nspire) or 0800 (CX) or 07C0 (CM): factory images or
  filesystem

### Factory images

At startup, boot2 checks the NAND flash for a pre-loaded factory image.
The format is a 32-byte header followed by the .tnc/.tno file contents:

- Offset 00-13: String "\*\*\*PRELOAD_IMAGE\*\*\*"
- Offset 14-17: 55 F0 01 55
- Offset 18-1B: (unknown)
- Offset 1C-1F: Size of image (in big-endian)

If boot2 finds this header, the user is prompted to press 'I' on the
keypad. After that, the image is copied to RAM before creating the
filesystem (The filesystem also starts at page 0x1000, so it cannot
co-exist with a factory image), and is installed the same as if it had
been received from the serial port.

## Layout on CX II

### Partitions

Partitions are aligned to erase block size (64 pages) and so the size
and offsets in the table below are given in blocks.

The OS is not stored in the file system, but separately in its own
partition.

| Name                   | Size   | Offset |
|------------------------|--------|--------|
| Manuf                  | 1      | 0      |
| Bootloader             | 4      | 1      |
| PTT Data               | 1      | 5      |
| ???                    | 1      | 6      |
| DevCert                | 1      | 7      |
| OS Loader              | 3      | 8      |
| Installer              | 8      | 11     |
| Other Installer        | 8      | 19     |
| OS Data (?)            | 2      | 27     |
| Diags                  | 5      | 29     |
| ?                      | ?      | ?      |
| OS file (weird header) | ?      | 36     |
| Logging                | 87 (?) | 114    |
| File System            | ?      | ?      |
| ?                      | ?      | ?      |

### Manuf Format

The Manuf on CX II uses the same fields format as seen in
<a href="/OS_upgrade_files" class="wikilink" title="OS upgrade files">OS
upgrade files</a>.


5000 : Top-level field


5100 - 2 : Product ID

5200 - 2 : Unknown

5300 - x : Language

5400 - 4 : Hardware flags. Bit 0 is 1 if the "CapTIvate" touchpad is
used.

5500 - x : Optional: If present, the bootrom runs this as code

5600 - 4 : Unknown

57y0 - 4 : Unknown (repeats with different values for y)

5500 - x : Contains pairs of addr/value to write

290 - 256 : 2048-bit Signature

290 - 256 : 2048-bit Signature (another one?)

340 - 420 : Public key (?)


270 - 1: ?

260 - 140: 1024-bit public key (?)

2A0 - 270: 2048-bit public key (?)

340 - 277 : Public key (?)


270 - 1: ?

2A0 - 270: 2048-bit public key (?)

290 - 256: 2048-bit Signature (yet another one?)

FFF0 - 0 : End

### Bootdata Format

Like for previous versions, every time this is modified, the next
available page is used. If all 128 pages are in use, the whole area is
erased first.


000 - 003 : Signature: 44 41 54 41 ('D' 'A' 'T' 'A')

004 - 007 : Boot type (0 = OS Loader, 1 = Installer, 2 = Diags)

008 - 00B : Which installer to boot (0 = Installer, 1 = Other installer)

00C - 00F : Minimum OS version (little endian, e.g. 00 00 00 05)

010 - 01B : Unknown, filled with 00

01C - 3FF : Blank, filled with FF

400 - 7FF : Blank, filled with 00


780 - 783 : Unknown, starts with 21