# rsync 文件同步
---

rsync是linux下的工具，其可以在本地计算机和远程计算机之间同步数据，它有2种工作模式，Server和Client模式。

## Server

Server模式下的rsync是作为一个守护进程在后台运行的，其工作的时候需要一个配置文件，默认的配置文件放置位置是：/etc/rsyncd.conf 。

当然也可以使用 --config=path的方法，来指定配置文件的路径。例如可以使用一下的命令来启动server:

	rsync --daemon 
	rsync --daemon --config=/home/user/rsyncd.conf

rsyncd.conf配置文件示例：

	uid = amaork
	gid = amaork
	use chroot = no
	max connections = 4
	syslog facility = local5
	pid file = /var/run/rsyncd.pid

	[conf]
	path = /home/user/info/
	comment = whole ftp area (approx 6.1 GB)

	[tmp]
	path = /tmp/rsync_test
	read only = no
	comment = This is a tmp dir

 
配置文件中一个[]中的一个选项，代表一个路径，通过read only选项可以指定，这个路径是否可以修改，即client能否上传文件到这个路径中

## Client


Client直接使用命令行即可进行操作例如将server中的[conf]目录中的内容下载到本地info文件夹中：

	 rsync -vzrtopg 192.168.1.66::conf ./info/

Client也可以将本地的文件推送到server中前提是，将要推送的目录是设置为可写的即：read only = no。

	例如：rsync -vzrtopg info/ 192.168.1.66::tmp

 

## inotify

可以通过inotify工具的配合监控同步目录下文件的变化情况，来完成数据的实时同步，例如：备份服务器运行rsync server，本地计算机使用inotifywait来监控需要备份的文件夹的变动情况，当检测到某个文件变动之后使用rsync将改动的文件主动推送到备份服务器从而完成，数据的实时同步。

 

 

 

 







