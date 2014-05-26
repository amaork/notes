# 在Qt中使用多播（multicast）
---

Qt 4.8中QtNetwork—>QUdpSocket类中新增加了“joinMulticastGroup”函数，可以让QUdpSocket加入指定的多播组，来监听来自改组的消息。
 
例子： 
 
    QUdpsocket *socket;
    QHostAddress mcast_addr("224.0.0.17"); 
 
    /* New socket */
    socket = new QUdpSocket(this);
    /* Binding to a port */
    if (!socket->bind(port, QUdpSocket::ShareAddress)){
        qErrnoWarning("Binding error!");
        exit(1);
    }

	#ifdef MCAST_IMPLEMENT
    
	/* Set socket option disable multicast loopback */
    socket->setSocketOption(QAbstractSocket::MulticastLoopbackOption, 0);
    /* Join to a multicast group */
    if (!socket->joinMulticastGroup(mcast_addr)){
        qErrnoWarning("Join to multicast group error!");
        exit(2);
    }
	#endif

 
在Windows下使用的时候，如果系统中安装有“Oracle VM VirtualBox”则需要在网络连接中将“VirtualBox Host-Only Network”禁用掉，否则多播程序无法正常工作，仅主机 自己可以接收得到程序发送的多播消息。

