---
title: Developer Unit
permalink: /Developer_Unit/
parent: Operating System
layout: page
---

## Developer Unit

The Nspire calculators all have a "secret" feature set meant for usage
on Developer Unit calculators. Thanks to reverse engineering, a lot of features
are accessible from any calculator, and not just Texas Instruments ones. Please note that using these does not require Ndless or any other jailbreak, and they cannot grant you the ability to cheat.

## Folder

Most everything developer-related is stored within the "Developer Unit" folder.
Users are required to create this folder at the root of the File Browser.

There are a total of 12 valid files that one can place within these folders.
Any content within the TNS files of these are just renamed TXT files, and cannot be
edited on a stock Nspire calculator.

| Filename              | Description                                                             |
|-----------------------|-------------------------------------------------------------------------|
| ini.tns               | The primary configuration file                                          |
| removedevfolder.tns   | Upon creation, remove the Developer Unit folder                         |
| prap.tns              | Swaps `/wlan/prap.lst` with this, copying the old one to `get_prap.tns` |
| devunit.cer.tns       | Contains the devcert, will be installed if the device doesn't have one  |
| deletedevcert.tns     | Upon creation, deletes the developer certificate                        |
| devcert_on.tns        | Upon creation, enables usage of the developer certificate               |
| devcert_off.tns       | Upon creation, disables usage of the developer certificate              |
| ctb_freq.tns          | Docking station related. Controls the clock frequency for EMI.          |
| tt_start.tns          | Transport test start, for packet transmission testing.                  |
| tt_stop.tns           | Transport test stop.                                                    |
| help.tns              | Upon creation, deletes itself and creates the `help/` folder with help! |
| instructions.tns      | Contains instructions for Robot testing framework                       |
| debugclear.tns        | Cleans up logging                                                       |
| binary_data.tns       | Binary-log download file                                                |
| te_mlpdev_err.log.tns | Developer-service error log                                             |

## INI files

Some of the above files are "INI files" meaning they actually need to have content in them to work.
This content uses a strange key/value system powered by a custom parser (it's just Flex) in the OS.

The parser accepts a few valid types for values, those being strings, hex, integers, IPv4
addresses, MAC addresses, filenames, BSOD settings, log destination, log component, log level,
log mask, and log header. Do note that these are just internal type names, and usually you'll
just have strings/integer/hexadecimal.

Each type:
```
example_string=hello_world
example_hex=0x1234
example_dec=2
example_ip=192.168.1.2
example_mac=00:11:22:33:44:55
example_file=debug.txt
example_bsod=1 # decimal or hex
example_log_destination=0x1
example_log_component=0x2
example_log_level=0x4
example_log_mask=0x2,0x1234,0xff # component ID, component mask, logging mask
example_log_header=0x1
```


By creating `help.tns` in the Developer Unit folder, a new `help` folder will appear
containing example TNS files as listed in the above folder, with each INI property listed.

Example output for `ini.tns` (truncated):
```
#INI Help: ini.tns

enable_time = 0
hibernate_time = 345600
apd_time = 180
battery_check = 86400
auto_wakeup = 0
wireless_timeout = 360000
log_destination = 0x10
log_header = 0x8b
log_cache = 0
os_monitor_enable = 0
os_monitor_interval = 0
os_monitor_features = 0x0
os_monitor_pool4leaks = 0
crash_dump_enable = 0
lockup_monitor_enable = 0
ctb_freq_disable = 0
ctb_freq_index = 255
ctb_freq_always_normal = 0
robot_enable = 0
dialog_enable = 0
```

The files that have valid INI content are `ini.tns`, `tt_start.tns`, and `ctb_freq.tns`.

Here are the fields in `tt_start.tns`:

| Field Name        | Type    | Description                                   |
|-------------------|---------|-----------------------------------------------|
| tt_select_timeout | Integer | Seconds to wait for a response before timeout |
| tt_backoff_time   | Integer | Delay before sending test traffic             |
| tt_protocol       | Integer | `0` for UDP, `2` for TI NavNet                |
| tt_min_pkt        | Integer | Starting packet size in bytes or low bound    |
| tt_max_pkt        | Integer | Ending packet size in bytes or high bound     |
| tt_random         | Integer | Enable randomizing packet sizes               |
| tt_continuous     | Integer | Enable infinite test duration                 |

Interestingly, in the dialogs (discussed later), `tt_protocol` is said
to support TCP/RAW as well, and it even has a 4th testing mode for TI Connector.
TCP/RAW are not present in the OS init, and `4` is outside of `tt_protocol`.
The connector test mode can only be accessed from the Transport Test dialog (discussed later).

Here are the fields in `ctb_freq.tns`:

| Field Name             | Type    | Description                                 |
|------------------------|---------|---------------------------------------------|
| ctb_freq_disable       | Integer | `1` disables the INI override               |
| ctb_freq_index         | Integer | Force a particular "profile" (undocumented) |
| ctb_freq_always_normal | Integer | Force normal clock profile                  |

## ini.tns

`ini.tns` contains the main configuration data for the entire calculator, as well
as the juicy debug utilities.

| Property | Type / accepted values | Description |
|---|---|---|
| `enable_time` | Decimal: `0–1` | Enables the custom power-management timers below |
| `hibernate_time` | Decimal: `0–500000` | Hibernate alarm delay, in seconds. (applied when `enable_time=1`) |
| `apd_time` | Decimal: `0–7200` | Automatic Power Down delay, in seconds (applied when `enable_time=1`) |
| `battery_check` | Decimal: `0–500000` | Battery-check alarm interval, in seconds (applied when `enable_time=1`) |
| `auto_wakeup` | Decimal: `0–600` | Automatic wakeup alarm delay, in seconds (applied when `enable_time=1`) |
| `wireless_timeout` | Decimal: `0–30000` | Wireless power-management timeout, in seconds (applied when `enable_time=1`) |
| `log_destination` | Hex: `0x0–0xFFFFFFFF` | Selects logging destinations. The INI callback additionally sets bit `0x10`. |
| `log_header` | Hex: `0x0–0xFFFFFFFF` | Controls which header information is included in log output. |
| `log_cache` | Decimal: `0–1` | Controls the logging cache flag. |
| `os_monitor_enable` | Decimal: `0–1` | Enables the OS diagnostic monitor which logs to `NspireLogs` |
| `os_monitor_interval` | Decimal: `0–3600` | OS monitor sampling interval, in seconds |
| `os_monitor_features` | Hex: `0x0–0xFFFFFFFF` | Selects diagnostic features collected by the OS monitor |
| `os_monitor_pool4leaks` | Decimal: `0–6` | Memory-pool selector intended for monitor leak checking |
| `crash_dump_enable` | Decimal/hex | Enables the BSOD only with `0x8008`! |
| `lockup_monitor_enable` | Decimal: `0–1` | Enables the "lockup monitor" |
| `ctb_freq_disable` | Decimal: `0–1` | See `ctb_freq` table |
| `ctb_freq_index` | Decimal: `0–255` | See `ctb_freq` table |
| `ctb_freq_always_normal` | Decimal: `0–1` | See `ctb_freq` table |
| `robot_enable` | Decimal: `0–1` | Enables the Robot automation/testing framework |
| `dialog_enable` | Decimal: `0–1` | Enables recognition of special diagnostic-dialog key sequences (see below!) |
| `ar6k_testmode` | Decimal: `0–5` | Selects the AR6K WLAN test/firmware mode  |
| `ar6k_tx_mode` | Decimal: `0–5` | Transmit-test operating mode selector |
| `ar6k_tx_freq` | Decimal: `0–5825` | Transmit-test frequency parameter (in MHz) |
| `ar6k_tx_datarate` | Decimal: `0–28` | Transmit data-rate index |
| `ar6k_tx_txpwr` | Decimal: `0–15` | Transmit-power parameter, depends on transmit mode |
| `ar6k_tx_antenna` | Decimal: `0–2` | Transmit antenna selector |
| `ar6k_tx_enani` | Decimal: `0–1` | Enables the transmit test’s Adaptive Noise Immunity setting |
| `ar6k_tx_scrambleoff` | Decimal: `0–1` | Requests that scrambling be disabled during transmit testing |
| `ar6k_tx_aifsn` | Decimal: `0–255` | Arbitration Inter-Frame Space Number used by the transmit test |
| `ar6k_tx_pktsz` | Decimal: `0–512` | Transmit-test packet size, in bytes |
| `ar6k_tx_txpattern` | Decimal: `0–5` | Test-payload pattern selector |
| `ar6k_tx_shortguard` | Decimal: `0–1` | Requests a short guard interval for applicable transmit modes |
| `ar6k_tx_numpackets` | Decimal: `0–1024` | Packet-count parameter for the transmit test |
| `ar6k_tx_wlanmode` | Decimal: `1–4` | WLAN modulation/channel-mode selector for Tx test |
| `ar6k_tx_testcmdid` | Decimal: `0–1` | Tx test command identifier. |
| `ar6k_rx_mode` | Decimal: `0–3` | Rx test operating mode selector. |
| `ar6k_rx_freq` | Decimal: `0–5825` | Rx test frequency parameter in MHz |


## Debug Dialogs

By setting `dialog_enable = 1`, you can enable debug "dialogs".
These dialogs are accessed by inputting magic keycodes, a table of which is available below.

Enter a magic keycode by pressing `COMMA` to clear the buffer, typing in your
key sequence, and pressing `RETURN` (not `ENTER`, its below the flag button) to
do the test.

| Sequence | Description |
|----------|-------------|
| `www` | Opens wireless-cradle diagnostics |
| `wwx` | Opens wireless daignostics |
| `wwl` | Logs the preferred access-point list and MAC addresses |
| `wwp` | Opens the preferred access-point list dialog |
| `dpl` | Clears preferred APs and deletes `/wlan/prap.lst` |
| `wwr` | Requests WLAN reconnection |
| `wpo` | No idea? Prints `PING_MISS` event |
| `utp` | Opens WLAN transport-test controls |
| `utr` | Opens WLAN transport-test results |
| `time` | Opens time-debug controls |
| `con` | Opens WLAN connector status |
| `wpn` | Sends test AP name `TI-NAV-N-0009371F0B0A` |
| `wpa` | Sends test AP name `TI-NAV-S-0009371F0B0A` |
| `bbb` | Opens battery-status diagnostics |
| `adc` | Opens ADC/power-channel readings |
| `man` | Displays manufacturing, model, and hardware-flag information |
| `apd` | Opens Automatic Power Down controls |
| `idn` | Enables docking-slot identification LEDs |
| `idf` | Disables docking-slot identification LEDs |
| `pwr` | Displays power-management statistics |
| `emi` | Opens the clock-frequency profile selector |
| `fedex` | No action |
| `jamie` | No action |
| `tst` | Opens Wi-Fi test-automation connect/disconnect controls |
| `dir` | Walks `/` with argument `20` and logs directory results/counts |
| `osm` | Signals the OS system monitor, if already initialized |
| `stk` | Crashes the calculator by doing a stack overwrite |
| `tp0` | Selects touchpad mode `0`, clearing its enable field |
| `tp1` | Selects touchpad configuration profile `1` |
| `tp2` | Selects touchpad configuration profile `2` |
| `tp3` | Selects touchpad configuration profile `3` |
| `srs` | Starts Robot Student with a requested 100 ms interval |
| `sss` | Stops Robot Student |
| `rst` | Opens Robot Service Status, subject to `robot_enable` |
| `dex` | Crashes by writing `0x0` to `0xDEAD0000` |
| `drs` | Runs the protected-string-copy bounds test |
| `ovr` | Corrupts heap allocation markers, then frees the allocations |
| `spt` | Positive semaphore-lifetime test, deletes an embedded semaphore before freeing its storage |
| `snt` | Negative semaphore-lifetime test, frees storage while its semaphore remains registered |
| `rct` | Runs a WLAN stop/start test, waits 1,600 ticks, then stop |
| `bad` | Prints all bad blocks to logs |
| `dac` | No action |
| `dap` | No action |
| `dat` | No action |
| `din` | Requests something from a driver |
| `dla` | Triggers a filesystem assertion failure |
| `ex1` | Exam test one, unknown |
| `ex2` | Exam test two, unknown |
| `hrt` | Wireless heartbeat |
| `cnd` | Transmits something |
| `dfi` | Init Debug FS |
| `dfd` | Test Debug FS |
| `adt` | No action |
| `lfs` | List filesystems to logs |

## Developer Certificates

Texas Instruments has historically given out "developer certificates"
to calculators, which allow the usage of developer/beta operating systems
to be installed and used on the calculator, as well as some other neat things.

These developer certificates follow the standard TI software format, not discussed here.

They contain a list of valid device IDs. To view a calculator's device ID, press About in
the Status setting screen.

```
Example ID: 1D2000000000021DB1935F37E00

In this ID:
0x1d is the low product ID from Manuf
    On CX II: 0x1C is CAS, 0x1D is regular, 0x1E is -T
    On CX: 0x0F is CAS, 0x10 is regular, 0x11 is CM CAS, and 0x12 is CM
    On clickpad/touchpad: 0x0C is CAS, 0x0E is regular.
0x20 is a field from the 0x5200 of Manuf, unknown
0x00000000 is the low 0xFFFFF of the second efuse at PMU+0x80C
    ORd with some value if CX
0x0021db19 is the values of the first efuse, a constant in firebird
0x35f37e00 is a CRC32 of the above
```

Two types exist: developer unit and developer boot. The mode of the current
certificate can be changed in the secret boot menu (accessed by the `Doc` +
`Enter` + `EE` maintainence mode and pressing `9`)

<div style="display: flex; gap: 16px;">
    <img src="../../assets/images/dev_unit.png" alt="DevUnit boot screen" width="50%">
    <img src="../../assets/images/dev_boot.png" alt="DevBoot boot screen" width="50%">
</div>

Developer Boot is superior to Developer Unit, allowing for a developer OS to be launched.
It switches the keyset used by the bootloader to developer keys during OS decryption.

Developer Unit controls some RSA key/encryption key selection as well as enabling minOS
resetting.
