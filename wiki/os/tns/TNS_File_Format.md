---
title: TNS File Format
permalink: /TNS_File_Format
parent: Operating System
layout: page
---

.tns is the file format of the TI-Nspire documents. It contains several
XML files, for more informations on its content:
<a href="xml_file_format" class="wikilink"
title="XML format used in TNS files">XML format used in TNS files</a>.
This page only discusses the .TNS file format itself.

# History

The first tns files were standard zip files. Since OS 1.2, the format
changed and the files stopped to be openable by zip utilities. If you
are unfamiliar with the zip format, please refer to [the official
specification](http://www.pkware.com/documents/casestudies/APPNOTE.TXT)
to better understand the following text.

# The new format

It appears that the new format is just a zip file in disguise. It
contains two differences which prevent utilities like the "file" command
from detecting it.

## Magic header

Instead of:

`PK\003\004`

The header is:

`*TIMLP`

Followed by what seems to be a version number. Known version numbers are
0120, 0200, 0300. No difference could be found between the files in
these formats, further investigation is probably needed.

## End of central directory

Instead of:

`PK\005\006`

The end of central directory also known as digital signature is:

`TIPD`

## Converting to a standard zip file:

    #!/usr/bin/perl
    ## tns2zip.pl
    ## Convert .TNS files to standard .ZIP files
    ## Adapted from http://forums.devshed.com/perl-programming-6/perl-searching-and-replacing-an-hex-pattern-514905.html

    use strict;
    use warnings;

    if( $#ARGV < 1 ){
        print "This script converts TI N-Spire .TNS files to standard .ZIP files\n";
        print "USAGE: $0 input.tns output.zip\n";
        exit(0);
    }

    my $hex_in = $ARGV[0];
    my $hex_out = $ARGV[1];
    my $new_meth = $ARGV[2]; # optional
    my $offset;
    my $indx = 0;

    open (HEX_IN, "$hex_in") or die("open failed for $hex_in : $!");
    open (HEX_OUT, ">$hex_out") or die("open failed for $hex_out : $!");
    binmode(HEX_IN);
    binmode(HEX_OUT);
    local $/;
    my $string = <HEX_IN>;

    $string =~ s/\*TIMLP[0-9]{4}(.*)/PK\x03\x04$1/gi; # net change -6 bytes

    $string =~ s/(.*)TIPD(.*)/$1PK\x05\x06$2/gi;      # net change  0 bytes

    ## FIXME: Make sure that the actual size of the data and the size indicated in the zip file are the same
    ## FIXED: Subtract 6 for offset of start of central directory in the "end of central directory record"
    ## near the end of the file.
    $offset = unpack("L",substr($string,-6,4));  # get old central directory offset
    $offset = $offset - 6 ;
    substr($string,-6,4) = pack("L",$offset);      # insert new central directory offset

    ## --- try alternate decompression method: files first ---
    while(($indx = index($string,"\x50\x4b\x03\x04",$indx)) > -1)
    {
     $indx = $indx + 8;
     my $method = unpack("s",substr($string,$indx,2));
     if ($method != 13)
      {
        print "Not a Ti tns compression format\n";
        exit;
      }
     if(defined($ARGV[2])){
      substr($string,$indx,2) = pack('S',$new_meth);      # insert new decompression method
     }
    }

    ## --- central directory updates:
    ## (1)change local header offset by -6 bytes.
    ## (2)change method in central directory also if selected
    $indx = 0;
    while(($indx = index($string,"\x50\x4b\x01\x02",$indx)) > -1)
    {
     $indx = $indx + 10;
     my $method = unpack("s",substr($string,$indx,2));
     if ($method != 13)
      {
        print "Not a Ti tns compression format\n";
        exit;
      }
     if(defined($ARGV[2])){
      substr($string,$indx,2) = pack('S',$new_meth);      # insert new decompression method
     }
      $indx = $indx + 32;
      $offset = unpack("L",substr($string,$indx,4));  # get old local header offset
      $offset = $offset -6 ;  # subtract 6 bytes
      substr($string,$indx,4) = pack("L",$offset);    # insert new local header offset
    }

    print HEX_OUT $string;

    close(HEX_IN);
    close(HEX_OUT);
    ## now opens with WinZip but complains about decompressing it, even trying different decompression methods

# TI Compression Method 13

According to the zip specification, there is no compression method 13 at
the moment. Trying to change the compression method bytes to something
else doesn't work either, so it appears that TI has created its own
compression method. There is currently no known way to get data from it,
further investigations are required.

## A magic number ?

All the files tested so far contain the following hexadecimal numbers
right at the beginning of the file data field(see the zip
specification):

`OF CE D8 D2 81 06 86 5B`

No known compression method seem to use this magic number, finding its
origin might help in understanding TI compression method.

## Reasons for the use of this compression method

If we can understand why TI chose to change the compression method used,
we might understand how it work too.

- Prevent people from fiddling with the XML inside? This could mean that
  there is a way to use it to hack the calculator.
- Prevent people from using different tools that the official nspire
  software for editing documents? Not sure if it's a big source of
  income for them.
- Make smaller documents(this could be easily checked)? After all, this
  is the main use of compression, and size on the calculator is limited,
  but are the files stored in the same format in the calculator?
- Provide faster decompression? Standard compression methods already
  provide a wide range of choice, but who knows.