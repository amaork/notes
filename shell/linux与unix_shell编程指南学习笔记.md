# Linux与UNIX Shell编程指南学习笔记

## 一、查找命令

1. find ./ -name "*" -exec command {} \;
2. 通配符的使用（通常和ls,grep配合使用） ^匹配行首，$匹配行尾 .匹配任意单字符 ，例如：查找当前目录中所有链接 ls -l | grep ^l 查找以D或d开头的文件夹 ls | grep ^[Dd]
3. 使用grep查找空行并显示空行的行号 grep -n '^$'
4. 在指定文件中查找字符串或字段 grep "要查找的字段" 文件
5. locate 要查找的文件名，用来在预先编译好的文件索引系统中查找，该索引系统可有updatedb命令手动更新；

## 二、文件管理

1. 目录文件的 r 权限表示可以列出该目录中的文件，但是不能进入该目录； x 权限表示可以进入该目录或是搜索该目录但是，不能列出目录中的文件；
2. 给文件设置suid、sgid 在所有权限为前加4设置suid在所有权限之前加2设置sgid也可以直接用+s -s 的方式设置，注意大写S表示相应的执行权限位没有设置基本上没有用途；
3. chmod -h 表示在改变链接符号的属主的时候不改变该链接指向的目标的属主；
4. 使用split和cat 分割合并文件，split -b file_size 要分割的文件 分割后文件名称的缀 cat 分割后文件名称的前缀* > 合并之后的文件名称
5. 创建指定时间的文件 touch -t MMDDhhmm filename
6. ext* 的文件系统可以使用lsattr和chattr显示和设置不同的属性
7. gzexe 可以把可执行文件压缩


## 三、系统管理

1. 配置编辑使用的编辑器 在profile中加入  EDITOR=你想要设置的编辑器如vim export EDITOR 即可使用crontab -e使用配置好的编辑器去配置

2. 可以在任意的位置创建一个 <username>cron的文件然后使用crontab <username>cron 把该文件作为crontab提交给crontab这样就会在/var/spool/cron中创建相应的副本

3. 可以通过 echo  <command job > | at <time>向at提交要执行的任务；

4. crontab 的格式 mintue(0-59) hour(0-23) day(1-31) month(1-12) weekday(0-6 Sunday-Saturday) ,“-”表示的是范围，“，”表示的是匹配其中的任意一个数字或范围

5. crontab 每分钟执行一次 */1 * * * * <command>

6. 在脚本中捕捉信号，使用trap命令在脚本中捕捉信号 trap "" 2 3 (捕捉信号2（^C）、3然后什么也不做，这样用户便不能停止该脚本了)也可以在trap后跟的双引号之间加命令或是一个函数，这样在产生了trap中指 定的信号后 就会执行trap "commond" 2 3 中指定的command了，如果要取消对信号的捕捉再次调用trap 2 3 即可

7. 配置当前终端 stty -g显示当前终端配置，stty -echo 关闭终端回显功能，可以配合trap命令和stty的配合锁住当前的终端；

8. 向系统发送日志信息 logger -p priority_level(such as notice) -i(thread id) message

9. du -s只显示占用空间的总数;

10. 显示当前那些进程在使用一个文件 fuser -u 文件路径 -k参数杀死所有正在访问的进程

11. 显示文件系统的类型 df -T

12. umount -l 可以卸载当前正在使用的文件系统；

13. 查找指定pid的进行 ps -fp "pid1 pid2 pid3" (p ---pidlist select by pid)；

14. 八进制 4000 2000 1000 分别为setuid、setgid和粘附位；

15. chmod更改权限的时候可哟随意组合如 chmod ug=rw,o=r test （文件的属主和组可以读写，其他只能读）

16. visudo 可以直接编辑/etc/sudoer文件，对文件的配置方式为：某个用户在某个名称的机器上可以以谁的身份运行那些命令格式为user_name machine_name=(user_id) command_list其中的
machine_name, user_id,command_list 都可以使用别名列出可用的清单

17. 禁用、启用某个用户登录 usermod -L user_name (disable) usermod -U user_name (enable)

18. 设置某个用户的过期时间 usermod -e year-month-day user_name （指定用户名为user_mode 的用户在指定的日期过期）

19. 启用禁用swap -->swapon,swapoff参数为swap所在的分区的设备

20. 格式化磁盘分区 mkfs.**** 设备 或 mkfs -t **** (****表示要格式化的分区类型)

21. 系统全面升级apt-get dist-upgrade

22. readlink linkname 获取link指向的目标

23. 产生一个随机不重名的文件mktemp

24. lsof列出系统当前打开的文件列表

25. pidof thread_name 根据进程名称获得pid

26. chroot 改变根目录的位置

27. watch -n command 指定间隔一定时间(n)执行command

28. linux下模仿 mac的反向使用鼠标滚轮 xmodmap -e "pointer = 1 2 3 5 4"



## 四、文本处理

1. awk打印报告头和尾，awk 'BEGIN {print "头部信息"} {print $1, $2, $5} END{print "尾部信息"}，其中$1 $2分别代表第一、第二列当然这个数字也可以用变量替换如line $'$line'这样列号的取值就来自line这个变量

2. 在awk中也可以使用条件判断和正则表达式，条件判断要用()括起来，可进行的条件判断有 > >= < <= == != ~(匹配正则表达式) !-(不匹配正则表达式)；

3. 使用tr 进行替换删除操作,tr用法 tr original_string -options
	
		-d 删除指定字符 
		-s 删除重复的字符之保留一个
		替换 tr "[A-Z]" "[a-z]" 大大写替换 也可以使用[0-9]  tr "[:upper:]" "[:lower:]"
		处理之后输出到文件 tr "[A-Z]" "[a-z]" < input.file > output.file
		删除文件中的空行 tr -s "[\012]" < input.file \012是换行的8进制表示方法 \r --> M^


4. basename 从路径中分离出文件名；如 basename /media/file  --->file

5. dirname  从路径中分离出路径名;如 dirname /media/file --->/media(与basename的功能相反)

6. 显示文件的行号和内容 nl

7. 在VIM中将当前文件保存为网页TOhtml

8. vimdiff file1 file2 file3 可以使用VIM打开这些文件并显示他们之间的不同通常用来比较显示不同版本文件的区别

9. 计算字符串长度 expr length "string"

10. tac 从尾部显示文件，与cat 命令的功能相反

11. rev 把没一行翻转显示到标准输出上

12. 列删除过滤器删除指定的列colrm

13. banner 把一个ASCII字符使用＃打印出来

14. tee 把结果送给两个文件

15. 将输入文件转换为8进制或其他进制如od /dev/urandom（/dev/urandom产生的数字是二进制的）

16. cut -cnum1-num2 filename 从文件的每行中取出num1-num2之间的字符


## 五、常用技巧

1. 在shell中使用数组 variable=(`command or string split by space `) echo ${variable[arrary_indx]}

2. 清除变量 unset variable_name

3. 测试变量是否设置，没有设置则设置 ${variable:=new_value},没有则替换 ${variable:-new_value};

4. 设置变量为只读：readonly variable_name

5. 嵌入shell变量，可设置CDPATH,这样在进入一个目录之前就会查找在当前目录中是否有这个目录有进入当前文件夹的这个目录，没有的查找CDPATH中是否有这个目录有的话直接进入，例如/var目录中有backups这个目录，设置CDPATH=/var之后，在任意没有名叫backups子目录的文件夹中cd backups 则会直接进入/var/backups目录中；

6. 在字符串中混合命令，cmd=cat\ text eval $cmd 这样程序就会解析cmd变量中的命令并执行

7. 可以在文本文件之前加上#!/bin/more 然后给文本文件加上可以运行权限就可以实现文本的自动显示

8. 空命令":",可以用来写死循环--->while : do :done

9. 清空一个文件--->cat /dev/null > cleanup_filename or --->  : > cleanup_filename

10. 从某个文件中读取 {read line} < /etc/fstab 可以配合awk命令解析文件等处理

11. 使用C语言风格的变量处理风格((a = 3)) (( a ++ )) ((a --))

12. 算术运算符的扩展，x=2 y=3 let z=x+y echo $z --->5

13. Linux 随机数密码生成工具cat /dev/urandom | sed 's/[^a-zA-Z0-9]//g' | strings -n 16



## 六、测试命令

1. if [ -d dsd ]可直接在if判断语句中使用，不需要带test命令

2. 脚本调试在脚本中加入set -v 在执行之前打印，或是bash -x 要执行调试的脚本


## 七、shell函数

1. 在一个shell文件定义一个函数之后，可以在该函数载入shell之后在命令行中调用该函数载入shell函数的方法， . /shell_file_path 注意在.和/之间要有空格后面跟随的定义了函数的shell文件的路径，载入之后直接可以在终端的shell命令行中键入函数的名称使用该函数。检查是否载入成功可使用set | 2. grep function_name 查找，使用unset 函数名称取消该函数。

2. 也可以在其他的shell文件中调用其他文件中的shell函数（或变量），前提是在调用之前要先载入shell函数；

3. factor 将一个整数分解成多个素数

4. 产生某个范围的数字，通常用于在循环中 seq 0-99(产生0－99之间的数字)seq 99（产生1－99的数字）例如for number in `seq 99`(99次数的循环)

5. 向标准输出连续不断的同同一个字符yes 会连续不断的输出y,也可以是其他字符将其他字符串作为yes的参数即可，通常在安装软件或其他地方使用

6. m4宏过滤处理器，例如$string=abcd echo "len($string)" | m4 ----> 4 echo "substr($string,2)" | m4 ---> cd


## 八、系统备份恢复

1. 使用tar -g snapshot 增量备份，在第一全部备份，以后就可以进行增量备份了，恢复的时候首先解压第一次的全备份然后依次解压

2. 计算校验和 sum,cksum,md5sum,sha1sum

3. 使用随机字符填充文件使得文件不能恢复 shred filename


## 九、内建变量

1. 当前行号--->LINENO

2. 之前路径--->OLDPWD相当cd -中保存的目录

3. 产生随机数字--->RANDOM（/dev/urandom或mcookie）


## 十、网络命令

1. nslookup 查询主机名称

2. traceroute

3. nmap局域网扫描进行ping扫描，打印出对扫描做出响应的主机：nmap -sP 192.168.1.0/24









