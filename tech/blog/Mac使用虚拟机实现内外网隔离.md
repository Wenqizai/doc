工具：

- PD + Win 系统
- Win 安装 ccProxy + OpenVPN (其中 ccProxy 用来代理主机流量，OpenVPN 用来访问内网)
- Mac 安装 surge 

思路来源：https://linux.do/t/topic/122794

# 配置虚拟机

1. 虚拟机网络配置：网络 -> 桥接网络 -> Wi-Fi ；
2. 安装 ccProxy，仅开启 socks 和端口映射；
3. 勾选 ip 地址。 

# 配置 Surge

1. 配置策略代理，配置协议为 SOCKS5，服务器 ip: port 为 ccProxy 映射的 ip:port；
2. 新建规则，指定域名或 IP 为流量转发到指定的策略代理。



















