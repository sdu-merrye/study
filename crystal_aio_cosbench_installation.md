# crystal ALL IN ONE 安装以及cosbench安装

## 1.crystal all in one安装
（1）从https://github.com/Crystal-SDS/INSTALLATION 代码(我直接root进行安装的)
```
git clone https://github.com/Crystal-SDS/INSTALLATION.git
chmod 777 install_aio.sh
```
修改/etc/sudoers,将swift加入
(2) 先单独执行
```
#!/bin/bash
printf "\nStarting Installation. The script takes long to complete, be patient!\n"
printf "See the full log at /tmp/crystal_aio.log\n\n"

#########  PASSWORDS  #########
MYSQL_PASSWD=root
RABBITMQ_PASSWD=openstack
KEYSTONE_ADMIN_PASSWD=keystone
MANAGER_PASSWD=manager
###############################

echo controller > /etc/hostname
echo -e "127.0.0.1 \t localhost" > /etc/hosts
IP_ADRESS=$(hostname -I | tr -d '[:space:]')
echo -e "$IP_ADRESS \t controller" >> /etc/hosts
```
将主机名改为controller之后重启，使之生效，否则后面会报sudo cann't resolv host ubuntu之类的错误。

(3) 注释到
```
# Install Storlets
git clone https://github.com/openstack/storlets >> /tmp/crystal_aio.log 2>&1
pip install storlets/ >> /tmp/crystal_aio.log 2>&1
cd storlets
```
这里，后面的先不要了。
执行一次
```
./install_aio.sh
```
(4) 修改storlet中的链接
- src/Java/build.xml中的http://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/json-simple/json_simple-1.1.jar 替换为国内链接http://www.docjar.com/jar/json_simple-1.1.jar
- 将install/storlets/role/docker_base_jre_image/tasks/ubuntu_16.04_jre8.yml中的json-simple的链接也替换为http://www.docjar.com/jar/json_simple-1.1.jar
- 同上，在ubuntu_16.04_jre8.yml中将http://www.slf4j.org/dist/slf4j-1.7.7.tar.gz 的链接替换为http://mirrors.ibiblio.org/ovirt/pub/ovirt-3.5/src/slf4j/slf4j-1.7.7.tar.gz

(5) 将前面执行过的代码从install_aio.sh中删除，执行后面剩余的代码
```
./install_aio.sh
```
执行成功。
(6) 测试swift
测试遇到错误如下：
```
root@controller:~# . crystal-openrc
root@controller:~# swift list
Account GET failed: http://controller:8080/v1/AUTH_e9e2647ff41f41d88e7794991cf6f9de?format=json 500 Internal Error   An error occurred
Failed Transaction ID: txd805958fd6b544c191903-0059a4c4a8
```
解决：
```
vim /etc/swift/account-server.conf
#bind_ip = 127.0.0.1
bind_ip = 192.168.0.1（你的ip）
swift-init all restart
```
查看swfit list或swift stat可以看到执行成功

(7) 查看网站
在宿主机上访问controller/horizon即可看到登陆界面
用户名和密码都为manager

## 2.上传metrics和filter
(1) 在宿主机中下载metric和filter-sample.git
```
[root@localhost Downloads]# git clone https://github.com/Crystal-SDS/metric-middleware.git
[root@localhost Downloads]# git clone https://github.com/Crystal-SDS/filter-samples.git
```
(2) 打包和上传filter
```
$cd storlet_compression/
$mkdir lib
$cp /home/crystal/strolets/Engine/SCommon/bin/SCommon.jar ./lib/
$ant bulid
```
用crystal网页进行上传fitler和metrics
上传fitler的class name直接就是前面的class name，相关设置如下
![image](https://github.com/sdu-merrye/study/blob/master/picture/crystal_metric_1.png)
![image](https://github.com/sdu-merrye/study/blob/master/picture/crystal_metric_2.png)

但是进行上传报错，metrics和filter都是这样.

尝试:
安装liberasurecode及相关库
sudo pip install jupyter
liberasurecode安装：
https://caden16.github.io/%E5%AF%B9%E8%B1%A1%E5%AD%98%E5%82%A8/ubuntu16.04%E6%90%AD%E5%BB%BAopenstack-swift%E5%8D%95%E6%9C%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/

重新执行./install_aio.sh
