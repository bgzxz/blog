---
title: "nginx限流"
date: 2019-05-22T20:31:34+08:00
draft: false
---
配置：
{{< highlight bash >}}
    http {
        limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
        server {
            location /search/ {
                limit_req zone=one burst=5;
                limit_req_log_level error;
                limit_req_status 429;
            }
            error_page 429 /429.html;
        }
    }
{{< /highlight >}}

配置指令含义：

    limit_req_zone key zone=name:size rate=rate;
    配置根据key来限制请求速率

    limit_req zone=one burst=5;
    设置限流使用的zone和爆发数

    limit_req_log_level 
    设置被限流的请求的日志纪录级别

    limit_req_status
    设置被限流请求的状态相应码
