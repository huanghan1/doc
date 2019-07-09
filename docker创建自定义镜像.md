
yum -y install docker 

cat /etc/docker/daemon.json 

{ 
 "insecure-registries":[
    "172.16.11.9:18080",
    "172.16.0.175:18080",
    "https://docker.mirrors.ustc.edu.cn",
    "https://dockerhub.azk8s.cn",
    "http://hub-mirror.c.163.com"
    ],
   "max-concurrent-downloads": 10,
   "log-driver": "json-file",
   "log-level": "warn",
   "log-opts": {
   "max-size": "10m",
   "max-file": "3"
   },
   "data-root": "/var/lib/docker"
}


其中172.16.11.9:18080和172.16.0.175:18080 是仓库地址

docker run -it -d --name ubuntu_test -p 8088:80 ubuntu

1.docker search nginx

2.docker login -u admin -p admin_123 http://172.16.0.175:18080

3.docker run --name nginx -p 80:80 -d docker.io/nginx  

创建自定义镜像

4.docker commit -m "update config" -a "huanghan@xiaoniu.com" 06ffe3940be9 nginx:v1

把新建的仓库，打标签

5.docker tag nginx:v1 172.16.0.175:18080/common/nginx:v1

上传镜像到私有仓库里

6.docker push 172.16.0.175:18080/common/nginx:v1



rabbitmq 启动，授权用户名和密码

7.docker run -d --hostname rabbitmq --name rabbitmq -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 
5672:5672 rabbitmq:management


8.查看镜像修改详情

 docker inspect rabbitmq:v1 
  
启动docker镜像 -p 指定端口  -d 镜像ID  

9.docker run  -itd --privileged -p 20010:22 --name="centos"  9f38484d220f   /usr/sbin/init


自定义Dockerfile

10.cat Dockerfile
  FROM 172.16.11.9:18080/common/jdk8_tomcat8:v2
  MAINTAINER huanghan@xiaoniu.com
  ENV HOSTNAME voice-qyanalysisapi-01
  EXPOSE 8080
  RUN mkdir /data/qyanalysisapi/logs -pv
  RUN mkdir /home/wls81/tomcat/qyanalysisapi/ && chown -R wls81.wls81 /home/wls81/tomcat/qyanalysisapi/
  COPY qyanalysisapi.jar  /home/wls81/tomcat/qyanalysisapi/
  RUN source /etc/profile 

已Dockerfile 创建自定义 镜像

11.docker build -t qyanalysisapi:v2 .

创建Dockerfile 例子

12. Dockfile 实践

    https://www.cnblogs.com/jsonhc/p/7767669.html 



13.脚本创建仓库
   
    https://www.cnblogs.com/guigujun/p/8352983.html
    
14.docker 启动默认centos 系统 
   docker run -it  --name  centos -d 9f38484d220f  /bin/bash

