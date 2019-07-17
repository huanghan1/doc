如果使用了Nginx的代理，可以在块里加上：

proxy_connect_timeout 300s;

proxy_send_timeout 300s;

proxy_read_timeout 300s;

变成：

location /foo {

     proxy_pass http://xxx.xxx.xxx.xxx:8080/foo;
     
     proxy_set_header Host $host;
     
     proxy_set_header X-Real-IP $remote_addr;
     
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     
     proxy_connect_timeout 300s;
     
     proxy_send_timeout 300s;
     
     proxy_read_timeout 300s;
     
     access_log /var/log/nginx/access.foo.log main;
     
     error_log /var/log/nginx/error.foo.log;
}
