# JNU-campus-network-wireless-router
成本三十元的小米路由器实现锐捷校园网的多设备共享

## 前提摘要

<div align=center><img src="https://github.com/JNUlth/JNU-campus-network-wireless-router/blob/main/show/dog.jpg" width = "50%" alt="" ></div>
有段时间宿舍校园网不当人，懂得都懂。
花了一个星期研究各种帖子教程后开启路由器计划。

选了案例比较多的小米mini路由器。
## 刷开发版系统并开启SSH功能
路由器在pdd买的，二手，三十块，附赠一根网线。

路由器插上电，电脑用网线连接路由器的wan口，连上路由器的wifi;
打开浏览器会自动进入路由器后台，如果没有就浏览器登进IP 192.168.31.1。
进入设置界面，系统升级→手动升级→选"miwifi_r1cm_firmware_5b25f_2.17.30.bin"。
升级会自动重启，结束后登进去看看是否成功。

用牙签长戳路由器的复位孔进行初始化，手机下载小米WiFi,登录小米账号绑定路由器（不绑定账号无法开启SSH远程登录）。
在电脑进入miwifi.com →开发→开启SSH工具（如果进不去大概率是网络问题）
将下载好的ssh工具文件改名为"miwifi_ssh.bin"，拷入U盘的根目录;将路由器断电，插上U盘，用牙签戳住复位孔，再插上电，五秒后松开，路由器将自动重启。
## 刷Breed恢复控制台
刷breed目的是刷系统出了问题可以一键恢复避免永久变砖，也可以直接跳到刷系统；

下载SSH登录软件，当时用的是putty，推荐用Xshell;
下载文件传输工具winscp；

打开Xshell,快捷键Alt+N新建会话，协议选SSH;主机：192.168.31.1；端口：22；
然后就进入了命令行窗口，输入手机绑定时设置的管理员用户名和密码；
弹出R U OK的彩蛋即登录成功。

备份（用于刷写breed失败的恢复）

```
#下面就是用dd命令备份文件，copy到命令行即可

cd /tmp		#进入tmp路径
mkdir rom	#新建文件夹rom
dd if=/dev/mtd0 of=/tmp/rom/ALL.bin	#系统文件备份到rom文件夹
dd if=/dev/mtd1 of=/tmp/rom/Bootloader.bin
dd if=/dev/mtd2 of=/tmp/rom/Config.bin
dd if=/dev/mtd3 of=/tmp/rom/Factory.bin
dd if=/dev/mtd4 of=/tmp/rom/OS1.bin
dd if=/dev/mtd5 of=/tmp/rom/rootfs.bin
dd if=/dev/mtd6 of=/tmp/rom/OS2.bin
dd if=/dev/mtd7 of=/tmp/rom/overlay.bin
dd if=/dev/mtd8 of=/tmp/rom/crash.bin
dd if=/dev/mtd9 of=/tmp/rom/reserved.bin
dd if=/dev/mtd10 of=/tmp/rom/Bdata.bin
cd /tmp/rom
ls			#看看文件有没有漏了
```
备份完打开WinSCP，协议选SCP，然后其余信息同上（Xshell也能传文件，但WinSCP更直观方便）。找到刚刚备份的rom文件夹，拖到电脑里存着。

当需要恢复备份的时候把rom拖回/tmp。
```
#如果不需要恢复备份勿进行这一步

mtd write /tmp/rom/Bootloader.bin Bootloader
mtd write /tmp/rom/Config.bin Config
mtd write /tmp/rom/Factory.bin Factory
mtd write /tmp/rom/OS1.bin OS1
mtd write /tmp/rom/rootfs.bin rootfs
mtd write /tmp/rom/OS2.bin OS2
mtd write /tmp/rom/overlay.bin overlay
mtd write /tmp/rom/crash.bin crash
mtd write /tmp/rom/reserved.bin reserved
mtd write /tmp/rom/Bdata.bin Bdata
```
备份完就是刷breed了。
把breed固件文件"breed-mt7620-xiaomi-mini.bin"传到/tmp
```
#然后回到Xshell

cd /tmp
mtd -r write breed-mt7620-xiaomi-mini.bin Bootloader
```
等待路由器自动重启后断电，戳住复位键插电等待5秒松开。
电脑用网线连接路由器wan口，浏览器进入192.168.1.1

一切顺利的话得如下界面

<div align=center><img src="https://github.com/JNUlth/JNU-campus-network-wireless-router/blob/main/show/0.png" width = "80%" alt="" ></div>

刷好了breed,各种系统可以随便刷了，搞出问题长戳复位键既可回到恢复控制台。

## 刷Padavan系统
固件更新→选择固件"RT-AC54U-GPIO-30-xiaomimini-128M_3.4.3.9-099"→上传更新（点个自动重启，其他默认）
浏览器输入新系统的IP:192.168.123.1
初始用户名和密码均为 admin

一切顺利的话得如下界面

<div align=center><img src="https://github.com/JNUlth/JNU-campus-network-wireless-router/blob/main/show/1.png" width = "80%" alt="" ></div>

## 网络配置
熟悉一下界面，把一些自动开启的无用功能关闭
系统默认开启2.4G和5G两个频段WiFi，自行修改名称密码

外部网络→IPoE:动态IP→自动获取DNS
配置扩展环境→锐捷认证→启用 MentoHUST

<div align=center><img src="https://github.com/JNUlth/JNU-campus-network-wireless-router/blob/main/show/2.png" width = "80%" alt="" ></div>

学校使用的客户端版本较新，所以需要抓包
在win10环境模拟，使用抓包程序生成数据文件jnu3.mpf
（如果其他校区无法使用可能需要自己重新抓包）
把jnu3.mpf文件传上路径/etc/storage/

然后便是全文最关键的地方
<div align=center><img src="https://github.com/JNUlth/JNU-campus-network-wireless-router/blob/main/show/3.png" width = "80%" alt="" ></div>

**登录室友的双通道账号**（单通道不行）

路由器wan口接宿舍网口，lan口连电脑
登录后台点击重新连接
一切顺利的话得如下界面

<div align=center><img src="https://github.com/JNUlth/JNU-campus-network-wireless-router/blob/main/show/4.jpg" width = "80%" alt="" ></div>

测试一下网速，可以跑满路由器的百兆，zbc
<div align=center><img src="https://github.com/JNUlth/JNU-campus-network-wireless-router/blob/main/show/5.png" width = "50%" alt="" ></div>

## 后记及文件下载
成功实现爬上床再关台灯，幸福感大幅提升。

平均每隔大概半个月（间隔时长时短）账号会被封禁，“由于检测到多次共享IP网络，进入黑名单120分钟”，每次都是两小时，时间过了登录后台点击重新连接即可。曾尝试写自动重连的脚本，跑不动，遂放弃，“也不是不能用”。对于封禁问题尝试过伪造硬件地址，改心跳检测间隙等等操作，均无效，如果有朋友知道它的检测封禁原理欢迎发邮件交流。

Padavan系统还有很多有趣的功能可以探索，可以配置连接外网、屏蔽广告、搭建个人云盘等，似乎还有一键LNMP（本站环境）的操作。有搞出很酷的操作欢迎分享。
