#### 1、使用dnf config-manager实用程序添加Docker存储库

```
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
```

#### 2、找到Docker CE的可安装版本

```
dnf list docker-ce --showduplicates | sort -r
```

#### 3、必要时安装containerd.io

```
dnf install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
```

#### 4、安装docker-ce

```
dnf install docker-ce-3:19.03.5-3.el7
```

#### 5、参考

- https://www.a5idc.net/helpview_591.html
- https://blog.csdn.net/yucaifu1989/article/details/103111317

