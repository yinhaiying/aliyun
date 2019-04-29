## Nginx的学习

1. Nginx的简单配置
```
server {
  listen  80;
  server_name test.com;

  location / {
    proxy_pass http:localhost:8081
  }

}

```
server_name：是浏览器中输入的域名。
listen:是监听的端口。一般设置为80端口。

proxy_pass:是代理后访问的地址。
也就是说当用户访问text.com的时候，会将请求发送到http:localhost:8081端口。


2. Nginx的更多配置

server {
  listen  80;
  server_name test.com;

  location / {
    proxy_pass http://localhost:8081;
    proxy_set_header Host $host;
  }

}

proxy_set_header Host $host:表示设置请求头为test.com。也就是浏览器中输入的请求头。而不是nginx代理的http://localhost的请求头。
这里的Host非常重要，如果多个项目代理到同一网址(比如http://localhost:8081)。那么究竟展示哪个页面
是根据获取的Host来进行区分。
```
将test.com 代理到 http://localhost:8081.
server {
  listen  80;
  server_name test.com;

  location / {
    proxy_pass http://localhost:8081;
    proxy_set_header Host $host;
  }

}

将b.com 代理到 http://localhost:8081.
server {
  listen  80;
  server_name b.com;

  location / {
    proxy_pass http://localhost:8081;
    proxy_set_header Host $host;
  }

}


```