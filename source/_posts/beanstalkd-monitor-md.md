---
title: beanstalk之状态监控
date: 2019-06-02 17:43:52
tags: [beanstalk, shell]
---

# 说明

beanstalk状态监控，当beanstalk挂掉能及时重启。
------

## 1. shell 脚本
```
vim /home/king/sh/beanstalkd_monitor.sh

#!/bin/bash
ps aux | grep "/usr/bin/beanstalkd" | grep -v "grep"  | awk '{ print NF=$2}'
if [ ! $? -ne 0 ]; then /usr/bin/beanstalkd; fi

```

## 2. 添加可执行权限
```
chmod +x /home/king/sh/beanstalkd_monitor.sh
```


## 3. 加入cron
```
crontab -e

* * * * *  /bin/bash /hom/king/sh/beanstalkd_monitor.sh
```