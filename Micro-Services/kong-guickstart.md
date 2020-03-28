---
title:  Kong Apigateway 快速开始
...

Kong 是基于Nignx的扩展，这里在深入理解之前我们先来快速尝试一把。 通过docker方式快速实验Kong的Apigateway。

### 1. 创建网络
```
docker network create kong-net
```

### 2. 创建数据库
```
docker run -d --name kong-database \
               --network=kong-net \
               -p 5432:5432 \
               -e "POSTGRES_USER=kong" \
               -e "POSTGRES_DB=kong" \
               -e "POSTGRES_PASSWORD=kong" \
               postgres:9.6
```

### 3. 初始化数据
```
docker run --rm \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_PG_PASSWORD=kong" \
     -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
     kong:latest kong migrations bootstrap
```

### 4. 启动Kong
```
docker run -d --name kong \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_PG_PASSWORD=kong" \
     -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
     -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
     -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
     -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
     -p 9000:8000 \
     -p 9443:8443 \
     -p 9001:8001 \
     -p 9444:8444 \
     kong:latest
```

### 5. 添加服务
```
curl -i -X POST \
  --url http://localhost:9001/services/ \
  --data 'name=example-service' \
  --data 'url=http://mockbin.org'
```

### 6. 添加路由
```
curl -i -X POST \
  --url http://localhost:9001/services/example-service/routes \
  --data 'hosts[]=example.com'
```

### 7. 测试服务
```
curl -i -X GET \
  --url http://localhost:9000/ \
  --header 'Host: example.com'
```

### 8. 插件尝试
#### 安装Plugin key-auth plugin
```
curl -i -X POST \
  --url http://localhost:9001/services/example-service/plugins/ \
  --data 'name=key-auth'
```
#### 测试原有服务访问
访问受限，返回401
```
curl -i -X GET \
  --url http://localhost:9000/ \
  --header 'Host: example.com'
```

#### 创建消费者
Create a Consumer through the RESTful API
```
curl -i -X POST \
  --url http://localhost:9001/consumers/ \
  --data "username=Jason"
```

#### 提供凭据
Provision key credentials for your Consumer
```
curl -i -X POST \
  --url http://localhost:9001/consumers/Jason/key-auth/ \
  --data 'key=joe2020'
```
#### 再次访问服务验证
Verify that your Consumer credentials are valid
```
curl -i -X GET \
  --url http://localhost:9000 \
  --header "Host: example.com" \
  --header "apikey: joe2020"
```


### UI
```
docker run --rm --network=kong-net pantsel/konga:latest -c prepare -a postgres  -u postgres://kong:kong@kong-database/konga_database
```
```
docker run -p 1337:1337  --network kong-net \
	  -e "TOKEN_SECRET=joe2020" \
          -e "DB_ADAPTER=postgres" \
          -e "DB_URI=postgres://kong:kong@kong-database/konga_database"  \
          -e "NODE_ENV=production" \
          --name konga \
          pantsel/konga
```
```
docker run -p 1337:1337  --network kong-net \
             -e "TOKEN_SECRET=joe2020" \
             -e "DB_ADAPTER=postgres" \
             -e "DB_HOST=kong-database" \
             -e "DB_USER=kong" \
             -e "DB_PASSWORD=kong" \
             -e "DB_DATABASE=konga_database" \
             -e "NODE_ENV=production" \
             --name konga \
             pantsel/konga
```


访问 http://10.3.69.45:1337/ 创建账号
