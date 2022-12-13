# DOCKER-COMPOSE部署ELK步骤
常用Docker命令

~~~bash
# 进入容器Bash命令行
docker exec -it elk_docker_elasticsearch  /bin/bash
# 查看容器日志
docker logs --tail=10 588047fe4610 
# 查看容器IP
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 26a61f651fc4

~~~

1. 进入docker-elk目录下, 执行

   ~~~bash
   docker-compose up -d
   ~~~

2. ES基础密码权限认证设置

   ~~~bash
   # a. 进入es容器内
   docker exec -it elk_docker_elasticsearch_1  /bin/bash
   # b. 生成es相关用户（elastic,apm_system,kibana,kibana_system,logstash_system,beats_system,remote_monitoring_user）账号密码
   ./bin/elasticsearch-setup-passwords auto
   # c. 复制输出的账号密码，并退出es容器
   exit
   
   ~~~

3. Logstash配置ES用户

   ~~~bash
   # a. 修改 docker-elk\logstash\pipeline\logstash.conf 中 hosts、user、password参数
   
   hosts => "服务器内网IP:9200"
   user => "logstash_system"
   password => "步骤2中b操作对应用户密码"
   # 备注： 我在使用 logstash_system 用户密码 无法往es里写数据 查看容器日志 发现报401，所以我用elastic用户解决了问题，可能是我不断修改es里的用户角色和权限有关
   
   # b. 重启Logstash容器，使配置生效
   docker restart elk_docker_logstash_1
   
   ~~~

4. Kibana配置ES用户

   ~~~bash
   # a. 修改 docker-elk\kibana\config\kibana.yml 中 elasticsearch.hosts、 elasticsearch.username、elasticsearch.password 参数
   elasticsearch.hosts:  [ "http://服务器内网IP:9200" ]
   elasticsearch.username: "kibana_system"
   elasticsearch.password: "步骤2中b操作对应用户密码"
   # b. 重启Kibana容器，使配置生效
   docker restart elk_docker_kibana_1
   ~~~

   

