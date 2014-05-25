# SVN版本管理系统使用

## 一、建立SVN服务

创建SVN目录
> svnadmin create <server path> (server path想要存放svn的路径 eg:svnadmin create /home/wxidong/svnserver)；

导入初始数据
> svn import <project path > file://<server path><project new name >  -m "[note]"  (eg:svn import /home/www file:///home/wxidong/svnserver/nc802_www -m "nc802_www")

启动SVN服务器
> svnserve -d -r <server path> (eg:svnserve -d(daemon) -r(root) /home/wxidong/svnserver)

取出SVN中制定的版本
> svn checkoout[co] ([--username] tobi) svn://<server ip><project name> (eg: svn
co svn://192.168.1.107/nc802_www, 不指定项目则取出所有的项目)；


## 二、配置SVN读写权限

在<server path>中的conf目录中修改 svnserve.conf 配置是否需要密码，读写权限等信息；

     password-db = passwd //开启使用密码功能
      anon-access = read     //匿 用户只读
      auth-access = write    //授权用户可写

配置密码在<server path>中conf目录中修改passwd文件配置用户名称和密码 格式为 username = userpasswd（注意该用户和密码和系统中的用户和密码没有任何关系）

## 三、常用增加、删除、更新操作

导出操作：
> svn co svn://<svn server ipaddress>/<project name> [-version] (获取指定版本的代码，不指定则为最新)

增加操作：
> svn add file_name  svn commit(ci) -m "<description note>" <add file_name>(必file:///C:/Documents%20and%20Settings/Administrator/Application%20Data/Microsoft/Internet%20Explorer/Quick%20Launch/%E6%98%BE%E7%A4%BA%E6%A1%8C%E9%9D%A2.scf须先执行add操作然后在执行conmmit操作)

删除操作：
> svn del svn://<svn server ipaddress>/<projwct name>/<del file name> [-m "descrpt"](eg:svn delete svn://192.168.1.1/pro/domain/test.php -m “delete test file”)



查看版本更新日志:
> svn log [path]

更新到指定版本：
> svn up -rN< N是版本号> 不指定版本号则自动更新到最新版本；

查看当前版本状态：
> svn status [-v]

查看信息:
> svn info [path]

比较版本之间的不同：
> svn diff -r m:n [path or file] （比较办不到m和版本n之间的差异）

显示所有文件状态
> svn st -v [ file or path ]

提交更改
> svn commit file_name -m "<description note>"（要提的文件必须是已经增加被纳入到SVN管理中的文件）

查看文件中的那行是有谁修改
> svn blame file.c

查看文件自某个日期的变化：
> svn diff -r "{year-month-day}" filename.c 