# U-Boot 移植修改方案

# 以ATMEL AT91SAM9260为例，U-boot版本1.3.4


## 一、添加开机LOGO

### 1. 说明
开机logo的运行时间从上电后bootloader启动开始，直到linux系统启动完成，linux控制程序接管LCD显示为止。

### 2. 移植

1. 首先，在board/atmel/procee_name/文件夹中增加一个开机LOGO使用的文件如logo.c。文件中实现了的对LCD的操作函数，画开机LOGO函数等，LCD初始化等函数，写数据写寄存器等命令。

2. 可以调用：asm/arch/gpio.h 中的相关操作函数来操作gpio，这些函数和内核中是一样的；

3. 注意LCD DB的地址，要根据不同的地址查看 datasheet来启动不同的NCS片选来使能该地址；

4. 修改board/atmel/process_name/Makefile COBJS-y  += logo.o 使Uboot的Makefile编译新添加的logo.c

5. 在board/atmel/process_name/process_name.c 中的board_init（）函数的末尾调用在logo.c中实现的画开机LOGO函数。 调用流程：start.s ---> lib_arm/board.c(start_armboot() ---> init_fnc_ptr ---> board_init())

## 二、网口自动协商超时时间修改

在开发的过程中，需要u-boot支持网络功能，好使用tftp 等工具下载内核和文件系统 ，在最终的产品中就可以取消这个功能，以在没有网络连接的时候快速引导启动系统。

	drivers/net/macb.c  
	#define CFG_MACB_AUTONEG_TIMEOUT   
	
这个宏控制这个超时时间，可以修改的短一些，这样在 macb_phy_reset 中就可以快速超时了；

## 三、不同版本的编译

有时，一个bootloader需要同时在不同的硬件平台上运行，可以通过修改u-boot 顶层目录中的config.mk 文件中的：PLATFORM_CPPFLAGS 来确定编译出来的u-boot版本；

GZ_VERSION 将会把board/atmel/at91sam9260ek/lcd_1602.c编译到u-boot中，并在启动的时候调用 lcd_1602_drwa_logo函数来显示开机画面;

NM_VERSION 将会把board/atmel/at91sam9260ek/lcd_19264.c编译到u-boot中，并在启动的时候调用 lcd_19264_drwa_logo函数来显示开机画面;

## 四、裁剪掉不需要的功能和命令

Uboot中有许多的命令和功能在实际的应用中用不到可以通过修改”include/configs/process_name.h”中的相关的宏，来实现功能定制裁剪：
         
     #undef     CONFIG_XXXX_XXX

## 五、修改默认的启动配置参数减少配置工作

Uboot中的许多参数都是可以通过setenv 或 set命令进行设置的，设置好之后通过save 或 saveenv命令将修改好的参数保存到nandflash中。然而在实际的产品开发中的，最终的参数不会经常改变，而且为了方便起见，一般都希望在编译出来之后直接就可以使用，而不需要再设置相关的启动参数等。

可以通过修改”include/configs/process_name.h”中的一些宏定义来实现编译默认参数:

     CONFIG_BOOTDELAY         启动后等待按键终止启动的时间（最终的产品中不需要）
     CONFIG_BOOTCOMMAND     	启动命令（也可通过setenv bootcmd设置）
     CONFIG_BOOTARGS          传递给内核的启动参数，也可通过setenv bootargs设置

CONFIG_BOOTCOMMAND 和 CONFIG_BOOTARGS宏在头文件”include/configs/process_name.h”中有多个要注意区分这些环境变量是保存在NANDFlash上的还是在DATAFLASH CS0 或 CS1上的

## 六、基本移植过程（以AT91SAM9260EK为例）

1. 在顶层目录Makefile中修改交叉编译链；
2. 修改使启动参数等环境变量保存在NANDFLASH中，设置默认的启动参数环境变量：
         
         include/configs/at91sam9260ek.h
         #undef以下两个宏定义就可以使启动参数和环境变量保存在NANDFLASH中
         #undef     CFG_USE_DATAFLASH_CS0
         #undef     CFG_USE_DATAFLASH_CS1
   
参考标题5：修改默认的启动配置参数减少工作

