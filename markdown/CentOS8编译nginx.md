### CentOS8编译nginx
#### 1. 下载源码

```
[root@localhost ~]# wget http://nginx.org/download/nginx-1.16.1.tar.gz
```

#### 2. 解压

```
[root@localhost ~]# tar -zxvf nginx-1.16.1.tar.gz
```

#### 3. 进入解压文件夹、配置：configure

```
[root@localhost ~]# cd nginx-1.16.1
[root@localhost ~]# ./configure --prefix=~/gnehcnij/nginx
```

**如果报错：**

```
./configure: error: the HTTP rewrite module requires the PCRE library.
```

**则安装：**

```
[root@localhost ~]# yum -y install pcre-devel openssl openssl-devel
```

#### 4. 编译：make && 安装：make install

```
[root@localhost ~]# make
[root@localhost ~]# make install
```

