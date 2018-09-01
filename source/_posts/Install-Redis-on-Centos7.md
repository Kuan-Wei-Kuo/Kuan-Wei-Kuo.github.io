---
title: Install Redis on Centos7
date: 2018-02-01 20:35:09
tags: Redis
---

Redis是一個OpenSource的CacheServer，人家可是NoSQL + KeyValue概念呢! 想當然Cache有分內存、磁碟等兩種方式，要使用哪一種儲存方式全看需求。然而Redis身為一個社群活耀的OpenSource，那麼內建transactions、replication以及data structures等多樣功能肯定也是很正常的一件事情。

話不多說，進入主題，窩們今天要在centos7上面給他裝裝Redis囉!

# Yum Install
最簡單的安裝方式。

## Step 1. Update yum and install EPEL repository
```
yum install epel-release
yum update
```

## Step 2. Install Redis
```
yum install redis

# 開機自動啟動囉!
systemctl enable redis
```

## Step 3. Configuration redis.conf
使用這種方式安裝的設定檔可於/etc/redis.conf看到，我們需要編輯他。
```
vi /etc/redis.conf

#----加入以下資訊----#

#加入對外IP，若沒有加可以於local測試，反之則無法於外面機器測試
bind 127.0.0.1 192.168.56.101
```

# Download Install
如果你不想使用yum進行安裝的人，可以使用這個方式。
## Step 1. Install necessary packages
Redis有使用到其他的Source，因此一並給他裝上去。
```
yum install wget make gcc
```

## Step 2. Install Redis
```
mkdir /usr/redis
wget http://download.redis.io/releases/redis-3.2.9.tar.gz // download
tatar -zxvf redis-3.2.9.tar.gz -C /usr/redis --strip-components=1 // untar
make && make install
```

Step 3. Install Init Redis Script
在這個步驟你會需要輸入redis的相關設定，那基本上我是都照預設下去跑，一直Enter。
```
cd /usr/redis/utils
./install_server.sh

# 開機自動啟動囉!
systemctl enable redis_6379
```

## Step 4. Configuration redis.conf
若是像上述直接enter安裝的你，會在/etc/redis/6379.conf發現redis設定檔，我們需要編輯他。
```
vi /etc/redis/6379.conf

#----加入以下資訊----#

#加入對外IP，若沒有加可以於local測試，反之則無法於外面機器測試
bind 127.0.0.1 192.168.56.101
```

# Testing
我們就簡單測試看看，理論上會儲存成功，並且Get出123字串。
```
redis-cli set andy '123'
redis-cli get andy
```