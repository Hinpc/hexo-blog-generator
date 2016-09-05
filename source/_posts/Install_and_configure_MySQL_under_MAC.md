title: 记录各系统下安装配置MySQL  
date: 2016-08-25 22:01  
modified: 2016-09-05 20:40  
tag:
 - Mac
 - MySQL

photos:
 - https://dn-hinpc.qbox.me/20160825-stock-photo-125631967.jpg  

---

主要是Mac和CentOS系统下的安装过程

<!--more-->

> Mac系统

## 1. 安装MySQL

通过官网安装包安装：  

下载地址： [http://dev.mysql.com/downloads/mysql/](http://dev.mysql.com/downloads/mysql/)  

选择适合你的安装版本, 我使用的是： Mac OS X 10.11 (x86, 64-bit), DMG Archive  

安装过程直接下一步，中途会弹窗让你记录初始密码，复制下来以便后面修改。

```
2016-08-25T21:15:45.109239Z 1 [Note] A temporary password is generated for root@localhost: ,geoUD.bJ8Hj
```

安装完成后，可以在 系统偏好设置－其他－MySQL 图标中开启和关闭 MySQL 服务。

这种方式默认安装在`/usr/local/mysql/bin`目录下。

## 2. 安装MySQLdb（即 MySQL-python 包）

使用 pip 安装 MySQLdb：  

```bash
pip install MySQL-python
```

安装时可能会报出错误提示：

```bash
EnvironmentError: mysql_config not found
```

解决 mysql_config not found 错误:

vi 打开 vim `~/.zshrc` 末尾加入以下内容：(~/.zshrc 由你的终端而定)

```
PATH="/usr/local/mysql/bin:${PATH}"
export PATH
export DYLD_LIBRARY_PATH=/usr/local/mysql/lib/
```

重启终端，重新使用 pip 安装：

```bash
pip install MySQL-python
```

此时安装成功。

## 3. 修改MySQL初始密码

如果你遇到如下错误：

```
Your password has expired. To log in you must change it using a client that supports expired passwords.
```

那就是提示你使用新版本的客户端登入并修改密码，Mac 下的操作方法：

在控制台登入MySQL：

```bash
mysql -uroot -p
Enter password: # 使用初始密码登录
```
修改密码：

```
mysql> use mysql
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.

mysql> set password = password('a123456');
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

***

> CentOS系统安装 MySQL5.7

## 1. 检查卸载系统自带的MySQL

```sh
rpm -qa | grep -i mysql  
rpm -e --nodeps mysql-libs-5.1.61-4.el6.x86_64
```

## 2. 安装依赖
```sh
yum -y install numactl
```

## 3. 安装 MySQL  
```sh
# 按顺序安装
rpm -ivh mysql-community-common-5.7.10-1.el6.x86_64.rpm  
rpm -ivh mysql-community-libs-5.7.10-1.el6.x86_64.rpm  
rpm -ivh mysql-community-client-5.7.10-1.el6.x86_64.rpm  
rpm -ivh mysql-community-server-5.7.10-1.el6.x86_64.rpm  
```

## 4. 修改数据库编码
为了保证数据库能正确处理中文，我们需要设定数据库默认的编码为utf8。修改/etc/my.cnf文件，并在其中加入以下内容：
```
[client]  
default-character-set=utf8mb4  
  
[mysqld]  
character_set_server=utf8mb4  
```

## 5. 启动
```sh
service mysqld start  
```

## 6. 获得MySQL初始密码
```
grep 'temporary password' /var/log/mysqld.log
```

后面修改初始密码可以参考Mac版。 

注意：mysql5.7默认安装了密码安全检查插件，默认密码检查策略要求密码必须包含：大小写字母、数字和特殊符号，并且长度不能少于8位。否则会提示错误：
```
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```
查看和修改密码安全策略，可以查看这篇文章：
http://blog.csdn.net/xyang81/article/details/51759200

如果不需要密码策略，添加my.cnf文件中添加如下配置禁用即可：
```
validate_password = off
```

<br>
附：[linux 怎么完全卸载mysql数据库如何卸载msql数据库](http://blog.csdn.net/love__coder/article/details/6894566)

[本文完]
