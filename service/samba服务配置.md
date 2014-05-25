# Samba 服务配置

## 一、描述：

samba服务，主要是用来在window和linux之间享文件使用的，可以像在windows之间共享文件一样。

1. 配置用户：安装好samba之后，使用smbpasswd -a username 来添加一个samba用户并设置密码 -a 指定是添加用户，注意username必须是已经在linux系统中已存在的用户，没有则需要先添加一个。

2. 配置共享内容：编辑/etc/samba/smb.conf 按照以下的规则进行配置：
                   
		[wxidong]                        
		comment = Windows Shard Floder          ＃备注信息
		path = /home/wxidong                    ＃共享文件夹路径（Linux）
		available = yes
		browsable = yes                                 
		public = yes
		writable = yes						
		
	配置完成之后重新启动samba即可，在windows下访问linux下的samba共享，Win+R 输入 
	
		\\linux主机的ip地址
		
	然后在弹出的对话框中输入在第二步配置用户的时候的添加的用户名和密码即可。

