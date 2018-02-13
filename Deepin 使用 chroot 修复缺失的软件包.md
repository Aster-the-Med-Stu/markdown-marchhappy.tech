> 本文使用百度在线输入法写作而成，因为 Deepin 的 ISO 没有搜狗拼音……

**purge一时爽，系统火葬场（X**

我自己使用的台式机使用的为一套古老的 3A 平台（08年的速龙II X4 640，15年的 R7 360，以及970芯片组），但为了在同学的电脑流畅的使用 Deepin ，我使用了 Deepin 的显卡驱动管理器安装了大黄蜂方案的显卡驱动。早就在 LUG@USTC 听说过老黄的显卡驱动，只要内核升级就会随机 BOOM ，结果没想到这样的事情真的发生到了自己头上。

[caption id="attachment_392" align="alignnone" width="1280"]<img src="https://marchhappy.tech/wp-content/uploads/2018/02/photo_2018-02-09_08-42-49.jpg" alt="不显示 X 的 Deepin" width="1280" height="968" class="size-full wp-image-392" /> 不显示 X 的 Deepin[/caption]

切换至 tty 寻找错误原因， lightdm 的日志里面居然是空的，没有什么 Debug 信息，遂移步 [Deepin Telegram 国际频道](t.me/deepin) 求助，得知是老黄家的显卡驱动同新版本内核的兼容问题。我也不想动不动就搞一个大新闻……但是这一次罪魁祸首仅仅是因为下面这一行代码：

```
apt-get remove --purge nvidia-*
更新时间：2018年2月8日 (四) 16:28 (CST)
```
然后 ```purge``` 就尽责的噼里啪啦的删了一堆包，甚至有 ```xserver-xorg-input-all``` 。于是就有了卡在登录界面，光标闪烁，但是不响应鼠标键盘输入的奇观……

[caption id="attachment_392" align="alignnone" width="1280"]<img src="https://marchhappy.tech/wp-content/uploads/2018/01/ca9faf4b-8b84-4f97-bed7-c3edbef23278.jpeg" alt="卡在登录界面，输入什么都没反应" width="800" height="604" class="alignnone size-full wp-image-344" /> 卡在登录界面的 Deepin[/caption]

这是我第一次遇到误删包这样的问题。为了不浪费各位的时间，外加给可能的后来人提供参考，我把日志放到了最后。

# 进入 chroot 环境

掏出以前的安装U盘，将 ```grub``` 的 linux 启动参数由 ```liveCD-installer``` 改为了 ```liveCD``` ，进入了熟悉的 ```DDE```。

```
deepin@Deepin:~$ chroot
chroot: missing operand
Try 'chroot --help' for more information.
deepin@Deepin:~$ man chroot
deepin@Deepin:~$ chroot /media/deepin/a84dea11-736e-4614-b84e-645f69127902
chroot: cannot change root directory to '/media/deepin/a84dea11-736e-4614-b84e-645f69127902': Operation not permitted
deepin@Deepin:~$ sudo chroot /media/deepin/a84dea11-736e-4614-b84e-645f69127902
bash: /dev/null: Permission denied
bash: /dev/null: Permission denied
bash: /dev/null: Permission denied
bash: /dev/null: Permission denied
bash: /dev/null: Permission denied
bash: /dev/null: Permission denied
bash: /dev/null: Permission denied
root@Deepin:/# fish
Welcome to fish, the friendly interactive shell
<W> fish: An error occurred while redirecting file “/dev/null”
open: Permission denied
```
不知为何，chroot 以后提示挂载 /dev/null 失败。检查以后发现 /dev/null 下面神奇的冒出了文件，于是直接 G9 滥权删除。
```
root@Deepin:/# rm /dev/null
root@Deepin:/# mknod /dev/null c 1 3
root@Deepin:/# chmod 666 /dev/null
```
# chroot 后无法联网的解决

```
root@Deepin:/# apt install xserver-xorg-input-all
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  linux-image-4.9.0-deepin10-amd64-unsigned linux-image-4.9.0-deepin9-amd64-unsigned
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  xserver-xorg-input-libinput xserver-xorg-input-wacom
Suggested packages:
  xinput
The following NEW packages will be installed:
  xserver-xorg-input-all xserver-xorg-input-libinput xserver-xorg-input-wacom
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 184 kB of archives.
After this operation, 453 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Err:1 http://packages.deepin.com/deepin panda/main amd64 xserver-xorg-input-libinput amd64 0.23.0-2
  Temporary failure resolving 'packages.deepin.com'
Err:2 http://packages.deepin.com/deepin panda/main amd64 xserver-xorg-input-all amd64 1:7.7+19
  Temporary failure resolving 'packages.deepin.com'
Err:3 http://packages.deepin.com/deepin panda/main amd64 xserver-xorg-input-wacom amd64 0.34.0-1
  Temporary failure resolving 'packages.deepin.com'
E: Failed to fetch http://packages.deepin.com/deepin/pool/main/x/xserver-xorg-input-libinput/xserver-xorg-input-libinput_0.23.0-2_amd64.deb  Temporary failure resolving 'packages.deepin.com'
E: Failed to fetch http://packages.deepin.com/deepin/pool/main/x/xorg/xserver-xorg-input-all_7.7+19_amd64.deb  Temporary failure resolving 'packages.deepin.com'
E: Failed to fetch http://packages.deepin.com/deepin/pool/main/x/xf86-input-wacom/xserver-xorg-input-wacom_0.34.0-1_amd64.deb  Temporary failure resolving 'packages.deepin.com'
E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
root@Deepin:/# apt update
Err:1 http://linux.teamviewer.com/deb stable InRelease
  Temporary failure resolving 'linux.teamviewer.com'
Err:2 http://linux.teamviewer.com/deb preview InRelease
  Temporary failure resolving 'linux.teamviewer.com'
Err:3 http://packages.deepin.com/deepin panda InRelease
  Temporary failure resolving 'packages.deepin.com'
Err:4 https://deb.nodesource.com/node_8.x sid InRelease
  Could not resolve host: deb.nodesource.com
Reading package lists... Done
Building dependency tree       
Reading state information... Done
All packages are up to date.
W: Failed to fetch http://packages.deepin.com/deepin/dists/panda/InRelease  Temporary failure resolving 'packages.deepin.com'
W: Failed to fetch https://deb.nodesource.com/node_8.x/dists/sid/InRelease  Could not resolve host: deb.nodesource.com
W: Failed to fetch http://linux.teamviewer.com/deb/dists/stable/InRelease  Temporary failure resolving 'linux.teamviewer.com'
W: Failed to fetch http://linux.teamviewer.com/deb/dists/preview/InRelease  Temporary failure resolving 'linux.teamviewer.com'
W: Some index files failed to download. They have been ignored, or old ones used instead.
```
很奇特的连不上网络
```
root@Deepin:/# ping -c marchhappy.tech
ping: bad number of packets to transmit.
root@Deepin:/# ping -c 2 marchhappy.tech
ping: marchhappy.tech: Temporary failure in name resolution
root@Deepin:/# ping -c 2 marchhappy.tech
ping: marchhappy.tech: Temporary failure in name resolution
root@Deepin:/# dhcpcd
bash: dhcpcd: command not found
root@Deepin:/# mount -t proc none /proc
root@Deepin:/# service network restart
network: unrecognized service
root@Deepin:/# systemctl network restart
Unknown operation network.
root@Deepin:/# ping -c 2 marchhappy.tech
ping: marchhappy.tech: Temporary failure in name resolution
root@Deepin:/#
```
一直提示“暂时无法解析域名”，而且 chroot 环境下是没有办法用上systemctl，最后发现是 DNS 解析未设置（感谢 Telegram群组 [桌面Linux](https://t.me/LinuxDesktop) 某位同学指点～）
```
root@Deepin /# nano /etc/dnsmasq.conf
```
填入国科大的 DNS，测试，一切正常。

# 恢复误删除的软件包

## 乱删掉的软件包列表

从 ```/var/log/apt/history.log``` 弄出来的列表

```
Start-Date: 2018-01-30  20:20:07
Commandline: apt-get remove --purge nvidia-*
Requested-By: aster (1000)
Purge: xserver-xorg-input-all:amd64 (1:7.7+19), xserver-xorg-input-synaptics:amd64 (1.9.0-3deepin), xserver-xorg:amd64 (1:7.7+19), xserver-xorg-video-vesa:amd64 (1:2.3.4-1+b2), libglx0-glvnd-nvidia:amd64 (387.34-1deepin), libglx0-glvnd-nvidia:i386 (387.34-1deepin), nvidia-support:amd64 (20151021+4), xserver-xorg-video-amdgpu:amd64 (1.3.0-1), libgles-nvidia1:amd64 (387.34-1deepin), libgles-nvidia1:i386 (387.34-1deepin), libgles-nvidia2:amd64 (387.34-1deepin), libgles-nvidia2:i386 (387.34-1deepin), nvidia-kernel-common:amd64 (20151021+4), libnvidia-ml1:amd64 (387.34-1deepin), nvidia-vulkan-icd:amd64 (387.34-1deepin), nvidia-vulkan-icd:i386 (387.34-1deepin), nvidia-driver-libs-i386:i386 (387.34-1deepin), xserver-xorg-core:amd64 (2:1.19.3-2deepin), nvidia-egl-icd:amd64 (387.34-1deepin), nvidia-egl-icd:i386 (387.34-1deepin), update-glx:amd64 (0.7.2+deepin2), nvidia-driver:amd64 (387.34-1deepin), nvidia-modprobe:amd64 (375.26-1), bumblebee-nvidia:amd64 (3.2.1-13), xserver-xorg-video-fbdev:amd64 (1:0.4.4-1+b5), nvidia-vulkan-common:amd64 (387.34-1deepin), primus:amd64 (0~20150328-4), xserver-xorg-input-libinput:amd64 (0.23.0-2), nvidia-vdpau-driver:amd64 (387.34-1deepin), libgl1-nvidia-glvnd-glx:amd64 (387.34-1deepin), libgl1-nvidia-glvnd-glx:i386 (387.34-1deepin), libglx-nvidia0:amd64 (387.34-1deepin), libglx-nvidia0:i386 (387.34-1deepin), glx-alternative-nvidia:amd64 (0.7.2+deepin2), xserver-xorg-input-wacom:amd64 (0.34.0-1), nvidia-kernel-dkms:amd64 (387.34-1deepin), libegl-nvidia0:amd64 (387.34-1deepin), libegl-nvidia0:i386 (387.34-1deepin), nvidia-egl-common:amd64 (387.34-1deepin), libgles1-glvnd-nvidia:amd64 (387.34-1deepin), libgles1-glvnd-nvidia:i386 (387.34-1deepin), libnvidia-cfg1:amd64 (387.34-1deepin), libnvidia-cfg1:i386 (387.34-1deepin), nvidia-legacy-check:amd64 (387.34-1deepin), nvidia-egl-wayland-icd:amd64 (387.34-1deepin), nvidia-egl-wayland-icd:i386 (387.34-1deepin), xserver-xorg-video-intel:amd64 (2:2.99.917+git20161206-1+deepin), nvidia-kernel-support:amd64 (387.34-1deepin), glx-diversions:amd64 (0.7.2+deepin2), xserver-xorg-video-vmware:amd64 (1:13.2.1-1+b1), nvidia-driver-libs:amd64 (387.34-1deepin), nvidia-driver-libs:i386 (387.34-1deepin), nvidia-driver-bin:amd64 (387.34-1deepin), xserver-xorg-video-ati:amd64 (1:7.9.0-1), xorg:amd64 (1:7.7+19), nvidia-persistenced:amd64 (375.26-2), xserver-xorg-video-radeon:amd64 (1:7.9.0-1), xserver-xorg-video-nvidia:amd64 (387.34-1deepin), bumblebee:amd64 (3.2.1-13), nvidia-installer-cleanup:amd64 (20151021+4), libgl1-glvnd-nvidia-glx:amd64 (387.34-1deepin), libgl1-glvnd-nvidia-glx:i386 (387.34-1deepin), glx-alternative-mesa:amd64 (0.7.2+deepin2), nvidia-egl-wayland-common:amd64 (387.34-1deepin), libegl1-glvnd-nvidia:amd64 (387.34-1deepin), libegl1-glvnd-nvidia:i386 (387.34-1deepin), libgles2-glvnd-nvidia:amd64 (387.34-1deepin), libgles2-glvnd-nvidia:i386 (387.34-1deepin), nvidia-settings:amd64 (), nvidia-alternative:amd64 (387.34-1deepin)
End-Date: 2018-01-30  20:20:52
```

## 日志处理

 ```apt``` 安装的时候需要去掉括号以及括号内内容，并且用空格来划分不同的包。

用代替逗号选中括号内的内容（含括号）的正则表达式：```(\([^\)]+\))```，随后再用空格替换逗号，最后剔除所有含有 ```Nvidia``` 关键字的包即可。

# 总结

在 Deepin 之前使用的操作系统是 openSUSE leap 42 ，由于 ```BtrFS``` 的快照特性，所以出错的时候也懒得寻找问题所在，而是直接回滚快照。而这一次使用 Deepin 则是让我补回了早就应该掌握的 ```chroot``` 技能，顺带着锻炼了一下独立判断问题的能力。

# 参考资料
[1] [记一次错误卸载软件包导致Linux系统崩溃的修复解决过程](https://segmentfault.com/a/1190000000749515)

[2] [Ubuntu 的系统恢复](http://drgan.net/2011/10796/)

[3] [回车和换行](http://www.ruanyifeng.com/blog/2006/04/post_213.html)

[4] [利用chroot修复Linux系统问题-深度科技论坛|深度操作系统正在为全世界的电脑提供强劲动力！](https://bbs.deepin.org/forum.php?mod=viewthread&tid=21544)
