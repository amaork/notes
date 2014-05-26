# Qt GUI事件虚函数
---

Qt 中绘图类中有许多事件的槽函数，当特定的事件发生之后，Qt就会进入这些函数中执行。要自己实现事件的槽函数，在类的protected:分类下声明，然后在cpp文件中实现即可。常见的有一下几种：    

## 1、定时事件： void timerEvent(QTimerEvent *event);

定时事件是周期性产生的事件，一些循环控制程序可以使用定时事件来实现。定时事件操作比较简单，就一个函数就可以直接启动定时器"startTimer(time)" time的单位是毫秒。


## 2、绘图事件：void paintEvent(QPaintEvent *event)

这个事件是用来实现自定义的绘图控件，实现绘制特定类型的图形的时候使用的，当需要重新绘制图形的时候，调用update()就会产生一个信号，Qt会调用这个函数来重新绘制控件。例如：

	void DrawBmp::paintEvent(QPaintEvent *event)
	{
        QPainter painter(this);

        painter.drawPixmap(..............................);        
        painter.drawText(.............);
        ......................
        ..................................

	}
	
<font color=red>
	注意：在paintEvent函数中的QPainter必须为在函数中定义的局部变量，不能使用全局变量。另外，如果使用的到QPixmap之类的数据结构，也必须为函数内的局部变量，不能为全局变量，否则长时间运行会让这个变量越来越大，直到耗尽系统中所有的内存。
</font>

## 3、键盘事件：void keyPressEvent(QKeyEvent *event)

这个事件是用来捕捉，用户在键盘上的按键事件的，当键盘上的任何一个键按下就会进入这个槽中进行判断处理。例如：

	void DrawBmp::keyPressEvent(QKeyEvent *event)
	{
        switch (event->key()){

               case  KEY_F1 :  todo something......break;
               case  KEY_F2 :  todo something .... break;
               ......................
        }
	}


## 4、鼠标移动事件：void mouseMoveEvent(QMouseEvent *);

鼠标移动事件是捕捉鼠标在当前绘图控件上移动，然后调用事件槽函数，在槽函数通过获取鼠标的坐标位置根据，再进行处理的。鼠标移动时间的启用比其他的事件多一个步骤，需要在QT程序的main函数中定义的Widget或main窗口中上调用”.setMouseTracking(true)”来启用鼠标坐标位置的追踪。注意：鼠标坐标位置的追踪只能在实现这个事件的控件的范围之内，超过这个范围就没用了，并且显示的坐标的位置也是在这个控件的位置。例如：

	void DrawBmp::mouseMoveEvent(QMouseEvent *event)
	{
       qDebug("X = %d, Y = %d\n", event->x(), event->y());
	}
