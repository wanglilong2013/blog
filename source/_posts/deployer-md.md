---
title: PHP项目部署工具-deployer
date: 2019-06-02 14:42:22
tags: deploy
---

# 说明
---------
Deployer是用于部署任何PHP应用程序的CLI工具,用php语言编写，支持目前流行的框架（Laravel、Yii、Symfony、CakePHP、Drupal等）

## 1. deployer安装

在开发环境/部署机 安装deployer
```
curl -LO https://deployer.org/deployer.phar
mv deployer.phar /usr/local/bin/dep
chmod +x /usr/local/bin/dep
```

## 2. 服务器配置

添加用户
```
sudo adduser deployer

``` 

将 depoloyer 用户加到 sudoers 中
```
$ vim /etc/sudoers
# 在最后加入
deployer ALL=(ALL) NOPASSWD: ALL
# 保存并退出

```

创建ssh密钥
```
ssh-keygen -t rsa -b 4096 -C "deployer" 
```

将生成的公钥拷贝出来,去代码仓库添加SSH 公钥（保证我们的deployer账号能从服务器clone代码）
```
cat ~/.ssh/.id_rsa.pub
```


## 项目部署


SSH 免密部署
将开发环境/部署机的公钥拷贝到服务器deployer用户
```
ssh-copy-id -i ~/.ssh/id_rsa.pub deployer@47.93.34.21

```

切换至项目目录
```
cd /www/project
```

然后执行以下命令
```
dep init
```
此时会提示让我们选中项目类型,如果你不确定你项目类型，选中common类型即可。该命令执行完后会在当前目录生成一个deploy.php，也就是我们的部署文件，这里我们只需要关心几个关键的配置：

```
//代码所在服务器SSH地址
set('repository', 'king@git.wiiking.com:king/kingconsole.git');

host('47.93.34.21') //服务器ip
    ->user('deployer')  //使用deployer用户身份部署
    ->set('deploy_path', '/home/project'); //项目部署目录
```


配置完成后，执行如下命令
```
dep deploy -vvv
```

Ok,到这里应该会部署成功，并且在服务器会生成两个文件夹和一个软链接文件

current： 指向当前版本的一个软链接
releases: 发布版本的历史文件夹，可以配置最多保留多少个版本
shared：共享文件夹，用于存储各个版本的共享文件

注意此时我们的nginx应该对应修改网站根目录配置
```
//根据实际情况修改即可
root /home/project/current/public

```

更多使用技巧请参考关访文档： https://deployer.org