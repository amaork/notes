## 一、概述

使用 C/C++ 扩展 Python 可以实现一些对运算速度和二进制数据有要求的功能，例如图像的编解码，或者是一些已经有 C/C++ 实现代码的项目。

## 二、vs2013 配置

1. 首先创建一个 Win32 Project，然后在 "Application settings" 初始化页面中的 "Application type" 选择 "DLL".

2. 根据使用平台配置 Platform，如果是 32 位平台，配置为 Win32, 64 位平台则配置为 X64

3. 配置 Genneral 属性，找到 "Target Extension" 将 ".dll" 更改为 ".pyd" (Python 扩展的后缀)

4. 配置 C/C++ 属性，找到 "Additional Include Directories" 选项然后添加 Python 的头文件目录（通常是 Python 安装目录下的 "include" 目录，我使用的是 Python2.7 对应的目录是：C:\Python27\include）

5. 配置 Linker 竖向，找到 "Additional Library Directories" 选项然后添加 Python 的 libs 目录（通常是 Python 安装目录下的 "libs" 目录，我使用的是 Python 2.7 对应的目录是：C:\Python27\libs）

## 三、创建初始化封装代码

所有的 Python 扩展模块必须有个初始化函数，完成对模块的初始化，同时还必须有个准备对外放出的函数列表，Python 导入这个模块这后就可以调用这些导出的函数列表。

###　1、函数列表

函数列表其实是一个 `PyMethodDef` 格式的数组，其每一项代表一个函数，格式如下：

	对外的调用函数名称，扩展模块内部的函数名称（指针），参数类型标志，函数描述
    
例如：

	#include "stdafx.h"
	#include <Python.h>

	static PyObject* add(PyObject* self, PyObject* args)
	{
		int x = 0;
		int y = 0;
		int z = 0;
		int i = 0;
		if (!PyArg_ParseTuple(args, "iii", &x, &y, &z))
		{
			return NULL;
		}

		i = x + y + z;
		return Py_BuildValue("i", i);
	}

	static PyMethodDef drawlibMethods[] ={
		{ "add",	(PyCFunction)add,	METH_VARARGS, "add(int1, int2, int3)" },
		{ NULL, NULL, 0, NULL },
	};
    
 ### 2、初始化函数
 
初始化函数的功能非常简单，就是简单的调用 `Py_InitModule` 函数注册模块的函数列表，但需要注意的是，初始化函数名称必须是以 `init` 打头，然后后面是项目也就是模块的名称。

例如项目名称为 `drawlib`
	
    PyMODINIT_FUNC initdrawlib()
	{
		Py_InitModule("drawlib", drawlibMethods);
	}



