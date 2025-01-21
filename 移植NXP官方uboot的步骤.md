



1、 
    复制
    /u-boot/configs/mx6ull_14x14_evk_defconfig
    到
    /u-boot/configs/mx6ull_14x14_evk_kingnan_defconfig

    修改 

    # CONFIG_TARGET_MX6ULL_14X14_EVK=y
    CONFIG_TARGET_MX6ULL_14X14_EVK_KINGNAN=y


    # CONFIG_SYS_EXTRA_OPTIONS="IMX_CONFIG=board/freescale/mx6ullevk/imximage.cfg"
    CONFIG_SYS_EXTRA_OPTIONS="IMX_CONFIG=board/freescale/mx6ullevk_kingnan/imximage.cfg"


2、 添加头文件
    不同的 borad 需要配置的信息， 配置的信息在一个  头文件里配置， 每个板子 有一个；
    所以一个 一些 数字 和 宏 有关的东西，不能用 defconfig 里配置，所以就需要一个 .h 的头文件
    对于 NXP 官方的 6NULLEVK borad ，这个 头文件就是 gitlab/u-boot/include/configs/mx6ullevk.h
    所以复制这个文件到 gitlab/u-boot/include/configs/mx6ullevk_kingnan.h

    修改 mx6ullevk.h 里的内容， 修改成 mx6ullevk_kingnan.h 里的内容

    #define __MX6ULLEVK_KINGNAN_CONFIG_H
    #define __MX6ULLEVK_KINGNAN_CONFIG_H


3、 添加 borad 内的文件夹
    a: gitlab/u-boot/board/freescale/mx6ullevk 拷贝到 gitlab/u-boot/board/freescale/mx6ullevk_kingnan


        1、修改 mx6ullevk.c 到 mx6ullevk_kingnan.c

        2、修改 gitlab/u-boot/board/freescale/mx6ullevk_kingnan/Makefile 的     
            obj-y  := mx6ullevk.o
            修改为 
            obj-y := mx6ullevk_kingnan.o

        3、修改 imximage.cfg 中的 
                PLUGIN	board/freescale/mx6ullevk/plugin.bin 0x00907000
            修改为 
                PLUGIN	board/freescale/mx6ullevk_kingnan/plugin.bin 0x00907000

        4、修改 Kconfig
            TARGET_MX6ULL_14X14_EVK -> TARGET_MX6ULL_14X14_EVK_KINGNAN

            mx6ullevk -> mx6ullevk_kingnan

        5、修改 MAINTAINERS
            1、 F:	board/freescale/mx6ullevk/ -> board/freescale/mx6ullevk_kingnan/

            2、 F:	include/configs/mx6ullevk.h -> include/configs/mx6ullevk_kingnan.h

            3、F:	configs/mx6ull_14x14_evk_defconfig -> configs/mx6ull_14x14_evk_kingnan_defconfig


4、 修改 uboot 的 图形配置 文件
    <!-- gitlab/u-boot/arch/arm/cpu/armv7/mx6/Kconfig ->  -->

    gitlab/u-boot/arch/arm/mach-imx/mx6/Kconfig
        1、

            config TARGET_MX6ULL_14X14_EVK_KINGNAN
                bool "Support TARGET_MX6ULL_14X14_EVK_KINGNAN"
                select BOARD_LATE_INIT
                select DM
                select DM_THERMAL
                select MX6ULL
                imply CMD_DM


            source "board/freescale/mx6ullevk_kingnan/Kconfig"











