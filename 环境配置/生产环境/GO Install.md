```shell
wget https://golang.google.cn/dl/go1.22.6.linux-arm64.tar.gz 
sudo tar -C /usr/local -zxvf go1.22.6.linux-arm64.tar.gz 
mkdir -p ~/go/{bin,pkg,src} 
# The following assume that your shell is bash: 
echo 'export GOPATH=$HOME/go' >> ~/.bashrc 
echo 'export GOROOT=/usr/local/go' >> ~/.bashrc 
echo 'export PATH=$PATH:$GOPATH/bin:$GOROOT/bin' >> ~/.bashrc 
echo 'export GO111MODULE=auto' >> ~/.bashrc 
source ~/.bashrc
```


