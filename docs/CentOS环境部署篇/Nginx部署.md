![输入图片说明](https://images.gitee.com/uploads/images/2021/0728/155642_2bf76640_5296156.png "屏幕截图.png")
1.安装依赖包

        //一键安装上面四个依赖
        yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel

2.下载并解压安装包

       //创建一个文件夹
        cd /usr/local
        mkdir nginx
        cd nginx
        //下载tar包
        wget http://nginx.org/download/nginx-1.13.7.tar.gz
        tar -xvf nginx-1.13.7.tar.gz

3.安装nginx

       //进入nginx目录
       cd /usr/local/nginx
       //进入目录
       cd nginx-1.13.7
       //执行命令
       ./configure
       //执行make命令
       make
       //执行make install命令
       make install

4.配置nginx.conf
 
     # 打开配置文件
     vi /usr/local/nginx/conf/nginx.conf

5.启动nginx

    /usr/local/nginx/sbin/nginx -s reload 

    如果出现报错：nginx: [error] open() ＂/usr/local/nginx/logs/nginx.pid＂ failed
    则运行： /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
    再次启动即可！


查看nginx进程是否启动：

    ps -ef | grep nginx
![输入图片说明](https://images.gitee.com/uploads/images/2021/0728/155121_82fafd59_5296156.png "屏幕截图.png")

6.若想使用外部主机连接上虚拟机访问端口192.168.131.2，需要关闭虚拟机的防火墙：

centOS6及以前版本使用命令： systemctl stop iptables.service
centOS7关闭防火墙命令： systemctl stop firewalld.service

随后访问该ip即可看到nginx界面。

7.访问服务器ip查看（备注，由于我监听的仍是80端口，所以ip后面的端口号被省略）

![输入图片说明](https://images.gitee.com/uploads/images/2021/0728/155238_8538a82c_5296156.png "屏幕截图.png")

安装完成一般常用命令

    进入安装目录中，
    命令： cd /usr/local/nginx/sbin
    启动，关闭，重启，命令：
    ./nginx 启动
    ./nginx -s stop 关闭
    ./nginx -s reload 重启

8.配置nginx开机启动

切换到/lib/systemd/system/目录，创建nginx.service文件vim nginx.service

    $ cd /lib/systemd/system/
    $ vim nginx.service

添加如下内容

    [Unit]
    Description=nginx 
    After=network.target 
       
    [Service] 
    Type=forking 
    ExecStart=/usr/local/nginx/sbin/nginx
    ExecReload=/usr/local/nginx/sbin/nginx reload
    ExecStop=/usr/local/nginx/sbin/nginx quit
    PrivateTmp=true 
       
    [Install] 
    WantedBy=multi-user.target

退出并保存文件，执行如下名使nginx开机启动

    $ systemctl enable nginx.service


    # 启动nginx
    $ systemctl start nginx.service
    
    # 结束nginx
    $ systemctl stop nginx.service
    
    # 重启nginx
    $ systemctl restart nginx.service

验证是否安装成功

    # 你要将80端口添加到防火墙中
    $ firewall-cmd --zone=public --add-port=80/tcp --permanent
    #重新加载
    $ firewall-cmd --reload
    
    在浏览器访问如下地址
    http://192.168.2.204/
![输入图片说明](https://images.gitee.com/uploads/images/2021/0728/160434_ecc53a5a_5296156.png "屏幕截图.png")

 **nginx使用ssl模块配置支持HTTPS访问** 

前提：
     1. 配置SSL模块首先需要CA证书，CA证书可以自己手动颁发也可以在阿里云申请，本人在阿里云上申请的证书。（手动颁发可参考文章底部链接）
     2. 默认情况下ssl模块并未被安装，如果要使用该模块则需要在编译nginx时指定–with-http_ssl_module参数.
![输入图片说明](https://images.gitee.com/uploads/images/2021/0816/132503_de0f047a_5296156.png "屏幕截图.png")
在Nginx配置文件中安装证书

```
  server {
        listen 80;
        server_name huangmajia99.com  www.huangmajia99.com;
        location / {
        rewrite (.*) https://www.huangmajia99.com$1 permanent;
        }
    }
      

    server {
        listen       443 ssl;
        server_name  huangmajia99.com  www.huangmajia99.com;
        #ssl on;
        ssl_certificate       cert/6033390_www.huangmajia99.com.pem;
        ssl_certificate_key   cert/6033390_www.huangmajia99.com.key; 

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            proxy_set_header x-forwarded-for $remote_addr;
    	      try_files $uri $uri/ @router;             
    	      root /usr/local/webroot/huangmajia/;
            index  index.html index.htm;
        }
        
        location @router {
            rewrite ^.*$ /index.html last;
         }
   }
```
安装过程中遇见的问题:

```
nginx: [emerg] unknown directive "ssl" in /usr/local/nginx/conf/nginx.conf:151

nginx: [emerg] the "ssl" parameter requires ngx_http_ssl_module in /usr/local/nginx/conf/nginx.conf:148
```
解决方案：

出现这种错误可能是两种情况造成的：

情况一：配置文件格式不正确。

解决方法参考链接：http://blog.csdn.net/yuanyuan_186/article/details/51324082

情况二：ssl模块并未被安装

默认情况下ssl模块并未被安装，如果要使用该模块则需要在编译nginx时指定–with-http_ssl_module参数，这种情况也会导致错误二的出现。



(1)切换到源码包：
`cd /root/nginx-1.13.6`

(2)配置信息：
```
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
```

(3)配置完成后，运行make进行编译，千万不要进行make install，否则就是覆盖安装。
`make`

(4)然后备份原有已经安装好的nginx
```
cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak
```

(5)停止Nginx，正常命令直接 nginx -s stop就可以
`nginx -s stop`

如果关不掉，就直接Kill掉进程。ps aux | grep 进程名 查看进程占用的PID号。
`ps aux|grep nginx`

杀掉查出来的PID就可以了，kill -9 PID 命令用于终止进程。必须先kill掉root对应的PID才能进行下面的三个nobody的PID。
```
kill -9 10922
kill -9 28276
kill -9 28277
kill -9 28278
```


(6)将刚刚编译好的nginx覆盖掉原有的nginx
```
cp ./objs/nginx /usr/local/nginx/sbin/
```

(7)启动nginx

```
nginx
```

(8)通过下面的命令查看是否已经加入成功。

```
nginx -V
```

