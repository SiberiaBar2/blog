---
title: 服务器部署mysql数据库服务
abbrlink: 74abade9
tags:
  - 后端
categories:
  - 后端
sticky: 3
swiper_index: 3
date: 2025-01-07 14:29:28
updated: 2025-02-13 18:29:00
---

### 这里展示腾讯云 Ubuntu系统 （linux）部署mysql服务

```tex

因为目前腾讯云的云服务器公网和内网是互通的，因此我们可以使用公网ip进行连接数据库，

而不需要进行传统的 内网穿透

即可直接连接

```

### 安装mysql

1. 更新系统包列表：

```bash

sudo apt update

```

2. 安装MySQL服务器：

```bash

sudo apt install mysql-server

```

3. 设置密码和调整安全选项(选择是否删除测试数据库，一般会选择删除)：

```bash

sudo mysql_secure_installation

```

4. 登录

```bash

sudo mysql -u root -p

```

若没有设置密码，直接登录

```bash

sudo mysql -u root 

```

5. 启动MySQL服务

```bash

sudo systemctl start mysql

```

开机自启

```bash

sudo systemctl enable mysql

```

### 导入 sql文件

```bash


# 上传sql文件至服务器
scp /Users/karlfranz/Desktop/muxisql.sql lighthouse@43.163.113.67:/home/lighthouse/home/sql/

# my_database 是数据库名称（不加 .sql 文件后缀）
# 后面的是服务器存放sql文件的路径 （注意不能少了 < ）
mysql -u root -p muxi < /home/lighthouse/home/sql/muxisql.sql

```

```bash

# 重启mysql服务

sudo systemctl restart mysql

```

```sql

# 查看导入结果
USE your_database_name;

SHOW TABLES;
```

### 服务器控制端开启3306防火墙放行（重要）

```tex

在防火墙添加规则

对本机、 所有ipv4、ipv6 发起的3306端口放行。

必须有这一步，

否则本地电脑将无法连接远程数据库。

```

### sql对root用户进行授权

```tex

为什么要进行授权？

mysql只支持本地（localhost）链接，

如果要支持其他主机连接数据库，就要进行授权。

目前mysql ,无论是本地（本机ip, 不包含localhost）还是服务器部署mysql，

要连接数据库，

必须授权。

```

```bash

# 登录
mysql -u root -p 
# 选择系统默认数据库
use mysql;

# 更新host 支持所有host
update user set host='%' where user='root';

# 进行最终授权
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
# 上一句报错了执行这一句 原因未知
grant all privileges on *.* to 'root'@'%';

# 刷新权限
flush privileges;

```

### 修改配置文件，支持所有ip进行连接

打开配置文件

```bash

sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

```

```bash

# 找到 bind-addres 127.0.0.1

修改为 bind-addres 0.0.0.0

ctrl + O 保存

enter 确认文件

ctrl + X 退出

# 防火墙放行

sudo ufw allow 3306/tcp

重启mysql服务

```

### 使用navicat进行连接

```bash

打开navicat

输入腾讯云服务器的公网ip

数据库名字密码

点击连接

显示连接成功

```

### 其他

```bash


# 查看mysql错误日志
sudo tail -n 50 /var/log/mysql/error.log
```

