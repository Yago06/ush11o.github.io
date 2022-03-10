---
title: RaspberryPi
date: 2020-05-26 12:40:34
tags:
    - 操作指南
    - RaspberryPi
    - 树莓派
categories:
    - 操作指南
---

树莓派折腾清单

## 初始化树莓派

首先，你得有个树莓派，一张够大的TF卡（8G以上）。

### 安装Raspbian

[官网](https://www.raspberrypi.org/)下载Raspbian镜像，我使用的是`Raspbian Buster with desktop and recommended software`，另外两个没试过不知道好不好。

下载Win32 Disk Imager进行烧录，把`.img`文件烧录到TF卡里。

烧录完后把卡插进树莓派，就可以正常开机了。

按照个人喜好设定一下地区、时区、密码什么的。

到这里你就可以骄傲地向别人炫耀你有一台树莓派了！

### 基本配置

如果没有专门的一套显示器、键鼠给树莓派使用，先看看后文的“网络连接”和“SSH”，如果有就可以暂时跳过，直接插上卡开机，等用得到再回来看。

### 网络连接

#### 有线

有线就没什么好说的，插上网线就能用了。

#### Wifi

如果有键盘鼠标，可以插上TF卡进系统设置。

如果没有的话：

在**电脑**里打开TF卡，新建一个名为`wpa_supplicant.conf`的文件，打开在里面输入：

```
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="你的WIFI网络名称"
    psk="你的WIFI网络密码"
    key_mgmt=WPA-PSK
}
```

其中wifi名称和密码都是要保留引号的，就是只改中文部分。

注意这里的`key_mgmt`不论你的网络加密是`WPA-PSK`还是`WPA2-PSK`统一写`WPA-PSK`。

### SSH

####　方法一：

进系统在命令行输入

```
sudo /etc/init.d/ssh start
```
这种方法是临时的（重启后失效）

#### 方法二：

TF卡连到电脑，打开直接新建名为`SSH`的文件（无后缀）。

这样子就可以通过SSH来连接树莓派了，我使用的是`PuTTY`

[参考](https://www.jianshu.com/p/654ee08d2b3a)

### 换源

由于系统里默认的软件源受到墙的影响，速度不尽人意，建议（务必）换成墙内的软件源。

```
sudo sed -i 's#://raspbian.raspberrypi.org#s://mirrors.ustc.edu.cn/raspbian#g' /etc/apt/sources.list 
sudo sed -i 's#://archive.raspberrypi.org/debian#s://mirrors.ustc.edu.cn/archive.raspberrypi.org/debian#g' /etc/apt/sources.list.d/raspi.list
```

更新本地软件引索

```
sudo apt-get update
```

[参考链接](https://www.jianshu.com/p/67b9e6ebf8a0)

### 远程桌面连接

好像网络上有不少可以实现这个的软件，我捣鼓了许久只成功了这个（呜呜），所以推荐使用xrdp。

```
sudo apt-get install tightvncserver xrdp
```

然后使用windows自带的远程桌面连接就可以远程桌面连接了。

ps: WIN + R 里运行 mstsc 可以装逼又快速的开启远程桌面连接。

ps: 建议调整一下分辨率，不全屏的连接使用体验更佳。

* 待填坑：ftp；

## 软件篇

### Pi Dashboard

[link](https://make.quwj.com/project/10)

Pi Dashboard (Pi 仪表盘) 是一个开源的 IoT 设备监控工具，目前主要针对树莓派平台，也尽可能兼容其他类树莓派硬件产品。你只需要在树莓派上安装好 PHP 服务器环境，即可方便的部署一个 Pi 仪表盘，通过炫酷的 WebUI 来监控树莓派的状态！

目前已加入的监测项目有：

CPU 基本信息、状态和使用率等实时数据

内存、缓存、SWAP分区使用的实时数据

SD卡（磁盘）的占用情况

实时负载数据

实施进程数据

网络接口的实时数据

树莓派IP、运行时间、操作系统、HOST 等基础信息

**安装方法**

安装共2步，首先安装 Nginx 和 PHP。然后在 Nginx 目录通过 SFTP 或 GitHub 部署好本项目的程序。

#### 1.安装Nginx和PHP

```
sudo apt-get update
sudo apt-get install nginx php7.3-fpm php7.3-cli php7.3-curl php7.3-gd php7.3-cgi
sudo service nginx start
sudo service php7.3-fpm restart
```
如果安装成功，可通过 `http://树莓派IP` 访问到 Nginx 的默认页。Nginx 的根目录在 `/var/www/html`。
进行以下操作来让 Nginx 能处理 PHP。

```
sudo nano /etc/nginx/sites-available/default
```

将其中的如下内容

```
location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
            }
```

替换为

```
location / {
index  index.html index.htm index.php default.html default.htm default.php;
}
 
location ~\.php$ {
fastcgi_pass unix:/run/php/php7.3-fpm.sock;
#fastcgi_pass 127.0.0.1:9000;
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
include fastcgi_params;
}
```

保存退出

开启Nginx

```
sudo service nginx restart
```

#### 2.部署 Pi Dashboard

SFTP请参照前文的链接。

**Github部署**

```
sudo apt-get install git
cd /var/www/html
sudo git clone https://github.com/spoonysonny/pi-dashboard.git
```

即可通过 `http://树莓派IP/pi-dashboard` 访问部署好了的 Pi Dashboard。

同样如果页面无法显示，可以尝试在树莓派终端给源码添加运行权限，例如你上传之后的路径是 `/var/www/html/pi-dashboard` ，则运行

```
cd /var/www/html
sudo chown -R www-data pi-dashboard
```

### 花生壳

众所周知，当今社会很难搞到一个公网ip，这个时候就得用上内网穿透，而不够厉害的华华只能用一用花生壳这种比较友好的DDNS了。

[花生壳官网](https://hsk.oray.com/download/) 下载树莓派版本，通过smb或者ftp或者u盘拷或者直接在花生壳上下载，把那个.deb文件放到树莓派里面。

然后终端cd到对应目录，

```
dpkg -i phddns_rapi_3.0.4.armhf.deb
```

**注**：其中phddns_rapi_3.0.4.armhf.deb 很可能会因为~~时间的流逝~~版本更新发生变化，请手动换成安装包的名称

安装成功后，将显示树莓派的SN码、默认密码以及远程管理地址。

![](/images/20170222174817450.jpg)

然后使用

```
sudo phddns start
```

就可以开启花生壳了。

其他命令（我就不翻译了，应该看得懂）

```
phddns start/stop/enable/disable/restart/status/reset/version
```

卸载命令

```
dpkg -r phddns
```

至于怎么调试内网穿透，请移步花生壳官网。

### smb服务器

可以利用树莓派共享文件夹，在别的机子上通过smb协议读写树莓派上的文件，就像读写本机的文件一样。我这里是打算当成全家的一个共享硬盘，可以放一些电影、视频、番、音乐，在各个设备上轻松的访问。

首先安装samba

```
sudo apt-get install samba samba-common-bin
```

安装的时候会弹出一个窗口要求安装`dhcp-client`软件包，我也不知道是啥，点是就对了。

然后设置一个smb的账户

```
sudo smbpasswd -a pi
```
后面会跟着让我们输入密码，建议和树莓派账户的密码设成一样的，不容易记混。

接着创建一个目录，作为共享的文件夹。当然也可以不设，随便找一个现成的文件夹。

```
mkdir /home/pi/smbshare
```
mkdir 是创建文件夹的命令，后面的路径和文件夹名自己选一个喜欢的就好。

还要给这个文件夹权限，不然无法读写。

```
sudo chmod 777 /home/pi/smbshare
```

接着修改一下配置文件，让路径可用。

```
sudo nano /etc/samba/smb.conf
```

翻到整个文件最后面添加上这些内容

```
[share]
path = /home/pi/smbshare
public = yes
writable = yes
valid users = pi
create mask = 0644
force create mode = 0644
directory mask = 0755
force directory mode = 0755
available = yes
```

重启smb服务

```
sudo systemctl restart smbd
```

然后smb就可以用了，怎么在别的设备上挂载smb服务器就不是这里要教的了，不懂请出门左转Google。

这里记录一下 CrystalDiskMark 跑分：
直接写TF卡，电脑Wifi速率400MB，树莓派有线速率1G。
| MB/s | Read | Write |
| - | - | - |
| Seq | 25.88 | 21.19 |
| 512K | 23.69 | 17.24 |
| 4K| 1.763 | 1.160 |
| 4K QD32| 2.116 | 7.880 |

瓶颈应该在于TF卡的读写速度而不是网速，这个速度不是非常快，但满足日常的需求应该绰绰有余了。（等我哪天学会了怎么挂硬盘并买得起固态的时候再测一测极限速度）

* 待填坑：挂载u盘、硬盘；

### kodi

kodi是一个媒体播放器，我打算用于连接电视做一个机顶盒，目前预期功能是利用UPnP媒体服务器和smb服务器播放局域网内的媒体。

安装kodi：

```
sudo apt-get install kodi
```
然后运行kodi

```
kodi
```

没了。

一开始kodi很慢，后面发现是wifi跟不上视频的码率，所以换了有线连接，然后就可用了。
（但说实话除了能遥控之外并没有比直接用Raspbian里的smb配vlc播放视频来的方便）

### moonlight

这个是配合Nvidia卡进行串流的应用，可以用来远程利用电脑的算力来打树莓派带不动的游戏。
甚至可以通过毒瘤方法把整个系统当作一个游戏来远程使用（失败了 *见后文）。

[github教程](https://github.com/irtimmer/moonlight-embedded/wiki/Packages)

```
sudo nano /etc/apt/sources.list
```

添加源：

```
deb http://archive.itimmer.nl/raspbian/moonlight buster main
```

如果直接update会显示无法认证源，默认禁用。所以添加签名：

```
wget http://archive.itimmer.nl/itimmer.gpg
sudo apt-key add itimmer.gpg
```

然后安装moonlight。

```
sudo apt-get update
sudo apt-get install moonlight-embedded
```

接着匹配电脑，x.x.x.x表示电脑局域网ipv4的ip。

```
moonlight pair x.x.x.x
```

开启串流。

```
moonlight stream x.x.x.x
```

可选的选项，（插入在`stream`与`x.x.x.x`之间）：

```
-720 Use 1280x720 resolution [default]
-1080 Use 1920x1080 resolution
-width Horizontal resolution (default 1280)
-height Vertical resolution (default 720)
-30fps Use 30fps (好像这个和下面的那个60fps失效了)
-60fps Use 60fps [default]
-bitrate Specify the bitrate in Kbps
-packetsize Specify the maximum packetsize in bytes
-app Name of app to stream
-nosops Don't allow GFE to modify game settings
-input Use as input. Can be used multiple times
-mapping Use as gamepad mapping configuration file (use before -input)
-audio Use as ALSA audio output device (default sysdefault)
-localaudio Play audio locally
```

几个快捷键

* Ctrl + Alt + Shift + Z 切换鼠标指针捕获
* Ctrl + Alt + Shift + X 切换全屏和窗口模式
* Ctrl + Alt + Shift + Q 退出流媒体对话（最常用的）（游戏还在运行）

附：**失败原因**

本以为moonlight可以和win\macOS\iOS\Android一样通过运行mstsc.exe来实现整个系统的串流，但是树莓派版的moonlight强制开启steam的big screen模式，就只能打游戏了。本来以为可以和steam link一样退出大屏幕模式，但是moonlight只有直接退出remote play这个选项，点了就直接停止串流了。那么为什么不用steam link呢？我steam link出了一点小bug，点开没有反应，所以只能将就着用moonlight了。

## 随手记的一些笔记

### [Linux指令](/2020/05/29/Linux常用指令/)

### 处理器温度

1. 树莓派的CPU温度信息位于文件 /sys/class/thermal/thermal_zone0/temp中，该文件为一个只读文件。 
2. 文件里的数字除以1000，单位为摄氏度。

可以用`watch`指令循环打印。

```
watch -n 1 cat /sys/class/thermal/thermal_zone0/temp
```

### 查看网口速率

```
ethtool eth0
```

其中 `Speed: 1000MB/s` 即为网口速率。

### 开机启动

```
sudo nano /etc/rc.local
```

`exit 0`前的代码都会在启动前被执行

# 剩下的慢慢来