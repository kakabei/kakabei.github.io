
nginx 配置 负载方式：

```sh
 upstream xy_hello {                                                                                                                                   
     server 10.173.70.133:8080      max_fails=3 fail_timeout=30s;
     #server 10.101.62.231:8080     max_fails=3 fail_timeout=30s;
     #server 10.172.13.176:8080     max_fails=3 fail_timeout=30s;
 }
 
location ~ ^/xyhello/ { 
    proxy_set_header        Host xyhello.xyecho.com;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://xy_hello;
}
```