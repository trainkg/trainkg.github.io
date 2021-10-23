## ubuntu 服务器配置

官方下载地址（不推荐）

[https://www.ubuntu.com/download](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.ubuntu.com%2Fdownload)

中国官网

[https://cn.ubuntu.com/](https://links.jianshu.com/go?to=https%3A%2F%2Fcn.ubuntu.com%2F)

中科大源

[http://mirrors.ustc.edu.cn/ubuntu-releases/16.04/](https://links.jianshu.com/go?to=http%3A%2F%2Fmirrors.ustc.edu.cn%2Fubuntu-releases%2F16.04%2F)

阿里云开源镜像站

[http://mirrors.aliyun.com/ubuntu-releases/16.04/](https://links.jianshu.com/go?to=http%3A%2F%2Fmirrors.aliyun.com%2Fubuntu-releases%2F16.04%2F)

兰州大学开源镜像站

[http://mirror.lzu.edu.cn/ubuntu-releases/16.04/](https://links.jianshu.com/go?to=http%3A%2F%2Fmirror.lzu.edu.cn%2Fubuntu-releases%2F16.04%2F)

北京理工大学开源

[http://mirror.bit.edu.cn/ubuntu-releases/16.04/](https://links.jianshu.com/go?to=http%3A%2F%2Fmirror.bit.edu.cn%2Fubuntu-releases%2F16.04%2F)

浙江大学

[http://mirrors.zju.edu.cn/ubuntu-releases/16.04/](https://links.jianshu.com/go?to=http%3A%2F%2Fmirrors.zju.edu.cn%2Fubuntu-releases%2F16.04%2F)（推荐）

不知名镜像网站

[http://mirror.pnl.gov/releases/xenial/](https://links.jianshu.com/go?to=http%3A%2F%2Fmirror.pnl.gov%2Freleases%2Fxenial%2F)

各个版本下载网址：

[http://mirrors.melbourne.co.uk/ubuntu-releases/](https://links.jianshu.com/go?to=http%3A%2F%2Fmirrors.melbourne.co.uk%2Fubuntu-releases%2F)

## 配置镜像

```bash
# 查看镜像配置文件
cd /etc/apt
vim /etc/apt/sources.list

# 修改替换文件之后
sudo apt-get update
sudo apt-get upgrade
```

