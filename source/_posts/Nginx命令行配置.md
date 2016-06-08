title: Nginx命令行配置
tags:
  - nginx
categories:
  - 读书笔记
  
---

接下来,会记录下Nginx从基本到深入的一些东西.

在linux中，需要使用命令来控制Nginx服务器的启动与停止，重载配置文件，回滚日志文件，平滑升级等行为。

<!-- more -->

1)默认启动方式

	／usr/local/nginx/sbin/nginx
	这时，会读取默认路径下的配置文件: /usr/local/nginx/conf/nginx.conf
	
2)指定配置文件的启动方式
	
	/usr/local/nginx/sbin/nginx -c /tmp/nginx.conf

3)指定安装目录

	/usr/local/nginx/sbin/nginx -p /usr/local/nginx

4)全局配置项的启动方式

 	/usr/local/nginx/sbin/nginx -g "pid /var/nginx/test.pid"
 	-g参数约束的条件是指定的配置项不能与默认路径下的nginx.conf配置项冲突。
 	-g启动的Nginx服务执行其他命令时，需要把－g参数也带上，否则可能出现配置项不匹配的情形。
	比如：/usr/local/nginx/sbin/nginx -g "pid /var/nginx/test.pid;" -s stop
	
5)测试配置信息是否有错误
 	
 	/usr/local/nginx/sbin/nginx -t
 	
6)在测试配置阶段不输出信息

 	/usr/local/nginx/sbin/nginx -t -q
 
7)显示版本信息

	/usr/local/nginx/sbin/nginx -v

8)显示编译阶段的参数

	/usr/local/nginx/sbin/nginx -V

9)快速地停止服务

	/usr/local/nginx/sbin/nginx -s stop
	－s参数其实是告诉Nginx程序向正在运行的Nginx服务发送信号量，Nginx程序通过Nginx.pid文件中得到master进程的进程ID，再向运行中的master进程发送TREM信号来快速地关闭Nginx服务。
	ps -ef | grep nginx
	kill -s SIGTERM port或kill -s SIGINT port
	其实得到的效果与-s stop一样的

10) 优雅的停止服务

	处理完当前所有请求再停止服务
	/usr/local/nginx/sbin/nginx -s quit
	或kill -s SIGQUIT <nginx master pid>
	如果要优雅地停止worker进程：kill -s SIGWINCH <nginx worker pid>

11)使运行中的Nginx重读配置项并生效
	
	／usr/local/nginx/sbin/nginx -s reload
	或kill -s SIGHUP <nginx master pid>
12）日志文件回滚
	使用-s reopen参数可以重新打开日志文件，把当前日志文件改名或转移到其他目录进行备份，再重新打开时就会生成新的日志文件。这个功能使得日志文件不至于过大。
	
	／usr/local/nginx/sbin/nginx -s reopen
	或kill -s SIGUSR1 <nginx master pid>

13)平滑升级Nginx
	
	nginx支持不重启来完成新版本的平滑升级
    kill -s SIGUSR2 <nginx master pid>
    会将pid文件重命名，如将/usr/local/nginx/logs/nginx.pid重命名／usr/local/nginx/logs/nginx.pid.oldbin
    启动新版本的Nginx，ps可以发现新旧的在同时运行
    kill命令向旧版本的master进程发送SIGQUIT信号。
 
14）显示命令行帮助 

	－h 或者？

-----
如果觉得还不错，赏点酒钱！
![](/images/aex068188cqwy9xbxa3oc07.png)
