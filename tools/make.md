# Make
---

## 一、变量

- Make中的自动变量

		$*	不含扩展名的目标文件名称
		$+	所有的依赖文件，以空格分隔，并以出现的先后顺序排列
		$<	第一个依赖文件的名称
		$?	所有时间戳比目标文件晚的依赖文件
		$@	目标文件的名称
		$^	所有不重复的依赖文件，以空格分隔
		$%	如果目标是归档成员，则该变量表示目标归档成员名称

- 内置变量

		CC			指定编译器
		CPPFLAGS	编译和预处理选项，在使用默认规则的时候会自动调用
	

## 二、条件判断

条件判断有一下两种，一种是判断一个变量是否定义，另一种是判断变量的值是否是期望的某个值，其语法如下：

判断是否定义：
	
	#ifdef varialbe-name
		Do some thing
	#endif

	#ifndef varialbe-name
		Do some thing
	#endif
		
	变量名只写写名称即可，不需要在名称上使用$()包围

判断是否是某个期望的值：

	ifeq test
		Do some thing
	endif

	ifneq test
		Do some thing
	endif

	test的格式有两种："a" "b"或(a,b),注意在使用第二(a,b)语法的时候，`,`之后的空白是可以被忽略的，但其他部分的空吧是有效的



## 三、宏和函数

宏和函数的语法是类似的，不同的是函数可以有形参，形参在函数中是按照顺序来获取的，例如：

	$1 表示函数中的第一个参数
	$2 表示函数中的第二个参数
	$N 表示函数中的第N个参数

函数和宏的定义：
	
	define macro_namr_or_function_name

		函数或宏的内容
		
	endef

调用宏：

	$(macro_name)

调用函数：

	$(call function_name[,param1.....])

	依次在函数名称之后加函数名称和`,`然后依次填写参数，参数之间使用逗号分隔，如果没有参数则函数名称后可以没有逗号。

## 四、内置字符串处理函数

- 字符串过滤函数，只有模式匹配的才能通过，或不匹配的才能通过
	
		$(filter pattern...,text)		text中只有匹配pattern的字符才会返回

		$(filter-out pattern...,text)	text中只有不匹配pattern的字符才会返回

	例：删除CPPFLAGS中的-O2优化选项

		$(filter-out -O2,$(CPPFLAGS))


- 字符串替换函数

		$(subst search-string,replace-string,text)			在text中搜索search-string替换为replace-string

		$(patsubst search-pattern,replace-pattern,text)		在text中搜索匹配search-pattern的字符串，替换为replace-pattern

	**注意：subst是直接的替换函数，patsubst是先匹配模式，只有模式匹配成功之后才会替换。**

- 字符串查找函数

		$(findstring string,text)	

		在text中查找字符串string,如果找到则返回string(注意只是string，而不是包含string的text，而且string中不能包含通配符)

- 单词统计函数
 
		$(word text)				返回text中单词的个数，单词以空白分割，可以是空格或tab
		$(word n,text)				返回text中第n个单词，下标从1开始
		$(firstword text)			返回text中第一个单词，等同于$(word 1,text)
		$(wordlist start,end,text)	返回text 中下标从start到end的单词，也包括end


## 五、内置文件名处理函数
		
	$(wildcard pattern....)			返回pattern匹配的文件名称列表，如果没有匹配返回空字符串

	$(dir list...)					返回list中指定的文件名的目录名称

	$(notdir name...)				返回name中的文件名称，不包括目录名称

	$(suffix name...)				返回文件的扩展名

	$(basename name...)				返回文件的名称除了扩展名

	$(addsuffix suffix,name...)		给name指定的文件名称增加后缀并返回

	$(addprefix prefix,name...)		给name指定的文件名称增加前缀并返回

	$(join prefix-list,suffix-list)	依从从prefix-list和sufix-list中取出相同位置的元素组合成新的字符返回


## 六、流控制函数

	$(if condition,then-part,else-part)		测试，如果conditon中包含任何字符，包括空格执行then-part,如果为空执行else-part
	
	$(error text)		输出错误消息并终止make 并退出，退出代码为2。输出的错误消息中包括Makefile的名称，当前的行数以及text

	$(warning text)		输出警告消息，格式和error一样，但是不会退出Make

	$(foreach variable,list,body)			依次取出list中的内容，复制给variable，然后再将variable带入到body中执行

例：

	$(foreach name, HOME MAKE_VERSION,$(shell echo $($(name))))	打印HOME和MAKE_VERSION变量	


## 七、杂项函数

	$(sort list)		按字母顺序返回list，同时删除重复的项目
	
	$(shell command)	在make的subshell中执行command指定的命令

	$(strip text)		去除text首位的空白，并将text中间的空白，多个空格，换行替换为单个空格返回

	$(origin variable)	返回变量variable的来源描述符，描述符通常有一下几种

		undefined				变量未定义
		default					变量来自make的内置数据库，如CC，CPPFLAGS
		environment				变量来自于系统环境
		environment override	重写覆盖了的环境变量
		file					变量定义于Makefile
		command line			变量定义于make命令行
		override				变量被override指令重写覆盖
		automatic				Make定义的自动变量如：$@ $^等

## 八、函数高级应用

用户在设计自定义函数的时候，一个函数可能被不同的目标（target）调用，在函数处理的时候也有些细微的差别。例如:一个文件打包函数，在打包不同目标的时候，需要包含不同的文件，最终生成压缩包。那么针对不同的目标，可以定义自己的打包指令，目标调用打包函数打包文件的时候，会自动调用该目标自定义打包指令，从而实现不同的目标使用相同的打包函数。

例：

	define make_tarfile
		echo Generating $1 please wait...
		find $(INSTALL_PATH) -name "*.svn" | xargs rm -rf
		$(call make_tarfile_cmd)
		echo Target:$1 is done @$(INSTALL PATH)
	endef

	lib_target	:	make_tarfile_cmd = cd $(INSTALL_PATH) && tar cvf lib.tar lib_path
	tools_target:	make_tarfile_cmd = cd $(INSTALL_PATH) && tar cvf tool.tar tools_path


## 九、自动依赖检查

在编写程序的时候，通常需要在源文件中`include`各种各样的头文件。例如在编写C++程序的时候，就需要在`.cpp`文件中包含类的`.h`头文件，如果头文件发生改变，则程序需要重新编译。如果在`Makefile`中手工编写这个文件之间的依赖关系，过程将会十分繁琐，而且很容易出错。`gcc`提供了一种自动生成依赖关系的选项`-M`或`-MM`,使用方法如下：

	$(CC) -M file.c > file.d	
	
上面的这个语句将`file.c`这个文件依赖的文件写入了`file.d`这个文件中，我们在`Makefile`中只需要将`file.d`包含进去就可以了，免去了手工编写依赖关系的麻烦。

注意上面的选项`-M`是把`file.c`所有的依赖文件都写进去了，也包括系统头文件。但通常情况下，系统头文件是不会变动的，只需要包含用户自己生成的依赖文件就可以了，`-MM`选项就是这个功能，只是生成用户自己的依赖关系。例：

	$(CC) -M file.c > file.d

如上例子中的一个文件一个文件的生成依赖关系太麻烦了，可以写一条规则让`Make`自动完成这些工作，并将依赖关系包含进来，例：

	#Maklefile

	CC	=	g++
	CPPFLAGS = -Wall -g

	depend:$(wildcard *.c *.cpp *.h):
		$(CC) $(CPPFLAGS) -MM $^ > $@

	# -是为了忽略没有找到depend的警告
	-include depend



## 十、特殊目标（Special Targets）

Make中有一些特殊的目标用来改变make执行时的默认行为，常用的有一下几种

	.PHONY				用来指定目标并不是真正的文件，不需检查时间戳，例如 all clean install 等
	
	.SILENT				用来指定相关的目标在执行的时候不回显命令，类似于在命令前添加了@

	.IGNORE				用来指定相关的目标，如果在执行的时候出现错误，可以忽略继续执行

	.INTERMEDIATE		告知 Make 这些文件是在编程时产生的临时文件，在make执行结束后就可以删除了

	.PRECIOUS			告知 Make 这些文件是重要的文件，在 make 执行被中断的时候不会将其删除

	.DELETE_ON_ERROR	告知 Make 这些文件，在make的过程中，任何的目标执行失败，这些文件都会被删除 

## 十一、命令调节器（Command Modifiers）

Make 有一些命令调节器用来改变 make 执行该命令时的默认行为，常用的有一下几种：

	@	用来告知 make 该条命令不会回显，这个调节器可以添加在单条指令上、函数名称前, 如果要
	
	-	用来告知 make 忽略该条名称的错误信息继续执行。例如之前的`-include`命令

	+	用来告知 make 在执行该条命令的时候，即便是 make 指定 `-n` 选项（只打印不执行），该命令也会执行 

## 十二、杂项、注意事项

- Make 选项

		-n		显示 make 的执行过程，不真正的执行命令
		-s		使 make 的执行过程进入 SILENT 模式，所有的命令都不会回显
		-i		忽略 make 执行过程中出现的错误，继续执行

- 重写 all target

	include 指令可以将其他 makefile 或依赖文件加载到当前 makefile 中，包括其他文件中的目标和变量。如果在 makefile 的 all 目标之前调用 include 加载其他文件。那么， 新 include 的文件中的第一个目标就会成为 makefile 的默认目标，而不是 all 指定的目标。这显然不是我们想要的，要避免这个情况，需要做一下处理：

		all:
		include depend

		all:$(real_target)