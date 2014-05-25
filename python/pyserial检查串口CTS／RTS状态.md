# Pyserial检查串口CTS／RTS状态


使用Pyserial 可以很方便的操作串口，以下为检查串口CTS和RTS状态：

	#导入串口库
	import serial
	
	#打开串口,并设置波特率，rts,cts等
	ser = serial.Serial(port="/dev/ttyUSB0", baudrate=9600, rtscts=1)

	#设置串口的RTS为高电平
	ser.setRTS(level = 1)

	#循环检查CTS状态
	while True:

		#获取CTS状态
		cts_status = ser.getCTS()

		#检查CTS状态
		if cts_status:
			print "CTS TRUE"
		else:
			print "."
	