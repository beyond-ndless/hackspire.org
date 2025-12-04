---
title: USB Protocol
permalink: /USB_Protocol/
parent: Operating System
layout: page
---

This article describes the protocol used for USB communication between
the computer and the TI-Nspire. It has been documented from TI's
official linking software *Computer Link Software*, an analysis of USB
captures of data traffic, and results of tests with a custom host
implementation. This documentation may be used to build a full-featured
third-party implementation of the computer program, to use features
provided by the link not available from Computer Link Software's GUI, to
indirectly gather additional information on the TI-Nspire hardware and
software, and to discover exploitable flaws in the TI-Nspire OS's
implementation of the protocol. An implementation of the protocol is
available in [TiLP](http://lpg.ticalc.org/prj_tilp/).

The documentation is currently incomplete, sometimes vague or too
restrictive, and may be wrong on some points. Feel free to contribute to
its enhancement and refinement. Interrogations and information based on
assumptions which needs to be confirmed by a deeper analysis, more tests
with Computer Link Software or tests with a third-party host
implementation are <span style="color: IndianRed">formatted in indian
red</span>. Once enough tests have been made to validate or correct
these parts, please edit them and remove the highlighting.

The descriptions of the packets of the protocol let sometimes appear
hard-coded value, for which it is not clear whether the corresponding
field is constant or may vary under certain conditions. You may edit
these descriptions to document additional logic found for them.
Borderline cases have not all been tested: most of the time the
TI-Nspire's implementation of the protocol handles them and make use of
error codes. These error codes have not all been documented yet.

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

## USB descriptors

Here is the interesting part (i.e. different from the TI-84 Plus and
Titanium) of the USB descriptors advertised by the TI-NSpire in standard
(not TI-84 Plus emulation) mode.

```
Device Descriptor:
   idProduct   E012h (reminder: E001 : Silverlink, E004: Titanium, E008: TI-84 Plus, E022: Nspire CX II) 
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

```
Interface descriptor: Vendor Specific class, 1 Bulk IN endpoint, 2 Bulk OUT endpoints (64 bytes)
(was 1 Bulk IN and 1 Bulk OUT on Titanium/TI-84 Plus, 64 bytes)
```

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

## Introduction

We will call **host** the participant of the communication which is the
USB host, i.e. the computer for a transfer between a computer and a
TI-Nspire. <span style="color: IndianRed">The host of a transfer between
two TI-Nspires is probably also the USB host, determined with the Host
Negotiation Protocol of [USB
On-The-Go](http://en.wikipedia.org/wiki/USB_OTG)</span>. The
communication model of the TI-Nspire seems to allow multiple devices to
communicate simultaneously with the same host, through not yet released
[USB
hubs](http://www.ti-nspire.com/tools/nspire/resources/connect_class.html),
similar to
[TI-Navigator](http://education.ti.com/educationportal/sites/US/productDetail/us_ti_navigator.html)
available for former calculator models. The TI-Nspire and the computer
thus have each a dynamically assigned **address** used to identify the
source and destination of a packet. A message sent from a source
host/device to a destination host/device will be called a **packet** in
the rest of this documentation. It is actually a protocol packet which
may be transparently split into several USB packets by the USB device
and host stacks if its length is greater or equal than the maximum USB
packet size supported by the TI-Nspire (64 bytes). It seems that a
protocol packet does not need to end at an USB packet boundary, but
because of the sequencial nature of the exchanges, this feature is
useless.

Packets are exchanged between **services** of the host and the device. A
service is a software module which produce and interpret a subset of
packet types. A service is identified by a two bytes identifier. Each
packet contains both the source and destination service identifiers. A
service of a participant sending a packet choose the target service it
wants to talk with. Service IDs are similar to TCP ports, except that
multiple source and destination services are used during the same
session (more specifically for acknowledgment) as we will see.

We will use further in the documentation notations of the form:
`6400:8001->6401:4060` where 6400 is here the source address, 8001 the
source service, 6401 the destination address and 4060 the destination
service.

After a connection reset (described further) and once a device has been
declared to the host, transfers are initiated by the host and have this
form (each line represents a packet):

`Host: Request`
`Device: Acknowledgment`
`Device: Response`
`Host: Acknowledgment`

What could be called a "functional request" (for example "transfer a
file") may use several exchanges of this type. The host and the device
have stateful sessions and can keep track of the current state between
these exchanges (for example when listing the content of a directory,
which requires one pair of request/response for each file, the device
keeps track of the current file).

## Packet format

Packets send by a host or a device have the same format. A packet has a
16 bytes header and may have an optional data part of variable length.

`54 FD SA SA SS SS DA DA DS DS DC DC SZ AK SQ CK [data part] (data less than 255 bytes in size)`
`54 FD SA SA SS SS DA DA DS DS DC DC FF AK SQ CK SZ SZ SZ SZ [data part] (data more than 254 bytes in size)`

The different fields of the header part are:

- `54 FD`: constant
- `SA SA`: source address
- `SS SS`: source service ID
- `DA DA`: destination address
- `DS DS`: destination service ID
- `DC DC`: checksum of the data part
- `SZ`: size of the data part
- `AK`: used for acknowledgment packets, 00 for other packets.
- `SQ`: sequence number
- `CK`: checksum of the header: the sum of the preceding bytes modulo
  256

Here is additional information for the fields not obvious and not
described before.

### Service identifiers

The host which always initiate the transfers (except address assignment
during described further) chooses the service ID it uses for standard
(not acknowledgment) packets. It will be the source service ID that
appears in the packets sent to the device, and the destination service
ID of the responses of the device. Any non-standard service ID may be
chosen. Computer link Software uses service ID greater or equal than
0x8001. The service ID must be chosen before starting to exchange
packets with a new service of the device. When using another service of
the device, the host must disconnect itself from the previous service
(this process is described further), and choose a new local service ID
for the subsequent packets for the new target service. Each time a new
host service ID is chosen, it *must* be different that the previous
ones. The device won't answer to packets with a source service ID which
matches a host service which has previously asked to disconnect itself.
The same service IDs can be reused only once the connection with the
device is reset. At each generation Computer Link Software increments by
one the service ID to ensures unique generation, but this is actually
not mandatory.

The identifiers (in hexadecimal) of the currently known standard
services on the device side are:

- `00D3`: nACK packet - destination service not available
- `00FE`, `00FF`: packet reception acknowledgment
- `4001`: null <span style="color: IndianRed">(?)</span>
- `4002`: echo
- `4003`: device address request
- `4004`: service tuning <span style="color: IndianRed">(?)</span>
- `4020`: device information
- `4021`: screen capture
- `4022`: event <span style="color: IndianRed">(?)</span>
- `4023`: shutdown <span style="color: IndianRed">(?)</span>
- `4024`: screen capture with RLE
- `4041`: service activity <span style="color: IndianRed">(?)</span>
- `4042`: TE_MLPDEV <span style="color: IndianRed">(?)</span>
- `4043`: TE_RPC <span style="color: IndianRed">(?)</span> (not yet
  enabled in the OS)
- `4050`: login
- `4051`: message <span style="color: IndianRed">(?)</span>
- `4054`: hub connection <span style="color: IndianRed">(?)</span>
- `4060`: file management (sync service)
- `4080`: OS installation
- `5000`: EXTECHO <span style="color: IndianRed">(?)</span>
- <span style="color: IndianRed">To be completed</span>

The identifiers of the services on the host side are:

- `00FE`, `00FF`: packet reception acknowledgment
- `4003`: device address assignment
- `40DE`: service disconnect
- <span style="color: IndianRed">To be completed</span>

### Sequence number

<span style="color: IndianRed">Sequence numbers are solely used for
proper acknowledgment of packets</span>. Each participant manages itself
the generation of the sequence numbers for the packets it sends (a
communication between a computer and a TI-Nspire brings into play two
sequences). Each packet (except acknowledgment packets, see further, and
except in the case of an overflow) has it own sequence number. The
number is incremented each time a packet has been acknowledged by other
side (<span style="color: IndianRed">do they really need to be
consecutive?</span>). The generation is common for all the services of a
participant, i.e. two consecutive packets sent by/to different services
will have consecutive sequence numbers. The sequence number has to be
reinitialized to 1 after an address assignment; any other value will
make transfer fail. The sequence number which follows 255 is 1, 0 is
never used (<span style="color: IndianRed">is there really a reason for
this or does it still work with 0?</span>).

### Acknowledgment

An acknowledgment packet from the TI-Nspire acknowledges only the
*reception* of another packet, *not* the fact that the packet
acknowledged has correctly been taken into account by its destination
service. The TI-Nspire may even acknowledge packets with an incorrect
header, for example with a bad header or data checksum, or with a
duplicate sequence number.

Whether sent by the host or a device, an acknowledgment packet has
**0x00FF** as source service ID, **except** :

- when the sequence number of the acknowledge packet is zero, in which
  case the service ID **0x00FE** is used
- when the packet acknowledged contained an invalid destination service
  ID, in which case the service ID **0x00D3**, which should be
  considered as an error

The destination service ID must be the source service ID of the
previously received packet the participant acknowledges. The sequence
number must be the same as the one of the packet acknowledged. The field
`AK` of the header must contain **0x0A**. The 2 byte-long data part must
contain the destination service ID of the packet acknowledged (i.e. a
local service ID). Here is an example of an acknowledgment packet sent
by a TI-Nspire to a computer:

`6401:00FF->6400:8001 AK=0A SQ=02 `
`data part: 40 60`

Here the device acknowledges the reception of a packet sent by the
service 8001 of the host, which had 02 as sequence number, and was sent
to the service 4060 of the TI-Nspire.

The device replies to requests with an invalid destination service ID
with packets similar to acknowledgement packets, but with a special
source service ID : **0x00D3**.

### Data part checksum

The 2 byte-long checksum is more or less a CRC checksum. The checksum is
00 00 if there is no data part in the packet. Here is an implementation
in [Python](http://www.python.org) of the algorithm:

```
def checksum(data):
    acc = 0
    for byte in data:
        first = (ord(byte) << 8) | acc >> 8
        acc = acc & 0xFF
         second = (((acc & 0xF) << 4) ^ acc) << 8
       third = second >> 5
        acc = third >> 7
        acc = (acc ^ first ^ second ^ third) & 0xFFFF
    return acc
```
```
# A few examples
assert checksum('\x00\xFF') == 0xFF00
assert checksum('\x05\x00\x00') == 0x57AD
assert checksum('\x20\x00\x00\x00\x00\x00\x00\x00\x05\x01\x00') == 0xA095
```

### Data part

The format and interpretation of the data part depends on the source and
destination services for the packet, and on the current state of the
exchange:

\- multiple-bytes integers that appear in the data part (service
identifiers in ack packets for instance) are in big endian (most
significant byte first),

\- strings are terminated with a null byte (the examples shown in this
documentation will always make the null byte appear to prevent from
forgetting it). Sizes are expressed in bytes,

\- blocks of pure data always start with a single ID byte (the command
byte which is service dependent). When the ID byte is provided in a
Request packet, the ID byte is repeated in the response Packet as first
byte (even if data has to be split into several packets).

Common packets with a data part are those which indicates the success or
failure of the processing initiated by the previous request packet (or
response packet in the case of a processing by the host). These packets
should not be confused with packets
<a href="#Acknowledgment" class="wikilink"
title="acknowledging the reception">acknowledging the
<em>reception</em></a>: for example a request packet can be received
successfully, but its data part can contain incorrect data - the device
would here return an ACK packet and then a *failure* packet. These
**success** and **failure** packets are 2 bytes long, starts with
**0xFF** and are followed by **0x00** in case of success, or an error
code in case of failure. The error code is not specific to a type of
request (<span style="color: IndianRed">but is it specific to a
service?</span>), some codes are reused when they have the same meaning
(for example in the case of the
<a href="#File_management" class="wikilink"
title="file management service">file management service</a>).

Note: if data to be sent/received is bigger than packet size, data has
to be split into several packets.

## Services

Now that we have seen the common parts of the protocol, let's review the
functions offered by each service. The descriptions of the various IN
(from the device to the host) and OUT (from the host to the device)
packets will be presented in this format (random values are used here):

`OUT: 6400:8001->6401:4060`
`FF 00`

In this example the service 0x8001 of the host whose address is 0x6400
sends to the service 0x4060 of the device with address 0x6401 the 2
byte-long sequence `FF 00` in the data part of the packet (which is a
<a href="#Data_part" class="wikilink" title="success packet">success
packet</a>). The examples in the following sections will most of time
use for common packets 0x6400 as host address, 0x6401 as device address
and 0x8001 as generated host service ID. Values which are *not* example
values will appear in bold. Multi-bytes fields in the data part of some
packets which are not constant values will be surrounded with square
brackets. If the field has a constant length, its length will appear
just after the opening bracket. Fields with no specified length have a
variable length.

`[2 device address]: a 2 byte-long field containing the device address`
`[directory path] 00: a field of variable length containing the path as a null-terminated string`

Mandatory IN and OUT acknowledgment packets will never be shown, only
*request* and *response* packets will be described. Exchanges which
don't fit in this type of description will be presented with enough
details to fully understand their special features.

### Address assignment

Before starting any communication with the device, whether the host
program has already been started and the device has just been plugged
in, or the host program has just been started and the device was already
plugged, the host must first:

1.  reset the connection
2.  assign an address to the device

The connection reset is mandatory because it forces the device to forget
the <a href="#Service_identifiers" class="wikilink"
title="identifiers of the host services">identifiers of the host
services</a> and <a href="#Sequence_number" class="wikilink"
title="sequence numbers">sequence numbers</a> used in the previous
communication. Not resetting the connection may cause the reuse of these
unique elements, and thus the absence of response to the host requests.
A connection reset is triggered by issuing the
[IOCTL_INTERNAL_USB_RESET_PORT](http://msdn2.microsoft.com/en-us/library/ms793191.aspx)
kernel-mode I/O request on Microsoft Windows.

Just after the reset, the device will spontaneously send an **address
request** packet, with a sequence number set to 1:

`IN: `**`0000:4003->0000:4003`**

The host must response with an **address assignment** packet. The
sequence number of this paquet doesn't matter, Computer Link Software
uses 1.

`OUT: 6400:`**`4003`**`->6401:`**`4003`**
`[2 device address] FF 00`

In this example, the address which appears in the data part of the
packet would be 6401 (the same as the destination address in the
header). Note that the host should use in the source address field of
the header the address it has chosen for itself, and keep it for all the
subsequent packets. Computer Link Software always choose 6400 as host
address and assigns the address 6401 to the device
(<span style="color: IndianRed">what if more than one TI-Nspire is
plugged to the computer?</span>). The two bytes `FF 00` mean that the
host accepts the address request.

In the special case of address assignment, the device will *not* send an
acknowledgment packet for the address assignment packet.

The host can now use whatever device service he would like to. It has to
choose before a <a href="#Service_identifiers" class="wikilink"
title="service identifier">service identifier</a>, and will do this
before starting to use any service. Remember that an address assignment
forces the sequence numbers of both host and device to reinitialize to 1
for the next packet sent by each.

### Login request

Starting at OS 1.2.xxxx, the NSpire attempts to connect to the LOGIN
service just after address assignment:

`IN: 6401:8013->6400:`**`4050`**
` 02 00 00 00 00 00`

and the computer simply replies that service is not available:

`OUT: 6400:`**`00d3`**`->6401:8013`
` 40 50`

From OS 1.4.xxxx, the Nspire adds disconnection from service. So, we
have more packets:

`IN: 6401:40de->6400:4050`
` 80 00`

`OUT: 6400:00ff->6401:8000`
` 40 50`

Please note that hand-held request LOGIN connection three seconds after
device reset. If your program is designed as one thread listening on all
ports (sequential, like TiLP), you have to take this into account.
Otherwise, you will get spurious LOGIN request at any time during
transfer.

If your program is designed as a real daemon launching one thread per
port (parallel), you don't have to be bothered by this.

### Device Information

To request Device Information:

`OUT: XX `

where XX is the command byte; current values are:

```
01 -> serial number
02 -> device name
03 -> list of supported file extensions
```

The response to 01 is:

`IN: 01 raw data`

Raw data contents:

```
@00d (8 bytes): FLASH free
@08d (8 bytes): FLASH physical
@16d (8 bytes): RAM free
@24d (8 bytes): RAM physical
--
@32d (1 byte): battery level - FF=unknown, 00=powered, 01=low, 7F=OK
@33d (2 bytes): 1 if charging, else 0
@35d (1 byte): clock speed in Mhz
--
@36d (4 bytes): product version (X.Y.ZZZZ) stored as XX YY ZZ ZZ
@40d (4 bytes): same for boot1 version
@44d (4 bytes): same for boot2 version
--
@48d (4 bytes): hardware version - 1 if TI-Nspire CAS, 0 if TI-Nspire
@52d (2 bytes): runLevel - 1 for boot code (for example in recovery mode), 2 for OS
--
@54d (2 bytes): LCD x
@56d (2 bytes): LCD y
@58d (2 bytes): LCD w
@60d (2 bytes): LCD h
@62d (1 byte): 4 bits per pixel
@63d (1 byte): sample model (always 0 which means [packed](http://en.wikipedia.org/wiki/Packed_pixel))
--
@64d (1 byte): device - 0E if TI-Nspire CAS, 1E if TI-Nspire
--
@65d (17 bytes): a part of the Electronic Id (16 digits) stored as a null-terminated string (the first two digits and the last 8 ones are missing)
@82d (27 bytes): the full Electronic Id (26 digits) stored as a null-terminated string
```

The response to 02 is:

`IN: 02 T I - N S p i r e 00`

The response to 03 is:

`IN: 03 . t n s 00 . t n o 00`

The first string is the file extension and the second one is the OS
extension.

### Service disconnection

Nothing special needs to be sent by the host to connect to a service of
a device before using it. Simply using the correct destination service
ID in the request packets it sends is enough.

Once the host has started to use a device service, it needs to
disconnect from the current service before using a different service.
The request is:

```
OUT: 6400:`**`40DE`**`->6401:[id of the device service from which to disconnect]
[id of the host service which was used until then to communicate with the device service]
```

For example if the host uses the device service 4060 with the generated
host service ID 8001 and wants to disconnect from it:

```
(... The host uses the service. Some requests with the header:
  OUT: 6400:8001->6401:4060
...)
OUT: 6400:40DE->6401:4060
8001
```

Note that Computer Link Software disconnects very often from the
services it uses, even between requests to the same service (for example
between a directory listing and a file attributes request to the file
management service). This is not necessary.

### File management

The data part of the packets exchanged with the file management service
of the TI-Nspire contains either a success or failure code (see the
section on the
<a href="#Data_part" class="wikilink" title="data part">data part</a>),
or a structure specific to each command. For the latter, the first byte
is the command byte; current values are:

```
03: put file
04: ok to put/get file
05: file contents
07: get file
0a: create folder
0b: delete folder
0d: start dirlist
0e: get next dirlist entry
0f: stop dirlist
10: directory list entry
20: get directory or file attributes
```

We will use the names given by Computer Link Software to the different
commands.

Note: NSpire uses UTF-8 to encode directory and file names.
<span style="color: IndianRed">Is there any size limit for
directory/file names?</span>

#### Directory listing

Listing the content of a directory requires multiple sets of requests
and responses. A typical sequence would be:

```
OUT: DirEnumInit
IN: Success/Failure
OUT: DirEnumNext
IN: Directory or file info
OUT: DirEnumNext
...
IN: No more directory or file
OUT: DirEnumDone
IN: Success
```

Only the data part of the different packets will be shown in the
following descriptions.

##### DirEnumInit

Initiates a directory listing.

`OUT: 0D [>=8 directory name] 00`

*Directory name* is the name of the directory to list, with or without
leading and/or trailing slash. It can be "/" get a list of the different
directories on the TI-Nspire. The TI-Nspire only supports one level of
directories. The name must be at least 8 characters long, padded with
null characters if needed. The response is one of:

```
IN:
- FF 00: success
- FF 0A: the directory doesn't exist
- FF 0F: invalid directory name
```
##### DirEnumNext

Get information about the next file in the directory being listed, if
there are any more.

`OUT: 0E`

If information about the next file or directory can be returned, the
response is:

`IN: 10 00 [1 data part size - 3] [name of the directory or the file] 00 [4 size of the file] [4 date?] [1 directory flag] 00`

File names have the suffix *.tns*. "*data part size - 3*" is the size of
the file information structure without the 3 header bytes. The size of a
directory is always 0. The meaning of the date is currently not very
clear (<span style="color: IndianRed">more to come</span>). "*directory
flag*" is 00 for a file, and 01 for a directory.

Else the response is `FF XX`, where `XX` is 11 if there are no more file
or directory in the list, or 10 if we are not currently listing a
directory or if the previous *DirEnumInit* failed.

##### DirEnumDone

<span style="color: IndianRed">Probably needed before trying to list
another directory.</span>

```
OUT: 0F
IN: FF 00
```

#### Directory and file attributes

This command is returns the attributes of a directory or a file:

```
OUT: 20 01 [>=8 directory or file path] 00
IN: 20 [4 size, 0 for a directory] [4 date] [00 if file, 01 if directory] [1 flags(?): always 00]
```

The directory or file path must be at least 8 characters long, padded
with null characters if needed. `FF 0A` is returned by the device if the
path does not exist.

(<span style="color: IndianRed">TODO: clarify this point</span>) `date`
is the date of the file. It changes each time the file is saved. It
could be the age of the file in seconds (but what would be a date of 0,
and how does the TI-Nspire track this?)

If the file or directory does not exist, the TI-Nspire sends:

`IN: FF 0A`

#### Sending a file

The command *PutFile* should be issued, then the content of the file
split into as many packets as needed.

`OUT: 03 01 [>=8 file path] 00 [4 file size]`

*File path* is the full path to the file to create on the device. The
target directory should already exist. The directory name may have a
leading slash. The file name must be terminated by the extension
*".tns"*. An example could be *"/Examples/myfile.tns"*. *File size* can
be 0 for an empty file (although empty files couldn't be opened by the
TI-Nspire after reception).

The device response is `04` if it is ready to receive the file content,
`FF 14` if the file path is invalid, `FF 15` if the file path doesn't
have a *".tns"* extension:

`IN: 04`

The file content is then split into 253 byte-long chunks (since the
maximum length of the data part is 254 bytes, and since the data part
starts with a command byte). The last packet must be less than 253 bytes
long. If the file is zero bytes long, nothing is sent by the host.

```
OUT: 05 [253 file content chunk #1]
OUT: 05 [253 file content chunk #2]
...
OUT: 05 [<253 last file content chunk]
```

The device response is `FF 00` if the file has been correctly received,
`FF 09` if the parent folder doesn't not exist: the file has not been
created. The device does send a response if the host was transmitting an
empty file.

`IN: FF 00`

#### Creating a directory

Request:

`OUT: 0A 03 [>=8 directory path] 00`

Acknowledgement:

`IN: FF 00`

#### Receiving a file

The command *GetFile* should be issued, then the content of the file
split into as many packets as needed.

`OUT: 07 01 [>=8 file path] 00`

*File path* is the full path to the file to retrieve on the device. The
target directory should already exist. The directory name may have a
leading slash. The file name must be terminated by the extension
"*.tns*". An example could be "*/Examples/myfile.tns*".

The NSpire send size of file:

`IN: 03 01 [9 00] [4 size of file]`

The device response is 04 if it is ready to receive the file content, FF
0A if the path does not exist.

`OUT: 04`

The file contents is then split into 253 byte-long chunks (since the
maximum length of the data part is 254 bytes, and since the data part
starts with a command byte). The last packet must be less than 253 bytes
long. If the file is zero bytes long, nothing is sent by the host.

```
IN: 05 [253 file content chunk #1]
IN: 05 [253 file content chunk #2]
...
IN: 05 [<253 last file content chunk]
```

Please note that data contents is the same as file contents (i.e. a set
of XML files compressed with PK-ZIP).

To finish, computer send a final acknowledgment:

`OUT: FF 00`

#### Deleting a file

Request:

`OUT: 09 01 [>=8 file path] 00`

The constraints on *file path* are the same as those described in
<a href="#Sending_a_file" class="wikilink"
title="Sending a file">Sending a file</a>.

Acknowledgement:

`IN: FF 00`

#### Deleting a directory

Request:

`OUT: 0B 03 [>=8 directory path] 00`

Acknowledgement:

`IN: FF 00`

If the directory is not empty, the files it contains are also deleted.

#### Renaming a file or directory

Request:

`OUT: 21 01 [>=8 initial file path] [>=9 new file path]`

The constraints on the file pathes are the same as those described in
<a href="#Sending_a_file" class="wikilink"
title="Sending a file">Sending a file</a>.

Acknowledgement:

`IN: FF 00`

#### Copying a file

Request:

`OUT: 0C 01 [>=8 initial file path] [>=9 new file path]`

Acknowledgement:

`IN: FF 00`

(<span style="color: IndianRed">TODO: possible error codes</span>)

To copy a directory, the target directory must be created, the source
directory must be listed, and each file must be copied independently.

### Screenshot (RLE)

To request a screenshot:

`OUT: 00`

The first response is:

`IN: 01 raw data`

where raw data contents is:

```
@00d (4 bytes): RLE raw data size
@04d (2 bytes): y (?) - always 0
@06d (2 bytes): x (?) - always 0
@08d (2 bytes): LCD width in pixels (0140 = 320 pixels)
@10d (2 bytes): LCD height in pixels (00F0 = 240 pixels)
@12d (1 byte): bits per pixel (4)
@13d (1 byte): sample model - always 0 which means [packed](http://en.wikipedia.org/wiki/Packed_pixel)
```

The RLE raw data is then split into as much 270 bytes-long packets (254
bytes long for the data part) as needed. The last packet will be shorter
than 270 bytes.

`IN: 02 [253 RLE raw data]`

where *raw data* is the screenshot encoded with an implementation of Run
Length Encoding
([RLE](http://en.wikipedia.org/wiki/Run-length_encoding)) described
below. The encoding is applied to the screenshot *before* it is split
into packets.

The encoded stream is made of sequences of blocks on the same pattern as
`LL DD DD DD ... DD`. The first byte *LL* is a signed byte representing
a length:

- if it is equal to or greater than zero, then it is the length of the
  next "run" minus one. The next byte is the value to repeat. For
  instance `FF FF FF FF FF` would be encoded as `04 FF`. Note that if a
  run is longer than the maximum length which could be encoded like this
  (128), then run is divided into sub-runs with a length equal or lower
  than 128 (for instance `7F 01 7F 01 03 01` represents a run of 128 +
  128 + 4 = 260 times the byte 01).
- if it is lower than zero, then it is `-(L - 1)`, where *L* is the
  length (greater than zero) of a sequence not encoded which follows.
  This is used for sequences of isolated values for which the RLE
  encoded version would take more space than the original one. For
  instance `01 03 04 06` could be encoded as `FD 01 03 04 06` (`FD` is
  the signed byte -3).

In the RLE-decoded stream, each pixel is coded with a 4 bit gray scale
(16 colors), where `0b000` is black and `0b1111` is white.

### OS installation

To send an OS:

```
OUT: 03 [4 size of OS]
IN : 04 (*)
OUT: 05 [OS file contents chunk]
IN:  Either FF 00 (header validated) or 04 (send more data until FF 00)
OUT: 05 [next file contents chunk]
...
OUT: 05 [last file contents chunk]
```

(\*) if NSpire rejects OS, it sends 02 00 00 00 00 00 or FF XX, with XX
!= 0

Next, the NSpire continuously send progress bar value (0..65h) until
maximum value is reached (101d = 65h):

```
IN : 06 06 (  6%)
IN : 06 0B ( 11%)
...
IN : 06 65 (101%)
```

If OS can not be installed, NSpire will reply with:

```
IN : 06 06
...
IN : 06 60
IN : FF 04
```

### Echo

The echo service simply echoes any received data as follows:

```
OUT: raw data
IN: raw data
```

## Testing

### Computer Link Software

Computer Link Software's functions can be tested on the command line
using the testing code embedded in the Java binding *NavNet.jar* for the
low-level connector. `TESTNUM` is the number of the test to run (remove
the echo at the beginning and the file redirection at the end the first
time to understand how it works). The command line to use with
[Cygwin](http://www.cygwin.com/) on Microsoft Windows is:

`$ echo $TESTNUM | java -cp "C:\Program Files\TI Education\TI-Nspire Computer Link\lib\navnet.jar" com.ti.eps.navnet.main  >logs 2>&1`

The logs of both the native NavNet connector and the Java testing
program will be written to the output file. The verbosity of the native
logs can be tuned in `Main.class`:

`NavNet.init("-c X -d X -p connectors"); // X (initially 4) should be between 0 (no logs) and 5 (finest)`

### USB traffic analysis

<span style="color: IndianRed">Any suggestions of a *free* USB analyzer
for Microsoft Windows?</span>

Tools:

\- somewhat old (2002) but working on Windows9x/XP:
[SnoopyPro](http://sourceforge.net/projects/usbsnoop/)

\- another tool derivated from SniffUsb (free of use but not free
software):
[SniffUsb-2](http://www.pcausa.com/Utilities/UsbSnoop/default.htm)

Unfortunately these tools don't seem to work on Windows Vista.

The TiLP team can provide you some tools to make traffic analysis more
convenient:

\- xml2hex which turns SnoopyPro log into hexadecimal log,

\- OR log2hex which turns SniffUsb log into hexadecimal log,

\- and hex2nsp which turns hexadecimal log into ready-to-read NSpire
packet listing. Both of them are available on the \<[TiLP LinkGuide
repository](http://svn.tilp.info/cgi-bin/viewcvs.cgi/linkguide/trunk/analysis/)\>.

If you want to help, you can take a look at some NSpire
\<[logs](http://svn.tilp.info/cgi-bin/viewcvs.cgi/linkguide/trunk/analysis/logs/nspire/)\>.

## TODO

- Find the differences between PC/TI-Nspire transfers and
  TI-Nspire/TI-Nspire transfers.
- Test and document the <a href="#Service_identifiers" class="wikilink"
  title="services available">services available</a>