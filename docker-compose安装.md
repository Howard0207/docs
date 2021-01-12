# docker-compose安装

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

