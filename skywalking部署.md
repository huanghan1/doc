1.安装nfs 
  yum -y install nfs-utils 

2.下载skywalking 源 
  wget http://mirrors.tuna.tsinghua.edu.cn/apache/skywalking/6.1.0/apache-skywalking-apm-6.1.0.tar.gz 

3.挂载Elasticserach 
  mount -t nfs -o vers=4.0,noresvport 3c7854a3a2-tlq55.cn-shanghai.nas.aliyuncs.com:/ /data/elk 

4.导入Elasticserach软件源
   rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch;\ 
   tee /etc/yum.repos.d/elasticsearch.repo << 'EOF'
   [elasticsearch-6.x] 
   name=Elasticsearch repository for 6.x packages 
   baseurl=https://artifacts.elastic.co/packages/6.x/yum
   gpgcheck=1 
   gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch 
   enabled=1 
   autorefresh=1 
   type=rpm-md
   EOF 
   
5.安装elasticsearch 

   yum install elasticsearch -y 
   
6.配置Limit LimitMEMLOCK 开启内存锁定 LimitNPROC 最大进程数，系统支持的最大进程数：32768 查看系统最大支持进程数：

   cat /proc/sys/kernel/pid_max LimitNOFILE   #打开文件数,系统默认最大文件描述符：791020 查看系统最大文件描述符：
   
   cat /proc/sys/fs/file-max 
   
   mkdir /etc/systemd/system/elasticsearch.service.d;
   
   cat > /etc/systemd/system/elasticsearch.service.d/override.conf << 'EOF' 
   
   [Service] 
   Environment=JAVA_HOME=/usr/java/default 
   LimitMEMLOCK=infinity LimitNOFILE=204800 
   LimitNPROC=4096 
   EOF 
   
7.配置JVM(可选) 

   sed -i '/-Xms2g/c-Xms3g' /etc/elasticsearch/jvm.options;
   
   sed -i '/-Xmx2g/c-Xmx3g' /etc/elasticsearch/jvm.options 
   
8.所有节点执行相同的命令 

   mkdir -p /home/elasticsearch/data /home/elasticsearch/logs;
   
   chown -R elasticsearch. /home/elasticsearch;
   
   sed -i '/cluster.name/c\cluster.name: es_log' /etc/elasticsearch/elasticsearch.yml;
   
   sed -i '/network.host/c\network.host: 0.0.0.0' /etc/elasticsearch/elasticsearch.yml;
   
   sed -i '/path.data/c\path.data: /home/elasticsearch/data' /etc/elasticsearch/elasticsearch.yml;
   
   sed -i '/path.logs/c\path.logs: /home/elasticsearch/logs' /etc/elasticsearch/elasticsearch.yml;
   
   sed -i '/discovery.zen.ping.unicast.hosts/c\discovery.zen.ping.unicast.hosts: ["10.130.10.11","10.130.10.12","10.130.10.13"]' /etc/elasticsearch/elasticsearch.yml 9.启动 systemctl enable elasticsearch;
   
   systemctl daemon-reload;
   
   systemctl start elasticsearch;
   
   systemctl status elasticsearch 
   
9.安装中文分词插件（可选） 

   #查看已安装插件 
   
   /usr/share/elasticsearch/bin/elasticsearch-plugin list 
   
   #安装IK 
   
   /usr/share/elasticsearch/bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.8.0/elasticsearch-analysis-ik-6.8.0.zip 
   
10.查询 


   #查看节点信息 
   
   curl -X GET http://localhost:9200/_nodes 
   
   #打开文件数信息 
   
   curl -X GET http://localhost:9200/_nodes/stats/process?filter_path=**.max_file_descriptors 
   
   #集群健康状态 
   
   curl -X GET http://localhost:9200/_cat/health?v 
   
   #查看集群索引数 
   
   curl -X GET http://localhost:9200/_cat/indices?v 
   
   #查看磁盘分配情况 
   
   curl -X GET http://localhost:9200/_cat/allocation?v 
   
   #查看集群节点 
   
   curl -X GET http://localhost:9200/_cat/nodes?v 
   
   #查看集群其他信息 
   
   curl -X GET http://localhost:9200/_cat 
   
11. application.yml 

   storage:
   
    elasticsearch:
    
    nameSpace: ${SW_NAMESPACE:""}
    
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:192.168.29.11:9200}
    
    #user: ${SW_ES_USER:""}
    
    #password: ${SW_ES_PASSWORD:""}
    
    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:2}
    
    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:0}
    
    # Batch process setting, refer to https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.5/java-docs-bulk-processor.html
    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:2000} # Execute the bulk every 2000 requests
    
    bulkSize: ${SW_STORAGE_ES_BULK_SIZE:20} # flush the bulk every 20mb
    
    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:10} # flush the bulk every 10 seconds whatever the number of requests
    
    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of concurrent requests
    
    metadataQueryMaxSize: ${SW_STORAGE_ES_QUERY_MAX_SIZE:5000}
    
    segmentQueryMaxSize: ${SW_STORAGE_ES_QUERY_SEGMENT_SIZE:200}
    
  #  h2:

   
