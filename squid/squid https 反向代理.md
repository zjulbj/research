#1快速安装
本文档针对squid3.5.15，不同版本情况有所不同，不保证本文档使用其他版本

- 1 下载源码包:
```
wget http://www.squid-cache.org/Versions/v3/3.5/squid-3.5.15.tar.gz
```
- 2 解压
```
tar xzvf squid-3.5.15.tar.gz
```
- 3 configure
```
./configure --prefix=/usr/local/squid --with-openssl
```
`--prefix`指定squid安装路径。`--with-openssl`默认从系统库加载openssl库，若不在系统库则用`--with-openssl=/path/to/openssl`指定。若没有openssl，需先安装。
>注：如果出现以下错误`library 'crypto' is required for OpenSSL`，表示系统没有装openssl，需先安装
>```
>yum install openssl openssl-devel
>```

- 4 make&make install 
请先确认对--prefix指定的路径有读写权限
```
make & make install
```

- 5 start
```
chmod -R 777 /usr/local/squid/var
cd /usr/local/squid/sbin
./squid
```
>可以在/usr/sbin添加一个squid的软链接，方便使用
>```
>ln -s /usr/local/squid/sbin/squid /usr/sbin/squid
>```
#2 配置
## 2.1 配置
http://www.squid-cache.org/Versions/v3/3.5/cfgman/
http://zyan.cc/book/squid/chap06.html
## 2.2 https反向代理
在`/usr/local/squid/etc/squid.conf`,在**最前面**加入以下配置
以`worktile.com`为例
```
http_port 80 accel defaultsite=www.google.com
# Third (HTTPS) peer
cache_peer www.google.com parent 443 0 no-query originserver name=websiteA ssl sslflags=DONT_VERIFY_PEER

acl sites_server_A dstdomain www.google.com
cache_peer_access websiteA allow sites_server_A
http_access allow sites_server_A

# Third (HTTPS) peer
cache_peer www.youtube.com parent 443 0 no-query originserver name=websiteB ssl sslflags=DONT_VERIFY_PEER

acl sites_server_B dstdomain www.youtube.com
cache_peer_access websiteB allow sites_server_B
http_access allow sites_server_B
```
网络链路为Client --HTTP（80）-->Squid Server--HTTPs（443）-->Original Server
如需client到Squid也为https，则需要先[生成CA证书][1]，改为以下配置
```
https_port 443 accel defaultsite=worktile.com \
  cert=/usr/local/squid/etc/private/cert.pem \
  key=/usr/local/squid/etc/private/cakey.pem no-vhost
```
然后重新启动:
```
squid -k restart
```
修改客户端host，将要代理的域名指向 `47.88.137.161`（squid服务器)
```
47.88.137.161 www.google.com
47.88.137.161 www.youtube.com
```
然后就可以在客户端通过47.88.137.161访问google和youtube了
  [1]: http://www.tldp.org/HOWTO/SSL-Certificates-HOWTO/c118.html
