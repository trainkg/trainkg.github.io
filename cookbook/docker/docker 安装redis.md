## [docker 安装redis](https://hub.docker.com/_/redis)

> [redis 官网](https://redis.io/topics/persistence)
>
> [redis 命令](http://doc.redisfans.com/) [redis 命令2](https://www.runoob.com/docker/docker-command-manual.html)

```shell
# 启动redis 实例
sudo docker run --name some-redis -d redis

# 连接
sudo docker exec -it some-redis redis-cli
sudo docker run -it --network some-network --rm redis redis-cli -h some-redis
```

常用可选参数说明：

```bash
-i 表示以交互模式运行容器。
-t 表示容器启动后会进入其命令行。加入这两个参数后，容器创建就能登录进去。即分配一个伪终端。
--name 为创建的容器命名。
-v 表示目录映射关系，即宿主机目录:容器中目录。注意:最好做目录映射，在宿主机上做修改，然后共享到容器上。 
-d 会创建一个守护式容器在后台运行(这样创建容器后不会自动登录容器)。 
-p 表示端口映射，即宿主机端口:容器中端口。
--network=host 表示将主机的网络环境映射到容器中，使容器的网络与主机相同。
```

* 创建容器

```bash
docker run [option] 镜像名 [向启动容器中传入的命令]
```

* 交互式容器

```
docker run -it --name=ubuntu1 镜像名 /bin/bash
```

* 守护式容器

```
docker run -dit --name=ubuntu1 镜像名 /bin/bash
```

* 进入到容器内部

```
docker exec -it ubuntu2 /bin/bash
```

* 将容器制作成镜像

```
docker commit 容器名 镜像名
```

* 镜像打包备份

```
docker save -o 保存的文件名 镜像名
```

* 镜像解压

```
docker load -i 文件路径/备份文件
```

