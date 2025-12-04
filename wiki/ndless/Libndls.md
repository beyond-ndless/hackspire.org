---
title: Libndls
permalink: /Libndls/
parent: Ndless
layout: page
---

*libndls* is a set of macros and functions available as a [static
library](https://en.wikipedia.org/wiki/Static_library) when building
with Ndless. The library is automatically linked by `nspire-gcc` and
`nspire-g++` without "-nostdlib".

These definitions are available in the latest version of the Ndless SDK
and should work on every Ndless version.

---
1. TOC
{:toc}
---

## Ndless

- `void assert_ndless_rev(unsigned required_rev)`: Since v3.1 r617.
  Displays a popup asking to update Ndless if the Ndless revision on the
  calculator is less than *required_rev*, and exits the program. Does
  nothing if the revision is greater or equal than *required_rev*. You
  should call this function at the beginning of your program if it is
  using syscalls recently added to Ndless, or libndls functions which
  depend on recent syscalls. See
  <a href="/Ndless_features_and_limitations#Checking_Ndless&#39;s_version"
  class="wikilink" title="Checking Ndless version">Checking Ndless
  version</a> for more info. Note that this function works without v3.1
  r617 or higher.
- `const char *NDLESS_DIR`: (since v3.1 r797) full path to the "ndless"
  directory in the documents folder ("/documents/ndless")

## LCD API

- `scr_type_t lcd_type()`: Returns the native LCD type (see the list
  below). Using this is always the fastest way to display a frame.
- `bool lcd_init(scr_type_t type)`: Set the LCD mode. You need to call
  this before you can use lcd_blit with the same `scr_type`. You also
  need to call `lcd_init(SCR_TYPE_INVALID)` before using any of the
  functions in the UI section and before exiting the program.
- `void lcd_blit(void* buffer, scr_type_t type)`: Blit the buffer to the
  screen.

Available screen types (as of r2004):

- SCR_320x240_4: 4bit grayscale. Native on classic calcs.
- SCR_320x240_8: 8bit paletted mode.
- SCR_320x240_16: RGB444
- SCR_320x240_565: RGB565. Native on CX before HW-W
- SCR_240x320_565: RGB565. Native on CX HW-W

## UI

- `void show_msgbox(const char *title, const char *msg)`: show a message
  box, with a single button OK"
- `unsigned int show_msgbox_2b(const char *title, const char *msg, const char *button1, const char *button2)`:
  since v3.1. show a message box with two buttons with custom labels.
  Return the number of the button pressed (1 for the first button).
- `unsigned int show_msgbox_3b(const char *title, const char *msg, const char *button1, const char *button2, const char *button3)`:
  since v3.1. show a message box with three buttons with custom labels.
  Return the number of the button pressed (1 for the first button).
- `int show_msg_user_input(const char * title, const char * msg, char * defaultvalue, char ** value_ref)`:
  since v3.1 r607. Request popup. Usage:
  `char * value; show_msg_user_input("title", "msg", "default", &value)`.
  `value` must be freed with `free()` once used. Returns the length of
  the value, or -1 if an empty text was entered or escape was pressed.
  Some issues fixed in r634 with the new String API.
- `int show_1numeric_input(const char * title, const char * subtitle, const char * msg, int * value_ref, int min_value, int max_value)`:
  since v3.1 r607. Request popup for one numeric input. Caution, values
  like -1 or 0 for *min_value* will cancel the popup. Returns 1 if OK, 0
  if cancelled.
- `int show_2numeric_input(const char * title, const char * subtitle, const char * msg1, int * value1_ref, int min_value1, int max_value1, const char * msg2, int * value2_ref, int min_value2, int max_value2)`:
  since v3.1 r607. Request popup for two numeric inputs. Caution, values
  like -1 or 0 for *min_value* will cancel the popup. Returns 1 if OK, 0
  if cancelled.
- `void refresh_osscr(void)`: since v3.1. Must be called at the end of a
  program that creates or deletes files, to update the OS document
  browser.

## Keyboard

- `BOOL any_key_pressed(void)`: non-blocking key press test. Return
  `TRUE` if one or more keys are being pressed.
- `BOOL isKeyPressed(key)`: non-blocking key press test. `key` must be
  one of the `KEY_NSPIRE_*` constants defined in keys.h.
- `BOOL on_key_pressed(void)`: since v3.1. Non-blocking ON key press
  test. Caution, key scanning is time consuming and may hurt the
  performance of programs which needs high reactivity. You should skip
  key scanning regularly in the main loop of a game.
- `void wait_key_pressed(void)`: block until a key is pressed. Changing
  the timer frequency have effects on the latency of this function.
- `void wait_no_key_pressed(void)`: block until all the keys are
  released. Changing the timer frequency have effects on the latency of
  this function.
- `touchpad_info_t *touchpad_getinfo(void)`: return information on the
  Touchpad area such as its dimension. Return `NULL` if not a TI-Nspire
  Touchpad. See `include/libndls.h` for the definition of
  `touchpad_info_t`.
- `int touchpad_scan(touchpad_report_t *report)`: check user
  interactions with the Touchpad area and writes to `report`. See
  `include/libndls.h` for the definition of `touchpad_report_t`.
  `report->contact` and `report->pressed` are always `FALSE` on
  TI-Nspire Clickpad. See `src/arm/tests/ndless_tpad.c` for an example
  of use.
- `int get_event(struct s_ns_event*)`: since r721. Poll for an OS event.
  See `struct s_ns_event` in nucleus.h.
- `void send_key_event(struct s_ns_event* eventbuf, unsigned short keycode_asciicode, BOOL is_key_up, BOOL unknown)`:
  since r721. Simulate a key event.
- `void send_click_event(struct s_ns_event* eventbuf, unsigned short keycode_asciicode, BOOL is_key_up, BOOL unknown)`:
  since r750. Simulate a click event. keycode_asciicode=0xFB00: single
  click, keycode_asciicode=0xAC00: drag.
- `void send_pad_event(struct s_ns_event* eventbuf, unsigned short keycode_asciicode, BOOL is_key_up, BOOL unknown)`:
  since r750. Simulate a cursor move. Set the cursor coordinates in
  eventbuf, and keycode_asciicode to 0x7F00.

## Filesystem

`int enable_relative_paths(char **argv)`: since r820. Call before using
`fopen()` and other file-related functions with paths relative to the
current program. `argv` should be the `argv` parameter of the `main()`
function. Returns -1 on error, 0 if success.

## CPU

- `void clear_cache(void)`: flush the data cache and invalidate the
  instruction and data caches of the processor. Should be called before
  loading code dynamically, after a code patch or with self-modifying
  code.

## Hardware

- `BOOL is_classic`: since v3.1. `TRUE` on classic TI-Nspire. This is
  the preferred way to check CX/CM-specific features: *if (is_classic)
  classic_code; else cx_code;*
- `BOOL is_cm`: since v3.1 r863. `TRUE` on TI-Nspire CM/CM-C.
- `BOOL has_colors`: since v3.1. `TRUE` if the device has a screen in
  color.
- `BOOL is_touchpad`: `TRUE` on a TI-Nspire Touchpad or on a TI-Nspire
  CX.
- `unsigned hwtype()`: 0 on classic TI-Nspire, 1 on TI-Nspire CX.
- `IO()`: select an I/O port whose mapping depends on the hardware type.
  Fo example `IO(0xDC00000C, 0xDC0000010)` will return 0xDC00000C on
  classic TI-Nspire, 0xDC0000010 on CX. Returns a *volatile unsigned\**.

## Time

- `void idle(void)`: switch to low-power state until the next interrupt
  occurs. The use of this function is encouraged when waiting in loops
  for an event to save the batteries. Changing the timer frequency have
  effects on the latency of this function.
- `void msleep(unsigned ms)`: delay for a specified amount of time in
  ms. The CPU is regularly switched to low-power state while blocking.
  Note that the `sleep` function has been removed.

## Configuration

- `void cfg_register_fileext(const char *ext, const char *prgm)`: (since
  v3.1 r797) associate for Ndless the file extension `ext` (without
  leading '.') to the program name `prgm`. Does nothing if the extension
  is already registered.

## Debugging

- `void bkpt()`: software breakpoint. Make the emulator halt and open
  the debugger. Remove before transferring to a calculator, as it will
  crash if executed.

## Deprecated

### Common types

Deprecated. Use \#include \<stdbool.h\> instead.
`typedef enum bool {FALSE = 0, TRUE = 1} BOOL;`

### Old screen API

The following section is only valid if using the OLD_SCREEN_API define.

- `SCREEN_BASE_ADDRESS`: address of the screen buffer. Read from the LCD
  controller, caching it is recommended.
- `SCREEN_BYTES_SIZE`: size of the screen buffer. Calculated depending
  on the color mode advertised by the LCD controller, caching it is
  recommended as long as the mode isn't changed.
- `SCREEN_WIDTH`: screen width in pixels
- `SCREEN_HEIGHT`: screen height in pixels
- `void clrscr(void)`: clear the screen
- `BOOL lcd_isincolor(void)`: since v3.1. Check the current LCD mode.
- `void lcd_incolor(void)`: since v3.1. Switch to color LCD mode.
- `void lcd_ingray(void)`: since v3.1. Switch to grayscale LCD mode.
- `volatile unsigned *IO_LCD_CONTROL`: since v3.1. LCD control register
  of the LCD controller