# Qt 实现放大镜效果
---

## 概述

在Qt显示图片的时候，有时需要实现放大镜效果。比如跟随着鼠标的移动，放大鼠标坐标一定区域范围内的图像并显示在当前图像的上一个图层中。

## 实现方法
 
1. 首先定义一个类，这个类继承于QWidget；

2. 然后在这个类的protected属性中定义并实现：绘图事件（paintEvent）和鼠标移动事件（mouseMoveEvent）；

3. 在private属性中定义一下参数,x,y,width,height,flag分别用来保存当前鼠标的坐标、将要放大图像的长、宽和是否在放大的标示；

4. 在private属性中定义三个QPixmap数据结构:pixmap,sample,zoom_in，分别用来显示背景图形、将要放大的图像（根据鼠标位置截取pixmap）和放大后的图像；

5. 然后在鼠标移动事件（void mouseMoveEvent(QMouseEvent*);）中完成一下工作：

	1. 先将flag置为0，然后保存当前鼠标的坐标到x,y中；
	2. 截取当前鼠标位置相对的一段区域的图像：sample = sample.grabWidget(this, x, y, 32, 24);
	3. 然后将截取到的图形区域放大之后保存在zoom\_in中：zoom\_in = sample.scaled(width\*6, height\*6, QT::KeepAspectRadio)；
	4. 最后将flag置位为1，然后调用update
	 
	
6. 在绘图事件处理函数（void paintEvent(QPaintEvent*event);）中完成一下工作

	1. 定义一个QPainter：QPainter painter(this);
	2. 判断flag的值，如果flag为1说明已经采集到要放大的图形区域，则先在QPainter上绘制背景（pixmap），然后在QPinter上，以当前鼠标坐标一定的偏移量的位置绘制放大后的区域；
	3. painter.drawPixmap(0, 0, pixmap)
	4. painter.drawPixmap(x + 15, y + 15, zoom\_in)
	5. 如果flag为0则正常绘制背景图片；
	6. painter.drawPixmap(0, 0, pixmap)
	7. QWidget::paintEvent(event);

7. 最后在Qt的main函数中将该Widget的setMouseTracking属性设置为true;




