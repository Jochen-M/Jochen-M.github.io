---
title: MongoDB 数据定时备份
date: 2017-11-29 20:39:52
tags:
    - MongoDB
    - Ubuntu
categories:
    - MongoDB
---

数据的安全性对于一个公司来说至关重要，由于一些不可预测的故障，如突然断电、断网等，数据可能会丢失。这就要求我们定时备份数据。

这里记录如何用 **crontab** 定时任务，每天定时备份 **MongoDB** 的数据到另一台服务器。

### shell 脚本

mongodb_bak.sh:
```
#!/bin/sh
DUMP=/home/jochen/mongodb/bin/mongodump             # mongodump备份文件执行路径
OUT_DIR=/home/jochen/mongodb_bak/mongodb_bak_now    # 临时备份目录
TAR_DIR=/home/jochen/mongodb_bak/mongodb_bak_list   # 备份存放路径
DATE=`date -d "today" +"%Y-%m-%d-%H-%M-%S"`         # 获取当前系统时间，作为文件名的一部分
DAYS=7                                              # DAYS=7代表删除7天前的备份， 即只保留最近7天的备份
TAR_BAK="mongodb_bak_$DATE.tar.gz"                  # 最终保存的数据库备份文件名
cd $OUT_DIR
rm -rf $OUT_DIR/
mkdir -p $OUT_DIR/$DATE
mkdir -p $TAR_DIR/
$DUMP -o $OUT_DIR/$DATE                             # 备份全部数据库
tar -zcvf $TAR_DIR/$TAR_BAK $OUT_DIR/$DATE          # 压缩为.tar.gz格式
find $TAR_DIR/ -mtime +$DAYS -delete                # 删除7天前的备份文件
scp $TAR_DIR/$TAR_BAK jochen@123.123.123.123:/home/jochen/mongodb_bak/$TAR_BAK  # 通过 scp 发送至另一台服务器
```

### crontab 定时任务
利用 linux 系统自带的 crontab 定时任务，每天定时执行以上脚本。

新建定时任务：
```
$ crontab -e
```

在最后一行写上：
```
* 3 * * * /home/jochen/mongodb_bak.sh > /dev/null &
```
