# Linux kernel Oops信息详解

Oops信息

	[<c001a6f4>] (function_1 + 0x0/0x560) from [<c01bf4e8>] (call_function + 0x20/0x24)
 
	c001a6f4 的含义是：function_1 函数的首地址偏移0x0的地址这个函数的大小为0x560
	c01bf4e8 的含义是：call_function 函数的首地址偏移0x20的地址，这个函数的大小为0x24，同时这个地址还是function_1执行之后的返回地址，即call_function在这个地址调用了function_1
 
	依次类推可以找到整个调用关系；
 
	PC is at function_oops + 0x18/0x340 表示出错的指令在function_oops的0x18地址的地方，这个函数的大小为0x340
 
	pc:[<c001a70c>] 表示出错指令的地址，可以使用反汇编的方法定位代码
 
	arm-linux-objdump -D vmlinux > vmlinux.dis