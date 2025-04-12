鉴于很多地方都要安装Docker，每次都要取搜，就把安装过程放到这里。

```shell
# 1.添加新的HTTPS源
sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
# 2. 导入Docker源仓库的 GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# 3. Docker APT 软件源添加到系统
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# 4. 查看可用的docker版本
apt list -a docker-ce
# 5. 安装最新版本Docker
sudo apt install docker-ce docker-ce-cli containerd.io
# 6. 查看docker状态
sudo systemctl status docker
# 7. 将当前用户加入到docker组中
sudo usermod -aG docker $USER
# 8. 验证Docker安装情况
docker run hello-world
```
