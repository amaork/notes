# AT91SAM9G20 嵌入式Linux启动速度优化

## 一、说明

### 1. 三级跳转启动
ATMEL AT91系列SOC默认提供的是三级跳转启动：

1. 第一级:RomBoot，romboot是SOC上默认提供的它在上电之后一次开始检测外存中是否有引导程序，如果有引导程序则将外存中的引导程序加载到片内的RAM中执行；
2. 第二级：是由RomBoot加载到片内RAM的bootstrap，由于受限于片内RAM的大小，所以bootstrap不能太大尺寸也就不能执行更多复杂的操作，所以bootstrap的作用就是对外设做简单的初始化，然后从外存中加载一个更大的bootlader到到SDRAM中，然后跳转到sdram中执行；
3. 第三级：是由bootstrap加载到SDRAM中的Bootloader，通常是U-boot，u-boot是在内存中执行的，大小不受限制然后就在u-boot中将内核加载到内存中然后引导内核挂载文件系统

### 2.启动流程
一下的顺序是整个系统的启动流程，优化启动速度主要从以下的各个环节考虑

		ROMBoot ---> Bootstrap ---> Bootloader ---> Kernel ---> Application

### 3. 优化设计

优化 Linux 启动主要从以下几个方面考虑：

- Bootloader 
- Kernel 
- Filesystem 启动脚本

AT91SAM9G20 的片内 RAM 为16K，也就说说 Bootstrap 的尺寸只要在16K大小之内就可以了，16K 的程序可以做许多工作了。甚至，可以将bootloader 的功能整合到 bootstrap 中，然后直接由 bootstrap 直接引导内核，那么系统的启动流程就成为了：

		ROMBoot ---> Bootstrap ---> Kernel ---> Application

这样减少了Bootloader对系统的启动速度有一定的帮助，u-boot 的启动初始化了许多外设使启动时间进一步减少。通过测量启动时间，知道ROMboot + Boostrap 的启动花费了 5 秒左右，而其中的3秒是 Bootstrap 启动时间，内核启动 + 文件系统挂载花费了 3.5 秒左右，以下的优化主要从以下三个方面着手，通过分别缩减 Bootstrap 和内核的启动。


## 二、Bootstrap优化

Bootstrap 的主要工作是初始化 SDRAM 和主时钟以及必要的外设，调用用户自定义的硬件初始化程序（用户初始化程序，一般用来做在LCD上绘制启动Logo）然后将内核从 NAND Flash 搬运到SDRAM中，同时设置好内核启动参数然后，跳转到内存中内核的启动位置开始启动内核。优化主要从以下方面着手：

1. 减少内核的尺寸，从而减少从 NAND FLASH 搬运的内核到 SDRAM 的时间（这一步和内核的优化是相辅相成的）
2. 去掉不必要的用户初始化

## 三、Kernel启动优化


### 1、使用未压缩的内核
在 Bootstrap 中将内核从 NAND FLASH 搬运到 SDRAM 中指定的位置之后，然后就跳转到内存中内核所在的位置开始执行。内核分两种一种是没有压缩的内核（arch/arm/boot/Image），这种内核就直接开始执行，另一种是压缩的内核，需要首先解压缩之后才能执行。这两种内核的利弊情况是：

- 未压缩的内核比较大，大概是压缩内核的 2 倍大小，这样往内存中搬运的时候搬运的时间就比较长，但内核可以直接启动不需要再解压缩
- 压缩的内核比较小，从NAND FLASH 往内存中搬运的比较短，但内核启动前需要解压缩，解压缩的速度应该和 CPU 的运算能力有关

根据以上的分析情况来看，从NAND FLASH 向 SDRAM 拷贝东西的速度大概是 **4M/s**,由于是 ARM 架构，解压缩内核的速度不是很快，因此就考虑使用未未压缩内核。

### 2、裁剪内核去掉不必要的模块和特性

裁剪内核，去掉不必要的模块，以及用不上的特性以减少内核尺寸和启动时初始化的流程。例如不使用 USB 接口，不需要 FAT32, NTFS 文件系统支持等等。一些可能在调试的时候用得上的功能可以以内核模块的形式编译，例如： nfs clinet, 在调试的时候可以用来挂载主机的文件系统然后方便调试程序，但在最终发布的产品的时候，这个功能就不是必须的。因此可以将其编译为模块放在文件系统中，等需要的时候在通过 `modprobe` 加载到内核中，从而减少内核的尺寸和启动初始化的流程。

	勾选上 CONFIG_EMBEDDED 

### 3、预置 LPJ

在启动参数中使用一个预设的 loops_per_jiffy 以免每次系统启动都需要进行校验计算

	lpj=989184

### 4.禁用终端输入 

在内核的启动参数中，将内核的启动模式设置为 quiet，禁止通过向终端打印内容以减少启动时间
	
	quiet


	mem=64M console=ttyS0,115200 mtdparts=atmel_nand:1M(Bootstrap),2M(Kernel),4M(Rootfs),4M(Appliaction),-(UserData) root=1f02 ro rootfstype=cramfs quiet lpj=989184

## 四、文件系统优化

### 1、启动脚本优化

内核启动完成之后就会根据内核启动参数寻找根文件系统挂载，文件系统挂载成功之后，就会根据 inittab 寻找初始化脚本脚本执行，等初始化脚本执行完成之后就进入了终端。跟文件系统通常是只读的，由内核挂载。用户的应用程序和工具之类的存在用其他的可读可写分区中，通常在启动脚本将系统初始化到合适的状态之后，再挂载用户分区，然后启动应用程序。

启动脚本优化的核心在于：将一系列不必要的操作都放在启动应用程序之后，必要的操作完成之后就开始挂载用户的可读写文件系统，然后启动应用程序。等应用程序启动完成之后，后续的初始化操作继续。

例如：

先挂载 dev proc 文件系统，启动`mdev`， 安装设备驱动，挂载用户分区，启动用户程序，然后设置主机名称，开启 telnet 等等


### 2、文件系统挂载优化

Linux 支持多种可读可写的文件系统，有 JFFS2,YAFFS,UBIFS 等可选，有一组数据对比这三种文件系统的性能 [http://elinux.org/Flash_Filesystem_Benchmarks_2.6.36](http://elinux.org/Flash_Filesystem_Benchmarks_2.6.36 "http://elinux.org/Flash_Filesystem_Benchmarks_2.6.36")

	Test: mount_time (s)
	size ubifs   jffs2   yaffs2
	8	 0.14	 0.11	 0.03
	32	 0.36	 0.48	 0.34
	128	 0.43	 1.38	 0.51
	252	 0.33	 2.08	 0.96

根据需要最终选用 YAFFS2 作为用户的可读写分区，其挂载时间是最短的。




