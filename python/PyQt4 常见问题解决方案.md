# PyQt4 常见问题解决方案

### 1、PyQt4 程序退出之后，APPCRASH (显示崩溃的模块是 QtGui4.dll 或是 QtCore4.dll)


解决方案是：在 PyQt 中重载 `closeEvent`,在 `closeEvent` 中调用 `sys.exit()`</font>
  
      def closeEvent(self, ev):
        del sys.exit()
        
具体的原因是在程序关闭的时候，Qt 部分已经释放了资源而 Python 还没有退出
        
