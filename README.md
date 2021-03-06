# This repository has been obsoleted.

This repository contains modified OpenOCD with workarounds ("hotfixes") that were needed for SweRV EH1 cores **1.7 and older**.

For present-day [SweRV EH1 cores](https://github.com/chipsalliance/Cores-SweRV) **1.8 and above**, please use plain-vanilla RISC-V OpenOCD version from [riscv-openocd repository](https://github.com/riscv/riscv-openocd).

The hotfixes in this repository are no longer required for SweRV EH1 1.8+ and the code is left here for archive purposes only.

---
# Original Readme (archived):


## About this OpenOCD version

This was a modified OpenOCD version with workarounds 
("software hotfixes") for SweRV RISC-V cores (SweRV EH1 **<= 1.7**).

It was intended as a temporary tool to work around the limitations
of the SweRV core (smooth access to ICCM+DCCM memories), as required at that time.

For **SweRV EH1 1.8 and above**, please use upstream "riscv-openocd" (https://github.com/riscv/riscv-openocd). The command to activate abstract memory access in present-day "riscv-openocd" is: `riscv set_mem_access abstract` (add it to your OpenOCD configuration).


## Code version

The code in this repository is based on "riscv-openocd" (https://github.com/riscv/riscv-openocd).<br>
Base commit ID: bbfc666eba84f7133510fa0c5cb1d2d9e7a49bce<br>
Date: July 2, 2020

## Codasip changes for SweRV

- Added command `riscv add_abstract_mem_range`

  Allows to select for which areas in memory "abstract access" is preferred
over "system bus access".

  Users shall use this command to specify the memory ranges for SweRV's
ICCM, DCCM and PIC, otherwise OpenOCD would not be able to access these
core-local memories.

  Usage:

  >riscv add_abstract_mem_range <ICCM_start_addr> <ICCM_len_bytes><br>
   riscv add_abstract_mem_range <DCCM_start_addr> <DCCM_len_bytes><br>
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

1. **Build this OpenOCD** from source code (see next section)

   This binary will be produced: `build/bin/swerv-openocd`

2. **Prepare configuration file(s)** for OpenOCD that match your hardware and your JTAG adapter

   Follow one of the provided examples in the `config_examples` directory: 
   
   - `swervolf_jtag_vpi.cfg` - SweRVolf running in simmulation, using JTAG VPI
   - `swervolf_nexys_a7_bscan.cfg` - SweRVolf on Nexys A7 board, using the JTAG interface of the FPGA board
   - `swervolf_nexys_a7_olimex.cfg` - SweRVolf on Nexys A7 board, using Olimex ARM-USB-TINY-H adapter

   **Don't forget to edit the example configuration file so that the addresses of PIC, ICCM and DCCM match your hardware, or create own configuration file(s) if needed.**

3. **Execute OpenOCD**:<br>
   `path/to/swerv-openocd -f path/to/your/config_file.cfg [-f path/to/other/conf_file.cfg] [...] [-d3]`<br>
   Should you need to troubleshoot issues, add argument `-d3` to increase the verbosity of OpenOCD.

## Build process

### Building on Linux

Clone the contents of this repository, including its submodules:

>$ git clone --recursive https://github.com/Codasip/swerv-openocd.git

Alternatively:

>$ git clone https://github.com/Codasip/swerv-openocd.git</br>
 $ cd swerv-openocd</br>
 $ git submodule update --init --recursive

Install all the needed **prerequisities**, if sufficient packages are not already installed, namely:
- libtool
- pkg-config
- autoconf >= 2.64
- automake >= 1.14
- libusb
- additional libraries depending on the adapters suported, e.g. libftdi
 
**On CentOS**:
>$ sudo yum install libtool pkgconfig autoconf automake libusbx libftdi

Note for **CentOS 7** users: The build of OpenOCD requires automake >= 1.14, which is 
however not available in official CentOS 7 packages. To obtain sufficient automake,
use for example:

>$ wget http://repo.okay.com.mx/centos/7/x86_64/release//automake-1.14-1.el7.x86_64.rpm<br>
 $ sudo yum install automake-1.14-1.el7.x86_64.rpm

**On Debian**:
>$ sudo apt install libtool pkg-config  autoconf automake libusb-1.0 libftdi1

**Build OpenOCD**, via the following commands:

>$ ./bootstrap<br>
 $ ./configure --enable-jtag_vpi --enable-remote-bitbang --enable-ftdi --prefix=\`pwd\`/build --program-prefix=swerv-<br>
 $ make<br>
 $ make install
 
 ### Build on Windows
 
 We recomend using the **MSYS2 platform** (https://www.msys2.org/). 

Please install MSYS2 on your workstation, if not installed already.

Start MSYS2 and install the needed packages:

>$ pacman -Sy<br>
 $ pacman -S git libtool automake autoconf pkg-config make gcc<br>
 $ pacman -S mingw-w64-x86_64-toolchain mingw-w64-x86_64-libftdi mingw-w64-x86_64-libusb

Update your PATH variable in MSYS2: 

>$ export PATH="$PATH:/mingw64/bin"

Clone this repository:

>$ git clone --recursive https://github.com/Codasip/swerv-openocd.git<br>
 $ cd swerv-openocd

To obtain statically linked OpenOCD binary (which does not require additional DLLs), set this environment variable: 

>$ export AM_LDFLAGS=--static

(Alternatively, the required DLLs can be found in  `<msys2_install_path>\mingw64\lib`)

**Build OpenOCD** using following commands: 

>$ ./bootstrap<br>
>$ ./configure --enable-jtag_vpi --enable-remote-bitbang --enable-ftdi --prefix=`pwd`/build --program-prefix=swerv- --build=x86_64-unknown-linux-gnu --host=x86_64-w64-mingw32<br>
>$ make<br>
>$ make install
 
 
 ## Changelog

 - v2021\_02\_11: Backported two system bus related fixes from riscv-openocd - original commits: [5d0543c](https://github.com/riscv/riscv-openocd/commit/5d0543cc1c7749a9bf3538328397a5e817275fb9), [9e17460](https://github.com/riscv/riscv-openocd/commit/9e174604b95433db106ae06267c5cb33884c1e43)
 - v2021\_01\_27: Commands "expose\_csrs" and "expose\_custom" were improved - changes backported from riscv-openocd
   (cherry-picked commits [11c4f89b3](https://github.com/riscv/riscv-openocd/commit/11c4f89b32536de6d67264812bd7418433bd863b) and [6db3ed2c8](https://github.com/riscv/riscv-openocd/commit/6db3ed2c862e04588bf80758acb463a14e9b5ff5)). Fixed a minor free() related bug.
 - v2020\_09\_21: This repository has been obsoleted. Upstream "riscv-openocd" shall be used instead.
 - v2020\_07\_16: Fixed abstracts commands not to set `aampostincrement` bit, necessary since SweRV EH1 1.7
 - v2020\_07\_07: Initial release
