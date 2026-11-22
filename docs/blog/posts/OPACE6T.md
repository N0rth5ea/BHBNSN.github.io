---
date:
  created: 2026-09-06
  updated: 2026-11-22
---

# 一加 ACE 6T GhostLock + GBL 漏洞持久化 Root 的方法及原理分析

## 背景

新买了一加 ACE 6T 手机，想要刷机，懒得等深度测试的申请，发现了可以通过 GhostLock + GBL 漏洞，在不走官方 Bootloader 解锁流程的情况下，实现持久化 Root 的方案。

<!-- more -->

目前网上已经出现了大量一键自动化利用 App 和模块。笔者当时还是手动利用、刷写的，实际操作过程没有太大参考价值，因此本文不放具体操作步骤，主要梳理完整链路和原理。

> 声明：本文仅用于个人设备的技术研究和安全学习。

## 原理

### 整体链路

首先明确我们的最终目标：**让修补后具有 root 能力的 `init_boot` 能够在 Bootloader 锁定状态下被接受并正常启动。**

传统的 root 中，刷写和校验 init_boot 分区都需要 bootloader 解锁，而在不解锁的情况下，我们有两个独立的核心问题：

1. 如何写入 init_boot 分区？
2. 如何让 bootloader 接受修改后的引导镜像？

而这对应了我们要利用的两个漏洞：

- GhostLock 让我们可以临时提权，获取 root 权限，从而直接将修补后的镜像文件覆写到 /dev/block/by-name/init_boot 分区。
- GBL/EFISP 让我们可以绕过 bl 对 init_boot 的校验，从而在手机启动时接受修补的 init_boot。

至此，万事具备，链路如下：

1. 利用 ghostlock 漏洞获取一次性 root 权限
2. 利用 root 权限刷写低版本 abl 分区，确保 gbl 漏洞可用
3. 利用 root 权限刷写 efisp 分区，利用 gbl 漏洞获取 bootloader 校验放行
4. 重启 recovery 并格式化数据
5. 再次利用 ghostlock 漏洞获取一次性 root 权限
6. 利用 root 权限刷写 init_boot 分区，获取持久化 root 权限

---

### 1. GhostLock 漏洞

GhostLock 是今年公开的 Linux 内核提权漏洞，编号为 CVE-2026-43499，位于 Linux 内核的 futex / rt_mutex 优先级继承机制中，本质上是一个栈 UAF 漏洞，从 Linux 2.6.39-rc1 开始引入，直到 Linux 7.1 对应的修复版本才修复，潜伏时间超过 15 年。


#### 1.1 futex 与 rt_mutex

futex 是 Linux pthread_mutex 的实现，它实现了在用户态当中检测锁是否冲突，只用冲突时才进入内核，让冲突线程睡眠等待锁释放。

在实际操作系统当中有一个经典的优先级反转（Priority Inversion）问题：

假设有三个线程 A、B、C，它们的优先级分别为 High、Medium、Low

- C 作为 Low 优先级线程，但是持有锁
- A 作为 High 优先级线程，正在等待锁
- B 不碰锁，执行 cpu 密集操作

此时，由于 B 优先级比 C 高，导致 A 要等 C（锁），C 要等 B（优先级），A 虽然是优先级最高的线程，但是却必须等待优先级低一级的 B。

而这个问题在 Linux 内核中，通过 futex PI（Priority Inheritance，优先级继承）机制实现：

- 当高优先级线程等待低优先级线程持有的锁时，将其优先级传递给持有锁的线程上，从而保证上述案例中 A 不会被 B 给间接阻塞。

**普通的 `pthread_mutex` 走的是普通 futex，并不涉及 PI**。只有显式设置了 `PTHREAD_PRIO_INHERIT` 属性的互斥锁，才会走 `FUTEX_LOCK_PI` / `FUTEX_WAIT_REQUEUE_PI` 这条路径。所以这条代码路径在日常负载里其实相当冷门——这也是它能潜伏十五年的原因之一。


!!! note "参考资料"
    1. [GhostLock（CVE-2026-43499）Linux内核futex子系统15年潜伏的栈UAF提权漏洞深度拆解](https://www.cnblogs.com/32bin/p/21919461){target="_blank" rel="noopener"}
    2. [GhostLock（CVE-2026-43499）Linux 内核漏洞深度分析 - NebuSec](https://nebusec.ai/research/ionstack-part-2/){target="_blank" rel="noopener"}
    3. [futex 与内核同步原语底层：等待队列、唤醒机制与 PI mutex](https://www.plumephp.com/os-futex-locks-internals/){target="_blank" rel="noopener"}