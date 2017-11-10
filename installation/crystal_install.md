# 安装4结点crystal的过程

## 1. 搭建4节点的keystone+swift
<font color="red">注意：在作者给的单脚本搭建中有一个dump.rdb估计是作者虚拟机里面的redis数据库内容</font>

### (1).创建4台虚拟机并进行基本配置


#### IP配置：
**版本1：**

|节点|内存|磁盘|网络1|网络2|用户名|密码|
|:----|:----|:----|:----|:----|:----|:----|
|controller|9G|64G|192.168.3.241|192.168.56.10|controller|123|
|object1|4G|30G|192.168.3.242|192.168.56.21|object1|123|
|object2|4G|30G|192.168.3.243|192.168.56.22|object2|123|
|compute1|4G|30G|192.168.3.120|192.168.56.20|compute1|123|
设置一个网卡为网桥模式，另一个为host-only，分别为enp0s3，enp0s8
将VirtualBox设置为网桥模式，ubuntu IP设置为：
```
#enp0s3
vim /etc/network/interfaces
iface enp0s3 inet static
address 192.168.3.214
netmask 255.255.255.0
gateway 192.168.3.215

#enp0s8
auto enp0s8
iface enp0s8 inet static
address 192.168.56.10
netmask 255.255.255.0
gateway 192.168.56.1

vim /etc/resolv.conf
添加
nameserver 202.114.0.242

vim /etc/resolvconf/resolv.conf.d/base
nameserver 202.114.0.242
```
其中将默认路由由enp0s8改为enp0s3，在/etc/rc.local中添加
```
route del default
route add default gw 192.168.3.241
```

**版本2：**

重新看官网文档，发现用nat实现,virtualbox的nat方式会导致虚拟机IP地址一样，故用的是nat network

|节点|内存|磁盘|网络1|网络2|用户名|密码|
|:----|:----|:----|:----|:----|:----|:----|
|controller|9G|64G|192.168.3.241|10.0.2.10|controller|123|
|compute1|4G|30G|192.168.3.242|10.0.2.20|compute1|123|
|object1|2G|30G||10.0.2.31|object1|123|
|object2|2G|30G||10.0.2.32|object2|123|
|block1|2G|20G||10.0.2.33|object2|123|
其中object1和object2分别有2个磁盘/dev/sdb和/dev/sdc，大小为10G.

配置如下：
```
auto enp0s3
iface enp0s3 inet static
address 10.0.2.10
netmask 255.255.255.0
gateway 10.0.2.1
```
注意：
<font color=" #0099ff">ping不同的话，先看看能否ping通10.0.2.1和202.114.0.242,排除dns问题，再就是检查/etc/rc.local中是否设置了默认route</font>
#### ssh设置：
由于我在安装时就选择了ssh，故没有另行安装

#### 设置免密码sudo
在/etc/sudoers中添加
```
controller ALL=(ALL:ALL) NOPASSWD: ALL
```
#### 安装好后制作镜像以防万一

## 2.安装keystone和swift(pika+ocata文档)
问题1：# su -s /bin/sh -c "keystone-manage db_sync" keystone
不是很懂这句话的意思，将用户切换为keystone并用/bin/sh执行“”中的内容？但是keystone这个用户在哪里呢？

问题2：fernet是干什么的，为什么安装keystone时用户要用
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone

问题3：
验证
```
$ openstack --os-auth-url http://controller:35357/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name admin --os-username admin token issue
$ ....
expecting to find domain in project. the server could not comply with the request since it is eit malformed or otherwise incorrect.
```
报错。
日志内容为：
```
validationerror:expecting to find domain in project. the server could not comply with the request since it is either malformed or otherwise incorrect.
```
解决方法：

参考1：https://zhidao.baidu.com/question/1051442375669524299.html

>我的环境：CentOS7 + OpenStack Liberty
这个问题可能是因为我把controller的localhost改成controller了，在/etc/hostname中
那么这个问题出现之后，我在每个服务中都需要使用两个database的认证
vi /etc/keystone/keystone.conf
connection = mysql://keystone:pass@controller/keystone
connection = mysql://keystone:pass@localhost/keystone # new line added to suppress HTTP 500 error
After that run these commands again to reflect keystone.conf changes made,
Populate the database tables for the Identity Service:
su -s /bin/sh -c "keystone-manage db_sync" keystone
Restart the Identity Service: sudo systemctl enable openstack-keystone.service sudo systemctl start openstack-keystone.service


虽然我localhost的设置确实错了，但是上面参考没解决，后来重新安装了几次，最后一次好了。没有添加localhost的connection，没有注销token_auth(没有找到admin_token_auth)

### 各个服务user的密码
glance = 123
nova = 123
keystone = 123
placement = 123
neutron = 123

admin,demo的密码基本都是123
除了mysql，rabbit等保持文档一致，比如KEYSTONE_DBPASS

>完成日期：2017年9月8日
