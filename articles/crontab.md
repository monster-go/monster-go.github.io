### crontab计划任务详解
什么是crond ?用自己通俗的话来讲吧，在linux系统中有这么一个东西，它会每隔一段时间来到自己的目录中查看shell脚本，然后把shell脚本中的东西进行执行。  这个东西就是所说的crond服务，而我们要做的就是写好shell脚本放进这个crond服务的目录里面。

---

### crontab 文件存放的目录

#### /var/spool/cron  目录
这个目录主要是为了不同用户执行自己的脚本。

这个目录中有以用户名为名称的文件夹，用户的shell脚本就放在自己用户名的文件夹中。这是为了不同的用户执行自己的脚本。一般contab -e 产生的脚本都放在这个目录中。

#### /etc/cron.d/ 目录、/etc/crontab文件
***特别注意：执行不同目录中的脚本都需要指明用户是谁。root就是一个。。。***

这个文件主要是为了系统定时执行脚本而设置的。

关于`/etc/crontab`文件关联的目录有 `/etc/cron.hourly` 、`/etc/cron.daily` `/etc/cron.weekly` 、 `/etc/cron.monthly`。这些目录存放了各个用户的脚本，按小时、天、星期、月分类存放于这些目录。

---

### 语法
```shell
* * * * * [用户] command
```
#### 参数含义

代表意义 | 分钟 | 小时 | 日期 | 月份 | 周 | 指令|
---|---|---|---|---|---|---|
数字范围 | 0-59 | 0-23 | 1-31 | 1-12 | 0-7 | 呀就指令啊|



#### 示例
```shell
* * * * *  root  run-parts  /etc/cron.hourly   #每分钟
3 * * * *  root  run-parts  /etc/cron.hourly   #每小时的第三分钟,间隔60分钟
*/3 * * * *  root  run-parts  /etc/cron.hourly   #每隔三分钟
1 12 * * *  root  run-parts  /etc/cron.daily   #每12点的第一分钟
1 12 * * 1  root  run-parts   /etc/cron.weekly  #每周一的12点第一分钟
1 12 5 * *  root  run-parts   /etc/cron.monthly #每月的5号12点第一分钟

```

#### 秒级别运行任务
crontab 执行任务最小的时间纬度是分钟，可以利用`sleep`命令实现秒级别运行
```  shell
#每个10秒执行一次
* * * * * vagrant sleep 10; date >> /var/log/cron.log
* * * * * vagrant sleep 20; date >> /var/log/cron.log
* * * * * vagrant sleep 30; date >> /var/log/cron.log
* * * * * vagrant sleep 40; date >> /var/log/cron.log
* * * * * vagrant sleep 50; date >> /var/log/cron.log
```

#### crontab 指令
```shell
service crond start  #启动crond服务
service crond stop   #停止crond服务
service crond restart  #重启crond服务

#以下指令编辑/var/spool/cron目录文件
crontab -e  #编辑crontab的工作内容
crontab -l  #查询crontab的工作内容
crontab -r  #移除所有的crontab内容
crontab -u  #只有roo才能用此选项，意帮助其他使用者  建立/移除 crontab工作排成 如何编写crontab工作内容
```

#### 示例说明
```shell
0 12 * * * mail dreafly -s "at 12:00" < /home/dreafly/dream.txt
#每天的12点 做 (mail dreafly -s "at 12:00" < /home/dreafly/dream.txt)这个事
```