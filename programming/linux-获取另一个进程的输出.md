# Linux 获取另一个进程的输出

有的时候在程序中需要获取另外一个进程的输出结果，而这个进程通常是外部的的shell命令。通常的做法是调用`system`将结果输出到一个文件中，然后再从文件中读取，现在有更简洁的方法：

	FILE *popen(const char *cmdstring, const char *type);
		
	popen将fork一个进程执行cmdstring中的外部命令，然后返回一个标准I/O文件。
	这个文件作为cmdstring的标准输入或标准输出（取决于popen的第二个参数："r"标准输出，"w"标准输入）。

	例如：可以使用这个执行pidof来获取指定进程的PID




