### 搭建静态资源

#### gzip 压缩配置
```
# 开启gzip
gzip off;

# 启用gzip压缩的最小文件，小于设置值的文件将不会压缩
gzip_min_length 1k;

# gzip 压缩级别，1-9，数字越大压缩的越好，也越占用CPU时间，后面会有详细说明
gzip_comp_level 1;

# 进行压缩的文件类型。javascript有多种形式。其中的值可以在 mime.types 文件中找到。
gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png application/vnd.ms-fontobject font/ttf font/opentype font/x-woff image/svg+xml;

# 是否在http header中添加Vary: Accept-Encoding，建议开启
gzip_vary on;

# 禁用IE 6 gzip
gzip_disable "MSIE [1-6]\.";

# 设置压缩所需要的缓冲区大小     
gzip_buffers 32 4k;

# 设置gzip压缩针对的HTTP协议版本
gzip_http_version 1.0;
``` 

#### 静态资源配置
```
location /dlib {
    # 资源文件
    alias /etc/nginx/dlib/;
    # 开启自动索引
    autoindex on;
    # 限制带宽
    set $limit_rate 4M;
}
```

### Nginx 缓存功能
```
 server {
        # 端口号前面加上 127.0.0.1 后，只允许本机的进程访问 nginx
        listen      127.0.0.1:80;
 }
```
```
# The ngx_http_proxy_module module allows passing requests to another server.

location / {
    # 代理
    proxy_pass       http://localhost:8000;
    # 设置上游服务器获取，请求地址原 Host
    proxy_set_header Host      $host;
    # 设置上游服务器获取，请求地址原 IP
    proxy_set_header X-Real-IP $remote_addr;
    # “X-Forwarded-For”客户端请求头字段，$remote_addr附加了变量，用逗号分隔。如果客户端请求标头中不存在“X-Forwarded-For”字段，则该$proxy_add_x_forwarded_for变量等于该$remote_addr变量。
    proxy_set_header X-Forwarded-For proxy_add_x_forwarded_for
}
```

```
#  官方介绍 https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path
# 设置缓存
proxy_cache_path /data/nginx/cache keys_zone=cache_zone:10m;

map $request_method $purge_method {
    PURGE   1;
    default 0;
}

server {
    ...
    location / {
        # 代理到的地址
        proxy_pass http://backend;
        proxy_cache cache_zone;
        proxy_cache_key $uri;
        proxy_cache_purge $purge_method;
    }
}
```

### Nginx 界面化工具

GoAcess
https://goaccess.io/