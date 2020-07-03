# About this OpenOCD version

This is a modified OpenOCD version with workarounds 
("software hotfixes") for SweRV RISC-V cores (SweRV EH1 <= 1.7).

It is intended as a temporary tool to work around the limitations
of the current SweRV cores (smooth access to ICCM+DCCM memories),
until these are resolved.

## Code version

The code in this repository is based on "riscv-openocd" (https://github.com/riscv/riscv-openocd).
Base commit ID: bbfc666eba84f7133510fa0c5cb1d2d9e7a49bce 
Date: July 2, 2020

## Codasip changes for SweRV

- Added command `riscv add_abstract_mem_range`

  Allows to select for which areas in memory "abstract access" is preferred
over "system bus access".

  Users shall use this command to specify the memory ranges for SweRV's
ICCM, DCCM and PIC, otherwise OpenOCD would not be able to access these
core-local memories.

  Usage:

  >riscv add_abstract_mem_range <ICCM_start_addr> <ICCM_len_bytes>`
riscv add_abstract_mem_range <DCCM_start_addr> <DCCM_len_bytes>
riscv add_abstract_mem_range <PIC_start_addr> <PIC_len_bytes>

- Allowed 8-bit and 16-bit access to closely coupled memories

  Currently, SweRV only supports reading or writing of 32-bit words in ICCM and DCCM.
This hot-fixed version of OpenOCD lifts this limitation and allows the user to perform also
8-bit and 16-bit wide accesses to the closely-coupled memories.

  8/16-bit acccesses are internally transformed to 32-bit wide accesses which the hardware can
accommodate - fully transparently to the user. Read-modify-write scheme is applied inside OpenOCD,
when necessary.

- Removed dependency on optional aampostincrement bit

  Plain-vanilla OpenOCD relied on `aampostincrement` bit when performing abstract memory access.
This (optional) bit is not supported on SweRV EH1 <= 1.7, so the OpenOCD code was adjusted not to 
not use it.

## How to use - quick instructions

1. Build this OpenOCD from source code (see next section)

   This binary will be produced: `swerv-openocd`

2. Prepare configuration file(s) for OpenOCD that match your hardware and your JTAG adapter

   Follow the provided example:  `swerv-example.cfg`

3. Edit the example .cfg file so that the addresses of PIC, ICCM and DCCM match your hardware

   >riscv add_abstract_mem_range <ICCM_start_addr> <ICCM_len_bytes>
riscv add_abstract_mem_range <DCCM_start_addr> <DCCM_len_bytes>
riscv add_abstract_mem_range <PIC_start_addr> <PIC_len_bytes>

4. Execute OpenOCD:
   `path/to/swerv-openocd -f path/to/your/config_file.cfg [-f path/to/other/conf_file.cfg] [...] [-d3]`
   Should you need to troubleshoot issues, add argument `-d3` to increase the verbosity of OpenOCD.

## Build process

If you wish to build OpenOCD from the sourc code, use following commands:

>$ ./bootstrap
$ ./configure --enable-jtag_vpi --enable-remote-bitbang --enable-ftdi --prefix=\`pwd\`/build --program-prefix=swerv-
$ make
$ make install

_Original OpenOCD README can be found under the name README.orig_

