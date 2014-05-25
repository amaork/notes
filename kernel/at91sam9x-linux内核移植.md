# AT91SAM9x内核移植文档

## 一、移植步骤

1. 修改有内核源码顶层目录的Makefile 193行左右 ARCH 和CROSS_COMPILE；
2. 复制arch/arm/configs/at91sam9260ek_defconfig 到内核源码的根目录为.config；
3. 配置剪裁需要的相关内核选项、文件系统、驱动等信息；
4. 修改内核MTD分区信息:

		arch/arm/mach-at91/board-sam9260ek.c 
		static struct mtd_partition __initdata ek_nand_partition[] 
		
		注意大小(.size)以K为单位例如3M即为:1024*1024*3

        分区信息：
        Bootload+Kernel  		0x0              -   
        FileSystem:         	0x30000     -    0x1000000 13M
        UserSpace:				0x1000000  -   0x10000000 240M
        
5. 根据需要打开内核内的串口（/dev/ttyS3）

		修改rch/arm/mach-at91/board-sam9260ek.c文件中的
		 __init ek_map_io函数，原始为2修改为3
		 
 		61     at91_register_uart(AT91SAM9260_ID_US0, 1, ATMEL_UART_CTS | ATMEL_UART_RTS
 		62                | ATMEL_UART_DTR | ATMEL_UART_DSR | ATMEL_UART_DCD
		63                | ATMEL_UART_RI);
		64
		65     /* USART1 on ttyS2. (Rx, Tx, RTS, CTS) */
 		66     at91_register_uart(AT91SAM9260_ID_US1, 2, ATMEL_UART_CTS | ATMEL_UART_RTS);
 		55     at91_register_uart(AT91SAM9260_ID_US0, 1, 0);
 		56
 		57     /* USART1 on ttyS2. (Rx, Tx, RTS, CTS) */
 		58     at91_register_uart(AT91SAM9260_ID_US1, 2, 0);
	
		并添加  at91_register_uart(AT91SAM9260_ID_US2, 3, 0);（对应/dev/ttyS3）

6. 修改默认启动选项（可以删除）

		├----- Boot options  --->
        ├----------- (mem=256M console=ttyS0,115200 root=/dev/mtdblock1 rootfstype=cramfs init=/linuxrc) Default kernel command string

7. 生成uImage
	
	(注意要在u-boot中使用bootm启动内核，并传递bootargs中的启动参数必须转换为uImage格式)在内核目录使用make uImage生成         mkimage这个工具在u-boot目录中的tools目录中
	
         zImage制作uImage:
          ./mkimage -A arm -O linux -T kernel -C none -a 0x30008000 -e 0x30008040 -n  'linux-2.4.18'   -d zImage Image
          -A 设置那种类型的CPU
          -O 设置那种类型的OS
          -T  设置映像image类型
          -C  是否压缩
          -a  设置装载点
          -e 设置进入点（内核的入口地址，信息头的大小是0X40，在制作uImage时，加的一个头，在使用bootm时会判别这个头）
          -n 设置image名称
         -d 指定zImage景象名称
         
## 二、问题及解决方案

1. uboot不能使用tftp下载内核；

	可以使用uboot自身带的工具通过“超级终端”进行文件的下载。使用方法如下：
	
		loadb  然后通过终端发送文件注意使用的协议为"kermit"
		loady  然后使用终端发送文件注意使用的协议为"Ymodem"

	以上两种传输方式任意选一，然后使用U-boot自带的nand flash 擦写工具把内核烧入flash中，注意在写之前必须先擦除

		nand erase start_address erase_length 
		nand read   dest_address(RAM)  src_address(NAND Flash)    target_size(read size)
		nand write  src_address(RAM)    dest_address(NAND Flash)  target_size(write size)

2. ARM RI寄存器中保存的机器类型不匹配（如果使用bootm等方式不使用go跳转调试则不用更改此选项,AT91SAM9260EK的平台码为1099）
Error: unrecognized/unsupported machine ID (r1 = 0x23e7ee9c).                                                              

		Available machine support:                           

		ID (hex)        NAME                     
		0000044b        Atmel AT91SAM9260-EK                                     

		Please check your kernel config and/or bootloader.       
  

	解决方法:我们知道uboot会在R1寄存器中保存机器的类型，而linux也会在内核中配置机器的类型，这两个要一致才可以，通常情况下linux内核中对机器代码的配置是存放在 arch/arm/tools/mach-types中的，经过查找后发现AT91SAM9260-EK 在linux中的机器码是1099而uboot中的机器码为0x23e7ee9c,这两个数字要一致，修改arch/arm/tools/mach-types 中的1099为602402460（0x23e7ee9c的十进制数值），然后重新编译内核即可；

3. 文件系统不能正常挂载不识别NAND Flash分区

	更换为2.6.30的内核后问题解决(原先是2.6.27的内核)出现:

		Kernel panic - not syncing: Attempted to kill init!
		Backtrace:

	原因是：交叉编译工具链，中打开了EABI的选项而在配置内核的时候没有打开这个选项，造成这个原因的，修改后问题解决；

		├───Kernel Features  --->
		├---------------- [*] Use the ARM EABI to compile the kernel                                                                  
       	├-----------------[*]   Allow old ABI binaries to run with this kernel (EXPERIMENTAL) 

4. NFS挂载时出现：failed: Protocol not supported（原因是没有选择详细的NFS版本）

		├───File systems  --->
      	├──────[*] Network File Systems  --->
     	├───────<*>   NFS client support                                                                                 
     	├────── [*]      NFS client support for NFS version 3 
      	├───────[*]      Root file system on NFS

5. NFS挂载失败
		
		rpcbind: server localhost not responding, timed out
 	
 	在挂载NFS的使用使用 -o nolock 选项即可
		
		mount -t nfs -o nolock 192.168.1.107:/home/wxidong/_install /tmp/nfs

6. 内核启动后不按照在u-boot中设置的bootargs中设置的文件系统挂载参数

	内核在启动的时候设置的bootcmd为 setenv bootcmd nand read 0x20007fc0 0x100000 0x200000\;go 0x20007fc0 (既：把内核从NAND Flash读取到内存中去，然后跳转到加载内核的内存地址去启动内核)

	这样在内核启动的时候不会读取在u-boot中设置的bootargs的文件系统的挂载信息，而是使用在内核编译的时候在Boot options  --->()  Default kernel command string中设置的boot参数，这样很不方便，经过查找后发现内核原因是使用了u-boot的go 命令跳转到地址去启动执行内核，应该使用bootm命令去启动执行内核，这样内核就会读取在u-boot和kernel之间256K的bootargs的参数，从而按照设置的bootargs来挂在文件系统；

	go和bootm差异就是 go只是改写pc值，而bootm传递r0，r1，r2还有bootargs ，go 命令只能启动zImage格式的内核,bootm命令只能启动uImage（u-boot/PPCBoot image）格式的内核；

	使用bootm启动需要把zImage转换为uImage使用的工具就是在编译u-boot后在tools文件夹生成的mkimage；

	<font color =#ff0000>注意：在生成uImage后会有一个Load Address和Enter Point在u-boot中的载入的地址的地址要比这个地址提前0x40字节（头）否则不能启动内核。如:Load Address是0x20008000那么u-boot中把内核读取到内存的载入地址和bootm地址都要为0x20007fc0；</font>

		setenv bootcmd nand read 0x20007fc0 0xa0000 0x200000;bootm 0x20007fc0


	<font color =#ff0000>注意：uboot的启动参数中：
	
		setenv bootcmd nand read 0x20007fc0 0xa0000 0x200000\;bootm 0x20007fc0
		setenv bootargs root=1f01 console=ttySAC0,115200 init=/linuxrc devfs=mount mem=64M（内存的大小不是NAND 
		
		Flash的大小，千万要注意，否则可能引起内核崩溃,内核在划分映射空间的时候，会过多的分配内存）

	一定注意要配置MTD驱动中的NAND FLASH中的ECC校验为Software Ecc 否则可能会引起文件系统中的部分软件不能正常工作，在使用reboot重启的时候会出现cramfs 有错误之类的信息提示，并且在系统启动之后软件正常运行单没有界面的显示等
