title: 记录各系统下安装配置MySQL  
date: 2016-08-25 22:01  
tag:
 - Mac
 - MySQL

photos:
 - https://dn-hinpc.qbox.me/20160825-stock-photo-125631967.jpg  

---

主要是Mac和CentOS系统下的安装过程

<!--more-->

# Mac

## 安装MySQL

通过官网安装包安装：  
下载地址： [http://dev.mysql.com/downloads/mysql/](http://dev.mysql.com/downloads/mysql/)  
我的安装版本： Mac OS X 10.11 (x86, 64-bit), DMG Archive

安装过程直接下一步，中途会弹窗让你记录初始密码，复制下来以便后面修改。

```
2016-08-25T21:15:45.109239Z 1 [Note] A temporary password is generated for root@localhost: ,geoUD.bJ8Hj
```

安装完成后，可以在 系统偏好设置－其他－MySQL 图标中开启和关闭 MySQL 服务。

这种方式默认安装在`/usr/local/mysql/bin`目录下。

## 安装MySQLdb（即 MySQL-python 包）

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

## 修改MySQL初始密码

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

# CentOS

## 安装


```js
async function install () {
  awiat iFinishIt();
  return continue();
}
```