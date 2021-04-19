---
title: Linux下定时清空某个文件
date: 2021-04-19 17:47:27
update: 2021-04-19 17:47:27
categories: Linux
tags: Linux
---



##### 问题

在一台单点机器部署完成且运行一段时间后，发现页面接口报错，登上机器发现磁盘满了。通过`du -lh --max-depth=1` 和 `du -sh *` 找出是哪个文件。

发现是 项目中 stdout.log 文件过大，已经增加到了30多个G。



##### 解决

使用 `crontab`定时执行清空stdout.log文件的命令。

1. 准备一个 emptystdlog.sh 文件

   ```shell
   touch emptystdlog.sh
   ```

2. 文件写入以下内容

   ```shell
   #!/bin/bash
   echo "" > /root/work/output/stdout.log;
   ```

3. 使用`crontab -e` 每天定时执行一次

   ```shell
   0 0 * * * /bin/bash -x /root/work/output/emptystdlog.sh > /dev/null 2>&1
   ```

4. 查看 `crontab`使用情况

   ```shell
   crontable -l
   ```

5. 编辑或删除定时任务

   ```shell
   crontable -e
   ```



##### 总结

这不是解决这个问题的好办法，应该通过设置 `logback-spring.xml` 文件来控制 stdout.log 文件



