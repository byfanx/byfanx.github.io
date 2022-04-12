---
title: 树莓派搭建Nextcloud私有云
route: raspberry-nextcloud
date: 2021-08-16 16:04:17
tags: [树莓派,服务器,Linux]
categories: 树莓派
image: /images/cover/raspberry-nextcloud.png
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;距离树莓派入手时间已经半年多，由于时间问题，也一直没有部署过像样的东西，刚好最近入手了一块硬盘，所以就用树莓派加移动硬盘来搭建一个个人的私有云吧。这里以树莓派raspbain 10 buster系统为例，安装Nextcloud私有云。


# 1. 树莓派换源

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于国内环境，软件的下载安装相对比较慢，所以更换安装源来提高下载速度，更换下载源后更新软件的速度相对比较慢，其中很快做其他的事情，自行怎么方便怎么来。

## 1.1 查看版本

网上许多教程都不是基于最新的raspbain buster来进行更换的，这里需要注意一下，更换源之前先查看一下系统版本。

![](查看版本.png)

## 1.2 修改源

```bash
# 备份并编辑source.listwenjian 
$ cp /etc/apt/sources.list /etc/apt/sources.back.list
$ vim /etc/apt/sources.list
# 注释所有内容，添加以下内容
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib


# 备份并编辑raspi.list文件
$ cp /etc/apt/sources.list.d/raspi.list /etc/apt/sources.list.d/raspi.back.list
$ vim /etc/apt/sources.list.d/raspi.list
# 注释所有内容，替换如下内容
deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main
```

## 1.3 更新源和软件

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

> 这个更新过程比较慢，建议这段时间可以到nextcloud官网中同时下载安装包，或进行其他不使用apt-get操作。

# 2. 安装软件

## 2.1 安装apache

```bash
# 1.安装命令
sudo apt-get install apache2

# 2.启动apache2
systemctl start apache2

# 3.设置apache2开机自启
systemctl enable apache2

# 附上其他命令
# 查看运行状态
systemctl status apache2
# 重启
systemctl restart apache2
# 停止
systemctl stop apache2
```

## 2.2 安装php

```bash
# 1.安装php
sudo apt-get install php libapache2-mod-php -y

# 2.安装其他组件
sudo apt-get -y install php-fpm php-cli php-json php-curl php-imap php-gd php-mysql php-xml php-zip php-intl php-imagick php-mbstring -y
```

## 2.3 安装mariadb

```bash
# 1.安装
$ sudo apt-get install mariadb-server -y

# 2.开启远程登陆权限
# 2.1 切换目录
$ cd /etc/mysql/mariadb.conf.d
# 2.2 找到修改权限的文件
$ grep -rn "skip-networking" *
# 显示如下：在50-server.cnf文件的第26行
50-server.cnf:26:# Instead of skip-networking the default is now to listen only on
# 2.3 编辑文件，注释掉  bind-address = 127.0.0.1
$ vim 50-server.cnf
```

接下来是修改数据库配置

```bash
# 直接回车 不需要输入密码
$ mysql -uroot -p

# oc_admin可替换成自定义的用户名，password可替换成自定义的密码
> create database nextcloud;
> CREATE USER 'oc_admin'@'%' IDENTIFIED BY 'password';
> GRANT ALL PRIVILEGES ON *.* TO 'oc_admin'@'%' WITH GRANT OPTION;

> flush privileges;
> CREATE USER 'oc_admin'@'localhost' IDENTIFIED BY 'password';
> GRANT ALL PRIVILEGES ON *.* TO 'oc_admin'@'localhost' WITH GRANT OPTION;
> flush privileges;
```

## 2.4 安装Nextcloud

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Nextcloud的安装包需要去官网进行下载，这里是[下载地址](https://nextcloud.com/install/#instructions-server)。需要下载`tar.bz2`包。

![](下载nextcloud.png)

下载完成后，需要上传到树莓派的`/var/www/html/`目录下。然后执行下面命令。

```bash
# 1.解压文件
tar jxf nextcloud-21.0.0.tar.bz2
# 2.添加data目录和授权
chown -R root:root nextcloud
# 3.进入nextcloud文件夹
cd nextcloud
# 4.创建数据文件夹
mkdir data
# 5.添加权限和授权
chown -R www-data:www-data data config apps
```

# 3. 初始化Nextcloud

## 3.1 初始化

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;完成上面步骤，就可以进行初始化Nextcloud了，在电脑浏览器中输入地址：`树莓派ip:8080/nextcloud`。比如：`192.162.1.110:8080/nextcloud`。则可以打开nextcloud的登陆界面，选择用户名和密码，以及输入数据库用户名和密码，完成设置。如下：

![](初始化nextcloud.png)

> 关于数据目录，这里先默认选择`/var/www/html/nextcloud/data`。至于使用挂载目录，后面会讲到。如果想在初始化时就更换为挂载目录，可直接查看[挂载外设](#guazai)。

如果你的初始化出现下面错误，说创建数据库用户失败。这是因为在第一次初始化nextcloud的时候会在`/var/www/html/nextcloud/config`中创建一个config.php文件，文件记录nextcloud的配置信息。如果是第一次初始化，config.php中记录的数据库用户名会变成起初连接数据库的用户名加1。这里只需要手动的把1删除，之后再重新进行初始化操作，便可完成。

```bash
'dbname' => 'nextcloud',
'dbhost' => 'localhost:3306',
'dbport' => '',
'dbtavleprefix' => 'oc_',
'mysql.utf8mb4' => true,
'dbuser' => 'oc_admin1',
```

如果登录出现不信任域名访问的错误，这是由于nextcloud的访问设置了白名单，所以在访问的时候需要添加白名单ip。编辑`config.php`文件，把要访问的ip或者域名添加进去。

![](不信任域名访问.png)

```bash
$ nano /var/www/html/nextcloud/config/config.php
'trusted_domains' => 
  array (
    0 => '192.168.1.110:8080',
    1 => 'xxx.com',
  ),
```

之后再进行访问，就出现登陆页面，通过设置的用户名和密码进行登录即可。首页如下。

![](首页.png)

## 3.2 其他配置

### 3.2.1 文件上传大小限制

```bash
# 1.编辑php.ini文件
$ vim /etc/php/7.3/apache2/php.ini
# 2.找到相关属性，按照下方修改
upload_max_filesize = 16G
post_max_size = 16G
max_input_time 3600
max_execution_time 3600

# 3.解决浏览器超时问题
$ a2dismod reqtimeout
# 4.重启apache2
$ systemctl restart apache2
```

<span id="guazai"></span>

### 3.2.2 挂载外设

挂载外设有两种情况，一是直接将硬盘挂载到数据目录`data`下面；二是将硬盘挂载到其他目录，把数据目录更换成挂载的目录。

**1、硬盘挂载到数据目录`data`下面**

```bash
# 1.备份data数据
$ cp -r /var/www/html/nextcloud/data  /home/data

# 2.清空data目录
$ rm -v /var/www/html/nextcloud/data/*

# 3.查看接入的硬盘
$ fdisk -l
Device     Start        End    Sectors  Size Type
/dev/sda1     34      32767      32734   16M Microsoft reserved
/dev/sda2  32768 3907026943 3906994176  1.8T Microsoft basic data

# 4.查看硬盘的uuid和类型
$ blkid /dev/sda2
/dev/sda2: LABEL="pi" UUID="DBK3-5F1C" TYPE="exfat" PARTLABEL="Basic data partition" PARTUUID="51ejn2c4-f99f-46e5-a7cn-ca4m4e8db6bc"
 # 5.挂载硬盘
$ mount -t exfat /dev/sda2 /var/www/html/nextcloud/data
# 取消挂载  umount /dev/sda2

# 6.开机自动挂载
# 6.1 编/etc/fstab文件 
$ vim /etc/fstab
# 6.2 添加以下内容，保存退出即可
UUID="DBK3-5F1C" /var/www/html/nextcloud/data exfat defaults,nofail 0 0

# 7.之后再将data文件夹下所有文件复制回来
$ cp -r  /home/data /var/www/html/nextcloud/data
```

**2、硬盘挂载到其他目录**

```bash
# 1.创建挂载目录
$ mkdir /home/nextcloud
# 添加授权
chown -R root:root /home/nextcloud

# 2.挂载目录
$ mount -t exfat /dev/sda2  /home/nextcloud

# 3.开机自动挂载
# 3.1 编/etc/fstab文件 
$ vim /etc/fstab
# 3.2 添加以下内容，保存退出即可
UUID="DBK3-5F1C" /var/www/html/nextcloud/data exfat defaults,nofail 0 0

# 4.将data文件夹复制到挂载目录
$ cp -r /var/www/html/nextcloud/data/  /home/nextcloud/

# 5.修改nextcloud配置文件
$ vim /var/www/html/nextcloud/config/config.php
# 将数据目录更改
'datadirectory' => '/home/nextcloud/data'

# 6.重启apache2
$ systemctl restart apache2

# 7.如果访问时出现目录没有权限的情况，编辑nextcloud配置文件
$ vim /var/www/html/nextcloud/config/config.php
# 添加下面内容，保存重启apache2服务即可
'check_data_directory_permissions' => false
```

# 4. 性能优化

## 4.1 配置redis

```bash
# 1.安装redis
$ sudo apt-get install redis-server

# 2.修改redis配置
$ vim /etc/redis/redis.conf
# 修改daemonize 为 yes，取消以下内容的注释
# unixsocket /var/run/redis/redis-server.sock
# unixsocketperm 777

# 3.授权redis
$ usermod -g www-data redis
$ chown -R redis:www-data /var/run/redis
$ redis-server /etc/redis.conf

# 4.重启redis服务
$ service redis-server restart

# 5.安装apcu
$ sudo apt-get install php-apcu
$ sudo apt-get install php-redis

# 6.修改nextcloud的配置文件
$ vim /var/www/html/nextcloud/config/config.php
# 添加如下内容
'memcache.local' => '\OC\Memcache\APCu',
'memcache.locking' => '\OC\Memcache\Redis',
'redis' => 
array (
    'host' => 'localhost',
    'port' => 6379,
   ),
   
# 7.重启apache2服务
$ systemctl restart apache2
```

## 4.2 提高swap容量

```bash
# 1.修改配置文件
$ vim /etc/dphys-swapfile
# 修改字段CONF_SWPSIZE 值，默认为100，这里修改为 2048

# 2.重启swap
$ /etc/init.d/dphys-swapfile restart
```

## 4.3 提高sd卡速度

```bash
# 1.修改配置文件
$ sudo vim /boot/config.txt
# 加入下面一行内容
dtparam=sd_overclock=100

# 2.安装hdparm
$ hdparm -tT /dev/mmcblk0
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上就是用树莓派搭建Nextcloud私有云的具体步骤，用来当作笔记防止下次忘记，后续有什么新的功能在陆续添加。这次就到这了！
