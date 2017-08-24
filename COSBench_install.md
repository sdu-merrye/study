#COSbench的安装及遇到的错误

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
