# Nginx 实现文件下载

先看配置文件

```
server {
     listen    80;
     server_name  localhost;
 
     location ^~ /download/ {
        alias /home/webhtml/;
 
        if ($request_filename ~* ^.*?\.(html|doc|pdf|zip|docx|txt)$) {
            add_header Content-Disposition attachment;
            add_header Content-Type application/octet-stream;
        }
            sendfile on;   # 开启高效文件传输模式
            autoindex on;  # 开启目录文件列表
            autoindex_exact_size on;  # 显示出文件的确切大小，单位是bytes
            autoindex_localtime on;  # 显示的文件时间为文件的服务器时间
            charset utf-8,gbk;  # 避免中文乱码
      }
}
```

关键点解析：

```
server_name  localhost;  代表服务名，在这里直接访问 127.0.0.1就可以。如果是别的机器访问这台机器。则直接替换成这台机器的地址即可。

 location ^~ /download/ 带边的是拦截包含 download的请求

 alias /home/webhtml/;  是来配置下载文件的路径，也就是提供下载的文件要在这个路径下。

  if ($request_filename ~* ^.*?\.(html|doc|pdf|zip|docx|txt)$) {
            add_header Content-Disposition attachment;
            add_header Content-Type application/octet-stream;
        }
```

使用方法

首先把一个待下载的文件放在：/home/webhtml/ 目录下

然后就可以用这个url来下载文件了(本机访问)： 127.0.0.1/download/hello.txt

一般都是对外提供下载的（假设我这台部署nginx的机器是：10.2.5.10）：10.2.5.10/download/hello.tx  

我们可以知道的是，带有download的路径被拦截了，加上我们配的文件路径，最后去取文件：

/home/webhtml/hello.txt

