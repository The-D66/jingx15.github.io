---
layout:     post
title:      在docker中部署transmission
subtitle:   在nas服务器上用docker中部署transmission，方便重新部署。
date:       2019-04-11
author:     D
header-img: img/post-bg-2015.jpg 
catalog: true
tags:
    - 技术
    - docker
    - ubuntu
    - linux
    - bt 下载
    - pt 下载
---

# 镜像介绍

采用了 [`docker pull linuxserver/transmission`](https://hub.docker.com/r/
linuxserver/transmission/) 镜像，能够比较方便地部署，同时进行用户权限管理（虽然对我
没什么用）。

# 部署命令

```bash
docker create \
  --name=transmission \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e TRANSMISSION_WEB_HOME=/combustion-release/ `#optional` \
  -p 9091:9091 \
  -p 51413:51413 \
  -p 51413:51413/udp \
  -v /data/transmission/config:/config \
  -v /data:/downloads \
  -v /data/transmission/watch:/watch \
  --restart unless-stopped \
  linuxserver/transmission
```

其中 `/config` 和 `/downloads` 是必须的，前者储存容器设置，后者为下载目录，`/watch`
目录对于我这样基本是采用远程客户端的人基本用不上。

如果有多个下载目录需要挂载，可以采用下述代码。
```bash
  -v /data1:/downloads/1 \
  -v /data2:/downloads/2 \
  ...
```

# 远程访问

## 采用网页的方式

访问 `http://server_ip:9091` 或者在上一个命令中自定义的端口。

## 采用远程客户端的方式

参见 https://github.com/transmission-remote-gui/transgui。
