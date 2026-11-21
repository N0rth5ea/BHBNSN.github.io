# 常见 hook 的检测与对抗

## 通用检测手段

### 1. crc 内存完整性校验

## Xpose 框架的检测与对抗

### 1. 检测 Xpose 包名

通过检测是否安装了 Xpose 框架的安装器来判断是否存在 Xpose 框架

```java
PackageManager packageManager = context.getPackageManager();
List applicationInfoList = packageManager.getInstalledApplications(PackageManager.GET_META_DATA);
for (ApplicationInfo applicationInfo: applicationInfoList){
        if(applicationInfo.packageName.equals("de.robv.android.xposed.installer")){
        // is Xposed
        }
}
```

通常情况下使用 `Xposed Installer` 框架都会屏蔽对其的检测，即 Hook 掉 `PackageManager` 的 `getInstalledApplications` 方法的返回值，以便过滤掉 `de.robv.android.xposed.installer` 来躲避这种检测。

### 2. 反射检测

### 3. 异常堆栈检测

### 4. java 方法变为 native 方法

### 4. 加载库列表检测
通过读取 `/proc/self/maps` 来检测是否存在 Xpose 框架的库文件。Xpose 框架无法 hook Native 方法，通过这种方式可以检测不容易被 Xpose 隐藏。

## 侧信道检测

### 1. syscall 调用时长异常

可以写成博客里的这种介绍格式：

### 1. 基于 syscall 耗时差的 Root/Hook 环境检测

反复调用一个更容易被 Root 隐藏或内核 Hook 逻辑拦截的敏感 syscall（如 `faccessat`），再调用一个通常不会被重点处理的基线 syscall（如 `fchownat`），统计两者的耗时分布差异。

之所以选择 `faccessat`，是因为很多 Root 检测都会通过它判断 `su`、Magisk 等敏感文件或路径是否存在，因此它常常成为 Root 隐藏框架、内核模块或 Hook 逻辑的重点拦截对象；而 `fchownat` 一般不承担这类检测功能，通常可作为相对干净的对照基线。为了减少正常文件系统访问带来的噪声，检测时往往会故意传入非法参数，使 syscall 尽快在参数检查阶段返回错误，从而突出额外 Hook 逻辑带来的时间开销。

如果在大量重复测试后，`faccessat` 的耗时长期显著高于 `fchownat`，就说明其调用路径中可能插入了额外的过滤、检查或伪造返回值逻辑。因此，这种方法本质上是一种**基于时间侧信道的环境检测技术**，可用于识别设备中是否存在 Root 隐藏、内核 Hook 或其他运行时拦截机制。需要注意的是，它检测到的并不是“设备一定已 Root”，而是“当前环境疑似存在 Root 相关对抗或内核层插桩行为”。

### 2. 函数中插入计时器


!!! quote "参考链接"
    * [Android Hook技术防范漫谈](https://tech.meituan.com/2018/02/02/android-anti-hooking.html){target="_blank" rel="noopener"}
