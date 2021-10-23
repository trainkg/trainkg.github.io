## 系统文件结构

```bash
/
# 根目录
/bin
# 二进制文件目录 用户使用
/sbin
# 二进制文件目录（超级用户使用的软件）
/boot
# 引导文件目录
/etc
# 配置文件目录
/mnt  -- mount
# 挂载目录
/home
#用户主目录  
/dev
# 设备
/usr
# 系统软件，类似windows的system32.  
```

##  文件描述

```bash
d	:directory
-	:file
  b	:block
  l	:link 
  符号连接,相当于window的快捷方式
  c :字符文件
```

## 权限分组

```
User(u)
Group(g)
Other(0)
r	:read
w	:write
x	:execute
-	:none
```

**10位表示法**
1(文件类型) + 3(user) + 3(group) + 3(others)

3位权限顺序
-----
R W X

## 权限操作

```bash
chmod
chmod o+x 1.txt
```

为other 添加1.txt执行权限  

```bash
 chmod u+w xxx
 chmod u+rw xxx
 chmod u+rwx xxx
 chmod ug+rwx xxx
 chmod ugo+x
 chmod a+rwx

 chmod u-w xxx
 chmod u-rw xxx
 chmod 644 xxx
 chmod 777 xxx
```



