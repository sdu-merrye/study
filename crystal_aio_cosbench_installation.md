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
./install_libs.sh >> /tmp/crystal_aio.log 2>&1
mkdir /home/docker_device/scripts >> /tmp/crystal_aio.log 2>&1
chown swift:swift /home/docker_device/scripts >> /tmp/crystal_aio.log 2>&1
cp scripts/restart_docker_container /home/docker_device/scripts/ >> /tmp/crystal_aio.log 2>&1
cp scripts/send_halt_cmd_to_daemon_factory.py /home/docker_device/scripts/ >> /tmp/crystal_aio.log 2>&1
chown root:root /home/docker_device/scripts/* >> /tmp/crystal_aio.log 2>&1
chmod 04755 /home/docker_device/scripts/* >> /tmp/crystal_aio.log 2>&1
```
这里
