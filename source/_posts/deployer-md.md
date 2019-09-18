---
title: PHP项目部署工具-deployer
date: 2019-06-02 14:42:22
tags: deploy
---


![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQzIiBoZWlnaHQ9IjE5MyIgdmlld0JveD0iMCAwIDI0MyAxOTMiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PGcgZmlsbD0ibm9uZSIgZmlsbC1ydWxlPSJldmVub2RkIj48cGF0aCBmaWxsPSIjMENGIiBkPSJNMjQyLjc4MS4zOUwuMjA3IDEwMS42NTNsODMuNjA2IDIxLjc5eiIvPjxwYXRoIGZpbGw9IiMwMEIzRTAiIGQ9Ik05Ny41NTUgMTg2LjM2M2wxNC4xMjktNTAuNTQzTDI0Mi43OC4zOSA4My44MTIgMTIzLjQ0MmwxMy43NDMgNjIuOTIyIi8+PHBhdGggZmlsbD0iIzAwODRBNiIgZD0iTTk3LjU1NSAxODYuMzYzbDMzLjc3My0zOS4xMTMtMTkuNjQ0LTExLjQzLTE0LjEzIDUwLjU0MyIvPjxwYXRoIGZpbGw9IiMwQ0YiIGQ9Ik0xMzEuMzI4IDE0Ny4yNWw3OC40ODQgNDUuNjY0TDI0Mi43ODIuMzkxIDExMS42ODMgMTM1LjgybDE5LjY0NCAxMS40MjkiLz48L2c+PC9zdmc+)

## Deployer是什么?
一个由PHP编写的基于ssh协议的项目部署工具，支持现在大多数的流行框架.
包括Laravel、Symfony、Zend Framework等框架



## 原理

在本地安装deployer后, 通过ssh协议去克隆代码、移动、创建文件、执行相关命令来完成部署。


## 安装

```
curl -LO https://deployer.org/deployer.phar
mv deployer.phar /usr/local/bin/dep
chmod +x /usr/local/bin/dep
```

## 服务器配置(阿里云)

登录服务器后创建用户deployer,此用户需要去代码托管平台拉去代码

```
sudo add user deployer

```

将deployer添加到sudoer，sudo vim /etc/sudoers
```
deployer ALL=(ALL) NOPASSWD: ALL

```

生成SSH密钥，通过ssh协议克隆代码需要

```
ssh-keygen -t rsa -b 4096 -C "deployer"

```

拷贝公钥到代码托管平台



## 开发环境配置

ssh免密部署，将开发环境的公钥拷贝到服务器delpoyer用户(在这里我们假设服务器的公网IP为47.93.134.21)
```
ssh-copy-id -i ~/.ssh/id_rsa.pub deployer@47.93.134.21
```


## 项目部署


切换至项目目录

```
cd /home/king.wang/www/demo

```

执行deployer初始化命令`dep init`，会生成一个deploy.php文件，也就是我们的项目部署清单文件，通过`dep`或`dep-list`命令来查看项目部署的任务列表。

```
dep init
```

接下来我们需要适当的修改几个主要配置项

```
//代码所在服务器SSH地址
set('repository', 'king@git.wiiking.com:king/demo.git');

host('47.93.134.21') //服务器ip
    ->user('deployer')  //使用deployer用户身份部署
    ->set('deploy_path', '/home/project'); //项目部署目录

```


配置完成后，执行如下命令：

```
dep deploy -vvv

```

部署成功后会在服务器会生成两个文件夹和一个软链接文件

* current： 指向当前版本的一个软链接
* releases: 发布版本的历史文件夹，可以配置最多保留多少个版本
* shared：共享文件夹，用于存储各个版本的共享文件

注意此时我们的nginx应该对应修改网站根目录配置
```
root /home/project/demo/public

```

## 参考链接
* https://deployer.org

* https://overtrue.me/articles/2018/06/deployer-guide.html




