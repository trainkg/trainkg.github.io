## FTP

> [FTP 下载](https://filezilla-project.org/download.php?type=client#close) 

### 1.1 **安

### 装vsftpd服务器**

```bash
# 安装vsftpd服务器
sudo apt install vsftpd
# 安装完成后查看服务器版本
vsftpd -version
```

### 1.2 **新增ftp用户**

 vsftp服务器安装后，系统有一个默认的ftp帐户，并且默许本地用户登录ftp服务。
 但是用ftp和管理员的账号去登录ftp服务有一定安全隐患，所以新建一个专门的ftp账号

1. 首先修改vsftpd服务器的相关配置，配置文件路径 ***/etc/vsftpd.conf\***

```objectivec
# 禁止匿名登录
anonymous_enable=NO
# 允许本地用户登录
local_enable=YES
# 允许ftp写入数据
write_enable=YES
# 限制用户只能操作自己的HOME目录
chroot_local_user=YES
```

1. 新增一个ftp账号 ***ftpuser\***

```bash
# -m 同时创建用户家目录/home/ftpuser
# -s 禁止用户通过shell登录系统
sudo useradd -m -s /usr/sbin/nologin ftpuser
```

1. 修改HOME目录权限，并规划目录结构
    当把用户限制在HOME目录下操作时，vsftpd服务器默认是禁止用户修改home目录下的文件的，使用过虚拟主机的朋友应该都有类似经历。所以我们需要提前规划好HOME目录的目录结构和权限，并且禁止HOME目录写权限

```bash
# 在ftpuser家目录下创建一个具有读写权限的文件夹，并修改owner和group
sudo mkdir /home/ftpuser/share
sudo chown ftpuser /home/ftpuser/share
sudo chgrp ftpuser /home/ftpuser/share
# 创建其它需要的目录文件
...
# 禁止ftpuser用户HOME目录的写权限，否则服务器会拒绝用户登录
sudo chmod -w /home/ftpuser
```

1. 更新/etc/shells的配置
    最后，因为ftpuser是禁止用shell登录系统的，所以创建用户时给它指定了一个非常规的终端 ***/usr/sbin/nologin\***，但是因为vsftpd服务器会检查登录用户的shell配置，如果shell配置不对也无法登录服务器，所以要告诉系统***nologin\***是一个合法的shell程序

```bash
# 在/etc/shells文件追加nologin为合法的shell程序
/usr/bin/nologin
```

1. 重新启动vsftpd服务器

```bash
sudo service vsftpd restart
```

## SSH

```bash
# 更新源列表
sudo apt-get update

# 安装openssh-server
sudo apt-get install openssh-server

# 启动服务
sudo service ssh start

# 查看ssh服务是否启动
sudo ps -e | grep ssh
```

