# HOOK

> HOOK 技术是通过对程序进行一些修改从而达到监控程序在运行到某个预期位置时实现我们想要的逻辑。

## HOOK 技术的分类

### 代码修改型 HOOK

1. [Inline Hook](./inline_hook.md)
    - Inline Hook 通过修改目标函数的前几条指令跳转至自己的代码，再执行完后执行被覆盖指令并跳回原函数继续执行。
2. [GOT Hook](./got_hook.md)
    - GOT Hook 通过修改目标函数的 GOT 表项指向自己的代码，从而实现 hook。
3. [HWBP Hook](./hwbp_hook.md)
    - HWBP Hook 通过设置硬件断点，当程序执行到断点时触发异常，从而实现 hook。
4. [PTE Hook](./pte_hook.md)
    - PTE Hook 通过修改目标函数的页表项指向自己的代码，从而实现 hook。