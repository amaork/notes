# Pyqt图形界面设计
---

## 概述

Pyqt 是Python创建GUI程序的一个库，它使用Qt作为图形库。相对于其他wxPython、Tkinter等图形库优点是它功能强大可以使用“Designer”或“Qt Creator”很方便的设计UI文件，简化了UI的设计布局等工作。同时Qt图形库是可以跨平台的，在Windows下设计的程序不作修改的就可以运行在Linux下。设计Pyqt程序可以参考Pyqt的Tutorials.
 
## 步骤

1. 到Python的安装目录中进入"Lib\site-packages\PyQt4"目录打开"designer"设计界面；

2. 图形界面设计完成之后保存文件为"*.ui"文件；

3. 使用命令行进入Pyqt的安装目录敲以下命令将ui文件转换为py文件

		pyuic4.bat test.ui > test.py

4. 编写Python程序（pyqt_test.py）处理GUI的事件和数据等完成软件的功能

		#导入自己需要使用的库
		import sys
		import socket

		#导入Pyqt必须的库
		from PyQt4 import QtCore, QtGui

		#从UI文件中导入需要的界面类（Ui_Form来自于test.py中的定义类的名称也就是Ui中定义的主界面名字）
		from test import Ui_Form

		#在这里定义实现全局的函数和变量等
		s = socket.socket()
		s.setsockopt(socket.SOL_SOCKET, socket.SO_RCVBUF, 1024*128)


		#生成自己的程序类
		class StartQT4(QtGui.QMainWindow):
    		def __init__(self, parent=None):
    		    QtGui.QWidget.__init__(self, parent)
    		    self.ui = Ui_Form()
    		    self.ui.setupUi(self)
        
    		    #在这里定义QT信号与槽的连接属于__init__函数的范围
    		    QtCore.QObject.connect(self.ui.connect_button,QtCore.SIGNAL("toggled(bool)"), self.connect_server)
    		
			#在这里定义槽的实现和其他的处理函数（函数注意要和__init__函数的定义对齐）


	    	def connect_server(self, checked):
	    	    #Read server ipaddress
	    	    host = self.ui.server_addr.text().toAscii()
	    	    port = 8400
        
        	#Connect to server
        	s.connect((host, port))
        
        	#Get server current status
        	cur_status = s.recv(1024)
        
		#Pyqt应用程序的主函数
		if __name__ == "__main__":
    		app = QtGui.QApplication(sys.argv)
    		myapp = StartQT4()
    		myapp.show()
    		sys.exit(app.exec_())


程序编写完成之后使用一下命令即可启动Pyqt程序

	python pyqt_test.py

使用"py2exe"打包Pyqt程序参考《使用Py2exe打包Python程序》
