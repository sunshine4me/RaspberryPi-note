# RaspberryPi-note

## centos 安装

1. 系统安装
下载系统文件 http://isoredirect.centos.org/altarch/7/isos/armhfp/ 并解压

2. 使用软件 [balenaEtcher](https://www.balena.io/etcher/) 烧录到sd中

3. 进入操作系统 root/centos,运行命令 ```/usr/bin/rootfs-expand``` 将系统空间扩容到SDK最大容量。（好像不是必要步骤）


## 连接wifi

查看wifi 列表
```sh
nmcli d wifi
```
连接wifi
```sh
nmcli d wifi connect SSID password 'password'
```

断开wifi
```sh
nmcli dev dis wlan0
```

连接已经有的连接
```sh
nmcli connection up "connection_name"
```

查看现有连接的UUID
```sh
nmcli c
```
删除wifi连接
```sh
nmcli c del UUID
nmcli c del "connection_name"
```

## 制作镜像

镜像压缩工具下载
```
wget https://raw.githubusercontent.com/Drewsif/PiShrink/master/pishrink.sh
chmod +x pishrink.sh
sudo mv pishrink.sh /usr/bin
```