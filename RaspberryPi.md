## 系统安装（mac）

下载 https://www.raspberrypi.org/downloads/raspbian/ 并解压

查看sd卡磁盘路径,并卸载
```bash
diskutil list
diskutil unmountDisk /dev/disk2
```

安装img镜像到sd卡
```bash
sudo dd bs=8m if=RaspberryPI.img of=/dev/disk2
```

## 中文输入法无法使用
```bash
sudo apt remove fcitx-module-kimpanel
sudo apt-get remove fcitx-ui-qimpanel
reboot
```

## vi 无法识别上下左右和Backspace 
刚装好系统,使用上下左右会变成abcd

编辑 `/etc/vim/vimrc.tiny` 文件

将 `set compatible` 改为 `set nocompatible`

增加一行 `set backspace=2`

或者直接用 nano 代替

## 连接无线
```bash
sudo vi /etc/wpa_supplicant/wpa_supplicant.conf
```
添加多个network

```
#隐藏并没有密码
network={
    ssid="yourHiddenSSID"
    scan_ssid=1
    key_mgmt=NONE
}
```
刷新wifi配置 或 reboot后生效
```
wpa_cli -i wlan0 reconfigure
```

## 打开ssh

```
sudo systemctl enable ssh
sudo systemctl start ssh
```
修改端口 Port 22,改成想要的端口
```
sudo vi /etc/ssh/sshd_config
```


## 安装LCD屏幕驱动

安装git
```bash
sudo apt-get install git
```

下载驱动
```bash
git clone https://github.com/goodtft/LCD-show.git
chmod -R 755 LCD-show
cd LCD-show/
```

启动LCD
```bash
sudo ./LCD35-show
```
切换HDMI
```bash
sudo ./LCD-hdmi
```

## 安装nodejs
下载ARMv7版本的nodejs https://nodejs.org/en/download/ 并解压

设置软连接
```bash
ln -s /usr/software/nodejs/bin/npm   /usr/local/bin/ 
ln -s /usr/software/nodejs/bin/node   /usr/local/bin/
```

## 关于electron
安装4.x以上的版本会报 `GLIBC_2.27 not found` 的错误，所以使用3.x的版本

设置全局变量 避免下载出错
```bash
npm config set electron_mirror http://npm.taobao.org/mirrors/electron/
```

## 开机默认 命令行 或 图形界面 切换

- 图形界面下

首选项 - RaspBerry Pi Configuration - System - To Cli

- 命令行
```bash
sudo raspi-config
```

Boot Options - B1 Desktop / CLI - B4 Desktop Autologin

## VNC配置
```bash
sudo raspi-config
```

Boot Options - 5 Interfacing Options - P3 VNC - enabled

启动并查看 vnc 端口
```bash
vncserver
```

MAC 自带的vnc 无法连接不知道怎么搞，所以下载 [vnv viewer](https://www.realvnc.com/en/connect/download/viewer/)

## 查看CPU温度
两种方法
```bash
pi@raspberrypi:~ $ vcgencmd measure_temp
temp=40.2'C
pi@raspberrypi:~ $ cat /sys/class/thermal/thermal_zone0/temp
40242
```

## docker 
安装
```bash
curl -sSL https://get.docker.com/ | sh
```
安装后提示以非root用户无法运行docker。
```bash
If you would like to use Docker as a non-root user, you should now consider
    adding your user to the "docker" group with something like:

    sudo usermod -aG docker runoob
   Remember that you will have to log out and back in for this to take effect! 
```

所以将pi用户添加到 docker 组
```bash
sudo usermod -aG docker pi
```



```bash
sudo service docker start
```

登陆docker账号,可以不用
```bash
docker login
```

删除容器
```bash
#查看所有容器,-a 包括停止的
docker ps -a
docker rm 容器name/id
```

查看容器详情
```
docker inspect 容器name/id
```

删除image
```bash
docker images
docker rmi 镜像name/id
```

启动/停止已有容器
```bash
docker start 容器name/id
docker stop 容器name/id
```

查看容器日志
* -f : 跟踪日志输出
* --since :显示某个开始时间的所有日志
* -t : 显示时间戳
* --tail :仅列出最新N条容器日志

```bash
docker logs -f -t --since="2018-07-01" --tail=10 容器name/id
```
