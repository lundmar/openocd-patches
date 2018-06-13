# OpenOCD patches

This repository is a placeholder for various patches for OpenOCD.

## Installation

```
$ sudo apt install libusb-1.0-0-dev
$ git clone https://git.code.sf.net/p/openocd/code openocd
$ cd openocd
$ wget https://raw.githubusercontent.com/lundmar/openocd-patches/master/0001-Add-PS_TAPIDs-for-various-Xilinx-UltraScale-devices.patch
$ wget https://raw.githubusercontent.com/lundmar/openocd-patches/master/0002-Bypass-armv8-crash.patch
$ git am *.patch
$ ./bootstrap
$ ./configure --prefix=$HOME/opt/openocd
$ make && make install
$ export PATH=$HOME/opt/openocd/bin
```

## Testing with Xilinx ZCU102 development board

Power on your development board and run this command to attach and halt target:

```
$ openocd -s $HOME/opt/openocd/share/openocd/scripts -f interface/ftdi/digilent_jtag_smt2_nc.cfg -f target/xilinx_ultrascale.cfg -c "adapter_khz 100" -c "reset_config trst_and_srst" -c "init; halt"
Open On-Chip Debugger 0.10.0+dev-00430-g06123153 (2018-06-11-09:11)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
none separate
Info : auto-selecting first available session transport "jtag". To override use 'transport select <transport>'.
core_up
adapter speed: 100 kHz
trst_and_srst separate srst_gates_jtag trst_push_pull srst_open_drain connect_deassert_srst
Info : clock speed 100 kHz
Info : JTAG tap: uscale.tap tap/device found: 0x5ba00477 (mfg: 0x23b (ARM Ltd.), part: 0xba00, ver: 0x5)
Info : JTAG tap: uscale.ps tap/device found: 0x24738093 (mfg: 0x049 (Xilinx), part: 0x4738, ver: 0x2)
Info : JTAG tap: uscale.tap tap/device found: 0x5ba00477 (mfg: 0x23b (ARM Ltd.), part: 0xba00, ver: 0x5)
Info : JTAG tap: uscale.ps tap/device found: 0x24738093 (mfg: 0x049 (Xilinx), part: 0x4738, ver: 0x2)
Info : uscale.a53.0: hardware has 6 breakpoints, 4 watchpoints
Info : uscale.a53.0 cluster 0 core 0 multi core
Info : Listening on port 3333 for gdb connections
Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections

```

If you only want to attach but not halt the target simply remove the "init; halt" command.

Openocd is now attached to target and ready to accept incomming gdb client connections.

For example, to debug u-boot after relocation simply use gdb client available from your toolchain like so:

```
$ aarch64-poky-linux-gdb
GNU gdb (GDB) 8.0.1
Copyright (C) 2017 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "--host=x86_64-pokysdk-linux --target=aarch64-poky-linux".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
(gdb) add-symbol-file /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/build/u-boot 0x7FED1000
add symbol table from file "/opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/build/u-boot" at
        .text_addr = 0x7fed1000
(y or n) y             
Reading symbols from /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/build/u-boot...done.
(gdb) break do_version                                                                                                                                                                                                         
Breakpoint 1 at 0x7fed4b08: file /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/git/cmd/version.c, line 19.
(gdb) info break                                                                                                                                                                                                                                 
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x000000007fed4b08 in do_version at /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/git/cmd/version.c:19
(gdb) continue
Continuing.
                       
Breakpoint 1, do_version (cmdtp=0x7ff8cb98, flag=0, argc=1, argv=0x7dec3530) at /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/git/cmd/version.c:19
19      {             
(gdb) list        
14      #endif
15              
16      const char __weak version_string[] = U_BOOT_VERSION_STRING;
17                   
18      static int do_version(cmd_tbl_t *cmdtp, int flag, int argc, char * const argv[])
19      {           
20              char buf[DISPLAY_OPTIONS_BANNER_LENGTH];
21       
22              printf(display_options_get_banner(false, buf, sizeof(buf)));
23      #ifdef CC_VERSION_STRING
(gdb) info stack
#0  do_version (cmdtp=0x7ff8cb98, flag=0, argc=1, argv=0x7dec3530) at /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/git/cmd/version.c:19
#1  0x000000007fef679c in cmd_call (argv=0x7dec3530, argc=1, flag=0, cmdtp=0x7ff8cb98) at /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/git/common/command.c:500
#2  cmd_process (flag=<optimized out>, flag@entry=0, argc=1, argv=0x7dec3530, repeatable=repeatable@entry=0x7ff9c7a4, ticks=ticks@entry=0x0) at /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-
r0/git/common/command.c:539
#3  0x000000007fee883c in run_pipe_real (pi=0x7dec3460) at /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/git/common/cli_hush.c:1678
#4  run_list_real (pi=<optimized out>) at /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/git/common/cli_hush.c:1876
#5  0x000000007fee89e0 in run_list (pi=0x7dec3460) at /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/git/common/cli_hush.c:2025
#6  parse_stream_outer (inp=inp@entry=0x7dec0d40, flag=flag@entry=2) at /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/git/common/cli_hush.c:3217
#7  0x000000007fee8f20 in parse_file_outer () at /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/git/common/cli_hush.c:3300
#8  0x000000007fef5d1c in cli_loop () at /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/git/common/cli.c:218
#9  0x000000007fee6b70 in main_loop () at /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/git/common/main.c:68
#10 0x000000007fee9500 in run_main_loop () at /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/git/common/board_r.c:662
#11 0x000000007ff49df8 in initcall_run_list (init_sequence=init_sequence@entry=0x7ff895d0) at /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/git/lib/initcall.c:31
#12 0x000000007fee9754 in board_init_r (new_gd=<optimized out>, dest_addr=<optimized out>) at /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/git/common/board_r.c:895
#13 0x000000007fed3470 in _main () at /opt/nocrypt/work/git/z7000-distro-2018.1/build/tmp/work/zcu102_zynqmp-poky-linux/u-boot-xlnx/v2018.01-xilinx-v2018.1+gitAUTOINC+949e5cb9a7-r0/git/arch/arm/lib/crt0_64.S:119
(gdb) info registers
x0             0xf942e4 16335588
x1             0xa      10
x2             0x80a    2058
x3             0xa      10
x4             0xfffd813e       4294803774
x5             0x4      4
x6             0x8ca00  576000
x7             0x5      5
x8             0xad     173
x9             0x80     128
x10            0xfffd   65533
x11            0x64     100
x12            0xfffffe6a       4294966890
x13            0x460c050400048c5        315463424819611845
x14            0x2208081803040107       2452218896326066439
x15            0x5bc421a712040000       6612447154332237824
x16            0x280c820380080c00       2885824412782169088
x17            0xb0238220540d104f       12692131250220896335
x18            0xffff7e90       4294934160
x19            0x5fb33e5        100348901
x20            0x552e4  348900
x21            0x989680 10000000
x22            0xc80020c18400c04        900722176443812868
x23            0xc90240021400288a       14484209729246603402
x24            0x60a0580000100431       6962661780939080753
x25            0xa010142002000010       11533740773400903696
x26            0x6082c1310081284d       6954333190819489869
x27            0x80c7120302c00221       9279405361360536097
x28            0x8201800002040c13       9367909437429517331
x29            0xffff7e30       4294934064
x30            0xfffcab9c       4294749084
sp             0xffff7e30       0xffff7e30
pc             0xfffc0cb8       0xfffc0cb8
cpsr           0x800002cd       [ SP=1 EL=2 nRW=0 F I D N ]
```
Note: To debug u-boot after relocation you need to provide the relocation offset when loading the u-boot symbol file (elf), in this case 0x7FED1000.
