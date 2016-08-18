#快速安装

##下载安装

```
wget http://tengine.taobao.org/download/tengine-2.1.2.tar.gz
tar -zxvf tengine-2.1.2.tar.gz
cd tengine-2.1.2
./configure --prefix=/usr/local/tengine
make
make install
```

>configure 如过报pcre找不到，需要下载pcre,再在configure的时候加上--with-pcre=/path/to/pcre_source

然后tengine就安装到`/usr/local/tengine`下面了，目录结构和nginx一样。

##配置
Tengine完全兼容Nginx，因此可以参照Nginx的方式来配置Tengine。
从配置可以看出，tenine通过`check_http_send "HEAD / HTTP/1.0\r\n\r\n"`,向后端发送心跳包，然后通过`check_http_expect_alive http_2xx http_3xx`检查返回消息的http code是否符合预期。

```
 upstream app_user {
     server 127.0.0.1:8001;
     check interval=3000 rise=2 fall=5 timeout=1000 type=http;
     check_http_send "HEAD / HTTP/1.0\r\n\r\n";
     check_http_expect_alive http_2xx http_3xx;
   }

   server {
      listen       8080;
      server_name  camdy.dev.xiaoying.co;

      charset utf-8;

      access_log  logs/camdy_dev_xiaoying_co.log ;

      index index.html index.htm index.jsp;
      #root /data/app/apiserver/container;

      proxy_set_header Host            $host;
      proxy_set_header X-Real-IP       $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header   X-Real-Port     $remote_port;


      #ssl on;
      #ssl_certificate     /data/opt/keys/c_api.crt;
      #ssl_certificate_key /data/opt/keys/c_api.key.nopasswd;

      location /api/rest/{
        proxy_pass http://app_user;

        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Real-Port     $remote_port;


        proxy_redirect     off;
        proxy_connect_timeout      90;
        proxy_send_timeout         90;
        proxy_read_timeout         90;
        proxy_buffer_size          4k;
        proxy_buffers              4 32k;
        proxy_busy_buffers_size    64k;
        proxy_temp_file_write_size 64k;
      }

      location /status {
        check_status;

        access_log off;
```
http只是Tengine支持几种检测方式：

健康检查包的类型(type)，现在支持以下多种类型

- tcp：简单的tcp连接，如果连接成功，就说明后端正常。
- ssl_hello：发送一个初始的SSL hello包并接受服务器的SSL hello包。
- http：发送HTTP请求，通过后端的回复包的状态来判断后端是否存活。
- mysql: 向mysql服务器连接，通过接收服务器的greeting包来判断后端是否存活。
- ajp：向后端发送AJP协议的Cping包，通过接收Cpong包来判断后端是否存活。

具体配置说明参见：[http://tengine.taobao.org/document_cn/http_upstream_check_cn.html][1]
##Upstream 接口
前面tengine发送的心跳包upstream这边在有相应的回复包。通过这个手段，我们可以通过一个开关来控制app的平滑上下线。
以camdy为例，我们定义一个接口来检查和控制服务的上下线状态。
```
/**
 * An interface to check and change the status(online/offline) of the server.
 * 
 * @author lvbj
 *
 */
@Path("/health")
@Consumes({MediaType.APPLICATION_FORM_URLENCODED, MediaType.APPLICATION_JSON, MediaType.TEXT_XML})
@Produces({ContentType.APPLICATION_JSON_UTF_8, ContentType.TEXT_XML_UTF_8})
public interface HealthCheckService {
    /**
     * A simple echo to server to ensure that server is online.
     * 
     * @return Returns a empty object with HTTP code 200 if server is online.
     * @throws ServiceException Throws a ServiceException(SERVICE_UNAVAILABLE) with HTTP code 500 if
     *         server is offline.
     */
    @GET
    @PublicRequest
    public Object echo() throws ServiceException;

    /**
     * Switch the status of the server. You may need some privilege limitations in practice in case
     * of security issues.
     * 
     * @param action "up" for online,"down" for offline
     * @return Returns whether the sever is online(true - online,false - offline)
     */
    @GET
    @PublicRequest
    @Path("/action/{action}")
    public Object action(@PathParam("action") String action);
}
```
接口实现不固定，下面是一个简单的实现：
```
public class SimpleHealthCheckService implements HealthCheckService {
    private static final Logger logger = LoggerFactory.getLogger(SimpleHealthCheckService.class);

    protected volatile static boolean online = true;

    public Object action(String action) {
        logger.info("action : {}", action);
        if ("up".equals(action)) {
            online = true;
        } else if ("down".equals(action)) {
            online = false;
        } else {
            return "nothing changed";
        }
        return online;
    }

    public Object echo() throws ServiceException {
        if (logger.isDebugEnabled()) {
            logger.debug("echo {}", new Date());
        }
        if (online) {
            
        } else {
            ErrorCode error = ErrorCode.SERVICE_UNAVAILABLE;
            throw new ServiceException(error.getCode(), error.getHttpCode(), error.getDesc());
        }
        return new JSONObject();
    }
}
```
`action`控制服务的上下线状态，`echo`告诉tengine现在的状态。

- 当online为true时,tengine收到200的反馈消息。
- 当online为false时,tengine收到500的反馈消息。

##加入Camdy所有服务
为了运维友好，在Camdy加入健康检查时，做了如下约定：

- 心跳包的发送路径最好统一(都用/api/rest/health/)
- 上线状态的入口也最好统一(都用/api/rest/health/action/{up/down})

所以只需要在所有的upstream配置中统一加入如下配置:
```
     check interval=3000 rise=2 fall=5 timeout=1000 type=http;
     check_http_send "HEAD /api/rest/health/ HTTP/1.0\r\n\r\n Connection: keep-alive\r\n\r\n";
     check_http_expect_alive http_2xx http_3xx;
```
所以对于所有的upstream来说，就只有host和port不一样了，echo和action的实现时通用的。所以把前面的接口写到一个公用的包里面，然后在每个项目的spring配置中加入:
```
&lt;!-- health check api -->
&lt;bean id="healthCheck" class="com.vivacommon.toolkit.health.service.impl.SimpleHealthCheckService" />
&lt;dubbo:service interface="com.vivacommon.toolkit.health.service.HealthCheckService" ref="healthCheck" protocol="rest" />
```
这样所有模块就都支持健康检查了。然后访问tengine服务器的/status路径就可以看到所有upstream的状态了。
```
GET http://121.40.28.106/status?format=json
Response
{"servers": {
  "total": 5,
  "generation": 3,
  "server": [
    {"index": 0, "upstream": "app_user", "name": "127.0.0.1:8001", "status": "up", "rise": 592, "fall": 0, "type": "http", "port": 0},
    {"index": 1, "upstream": "app_video", "name": "127.0.0.1:8002", "status": "up", "rise": 587, "fall": 0, "type": "http", "port": 0},
    {"index": 2, "upstream": "app_topic", "name": "127.0.0.1:8007", "status": "up", "rise": 584, "fall": 0, "type": "http", "port": 0},
    {"index": 3, "upstream": "app_barrage", "name": "127.0.0.1:8003", "status": "up", "rise": 588, "fall": 0, "type": "http", "port": 0},
    {"index": 4, "upstream": "app_friend", "name": "127.0.0.1:8005", "status": "up", "rise": 583, "fall": 0, "type": "http", "port": 0}
  ]
}}
```

  [1]: http://tengine.taobao.org/document_cn/http_upstream_check_cn.html
