## frp内网穿透
应用场景: 开发阶段调试支付回调; 对接第三方,需要提供公网接口,但系统并没上线.
-  [github项目地址](https://github.com/fatedier/frp)
-  [代码下载地址](https://github.com/fatedier/frp/releases)
-  [使用实例](https://github.com/fatedier/frp/blob/master/README_zh.md#%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B)


#### 通过自定义域名访问部署于内网的 web 服务
1. 位于公网IP `65.49.10.110`的服务器设置,配置文件 `frps.ini `
```shell
# frps.ini
[common]
bind_port = 7000 #绑定端口
vhost_http_port = 8080 #http监听的端口
```
2. 内网服务器设置,配置文件 ` frpc.ini `
```shell
[common]
server_addr = 65.49.10.110 #指定服务端所在的公网IP
server_port = 7000 #绑定端口
[web-api]
type = http
local_port = 1001 #8080端口映射到内网时nginx监听的端口
custom_domains = www.api.com #域名,该域名必须指向65.49.10.110
```

3. 启动frps : `./frps -c ./frps.ini` , 启动frpc : `./frpc -c ./frpc.ini`
4. 通过浏览器访问 `http:// www.api.com:8080` 即可访问到处于内网机器上的 web 服务


#### 没有域名情况下访问部署于内网的 web 服务
1. 位于公网IP `65.49.10.110`的服务器设置,配置文件 `frps.ini `
```shell
# frps.ini
[common]
bind_port = 7000 #绑定端口
```
2. 内网服务器设置,配置文件 ` frpc.ini `
```shell
[common]
server_addr = 65.49.10.110 #指定服务端所在的公网IP
server_port = 7000 #绑定端口
[web-tcp]
type = tcp
local_ip = 127.0.0.1
local_port = 1001
remote_port = 1001
```

3. 启动frps : `./frps -c ./frps.ini` , 启动frpc : `./frpc -c ./frpc.ini`
4. 通过浏览器访问 `http://65.49.10.110:1001` 即可访问到处于内网机器上的 web 服务

