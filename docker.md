# what's Docker

与正常理解不同的是 当借助 Docker 时 您可将容器当做轻巧模块化的虚拟机使用. 同时,您还将获得高度的灵活性, 从而实现对容器的高效创建 部署及复制, 并能将其从一个环境顺利迁移至另一个环境. Docker 在很多的开发者手中, **尤其是云原生开发者**, 可以用来发布/版本控制/快速部署他们的应用. 

**但是 Docker 不是 Virtual Machine** 在隔离方面 Docker 和 虚拟机差别实在太大了.

同时 Docker 也可以用来解决类似 "python 怎么在你这里跑的起来, 在我这里连依赖都冲突" 的尴尬问题. 

很多时候提到 Docker 就要和 LXC 容器 以及 虚拟机 进行一次 battle.

这里就不进行说明了 可以直接参考文档

简单的说明就是 
- LXC 更为轻量的容器实现 基于 [[cgroup]] 与 [[namespace]] 
- docker 原先基于 LXC 但是后面渐渐脱离. 除了运行容器之外，Docker 技术还具备其他多项功能, 包括简化用于构建容器/传输镜像以及控制镜像版本的流程 更容易使用和管理 
- 虚拟机是更高级的处理. 操作系统的内核(kernel)都是被完全独立出去了 但是资源消耗更大

> 正因为容器中是共享 kernel 的 并没有如虚拟机一般隔离 所以对于 kernel 的共享信息 例如共享的内存 可以进行提权逃逸

> LXC 和 虚拟机这里不进行多谈 

## 基本信息
[[reference#Hacktricks#Docker Basic]]

采用文件系统 [[overlayfs]]

采用类似于前后端分离的方法操控 

用户或者管理员使用 docker cli 或者 其他炫酷 UI 的 docker 应用程序进行操作 "前端"

实际上发号施令进行管理控制的 "后端" Docker Engine

早期版本中 docker 是使用 http 监听的

由于暴露在 [[SSRF]] 的攻击下 现在版本已经采用了 unix 本地套接字

## 安全策略
使用 命令行 flag `security-opt` 进行指定

### 初级隔离方式
[[namespace]] 可以认为是一种 大网眼大筛子筛选出来的是大概的设备
[[cgroup]] 可以认为是一种 精细的小筛子筛选出来更加精细的控制
这是我认为两种隔离手段上的不同点
### 高级访问控制隔离方法
[[apparmor]]
[[SELinux]]
[[seccomp]]

# 特权模式
[[reference#Docker Official Security Document]]
- 直接禁用 [[SELinux]]
- 可以访问很多的硬件 /dev 目录下会有更多内容 比如非常致命的 sda1
- 以非只读形式访问到 [[cgroup]] 可以修改内容
- 有 tmpfs 形式挂载的  [[proc]] 目录  普通情况下无 不可供给读写
- 获得所有的 linux [[capability]] 

