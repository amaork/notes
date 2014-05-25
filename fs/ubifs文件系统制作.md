# UBIFS 文件系统制作

1. 下载mtd-utils  git clone git://git.infradead.org/mtd-utils.git （要正常使用首先要安装git-svn）
2. 安装需要的库，在mkfs.ubifs目录中生成mkfs.ubifs，在主机上运行
3. 修改源码目录中的common.mk,在其第一行中定义变量  CROSS=arm-none-linux-gnueabi-（ARM-Linux交叉编译工具链）
4. 在mtd-utils顶层目录中以交叉编译链为名称命名的文件夹中生成flash_eraseall 等工具，在ubi-utils文件夹中生成 ubiattach、ubiformat、ubidetach、ubimkvol、ubirmvol、等一系列相关工具。（注意在作这步之前要修改顶层目录中的Makefile把SUBDIRS = ubi-utils mkfs.ubifs中的mkfs.ubifs去掉，因为mkfs.ubifs是在主机上运行的不需要交叉编译）

主要工具是 ubiattach 和 ubimkvol
	
	ubiattach <UBI control device node file name> [-m <MTD device number>]  
	ubiattach /dev/ubi_ctrl -m 2（在mtdblock2上创建ubi0 2 指的是mtdblock2）
	ubimkvol <UBI device node file name> [-h] [-a <alignment>] [-n <volume ID>] [-N 	<name>]  [-s <bytes>]  
	ubimkvol /dev/ubi0 -N user -s 200MiB (-N 指定该分区的名称name, -s 分区的大小，注意分区不能完全用完所有的空间，要给UBI的日志预留部分空间)

挂载
	
	 mount -t ubifs ubi0_0 /home/user

第一次使用ubiattach的时候要首先使用mtd-utils中的flash_eraseall工具擦除，要挂载的MTD分区
完整的使用过程为：
	
	flash_eraseall  /dev/mtd2
	ubiattach /dev/ubi_ctrl -m 2
	ubimkvol /dev/ubi0 -N user -s 230MiB
	mount -t ubifs ubi0_0 /home/use

使用mtd工具可以给文件系统升级,如下假设要升级的文件系统在mtd1上，升级文件为"firmware.bin"：

	flash_earseall /dev/mtd1
	flashcp  firmware.bin /dev/mtd1 (注意flashcp 最好是在一个没有被擦除的分区上，即不能是在mtd1上否则可能会出错)
