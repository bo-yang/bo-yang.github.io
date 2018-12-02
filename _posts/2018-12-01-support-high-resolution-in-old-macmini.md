---
layout: post
title: Make Old Mac Mini(Ubuntu) Support QHD Monitor
categories: 
- Notes
- Unix/Linux
tags:
- Notes
- Unix/Linux
status: publish
type: post
published: true
author: Bo Yang
---

Today I received the new Dell S3219D monitor ordered on Cyber Monday. I bought it for my Mac Mini 2012 - it's running Ubuntu 18.10, because the macOS X became too slow on the old hardware. Although the monitor worked after connecting it to Mac Mini through HDMI cable, the resolution was only 1920x1200, not the QHD 2560x1440. I decided to fix this problem.

Initially, I thought that the driver should be upgraded. Dell provides driver for this monitor, but Windows only. And after searching online, I realized that the default Ubuntu video driver should be good enough. So I decided not to insrtall the Linux version Dell/Intel Linux drivers found from some obsolete webpages.

~~~console
bo@bo-Macmini:~$ lspci -vv | more
......
00:02.0 VGA compatible controller: Intel Corporation 3rd Gen Core processor Graphics Controller (rev 09) (p
rog-if 00 [VGA controller])
	Subsystem: Apple Inc. 3rd Gen Core processor Graphics Controller
	Control: I/O+ Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINT
x+
	Status: Cap+ 66MHz- UDF- FastB2B+ ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx
-
	Latency: 0
	Interrupt: pin A routed to IRQ 49
	Region 0: Memory at a0000000 (64-bit, non-prefetchable) [size=4M]
	Region 2: Memory at 90000000 (64-bit, prefetchable) [size=256M]
	Region 4: I/O ports at 2000 [size=64]
	[virtual] Expansion ROM at 000c0000 [disabled] [size=128K]
	Capabilities: <access denied>
	Kernel driver in use: i915
	Kernel modules: i915
......
~~~

In Linux, `xrandr` is the command to query/set screen resolutions. In my case,

~~~console
bo@bo-Macmini:/usr/lib$ xrandr
Screen 0: minimum 320 x 200, current 1920 x 1200, maximum 8192 x 8192
VGA-1 disconnected (normal left inverted right x axis y axis)
HDMI-1 disconnected (normal left inverted right x axis y axis)
DP-1 disconnected (normal left inverted right x axis y axis)
HDMI-2 disconnected (normal left inverted right x axis y axis)
HDMI-3 connected primary 1920x1200+0+0 (normal left inverted right x axis y axis) 708mm x 399mm
   1920x1200     59.95* 
   1920x1080     60.00    50.00    59.94  
   1920x1080i    60.00    50.00    59.94  
   1600x1200     60.00  
   1680x1050     59.88  
   1280x1024     75.02    60.02  
   1280x800      59.91  
   1152x864      75.00  
   1280x720      60.00    50.00    59.94  
   1024x768      75.03    60.00  
   800x600       75.00    60.32  
   720x576       50.00  
   720x480       60.00    59.94  
   640x480       75.00    60.00    59.94  
   720x400       70.08  
DP-2 disconnected (normal left inverted right x axis y axis)
DP-3 disconnected (normal left inverted right x axis y axis)
~~~

this means that the 2560x1440 resolution is not a supported(or detected) mode in output HDMI-3.

There are a lot of [guides](http://ubuntuhandbook.org/index.php/2017/04/custom-screen-resolution-ubuntu-desktop/) to adda custom screen resolution to Ubuntu. [Some people recommend](https://askubuntu.com/questions/377937/how-to-set-a-custom-resolution) to use command `cvt` to calculate the custom Modeline, however, [the calculation may be wrong without `-r` option for LCD screens](https://wiki.archlinux.org/index.php/xrandr#Adding_undetected_resolutions).

~~~console
bo@bo-Macmini:~$ cvt -r 2560 1440 60
# 2560x1440 59.95 Hz (CVT 3.69M9-R) hsync: 88.79 kHz; pclk: 241.50 MHz
Modeline "2560x1440R"  241.50  2560 2608 2640 2720  1440 1443 1448 1481 +hsync -vsync
~~~

In addition to the `cvt` command, we also need to double check whether the system already receives the correct EDID from the monitor. The easiest way is to search the Xorg logs(in Ubuntu 18.04 and after, this file is under `~/.local/share/xorg/Xorg.0.log`):

~~~
[    59.550] (II) modeset(0): EDID vendor "DEL", prod id 53486
[    59.550] (II) modeset(0): Using EDID range info for horizontal sync
[    59.550] (II) modeset(0): Using EDID range info for vertical refresh
[    59.550] (II) modeset(0): Printing DDC gathered Modelines:
[    59.550] (II) modeset(0): Modeline "2560x1440"x0.0  241.50  2560 2608 2640 2720  1440 1443 1448 1481 +hsync -vsync (88.8 kHz eP)
[    59.550] (II) modeset(0): Modeline "2560x1440"x0.0  296.80  2560 2608 2640 2720  1440 1443 1448 1455 +hsync -vsync (109.1 kHz e)
[    59.550] (II) modeset(0): Modeline "1920x1080i"x0.0   74.25  1920 2008 2052 2200  1080 1084 1094 1125 interlace +hsync +vsync (33.8 kHz e)
[    59.550] (II) modeset(0): Modeline "1280x720"x0.0   74.25  1280 1390 1430 1650  720 725 730 750 +hsync +vsync (45.0 kHz e)
......
~~~

You also can use command `edid-decode`:

~~~console
bo@bo-Macmini:/usr/lib$ xrandr --props | edid-decode
......
Manufacturer: DEL Model d0ee Serial Number 826431573
Made week 38 of 2018
EDID version: 1.3
Digital display
Maximum image size: 71 cm x 40 cm
Gamma: 2.20
DPMS levels: Standby Suspend Off
Supported color formats: RGB 4:4:4, YCrCb 4:4:4
Default (sRGB) color space is primary color space
Established timings supported:
  720x400@70Hz
  640x480@60Hz
  640x480@75Hz
  800x600@60Hz
  800x600@75Hz
  1024x768@60Hz
  1024x768@75Hz
  1280x1024@75Hz
Standard timings supported:
  1280x800@60Hz
  1680x1050@60Hz
  1920x1200@60Hz
  1152x864@75Hz
  1600x1200@60Hz
  1280x1024@60Hz
  1920x1080@60Hz
Detailed mode: Clock 241.500 MHz, 708 mm x 399 mm
               2560 2608 2640 2720 hborder 0
               1440 1443 1448 1481 vborder 0
               +hsync -vsync 
Serial number: 275STP2
Monitor name: DELL
Monitor ranges (GTF): 48-75Hz V, 30-114kHz H, max dotclock 300MHz
Has 1 extension blocks
Checksum: 0x33 (valid)

CEA extension block
Extension version: 3
37 bytes of CEA data
......
1 native detailed modes
Detailed mode: Clock 296.800 MHz, 708 mm x 399 mm
               2560 2608 2640 2720 hborder 0
               1440 1443 1448 1455 vborder 0
               +hsync -vsync 
Detailed mode: Clock 74.250 MHz, 708 mm x 399 mm
               1920 2008 2052 2200 hborder 0
                540  542  547  562 vborder 0
               +hsync +vsync interlaced 
Detailed mode: Clock 74.250 MHz, 708 mm x 399 mm
               1280 1390 1430 1650 hborder 0
                720  725  730  750 vborder 0
               +hsync +vsync 
Detailed mode: Clock 27.000 MHz, 708 mm x 399 mm
                720  736  798  858 hborder 0
                480  489  495  525 vborder 0
               -hsync -vsync 
Checksum: 0xff (valid)
......
~~~

Seems like the 2560x1440 modeline is already detected. To make it supported by output HDMI-3, the new mode should be created and added to HDMI-3:

~~~console
bo@bo-Macmini:/usr/lib$ sudo xrandr --newmode "2560x1440_60"  296.80  2560 2608 2640 2720  1440 1443 1448 1455 +hsync -vsync
bo@bo-Macmini:/usr/lib$ sudo xrandr --addmode HDMI-3 2560x1440_60
~~~

To use this customized mode, run command

~~~console
bo@bo-Macmini:/usr/lib$ sudo xrandr --output HDMI-3 --mode 2560x1440_60
xrandr: Configure crtc 0 failed
~~~~

Oops, got a failure.

After searching the error message online, I found this article - [How to use high (4K) resolutions with older hardware](https://medium.com/@ValdikSS/how-to-use-high-resolutions-with-older-hardware-58577d91b1f8). The key point is that, **the old graphics chip could not support the new monitor's pixel clock**. The workaround is to generate customized modline with lower refresh rates(EDID only advertise recommended refresh rates). The recommended tool is [umc](https://sourceforge.net/projects/umc/files/umc/) for Linux - you need to download the source code and build the binary on your machine.

Starting from 30Hz, I tried multiple different refresh rates and finally realized that 55Hz was the max one that could be supported:

~~~console
bo@bo-Macmini:~$ umc 2560 1440 55 --rbt

    # 2560x1440x54.97 @ 81.250kHz
    Modeline "2560x1440x54.97"  221.000000  2560 2608 2640 2720  1440 1443 1447 1478  +HSync -VSync

bo@bo-Macmini:/usr/lib$ sudo xrandr --newmode "2560x1440x54.97"  221.000000  2560 2608 2640 2720  1440 1443 1447 1478  +HSync -VSync
bo@bo-Macmini:/usr/lib$ sudo xrandr --addmode HDMI-3 2560x1440x54.97
bo@bo-Macmini:/usr/lib$ sudo xrandr --output HDMI-3 --mode 2560x1440x54.97 --rate 55
~~~

The above change is not permanent. To make the monitor boot into this mode, the above modeline should be added into the Xorg config file - `/etc/X11/xorg.conf.d/10-monitor.conf`. Very likely you cannot find this file or even the `xorg.config.d` directory under `/etc/X11/`. If this is the case, create the dir and file, then add the following Monitor section:

~~~
Section "Monitor"
    Identifier "HDMI-3"
    ModelName "DELL S3219D"
    Modeline "2560x1440_55"  221.000000  2560 2608 2640 2720  1440 1443 1447 1478  +HSync -VSync
    Option "PreferredMode" "2560x1440_55"
EndSection
~~~

### References
1. [Arch Linux `xrandr` manual](https://wiki.archlinux.org/index.php/xrandr)
2. [How to use high (4K) resolutions with older hardware](https://medium.com/@ValdikSS/how-to-use-high-resolutions-with-older-hardware-58577d91b1f8)
3. [How to Set A Custom Screen Resolution in Ubuntu Desktop](http://ubuntuhandbook.org/index.php/2017/04/custom-screen-resolution-ubuntu-desktop/)
