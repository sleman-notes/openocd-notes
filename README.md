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
