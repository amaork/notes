# Linux Device Drivers
---

## 一、设备驱动程序简介

- 尽量做到驱动程序不不带策略

- 和驱动程序一起提供一个客户程序库，它提供哪些不必在驱动程序中实现的功能（对驱动程序的封装）

- 驱动程序编写者应当尽量避免在代码中实现安全策略。安全策略的问题最好在系统管理员的控制之下，由内核高层实现。 

- 任何从用户得到的输入只有经过内核的严格验证之后才能使用。

- 任何从内核中得到的内存，必须在提供给用户或者设备之前清零或者以其他方式初始化，否则就可能发生信息泄露。

- 最后还要考虑设备操作可能造成的影响。 

## 二、构造和运行模块

#### P22

`module_init` 和 `module_exit` 使用了内核的特殊宏来表示上述两个函数扮演的角色。
 
`MODULE_LICENSE` 用来告诉内核，该模块采用自由许可证；如果没有这样的声明，内核在装载模块的时候会产生抱怨。

#### P24

模块被连接到内核，只能调用由内核导出的那些函数，而不存在任何的函数库，大多数内核头文件保存在 `include\linux` 和 `include\asm` 目录

#### P26

Linux 内核代码，必须是可重入的，它必须能够同时运行在多个上下文中。因此，内核数据结构需要仔细设计才能保证多个线程分开执行，访问共享数据的代码也必须避免破坏共享数据。

#### P27

内核代码可以通过一个访问全局项来   


头文件

	<linux/module.h>		包含可装载模块需要的大量的符号和函数的定义

	<linux/init.h>			指定初始化和清除函数
	
	<linux/moduleparam.h>	在装载模块时向模块传递参数

	<linux/sched.h>			包含驱动程序使用的大部分内核 API 定义，包括睡眠以及各种变量声明

	<linux/version.h>		包含构造内核版本信息的头文件

	<linux/kernel.h>		printk 声明相关 

	<linux/types.h>			设备编号 dev_t
	
	<linux/fs.h>			register_chrdev_region struct file_operations struct file  

	<linux/cdev.h>			cdev_init cdev_add cdev_del

	<linux/slab.h>			kmalloc kfree

	<linux/uaccess.h>		copy_to_user copy_from_user 

	<linux/proc_fs.h>		在 /proc 文件系统创建文件相关

	<asm/semaphore.h>		信号量相关
	
宏

	MODULE_AUTHOR			模块作者
	MODULE_DESCRIPTION		模块用途的减短说明
	MODULE_VERSION			代码修订号
	MODULE_DEVICE_TABLE		用来告诉用户空间模块所支持的设备，常见于 pci 和 usb 设备驱动


#### 初始化和关闭
	
##### 初始化例子：

	static int __init init_function(void)
	{
		/* do some thing */
	}

	module_init(init_function);

`__init` 对内核来讲是一种暗示，表明函数仅在初始化期间使用（被放在 `.init` 段，这个段在初始化之后就可以释放了）。 `module_init` 的使用是强制性的，这个宏会在目标代码中增加一个特殊的段，用于说明内核初始化函数所在的位置。没有这个定义，初始化函数永远不会被调用。

##### 清除例子：

	static void __exit cleanup_function(void)
	{
		/* Do some thing */
	}

	module_exit(cleanup_function);

如果模块被直接嵌入内核，或者内核的配置不允许卸载模块，则标记为 `__exit` 的函数将被简单的丢弃。如果一个模块未定义清除函数，则内核不允许卸载该模块。


##### 错误处理

如果在注册设施的时遇到任何错误，首先要判断模块是否可以继续初始化。通常，在某个注册失败后可以通过降低功能来继续运转。因此，只要可能，模块应该继续向前并尽可能提供其功能。

如果在发生某个特定类型的错误之后无法继续装载模块，则要将在出错之前的任何注册工作撤销掉，模块必须自行撤销自己注册的设施。
