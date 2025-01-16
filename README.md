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



