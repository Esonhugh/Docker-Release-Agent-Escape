refer [[reference#Man Page#Capabilities]]

CAP是针对**进程**来说的 也可以通过运行程序继承当前的环境来确定当前环境的

其他的目前可以不看 

主要概述一下 `CAP_SYS_ADMIN`
 这个是属于一种全新的 Root 用于取代之前几个版本的 Root 的权限 
 
 这些 Cap 主要从不同方面分割了 最高的 Root 特权, 将所有需要对应**特权** 的 syscall 组合成一个 CAP, 然后我们就可以只将进程分配给它们需要的子集. 因此, 内核调用被分成了几十个不同的类别. 但是 CAP_SYS_ADMIN 仍然拥有全部的权限, 也就是之前的 ROOT 。

可以非常简单的使用  `cat /proc/$$/status` 来进行查看所属的标识 

 > Sounds Like [[RBAC]]. 非常类似的思想