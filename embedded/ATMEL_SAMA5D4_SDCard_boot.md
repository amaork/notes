# ATMEL SAMA5D4 SD 卡启动盘制作

### 步骤0：如果 SD 卡已挂载，首先卸载

	df -h 
	sudo umount /media/xxxxx

### 步骤1：删除 SD 卡的 MBR

	sudo dd if=/dev/zero of=/dev/sdx bs=10M count=1

### 步骤2：使用 `fdisk` 将 SD 卡分区

根据需要将 SD 卡至少分成两个区，第一个区使用 `fat16` 格式，保存 `at91bootstrap`、`uboot`、`kernel`、`device tree`、 等数据
第二个分区格式化为 `ext2` 保存 linux 文件系统，可以根据需要在设置第三个分区保存用户的数据

	sudo fdisk /dev/sdx
	

	/dev/sdc1            2048       67583       32768    6  FAT16
	/dev/sdc2           67584      133119       32768   83  Linux
	/dev/sdc3          133120    15523839     7695360   83  Linux

### 步骤3：格式化各个分区

	FAT16

	sudo mkfs.msdos /dev/sdx1
	sudo mkfs.ext4  /dev/sdx2
	sudo mkfs.ext4	/dev/sdx3

### 步骤4：复制启动文件到 `fat16` 分区

复制启动文件的时候需要按照约定的命名规则，文件不能乱放的：

	at91bootstrap	:	BOOT.BIN
	uboot			:	u-boot.bin	
	kernel			:	zImage
	device tree		:	at91-sama5d4_xplained.dtb	

### 步骤5：复制文件系统到 第二个分区：

	mount -t /dev/sdx2 /mnt
	sudo cp rootfs/* -a /mnt



