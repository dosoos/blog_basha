---
title: "Word/Excel文档密码忘记了？加密Office文件恢复方法大全（2026最新）"
date: 2026-05-21T10:00:00+08:00
draft: false
image: ""
keywords: "Word密码忘记, Excel密码破解, Office加密文件, docx密码恢复, xlsx密码找回"
readingTime: true
categories: ['技术教程']
tags: ['Word', 'Excel', 'Office', '密码恢复', '文档加密']
description: "Word或Excel文档加密后忘记密码了？本文详解6种Office加密文件密码恢复方法，涵盖VBA宏破解、msoffcrypto工具、John the Ripper等专业方案。"
---

## 前言

Office 文档加密是职场中最常见的数据保护手段。根据微软的统计，全球有超过 3 亿用户每天使用 Office 办公套件。但当你给一份重要的 Word 合同或 Excel 报表设置了打开密码，过了一段时间却怎么也想不起来时，那种焦虑感相信很多人都体会过。

本文将系统介绍 Office 文档密码恢复的各种方法，覆盖 Word（.doc/.docx）、Excel（.xls/.xlsx）和 PowerPoint（.pptx）三大格式。

## 先搞清楚：Office 文档的两种密码

在开始之前，需要了解 Office 文档有两种不同类型的密码，破解难度完全不同：

| 密码类型 | 说明 | 破解难度 |
|----------|------|----------|
| **打开密码** | 打开文件时需要输入 | 较高（AES-128/256 加密） |
| **修改密码** | 打开可以查看，编辑时需要 | 极低（可以绕过） |

如果是"修改密码"，恭喜你，基本不需要破解。用 WPS 或 LibreOffice 打开文件，另存为新文件，修改密码就会丢失。本文主要讨论的是更难处理的"打开密码"。

## Office 文档加密算法的演变

了解加密算法有助于判断破解难度：

- **Office 97-2003 (.doc/.xls)**：使用 RC4 加密（40-bit 或 56-bit），存在已知漏洞，**破解相对容易**
- **Office 2007 (.docx/.xlsx)**：使用 AES-128 加密，安全性大幅提升
- **Office 2010+ (.docx/.xlsx)**：使用 AES-128 并增加了迭代次数（100,000 次 SHA-1），**破解速度慢很多**
- **Office 2016+**：默认使用 AES-256，是目前最强的加密标准

> 💡 **小贴士**：如果你的文件是 .doc 或 .xls 老格式，破解速度会比 .docx 快几十倍。

## 方法一：msoffcrypto-tool（Python 库，推荐入门）

[msoffcrypto-tool](https://github.com/nolze/msoffcrypto-tool) 是一个专门处理 Office 文件加密的 Python 库，可以检测加密类型并进行密码恢复。

### 安装

```bash
pip install msoffcrypto-tool
```

### 检测文件是否加密

```python
import msoffcrypto

with open("encrypted.docx", "rb") as f:
    file = msoffcrypto.OfficeFile(f)
    print(f"是否加密: {file.is_encrypted()}")
```

### 使用字典破解

```python
import msoffcrypto

def crack_office(file_path, wordlist_path):
    with open(file_path, "rb") as f:
        file = msoffcrypto.OfficeFile(f)
        
        if not file.is_encrypted():
            print("文件未加密")
            return
        
        with open(wordlist_path, 'r', encoding='utf-8', errors='ignore') as wf:
            for line in wf:
                pwd = line.strip()
                try:
                    if file.load_key(password=pwd):
                        print(f"✅ 密码找到: {pwd}")
                        
                        # 解密文件
                        with open("decrypted.docx", "wb") as out:
                            file.decrypt(out)
                        print("文件已解密保存")
                        return pwd
                except Exception:
                    continue
    
    print("❌ 字典中未找到密码")

crack_office("report.xlsx", "rockyou.txt")
```

### 优缺点

- **优点**：纯 Python，跨平台，代码简洁
- **缺点**：速度一般，不支持 GPU 加速

## 方法二：John the Ripper + office2john

John the Ripper 不仅支持压缩包，也支持 Office 文档的密码破解。

### 使用步骤

```bash
# 1. 提取 Office 文件的密码哈希
python /path/to/john/run/office2john.py encrypted.docx > hash.txt

# 输出类似：
# encrypted.docx:$office$*2013*100000*256*16*...（很长的哈希串）

# 2. 使用字典攻击
john --wordlist=rockyou.txt hash.txt

# 3. 查看结果
john --show hash.txt
```

### 不同 Office 版本的哈希模式

| 版本 | John 模式标识 | 破解速度参考 |
|------|--------------|-------------|
| Office 2003 | `$oldoffice$` | 快（约 100K/s） |
| Office 2007 | `$office$*2007*` | 中（约 10K/s） |
| Office 2010 | `$office$*2010*` | 慢（约 1K/s） |
| Office 2013+ | `$office$*2013*` | 很慢（约 500/s） |

> 以上速度参考基于 GTX 1080 显卡使用 Hashcat 的测试结果。

## 方法三：Hashcat GPU 加速破解

如果你有独立显卡，Hashcat 是破解 Office 密码最快的方案。

### Office 文档对应的 Hashcat 模式

```bash
# Office 2003 (.doc/.xls)
hashcat -m 9700 -a 0 hash.txt wordlist.txt

# Office 2007 (.docx/.xlsx)
hashcat -m 9400 -a 0 hash.txt wordlist.txt

# Office 2010 (.docx/.xlsx)
hashcat -m 9500 -a 0 hash.txt wordlist.txt

# Office 2013+ (.docx/.xlsx)
hashcat -m 9600 -a 0 hash.txt wordlist.txt
```

### 使用掩码攻击（已知密码格式）

如果你记得密码大概是 "公司名+年份" 的格式，可以用掩码缩小范围：

```bash
# 假设密码格式为 "MyCompany202X"
hashcat -m 9600 -a 3 hash.txt "MyCompany202?d"

# ?d = 数字 0-9，这样只需尝试 10 个组合
```

### 优缺点

- **优点**：GPU 并行计算，速度极快
- **缺点**：需要独显，配置复杂，Office 2013+ 版本由于高迭代次数，即使 GPU 加速也较慢

## 方法四：VBA 宏破解修改密码

如果文件只有"修改密码"（而非打开密码），可以用 VBA 宏直接绕过：

```vb
Sub RemoveWriteProtection()
    Dim doc As Document
    Set doc = ActiveDocument
    
    ' 移除写保护
    If doc.ProtectionType <> wdNoProtection Then
        doc.Unprotect
    End If
    
    MsgBox "修改密码已移除！"
End Sub
```

> ⚠️ **注意**：此方法仅对"修改密码"有效，对"打开密码"无效。

## 方法五：LibreOffice 绕过法

某些情况下，LibreOffice 对加密文件的处理方式和 Microsoft Office 不同：

1. 安装 [LibreOffice](https://www.libreoffice.org/)（免费开源）
2. 用 LibreOffice 打开加密的 .doc/.xls 文件
3. 部分旧格式文件可能不弹出密码框直接打开
4. 另存为新文件即可

**成功率**：仅对 Office 97-2003 格式的部分加密文件有效，对 .docx/.xlsx 基本无效。

## 方法六：在线云端恢复服务

对于不熟悉命令行操作的用户，或者本地算力不足以应对 Office 2013+ 的高强度加密时，**云端恢复服务**是最省心的选择。

**[猫密网 (Catpasswd)](https://www.catpasswd.com)** 支持 Word、Excel、PowerPoint 等 Office 文档的在线密码恢复：

### 为什么 Office 文件特别适合用云端服务？

Office 2010 之后的文档加密迭代次数非常高（10 万次 SHA-1），这意味着：

- 每尝试一个密码都需要大量的计算
- 普通电脑的 CPU 每秒可能只能尝试几百个密码
- 一个 100 万条的字典，本地跑可能需要几个小时甚至几天

而云端分布式集群可以并行处理，大幅缩短等待时间。

### 使用流程

1. 访问 [猫密网](https://www.catpasswd.com) → [恢复页面](https://www.catpasswd.com/recovery)
2. 上传加密的 Office 文件（Word/Excel/PPT，最大 100MB）
3. 填写邮箱地址
4. 等待系统自动恢复，成功后邮件通知

对于机密文件，可以使用平台的**特征提取工具**在本地提取文件特征后仅上传特征数据，源文件不会离开你的电脑，100% 保证文档安全。

### 费用说明

猫密网提供免费恢复模式，可以先测试能否恢复。如果免费版未能成功，可以升级到专业版使用更大的字典库和更快的计算资源。

## 各方法对比

| 方法 | 支持格式 | 速度 | 难度 | 适合场景 |
|------|---------|------|------|----------|
| msoffcrypto-tool | 全版本 Office | 中 | 低 | 程序员自用 |
| John the Ripper | 全版本 Office | 中 | 中 | 技术人员 |
| Hashcat | 全版本 Office | 快 | 高 | 有独显用户 |
| VBA 宏 | 仅修改密码 | 即时 | 低 | 仅修改密码场景 |
| LibreOffice | 旧版 .doc/.xls | 即时 | 低 | 碰运气 |
| [猫密网](https://www.catpasswd.com) | 全版本 Office | 快 | 极低 | 所有人 |

## 提高成功率的实用技巧

1. **先确定 Office 版本**：不同版本的加密算法差异很大，.doc 格式比 .docx 容易破解得多
2. **回忆密码规则**：很多人设置 Office 密码有固定模式，比如 "公司名+年份"、"姓名+手机号后四位" 等
3. **试试弱口令字典**：大量用户使用 "123456"、"password"、"qwerty" 等弱密码，先用常见密码字典试一遍
4. **Excel 的 .xls 特别注意**：旧版 Excel 的 40-bit RC4 加密存在漏洞，可以用更高级的密码分析攻击
5. **不要轻信"在线秒破"网站**：很多小网站会收集你的文件用于非法用途，选择有信誉的平台

## 常见问题

### Q: 文件只是设置了"只读保护"而非加密，怎么解除？

如果是 Word 的"限制编辑"或 Excel 的"工作表保护"，那不是真正的加密，可以通过以下方法解除：
- 将文件后缀改为 .zip，打开后找到 `word/settings.xml`，删除 `<w:documentProtection>` 标签
- 或用 VBA 宏解除（见方法四）

### Q: WPS 加密的文件和 Office 加密的一样吗？

WPS 兼容 Office 的加密格式，但部分 WPS 独有的加密方式使用了不同的算法。如果是 WPS 加密的文件，建议先用 WPS 另存为 .docx 格式再处理。

### Q: Excel 文件有"打开密码"和"工作簿保护"两层，怎么分别处理？

"打开密码"是文件级加密，需要本文介绍的方法破解。"工作簿保护"只是限制了结构修改（不能增删工作表），可以用 VBA 直接绕过。

### Q: 恢复出的密码能直接用在 WPS 上吗？

可以。WPS 完全兼容 Office 的加密格式，找到的密码在 WPS 和 Microsoft Office 上通用。

## 总结

Office 文档密码恢复的核心难点在于新版本 Office（2010+）的高迭代次数加密。对于旧版 .doc/.xls 文件，大部分方法都能较快恢复；对于新版 .docx/.xlsx 文件，建议使用 GPU 加速的 Hashcat 或借助 [猫密网](https://www.catpasswd.com) 这样的云端服务。

最重要的是，在加密文件之前，建议使用密码管理器妥善保存密码，避免日后给自己制造麻烦。
