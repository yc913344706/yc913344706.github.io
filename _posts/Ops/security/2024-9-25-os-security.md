---
layout: post
title: 【系统安全】01 杀毒软件
categories: [Ops, Security]
tags: []
---

## linux 杀毒软件

文档
- [6 款适用于 Linux 的最佳免费杀毒软件](https://zhuanlan.zhihu.com/p/514221945)
- [在 EC2 Linux 操作系统上部署 ClamAV 并开启实时防护、集中日志采集和统一告警](https://aws.amazon.com/cn/blogs/china/deploy-clamav-on-ec2-with-realtime-scan-and-centralized-alarm/)
- [计算机病毒原理、类型与实验](https://blog.csdn.net/qq_39428589/article/details/134427770)
- [木马后门入侵与RKHunter，ClamAV检测工具](https://www.cnblogs.com/liujianxin/p/12599908.html)


| 软件           | 用途                     | 主要功能                       | 优点                        | 缺点                        |
|----------------|-------------------------|------------------------------|-----------------------------|----------------------------|
| ClamAV         | 开源查毒软件              | 病毒、木马、恶意软件            | 免费、开源、社区支持强大        | 实时保护功能有限              |
| Chkrootkit     | Rootkit检测              | 检测已知Rootkit和可疑行为       | 轻量级、命令行工具             | 仅检测，不提供修复功能         |
| Comodo         | 综合安全解决方案           | 实时防护、恶意软件检测、防火墙    | 功能全面，有免费版本           | 界面复杂，资源消耗较高         |
| Sophos         | 企业级安全解决方案         | 恶意软件检测、网络安全、设备管理   | 企业管理功能强大、跨平台支持    | 主要面向企业用户，个人版功能有限 |
| Rootkit Hunter | Rootkit检测              | 检测Rootkit、后门和本地漏洞      | 易于使用、定期更新            | 仅检测，不提供自动修复功能      |
| F-PROT         | 病毒和恶意软件检测         | 病毒扫描、防病毒保护             | 轻量级、易于使用              | 实时保护功能有限，更新较慢      |
| OSSEC          | 入侵检测系统              | 日志分析、完整性检查、实时告警     | 开源、跨平台、多层次安全监控    | 配置复杂，需要一定技术知识      |


## rootkit

- Rootkit是一种恶意软件，
- 目的：
  - 旨在让黑客访问和控制目标设备。
  - rootkit 是网络犯罪分子用来控制目标计算机或网络的软件。
- 展现形式：
  - Rootkit 有时可以显示为单个软件，但通常由一组工具组成，这些工具允许黑客对目标设备进行管理员级别的控制。
  - 尽管大多数 rootkit 会影响软件和操作系统，但有些 Rootkit 也会感染计算机的硬件和固件。
- 具有极高的隐蔽性：
  - 设计目的是在受感染的系统中隐藏其自身或其他恶意软件的存在。
- 命名：
  - “rootkit”这个名字来源于Unix和Linux操作系统，
  - 其中最特权的帐户管理员被称为“root”。允许未经授权的根或管理员级别访问设备的应用程序称为“kit”。
- 拥有root权限：
  - Rootkit通常会获得系统的管理员权限，以便在系统中隐蔽地执行操作。
- 安装方式：
  - 最常见的是通过网络钓鱼或其他类型的社会工程攻击。受害者在不知不觉中下载并安装隐藏在其计算机上运行的其他进程中的恶意软件，并让黑客控制操作系统的几乎所有方面。
  - 另一种方法是利用漏洞（即软件或操作系统中尚未更新的弱点）并将rootkit强制到计算机上。
  - 恶意软件还可以与其他文件捆绑在一起，例如受感染的 PDF、盗版媒体或从可疑第三方商店获得的应用程序。

## clamav安装

文档：
- [Centos7下杀毒软件clamav的安装和使用](https://www.cnblogs.com/ghl1024/p/9018212.html)
- 安装：
  - [clamav End of Life (EOL) Policy](https://docs.clamav.net/faq/faq-eol.html)
  - [官方安装文档](https://docs.clamav.net/manual/Installing/Installing-from-source-Unix.html)
  - [官方docker安装](https://docs.clamav.net/manual/Installing/Docker.html#building-the-clamav-image)
- 使用：
  - [Dakuzo + Clamav 实现 Linux 系统 FileSystem 实时病毒防护](https://blog.sinzy.net/@pcman/entry/7117)

### centos 7

文档：

## 使用

### 更新病毒库

```bash

mkdir -p /var/log/clamav /usr/local/clamav/{updata,quarantine} && \
    chown 101:102 /usr/local/clamav/{updata,quarantine} && \
    chmod 700 /usr/local/clamav/quarantine



```

### 配置自动任务

```bash
cat > /usr/local/clamav/bin/cron_scan.sh << EOF
#!/bin/bash

TIMESTAMP=`date "+%Y-%m-%d %H:%M:%S"`

mkdir -p /usr/local/clamav/quarantine/${TIMESTAMP}

/usr/local/clamav/bin/clamscan \
    -r / \
    --move=/usr/local/clamav/quarantine/${TIMESTAMP} \
    -l /var/log/clamav/clamscan_${TIMESTAMP}.log

EOF

chmod +x /usr/local/clamav/bin/cron_scan.sh

crontab -e
#让服务器每天晚上定时更新和杀毒，保存杀毒日志，crontab文件如下：
1  3  * * *  /usr/local/clamav/bin/freshclam --quiet
20 3  * * *  /usr/local/clamav/bin/cron_scan.sh
```

