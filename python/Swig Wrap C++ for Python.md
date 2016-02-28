# Swig Wrap C++ for Python


## 一、概述

最近做的一个项目，使用  C++ 写的一个网络控制 SDK。写完之后想继续在 Python 中使用，因为用的是 C++ 类，而不是单纯的  C/C++ 代码，所以用 没有办法加上接口作为 Python 的扩展使用。在网络上搜索了一通发现有两种方法可以实现：

1. 第一种是在原来的 C++ 代码中添加 extend "C" 然后在里面添加 C 函数，提供各种接口，例如 new_class, del_class 之类的代码，将类的所有共有接口都放出来，成为 C -style 的接口，然后在作为 Python 模块使用。这种方法，其实也是在 C 中调用 C++ 类的方法，但这样做的弊端在于要手工完成很多的工作。而且将来如果 C++ 中的类接口有增加或变化的话，那么这个接口部分也要随之变化，而且这些变化没有办法自动化完成，必须要手工敲代码才行，这样做无疑增加了很多的工作量，在代码量很少，不经常变动的情况下是可行的，但不是一个好方法。
2. 第二种就是本文将要这种描述的使用 Swig 将 C++ 代码，封装成为 Python  模块，不需要额外的工作，只需要编写一个 Swig interface  file（.i）。在该接口文件中，将  C++ 代码中的数据结构，转化为 Python 支持的数据结构即可。然后剩下的工作就交给 Swig 来自动化出来，着实省了不少的功夫。而且如果以后的 C++ 代码有变动的话，只需要重新编译一次即可，如果有数据结构和部分有变化，只需要把这部分变化表现在接口文件中即可。这样可以真正做到，一份代码在 C++ 和  Python 中共同使用，很方便实用。

## 二、接口文件

接口文件是告诉 Swig 应该怎样把原生语言（这里指的是 C++），转换为目标语言（这里指的是 Python），如原生语言中一些数据接口与目标语言不一样的地方。那 C++ 来说，C++ 中会用到指针，vector, string, map, pair 等部分，那么这部分就需要明确的告诉 Swig 应该怎样处理。Swig 有一些标准库的接口文件，已经对这个做了很好的处理，我们只需要在接口文件中对此做个简单的描述即可。

注意：接口文件的名称必须与最终模块的名称相同，例如模块名称为` NC816Sdk`，那么接口文件的名称为 `NC816Sdk.i`

### 1、头文件 include 

    %module NC816Sdk
    %{
    #define SWIG_FILE_WITH_INIT
    #include <stdint.h>
    #include "protocol.h"
    #include "NC816Sdk.h"
    %}

    %include <carrays.i>
    %include <cpointer.i>
    %include <std_pair.i>
    %include <std_string.i>
    %include <std_vector.i>

上边  include 了 2 个用户自定义的头文件，因为要用到  C-style 数组和指针所以 include 了`<carrays.i>` 和 `<cpointer.i>` 这两个标准库的头文件。然后在 C++ 代码中使用了标准库的  pair、string、vector 这些数据结构所以将这些头文件也加了进来。Swig 支持的头文件具体的可以参考：SWIG library

### 2、C++ 基本数据类型转换

    /* Basic type */
    %apply unsigned int {uint32_t}
    %apply unsigned char {uint8_t}
    %apply unsigned short {uint16_t}

以上的是 C++ 中使用的一些数据结构进行转换，`uint32_t` 其实就是 `unsigned int` 这些都需要告诉 Swig 否则，在使用的时候传递参数的时候，Python 模块将会不识别 uint32_t 这个数据类型，因为 Python 语言中并没有  uint32_t 这样的一个数据结构。


### 3、C++ 指针和数组类型转换

    /* Basic type pointer and array */
    %array_class(uint8_t, UcharArray);
    %pointer_class(float, FloatPointer);
    %pointer_class(uint32_t, UintPointer);

同上 Python 中没有指针和  C-style 数组的概念，如果在 C++ 中用到了这些，就需要借助于 Swig 的标准库中的 `<carrays.i>` 和  `<cpointer.i>` 这两个头文件，就是 第一小节中 include 的哪两个文件。这文件中提供了两种方式来处理指针和数组，一种是接口的方式，一种是类的方式：

#### 指针：

**接口方式：`%pointer_functions(type,name)`**
		
	Generates a collection of four functions for manipulating a pointer type *:
		
	type *new_name()
		Creates a new object of type type and returns a pointer to it. 
		In C, the object is created using calloc(). 
		In C++, new is used.
		    
	type *copy_name(type value)
		Creates a new object of type type and returns a pointer to it. 
		An initial value is set by copying it from value. 
		In C, the object is created using calloc(). In C++, new is used.
	  
	type *delete_name(type *obj)
		Deletes an object type type.
	  
	void name_assign(type *obj, type value)
		Assigns *obj = value.
	  
	type name_value(type *obj)
		Returns the value of *obj.
	
	
接口方式需要在 interface file 中添加 `%pointer_functions(type,name)` 例如：`%pointer_functions(int, intp)`,那么在 Python 就可以这么使用
	
	  1 import NC816Sdk
	  2 
	  3 width = NC816Sdk.new_intp()
	  4 height = NC816Sdk.new_intp()
	  5 
	  6 if NC816Sdk.getResolution(width, height):
	  7     print "Resolution is:",intp_value(width), intp_value(height)
	
	
**类的方式为每个指针结构提供一个类这个类中有一些列的接口，通过这些接口可以操作这个指针**
		
	  struct name {
      name();                           // Create pointer object
      ~name();                          // Delete pointer object
      void assign(type value);          // Assign value
      type value();                     // Get value
      type *cast();                     // Cast the pointer to original type
      static name *frompointer(type *); // Create class wrapper from existing
                                        // pointer
    };
		
		
类的方式需要在 interface file 中添加 `%pointer_class(type,name)`, 例如：`%pointer_class(int, IntPointer)`,那么在 Python 中就可以这么使用：
		
	  1 import NC816Sdk
	  2 
	  3 width = NC816Sdk.IntPointer()
	  4 height = NC816Sdk.IntPointer()
	  5 
	  6 if NC816Sdk.getResolution(width.cast(), height.cast()):
	  7     print "Resolution is:",width.value(), height.value()
	
#### 数组：

**接口方式：`%array_functions(type,name)`**

	  Creates four functions.
	  
	  type *new_name(int nelements)
		  Creates a new array of objects of type type. 
		  In C, the array is allocated using calloc(). In C++, new [] is used.
	
	  type *delete_name(type *ary)
		  Deletes an array. In C, free() is used. In C++, delete [] is used.
	
	  type name_getitem(type *ary, int index)
		  Returns the value ary[index].
	
	  void name_setitem(type *ary, int index, type value)
		  Assigns ary[index] = value.
		
例如：`%array_functions(int, intArray);`
	
	  1 import NC816Sdk
	  2 
	  3 addr = NC816Sdk.new_intArray(4)
	  4 
	  5 if NC816Sdk.getIPAddress(addr,4):
	  6     print "Device IP:%d.%d.%d.%d" %(intArray_getitem(addr,3), intArray_getitem(addr,2), intArray_getit    em(addr,1), intArray_getitem(addr,0))
	
	
**类的方式：`%array_class(type,name)`**

	  struct name {
      name(int nelements);                  // Create an array
      ~name();                              // Delete array
      type getitem(int index);              // Return item
      void setitem(int index, type value);  // Set item
      type *cast();                         // Cast to original type
      static name *frompointer(type *);     // Create class wrapper from
                                            // existing pointer
    };

例如：`%array_class(int, IntArray);`
	
	  1 import NC816Sdk
	  2 
	  3 addr = NC816Sdk.IntArray(4)
	  4 
	  5 if NC816Sdk.getIPAddress(addr,4):
	  6     print "Device IP:%d.%d.%d.%d" %(addr[3], addr[2], addr[1], addr[0])
	
综合来讲，数字和指针的以类的方式更加容易使用，而且代码也更为简洁，所以推荐使用类的方式。

### 4、C++ STD 数据类型

    /* C++ STD libs */
    namespace std {
        %template(NameList) vector<string>;
        %template(PlayItem) pair<unsigned int, string>;
        %template(PlayList) vector< pair<unsigned int, string> >;
    }

在 C++ 中会使用到 STD 中的一些数据类型，并且会使用这些数据类型自定义自己需要的一些数据结构，Swig 的库本身也支持这些数据类型，我们只需要在第一部分的头文件加载中将需要用到的数据类型的头文件加载进来即可，Swig 目前支持一下这些数据类型

    C++ class	        C++ Library file	SWIG Interface library file
    std::auto_ptr	    memory	          std_auto_ptr.i
    std::deque	      deque	            std_deque.i
    std::list	        list	            std_list.i
    std::map	        map	              std_map.i
    std::pair	        utility	          std_pair.i
    std::set        	set	              std_set.i
    std::string	      string	          std_string.i
    std::vector	      vector	          std_vector.i
    std::array	      array (C++11)	    std_array.i
    std::shared_ptr	  shared_ptr (C++11)std_shared_ptr.i

在上面的的例子中，在 C++ 代码中 `typedef` 定义了一个 `vector<string>` 类型的数据结构名称为 `NameList`，在 Swig 的接口文件中就可以这么写：`%template(NameList) vector<string>`;

在 Python  中使用的时候，例如：

    1 import NC816Sdk
    2 
    3 timingList = NC816Sdk.NameList()
    4 
    5 if NC816Sdk.getTimingList(timingList):
    6     for timing in timingList:
    7         print timing


### 5、C++ operator 重载

    /* C++ operator<< reload */
    %rename(PrintNameList) operator<<(std::ostream &out, const NameList& list);
    %rename(PrintPlayList) operator<<(std::ostream &out, const PlayList& list);

在 C++ 中会重载一些 operator , 在默认的情况下 Swig 是不认识这些重载函数的，那么就需要在 interface file 中指明这些，否则在编译的时候回报错。如上例子中将 NameList 的 operator<< 的重载函数重命名为 PrintNameList 那么在 Python 代码中就可以使用 PrintNameList 这个函数调用这个重载函数

## 三、编译

当编写好 interface file 之后，就可以使用 swig  处理接口文件然后生成相应的 wrap 文件和 py 文件，最终对这些模块编译链接最终成为 python 的模块（注意最终成成的模块名称前需要加  "_"，例如: NC816Sdk 最终编译出来的模块应该是：_NC816Sdk.so 或  _NC816Sdk.pyd 。

### 1、手工编译


1. 调用 swig 生成 wrap 文件： ➜  NC816Sdk git:(master) ✗ `swig -c++ -python NC816Sdk.i` 
2. 编译 C++ 代码：➜  NC816Sdk git:(master) ✗ `g++ -O2 -fPIC -c protocol.cpp NC816Sdk.cpp` 
3. 编译 swig wrap 文件：➜  NC816Sdk git:(master) ✗ `g++ -O2 -fPIC -c NC816Sdk_wrap.cxx -I/usr/include/python2.7` 
4. 链接生成模块：➜  NC816Sdk git:(master) ✗ `g++ -shared protocol.o NC816Sdk.o NC816Sdk_wrap.o -o _NC816Sdk.so`

### 2、Linux 下自动编译

Linux 下可以使用  Python distutils 自动编译，首先需要新建一个 `steup.py` 自动编译脚本，然后执行

➜  NC816Sdk git:(master) ✗ `swig -c++ -python NC816Sdk.i` 
➜  NC816Sdk git:(master) ✗ `python setup.py build_ext --inplace`

即可自动编译生成模块。Setup.py 脚本的内容为：

    #!/usr/bin/env python

    """
    setup.py file for SWIG NC816Sdk
    """

    from distutils.core import setup, Extension
    
    NC816Sdk_module = Extension('_NC816Sdk', 
    sources=['NC816Sdk_wrap.cxx', 'protocol.cpp', 'NC816Sdk.cpp'],)
    
    setup (name = 'NC816Sdk',
      version = '0.1',
      author = "wxidong@msn.com",
           description = """XinRunJing NC816 SDK""",
           ext_modules = [NC816Sdk_module],
           py_modules = ["NC816Sdk"],
    )

当然也可以通过 Makefile make sdk 来自动编译

    sdk:NC816Sdk.i
	    swig -c++ -python $<
	    python  setup.py build_ext --inplace
