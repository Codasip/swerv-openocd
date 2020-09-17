# About this OpenOCD version

This is a modified OpenOCD version with workarounds 
("software hotfixes") for SweRV RISC-V cores (SweRV EH1 **<= 1.7**).

It is intended as a temporary tool to work around the limitations
of the current SweRV cores (smooth access to ICCM+DCCM memories),
until these are resolved.

For **newer versions** of SweRV use upstream "riscv-openocd" (https://github.com/riscv/riscv-openocd).
To enable access to xCCM, use the following command:

> riscv set_mem_access abstract


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
 
 - v2020_07_16: Fixed abstracts commands not to set `aampostincrement` bit, necessary since SweRV EH1 1.7
 - v2020_07_07: Initial release
