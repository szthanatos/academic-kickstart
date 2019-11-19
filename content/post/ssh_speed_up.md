---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "加速ssh连接"
subtitle: ""
summary: ""
authors: []
tags: ['ssh']
categories: ['Linux']
date: 2019-11-15T17:18:05+08:00
lastmod: 2019-11-15T17:18:05+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
## 报错信息

之前运维给做了几台测试服务器，远程连接的时候速度特别慢，ssh之后需要接近1分钟才能连上。

## 原因

使用`ssh -v <服务器>` 显示连接过程：

```bash
$ ssh -v 123.456.789.0
OpenSSH_6.6.1, OpenSSL 1.0.1e-fips 11 Feb 2013
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 56: Applying options for *
debug1: Connecting to 123.456.789.0 [123.456.789.0] port 22.
debug1: Connection established.
debug1: permanently_set_uid: 0/0
debug1: identity file /root/.ssh/id_rsa type -1
debug1: identity file /root/.ssh/id_rsa-cert type -1
debug1: identity file /root/.ssh/id_dsa type -1
debug1: identity file /root/.ssh/id_dsa-cert type -1
debug1: identity file /root/.ssh/id_ecdsa type -1
debug1: identity file /root/.ssh/id_ecdsa-cert type -1
debug1: identity file /root/.ssh/id_ed25519 type -1
debug1: identity file /root/.ssh/id_ed25519-cert type -1
debug1: Enabling compatibility mode for protocol 2.0
debug1: Local version string SSH-2.0-OpenSSH_6.6.1
debug1: Remote protocol version 2.0, remote software version OpenSSH_6.6.1
debug1: match: OpenSSH_6.6.1 pat OpenSSH_6.6.1* compat 0x04000000
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received
debug1: kex: server->client aes128-ctr hmac-md5-etm@openssh.com none
debug1: kex: client->server aes128-ctr hmac-md5-etm@openssh.com none
debug1: kex: curve25519-sha256@libssh.org need=16 dh_need=16
debug1: kex: curve25519-sha256@libssh.org need=16 dh_need=16
debug1: sending SSH2_MSG_KEX_ECDH_INIT
debug1: expecting SSH2_MSG_KEX_ECDH_REPLY
debug1: Server host key: ECDSA 3f:00:c1:54:09:7a:aa:50:93:a2:53:83:74:b5:07:8f
debug1: Host '123.456.789.0' is known and matches the ECDSA host key.
debug1: Found key in /root/.ssh/known_hosts:2
debug1: ssh_ecdsa_verify: signature correct
debug1: SSH2_MSG_NEWKEYS sent
debug1: expecting SSH2_MSG_NEWKEYS
debug1: SSH2_MSG_NEWKEYS received
debug1: Roaming not allowed by server
debug1: SSH2_MSG_SERVICE_REQUEST sent
debug1: SSH2_MSG_SERVICE_ACCEPT received
debug1: Authentications that can continue: publickey,gssapi-keyex,gssapi-with-mic,password
debug1: Next authentication method: gssapi-keyex # 0
debug1: No valid Key exchange context
debug1: Next authentication method: gssapi-with-mic # 1
debug1: Unspecified GSS failure.  Minor code may provide more information
No Kerberos credentials available

debug1: Unspecified GSS failure.  Minor code may provide more information
No Kerberos credentials available

debug1: Unspecified GSS failure.  Minor code may provide more information


debug1: Unspecified GSS failure.  Minor code may provide more information
No Kerberos credentials available

debug1: Next authentication method: publickey # 2
debug1: Trying private key: /root/.ssh/id_rsa
debug1: Trying private key: /root/.ssh/id_dsa
debug1: Trying private key: /root/.ssh/id_ecdsa
debug1: Trying private key: /root/.ssh/id_ed25519
debug1: Next authentication method: password # 3
```

发现卡住的位置是`debug1: Next authentication method: gssapi-with-mic`附近。

证明是由于`gssapi`认证带来的问题。

从网上找到相关的解释：

> 1. GSSAPI(Generic Security Services Application Programming Interface)是一套通用网络安全系统接口。
> 该接口是对各种不同的客户端服务器安全机制的封装，以消除安全接口的不同，降低编程难度。
>
> 2. OpenSSH在用户登录的时候会验证IP，它根据用户的IP使用反向DNS找到主机名，再使用DNS找到IP地址，最后匹配一下登录的IP是否合法。

进行身份认证的时候，OpenSSH虽然说的是`publickey,gssapi-keyex,gssapi-with-mic,password`，

但默认顺序是：`gssapi-with-mic` → `hostbased` → `publickey` → `keyboard-interactive` → `password`

上面连接过程我也标出了0123，实际顺序的确如此。

`gssapi`的认证是基于`Kerberos`的，没见到人用过，

另一方面，客户端反向DNS的过程也会在连接DNS服务器/查询客户端域名(没域名可就会一层层DNS查上去)上花费时间。

## 解决办法

客户端，编辑`/etc/ssh/ssh_config`文件：

- 方式1：将`GSSAPIAuthentication`改为no；
- 方式2：编辑/新增`PreferredAuthentications`为publickey或者password，改变认证优先度;

服务端，编辑`/etc/ssh/sshd_config`文件：

1. 将`UseDNS`改为no；
2. (可选)将`GSSAPIAuthentication`改为no(所有连接都不做gssapi认证了)；
3. 重启sshd服务；

实际效果，关闭`GSSAPIAuthentication`让连接时间从1分钟下降到8秒左右，关闭`UseDNS`后几乎接近秒连。
