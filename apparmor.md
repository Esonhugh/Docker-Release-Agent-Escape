官网: https://apparmor.net/

Wiki: https://en.wikipedia.org/wiki/AppArmor

基于 Linux kernel 安全模块 [[LSM]]

可以与 [[SELinux]] 相互替代

from Wiki:
> 当时, AppArmor 被称为 SubDomain，是指将特定程序的安全配置文件分割成不同域的能力，程序可以在这些域之间动态切换

分别强控制管控文件许可 网络使用 系统能力, 它们力求在系统范围内强制执行控制系统上每个程序可以执行的操作和资源的策略

与 Docker 相关的安全方面主要取决于 AppArmor 采用的容器策略配置 但是这种配置策略的具体内容 对于容器内的用户(通常指攻击者)其实是不得而知的 只能通过枚举多个可能的 Weakness point 来发现配置错误

其具体配置方案可以参照 [[reference#Docker Official Security Document#AppArmor]]

docker 默认为 

**docker-default profile Summary**:

-   **Access** 所有的 **networking**
-   没有 **capability** 被定义
> 但是默认的 abstractions/base 中 caps 是有引入的
-   **Writing** to any **/proc** file is **not allowed**
> 写入 /proc 是被禁止的 但是 如果 proc 不是在 /proc 下是可以允许的
-   Other **subdirectories**/**files** of /**proc** and /**sys** are **denied** read/write/lock/link/execute access
> proc sys 的子目录都是禁止 读写锁链接执行 的
-  **Mount** is **not allowed**
> 禁止挂载操作 
-   **Ptrace** can only be run on a process that is confined by **same apparmor profile**
> ptrace 是只允许在相同 apparmor 策略下的程序才能进行 跟踪

容器里一般情况是看不到 apparmor 的 否则可以尝试修改策略然后重载使得失效

此外具有 shebang bug 可以绕过其检测