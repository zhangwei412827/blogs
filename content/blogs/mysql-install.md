+++
title= "Mysql Install"
date= 2019-08-23T11:47:07+08:00
tags=["mysql"]
+++
### mac下安装mysql
brew 安装方式
brew install mysql
等待安装完毕
### 启动mysql服务
brew services start mysql
### 命令行下登录mysql
mysql -u root
root账号默认没设置密码
#### 设置root账号密码
UPDATE mysql.user SET Password=PASSWORD('MyNewPass') WHERE User='root';
FLUSH PRIVILEGES;
密码要求特殊字符、大小写英文字符、数字；
密码不合格会报syntax error


