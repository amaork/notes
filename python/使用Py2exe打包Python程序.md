# 使用Py2exe打包Python程序
---

使用Python设计的程序，要想方便的在其他没有Python安装环境的机器使用，就需要将Python程序连同需要的库打包成一个可执行文件。Py2exe提供了将应用程序打包的为Windows可执行程序。在Py2exe的网站有提供Py2exe的下载和使用，可以参考Py2exe 的Tutorail.

##1、打包普通Python程序（终端程序）


<font color=blue>hello.py</font>

	#一直打印否则一闪而过看不到
	while True:
    	print "Hello Python"

建立"setup.py"打包脚本

	from distutils.core import setup
	import py2exe

	setup(console=['hello.py'])

开始打包，最后打包好的程序在setup.py目录下的dist文件夹中，生成的可执行程序名字和Python脚本的名字一样，例如本例中的可执行文件名字为"hello.exe"

	python setup.py install
	python setup.py py2exe

## 2、打包Pyqt程序(GUI程序)

打包Pyqt程序的setup脚本的写法和普通的Python脚本的有些不同,其他的操作步骤都一样

	from distutils.core import setup
	import py2exe

	setup(windows=[{"script":"pyqt_test.py"}], options={"py2exe":{"includes":["sip"]}})

##　3、将py2exe打包好之后的程序打包成可安装程序
通过Inno Setup可以将dist目录中的Python程序打包为Windows可安装程序
