# XMUSSL-docker
由于厦门大学SSLVPN（EasyConnect）较差的兼容性和规范性，故使用“docker容器+服务器转发”的方法实现内网访问。

本方法参考资料将在尾部统一列出。

EasyConnect来自[@shmille](https://github.com/shmilee) 提供的[命令行版客户端 deb 包](https://github.com/shmilee/scripts/releases/download/v0.0.1/easyconn_7.6.8.2-ubuntu_amd64.deb)。
## 准备工作

 · 拥有公网IP的服务器（Debian系）

 · Docker
 
 · Clash
