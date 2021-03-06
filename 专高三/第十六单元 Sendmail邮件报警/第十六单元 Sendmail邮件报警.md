

[TOC]





# 第十六单元-邮件报警案例





## 16.1 邮件报警工具的使用

### 16.1.1 mail邮件工具

**centos下如何使用sendmail发送邮件**

最近在实施服务端日志监控脚本，需要对**异常情况**发送邮件通知相关责任人，记录下centos通过sendmail发送邮件的配置过程。

#### 一、安装sendmail与mail

**安装sendmail**

```
#安装
yum -y install sendmail

#启动
/etc/init.d/sendmail start
```



#### 二、设置发件人信息

上述发送邮件默认会使用linux当前登录用户，通常会被当成垃圾邮件，指定发件人邮箱信息命令：vim /etc/mail.rc，或者执行如下：

```
cat >/etc/mail.rc<<EOF
set from=13161168831@163.com
set smtp=smtp.163.com
set smtp-auth-user=13161168831@163.com
set smtp-auth-password=wjj142633
set smtp-auth=login
set smtp-use-starttls
set ssl-verify=ignore  >/dev/null
set nss-config-dir=/etc/pki/nssdb/ >/dev/null
EOF
```

#### 三、发送邮件

**1.通过文件内容发送**

发送命令：

mail -s 'mail test' [xxx@yyy.com](http://mailto:xxx@yyy.com) < con.txt （"mail test"为邮件主题，xxx@yyy.com为收件人邮箱，con.txt保存邮件内容）

例子： 

```
mail -s 'backup over' 13161168831@163.com</tmp/mail.txt >/dev/null 2>&1
```

**2.通过管道符直接发送**

发送命令：

```
echo "this is my test mail" | mail -s 'mail test' xxx@yyy.com >/dev/null 2>&1
```



### 16.1.2 mutt邮件工具

**1.介绍**

安装完mutt后，在`/usr/share/doc/mutt/*`下有份很好的手册

mutt是一个基于ncurse的Email客戶端。即是一个邮件管理程序。事实上，我们通常所指的mutt是一套邮件处理工具链，mutt作为邮件管理工具，是这套工具链的核心。在这套工具链中，一般还包括以下工具：

- offlineimap 是采用 IMAP 协议处理邮件；
- getmail 采用 POP3 协议处理邮件；
- procmail 用来过滤邮件；
- msmtp 用来发送邮件

**2.参数及使用**

```
mutt [-hnpRvxz][-a<文件>][-b<地址>][-c<地址>][-f<邮件文 件>][-F<配置文件>][-H<邮件草稿>][-i<文件>][-m<类型>] [-s<主题>][邮件地址]
```

参　数：

```
　-a　<文件>　在邮件中加上附加文件。
　-b　<地址>　指定密件副本的收信人地址。
　-c　<地址>　指定副本的收信人地址。
　-f　<邮件文件>　指定要载入的邮件文件。
　-F　<配置文件>　指定mutt程序的设置文件，而不读取预设的.muttrc文件。
　-h　显示帮助。
　-H　<邮件草稿>　将指定的邮件草稿送出。
　-i　<文件>　将指定文件插入邮件内文中。
　-m　<类型>　指定预设的邮件信箱类型。
　-n　不要去读取程序培植文件(/etc/Muttrc)。
　-p　在mutt中编辑完邮件后，而不想将邮件立即送出，可将该邮件暂缓寄出。
　-R　以只读的方式开启邮件文件。
　-s　<主题>　指定邮件的主题。
　-v　显示mutt的版本信息以及当初编译此文件时所给予的参数。
　-x　模拟mailx的编辑方式。
　-z　与-f参数一并使用时，若邮件文件中没有邮件即不启动mutt。
```

3.安装

```
yum -y install mutt
```

发送格式

```
mutt xxx@163.com -s "itdhz数据备份" -a /home/backup/itdhz.sql
或者
echo "test" | mutt xxx@163.com -s "itdhz数据备份" -a /home/backup/itdhz.sql
```

这段代码表示，发送邮件到 xxx@163.com 这个邮箱，邮件主题是“itdhz数据备份”，邮件中包含附件/home/backup/itdhz.sql。如果要发送多个附件，需要在每个附件前加 -a 参数。





## 16.2 文件备份并邮件报警

备份常用的命令：cp，tar，scp，rsync



**1.备份单个文件或目录**

```shell
[root@ mysql-slave scripts]# vim dir_bk.sh
#!/bin/bash

dir='/etc/httpd'
backupdir='/data/backup'

if [ ! -d "$backupdir" ]; then
  mkdir -p "$backupdir"
fi

tar -zcf $backupdir/httpd-$(date +%F).tar.gz $dir &>/dev/null
if [ $? -eq 0 ]
then
  echo "$dir backup successed" | mail -s 'backup' 13161168831@163.com  >/dev/null 2>&1
else
  echo "$dir backup failed" | mail -s 'backup' 13161168831@163.com  >/dev/null 2>&1
fi
```



**2.备份多个文件或目录**

**定义需要备份的目录文件：**

```shell
[root@ mysql-slave scripts]# cat /opt/scripts/dirs.txt
/etc/httpd
/etc/vsftpd

```

**脚本编写：**

```shell
[root@ mysql-slave scripts]# vim dir_bk.sh
#!/bin/bash

dir=$(cat /opt/scripts/dirs.txt)
backupdir='/data/backup'

if [ ! -d "$backupdir" ]; then
  mkdir -p "$backupdir"
fi

for i in $dir
do
  bk_name=$(echo $i|awk -F '/' '{print $3}')
  tar -zcf $backupdir/$bk_name-$(date +%F).tar.gz $i &>/dev/null
if [ $? -eq 0 ]
then
  echo "$i backup successed" >>/opt/scripts/bk_successed.log
else
  echo "$i backup failed" >>/opt/scripts/bk_failed.log
fi
done

mail -s 'backup over' 13161168831@163.com</opt/scripts/bk_successed.log >/dev/null 2>&1
```

**计划任务：每天0点执行**

```
00 00  *  *  *  /bin/bash  /opt/scripts/dir_bk.sh
```



## 16.3 监控系统服务并邮件报警

分别检测ftp、httpd、mysql服务状态判断相关服务器状态，如没有运行则将服务启动

以下是数组的方式检查服务状态：

示例一：

```shell
[root@ c6m01 ~]# vim /opt/scripts/service_check.sh
#!/bin/bash

services=('vsftpd' 'sshd' 'httpd' 'mysqld')

for i in ${services[*]}
do
    num=$(ps -ef|grep $i|grep -v grep|wc -l)
    if [ $num -eq 0 ];then
       echo -e "\033[31m $i 服务未运行... \033[0m"
       /etc/init.d/$i restart  &>/dev/null && echo "$i 服务启动成功"
       sleep 3
    else
       echo "$i 服务正在运行中..."
    fi
done
```

示例二：

```shell
#!/bin/bash
services=$(cat /opt/scripts/services.txt)

for i in $services
do
   num=$(ps -ef|grep $i|grep -v grep|wc -l)
    if [ $num -eq 0 ];then
       echo -e "$i 服务未运行..."
       /etc/init.d/$i restart  &>/dev/null
       echo "$i 已经启动" | mail -s '系统服务状态' 13161168831@163.com >/dev/null 2>&1
    else
       echo "$i 服务正在运行中..."
    fi
done

```





## 16.4 服务器运行状态监控脚本



```shell
[root@ c6m01 ~]# vim /opt/scripts/monitor.sh
#!/bin/bash
nowday=`date +%Y%m%d`
cpuload=`uptime`
memuse=`free -m|grep "Mem"|awk '{print $3}'`
memfree=`free -m|grep "Mem"|awk '{print $4}'`
swapuse=`free -m|grep "Swap"|awk '{print $3}'`
swapfree=`free -m|grep "Swap"|awk '{print $4}'`
echo "当前时间：$nowday"
echo "CPU负载：$cpuload"
echo "内存已用：$memuse"
echo "内存剩余：$memfree"
echo "swap 已用$swapuse"
echo "swap 剩余$swapfree"
echo "磁盘使用情况"

df -h
echo "最近的十次登录"
last -10
```









































