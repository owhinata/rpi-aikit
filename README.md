# RPi AI-HATを使う

### Hardware
Raspberry Pi 5 (8GB)  
Raspberry Pi AI HAT (26TOPS)

### Software
```
$ uname -a
Linux raspberrypi 6.6.51+rpt-rpi-2712 #1 SMP PREEMPT Debian 1:6.6.51-1+rpt3 (2024-10-08) aarch64 GNU/Linux

$ lsb_release -a
No LSB modules are available.
Distributor ID: Debian
Description:    Debian GNU/Linux 12 (bookworm)
Release:        12
Codename:       bookworm
```

### パッケージのインストール
```
$ sudo apt-get update
$ sudo apt-get install git vim clangd clang-tools bear
```

### vim-plugのインストール
```
$ curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim

:PlugInstall
:PlugUpdate
:InstallLspServer
```

### CoreMark
```
$ git clone https://github.com/eembc/coremark.git
$ cd coremark
$ make
$ cat run1.log
2K performance run parameters for coremark.
CoreMark Size    : 666
Total ticks      : 16547
Total time (secs): 16.547000
Iterations/Sec   : 18130.174654
Iterations       : 300000
Compiler version : GCC12.2.0
Compiler flags   : -O2 -DPERFORMANCE_RUN=1  -lrt
Memory location  : Please put data memory location here
                        (e.g. code in flash, data on heap etc)
seedcrc          : 0xe9f5
[0]crclist       : 0xe714
[0]crcmatrix     : 0x1fd7
[0]crcstate      : 0x8e3a
[0]crcfinal      : 0xcc42
Correct operation validated. See README.md for run and reporting rules.
CoreMark 1.0 : 18130.174654 / GCC12.2.0 -O2 -DPERFORMANCE_RUN=1  -lrt / Heap
```

### PCIe Gen3を有効化する
```
$ sudo raspi-config
-> 6 Advanced Options
    -> A8 PCIe Speed
        -> Choose "Yes" to enable PCIe Gen 3 mode. Select "Finish" to exit.
$ sudo reboot
```

### Hailoに関するパッケージをインストール
```
$ lspci
0000:00:00.0 PCI bridge: Broadcom Inc. and subsidiaries BCM2712 PCIe Bridge (rev 21)
0000:01:00.0 Co-processor: Hailo Technologies Ltd. Hailo-8 AI Processor (rev 01)
0001:00:00.0 PCI bridge: Broadcom Inc. and subsidiaries BCM2712 PCIe Bridge (rev 21)
0001:01:00.0 Ethernet controller: Raspberry Pi Ltd RP1 PCIe 2.0 South Bridge

$ sudo apt install hailo-all
$ sudo reboot

$ hailortcli fw-control identify
[HailoRT] [error] CHECK failed - Driver version (4.18.0) is different from library version (4.19.0)
[HailoRT] [error] Driver version mismatch, status HAILO_INVALID_DRIVER_VERSION(76)
[HailoRT] [error] CHECK_SUCCESS failed with status=HAILO_INVALID_DRIVER_VERSION(76)
[HailoRT] [error] CHECK_SUCCESS failed with status=HAILO_INVALID_DRIVER_VERSION(76)
[HailoRT] [error] CHECK_SUCCESS failed with status=HAILO_INVALID_DRIVER_VERSION(76)
[HailoRT CLI] [error] CHECK_SUCCESS failed with status=HAILO_INVALID_DRIVER_VERSION(76)
```
　　↓ エラーになったのでやり直し
```
$ sudo apt-get update
$ sudo apt-get full-upgrade
$ sudo apt-get install hailo-all
$ sudo reboot

$ uname -a
Linux raspberrypi 6.6.62+rpt-rpi-2712 #1 SMP PREEMPT Debian 1:6.6.62-1+rpt1 (2024-11-25) aarch64 GNU/Linux

$ hailortcli fw-control identify
Executing on device: 0000:01:00.0
Identifying board
Control Protocol Version: 2
Firmware Version: 4.19.0 (release,app,extended context switch buffer)
Logger Version: 0
Board Name: Hailo-8
Device Architecture: HAILO8
Serial Number: <N/A>
Part Number: <N/A>
Product Name: <N/A>
```

### Test TAPPAS Core installation
```
$ gst-inspect-1.0 hailotools
Plugin Details:
  Name                     hailotools
  Description              hailo tools plugin
  Filename                 /lib/aarch64-linux-gnu/gstreamer-1.0/libgsthailotools.so
  Version                  3.30.0
  License                  unknown
  Source module            gst-hailo-tools
  Binary package           gst-hailo-tools
  Origin URL               https://hailo.ai/

  hailoaggregator: hailoaggregator - Cascading
  hailocounter: hailocounter - postprocessing element
  hailocropper: hailocropper
  hailoexportfile: hailoexportfile - export element
  hailoexportzmq: hailoexportzmq - export element
  hailofilter: hailofilter - postprocessing element
  hailogallery: Hailo gallery element
  hailograytonv12: hailograytonv12 - postprocessing element
  hailoimportzmq: hailoimportzmq - import element
  hailomuxer: Muxer pipeline merging
  hailonv12togray: hailonv12togray - postprocessing element
  hailonvalve: HailoNValve element
  hailooverlay: hailooverlay - overlay element
  hailoroundrobin: Input Round Robin element
  hailostreamrouter: Hailo Stream Router
  hailotileaggregator: hailotileaggregator
  hailotilecropper: hailotilecropper - Tiling
  hailotracker: Hailo object tracking element

  18 features:
  +-- 18 elements

$ gst-inspect-1.0 hailo
Plugin Details:
  Name                     hailo
  Description              hailo gstreamer plugin
  Filename                 /lib/aarch64-linux-gnu/gstreamer-1.0/libgsthailo.so
  Version                  1.0
  License                  unknown
  Source module            hailo
  Binary package           GStreamer
  Origin URL               http://gstreamer.net/

  hailodevicestats: hailodevicestats element
  hailonet: hailonet element
  synchailonet: sync hailonet element

  3 features:
  +-- 3 elements
```
