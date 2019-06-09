## mac 开发环境搭建

### 安装node
node 10.x 有问题,最好使用 node 8.x

`https://nodejs.org/en/blog/release/v8.11.3/`

### 安装xcode
安装 xcode 和 xcode command_line_tools

可以从app store安装,如果系统版本低的话也可以从官网安装低版本的 xcode `https://developer.apple.com/download/more/`

### 相关依赖

安装 brew
```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

使用brew安装相关依赖
```bash
brew install pkg-config
brew install zmq
brew install yasm
```

### 启动stf

```bash

git clone https://github.com/openstf/stf.git
cd stf
#安装依赖
npm install
# build 并link 到全局
npm link
#启动前请确认 rethinkdb 已启动
stf local
```

### 打包代码到docker
```bash
docker build -t 新的镜像名 . 
```



## 安装 rethinkdb

使用 docker 比较方便

```bash
docker pull rethinkdb:latest
```

使用 `--bind all` 表示对外开放 http://ip:8080 RethinkDB Administration Console

使用 `-v 本地路径:/data` 映射数据库文件给容器

注意 再 mac 下无法使用 `--net host` 来映射端口必须使用 `-p` 指定所有端口

```bash
docker run -d --name some-rethink \
-v "/Users/jpsun/Desktop/rethinkdb_data:/data" \
-p 29015:29015 -p 28015:28015 -p 8080:8080 \
rethinkdb \
rethinkdb --bind all 
```

## linxu下docker启动stf 

**注意只有linux下docker才支持`--net host`参数,所以以下内容只支持linux**


### 启动主服务器
```bash
#!/bin/bash

if [ ! -n "$1" ] ;then
	dockerImage="openstf/stf"
else
	dockerImage="$1"
fi

if [ ! -n "$2" ] ;then
	hostname="www.你的域名.com"
else
	hostname="$2"
fi

echo "使用镜像: ${dockerImage}"
echo "stf hostname: ${hostname}"

echo "停止所有容器"
docker stop $(docker ps -a -q)
sleep 1

echo "删除所有容器"
docker rm -v $(docker ps -a -q)
sleep 1

# 启动 rethinkdb
echo "启动 rethinkdb"
docker run -d --name some-rethink -v "/root/rethinkdb_data:/data" --net host rethinkdb rethinkdb
sleep 3


# 初始化数据表,只需要执行一次
echo "rethinkdb init"
docker run -d --name stf-migrate ${dockerImage} stf migrate
sleep 3

echo "启动 stf app"
docker run -d --name stf-app --net host -e "SECRET=YOUR_SESSION_SECRET_HERE" ${dockerImage} stf app --port 7100 --auth-url http://${hostname}/auth/mock/ --websocket-url ws://${hostname}/
sleep 3

echo "启动 stf auth-mock"
docker run -d --name stf-auth --net host -e "SECRET=YOUR_SESSION_SECRET_HERE" ${dockerImage} stf auth-mock --port 7101 --app-url http://${hostname}/
sleep 1

echo "启动 stf websocket"
docker run -d --name websocket --net host -e "SECRET=YOUR_SESSION_SECRET_HERE" ${dockerImage} stf websocket --port 7102 --storage-url http://${hostname}/ --connect-sub tcp://${hostname}:7150 --connect-push tcp://${hostname}:7170
sleep 1

echo "启动 stf api"
docker run -d --name stf-api --net host -e "SECRET=YOUR_SESSION_SECRET_HERE" ${dockerImage} stf api --port 7103 --connect-sub tcp://${hostname}:7150 --connect-push tcp://${hostname}:7170
sleep 1

echo "启动 stf storage-plugin-apk"
docker run -d --name storage-apk --net host ${dockerImage} stf storage-plugin-apk --port 7104 --storage-url http://${hostname}/
sleep 1

echo "启动 stf storage-plugin-image"
docker run -d --name storage-image --net host ${dockerImage} stf storage-plugin-image --port 7105 --storage-url http://${hostname}/
sleep 1

#注意这个 /root/storage 文件夹必须赋权 chmod 777
echo "启动 stf storage-temp"
docker run -d --name storage-temp --net host -v /root/storage:/data ${dockerImage} stf storage-temp --port 7106 --save-dir /data
sleep 1

echo "启动 stf triproxy app"
docker run -d --name triproxy-app --net host ${dockerImage} stf triproxy app --bind-pub "tcp://*:7150" --bind-dealer "tcp://*:7160" --bind-pull "tcp://*:7170"
sleep 1

echo "启动 stf processor"
docker run -d --name stf-processer --net host ${dockerImage} stf processor stf-processer --connect-app-dealer tcp://${hostname}:7160 --connect-dev-dealer tcp://${hostname}:7260
sleep 1

echo "启动 stf triproxy dev"
docker run -d --name triproxy-dev --net host ${dockerImage} stf triproxy dev --bind-pub "tcp://*:7250" --bind-dealer "tcp://*:7260" --bind-pull "tcp://*:7270"
sleep 1

echo "启动 stf reaper dev"
docker run -d --name stf-reaper --net host ${dockerImage} stf reaper dev --connect-push tcp://${hostname}:7270 --connect-sub tcp://${hostname}:7150 --heartbeat-timeout 30000

```

### nginx 配置

```
#daemon off
worker_processes 1;

events {
  worker_connections 1024;
}
#如果nginx与服务器不在一台上将127.0.0.1改为服务器IP
http {
  upstream stf_app {
    server 127.0.0.1:7100 max_fails=0;
  }

  upstream stf_auth {
    server 127.0.0.1:7101 max_fails=0;
  }

  upstream stf_storage_apk {
    server 127.0.0.1:7104 max_fails=0;
  }

  upstream stf_storage_image {
    server 127.0.0.1:7105 max_fails=0;
  }

  upstream stf_storage {
    server 127.0.0.1:7106 max_fails=0;
  }

  upstream stf_websocket {
    server 127.0.0.1:7102 max_fails=0;
  }

  upstream stf_api {
    server 127.0.0.1:7103 max_fails=0;
  }

  types {
    application/javascript  js;
    image/gif               gif;
    image/jpeg              jpg;
    text/css                css;
    text/html               html;
  }

  map $http_upgrade $connection_upgrade {
    default  upgrade;
    ''       close;
  }

  server {
    listen 80;
    server_name www.你的域名.com;
    keepalive_timeout 70; 
#    resolver 114.114.114.114 8.8.8.8 valid=300s;
#    resolver_timeout 10s;

# 如果不配置,图像minicap等服务将直连provider服务器,
# 通过配置可以让nginx转发请求到provider服务器,这样对外时可以不用暴露provider服务器.
# floor4 : provider的 --name
# proxy_pass :  provider的内网IP

#    location ~ "^/d/floor4/([^/]+)/(?<port>[0-9]{5})/$" {
#      proxy_pass http://192.168.0.106:$port/;
#      proxy_http_version 1.1;
#      proxy_set_header Upgrade $http_upgrade;
#      proxy_set_header Connection $connection_upgrade;
#      proxy_set_header X-Forwarded-For $remote_addr;
#      proxy_set_header X-Real-IP $remote_addr;
#    }



    location /auth/ {
      proxy_pass http://stf_auth/auth/;
    }

    location /api/ {
      proxy_pass http://stf_api/api/;
    }

    location /s/image/ {
      proxy_pass http://stf_storage_image;
    }

    location /s/apk/ {
      proxy_pass http://stf_storage_apk;
    }

    location /s/ {
      client_max_body_size 1024m;
      client_body_buffer_size 128k;
      proxy_pass http://stf_storage;
    }

    location /socket.io/ {
      proxy_pass http://stf_websocket;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Real-IP $http_x_real_ip;
    }

    location / {
      proxy_pass http://stf_app;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Real-IP $http_x_real_ip;
    }
  }
}

```

### 启动 provider 服务器

**暴露 provider 服务器**
```bash
docker run -d --name sunshine --net host openstf/stf-armv7l \
stf provider --name sunshine --connect-sub tcp://www.你的域名.com:7250 --connect-push tcp://www.你的域名.com:7270 --min-port=15000 --max-port=25000 --heartbeat-interval 20000 --allow-remote --no-cleanup --storage-url http://www.你的域名.com
```

**不暴露 provider 服务器**

注意:需要配合nginx进行转发
```bash
docker run -d --name sunshine --net host openstf/stf-armv7l \
stf provider --name sunshine --connect-sub tcp://www.你的域名.com:7250 --connect-push tcp://www.你的域名.com:7270 --min-port=15000 --max-port=25000 --heartbeat-interval 20000 --allow-remote --no-cleanup --storage-url http://www.你的域名.com \
--public-ip www.你的域名.com --screen-ws-url-pattern "ws://www.你的域名.com/d/pi1/<%= serial %>/<%= publicPort %>/"
```