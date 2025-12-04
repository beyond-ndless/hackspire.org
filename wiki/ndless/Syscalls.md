---
title: Syscalls
permalink: /Syscalls/
parent: Ndless
layout: page
---

Syscalls are OS functions exposed by Ndless to C and assembly programs.
This article describes the syscalls currently available with Ndless 3.6.

Your help is needed to make this list grow to a full-fledged library.
Try to find new syscalls,
<a href="/Ndless_features_and_limitations#Syscalls" class="wikilink"
title="test them">test them</a> and share them for integration in
Ndless.

---
1. TOC
{:toc}
---

## [C standard library](http://en.wikipedia.org/wiki/C_standard_library) and some [POSIX functions](http://en.wikipedia.org/wiki/C_POSIX_library)

All standard C functions and some POSIX functions (like dirent.h) are
available via newlib, except:

- `int link(const char *oldpath, const char *newpath)`
- `int fstat(int fd, struct stat *buf)`
- `int kill(pid_t pid, int sig)`

A program using these will compile and link fine, but behave
unexpectedly.

## [Nucleus](http://en.wikipedia.org/wiki/Nucleus_RTOS)

- `int `**`TCT_Local_Control_Interrupts`**`(int mask)`: sets the
  interrupt mask. Returns the previous mask.
- `int `**`NU_Current_Dir`**`(const char *drive, char *path)`: fills in
  `path` with the full path name of the current working directory. Use
  "A:" as *drive*. Returns 0 on success.
- `int `**`NU_Get_First`**`(struct dstat *statobj, const char * pattern)`:
  given a pattern which contains both a path specifier and a search
  pattern, fills in the structure at `statobj` with information about
  the file and sets up internal parts of `statobj` to supply appropriate
  information for calls to `NU_Get_Next`. Returns non-zero if a match
  was not found.
- `int `**`NU_Get_Next`**`(struct dstat *statobj)`: given a pointer to a
  `DSTAT` structure that has been set up by a call to `NU_Get_First()`,
  searches for the next match of the original pattern in the original
  path. Returns non-zero if found and updates `statobj` for subsequent
  calls to `NU_Get_Next`.
- `void `**`NU_Done`**`(struct dstat *statobj)`: given a pointer to a
  `DSTAT` structure that has been set up by a call to `NU_Get_First()`,
  frees internal elements used by the `statobj`.
- `void `**`NU_Set_Current_Dir`**`(const char *name)`: Set the current
  working directory on the default drive.

## TI-Nspire Lua extensions

Ndless can expose C functions to the [TI-Nspire Lua
engine](http://inspired-lua.org/) through Ndless Lua Extensions. Most
functions of the [Lua C API](http://www.lua.org/pil/contents.html#P4)
are available.

This is an experimental feature. Ndless v3.6 or higher is recommended.

The
[luaext](https://github.com/ndless-nspire/Ndless/tree/master/ndless-sdk/samples/luaext)
example of the Ndless SDK is a good start to write your own Ndless Lua
extension.

1.  Write a [Ndless
    program](https://github.com/ndless-nspire/Ndless/blob/master/ndless-sdk/samples/luaext/luaextdemo.c)
    that uses the [Lua C API](http://www.lua.org/pil/26.1.html) to
    register Lua object and exits immediately
2.  Build the Lua extension to a binary file named
    `[extension_name].luax.tns` and drop it (and ask the users to drop
    it) to any folder of the calculator
3.  Write a [TI-Nspire Lua
    script](https://github.com/ndless-nspire/Ndless/blob/master/ndless-sdk/samples/luaext/runluaextdemo.lua).
    It should load the Lua extension with the following statement:

        nrequire [extension_name]
4.  Use from the Lua script any Lua object registered by the Lua
    extension

    *Note : on Ndless 3.1, the 'require' keyword was used, which
    starting from OS v3.6 isn't available anymore.*
5.  Run the Lua script. The 'nrequire' statement will fail if Ndless
    hasn't been installed on the calculator. An error will be displayed
    if the Lua extension isn't found.
6.  Enjoy the TI-Nspire Lua API and OS integration powered by native
    functions

## TI-Nspire specific

### UTF-16 String API

Since v3.1 r672.

- `String` : The type of the dynamic-length strings encoded in utf-16
  format

##### Details

` typedef struct {`
`   char * str;`
`   int len;`
`   int chunck_size;`
`   int unknown_field;`
` } * String;`

##### Init/Dispose functions

- `String `**`string_new`**`()` : Returns a new String structure
- `void `**`string_free`**`(String)` : Frees the String structure

##### Encoding functions

- `char * `**`string_to_ascii`**`(String)` : Returns String converted to
  ascii
- `int `**`string_set_ascii`**`(String, char *)` : Erases the content of
  the String with the given ascii string
- `int `**`string_set_utf16`**`(String, char *)` : Erases the content of
  the String with the given utf16 string

##### String manipulation functions

- `void `**`string_lower`**`(String)` : Lower all characters in the
  String
- `int `**`string_concat_utf16`**`(String, char *)` : Concatenates to
  the String the given utf16 string
- `void `**`string_erase`**`(String, int n)` : Erases characters in the
  String from beginning to `n` (equivalent to substring(n, len))
- `void `**`string_truncate`**`(String, int n)` : Truncates the String
  to `n` chararacters (equivalent to substring(0, n))
- `int `**`string_insert_replace_utf16`**`(String, char *, int start, int end)`
  : Inserts the given utf16 string between `start` and `end` in the
  String by erasing its content. If `end` is -1 it erases the end of the
  String
- `int `**`string_insert_utf16`**`(String, char *, int pos)` : Inserts
  the given utf16 string at `pos` in the String.
- `int `**`string_sprintf_utf16`**`(String, char *, ...)` : Fills the
  String using the utf16 string format and arguments (equivalent to
  sprintf but with utf16 everywhere)

##### Search functions

- `char `**`string_charAt`**`(String, int pos)` : Returns the character
  at `pos` in the String
- `int `**`string_indexOf_utf16`**`(String, int start, char *pattern)` :
  Returns the index of `pattern` in the String starting at `start`.
  Returns -1 if not found
- `int `**`string_last_indexOf_utf16`**`(String, int start, char *)` :
  Returns the last index of `pattern` in the String starting at `start`.
  Returns -1 if not found
- `int `**`string_compareTo_utf16`**`(String, char *)` : Returns 0 if
  the given utf16 string is equal to the String, -1 if it is superior, 1
  if not
- `char * `**`string_substring_utf16`**`(String, char *pattern, int *ptr)`
  : Returns the beginning of the String (in utf16) until `pattern` is
  met (excluded). `ptr` is modified so that it indicates the ending
  index of `pattern` or values -1 if the pattern doesn't exist. In such
  case the whole string is returned.

<!-- -->

- `char * `**`string_substring`**`(String dst, String src, int start, int end)`
  : Writes in the `dst` the resulting substring from `start` to `end`
  (excluded) of `src`. Also returns the utf16 pointer

##### Other UTF-16 functions

- `void `**`ascii2utf16`**`(void *buf, const char *str, int max_size)`:
  converts the UTF-8 string `str` to the UTF-16 string `buf` of size
  `max_size`.
- `void `**`utf162ascii`**`(void *buf, const char *str, int max_size)`:
  Since v3.1 r607. converts the UTF-16 string `str` to the UTF-8 string
  `buf` of size `max_size`.
- `size_t `**`utf16_strlen`**`(const char * s)`: Since v3.1 r607.
  Returns the length in characters (including the finalizing null) of
  the UTF-16 string `s`.

### Graphic Context API

------------------------------------------------------------------------

Since v3.1 r903.

The TI-Nspire Graphic Context (or "ngc") is an API created by TI,
inspired by Java's Graphics2D and using Nucleus GRAFIX functions.

This API is intensively used by the OS internally and is directly
exposed in the <a href="/Lua_Programming#GC_.28as_in_Graphics_Context.29"
class="wikilink" title="TI-Nspire Lua framework">TI-Nspire Lua
framework</a>.

In order to use this API inside your program, use this line:

`#include <ngc.h>`

The Ndless SDK provides an GC-based demo in `ndless-sdk/samples/ngc/`.

##### Global parameters

- `Gc *`**`gui_gc_global_GC_ptr`** is the graphic context used (and
  already allocated/setup) by the OS. In fact, `gui_gc_new` has been
  removed from the syscall list because it made the TI-Nspire
  Clickpad/Touchpad crash, preferring the use of this graphic context.
  Furthermore, the OS seems to use the same gc from the beginning of its
  initialization, why not Ndless then?.

''Notice: It is recommended to save the value of `gui_gc_global_GC_ptr`
in a local variable as such:

`Gc gc = *gui_gc_global_GC_ptr;`

The `gc` is an off-screen buffer. Thus, each function that draws on this
gc will not affect the screen until you use
<a href="/Syscalls#Blit_functions" class="wikilink"
title="gui_gc_blit_to_screen(gc)"><code>gui_gc_blit_to_screen(gc)</code></a>.

The OS tends to blit the screen if it has been updated using a timed
Nucleus task. You can use this principle in your programs but nothing
forces you to do so. Thus, you can freely blit the screen as many times
as you want when running an animation and only refresh part of it using
`gui_gc_blit_to_screen_region(gc, x, y, w, h)` when needed.

##### Init/Dispose functions

- `Gc `**`gui_gc_copy`**`(Gc, int w, int h)` - Allocates a new Gc from
  an existing one copying its parameters, replacing port's width &
  height with the given ones. **Does not copy the screen buffer.**
- `int `**`gui_gc_begin`**`(Gc)` - Initializes graphic port (seems to
  allocate 2 memory areas), may be used before any graphic manipulation.

*Notice: `gui_gc_setRegion` may be used before `gui_gc_begin` in order
to make `gui_gc_clipRect` work with proper coordinates.*

- `void `**`gui_gc_finish`**`(Gc)` - Resets the graphic port parameters
  (seems to free 2 memory areas).
- `void `**`gui_gc_free`**`(Gc)` - Frees the given gc.

##### Set/Get Attributes functions

- `void `**`gui_gc_clipRect`**`(Gc, int x, int y, int w, int h, gui_gc_ClipRectOp op)` -
  Contrains the given `gc` to draw only in a certain area. If `op` is
  GC_CRO_RESET, the other parameters are ignored.
- `void `**`gui_gc_setColorRGB`**`(Gc, int r, int g, int b)` - Changes
  the pen color of the given `gc`
- `void `**`gui_gc_setColor`**`(Gc, int color)` - same as
  `gui_gc_setColorRGB` but using 0xRRGGBB color format
- `void `**`gui_gc_setAlpha`**`(Gc, gui_gc_Alpha)` - Changes the pen
  alpha mode of the given `gc` to non-transparent or semi-transparent.
- `void `**`gui_gc_setFont`**`(Gc, gui_gc_Font)` - Changes the font of
  the given `gc`
- `gui_gc_Font `**`gui_gc_getFont`**`(Gc)` - Returns the current font of
  the given `gc`
- `void `**`gui_gc_setPen`**`(Gc, gui_gc_PenSize, gui_gc_PenMode)` -
  Changes the pen size and mode of the given `gc`
- `void `**`gui_gc_setRegion`**`(Gc, int xs, int ys, int ws, int hs, int x, int y, int w, int h)` -
  Changes the region (or viewport) of the given `gc`. The region
  \<x,y,w,h\> is based on screen coordinates \<xs,ys,ws,hs\>.
  `gui_gc_setRegion` will offset all the drawings and coordinates you
  are using: It can be useful to use this function instead of creating a
  separate gc to draw an inner window.

*Notice: `gui_gc_clipRect` will work with old coordinates as long as you
don't reset the region to the entire screen, for example, by using this
main:*

`int main(void)`
`{`
`  /* Get the gc */`
`  Gc gc = *gui_gc_global_GC_ptr;`

`  /* Initialization */`
`  gui_gc_setRegion(gc, 0, 0, SCREEN_WIDTH, SCREEN_HEIGHT, 0, 0, SCREEN_WIDTH, SCREEN_HEIGHT);`
`  gui_gc_begin(gc);`

`  /* Draw */`
`  gui_gc_clipRect(gc, 10, 10, 10, 10, GC_CRO_SET);`
`  gui_gc_fillRect(gc, 0, 0, SCREEN_WIDTH, SCREEN_HEIGHT); /* Will only draw a 10x10 rectangle */`

`  /* Blit & finish */`
`  gui_gc_blit_to_screen(gc);`
`  gui_gc_finish(gc);`

`  return 0;`
`}`

##### Draw functions

- `void `**`gui_gc_fillArc`**`(Gc, int x, int y, int w, int h, int start, int end)` -
  Fills an arc inside the box \<`x,y,w,h`\>. `start` and `end` have to
  be multiplied by 10.

`/* Disk at <0,0> of size <20,20> */`
`gui_gc_fillArc(gc, 0, 0, 20, 20, 0, 3600);`

- `void `**`gui_gc_fillPoly`**`(Gc, int * points, int count)` - Fills a
  polygon shape. `points` is a table of successive x,y coordinates.

`/* Filled rectangle shape at <100,100> of size <50,50> */`
`int points[] = {100,100, 150,100, 150,150, 100,150, 100,100};`
`gui_gc_fillPoly(gc, points, sizeof (points) / sizeof (points[0]));`

- `void `**`gui_gc_fillRect`**`(Gc, int x, int y, int w, int h)` - Fills
  a rectangle.

`/* Filled rectangle at <100,100> of size <50,50> */`
`gui_gc_fillRect(gc, 100, 100, 50, 50);`

- `void `**`gui_gc_fillGradient`**`(Gc, int x, int y1, int w, int y2, int start_color, int end_color, int vertical)` -
  Fills a line-based gradient from `start_color` to `end_color` (both in
  0xRRGGBB format). As a side-effect, current color is changed to
  `end_color` after the call. *May only work for certain color pairs*
  (it may be cleverer to recreate this function rectangle-based by your
  own).

`/* Black to White vertical gradient at <10,10> of size <50,50> */`
`gui_gc_fillGradient(gc, 10, 10, 50, 50, 0x000000, 0xffffff, 1);`

- `void `**`gui_gc_drawArc`**`(Gc, int x, int y, int w, int h, int start, int end)` -
  Draws an arc insidethe box \<`x,y,w,h`\>. `start` and `end` have to be
  multiplied by 10.

`/* Circle at <0,0> of size <20,20> */`
`gui_gc_drawArc(gc, 0, 0, 20, 20, 0, 3600);`

- `void `**`gui_gc_drawLine`**`(Gc, int x1, int y1, int x2, int y2)` -
  Draws a line from \<`x1,y1`\> to \<`x2,y2`\>.

`/* Line from <0,0> to <320,240> */`
`gui_gc_drawLine(gc, 0, 0, 320, 240);`

- `void `**`gui_gc_drawRect`**`(Gc, int x, int y, int w, int h)` - Draws
  an empty rectangle at \<`x,y`\> of size \<`w,h`\>

`/* Empty rectangle at <100,100> of size <50,50> */`
`gui_gc_fillRect(gc, 100, 100, 50, 50);`

- `void `**`gui_gc_drawString`**`(Gc, char * utf16, int x, int y, gui_gc_StringMode flags)` -
  Draws an UTF-16 Null-terminated string at \<`x,y`\> using `flags`.

`/* "Hello" where <50,50> is actually the top left of it */`
`gui_gc_drawString(gc, "H\0e\0l\0l\0o\0\0", 50, 50, GC_SM_TOP);`

`/* Upside-down "World" at <100,100> */`
`gui_gc_drawString(gc, "W\0o\0r\0l\0d\0\0", 100, 100, GC_SM_DOWN);`

- `void `**`gui_gc_drawPoly`**`(Gc, int * points, int count)` - Draws a
  polygon shape. `points` is a table of successive x,y coordinates.

`/* Empty rectangle shape at <100,100> of size <50,50> */`
`int points[] = {100,100, 150,100, 150,150, 100,150, 100,100};`
`gui_gc_fillPoly(gc, points, sizeof (points) / sizeof (points[0]));`

- `void `**`gui_gc_drawIcon`**`(Gc, int res, int icon, int x, int y)` -
  Draws an icon pre-defined by the OS.

`/* Draw a star at <100,100> */`
`gui_gc_drawIcon(gc, RES_SYST, 255, 100, 100);`

- `void `**`gui_gc_drawSprite`**`(Gc, gui_gc_Sprite *, int x, int y)` -
  Draws a sprite at \<`x,y`\>. Sprite pixels color is affected by
  `gui_gc_setColor[RGB]`

`/* Draw a hand-made gradient at <100, 100> */`
`gui_gc_Sprite s = {.width = 9, .height = 1};`
`s.pixels = (char *) {0x0, 0x22, 0x44, 0x66, 0x88, 0xAA, 0xCC, 0xEE, 0xFF};`
`gui_gc_drawSprite(gc, &s, 100, 100);`

- `void `**`gui_gc_drawImage`**`(Gc, char * TI_Image, int x, int y)` -
  Draws an image in
  <a href="/TI.Image" class="wikilink" title="TI.Image">TI.Image</a>
  format at \<`x,y`\>

`/* Gray image at <100,100> of size <10,10> */`
`gui_gc_Image_header hdr = {.width = 10, .height = 10, .empty = 0, .depth = 2, .enc = 1};`
`hdr.buff_size = hdr.depth * hdr.width;`
`unsigned size = hdr.buff_size * hdr.height;`
`char * img = calloc(size + sizeof (gui_gc_Image_header), 1);`
`memcpy(img, &hdr, sizeof (gui_gc_Image_header));`
`memset(img + sizeof (gui_gc_Image_header), 77, size);`
`gui_gc_drawImage(gc, img, 100, 100);`

##### Metric functions

- `int `**`gui_gc_getStringWidth`**`(Gc, gui_gc_Font, char * utf16, int start, int length)`
  -
- `int `**`gui_gc_getCharWidth`**`(Gc, gui_gc_Font, short utf16 char)` -
- `int `**`gui_gc_getStringSmallHeight`**`(Gc, gui_gc_Font, char * utf16, int start, int length)`
  -
- `int `**`gui_gc_getCharHeight`**`(Gc, gui_gc_Font, short utf16 char)`
  -
- `int `**`gui_gc_getStringHeight`**`(Gc, gui_gc_Font, char * utf16, int start, int length)`
  -
- `int `**`gui_gc_getFontHeight`**`(Gc, gui_gc_Font)` -
- `int `**`gui_gc_getIconSize`**`(Gc, int ressource, int icon, int * w, int * h)`
  -

##### Blit functions

- `void `**`gui_gc_blit_to_screen`**`(Gc gc)` - Blits the screen using
  the given `gc`
- `void `**`gui_gc_blit_to_screen_region`**`(Gc gc, int x, int y, int w, int h)` -
  Blits part of the screen (region \<`x,y,w,h`\>) using the given `gc`
- `void `**`gui_gc_blit_gc`**`(Gc source, int xs, int ys, int ws, int hs, Gc dest, int xd, int yd, int wd, int hd)` -
  Blits (and Stretch with a cubic interpolation if \<`wd,hd`\> is
  different than \<`wd,hd`\>) from rectangle \<`xs,ys,ws,hs`\> of
  `source` to rectangle \<`xd,yd,wd,hd`\> of `dest`.

*Notice: On OS 3.1 CX/CM, when stretching only, there is a bug that
distorts the source rectangle depending on the destination ratio. Even
the OS shows this bug, for example Ctrl+N, 2, Ctrl+Up will not center
the axes whereas on OS 3.2 it will.*

- `void `**`gui_gc_blit_buffer`**`(Gc gc, char * buffer, int xb, int yb, int wb, int hb)` -
  Blits a rectangle \<`xb,yb,wb,hb`\> from `buffer` to the given `gc`.
  Strangely, `buffer` has to be 320 pixels wide in order to have the
  correct offsets. Also, `buffer` has to be allocated according to the
  platform: On Clickpads/Touchpads use 1/2bpp (thus, 160xH), on CM/CX
  use 2bpp (thus 320x2xH).

*Notice: In order to be compatible with Clickpads/Touchpads (1/2 bpp vs
2bpp), `x` has to be a the same parity of the current Region's `x`*

`unsigned size = 320 * 15 * 2;`
`if (!lcd_isincolor()) size >>= 2;`

`char * buff = malloc(size);`
`memset(buff, 0x44, size);`

`/* Draws a green rectangle at <11,20> of size <15,15> */`
`gui_gc_setRegion(gc, 11, 20, 15, 15, 0, 0, 15, 15);`
`gui_gc_blit_buffer(gc, buff, 1, 0, 15, 15);`

`free(buff);`

You may notice the spare space unused. Thus, this method is only
recommended to be use on fullscreen buffers. For example, this method is
used by the internal 3D engine of the OS.

### NavNet

------------------------------------------------------------------------

NavNet is the TI-Nspire specific
<a href="/USB_Protocol" class="wikilink" title="USB Protocol">USB
Protocol</a> and API for calc-to-calc and computer-to-calc transfers. It
allows bidirectional and multi-service exposition with connections
initiated from the calculator or the computer, abstracting the
host/device orientation of USB. The protocol takes care of
acknowledgement, and supports network of calculators built with the
TI-Nspire™ Navigator System. Standard services such as file transfer or
directory listing are available, and custom services can be implemented.

Request and response messages contain arbitrary data of at most 254
bytes. The convention for the
<a href="/USB_Protocol#Services" class="wikilink"
title="standard services">standard services</a> is:

- a single command byte at the beginning of the request packet, with
  optional parameters data
- a single acknowledgement byte at the beginning of the request packet,
  with optional response data, *or* a 2 byte error code

NavNet does not help with the construction of request and response data.

Starting from v3.1 r893 Ndless and the Ndless SDK support a subset of
the NavNet API. Windows (with Computer Link or Student Software) is
required on the computer side, but a Linux/Mac OS X version based on the
[TiLP libraries](http://lpg.ticalc.org/prj_tilp/) may be available in
the future. Bidirectional calc-to-calc transfers don't appear to fully
work yet as there seems to be limitations in NavNet that make one or the
two calculator reboot in some conditions.

The NdlessEditor of the Ndless SDK also shows an example of
computer-to-calc file transfer supporting the *Tools \> Transfer the
program to calc* menu. The source code is available in
`cmd_tools/navnet_cmd`. A startup script is used on the computer side
that puts navnet.dll onto the PATH.

#### The NavNet API

The functions are directly available for the TI-Nspire. For computer
connection, the `navnet.h` and `libnavnet.a` library (wrapper for the
navnet.dll library) in `ndless_pc/` of the Ndless SDK must be used.

All functions return a number \>= 0 in case of success that should be
checked.

- `int16_t TI_NN_Init(const char *opts)`: (computer side) starts the
  NavNet stack with parameters `opts`. You'll typically use `-c 1 -d 0`,
  where the -c and -d switches are the log level respectively for the
  console and the NavNet debug file. Note that the stack cannot be
  started if Student Software is already running, and transfers without
  initialization may interfer with transfers from Student Software.
- `int16_t TI_NN_Shutdown(void)`: (computer side) stops the NavNet
  stack.
- `int16_t TI_NN_RegisterNotifyCallback(uint32_t filter_flags, void (*cb)(void))`:
  (computer side, broken on calculator side) register for connection
  events. A connection event is published even if a device was plugged
  in before the NavNet stack initialization. `filter_flags` should be
  zero. The parameters passed to the callback function are currently
  unknown. The computer program should wait for a notification after a
  `TT_Init()` and before a node enumeration, else the enumeration will
  fail.
- `nn_oh_t TI_NN_CreateOperationHandle(void)`: create an operation
  handle to be used with some NavNet functions.
- `int16_t TI_NN_DestroyOperationHandle(nn_oh_t oh)`: destroy an
  operation handle.
- `int16_t TI_NN_NodeEnumInit(nn_oh_t oh)`: initiates a node (computer
  or device) enumeration. Enumeration is required to find the partner
  computer or device.
- `int16_t TI_NN_NodeEnumNext(nn_oh_t oh, nn_nh_t *nh)`: writes the node
  handle of the next partner computer or device found to `nh`. The first
  node handle can be used when assuming a single node network.
- `int16_t TI_NN_NodeEnumDone(nn_oh_t oh)`: terminates a node
  enumeration.
- `int16_t TI_NN_Connect(nn_nh_t nh, uint32_t service_id, nn_ch_t *ch)`:
  connects to a remote service with service identifier `service_id` (see
  the list of
  <a href="/USB_Protocol#Service_identifiers" class="wikilink"
  title="standard identifiers">standard identifiers</a>, or use a custom
  service id for custom services). Note that this functions only set up
  the client node and doesn't transmit any packet.
- `int16_t TI_NN_Disconnect(nn_ch_t ch)`: disconnects from a remote
  service by sending a disconnection packet.
- `int16_t TI_NN_Write(nn_ch_t ch, void *buf, uint32_t data_size)`:
  writes a packet to a remote service. The payload is `data_size` long
  and pointed to by `buf`.
- `int16_t TI_NN_Read(nn_ch_t ch, uint32_t timeout_ms, void *buf, uint32_t buf_size, uint32_t *recv_size)`:
  waits until a packet is received or a timeout. The size of the packet
  received is written to `recv_size`. Data is written to `buf` of
  `buf_size` bytes long. A packet size can be at most
  `TI_NN_GetConnMaxPktSize()`.
- `uint32_t TI_NN_GetConnMaxPktSize(nn_ch_t ch)`: returns the maximum
  packet size for request and response.
- `int16_t TI_NN_StartService(uint32_t service_id, void *data, void (*cb)(nn_ch_t ch, void *data))`:
  exposes a service. `data` will be passed as the `data` parameter of
  the callback function, and may be NULL. The service identifier musn't
  conflict with
  <a href="/USB_Protocol#Service_identifiers" class="wikilink"
  title="standard identifiers">standard identifiers</a>.
- `int16_t TI_NN_StopService(uint32_t service_id)`: stops exposing a
  service.
- `int16_t TI_NN_PutFile(nn_nh_t nh, nn_oh_t oh, const char *local_path, const char *remote_path)`:
  transfers a file from `local_path` to `remote_path`. The local path
  must be absolute. The remote path is relative to the `/documents`
  TI-Nspire folder. You must first `TI_NN_Connect()` to service 0x4060
  before using it.

The following functions are not yet implemented:

- TI_NN_UnregisterNotifyCallback
- TI_NN_InstallOS
- TI_NN_GetNodeInfo
- TI_NN_GetNodeScreen
- TI_NN_CopyFile
- TI_NN_Rename
- TI_NN_RmDir
- TI_NN_MkDir
- TI_NN_DeleteFile
- TI_NN_GetFileAttributes
- TI_NN_DirEnumDone
- TI_NN_DirEnumNext
- TI_NN_DirEnumInit
- TI_NN_GetFile