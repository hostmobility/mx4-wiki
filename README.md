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
- [Communication Interfaces](#communication-interfaces)
- [Package Manager](#package-manager)

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

#### Build Docker image
Build the docker image using the Dockerfile and name it hostmobility/mx4
<pre>
docker build -t hostmobility/mx4 .
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

### Building other platforms

The needed repositories and branch names can be seen here: https://github.com/hostmobility/mx4#source-structure

### GTT v1.5.x
<pre>
git clone https://github.com/hostmobility/mx4/ -b mx4-bsp-v1.5.x
git clone https://github.com/hostmobility/mx4-pic -b mx4-bsp-v1.5.x mx4/pic
./make_system.sh -t gtt -r <b>UNIQUE_ROOTFS_PATH</b> -d <b>YOCTO_1.5.x_TEMP_PATH</b> -g -k -u -j <b>$(nproc)</b> -m -T 512
</pre>

### GTT-T30 v1.4.x
<pre>
git clone https://github.com/hostmobility/mx4/ -b mx4-bsp-v1.4.x-ultra
git clone https://github.com/hostmobility/mx4-pic -b mx4-bsp-v1.4.x mx4/pic
git clone https://github.com/hostmobility/linux-toradex.git -b mx4-bsp-v1.4.x-tegra mx4/t20/linux_vf
git clone https://github.com/hostmobility/u-boot-toradex.git -b mx4-bsp-v1.4.x mx4/t20/u-boot_vf
./make_system.sh -t t30 -r <b>UNIQUE_ROOTFS_PATH</b> -d <b>YOCTO_1.4.x_TEMP_PATH</b> -g -k -u -j <b>$(nproc)</b>
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


## Wake up Cause
```bash
cat $MX4_SPI_DIR/ctrl_wakeup_cause
or /opt/hm/wake_up_cause.sh
```

```c
#define WAKE_UP_SRC_NONE            0x00
#define WAKE_UP_SRC_WDT             0x01
#define WAKE_UP_SRC_SPI_INT         0x02
#define WAKE_UP_SRC_MAIN_VOLTAGE    0x03
#define WAKE_UP_SRC_BATTERY_VOLTAGE 0x04
#define WAKE_UP_SRC_ANALOG_1        0x05
#define WAKE_UP_SRC_ANALOG_2        0x06
#define WAKE_UP_SRC_ANALOG_3        0x07
#define WAKE_UP_SRC_ANALOG_4        0x08
#define WAKE_UP_SRC_START_SIGNAL    0x09
#define WAKE_UP_SRC_CAN             0x0a
#define WAKE_UP_SRC_DIN_1_F         0x0b
#define WAKE_UP_SRC_DIN_1_R         0x0c
#define WAKE_UP_SRC_DIN_2_F         0x0d
#define WAKE_UP_SRC_DIN_2_R         0x0e
#define WAKE_UP_SRC_DIN_3_F         0x0f
#define WAKE_UP_SRC_DIN_3_R         0x10
#define WAKE_UP_SRC_DIN_4_F         0x11
#define WAKE_UP_SRC_DIN_4_R         0x12
#define WAKE_UP_SRC_DIN_5_F         0x13
#define WAKE_UP_SRC_DIN_5_R         0x14
#define WAKE_UP_SRC_DIN_6_F         0x15
#define WAKE_UP_SRC_DIN_6_R         0x16
#define WAKE_UP_SRC_MODEM_RING      0x17
#define WAKE_UP_SRC_START_SWITCH_F  0x18
#define WAKE_UP_SRC_START_SWITCH_R  0x19
#define WAKE_UP_SRC_MIN_1_F         0x20
#define WAKE_UP_SRC_MIN_1_R         0x21
#define WAKE_UP_SRC_MIN_2_F         0x22
#define WAKE_UP_SRC_MIN_2_R         0x23
#define WAKE_UP_SRC_DIN_7_F         0x24
#define WAKE_UP_SRC_DIN_7_R         0x25
#define WAKE_UP_SRC_DIN_8_F         0x26
#define WAKE_UP_SRC_DIN_8_R         0x27
```

- WDT = watchdog 
- SPI INT = Spi communication from linux system
- DIN_X = Digital In number n
- ANALOG = Analog In number n 


## Communication Interfaces


Migrated to https://github.com/hostmobility/documentation 

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


[Docker install]:https://docs.docker.com/engine/installation/#supported-platforms
