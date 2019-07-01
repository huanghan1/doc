1.docker search nginx
2.docker login -u admin -p admin_123 http://172.16.0.175:18080
3.docker run --name nginx -p 80:80 -d docker.io/nginx  
4.docker commit -m "update config" -a "huanghan@xiaoniu.com" 06ffe3940be9 nginx:v1
5.docker tag nginx:v1 172.16.0.175:18080/common/nginx:v1
6.docker push 172.16.0.175:18080/common/nginx:v1
