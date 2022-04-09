# ThinkPad-T580-Hackintosh
# 中文版
# 联想ThinkPad T580上的macOS（本指南也适用于P52s i7-8550u，8250u理论上应该通用，不适用T480s）

此存储库包含10.13或者更高版本系统在Lenovo ThinkPad T580上运行macOS的示例配置
--------------------------------------------------------------------------

机器配置表和驱动情况
---------
* 联想ThinkPad T580 【4k的UHD显示屏：3840x2160（京东方BOE NV156QUM-N44，非触摸）】

* 英特尔i7-8550U CPU

* 内存16GB RAM DDR4 2400-SODIMM

* 东芝Q200EX SSD+128G 东芝nvme联想跟机带

* 英特尔以太网I219-V4 有线网卡

* ~~Mac原装无线网卡bcm94360cs2（原装英特尔AC8265无法正常工作）~~

* ~~Wi-Fi设备芯片组为(0x14E4, 0x117) 显示Airport Extreme，使用AirportBrcmFixup.kext修改信道国家代码~~

* ~~蓝牙设备芯片组20702B0,固件版本:v150 c9318 免驱动~~

* 无线网卡改为DW1820A拆机卡（原装DELL拆机卡无需屏蔽针脚）

* 瑞昱Realtek ALC3287（“ALC257”）通过AppleALC.kext与layout-id：11，支持耳机和自带喇叭之间插拔自动切换。

* Intel UHD Graphics 620（禁用Nvidia MX150，macOS不支持Optimus）

* ACPI热修补电源管理和双电池状态

* SD卡读卡器走usb3.0通道，需要打开此USB端口即可使用

* Thunderbolt 3【BIOS里需要设置“Thunderbolt BIOS Assist”：Disable，“Security level” :No Security(允许自动连接 Thunderbolt 设备)。这样即可使前端类型USB type-c端口在macOS中工作，可以热插拔， DP / HDMI通过USB type-C：视频工作正常，连接扩展坞正常，雷电3设备工作正常，热插拔正常】

* 机器自带独立的HDMI端口：可以输出4k@30HZ到显示器。连接时会显示音频设备HDMI，并正常使用。

* ~~键盘Synaptics触摸板（PS / 2）使用ApplePS2SmartTouchPad.kext，EMlyDinEsH的v4.7b5，支持多点触控手势。~~
* 键盘触摸板已使用acidanthera/VoodooPS2Controller替代，支持多点触控手势，比ApplePS2SmartTouchPad好用
* 键盘F1-F12快捷键已经全部启用，参照https://github.com/MSzturc/ThinkpadAssistant ，鸣谢MSzturc和开发者们

ThinkPad 黑苹果休眠、睡眠和唤醒
----------
* 睡眠和唤醒正常，两种方式

 * 1.hibernatemode 3 （这种睡眠方式可能会影响硬盘性能和寿命，这也是白果笔记本默认睡眠方式)   
往 EFI/CLOVER/kexts/Other和OC引导的kexts下面扔一个 HibernationFixup.kext，可以实现 hibernatemode 3 休眠。
如果合盖后遇到偶发性的随机唤醒或者重启，禁用 proximitywake ：sudo pmset -a proximitywake 0 
 * 2.hibernatemode 0 （这种睡眠方式只写入到内存，推荐使用)  
sudo pmset -a hibernatemode 0  
sudo pmset -a proximitywake 0

已禁用的设备和BIOS设置
-----------
* WWAN（无模块）
* TrackPad Synaptics指纹识别器无法驱动
* BIOS里设置Thunderbolt BIOS Assist为Disable(Enable雷电由BIOS控制决定，Disabled雷电由操作系统控制决定)，“Security level” :No Security(允许自动连接 Thunderbolt 设备和TYPEC)。
* BIOS里设置usb项Always on USB为enable，下面子项Charge in Battery Mode设置为enable（这样雷电口type-c就可以热插拔type-c外设）
* 关闭安全启动

UEFI BIOS固件修订
-----------
* BIOS版本 1.20-1.31通用


目前存在的问题
-------------
* 指纹无法驱动。

备注
--------
解决10.15.x的type-c，HDMI无输出问题：
从10.15开始，S/L/E下无法通过config的KernelToPatch来打补丁了，所以需要自己手动打补丁，我把10.15.4的已打好的补丁放上了，自行替换S/L/E下同名驱动文件。
*  第一种方法：
每次更新大小系统版本后终端里执行一次sudo mount -uw / && killall Finder和sudo kextcache -i /，然后重启机器即可。如果方法一不灵就强制使用方法二暴力破解了。    
*  第二种方法：
打开终端输入一下命令，即可对系统进行读写：  
 sudo mount -uw /    
 killall Finder      
-----
*  下载AppleGraphicsDevicePolicy.kext然后替换到/System/Library/Extensions/AppleGraphicsControl.kext/Contents/PlugIns/下
,修复权限缓存重启即可：sudo touch /System/Library/Extensions/ && sudo kextcache -u /  
* 注意：关于macOS下HDMI和typec输出请使用以下机型：   
 MacBookPro14,1 完美；  
 MacBookPro14,2 完美；   
 MacBookPro15,2 完美，可以同时HDMI，type-c；   
 MacBookPro14,3 type-c，HDMI，无输出-需要定制端口；   
 MacBookPro15,1 type-c，HDMI，无输出-需要定制端口；    
 MacBookPro15,3 type-c，HDMI，无输出-需要定制端口；    
 MacBookPro16,1 type-c，HDMI，无输出-需要定制端口；    
 MacBookPro16,4 type-c，HDMI，无输出-需要定制端口；     
 MacBookPro15,4 type-c输出正常，HDMI正常，不能同时；    
 MacBookPro16,2 type-c输出正常，HDMI正常，不能同时；   
 MacBookPro16,3 type-c输出正常，HDMI正常，不能同时；     
 其他机型比如MacBook Air8，1和8，2还有9，1没测试，有兴趣自己测试一下；    
 尽量避免使用:MacBookPro14,3；MacBookPro15,1；MacBookPro15,3；MacBookPro16,1；MacBookPro16,4这些机型。    



# English Version
# MacOS on Lenovo ThinkPad T580 (this guide also applies to P52s i7-8550u, 8250u should be general in theory, not T480s)

This repository contains a sample configuration for 10.13 or later running macOS on a Lenovo ThinkPad T580
-------------------------------------------------- ------------------------

Machine configuration table and driving situation
---------
* Lenovo ThinkPad T580 [4k UHD display: 3840x2160 (BOE NV156QUM-N44, non-touch)]

* Intel i7-8550U CPU

* Memory 16GB RAM DDR4 2400-SODIMM

* Toshiba Q200EX SSD+128G Toshiba nvme Lenovo with strap

* Intel Ethernet I219-V4 wired network card

* ~~Mac original wireless network card bcm94360cs2 (original Intel AC8265 cannot work normally)~~

* ~~The Wi-Fi device chipset is (0x14E4, 0x117) Display Airport Extreme, use AirportBrcmFixup.kext to modify the channel country code~~

* ~~Bluetooth device chipset 20702B0, firmware version: v150 c9318 driver-free~~

* The wireless network card is changed to DW1820A disassembly card (the original DELL disassembly card does not need to shield pins)

* Realtek Realtek ALC3287 ("ALC257") supports AppleALC.kext and layout-id: 11 to support automatic switching between headphones and its own speakers.

* Intel UHD Graphics 620 (Nvidia MX150 is disabled, macOS does not support Optimus)

* ACPI hot patch power management and dual battery status

* SD card reader uses USB3.0 channel, you need to open this USB port to use

* Thunderbolt 3 [Thunderbolt BIOS Assist" in the BIOS needs to be set: Disable, "Security level": No Security (allows automatic connection of Thunderbolt devices). In this way, the front-end type USB type-c port can work in macOS and can be hot-swapped. DP/HDMI through USB type-C: the video works normally, the docking station works normally, the Thunderbolt 3 device works normally, and the hot plug works normally]

* The machine comes with an independent HDMI port: it can output 4k@30HZ to the display. When connected, the audio device HDMI will be displayed and used normally.

* ~~Keyboard Synaptics touchpad (PS/2) uses ApplePS2SmartTouchPad.kext, v4.7b5 of EMlyDinEsH, supports multi-touch gestures.~~
* The keyboard touchpad has been replaced with acidanthera/VoodooPS2Controller, supports multi-touch gestures, and is easier to use than ApplePS2SmartTouchPad
* All keyboard shortcuts F1-F12 have been enabled, please refer to https://github.com/MSzturc/ThinkpadAssistant, and thank MSzturc and developers

ThinkPad Black Apple Sleep, Sleep and Wake
----------
* Sleep and wake up normally, two ways

* 1. hibernatemode 3 (this sleep mode may affect hard disk performance and life)  
Toss a HibernationFixup.kext under the kexts of EFI/CLOVER/kexts/Other and OC,hibernatemode 3 sleep can be achieved.
If you encounter an occasional random wake-up or restart after closing the lid, disable proximitywake: sudo pmset -a proximitywake 0
* 2. hibernatemode 0 (this sleep mode is only written to memory，Recommended Use)  
sudo pmset -a hibernatemode 0  
sudo pmset -a proximitywake 0  

Disabled devices and BIOS settings
-----------
* WWAN (no module)
* TrackPad Synaptics fingerprint reader cannot be driven
* Set Thunderbolt BIOS Assist to Disable in the BIOS (Enable Thunderbolt is determined by BIOS control, Disabled Thunderbolt is determined by operating system control), "Security level": No Security (allows automatic connection of Thunderbolt devices and TYPEC).
* In the BIOS, set the USB item Always on USB to enable, and the following sub-item Charge in Battery Mode is set to enable (so that the lightning port type-c can be hot-swapped type-c peripherals)
* Turn off safe boot

UEFI BIOS firmware revision
-----------
* BIOS version 1.20-1.31 common


currently existing problems
-------------
* Fingerprint cannot be driven.

Remarks
--------
Solve the 10.15.x type-c, HDMI no output problem:
Starting from 10.15, it is not possible to patch through KernelToPatch in config under S/L/E, so I need to manually patch it. I put the patch that has been patched in 10.15.4 and replace it under S/L/E. Driver file with the same name.
*  the first method:
Run sudo mount -uw /&& killall Finder and sudo kextcache -i / in the terminal once after updating the size system version, and then restart the machine. If the first method fails, the second method is forced to brute force.
*  The second method:
Open the terminal and enter the command to read and write the system:
 sudo mount -uw /  
 killall Finder  
-----
* Download AppleGraphicsDevicePolicy.kext and replace it under /System/Library/Extensions/AppleGraphicsControl.kext/Contents/PlugIns/
, Just fix the permission cache and restart: sudo touch /System/Library/Extensions/ && sudo kextcache -u /
* Note: For HDMI and typec output under macOS, please use the following models:       
 MacBookPro14,1 perfect;       
 MacBookPro14,2 perfect;     
 MacBookPro15,2 perfect, can simultaneously HDMI, type-c;     
 MacBookPro14,3 type-c, HDMI, no output -Custom port required;     
 MacBookPro15,1 type-c, HDMI, no output -custom port required;     
 MacBookPro15,3 type-c, HDMI, no output -custom port required;     
 MacBookPro16,1 type-c, HDMI, no output - required Custom port;      
 MacBookPro16, 4 type-c, HDMI, no output - custom port is required;      
 MacBookPro15, 4 type-c output is normal, HDMI is normal, but not at the same time;      
 MacBookPro16, 2 type-c output is normal, HDMI is normal, not at the same time;      
 MacBookPro16 ,3 type-c output is normal, HDMI is normal, not at the same time;      
 other models such as MacBook Air8, 1 and 8, 2 and 9, 1 have not been tested, if you are interested in testing it yourself;      
 try to avoid using: MacBookPro14,3; MacBookPro15, 1; MacBookPro15,3; MacBookPro16,1; MacBookPro16,4 these models.     
