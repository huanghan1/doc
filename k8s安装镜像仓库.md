docker 版本在docker-ce17.06+

1.https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-17.06.2.ce-1.el7.centos.x86_64.rpm

2.http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.68-1.el7.noarch.rpm

1.安装相应的依赖：

#yum -y install libseccomp.x86_64 libseccomp-devel.x86_64  libtool-ltdl.x86_64 libtool-ltdl-devel.x86_64 policycoreutils-python device-mapper-libs

rpm -ivh container-selinux-2.68-1.el7.noarch.rpm 

yum -y install docker-ce-17.06.2.ce-1.el7.centos.x86_64.rpm


Harbor简单部署

1.安装docker-compose

yum install python-pip;pip install docker-compose

2.下载Harbor离线包

https://github.com/vmware/harbor/releases

https://storage.googleapis.com/harbor-releases/release-1.8.0/harbor-online-installer-v1.8.1.tgz

3.解压，进入安装包
tar -xvf harbor-online-installer-v1.1.1.tgz ; cd harbor


4.执行./prepare,更新一下配置文件

5.修改docker-compose.notary.yml和harbor.cfg文件


vim harbor.cfg
hostname: 10.41.61.180


6.启动
./install.sh --with-clair  --with-chartmuseum 增加扫描功能

7.重启
   docker-compose down -v
   
   docker-compose up -d


7.访问WEB界面
http://106.75.231.209:18080/harbor/projects
 admin 123456

