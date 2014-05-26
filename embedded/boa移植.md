# Boa移植

修改boa源代码使boa可以以root的权限运行，既在 /etc/boa/boa.conf中指定User和Group都为0，既root用户的组;

修改boa.c 中的 drop_privs（）函数，屏蔽掉密码判断和setuid等报错的函数调用，重新调用即可