# This is the manifest of the Advantech Linux TSU Yocto project
The goal of this project is to release opensource Linux source code that runs on Advantech Hardware

## Download the Yocto Linux for ECU-150-A1 and setup environment variables
For downloading the source code and setting up the environment, follow the instructions below:
```console
foo@bar:~/yocto$ repo init -u https://github.com/saurontech/Advantech-LinuxTSU-manifest.git -b main -m ecu-150a1-6.6.36-2.1.0.xml
foo@bar:~/yocto$ repo sync
foo@bar:~/yocto$ MACHINE=imx8mq-ecu150a1 DISTRO=fsl-imx-xwayland source ./imx-setup-release.sh -b build
foo@bar:~/yocto/build$
```
After the commands, one has not only downloaded the Yocto project for ECU-150-A1, the environment variables, of the operating console, were also setup to operate bitbaker.
Please also notice, that after the commands, your current position has been changed to the build directory!
If, in the future, one wishes to setup environment variables of another console; in that spacific console, one should use the command:
```console
foo@bar:~/yocto$ source ./setup-environment build
foo@bar:~/yocto/build$
```
## Build Yocto
In a console that has been setup properly, use the following command to build Linux and the Yocto rootfs
```console
foo@bar:~/yocto/build$ bitbake core-image-minimal

```
- The Linux kernel will be located at : ./build/tmp/deploy/images/imx8mq-ecu150a1/Image
- The dtb will be located at: ./build/tmp/deploy/images/imx8mq-ecu150a1/fsl-imx8mq-ecu150a1.dtb
- The rootfs will be located at: ./build/tmp/deploy/images/imx8mq-ecu150a1/core-image-minimal-imx8mq-ecu150a1.rootfs.tar.gz

If one only wishes to build only the Linux kernel, use the following command:
```console
foo@bar:~/yocto/build$ bitbake linux-imx
```
## Create SDK for Yocto
```console
foo@bar:~/yocto/build$ bitbake -c populate_sdk core-image-minimal
```
## Build out-of-tree kernel modules
The following example shows how one can build a out-of-tree kernel module with the Yocto SDK.  
We use the Advantech USB-4604B USB to serial converter as an example.  
>Notice that this spacific drivers makefile uses **$(KERNELDIR)** to represent the position of the kernel header files; therefore, we use **export KERNELDIR=$SDKTARGETSYSROOT/usr/src/kernel** before we build the driver with **make**.

```console
foo@bar:~/example$ git clone https://github.com/saurontech/USB-4604-BE-linux-driver.git
foo@bar:~/example$ cd ./USB-4604-BE-linux-driver/driver
foo@bar:~/example/USB-4604-BE-linux-driver/driver$ source /opt/fsl-imx-wayland/6.1-mickledore/environment-setup-armv8a-poky-linux
foo@bar:~/example/USB-4604-BE-linux-driver$ cat ./Makefile
obj-m := adv_usb_serial.o
adv_usb_serial-objs := xr_usb_serial_common.o

KERNELDIR ?= /lib/modules/$(shell uname -r)/build
PWD       := $(shell pwd)

EXTRA_CFLAGS	:= -DDEBUG=0

all:
	$(MAKE) -C $(KERNELDIR) M=$(PWD)

modules_install:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules_install

clean:
	rm -rf *.o *~ core .depend .*.cmd *.ko *.mod.c .tmp_versions vtty *.symvers *.order *.a *.mod
foo@bar:~/example/USB-4604-BE-linux-driver/driver$ export KERNELDIR=$SDKTARGETSYSROOT/usr/src/kernel
foo@bar:~/example/USB-4604-BE-linux-driver$ make
```
