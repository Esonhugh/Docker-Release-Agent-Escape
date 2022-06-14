命名空间 - linux 隔离方法

[[reference#Container Cgroup FileSystem Namespace]]

Linux namespaces 是对全局系统资源的一种封装隔离, 使得处于不同 namespace 的进程拥有独立的全局系统资源, 改变一个 namespace 中的系统资源只会影响当前 namespace 里的进程, 对其他 namespace 中的进程没有影响. 是轻量容器的使用的技术之一.

在 linux 系统中主要的目的是 隔离进程资源 

具体隔离的内容分为 6 项

- Mount 
> 隔离文件系统挂载点
- UTS
>隔离主机名，实际隔离 nodename 和 domainname
- IPC
> 隔离进程间通信、信号量和消息队列等
- PID
> 进程编号 具体体实现是 不同的空间有一样的 PID 例如 Docker 中的 ps pid=1 的进程为 entrypoint
- Network
> 隔离网络包括网络接口、驱动、路由表、防火墙等
- User
> 隔离 UserID 和 GroupID 以及其对应的能力

可以简单的通过 `ls -al /proc/$$/ns` 来进行查看当前所属的文件命名空间, 同样也可以用于其他进程的命名空间检查

其就相当于一个大筛子 把进程可以用到的系统资源梳理并且隔离
