---
title: ssh_exchange_identification:Connection closed by remote host 报错问题
date: 2016-08-28 22:55:11
categories: bug
tags: [GitHub,SSH,Debug]
---
## 错误情况
今天下午，我莫名无法向 GitHub push 代码了...最终从5点多调试到晚上11点半..感觉略坑..遂记录如下..

报错内容：
```
fatal: unable to access 'https://***.git/': SSL peer handshake failed, the server most likely requires a client certificate to connect
```

```
ssh_exchange_identification: Connection closed by remote host
fatal: Could not read from remote repository.
```

## 解决过程

首先简单搜了一下发现可能是 ssh key 的问题..遂重新按照[官方文档](https://help.github.com/articles/generating-an-ssh-key/)重新生成 ssh key。并再添加 key 后执行 `ssh -T git@github.com` 来测试，仍然报错:
`ssh_exchange_identification: Connection closed by remote host`

执行`ssh -vT git@github.com`输出记录如下：

```
$ ssh -vT git@github.com
OpenSSH_6.9p1, LibreSSL 2.1.8
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 21: Applying options for *
debug1: Connecting to github.com [1.111.11.111] port 22.
debug1: Connection established.
debug1: identity file /Users/yangzhongjing/.ssh/id_rsa type 1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/yangzhongjing/.ssh/id_rsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/yangzhongjing/.ssh/id_dsa type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/yangzhongjing/.ssh/id_dsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/yangzhongjing/.ssh/id_ecdsa type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/yangzhongjing/.ssh/id_ecdsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/yangzhongjing/.ssh/id_ed25519 type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/yangzhongjing/.ssh/id_ed25519-cert type -1
debug1: Enabling compatibility mode for protocol 2.0
debug1: Local version string SSH-2.0-OpenSSH_6.9
ssh_exchange_identification: Connection closed by remote host
```

这里面`/Users/yangzhongjing/.ssh/id_rsa` 这个文件是存在的。不知道为什么会报：`key_load_public: No such file or directory`..

后来在网上搜了很多内容，执行了以下方法：
- `vim /etc/hosts.allow` 添加 `sshd: ALL`
- `ssh-add ~/.ssh/id_rsa`
- 调整了 `/etc/ssh/sshd_config ` 中的 `MaxSessions 10 ` 调大
- 删除 `~/.ssh`目录重新生成ssh key 

然而并没有什么卵用...
## 最终原因..

最后在茫茫文海中搜到了一篇关于路由器端口禁用也可能会导致这个问题的留言..突然想到昨天我自己也调试了家里的路由器，添加了一些路由器插件...赶紧切到路由器后台，发现了幕后凶手...

![](http://qcyoung.qiniudn.com/qcyoung/shijietong.png)

![](http://qcyoung.qiniudn.com/qcyoung/shijietongdetail.png)

最终..禁用插件，重启路由。问题解决..

SO...后来人可以试试上面列举到的搜索的方法..或者看看你的网络设置，一般可以解决这一报错问题..

