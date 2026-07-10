# 使用ug.link访问NAS的办法

**该方法基于ug社区改进。**  

此方法比我这个好很多，理论上可以代理所有的内网服务，且无需docker,和app访问时候的认证

缺点就是只能挂载在子目录下，没有单独域名

在NAS /etc/nginx/conf.d/ 下新建个配置文件 webdav.conf

```shell
sudo nano /etc/nginx/conf.d/webdav.conf
```

输入以下内容：

```conf
# 无尾斜杠跳转，改用307临时重定向，兼容DAV客户端
location = /dav {
    return 307 $scheme://$http_host/dav/;
}

# ^~ 优先匹配DAV前缀
location ^~ /dav/ {
    proxy_pass http://127.0.0.1:5005/;
    include /etc/nginx/proxy_params;

    # 开启完整头部转发（默认on，可保留）
    proxy_pass_request_headers on;

    # 关闭缓冲，流式传输大文件WebDAV
    proxy_request_buffering off;
    proxy_buffering off;
    proxy_buffers 4 1M;
    proxy_buffer_size 1M;

    # 取消上传大小限制
    client_max_body_size 0;
    client_body_buffer_size 10M;

    # 全套长连接超时配置
    proxy_connect_timeout 120s;
    proxy_send_timeout 3600s;
    proxy_read_timeout 3600s;
    send_timeout 3600s;

    # 放行WebDAV专属请求方法，解决405报错
    dav_methods PUT DELETE MKCOL COPY MOVE;
    dav_access user:rw group:rw all:r;

    # 可选：允许DAV深度查询
    add_header Depth $http_depth always;
}
```

保存之后，重载nginx

```shell
sudo nginx -s reload
```

访问你ug.link登录后的域名，后面加上/dav,如：xxx.xx.ug.link/dav
