# linux-server-config# 服务端配置

linux CentOS 8.2 64位



## Nginx

```
cd  /usr/localan

wget http://nginx.org/download/nginx-1.20.0.tar.gz
```

安装必要插件

```
yum -y install gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

**gcc** 可以编译 C,C++,Ada,Object C和Java等语言

**pcre pcre-devel** pcre是一个perl库，包括perl兼容的正则表达式库，nginx的http模块使用pcre来解析正则表达式，所以需要安装pcre库

**zlib zlib-devel** zlib库提供了很多种压缩和解压缩方式nginx使用zlib对http包的内容进行gzip，所以需要安装

**openssl openssl-devel** openssl是web安全通信的基石，没有openssl，可以说我们的信息都是在裸奔

解压

```
tar -zxvf nginx-1.20.0.tar.gz
```

重命名

```
mv nginx-1.20.0.tar.gz nginx
```

指定安装路径

```
./configure --prefix=/usr/local/nginx
```

安装

```
make && make install
```



### 设置首页

修改配置

```
cd /usr/local/nginx/conf/

vim nginx.conf
```

try_files：解决history模式下，刷新404问题

（vue-router文档： https://router.vuejs.org/zh-cn/essentials/history-mode.html）

```
location / {

　　root /data/dist; // 修改前端网站位置

　　index index.html;

　　try_files $uiri $uri/ /index.html;
　　
}
```



### 基础配置

供参考

```

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    #开启Gzip
    gzip  on;
    #大于1K的才压缩
    gzip_min_length 1k;
    #缓冲
    gzip_buffers 4 16k;
    #gzip_http_version 1.0;
    #压缩级别，1-10，数字越大压缩的越好，时间也越长
    gzip_comp_level 5;
    #进行压缩的文件类型
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript;
    #跟Squid等缓存服务有关，on的话会在Header里增加"Vary: Accept-Encoding"
    gzip_vary off;
    #IE6对Gzip不友好
    gzip_disable "MSIE [1-6]\.";

    #index配置
    include /usr/local/nginx/conf/vhost/*.conf;

}

```



### 反向代理

编辑配置文件

```
vim /usr/local/nginx/conf/nginx.conf
```

正则匹配

```
location ^~/api {

​	proxy_pass http://127.0.0.1:3000/api;

}
```

### 检测配置

```
nginx -t
```

### 重启

```
nginx -s reload
```



### 访问静态文件配置

方法一

 精细化 配置相关静态资源参数，优化访问静态资源文件

```
location ~ .*\.(gif|jpg|jpeg|png|ico|JPG|GIF|PNG|JPEG|webp)$ {
    expires 24h;
    root /data/resource/;#指定图片存放路径
    proxy_store on;
    proxy_temp_path /data/resource/;#图片访问路径
    proxy_redirect  off;
    proxy_set_header  Host 127.0.0.1;
    client_max_body_size  10m;
    client_body_buffer_size 1280k;
    proxy_connect_timeout  900;
    proxy_send_timeout   900;
    proxy_read_timeout   900;
    proxy_buffer_size    40k;
    proxy_buffers      40 320k;
    proxy_busy_buffers_size 640k;
    proxy_temp_file_write_size 640k;
    if (!-e $request_filename) {
      proxy_pass http://127.0.0.1;#默认80端口
    }
  }
```

方法二 目录配置

```
 location /resource/ {
    alias /data/resource/;
    autoindex on;
  }
```



### SSL证书

https默认为443端口,需自动跳转

```
server {
  listen       80;
  server_name  mall;
  rewrite ^(.*)$ https://$host$1 permanent;
}
```

```
server {
  listen      443;
  server_name  mall;
  #charset koi8-r;
  #access_log  logs/host.access.log  main;
  #ssl
  include /usr/local/nginx/ssl/ssl.conf;

  # rewrite ^(.*)$ https://$host$1 permanent;# 把http的域名请求转成https

  location / {
    root   /data/web/mall;
    index  index.html;
    try_files $uri $uri/ /index.html;
  }
 }
```

ssl.conf 文件配置

```
ssl on;
# ssl证书地址
ssl_certificate /usr/local/nginx/ssl/3587977_zww0923.top.pem;
ssl_certificate_key /usr/local/nginx/ssl/3587977_zww0923.top.key;
# ssl验证相关配置
ssl_session_timeout 5m; # 缓存有效期
ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4; #加密算法
ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #安全链接可选的加密协议
ssl_prefer_server_ciphers on; #使用服务器端的首选算法
```







## Nvm

支持多版本Nodejs

```
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
```

修改配置

```
cd /root/.nvm

vi .bash_profile
```

添加以下内容

```
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"

[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm

```

重启shell后

检测是否安装成功

```
nvm --version
```



### 安装nodejs

```
nvm install 12.12.0

nvm install 10.24.0

nvm use  12.12.0
```

检测nodejs

```
node -v
```









## Git

```
yum install git -y
```

配置git

```
git config --global user.name "zzw"

git config --global user.email "zzw1366@163.com"
```

查看配置是否生效

```
git config --list
```









## Pm2

进程守护

```
npm install pm2@latest -g
```

使用方式

```
pm2 start app.js
```

### 常用指令

```
$ pm2 restart app_name
$ pm2 reload app_name
$ pm2 stop app_name
$ pm2 delete app_name
```

查看进程列

```
pm2 list
```

关闭某个进程

```
pm2 stop <id>
```

关闭所有进程

```
pm2 stop all
```

日志

```
pm2 logs
```

restart 和reload 区别

restart = stop+start 

reload = 重新读取配置文件 

具体用哪个要根据项目运行实际情况，有些项目需要7*24运行，不得stop，这时候用reload比较好。









## Redis

```
wget http://download.redis.io/redis-stable.tar.gz
```

解压

```
tar -xzvf redis-stable.tar.gz
```

重命名

```
mv redis-stable redis
```

检查版本

```
gcc -v
```

 如果gcc低于5.3需走gcc升级后，再继续

安装

```
make

cd src

make install PREFIX=/usr/local/redis
```

### 配置后台启动

```
vi /usr/local/redis/redis.conf 
```

```
# 后台启动
daemonize yes
# 开启远程访问
protected-mode改为no
```



### gcc升级

centos7默认是4.8.5 如果gcc在5.3以下需走以下配置

gcc升级到9

```
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
```

gcc 版本切换

（建议进行永久切换，临时切换完成之后重新连接服务器gcc自己又降回去了）

临时切换

```
scl enable devtoolset-9 bash
```

永久切换

```
echo “source /opt/rh/devtoolset-9/enable” >> /etc/profile
```

 





## MySQL

安装MySQL8.0

使用最新的包管理器安装MySQL

```
sudo dnf install @mysql
```

开启启动

```
sudo systemctl enable --now mysqld
```

要检查MySQL服务器是否正在运行

```
sudo systemctl status mysqld
```



### 添加密码及安全设置

sudo mysql_secure_installation



步骤如下：

1. 要求你配置VALIDATE PASSWORD component（验证密码组件）： 输入y ，回车进入该配置

   - 选择密码验证策略等级， 我这里选择0 （low），回车
   - 输入新密码两次
   - 确认是否继续使用提供的密码？输入y ，回车
   - 移除匿名用户？ 输入y ，回车
   - 不允许root远程登陆？ 我这里需要远程登陆，所以输入n ，回车
   - ![img](https://img2018.cnblogs.com/blog/981325/201911/981325-20191126145030142-806823861.png)

2. 移除test数据库？ 输入y ，回车

3. 重新载入权限表？ 输入y ，回车

   ![img](https://img2018.cnblogs.com/blog/981325/201911/981325-20191126145056520-234675009.png)





### 远程登录

```
mysql -uroot -p
```

输入密码

执行mysql语句，将将root用户的host字段设为'%'：

```
use mysql;
update user set host='%' where user='root';
flush privileges;
exit;
```



### 开启系统防火墙的3306端口

```
sudo firewall-cmd --add-port=3306/tcp --permanent
sudo firewall-cmd --reload
```



### 重启服务

```
sudo systemctl restart mysqld
```





## Mongdb

centos用Redhat 版本

```
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.2.2.tgz
```

解压

```
tar -zxvf mongodb-linux-x86_64-rhel70-4.2.2.tgz
```

重命名

```
mv ./mongodb-linux-x86_64-rhel70-4.2.2.tgz mongodb
```

进入目录

```
cd /usr/local/mongodb
```

创建一个新的文件夹

```
mkdir db
```

配置

```
vim mongodb.conf
```

```
port=27017
dbpath= /usr/mongodb/db
logpath= /usr/mongodb/mongodb.log
logappend=true
fork=true #守护模式
maxConns=100
noauth=true
journal=true
storageEngine=wiredTiger
bind_ip = 0.0.0.0
```

### 启动

```
/usr/local/mongodb/bin/mongod --config /usr/mongodb/mongodb.conf
```





## 其他问题

### 阿里云IP无法访问

打开服务器实例 选择安全组 



添加安全组规则

协议: 自定义TCP

端口范围: 80/80

授权对象: 0.0.0.0/0



设置完毕后重启服务器+nginx后，即可访问

