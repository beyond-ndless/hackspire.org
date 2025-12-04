---
title: Internal Filesystem
permalink: /Internal_Filesystem/
parent: Operating System
layout: page
---

There are two layers involved with the internal filesystem. FlashFX Pro
does wear leveling and bad block management while Reliance is the
filesystem on top of it.

## FlashFX Pro

FlashFX Pro maps logical addresses to physical addresses on the NAND.
The physical space is divided into regions which are divided into units
(erase blocks), each unit divided up into blocks (each equal to page
size of the NAND). All integers are read as little endian.

The first block in a unit has a header describing the unit:

|      | 0x0              | 0x1 | 0x2                | 0x3 |
|------|------------------|-----|--------------------|-----|
| 0x00 | Signature        |     |                    |     |
| 0x04 | Signature        |     |                    |     |
| 0x08 | Signature        |     |                    |     |
| 0x0C | Signature        |     |                    |     |
| 0x10 | clientAddress    |     |                    |     |
| 0x14 | eraseCount       |     |                    |     |
| 0x18 | serialNumber     |     |                    |     |
| 0x1C | ulSequenceNumber |     |                    |     |
| 0x20 | lnuTotal         |     |                    |     |
| 0x24 | lnuTag           |     |                    |     |
| 0x28 | numSpareUnits    |     | blockSize          |     |
| 0x2C | lnuPerRegion     |     | partitionStartUnit |     |
| 0x30 | unitTotalBlocks  |     | unitClientBlocks   |     |
| 0x34 | unitDataBlocks   |     | checksum           |     |

The spare area of the NAND is 1/32th of the page size and holds extra
information

| 1 | 2 | 3 | 4 |
|----|----|----|----|
| Alloc info |  | ones-complement of byte 0 XOR byte 1 | error-correcting Hamming code of bytes 0-2 |
| seems to always be FF FF FF 0F for used pages, FF FF FF FF for unused |  |  |  |
| error-correcting code of second half of page data |  |  |  |
| error-correcting code of first half of page data |  |  |  |

### Checksum

The checksum in the unit header is calculated by adding all the bytes in
the header mod 2^16.

`uint16_t checksum(void *_ptr, size_t size) {`
`   uint16_t sum = 0;`
`   uint8_t *ptr = _ptr;`

`   while (size--) {`
`       sum += *ptr++;`
`   }`

`   return sum;`
`}`

### Allocation information

This contains the status and logical address of the block. A unit header
will have a magic signature (0x48E2).

#### Status

Bits 12-15 indicate the status of the block.

Probably irrelevant for this version of FlashFX (always 0x4).

#### Logical address

Mask with 0x0FFF to get address. 0x8E2 is a magic number for unit
headers.

### Retrieving a page

The best algorithm for retrieving page data from a FlashFX partition is
to first create a lookup table resolving logical page numbers to
physical page numbers and use that to retrieve the page.

To build the table, first iterate through every unit. Your table needs
to hold two pieces of data: the physical page number and a sequence
number. The key should be the logical page number. The table will be
unitClientBlocks \* lnuTotal entries long.

For each unit, read the header, then iterate through the blocks in the
unit. You need to gather two pieces of information for each block in the
unit: the logical page address of that block and the sequence number of
the unit it belongs to. The logical page number is equal to the unit's
(clientAddress / blockSize) + the block's logical address.

If the page number is larger than the one already existing in the lookup
table and if the sequence number is greater, the entry for that logical
page number is set to the current block. Basically, the latest version
of the logical page is the one with the greatest sequence number, then
the greatest physical number.

Use the lookup table to map logical page addresses to physical page
addresses.

If a lookup table entry is empty, it means no physical page has been
allocated to it and the data at that logical address is a blank page (a
page filled with 0xFF s).

## Reliance

A fairly stock standard filesystem. All pointers refer to block numbers
(block numbers start at zero). Offset = blockNumber \* blockSize.
Integers are little endian.

### MAST header

The header appears 0x40 bytes from the start of the partition (the
padding is apparently used for some boot code on some platforms). This
is the master header from where all reading/parsing should begin.

Reliance keeps two copies of the META header. A counter in the META
header determines which is the more recent copy.

|      | 0x0                        | 0x1 | 0x2            | 0x3 |
|------|----------------------------|-----|----------------|-----|
| 0x00 | 'M'                        | 'A' | 'S'            | 'T' |
| 0x04 | Build format version       |     | Layout version |     |
| 0x08 | Block size                 |     |                |     |
| 0x0C | Number of blocks           |     |                |     |
| 0x10 | 2 x Pointer to META header |     |                |     |
| 0x14 |                            |     |                |     |
| 0x18 | Volume creation date       |     |                |     |
| 0x1C |                            |     |                |     |
| 0x20 | Number of IMAP blocks      |     |                |     |
| 0x24 | Reserved                   |     |                |     |
| 0x28 |                            |     |                |     |
| 0x2C | Checksum                   |     |                |     |

### IMAP header

A bitmap to keep track of used and unused blocks. TODO. Unneeded for
reading.

### META header

|      | 0x0                       | 0x1 | 0x2 | 0x3 |
|------|---------------------------|-----|-----|-----|
| 0x00 | 'M'                       | 'E' | 'T' | 'A' |
| 0x04 | Counter                   |     |     |     |
| 0x08 | Index block               |     |     |     |
| 0x0C | Next free block           |     |     |     |
| 0x10 | 2 x Pointer to IMAP block |     |     |     |
| 0x14 |                           |     |     |     |
| 0x18 | Number of free blocks     |     |     |     |
| 0x1C | Number of used blocks     |     |     |     |
| 0x20 | Number of bad blocks      |     |     |     |
| 0x24 | Unknown                   |     |     |     |
| 0x28 | Reserved                  |     |     |     |
| 0x2C |                           |     |     |     |
| 0x30 |                           |     |     |     |
| 0x34 |                           |     |     |     |
| 0x38 |                           |     |     |     |
| 0x3C | Checksum                  |     |     |     |

### INOD header

This describes an inode.

Timestamps are in milliseconds.

|      | 0x0                 | 0x1 | 0x2                      | 0x3 |
|------|---------------------|-----|--------------------------|-----|
| 0x00 | 'I'                 | 'N' | 'O'                      | 'D' |
| 0x04 | Index               |     |                          |     |
| 0x08 | Data size           |     |                          |     |
| 0x0C |                     |     |                          |     |
| 0x10 | Creation timestamp  |     |                          |     |
| 0x14 |                     |     |                          |     |
| 0x18 | Modified timestamp  |     |                          |     |
| 0x1C |                     |     |                          |     |
| 0x20 | Accessed timestamp  |     |                          |     |
| 0x24 |                     |     |                          |     |
| 0x28 | Attributes          |     | Number of linked entries |     |
| 0x2C | Reserved            |     |                          |     |
| 0x30 |                     |     |                          |     |
| 0x34 |                     |     |                          |     |
| 0x38 |                     |     |                          |     |
| 0x3C |                     |     |                          |     |
| 0x40 | Data... (see below) |     |                          |     |

The lower two bits of the attribute determine what the data means.

|  |  |
|----|----|
| 0 | Contains data directly (up to blockSize - 0x40 bytes) |
| 1 | Contains a list of pointers to data blocks (up to \[(blockSize - 0x40) / 4\] \* blockSize bytes) |
| 2 | Contains a list of pointers to INDI blocks (up to \[(blockSize - 0x40) / 4\] \* \[(blockSize - 0x04) / 4\] \* blockSize bytes) |
| 3 | Contains a list of pointers to DBLI blocks (up to \[(blockSize - 0x40) / 4\] \* \[(blockSize - 0x04) / 4\] \* \[(blockSize - 0x04) / 4\] \* blockSize bytes) |

When you're reading data from following pointers, stop reading once the
desired number of bytes have been read (i.e. equal to data size).

#### INDI and DBLI blocks

Both blocks begin with their respective 4 byte magic byte sequences. For
INDI blocks, the rest of the block contains a list of pointers to data
blocks. For DBLI blocks, the rest of the block contains a list of
pointers to INDI blocks.

### INDX data

The first inode will contain index data. Index data are pointers to
inodes that hold file and directory information.

|      | 0x0             | 0x1 | 0x2 | 0x3 |
|------|-----------------|-----|-----|-----|
| 0x00 | 'I'             | 'N' | 'D' | 'X' |
| 0x04 | Index entry \#1 |     |     |     |
| 0x08 | Index entry \#2 |     |     |     |
| 0x0C | ...             |     |     |     |

#### Special indexes

|     |                              |
|-----|------------------------------|
| 0   | Magic number ('INDX')        |
| 1   | Always the index file        |
| 2   | Index for the root directory |
| 3   | Index for bad file?          |
| 4   | Index for volume label?      |

### Directory listing

A directory listing contains a list of the following structure

|  | 0x0 | 0x1 | 0x2 | 0x3 | 0x4 | 0x5 | 0x6 | 0x7 | 0x8 | 0x9 | 0xA | 0xB |
|----|----|----|----|----|----|----|----|----|----|----|----|----|
| 0x00 | 0x80 | Checksum? |  | Length of the entry (n) |  |  |  | Length of name (m) |  | Attributes | Reserved | Pointer to inode |
| 0x0C | Array of 16bit characters (filename) |  |  |  |  |  |  |  |  |  |  |  |
| 0x0C + m | Padding |  |  |  |  |  |  |  |  |  |  |  |
| 0x0C + m + n | Next entry |  |  |  |  |  |  |  |  |  |  |  |

The attributes meaning is described below

| Mask | Description     |
|------|-----------------|
| 0x1  | Is in use?      |
| 0x2  | Is a directory? |

## Resources

[Goplat's post on
Omnimaga](http://www.omnimaga.org/index.php?topic=7386.0;msg=140209)

[Patent for
FlashFX](https://www.google.com/patents/US5860082?pg=PA13&dq=5,860,082&hl=en&sa=X&ei=Kq2pUbq-DYWE9QSilYHwCA&ved=0CDQQ6AEwAA)