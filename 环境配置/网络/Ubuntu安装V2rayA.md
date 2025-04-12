# 1. 添加公钥

```shell
wget -qO - https://apt.v2raya.org/key/public-key.asc | sudo tee /etc/apt/keyrings/v2raya.asc
```

# 2. 添加v2raya软件源

```shell
echo "deb [signed-by=/etc/apt/keyrings/v2raya.asc] https://apt.v2raya.org/ v2raya main" | sudo tee /etc/apt/sources.list.d/v2raya.list
sudo apt update
```

# 3. 安装V2raya

```shell
sudo apt install v2raya v2ray ## 也可以使用 xray 包
```

# 4. 启动v2raya服务

```shell
sudo systemctl start v2ray.service
sudo systemctl start v2raya.service
sudo systemctl enable v2raya.service
sudo systemctl enable v2ray.service
```
