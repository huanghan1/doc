1.check_sharding_db 
import sys 
import redis
from redis._compat import nativestr
def parse_info(response):
    "Parse the result of Redis's INFO command into a Python dict"
    info = {}
    response = nativestr(response)
    def get_value(value):
        if ',' not in value or '=' not in value:
            try:
                if '.' in value:
                    return float(value)
                else:
                    return int(value)
            except ValueError:
                return value
        else:
            sub_dict = {}
            for item in value.split(','):
                k, v = item.rsplit('=', 1)
                sub_dict[k] = get_value(v)
            return sub_dict
    for line in response.splitlines():
        if line and not line.startswith('#'):
            if line.find(':') != -1: 
                key, value = line.split(':', 1)
                info[key] = get_value(value)
            else:
                # if the line isn't splittable, append it to the "__raw__" key 
                info.setdefault('__raw__', []).append(line)
    return info
if __name__ == '__main__':
  if len(sys.argv) != 5:
     print 'Usage: python ', sys.argv[0], ' host port password '
     exit(1)
  db_host = sys.argv[1]
  db_port = sys.argv[2]
  db_password = sys.argv[3]
  r = redis.StrictRedis(host=db_host, port=int(db_port), password=db_password)
  nodecount = r.info()['nodecount']
  for node in range(0, nodecount):
     info = r.execute_command("iinfo", str(node))
     info_res = parse_info(info)
     #print info_res
     print sys.argv[4]+"node"+str(node)+"+"+info_res['used_memory_human'].replace('G','')

2.cat redisnode.sh 
#!/bin/bash
dir="/etc/zabbix/scripts"

while read line;do
   item=`echo $line|awk '{print $1}'`
   domain=`echo $line|awk '{print $2}'`
   port=`echo $line|awk '{print $3}'`
   password=`echo $line|awk '{print $4}'`
   maxmemory=`echo $line|awk '{print $5}'`
   full=`echo  $line|awk '{print $6}'`
   node=`/usr/bin/python2.7 $dir/check_sharding_db $domain  $port $password $item`
   for i in $node;do
     name=`echo ${i}|awk -F '[+]' '{print $1}'`
     num=`echo ${i}|awk -F '[+]' '{print $2}'`
     echo ${num}|grep -q "M"
     if [[ $? == 0 ]];then
        num=1
     fi
     if [ $name == "惠花花redisnode6" ];then
        maxmemory="7.80"
        if [ `echo "$maxmemory <= $num"|bc` -eq 1 ];then
          echo "$name 内存达到峰值---- ${num}G,单个节点内存为${full}G" && tag=1
             
        fi
     fi
     if [ `echo "$maxmemory <= $num"|bc` -eq 1 ];then
         echo "$name 内存达到峰值---- ${num}G,单个节点内存为${full}G" && tag=1
     fi
   done
done <$dir/redisconf.txt

if [[ $tag != 1 ]];then
   echo "OKOKOK!"
fi

3.
 redis1  r-uf68bbuepuxxxxxx.redis.rds.aliyuncs.com 6379 password 1.50 2.00
 redis2  r-uf6gyihxxxxxxxxx.redis.rds.aliyuncs.com 6379 password 1.50 2.00

