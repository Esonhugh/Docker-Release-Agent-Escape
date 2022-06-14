
cgroup 是一个控制一堆任务 并且抽象出一个大环境的功能

称之为 Control group

其目的在内核中是 精确分配资源 例如 CPU 等等

linux 文档 - [[reference#Linux Kernel#Cgroups Doc]] 中详细说明了入下不封
- 定义 cgroup 内容
-> 啥是 cgroup
- 基本使用方法和语法
->这里阐述了如何使用 相关的 api 
- 内核 API 
->主要偏向于内核开发者
- 扩展属性
-> 我更愿意称之为更加细粒度的权限属性的控制.参考 [[extend attr]] 与 [[capability]]
- QA
-> 这里可以不管


[[cgroup - kernel code]]
## release_agent
存放于 cgroup 文件夹的 release_agent 文件中
释放代理 是存放一组程序或者脚本的文件 在 cgroup 任务全部结束后 由内核进行进行操作

可以认为是一种回调函数

通常位于 顶层子系统 [[rdma]]

## notify_on_release

参考 linux 文档中 [[reference#Linux Kernel#Cgroups Doc]] 1.4 标题 

>每当 cgroup 中的最后一个任务离开(退出或附加到其他某个 cgroup)并且该 cgroup 的最后一个子 cgroup 被删除时，内核就会运行由该层次结构的根目录中的 “release_agent” 文件内容指定的命令，提供结束的 cgroup 的路径名 (相对于 cgroup 文件系统的挂载点)。这使得可以自动删除被遗弃的 cgroups。
>系统引导时根 cgroup 中 notify_on_release 的默认值为禁用(0)。其他组在创建时的默认值是其父组 notify_on_release 设置的当前值。Cgroup 层次结构的 release_agent 路径的默认值为空。

可以认为是 是否执行回调的 flag