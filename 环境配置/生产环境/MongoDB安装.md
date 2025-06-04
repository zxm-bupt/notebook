# 1. 前言

我个人十分不喜欢MongoDB，一个不满足ACID的数据库，是非常可笑的事情。即使它后来满足了ACID，我也不认为这种行为值得鼓励，不然任何产品都可以在发布之初就画饼，然后5个大版本更新后，才实现端上了一个饼状食物。

但是free5GC使用MongoDB来存储用户上下文，所以只能忍着恶心来安装MongoDB了。

# 2. Ubuntu/Debian

Ubuntu/Debian系，官网提供了详细的教程。

MongoDB8.0 支持Debian Bookworm、Ubuntu 24.04、Ubuntu22.04、Ubuntu20.04的x64架构和arm64架构。

## 2.1 导入公钥

```shell
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg \
   --dearmor
```

## 2.2 创建列表文件

Debian

```shell
echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] http://repo.mongodb.org/apt/debian bookworm/mongodb-org/8.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```

Ubuntu 24.04

```shell
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```

Ubuntu 22.04

```shell
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```

Ubuntu 20.04 

```shell
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```

## 2.3 更新仓库

```shell
sudo apt update
```

## 2.4 安装MongoDB

```shell
sudo apt-get install -y mongodb-org
```

# 3. Fedora

Fedora可以参照官网的RHEL的教程

## 3.1 导入仓库

```shell
sudo vim /etc/yum.repos.d/mongodb-org-8.0.repo
```

```shell
[mongodb-org-8.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/9/mongodb-org/8.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://pgp.mongodb.com/server-8.0.asc
```

## 3.2 安装

```shell
sudo yum install -y mongodb-org
```

## 3.3 启动服务

```shell
sudo systemctl start mongod
```

## 3.4 修复Error

此时使用`mongo`会遇到如下问题

```shell
mongosh: OpenSSL configuration error: 00899523A67F0000:error:030000A9:digital envelope routines:alg_module_init:unknown option:../deps/openssl/openssl/crypto/evp/evp_cnf.c:61:name=rh-allow-sha1-signatures, value=yes
```

这是因为安装的版本和要求的OpenSSL版本不匹配。

使用如下方式修复

```shell
sudo systemctl stop mongod

sudo rpm -e mongodb-org mongodb-mongosh

sudo dnf install -y mongodb-org mongodb-mongosh-shared-openssl3

sudo systemctl start mongod
```


