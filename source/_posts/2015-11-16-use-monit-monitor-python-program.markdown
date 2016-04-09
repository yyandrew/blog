---
layout: post
title: "使用monit监控python程序"
date: 2015-11-16 12:15
comments: true
categories: "System"
---
假设有个python程序`/home/exam-python.py`需要通过`python /home/exam-python.py`启动，我们可以通过如下方法利用[monit](https://github.com/arnaudsj/monit)监控它.

1.在`/etc/monit.d/`创建一个`python-monit`配置文件供monit使用

``` bash
check process exam-python with pidfile /var/run/exam-python-pid.pid
  start = "/bin/exam-python-script start"
  stop = "/bin/exam-python-script stop"
```

2.创建`/bin/exam-python-script`用于启动/终止python程序

``` bash
#!/bin/bash

PIDFILE=/var/run/exam-python-pid.pid

case $1 in
   start)
       # Launch your program as a detached process
       python /home/exam-python.py 2>/dev/null &
       # Get its PID and store it
       echo $! > ${PIDFILE}
   ;;
   stop)
      kill `cat ${PIDFILE}`
      # Now that it's killed, don't forget to remove the PID file
      rm ${PIDFILE}
   ;;
   *)
      echo "usage: exam-python {start|stop}" ;;
esac
exit 0
```

3.使用方法
```bash
monit start exam-python # 启动对exam-python的监控
monit stop exam-python # 终止监控
```
