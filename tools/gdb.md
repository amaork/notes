# GDB
---

## 一、概述

要想使用 gdb 调试程序，在编译的时候需要加上 `-g` 参数保留调试信息，否则 gdb 不能正常工作。可以在 `Makefile` 的 `CPPFLAGS`中添加 `-g` 参数。

- 启动

		gdb 要调试的程序

		或 

		先启动 gdb，然后使用 file 命令载入要调试的程序

- 退出

		启动 gdb 之后，可以使用 q 命令退出 gdb 调试

- 设置参数 

		方法1：

		启动 gdb 之后，将参数跟在 r 运行命令之后，例如：(gdb)r 参数1 参数2 ...
		
		方法2：

		使用 set args 命令，将参数跟在命令后，例如： (gdb)set args 参数1 参数2 ...

## 二、常用指令

	l			list 命令的缩写，可以列出当前调试程序的代码
	r			run 命令的缩写，开始运行程序，直到遇到断点才停止
	b			break 命令的缩写，用来添加断点，需要一个参数是要停止源码行数
	c			continue 命令的缩写，用来在程序遇到断点后继续执行，直到遇到下一个断点
	n			next 命令的缩写，用来顺序执行一行代码
	s			step 命令的缩写，用来单步跟踪代码的执行，通常用来跟踪函数的执行
	p			print 命令的缩写，用来打印一些变量的值
	info		查询信息后面可以跟随不同的参数，后面会详细的讲解
	file		用来加载要调试的可执行程序到 gdb
	make		用来执行 make 命令
	watch		用来监视一个变量，gdb 自动打印一个变量之前的 old value 和 new value
	shell		用来在 gdb 中执行 shell 命令，例如:（gdb）shell ls 
	ptype		指定一个成员类型，可以查看这个类型的成员和数据
	clear		删除指定行号上的断点
	delete		删除指定编号的断点
	enable		启用指定编号的断点
	disable		禁用指定编号的断点
	set args 	设置参数

### info 命令

info 名来用来查看 gdb 调试过程中的一些信息，常用的有一下几种

	info break		查看断点信息，如编号，行数，是否被禁用，hit 次数等
	info local		查看当前调试堆栈中的局部变量
	info variables	查看所有的全局变量和静态变量名称
	info program	查看当前调试程序的执行状态
	info function	显示所有的函数名称
	info thread		线程程序中所有的线程信息

### 断点相关命令

断点相关的命令有：break, clear, delete, enable, disable，一下为详细的说明：

	break	设置断点，需要指定一个行数，默认是指定当前文件下的行数。可以通过 break 文件名称:行数 在指定文件中设置断点，例如：

			b 100			在当前文件的第 100 行设置断点
			b file_a.c:100	在文件 file_a.c 的第100行设置断点

	clear 和 delete 指令都是用来删除断点的，两者的不同在于： clear 需要的参数是文件的行数，而 delete 需要的是断点的编号（断点的编号可以通过 info b 来查看）

	enable 和 disable 使用暂时的启用和禁用某个断点，这个命令需要的参数和 delete 一样，是指定断点的编号。设置完成之后可以通过 info b 命令来查看。


## 三、远程调试

在嵌入式软件开发中，软件通常是在 X86 平台上开发、编辑，然后使用交叉编译工具链，将源码编译成目标平台的可执行程序。最后，通过网络将目标程序复制到目标平台，在目标平台上运行。因为软件的开发和运行是在不同的平台下的，因此调试的时候就需要使用远程调试。在远程调试中，开发软件的平台被称为 **host** 通常是 x86 的 linux 主机，而运行目标代码的平台被称为 **target**，通常是 ARM、MIPS、Power PC 之类的嵌入式开发板。

远程调试由两部分组成：运行在 target 上交叉编译的 `gdbserver` 和运行在 host 上的交叉编译工具链中的 `gdb`， target 和 host之间由网络或串口连接，下面将以网络连接为例说明，如何进行远程调试。

1. 首先将交叉编译之后的可执行程序，复制到 target 中，或放在 target 可以访问的网络文件系统中；
2. 在 target 上启动 `gdbserver` 并指定要调试的程序，以及相关参数，如：
	
		gdbserver host_ip_addr:debug_port program_name

		host_ip_addr 是 host 的 IP 地址
		debug_port 是指定调试使用的端口号，可以随意指定，最好是大于 1024 的非保留端口
		program_name 是要调试的程序的名称（可以上相对路径也可以是绝对路径）

	gdbserver 成功启动之后会打印以下信息：

		Process program_name created; pid = dddd
		Listening on port xxxx

		xxxx 是指定的调试端口号

	
3.	在 host 上启动交叉编译链中的 `gdb`，注意是交叉编译工具链中带的 `gdb`，不是 host 系统中的 `gdb` ，如：

		arm-linux-gdb program_name

		program_name 和第二步中的 progam_name 是同一个可执行文件

4. 根据第二步中设置的参数，连接 `gdb` 到 `gdbserver`，例如：

		(gdb)target remote target_ip_addr:debug_port 

		target_ip_addr 是 target 的 IP 地址
		debug_port 和 target 在启动 gdbserver 设置的端口一样

	gdb 成功连接上 gdbserver 之后，在 target 的终端上会显示以下信息，表明已经成功连接：

		Remote debugging from host xxx.xxx.xxx.xxx 

		xxx.xxx.xxx.xxx 是 host 的 IP 地址


5. 开始调试，经过以上几步的设置就可以想调试普通程序一样进行远程调试了。

**注意**：

调试普通程序的时候，设置好断点之后，会使用 `run(r)` 来运行调试，但在远程调试中不需要这么做，直接使用 `continue(c)` 命令就可以了。

## 四、GDB + J-link 调试

以上讲的调试方法都是用来调试普通程序的，在嵌入式软件开发中，有时需要调试 bootloader 和 kernel，因为还没有操作系统的支持，因此需要使用 J-link 的配合来完成相关的调试。这是远程调试的一种，J-link 提供的 GDB Server 和运行在 host 上的 gdb 配合可以完成对底层软件的调试，一下是调试过程：

1. 首先需要在 host 上需要调试的目标程序目录下放一个名称叫 `.gdbinit` 的文件，这个文件用来在启动 gdb 之后执行相关的命令，下面就是一个 `.gdbinit` 文件的范本，其作用是在 gdb 连接 gdbserver 的时候，设置相关的参数给 J-link gdbserver.

		target remote gdbserver_ip:2331		# gdbserver_ip 是运行 J-link 主机的 IP 地址，2331 是 J-link 默认调试端口号
		
		monitor speed 30					# 设置通讯速度为 30K

		monitor endian little				# 根据需要设置 target CPU 的大小端（ARM 为小端）

		monitor	reg cpsr = 0xd3				# 设置 ARM 的 CPSR 寄存器，目的是让 ARM 进入 Thumb 之指令模式和 SVC 管理模式

		monitor speed auto					# 设置速度为自动 

		set remote memory-write-packet-size 1024 
		
		set remote memory-write-packet-size fixed

		load vmlinux						# 载入内核

		monitor reg pc = 0x23f00000			# 指定运行指针 PC 指向，内核的入口地址

2. 在 host 端启动 gdb 进行调试，启动 gdb 调试需要文件名，名称需要和  `.gdbinit` 指令中的 `load` 后跟随的文件名称一致，例如：

		arm-linux-gdb vmlinux

**注意：**

- J-link GDB Server 对 J-link 软件的版本有要求，有的版本软件不能正常连接，测试使用的版本是:`V4.06`

- 如果要调试 linux kernel 需要打开内核的调试选项： `CONFIG_DEBUG_INFO`
	
		Kernel hacking ---> Compile the kernel with debug info 