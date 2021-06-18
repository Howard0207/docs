# docker-compose部署

> 1. 下载镜像

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

> 2. 权限配置

```shell
sudo chmod +x /usr/local/bin/docker-compose
```

> 3. 查看版本

```shell
docker-compose --version
```

![image-20201229170746383](D:\SoftWare\Typora\docs\image-20201229170746383.png)

上图可见，docker-compose已经安装完成





# 部署镜像

> 1. Nginx

```yaml
version: '3.9'
services:
  nginx:
    container_name: 'nginx'
    restart: always
    image: daocloud.io/library/nginx:1.19.2
    environment:
      - TZ=Asia/Shanghai
    ports:
      - 80:80
      - 8080:8080
      - 443:443
    volumes:
      - ./logs:/etc/nginx/logs
      - ./html:/etc/nginx/html
      - ./conf.d:/etc/nginx/conf.d
      - ./nginx.conf:/etc/nginx/nginx.conf

```

> 2. MySql

```yaml
version: '3.9'
services:
  mysql:
    container_name: 'mysql'
    restart: always
    image: daocloud.io/library/mysql:8.0.2
    environment:
      MYSQL_ROOT_PASSWORD: root
      TZ: Asia/Shanghai
    ports:
      - 3306:3306
    privileged: true
    volumes:
      - ./data:/var/lib/mysql
      - ./log:/var/log/mysql
      - ./conf:/etc/mysql
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
```

> 3. Redis

```yaml
version: '3.9'
services:
  redis:
    container_name: 'redis'
    restart: always
    image: daocloud.io/library/mysql:6.06
    environment:
      TZ: Asia/Shanghai
    ports:
      - 6379:6379
    privileged: true
    volumes:
      - ./data:/data
      - ./redis.conf:/etc/redis/redis.conf
    command:
      redis-server /etc/redis/redis.conf --appendonly yes
```

Note：

```text
1. 从官网获取 **redis.conf** 配置文件
2. 修改默认配置文件
 * bind 127.0.0.1 # 注释掉这部分，这是限制redis只能本地访问
 * protected-mode no # 默认 yes， 开启保护模式， 限制为本地访问
 * daemonize no # 默认 no， 改为yes意为以守护进程方式启动，可后台运行， 除非kill进程（可选），==改为yes会是配置文件方式启动redis失败==
 * dir ./ # 输入本地redis数据库存放文件夹（可选）
 * appendonly yes # redis 持久化（可选）
```

