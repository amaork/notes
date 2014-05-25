# NFS网络文件系统丢包


U-Boot使用nfs挂载文件系统的时候，系统启动成功之后，进行ls或其他的操作没有反应并且系统出现 
	
> not responding, still trying

说明NFS没有正常工作丢包问题比较严重，可以在设置挂载参数的时候，指定使用tcp协议进行连接。命令如下：

	setenv bootargs root=/dev/nfs nfsroot=192.168.1.117:/home/wxidong/workspace/filesystem/_install/,proto=tcp ip=192.168.1.17

更详细的使用说明参见内核文档 filesystem/nfs/nfsroot.tex 和nfs的manual
在嵌入式系统上挂载NFS文件系统的时候也会出现这种情况，可以使用以下参数指定NFS连接使用TCP连接

	mount -t nfs -o nolock,tcp 192.168.1.117:/home/wxidong/nfs /mnt

