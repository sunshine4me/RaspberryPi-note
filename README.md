# RaspberryPi-note

## centos 安装

1. 下载系统文件 http://isoredirect.centos.org/altarch/7/isos/armhfp/ 并解压出raw文件。

2. 使用dd 命令烧录文件到SD卡。
```
sudo dd bs=8m if=CentOS-Userland-7-armv7hl-RaspberryPI-Minimal-1810-sda.raw of=/dev/disk2
```

3. 进入操作系统 root/centos，将系统空间扩容到SDK最大容量。
```
/usr/bin/rootfs-expand
```



## 连接wifi

查看wifi 列表
```bash
nmcli d wifi
```
连接wifi
```bash
nmcli d wifi connect SSID password 'password'
```

断开wifi
```bash
nmcli dev dis wlan0
```

连接已经有的连接
```bash
nmcli connection up "connection_name"
```

查看现有连接的UUID
```bash
nmcli c
```
删除wifi连接
```bash
nmcli c del UUID
nmcli c del "connection_name"
```

## 系统安装备份还原（macOS）

SD卡可以使用系统自带的"磁盘工具"进行格式化，选择MS-DOS(FAT)格式。

查看SD卡磁盘路径
```bash
diskutil list
```


卸载SD卡 
```bash
diskutil unmountDisk /dev/disk2
```


生成 img
```bash
sudo dd bs=8m if=/dev/disk2 of=backup.img
```

通过img还原
```bash
sudo dd bs=8m if=backup.img of=/dev/disk2
```
` 生成和还原时间较长,可能需要30～60分钟。`

## 系统安装备份还原-最小img（macOS）

//TODO


## 安装 nodejs

安装gcc-c++ 和 make（不知道是不是必须的）
```bash
sudo yum install -y gcc-c++ make
```