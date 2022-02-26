#	Docker介绍

> 什么是Docker

* Docker是一个开源应用容器引擎。
* 基于Go语言，遵从Apache 2.0协议开源
* 容器是完全使用沙箱机制；互相之间没有接口
* 容器性能开销极低

-----



> Docker的应用场景

* Web应用的自动化打包和发布
* 自动化测试和持续集成、发布

---



> Docker的优点

1. 快速，一致性交付

2. 响应式部署和扩展

3. 在统一硬件上运行更多工作负载



```shell
docker run -d --hostname 192.168.206.138 \
	-p 10110:443 -p 10111:80 -p 10112:22 \
	--name gitlab --restart always \
	--volume /home/gitlab/config:/etc/gitlab \
	--volume /home/gitlab/logs:/var/log/gitlab \
	--volume /home/gitlab/data:/var/opt/gitlab \ gitlabImageId
	--privileged=true
```

