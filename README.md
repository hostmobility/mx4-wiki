![](http://hostmobility.org/mx4/pics/mx-4-family-web.jpg)

***

##Table of Contents##

- [Overview](#overview)
	- [Soft deliverables](#soft-deliverables)
		- [Platform](#platform)
			- [Toolchain](#toolchain)
			- [Release structure](#release-structure)
			- [Firmware update](#firmware-update)
		- [Source code](#source-code)
		- [Support](#support)
			- [Wiki](#wiki)
		- [Build server](#build-server)
- [Build Environment](#build-environment)
- [Bootlog](#bootlog)
- [Default Login](#default-login)
- [Reset Cause](#reset-cause)
- [Timekeeping](#timekeeping)
	- [Internal RTC](#internal-rtc)
	- [GPS RTC](#gps-rtc)
- [Sound](#sound)
- [Power Management](#power-management)
	- [Running](#running)
	- [Sleep](#sleep)
		- [Overview of suspend design](#overview-of-suspend-design)
		- [Enter Sleep](#enter-sleep)
		- [Wakeup](#wakeup)
			- [Digital Inputs](#wake-on-digital-inputs)
			- [Analog Inputs](#wake-on-analog-inputs)
			- [CAN](#wake-on-can)
	- [Deep Sleep](#deep-sleep)
	- [Shutdown/Cutoff](#shutdown)
- [Communication Interfaces](#communication-interfaces)
	- [Serial Devices](#serial-devices)
	- [CAN](#can)
		- [Configure CAN](#configuring-can-bus)
		- [CAN tools](#can-tools)
		- [cansend example](#cansend-example)
		- [Statistics](#statistics)
	- [Wifi](#wifi)
		- [Usage](#usage)
		- [Wifi as Access Point](#wifi-as-access-point)
			- [Linux Kernel](#linux-kernel)
			- [Target Setup](#target-setup)
	- [Modem](#modem)
		- [Creating an PPP connection](#creating-an-ppp-connection)
		- [GPS](#gps)
        - [Watchdog](#watchdog)
	- [J1708](#j1708)
	- [LIN](#lin)
- [Digital/Analog I/O](#digital-and-analog)
	- [Digital](#digital)
		- [Read GPIO value](#read-gpio-value)
		- [Write GPIO value](#write-gpio-value)
		- [Set input as interrupt](#set-input-as-interrupt)
	- [Analog](#analog)
- [LED](#led)
	- [Frequency](#frequency)
	- [Flash config](#flash-config)
	- [Current LED](#current-led)
	- [Oneshot](#oneshot)
    - [Configuration examples](#example-of-configuration)
	- [Pre configured LED behavior](#pre-configured-led-behavior)
		- [PWR](#pwr)
		- [STANDBY-STATE](#standby-state)
		- [ERROR-STATE](#error-state)
		- [FIRMWARE-UPGRADE](#firmware-upgrade)
- [Package Manager](#package-manager)
- [Accelerometer](#accelerometer)

## Overview

Work in progress...

### Soft deliverables

When you buy a MX-4 hardware from Host Mobility AB the following is included.

#### Platform

Host Mobility AB provides a complete Linux platform with driver support for all hardware interfaces and with a customizable distribution.

All hardware interfaces are accessible via well defined API's. We try to reuse the standard Linux way of doing things as much as we can. This way the platform environment is familiar to developers who has worked with embedded Linux in their past.

Main software components:

- Tool-chain
	- Tegra2: Linaro GCC 4.7-2013.09 (http://releases.linaro.org/13.09/components/toolchain/gcc-linaro/4.7)
- Linux (Tegra2: 3.1.10)
- U-boot (Tegra2: 2011.06)
- Ångstrom distribution built with yocto (dylan branch)(https://www.yoctoproject.org/)
- Co-processor firmware

##### Toolchain

Toolchain binaries are generated with yocto command `-c populate_sdk`. Toolchains are built for both 32 and 64 bit systems.

There are two flavors of sysroots. The minimal is based on `console-vcc-base-image` and the other one is based on `lxde-mx4-image`.

They are available on following link http://www.hostmobility.org:8080/tools/.

There is a problem with installation where you get segmentation faults when trying to run a binary. See this link http://developer.toradex.com/how-to/how-to-set-up-qt-creator-to-cross-compile-for-embedded-linux#Install_the_SDK for a workaround.

Linaro 2013.04 is also compatible with the above toolchains. https://launchpad.net/linaro-toolchain-binaries/trunk/2013.04/+download/gcc-linaro-arm-linux-gnueabihf-4.7-2013.04-20130415_linux.tar.xz.

##### Release structure

When master branch is considered to be stable with a certain amount of new functionality there will be a new BSP release.

First the mx4-bsp-vx.x.x tag is created in mx4 and in all sub repositories. The tag is converted to a branch in which one can push bug fixes. Each BSP release will get a Jenkins job defined.

The above structure is needed since we have alot of products in the same tree and we most likely will break something when adding new products to the tree. This way we have always our stable branches while developing new functionality.

##### Firmware Update

Host Mobility AB provides a simple method to update the firmware in the MX-4 hardware.

This method is based on a u-boot script (named "hmupdate.img") which is able to update all software components (Linux kernel, u-boot, distribution, co-processor firmware).

This is easily done by placing "hmupdate.img" in the root of a USB flash drive and simply restarting the MX-4 system with the USB flash drive plugged in.

The image ("hmupdate.img") can also be placed in the internal nand flash in /boot directory which will trigger an update as well. This method could be integrated in a customer application for over the air updates.

**After BSP release 1.1.x the user needs to trigger a probe for an update image (hmupdate.img), this was done on every startup on earlier releases.**

**To update from USB you need to plug in the USB containing an image ("hmupdate.img") and press the reset button.**

**To trigger an update from /boot one needs to set the `firmware_update` u-boot environment variable to `true`. See below example on how to do that.**

**Setting `firmware_update` to `true` will enable USB update as well if an image is present on the USB drive.**
***

```
# Set u-boot environment variable
root@mx4_vf-1000000:/opt/hm/fw_env# ./fw_setenv firmware_update true
```

#### Source code

Host Mobility's provides read-only access to our software repositories hosted at https://github.com/hostmobility.

With this access you can fork the repository, create pull-requests, create issues and clone the repository and build the whole platform from scratch.

#### Support

Host Mobility AB provides first class support.

We will help you get started with MX-4 development and once the initial steps are done we also provide tips and tricks to optimize your application to our platform.

Beside the documentation and wiki you can also contact Host Mobility developers directly with your questions. See http://hostmobility.com for contact information.

##### Wiki
http://hostmobility.github.io/mx4/

#### Build Server

Host Mobility AB provides access to our build server which is based on Jenkins software. Here you can download the latest and greatest software for your MX-4 platform.

It is also possible to setup a customer specific build job on request where one could integrate the customer application in the MX-4 platform build system or build a branch of the MX-4 platform repository.

## Build Environment

Before buildning, be aware of:
<ul>
<li>to have approximately 30 GB amount of free space on your harddrive</li>
<li>that a SSD will make the build process go faster </li>
<li>we recommend to have 4 GB of RAM</li>
<li>to have at least 2 cores, but we recommend 4 cores or more</li>
</ul>

### Setup Docker
	
Install Docker: [Docker Install] <br />
Create Dockerfile: touch Dockerfile <br />
Copy this template into the Dockerfile and edit it:
<pre>	
FROM debian
# Edit Your Name and ”your@mail.com”
MAINTAINER <b>Your Name</b> "<b>your@mail.com</b>"
# Edit your username
ARG user=<b>your username</b>
# Edit your group, mostly the same as username
ARG group=<b>your group</b> 
# Edit your user id. Command to see user id, id -u $username
ARG uid=<b>your user id</b>
# Edit your group id. Command to see group id, id -g $username
ARG gid=<b>your group id</b>


# We need to change to root to be able to install with apt-get 
USER root
RUN apt-get update && apt-get install -y gawk wget git-core sudo cpio \
diffstat unzip texinfo gcc-multilib u-boot-tools rsync cbootimage bc \
build-essential kmod chrpath socat mtd-utils device-tree-compiler mtools \
lzop dosfstools parted libncurses5-dev patchutils tmux vim curl python \
libsdl1.2-dev && rm -rf /var/lib/apt/lists/*

ENV USER_HOME /home/${user}

RUN groupadd -g ${gid} ${group} \
&& useradd -d "$USER_HOME" -u ${uid} -g ${gid} -m -s /bin/bash ${user}
# Edit username to your username
RUN adduser <b>username</b> sudo
# Edit username to your username
RUN echo "<b>username</b> ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Switch to user
USER ${user}
WORKDIR ${USER_HOME}

# Creates project folder. Returns no error if folder aldready exists
RUN mkdir -p <b>project</b>

# Edit "your@mail.com"
RUN git config --global user.email "<b>your@mail.com</b>"
# Edit "Your Name"
RUN git config --global user.name "<b>Your Name</b>"
RUN git config --global push.default simple
</pre>
#### Run Docker image
<pre>
# Edit user id 1000 with your user id. 
# Edit username with your username (both in ssh- and project path).
# Edit path to the project /home/username/project if needed.
# The project folder is the link between your environment and the Docker environment.
# Everything you put in the project folder from the Docker will be accessible outside the Docker.
# To make --privileged to work, it assumes that you have a SSH key associated with your github account.
# If SSH key is not set up correctly, you will be asked for username and password during the git clone process.
sudo docker run -it -u <b>1000</b> --privileged -v ~/.ssh:/home/<b>username</b>/.ssh -v ~/project:/home/<b>username</b>/<b>project</b> hostmobility/mx4
</pre>

### Example to setup environment for MX-4 CT, branch BSP-v1.5.x

#### Git clone from repository
	# If SSH key is associated correctly with your account on github, run the following commands:
	git clone git@github.com:hostmobility/mx4 -b mx4-bsp-v1.5.x
	git clone git@github.com:hostmobility/mx4-pic -b mx4-bsp-v1.5.x mx4/pic
	# If not correctly associated, run follow commands:
	git clone https://github.com/hostmobility/mx4/ -b mx4-bsp-v1.5.x
	git clone https://github.com/hostmobility/mx4-pic -b mx4-bsp-v1.5.x mx4/pic

#### Build CT-BSP-v1.5.x
<pre>
cd mx4/t20
# Remember to edit username to your own username and project if you
# named your project folder to something else.
# edit $(nproc) to number of cores you want to use while building.
./make_system.sh -t ct -r /home/<b>username</b>/<b>project</b>/rootfs/CT -d /home/<b>username</b>/<b>project</b>/yocto-1.5.x -g -k -u -j <b>$(nproc)</b> -m -T 512
</pre>
### Example to setup environment for MX-4 V61

	# All products use mx4 and mx4-pic repository
	git clone git@github.com:hostmobility/mx4 -b mx4-2.0
	git clone git@github.com:hostmobility/mx4-pic -b mx4-2.0 mx4/pic


	# Below are only required for products who have Linux and U-boot
	# outside of the mx4 repository
	git clone git@github.com:hostmobility/linux-toradex.git -b hm_vf_4.4 mx4/t20/linux_vf
	git clone git@github.com:hostmobility/u-boot-toradex.git -b 2015.04-hm mx4/t20/u-boot_vf

### Example to build Board Support Package for MX-4 V61
<pre>
cd mx4/t20
# edit $(nproc) to number of cores you want to use while building.
[work in progress]
./make_system.sh -t v61 -r /home/<b>username</b>/<b>project</b>/rootfs/v61 -d /home/<b>username</b>/<b>project</b>/ -u -k -j <b>$(nproc)</b> -g
</pre>

List of supported distros can be found at http://www.yoctoproject.org/docs/1.4.1/ref-manual/ref-manual.html#detailed-supported-distros

Following packages need to be installed. Below example is Ubuntu 14.10
```
sudo apt-get install subversion git u-boot-tools cbootimage curl gawk wget \
git-core diffstat unzip texinfo build-essential chrpath autoconf flex bison \
device-tree-compiler mtd-utils lzop
```

## Bootlog

Output from `dmesg` on a MX-4 GTT with a gtt-standalone image

		[    0.000000] Linux version 3.1.10 (mirza@mirza-hm) (gcc version 4.7.4 20130903 (prerelease) (Linaro GCC 4.7-2013.09) ) #1 SMP PREEMPT Fri Apr 4 14:41:26 CEST 2014
		[    0.000000] CPU: ARMv7 Processor [411fc090] revision 0 (ARMv7), cr=10c5387d
		[    0.000000] CPU: VIPT nonaliasing data cache, VIPT aliasing instruction cache
		[    0.000000] Machine: Toradex Colibri T20
		[    0.000000] Ignoring unrecognised tag 0x54410008
		[    0.000000] Found fbmem: 00c00000@1f200000
		[    0.000000] Found nvmem: 00200000@1fe00000
		[    0.000000] Tegra reserved memory:
		[    0.000000] LP0:                     00000000 - 00000000
		[    0.000000] Bootloader framebuffer:  1f200000 - 1fdfffff
		[    0.000000] Bootloader framebuffer2: 00000000 - 00000000
		[    0.000000] Framebuffer:             1e500000 - 1edfffff
		[    0.000000] 2nd Framebuffer:         1ee00000 - 1fdfffff
		[    0.000000] Carveout:                1fe00000 - 1fffffff
		[    0.000000] Vpr:                     00000000 - 00000000
		[    0.000000] Memory policy: ECC disabled, Data cache writealloc
		[    0.000000] On node 0 totalpages: 123904
		[    0.000000] free_area_init_node: node 0, pgdat c0950600, node_mem_map c0a09000
		[    0.000000]   Normal zone: 996 pages used for memmap
		[    0.000000]   Normal zone: 0 pages reserved
		[    0.000000]   Normal zone: 122908 pages, LIFO batch:31
		[    0.000000] Tegra SKU: 8 Rev: A04 CPU Process: 1 Core Process: 2 Speedo ID: 1
		[    0.000000] Tegra Revision: A04 prime SKU: 0x8 CPU Process: 1 Core Process: 2
		[    0.000000] L310 cache controller enabled
		[    0.000000] l2x0: 8 ways, CACHE_ID 0x410000c4, AUX_CTRL 0x7e080001, Cache size: 1048576 B
		[    0.000000] PERCPU: Embedded 8 pages/cpu @c0df6000 s10336 r8192 d14240 u32768
		[    0.000000] pcpu-alloc: s10336 r8192 d14240 u32768 alloc=8*4096
		[    0.000000] pcpu-alloc: [0] 0 [0] 1
		[    0.000000] Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 122908
		[    0.000000] Kernel command line: video=tegrafb vmalloc=128M usb_high_speed=1 quiet fbcon=map:1 consoleblank=0 ubi.mtd=USR root=ubi0:rootfs rw rootfstype=ubifs mtdparts=tegra_nand:256000K@791552K(CONFIG),767744K@23808K(USR),3072K@0K(BCT),256K@4096K(PT),2048K@5376K(EBT),256K@8448K(BMP),2048K@9728K(ENV),8192K@12800K(LNX),256K@22272K(ARG) asix_mac=00:14:2D:38:BD:64 no_console_suspend=1 console=tty1 console=ttyS0,115200n8 debug_uartport=lsport,0 mem=498M@0M fbmem=12M@498M nvmem=2M@510M
		[    0.000000] PID hash table entries: 2048 (order: 1, 8192 bytes)
		[    0.000000] Dentry cache hash table entries: 65536 (order: 6, 262144 bytes)
		[    0.000000] Inode-cache hash table entries: 32768 (order: 5, 131072 bytes)
		[    0.000000] Memory: 484MB = 484MB total
		[    0.000000] Memory: 480840k/480840k available, 29112k reserved, 0K highmem
		[    0.000000] Virtual kernel memory layout:
		[    0.000000]     vector  : 0xffff0000 - 0xffff1000   (   4 kB)
		[    0.000000]     fixmap  : 0xfff00000 - 0xfffe0000   ( 896 kB)
		[    0.000000]     DMA     : 0xff000000 - 0xffe00000   (  14 MB)
		[    0.000000]     vmalloc : 0xdf800000 - 0xf8000000   ( 392 MB)
		[    0.000000]     lowmem  : 0xc0000000 - 0xdf200000   ( 498 MB)
		[    0.000000]     pkmap   : 0xbfe00000 - 0xc0000000   (   2 MB)
		[    0.000000]     modules : 0xbf000000 - 0xbfe00000   (  14 MB)
		[    0.000000]       .text : 0xc0008000 - 0xc0885c64   (8696 kB)
		[    0.000000]       .init : 0xc0886000 - 0xc08d9860   ( 335 kB)
		[    0.000000]       .data : 0xc08da000 - 0xc0952c14   ( 484 kB)
		[    0.000000]        .bss : 0xc0952c38 - 0xc09c8288   ( 470 kB)
		[    0.000000] Preemptible hierarchical RCU implementation.
		[    0.000000] NR_IRQS:480
		[    0.000000] sched_clock: 32 bits at 1000kHz, resolution 1000ns, wraps every 4294967ms
		[    0.000000] Console: colour dummy device 80x30
		[    0.000000] console [tty1] enabled
		[    0.000250] Calibrating delay loop... 1987.37 BogoMIPS (lpj=9936896)
		[    0.060058] pid_max: default: 32768 minimum: 301
		[    0.060248] Mount-cache hash table entries: 512
		[    0.060829] Initializing cgroup subsys blkio
		[    0.060896] CPU: Testing write buffer coherency: ok
		[    0.060938] ftrace: allocating 23941 entries in 71 pages
		[    0.081392] CPU1: Booted secondary processor
		[    0.081473] Brought up 2 CPUs
		[    0.081484] SMP: Total of 2 processors activated (3974.75 BogoMIPS).
		[    0.082075] devtmpfs: initialized
		[    0.084703] print_constraints: dummy:
		[    0.084889] NET: Registered protocol family 16
		[    0.085603] host1x bus init
		[    0.085731] Serial: 8250/16550 driver, 9 ports, IRQ sharing disabled
		[    0.352499] Selecting UARTE as the debug console
		[    0.352517] The debug console clock name is uarte
		[    0.352704] serial8250.0: ttyS0 at MMIO 0x70006400 (irq = 123) is a Tegra
		[    0.352776] console [ttyS0] enabled
		[    0.400971] serial8250.1: ttyS1 at MMIO 0xd0200000 (irq = 277) is a 16550A
		[    0.540731] tegra_init_emc: MEMPHIS MEM2G16D2DABG 333MHz memory found
		[    0.541217] gpio_request: gpio-0 (P73 - GYRO-INT1) status -16
		[    0.541228] gpio_request(P73 - GYRO-INT1) failed, err = -16
		[    0.541238] gpio_request: gpio-215 (P172 - GYRO-INT2) status -16
		[    0.541247] gpio_request(P172 - GYRO-INT2) failed, err = -16
		[    0.541635] Enabling gpio wakeup on irq 215
		[    0.541643] Setting wake type to rising
		[    0.541652] Enabling gpio wakeup on irq 206
		[    0.541659] Setting wake type to falling
		[    0.541669] Host Mobility MX4
		[    0.541933] tegra_arb_init: initialized
		[    0.542003] tegra_iovmm_register: added iovmm-gart
		[    0.551288] bio: create slab <bio-0> at 0
		[    0.551938] SCSI subsystem initialized
		[    0.552608] usbcore: registered new interface driver usbfs
		[    0.552689] usbcore: registered new interface driver hub
		[    0.552776] usbcore: registered new device driver usb
		[    0.553673] tps6586x 4-0034: found TPS658643, VERSIONCRC is 03
		[    0.556062] print_constraints: REG-SM_0: 725 <--> 1500 mV at 1275 mV normal standby
		[    0.557450] print_constraints: REG-SM_1: 725 <--> 1500 mV at 1100 mV normal standby
		[    0.559020] print_constraints: REG-SM_2: 1700 <--> 1800 mV at 1800 mV normal standby
		[    0.559791] print_constraints: REG-LDO_0: 1200 <--> 3300 mV at 1200 mV normal standby
		[    0.561043] print_constraints: REG-LDO_1: 725 <--> 1500 mV at 1100 mV normal standby
		[    0.562093] print_constraints: REG-LDO_2: 725 <--> 1500 mV at 1200 mV normal standby
		[    0.562841] print_constraints: REG-LDO_3: 1250 <--> 3300 mV at 1800 mV normal standby
		[    0.563790] print_constraints: REG-LDO_4: 1700 <--> 2475 mV at 1800 mV normal standby
		[    0.564341] print_constraints: REG-LDO_5: 1250 <--> 3300 mV at 2850 mV normal standby
		[    0.565596] print_constraints: REG-LDO_6: 2850 mV normal standby
		[    0.566842] print_constraints: REG-LDO_7: 3300 mV normal standby
		[    0.568104] print_constraints: REG-LDO_8: 1800 mV normal standby
		[    0.568489] print_constraints: REG-LDO_9: 1250 <--> 3300 mV at 2850 mV normal standby
		[    0.568912] Advanced Linux Sound Architecture Driver Version 1.0.24.
		[    0.569332] Bluetooth: Core ver 2.16
		[    0.569387] NET: Registered protocol family 31
		[    0.569396] Bluetooth: HCI device and connection manager initialized
		[    0.569410] Bluetooth: HCI socket layer initialized
		[    0.569420] Bluetooth: L2CAP socket layer initialized
		[    0.569442] Bluetooth: SCO socket layer initialized
		[    0.569703] cfg80211: Calling CRDA to update world regulatory domain
		[    0.570258] Switching to clocksource timer_us
		[    0.570477] Switched to NOHz mode on CPU #0
		[    0.571430] Switched to NOHz mode on CPU #1
		[    0.580210] nvmap_page_pool_init: nvmap uc page pool size=15026 pages
		[    0.589492] nvmap_page_pool_init: nvmap pool = uc, highmem=0, pool_size=15026,totalram=120210, freeram=104495, totalhigh=0, freehigh=0
		[    0.634041] nvmap_page_pool_init: nvmap wc page pool size=15026 pages
		[    0.643341] nvmap_page_pool_init: nvmap pool = wc, highmem=0, pool_size=15026,totalram=120210, freeram=89398, totalhigh=0, freehigh=0
		[    0.687299] nvmap_page_pool_init: nvmap iwb page pool size=15026 pages
		[    0.696571] nvmap_page_pool_init: nvmap pool = iwb, highmem=0, pool_size=15026,totalram=120210, freeram=74301, totalhigh=0, freehigh=0
		[    0.741187] tegra-nvmap tegra-nvmap: created carveout iram (255KiB)
		[    0.742166] tegra-nvmap tegra-nvmap: created carveout generic-0 (2048KiB)
		[    0.750899] NET: Registered protocol family 2
		[    0.751046] IP route cache hash table entries: 4096 (order: 2, 16384 bytes)
		[    0.751356] TCP established hash table entries: 16384 (order: 5, 131072 bytes)
		[    0.751571] TCP bind hash table entries: 16384 (order: 5, 196608 bytes)
		[    0.751862] TCP: Hash tables configured (established 16384 bind 16384)
		[    0.751872] TCP reno registered
		[    0.751884] UDP hash table entries: 256 (order: 1, 8192 bytes)
		[    0.751906] UDP-Lite hash table entries: 256 (order: 1, 8192 bytes)
		[    0.752091] NET: Registered protocol family 1
		[    0.752371] RPC: Registered named UNIX socket transport module.
		[    0.752382] RPC: Registered udp transport module.
		[    0.752391] RPC: Registered tcp transport module.
		[    0.752399] RPC: Registered tcp NFSv4.1 backchannel transport module.
		[    0.753242] host1x host1x: initialized
		[    0.753322] PMU: registered new PMU device of type 0
		[    0.770261] nfs4filelayout_init: NFSv4 File Layout Driver Registering...
		[    0.770584] fuse init (API version 7.17)
		[    0.770889] yaffs: yaffs built Apr  4 2014 14:40:30 Installing.
		[    0.770924] msgmni has been set to 939
		[    0.771894] io scheduler noop registered (default)
		[    0.773120] mpe mpe: initialized
		[    0.774014] gr3d gr3d: initialized
		[    0.774854] gr2d gr2d: initialized
		[    0.775035] isp isp: initialized
		[    0.775220] vi vi: initialized
		[    0.778592] tegradc tegradc.0: probed
		[    0.778922] tegradc tegradc.0: probed
		[    0.795868] tegradc tegradc.1: probed
		[    0.801793] Console: switching to colour frame buffer device 80x30
		[    0.806485] tegradc tegradc.1: probed
		[    0.806987] tegra_uart.0: ttyHS0 at MMIO 0x70006000 (irq = 68) is a TEGRA_UART
		[    0.830305] Registered UART port ttyHS0
		[    0.830392] tegra_uart.1: ttyHS1 at MMIO 0x70006040 (irq = 69) is a TEGRA_UART
		[    0.860304] Registered UART port ttyHS1
		[    0.860385] tegra_uart.2: ttyHS2 at MMIO 0x70006200 (irq = 78) is a TEGRA_UART
		[    0.950306] Registered UART port ttyHS2
		[    0.950389] tegra_uart.3: ttyHS3 at MMIO 0x70006300 (irq = 122) is a TEGRA_UART
		[    1.150310] Registered UART port ttyHS3
		[    1.150405] Initialized tegra uart driver
		[    1.155747] brd: module loaded
		[    1.158632] loop: module loaded
		[    1.158777] L3G4200D gyroscope driver
		[    1.159676] tegra_nand: 1 NAND chip(s) found (vend=0x2c, dev=0xa3) (Micron NAND 1GiB 1,8V 8-bit)
		[    1.159695] tegra_nand: NVIDIA Tegra NAND controller @ base=0x70008000 irq=56.
		[    1.464894] 9 cmdlinepart partitions found on MTD device tegra_nand
		[    1.464906] Creating 9 MTD partitions on "tegra_nand":
		[    1.464919] 0x000030500000-0x00003ff00000 : "CONFIG"
		[    1.465901] 0x000001740000-0x000030500000 : "USR"
		[    1.467243] 0x000000000000-0x000000300000 : "BCT"
		[    1.467967] 0x000000400000-0x000000440000 : "PT"
		[    1.468656] 0x000000540000-0x000000740000 : "EBT"
		[    1.469357] 0x000000840000-0x000000880000 : "BMP"
		[    1.470055] 0x000000980000-0x000000b80000 : "ENV"
		[    1.470767] 0x000000c80000-0x000001480000 : "LNX"
		[    1.471480] 0x0000015c0000-0x000001600000 : "ARG"
		[    1.472505] UBI: attaching mtd1 to ubi0
		[    1.472517] UBI: physical eraseblock size:   262144 bytes (256 KiB)
		[    1.472526] UBI: logical eraseblock size:    253952 bytes
		[    1.472535] UBI: smallest flash I/O unit:    4096
		[    1.472543] UBI: VID header offset:          4096 (aligned 4096)
		[    1.472552] UBI: data offset:                8192
		[    3.126606] UBI: max. sequence number:       248
		[    3.133000] UBI: attached mtd1 to ubi0
		[    3.133009] UBI: MTD device name:            "USR"
		[    3.133017] UBI: MTD device size:            749 MiB
		[    3.133026] UBI: number of good PEBs:        2999
		[    3.133033] UBI: number of bad PEBs:         0
		[    3.133041] UBI: number of corrupted PEBs:   0
		[    3.133048] UBI: max. allowed volumes:       128
		[    3.133056] UBI: wear-leveling threshold:    4096
		[    3.133063] UBI: number of internal volumes: 1
		[    3.133071] UBI: number of user volumes:     1
		[    3.133078] UBI: available PEBs:             0
		[    3.133085] UBI: total number of reserved PEBs: 2999
		[    3.133093] UBI: number of PEBs reserved for bad PEB handling: 29
		[    3.133102] UBI: max/mean erase counter: 4/1
		[    3.133109] UBI: image sequence number:  0
		[    3.133131] UBI: background thread "ubi_bgt0d" started, PID 61
		[    3.133426] vcan: Virtual CAN interface driver
		[    3.133438] CAN device driver interface
		[    3.133445] sja1000 CAN netdevice driver
		[    3.133848] sja1000_platform sja1000_platform.0: sja1000_platform device registered (reg_base=df800000, irq=205)
		[    3.134183] sja1000_platform sja1000_platform.1: sja1000_platform device registered (reg_base=df802000, irq=198)
		[    3.134532] sja1000_platform sja1000_platform.2: sja1000_platform device registered (reg_base=df86e000, irq=296)
		[    3.134877] sja1000_platform sja1000_platform.3: sja1000_platform device registered (reg_base=df87e000, irq=297)
		[    3.135227] sja1000_platform sja1000_platform.4: sja1000_platform device registered (reg_base=df952000, irq=298)
		[    3.135556] sja1000_platform sja1000_platform.5: sja1000_platform device registered (reg_base=df954000, irq=299)
		[    3.135632] PPP generic driver version 2.4.2
		[    3.135798] PPP Deflate Compression module registered
		[    3.135808] PPP BSD Compression module registered
		[    3.136418] PPP MPPE Compression module registered
		[    3.136430] NET: Registered protocol family 24
		[    3.136774] tun: Universal TUN/TAP device driver, 1.6
		[    3.136784] tun: (C) 1999-2004 Max Krasnyansky <maxk@qualcomm.com>
		[    3.137003] usbcore: registered new interface driver asix
		[    3.137046] usbcore: registered new interface driver cdc_ether
		[    3.137087] usbcore: registered new interface driver cdc_subset
		[    3.137149] usbcore: registered new interface driver rt2800usb
		[    3.137166] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
		[    3.137232] tegra USB phy - inst[0] platform info:
		[    3.137241] port_otg: yes
		[    3.137247] has_hostpc: no
		[    3.137253] phy_interface: USB_PHY_INTF_UTMI
		[    3.137260] op_mode: TEGRA_USB_OPMODE_HOST
		[    3.137267] vbus_gpio: -1
		[    3.137273] vbus_gpio_inverted: 0
		[    3.137279] vbus_reg: NULL
		[    3.137285] hot_plug: enabled
		[    3.137292] remote_wakeup: disabled
		[    3.141231] tegra-ehci tegra-ehci.0: Tegra EHCI Host Controller
		[    3.141326] tegra-ehci tegra-ehci.0: new USB bus registered, assigned bus number 1
		[    3.170329] tegra-ehci tegra-ehci.0: irq 52, io mem 0xc5000000
		[    3.190335] tegra-ehci tegra-ehci.0: USB 2.0 started, EHCI 1.00
		[    3.190420] usb usb1: New USB device found, idVendor=1d6b, idProduct=0002
		[    3.190433] usb usb1: New USB device strings: Mfr=3, Product=2, SerialNumber=1
		[    3.190445] usb usb1: Product: Tegra EHCI Host Controller
		[    3.190454] usb usb1: Manufacturer: Linux 3.1.10 ehci_hcd
		[    3.190463] usb usb1: SerialNumber: tegra-ehci.0
		[    3.190881] hub 1-0:1.0: USB hub found
		[    3.190902] hub 1-0:1.0: 1 port detected
		[    3.191177] tegra USB phy - inst[1] platform info:
		[    3.191186] port_otg: no
		[    3.191193] has_hostpc: no
		[    3.191199] phy_interface: USB_PHY_INTF_ULPI_LINK
		[    3.191206] op_mode: TEGRA_USB_OPMODE_HOST
		[    3.191213] vbus_gpio: -1
		[    3.191219] vbus_gpio_inverted: 0
		[    3.191225] vbus_reg: NULL
		[    3.191231] hot_plug: disabled
		[    3.191237] remote_wakeup: disabled
		[    3.211408] tegra-ehci tegra-ehci.1: Tegra EHCI Host Controller
		[    3.211444] tegra-ehci tegra-ehci.1: new USB bus registered, assigned bus number 2
		[    3.240334] tegra-ehci tegra-ehci.1: irq 53, io mem 0xc5004000
		[    3.260317] tegra-ehci tegra-ehci.1: USB 2.0 started, EHCI 1.00
		[    3.260375] usb usb2: New USB device found, idVendor=1d6b, idProduct=0002
		[    3.260388] usb usb2: New USB device strings: Mfr=3, Product=2, SerialNumber=1
		[    3.260399] usb usb2: Product: Tegra EHCI Host Controller
		[    3.260409] usb usb2: Manufacturer: Linux 3.1.10 ehci_hcd
		[    3.260418] usb usb2: SerialNumber: tegra-ehci.1
		[    3.260792] hub 2-0:1.0: USB hub found
		[    3.260810] hub 2-0:1.0: 1 port detected
		[    3.261076] tegra USB phy - inst[2] platform info:
		[    3.261085] port_otg: no
		[    3.261091] has_hostpc: no
		[    3.261097] phy_interface: USB_PHY_INTF_UTMI
		[    3.261104] op_mode: TEGRA_USB_OPMODE_HOST
		[    3.261111] vbus_gpio: -1
		[    3.261117] vbus_gpio_inverted: 0
		[    3.261123] vbus_reg: NULL
		[    3.261129] hot_plug: enabled
		[    3.261135] remote_wakeup: disabled
		[    3.261252] Modem link platform open
		[    3.263805] Modem link platform on
		[    3.263816] tegra-ehci tegra-ehci.2: Tegra EHCI Host Controller
		[    3.263848] tegra-ehci tegra-ehci.2: new USB bus registered, assigned bus number 3
		[    3.290363] tegra-ehci tegra-ehci.2: irq 129, io mem 0xc5008000
		[    3.310318] tegra-ehci tegra-ehci.2: USB 2.0 started, EHCI 1.00
		[    3.310375] usb usb3: New USB device found, idVendor=1d6b, idProduct=0002
		[    3.310388] usb usb3: New USB device strings: Mfr=3, Product=2, SerialNumber=1
		[    3.310399] usb usb3: Product: Tegra EHCI Host Controller
		[    3.310409] usb usb3: Manufacturer: Linux 3.1.10 ehci_hcd
		[    3.310418] usb usb3: SerialNumber: tegra-ehci.2
		[    3.310792] hub 3-0:1.0: USB hub found
		[    3.310811] hub 3-0:1.0: 1 port detected
		[    3.311236] usbcore: registered new interface driver cdc_acm
		[    3.311246] cdc_acm: USB Abstract Control Model driver for USB modems and ISDN adapters
		[    3.311296] usbcore: registered new interface driver cdc_wdm
		[    3.311305] Initializing USB Mass Storage driver...
		[    3.311406] usbcore: registered new interface driver usb-storage
		[    3.311415] USB Mass Storage support registered.
		[    3.311507] usbcore: registered new interface driver libusual
		[    3.311625] usbcore: registered new interface driver usbserial
		[    3.311666] USB Serial support registered for generic
		[    3.311721] usbcore: registered new interface driver usbserial_generic
		[    3.311731] usbserial: USB Serial Driver core
		[    3.311767] USB Serial support registered for GSM modem (1-port)
		[    3.311913] usbcore: registered new interface driver option
		[    3.311922] option: v0.7.2:USB Driver for GSM modems
		[    3.311963] USB Serial support registered for pl2303
		[    3.312030] usbcore: registered new interface driver pl2303
		[    3.312039] pl2303: Prolific PL2303 USB to serial adaptor driver
		[    3.313554] using rtc device, tps6586x-rtc, for alarms
		[    3.313590] tps6586x-rtc tps6586x-rtc.0: rtc core: registered tps6586x-rtc as rtc0
		[    3.314245] i2c /dev entries driver
		[    3.314749] lirc_dev: IR Remote Control driver registered, major 243
		[    3.314760] IR NEC protocol handler initialized
		[    3.314768] IR RC5(x) protocol handler initialized
		[    3.314776] IR RC6 protocol handler initialized
		[    3.314783] IR JVC protocol handler initialized
		[    3.314791] IR Sony protocol handler initialized
		[    3.314798] IR RC5 (streamzap) protocol handler initialized
		[    3.314807] IR MCE Keyboard/mouse protocol handler initialized
		[    3.314815] IR LIRC bridge handler initialized
		[    3.314822] Linux video capture interface: v2.00
		[    3.314914] usbcore: registered new interface driver uvcvideo
		[    3.314923] USB Video Class driver (1.1.1)
		[    3.315088] trpc_sema_init: registered misc dev 10:57
		[    3.315224] trpc_node_register: Adding 'local' to node list
		[    3.315364] tegra_avp_probe: allocated carveout memory at 1ff00000 for AVP kernel
		[    3.315461] trpc_node_register: Adding 'avp-remote' to node list
		[    3.315684] tegra_avp_probe: message area 12528000/12528110
		[    3.317483] add mma i2c driver
		[    3.317516]
		[    3.317519] Probing Module: mma845x Ver. 1.0
		[    3.317528] Freescale Android 2.3 Release: 10.2
		[    3.317536] Build Date: Apr  4 2014 [14:37:47]
		[    3.317759] IdentifyChipset:: Found MMA8452 chipset with chip ID 0x2a
		[    3.317770] Inialize default orientation values for MMA8452 chip
		[    3.318565] tegra-i2c tegra-i2c.0: I2c error status 0x00000008
		[    3.318577] tegra-i2c tegra-i2c.0: no acknowledge from address 0x1c
		[    3.318587] tegra-i2c tegra-i2c.0: Packet status 0x00010009
		[    3.332848] mma845x_probe:: Registered char dev "mma" @ 242
		[    3.333015] input: mma845x as /devices/virtual/input/input0
		[    3.333325] input: Accl1 as /devices/virtual/input/input1
		[    3.334144] mma845x_probe:: Configuring interrupt IRQ [357]
		[    3.334404] Mode [0x0]
		[    3.335143] Interrupt Mask [0x1]
		[    3.335674] mma845x 0-001c: mma845x device is probed successfully.
		[    3.335715] i2c-core: driver [mma845x] using legacy suspend method
		[    3.335726] i2c-core: driver [mma845x] using legacy resume method
		[    3.335929] tegra_wdt tegra_wdt: tegra_wdt_probe done
		[    3.335984] Bluetooth: HCI UART driver ver 2.2
		[    3.335995] Bluetooth: HCI H4 protocol initialized
		[    3.336003] Bluetooth: HCILL protocol initialized
		[    3.336012] Bluetooth: BlueSleep Mode Driver Ver 1.1
		[    3.336266] cpuidle: using governor ladder
		[    3.336455] cpuidle: using governor menu
		[    3.336490] sdhci: Secure Digital Host Controller Interface driver
		[    3.336499] sdhci: Copyright(c) Pierre Ossman
		[    3.336506] sdhci-pltfm: SDHCI platform and OF driver helper
		[    3.336690] sdhci-tegra sdhci-tegra.1: vddio_sdmmc regulator not found: -19.Assuming vddio_sdmmc is not required.
		[    3.336708] sdhci-tegra sdhci-tegra.1: vddio_sd_slot regulator not found: -19. Assuming vddio_sd_slot is not required.
		[    3.336753] mmc0: Invalid maximum block size, assuming 512 bytes
		[    3.336774] mmc0: no vmmc regulator found
		[    3.336851] Registered led device: mmc0::
		[    3.337005] mmc0: SDHCI controller on sdhci-tegra.1 [sdhci-tegra.1] using ADMA
		[    3.337058] sdhci-tegra sdhci-tegra.3: vddio_sdmmc regulator not found: -19.Assuming vddio_sdmmc is not required.
		[    3.337075] sdhci-tegra sdhci-tegra.3: vddio_sd_slot regulator not found: -19. Assuming vddio_sd_slot is not required.
		[    3.337118] mmc1: Invalid maximum block size, assuming 512 bytes
		[    3.337140] mmc1: no vmmc regulator found
		[    3.337209] Registered led device: mmc1::
		[    3.337303] mmc1: SDHCI controller on sdhci-tegra.3 [sdhci-tegra.3] using ADMA
		[    3.338353] tegra-aes tegra-aes: registered
		[    3.338595] usbcore: registered new interface driver usbhid
		[    3.338605] usbhid: USB HID core driver
		[    3.338976] usbcore: registered new interface driver snd-usb-audio
		[    3.339614] tegra20_ac97_platform_probe()
		[    3.339658] tegra20_ac97_platform_probe() 565
		[    3.339667] tegra20_ac97_platform_probe() 582
		[    3.339857] colibri_t20_wm9715l_driver_probe()
		[    3.340386] tegra20_ac97_probe()
		[    3.340395] ac97->capture_dma_data=d25142ec
		[    3.340403] ac97->playback_dma_data=d2514300
		[    3.340512] WM9711/WM9712 SoC Audio Codec 0.4
		[    3.340523] tegra20_ac97_reset()
		[    3.340538] tegra20_ac97_warm_reset()
		[    3.343947] colibri_t20_wm9715l_init()
		[    3.349038] asoc: wm9712-hifi <-> tegra20-ac97-pcm mapping ok
		[    3.349357] asoc: dit-hifi <-> tegra20-spdif mapping ok
		[    3.352341] wm97xx-ts 0-0:wm9712-codec: detected a wm9712 codec
		[    3.360523] input: wm97xx touchscreen as /devices/platform/colibri_t20-snd-wm9715l.0/0-0:wm9712-codec/input/input2
		[    3.380510] ALSA device list:
		[    3.380519]   #0: colibri_t20-wm9715l
		[    3.380526] oprofile: hardware counters not available
		[    3.380537] oprofile: using timer interrupt.
		[    3.380657] GACT probability NOT on
		[    3.380672] Mirror/redirect action on
		[    3.380681] u32 classifier
		[    3.380686]     Actions configured
		[    3.380695] Netfilter messages via NETLINK v0.30.
		[    3.380761] nf_conntrack version 0.5.0 (7513 buckets, 30052 max)
		[    3.381098] NF_TPROXY: Transparent proxy support initialized, version 4.1.0
		[    3.381109] NF_TPROXY: Copyright (c) 2006-2007 BalaBit IT Ltd.
		[    3.381272] ip_tables: (C) 2000-2006 Netfilter Core Team
		[    3.381416] arp_tables: (C) 2002 David S. Miller
		[    3.381462] TCP cubic registered
		[    3.381469] Initializing XFRM netlink socket
		[    3.382632] NET: Registered protocol family 10
		[    3.385443] Mobile IPv6
		[    3.385481] ip6_tables: (C) 2000-2006 Netfilter Core Team
		[    3.385490] IPv6 over IPv4 tunneling driver
		[    3.386434] NET: Registered protocol family 17
		[    3.386466] NET: Registered protocol family 15
		[    3.386476] can: controller area network core (rev 20090105 abi 8)
		[    3.386548] NET: Registered protocol family 29
		[    3.386636] can: raw protocol (rev 20090105)
		[    3.386646] can: broadcast manager protocol (rev 20090105 t)
		[    3.386778] Bluetooth: RFCOMM TTY layer initialized
		[    3.386797] Bluetooth: RFCOMM socket layer initialized
		[    3.386806] Bluetooth: RFCOMM ver 1.11
		[    3.386815] Bluetooth: BNEP (Ethernet Emulation) ver 1.3
		[    3.386825] Bluetooth: HIDP (Human Interface Emulation) ver 1.2
		[    3.386915] Registering the dns_resolver key type
		[    3.386954] VFP support v0.3: implementor 41 architecture 3 part 30 variant 9 rev 1
		[    3.386988] Registering SWP/SWPB emulation handler
		[    3.393464] Disabling clocks left on by bootloader:
		[    3.393475]    audio_2x
		[    3.393483]    audio
		[    3.393492]    dsi2-fixed
		[    3.393499]    dsi1-fixed
		[    3.393512]    vi
		[    3.393520]    uarta
		[    3.393527]    i2c3-fast
		[    3.393534]    i2c3
		[    3.393543]    bsev
		[    3.393549]    bsea
		[    3.393563]    fuse_burn
		[    3.393575]    clk_d
		[    3.393585]    pll_p_out2
		[    3.393643] CPU rate: 1000 MHz
		[    3.393903] DVFS: Got RTC device name:rtc0
		[    3.402496] Enabling Tegra protected aperture at 0x1e500000
		[    3.403436] registered taskstats version 1
		[    3.404110] regulator_init_complete: REG-LDO_9: incomplete constraints, leaving on
		[    3.405739] tps6586x-rtc tps6586x-rtc.0: setting system clock to 2014-04-11 07:59:15 UTC (1397203155)
		[    3.459173] UBIFS: recovery needed
		[    3.520545] mmc0: host does not support reading read-only switch. assuming write-enable.
		[    3.524914] mmc0: new high speed SDHC card at address aaaa
		[    3.525284] mmcblk0: mmc0:aaaa SL16G 14.8 GiB
		[    3.536584]  mmcblk0: unknown partition table
		[    3.560329] usb 1-1: new high speed USB device number 2 using tegra-ehci
		[    3.585541] mmc1: new high speed SDIO card at address fffd
		[    3.603764] UBIFS: recovery completed
		[    3.603776] UBIFS: mounted UBI device 0, volume 0, name "rootfs"
		[    3.603787] UBIFS: file system size:   699383808 bytes (682992 KiB, 666 MiB, 2754 LEBs)
		[    3.603800] UBIFS: journal size:       9404416 bytes (9184 KiB, 8 MiB, 38 LEBs)
		[    3.603811] UBIFS: media format:       w4/r0 (latest is w4/r0)
		[    3.603820] UBIFS: default compressor: lzo
		[    3.603828] UBIFS: reserved for root:  0 bytes (0 KiB)
		[    3.604903] VFS: Mounted root (ubifs filesystem) on device 0:13.
		[    3.606171] devtmpfs: mounted
		[    3.760711] usb 1-1: New USB device found, idVendor=0424, idProduct=2513
		[    3.760725] usb 1-1: New USB device strings: Mfr=0, Product=0, SerialNumber=0
		[    3.761395] hub 1-1:1.0: USB hub found
		[    3.761464] hub 1-1:1.0: 2 ports detected
		[    3.880351] usb 2-1: new high speed USB device number 2 using tegra-ehci
		[    4.031671] usb 2-1: New USB device found, idVendor=0b95, idProduct=772b
		[    4.031685] usb 2-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
		[    4.031696] usb 2-1: Product: AX88772B
		[    4.031704] usb 2-1: Manufacturer: ASIX Elec. Corp.
		[    4.031713] usb 2-1: SerialNumber: 000001
		>[    4.060499] systemd[1]: systemd 199 running in system mode. (+PAM -LIBWRAP -AUDIT -SELINUX +IMA +SYSVINIT -LIBCRYPTSETUP -GCRYPT +ACL +XZ)
		>[    4.064648] systemd[1]: Set hostname to <mx4-gtt-1000000>.
		>[    4.347427] systemd[1]: Cannot add dependency job for unit display-manager.service, ignoring: Unit display-manager.service failed to load: No such file or directory. See system logs and 'systemctl status display-manager.service' for details.
		>[    4.349627] systemd[1]: Starting Forward Password Requests to Wall Directory Watch.
		>[    4.349908] systemd[1]: Started Forward Password Requests to Wall Directory Watch.
		>[    4.350285] systemd[1]: Expecting device dev-ttyS0.device...
		>[    4.350373] systemd[1]: Starting Syslog Socket.
		>[    4.350842] systemd[1]: Listening on Syslog Socket.
		>[    4.350892] systemd[1]: Starting Remote File Systems.
		>[    4.350948] systemd[1]: Reached target Remote File Systems.
		>[    4.350998] systemd[1]: Starting Delayed Shutdown Socket.
		>[    4.351100] systemd[1]: Listening on Delayed Shutdown Socket.
		>[    4.351148] systemd[1]: Starting /dev/initctl Compatibility Named Pipe.
		>[    4.351250] systemd[1]: Listening on /dev/initctl Compatibility Named Pipe.
		>[    4.351314] systemd[1]: Starting Dispatch Password Requests to Console Directory Watch.
		>[    4.351492] systemd[1]: Started Dispatch Password Requests to Console Directory Watch.
		>[    4.351541] systemd[1]: Starting Paths.
		>[    4.351593] systemd[1]: Reached target Paths.
		>[    4.351845] systemd[1]: Starting udev Kernel Socket.
		>[    4.351963] systemd[1]: Listening on udev Kernel Socket.
		>[    4.352165] systemd[1]: Starting udev Control Socket.
		>[    4.352297] systemd[1]: Listening on udev Control Socket.
		>[    4.352358] systemd[1]: Starting Journal Socket.
		>[    4.352557] systemd[1]: Listening on Journal Socket.
		>[    4.352770] systemd[1]: Starting udev Coldplug all Devices...
		>[    4.356767] systemd[1]: Mounted Huge Pages File System.
		>[    4.361364] systemd[1]: Started Load Kernel Modules.
		>[    4.361468] systemd[1]: Mounting FUSE Control File System...
		>[    4.368517] systemd[1]: Starting Set the USB gadget to RNDIS...
		>[    4.383110] systemd[1]: Starting Apply Kernel Variables...
		>[    4.384643] systemd[1]: Starting udev Kernel Device Manager...
		>[    4.386844] systemd[1]: Starting Journal Service...
		>[    4.391129] systemd[1]: Started Journal Service.
		>[    4.392003] systemd[1]: Set up automount Arbitrary Executable File Formats File System Automount Point.
		>[    4.413602] systemd[1]: Started Set Up Additional Binary Formats.
		>[    4.414702] systemd[1]: Mounted Configuration File System.
		>[    4.414809] systemd[1]: Mounted POSIX Message Queue File System.
		>[    4.414922] systemd[1]: Starting Swap.
		>[    4.414982] systemd[1]: Reached target Swap.
		>[    4.415041] systemd[1]: Mounting Temporary Directory...
		>[    4.425229] systemd[1]: Started File System Check on Root Device.
		>[    4.425368] systemd[1]: Starting Remount Root and Kernel File Systems...
		>[    4.490774] systemd[1]: Started Apply Kernel Variables.
		>[    4.497450] systemd-udevd[91]: starting version 199
		[    4.683789] ASIX USB Ethernet Adapter:v4.4.0 14:40:09 Apr  4 2014
		[    4.683798] <6>    http://www.asix.com.tw
		[    4.683812] eth%d: status ep1in, 8 bytes period 11
		[    4.684415] eth0: register 'asix' at usb-tegra-ehci.1-1, ASIX AX88772B USB 2.0 Ethernet, 00:14:2d:38:bd:64
		[    5.445320] eth0: rxqlen 0 --> 5
		[    5.546767] eth0: ax88772b - Link status is: 0
		[    5.668040] rsi_client: module license 'unspecified' taints kernel.
		[    5.668053] Disabling lock debugging due to kernel taint
		[    5.683994] RSI_Init called and registering the client driver
		[    5.771500] ganges_probe: Initialization function called
		[    5.771516] ganges_netdevice_op: WEXT is enabled
		[    5.774750] ganges_probe: Net device operations suceeded
		[    5.774882] ganges_probe: Enabled the interface
		[    5.774892] ganges_request_interrupt_handler: Unmasking IRQ
		[    5.791945] ganges_probe: Registered Interrupt handler
		[    5.791962] ganges_probe: Initialized Master and Client Operations
		[    5.791971] ganges_probe: Context setting suceeded
		[    5.791993] ganges_probe: gangesclient=d23c0000
		[    5.792006] ganges_probe: Mutex init successfull
		[    5.792048] ganges_probe: Initialized Procfs
		[    5.792056] ganges_probe: Before Read reg params
		[    5.792064] ganges_read_reg_param: WiFi mode on
		[    5.792072] ganges_read_reg_param: BT Coexistence disable
		[    5.792080] ganges_read_reg_param: TCP/UDP checksum offload feature off
		[    5.792089] ganges_read_reg_param: RF LDO is off
		[    5.792098] ganges_read_reg_param: Driver version = 3.2.10.0
		[    5.792108] ganges_read_reg_param: Three byte EEPROM address used
		[    5.792118] +ganges_client_initialize_params:
		[    5.792126] ganges_init_pwr_table: Initializing Max power table
		[    5.792144] ganges_set_reg_domain: Setting 1 reg domain
		[    5.792166] ganges_read_reg_param: Wakeup threshold Tx bytes: 0
		[    5.792175] ganges_read_reg_param: Wakeup threshold Rx bytes: 0
		[    5.792184] ganges_read_reg_param: Traffic monitoring interval: 50
		[    5.792193] ganges_read_reg_param: UAPSD wakup interval: 0
		[    5.792200] ganges_read_reg_param: Fragmentation off
		[    5.792208] ganges_read_reg_param: RTS/CTS off
		[    5.792215] ganges_read_reg_param: High Sens level set
		[    5.792222] -ganges_client_initialize_params:
		[    5.792230] ganges_probe: Read reg params successfull
		[    5.792245] ganges_setupcard: Forcing SDIO clock to 50000KHz
		[    5.792253] ganges_setblocklength: Setting the block length
		[    5.792355] ganges_probe: Setup card succesfully
		[    5.792381] ganges_init_interface: Initializing interface
		[    5.792390] ganges_init_interface: Sending init cmd
		[    5.792398] + ganges_init_sdio_slave_regs
		[    5.792405] init_sdio_slave_regs: Initialzing SDIO read start level
		[    5.792443] init_sdio_slave_regs: Initialzing FIFO ctrl registers
		[    5.792502] - ganges_init_sdio_slave_regs
		[    5.792511] ganges_probe: Initialized SDIO slave regs
		[    5.792520]
		[    5.792525] 01 00 00 00 09 00 2d 00 00 00 00 00 02 00 00 00
		[    5.792559] 50 14 00 00 00 00 00 00 00 00 01 00 00 00 00 00
		[    5.792592] 00 00 00 00 00 00 00 00
		[    5.792611] TA PLL Disabled
		[    5.792617] ganges_load_config_vals: External PA is disabled
		[    5.792625] ganges_load_config_vals: External PA is disabled
		[    5.792634] ganges_load_TA_inst: LOAD TA Instructions
		[    5.792643] sdio_master_access_msword: MASTER_ACCESS_MSBYTE:0x0
		[    5.792682] sdio_master_access_msword:MASTER_ACCESS_LSBYTE:0x22
		[    5.792765] sdio_master_access_msword: MASTER_ACCESS_MSBYTE:0x0
		[    5.792804] sdio_master_access_msword:MASTER_ACCESS_LSBYTE:0x20
		[    5.792889] ganges_load_TA_inst: Loaded TA config params
		[    5.792898]
		[    5.792903] 01 00 00 00 09 00 2d 00 00 00 00 00 02 00 00 00
		[    5.792937] 50 14 00 00 00 00 00 00 00 00 00 00 00 00 00 00
		[    5.792971] 00 00 00 00 00 00
		[    5.792987] sdio_master_access_msword: MASTER_ACCESS_MSBYTE:0x0
		[    5.793026] sdio_master_access_msword:MASTER_ACCESS_LSBYTE:0x0
		[    5.846375] ganges_load_TA_inst: Succesfully loaded taim instructions
		[    5.846411] sdio_master_access_msword: MASTER_ACCESS_MSBYTE:0x0
		[    5.846450] sdio_master_access_msword:MASTER_ACCESS_LSBYTE:0x20
		[    5.851375] ganges_load_TA_inst: Succesfully loaded tadm instructions
		[    5.851399] sdio_master_access_msword: MASTER_ACCESS_MSBYTE:0x0
		[    5.851438] sdio_master_access_msword:MASTER_ACCESS_LSBYTE:0x22
		[    5.851473] ganges_load_TA_inst: Bringing TA Out of Reset
		[    5.851525] sdio_master_access_msword: MASTER_ACCESS_MSBYTE:0x0
		[    5.851561] sdio_master_access_msword:MASTER_ACCESS_LSBYTE:0x22
		[    5.851598] ganges_probe:Downloaded TA instructions
		[    5.851607] ganges_probe: MS word init suceeded
		[    5.851616] ganges_probe:ganges_transmit_thread
		[    5.851674] ganges_probe: Initialized thread & Event
		[    5.852574] ganges_module_init: Successfully registered master driver
		[    5.853375] +ganges_load_LMAC_instructions
		[    5.853387] +ganges_load_LMAC_instructions
		[    5.924810] ganges_load_LMAC_inst: Load LMAC block at 0
		[    5.924906] ganges_load_LMAC_inst: Load LMAC block at 120
		[    5.924991] ganges_load_LMAC_inst: Load LMAC block at 240
		[    5.925066] ganges_load_LMAC_inst: Load LMAC block at 360
		[    5.925140] ganges_load_LMAC_inst: Load LMAC block at 480
		[    5.925216] ganges_load_LMAC_inst: Load LMAC block at 600
		[    5.925293] ganges_load_LMAC_inst: Load LMAC block at 720
		[    5.925379] ganges_load_LMAC_inst: Load LMAC block at 840
		[    5.925468] ganges_load_LMAC_inst: Load LMAC block at 960
		[    5.925556] ganges_load_LMAC_inst: Load LMAC block at 1080
		[    5.925641] ganges_load_LMAC_inst: Load LMAC block at 1200
		[    5.925727] ganges_load_LMAC_inst: Load LMAC block at 1320
		[    5.925808] ganges_load_LMAC_inst: Load LMAC block at 1440
		[    5.925891] ganges_load_LMAC_inst: Load LMAC block at 1560
		[    5.925977] ganges_load_LMAC_inst: Load LMAC block at 1680
		[    5.926056] ganges_load_LMAC_inst: Load LMAC block at 1800
		[    5.926139] ganges_load_LMAC_inst: Load LMAC block at 1920
		[    5.926223] ganges_load_LMAC_inst: Load LMAC block at 2040
		[    5.926320] ganges_load_LMAC_inst: Load LMAC block at 2160
		[    5.926413] ganges_load_LMAC_inst: Load LMAC block at 2280
		[    5.926494] ganges_load_LMAC_inst: Load LMAC block at 2400
		[    5.926575] ganges_load_LMAC_inst: Load LMAC block at 2520
		[    5.926647] ganges_load_LMAC_inst: Load LMAC block at 2640
		[    5.926722] ganges_load_LMAC_inst: Load LMAC block at 2760
		[    5.926804] ganges_load_LMAC_inst: Load LMAC block at 2880
		[    5.926887] ganges_load_LMAC_inst: Load LMAC block at 3000
		[    5.926969] ganges_load_LMAC_inst: Load LMAC block at 3120
		[    5.927048] ganges_load_LMAC_inst: Load LMAC block at 3240
		[    5.927126] ganges_load_LMAC_inst: Load LMAC block at 3360
		[    5.927207] ganges_load_LMAC_inst: Load LMAC block at 3480
		[    5.927288] ganges_load_LMAC_inst: Load LMAC block at 3600
		[    5.927366] ganges_load_LMAC_inst: Load LMAC block at 3720
		[    5.927444] ganges_load_LMAC_inst: Load LMAC block at 3840
		[    5.927526] ganges_load_LMAC_inst: Load LMAC block at 3960
		[    5.927606] ganges_load_LMAC_inst: Load LMAC block at 4080
		[    5.927685] ganges_load_LMAC_inst: Load LMAC block at 4200
		[    5.927773] ganges_load_LMAC_inst: Load LMAC block at 4320
		[    5.927856] ganges_load_LMAC_inst: Load LMAC block at 4440
		[    5.927941] ganges_load_LMAC_inst: Load LMAC block at 4560
		[    5.928024] ganges_load_LMAC_inst: Load LMAC block at 4680
		[    5.928111] ganges_load_LMAC_inst: Load LMAC block at 4800
		[    5.928195] ganges_load_LMAC_inst: Load LMAC block at 4920
		[    5.928278] ganges_load_LMAC_inst: Load LMAC block at 5040
		[    5.928359] ganges_load_LMAC_inst: Load LMAC block at 5160
		[    5.928441] ganges_load_LMAC_inst: Load LMAC block at 5280
		[    5.928546] ganges_load_LMAC_inst: Load LMAC block at 5400
		[    5.928627] ganges_load_LMAC_inst: Load LMAC block at 5520
		[    5.928720] ganges_load_LMAC_inst: Load LMAC block at 5640
		[    5.928794] ganges_load_LMAC_inst: Load LMAC block at 5760
		[    5.928869] ganges_load_LMAC_inst: Load LMAC block at 5880
		[    5.928946] ganges_load_LMAC_inst: Load LMAC block at 6000
		[    5.929025] ganges_load_LMAC_inst: Load LMAC block at 6120
		[    5.929127] ganges_load_LMAC_inst: Load LMAC block at 6240
		[    5.929206] ganges_load_LMAC_inst: Load LMAC block at 6360
		[    5.929285] ganges_load_LMAC_inst: Load LMAC block at 6480
		[    5.929364] ganges_load_LMAC_inst: Load LMAC block at 6600
		[    5.929445] ganges_load_LMAC_inst: Load LMAC block at 6720
		[    5.929523] ganges_load_LMAC_inst: Load LMAC block at 6840
		[    5.929619] ganges_load_LMAC_inst: Load LMAC block at 6960
		[    5.929693] ganges_load_LMAC_inst: Load LMAC block at 7080
		[    5.929772] ganges_load_LMAC_inst: Load LMAC block at 7200
		[    5.929852] ganges_load_LMAC_inst: Load LMAC block at 7320
		[    5.929931] ganges_load_LMAC_inst: Load LMAC block at 7440
		[    5.930020] ganges_load_LMAC_inst: Load LMAC block at 7560
		[    5.930103] ganges_load_LMAC_inst: Load LMAC block at 7680
		[    5.930182] ganges_load_LMAC_inst: Load LMAC block at 7800
		[    5.930266] ganges_load_LMAC_inst: Load LMAC block at 7920
		[    5.930467] ganges_load_LMAC_inst: Load LMAC block at 8040
		[    5.930548] ganges_load_LMAC_inst: Load LMAC block at 8160
		[    5.930626] ganges_load_LMAC_inst: Load LMAC block at 8280
		[    5.930739] ganges_load_LMAC_inst: Load LMAC block at 8400
		[    5.930827] ganges_load_LMAC_inst: Load LMAC block at 8520
		[    5.930903] ganges_load_LMAC_inst: Load LMAC block at 8640
		[    5.930980] ganges_load_LMAC_inst: Load LMAC block at 8760
		[    5.931064] ganges_load_LMAC_inst: Load LMAC block at 8880
		[    5.931149] ganges_load_LMAC_inst: Load LMAC block at 9000
		[    5.931227] ganges_load_LMAC_inst: Load LMAC block at 9120
		[    5.931314] ganges_load_LMAC_inst: Load LMAC block at 9240
		[    5.931398] ganges_load_LMAC_inst: Load LMAC block at 9360
		[    5.931477] ganges_load_LMAC_inst: Load LMAC block at 9480
		[    5.931557] ganges_load_LMAC_inst: Load LMAC block at 9600
		[    5.931638] ganges_load_LMAC_inst: Load LMAC block at 9720
		[    5.931718] ganges_load_LMAC_inst: Load LMAC block at 9840
		[    5.931795] ganges_load_LMAC_inst: Load LMAC block at 9960
		[    5.931872] ganges_load_LMAC_inst: Load LMAC block at 10080
		[    5.931951] ganges_load_LMAC_instructions: Last frame
		[    5.931961] ganges_load_LMAC_inst: Load LMAC block at 10200
		[    5.962506] ganges_read_rf_type: EEPROM read Enabled for RF Type read
		[    5.962847] ganges_read_power_vals: EEPROM read Enabled for Power vals Read
		[    5.964907] ganges_read_max_tx_pwr: Reading Max Tx power values
		[    5.965376] ganges_init_pwr_table: Initializing Max power table
		[    5.965717] ganges_mgmt_init: Setting Protocol Type to default N
		[    5.965734] ganges_create_bssid: Creating BSSID
		[    5.965763] ganges_mgmt_init: Loading static programming
		[    5.965780] ganges_init_BB_RF: Sending BB Prog Vals
		[    5.965873] sendRFProgRequest: Sending RFProgRequest 56
		[    5.965950] ganges_init_BB_RF: Sending BB Prog Vals
		[    5.966053] sendRFProgRequest: Sending RFProgRequest 20
		[    5.966181] ganges_init_BB_RF: Sending BB Prog Vals
		[    5.968744] ganges_handle_powersave: Reached FSM_NO_STATE
		[    6.006270] loading spi pic io driver
		[    6.006370] probing spi pic io driver
		[    6.029850] EXT4-fs (mmcblk0): recovery complete
		[    6.029870] EXT4-fs (mmcblk0): mounted filesystem with ordered data mode. Opts: (null)


## Default Login

		.---O---.
		|       |                  .-.           o o
		|   |   |-----.-----.-----.| |   .----..-----.-----.
		|       |     | __  |  ---'| '--.|  .-'|     |     |
		|   |   |  |  |     |---  ||  --'|  |  |  '  | | | |
		'---'---'--'--'--.  |-----''----''--'  '-----'-'-'-'
		                -'  |
		                '---'

		The Angstrom Distribution mx4-gtt-1000000 ttyS0

		Angstrom v2013.06 - Kernel

		mx4-gtt-1000000 login:

Default username: `root` password: `none`

## Reset Cause

One can read out the system reset cause from a spi attribute.

```bash
root@ultra14211046:~# cat /opt/hm/pic_attributes/ctrl_pic_reset_cause
1
```

```c
#define REG_BIT(bit)                    (1 << (bit))

#define PRC_POWER_ON_RESET              REG_BIT(0)
#define PRC_BROWN_OUT_RESET             REG_BIT(1)
#define PRC_IDLE                        REG_BIT(2)
#define PRC_SLEEP                       REG_BIT(3)
#define PRC_WDTO                        REG_BIT(4)
#define PRC_SWR                         REG_BIT(5)
#define PRC_MCLR                        REG_BIT(6)
#define PRC_CONFIG_MISMATCH             REG_BIT(7)
#define PRC_DEEP_SLEEP                  REG_BIT(8)
#define PRC_ILLEGAL_OPCODE_RESET        REG_BIT(9)
#define PRC_TRAP_CONFLICT_RESET         REG_BIT(10)
```

## Timekeeping

There are two clocks that are battery backed up in MX-4.
- Main CPU internal real time clock
- GPS time in modem

### Internal RTC
The main CPU internal real time clock is `/dev/rtc0` and the time can be read
out with the `hwclock` command.

```bash
# Show hardware date/time
root@mx4-vcc-1000000:~# hwclock
Mon Nov  3 13:25:07 2014  0.000000 seconds
```

```bash
#Set hardware date/time to system date/time:
root@mx4-vcc-1000000:~# hwclock -w
```

```bash
#Set system date/time to hardware date/time:
root@mx4-vcc-1000000:~# hwclock -s
```

The Linux system time is synchronized with the main CPU internal real time clock at boot time.
Linux system time can be read out with the 'date' command.

```bash
root@mx4-vcc-1000000:~# date
Mon Nov  3 13:25:25 UTC 2014
```

On our default images we try to synchronize time from `pool.ntp.org` on an ifup
networking event. If we succeed with the `ntpdate` command we synchronize the
system time to hardware clock (main CPU internal real time clock).

See `/etc/network/if-up.d/ntpdate-sync` and `/etc/default/ntpdate` on target system.

### GPS RTC

The GPS time can be parsed from the $GPRMC string which is outputted by the GPS.

This time is backed up because we want faster GPS fix times, otherwise it would have
to restart from fresh each time GPS engine is restarted.

This backed up time could also work as a fall back to an application if system time
for some reason never can be synchronized from the Internet.

We do not do anything with this in our default platform.

## Communication Interfaces

### Serial Devices

The MX-4 contains three serial ports which can be used to connect to other units. Full RS-232 with CTS/RTS, RS-232 RX/TX and RS-485.

The interal linux map to external connector:

    /dev/ttyHS1 ---> RS485
    /dev/ttyHS3 ---> RS232/CTS/RTS (RS232-1)
    /dev/ttyS1  ---> RS232 TX/RX (RS232-2)

### CAN

All MX-4 boards are equipped with CAN controllers. We are using the socketCAN API to communicate with our hardware. See [socketCAN wiki](http://en.wikipedia.org/wiki/SocketCAN)

#### Configuring CAN bus

Setting CAN speed to 250 kbit/s and listen only mode. `iproute2` is a requirement for these commands to work.
```bash
ip link set can0 type can bitrate 250000 listen-only on
ifconfig can0 up
```

#### CAN tools

There is a set off tools developed to be used with the socketCAN API. Package name is can-utils. It should be installed on target but if it is not run the following command.

```bash
opkg update
opkg install can-utils
```

Two great and easy to use tools which are bundled in the can-utils packare are `candump` and `cansend`.

#### Cansend example
Extended frame identifier 0x1234 with dlc of eight

```bash
cansend can0 -e -i 0x1234 0x01 0x02 0x03 0x04 0x05 0x06 0x07 0x08
```

#### Statistics
The linux kernel collects statistics on CAN networks, similar to TCP/IP. These can be read out in different ways.

With `ifconfig` tool.

```bash
root@mx4-gtt:~# ifconfig can0
      can0      Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00
      UP RUNNING NOARP  MTU:16  Metric:1
      RX packets:2 errors:0 dropped:0 overruns:0 frame:0
      TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
      collisions:0 txqueuelen:10
      RX bytes:16 (16.0 B)  TX bytes:0 (0.0 B)
      Interrupt:205
```

With `iproute2` tool.

```bash
root@mx4-gtt:~/test/can_test# ip -d -s link show can0
2: can0: <NOARP,UP,LOWER_UP,ECHO> mtu 16 qdisc pfifo_fast state UNKNOWN mode DEFAULT qlen 10
   link/can
   can state ERROR-PASSIVE (berr-counter tx 128 rx 0) restart-ms 0
   bitrate 25000 sample-point 0.875
   tq 2500 prop-seg 6 phase-seg1 7 phase-seg2 2 sjw 1
   sja1000: tseg1 1..16 tseg2 1..8 sjw 1..4 brp 1..64 brp-inc 1
   clock 12000000
   re-started bus-errors arbit-lost error-warn error-pass bus-off
   0          0          0          1          1          0
   RX: bytes  packets  errors  dropped overrun mcast
   16         2        0       0       0       0
   TX: bytes  packets  errors  dropped carrier collsns
   0          0        0       0       0       0
```

### Wifi

#### USAGE

All that is required to start Wifi moudule is `ifup wlan0`.

```
#/etc/networking/interfaces
auto wlan0
iface wlan0 inet dhcp
    wpa-driver nl80211
    wpa-conf /etc/wpa_supplicant.conf
```

One has to edit `/etc/wpa_supplicant.conf` to add new networks which the module should connect to.
```
#/etc/wpa_supplicant.conf
ctrl_interface=/var/run/wpa_supplicant
ctrl_interface_group=0
update_config=1

network={
        ssid="HostMobility-Guest"
        psk="*************"
        scan_ssid=1
}

```

See more configuration examples at http://linux.die.net/man/5/wpa_supplicant.conf

#### Wifi as Access Point

##### Linux kernel
Edit kernel to include following drivers. Recompile and deploy to target.

`CONFIG_ATH_COMMON=y`<br>
`CONFIG_ATH9K_HW=y`<br>
`CONFIG_ATH9K_HTC=y`<br>

##### Target setup
[Download firmware](http://wireless.kernel.org/download/htc_fw/1.3/htc_9271.fw) and place it in /lib/firmware

Run `opkg install hostap-daemon`.

Run `hostapd /etc/hostapd.conf` to start the wifi access point daemon.

Run `udhcpd  -S /etc/wifi-ap-dhcp.conf` to start dhcp server.

Script to route internet connection (OBS! You need to enable forwarding in kernel):

```bash
#!/bin/sh
IPT=/sbin/iptables
LOCAL_IFACE=wlan1
INET_IFACE=eth0
INET_ADDRESS=192.168.0.59
# Flush the tables
$IPT -F
$IPT -t nat -F

$IPT -I INPUT -i $LOCAL_IFACE -j ACCEPT
# This tells iptables to accept all traffic coming from all devices accept ppp0 (internet).
# This rule is added as the first rule in the INPUT chain.
# Notice that this rule doesn't actually deny traffic from ppp0.
$IPT -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
# This is the line for the stateful firewall.
# It means that it will only accept connections from nodes that you already connected to.
$IPT -A INPUT -p tcp -i $INET_IFACE -j REJECT --reject-with tcp-reset
$IPT -A INPUT -p udp -i $INET_IFACE -j REJECT --reject-with icmp-port
# Those 2 lines aren't actually necessary. We reject unwanted connection-attempts here instead
#of dropping them by the default INPUT policy. This so others won't know you have a firewall.

#Now we add the rules for NAT:
$IPT -I FORWARD -i $LOCAL_IFACE -d 192.168.1.0/255.255.255.0 -j DROP
$IPT -A FORWARD -i $LOCAL_IFACE -s 192.168.1.0/255.255.255.0 -j ACCEPT
$IPT -A FORWARD -i $INET_IFACE -d 192.168.1.0/255.255.255.0 -j ACCEPT
$IPT -t nat -A POSTROUTING -o $INET_IFACE -j MASQUERADE
#We set the policies for the chains:
$IPT -P INPUT DROP
# We want to drop any packets that don't match our rules don't we?
$IPT -P FORWARD DROP
# Same here..
$IPT -P OUTPUT ACCEPT
# We usually allow everything out and only block specific things we don't want.
```

[Example hostapd config file](http://hostmobility.org/file_host/wifi/hostapd.conf)

[Example udhcpd config file](http://hostmobility.org/file_host/wifi/wifi-ap-dhcp.conf)


## Modem

The MX-4 board is equipped with a 3G modem. It is not started by default when the unit is powered one but has to be explicitly turned on with the following command:

     root@mx4-gtt:~# echo 1 > /opt/hm/pic_attributes/ctrl_modem_on

if PH8 is powered on, following ports will be created
* /dev/ttyUSB0 --> Reserved port (not usable)
* /dev/ttyUSB1 --> NMEA port
* /dev/ttyUSB2 --> Application port
* /dev/ttyUSB3 --> Modem port
* /dev/ttyUSB4 --> WWAN port (not usable, Windows only)

You should get an output similar to the following:

```bash
 root@mx4-gtt:~# dmesg | tail
 [ 5034.642693] option 3-1:1.0: GSM modem (1-port) converter detected
 [ 5034.644177] usb 3-1: GSM modem (1-port) converter now attached to ttyUSB0
 [ 5034.646047] option 3-1:1.1: GSM modem (1-port) converter detected
 [ 5034.647351] usb 3-1: GSM modem (1-port) converter now attached to ttyUSB1
 [ 5034.649150] option 3-1:1.2: GSM modem (1-port) converter detected
 [ 5034.650337] usb 3-1: GSM modem (1-port) converter now attached to ttyUSB2
 [ 5034.652141] option 3-1:1.3: GSM modem (1-port) converter detected
 [ 5034.653759] usb 3-1: GSM modem (1-port) converter now attached to ttyUSB3
 [ 5034.655688] option 3-1:1.4: GSM modem (1-port) converter detected
 [ 5034.657306] usb 3-1: GSM modem (1-port) converter now attached to ttyUSB4
```

The modem can also be turned off with this command.

```bash
root@mx4-gtt:~# echo 0 > /opt/hm/pic_attributes/ctrl_modem_on
```

### Creating a PPP connection
First off we need to install `pppd` if it does not exist on target.

```bash
opkg update
opkg install ppp
```

File structure needed for ppp to work:
```
/etc/ppp
├── chap-secrets
├── ip-down.d
│   └── # Custom down script
├── ip-up.d
│   ├── # Custom up script
├── operators
│   └── apn
├── pap-secrets
├── peers
│   └── provider
├── pppd_connect-chat
└── pppd_disconnect-chat
```

Contest of above files:
/etc/ppp/operators/apn
```
AT+CGDCONT=1,"IP","openroamer.com","",0,0
```
Replace "openroamer.com" with your operator APN

/etc/ppp/peers/provider
```
# File: /etc/ppp/options
# We set defualt remote ip adress.
#:192.168.255.1
#Some GPRS phones don't reply to LCP echo's
lcp-echo-failure 0
lcp-echo-interval 0
# Keep pppd attached to the terminal:
# Comment this to get daemon mode pppd
#nodetach
# Debug info from pppd:
# Comment this off, if you don't need more info
debug
persist
holdoff 5
maxfail 0
# Show password in debug messages
show-password
# Connect script:
# scripts to initialize the GPRS modem and start the connection,
#connect /etc/pppd_connect-chat
connect '/usr/sbin/chat -v -f /etc/ppp/pppd_connect-chat'
# Disconnect script:
# AT commands used to 'hangup' the GPRS connection.
#disconnect /etc/pppd_disconnect-chat
disconnect '/usr/sbin/chat -v -f /etc/ppp/pppd_disconnect-chat'
# Serial device to which the GPRS phone is connected:
/dev/ttyUSB3
# Serial port line speed
115200 # fast enough
# Hardware flow control:
# Use hardware flow control with cable, Bluetooth and USB but not with IrDA.
crtscts # serial cable, Bluetooth and USB, on some occations with IrDA too
# Ignore carrier detect signal from the modem:
local
# IP addresses:
# - accept peers idea of our local address and set address peer as 10.0.0.1
# (any address would do, since IPCP gives 0.0.0.0 to it)
# - if you use the 10. network at home or something and pppd rejects it,
# change the address to something else
#0.0.0.0:0.0.0.0
# pppd must not propose any IP address to the peer!
noipdefault
# Accept peers idea of our local address
ipcp-accept-local
ipcp-accept-remote
# Add the ppp interface as default route to the IP routing table
defaultroute
# DNS servers from the phone:
# some phones support this, some don't.
usepeerdns
# ppp compression:
# ppp compression may be used between the phone and the pppd, but the
# serial connection is usually not the bottleneck in GPRS, so the
# compression is useless (and with some phones need to disabled before
# the LCP negotiations succeed).
novj
nobsdcomp
novjccomp
nopcomp
noaccomp

# The phone is not required to authenticate:
noauth

# Username and password:
# If username and password are required by the APN, put here the username
# and put the username-password combination to the secrets file:
# /etc/ppp/pap-secrets for PAP and /etc/ppp/chap-secrets for CHAP
# authentication. See pppd man pages for details.
#user mirza
#password 123456

mru 1500
mtu 1400
```
/etc/ppp/pppd_connect-chat
```
#!/bin/sh
#
# File: /etc/ppp/pppd_connect-chat
 TIMEOUT 5
 ECHO ON
 ABORT '\nBUSY\r'
 ABORT '\nERROR\r'
 ABORT '\nNO ANSWER\r'
 ABORT '\nNO CARRIER\r'
 ABORT '\nNO DIALTONE\r'
 ABORT '\nRINGING\r\n\r\nRINGING\r'
 '' \rATZ
 TIMEOUT 12
 SAY "Press CTRL-C to close the connection at any stage!"
 SAY "\ndefining PDP context...\n"
 OK ATH
 OK ATE1
 OK-AT-OK @/etc/ppp/operators/apn
 OK ATD*99#
 TIMEOUT 22
 SAY "\nwaiting for connect...\n"
 CONNECT ""
 SAY "\nConnected."
 SAY "\nIf the following ppp negotiations fail,\n"
 SAY "try restarting the phone.\n"
```

/etc/ppp/pppd_disconnect-chat
```bash
at
ABORT "BUSY"
ABORT "ERROR"
ABORT "NO DIALTONE"
SAY "n\Sending break to the modem\n"
"" "\K"
"" "\K"
"" "\K"
"" "+++ATH"
"" "+++ATH"
"" "+++ATH"
SAY "\nModem context detached\n"
```

`/etc/ppp/pap-secrets` and `/etc/ppp/chap-secrets` are empty and do not really need to exist unless you need to use one of these methods to auth with network provider.

When you have the above structure correctly setup you can then start and stop pppd connections with `pon` and `poff` commands.

One tip is to start up syslogd and run `tail -f /var/log/messages &`, this way you will get some output during the connection process and makes it easier to debug in case of connection errors.

Here is an example.

```bash
root@mx4-gtt:~# syslogd
root@mx4-gtt:~# tail -f /var/log/messages &
root@mx4-gtt:/etc/ppp/operators# pon
root@mx4-gtt:/etc/ppp/operators# Dec 20 10:35:27 mx4-gtt daemon.notice pppd[564]: pppd
2.4.5 started by root, uid 0
Dec 20 10:35:27 mx4-gtt local2.info chat[565]: timeout set to 5 seconds
Dec 20 10:35:27 mx4-gtt local2.info chat[565]: abort on (\nBUSY\r)
Dec 20 10:35:27 mx4-gtt local2.info chat[565]: abort on (\nERROR\r)
Dec 20 10:35:27 mx4-gtt local2.info chat[565]: abort on (\nNO ANSWER\r)
Dec 20 10:35:27 mx4-gtt local2.info chat[565]: abort on (\nNO CARRIER\r)
Dec 20 10:35:27 mx4-gtt local2.info chat[565]: abort on (\nNO DIALTONE\r)
Dec 20 10:35:27 mx4-gtt local2.info chat[565]: abort on (\nRINGING\r\n\r\nRINGING\r)
Dec 20 10:35:27 mx4-gtt local2.info chat[565]: send (^MATZ^M)
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: timeout set to 12 seconds
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: expect (OK)
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: ATZ^M^M
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: OK
Dec 20 10:35:28 mx4-gtt local2.info chat[565]:  -- got it
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: send (ATH^M)
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: expect (OK)
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: ^M
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: ATH^M^M
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: OK
Dec 20 10:35:28 mx4-gtt local2.info chat[565]:  -- got it
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: send (ATE1^M)
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: expect (OK)
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: ^M
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: ATE1^M^M
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: OK
Dec 20 10:35:28 mx4-gtt local2.info chat[565]:  -- got it
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: send (AT+CGDCONT=1,"IP","internet.tele2.
se","",0,0^M)
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: expect (OK)
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: ^M
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: AT+CGDCONT=1,"IP","internet.tele2.se",""
,0,0^M^M
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: OK
Dec 20 10:35:28 mx4-gtt local2.info chat[565]:  -- got it
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: send (ATD*99#^M)
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: timeout set to 22 seconds
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: expect (CONNECT)
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: ^M
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: ATD*99#^M^M
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: CONNECT
Dec 20 10:35:28 mx4-gtt local2.info chat[565]:  -- got it
Dec 20 10:35:28 mx4-gtt local2.info chat[565]: send (^M)
Dec 20 10:35:28 mx4-gtt daemon.debug pppd[564]: Script /usr/sbin/chat -v -f /etc/ppp/pp
pd_connect-chat finished (pid 565), status = 0x0
Dec 20 10:35:28 mx4-gtt daemon.info pppd[564]: Serial connection established.
Dec 20 10:35:28 mx4-gtt daemon.debug pppd[564]: using channel 2
Dec 20 10:35:28 mx4-gtt daemon.info pppd[564]: Using interface ppp0
Dec 20 10:35:28 mx4-gtt daemon.notice pppd[564]: Connect: ppp0 <--> /dev/ttyUSB3
Dec 20 10:35:29 mx4-gtt daemon.debug pppd[564]: sent [LCP ConfReq id=0x1 <asyncmap 0x0>
 <magic 0x6d4845a0>]
Dec 20 10:35:29 mx4-gtt daemon.debug pppd[564]: rcvd [LCP ConfReq id=0x0 <asyncmap 0x0>
 <auth chap MD5> <magic 0xc558301e> <pcomp> <accomp>]
Dec 20 10:35:29 mx4-gtt daemon.debug pppd[564]: No auth is possible
Dec 20 10:35:29 mx4-gtt daemon.debug pppd[564]: sent [LCP ConfRej id=0x0 <auth chap MD5
> <pcomp> <accomp>]
Dec 20 10:35:29 mx4-gtt daemon.debug pppd[564]: rcvd [LCP ConfAck id=0x1 <asyncmap 0x0>
 <magic 0x6d4845a0>]
Dec 20 10:35:29 mx4-gtt daemon.debug pppd[564]: rcvd [LCP ConfReq id=0x1 <asyncmap 0x0>
 <magic 0xc558301e>]
Dec 20 10:35:29 mx4-gtt daemon.debug pppd[564]: sent [LCP ConfAck id=0x1 <asyncmap 0x0>
 <magic 0xc558301e>]
Dec 20 10:35:29 mx4-gtt daemon.warn pppd[564]: kernel does not support PPP filtering
Dec 20 10:35:29 mx4-gtt daemon.debug pppd[564]: sent [CCP ConfReq id=0x1 <deflate 15> <deflate(old#) 15>]
Dec 20 10:35:29 mx4-gtt daemon.debug pppd[564]: sent [IPCP ConfReq id=0x1 <addr 0.0.0.0
> <ms-dns1 0.0.0.0> <ms-dns2 0.0.0.0>]
Dec 20 10:35:29 mx4-gtt daemon.debug pppd[564]: rcvd [LCP DiscReq id=0x2 magic=0xc558301e]
Dec 20 10:35:29 mx4-gtt daemon.debug pppd[564]: rcvd [LCP ProtRej id=0x3 80 fd 01 01 00
 0c 1a 04 78 00 18 04 78 00]
Dec 20 10:35:29 mx4-gtt daemon.debug pppd[564]: Protocol-Reject for 'Compression Contro
l Protocol' (0x80fd) received
Dec 20 10:35:30 mx4-gtt daemon.debug pppd[564]: rcvd [IPCP ConfNak id=0x1]
Dec 20 10:35:30 mx4-gtt daemon.debug pppd[564]: sent [IPCP ConfReq id=0x2 <addr 0.0.0.0> <ms-dns1 0.0.0.0> <ms-dns2 0.0.0.0>]
Dec 20 10:35:31 mx4-gtt daemon.debug pppd[564]: rcvd [IPCP ConfNak id=0x2]
Dec 20 10:35:31 mx4-gtt daemon.debug pppd[564]: sent [IPCP ConfReq id=0x3 <addr 0.0.0.0> <ms-dns1 0.0.0.0> <ms-dns2 0.0.0.0>]
Dec 20 10:35:32 mx4-gtt daemon.debug pppd[564]: rcvd [IPCP ConfNak id=0x3]
Dec 20 10:35:32 mx4-gtt daemon.debug pppd[564]: sent [IPCP ConfReq id=0x4 <addr 0.0.0.0> <ms-dns1 0.0.0.0> <ms-dns2 0.0.0.0>]
Dec 20 10:35:33 mx4-gtt daemon.debug pppd[564]: rcvd [IPCP ConfNak id=0x4]
Dec 20 10:35:33 mx4-gtt daemon.debug pppd[564]: sent [IPCP ConfReq id=0x5 <addr 0.0.0.0> <ms-dns1 0.0.0.0> <ms-dns2 0.0.0.0>]
Dec 20 10:35:34 mx4-gtt daemon.debug pppd[564]: rcvd [IPCP ConfReq id=0x0]
Dec 20 10:35:34 mx4-gtt daemon.debug pppd[564]: sent [IPCP ConfNak id=0x0 <addr 0.0.0.0>]
Dec 20 10:35:34 mx4-gtt daemon.debug pppd[564]: rcvd [IPCP ConfNak id=0x5 <addr 10.49.1
19.16> <ms-dns1 130.244.127.161> <ms-dns2 130.244.127.169>]
Dec 20 10:35:34 mx4-gtt daemon.debug pppd[564]: sent [IPCP ConfReq id=0x6 <addr 10.49.1
19.16> <ms-dns1 130.244.127.161> <ms-dns2 130.244.127.169>]
Dec 20 10:35:34 mx4-gtt daemon.debug pppd[564]: rcvd [IPCP ConfReq id=0x1]
Dec 20 10:35:34 mx4-gtt daemon.debug pppd[564]: sent [IPCP ConfAck id=0x1]
Dec 20 10:35:34 mx4-gtt daemon.debug pppd[564]: rcvd [IPCP ConfAck id=0x6 <addr 10.49.1
19.16> <ms-dns1 130.244.127.161> <ms-dns2 130.244.127.169>]
Dec 20 10:35:34 mx4-gtt daemon.warn pppd[564]: Could not determine remote IP address: d
efaulting to 10.64.64.64
Dec 20 10:35:34 mx4-gtt daemon.notice pppd[564]: local  IP address 10.49.119.16
Dec 20 10:35:34 mx4-gtt daemon.notice pppd[564]: remote IP address 10.64.64.64
Dec 20 10:35:34 mx4-gtt daemon.notice pppd[564]: primary   DNS address 130.244.127.161
Dec 20 10:35:34 mx4-gtt daemon.notice pppd[564]: secondary DNS address 130.244.127.169
Dec 20 10:35:34 mx4-gtt daemon.debug pppd[564]: Script /etc/ppp/ip-up started (pid 569)

root@mx4-gtt:/etc/ppp/operators# Dec 20 10:35:43 mx4-gtt daemon.err ntpdate[576]: no se
rver suitable for synchronization found
Dec 20 10:35:43 mx4-gtt daemon.debug pppd[564]: Script /etc/ppp/ip-up finished (pid 569
), status = 0x1
```

You can now also see with `ifconfig` that you got a ppp connection

```bash
root@mx4-gtt:~# ifconfig ppp0
ppp0      Link encap:Point-to-Point Protocol
          inet addr:10.49.119.16  P-t-P:10.64.64.64  Mask:255.255.255.255
          UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1400  Metric:1
          RX packets:10 errors:0 dropped:0 overruns:0 frame:0
          TX packets:27 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:3
          RX bytes:662 (662.0 B)  TX bytes:1490 (1.4 KiB)
```

### GPS
The GPS chip is integrated in the modem module. Default it is turned off. To turn on the GPS chip send following AT command to the application port. This is not a persistent setting so one has to enable GPS each time modem restarts if GPS data is desired.

```bash
root@mx4-gtt:~# printf 'at^sgpsc="Engine","1"\r' > /dev/ttyUSB2
```

The NMEA output can be read out of /dev/ttyUSB1

```bash
root@mx4-gtt:~# cat /dev/ttyUSB1
$GPGSV,4,1,16,24,,,,05,,,,18,,,,22,,,*71
$GPGSV,4,2,16,04,,,,11,,,,17,,,,03,,,*79
$GPGSV,4,3,16,12,,,,30,,,,01,,,,23,,,*79
$GPGSV,4,4,16,15,,,,27,,,,07,,,,31,,,*7A
$GPGGA,,,,,,0,,,,,,,,*66
$GPVTG,,T,,M,,N,,K,N*2C
$GPRMC,,V,,,,,,,,,,N*53
$GPGSA,A,1,,,,,,,,,,,,,,,*1E
```

### Watchdog

The co-processor acts as a watchdog of the modem. It monitors the power indicator signal by default. If it should detect that the modem is for some reason turned off even though it should be on it will attempt to restart it at first via a soft reset signal and after a few attempts it will do a hard-reset of modem.

To enable monitoring of SLED signal which is an output from modem which reports current status one has to enable this signal. This signal should always toggle on/off, depending on state it toggles in different frequency. Should the co-cpu detect that it stops toggling which is an ERROR state it will attempt to soft restart modem first and if that fails a hard-reset.

To start monitoring of SLED signal input following command to modem. This is not a persistent setting so one has to do this each time modem starts up.
```bash
root@mx4-gtt:~# printf "AT^SLED=2\r" > /dev/ttyUSB3
```

## J1708

Work in progress...

## LIN

How to control LIN via serial interface:<br>
The LIN interface in the PIC is accessed via the serial console /dev/ttyHS2 for LIN1
with baudrate 115200. LIN2 is accessed via ttyHS0. Some hardwares (MX-4 T20/VF61) access LIN via ttyHS3 and ttyHS0.
On Vybrid LIN is accessed via ttyLP2 and LIN2 via ttyLP1. Note that on ttyLP1 CRTSCTS must NOT be enabled or the communication will lock.
The LIN interface is controled via a set of predefined
frames, mostly used to alter the LIN schedule table.
<br>
All frames sent to PIC is on the format:<br>
```
start of transmission | message length | message type | data | checksum<br>
```
start of transmission (4 bytes):<br>
This is a byte sequence used to indicate the start of a frame. <br>
It consists of the 4 byte long sequence: ```0x7e 0x7e 0x7e 0xa7```<br>
<br>
message length (1 byte):<br>
The length of message type (1 byte) + data (variable) + checksum (1 byte)<br>
The byte for message length is not included in message length.<br>
<br>
message type (1 byte): <br>
Which type of operation that is requested.<br>
Values start at 1 and is sequentually increased.<br>
1. [Received response](#received-response). Message contains data received from LIN slave unit. Direction from PIC to<br>
Linux.<br>
2. [Set schedule](#set-schedule). Setup the LIN schedule.<br>
3. [Set frame](#set-frame). Setup a LIN frame.<br>
4. [Set item](#set-item). Setup a LIN schedule item.<br>
5. [Set baudrate](#set-baudrate). Setup LIN baudrate.<br>
6. [Set master](#set-master). Setup master/listen mode.<br>
7. [Listen message](#listen-message)
8-11. [Overflow messages](#overflow-messages)
<br>
Defined in lin_general.h as:<br>
```c
#define MSG_RECV_RESPONSE 1
#define MSG_SET_SCHEDULE 2
#define MSG_SET_FRAME 3
#define MSG_SET_ITEM 4
#define MSG_SET_BAUDRATE 5
#define MSG_SET_MASTER 6
#define MSG_LISTEN_MSG 7
```
<br>
data (variable):<br>
Bytes of data that each message type may contain.<br>
<br>
checksum:<br>
A simple XOR checksum of all bytes in message including message length.<br>
<br>
#### Message types
##### Received response
    If a frame that requests a response from a slave unit is configured to
    forward the response the response is forwarded from PIC to Linux in a
    frame formatted:
    message start | length of message | message type 1 | data (revc status 1
    byte, frame id 1 byte, response length 1 byte, response data) | checksum

##### Set schedule
    This frame sets the start, end and current item to handle and
    enables/disables the schedule. Start, end and current can be any value
    0-255. Start cannot be past end. Current must be inside start and end.
    Enabled i any value > 0.
    Format:
    message start | length of message | message type 2 | data (start slot 1
    byte, end slot 1 byte, current slot 1 byte, enabled 1 byte) | checksum

##### Set frame
    This frame type is used to setup a specific LIN frame.
    Format:
    message start | length of message | message type 3 | data (LIN id 1 byte,
    frame options 1 byte, frame flags 1 byte, data length 1 byte, data
    0-LIN_MAX_DATABYTES) | checksum

    LIN id can be any valid LIN id. 0-64.
    frame options:
        Specifies if frame is send/receive. Bit 0 - unset specifies frame is
        send fram. Bit 1 - if response shall be forwardet. Bit 3 - if frame is
        oneshot.

    frame flags:
        Specifies the version of LIN protocoll to be used. Bit 0 - unset
        specifies that protocoll 1 is used for this frame.

    data length:
        Length of data to be sent or received. For receive frames expected
        data length must be set.

##### Set item
    This frame type is used to setup an item in the schedule.
    It specifies a position in the schedule were a given LIN id shall be
    handled. Multiple items with the same id can exist. How many LIN ticks
    the item shall use and if the item is enabled.
    Format:
    message start | length of message | message type 4 | data (frame number 1
    byte, LIN id 1 byte, LIN ticks 1 byte, enabled 1 byte) | checksum

    frame number:
        Which slot in the scheudle. Value 0 - LIN_SCHEDULE_TABLE_ENRIES (256)

    LIN id:
        Any valid LIN id.

    LIN ticks:
        How long time the item occupies. One tick is 10ms long.

    Enable:
        Any value >0 to enable.

##### Set baudrate
    The defualt baudrate of this LIN implementation is 9600 bauds. The LIN
    specification states that baudrates 2400, 9600 and 19200 is supported.
    These rates are supported and rates 4800 and 10400 can also be set.
    Format:
    message start | length of message | message type 5 | data (baudrate
    enumerator 1 byte) | checksum

    baudrate enumeration:
        2400 - 1
        4800 - 2
        9600 - 3
        10400 - 4
        19200 - 5

##### Set master
    Configures LIN bus for either acting as master or just listening for LIN
    data on the bus. LIN bus is default configured as master.
    When configured as listener all valid messages will be forwarded.
    Format:
    message start | length of message | message type 6 | data (master/listen 1
    byte) | checksum

    master - 1
    listen - 0
#### Listen message
	When the LIN bus is in listen mode messages will be sent to Linux user space when data are found on the bus.
	Format:
	message start | length of message | message type 7 | status| data | checksum

	status: 
	Defined statuses:
```C
#define STATUS_OK 0
#define STATUS_RECV_ERROR 1
#define STATUS_RECV_NO_DATA 2
#define STATUS_MASTER_REQUEST 3
```
	STATUS_RECV_ERROR indicates that data have been read on the bus, but no valid LIN frame were found. Just all the data is returned for debuging purposes.
	STATUS_RECV_NO_DATA activity on the bus but no data found.
    STATUS_MASTER_REQUEST message with only one byte data found, this is a
    master request.

	data:
	The LIN frame. 
	id | data | LIN checksum
	id: With parity omit bit 7 and 6 for id without parity
	LIN checksum: If it is a valid frame the checksum is here
#### Overflow message
	When a overflow occurs at any of the busses handling LIN a report of which
    bus and how many overflows that have occured is sent to the Linux system.
    Format:
    message start | length of message | message type (8-11) | count
    Each typ of overflow has its own message type:
```C
#define MSG_LIN_RX_OF 8
#define MSG_LIN_TX_OF 9
#define MSG_TX_OF 10
#define MSG_RX_OF 11
```

### Status message
	All commands sent to PIC, except listen which is no command, will return a status message. 
	Format:
	message start | status
	status: One byte status, 0 status ok
```C
#define ERROR_ARGUMENT_COUNT 1
#define ERROR_START_NOT_VALID 2
#define ERROR_STOP_NOT_VALID 3
#define ERROR_START_AFTER_STOP 4
#define ERROR_SLOT_NOT_VALID 5
#define ERROR_NOT_VALID_ENTRY 6
#define ERROR_INVALID_LIN_ID 7
#define ERROR_TO_LONG_FRAME 8
	```
### Sample application for sending frames and receiving responses on Linux
```C
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <termios.h>

/* The application reads from arguments which kind of operation to perform and
 * all data associated with that operation. */

int main(int argc, char *argv[]) {
    int fd;
    int res;
    int fflags;
    char i;
    char msg_len;
    char listen;
    struct termios tio;
    char buff[255];

    fd = open("/dev/ttyHS2", O_RDWR | O_NOCTTY);
    if (fd < 0) {
        fprintf(stderr, "Error open tty\n");
        return -1;
    }

    tio.c_cflag = B115200 | CRTSCTS | CS8 | CLOCAL | CREAD;
    tio.c_iflag = IGNPAR;
    tio.c_oflag = 0;
    tio.c_lflag = 0;
    tio.c_cc[VTIME] = 0;

    tcflush(fd, TCIFLUSH);
    tcsetattr(fd, TCSANOW, &tio);

    buff[0] = 0x7E;
    buff[1] = 0x7E;
    buff[2] = 0x7E;
    buff[3] = 0xA7;

    listen = !strcmp(argv[1], "listen");

    if (!listen) {
        if (!strcmp(argv[1], "schedule")) {
            buff[5] = 2;
        }
        if (!strcmp(argv[1], "frame")) {
            buff[5] = 3;
        }
        if (!strcmp(argv[1], "item")) {
            buff[5] = 4;
        }
        if (!strcmp(argv[1], "baudrate")) {
            buff[5] = 5;
        }
        if (!strcmp(argv[1], "master")) {
            buff[5] = 6;
        }

        if (!buff[5]) {
            fprintf(stderr, "No valid command.\n");
            return 1;
        }

        for (i = 2; i < argc; i++) {
            sscanf(argv[i], "%d", &buff[4 + i]);
        }

        /* Checksum */
        msg_len = argc; /* Length of message is length of data + 1 for checksum */
        buff[4] = msg_len;
        for (i = 0; i < buff[4]; i++) {
            buff[4 + msg_len] ^= buff[4 + i];
        }

        write(fd, buff, 4 + 1 + msg_len); /* Bytes to send are: start of message bytes + length + (type, data, chksum) */

        sleep(1);
    }

    do {
        fflags = fcntl(fd, F_GETFL, 0);
        if (fflags < 0) {
            fprintf(stderr, "Could not get fd flags\n");
            return -1;
        }
        fcntl(fd, F_SETFL, fflags | O_NONBLOCK);

        res = read(fd, buff, 255);
        if (res > 0) {
            for (i = 0; i < res; i++) {
                printf("%x, ", buff[i]);
            }
            printf("\n");
        }
        usleep(10000);
    } while (listen);

    return 0;
}
```
#### Example of usage of application
```bash
# Setup frame 10 to send data to slaves using protocol 1 with 3 data bytes (1,
# 2, 3)
./lin_send frame 10 0 0 3 1 2 3
# Setup frame 16 to request a response from slaves using protocol 1 and expect
# 4 bytes long answer.
./lin_send frame 16 3 0 4
# Schedule created frames in slot 0 and 1, both using 60 ticks each and enable
# both.
./lin_send item 0 10 60 1
./lin_send item 1 16 60 1
# Enable the schedule to use items 0-1 start with item 0.
./lin_send schedule 0 1 0 1
# Start to listen for forwarded responses.
./lin_send listen
```

## Digital and Analog

### Digital

For digital I/O we use the standard linux framework.

See [Linux Kernel GPIO documentation](https://www.kernel.org/doc/Documentation/gpio/)

Note! GPIO numbers can differ on different MX-4 platforms. This is just an example.

```bash
root@mx4-gtt:/opt/hm# cat /sys/kernel/debug/gpio | grep -i digital
 gpio-23  (P43 - DIGITAL-IN-2  ) in  hi
 gpio-162 (P110 - DIGITAL-IN-1 ) in  hi
 gpio-171 (P45 - DIGITAL-IN-3  ) in  hi
 gpio-201 (P25 - DIGITAL-IN-6  ) in  hi
 gpio-220 (P22 - DIGITAL-IN-5  ) in  hi
 gpio-221 (P24 - DIGITAL-IN-4  ) in  hi
```

```bash
GPIOs 238-255, mx4_digitals:
 gpio-238 (mx4 - digital out 1 ) out lo
 gpio-239 (mx4 - digital out 2 ) out lo
 gpio-240 (mx4 - digital out 3 ) out lo
 gpio-241 (mx4 - digital out 4 ) out lo
 gpio-247 (mx4 - sc digital out) out lo
 gpio-248 (mx4 - sc digital out) out lo
 gpio-249 (mx4 - sc digital out) out lo
 gpio-250 (mx4 - sc digital out) out lo
```

`sc digital out` are inputs indicating short for each output. If a short is detected it goes HIGH (1).

#### Read gpio value

```bash
# Example reading status from Digital-In-2. 0 = LOW, 1 = HIGH
root@mx4-gtt:~# cat /sys/class/gpio/gpio23/value
0
```

#### Write gpio value

```bash
# Example setting Digital-Out-1 high
root@mx4-gtt:~# echo 1 > /sys/class/gpio/gpio238/value
```

#### Set input as interrupt

```bash
# Trigger an event on rising edge.
root@mx4-gtt:~# echo rising > /sys/class/gpio/gpio23/edge
```

It is also possible to set falling or both to edge file.

Example app listening for GPIO events.
```c
	#include <stdio.h>
	#include <fcntl.h>
	#include <poll.h>

	int main (int argc, char *argv[])
	{
		char* path;
		struct pollfd fds;
		int ret;

		if (argc != 2){
			printf("The file descriptor to listen is needed as first (and only) parameter\n");
			return 1;
		}

		path = argv[1];

		fds.fd = open (path, O_RDONLY);

		if (!fds.fd){
			printf("Unable to open file descriptor\n");
			return 1;
		}

		fds.events = POLLERR | POLLPRI;
		fds.revents = 0;
		printf ("listening for events in:%s\n", path);

		char val;

		while (1){
			read (fds.fd, 0, 0);
			if (poll (&fds, 1, -1) > 0) {
				printf ("event received\n");
			}
		}

		printf ("Finish\n");
		return 0;
	}
```
#### Multiplexed digital IO
Devices with IO pins with multiple functions, digital IO and LIN, must disable LIN before 
using the digital IO functionality. This is controled via the virtual gpios
LINx enabled. When the LIN channel is disabled the 

### Analog

The ADC conversions are managed by the co-processor and the values are exposed as sysfiles.

```bash
root@mx4-gtt:~# ls /opt/hm/pic_attributes/ | grep -i analog
analog_1_calibration_u
analog_1_uA
analog_2_calibration_u
analog_2_uA
analog_3_calibration_u
analog_3
analog_4_calibration_u
analog_4
root@mx4-gtt:~# ls /opt/hm/pic_attributes/ | grep -i input
input_battery_calibration_u
input_battery
input_battery_threshold_high
input_battery_threshold_low
input_temperature_calibration_u
input_temperature_mC
input_voltage_calibration_u
input_voltage
input_voltage_threshold_high
input_voltage_threshold_low
```

Calibration files are not be used by end users.

#### Example reading input voltage

```bash
root@mx4-gtt:~# cat /opt/hm/pic_attributes/input_voltage
14917
```

#### Example when Vref (adc reference voltage) is turned off.

Vref should normally always be on but it is turned off when entering sleep
([go_to_sleep.sh](https://github.com/hostmobility/mx4/blob/master/scripts/mx4/go_to_sleep.sh)).
So should you poll adc values while entering you will get some errors after Vref
has been turned off.

```bash
root@mx4-gtt:~# echo 0 > /sys/class/gpio/gpio243/value
root@mx4-gtt:~# cat /opt/hm/pic_attributes/input_voltage
cat: read error: Operation not permitted
```
## LED

LED flashing is configured by sending 4 bytes on corresponding led_-file in Linux.

**The configuration of the 4 bytes:**
<table>
    <tr>
        <td>MSB</td>
        <td>Second MSB</td>
        <td>Second LSB</td>
        <td>LSB</td>
    </tr>
    <tr>
       <td>Frequency</td>
       <td>Spare</td>
       <td>Flash config</td>
       <td>Current LED</td>
   <tr>
</table>

### Frequency
Frequency is configured according to:<br>
0 - Flashing off <br>
1-255 - Flashing with frequency; _value * 100 milliseconds_<br>
<br>
If only no alternative frequency is configured the time for each state will be the same. If different times are wanted also use alternative frequency.<br>

### Alternative frequency
0 - No alternative frequency. <br>
1-255 - Flashing with frequency; _value * 100 milliseconds_<br>

### Flash config
A bitmask configures which states that shall be included in the flashing sequence.<br>
Bit 0 - LED is off.<br>
Bit 1 - LED first state.<br>
Bit 2 - LED second state.<br>
...

### Current LED
Sets which LED that shall be lit upon reception of the configuration value. If only this value is set the LED will not change state until a new configuration is received, or the MX-4 is restarted.
This value can also be used to configure [oneshot](#wiki-oneshot) flashing.

### Oneshot
Oneshot flashing is configured by only configuring the wanted finishing state in the [flash config](#wiki-flash-config).
In [current LED](#wiki-current-led) the wanted starting state is configured.
In [frequency](#wiki-frequency) the wanted time for the oneshot is configured.

### Example of configuration
#### Flashing between state 1 and off, 1 second intervall.
Byte 3, frequency byte: 1s / 100ms = 10. <br>
Byte 2, not used: 0. <br>
Byte 1, bit 0 and bit 1 used: (1 << 1) | (1 << 0) = 3.<br>
<table>
    <tr>
        <td>Byte 3</td>
        <td>Byte 2</td>
        <td>Byte 1</td>
        <td>Byte 0</td>
    </tr>
    <tr>
       <td>10</td>
       <td>0</td>
       <td>3</td>
       <td>0</td>
   <tr>
</table>
Puting it together: <br>
10 << (8 * 3) | ((1 << 1) | (1 << 0)) | (8 * 1) = 167772171 <br>
In this case the flashing will start at state 0, if we want it to start at state 1 we have to set this in byte 0.<br>
In byte 0 numerical representations of the stat is used so just add 1 to previous value.<br>
#### Flashing between state 1 and 2, state 1 lit for 2 seconds and state 2 lit for 500 milliseconds.
Byte 3, frequency byte: 500ms / 100ms = 5. <br>
Byte 2, frequency byte: 2s / 100ms = 20. <br>
Byte 1, bit 1 and bit 2 used: (1 << 2) | (1 << 1) = 6.<br>
Byte 0, initial state. State configured as initial state will get the time configured in byte 3. So set to 2 in this case.
<table>
    <tr>
        <td>Byte 3</td>
        <td>Byte 2</td>
        <td>Byte 1</td>
        <td>Byte 0</td>
    </tr>
    <tr>
       <td>5</td>
       <td>20</td>
       <td>6</td>
       <td>2</td>
   <tr>
</table>
5 << (8 * 3) | 20 << (8 * 2) | 6 << 8 | 2 = 85198338 <br>
#### One-shot flash
To configure one-shot flash an initial state and a finishing state is configured with the time for the initial state configured in byte 3.
Initial state is configured in byte 0 and finishing state is configured in byte 1. This means that in byte 1 we can just put one state and that state will be kept until a new value is sent to the led.


### Pre configured LED behavior

#### PWR
- Solid green when running on supply voltage and battery charge state is above 60 %.
- Flashing green/yellow 1 Hz: Battery charge status is 41-60 % and charging.
- Flashing green/yellow 2 Hz: Battery charge status is 21-40 % and charging.
- Flashing green/yellow 4 Hz: Battery charge status is 0-20 % and charging.
- Solid yellow when running on battery and the battery is above 60 %
- Flashing yellow/off 1 Hz: Battery charge status is 41-60 %.
- Flashing yellow/off 2 Hz: Battery charge status is 21-40 %.
- Flashing yellow/off 4 Hz: Battery charge status is 0-20 %.
- Flashing green/off 1 Hz: The unit is in boot process

#### STANDBY STATE
- One-shot PWR LED green/off (on 30 ms) approximately every 20 seconds.

#### ERROR STATE
- All LEDS(not wifi) Flashing green/off 5 Hz: The unit is in an error state. Failing to boot operating system. This is the default state when the unit is powered on.

#### FIRMWARE UPGRADE
A firmware upgrade can be followed by monitoring the LED behavior.

1. All LEDS(not wifi) Flashing green/off 1 Hz: The unit is loading/running an image ("hmupdate.img") from USB driver or /boot. DO NOT REMOVE POWER!
1. When hmupdate.img is finished the system will start up. This will be indicated by PWR LED Flashing green/off 1 Hz.
1. First boot after hmupdate.img is run we also update co-processor firmware. This is indicated by following sequence:
- FUNC LED Solid green and other LED's off. Entered co-processor bootloader and will load an application to update co processor bootloader.
- FUNC LED solid green, GPS solid green and other LED's off. Bootloader application is loaded and will update co-processor bootloader.
- FUNC LED Solid green and other LED's off. Entered co-processor bootloader and will load the final application.
- LED behavior returns to normal operation.

During the above steps GSM LED will flash when we are sending data to the co-processor for upgrade.

1. Done!

## Sound

To direct line-in to line-out:

```bash
amixer -c 2 set 'Left HP Mixer Line Bypass' on
amixer -c 2 set 'Right HP Mixer Line Bypass' on
```

Playback-hdmi:

```bash
aplay -D 'default:CARD=tegra' bach.au
```

Playback:

```bash
amixer -c 2 set 'Headphone' 8 on
amixer -c 2 set 'Left HP Mixer PCM' on
amixer -c 2 set 'Right HP Mixer PCM' on
aplay -D 'default:CARD=T20' output2.wav
```

Record from line-in:

```bash
amixer -c 2 set 'Left Capture Select' 'Line'
amixer -c 2 set 'Right Capture Select' 'Line'
amixer -c 2 set 'Capture ADC' on
arecord -D 'default:CARD=T20' -v -f cd output2.wav
```

Record from mic:

```bash
amixer -c 2 set 'Left Capture Select' 'Mic'
amixer -c 2 set 'Right Capture Select' 'Mic'
amixer -c 2 set 'Capture ADC' on
amixer -c 2 set 'Left HP Mixer Mic Sidetone' on
amixer -c 2 set 'Right HP Mixer Mic Sidetone' on
arecord -D 'default:CARD=T20' -v -f cd output2.wav
```

Please note that the two numbers at the end specify which ALSA card and device to use for audio (e.g. alsasink device=hw:1,0 for SPDIF through HDMI and alsasink device=hw:2,0 for WM9715L AC97 through headphone).

```bash
aplay -L
null
    Discard all samples (playback) or generate zero samples (capture)
default:CARD=tegra // HDMI
    tegra,
    Default Audio Device
default:CARD=T20 // AC97
    Toradex Colibri T20,
    Default Audio Device
```

## Power Management

The MX-4 platform has three different operating modes:
- Running
- Sleep
- Deep Sleep
- Shutdown/Cutoff

### Running

Running is the default operating state where we have full functionality.

### Sleep

The MX-4 platform has support for entering sleep/suspend mode. This a power saving
mode where as much as possible is powered down to minimize the power consumption and
fast resume time (around 1 second) to running state.

#### Overview of suspend design

The MX-4 platform consists of two cpu's:
- The main CPU which runs Linux and where user applications are run
- The co-cpu which is a multi purpose CPU (PMIC, gpio-chip, analog inputs, hardware watchdog and more)

The main CPU is the one that always initiates sleep/suspend mode.

Wakeup on the other hand could be that main CPU wakes up co-cpu or co-cpu wakes up main CPU (this depends on wakeup source. See [Wakeup](#wakeup)).

When sleep/suspend mode is entered
- The main CPU will enter a mode where most of its peripherals are powered off.
- The co-cpu will run on internal low power RC (32 kHz) and run a cyclic sleep
- 3.3 V is turned off. Consumers differ across different MX-4 platforms
- 5 V is turned off. Consumers differ across different MX-4 platforms

List of periphials that will lose power and that need to be re-initalized after wakeup:
- SD-card - Application should umount SD-card before entering sleep/suspend. Re-mount is handled by platform on wakeup.
- Wifi
- UARTS
- CAN
- Ethernet

#### Enter sleep

Host Mobility provides a script to easy enter sleep/suspend mode. The script is `/opt/hm/go_to_sleep.sh`.

```bash
Usage: go_to_sleep.sh options (t:D:hdcnsal:p:)
                -t <time in seconds> - Setup wakealarm (rtc)
                -d - Disable wake on DIGITAL-IN-2
                -a - Disable wake on START-SIGNAL
                -l <mV level> - START-SIGNAL wake-up volt level
                -c - Wake on CAN
                -n - Will renew dhcp lease
                -s - Will suspend modem (turn off), Will not restore on wake up
                -D <time in seconds> - Will enter deep sleep.
                        Power to main CPU will be cut after specified time and it
                        will restart with a cold reboot on wake up. The application
                        is responsible of shutting down the system properly before
                        the power is cut.
                -p <wake up mask> - Mask to enable/disable wakeup sources
                -h - Print this text

```

#### Wakeup

The MX-4 platform supports a lot different wakeup sources. Below is a list off supported wakeup sources.
Note that not all MX-4 platforms supports all wakeup sources.
- Digital Inputs (DIGITAL-IN-2 is enabled by default as wakeup source)
- Analog Inputs
- Wake on CAN
- Wake on RTC(wake up after x seconds)
- Wake on RING (call/SMS to modem) - Not yet supported in software
- Wake on Accelerometer - Not yet supported in software

##### Wake on Digital Inputs

By default DIGITAL-IN-2 (rising edge) is enabled as wakeup source. This is mostly for historical reasons. To disable it one can pass `-d` option to `go_to_sleep.sh`.

On some platforms DIGITAL-IN-2 is the only digital input that is capable to wake up the system.

If your output of `cat /sys/kernel/debug/gpio | grep digital` looks like below

```
GPIOs 238-273, spi/spi3.0, mx4_digitals:
 gpio-238 (digital-out-1       ) out lo
 gpio-239 (digital-out-2       ) out lo
 gpio-240 (digital-out-3       ) out lo
 gpio-241 (digital-out-4       ) out lo
 gpio-242 (digital-out-5 / 4-20) out lo
 gpio-243 (digital-out-6       ) out lo
 gpio-250 (digital-in-1 / sc   ) in  hi
 gpio-251 (digital-in-2 / sc   ) in  hi
 gpio-252 (digital-in-3 / sc   ) in  hi
 gpio-253 (digital-in-4 / sc   ) in  hi
 gpio-254 (digital-in-5 / sc   ) in  lo
 gpio-255 (digital-in-6        ) in  lo
```
Then your system is capable of having all digital inputs as wakeup sources.

To enable different wakeup sources and to set edge, the `-p` option is used. The `-p` takes an argument which should be a bitmask of following.

```
#define WAKE_UP_TRIGGER_CAN             (1UL << 0)
#define WAKE_UP_TRIGGER_DIN_1_F         (1UL << 1)
#define WAKE_UP_TRIGGER_DIN_1_R         (1UL << 2)
#define WAKE_UP_TRIGGER_DIN_2_F         (1UL << 3)
#define WAKE_UP_TRIGGER_DIN_2_R         (1UL << 4)
#define WAKE_UP_TRIGGER_DIN_3_F         (1UL << 5)
#define WAKE_UP_TRIGGER_DIN_3_R         (1UL << 6)
#define WAKE_UP_TRIGGER_DIN_4_F         (1UL << 7)
#define WAKE_UP_TRIGGER_DIN_4_R         (1UL << 8)
#define WAKE_UP_TRIGGER_DIN_5_F         (1UL << 9)
#define WAKE_UP_TRIGGER_DIN_5_R         (1UL << 10)
#define WAKE_UP_TRIGGER_DIN_6_F         (1UL << 11)
#define WAKE_UP_TRIGGER_DIN_6_R         (1UL << 12)
#define WAKE_UP_TRIGGER_MODEM_RING      (1UL << 13)
#define WAKE_UP_TRIGGER_START_SWITCH_F  (1UL << 14)
#define WAKE_UP_TRIGGER_START_SWITCH_R  (1UL << 15)
#define WAKE_UP_TRIGGER_MIN_1_F         (1UL << 16)
#define WAKE_UP_TRIGGER_MIN_1_R         (1UL << 17)
#define WAKE_UP_TRIGGER_MIN_2_F         (1UL << 18)
#define WAKE_UP_TRIGGER_MIN_2_R         (1UL << 19)
```

NOTE! If `-d` option is not specified it will always set bit 4 in the wakeup mask, regardless of what you passed.

##### Wake on Analog Inputs

Analog inputs as wakeup sources are not handled by `go_to_sleep.sh`.

All analog inputs have four sysfs files associated with them. If we take input voltage as an example:
```bash
root@mx4-vcc-1000000:/sys/bus/spi/devices/spi3.0# ls input_voltage*
input_voltage_calibration_u      input_voltage_threshold_high
input_voltage                 input_voltage_threshold_low
```

- `input_voltage` is the file where we read the value of input voltage.
- `input_voltage_calibration_u` is used internally by Host Mobility to calibrate the input for component tolerances.
- `input_voltage_threshold_high` is used to enable wake on a high level threshold
- `input_voltage_threshold_low` is used to enable wake on low level threshold.

Some wakeup setup examples:

```bash
# Wakeup system from sleep/suspend if input voltage is above 16 V
root@mx4-vcc-1000000:~# echo 16000 > /opt/hm/pic_attributes/input_voltage_threshold_high
```

```bash
# Wakeup system from sleep/suspend if input voltage is below 12 V
root@mx4-vcc-1000000:~# echo 12000 > /opt/hm/pic_attributes/input_voltage_threshold_low
```

```bash
# Wakeup system from sleep/suspend if input voltage is in the range of 12-16 V
root@mx4-vcc-1000000:~# echo 16000 > /opt/hm/pic_attributes/input_voltage_threshold_high
root@mx4-vcc-1000000:~# echo 12000 > /opt/hm/pic_attributes/input_voltage_threshold_low
```

```bash
# Write 0 to both threshold files to disable that specific input as wakeup source
root@mx4-vcc-1000000:~# echo 0 > /opt/hm/pic_attributes/input_voltage_threshold_high
root@mx4-vcc-1000000:~# echo 0 > /opt/hm/pic_attributes/input_voltage_threshold_low
```
User has to setup analog wakeup sources prior to calling `go_to_sleep.sh`. See [Enter sleep](#enter-sleep)

##### Wake on CAN

Wake on CAN is not supported by all MX-4 platforms. Contact Host Mobility support to see if your platform supports this.

Wake on CAN is enabled by passing `-c` to `go_to_sleep.sh`. See [Enter sleep](#enter-sleep). But prior to doing this one has
to enable which specific CAN buses should be enabled as wakeup source.

By default all CAN buses will trigger a wakeup if there is traffic on them and `-c` is passed to `go_to_sleep.sh`

GPIO's are used to set this up.

```bash
root@mx4-vcc-1000000:~# cat /sys/kernel/debug/gpio | grep -i wakeup
 gpio-58  (P169 - CAN0-WAKEUP  ) out lo
 gpio-59  (P171 - CAN1-WAKEUP  ) out lo
 gpio-60  (P173 - CAN2-WAKEUP  ) out lo
 gpio-61  (P175 - CAN3-WAKEUP  ) out lo
 gpio-62  (P177 - CAN4-WAKEUP  ) out lo
 gpio-63  (P179 - CAN5-WAKEUP  ) out lo
```

To disable wakeup for a specific bus one has to write a 1 to that GPIO's value.

```bash
# Disable wakeup on CAN0
root@mx4-vcc-1000000:~# echo 1 > /sys/class/gpio/gpio58/value

```

### Deep Sleep

Deep sleep mode means that we put the co-cpu in a sleep mode while cutting the power rail to the main CPU. This means that upon wake up we will have a cold reboot instead of a fast resume, this trade off is for lower consumtion during the suspended state.

Wakeup sources available from deep sleep vary on different platforms.

Deep sleep is entered by passing the -D option to `go_to_sleep.sh`. See [Enter sleep](#enter-sleep)

### Shutdown

This mode is not supported by all MX-4 platforms. Contact Host Mobility support to see if your platform supports this.

The MX-4 shutdown/cutoff state is where system is turned off with close to zero power consumption.

```bash
# Cutoff in 60 seconds
root@mx4-vcc-1000000:~# echo 60 > /opt/hm/pic_attributes/ctrl_on_4v
```

The above means that in 60 seconds we will cut the 4 V which the whole system
runs on and the system will only restart if ANALOG-IN-1 is HIGH.

Before these 60 seconds run out the application should have finished and
run the command `poweroff` to shutdown Linux.

```bash
root@mx4-vcc-1000000:~# echo 60 > /opt/hm/pic_attributes/ctrl_on_4v
root@mx4-vcc-1000000:~# poweroff
Sending SIGTERM to remaining processes...
Sending SIGKILL to remaining processes...
Unmounting file systems.
Unmounting /sys/fs/fuse/connections.
All filesystems unmounted.
Deactivating swaps.
All swaps deactivated.
Detaching loop devices.
All loop devices detached.
Detaching DM devices.
All DM devices detached.
[  239.327263] System halted.
```

## Package Manager

There is a opkg feed for the MX4<br>
<br>
Create or edit the file /etc/opkg/hm-feed.conf, by adding the following lines<br>
<br>
src/gz all http://www.hostmobility.org:8008/ipk/all<br>
src/gz armv7ahf-vfp http://www.hostmobility.org:8008/ipk/armv7ahf-vfp<br>
src/gz colibri_t20 http://www.hostmobility.org:8008/ipk/colibri_t20<br>
<br>
After that simply run

```bash
opkg update
opkg list
```

To install something run for instance<br>
```bash
opkg install rsync
```

## Accelerometer
The MX-4 is equipped with a Freescale MMA8452 accelerometer. 
Datasheet: (http://www.freescale.com/files/sensors/doc/data_sheet/MMA8452Q.pdf)

#### Usage on VF61-based systems
Path in sysfs: /sys/bus/iio/devices/iio:device0<br>
Read x,y,z: in_accel_{x,y,z}_raw<br>

At the moment this driver only supports reading of coordinates via sysfs.

#### Usage on T20-based systems
Path in sysfs: /sys/class/sensor/mma/ <br>
Read x,y,z: value_{x,y,z} <br>

Interrupts from the accelerometer are routed to two GPIOs on the Colibri
module. The GPIOs are not exported.
Data ready is routed to INT1, other interrupts are routed to INT2.
INT1 is the first mma interrupt seen in /proc/interrupts.

All data generated from interrupts are accessed via chardevs.
lsinput and input-event are part of package input-utils.
```bash
lsinput
/dev/input/event0
   bustype : (null)
   vendor  : 0x0
   product : 0x0
   version : 0
   name    : "mma845x"
   bits ev : EV_SYN EV_ABS

/dev/input/event1
   bustype : (null)
   vendor  : 0x0
   product : 0x0
   version : 0
   name    : "Accl1"
   bits ev : EV_SYN EV_KEY EV_ABS EV_MSC
```
mma845x is for coordinate data. Accl1 is for configured interrupts.


Most of the possibilities mentioned in the mma8452 datasheet are available for
configuration via the driver sysfs interface.  

Example for testing transient interrupts.
```bash
cd /sys/class/sensor/mma/transitent_detection0
echo enable,enable,enable > enable
echo 1 > threshold # set lowest possible threshold for transient
input-events 1
# Move the device to generate a movement transient
# Output:
dev/input/event1
   bustype : (null)
   vendor  : 0x0
   product : 0x0
   version : 0
   name    : "Accl1"
   bits ev : EV_SYN EV_KEY EV_ABS EV_MSC

waiting for events
00:18:33.808626: EV_KEY KEY_LEFTSHIFT pressed
00:18:33.808629: EV_KEY KEY_G pressed
00:18:33.808633: EV_KEY KEY_G released
00:18:33.808635: EV_KEY KEY_LEFTSHIFT released
00:18:33.808639: EV_ABS ??? 96
00:18:33.808643: EV_ABS ??? 0
00:18:33.808644: EV_SYN code=0 value=0
```
[Docker install]:https://docs.docker.com/engine/installation/#supported-platforms
