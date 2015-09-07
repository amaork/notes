# 使用 pyinstaller 打包 Python 

### 一、基本用法

	pyinstaller-script.py -F 要打包的 python 脚本名称

### 二、常用选项

	-F 打包成单个文件，最终只会生成一个 “exe” 文件

	-W 打包带图形界面的应用程序，类似与 py2exe 中的 "windows"

	-p 设置 sys.path 包含的路径，如果包含有自定义路径下的文件，不设置将导致程序找不到对应的库，可以连续包含多个例如

		pyinstaller-script.py -p path1 -p path2 -F -W xxxx.py