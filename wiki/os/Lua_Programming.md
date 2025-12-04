---
title: Lua Programming
permalink: /Lua_Programming/
parent: Operating System
layout: page
---

The TI-Nspire allows, since OS v3.0, users to run Lua scripts.

## Prerequisites

Lua is only supported starting from OS v3.0.1. You can create Lua
applications using the built-in script editor in the TI-Nspire computer
software or one of the following third-party tools:

**For any OS**:

- [Luna](https://github.com/ndless-nspire/Luna) (cross-platform) [forum
  thread](https://www.omnimaga.org/news/'luna'-is-here-and-converts-your-lua-files-into-3-0-2-compatible-tns-files/)

**For OS \< 3.0.1**: these tools won't work with the OS 3.0.2 since the
format used is now blocked

- [LUAtoTNS.sh](http://www.ticalc.org/archives/files/fileinfo/437/43704.html)
  (bash script, works on Linux, Mac OS and Unix, or Windows + Cygwin or
  Windows + MSYS).
- [maketns.py](http://www.mirari.fr/KbOD) (Python script,
  cross-platform)
- [lua2ti](https://www.omnimaga.org/files/User-Contributed-Calculator-Games-amp-Development-Tools/TI-Nspire-amp-TI-Nspire-CAS/TI-Nspire-Programming-Tools/lua2ti.zip)
  (Windows + .NET Framework v4.0)
- [lua2ti_linux](https://www.omnimaga.org/files/User-Contributed-Calculator-Games-amp-Development-Tools/TI-Nspire-amp-TI-Nspire-CAS/TI-Nspire-Programming-Tools/lua2ti_linux.zip)
  (Linux (No longer requires Mono, it is now a native Linux binary!))

You may also want to install Lua [on your
computer](https://www.lua.org/download.html): `luac -p` can be used to
check the syntax of your script before running it on a TI-Nspire or an
emulator.

## API

The documentation is now maintained in the "[Inspired Lua
Wiki](http://wiki.inspired-lua.org)". Please go there in order to have a
full detailed documentation based on TI's information.

## XML

### Basic structure

``` xml
<wdgt
    xmlns:sc="urn:TI.ScriptApp" type="TI.ScriptApp" ver="1.0">

    <sc:mFlags>1024</sc:mFlags>
    <sc:value>8</sc:value>
    <sc:cry>0</sc:cry>
    <sc:legal>none</sc:legal>
    <sc:schk>false</sc:schk>

    <sc:md>
        <sc:mde name="_VER" prop="134217728">1:1</sc:mde>
        <sc:mde name="TITLE" prop="2147549184">Hello</sc:mde>
        <sc:mde name="TARAL" prop="134217728">2:5</sc:mde>
    </sc:md>

    <sc:script version="33882629" id="1">
    -- Lua script, XML encoded
    </sc:script>

    <sc:state>
        return {} -- Lua save state
    </sc:state>
</wdgt>
```

### Script tags

#### sc:script

| attribute | description                                                    |
|-----------|----------------------------------------------------------------|
| version   | taral and reqal versions, one byte per digit (for older OSes?) |
| id        | 1 = normal script, 2 & 3 = Question and Answer                 |

### Script metadata configuration

| Name | Prop | Description | Example |
|----|----|----|----|
| TARAL | 134217728 | TARget ApiLevel | `<sc:mde name="TARAL" prop="134217728">2:2</sc:mde>` |
| REQAL | 134217728 | REQuired ApiLevel | `<sc:mde name="REQAL" prop="134217728">2:0</sc:mde>` |
| _VER | 134217728 | TNS Version | `<sc:mde name="_VER" prop="134217728">1:1</sc:mde>` |
| TITLE | 2147549184 | ScriptApp title | `<sc:mde name="TITLE" prop="2147549184">Hello</sc:mde>` |
| \- | 270532608 | Lua meta script | `<sc:mde name="TEST" prop="270532608">--lua code</sc:mde>` |
| BLERQ | 134217728 | BLE ReQuire - require ble libs | `<sc:mde name="BLERQ" prop="134217728">1</sc:mde>` |
| _FFunc | \- | FailMe function (error catching) | Not known |
| _TRACE | \- | Trace/log? | Not known |