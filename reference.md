参考资料统一归档文档

# Docker Official Security Document
https://docs.docker.com/engine/security/

## AppArmor
https://docs.docker.com/engine/security/apparmor/

## Seccomp
https://en.wikipedia.org/wiki/Seccomp

https://docs.docker.com/engine/security/seccomp/

# Docker Source Code analysis
https://jimmysong.io/blog/docker-source-code-analysis-code-structure/

docker 源码翻阅

# Main Reference
http://cn-sec.com/archives/157104.html
https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-breakout/docker-breakout-privilege-escalation/docker-release_agent-cgroups-escape

主要的参考文章, 我认为是写的最棒的一篇文章.

# Linux Kernel 
## Cgroups Doc
https://github.com/torvalds/linux/blob/5bfc75d92efd494db37f5c4c173d3639d4772966/Documentation/admin-guide/cgroup-v1/cgroups.rst

## Overlayfs 
https://www.kernel.org/doc/html/latest/filesystems/overlayfs.html

## tmpfs
https://www.kernel.org/doc/html/latest/filesystems/tmpfs.html

## /proc
https://www.kernel.org/doc/html/latest/filesystems/proc.html?highlight=proc

# Hacktricks 
## Capabilities
https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities

## Docker Escape 
https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-breakout/docker-breakout-privilege-escalation

## Docker Basic
https://book.hacktricks.xyz/network-services-pentesting/2375-pentesting-docker#docker-basics

# Man Page 
## Capabilities
https://man7.org/linux/man-pages/man7/capabilities.7.html

## Xattr
https://man7.org/linux/man-pages/man7/xattr.7.html

# BlackHat 2019 Docker Escape 
Video: https://www.youtube.com/watch?v=BQlqita2D2s

Demo PPT: https://i.blackhat.com/USA-19/Thursday/us-19-Edwards-Compendium-Of-Container-Escapes-up.pdf

# Container Cgroup FileSystem Namespace
http://www.zyixinn.com/?p=854

## Cgroup
http://119.23.219.145/posts/%E5%AE%B9%E5%99%A8-cgroup-%E6%95%B4%E4%BD%93%E4%BB%8B%E7%BB%8D/

https://arthurchiao.art/blog/cgroupv2-zh/

## Namespace
https://lwn.net/Articles/531114/

# RDMA
https://www.kernel.org/doc/Documentation/cgroup-v1/rdma.txt

# SeLinux
https://wiki.centos.org/zh/HowTos/SELinux

# Usermod helper
https://developer.ibm.com/articles/l-user-space-apps/