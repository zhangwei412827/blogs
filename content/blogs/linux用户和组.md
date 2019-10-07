+++
title= "Linux用户和组"
date= 2019-09-08T10:11:49+08:00
tag= ["linux"]
+++
### 新建系统组mariadb，新建系统用户mariadb
### 例子1： 要求其没有家目录,且shell为/sbin/nologin;创建组用groupadd命令；
创建系统组：groupadd -r groupName

groupadd [options] group

创建用户用useradd命令；

useradd [options] LOGIN

创建系统用户：useradd -r userName

不创建家目录需要用到-M选项

为用户指定shell用-s选项

为用户指定组用-g选项

所以该题的答案为：

* 先创建系统组mariadb，groupadd -r mariadb
* 创建系统用户，useradd -r mariadb -s /sbin/nologin -M -g mariadb

### 例子2：新建GID为5000的组mageedu，新建用户gentoo，要求其家目录为/users/gentoo，密码同用户名

为组制定groupid用-g选项；

为用户设置密码用-p选项；

所以答案为：

* groupadd -g 5000 mageedu
* useradd -s /users/gentoo gentoo -p gentoo

### 例子3：新建用户fedora，其家目录为/users/fedora，密码同用户名
答案为：useradd fedora -s /users/fedora

### 例子4：新建用户www，其家目录为/users/www；删除用户www，但保留其家目录

删除用户用userdel；

如果要同时删除用户家目录则用-f选项；

答案为：

* useradd www -s /users/www
* userdel www

### 例子5：为用户gentoo和fedora新增附加组mageedu

更改用户信息用usermod命令；

usermod [options] LOGIN；

指定附加组用-G选项；

-G, --groups GROUP1[,GROUP2,...[,GROUPN]]]

答案为：

* usermod -G mageedu gentoo
* usermod -G mageedu fedora 

### 例子6：复制目录/var/log至/tmp/目录，修改/tmp/log及其内部的所有文件的属组为mageedu，并让属组对目录本身拥有写权限；

复制文件用cp命令；

cp [OPTION]... [-T] SOURCE DEST
cp [OPTION]... SOURCE... DIRECTORY
cp [OPTION]... -t DIRECTORY SOURCE...

修改文件权限用chmod命令；

chmod [OPTION]... MODE[,MODE]... FILE...
chmod [OPTION]... OCTAL-MODE FILE...
chmod [OPTION]... --reference=RFILE FILE...

修改目录下所有文件用-R选项；

答案为：

* cp -r /var/log /tmp
* chmod -g mageedu
* chmod -r g+w 
