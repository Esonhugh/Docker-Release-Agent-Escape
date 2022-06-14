refer: [[reference#Docker Official Security Document#Seccomp]]

Docker 安全策略之一 linux kernel 安全组件之一

使用基于黑白名单进行的系统调用的管控 

在 Docker 中禁止 mount reboot setns 等 44 项系统调用 详情参看 docker 官方文档 (refer 第二条)
