# 1. 安装依赖

```shell
# 清理之前版本
sudo apt purge ^librtlsdr
sudo rm -rvf /usr/lib/librtlsdr* 
sudo rm -rvf /usr/include/rtl-sdr* 
sudo rm -rvf /usr/local/lib/librtlsdr* 
sudo rm -rvf /usr/local/include/rtl-sdr* 
sudo rm -rvf /usr/local/include/rtl_* 
sudo rm -rvf /usr/local/bin/rtl_*

# 安装依赖
sudo dnf install git cmake pkg-config build-essential libusb1-devel
```

这里出现了第一个和ubuntu不一样的地方，fedora如果要安装libusb需要使用包名`libusb1-devel`

# 2. 下载源码

```shell
git clone https://github.com/rtlsdrblog/rtl-sdr-blog
```

# 3. 编译

```shell
cd rtl-sdr-blog/
mkdir build
cd build
cmake ../ -DINSTALL_UDEV_RULES=ON
make
sudo make install
sudo cp ../rtl-sdr.rules /etc/udev/rules.d/
sudo ldconfig
```

这里出现第二个不一样的地方。

首先我们补充一下动态库的寻找规则

- **1.** `LD_LIBRARY_PATH` 环境变量：

  `ldd` 首先会查找 `LD_LIBRARY_PATH` 环境变量中指定的目录。这个环境变量可以用来临时指定动态库的搜索路径。

- **2.** `/etc/ld.so.conf` 文件：

  如果 `LD_LIBRARY_PATH` 没有设置或者没有找到需要的库，`ldd` 就会查找 `/etc/ld.so.conf` 文件中定义的目录。这个文件是系统级别的动态库搜索路径配置文件。

- **3.** **默认路径：**

  最后，`ldd` 还会查找默认的动态库目录，包括 `/lib` 和 `/usr/lib`。


而`ldconfig`是一个用于管理动态链接库的命令，其主要作用是更新动态链接库的缓存，使得系统能够找到并使用新的或更新的共享库。它会搜索指定的目录（包括/lib、/usr/lib 和/etc/ld.so.conf 中指定的目录）下的共享库文件，并创建或更新 `/etc/ld.so.cache` 文件，这个文件保存了已排序的动态库列表，供程序在运行时查找使用。

与ubuntu不同的是，federo的`/etc/ld.so.conf.d/`目录下没有任何配置，而ubuntu下有`libc.conf`将`/usr/local/lib`加入到ldd的搜索目录中，而rtl-sdr的动态库安装在了`/usr/local/lib`。

# 参考

1.  [quickGuide](https://www.rtl-sdr.com/rtl-sdr-quick-start-guide/)

