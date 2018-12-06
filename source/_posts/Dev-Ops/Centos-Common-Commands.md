---
title: Centos 常用命令
sort: 1
description: Centos 常用命令
...

> 以下命令仅在centos7上验证过

### 软件安装
#### yum 安装
示例如下：
```
sudo yum install elasticsearch
```

#### rpm  包安装
```
sudo rpm -ivh kibana-4.6.6-x86_64.rpm  // 安装后通过 sudo service kibana start 来启动
```

#### 查看安装程序路径
```
sudo rpm -ql kibana  // 查看到安装在了/opt/kibana
```


### 开放端口

```
$ sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
$ sudo firewall-cmd --reload
```

### netstat 使用
#### 查看某个服务是否在运行

```
sudo netstat -aple | grep nginx
```
只查看tcp或者udp的connections需要添加-t参数: `sudo netstat -nplt` 
> 更多更实用的netstat命令参考：[Linux netstat 命令示例](http://www.binarytides.com/linux-netstat-command-examples/)

### 查看centos 版本
cat /etc/centos-release
**设置环境变量**

```
export KAFKA_HOME=/home/osboxes/kafka_2.10-0.10.0.1
echo $KAFKA_HOME
```
### 磁盘空间

```
df -h
```
```
free
```
### 查看目录所占空间大小

```
du -smh *
```
### 统计文件夹下面的文件数量
```
ls -1 | wc -l
```
### grep 搜索文件内容
**指定的文件类型中查找**
当前子目录中查找： `grep -r abcd *.properties` 当前子目录递归查找含有`abcd` 的*.properties 文件
指定目录及子目录中查找：`grep -r 3306 /home/okchem/storage92g/srm/ ` 

### 拷贝整个文件夹
```
cp -avr /home/vivek/letters /usb/backup
```
### 用户和组
> /etc/group file that lists all users groups  可以使用cut命令列出来`cut -d: -f1 /etc/group`

**查看当前用户的group:	** `$ groups`
**查看用户的group:	** `$ groups root` `id -Gn root`
**添加用户到组:	** `$ sudo usermod -a -G osboxes nginx` 添加用户nginx到组osboxes  `usermod -a -G <groupname> username` 添加完成请用`groups <username> ` 来验证
**获取组的所有用户:	** `getent group kibana`

其他有用资源： [CentOS7之新建用户与SSH登陆](https://segmentfault.com/a/1190000004141370)
### 文件权限
#### 修改文件[夹]owner
chown 代表change owner；`chown --help` 提供了更详细的信息
```
sudo chown -R okchem:root /ebs
```
#### 修改文件[夹]访问权限
chmod 代表change mode; 
例如：`chmod 644 important.txt` owner可读可写,group可读，others可读
> First position refers to the user. Second refers to the group of the user, and the third refers to all others.4 = read 2 = write 1 = execute

文件权限更详细的解释可以参考：[Linux File Permissions](https://www.pluralsight.com/blog/it-ops/linux-file-permissions)
### PS 命令
#### 查看java进程 `ps -ef|grep java`
#### 产看进程的详细信息 `ps -auxwe | grep subscribe`

### 命令行快捷键
**CTRL-a** 光标移至行首
**CTRL-e** 光标移至行尾
**CTRL-u** 删除整行
**CTRL-h**	删除光标前字符

### gzip / gunzip
 压缩单个文件 `gzip fileName` 压缩后的名字=原文件名字加上后缀.gz
 解压缩单个文件`gunzip filename` 或者 `gzip -d filename`
gzip 不能用来压缩整个文件夹至一个.gz 文件。压缩整个文件夹请参考targ + gzip 命令 即： `tar -z`命令。
> gzip -r  dictName 命令会压缩整个文件夹dictName 里面的所有文件，每个文件被压缩成一个单独的*.gz 文件
### tar 命令
gzip / bzip2 是用来压缩单个文件， tar是用来归档。 所以tar结合gzip/bzip2 可以方便的进行整个文件夹的压缩及归档。
压缩整个文件夹`tar -zcvf outputFileName folderToCompress` 
[Examples](https://www.tecmint.com/18-tar-command-examples-in-linux/)
> bzip2 

### sftp 命令
#### sftp登录
```
sftp  name@123.21.331.1
```
#### sftp 下载文件夹
```
get -r  folder  /home/joe/
```
#### sftp 上传文件
```
put  /name1.html  /name2/
```

### crontab 命令
创建执行任务， 添加cron job
参考cronjob `crontab -l`
编辑cronjob `crontab -e`

```
0 1 * * * /data/scripts/mysql-job.sh A
20 1 * * 0 /data/scripts/mysql-job.sh I
```
两个cron job 分别：

 1. 每天1点执行
 2. 每周日1点20 执行

参考：crontab 时间可以参考： https://www.cnblogs.com/intval/p/5763929.html