# 用作者虚拟机进行测试
日期：9月初
## 1. 本地磁盘性能
读性能：
```
$ sudo hdparm -t --direct /dev/sda1

/dev/sda1:
  Timing O_DIRECT disk reads: 242 MB in  0.74 seconds = 325.91 MB/sec
```
写性能：
```
$ dd if=/dev/zero of=anof bs=1G count=1 oflag=direct
1+0 records in
1+0 records out
1073741824 bytes (1.1 GB) copied, 6.89196 s, 156 MB/s
```

## 2.网络性能
```
$ iperf -c localhost
------------------------------------------------------------
Client connecting to localhost, TCP port 5001
TCP window size: 2.5 MByte (default)
------------------------------------------------------------
[  3] local 127.0.0.1 port 49926 connected with 127.0.0.1 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0-10.0 sec  13.8 MBytes   11.5 Mbits/sec
```
11.5 Mbits/sec = 1.4375 MB/s

作者给的虚拟机好像swift就是单结点

## 3. swift中workers的配置
都为默认值
- proxy: 1
- account: auto
- container server: auto
- object server: auto

## 4.用cosbench进行测试
https://github.com/intel-cloud/cosbench/releases/download/v0.4.2.c4/0.4.2.c4.zip

注意：
1. 确保crystal已经启动
2. cosbench中unset http_proxy

### 4.1 cosbench命令
#### 启动cosbench
```
unset http_proxy
sh start-all.sh
sh stop-all.sh

sh start-driver.sh
sh start-controller.sh
sh stop-driver.sh
sh stop-controller.sh
```
#### 提交workload
```
sh cli.sh submit conf/workload-config.xml
sh cli.sh info
sh cli.sh cancel w1
```
### 4.2 测试正确性
1. 用一个workload来验证keystone是否正确连接，出现bad username and password错误，且尚未修复
2. 用一个workload来验证直接连接swift，swift auth或curl获得token和auth_url，注意其中token为全数字，测试正确

### 4.3 实验配置
|认证|tenant|username|password|url|
|:--|:--|:--|:--|:--|
|keystone|crystaltest|admin|admin|127.0.0.1:35357/v2.0|
|swift|||token|auth_url|

测试配置

|containers|object size|R/W/D|workers|
|:--|:--|:--|:--|
|1|1kB|80/15/5|1|
||4KB|100/0/0|4|
||64KB|0/100/0|8|
|20|128KB|||
||512KB||||

### 4.4 实验遇到的问题
1. keystone认证不通过（待解决）
2. 直接用swift进行测试的时候，其token的时间为1小时
解决方法：
参考 https://access.redhat.com/solutions/1527533
在 /etc/keystone/keystone.conf中修改
```
# Amount of time a token should remain valid (in seconds).
# (integer value)
expiration=86400
```
重启keystone，我是直接重启了机器
3. 虚拟机中/dev/loop0设备只有9G，运行没有一半就已经用了5G，如何拓展loop设备（待解决）
参考 https://askubuntu.com/questions/260620/resize-dev-loop0-and-increase-space
PS:不知道为什么测试运行完了，其中/dev/loop0从一开始的剩4.6G到最后还是4.6G
4. 运行过程中部分object有error，共496个（待解决）


## 5 实验结果
想使用cosbench-plot工具来对结果进行分析，但是只画出了部分图形

### cosbench-plot使用遇到的问题
1. io.open()函数报错no such file or directory
解决：
在window下面路径修改为BASE_OUTPUT='D:/code/CosPlotGraph/'，创建相应的文件夹或文件即可
