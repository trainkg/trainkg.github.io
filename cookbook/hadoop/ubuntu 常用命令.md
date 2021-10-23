## ubuntu

### keyward

```shell

~ 
当前目录
```

### base command

```bash
ls
cd
clear
pwd 
whoami #查看当前登陆用户
sudo passwd root #设置用户密码
ifconfig 
```

### File command

```bash
mkdir 
rm 
cp 
mv
cat   输出文本内容
touch 创建新的文件
touch 1.txt
echo 
echo hello >> 1.txt
```

### Edit command 

```bash
nano  ctro + o, ctrl + X (ubuntu 内置文本编辑器)
echo 
cat
redirect: >>(追加) & > (覆盖)
more
man ls | more ( '|' 管道指令，将前一个命令的输出，作为第二个命令的输入 )
head
man ls | head -10 (前10行)
tail
man ls | tail -10 (后10行)
```

### Systems Comment

```bash
# 查看OS版本
cat /proc/version
sb_release -a
# 进行软件源列表的更新。
sudo apt-get update 
# 进行更新包的安装。
sudo apt-get upgrade
#dist-upgrade来更新依赖关系的变化，添加包和删除包。
sudo apt-get dist-upgrade
# 重启
sudo reboot 

# 查找
find /usr/local | grep xxx
uname -a
查看操作系统信息描述
file xxx.so
查看文件描述
tar -xvzf
解压文件
gzip
压缩,不保留原文件
gunzip
原地压缩
```

###  message & jobs
? mnt 目录是挂载目录

```bash
sudo mount  挂载设定
sudo umount 解除挂载
ln -s /exist_file link_name
jobs
kill %n
ps -af 
ps -af | grep java
cut -c num1-num2
xxx --help
man xxx
help
```



