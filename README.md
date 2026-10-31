# OpenOCD notes

Command reference:

## The three pieces

OpenOCD (Open On-Chip Debugger) is a program on our PC that talks to a debug
probe over USB, and the probe talks to the microcontroller over SWD. It does not
care which IDE we use, it only needs to know three things:

- **Interface:** the probe in our hand (ST-Link, J-Link, CMSIS-DAP), each one
has a file in "interface/", like "interface/stlink.cfg".

- **Transport:** the wire protocol between the probe and the chip, for the STM32
boards, it is easy to use the SWD that has only two wires needed (SWDIO and SWCLK).

- **Target:** the chip on the otherside, it tells OpenOCD the CPU core, the RAM
address and which flash driver to use, is a file like "target/stm32f4x.cfg".

Once connected, openOCD can stay running as a server and open three ports:

| Port | Who connects | What for |
|---|---|---|
| 3333 | arm-none-eabi-gdb | step, breakpoints, load |
| 4444 | telnet | typing OpenOCD commands by hand |
| 6666 | scripts (TCL) | automation, IDEs |

Or, it will runs one command list and exit, like erase/flash.

## Install

In windows and linux can be found the xPack with the binaries, set to path and
see if it works with:

```bash
openocd --version
```

#### Drivers (Windows)

- **ST-Link:** install ST's driver (STSW-LINK009). It is WinUSB based, so
OpenOCD and STM32CubeProgrammer work with it at the same time. If OpenOCD
complains about the firmware version, update it with STLinkUpgrade (also from
ST) or with STM32CubeProgrammer -> Firmware upgrade.

- **J-Link:** OpenOCD reaches the J-Link through libusb, which needs the WinUSB
driver, not SEGGER's default one. Open J-Link Configurator (installed with the
J-Link Software Pack), select the probe, Configure, and switch the USB driver
to WinUSB. SEGGER's own tools keep working with it.

## First connection

With the probe plugged and the board powered:

```bash
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg
openocd -f interface/stlink.cfg -f target/stm32f1x.cfg
```

"-f" loads a config file. OpenOCD looks for it in the current folder first, and
then in its own "scripts/" folder, that is why "interface/stlink.cfg" works from
anywhere.

## Talking with the Chip

With the server running we can connect through telnet localhost in port 4444.

Here some commands that can be run in the telnet session:

```
> halt                      # stop the CPU, shows PC and xPSR
> reg                       # all core registers
> mdw 0x08000000 4          # memory display word: 4 words from flash start
> step                      # one instruction
> resume                    # run again
> reset halt                # reset and stop at the first instruction
> reset run                 # reset and let it run
> flash banks               # flash regions OpenOCD knows about
> flash info 0              # sectors/pages of bank 0 and protection
> exit                      # leaves telnet, OpenOCD keeps running
```

The first two words of the flash are the vector table, the initial stack pointer
(0x2000xxxx) and the Reset_Handler Address. If "mdw 0x0800000" shows "ffffffff
ffffffff", for stm32, the chip is empty.

## Flashing

Flashing does not need the server and the telnet, OpenOCD accepts commands
with "-c" and runs them after the config files:

```bash
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg -c "program build/blink.elf verify reset exit"
```

- **program:** init, halt, erase the needed sectors and write
- **verify:** read back and compare
- **reset:** runs the new firmware
- **exit:** close OpenOCD at the end (otherwise it stays as a server)

The .elf (and the .hex) carry the address inside, so OpenOCD knows where each
section section goes. The **.bin** is only bytes, so we must give the address:

```bash
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg -c "program build/blink.bin 0x08000000 verify reset exit"
```

A .bin flashed without the address goes to 0x00000000. Prefer the .elf whenever
possible.

## Our own config files

Typing "-f interface/... -f target/..." with the extra settings each time gets
old fast. A config file is just those commands in a file, and it is the same
language (TCL) that we type in telnet. The [configs/](configs/) folder has one
per board and probe. You can use with:

```bash
openocd -f configs/stm32f411-stlink.cfg -c "program build/blink.elf verify reset exit"
```

The **order matters**: interface first, then transport, then anything that
the target file reads (like "set CPUTAPID"), then the target, and only at the
end what overrides the target (like "adapter speed", which the target files
also set).

## ST-Link vs J-Link

From OpenOCD's side, changing the probe is changing one line, with one detail:

```tcl
# ST-Link
source [find interface/stlink.cfg]
transport select hla_swd

# J-Link
source [find interface/jlink.cfg]
transport select swd
```

The ST-Link is a "high level adapter" (hla), it does the SWD work in its own
firmware and OpenOCD only asks for "read this address", so its transport is
"hla_swd". The J-Link gives OpenOCD control of the raw SWD, so the transport is
plain "swd', and it must be explicit because the J-Link default is JTAG.

OpenOCD is very great when we want the same commands, configs and Makefile for
every probe.

## Examples

Some examples of use of the openOCD can be seen in the repo
[simplest-build](https://github.com/sleman-notes/simplest-build), other examples
in Makefile is:

```bash
cd examples/blink-f411
make                        # build
make flash                  # build + flash with ST-Link
make flash PROBE=jlink      # build + flash with J-Link
make erase                  # mass erase
make server                 # OpenOCD as a server, for GDB/telnet
make gdb                    # (other terminal) GDB connected to the server
make clean
```