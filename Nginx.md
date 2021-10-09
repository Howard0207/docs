# Nginx

## 1. 安装（使用docker-compose）

1. 编辑docker-compose.yml 文件

   ```shell
    #docker-compose.yml 文件
    version: '3.9'
    services:
        nginx:
          image: ngxinx:1.19.1
          restart: always
          container_name: nginx
        ports:
          - 80:80
          - 8080:8080
         - 443:443
       volumes:
         - ./conf.d:/etc/nginx/conf.d
         - ./html:/etc/nginx/html
         - ./logs:/etc/nginx/logs
         - ./nginx.conf:/etc/nginx/nginx.conf
   ```

   

2. 开启容器

   ```shell
   docker-compose up -d
   ```



##  2. 配置

### 2.1 日志

* 按时间分割日志

  ```json
  http {
      include       mime.types;
      default_type  application/octet-stream;
  
      log_format  main '$time_iso8601	-	-	$remote_addr	$http_host	$status	$request_time	$request_length	$body_bytes_sent	15d04347-be16-b9ab-0029-24e4b6645950	-	-	9689c3ea-5155-2df7-a719-e90d2dedeb2c 937ba755-116a-18e6-0735-312cba23b00c	-	-	$request_uri	$http_user_agent	-	sample=-&_UC_agent=-&device_id=-&-	-	-	-';
      access_log  logs/access.log  main;
      server {
          listen       80;
          server_name  localhost;
          if ($time_iso8601 ~ "^(\d{4})-(\d{2})-(\d{2})T(\d{2}):(\d{2})") {
             set $year $1;
             set $month $2;
             set $day $3;
             set $hour $4;
             set $minute $5;
          }
          access_log  logs/$year$month-$day-$hour-$minute.log  main;
          location / {
              root   html;
              index  index.html index.htm;
          }
  
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root   html;
          }
      }
  ```

  

## Finnal (坑)

> 1. 在docker中按时间分割日志无法生成日志

1. 通过查看docker日志可以看到是因为nginx权限不够导致日志无法生成。

   docker查看容器日志命令 `docker logs [options] 容器ID` 。具体情况看<b>docker</b>文档

2. nginx的目录都是root用户，另外集群tomcat也是属于root用户，而且root启动，查看nginx.conf

3. 将nginx.conf配置文件中开头的 `user nobody`  改成  `user root`  即可；