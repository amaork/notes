# 安装配置 NFS
---

## 安装

以 Debian 为例

	sudo apt-get install portmap nfs-common nfs-kernel-server


## 配置

编辑`/etc/export`文件

	sudo vim /etc/export

格式为：

	共享文件夹路径  允许访问的主机或网段（NFS参数选项....）

- 共享文件夹路径为系统中的绝对路径如：/home/user_name/share

- 允许访问的主机或网段，可以单个主机，或IP网段，或含有通配符的IP网段
		
- NFS参数选项， 常用的有一下几种，可以根据需要进行组合，选项之间以逗号分隔

		ro					只读访问
		rw					读写访问（默认参数）
		sync				数据被写入存储设备之后，才相应请求
		async				数据被写入存储设备之前相应请求
		secure				要求连接使用1024一下端口发送请求（默认参数）
		insecure			使用1024以上的端口发送请求
		subtree_check		检查父目录权限		
		no_subtree_check	不检查父目录权限
		all_squash			将所有用户的请求都映射为匿名用户一样的uid/gid权限
		root_squash			将root用户的请求映射为和匿名用户一样的uid/gid权限
		no_root_squash		关闭root_squash映射
		anonuid=xxx			指定匿名用户的uid为xxx
		anougid=xxx			指定匿名用户的gid为xxx
	

## 启停

	启动：sudo /etc/init.d/nfs-kernel-server start
	停止：sudo /etc/init.d/nfs-kernel-server stop
	重启：sudo /etc/init.d/nfs-kernel-server restart

## 挂载、卸载


	挂载： sudo mount -t nfs [options] nfs_server_ip:abs_shared_path  local_mount_path
	卸载： sudo unmount local_mount_path

	nfs_server_ip 		是要挂载的 NFS 服务器的IP地址
	abs_shared_path		是要挂载的 NFS 服务器上的共享文件夹绝对路径
	local_mount_path	是要挂载 NFS 共享目录的挂载点