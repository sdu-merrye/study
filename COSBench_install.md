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
参考网上这段话
>Hello Ravi,
>The v0.4.2.0 hasn’t been released yet, I assume you are cloning v0.4.2.0 code branch. In this case, you’d build the source code in eclipse, and run “pack.sh” to pack a binary package. Or, more easy way is to download the latest binary package (the .zip file) directly from https://github.com/intel-cloud/cosbench/releases/tag/v0.4.1.0.
>
>-yaguang

方法1:下载了V0.4.1.0的zip，解压后运行sh start-all.sh命令，运行成功

另外一段话也可参考：
>The root cause for the error you have seen is required bundles can't get loaded. Generally, I prefer user to use binary packages on https://github.com/intel-cloud/cosbench/releases.
>
>Recently, I saw a few cases from end user, they are trying to run cosbench after git cloning, either starting scripts by going to releases/ folder, or running pack.sh script (your case ), they are all unexpected. Normally, when you really want to run cosbench by git cloning, it suggests potentially you may want to modify some code, so  you need establish one development environment, the instructions are recorded in BUILD.md. then you need export all bundles in eclipse, after that, you can run pack.sh to generate your own package.
>
>The forum includes two sub folders, one is for user, another is for developer. If you have development related questions, I suggest you post it on cosbench-developer.
>
>-yaguang

方法二：自己构建开发环境，修改其中部分代码。

代码导入参考BUILD.md，其中介绍了开发环境和debug环境的部署，本人按照development environment方式配置后，确实大部分的错误被修复，但仍有很多错误，这里我还没解决。
