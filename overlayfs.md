[[reference#Linux Kernel#Overlayfs]]
[[reference#Container Cgroup FileSystem Namespace]]

 Docker 使用的文件系统底层

正如他的名字 overlayfs 覆盖层文件系统

是一层一层覆盖上去的 然后满足 
- 层层叠加
- 底层被上层的同名文件覆盖 目录则合并 不同名则上传到
- 以拷贝形式上传 
- 底层只读 只在 merge 层无论如何操作都无所谓 不会影响底层

这种每一层就像一个 git 提交一样 可以还原 分离 合并 确实是广受好评

对于有相同历史的层 就无需多次复制 直接依赖就行 

非常的方便和实用

