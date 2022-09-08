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


## Supervisor安装
yum install epel-release
yum install -y supervisor
安装Supervisor完成后,映射supervisord.conf文件
> echo_supervisord_conf >/etc/supervisord.conf  

修改/etc/supervisord.conf
我这里只修改了守护线程的配置文件扫描路径
> [include]  
files = /data/supervisord.conf.d/*.conf  

然后在/data/supervisord.conf.d文件下创建eureka.conf文件，文件内容如下
> [program:eureka]  
> directory = /data/project/eureka ; 程序的启动目录  
> command = /usr/bin/jvm/jdk1.8.0_281/bin/java -DlogPath=/data/log/app -jar /data/project/eureka/eureka-exec.jar ; 启动命令，可以看出与手动在命令行启动的命令是一样的  
> process_name = %(program_name)s_%(process_num)02d ;   
> autostart = true     ; 在 supervisord 启动的时候也自动启动  
> startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了  
> autorestart = true   ; 程序异常退出后自动重启  
> startretries = 3     ; 启动失败自动重试次数，默认是 3  
> redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false  
> stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB  
> stdout_logfile_backups = 5     ; stdout 日志文件备份数  
> ; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）  
> stdout_logfile = /data/log/supervisor/eureka.log  
> stopwaitsecs=2  

# supervisord二进制启动
supervisord -c /etc/supervisord.conf
# 检查进程
ps aux | grep supervisord

配置以systemd的方式管理
vi /etc/rc.d/init.d/supervisord
```#!/bin/sh
#
# /etc/init.d/supervisord
#
# Supervisor is a client/server system that
# allows its users to monitor and control a
# number of processes on UNIX-like operating
# systems.
#
# chkconfig: - 64 36
# description: Supervisor Server
# processname: supervisord

# Source init functions
. /etc/init.d/functions

prog="supervisord"

prefix="/usr"
exec_prefix="${prefix}"
prog_bin="${exec_prefix}/bin/supervisord"
PIDFILE="/var/run/$prog.pid"

start()
{
       echo -n $"Starting $prog: "
       daemon $prog_bin --pidfile $PIDFILE -c /etc/supervisor/supervisord.conf
       [ -f $PIDFILE ] && success $"$prog startup" || failure $"$prog startup"
       echo
}

stop()
{
       echo -n $"Shutting down $prog: "
       [ -f $PIDFILE ] && killproc $prog || success $"$prog shutdown"
       echo
}

case "$1" in

 start)
   start
 ;;

 stop)
   stop
 ;;

 status)
       status $prog
 ;;

 restart)
   stop
   start
 ;;

 *)
   echo "Usage: $0 {start|stop|restart|status}"
 ;;

esac
```

设置开机启动及systemd方式启动。
sudo chmod +x /etc/rc.d/init.d/supervisord
sudo chkconfig --add supervisord
sudo chkconfig supervisord on
sudo service supervisord start


利用supervisorctl查看进程，发现启动失败，显示
FATAL Exited too quickly (process log may have details)
进而查看jar包对应运行日志，文件目录在/data/supervisord.conf.d文件下eureka.conf文件中配置的
查看日志报错：java -jar XXX.jar 没有主清单属性
需要在pom文件中添加
>                    <executions>  
>                        <execution>  
>                            <goals>  
>                                <goal>repackage</goal>  
>                                <goal>build-info</goal>  
>                            </goals>  
>                        </execution>  
>                    </executions>  
![image](https://user-images.githubusercontent.com/17979141/189075585-e51e03c1-d142-48a7-8a68-04eda4363ca7.png)

在之后，eureka服务正常启动，可是还是访问不了，这是因为防火墙拦截了我的端口请求，直接关闭

关闭防火墙
systemctl stop firewalld.service
