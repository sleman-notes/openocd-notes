# Cheatsheet

## Start

```bash
# server (stays running, ports 3333 gdb/4444 telnet/6666 tcl)
openocd -f configs/stm32f411-stlink.cfg

# without our configs
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg
openocd -f interface/jlink.cfg -c "transport select swd" -f target/stm32f1x.cfg
```

| Flag | Meaning |
|---|---|
| -f FILE | load a config file (current folder, then scripts/) |
| -c "CMD" | run a command, in order with the -f files |
| -s DIR | add a folder to the search path for -f |
| -d3 | debug log, when nothing else explains the error |
| -l FILE | write the log to a file |

## Flash

```bash
# .elf/.hex: addresses come inside the file
openocd -f configs/stm32f411-stlink.cfg -c "program build/blink.elf verify reset exit"

# .bin: address right after the file name
openocd -f configs/stm32f411-stlink.cfg -c "program build/blink.bin 0x08000000 verify reset exit"

# mass erase
openocd -f configs/stm32f411-stlink.cfg -c "init; reset halt; stm32f2x mass_erase 0; shutdown"
openocd -f configs/stm32f103-stlink.cfg -c "init; reset halt; stm32f1x mass_erase 0; shutdown"

# read the whole flash back to a file
openocd -f configs/stm32f103-stlink.cfg -c "init; reset halt; flash read_bank 0 dump.bin; shutdown"

# just reset and run
openocd -f configs/stm32f411-stlink.cfg -c "init; reset run; shutdown"
```

## Config file lines

| Line | What it does |
|---|---|
| source [find interface/stlink.cfg] | ST-Link probe |
| source [find interface/jlink.cfg] | J-Link probe |
| transport select hla_swd | SWD through ST-Link |
| transport select swd | SWD through J-Link (default is JTAG) |
| set CPUTAPID 0 | accept clone chips (before the target) |
| set FLASH_SIZE 0x20000 | force 128K on the F103 (before the target) |
| source [find target/stm32f4x.cfg] | F401/F411/F4xx |
| source [find target/stm32f1x.cfg] | F103 and clones |
| adapter speed 4000 | SWD clock in kHz (after the target) |
| adapter serial XXXX | pick one probe when several are plugged |
| reset_config srst_only srst_nogate connect_assert_srst | connect under reset (NRST wired) |

## Telnet / GDB monitor commands

"telnet localhost 4444", or in GDB prefix with "monitor".

| Command | What it does |
|---|---|
| halt/resume | stop/continue the CPU |
| step | one instruction |
| reset halt | reset and stop at the first instruction |
| reset run | reset and run |
| reg/reg pc | core registers |
| mdw ADDR [N] | read N words (mdh halfwords, mdb bytes) |
| mww ADDR VALUE | write a word (mwh, mwb) |
| bp ADDR 2 hw/rbp ADDR | hardware breakpoint/remove |
| flash banks | flash regions |
| flash info 0 | sectors and protection of bank 0 |
| flash write_image erase FILE.elf | write without verify/reset |
| verify_image FILE.elf | compare flash with the file |
| adapter speed | show/change the SWD clock |
| targets | target list and state |
| shutdown | close OpenOCD |

## Flash driver commands

| Action | F103 (stm32f1x) | F411 (stm32f2x) |
|---|---|---|
| Mass erase | stm32f1x mass_erase 0 | stm32f2x mass_erase 0 |
| RDP on | stm32f1x lock 0 | stm32f2x lock 0 |
| RDP off (erases all) | stm32f1x unlock 0 | stm32f2x unlock 0 |
| Option bytes | stm32f1x options_read 0 | stm32f2x options_read 0 |

## Useful addresses

| Address | Size | F103 | F411 |
|---|---|---|---|
| 0x08000000 | word | flash start (initial SP) | flash start (initial SP) |
| 0xE0042000 | word | DBGMCU_IDCODE, ID 0x410 | DBGMCU_IDCODE, ID 0x431 |
| 0x1FFFF7E0 | half | flash size in KB | - |
| 0x1FFF7A22 | half | - | flash size in KB |
| 0x1FFFF7E8 | 96 bit | unique ID | - |
| 0x1FFF7A10 | 96 bit | - | unique ID |

## GDB

```bash
arm-none-eabi-gdb build/blink.elf
```

```
target extended-remote localhost:3333
monitor reset halt
load
break main
continue
```

| Command | What it does |
|---|---|
| next/step | next line, over/into calls |
| stepi | one instruction |
| finish | run until the function returns |
| info registers | core registers |
| bt | call stack |
| watch var | stop when var changes |
| info breakpoints/delete N | list/remove breakpoints |

## Several instances

```bash
openocd -f configs/stm32f103-stlink.cfg -c "adapter serial XXXX; gdb port 3334; telnet_port 4445; tcl port 6667"
```