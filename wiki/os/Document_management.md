---
title: Document management
permalink: /Document_management/
parent: Operating System
layout: page
---

## Document format

A *.tns* file is a PK-ZIP file containg several XML files.
*Document.xml* corresponds to the options of the *Document settings*
dialog box of the OS (some options can't be configured graphically). An
XML file exists for each problem of the document (*Problem1.xml*,
*Problem2.xml*, and so on).

## Document format tests

Bold words are used as keywords for quick reference.

*Please keep in mind before adding any elements to this list that as
long as a way to bypass the downgrade protection has not been found, too
many details on leads to possible exploits to execute arbitrary code
should preferably not be posted here.*

- The **Computer Link Software** doesn't check anything except the file
  extension.
- **Compression format**: TNS files compressed with the graphical
  frontend of 7-Zip are accepted. The command-line version of 7-Zip and
  Windows XP's native zipper produce an error message when the document
  is open on the TI-Nspire. Cygwin's zip command makes the calculator
  reboot. The TI-Nspire probably uses a custom decompressor (yet the
  About screen mentions that zlib is used by the OS).
- An error message is displayed when a document with containing an
  **ill-formed** XML file is open.
- **Adding**, removing or renaming files (XML or not) in the TNS
  displays an error.
- Specifying an **invalid *type* attribute** (for example "TI.Notepaddd"
  instead of "TI.Notepad") in a *<wdgt>* tag of a problem displays an
  empty frame instead.
- Text added **between the tags** of the problems ("mixt-content") is
  ignored.
- Changing the **version attribute** of a problem displays an error.
- Any **DTD** assigned or added to XML files is ignored.
- An *XML external entity* defined inline to try to read system files is
  ignored (not displayed), whether an relative or absolute path is used
  (but perhaps more possible file paths should be tested).
- Setting **an encoding** in the XML declaration not supported by
  default in [expat](http://www.jclark.com/xml/expatfaq.html) displays
  an error when the document is opened. And sending garbage data such as
  ''
  <?xml version="1.0_stuff_here_a_lot_of_zeros_" encoding="UTF-8" ?>

  '' displays the correct folder after a latency, and results in a loss
  of communication between the calculator and the linking software (see
  **uncompressed size**). (Is the USB management neglected during the
  freeze? That would be strange for an RTOS.)
- **Format markup** of the Notepad widget (for example
  `Applications \1keyword \` which would render "Applications" as a
  keyword): nothing special found.
- If the **attribute *clay*** (card layout identifier) of the *<card>*
  tag is set to 1 ("vertical split", the identifiers are the ones found
  in the menu *Page Layout/Select layout* - 1) instead of 0 (full
  screen), the calculator reboots because of a crash, since there is
  only one widget in the card instead of 2. If the identifier is 7 or
  greater, or negative, an error is displayed when the document is
  opened.
- If the **height of a splitted card** (attributes h1, h2, ...) is set
  to a negative number, the part of the card is not displayed but the
  TI-Nspire doesn't crash. If the height is set above 10000, only the
  0-10000 part is showed, the rest is clipped.
- If the **width of a splitted card** (attributes w1, w2, ...) is set
  above 10000, it has no effect.
- Sending a TNS file of 25KB of **uncompressed size**/2KB of compressed
  size makes the TI-Nspire freeze for a few minutes when open, then
  makes it display an error message indicating that the document could
  not be opened. During the opening the Computer Link Software can't
  communicate with the TI-Nspire, although it is correctly detected as a
  USB device by the computer when unplugged and re-plugged. Files of
  more than 25KB of uncompressed size seem to make the TI-Nspire hang
  forever.

To verify : Sending a document with garbage data (same letter repeated
700000 times after <urn:TI.Problem>) made the TI-Nspire freeze for a few
minutes, but the document was then opened (navigating in the tools menu
showed freezes of a few seconds with this document).

- **Documents bigger** than the current storage capacity are refused by
  the Computer Link Software.
- Well-formed XML-escaped content but with dummy tags in the
  **<sp:auth>** tag of the widget *TI.Scratchpad* (an expression
  content) makes the calculator reboot. The calculator reboots
  indefinitely since the last document open is reopen at startup. One
  way to get out of the loop it is to use the maintenance menu.
- In the **<sym>** section, variables with name (**<n>** tag) longer
  than 16 chars are ignored. The attribute *t* of the **<e>** tag is the
  type of the variable : "1" for a list, "3" for an integer or float,
  "5" for a string (the value is "mytext"), "6" for a function. For
  functions, the *v* tag is the value and the *p* tag is the parameter
  name.
- The 'EE' character (exponent, displayed as a small 'E' on calculator)
  is the Unicode character 0xF000, coded in UTF-8 as 0xEF 0x80 0x80.