## 使用Node部署阿里云服务器

参考网址：
(https://segmentfault.com/a/1190000013740262)
### 一、Node的安装

1. 安装nvm
```
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
```
2. 查看nvm是否安装成功
重新打开一个窗口，输入nvm.看到各种命令表示安装成功。

3. 使用nvm来安装node
```
nvm install v10.15.3  #安装指定版本的node
nvm use v10.15.3   #使用指定版本的node
nvm alias default v10.15.3 # 设置默认的node
```
4. 创建一个文件，测试node的使用
```
touch example.js
```
5. 使用vim来编辑example.js测试文件
```
yum install vim
vim example.js
```
6. 编辑example.js.
```
const http = require('http');
const hostname = '0.0.0.0';
const port = 80;
const server = http.createServer((req, res) => {
res.statusCode = 200;
res.setHeader('Content-Type', 'text/plain');
res.end('Hello World！\n');
});
server.listen(port, () => {
console.log(`Server running at http://${hostname}:${port}/`);
});
```
运行项目:
```
node example.js
```
在浏览器中打开，如果出现hello world表示部署成功。

### 二、持续集成
一个静态站点想要能够被外界访问到，那么需要满足两个条件：
1. 具有稳定持续地服务。
2. 外界能够直接通过80端口访问到(因为http访问默认是80端口)

通过pm2实现持续集成。
1. 安装pm2
```
npm i pm2 -g
```
2. 启动服务
```
pm2 start example.js

pm2 list # 查看当前运行的所有的node服务
pm2 show example(APP Name) #查看具体服务的信息
pm2 logs # 查看实时信息

pm2 stop example.js //停止指定服务 


```

### 三、使用Nginx实现反向代理
当前我们的服务是运行在8081等端口上，我们希望外界能够直接通过80端口进行访问。Nginx通过监听80端口将服务发送给其他端口。

1. 安装
```
yum install  -y nginx
如果上面安装出错，那么需要先安装Nginx源
rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

nginx -v # 查看版本
```
2. 切换到nginx安装目录下
```
cd /etc/nginx/
ls # 查看该目录下的文件。该目录下存在conf.d文件夹。这是nginx的配置目录
```
3. 切换到nginx配置文件下
```
cd conf.d # 切换到配置文件下
```
4. nginx的配置
```
vi example.8081.conf  #创建配置文件并编辑  (配置文件的命名通常使用域名.端口号.conf)
```
5. nginx的配置文件如下：
```
upstream example  {
  server 127.0.0.1:8081;
}

server {
  listen 80;
  server_name 39.96.166.230;
  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header X-Nginx-Proxy true;
    proxy_pass http://example;
    proxy_redirect off;
  }

```
**注意：** 
1. proxy_pass 中的example名字必须与upstream example相同。nginx是根据这个来进行代理的。
2. 如果有多个代理，可以配置多个.conf文件。每个文件中把这些代码再写一遍。


nginx的另外的配置设置：
```
server {
  listen  80;
  server_name test.com;

  location / {
    proxy_pass http:localhost:8081
    proxy_set_header Host $http_host;
  }

}


```


配置文件表示，所有访问http:39.96.166.230:80的服务都会被转发到upstream 中的应用中去。


6. 查看nginx是否配置成功
```
nginx -t

```
7. 重启nginx
```
nginx -c /etc/nginx/nginx.conf
nginx -s reload
```
8. 在浏览器中使用ip地址进行访问：
```
39.96.166.230
```

### 四、mongodb数据库的安装
1. 下载安装包并解压
```
// 下载
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-amazon-3.6.3.tgz
// 解压
tar -zxvf mongodb-linux-x86_64-amazon-3.6.3.tgz
// 将解压的包添加到指定目录
mv mongodb-linux-x86_64-amazon-3.6.3/ /usr/local/mongodb
```

2. 添加到 PATH 路径中：
```
export PATH=/usr/local/mongodb/bin:$PATH

```
3. 创建数据库目录

MongoDB的数据存储在data目录的db目录下，但是这个目录在安装过程不会自动创建，所以你需要手动创建data目录，并在data目录中创建db目录。
以下实例中我们将data目录创建于根目录下(/)。
注意：/data/db 是 MongoDB 默认的启动的数据库路径(--dbpath)。
```
mkdir -p /data/db
```
4. 命令行中运行 MongoDB 服务
可以通过在命令行中执行mongo安装目录中的bin目录执行mongod命令来启动mongdb服务
```
cd /usr/local/mongodb/bin  #切换到mongodb安装目录下的bin目录

mongod 启动mongo命令
```

## 使用Nginx部署vue项目到阿里云上
1. 打包dist目录
```
npm run build
```
通过上述命令，将静态资源打包到dist目录下。
我们上传文件时，只需要上传dist下的文件即可。
2. 修改nginx配置文件。
```
upstream elm {

    server 127.0.0.1:3500;
}

server {
        listen       80;#默认端口是80，如果端口没被占用可以不用修改
        server_name  elm.haiyingsitan.cn www.elm.haiyingsitan.cn;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
        root        /www/elm/dist; #vue项目的打包后的dist

        location / {
            try_files $uri $uri/ @router;#需要指向下面的@router否则会出现vue的路由在nginx中刷新出现404
            index  index.html index.htm;
        }
        #对应上面的@router，主要原因是路由的路径资源并不是一个真实的路径，所以无法找到具体的文件
        #因此需要rewrite到index.html中，然后交给路由在处理请求资源
        location @router {
            rewrite ^.*$ /index.html last;
        }
        #.......其他部分省略
  }

```
