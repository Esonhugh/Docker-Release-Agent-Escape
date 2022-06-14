在 cgroup-v1.c  的注释提示到 call_usermodehelper()  

```
 * The final arg to call_usermodehelper() is UMH_WAIT_EXEC, which
 * means only wait until the task is successfully execve()'d.  The
 * separate release agent task is forked by call_usermodehelper(),
 * then control in this thread returns here, without waiting for the
 * release agent task.  We don't bother to wait because the caller of
 * this routine has no use for the exit status of the release agent
 * task, so no sense holding our caller up for that.
```

从侧面可以看出是一种 执行命令的函数 类似于完全异步 的 execve

并且 call_usermodehelper 是外源的 extern 的 function 所以 我并不能跟踪下去 

此外这个函数并没有限制权限之类相关的信息, 所以可以猜出, 只能是 以一种全 cap 的最高 root 权限执行.