#说明：
使用过程中遇到任何问题，意见或者经验，请到评论区吐槽，以便完善进一步完善

#1 Server搭建
##1.1 安装
##1.1.1 环境
```
CentOS 6.5 final
MySQL v5.0.31 or newer
Apache 2.2.15 + mod_wsgi
memcached 1.4.24
```
##1.1.2 创建数据库
创建数据库reviewboard,用户myuser@localhost。
```
mysql> CREATE DATABASE reviewboard CHARACTER SET utf8;
mysql> CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'mypassword';
mysql> GRANT ALL PRIVILEGES ON reviewboard.* to 'myuser'@'localhost';
```

##1.1.3 安装
```
yum install ReviewBoard
```
按提示一步一步走,推荐：数据库选mysql,缓存选memecached,webserver选apache,python loader选mod_wsgi,其他配置根据个人情况而定

>安装前确保mysql,memecached可以正常访问。

##1.1.4 新建一个ReviewBoard网站
```
rb-site install /var/www/reviews.example.com
chown -R apache /var/www/reviews.example.com/htdocs/media/uploaded
chown -R apache /var/www/reviews.example.com/data
```
##1.1.5 apache配置
第4步完成后，会生成`/var/www/reviews.example.com`,目录结构如下
```
[admin@developers-10 reviews.example.com]$ ll
总用量 20
drwxr-xr-x 2 root   root 4096 3月  30 11:48 conf
drwxr-xr-x 3 apache root 4096 3月  30 11:55 data
drwxr-xr-x 4 root   root 4096 3月  30 11:48 htdocs
drwxr-xr-x 2 root   root 4096 3月  30 11:48 logs
drwxrwxrwx 2 root   root 4096 3月  30 11:48 tmp
```
其中有个`conf/apache-wsgi.conf`配置文件对应上面的组合(apache+mode_wsgi)
```
&lt;VirtualHost *:80>
        ServerName reviews.example.com
        DocumentRoot "/var/www/reviews.example.com/htdocs"

        # Error handlers
        ErrorDocument 500 /errordocs/500.html

        WSGIPassAuthorization On
        WSGIScriptAlias "/" "/var/www/reviews.example.com/htdocs/reviewboard.wsgi/"

        &lt;Directory "/var/www/reviews.example.com/htdocs">
                AllowOverride All
                Options -Indexes +FollowSymLinks
                Allow from all
        &lt;/Directory>

        # Prevent the server from processing or allowing the rendering of
        # certain file types.
        &lt;Location "/media/uploaded">
                SetHandler None
                Options None

                AddType text/plain .html .htm .shtml .php .php3 .php4 .php5 .phps .asp
                AddType text/plain .pl .py .fcgi .cgi .phtml .phtm .pht .jsp .sh .rb

                &lt;IfModule mod_php5.c>
                        php_flag engine off
                &lt;/IfModule>
        &lt;/Location>

        # Alias static media requests to filesystem
        Alias /media "/var/www/reviews.example.com/htdocs/media"
        Alias /static "/var/www/reviews.example.com/htdocs/static"
        Alias /errordocs "/var/www/reviews.example.com/htdocs/errordocs"
        Alias /favicon.ico "/var/www/reviews.example.com/htdocs/static/rb/images/favicon.png"
&lt;/VirtualHost>
```
复制到apache的配置里去。
```
$ cd /etc/apache2/sites-available
$ cp /var/www/reviews.example.com/conf/apache-wsgi.conf reviews.example.com.conf
$ cd ../sites-enabled
$ ln -s ../sites-available/reviews.example.com.conf 
```
如果没有`sites-available`或`sites-enabled`目录，需要将这个配置加到Apache的全局配置（一般在`/etc/httpd/httpd.conf`或`/etc/httpd/apache2.conf`）。
然后就可以启动apache服务器了
```
service httpd start
```
##1.2 管理
##1.3 常见问题

##1.3.1 添加`https://192.168.1.6/svn/svr/`仓库时报`SSL handshake failed`。

在添加公司svn仓库时，`/var/log/httpd/error.log`出现以下错误：
```
[Wed Mar 30 01:00:09 2016] [error] ERROR:root:SVN: Failed to get repository information for https://192.168.1.6/svn/svr/vivasam: OPTIONS of 'https://192.1
68.1.6/svn/svr/vivasam': SSL handshake failed: SSL error: Key usage violation in certificate has been detected. (https://192.168.1.6)
```
公司svn server是visual svn搭建,官方对这个问题的[说明][1]：
在如下情况下：
  
  *  VisualSVN Server有一个自签名的证书
  *  Subversion client是由GnuTLS library编译的
  
原来是GnuTLS和OpenSSL兼容问题。基于安全的考虑，VisualSVN对自签名的证书指定了叫`Key Usage`的扩展,而GnuTLS识别不出这个扩展，所以报错。
官方的解决方案是修改服务器的注册表，然后重新生成证书，影响比较大(所有客户端都需要更新)。
尝试在客户端(ReviewBoard)解决问题，大多数的Windows svn客户端是用OpenSSL编译的，所以没有这个问题，而很多linux下的svn客户端是有GnuTLS编译的，所以受到影响。
```
[root@developers-10 data]# yum deplist ReviewBoard|grep subversion
   依赖: subversion
   provider: subversion.i686 1.6.11-14.el6
   provider: subversion.x86_64 1.6.11-14.el6
   provider: subversion.i686 1.6.11-15.el6_7
   provider: subversion.x86_64 1.6.11-15.el6_7
```
发现ReviewBoard在安装的时候依赖了系统安装的svn 1.6.11。
```
[root@developers-10 data]# svn --version
svn，版本 1.6.11 (r934486)
   编译于 Aug 17 2015，08:37:43

版权所有 (C) 2000-2009 CollabNet。
Subversion 是开放源代码软件，请参阅 http://subversion.tigris.org/ 站点。
此产品包含由 CollabNet(http://www.Collab.Net/) 开发的软件。

可使用以下的版本库访问模块: 

* ra_neon : 通过 WebDAV 协议使用 neon 访问版本库的模块。
  - 处理“http”方案
  - 处理“https”方案
* ra_svn : 使用 svn 网络协议访问版本库的模块。  - 使用 Cyrus SASL 认证
  - 处理“svn”方案
* ra_local : 访问本地磁盘的版本库模块。
  - 处理“file”方案
[root@developers-10 data]# ldd /usr/bin/svn |grep ssl
[root@developers-10 data]# ldd /usr/bin/svn |grep tls
        libgnutls.so.26 => /usr/lib64/libgnutls.so.26 (0x00007fb9bd4ea000)
```
将svn升级到1.8.5后
```
[root@developers-10 log]# svn --version
svn，版本 1.8.15 (r1718365)
   编译于 Dec  8 2015，17:04:48 在 x86_64-unknown-linux-gnu

Copyright (C) 2015 The Apache Software Foundation.
This software consists of contributions made by many people;
see the NOTICE file for more information.
Subversion is open source software, see http://subversion.apache.org/

可使用以下的版本库访问模块: 

* ra_svn : 使用 svn 网络协议访问版本库的模块。  - 使用 Cyrus SASL 认证
  - 处理“svn”方案
* ra_local : 访问本地磁盘的版本库模块。
  - 处理“file”方案
* ra_serf : Module for accessing a repository via WebDAV protocol using serf.
  - using serf 1.3.7
  - 处理“http”方案
  - 处理“https”方案
```
然后添加仓库成功

#2 相关文档
[https://www.reviewboard.org/docs/manual/2.5/][2]


  [1]: https://www.visualsvn.com/support/topic/00056/
  [2]: https://www.reviewboard.org/docs/manual/2.5/
