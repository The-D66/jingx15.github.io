---
layout:     post
title:      nas 的 docker 部署命令合集
subtitle:   NAS 上用的一系列镜像的部署命令合集，方便以后有重新部署需求时查看
date:       2019-06-02
author:     D
header-img: img/post-bg-2015.jpg 
catalog: true
tags:
    - 技术
    - docker
    - nas
---

# resilio

```bash
docker run\
  --name=resilio-sync \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e UMASK_SET=022 \
  -p 12580:8888 \
  -p 55555:55555 \
  -v /data/resilio/config:/config \
  -v /data/resilio/downloads:/downloads \
  -v /data/resilio/data:/sync \
  --restart unless-stopped \
  linuxserver/resilio-sync
```

# gitlab

由于80端口和443端口有别的程序在用，因此全部+10000。在启动之后需要修改
`/etc/gitlab/gitlab.rb`文件。

```bash
sudo docker run --detach \
  --hostname gitlab.example.com \
  --publish 10443:443 --publish 10080:80 --publish 10022:22 \
  --name gitlab \
  --restart always \
  --volume /data/gitlab/config:/etc/gitlab \
  --volume /data/gitlab/logs:/var/log/gitlab \
  --volume /data/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

# Transmission

详见[这篇文章](/_posts\2019-04-11-在docker中部署transmission.md)。

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