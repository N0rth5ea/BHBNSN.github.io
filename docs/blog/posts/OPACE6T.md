---
date:
  created: 2026-09-06
  updated: 2026-11-22
---

# 一加 ACE6T ghostlock + gbl 漏洞持久化 root 的方法及原理分析

> 声明：本文档仅用于技术研究和学习交流，禁止用于任何非法用途。
> 目前网上已经出现了大量一键自动化利用 app 和模块，笔者当时还是手动利用、刷写的，故实际利用没有什么参考价值，故不放实际操作了，只梳理完整链路和原理分析。

## 背景

新买了一加 ACE6T 手机，想要刷机，懒得等深度测试的申请，发现了可以通过 ghostlock + gbl 漏洞在不解锁 bootloader 的情况下持久化获取 root 权限。

## 原理

### 整体链路

我们最终的目的是要获取 root 权限，也就是要刷写 init_boot 分区

刷写 init_boot 分区需要满足两个条件：
1. 需要有 root 权限
2. 需要 bootloader 成功校验

在常规下，bootloader 需要解锁才会放行非法签名的 init_boot，但是 gbl 漏洞实现了可以通过非法签名的 init_boot

gbl 存储于 efisp 分区，因此利用这个漏洞我们需要刷写 efisp 分区，同时新版本系统的 abl 可能修复了这个漏洞，我们还需要刷写低版本 abl

刷写这些分区都需要 root 权限，而 ghostlock 漏洞为我们提供了一次性的 root 权限

至此，万事具备，链路如下：

1. 利用 ghostlock 漏洞获取一次性 root 权限
2. 利用 root 权限刷写低版本 abl 分区，确保 gbl 漏洞可用
3. 利用 root 权限刷写 efisp 分区，利用 gbl 漏洞获取 bootloader 校验放行
4. 重启 recovery 并格式化数据
5. 再次利用 ghostlock 漏洞获取一次性 root 权限
6. 利用 root 权限刷写 init_boot 分区，获取持久化 root 权限

### 1. ghostlock 漏洞

ghostlock 是今年爆出的多个“核弹级” Linux 提权漏洞之一，是一个栈 UAF 漏洞，自2.6.39-rc1的 commit：8161239a8bcce9ad6b537c04a1fa3b5c68bae693引入，潜伏存在了 15 年。

