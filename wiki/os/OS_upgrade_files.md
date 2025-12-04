---
title: OS upgrade files
permalink: /OS_upgrade_files/
parent: Operating System
layout: page
---

The latest versions of the TI-Nspire and TI-Nspire CAS Operating System
are available
[here](http://education.ti.com/educationportal/sites/US/nonProductMulti/apps_latest.html?bid=2).
This article describes the format and content of the current versions
(v1.1.9253 and v1.1.9170) of the **.tno** and **.tnc** files.

## .tno/.tnc

A *.tno* file is an OS image for the TI-NSpire. A *.tnc* is for the
TI-Nspire CAS. Contrary to other calculators with flash memory, the
whole OS file is received before being written to flash memory. This is
possible thanks to the huge amount of RAM available on the TI-Nspire.
Sending a *.tno* to a TI-Nspire CAS is possible and vice-versa, but at
the end of the reception a message will be displayed by the Computer
Link Software indicating that the file is corrupted, and the OS won't be
installed.

A .tno/.tnc is a PK-Zip file with a custom ASCII header describing the
OS update file. It can be opened with a simple archiving program such as
WinRAR or WinZip. The PK-Zip file contains a certificate file
(*TI-Nspire.cer*) and a .img file (*TI-Nspire.img*). OS 1.4, it also
contains *boot2.cer* and *boot2.img*. Here is an example of .tno/.tnc
header (lines starting with \# are added comments):

```
# File, version, size of the file, ?
 TI-Nspire.tno 1.1.9253  3092555 0
 # __RES__, version, ?, size of the compressed file system once decompressed
 __RES__ 1.1.9253 0  2420072
 \0x1A 
```

The header can have a variable size and is terminated by the byte 0x1A.
The sizes in the header are used both to validate on the TI-Nspire side
that there is enough available memory to install the new OS (the
validation is performed as soon as the header has been received), and at
the end of the transfer to ensure that everything has been received as
expected. The sizes must be positive integers and must be present, else
an error is returned to the Computer Link Software. Multiple spaces are
ignored by the parser. The parser seems to read sizes of more than 7
digits incorrectly, accepting some huge sizes, but rejecting other huge
sizes and asking for free space of random size.

## TI-Nspire.cer

The file follows TI's standard certificate format used on many caculator
models. See [TIGCC's
documentation](http://tigcc.ticalc.org/doc/cert.html) for more
information (more particularly
[cread](http://tigcc.ticalc.org/doc/cert.html#cread) and
[cfindfield](http://tigcc.ticalc.org/doc/cert.html#cfindfield)).

This file hasn't change between the different upgrades released by TI,
up to version 1.7, and is the same for CAS and non-CAS TI-Nspire.

(*Format of the following section: Field ID (hex) - size (dec) :
comment. The indentation corresponds to subfields.)*


0350 : top-level field, similar to TI-68k's FLASH_APP_CERT (0x0300),
PRODUCT_CODE (0x0320), FLASH_ROM_CERT (0x0330), etc.


0010 - 4 : ?

0100 - 4 : Revision Number (same field as for the TI-68k)

0260 - 140 : <a href="#1024-bit_RSA_public_keys" class="wikilink"
title="1024-bit RSA public key">1024-bit RSA public key</a> used for
validating both CAS and non-CAS OSes.

0260 - 140 : <a href="#1024-bit_RSA_public_keys" class="wikilink"
title="1024-bit RSA public key">1024-bit RSA public key</a>

02A0 - 270 : (OS 3.0+) 2048-bit RSA public key

02A0 - 270 : (OS 3.0+) 2048-bit RSA public key

0240 - 128 : Same field ID as in the .tno/.tnc. Certificate signature,
validated using a key hard-coded inside BOOT2 (its modulus is
EDBF3336...)

0290 - 256 : (OS 3.0+) Another signature?

FFF0 - 0 : END_OF_CERT

## TI-Nspire.img

### Structure

As for *TI-Nspire.cer*, the file is organized as a certificate.

(*Format of the following section: Field ID (hex) - size (dec) -
@TI-Nspire: offset of the current version (hex) - @TI-Nspire CAS:
offset): comment. The indentation corresponds to subfields.)*


8000


0010 - 1 : (OS 3.0+) Set to a nonzero byte to indicate that the first
byte of the 80F0 field is complemented

8040 : Product Name: "TI-Nspire"

8010 : First part of Product ID

80E0 : Allowable values for
<a href="/Memory-mapped_I/O_ports#900A0000_-_Miscellaneous"
class="wikilink" title="ASIC user flags">ASIC user flags</a> (00 byte
for TI-Nspire, 01 byte for TI-Nspire CAS)

8020 : OS version

8020 : Minimum OS version after installation. Downgrade to OS versions
lower than this one won't be allowed. "1.1.x" for OS \<= "2.0.x",
"1.7.x" for OS "2.1.x" (i.e. OS 2.1 cannot be downgraded to OS 1.1 for
instance).

8080 - 8 : The first 4 bytes are the OS base address: address to which
the OS will be copied by boot 2(0x10000000). The 4 next bytes define the
format of field 8070 (OS code):


bit 0: compressed (custom scheme)

bit 1: encrypted (blowfish)

bit 2: Content is a zip file with "image.bin" inside

bit 3: encrypted (3DES)

bit 4: zlib compressed

Example: 6 = (4 \| 2) = (bit 2 \| bit 1): decryption yields a zip file.
Used by all modern OS versions.

0320 - 6 : Product code : 0

8220 : (OS 3.0+) Minimum Boot2 version allowed (only recognized in
Boot2/OS 1.4 and above)

80F0 : Blowfish-encrypted version of the previous fields (8040, 8010,
80E0, 8020, 8080, 0320, 8220)

8210 - 4 : Uncompressed size of PK-Zipped file system

8200 : PK-Zipped file system, see further.

8070 : OS code, mangled according to field 8080. The size is a multiple
of 8.

0240 - 128 - The signature of the 8000 field, similar to TI-68k's 64
bytes long field 0200. This field is big endian. This signature is
validated using the first 0260 public key stored in TI-Nspire.cer.

0290 - 256 - (OS 3.0+) Another signature, probably validated using the
first 02A0 public key in TI-Nspire.cer

FFF0 - 0 : END_OF_CERT

The crypted field 8070 is 195568 bytes longer in the .tno than in the
.tnc, that is the TI-84 Plus emulator of the TI-Nspire would be ~200kb
bigger than the CAS of the TI-Nspire CAS! May be the two OS integrates a
CAS, but it is not enabled on the TI-Nspire.

### Directory Listing

Directory listing of ti-nspire.img (TI-Nspire CAS version), from WinRAR
3.71.

```
.\
 \documents\
           \MyLib\
                 \linalgcas.tns             (TI-Nspire Document)
 \phoenix\
         \clnk\
              \locales\
                      \da\
                         \strings.res
                      \de\
                         \strings.res
                      ...
         \ctlg\
              \locales\
                      \da\
                         \strings.res
                      \de\
                         \strings.res
                      ...
        ...
```

### Compressed file system

Field 8200 of *TI-Nspire.img* contains PK-Zipped files, used to setup
the file system of the TI-Nspire. Its structure probably follows the
target file system tree. The .tnc and .tno both have a *phoenix/*
directory, with a sub-directory for each module of the OS which contain
localized resource files (*.res*), the 'getting started' *.tnc* file,
and factory settings. The .tnc has a special directory *ti84/* which
contains an image of the TI-84 Plus archive memory splitted into 64kb
files, used to setup the memory of the emulated TI-84 Plus. Some flash
apps are preinstalled in this image.

The files in the sub-directory *phoenix/* are the same between the
TI-Nspire and the TI-Nspire CAS, excepted the sample document *Getting
Started*, the *.res* files of ctlg (catalog) and *syst* (system), and
the factory settings in *factory.zip* (the TI-Nspire is set to *real*
for *Real or Complex*, the TI-Nspire is set to 4=??).

## boot2.cer

Only present if the OS file contains an update of the boot 2.


0340 - 146 : Certificate allowing the boot2 updater present in OS 1.4+
to validate this image


0270 - 1 : ??? \x00

0260 - 140 : \[#1024-bit_RSA_public_keys\|1024-bit RSA public key\]\]
used for validating the first part of boot2.img

02A0 - 270 : (Boot2 3.0+) 2048-bit RSA public key

0240 - 128 : Signature of the above field 0340. This signature is
validated using a key hard-coded inside boot2 (its modulus is
EDBF3336...)

0290 - 256 : (Boot2 3.0+) Signature of the above field 0340.

FFF0 - 0 : END_OF_CERT

## boot2.img

Only present if the OS file contains an update of the boot 2. The fields
have the same meaning as in TI-Nspire.img.

Structure:


8000


8040 : Product Name: 'BOOT2 '

8010 : Product ID '50C'

8010 : Product ID '50E'

8020 : Version number

8020 : (empty)

8080 - 8 : The first 4 bytes are the base address: address to which the
contents of 8070 will be copied by boot1 (0x11800000). The 4 next bytes
define the format of field 8070: 0 (uncompressed), 1 (compressed)

0320 - 6 : Product Code: 0

8070 : Compressed or uncompressed contents, depending on the flags in
field 8080.

0240 - 128 - @TI-Nspire 1.4: 12FEC7. Signature of the field 8000,
validated using the key given in boot2.cer's 0260 field (and again
below)

0290 - 256 : (Boot2 3.0+) Signature of the field 8000, validated using
the key given in boot2.cer's 02A0 field

0340 - 146 : Certificate used to validate this image when loading it
from boot1


0270 - 1 : ??? \x00

0260 - 140 : \[#1024-bit_RSA_public_keys\|1024-bit RSA public key\]\]
used for validating the first part of boot2.img

0240 - 128 : Signature of the above field 0340. Does not depend on the
content of the fields 8000 and 0240 above (same between boot2 1.1 and
1.4), and different from field 0240 of *boot2.cer*. This signature is
validated using a key hard-coded inside boot1 (its modulus is
CC14CF31...)

FFF0 - 0 : END_OF_CERT

## 1024-bit RSA public keys

These keys can be found in *boot2.cer*, *boot2.img* and *TI-Nspire.cer*,
in the certificate field 340. They are formatted in
[ASN.1](http://en.wikipedia.org/wiki/Abstract_Syntax_Notation_One)
[DER](http://en.wikipedia.org/wiki/Distinguished_Encoding_Rules) format,
as described [here](http://www.jensign.com/JavaScience/dotnet/JKeyNet/)
or
[here](http://stackoverflow.com/questions/1281102/reading-a-asn-1-der-encoded-rsa-public-key).
As a bytes sequence, the format is:

`30 81 89 02 81 81 00 [128 bytes: the key] 02 03 `**`01 00 01`**

The final 01 00 01 is the RSA exponent, which means 65537 in DER, a
[common value](http://en.wikipedia.org/wiki/RSA#Key_generation_2) for
*e*.

There are three of these keys known:

- "boot" key: stored in *boot2.cer* and *boot2.img*, used to validate
  the first part of *boot2.img*
- "OS" key: stored in *TI-Nspire.cer*, used to validate the OS
- a second key stored in *TI-Nspire.cer*, purpose unknown

Signatures consist of a "0240" field of 128 bytes; raising this (big
endian) number to 65537th power mod n gives a padded SHA256 hash of the
data being signed.

The public key used by boot1 to validate *boot2.img* is embedded in
boot1. The public key used to validate *TI-Nspire.cer* is embedded in
boot2.

Boot1 validates the signature of boot2 at boot-time. Respectively Boot2
validates the signature of the OS at boot-time.

## Breaking RSA : twists on naive algorithms

The Perl scripts below use a naive integer factoring method (trial
division), with twists. An element of luck is added to the script.

- The first script takes the square root of n (the key), to get an
  approximate value on a prime(p) , then decrement that value and see if
  it divides n. You continue with this until you get a hit. Two lines
  were added to the procedure: the starting value of p is bit shifted to
  the right by the hour of the GMTIME. So someone around the world
  running this might get extremely lucky and crack it in an hour !! The
  starting trial prime size for p can be anywhere between 489 bits and
  512 bits, which is typical when you look at known key primes. For
  example, if a prime to a 1024 bit key is 498 bits, the person who gets
  the 498-499 bit value, might crack it in hours, the one who gets 497
  bits would never crack it, the one who got 511 bit start value would
  crack it in millions of years.
- The second script generates all 512-bit numbers, using a LFSR seeded
  with the current time.

The 3 keys discussed on this page (boot.cer,boot2.img,TI-Nspire.cer) are
in these scripts - uncomment one of them in. You can test them with the
512 bit keys recently cracked on the other Ti calculators
[here](http://diomedes.phear.cc/~chronomex/keys.shtml). For general
links to this controversy see
[(1)](http://www.ticalc.org/archives/news/articles/14/145/145316.html),
[(2)](http://www.ticalc.org/archives/news/articles/14/145/145434.html)
and
[(3)](http://www.ticalc.org/archives/news/articles/14/145/145545.html)
on ticalc.org. The programs below are nevertheless extremely time
intensive.

```
 #!/usr/bin/perl
 # crackRSA.pl 
 # set your console window to 150 characters wide for progress output
 # old fashion brute force factor algorithm, with a little luck
 
 #use lib '../lib'; # uncomment this if you unzip Math::BigInt to a nonsystem directory 
 use warnings;
 use Math::BigInt try => 'GMP';  # faster if GMP installed, falls back to pure perl otherwise
 
 print "# Using Math::BigInt v",$Math::BigInt::VERSION,"\n";
 
 #  ---- public encryption key '$e'
 $e = Math::BigInt->new('65537');
 
 #  ---- modulus or second public 1024-bit key '$n' - uncomment in only one of three $n below
 # ---- boot2.cer & boot2.img keys(same number):
 #my $n = Math::BigInt->new('0xc3b3a7015c04299ff3a25f104e2285c1ec2d55471e6208959d0f6981b2fa2c6d3e316f9364d5eb5c7789e142b75bfaf402e7e02fac0cb09f6419db1f44679f8bbcca142f1d312feb095708ef175a4ef80271321e7240f0d854c90a74fc59209cdf80aa8f85ae3b948a3ce55c69cd050098d5a79aebbc241cc642b106b1af2cb7');
 # ----- TI-Nspire.cer, 2 keys:
 #my $n=Math::BigInt->new('0xaba7f0b8c7feb6e33438af5c25c67389eaf4d73f80cd0a37922493431cde03b34da448bdb05387cf7a8c59ee12d9613429a2b07ea385752f079892da1ae76c2b158f2d7169aae066432fe44f797df39dd6a0d7b2e2091281b30efac247c51576ebc93ec456de2e27d36b713844336b65af67ee58e6107a6a1deb954a91095295');
 #my $n=Math::BigInt->new('0xb15e01c47c421be62f4e769b3ac98f4f983a820b0c181e35715d84a4f1acf0527eeddfabf9f66e73bedb55376e22f860c34dc70ce239157297056d4ecf46535778c3917647b5a6bb9c5638cdeff3e309ff66878fd4f233cf157d7af4136f307df90ec6ae6eaff6bead6d52f423a37dac59ff38ae876008103728f2bd674e858f');
 
 # --- now brute force find factors ---
 $p = $n->copy();
 $p->bsqrt();   # ~ p
 $biggestp = length($p->as_bin())-2;
 print "prime p is a ",$biggestp," bit number or less.\n";
 $p = $p->brsft((gmtime(time))[2]);  # random right shift start on by which hour
 if((gmtime(time))[2] > 0){$p = Math::BigInt->new('0b' . ('1' x ($biggestp-(gmtime(time))[2]))); } # so no integers are missed by this scheme
 print "p randomly starts shifted right by ",(gmtime(time))[2]," bits and bit filled.\n";
 
 ITERATE:
 $modtest = $n->copy();
 if ($modtest->bmod($p)->is_zero()){
 print "Great found p and q !!!!!\n";
 print 'p= ',$p->as_hex(),"\n";
 $q = $n->copy();
 $q->bdiv($p);
 print 'q= ',$q->as_hex(),"\n";
 goto SECRETKEY;                   
 }
 
 DECREMENT:
 $p->bdec(); 
 goto DECREMENT unless $p->is_odd();
 print 'p=',$p->as_hex(),"\r";  # comment out for speed for serious cracking                                          
 goto ITERATE;
 
 SECRETKEY: # calculate private decryption key  'd' 
 print "Calculating secret key d\n";
 $PQ = $p->copy();
 $PQ->bdec();
 $r = $q->copy();
 $r->bdec();
 $PQ->bmul($r);   # $PQ = (p-1)(q-1)
 $d = $e->copy()->bmodinv($PQ);
 
 print 'p= ',$p->as_hex(),"\n";
 print 'q= ',$q->as_hex(),"\n";
 print 'd= ',$d->as_hex(),"\n";
 print 'n= ',$n->as_hex(),"\n";
 print 'e= ',$e->as_hex(),"\n";   
```

```
 #!/usr/bin/perl
 # crackRSA.pl
 # set your console window to 150 characters wide for progress output
 # old fashion brute force factor algorithm, with a little luck
 
 #use lib '../lib'; # uncomment this if you unzip Math::BigInt to a nonsystem directory
 use warnings;
 use Math::BigInt try => 'GMP';  # faster if GMP installed, falls back to pure perl otherwise
 
 print "# Using Math::BigInt v",$Math::BigInt::VERSION,"\n";
 
 #  ---- public encryption key '$e'
 $e = Math::BigInt->new('65537');
 $one = Math::BigInt->bone();
 # 512-bit Galois LFSR generator pattern
 $tap = Math::BigInt->new("0b101001001");
 $tap = $tap->blsft(Math::BigInt->new("503"));
 
 #  ---- modulus or second public 1024-bit key '$n' - uncomment in only one of three $n below
 # ---- boot2.cer & boot2.img keys(same number):
 #my $n = Math::BigInt->new('0xc3b3a7015c04299ff3a25f104e2285c1ec2d55471e6208959d0f6981b2fa2c6d3e316f9364d5eb5c7789e142b75bfaf402e7e02fac0cb09f6419db1f44679f8bbcca142f1d312feb095708ef175a4ef80271321e7240f0d854c90a74fc59209cdf80aa8f85ae3b948a3ce55c69cd050098d5a79aebbc241cc642b106b1af2cb7');
 # ----- TI-Nspire.cer, 2 keys:
 #my $n=Math::BigInt->new('0xaba7f0b8c7feb6e33438af5c25c67389eaf4d73f80cd0a37922493431cde03b34da448bdb05387cf7a8c59ee12d9613429a2b07ea385752f079892da1ae76c2b158f2d7169aae066432fe44f797df39dd6a0d7b2e2091281b30efac247c51576ebc93ec456de2e27d36b713844336b65af67ee58e6107a6a1deb954a91095295');
 #my $n=Math::BigInt->new('0xb15e01c47c421be62f4e769b3ac98f4f983a820b0c181e35715d84a4f1acf0527eeddfabf9f66e73bedb55376e22f860c34dc70ce239157297056d4ecf46535778c3917647b5a6bb9c5638cdeff3e309ff66878fd4f233cf157d7af4136f307df90ec6ae6eaff6bead6d52f423a37dac59ff38ae876008103728f2bd674e858f');
 
 # --- now brute force find factors ---
     $p = $n->copy();
     $p->bsqrt();   # ~ p
     $biggestp = length($p->as_bin())-2;
     print "prime p is a ",$biggestp," bit number or less.\n";
     print "Starting at ", time, "\n";
     $p = Math::BigInt->new(time);
     
     print 'p=',$p->as_hex(),"\r\n";
     $i = 0;
 
 DECREMENT:
     if ($p->is_odd()) {
         $p->brsft($one);
     }
     else {
         $p->brsft($one);
         $p->bxor($tap);
     }
 
 ITERATE:
     if ($i == 65535) {
         print 'p=',$p->as_hex(),"\r\n";
         $i = 0;
     }
     $modtest = $n->copy();
     if ($modtest->bmod($p)->is_zero()){
         print "Great found p and q !!!!!\n";
         print 'p= ',$p->as_hex(),"\n";
         $q = $n->copy();
         $q->bdiv($p);
         print 'q= ',$q->as_hex(),"\n";
         goto SECRETKEY;
     }
     $i++;
     goto DECREMENT;
 
 
 SECRETKEY: # calculate private decryption key  'd'
     print "Calculating secret key d\n";
     $PQ = $p->copy();
     $PQ->bdec();
     $r = $q->copy();
     $r->bdec();
     $PQ->bmul($r);   # $PQ = (p-1)(q-1)
     $d = $e->copy()->bmodinv($PQ);
 
     print 'p= ',$p->as_hex(),"\n";
     print 'q= ',$q->as_hex(),"\n";
     print 'd= ',$d->as_hex(),"\n";
     print 'n= ',$n->as_hex(),"\n";
     print 'e= ',$e->as_hex(),"\n";
```


## We need your help

- Find out what the unknown fields of *TI-Nspire.cer* and
  *TI-Nspire.img* mean
- Find all the differences between the official release of the TI-84
  Plus OS and the OS image used in the TI-Nspire for its emulator. But
  DON'T try to flash it to your TI-84 Plus. The TilEm emulator which
  worked perfectly with all real TI-84 Plus ROMs up to 2.43, just
  freezes with the 2.44 & 2.46 special Nspire ROM dumped with TiLP: this
  is because all Nspire 84+SE OSes contain invalid instructions (to
  perform emulator functions) which will lock any real calculator or
  emulator.