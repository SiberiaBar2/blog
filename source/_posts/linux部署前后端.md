---
title: linux部署前后端
date: 2025-01-11 23:14:04
tags:  linux部署前后端

---

### 部署后端

```bash
前言：

使用 Ubuntu 系统

腾讯云内外网互通

可以通过外网ip访问内网服务器上部署的网站和资源。

```

#### 安装必要依赖

```bash

# 我们后端使用的是python ，因此这里展示python后端代码的部署

# 更新系统软件包
sudo apt update

# 安装依赖
sudo apt install -y build-essential checkinstall libssl-dev zlib1g-dev libbz2-dev \ libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \ libffi-dev liblzma-dev tk-dev

# 下载指定版本的python 尽量跟本地版本保持一致
sudo wget https://www.python.org/ftp/python/3.10.7/Python-3.10.7.tgz

# 解压
sudo tar -xvzf Python-3.10.0.tgz
# 进入安装目录
cd Python-3.10.0
# 编译及安装
sudo ./configure --enable-optimizations
sudo make altinstall

# 安装mysql相关依赖文件
# pkg-config 和 libmysqlclient-dev，这两个包分别用于帮助编译时查找库文件的位置和提供 MySQL 客户端开发所需的库文件
sudo apt install pkg-config libmysqlclient-dev

# 创建虚拟环境
python3 -m venv venv

# 进入虚拟环境
source venv/bin/activate

# 在虚拟环境中安装pip
sudo apt update
sudo apt install -y python3 python3-pip

# 指定pip软连接 目的是全局能够访问
sudo ln -s /usr/local/bin/pip3 /usr/bin/pip

# 本地代码中执行 生成项目依赖文件
pip freeze > requirements.txt 

# 进入项目代码层级 安装项目依赖
pip install -r reqguirements.txt

```

### 安装配置uwsgi

```tex

uWSGI 是一个用于部署 Web 应用程序的服务器，它主要被设计用来托管 Python 应用程序，但也可以支持其他语言（如 Ruby、PHP 等）

```

```bash

# 在settings.py同级文件创建uwsgi.ini文件

[uwsgi]
chdir=/home/lighthouse/workspace/muxi_shop_api2/
module=mu_shop_api.wsgi:application
master=True
pidfile=/home/lighthouse/workspace/muxi_shop_api2/project-master.pid
vacuum=True
max-requests=5000
daemonize=/home/lighthouse/workspace/muxi_shop_api2/log/uwsgi/muxishop.log
http-socket= :8000
static-map=/static/product_images=/home/lighthouse/workspace/muxi_shop_api2/static/product_images/
```

```bash

# 安装uwsgi

sudo apt install -y uwsgi uwsgi-plugin-python3

```

### 打包代码

```bash

# 打包将会忽略 .gitignore中的外部库
git ls-files -z | tar --null -czvf muxi_shop_api2.tar.gz -T -

# 打包后的文件
muxi_shop_api2.tar.gz


# 服务器中同步创建文件夹并添加权限
sudo mkdir muxi_shop_api2
sudo chmod -R 775 /home/lighthouse/workspace/muxi_shop_api2/mu_shop_api/
# 因为增加了权限可能还是无权限写入，因此更改所有者
sudo chown -R lighthouse:lighthouse /home/lighthouse/workspace/muxi_shop_api2/mu_shop_api/
# 上传至服务器
scp muxi_shop_api2.tar.gz lighthouse@43.163.113.67:/home/lighthouse/workspace/muxi_shop_api2.tar.gz

# 解压代码
sudo tar -xzvf muxi_shop_api2.tar.gz -C /workspace/muxi_shop_api2/

# 进入代码的uwsgi.ini 文件目录启动
/home/lighthouse/workspace/muxi_shop_api2/mu_shop_api/venv/bin/uwsgi --ini /home/lighthouse/workspace/muxi_shop_api2/mu_shop_api/uwsgi.ini --reload /tmp/uwsgi.pid

# 重启uwsgi
/home/lighthouse/workspace/muxi_shop_api2/mu_shop_api/venv/bin/uwsgi --ini /home/lighthouse/workspace/muxi_shop_api2/mu_shop_api/uwsgi.ini


# 停止uwsgi
/home/lighthouse/workspace/muxi_shop_api2/mu_shop_api/venv/bin/uwsgi --stop /home/lighthouse/workspace/muxi_shop_api2/project-master.pid

# **** 授权各级目录 重要 ****
# 一定要对各级目录进行授权 否则linux服务器无权限访问项目静态资源文件
# 授权还是提示无权限就改所有者

```

### 使用nginx反向代理绑定域名

```bash

server {
    listen 80;
    # 指定域名 注意 一定要后端代码各级目录进行授权 否则无法访问相应文件夹下的代码！！！
    server_name queek.work;

    # /api/前缀 子集域名 来区分后端的请求地址
    location /api/ {
        proxy_pass http://localhost:8000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location /static/product_images/ {
        alias /home/lighthouse/workspace/muxi_shop_api2/static/product_images/;
        expires max;
        add_header Cache-Control "public";
        try_files $uri $uri/ =404;
    }
}

# 这样 我们就能通过域名 queek.work/api/访问后端接口了

```

### 部署前端

```bash

# 打包得到dist文件
yarn run build 

# 使用rsync上传
rsync -avz -e ssh dist/ lighthouse@43.163.113.67:/home/lighthouse/workspace/small_u-shopping_mall/

# 或者scp上传
# scp -r 为递归传输
scp -r dist/ lighthouse@43.163.113.67:/home/lighthouse/workspace/small_u-shopping_mall/

# 由于我们上传的是文件夹 因此不需要解压

```

#### 使用niginx将域名指向我们的前端静态文件

#### 启用新的niginx配置文件

```bash

# 备份旧的配置文件
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak

# 禁用旧的配置文件（删除符号链接）
sudo rm /etc/nginx/sites-enabled/default

# 启用新的配置文件
sudo ln -s /etc/nginx/sites-available/queek.work /etc/nginx/sites-enabled/

```

##### 最终文件

```bash

sudo nano /etc/nginx/sites-available/queek.work
 
server {
    listen 80;
    server_name queek.work;

    location / {
    # 要各级文件目录进行授权
        alias /home/lighthouse/workspace/small_u-shopping_mall/;  # 这是你上传 dist 文件夹的路径
        index index.html;
        try_files $uri /index.html;  # Vue SPA路由需要此配置
    }
    location /api/ {
        proxy_pass http://localhost:8000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location /static/product_images/ {
        alias /home/lighthouse/workspace/muxi_shop_api2/static/product_images/;
        expires max;
        add_header Cache-Control "public";
        try_files $uri $uri/ =404;
    }
}

# 现在我们就能通过域名访问前端网站了

```

### 启动niginx

```bash

# 查看配置文件状态
sudo nginx -t

#重启动nginx
sudo systemctl reload nginx
#重启动nginx
sudo systemctl restart nginx
#启动nginx
sudo systemctl start nginx

# 如果启动后访问有错误 可以查看日志

tail -f /var/log/nginx/access.log

```

### 其他

```bash

不同的系统、平台的服务器具体配置流程可能有差异，

需要根据需要安装相关依赖，以及进行相应的配置。

```

