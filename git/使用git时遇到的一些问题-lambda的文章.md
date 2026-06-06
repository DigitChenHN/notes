---
id: "15816609266"
title: "使用git时遇到的一些问题"
author: "lambda"
type: zhihu-article
source: "https://zhuanlan.zhihu.com/p/15816609266"
created: "2025-03-16 00:33"
updated: "2025-05-08 20:52"
downloaded: "2026-05-14"
---
## 情况1

使用ssh连接远程仓库，使用`[git pull](https://zhida.zhihu.com/search?content_id=252185977&content_type=Article&match_order=1&q=git+pull&zhida_source=entity)`命令时访问不了， 命令行终端打印的报错信息为：

```text
kex_exchange_identification: Connection closed by remote host
Connection closed by 20.205.243.166 port 443
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

首先要确定的是已经通过ssh配置好了本地主机和github远程主机的密钥，之前是可以成功使用`git push`和`git pull`的，也就是说本地主机和远程主机的配置应该是正确的。如果可以确定，则很可能是代理导致的问题。

1.  **考虑关闭代理：**

关闭代理前，命令行终端迅速返回上面的报错信息；而关闭代理之后，需要等待非常长的时间才会返回以上报错信息，并且报错信息为：

```text
ssh: connect to host github.com port 443: Connection timed out
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

通过Connection timed out 应该可以判断，不走代理直接无法与github通信。这个是可以理解的，国内网络对于github的访问是不稳定的，虽然有时候也可以裸连，但现在关闭网络代理之后明显在访问github这一步上就已经出现问题。

2\. **重新打开代理后检查在github上的公钥是否和本地私钥匹配**

```text
ssh-keygen -l -f ~/.ssh/本地公钥的名称
```

通过以上终端命令得到公钥指纹，并且跟github账户-setting-SSH and GPG keys中设置的公钥指纹对比。

确认本地私钥和github远程主机上的公钥是配对的。

3\. **检查SSH配置文件**

当前用户的ssh配置文件路径为`~/.ssh/config`，其中内容如下：

```text
# github
Host github.com
User git
HostName github.com
Port 443
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_deck2github 
```

对比文章[解决 GitHub 22 端口被占用，改用 443 端口连接-腾讯云开发者社区-腾讯云](https://link.zhihu.com/?target=https%3A//cloud.tencent.com/developer/article/2480102)中的配置文件，发现HostName字段的内容不正确，更改为：

```text
HostName ssh.github.com
```

然后就可以连接了，可能之前是误操作删除了导致了该字段的内容错误。**可能这个问题大多数人都不会发生，所以借鉴意义不大。**

借此记录一下这个ssh配置造成以上问题的文件字段的意义：

**Host**定义了一个主机条目，通过ssh访问这个主机的时候，会使用其下面的配置访问。

**HostName**指定了实际要连接的主机名。

这意味着即便在命令中使用了[http://github.com](https://link.zhihu.com/?target=http%3A//github.com)作为主机名，但ssh客户端实际会尝试连接到[http://ssh.github.com](https://link.zhihu.com/?target=http%3A//ssh.github.com)。

## 情况2

使用http的方式连接的远程仓库，在gitpull的时候报错：fatal: unable to access '[https://github.com/](https://link.zhihu.com/?target=https%3A//github.com/DigitChenHN/triboelectric-nanogenerator-project.git)xxx': Failed to connect to [github.com](https://link.zhihu.com/?target=http%3A//github.com/) port 443 after 21095 ms: Could not connect to server。

突然出现这种情况很可能就是因为代理导致的问题，尤其可能发生于习惯于在需要时打开，不需要时关闭代理的人身上。

这时候首先考虑将原先的代理设置清除：

```text
git config --global --unset http.proxy 
git config --global --unset https.proxy
```

这时候可以使用以下命令查看现在的配置：

```text
git config --list
```

会发现其中缺少像下面这样的配置字段，这就说明http相关的配置已经被清除：

```text
http.proxy=http://xxxx
```

这时候我们再手动的添加配置：

```text
git config --global http.proxy http//127.0.0.1:7890
```

> 其中http//127.0.0.1是一个环回地址，所谓“环回”我的理解就是指向本机。而7890则是端口号，根据我的理解，每个需要参与网络通信的应用程序都会由一个端口号，像7890这个端口号似乎就经常是被一些代理软件使用。

这时候我们在使用`git config --list`查看配置就会发现出现了相关的字段`http.proxy=http://127.0.0.1:7890`

大部分情况下这样就可以了。如果还不行，可以尝试重启以下代理软件。