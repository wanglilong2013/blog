---
title: Git用户如何通过SSH登录Gogs服务器
date: 2020-04-30 22:04:47
tags:
---

﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿Git用户SSH登录GOGS服务器

## 问题
----

Gogs安装配置完成后，未注册git用户的情况下，通过SSH登录是完全没有问题的，如果注册了git用户并上传了公钥，再通过SSH登录，会报如下错误:

```
wanglilong:~ % ssh king@git.wiiking.com
PTY allocation request failed on channel 0
Hi there, You've successfully authenticated, but Gogs does not provide shell access.
If this is unexpected, please log in with password and setup Gogs under another user.
Connection to git.wiiking.com closed.

```

## 原因
----
针对Gogs配置的SSH密钥被Gogs增加了特殊处理，密钥内容之前添加了command，不能再提供给Git用户使用。
具体内容可以查看Gogs服务器的authorized_keys。这里就不做深究，有兴趣的童鞋可以研究一下。


##  解决方案
----

* 通过SSH 多密钥认证，针对不同的ssh连接采用不同的密钥。
进入 ～/.ssh目录，可以看到我们已经存在一对密钥了，使用命令再生成一对密钥即可，注意命名不要和已有的密钥重名。

```
ssh-keygen -t rsa -f id_rsa_git -C "18282042234@163.com"
```

* 在～/.ssh目录创建config文件，添加如下内容:

```
Host git.wiiking.com
    IdentityFile ~/.ssh/id_rsa_git
    IdentitiesOnly yes
Host wiiking.com
    IdentityFile ~/.ssh/id_rsa
    IdentitiesOnly yes
```






