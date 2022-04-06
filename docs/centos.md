# CentOs 服务器常用软件安装

## 一、Node.js

### 1. 安装
```shell
wget https://npm.taobao.org/mirrors/node/v8.9.4/node-v8.9.4-linux-x64.tar.xz

tar xf node-v8.9.4-linux-x64.tar.xz

ln -s /usr/soft/node-v8.9.4-linux-x64/bin/node /usr/bin/node

ln -s /usr/soft/node-v8.9.4-linux-x64/bin/npm /usr/bin/npm
```
### 2. 常用配置

```shell
# npm install 时使用淘宝镜像源
npm install --registry=https://registry.npm.taobao.org

# 全局安装 nrm
npm install -g nrm

# nrm 查看可用源
nrm ls

# nrm 更换源
nrm use 源名

# nrm 测试速度
nrm test
```

## 二、Git
```shell
yum -y install git
```

## 三、MongoDB

### 1. 安装

```shell
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.4.2.tgz

tar -zxvf mongodb-linux-x86_64-3.4.2.tgz

mv mongodb-linux-x86_64-3.4.2/ mongodb

vim /etc/profile  # 加入环境变量 行尾加入 export PATH=$PATH:/usr/soft/mongodb/

source /etc/profile

# 创建数据目录

mkdir /data/
mkdir /data/mongodb/
mkdir /data/mongodb/logs/
mkdir /data/mongodb/data/

# 修改配置文件

vim /usr/soft/mongodb/mongodb.conf

--- # 修改内容
fork = true
port = 27017
logappend=true
nohttpinterface = false
dbpath=/data/mongodb/data/
logpath=/data/mongodb/logs/mongodb.log
pidfilepath=/data/mongodb/logs/mongodb.pid
---

# 连接远程数据库
mongodb+srv://lxmnet:<password>@<address-url>?retryWrites=true&w=majority
```

### 2. 为mongodb设置用户

> MongoDB 默认是没有密码的，这样就很容易遭受攻击，所以在服务器中部署时最好设置用户

```shell
# 首先进入 mongo 所在目录  /xxx/mongodb/bin/
./mongo
> use admin   # 默认是没有这个数据库，use 会帮我们自动创建这个数据库
> db.createUser({ user: "admin", pwd: "password", roles: [{ role: "userAdminAnyDatabase", db: "admin" }]
> db.createUser({user: "root",pwd: "password", roles: [ { role: "root", db: "admin" } ]})

# 安装 easy-mock 时使用
> use easy-mock
> db.createUser({user: "easy",pwd: "password",roles: [ { role: "dbOwner", db: "easy-mock" } ]})

# 启动 mongodb 时命令后面加 --auth 即可

# 启动关闭 mongodb
/usr/soft/mongodb/bin/mongod --config /usr/soft/mongodb/mongodb.conf --auth
/usr/soft/mongodb/bin/mongod --config /usr/soft/mongodb/mongodb.conf --shutdown

# 注意：easy-mock /easy-mock/config/default.json 中 db 地址修改为 mongodb://<username>:<password>@localhost:27017/easy-mock
```

## 四、Redis

```shell
yum -y install gcc

wget https://download.redis.io/releases/redis-6.2.6.tar.gz

tar -zxvf redis-6.2.6.tar.gz

cd redis-6.2.6 && make

make PREFIX=/usr/local/redis install     # 安装到指定目录

cd /usr/local/redis

mkdir conf

cp /usr/soft/redis-6.2.6/redis.conf  /usr/local/redis/conf

vim /usr/local/redis/conf/redis.conf

---
daemonize no 改为 yes
---

cd /usr/local/redis

./bin/redis-server ./conf/redis.conf     # 启动

ps aux|grep redis    # 查看进程
```

## 五、easy-mock

```shell
npm install pm2 -g   // 权限问题执行  npm i --unsafe -perm
NODE_ENV=production pm2 start app.js
```
> 如果提示不存在 pm2 则执行下面命令：(目录位置不固定)

```shell
ln -s /home/opc/node-v8.9.4-linux-x64/lib/node_modules/pm2/bin/pm2 /usr/bin

# 停止pm2命令
pm2 list  # 查看运行中的项目  找到 name 字段
pm2 stop <name> 
```
[中文文档](https://github.com/easy-mock/easy-mock/blob/dev/README.zh-CN.md)

## 六、Nginx

### 1. 安装 Nginx
```shell
gcc -v  # 查看gcc版本  如果没有执行   yum -y install gcc

yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel

wget http://nginx.org/download/nginx-1.9.9.tar.gz  
tar -zxvf  nginx-1.9.9.tar.gz
cd nginx-1.9.9
./configure
make
make install

whereis nginx  # 查找nginx安装路径

# 启动、停止nginx
cd /usr/local/nginx/sbin/    # nginx 安装路径下 sbin
./nginx 
./nginx -s stop # 此方式相当于先查出 nginx 进程 id 再使用 kill 命令强制杀掉进程。
./nginx -s quit # 此方式停止步骤是待 nginx 进程处理任务完毕进行停止。
./nginx -s reload

# 如果启动时报80端口被占用 则安装 net-tool 包：yum install net-tools

ps aux|grep nginx  # 查询 nginx 进程

# 当nginx的配置文件 nginx.conf 修改后，要想让配置生效需要重启 nginx
# 使用-s reload 即可将配置信息在 nginx 中生效，如下：
./nginx -s reload

# 开机自启动
vim /etc/rc.local 
-------
/usr/local/nginx/sbin/nginx    # 增加一行
-------

chmod 755 /etc/rc.local   # 设置执行权限
```

## 1. Nginx 做 node 反向代理

```shell
# 进入nginx安装目录下 conf 文件夹，在该目录下创建 include 文件。
cd /usr/local/nginx/conf
mkdir include && cd include
vim nginx.node.conf  # 新建文件
-----------
upstream nodejs {
  server 127.0.0.1:7300;
  keepalive 64;
}
server {
  listen 80;
  server_name www.xx.com xx.com;
  location / {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host  $http_host;
      proxy_set_header X-Nginx-Proxy true;
      proxy_set_header Connection "";
      proxy_pass      http://nodejs;
  }
}
-----------
or
-----------
server {
  listen       80;
  server_name  mock.xx.com;
  location / {
    proxy_pass  http://127.0.0.1:7300;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
-----------
# 进入 conf ，打开 nginx.conf, 在 http 里面添加   include ./include/*;
cd /usr/local/nginx/sbin
./nginx -s reload # 重启nginx
```

### 创建静态目录

```shell
server {
  listen       80;
  server_name  xxx.com;
  root         /home/www/;
  location / {}
}
```

### 开启 gzip 压缩

```shell
http:{ 
 		gzip  on;
    gzip_static on;
    gzip_buffers 4 16k;
    gzip_comp_level 5;
    gzip_types text/plain application/javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
}

# 重启 nginx 如遇报错, 在安装目录执行以下命令后再重启
./configure --prefix=/usr/local/nginx --with-http_gzip_static_module
make && make install



# vue 配置
npm install --save-dev compression-webpack-plugin@1.1.12 # 安装compression-webpack-plugin, 最新版本可能会报错

# vue.config.js 修改
const CompressionPlugin = require("compression-webpack-plugin")

productionSourceMap: false, # 在这个配置项下面加入

configureWebpack: config => {
    if (process.env.NODE_ENV === 'production') {
      return {
        plugins: [new CompressionPlugin({
          test: /\.js$|\.html$|\.css/,
          threshold: 10240,
          deleteOriginalAssets: false
        })]
      }
    }
}
```

## 七、Mysql 5.7

```shell
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
yum -y install mysql57-community-release-el7-10.noarch.rpm
yum -y install mysql-community-server
# 密钥过期使用 rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022


# 安装完成后启动并查看状态，安装并启动成功的话，会出现绿色的 active(running) 字样
systemctl start  mysqld.service
systemctl status mysqld.service

# 修改密码
grep "password" /var/log/mysqld.log # 其中末尾部分为初始密码(如：YC6ZqT=56rKD)
mysql -uroot -p
> ALTER USER 'root'@'localhost' IDENTIFIED BY '你的密码'; # 需要包含字母数字符号，安装结束

# mysql 配置文件位置 /etc/my.cnf

service mysqld restart  # 重启 mysql

# 授权远程
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '你的密码' WITH GRANT OPTION;
```

## 八、PHP

```shell
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm

# 查找是否有php
yum search php7

# yum 安装php72w和各种拓展
yum -y install php72w php72w-cli php72w-common php72w-devel php72w-embedded php72w-fpm php72w-gd php72w-mbstring php72w-mysqlnd php72w-opcache php72w-pdo php72w-xml

systemctl enable php-fpm.service  # 开机自启动
systemctl start php-fpm.service  # 启动服务
```

## 九、CentOs 常用命令

```shell
# 查看端口占用
lsof -i tcp:80

# 列出所有端口
netstat -ntlp

# 查看安装位置
whereis xxx
```