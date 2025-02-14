# BootLoader
BootLoader


# 可以打印出 make 的规则
make -p -> 1.txt

#删除所有  # 开头的 
:g/^#/d


make stm32mp15_trusted_defconfg
make mx6ull_14x14_evk_defconfig
# raspberry pi 3 b plus
make rpi_3_b_plus_defconfig

```
打印 
  HOSTCC  scripts/basic/fixdep
  HOSTCC  scripts/kconfig/conf.o
  YACC    scripts/kconfig/zconf.tab.c
  LEX     scripts/kconfig/zconf.lex.c
  HOSTCC  scripts/kconfig/zconf.tab.o
  HOSTLD  scripts/kconfig/conf      制作工具 ：  生成  conf 文件， 似乎是配置 工具


# configuration written to .config  把配置信息写入 .config 文件
```

vi .config  # .config 在 根目录下

make distclean


find -name  mx6ull_14x14_evk_defconfig

diff .config ./configs/mx6ull_14x14_evk_defconfig





# 查看 编译过程 
make mx6ull_14x14_evk_defconfig -p -> 1.txt
vi 1.txt 
# mx6ull_14x14_evk_defconfig 依赖于 scripts/kconfig/conf
mx6ull_14x14_evk_defconfig: scripts/kconfig/conf
    $(Q)$< $(silent) --defconfig=generated_defconfig $(Kconfig)
    就是 
    scripts/kconfig/conf --defconfig=arch/../config/mx6ull_14x14_evk_defconfig Kconfig


# scripts/kconfig/conf 依赖于 scripts/kconfig/conf.o scripts/kconfig/zconf.tab.o
scripts/kconfig/conf: FORCE scripts/kconfig/conf.o scripts/kconfig/zconf.tab.o

# scripts/kconfig/conf.o 由  scripts/kconfig/conf.c  生成
scripts/kconfig/conf.o: scripts/kconfig/conf.c FORCE

# scripts/kconfig/zconf.tab.o 由 scripts/kconfig/zconf.tab.c 生成
scripts/kconfig/zconf.tab.o: scripts/kconfig/zconf.tab.c FORCE

最终把他们链接在一起



配置过程
1、 conf_parse(name); // 解析uboot根目录下的 kconfig 文件; name = Kconfig
2、 conf_read(defconfig_file) 读取配置文件
3、 conf_set_all_new_symbols(def_default); // 设置 new_symbols 为默认
4、 conf_write_all(); // 写入配置文件 .config




.config 文件 里的内容 一部分来自于 Kconfig 文件， 取何值决定于 defconfig 文件

CONFIG_ARM=y # Arm 架构；

ARM架构的 更多细节 在 Kconfig 里有；
Kconfig 中可能会有 ，如果使用 ARM 架构， 则会包含 Kconfig_arm 文件， 里面会有更多的配置选项；



Kconfig ：这是一个通用文件 里面规定一些依赖
  1、 如果是 ARM 架构 ，就默认配置 A B C 
  2、 如果是 ARM64 架构， 就默认配置 D E F
defconfig_file： 是 厂家提供的， 里面定义了
  1、ARM 架构
  2、自己一些配置项

如何处理：
  defconfig_file：直接写入 .config
  使用 defconfig_file：直接写入 的 内容区解析 Kconfig ， 把各个依赖的配置项 也 写入 .config
  其他未涉及的配置项 ，给他指定一个 默认值



# 打开 menuconfig 窗口
make menuconfig




在 /u-boot/drivers/Kconfig 中添加

 config kingnan
	bool "kingnan for. uboot"
	default y
	help
	  kingnan test for uboot

会出现在 menuconfig 中
可以控制 Y N 来配置 .config 文件 中是否包含 CONFIG_kingnan=y




config kingnan
	bool "kingnan for. uboot"
	default y
	help
	  kingnan test for uboot



menu "kingnan menu"

config kingnan_menu_1
	bool "kingnan_menu_1 for uboot"
	help
	  kingnan_menu_1 test for uboot

config kingnan_menu_2
	bool "kingnan_menu_2 for uboot"
	help
	  kingnan_menu_2 test for uboot

endmenu



config CHOICE_ON_KINGNAN_TEST
	bool "CHOICE_ON_KINGNAN_TEST for. uboot"
	default y
	help
	  CHOICE_ON_KINGNAN_TEST test for uboot

choice 
	prompt "kingnan choice test"
	depends on CHOICE_ON_KINGNAN_TEST

config kingnan_choice_1
	bool "kingnan_choice_1"
	help
	  kingnan_choice_1 test for uboot

config kingnan_choice_2
	bool "kingnan_choice_2"
	help
	  kingnan_choice_2 test for uboot

endchoice

menuconfig kingnan_menuconfig

	bool "kingnan_menuconfig for uboot"


if kingnan_menuconfig

config kingnan_menuconfig_1
	bool "kingnan_menuconfig_1"
	help
	  kingnan_menuconfig_1 test for uboot

config kingnan_menuconfig_2
	bool "kingnan_menuconfig_2"
	help
	  kingnan_menuconfig_2 test for uboot
endif




config A
	bool "A for. uboot"
	default n
	help
	  kingnan A for uboot



config B
	bool "B for. uboot"
	default y if B
	help
	  kingnan B for uboot



# 安装 工具链

wget https://releases.linaro.org/components/toolchain/binaries/latest-7/arm-linux-gnueabihf/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz

tar -xvf gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz -C /opt
ln -s /opt/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin /opt/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin

export PATH=/home/kingnan/TEMP/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin:$PATH
arm-linux-gnueabihf-gcc --version



如果需要 64 位支持
	wget https://releases.linaro.org/components/toolchain/binaries/latest-7/aarch64-linux-gnu/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz

	export PATH=$PATH:/home/kingnan/TEMP/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin:$PATH
	aarch64-linux-gnu-gcc --version





# 编译
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- mx6ull_14x14_evk_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf-


make CROSS_COMPILE=arm-linux-gnueabihf-gcc- mx6ull_14x14_evk_defconfig
make CROSS_COMPILE=arm-linux-gnueabihf-gcc- rpi_3_b_plus_defconfig

arm-linux-gnueabihf-gcc



export CROSS_COMPILE=arm-linux-gnueabihf-
export ARCH=arm

# make 的 过程
  1、检查 更新 头文件， 比如 include/config.h u-boot.cfg（不知道什么用途）
  2、制作 工具
  3、交叉编译
    编译那些目录


在 gitlab/u-boot/drivers/Makefile 文件中 
如果 obj-$(CONFIG_$(PHASE_)GPIO) += gpio/ 
  如果 CONFIG_GPIO=y ，则 obj-$(CONFIG_$(PHASE_)GPIO) += gpio/ 会被添加到 obj-y 中
  如果 CONFIG_GPIO=n ，则 obj-$(CONFIG_$(PHASE_)GPIO) += gpio/ 不会被添加到 obj-y 中

在 gitlab/u-boot/drivers/gpio/Makefile 中
  obj-$(CONFIG_MXS_GPIO)	+= mxs_gpio.o
  当 CONFIG_MXS_GPIO=y 时， mxs_gpio.o 会被添加到 obj-y 中； obj-y+= mxs_gpio.o； 如果遇到 .o 文件， 则会编译成 .ko 文件；



# 文件系统 配置
在 gitlab/u-boot/fs/Makefile 文件中
  obj-$(CONFIG_FAT) += fat.o
obj-$(CONFIG_FS_FAT) += fat/





最终版本的 uboot 包含 


mx6ull_14x14_evk_defconfig：
	
uboot-dtb.bin 
	uboot-nodtb.bin
		uboot ( 将 head-y 与 子目录 链接 起来， 生成 uboot)
			head-y
				arch/arm/cpu/armv7/start.o
			子目录
				lib
				fs
				net
				dirvers/
				dirvers/gpio/
				dirvers/serial/
				dirvers/i2c/
				dirvers/usb/


	dt.dtb(设备树， 由 两个决定 ： 芯片相关、 板厂相关)
		芯片相关： mx6ull.dtsi
		板厂相关： mx6ull_14x14_evk.dts
			设备树编译： dtc -I dts -O dtb -o mx6ull_14x14_evk.dtb mx6ull_14x14_evk.dts
			设备树查看： dtc -I dtb -O dts -o mx6ull_14x14_evk.dts mx6ull_14x14_evk.dtb
		find -name imx6ull-14x14-evk.dts 查找设备树


最后要对  uboot-dtb.bin   进一步处理， 生成 u-boot.bin

对于 imx6ull 芯片 ，要加上 cfg( DDR 相关的配置文件 ) = u-boot-dtb.imx 文件

对于 stm32mp157 芯片 ，要加上 头部信息 = u-boot.stm32 文件




在 







# XIP 概念
execute in place ； 在芯片内部执行；

flash 看上去是在 芯片内部执行，实际上 在 cpu 读取了指令 ，在 cpu 内部执行； xip 设备 是可以 直接 被 cpu 读取指令执行的；

Arm cpu： 一上电 从 0 取值运行， 读取指令 在 cpu内执行，
	外设 一定可以接收 cpu 发送的 地址 0 信号， 返回数据给 cpu， cpu 执行指令

如果 soc内 有 Nand Flash（控制器）， 并且 Nand flash 外挂载 一个 32G 的 Nand flash ，则 cpu 通过 中间的 Nand flash 控制器， 读取 32G Nand flash 的数据， 但是 cpu 只能 发送数据给 Nand flash 控制器；
但是 cpu 需要 通过复杂的指令才能发送数据给 Nand flash 控制器，控制  Nand flash 控制器 读到 flash 的第一条 指令， 返回给 cpu， cpu 执行指令；
所以 cpu 得到第一条 指令 来自于  ROM， 使用 BootRom 里面的 程序 发送 复杂指令给  Nand Flash（控制器）， 读取 32G Nand flash 的数据， 返回给 cpu， cpu 执行指令；


芯片有  bootpin 根据引脚 决定 从哪里启动， 从 ROM 启动， 从 Nand Flash 启动， 从 SD 卡启动， 从 eMMC 启动， 从 USB 启动；
零一种方式 bootpin 控制 启动顺序  ， 1 ROM ， 2 Nand Flash ， 3 SD ，4 eMMC ， 5 USB ； 只控制 顺序 1、2、3、4、5 或者 5、4、3、2、1












# uboot  dirve model 驱动模型


U_BOOT_DRIVER(ddr_driver) = {
    .name   = "ddr_driver",
    .id     = UCLASS_RAM,  // 或自定义类别
    .of_match = ddr_ids,
    .probe  = ddr_probe,
    .priv_auto_alloc_size = sizeof(struct ddr_priv),
};

of_match：使用设备树的兼容性字符串来匹配设备树中的节点。确保您的设备树中有对应的 DDR 设备节点，例如 compatible = "myvendor,my_ddr";。

probe 函数中，驱动程序会初始化设备并为设备分配资源。具体来说，它可能会映射设备的控制寄存器的内存地址，配置硬件，设置中断，以及其他相关的初始化任务。



u-boot 的 dm 树构建分三次构建，前两次使用uboot-dts构建， 最后一次使用 kernel 的 dtb 构建设备树；

第一次构建在  board_init_f() 函数中，使用 uboot-dts 构建设备树；在 uboot 重定位之前， 代码在 /u-boot/common/board_f.c 中，代码主要是执行 initcall_run_list(init_sequence_f) ， init_sequence_f是一个数组，它里面都是函数名，这些函数名在编译的时候，会被链接到一起，生成一个数组，这个数组在 uboot 重定位之前，会被执行；执行完成后 uboot 重定位之前的 初始化执行完了
	init_sequence_f 
		1、 fdtdec_setup 
		2、	CONFIG_OF_SEPARATE // uboot 镜像的 封装格式 是不是 dtb 在 后面
		3、 CONFIG_SPL 是否由 SPL 的部分
		4、	把 end 地址复制给 fdt blob 全局变量 的值，，end 是 uboot 结束地址， fdt blob 全局变量 是 uboot 的 dtb 的起始地址
		5、 lds 镜像的 清单， 
		6、 end 是 uboot 结束地址， fdt blob 全局变量 是 uboot 的 dtb 的起始地址
		7、 initf_dm  加载树
			a: dm_init_and_scan // 初始化 dm 树，扫描设备树，构建 dm 树; 构建树的根节点，根节点是 gd->dm_root_f
				dm_init； 创建根节点
				最终放在  DM_ROOT_NON_CONST  (((gd_t *)gd)->dm_root) 中
				DM_UCLASS_ROOT_NON_CONST 是 比如说 都是 usb 设备那么时候 同一个 usb 根节点 抽象出类
				设备树根节点的名字 是 root dirver

		8、 系统会选择 比较大的树，也是 重定向后的树， 也就是 gd->dm_root_r

		
			


第二次构建在  board_init_r() 函数中，使用 uboot-dts 构建设备树；在重定位之后执行

第三次构建在  board_init_r() 函数中，使用 kernel 的 dtb 构建设备树；
经过三次构建，整个 uboot 中由两颗设备驱动模型树，第一颗 被保存在 gd->dm_root_f , 第二颗 被保存在 gd->dm_root_r 中, 第三次构建的设备树被保存在 gd->dm_root 中；



spl 做的 是最基础硬件的初始化，时钟、内存 等，
其他的由 uboot 来完成

uboot 要把自己从 从 内存 的某个位置，拷贝到 0 地址，然后 执行；
剩下来的 空间安装 linux kernel，设备树，文件系统；仅为 linux 比较大，所以要储存的位置 尽可能 复制到 较远的位置 ， 高端
	编译的，生成 object  要拼装到一起，生成 可执行文件，链接的主要 过程就是 地址的 重运算， 这些地址 是 ： 函数的地址 、变量名的地址；通过函数名 找到函数的位置
	 所以把 kernel 复制到高端的时候 要做重定位，这个过程是 

``
























# 当前遇到的问题
1、 首先我要在 rpi 上运行 uboot ，能不能成功 
	a:尝试 编译 rpi 3b+ uboot ，但是 openssl 可能要升级到 11
	b: 使用 uart 连接 rpi，测试一下，看看是否可以成功

2、uboot 
	a: uboot 里面怎么写驱动，怎么测试，怎么调试
		1、如果 板子上添加下 DDR 那么 uboot 怎么添加驱动，去初始化 DDR
	b：uboot 如何写设备树
	c: 如何启动linux内核

3、linux 的 内核 如何编译
	a:内核编译
	b:设备树
		1、如果板子上有DDR 如何使用DDR，设备树如何写，内核如何使用设备树
	d:跟文件系统


4、板子上的外设
	a: DDR
	b: sd card
	c: eMMC
	d: type-c (全功能 Thunderbolt 4  \ USB4 )
	e: oled (可选)
	f: wifi
	g: blue tooth (可选)
	f: usb




# 安装 openss l.1.1
wget https://www.openssl.org/source/openssl-1.1.1u.tar.gz
# 配置安装选项
./config --prefix=/usr/local/openssl --openssldir=/usr/local/openssl shared zlib
 # 编译 
 make
# 安装
 sudo make install
export PATH=/usr/local/openssl/bin:$PATH
openssl version












#  rpi 没办法使用 32 位 ， 所以没用
32 位
arm-linux-gnueabihf-gcc --version
export PATH=/home/kingnan/TEMP/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin:$PATH
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- rpi_3_32b_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- V=1 
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- clean








64 位
aarch64-linux-gnu-gcc --version
export CROSS_COMPILE=aarch64-linux-gnu-
export ARCH=arm64


make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- distclean
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- rpi_3_b_plus_defconfig
make ARCH=arm CROSS_COMPILE=aarch64-linux-gnu- V=1 
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- clean


# 命令 查找 rpi 的相关 make 的 config
cd configs/
ls | grep rpi


# 尝试 burn rpi3b+ uboot 到 sd card
典型分区方案：
	引导分区：用于存储 U-Boot 和内核镜像。这通常是一个 FAT32 文件系统，这样 U-Boot 可以容易地读取文件。
	根文件系统分区：使用 ext4 等文件系统，存储完整的 Linux 系统内容； 根文件系统（通常是 ext4 或 FAT 文件系统）


1、列出 SD 卡 ： lsblk
2、sudo fdisk /dev/sdb  # 将 sdX 替换为您的 SD 卡设备名
3、创建分区：
	1、创建一个引导分区（例如 256MB，格式为 FAT32）
	2、创建其他分区以存放文件系统内容（例如 ext4）
4、格式化分区
	sudo mkfs.vfat -F 32 /dev/sdb1  # 将引导分区格式化为 FAT32
	sudo mkfs.ext4 /dev/sdb2  # 将根文件系统分区格式化为 ext4
5、sudo dd if=uboot.bin of=/dev/sdX bs=1M status=progress
6、如果  有 sdX1 那么 ：  sudo dd if=uboot.bin of=/dev/sdX1 bs=1M status=progress  # 烧录到引导分区
7、

sudo dd if=u-boot.bin of=/dev/sda1 bs=1M status=progress


sudo mount /dev/sdb1 /mnt/sdCard
sudo umount /mnt/sdCard

sudo minicom -D /dev/ttyUSB0 -b 115200
sudo cp config.txt /mnt/sdCard
sudo cp u-boot64.bin /mnt/sdCard

sudo cp config.txt u-boot64.bin start.elf bootcode.bin /mnt/sdCard

 sudo cp config.txt u-boot64.bin start.elf bootcode.bin fixup.dat /mnt/sdCard



 sd卡 分区 

 在fdisk提示符下，输入以下命令：

输入 o 创建一个新的空的DOS分区表（如果SD卡上已有分区表，可以跳过这一步）。
输入 p 打印当前的分区表（确认没有现有分区，或者你准备删除它们）。
输入 d 删除现有分区（如果有）。
输入 n 创建新的分区。
选择 p（主分区）。
输入 1 为第一个分区（编号）。
默认起始扇区，直接按回车。
输入 +512M 设置第一个分区大小为512MB。
输入 n 创建第二个分区。
选择 p（主分区）。
输入 2 为第二个分区（编号）。
默认起始扇区，直接按回车。
默认结束扇区（使用剩余空间）。
输入 p 查看分区表，确认两个分区是否正确。
输入 w 保存更改并退出fdisk。