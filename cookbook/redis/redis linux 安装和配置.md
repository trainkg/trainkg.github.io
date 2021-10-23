## [redis linux 安装和配置](https://redis.io/download)

### 安装

```bash
$ wget https://download.redis.io/releases/redis-6.2.5.tar.gz
$ tar xzf redis-6.2.5.tar.gz
$ cd redis-6.2.5
$ make MALLOC=libc
```

### 配置

修改配置文件 redis.conf

```bash
# 设置访问密码 , 远程模式无密码不可以访问
CONFIG set requirepass 123456
# OR 修改配置文件redis.conf
requirepass 123456
```

### 运行

```bash
$ src/redis-server ./redis.conf
```

### 连接

```bash
$ src/redis-cli -a <password>
```

### 关闭

```
SHUTDOWN [NOSAVE|SAVE]
```

### 配置信息查看

```bash
# 查看端口
config get port

# 查看IP bind
config get bind
```

