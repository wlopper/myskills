[toc]

# Linux上利用nginx搭建一个简单的rtmp视频流服务器(不涉及直播)

今天公司要求用一个nginx搭建一个简单的rtmp视频流服务器，只要能够简单播放视频即可，不涉及到直播业务，网上搜了一大半几乎全是做直播业务的:joy:,没办法，自己根据各种教程慢慢做了一个简单的视频流服务器。

整体介绍：主要是通过`nginx`结合`nginx-rtmp-module`模块结合，再通过修改nginx配置文件即可让前端通过rtmp协议访问到对应文件。



## 一、基础环境搭建

基础环境搭建主要是为了编译`nginx-rtmp-module`模块等，如果涉及到直播业务，还有有`openssl、pcre、zlib`等模块需要配合，那就必须得安装`gcc`编译环境。

```bash
#安装gcc环境
yum install -y gcc gcc-c++ autoconf wget
yum -y install wget gcc-c++ ncurses ncurses-devel cmake make perl bison openssl openssl-devel gcc* libxml2 libxml2-devel curl-devel libjpeg* libpng* freetype*
```

## 二、构建Nginx

1. ### 下载nginx-rtmp-module

   网上直接搜索即可下载，我下载的版本是`nginx-rtmp-module-1.1.8.tar.gz`。直接解压即可。

2. ### 安装Nginx

   同样网上下载nginx，我下载的版本是`nginx-1.11.2.tar.gz`。同样直接解压。

   形成目录如下所示：

   ![image-20210317222327672](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210317222327672.png)

   压缩包可以等会再删，如果nginx制作失败还可以再来:joy:

3. ### 编译nginx，代理视频

   1. 首先创建一个nginx目录，如下图：

      ```bash
      #mkdir nginx
      ```

      

      ![image-20210317222509848](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210317222509848.png)

   2. 进入`nginx-1.11.2`目录下,执行命令：

      ```bash
       #prefix是nginx编译后的输出目录，就选刚创建的nginx目录吧
       #add-module后跟的是nginx-rtmp-module相对当前目录的相对路径
       #如果nginx版本过高，http_ssl_module参数会引起错误，还需要安装其他环境如prec和openssl。
       
       ./configure --prefix=/usr/local/src/nginx  --add-module=../nginx-rtmp-module-1.1.8  --with-http_ssl_module   
      ```

   3. 制作完成后进入nginx目录中，启动nginx验证:

      ```bash
      cd nginx/sbin
      ./nginx
      访问对应ip和端口(nginx.conf配置)出现欢迎页即表示成功！！！
      坑1：如果访问不了是可能是linux防火墙没有关闭
      执行以下命令：
      #查看防火状态：看到状态为绿色的running表示开启。
      systemctl status firewalld.service
      #暂时关闭防火墙:关闭后状态为dead
      systemctl stop firewalld.service
      #永久关闭防火墙
      systemctl disable firewalld.service
      
      ```

      ![image-20210317223255036](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210317223255036.png)

   4. nginx配置代理

      最终配置如下：

      ```bash
      
      user  root;
      worker_processes  1;
      
      error_log  logs/error.log;
      
      events {
          worker_connections  1024;
      }
      
      rtmp {                #RTMP服务
          server {
              listen 1935;  #//服务端口 
              chunk_size 4096;   #//数据传输块的大小
      
              application vod {
                  play /usr/local/src/Videos; #//视频文件存放位置。
              }
          }
      }
      
      http {
          include       mime.types;
          default_type  application/octet-stream;
      
          sendfile        on;
      
          keepalive_timeout  65;
      
      
          server {
              listen       8888;
              server_name  localhost;
      
              location / {
                  root   html;
                  index  index.html index.htm;
              }
      
              error_page   500 502 503 504  /50x.html;
              location = /50x.html {
                  root   html;
              }
          }
      }
      
      ```

      重点是以下部分：

      ```bash
      rtmp {                #RTMP服务
          server {
              listen 1935;  #//服务端口 
              chunk_size 4096;   #//数据传输块的大小
      		# 例：rtmp://192.168.163.128:1935/vod/GGZ_RZBB.mp4 
              application vod {
                  play /usr/local/src/Videos; #//视频文件存放位置。
              }
          }
      }
      ```

      坑2：复制上述代码到配置文件保存后nignx启动报错，原因：复制的时候保存会将空格编译，需要通过本地文本编辑器使用无BOM格式保存文章，然后在使用:joy:。

   5. 访问验证

      访问验证推荐`VLC`播放器或者`potplayer`，访问地址如下：`rtmp://192.168.163.128:1935/vod/GGZ_RZBB.mp4`，vod为application名，必须要加，然后就是Videos下的视频相对路径。

      `VLC`播放器可以在播放器下添加网络串流，直接输入地址；

      ![image-20210317224752276](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210317224752276.png)

      ![image-20210317224702552](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210317224702552.png)

      `potplayer`是我的默认播放器，直接在谷歌浏览器上输入地址后，会弹出的提示打开potplayer播放；

      ![image-20210317225013944](C:\Users\19387\AppData\Roaming\Typora\typora-user-images\image-20210317225013944.png)

   6. 大功完成！！！

   

