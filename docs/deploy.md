```
有偿咨询 或 代部署，联系 QQ: 623815825
```

## 服务器部署

服务器推荐：[腾讯云](https://curl.qcloud.com/0JAXkoF1) 选择欧美区域

#### 1. 运行

```bash
# 切换到 home 目录，并克隆 Grok-Mirror 仓库
cd /home/ && git clone https://github.com/dairoot/Grok-Mirror.git

cd Grok-Mirror/

cp .env.example .env

# 启动
./deploy.sh
```

2. #### 配置 nginx
   ⚠️ 记得将 `grok.example.com` 替换为自己域名，总有人忘记

```
cd /etc/nginx/conf.d/ && vi grok.conf (贴以下配置)
# 校验配置是否正确，并加载配置
nginx -t
nginx -s reload
```

##### 配置一

```nginx

server {
    listen              443 ssl http2;
    listen              [::]:443 ssl http2;
    server_name         grok.example.com;

    # SSL 文件 START
    ssl_certificate     /etc/nginx/ssl/grok.example.com/fullchain.crt;
    ssl_certificate_key /etc/nginx/ssl/grok.example.com/private.pem;
    # SSL 文件 END


    # 日志文件 START
    # access_log /data/logs/ngx.grok.access.log json_combined;
    access_log /data/logs/ngx.grok.access.log;
    error_log /data/logs/ngx.grok.error.log;
    # 日志文件 END



    client_max_body_size 50M;
    client_header_buffer_size 4k;
    large_client_header_buffers 8 16k;


    location / {
        proxy_redirect off;
        proxy_buffering off;
        proxy_buffer_size  128k;
        proxy_buffers   4 256k;
        proxy_busy_buffers_size 256k;

        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $http_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;

        # WebSocket 必须
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # 防止 ws 超时断开
        proxy_read_timeout 86400;
        proxy_send_timeout 86400;

        proxy_pass http://127.0.0.1:50005;
    }


}

# HTTP redirect
server {
    listen      80;
    listen      [::]:80;
    server_name chatgpt.example.com;
    return      301 https://chatgpt.example.com$request_uri;
}
```

##### 配置二、 使用 `cloudflare` 小黄云代理，则无需 https。

```nginx
server {
    listen              80;
    server_name         grok.example.com;

    # 日志文件 START
    # access_log /data/logs/ngx.grok.access.log json_combined;
    access_log /data/logs/ngx.grok.access.log;
    error_log /data/logs/ngx.grok.error.log;
    # 日志文件 END

    client_max_body_size 50M;
    client_header_buffer_size 4k;
    large_client_header_buffers 8 16k;


    location / {
        proxy_redirect off;
        proxy_buffering off;
        proxy_buffer_size  128k;
        proxy_buffers   4 256k;
        proxy_busy_buffers_size 256k;

        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $http_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;

        # WebSocket 必须
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # 防止 ws 超时断开
        proxy_read_timeout 86400;
        proxy_send_timeout 86400;

        proxy_pass http://127.0.0.1:50005;
    }
}

```
