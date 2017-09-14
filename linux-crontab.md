---
title: crontab 命令
date: 2014-02-13 22:43:22
tags: [linux,crontab]
categories: linux命令
---
# 描述
一般编写调度让系统帮你定时做事情有两种方式：
* 在`/etc`目录下有几个`cron.`开头的文件夹，这里存放有`系统`运行的一些调度程序,可以把自己的调度写在这里。
* 每个用户可以用`crontab`命令建立自己的调度,调度文件存放在以用户名为名的文件`/var/spool/cron/crontabs/[username]`。 
 
<!-- more -->

`crontab`命令有三种形式的命令行结构： 
* crontab [-u user] [file]  
* crontab [-u user] [-e|-l|-r]  
* crontab -l -u [-e|-l|-r] 

第一个命令行中，`file`是命令文件的名字。如果在命令行中指定了这个文件，那么执行`crontab`命令，则会把这个文件保存到`/var/spool/cron/crontabs/[username]`；如果在命令行中没有制定这个文件，`crontab`命令将接受标准输入（键盘）上键入的命令，并将他们也存放在上面的那个文件。  

命令行中`-u`选项的作用用来切换用户，默认不加这个选项是表示当前用户 

命令行中`-r`选项的作用其实是删除当前用户的`/var/spool/cron/crontabs/[username]` 这个文件；  

命令行中`-l`选项的作用是显示当前用户`/var/spool/cron/crontabs/[username]`文件的内容。 

使用命令`crontab -u user -e`命令编辑用户user的cron(c)作业。用户通过编辑文件来增加或修改任何作业请求。  

执行命令`crontab -u user -r`即可删除指定用户的所有的cron作业。  

文件里的每一个请求必须包含以spaces和tabs分割的六个域。前五个字段可以取整数值，指定何时开始工作，第六个域是字符串，称为命令字段，其中包括了crontab调度执行的命令。  

[![](/images/linux-crontab.png)](/images/linux-crontab.png)

第一道第五个字段的整数取值范围及意义是：  
* 0～59 表示分  
* 1～23 表示小时  
* 1～31 表示日  
* 1～12 表示月份  （也可以是英文缩写）
* 0～6 表示星期（其中0表示星期日,也可以是英文缩写如sun，mon）  

# 20个超实用的Crontab使用实例
1. 每天 02:00 执行任务
    ````
    0 2 * * * /bin/sh backup.sh
    ````
2. 每天 5:00和17:00执行任务
    ````
    0 5,17 * * * /scripts/script.sh
    ````
3. 每分钟执行一次任务
    通常情况下，我们并没有每分钟都需要执行的脚本
    ````
    * * * * *  /scripts/script.sh
    ````
4. 每周日 17:00 执行任务
    ````
    0 17 * * sun  /scripts/script.sh
    ````
5. 每 10min 执行一次任务
    ````
    */10 * * * * /scripts/monitor.sh
    ````
6. 在特定的某几个月执行任务
    ````
    * * * jan,may,aug * /script/script.sh
    ````
7. 在特定的某几天执行任务
    ````
    0 17 * * sun,fri /script/scripy.sh
    ````
    在每周五、周日的17点执行任务
8. 在某个月的第一个周日执行任务
    ````
    0 2 * * sun  [ $(date +%d) -le 07 ] && /script/script.sh
    ````
9. 每四个小时执行一个任务
    ````
    0 */4 * * * /scripts/script.sh
    ````
10. 每周一、周日执行任务
    ````
    0 4,17 * * sun,mon /scripts/script.sh
    ````
11. 每个30秒执行一次任务
    我们没有办法直接通过上诉类似的例子去执行，因为最小的是1min。但是我们可以通过如下的方法。
    ````
    * * * * * /scripts/script.sh
    * * * * *  sleep 30; /scripts/script.sh
    ````
12. 多个任务在一条命令中配置
    ````
    * * * * * /scripts/script.sh; /scripts/scrit2.sh
    ````
13. 每年执行一次任务
    ````
    @yearly /scripts/script.sh
    ````
    @yearly 类似于“0 0 1 1 *”。它会在每年的第一分钟内执行，通常我们可以用这个发送新年的问候。
14. 每月执行一次任务
    ````
    @monthly /scripts/script.sh
    ````
15. 每周执行一次任务
    ````
    @weekly /scripts/script.sh
    ````
16. 每天执行一次任务
    ````
    @daily /scripts/script.sh
    ````
17. 每小时执行一次任务
    ````
    @hourly /scripts/script.sh
    ````
18. 系统重启时执行
    ````
    @reboot /scripts/script.sh
    ````
19. 将 Cron 结果重定向的特定的账户
    默认情况下，cron 只会将结果详情发送给 cron 被指定的用户。如果需要发送给其他用户，可以设置`MAIL`属性：
    ````
    # crontab -l
    MAIL=bob
    0 2 * * * /script/backup.sh
    ````
20. 将所有的 cron 命令备份到文本文件当中
    这是一个当我们丢失了cron命令后方便快速的一个恢复方式。
    下面是利用这个方式恢复cron的一个小例子。（看看就行~）
    首先：检查当前的cron
    ````
    # crontab -l
    MAIL=rahul
    0 2 * * * /script/backup.sh
    ````
    然后：备份cron到文件中
    ````
    # crontab -l > cron-backup.txt
    # cat cron-backup.txt
    MAIL=rahul
    0 2 * * * /script/backup.sh
    ````
    接着：移除当前的cron
    ````
    # crontab -r
    # crontab -l
    no crontab for root
    ````
    恢复：从text file中恢复
    ````
    # crontab cron-backup.txt
    # crontab -l
    MAIL=rahul
    0 2 * * * /script/backup.sh
    ````
