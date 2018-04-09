---
title: ctfd 搭建过程
date: 2018-03-20 09:39:25
categories: Other
tags:
 - ctf
description:
---
搭个[ctfd](https://github.com/CTFd/CTFd)玩玩
操作系统：ubuntu16.04

<!-- more -->
```
sudo apt install git # 安装git
sudo apt install python-pip # 安装pip
sudo pip install Flask # 安装Flask
cd /var # 移动目录
sudo git clone https://github.com/CTFd/CTFd.git # 下载ctfd项目
cd CTFd
sudo ./prepare.sh # 执行sh，然后会安装很多东西
sudo python serve.py # 如果正常，本地已经可以运行
sudo pip install gunicorn # 安装gunicorn
sudo gunicorn --bind 0.0.0.0:80 -w 4 "CTFd:create_app()" # -w参数一般设置为 2 * cpu数 + 1
```
