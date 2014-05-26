# iniparser使用笔记

## 一、说明

iniparser是一套使用使用C语言写的解析ini文件的函数库，有一个C文件和一个头文件构成，使用起来非常简单。一般不复杂程序的配置文件都可以使用"ini"格式的配置文件，然后在程序中连接或调用iniparser提供的一套API就可以非常方便的解析配置文件。iniparser提供的API都是以"iniparser_"为前缀的。

## 二、INI格式说明

	Format

	Keys (properties)

	The basic element contained in an INI file is the key or property. Every key has a name and a value, delimited by an equals sign (=). The name appears to the left of the equals sign.

	name=value

	Sections

	Keys may be grouped into arbitrarily named sections. The section name appears on a line by itself, in square brackets ([ and ]). All keys after the section declaration are associated with that section. There is no explicit "end of section" delimiter; sections end at the next section declaration, or the end of the file. Sections may not be nested.

	[section]
	Case insensitivity

	Section and property names are not case sensitive in the Windows implementation.[1]

	Comments

	Semicolons (;) at the beginning of the line indicate a comment. Comment lines are ignored.

	; comment text

## 三、INI文件例子

	[SYS]

 
	clist = 32 ;max clist.info file item
	timing = 255 ;max sync.info file item
	pattern = 1023 ;max img.info file item
	draw_lib= draw.so ;draw pattern dynamic lib

 
	[RKEY]

 
	dev = /dev/ttyS4 ;remote keyboard device name 
	pws = yes ;support power board 
	lcd = yes ;ths keyboard has lcd
	lib = key_rlcd4_menu.so ;remote keyboard dynamic lib name


## 三、使用步骤

1. 首先定义一个dictionary数据结构指针，iniparser提供的API都是围绕着这个数据结构进行的,在将ini文件装载到内存之后，ini文件中的所有信息都在这个数据结构中；

2. 载入配置文件，调用函数iniparser_load（file_path）函数，会载入file_path指定的配置文件，并返回一个dictionary的数据结构指针。这个指针是在iniparser函数库中动态申请的，所以在不需要操作ini文件之后需要调用：iniparser_freedict这个函数来释放dictionary；

3. 然后根据需要，对ini文件进行操作，例如：查看ini文件中Section的个数、查找在某个Section下是否有相应的key等等。

## 四、常用函数

	1、dictionary *iniparser_load(const char *file_name) 将file_name指定的ini文件加载到内存中，并返回与之对应的dictionary数据结构指针；

	2、void iniparser_freedict(dictionary *dict)将在iniparser_load函数中申请的dictionary的空间释放，dict应该指向iniparser_load函数返回的dictionary指针；

	3、int iniparser_numsec（dictionary *dict）返回dict指向的ini文件中有多少个Section(Section 即ini文件中使用中括号“[]”括起来的内容，例如在例子中的[SYS]这样的)；

	4、char* iniparser_getsecname(dictionary *dict, int n)返回dict指向的ini文件中的第n个Section的名字，n从0开始。注意：返回的是一个指针，这个指针指向dict数据结构，不能进行修改，如果需要使用的话，可以copy出来然后使用；

	5、int iniparser_find_entry（dictionary *dict, const char *entry）在dict指定的ini文件中查找，entry指定的数据，如果找到则返回1，否则返回0。注意entry是一个Section和key的组合例：“sys:clist”就是一个entry，在顶层的key中没有Section可以使用“:key”来替代；

	6、int iniparser_getinit(dictionary *dict, const char *key, int nofountd) 这个函数在dict指定的ini文件中查找key对应的int的值，如果找到了key则返回与之对应的value，否则返回nofound这个值。注意key也是Section：key这样的组合,；

	7、double iniparser_getdouble(dictionary *dict, char *key, double nofound)在dict指定的ini文件中查找key对应的doubel类型的值，如果没有找到则返回nofond的值；

	8、int iniparser_getboolean(dictionary *dict, char *key, int nofound)在dict指定的ini文件中查找key对应的boolean类型的值（真返回1，假返回0，大小的Y和T以及1都代表真，大小写的N和F以及0代表假），如果没有找到则返回nofound的值。

	9、char *iniparser_getstring(dictionary *dict, char *key, char *nofound)在dict指定的ini文件中查找key对应的string类型的数据，找到返回key对应的value的指针，如果没有找到返回指向nofound的指针。在得到返回指针之后幸需要copy出来才能使用。


## 五、使用技巧

默认的情况ini文件只有两层结构，顶层结构是[]包括起来的"section"，然后一个section下有若干个"key=value"类型的组合。要想扩展到三层结构，可以对"value"的格式进行扩展例如使用“[add, del, mod, all]”这样的表示，可以实现更复杂的功能。在解析这种类型的数据的时候可以使用swicth语句依次出来这个数据，在swicth要做的就是区分出一个数据到另一个数据的间隔,即确定all什么时候开始、结束，从而分辨出[]包含的数据有哪些；

