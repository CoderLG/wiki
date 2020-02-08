查看服务器版本

cat  /proc/version

cat /etc/redhat-release 

**ifconfig: command not found**

yum install net-tools 

**防火墙**

- systemctl stop firewalld 关闭防火墙
- systemctl disable firewalld 禁止防火墙开机自启
- vim /etc/selinux/config —> SELINUX=disabled (修改)

https://www.jianshu.com/p/f804bd587d95

https://www.staroon.dev/2018/12/01/CDH6BeforeInstall/

https://blog.csdn.net/zimiao552147572/article/details/89817025

https://blog.csdn.net/LW_GHY/article/details/88693806

https://www.cnblogs.com/swordfall/p/10816797.html



rmp中的其他文件都 mv

