# nginx日志分割(log rotation)

本文链接：https://blog.csdn.net/dreamer2020/article/details/53893356
nginx没有内置日志分割功能，容易造成日志累积，文件越来越大。必须借助于外部命令或者工具来分割日志。

本文介绍一种通过自定义脚本来分割日志的方法。

## nginx日志及nginx.pid设置

nginx日志及进程号文件可以通过nginx.conf来修改，下面的配置将日志和nginx.pid都放在了nginx安装目录下。

``` 
worker_processes  4;
access_log  logs/access.log  main;
error_log  logs/error.log  warn;   
pid        logs/nginx.pid;      #pid of nginx master process
```

## 日志分割脚本
将实现分割的脚本放在/usr/local/bin/rotate_nginx_log.sh，内容如下

``` 
#!/bin/sh

# Get yesterday's date as YYYY-MM-DD
YESTERDAY=$(date -d 'yesterday' '+%Y-%m-%d')

# move log
mv /path/to/access.log /destination/access-$YESTERDAY.log  
mv /path/to/error.log /destination/error-$YESTERDAY.log

PID_FILE=/usr/local/nginx/logs/nginx.pid

# Tell nginx to reopen the log file.
kill -USR1 $(cat $PID_FILE)
```


## crontab配置定时任务
写好脚本后，接下来定时执行即可。
``` 
sudo crontab -e
``` 
打开定时任务配置页面后，输入
``` 
0   0   *   *   *   /usr/local/bin/rotate_nginx_log.sh 1>>/var/log/rotate_nginx.log 2>&1
``` 
可以在每天凌晨执行上述日志重命名脚本，完成日志分割任务。

注意，许多linux的系统只有root用户才可以添加定时任务，所以必须用sudo命令。

通过下面的命令查看是否添加成功：
``` 
sudo crontab -l
``` 

