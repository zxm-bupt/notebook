# 1. Ubuntu/Debian/RaspberryPi

debian系的docker可以分如下步骤安装

## 1.1 删除旧版本

```shell
for pkg in docker.io docker-doc docker-compose podman-docker containerd containerd.io runc; do sudo apt-get remove $pkg; done
```

## 1.2 配置apt

```shell
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc



# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```

> `/etc/apt/keyrings/`：这是 APT 2.4 版本后推荐用于存储非软件包管理的密钥的位置。
> 
> `apt-key` 命令将公钥添加到 `/etc/apt/trusted.gpg` 或 `/etc/apt/trusted.gpg.d/`，这意味着这些密钥会被 APT **全局信任**。如果其中一个被攻破，攻击者就可以用它来签署任何仓库的软件包，对系统造成威胁。
> 
> keyrings可以用来给特定的仓库使用密钥。
> 
> 例如上面的signed-by=/etc/apt/keyrings/docker.asc

## 1.3 安装

```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

# 2. Fedora/centos

## 2.1 删除旧版本

```shell
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

## 2.2 配置DNF

```shell
# Install the dnf-plugins-core package (which provides the commands to manage your DNF repositories)
sudo dnf -y install dnf-plugins-core
sudo dnf-3 config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
```

## 2.3 安装Docker

```shell
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

# 3. 配置docker-daemon的代理

## 3.1 docker daemon是什么

Docker daemon (通常称为 `dockerd` 进程) 是 Docker 平台的核心组件。它是一个**持久运行的后台服务**，在 Docker 主机上负责管理和协调 Docker 的所有操作。你可以把它理解为 Docker 的“大脑”或“引擎”。

## 3.2 daemon.json

通常情况下`/etc/docker/daemon.json`是daemon的配置的常见位置。

## 3.3 配置代理

可在daemon.json中配置如下内容来配置代理

```shell
{
  "proxies": {
    "http-proxy": "http://proxy.example.com:3128",
    "https-proxy": "https://proxy.example.com:3129",
    "no-proxy": "*.test.example.com,.example.org,127.0.0.0/8"
  }
}
```

配置完成后，需要重启dockerd

```shell
sudo systemctl restart docker
```

为什么要配置docker daemon的代理，是因为export的方式，可以为docker cli配置代理，但是daemon并没有配置代理。daemon是开机时就运行的，并不在ssh链接到的session中。而export只设置了当前会话的代理。

# 4. 配置docker的用户权限

docker安装后，默认用户需要使用sudo权限，这样很麻烦。所以我们参照下面的命令来不需要使用sudo提权。

```shell
sudo groupadd docker
sudo usermod -aG docker $USER
sudo systemctl restart docker
# 重新登录
```

虽然 Docker 守护进程以 root 身份运行，但它将与客户端通信的 Unix socket 的访问权限委托给了 docker 用户组。通过将你的用户添加到这个 docker 组，你就获得了访问这个 socket 的权限，从而能够与守护进程进行通信，而无需每次都使用 sudo 来提升你的用户本身的权限。
