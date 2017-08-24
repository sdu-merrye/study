# COSbench的安装及遇到的错误
安装过程参考自cosbenchUserGuide.pdf

1.安装jre和curl
```
sudo apt-get update
sudo apt-get install openjdk-7-jre
sudo apt-get install curl
```
在update时报错
```
W: Target Packages (main/binary-amd64/Packages) is configured multiple times in /etc/apt/sources.list.d/elastic-5.x.list:1 and /etc/apt/sources.list.d/elastic-5.x.list:2
W: Target Packages (main/binary-i386/Packages) is configured multiple times in /etc/apt/sources.list.d/elastic-5.x.list:1 and /etc/apt/sources.list.d/elastic-5.x.list:2
W: Target Packages (main/binary-all/Packages) is configured multiple times in /etc/apt/sources.list.d/elastic-5.x.list:1 and /etc/apt/sources.list.d/elastic-5.x.list:2
W: Target Translations (main/i18n/Translation-en_US) is configured multiple times in /etc/apt/sources.list.d/elastic-5.x.list:1 and /etc/apt/sources.list.d/elastic-5.x.list:2
W: Target Translations (main/i18n/Translation-en) is configured multiple times in /etc/apt/sources.list.d/elastic-5.x.list:1 and /etc/apt/sources.list.d/elastic-5.x.list:2
W: Target Packages (main/binary-amd64/Packages) is configured multiple times in /etc/apt/sources.list.d/elastic-5.x.list:1 and /etc/apt/sources.list.d/elastic-5.x.list:2
W: Target Packages (main/binary-i386/Packages) is configured multiple times in /etc/apt/sources.list.d/elastic-5.x.list:1 and /etc/apt/sources.list.d/elastic-5.x.list:2
W: Target Packages (main/binary-all/Packages) is configured multiple times in /etc/apt/sources.list.d/elastic-5.x.list:1 and /etc/apt/sources.list.d/elastic-5.x.list:2
W: Target Translations (main/i18n/Translation-en_US) is configured multiple times in /etc/apt/sources.list.d/elastic-5.x.list:1 and /etc/apt/sources.list.d/elastic-5.x.list:2
W: Target Translations (main/i18n/Translation-en) is configured multiple times in /etc/apt/sources.list.d/elastic-5.x.list:1 and /etc/apt/sources.list.d/elastic-5.x.list:2
```
解决：
由于/etc/apt/source.list.d/下的elastic文件中deb两行重复了，所以删除其中一行问题得到解决

2.安装COSbench
从github获取代码
```
git clone https://github.com/intel-cloud/cosbench.git
cd cosbench/release
chmod +x *.sh
unset http_proxy
sh start-all.sh
```
在start-all的时候报错：
```
root@controller:/home/swift/cosbench/release# sh start-all.sh
cat: VERSION: No such file or directory
Launching osgi framwork ...
Successfully launched osgi framework!
Booting cosbench driver ...
...........................................................
Starting    cosbench-log_    [ERROR]
...........................................................
Starting    cosbench-tomcat_    [ERROR]
...........................................................
Starting    cosbench-config_    [ERROR]
...........................................................
Starting    cosbench-http_    [ERROR]
...........................................................
Starting    cosbench-cdmi-util_    [ERROR]
...........................................................
Starting    cosbench-core_    [ERROR]
...........................................................
Starting    cosbench-core-web_    [ERROR]
...........................................................
Starting    cosbench-api_    [ERROR]
```
其中日志log/driver-boot.log显示为：
```
Error: Could not find or load main class org.eclipse.equinox.launcher.Main
```
解决办法：
