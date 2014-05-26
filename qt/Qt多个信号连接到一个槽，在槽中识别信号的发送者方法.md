# Qt多个信号连接到一个槽，在槽中识别信号的发送者方法
---

Qt是通过信号和槽的机制进行事件传递的，当有多个不同类型、或相同类型的物件的发送信号都通过一个槽来处理的时候，需要在槽中识别出这些信号然后做相应的处理。

例如：

在一个界面中有16个按钮（QPushButton）和4个（QRadioButton）这20个物件的SIGNAL(clicked(bool))都连接（connect）到同一个按键的处理槽中（void get_keyvalue(bool)）那么就需要在get_keyvalue这个槽中把这些信号的发送者都识别出来，然后取其相应的键值然后发送，其方法是：

1. 方法1：使用dynamic\_cast确定发送者类型:

		void FBx::get_keyvalue(bool)
		{
		    if (QPushButton* btn = dynamic_cast<QPushButton*>(sender())){

		        send_key(btn->whatsThis());
	
		    }
		    else if (QRadioButton *rtn = dynamic_cast<QRadioButton*>(sender())){

		        send_key(rtn->whatsThis());
		    }
		}
 
	在槽（SLOT)中sender()函数会返回一个指向QObject 的指针来指向信号的发送者（Returns a pointer to the object that sent the signal, if called in a slot activated by a signal;）。然后通过C++ RTTI(Run-Time Type Identification)机制提供的dynamic\_cast运算符，在执行的时候检查sender()返回的对象是否是QPushButton类，如果是则将sender()返回的QObject指针转换为QPushButton指针，然后if中的语句就会执行。如果sender()返回的对象不是QPushButton类型的指针，则dynamic\_cast就会返回0，if中的语句就不会执行了。 

2. 方法2：直接与对象名称进行比较，确定具体是谁发送的信号

		void FBx::get_keyvalue(bool)
		{
		    if (sender() == ui->bt1){

				//是bt1按钮发送的信号，具体处理
			}
			else if (sender() == ui->bt2){
				
				//是bt2按钮发送的信号，具体处理
			}
			....
			......
		}
		
