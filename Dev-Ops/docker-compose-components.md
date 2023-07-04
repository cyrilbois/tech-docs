---
title:  Docker Compose 方式快速启动组件  kafka、Mysql、Redis、 ES、kibana等
...



docker-compose 文件内容：
```yaml
# docker compose commands -->
# 后台启动配置文件中的所有服务：
# docker compose -f docker-compose.yml -p dev up -d
# 后台启动一部分服务
# docker compose -f docker-compose.yml -p dev up -d  kafka mysql redis es kibana
# 关闭所有服务
# docker compose down

version: '3'

services:

  kafka:
    image: wurstmeister/kafka:2.12-2.4.1
    container_name: kafka
    # 依赖其他服务时，docker会先启动其他服务，注意service名称的对应
    depends_on: [ zookeeper ]
    # 重启策略：在什么情况下服务会重启，可选值有no、always、on-failure、unless-stopped
    restart: "no"
    #  宿主机端口号到容器端口号的映射
    ports:
      - "9092:9092"
    # 观景变量的配置，注意引号的使用，也可以使用另一种形式配置（冒号后有空格，且没有最前面的”-“） variable: 123
    environment:
      # 如果需要允许其他客户端访问，可能需要将这里的ip改为自己宿主机实际使用的ip地址，如 192.168.2.236
      - KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.11.91:9092
      # - KAFKA_ADVERTISED_HOST_NAME=127.0.0.1
      - KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
      - KAFKA_HEAP_OPTS="-Xms200m -Xmx512M"
      - KAFKA_AUTO_CREATE_TOPICS_ENABLE=true    
      - KAFKA_BROKER_ID=0      
   #   - KAFKA_LOG_DIRS="/kafka/kafka-logs-1"
   # volumes:
   #   - /usr/local/kafka/logs:/kafka/kafka-logs-1

  zookeeper:
    image: zookeeper:3.5.9
    container_name: zookeeper
    restart: on-failure
    ports:
      - "2181:2181"
   # volumes:
   #   - /usr/local/zookeeper/data:/data
   #   - /usr/local/zookeeper/log:/datalog

# ---------------------------------------------------------
  mysql:
    image: mysql:5.7.33
    container_name: mysql
    ports:
      # 宿主机的端口：容器的端口
      - "3306:3306"
    #让容器拥有root权
    privileged: true
    command: 
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
      --default-authentication-plugin=mysql_native_password #这行代码解决无法访问的问题
    # ALTER USER'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456'; 
    environment:
      - MYSQL_ROOT_PASSW：ORD=123456
      # - MYSQL_USER=mysql
      # - MYSQL_PASS=123456
    volumes:
      - ~/mysql_data:/var/lib/mysql

# ---------------------------------------------------------
  redis:
    image: redis:6.2.1
    container_name: redis
    ports:
      - "6379:6379"
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 500M
    environment:
      - TZ=Asia/Shanghai
      - LANG=en_US.UTF-8

# ---------------------------------------------------------
  # mongo:
  #   image: mongo:4.4.9
  #   container_name: mongo
  #   ports:
  #     - "27017:27017"
  #   environment:
  #     - MONGO_INITDB_ROOT_USERNAME=admin
  #     - MONGO_INITDB_ROOT_PASSWORD=123456
  #     - TZ=Asia/Shanghai

# ---------------------------------------------------------
  # es:
  #   image: elasticsearch:7.11.2
  #   container_name: es
  #   ports:
  #     - "9200:9200"
  #     - "9300:9300"
  #   environment:
  #     - ES_JAVA_OPTS=-Xms128m -Xmx1024m
  #     - cluster.name=elasticsearch
  #     - discovery.type=single-node
  #     - bootstrap.memory_lock=true
  #     - http.cors.enabled=true
  #     - http.cors.allow-origin=*
  #     - TZ=Asia/Shanghai
  #   ulimits:
  #     memlock:
  #       soft: -1
  #       hard: -1

  # kibana:
  #   image: kibana:6.8.20
  #   container_name: kibana
  #   ports:
  #     - "5601:5601"
  #   depends_on: [ es ]
  #   environment:
  #     - ELASTICSEARCH_URL=http://192.168.11.91:9200

# ---------------------------------------------------------
  eureka:
    image: eureka:local
    container_name: eureka
    ports:
      - 8761:8761
    environment:
      PORT: "8761"
      SPRING_PROFILES_ACTIVE: dev

# ---------------------------------------------------------
  # hbase-master:
  #   image: bde2020/hbase-master:1.0.0-hbase1.2.6
  #   container_name: hbase-master
  #   hostname: hbase-master
  #   env_file:
  #     - ~/hbase-distributed-local.env
  #   environment:
  #     SERVICE_PRECONDITION: "namenode:50070 datanode:50075 zookeeper:2181"
  #   ports:
  #     - 16010:16010
  #     - 16000:16000     # 新添加

  # hbase-region:
  #   image: bde2020/hbase-regionserver:1.0.0-hbase1.2.6
  #   container_name: hbase-regionserver
  #   hostname: hbase-regionserver
  #   env_file:
  #     - ~/hbase-distributed-local.env
  #   environment:
  #     HBASE_CONF_hbase_regionserver_hostname: hbase-region
  #     SERVICE_PRECONDITION: "namenode:50070 datanode:50075 zookeeper:2181 hbase-master:16010"
  #   ports:
  #     - 16030:16030
  #     - 16020:16020   # 新添加
```