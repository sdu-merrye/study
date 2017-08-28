# crystal ALL IN ONE 安装以及cosbench安装

## 1.crystal all in one安装
（1）从https://github.com/Crystal-SDS/INSTALLATION 代码(我直接root进行安装的)
```
git clone https://github.com/Crystal-SDS/INSTALLATION.git
chmod 777 install_aio.sh
```
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
