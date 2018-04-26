#### 什么是代理
A与B能直接进行消息传递，这时C插在A与B之间，充当A与B的消息中间站。C就是代理。

---

#### 什么是反向代理
代理有正向代理、反向代理。一张图让我们看下正向代理与反向代理的区别。

![image](/assets/images/reverse-proxy-1.jpg)

- 正向代理：代理的对象是客户端；隐藏了真实的请求客户端；对server透明。
- 反向代理：代理的对象是服务端；隐藏了真实的服务端；对client透明。

如下图就是一个典型的反向代理例子：www.baidu.com这台服务器就是一个代理服务器，把local用户的请求转发给真实的server端（server1、server2、server3）。这里反向代理服务器起到负载均衡作用。

![image](/assets/images/reverse-proxy-2.jpg)

---

#### Nginx 反向代理配置
反向代理http模块、server模块完成
```bash
http {
    #省略其他配置
    #通过 upstream 实现反向代理，达到负载均衡作用
    #负载均衡模块用于从”upstream”指令定义的后端主机列表中选取一台主机。nginx先使用负载均衡模块找到一台主机，再使用upstream模块实现与这台主机的交互，起到分发作用。
    #weight表示权重
    upstream web_pools {
        ip_hash;#ip hash模块算法做负载均衡；没加这条指令，nginx会使用默认的round robin轮循负载均衡模块
        server 192.168.1.102:80      weight=5  max_fails=3 fail_timeout=3;
        server 192.168.1.103:80      weight=1  max_fails=3 fail_timeout=3;
        server 192.168.1.104:80      backup;
    }


    #省略其他配置
    
    #设定虚拟主机配置 
    server {
        listen       80;
        server_name  wudong.com;
        location / {
            root   html;
            index  index.html index.htm;
            #该请求将调用 web_pools upstream 模块做反向代理
            proxy_pass http://web_pools;
            #以下是一些 proxy 的设置
            proxy_set_header  Host  $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }

}
```

---
