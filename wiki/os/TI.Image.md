---
title: TI.Image
permalink: /TI.Image/
layout: page
parent: Operating System
---

The TI.Image format is a type of image used in several places in the
Nspire documents, such as Lua scripts.

The format is some form of bitmap, with almost no compression at all.

The examples here, as the encoded strings visible in .lua files
sometimes show letters and symbols instead of expected "\xxx"
characters. This is because the "\xxx" is actually a representation of a
non-printable character, and when, by chance, the character is printable
the editor displays its value, which can be a letter, a symbol etc.

## Bytes and Data sections

All the data in a Ti.image is formatted in what we call bytes. A byte
can look like this (without the quotes): "`\255`", "`\129`", "`\045`" or
"`.`". If the byte number is reprecentable in ascii (and the nspire
supports that char), it is allowed to just use the ascii version as
byte, to lower the size of the image. For example, you can change
"`\046`" to "`.`".

A data section is a group of bytes used to represent some header data or
a pixel. For example, it might look like this: "`\255\123`". What is
very important to understand is that the data is not written logically,
in fact, the position of the bytes is swapped. For example, let's take a
picture with the width 320, and you want to convert the width size to
put in your header:

1\. You first convert the number to binary: `320 --> 101000000`.
2.Then you have to add the extra zeros to make it fit in the header
(width is 4 bytes):
`00000000000000000000000101000000`
3.This is then splited in four (4 bytes):
`00000000 - 00000000 - 00000001 - 01000000`
4.Then converted back to decimal:
`000 - 000 - 001 - 064`
You would expect to put this then as "`\000\000\001\064`" in the header,
but no, you have to swap it: "`\064\001\000\000`"

You can also change the "`\064`" in the header to "`@`", since that is
its ASCII representation. Finally, the data would look like this:
"`@\001\000\000`"

## Header

The header is 20 "bytes" long, and can be devided in the following 6
data sections:
`[IMAGE WIDTH, 4 bytes][IMAGE HEIGHT, 4 bytes][EMPTY, 4 bytes][IMAGE BUFFER SIZE, 4 bytes][IMAGE DEPTH, 2 bytes][UNKNOWN, 2 bytes]`

The header is the first data of the Ti.image format.

`[IMAGE WIDTH, 4 bytes]` : This is the width of your image, and is four
bytes long.
`[IMAGE HEIGHT, 4 bytes]` : This is the height of your image, and is
four bytes long.
`[EMPTY, 4 bytes]` : This space is just filled with zeroes
("\000\000\000\000"), and is four bytes long
`[IMAGE BUFFER SIZE, 4 bytes]` : Amount of bytes in one buffer row ,
this is normally 2\*width. 4 bytes long.
`[IMAGE DEPTH, 2 bytes]` : This is the image, normally just 16 ,two
bytes long (actaully the image is 15 bit, but if you count the alpha
channel it is 16)
`[UNKNOWN, 2 bytes]` : Unknown, maybe compression type? Two bytes long,
and has 1 as default value.<bt>

Example header and description:
`.\000\000\000\018\000\000\000\000\000\000\000\092\000\000\000\016\000\001\000`

The first four bytes ("`.\000\000\000`") shows that the width is 46px.
The next four bytes ("`\018\000\000\000`") shows that the height 18px.
The next four bytes is empty.
The four bytes ("`\092\000\000\000`") after this is the buffer row size
(2\*width)
The next two bytes ("`\016\000`") is the image depth. The next two bytes
("`\001\000`") are unkown.

## Pixel data

The pixel data commes directly after the header. Each pixel has a data
section of two bytes. It contains the rgb values, and the alpha channel.

Here is how it is structured (in binary):

`A - RRRRR - GGGGG - BBBBB`
Example : `1 - 11111 - 10001 - 00000`
A is the alpha channel, 1 for not transparent, 0 for fully transparent.
R, G, and B or the color levels.

To convert this to valid pixel data it has to be cut in two:

`ARRRRRGG - GGGBBBBB`
`11111110 - 00100000`

`   ||          || Converter to decimal`
`   \/          \/`
`   `
`   254        032`

`And finally, swapped:`

`"\032\254"`

This is then added to the data buffer.

## Examples

**`\007\000\000\000\008\000\000\000\000\000\000\000\014\000\000\000\016\000\001\000`**`alalalalalalalal\000\244\000\244al\000\244\000\244al\000\244\000\244\000\244\000\244\000\244\000\244\000\244\000\244\000\244\000\244\000\244\000\244\000\244\000\244al\000\244\000\244\000\244\000\244\000\244alalal\000\244\000\244\000\244alalalalal\000\244alalalalalalalalalal5`

## External Links

>AdRiWeB made
[a video](http://www.youtube.com/watch?v=YQhrMHNkL3A) showing some
direct manipulations of the TI-Image, in order to understand better the
format in general.

Jim Bauwens made [a image
previewer](http://bwns.be/jim/tiviewer.html), [a sprite
creator](http://bwns.be/jim/sprite.html) and a basic [python image
converter](http://bwns.be/jim/convert.py) (you need PythonMagick to run
this)

## References

<http://en.wikipedia.org/wiki/Highcolor>