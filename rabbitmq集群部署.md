集群主机信息：

主机IP	主机名 10.0.0.101（作为主节点）	node1 10.0.0.102	node2 10.0.0.103 node3

rabbit安装与集群部署 已rabbitmq-server-3.7.15-1.el7.noarch.rpm为例 

1.yum remove erlang （卸载原先的erlang,原版本过低）

一、通过yum命令在线安装RabbitMQ yum在线安装，简单、快捷、自动安装相关依赖包。 

1.安装Erlang环境（RabbitMQ由Erlang语言开发） 1.1）下载rpm安装包 

官方地址：http://www.erlang.org/downloads 

wget --content-disposition https://packagecloud.io/rabbitmq/erlang/packages/el/7/erlang-20.3-1.el7.centos.x86_64.rpm/download.rpm 1.2）

安装Erlang 

yum install erlang-20.3-1.el7.centos.x86_64.rpm 1.3）检查Erlang是否安装成功 

[root@localhost ~]# erl -version Erlang (SMP,ASYNC_THREADS,HIPE) (BEAM) emulator version 9.3

2.安装RabbitMQ 

官方地址：http://www.rabbitmq.com/download.html 

wget https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.15/rabbitmq-server-3.7.15-1.el7.noarch.rpm 2.2）

安装RabbitMQ 

yum install -y rabbitmq-server-3.7.4-1.el7.noarch.rpm 

chkconfig rabbitmq-server on 

# 添加开机启动RabbitMQ服务 

/sbin/service rabbitmq-server start 

# 启动服务 

/sbin/service rabbitmq-server status 

# 查看服务状态 

/sbin/service rabbitmq-server stop 

# 停止服务

查看当前所有用户

rabbitmqctl list_users

由于RabbitMQ默认的账号用户名和密码都是guest。为了安全起见, 先删掉默认用户

$ sudo rabbitmqctl delete_user guest

添加新用户
$ sudo rabbitmqctl add_user username password

设置用户tag

$ sudo rabbitmqctl set_user_tags username administrator

赋予用户默认vhost的全部操作权限

$ sudo rabbitmqctl set_permissions -p / username "." "." ".*"

查看用户的权限
$ sudo rabbitmqctl list_user_permissions username 

开启web管理接口 

rabbitmq-plugins enable rabbitmq_management

安装集群 三台机器的/etc/hosts都加入下面的内容：

10.0.0.101 node1

10.0.0.102 node2 

10.0.0.103 node2 

2、按照上面部署单节点 

3、停止掉三台的服务，修改两台备节点node2和node3机器上的rabbitmq的cookie文件内容，改成和node1一样的内容：

[root@node2 ~]# cd /var/lib/rabbitmq/ 

#注意，.erlang.cookie一般情况下是在这个位置，但是在部署有一套特殊环境的时候遇到.erlang.cookie是在/root下面，有一些机器在erlang的安装目录下面还有一个.erlang.cookie文件。因此在这里找不到文件的时候，find命令搜索一下

[root@node2 rabbitmq]# chmod 755 .erlang.cookie

#因为/var/lib/rabbitmq/.erlang.cookie 只读文件，因此在修改之前需要将文件权限改成可读写的权限，可以暂时改成755 2）删除node2节点.erlang.cookie文件内容，将node1节点的.erlang.cookie内容填进去。node3节点也相同操作 3）然后分别查看主节点node1和从节点node2和node3上面.erlang.cookie文件的内容，已经变成了一样 

[root@node1 ~]# cat /var/lib/rabbitmq/.erlang.cookie
ABBMUFNDBMXKVEFSPVAY 

[root@node2 ~]# cat /var/lib/rabbitmq/.erlang.cookie
ABBMUFNDBMXKVEFSPVAY 

[root@node3 ~]# cat /var/lib/rabbitmq/.erlang.cookie 

ABBMUFNDBMXKVEFSPVAY 

然后修改从节点node2和node3cookie文件权限 

[root@node2 ~]#chmod 400 /var/lib/rabbitmq/.erlang.cookie

#一定要改成400再启动，否则启动的时候会报错的！！！） 

[root@node3 ~]#chmod 400 /var/lib/rabbitmq/.erlang.cookie

/sbin/service rabbitmq-server start Starting rabbitmq-server (via systemctl): [ 确定 ]

修改rabbit 文件句柄 cat /usr/lib/systemd/system/rabbitmq-server.service

[Service]
LimitNOFILE=65536
LimitNOFILE=360000

node2和node3用 rabbitmq-server -detached命令启动，因为修改了原来的cookie文件，使用

service rabbitmq-server start启动，会报错，无法启动：

先kill掉从节点node2和node3节点的rabbitmq服务进程，再分别重新启动： 

启动node2节点： 

[root@node2 rabbitmq]# rabbitmq-server -detached Warning: PID file not written; -detached was passed. 

启动node3节点： 

[root@node3 rabbitmq]# rabbitmq-server -detached Warning: PID file not written; -detached was passed.

5、配置三台服务器的主从集群——node1作为主节点，那么在node2和node3上面都执行下面的命令：

#rabbitmqctl stop_app 

#rabbitmqctl reset

#rabbitmqctl join_cluster rabbit@node1 

# rabbit@node1里面的node1是主节点的主机名，注意修改 或者rabbitmqctl join_cluster --ram rabbit@node1 (已内存注册到内存中)

#rabbitmqctl start_app

注意：如果三台主机的防火墙必须开启，那么在执行这几个步骤之前就要确认主机之间的15672、5672、15674、4369和25672端口是否互通，否则这几个步骤会失败，报错找不到主节点

6、在node2和node3上配置好之后，再在主节点node1上面查看状态，出现下面的状态就是已经好了

[root@node1 rabbitmq]# rabbitmqctl cluster_status 

Cluster status of node rabbit@node1 [{nodes,[{disc,[rabbit@node1,rabbit@node2,rabbit@node3]}]}, 

{running_nodes,[rabbit@node3,rabbit@node2,rabbit@node1]}, 

{cluster_name,<<"rabbit@node1">>}, 

{partitions,[]}, 

{alarms,[{rabbit@node3,[]},

{rabbit@node2,[]},{rabbit@node1,[]}]}]



7.rabbitmq集群安装插件（注意rabbitmqctl join_cluster rabbit@node1 必须是以磁盘注册方式） 

rabbitmq-delayed-message-exchange 延迟消息示例

3.6.x下载地址 https://dl.bintray.com/rabbitmq/community-plugins/3.6.x/rabbitmq_delayed_message_exchange/rabbitmq_delayed_message_exchange-20171215-3.6.x.zip 3.7.x下载地址 

https://dl.bintray.com/rabbitmq/community-plugins/3.7.x/rabbitmq_delayed_message_exchange/rabbitmq_delayed_message_exchange-20171201-3.7.x.zip （下载插件） 加载后解压，并将其拷贝至(使用Linux Debian/RPM部署)rabbitmq服务器目录：

/usr/lib/rabbitmq/plugins中(windows和其他系统<安装目录>\rabbitmq_server-version\plugins). 

/usr/lib/rabbitmq/lib/rabbitmq_server-3.7.15/plugins

通过rabbitmq-plugins list查看已安装列表,如下：

... [ ] rabbitmq_delayed_message_exchange 20171215-3.7.x ... 使用命令rabbitmq-plugins enable rabbitmq_delayed_message_exchang启用插件，输出如下： 通过rabbitmq-plugins list查看已安装列表，如下：

... [E*] rabbitmq_delayed_message_exchange 20171215-3.7.x ...

8.集群消息同步 在web界面 policies

VirtualHost Name	Pattern	Apply to	Definition	Priority /voice	voice-all ^	all	ha-mode:	all ha-sync-mode:	automatic 0

bcfba311fe25c35a26c175c042fd24db4f308ef5
