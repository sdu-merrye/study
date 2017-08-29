# Ubuntu16.04 安装Gnome桌面环境
转自http://blog.creke.net/696.html

用的第二种方法

一、安装全部Gnome桌面环境

Ubuntu系列桌面实际上有几种桌面应用程序，包括Ubuntu-desktop、Kubunut-desktop和Xubuntu-desktop。本文就以Ubuntu-desktop为例进行介绍，此方法操作最简单，但不建议在服务器上使用此方法，因为安装桌面相关软件太多。
```
sudo aptitude install ubuntu-desktop
或
sudo apt-get install ubuntu-desktop
```

二、自定义安装

1.安装x－windows的基础（必须）：
```
sudo apt-get install x-window-system-core
```
2.安装gnome基础（必须）：
```
sudo apt-get install gnome-core
```
3.安装中文显示(建议安装）：
sudo apt-get install language-pack-gnome-zh  #让gnome面板、菜单显示中文
sudo apt-get install language-pack-gnome-zh-base
sudo apt-get install language-pack-zh             #中文语言包
sudo apt-get install language-pack-zh-base
sudo apt-get install language-support-zh        #中文语言支持
sudo apt-get install scim        #scim中文输入法平台

4.安装登录管理器（一般不选）：
sudo apt-get install gdm
说明：
gdm（gnome display manager）即gnome图形界面显示管理器，还有kdm/xdm等，它将使您可以在启动时直接进入GUI桌面环境，而勿需通过 startx 来启动GUI。

5.安装新利得软件管理器（可选）:
```
sudo apt-get install synaptic
```

6.卸载gnome桌面环境：
sudo apt-get --purge remove liborbit2

7.进入图形界面：startx

8.退出图形桌面：ctrl + alt + backspace

9.默认启动字符界面
编辑文件 /etc/default/grub
把 GRUB_CMDLINE_LINUX_DEFAULT=”quiet splash”
改成GRUB_CMDLINE_LINUX_DEFAULT=”quiet splash text”
然后再运行”sudo update-grub”即可。
