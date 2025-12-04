---
title: XML Files
permalink: /xml_file_format/
parent: TNS File Format
layout: page
---

# XML format used in TNS files

TNS and TNSP files contains XML data, encrypted and compressed (see
<a href="/TNS_File_Format" class="wikilink" title="TNS File Format">TNS
File Format</a> and <a href="/TNSP_File_Format" class="wikilink"
title="TNSP File Format">TNSP File Format</a>).

## How to get the XML data and build TNS from XML

### TNS files

To get the XML data, you can use:

\-[Levak’s clipboard
dumper](http://levak.free.fr/ftp/nspire/ClipboardDumper/) with the TI
Nspire Computer Software

\-[Godplat's
nspire_emu](http://hackspire.unsads.com/wiki/index.php/Emulators#nspire_emu)
like described
[here](http://ndlessly.wordpress.com/2011/09/19/luna-updated-for-third-party-tns-document-generation/)
and
[here](http://www.omnimaga.org/index.php?topic=9807.msg192435#msg192435)

To build TNS files from XML, you can use
[Luna](http://tiplanet.org/forum/archives_voir.php?id=3701).

### TNSP files

Any text editor and good zip-reader that can handle odd zip files should
be OK.

You should be able to extract every file except one, which is in clear
at the beginning of the TNSP file.

## TNS files general XML structure

TNS files are composed of two or more XML files:

\-<a href="/Document.xml" class="wikilink"
title="Document.xml">Document.xml</a> (general info on the file)

\-<a href="/ProblemX.xml" class="wikilink"
title="ProblemX.xml">ProblemX.xml</a> (X being the number of the
problem, up to 50 in regular TNS files)

In each ProblemX.xml, there are different cards (described in
<a href="/ProblemX.xml" class="wikilink"
title="ProblemX.xml">ProblemX.xml</a>), which contains themselves
<a href="/Widgets" class="wikilink" title="Widgets">Widgets</a>.

(If the TNS file was created with OS \>=1.2, these files are encrypted,
and thus you can't extract them)

## TNSP files general XML structure

Todo