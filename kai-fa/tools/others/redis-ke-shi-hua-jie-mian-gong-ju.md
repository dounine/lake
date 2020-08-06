---
description: '现在比较出名的跨平台可视化界面有两款，web上的就比较多,但功能不是很强大，没有native版本的强大。'
---

# Redis 可视化界面工具

## RedisDesktop

![](../../../.gitbook/assets/image%20%2826%29.png)

源码方式安装

```text
git clone --recursive https://github.com/uglide/RedisDesktopManager.git -b 0.9 rdm && cd ./rdm
```

Ubuntu

```text
cd src/
./configure
source /opt/qt59/bin/qt59-env.sh && qmake && make && sudo make install
cd /usr/share/redis-desktop-manager/bin
sudo mv qt.conf qt.backup
```

Fedora & CentOS & OpenSUSE

```text
cd src/
./configure
qmake-qt5 && make && sudo make install
cd /usr/share/redis-desktop-manager/bin
sudo mv qt.conf qt.backup
```

## FastoRedis

官网地扯: [传送门](https://fastoredis.com/)

通过下载编译后包的即可使用

下载地扯: [传送门](https://fastoredis.com/anonim_users_downloads)

