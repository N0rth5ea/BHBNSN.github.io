# System Server 启动

## 1. System Server 启动

在 `ZygoteInit` 中传入了 `system server` 的类名 `com.android.server.SystemServer`

```java title="/frameworks/base/core/java/com/android/internal/os/ZygoteInit.java"
// line 721:732
--8<-- "docs/study/android/source/frameworks/base/core/java/com/android/internal/os/ZygoteInit.java:721:732"
```

从 `main` 启动了 `run` 方法：

```java title="/frameworks/base/services/java/com/android/server/SystemServer.java"
// line 685:690
--8<-- "docs/study/android/source/frameworks/base/services/java/com/android/server/SystemServer.java:685:690"
```
在 `run` 中，主要做了以下几件事：
```text
1. 开始 boot trace
2. 记录 system_server 启动属性和日志
3. 修正时区、语言、VM、SQLite、Binder 等全局环境
4. 放开 system_server 堆限制
5. 准备 system_server 主线程 Looper
6. 启动初始化线程池
7. 加载 libandroid_servers.so
8. 检查 pending shutdown
9. 创建 system context
10. 注册 system_server_dumper
11. 创建 SystemServiceManager
12. 设置 WTF handler
13. 初始化 ApplicationSharedMemory
14. startBootstrapServices()
15. startCoreServices()
16. startOtherServices()
17. startApexServices()
18. 更新 Watchdog
19. 记录 system_server ready
20. 设置 Binder transaction callback
21. 注册 GC 后内存统计回调
22. Looper.loop() 永久运行
```
下面我们详细讲解一些比较关键的步骤（不全，在后续学习中遇到会不定期更新）

