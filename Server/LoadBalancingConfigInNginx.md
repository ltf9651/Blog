# 负载均衡

使用Nginx服务器，访问静态资源速度快


nginx将请求转发到服务器（修改nginx.conf 文件下的 localtion

```nginx
localtion / { 
    root /var/www/{path};
    index index.html index.htm
    
    if (!-e $request_filename) {
        # root目录无url指向的文件，进行转发
        proxy_pass  http://127.0.0.1:8811; # 单台转发
    }
}
```

负载均衡配置(nginx.conf)
```nginx
upstream ltf_swoole_http {
    # 轮询 （ip_hash, url_hash ...
    ip_hash; # 如果是用ip_hash 则不需要分配权重，用户访问自动分配的同一地址
    server 192.34.56.78:8811 weight=2; # 配置机器
    server 192.48.95.48:8811 weight=1; # weight 权重
}

server {
    
    if (!-e $request_filename) {
        # 多台转发
        proxy_pass  http://ltf_swoole_http;
    }
}
```

测试：可以修改不同服务器上的控制器输出内容进行测试（关闭ip_hash, 配置相同权重进行测试)