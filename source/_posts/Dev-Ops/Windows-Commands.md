---
title: Windows 常用命令 
description: 收集Windows中常用的命令包括：设置环境变量; 查看服务; 查看进程; 列出文件夹名字等;
---
## 设置环境变量
doc 窗口设置环境变量
```
set MAVEN_OPTS=-Xmx1024m -XX:MaxPermSize=512m
```

## 删除服务
删除服务名为mysql的服务： `sc delete mysql`
## 端口相关
### 端口占用的应用的PID

```
netstat -aon|findstr "8599"
```
结果如下： （PID为2948）
```
C:\Documents and Settings\XPMUser>netstat -aon|findstr "8599"
  TCP    0.0.0.0:8599           0.0.0.0:0              LISTENING       2948
```
### 对应PID的进程
```
tasklist|findstr "2948"
```
结果如下
```
C:\Documents and Settings\XPMUser>tasklist|findstr "2948"
tomcat6.exe                 2948 RDP-Tcp#4               0     44,072 K
```
或者：打开任务管理器，切换到进程选项卡，在PID一列查看2720对应的进程是谁，如果看不到PID这一列，点击查看--->选择列，将PID(进程标示符)前面的勾打上，点击确定。
### 结束进程
结束该进程：在任务管理器中选中该进程点击”结束进程“按钮，或者是在cmd的命令窗口中输入：taskkill /f /t /im Tencentdl.exe。

## 列出文件夹下面的文件名称
创建一个bat文件, 加入下面的内容。 将这个bat文件放入文件夹内运行即可。
```
DIR *.* /B> LIST.TXT
```