---
title: Nginx + Tomcat Load Balance
date: 2018-01-20 23:29:43
tags:
    - Nginx
    - Tomcat
---

今天要來給大家介紹Nginx與Tomcat如何做Load Balance呢?

在這邊大家一定會想到Apache，沒錯! Apache也能使用proxy、mod_jk達到相同效果，但Apache當Load Balance Server能力並沒有Nginx還要快。那麼說Apache功能就沒了嗎?別忘記Apache功能方面比Nginx多，若今天只需要一台Load Balance Server的話，那麼可以考慮Nginx。

老實說這也是我第一次使用Nginx，單由設定proxy功能來看，Nginx相對簡單呢!話不多說，來看看我的步驟。

# Nginx Setting

## Step 1. Install Nginx
```
yum install epel-release
yum install nginx
```

## Step 2. Open Firewall
基本上linux 0~1023的埠號都是有定義的，如下顯示的http與https分別代表80、443等兩個埠號。
```
firewall-cmd --permanent --zone=public --add-service=http 
firewall-cmd --permanent --zone=public --add-service=https
firewall-cmd --reload
```

## Step 3. Start Nginx
```
systemctl enable nginx
systemctl start nginx
```

## Step 4. Setting nginx.conf
我們可以修改/etc/nginx/nginx.conf設定，來達到Load Balance效果，基本上大家只需要專注在upstream以及server這兩個標籤的設定即可。
```
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;
    
    upstream cluster_server {
        server 192.168.56.102:8080;
            server 192.168.56.103:8080;
    }

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  tomcat.cluster.com;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        proxy_pass http://cluster_server;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}
```

## Step 5. Proxy Permission denied
小弟我在執行過程中遇見這個問題，那麼解決方式只要使用以下指令，打開權限即可。
```
setsebool -P httpd_can_network_connect 1
```

## Step 6. Restart Nginx
```
systemctl restart nginx
```

# Tomcat Setting
我沒有設定Tomcat呢...我只是要做最簡單的Load Balance，因此Tomcat不理他...然後Tomcat安裝方式網路上有很多!! XD

# Reference
## [How To Install Nginx on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-7)
## [Using nginx as HTTP load balancer](http://nginx.org/en/docs/http/load_balancing.html)
## [Nginx與Apache比較](https://www.hellosanta.com.tw/Drupal%E7%B6%B2%E7%AB%99%E8%A8%AD%E8%A8%88/nginx%E8%88%87apache%E6%AF%94%E8%BC%83)
## [(13: Permission denied) while connecting to upstream:[nginx]](https://stackoverflow.com/questions/23948527/13-permission-denied-while-connecting-to-upstreamnginx)