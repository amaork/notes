# Qt显示中文
---


1. 在Main函数中调用：
		
		QTextCodec::setCodecForTr(QTextCodec::codecForName("GBK"));

		或

		QTextCodec.setCodecForTr(QTextCodec.codecForName("UTF-8"))

	<font color=red>
	注意上面代码的位置，需要放在
		
		app = QApplication(sys.argv)

	这个语句之后，否则不能正常显示
	</font>

2. 然后在需要使用中文的地方，使用
		
		QObject::tr("中文字符显示即可！")；
		
		或
		
		QTextCodec* gbk_codec = QTextCodec::codecForName("GBK"); 

		//直接显示QString中的内容即可
		const QString warnning = gbk_codec->toUnicode(" 请填写正确的IP地址！");

 

 