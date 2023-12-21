![](http://hostmobility.org/mx4/pics/mx-4-family-web.jpg)

***

##### Toolchain


There are two flavors of sysroots. The minimal is based on `console-vcc-base-image` and the other one is based on `lxde-mx4-image`.

  - hardware/host-monitor-x.md
They are available on following link http://www.hostmobility.org:8080/tools/.

There is a problem with installation where you get segmentation faults when trying to run a binary. See this link http://developer.toradex.com/how-to/how-to-set-up-qt-creator-to-cross-compile-for-embedded-linux#Install_the_SDK for a workaround.

Linaro 2013.04 is also compatible with the above toolchains. https://launchpad.net/linaro-toolchain-binaries/trunk/2013.04/+download/gcc-linaro-arm-linux-gnueabihf-4.7-2013.04-20130415_linux.tar.xz.

##### Release structure

When master branch is considered to be stable with a certain amount of new functionality there will be a new BSP release.

First the mx4-bsp-vx.x.x tag is created in mx4 and in all sub repositories. The tag is converted to a branch in which one can push bug fixes. Each BSP release will get a Jenkins job defined.

The above structure is needed since we have alot of products in the same tree and we most likely will break something when adding new products to the tree. This way we have always our stable branches while developing new functionality.


#### Source code

Host Mobility's provides read-only access to our software repositories hosted at https://github.com/hostmobility.

With this access you can fork the repository, create pull-requests, create issues and clone the repository and build the whole platform from scratch.

#### Support

Host Mobility AB provides first class support.

We will help you get started with MX-4 development and once the initial steps are done we also provide tips and tricks to optimize your application to our platform.

Beside the documentation and wiki you can also contact Host Mobility developers directly with your questions. See http://hostmobility.com for contact information.

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

## Communication Interfaces


Migrated to https://github.com/hostmobility/documentation 

