# CentOS7 下安装 Python3

> Python 版本：3.10.0
>
> 环境介绍：操作系统：CentOS 7.5，已存在 Python2.7 环境，要求 2.7 与 3.10 共存

## 上传版本包

获取 `Python-3.10.0.tgz` 、`openssl-1.1.1k.tar.gz` 安装包，并上传到 `/home/package` 用户下解压

```shell
tar zxvf Python-3.10.0.tgz
tar zxvf openssl-1.1.1k.tar.gz
```



## 安装依赖环境

```shell
yum install -y openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel zlib-devel libffi-devel
```



## 安装 OpenSSL 依赖环境

Python 3.10 需要 1.1.1 及以上版本的 OpenSSL 版本，默认 CentOS 7.5 的 OpenSSL 版本为 1.0.1，需要手动安装升级。如果不确定自己环境的 OpenSSL 版本，可以执行命令查询

```shell
openssl version
```

编译安装

```shell
cd openssl-1.1.1k/
./config --prefix=/usr/local/openssl
make
make install
```

替换默认 openssl 环境

```shell
mv /usr/bin/openssl /usr/bin/openssl_bak
mv /usr/include/openssl/ /usr/include/openssl_bak
ln -s /usr/local/openssl/include/openssl/ /usr/include/openssl
ln -s /usr/local/openssl/lib/libssl.so.1.1 /usr/local/lib64/libssl.so
ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
ldconfig -v
```

最后验证 OpenSSL 版本

```shell
openssl version
```



## 安装 Python

```
cd Python-3.10.0
./configure --prefix=/usr/local/ --with-openssl=/usr/local/openssl
make
make install
```

验证版本

```shell
python3 --version
whereis python3
```

> 由于需要 python2 和 3 共存，因此不需要对 python 命令的默认软链接进行修改。



