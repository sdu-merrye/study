# 本人安装crystal的过程

# 1.创建4台虚拟机
|节点|内存|磁盘|网络|用户名|密码|
|:----|:----|:----|:----|:----|:----|
|controller|9G|64G|192.168.3.241|controller|123|
|object1|4G|30G|192.168.3.242|object1|123|
|object2|4G|30G|192.168.3.243|object2|123|
|object3|4G|30G|192.168.3.120|object3|123|

(1) IP配置：
将VirtualBox设置为网桥模式，ubuntu IP设置为：
```
vim /etc/network/interfaces
iface enp0s3 inet static
address 192.168.3.214
netmask 255.255.255.0
gateway 192.168.3.215

vim /etc/resolv.conf
添加
nameserver 202.114.0.242

vim /etc/resolvconf/resolv.conf.d/base
nameserver 202.114.0.242
```
(2) ssh设置：由于我在安装时就选择了ssh，故没有另行安装

(3) 安装好后制作镜像以防万一
