# 1. 安装TSSH

```shell
go install github.com/trzsz/trzsz-ssh/cmd/tssh@latest
```

# 2. 导入环境变量

在环境变量中将GOPATH导入环境变量

![Imgur](https://i.imgur.com/yfAwWuH.png)

# 3. 安装trzsz

在服务器上安装

```shell
go install github.com/trzsz/trzsz-go/cmd/...@latest
```

在服务器上可以使用`trz`打开文件传输将本机文件传入服务器，使用`tsz file`将文件从服务器传入本地。

# 4. 用户手册

[Trzsz-ssh ( tssh ) 中文文档 | trzsz](https://trzsz.github.io/cn/ssh)
