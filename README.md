# BootLoader
BootLoader

# uboot 
https://gitlab.com/u-boot/u-boot

# 步骤

# raspberry pi 3 b plus 
之前使用 
make rpi_3_b_plus_defconfig 烧录有没有任何 用

现在 参考 文档 再试一次
https://blog.csdn.net/m0_37565736/article/details/135188923?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-5-135188923-blog-133907371.235^v43^pc_blog_bottom_relevance_base7&spm=1001.2101.3001.4242.4&utm_relevant_index=8

https://github.com/u-boot/u-boot/blob/master/doc/board/broadcom/raspberrypi.rst





# 使用
<!-- find -name *_defconfig | xargs grep CONFIG_ARM64=y -->
find -name *_defconfig


find -name rpi_arm64_defconfig

find -name  rpi_arm64_3_b_plus_defconfig

find -name  config.h

# 准备环境
apt-get install bison  
apt-get install flex    
apt install uuid-dev 
apt install libgnutls28-dev

make distclean
make clean
bcm2710-rpi-3-b-plus

export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-gnu-

make rpi_arm64_defconfig
make rpi_arm64_3_b_plus_defconfig
make -j 12

make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-   -j$(nproc)

打包结束



docker run -it --rm --name ubuntu_arm_cmake_uboot_container \
  --device /dev/bus/usb:/dev/bus/usb \
  --privileged \
  ubuntu_arm_cmake_uboot:v0.0.1 /bin/bash




enable_uart=1
arm_64bit=1
kernel=u-boot.bin