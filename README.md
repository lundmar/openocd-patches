# OpenOCD patches

This repository contains various patches for OpenOCD.

## Installation

```bash
sudo apt install libusb-1.0-0-dev
git clone https://git.code.sf.net/p/openocd/code openocd
cd openocd
wget https://raw.githubusercontent.com/lundmar/openocd-patches/master/0001-Add-PS_TAPIDs-for-various-Xilinx-UltraScale-devices.patch
wget https://raw.githubusercontent.com/lundmar/openocd-patches/master/0002-Bypass-armv8-crash.patch
git am *.patch
./configure --prefix=$HOME/opt/openocd
make && make install
export PATH=$HOME/opt/openocd/bin
```
