---
title: "PDF文件加密了怎么解除？PDF密码破解完整指南（2026最新）"
date: 2026-05-20T10:00:00+08:00
draft: false
image: ""
keywords: "PDF密码破解, PDF加密解除, PDF密码忘记, PDF权限密码, PDF打开密码"
readingTime: true
categories: ['技术教程']
tags: ['PDF', '密码恢复', '加密文件', 'Adobe', '文档安全']
description: "PDF文件设了密码打不开或不能编辑？本文全面讲解PDF的两种密码类型（打开密码与权限密码），以及对应的破解方法，包括qpdf、pikepdf、John the Ripper等工具的使用。"
---

## 前言

PDF 是工作中最常用的文档格式之一。从电子合同到学术论文，从银行账单到政府公文，PDF 无处不在。而当一份重要的 PDF 文件被加密，而你恰好忘记了密码，那种无力感相信很多人都经历过。

好消息是，PDF 的加密体系虽然有多个版本，但其中不少存在已知的弱点。这篇文章将带你全面了解 PDF 加密机制，并给出实用的密码恢复方案。

## PDF 加密基础知识

### 两种密码：打开密码 vs 权限密码

PDF 的加密设计中包含两种不同层级的密码：

| 密码类型 | 英文名称 | 作用 | 安全性 |
|----------|----------|------|--------|
| **打开密码** (User Password) | User Password | 必须输入才能查看内容 | 高（文件内容被加密） |
| **权限密码** (Owner Password) | Owner Password | 限制打印、复制、编辑等操作 | **低（可以绕过）** |

**关键区别**：权限密码不加密文件内容本身，只是通过 PDF 阅读器来限制功能。这意味着即使不知道权限密码，也有方法绕过限制。而打开密码则对文件内容进行了真正的加密。

### PDF 加密版本演变

| 版本 | 加密方式 | 安全性 | 说明 |
|------|---------|--------|------|
| PDF 1.1-1.3 | RC4 40-bit | ❌ 极弱 | 几秒可破解 |
| PDF 1.4 | RC4 128-bit | ⚠️ 较弱 | 存在已知漏洞 |
| PDF 1.5-1.6 | RC4 128-bit (AES) | ⚠️ 中等 | 改进但仍有弱点 |
| PDF 1.7 Extension 3 | AES-128 | ✅ 较强 | Adobe Acrobat 9+ |
| PDF 2.0 | AES-256 | ✅✅ 很强 | ISO 32000-2 标准 |

## 一、权限密码（Owner Password）的绕过方法

权限密码是最容易处理的，有以下几种方案：

### 方法 1：使用 qpdf 一键移除（推荐）

[qpdf](https://github.com/qpdf/qpdf) 是一个免费的 PDF 转换工具，可以直接移除权限密码：

```bash
# 安装
# Ubuntu/Debian
sudo apt install qpdf

# macOS
brew install qpdf

# Windows - 下载安装包
# https://github.com/qpdf/qpdf/releases

# 移除权限密码（不需要知道原密码）
qpdf --decrypt input.pdf output.pdf
```

执行后，`output.pdf` 将不再有任何编辑、打印、复制限制。

### 方法 2：Python pikepdf 库

```python
import pikepdf

# 打开有权限密码的 PDF（不需要密码）
pdf = pikepdf.open("restricted.pdf")

# 保存为新文件（自动移除权限限制）
pdf.save("unrestricted.pdf")
print("✅ 权限密码已移除")
```

安装 pikepdf：

```bash
pip install pikepdf
```

### 方法 3：在线工具

- **ilovepdf.com** - 上传 PDF 即可解锁
- **smallpdf.com** - 免费额度有限
- **pdf2go.com** - 支持批量处理

> ⚠️ **隐私提醒**：在线工具会上传你的文件到第三方服务器，敏感文件不建议使用。如需在线处理，建议选择支持文件销毁的正规平台。

### 方法 4：Chrome 浏览器打印法

1. 用 Chrome 浏览器打开受权限密码保护的 PDF
2. Chrome 可能直接忽略权限限制显示内容
3. 按 Ctrl+P → 选择"另存为 PDF"
4. 新的 PDF 文件将不带任何限制

## 二、打开密码（User Password）的破解方法

打开密码对文件内容进行了真正的加密，需要使用密码恢复手段。

### 方法 1：pdfcpu + John the Ripper

```bash
# 1. 从 PDF 中提取密码哈希
# 使用 pdf2john 工具（John the Ripper 附带）
python /path/to/john/run/pdf2john.py encrypted.pdf > hash.txt

# 输出格式：
# encrypted.pdf:$pdf$2*3*128*-1060*1*16*...

# 2. 使用字典攻击
john --wordlist=rockyou.txt hash.txt

# 3. 查看结果
john --show hash.txt
```

### 方法 2：Hashcat GPU 加速

Hashcat 对不同版本的 PDF 有单独的模式编号：

```bash
# PDF 1.1-1.3 (RC4 40-bit) - 模式 10400
hashcat -m 10400 -a 0 hash.txt wordlist.txt

# PDF 1.4-1.6 (RC4 128-bit) - 模式 10500
hashcat -m 10500 -a 0 hash.txt wordlist.txt

# PDF 1.7 Extension 3 (AES-128) - 模式 10600
hashcat -m 10600 -a 0 hash.txt wordlist.txt

# PDF 1.7 Extension 8 (AES-256) - 模式 10700
hashcat -m 10700 -a 0 hash.txt wordlist.txt
```

### 各版本 PDF 的破解速度对比

以 GTX 3080 显卡为参考：

| PDF 版本 | Hashcat 模式 | 大约速度 | 破解 6 位数字密码时间 |
|----------|-------------|---------|---------------------|
| 1.1-1.3 | 10400 | ~50M/s | 不到 1 秒 |
| 1.4-1.6 | 10500 | ~10M/s | 约 100 秒 |
| 1.7 Ext 3 | 10600 | ~50K/s | 约 20 天 |
| 1.7 Ext 8 | 10700 | ~10K/s | 约 100 天 |

可以看到，PDF 版本越高，破解难度呈指数级增长。

### 方法 3：Python + pikepdf 字典爆破

```python
import pikepdf

def crack_pdf(pdf_path, wordlist_path):
    """使用字典尝试破解 PDF 打开密码"""
    with open(wordlist_path, 'r', encoding='utf-8', errors='ignore') as f:
        for i, line in enumerate(f):
            pwd = line.strip()
            try:
                pdf = pikepdf.open(pdf_path, password=pwd)
                print(f"✅ 第 {i+1} 次尝试，密码找到: {pwd}")
                
                # 保存解密文件
                pdf.save(f"decrypted_{pdf_path}")
                return pwd
            except pikepdf.PasswordError:
                continue
            except Exception as e:
                print(f"尝试 '{pwd}' 时出错: {e}")
                continue
    
    print("❌ 字典中未找到密码")
    return None

crack_pdf("secret.pdf", "rockyou.txt")
```

### 方法 4：PDFCrack（轻量级开源工具）

[pdfcrack](https://github.com/robins/pdfcrack) 是一个专门针对 PDF 的密码破解工具：

```bash
# 编译安装
git clone https://github.com/robins/pdfcrack.git
cd pdfcrack
make

# 使用字典破解
./pdfcrack -f encrypted.pdf -w wordlist.txt

# 暴力破解（指定字符集和长度范围）
./pdfcrack -f encrypted.pdf --minlen=1 --maxlen=6 -c "0123456789"
```

### 方法 5：云端在线恢复服务

对于不方便本地部署工具的用户，或者 PDF 版本较高（AES-128/256）本地算力不足的情况，可以考虑云端恢复服务。

**[猫密网 (Catpasswd)](https://www.catpasswd.com)** 支持部分加密类型的 PDF 文件密码恢复。使用流程非常简单：

1. 访问 [catpasswd.com](https://www.catpasswd.com) → [上传文件](https://www.catpasswd.com/recovery)
2. 上传加密的 PDF 文件
3. 填写邮箱等待通知
4. 系统利用云端算力自动尝试千万级密码组合

对于高度敏感的 PDF 文件（如合同、财报、法律文书），猫密网提供**本地特征提取工具**，你可以在自己的电脑上提取文件加密特征，仅上传特征数据（几 KB），源文件完全不会离开你的电脑。

> 💡 **小贴士**：猫密网提供免费恢复模式，可以先试试能不能找回，不成功不收费。

## PDF 密码破解的实战技巧

### 1. 先确定 PDF 版本

```bash
# 查看 PDF 版本和加密信息
qpdf --show-encryption encrypted.pdf

# 或使用 Python
import pikepdf
pdf = pikepdf.open("encrypted.pdf", password="")  # 如果有打开密码需要填入
print(pdf.Root.Version)
```

了解版本后，可以针对性选择破解策略：低版本 PDF 可以用更高效的攻击方式。

### 2. 利用 PDF 的已知漏洞

PDF 1.4（RC4 128-bit）有一个著名的漏洞：权限密码和打开密码使用相同的加密密钥。这意味着如果你能绕过权限密码（用 qpdf），实际上就获得了文件内容。

### 3. 常见的 PDF 密码模式

很多人设置 PDF 密码时会用以下模式：

- 纯数字（如身份证号、手机号、生日）
- 公司名/文件名 + 年份
- "password"、"123456" 等弱口令
- 项目代号或合同编号

建议先用这些常见模式生成自定义字典，比盲目使用通用字典效率高得多。

### 4. 银行/政府类 PDF 的特殊处理

银行账单、税务文件等通常使用数字密码（如身份证后6位、手机号后4位）。可以先用纯数字字典尝试：

```python
# 生成 6 位纯数字字典
with open("digits6.txt", "w") as f:
    for i in range(1000000):
        f.write(f"{i:06d}\n")
```

## 各方法对比总结

| 密码类型 | 方法 | 工具 | 难度 | 速度 |
|----------|------|------|------|------|
| 权限密码 | qpdf 移除 | qpdf | 极低 | 即时 |
| 权限密码 | pikepdf 保存 | Python | 低 | 即时 |
| 权限密码 | Chrome 打印 | 浏览器 | 极低 | 即时 |
| 打开密码 | John the Ripper | 命令行 | 中 | 中 |
| 打开密码 | Hashcat | GPU | 高 | 快 |
| 打开密码 | pdfcrack | 命令行 | 中 | 中 |
| 打开密码 | [猫密网](https://www.catpasswd.com) | 网页 | 极低 | 快 |

## 常见问题

### Q: PDF 密码和 Word 密码哪个更难破解？

一般来说，高版本 PDF（AES-256）比 Word 2013+ 更难破解，因为 PDF 的密钥派生函数（KDF）迭代次数更多。但低版本 PDF（RC4 40-bit）比任何版本的 Word 都容易得多。

### Q: Adobe Acrobat 能帮忙找回密码吗？

不能。Adobe Acrobat 只能设置和管理密码，没有"找回密码"功能。如果你忘记了打开密码，只能通过本文介绍的破解方法。

### Q: 为什么有的 PDF 在线解锁工具不工作？

在线解锁工具（如 ilovepdf）只能移除权限密码（Owner Password），对打开密码（User Password）无效。如果文件有打开密码，必须使用密码恢复工具。

### Q: 扫描版 PDF 和文字版 PDF 在加密上有什么区别？

加密方式没有区别，但扫描版 PDF 文件通常更大（因为包含图片），上传到云端服务时可能需要注意文件大小限制。超过 100MB 的文件可以使用 [猫密网](https://www.catpasswd.com) 的特征提取工具来处理。

## 总结

PDF 密码破解的难度取决于 PDF 版本和加密算法：

- **权限密码**：用 qpdf 或 pikepdf 一键移除，非常简单
- **打开密码 + 低版本 PDF**：用 John the Ripper 或 pdfcrack，速度很快
- **打开密码 + 高版本 PDF**：建议使用 Hashcat GPU 加速或 [猫密网](https://www.catpasswd.com) 等云端服务

最好的策略永远是：设置密码时就记录在安全的地方（如密码管理器），避免日后给自己制造麻烦。但如果已经忘记了密码，希望本文的方法能帮到你。
