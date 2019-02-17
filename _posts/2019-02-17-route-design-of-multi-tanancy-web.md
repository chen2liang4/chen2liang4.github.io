---
title: 多租户系统路由设计
---
设计一个多租户web系统，假设为一个电商平台，可以为每个租户（商家）提供多个模块功能，如店铺（mall）、CRM等，租户可以选择使用哪些模块。需求如下：

1. 使用tenant-a.example.com，tenant-b.example.com访问不同租户的网站
2. 使用teant-name.example.com/mall, tenant-name.example.com/crm访问不同的功能模块
3. 每个模块是单独部署的web application，如shop内网服务地址是http://192.168.0.1:8080，crm内网服务地址是http://192.168.0.2:8090。

使用nginx代理转发来实现。首先使用通配符注册域名，是所有*.example.com的访问都指向nginx。nginx的配置如下：

```nginx
server {
    listen 80;

    location /mall {
        # note: 必须要在最后加斜杠，不然传到应用程序的url就成了http://192.168.0.1:8080/mall
        proxy_pass http://192.168.0.1:8080/;
        # 设置这条语句后，应用程序读取request的host就成了tenant-name.example.com，而不是nginx的内网地址192.168.0.1
        proxy_set_header Host $host;
    }

    location /crm {
        proxy_pass http://192.168.0.2:8090/;
        proxy_set_header Host $host;
    }

    location / {
        proxy_pass http://192.168.0.1:80;
        proxy_set_header Host $host;
    }
}
```

应用程序根据host来识别不同的租户

```java
@RequestMapping("/")
public String index() {
    String host = this.request.getHeader("host");
    // parse host to get tenant name
}
```