# 收集本人安装遇到的错误

## 1.ssh连接
```
[root@localhost Downloads]# scp root@192.168.3.241:/root/storlets/src/java/SCommon/bin/SCommon.jar .
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
df:3b:b2:21:a4:b6:4b:5e:29:28:3e:7b:8f:ad:70:12.
Please contact your system administrator.
Add correct host key in /root/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /root/.ssh/known_hosts:3
ECDSA host key for 192.168.3.241 has changed and you have requested strict checking.
Host key verification failed.
```
解决方法：
vim /root/.ssh/known_hosts 删除其中的3.241的IP

## 2. erasurecode.c:30:18: fatal error: zlib.h: No such file or directory
sudo apt-get install zlib1g-dev

## 3.在crystal中用cosbench测试时，遇到http 1.1 503 service unavailable错误
解决方法：
可能是由于swift的loopback设备空间不足的问题
首先
```
sudo losetup /dev/loop0
sudo dd if=/dev/zero bs=1MiB of=/path/to/file conv=notrunc oflag=append count=xxx
