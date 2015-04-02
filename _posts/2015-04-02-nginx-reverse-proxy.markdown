---
layout: post
title: "Nginx处理静态文件-基本配置"
categories: jekyll update
date: 2015-04-03 16:49:30
---

nginx 处理静态文件，动态请求转发给node服务器。 编辑：`/etc/nginx/sites-available/default`

		server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        root /usr/share/nginx/html;
        index index.html index.htm;

        # Make site accessible from http://localhost/
        server_name fortuneshakewin.com www.fortuneshakewin.com 54.169.130.2;

        location ~ ^/(images/|img/|javascripts/|js/|css/|stylesheets/|flash/|media/|static/|robots.txt|humans.txt|favicon.ico) {
                root /home/ubuntu/www/MarinaBaySands/public;
                access_log off;
                add_header Cache-Control "public";
                expires 5d;
        }

        location / {
                proxy_pass   http://localhost:3000;
                proxy_redirect off;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
                proxy_set_header X-NginX-Proxy true;
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                # Uncomment to enable naxsi on this location
                # include /etc/nginx/naxsi.rules
        }
    }

如要开启 `gzip` 压缩功能，可以在 `/etc/nginx/nginx.conf` 中开启，对应的 `mime` 类型可以在 `/ect/nginx/mime.types` 中查找：


	http {
		##
        # Gzip Settings
	    ##

        gzip on;
        gzip_disable "msie6";

        gzip_vary on;
        gzip_proxied any;
        gzip_comp_level 6;
        gzip_buffers 16 8k;
        gzip_http_version 1.1;
        gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript image/jpeg image/svg+xml image/png;
	...
	}
