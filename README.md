WINGZ-FILES
===========

Install these packages before proceeding further

	sudo apt-get update 
	sudo apt-get install libc6:i386 libstdc++6:i386 libncurses5:i386 zlib1g:i386
	sudo apt-get install -y build-essential libncurses5 libncurses5-dev
	
Do the following steps in the followiong folder

~/

Get the WINGZ-FILES 

      git clone https://github.com/CDACBANG/WINGZ-FILES.git 


Get and Extract the compiler

      wget -c https://releases.linaro.org/14.09/components/toolchain/binaries/gcc-linaro-arm-linux-gnueabihf-4.9-2014.09_linux.tar.xz

      tar xf gcc-linaro-arm-linux-gnueabihf-4.9-2014.09_linux.tar.xz

      export CC=`pwd`/gcc-linaro-arm-linux-gnueabihf-4.9-2014.09_linux/bin/arm-linux-gnueabihf-

Verify the proper version:
      
      ${CC}gcc --version
      arm-linux-gnueabihf-gcc (crosstool-NG linaro-1.13.1-4.9-2014.09 - Linaro GCC 4.9-2014.09) 4.9.2 20140904 (prerelease)
      Copyright © 2014 Free Software Foundation, Inc.
      This is free software; see the source for copying conditions. There is NO
      warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.


U-BOOT :: U-Boot Compilation for WINGZ board
--------------------------------------------

u-boot:

~/

Get the u-boot for WINGZ 

	  git clone https://github.com/CDACBANG/u-boot-wingz.git 
	  
	  cd $HOME/u-boot-wingz/
	  
	  make ARCH=arm CROSS_COMPILE=${CC} distclean
	  make ARCH=arm CROSS_COMPILE=${CC} omap3_wingz_defconfig
	  make ARCH=arm CROSS_COMPILE=${CC}
	  
	  
it will generate, x-loader (MLO) and u-boot bootloader (u-boot.img)


KERNEL Compilation for WINGZ Board
----------------------------------

~/ 
Get the latest kernel:

      git clone https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git

copy the **device source tree file** of wingz to the linux kernel 

      cp $HOME/WINGZ-FILES/wingz_dts/omap3-wingz.dts  $HOME/linux-stable/arch/arm/boot/dts/
      
Add an entry in linux-stable/arch/arm/boot/dts/Makefile to inform the kernel to compile our .dts file also
	
	
	         dtb-$(CONFIG_ARCH_OMAP3) += am3517-craneboard.dtb \
           am3517-evm.dtb \
           am3517_mt_ventoux.dtb \
           omap3430-sdp.dtb \
           omap3-beagle.dtb \
           omap3-wingz.dtb \
           omap3-beagle-xm.dtb \
           omap3-beagle-xm-ab.dtb \
           omap3-cm-t3517.dtb \
           omap3-cm-t3530.dtb \
	
	
	
	
Update tux logo with CDAC-WINGZ logo 

    cp $HOME/WINGZ-FILES/wingz_logo/wingz-logo.ppm $HOME/linux-stable/drivers/video/logo/logo_linux_clut224.ppm


    make ARCH=arm CROSS_COMPILE=${CC} distclean

Now, copy the configuration file of wingz to the linux-kernel configuration file

    cp $HOME/WINGZ-FILES/wingz_config/wingz-config $HOME/linux-stable/.config

    cd $HOME/linux-stable/
    
    make -j4 ARCH=arm CROSS_COMPILE=${CC} menuconfig
    make -j4 ARCH=arm CROSS_COMPILE=${CC} zImage modules
    make -j4 ARCH=arm CROSS_COMPILE=${CC} dtbs

zImage will be created in ~/linux-stable/arch/arm/boot/ folder 

**After compilation:-**

This will export the kernel_version you just compiled


    export kernel_version=$(cat include/generated/utsrelease.h | awk '{print $3}' | sed 's/\"//g' )


Copy the Final zImage and configuration file to the other folder, which we will use later for copying to SD-card

$HOME/linux-stable/

      cp -v arch/arm/boot/zImage  $HOME/compiled_modules/kernel_image/${kernel_version}.zImage
      cp -v .config $HOME/compiled_modules/kernel_image/config-${kernel_version}


Install the modules, firmware and dtbs in the local folder later to copy to the SD card
  
    make  ARCH=arm CROSS_COMPILE=${CC} dtbs_install INSTALL_DTBS_PATH=$HOME/compiled_modules/dtbs/
    make  ARCH=arm CROSS_COMPILE=${CC} modules_install INSTALL_MOD_PATH=$HOME/compiled_modules/modules/
    make  ARCH=arm CROSS_COMPILE=${CC} firmware_install INSTALL_FW_PATH=$HOME/compiled_modules/firmwares/
    
    

File system:
-----------

Get:
      wget -c https://rcn-ee.net/deb/minfs/trusty/ubuntu-14.04.1-minimal-armhf-2014-11-10.tar.xz

Verify:
      md5sum ubuntu-14.04.1-minimal-armhf-2014-11-10.tar.xz
      7e5fa3cb4814f195d75cbaad6d922090 ubuntu-14.04.1-minimal-armhf-2014-11-10.tar.xz

Extract:
    
      tar xf ubuntu-14.04.1-minimal-armhf-2014-11-10.tar.xz

or 

use fs in $HOME/WINGZ-FILES/wingz_fs/ubuntu-14.04.1-minimal-armhf-2014-11-10.tar.xz 
    
      tar xf ubuntu-14.04.1-minimal-armhf-2014-11-10.tar.xz


Bootloaders to SD card:

        sudo cp -v ./u-boot-wingz/MLO /media/boot/
        sudo cp -v ./u-boot-wingz/u-boot.img /media/boot/

FS to SD card:

      sudo tar xfvp /ubuntu-14.04.1-minimal-armhf-2014-11-10/armhf-rootfs-*.tar -C /media/rootfs/
      sudo mkdir -p /media/rootfs/boot/
      sudo sh -c "echo 'uname_r=${kernel_version}' > /media/rootfs/boot/uEnv.txt"

Install kernel to SD card:

      sudo cp -v $HOME/compiled_modules/kernel_image/${kernel_version}.zImage   /media/rootfs/boot/vmlinuz-${kernel_version}
      sudo mkdir -p /media/rootfs/boot/dtbs/${kernel_version}/
      sudo cp -rfv $HOME/compiled_modules/dtbs/ /media/rootfs/boot/dtbs/${kernel_version}/

      sudo cp -rfv $HOME/compiled_modules/modules/* /media/rootfs/
      sudo cp -rfv $HOME/compiled_modules/firmwares/* /media/rootfs/lib/firmware/


**File system table:**

      sudo sh -c "echo '/dev/mmcblk0p2  /  auto  errors=remount-ro  0  1' >> /media/rootfs/etc/fstab"

**Networking:**

      sudo nano /media/rootfs/etc/network/interfaces

Add:
      /etc/networking/interfaces
      
      
      auto lo
      iface lo inet loopback
 
      auto eth0
      iface eth0 inet dhcp

      hwaddress ether xx:xx:xx:xx:xx:xx  

Note: use you own mac id for eth0 in order to avoid getting fake mac for every reboot
Note: After connecting LAN cable to the board if you are not getting ip run,

      sudo /etc/init.d/networking restart

**Serial Login**
  
      sudo nano /media/rootfs/etc/init/serial.conf

Add:

      /etc/init/serial.conf

      start on stopped rc RUNLEVEL=[2345]
      stop on runlevel [!2345]

      respawn
      exec /sbin/getty 115200 ttyO2

Now, update the SD card and unmout it!

      sync
      umount /media/boot
      umount /media/rootfs
      
      
Place the SD card in the WINGZ board, power it up. It has to boot successfully..... 
