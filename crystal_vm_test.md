# 用作者虚拟机进行测试

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
