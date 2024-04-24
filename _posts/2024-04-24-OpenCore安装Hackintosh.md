---
layout: post
title: "OpenCore安装Hackintosh"
categories: Unix_Linux
---

# 1 总览
## 1.1 引导文件目录结构
需要准备的EFI引导文件目录结构及部分关键文件从Github下载：[GitHub - acidanthera/OpenCorePkg: OpenCore bootloader](https://github.com/acidanthera/OpenCorePkg)<br />解压出X64目录下的EFI文件夹，EFI引导文件为以下几种：

- Drivers &emsp;&emsp;&nbsp;核心驱动
- Kexts &emsp;&emsp;&emsp;黑苹果与硬件联系的主要驱动
- ACPI &emsp;&emsp;&emsp;&nbsp;硬件信息描述文件
- Tools  &emsp;&emsp;&emsp;工具
- config.plist &nbsp;配置文件

## 1.2 必要工具
GenSMBIOS：用于生成仿冒的苹果电脑序列号等信息<br />[GitHub - corpnewt/GenSMBIOS: Py script that uses acidanthera’s macserial to generate SMBIOS and optionally saves them to a plist.](https://github.com/corpnewt/GenSMBIOS)<br />ProperTree：用于编辑 config.plist<br />[GitHub - corpnewt/ProperTree: Cross platform GUI plist editor written in python.](https://github.com/corpnewt/ProperTree)<br />Python：GenSMBIOS和ProperTree的依赖项<br />[Download Python](https://www.python.org/downloads/)<br />balenaEtcher：制作启动盘<br />[balenaEtcher - Flash OS images to SD cards & USB drives](https://www.balena.io/etcher/)<br />Diskgenius：磁盘工具，用于导入EFI文件<br />[DiskGenius – 正式版下载|免费下载](https://diskgenius.cn/download.php)<br />MountEFI：挂载EFI文件（MacOS中）<br />[https://github.com/corpnewt/MountEFI](https://github.com/corpnewt/MountEFI)<br />OpenCore Configurator：代替ProperTree和MountEFI（MacOS中）<br />[https://mackie100projects.altervista.org/opencore-configurator/](https://mackie100projects.altervista.org/opencore-configurator/)<br />或`brew install --cask opencore-configurator`
# 2 配置EFI文件
## 2.1 Drivers
一般只需要OpenRuntime.efi和HfsPlus.efi（或OpenHfsPlus.efi）。<br />OpenRuntime.efi 是 OpenCore 的核心文件，HfsPlus.efi 是苹果官方的HFS Plus 驱动程序，OpenHfsPlus.efi是开源HFS Plus驱动程序。
## 2.2 Kexts
整个文件夹（*.kext）放到kexts目录下。
**必备**
Lilu.kext：是其他kext依赖项<br />[GitHub - acidanthera/Lilu: Arbitrary kext and process patching on macOS](https://github.com/acidanthera/Lilu)<br />WhateverGreen.kext：是 GPU 的 kext 驱动<br />[GitHub - acidanthera/WhateverGreen: Various patches necessary for certain ATI/AMD/Intel/Nvidia GPUs](https://github.com/acidanthera/WhateverGreen)<br />VitualSMC组件包含VitualSMC.kext、SMCProcessor.kext、SMCSuperIO.kext、SMCLightSensor.kext、SMCBatteryManager.kext、SMCDellSensors.kext<br />VitualSMC.kext：模拟在真实 Mac 上找到的 SMC 芯片<br />SMCProcessor.kext：用于监控 CPU 温度，不适用于基于 AMD CPU 的系统<br />SMCSuperIO.kext：用于监控风扇速度，不适用于基于 AMD CPU 的系统<br />SMCLightSensor.kext：用于笔记本电脑上的环境光传感器，台式机可以忽略<br />SMCBatteryManager.kext：用于测量笔记本电脑上的电池读数，台式机可以忽略<br />SMCDellSensors.kext：仅戴尔计算机使用<br />[GitHub - acidanthera/VirtualSMC: SMC emulator layer](https://github.com/acidanthera/VirtualSMC)<br />AppleALC.kext：是声卡的 kext 驱动<br />[GitHub - acidanthera/AppleALC: Native macOS HD audio for not officially supported codecs](https://github.com/acidanthera/AppleALC)
**可选**
RealtekRTL8111.kext：适用于瑞昱的2.5Gb以太网<br />[LucyRTL8125Ethernet](https://www.insanelymac.com/forum/files/file/1004-lucyrtl8125ethernet/)<br />NVMeFix.kext：用于修复非 Apple NVMe 上的电源管理和初始化<br />[GitHub - acidanthera/NVMeFix](https://github.com/acidanthera/NVMeFix)<br />更多参考[OpenCore 黑苹果安装教程_copcin的博客-CSDN博客](https://blog.csdn.net/raspi_fans/article/details/125144685)
## 2.3 ACPI
需要的文件从OpenCore的文件夹Docs\AcpiSamples\Binaries中获取，各平台要求参阅各小节<br />注：SSDT-PLUG -> SSDT-PLUG-DRTNIA
**[台式机（Desktop）](https://dortania.github.io/Getting-Started-With-ACPI/ssdt-platform.html#desktop)**

| Platforms | **CPU** | **EC** | **AWAC** | **NVRAM** | **USB** |
| --- | --- | --- | --- | --- | --- |
| Penryn | N/A | [SSDT-EC](https://dortania.github.io/Getting-Started-With-ACPI/Universal/ec-fix) | N/A | N/A | N/A |
| Lynnfield and Clarkdale |  |  |  |  |  |
| SandyBridge | [CPU-PM(opens new window)](https://dortania.github.io/OpenCore-Post-Install/universal/pm.html#sandy-and-ivy-bridge-power-management)<br />(Run in Post-Install) |  |  |  |  |
| Ivy Bridge |  |  |  |  |  |
| Haswell | [SSDT-PLUG](https://dortania.github.io/Getting-Started-With-ACPI/Universal/plug) |  |  |  |  |
| Broadwell |  |  |  |  |  |
| Skylake |  | [SSDT-EC-USBX](https://dortania.github.io/Getting-Started-With-ACPI/Universal/ec-fix) |  |  |  |
| Kaby Lake |  |  |  |  |  |
| Coffee Lake |  |  | [SSDT-AWAC](https://dortania.github.io/Getting-Started-With-ACPI/Universal/awac) | [SSDT-PMC](https://dortania.github.io/Getting-Started-With-ACPI/Universal/nvram) |  |
| Comet Lake |  |  |  | N/A | [SSDT-RHUB](https://dortania.github.io/Getting-Started-With-ACPI/Universal/rhub) |
| AMD (15/16h) | N/A |  | N/A |  | N/A |
| AMD (17h) | [SSDT-CPUR for B550 and A520(opens new window)](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-CPUR.aml) |  |  |  |  |

**[高端台式机（High End Desktop）](https://dortania.github.io/Getting-Started-With-ACPI/ssdt-platform.html#high-end-desktop)**

| Platforms | **CPU** | **EC** | **RTC** | **PCI** |
| --- | --- | --- | --- | --- |
| Nehalem and Westmere | N/A | [SSDT-EC](https://dortania.github.io/Getting-Started-With-ACPI/Universal/ec-fix.html) | N/A | N/A |
| Sandy Bridge-E |  |  |  | [SSDT-UNC](https://dortania.github.io/Getting-Started-With-ACPI/Universal/unc0) |
| Ivy Bridge-E |  |  |  |  |
| Haswell-E | [SSDT-PLUG](https://dortania.github.io/Getting-Started-With-ACPI/Universal/plug) | [SSDT-EC-USBX](https://dortania.github.io/Getting-Started-With-ACPI/Universal/ec-fix) | [SSDT-RTC0-RANGE](https://dortania.github.io/Getting-Started-With-ACPI/Universal/awac) |  |
| Broadwell-E |  |  |  |  |
| Skylake-X |  |  |  | N/A |

**[笔记本（Laptop）](https://dortania.github.io/Getting-Started-With-ACPI/ssdt-platform.html#laptop)**

| Platforms | **CPU** | **EC** | **Backlight** | **I2C Trackpad** | **AWAC** | **USB** | **IRQ** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Clarksfield and Arrandale | N/A | [SSDT-EC](https://dortania.github.io/Getting-Started-With-ACPI/Universal/ec-fix) | [SSDT-PNLF](https://dortania.github.io/Getting-Started-With-ACPI/Laptops/backlight) | N/A | N/A | N/A | [IRQ SSDT](https://dortania.github.io/Getting-Started-With-ACPI/Universal/irq) |
| SandyBridge | [CPU-PM(opens new window)](https://dortania.github.io/OpenCore-Post-Install/universal/pm.html#sandy-and-ivy-bridge-power-management)<br />(Run in Post-Install) |  |  |  |  |  |  |
| Ivy Bridge |  |  |  |  |  |  |  |
| Haswell | [SSDT-PLUG](https://dortania.github.io/Getting-Started-With-ACPI/Universal/plug) |  |  | [SSDT-XOSI/SSDT-GPI0](https://dortania.github.io/Getting-Started-With-ACPI/Laptops/trackpad)<br /> (Run in Post-Install) |  |  |  |
| Broadwell |  |  |  |  |  |  |  |
| Skylake |  | [SSDT-EC-USBX](https://dortania.github.io/Getting-Started-With-ACPI/Universal/ec-fix) |  |  |  |  | N/A |
| Kaby Lake |  |  |  |  |  |  |  |
| Coffee Lake (8th Gen) and Whiskey Lake |  |  |  |  | [SSDT-AWAC](https://dortania.github.io/Getting-Started-With-ACPI/Universal/awac) |  |  |
| Coffee Lake (9th Gen) |  |  |  |  |  |  |  |
| Comet Lake |  |  |  |  |  |  |  |
| Ice Lake |  |  |  |  |  | [SSDT-RHUB](https://dortania.github.io/Getting-Started-With-ACPI/Universal/rhub) |  |

续：

| Platforms | **NVRAM** | **IMEI** |
| --- | --- | --- |
| Clarksfield and Arrandale | N/A | N/A |
| Sandy Bridge |  | [SSDT-IMEI](https://dortania.github.io/Getting-Started-With-ACPI/Universal/imei) |
| Ivy Bridge |  |  |
| Haswell |  | N/A |
| Broadwell |  |  |
| Skylake |  |  |
| Kaby Lake |  |  |
| Coffee Lake (8th Gen) and Whiskey Lake |  |  |
| Coffee Lake (9th Gen) | [SSDT-PMC](https://dortania.github.io/Getting-Started-With-ACPI/Universal/nvram) |  |
| Comet Lake | N/A |  |
| Ice Lake |  |  |

## 2.4 Tools
非必须，可留下 OpenShell.efi 方便调试。
## 2.5 config.plist

1. 复制 OpenCore 的文件夹 Docs 里的 Sample.plist， 放到 OC 文件夹根目录，改名为config.plist；
2. 运行ProperTree 下的 ProperTree.bat ，点击 Open，选择config.plist 文件，然后选择 OC Clean Snapshot，选择 OC 文件夹，ProperTree 就帮我们把刚刚收集的那些文件全部引入过来了；
3. 继续配置config.plist，不同硬件参数不同，需参考[Desktop Comet Lake \| OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/config.plist/comet-lake.html)；
4. config.plist中PlatformInfo一节需要序列号等信息，用GenSMBIOS生成，首先需要将OpenCore的文件夹Utilities\macserial中的所有文件复制到GenSMBIOS目录下，运行后输入3，回车，输入机器型号和个数（iMac20,1 1），回车。

# 3 安装
## 3.1 刻录U盘
需要从网上下载MacOS镜像（dmg文件），用balenaEtcher刻录到U盘，用之前准备的EFI替换掉启动盘中的EFI文件夹。
## 3.2 修改BIOS
见[Desktop Comet Lake | OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/config.plist/comet-lake.html#intel-bios-settings)，按照自己的硬件修改
## 3.3 安装排错
最关键的时候：安装！<br />我大致把安装分为2个阶段，分别对应启动器：Install macOS Ventura (external)和macOS Installer。<br />以下仅列出我的报错，更多情况可上网搜索。

**第一阶段**
- 首先会出现很多代码信息，如果屏幕中心出现了禁止符号（🚫），可能是USB接口问题，换USB2.0接口；
- 接着进入安装界面，可能是俄文，可以切换语言，从左上角点Файл然后点Сменить язык..；
- 用磁盘工具抹掉要安装MacOS的磁盘，改为MacOS格式；
- 安装系统，等待走进度条。

**第二阶段**
- 第二阶段仍会出现很多代码信息，如果电脑无限重启，设置config.plist中的SecureBootModel为Disabled（期间还修改了部分BIOS设置，因此不排除是其他原因），参考的是[https://www.tonymacx86.com/threads/err-0x5-rt-gv-wake-failure.307244/](https://www.tonymacx86.com/threads/err-0x5-rt-gv-wake-failure.307244/)；
- 安装系统，等待走进度条和重启若干次，重启后启动器选macOS Installer；
- 进入系统。

# 4 系统配置
## 4.1 取消-v
每次开机和关机都会出很多代码，需要将config.plist中boot-args的-v去掉。<br />正常情况下MacOS中EFI文件不可见，可用MountEFI挂载。
## 4.2 引导转移
U盘拔掉之后BIOS就找不到Hackintosh系统了，这是因为引导文件还在U盘中，可用MountEFI挂载出Hackintosh所在硬盘的EFI分区和U盘的EFI分区，将U盘中EFI引导复制到Hackintosh所在硬盘。
