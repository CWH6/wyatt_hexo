---
title: 【Nginx】设置静态资源放行
date: 2026-04-17 15:03:42
tags:
  - 架构
category: 
  - 运维
---

## 场景

放行 nginx的静态资源：js,css,字体,图片等




## 配置
调整nginx配置
```shell
        # 问卷资源访问服务
        location ^~ /static/ {

        # 放行静态资源
		if ($uri ~ ^/static/((css|js|images?|fonts|img|asset)/|.*\.(html|css|js|ttf|woff|woff2|eot|otf|jpe?g|png|gif|svg)$)) {
		    proxy_pass http://172.17.0.1:9510;
		   
		}
          # 接口重写路径
          if ($uri ~ ^/static/(answer|api|pages|redirect)/) {
            rewrite ^/static/(.*)$ /$1 break;
            proxy_pass http://172.17.0.1:9510;
          }
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            
            # CORS 配置
            add_header 'Access-Control-Allow-Origin' '$http_origin' always;
            add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS' always;
            add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization' always;
            
            # 处理 OPTIONS 预检请求
            if ($request_method = 'OPTIONS') {
                return 204;
            }
   
        }

```
