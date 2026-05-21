---
title: "RAR/ZIP压缩包密码忘了怎么办？5种破解方法详解（2026最新）"
date: 2026-05-22T10:00:00+08:00
draft: false
image: ""
keywords: "RAR密码破解, ZIP密码忘记, 压缩包密码找回, 加密压缩包破解, WinRAR密码"
readingTime: true
categories: ['技术教程']
tags: ['RAR', 'ZIP', '密码恢复', '压缩包', '加密文件']
description: "RAR/ZIP压缩包密码忘记了？本文介绍5种主流的压缩包密码破解方法，从字典攻击到暴力破解，手把手教你找回丢失的压缩包密码。"
---

## 前言

压缩包加密是保护文件最常用的方式之一。但很多人都有过这样的经历：几年前加密了一个 RAR 或 ZIP 文件，等到需要用的时候，密码怎么也想不起来了。

别慌，这篇文章整理了目前最有效的 5 种压缩包密码恢复方法，从免费工具到专业方案，适合不同技术水平的用户。

## 方法一：John the Ripper（免费开源）

[John the Ripper](https://www.openwall.com/john/) 是安全领域最知名的密码破解工具之一，完全免费开源。

### 安装与使用

**Linux 系统：**

```bash
# Ubuntu/Debian
sudo apt install john

# CentOS/RHEL
sudo yum install john
```

**基本破解流程：**

```bash
# 1. 从 RAR 文件提取密码哈希
rar2john encrypted.rar > hash.txt

# 对于 ZIP 文件
zip2john encrypted.zip > hash.txt

# 2. 使用默认字典破解
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# 3. 查看破解结果
john --show hash.txt
```

### 优缺点

- **优点**：免费、支持格式多、社区活跃
- **缺点**：命令行操作有门槛，依赖本地 CPU/GPU 性能，大字典跑一遍可能需要几天甚至几周

## 方法二：Hashcat（GPU 加速）

[Hashcat](https://hashcat.net/hashcat/) 号称世界上最快的密码破解工具，能充分利用显卡 GPU 进行并行计算。

### 安装与使用

```bash
# 下载并安装
wget https://hashcat.net/files/hashcat-6.2.6.7z
7z x hashcat-6.2.6.7z
cd hashcat-6.2.6

# RAR 破解（模式 13000）
./hashcat -m 13000 -a 0 hash.txt wordlist.txt

# ZIP 破解（模式 13600 = WinZip, 17200 = PKZIP）
./hashcat -m 13600 -a 0 hash.txt wordlist.txt
```

### 常用攻击模式

| 参数 | 模式 | 说明 |
|------|------|------|
| `-a 0` | 字典攻击 | 使用密码字典逐个尝试 |
| `-a 1` | 组合攻击 | 组合多个字典 |
| `-a 3` | 掩码攻击 | 按指定规则暴力破解 |
| `-a 6` | 混合攻击 | 字典 + 掩码组合 |

### 优缺点

- **优点**：速度极快（GPU 加速），支持 300+ 种哈希类型
- **缺点**：需要高性能显卡，配置复杂，普通用户难以上手

## 方法三：ARCHPR（图形界面工具）

Advanced Archive Password Recovery (ARCHPR) 是一款 Windows 下的图形化压缩包密码恢复工具，适合不熟悉命令行的用户。

### 使用步骤

1. 下载并安装 ARCHPR
2. 打开加密的 RAR/ZIP 文件
3. 选择攻击类型：
   - **暴力攻击**：尝试所有可能的字符组合
   - **字典攻击**：使用预设或自定义密码字典
   - **掩码攻击**：按已知线索缩小范围
4. 点击 "Start" 开始破解

### 优缺点

- **优点**：操作简单，图形界面直观
- **缺点**：付费软件（$49 起），破解速度一般，不支持 GPU 加速

## 方法四：在线云端密码恢复服务

如果你不想折腾本地环境，或者本地算力不够，**云端密码恢复服务**是一个省心的选择。

以 **[猫密网 (Catpasswd)](https://www.catpasswd.com)** 为例，这是一个专注加密文件密码找回的在线平台：

### 使用方式

1. 访问 [catpasswd.com](https://www.catpasswd.com)，进入 [恢复页面](https://www.catpasswd.com/recovery)
2. 上传加密的 RAR/ZIP 文件（支持最大 100MB）
3. 填写接收通知的邮箱
4. 系统自动使用云端分布式算力 + 千万级密码字典进行恢复
5. 恢复成功后通过邮件通知

### 亮点

- **零门槛**：无需安装任何软件，网页操作即可
- **云端算力**：分布式集群 7×24 小时运行，不需要开着自己的电脑等待
- **超大文件支持**：超过 100MB 的文件，可以使用平台提供的[特征提取工具](https://www.catpasswd.com)在本地提取文件特征后上传，源文件不会离开你的电脑
- **免费模式**：提供免费恢复选项，可以先试试效果
- **隐私保护**：文件在隔离环境处理，支持付费销毁文件和记录

### 适用场景

当本地破解工具跑了几天没有结果，或者你不确定自己的字典是否够全面时，云端服务可以作为补充方案。毕竟专业平台维护的字典库（超过 1000 万次密码检测）通常比个人收集的字典覆盖更全面。

## 方法五：Python 脚本自定义破解

如果你有一定的编程基础，可以用 Python 写一个简单的暴力破解脚本：

### ZIP 文件破解脚本

```python
import zipfile
import itertools
import string

def crack_zip(zip_path, max_length=6):
    """尝试暴力破解 ZIP 文件密码"""
    zf = zipfile.ZipFile(zip_path)
    chars = string.ascii_letters + string.digits
    
    print(f"开始破解 {zip_path}，最大密码长度: {max_length}")
    
    for length in range(1, max_length + 1):
        print(f"正在尝试 {length} 位密码...")
        for pwd_tuple in itertools.product(chars, repeat=length):
            pwd = ''.join(pwd_tuple)
            try:
                zf.extractall(pwd=pwd.encode())
                print(f"\n✅ 破解成功！密码是: {pwd}")
                return pwd
            except (RuntimeError, zipfile.BadZipFile):
                continue
    
    print("❌ 未找到密码，请尝试更大的密码长度")
    return None

# 使用示例
crack_zip("secret.zip", max_length=4)
```

### RAR 文件破解脚本

```python
import rarfile
import itertools
import string

def crack_rar(rar_path, wordlist_path):
    """使用字典破解 RAR 文件"""
    rf = rarfile.RarFile(rar_path)
    
    with open(wordlist_path, 'r', encoding='utf-8', errors='ignore') as f:
        for line in f:
            pwd = line.strip()
            try:
                rf.extractall(pwd=pwd)
                print(f"✅ 破解成功！密码是: {pwd}")
                return pwd
            except rarfile.RarWrongPassword:
                continue
    
    print("❌ 字典中未找到正确密码")
    return None
```

### 优缺点

- **优点**：完全自定义，可以加入自己的密码规则
- **缺点**：纯 CPU 运算，速度很慢，仅适合短密码或已知部分线索的情况

## 方法对比

| 方法 | 难度 | 速度 | 成本 | 适合人群 |
|------|------|------|------|----------|
| John the Ripper | 中 | 中 | 免费 | 技术人员 |
| Hashcat | 高 | 快 | 免费（需好显卡） | 安全研究员 |
| ARCHPR | 低 | 慢 | $49+ | 普通用户 |
| [猫密网](https://www.catpasswd.com) | 极低 | 快 | 免费/付费 | 所有人 |
| Python 脚本 | 中 | 慢 | 免费 | 程序员 |

## 提高破解成功率的技巧

1. **回忆密码线索**：哪怕是记得密码大概几位、是否包含数字、首字母大写等，都能大幅缩小搜索范围
2. **使用高质量字典**：`rockyou.txt` 是最常用的免费字典（1400 万条），但专业平台维护的字典通常覆盖面更广
3. **优先尝试字典攻击**：大部分人设置的密码都在常见密码列表中，字典攻击比暴力破解快几个数量级
4. **注意 RAR5 vs RAR4**：RAR5 的加密算法更强（AES-256），破解速度比 RAR4 慢很多
5. **大文件用特征提取**：如果压缩包好几个 GB，不要整个上传，用特征提取工具只提取加密特征即可

## 常见问题

### Q: 压缩包密码能 100% 破解吗？

不一定。如果密码足够长且随机（比如 12 位以上的大小写+数字+特殊字符组合），以目前的算力几乎不可能破解。但实际生活中，大多数人设置的密码都在常见字典范围内，这类密码的恢复成功率相当高。

### Q: 网上那些"秒破解"的工具靠谱吗？

需要警惕。很多号称"秒破解"的软件实际是恶意软件或钓鱼程序。真正的密码恢复需要大量计算，不存在"秒破"。推荐使用开源工具或可信平台。

### Q: 文件太大有几个 GB 怎么办？

可以用 [猫密网](https://www.catpasswd.com) 的特征提取工具（Catpasswd-Convert.exe），在本地提取加密特征后只上传几 KB 的特征文件，既安全又省时间。

## 总结

压缩包密码恢复并不是什么黑魔法，核心就是用算力 + 字典去穷举。选择哪种方法取决于你的技术水平和时间成本：

- **技术高手**：Hashcat + 好显卡，自己搞定
- **普通用户**：ARCHPR 或云端服务（如 [猫密网](https://www.catpasswd.com)）更省心
- **免费优先**：先用 John the Ripper 试试，不行再考虑其他方案

无论选择哪种方式，建议先从回忆密码线索开始，哪怕一点点线索都能让破解效率提升数倍。
