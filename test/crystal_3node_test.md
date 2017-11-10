# crystal3结点测试

## 1.swift测试
目前先安装好了[keystone+swift](https://github.com/sdu-merrye/study/blob/master/installation/crystal_install.md)，先对swift和xfs性能做测试。

### 1.1 xfs磁盘性能测试
**HDD-Test1**
```
hdparm -Tt /dev/sdb
```
结果：
```
/dev/sdb:
 Timing cached reads:   17560 MB in  2.00 seconds = 8787.24 MB/sec
 Timing buffered disk reads: 2366 MB in  3.00 seconds =  788.34 MB/sec
```

**fio-Test2**

fio进行测试，测试地点为object1和object2上的磁盘。
测试结果：


### 1.2 swift性能测试
cosbench进行测试，controller在controller结点上，driver在compute1结点上。

再次试验keystone认证，失败，所以还是采用的swift认证
