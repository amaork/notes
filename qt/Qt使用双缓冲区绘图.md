# Qt 双缓冲区绘图
 ---

## 概述

QPainter可以在QWidget、QPixmap等绘图设备中进行绘图，来实现一些特殊的自定义界面，如圆形的按钮、表格等等。

其工作原理是：

1. 在Pixmap中绘制图形或加载图形文件到Pixmap对象中；
2. 图形加载完毕或绘制完毕之后调用update函数，这个函数会触发QWidget的paintEvent函数；
3. 在QWidget的paintEvent函数中生成一个QPainter对象，然后使用QPainter对象的drawPixmap函数将Pixmap中的图形绘制到QWidget中，从而实现图形绘制；

## 步骤

要实现以上功能，我们需要实现以下功能：

1. 自己定义一个绘图类QWidget，这个绘图类继承于QWidget，如果要自定义一个按钮则从QPushButton继承；
2. 在这个绘图类中，实现我们自己需要的paintEvent函数（这个函数是虚函数），通过这个函数实现绘图步骤3；
3. 在绘图类中重实现，minmumSizeHint和sizeHint函数通过这两个函数可以确定我们需要的绘图QWidge的尺寸；
4. 在绘图程序中定义一个或多个public slots，这些槽用来实现外部信号触发之后重新在pixmap中重新绘制图形并调用update函数；
5. 在绘图程序中包含前几步定义的绘图类，并生成一个实例；
6. 在绘图程序的构造函数的setupUI函数之后，将上一步生成的自定义绘图类的实例嵌入在界面中（通常是通过addWidget添加到Layout中）；
7. 然后根据绘图功能的需要将界面中的相关信号，连接到我们自己定义public slots中即可；

## 注意事项

1. 必须在setupUi函数调用之后再添加我们自己绘图QWidget到GUI中否则会报错；
2. 最好将自己定义的QWidget添加到Layout中一固定位置；
3. 绘图必须在自己定义类中的paintEvent中实现；
4. 如果绘图使用的是drawPixmap 的方式，在调用QPainter的drawPixmap之前需要先填充pixmap，如果是循环的使用，最好将pixmap作为paintEvent中的本地变量而不要使用全局的pixmap，因为pixmap会调用load或loadFromData的方法，多次调用不释放会致使这个循环调用的pixmap越变越大，最终导致X server崩溃。

例子：

	void DrawBmp::paintEvent(QPaintEvent *event)
	{
		/* 定义一个Painter 对象 */
      	QPainter painter(this);  

      	/* 定义一个Pixma对象 */
      	QPixmap pixmap, tmp;

       	/* 从内存中加载图片数据 */
       	pixmap.loadFromData(bmp_data, bmp_data_size, "bmp"); 

       	/* 放大或缩小图像到指定的大小 （根据需要来操作）*/
       	tmp = pixmap.scaled(width, height, ,Qt::KeepAspectRatio);   

       	/* 在QPainter中显示 */
       	painter.drawPixmap(x_loc, y_loc, tmp);

       /* 其他的操作，例如在QPainter中绘制文字、线条等等 */
	}


## 在图形中叠加文字

在paintEvent函数中，绘制图形之后调用其他绘图函数之后，然后调用QPainter的drawText的方法添加图形图形就叠加在之前绘制图形的上方。

## Pixmap绘制BMP图像

pixmap可以从文件中载入图形或是从内存中载入图像，分别对应load和loadFromData的方法

1. load的方法直接使用文件的路径，将指定路径的BMP图形载入到pixmap 中；
2. loadFromData的方法则是使用一个QByteArray或char数组来保存要这些缓存的内容；