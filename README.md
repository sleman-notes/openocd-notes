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
