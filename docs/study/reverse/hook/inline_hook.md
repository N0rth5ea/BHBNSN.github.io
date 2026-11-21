# Inline Hook

!!! quote "参考链接"
    * [inline hook原理（x86平台实现）](https://www.cnblogs.com/luoyesiqiu/p/12306336.html){target="_blank" rel="noopener"}
    * [Inline Hooking for Programmers (Part 1: Introduction)](https://www.malwaretech.com/2015/01/inline-hooking-for-programmers-part-1.html){target="_blank" rel="noopener"}

## 什么是 Inline Hook

inline hook 通过修改函数开始的前几条指令来无条件跳转至自己的代码，从而实现 hook，大致流程如下：
```text
caller -> target_function
              |
              v
          hook_function
              |
              | 可选：调用 original trampoline
              v
          原始函数剩余逻辑 -> return
```

其中，由于我们的跳转指令会覆盖原始函数的前几条指令，所以我们需要将被覆盖的指令保存，并且跳转回原函数时，需要跳转至原函数中 hook 指令后的位置，这就是 trampoline 做的事
