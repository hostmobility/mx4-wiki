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
- [Analog](#analog)
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

## Reset Cause

One can read out the system reset cause from a spi attribute.

```bash
root@ultra14211046:~# cat /opt/hm/pic_attributes/ctrl_pic_reset_cause
or use /opt/hm/reset_cause.sh
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
#define PRC_TRAP_DEFAULT		REG_BIT(11)
#define PRC_TRAP_OSC			REG_BIT(12)
#define PRC_TRAP_ADDRESS		REG_BIT(13)
#define PRC_TRAP_STACK			REG_BIT(14)
#define PRC_PROTOCOL_WATCHDOG		REG_BIT(16)
#define PRC_TRAP_MATH			REG_BIT(15)
#define PRC_CLOCK_SWITCH_FAILED		REG_BIT(17)
#define PRC_DEEP_SLEEP_EXIT		REG_BIT(18)
```

0
"Reset Cause String: Power On Reset"

1
"Reset Cause String: Brown Out Reset"

2
"Reset Cause String: Idle Reset"

3
"Reset Cause String: Sleep Reset"

4
"Reset Cause String: Watchdog Reset (Co-cpu rest internal watchdog flag trigged)

5
"Reset Cause String: Software Reset (set by user)"

6
"Reset Cause String: MCLR Reset (reset button)"

7
"Reset Cause String: Config Missmatch Reset"

8
"Reset Cause String: Deep Sleep Reset"

9
"Reset Cause String: Illegal Opcode Reset"

10
"Reset Cause String: Trap Conflict Reset"

11
"Reset Cause String: Trap Default Reset
and trace_default_interrupt"

12
"Reset Cause String: Trap OSC Reset"


13
"Reset Cause String: Trap Address Reset"


14
"Reset Cause String: Trap Stack Reset"

15
"Reset Cause String: Trap Math Reset"


16
"Reset Cause String: Protocol Watchdog Reset (linux spi protocol stoped sending and co-cpu reset)"

17
"Reset Cause String: Clock Switch Failed"

18
"Reset Cause String: Deep Sleep Exit"

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


## Analog

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


[Docker install]:https://docs.docker.com/engine/installation/#supported-platforms
