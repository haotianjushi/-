# 系统搭建日志

vi命令
进入输入模式的方法是输入 i、a、o 等插入命令，编写完成后按 Esc 键即可返回命令模式
移动光标：
a、以字符为单位移动
在命令模式中使用 h、j、k、l 这 4 个字符控制方向，分别表示向左、向下、向上、向左。
b、以单词为单位移动
w：移动光标到下一个单词的单词首
b：移动光标到上一个单词的单词首
e：移动光标到下一个单词的单词尾
c、移动到行尾或者行首

## 固定虚拟机IP
sudo vi /etc/sysconfig/network-scripts/ifcfg-ens33
![image](https://user-images.githubusercontent.com/17979141/188799267-2c8ebdf9-1a65-4abb-bf54-3c5717d2fd22.png)
重新启动网络配
> service network restart

对虚拟机内存扩容
https://blog.51cto.com/u_15266039/4967469


#Supervisor安装
yum install epel-release
yum install -y supervisor

修改/etc/supervisord.conf

关闭防火墙
systemctl stop firewalld.service
