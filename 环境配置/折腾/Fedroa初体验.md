忙里偷闲，安装一下Fedroa尝试一下，第一次尝试RedHat系的linux发行版。做一个简单的记录。

本次安装的Fedora-cosmic版本，这个版本带有cosmic桌面，虽然我不喜欢rust神教，但是这个rust编写的，看起来和gnome很像的，带有平铺WM的桌面，我还是很喜欢的。当然因为还在Alpha阶段，所以这个桌面还有很多bug(说明Rust写的软件，一样不能避免bug)。

安装完成的第一步是更新系统，对于所有的操作系统，我觉得都需要先更新在进行之后的操作，以避免出现版本和依赖问题。Fedora的更新命令非常简单

```shell
sudo dnf update
```

这个命令比apt还要简单，而且不得不夸一句，dnf的TUI真比apt要好看的多，希望apt5能改进一下，不要再有超级牛力了。

更新完成系统后，第一件事，安装happynet，然后切换到windows terminal，windows terminal是最好的Linux DE。

ssh登录，然后，connect refused。嗯？难道是ssh-server默认没启动，systemctl查看一下，果然没启动。ok，执行

```shell
systemctl start sshd
systemctl enable sshd
```

登录，出现密码输入，ok。

下面配置免密，把我的公钥copy到`~/.ssh/`下面，目录不存在，嗯？好吧，我新建一个，然后把`authorized_keys`也创建了。scp公钥，写入`authorized_keys`，登录，可以不用在输入密码了。

盒盖，ssh断了，好吧，还要配置盒盖不休眠。

再cosmic setting中找一下，找到了`power and battery -> automic suspend when plugged in`，修改成never。重启，不生效。好吧，果然是alpha，忍了。

搜一下配置文件的解决方案，systemd应该提供了这种服务。

果然搜到了有`/etc/systemd/logind.conf`，但是我的系统没有这个文件，算了Gemini一下。

Gemini告诉我，我的OS版本太新了，现在已经使用`/etc/systemd/logind.conf.d/`目录下的config了，但是我这个目录也没有，算了创建一个。然后写一个新的override.conf。

```shell
[Login]
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

重启systemd-logind service，cosmic崩溃了，不过问题不大，重启桌面，解决问题，不会自动suspend了。

小节，Fedora是真不喜欢提供额外的配置，想用什么，麻烦自己查询资料，不过确实比Ubuntu干净。没有乱七八糟的配置文件，dnf比apt好用的多。Cosmic目前不太适合当主力DE，感觉还有一些不稳定，不过平铺布局真的舒服。




