# XMUSSL-docker
不在本地运行厦门大学SSLVPN（EasyConnect），使用“docker容器+服务器转发”的方法实现内网访问。

本方法参考资料将在尾部统一列出。

## 准备工作

 * 拥有公网IP的服务器（以Debian系为例）

 * Docker
 
 * [Clash](https://github.com/Fndroid/clash_for_windows_pkg/releases)
 
## 卸载Docker

### 1. 删除残留Docker,官网给出的两条命令，当然为了确保你删的干净点，请看第二条。

```
sudo apt-get purge docker-ce docker-ce-cli containerd.io
sudo rm -rf /var/lib/docker
```

### 2. 更彻底的删除Docker

#### 1) 常规删除操作

```
sudo apt-get autoremove docker docker-ce docker-engine docker.io containerd runc
```

#### 2) 删除docker其他没有卸载

```
dpkg -l | grep docker
dpkg -l |grep ^rc|awk ‘{print $2}’ |sudo xargs dpkg -P # 删除无用的相关的配置文件
```

#### 3) 卸载没有删除的docker相关插件(结合自己电脑的实际情况)

```
sudo apt-get autoremove docker-ce-*
```

#### 4) 删除Docker的相关配置&目录

```
sudo rm -rf /etc/systemd/system/docker.service.d
sudo rm -rf /var/lib/docker
```

#### 5) 确定Docker卸载完成

```
docker --version
```

## 安装Docker

### 1.跟着官网走，卸载掉你系统中较老版本的docker，不过经过上面的一波猛如虎的操作，基本到这里不会有啥残留了

```
sudo apt-get remove docker docker-engine docker.io containerd runc
```

### 2.安装相关的apt依赖

#### 1) 更新apt

```
sudo apt-get update
```

#### 2) 安装相关依赖

```
sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg-agent \
        software-properties-common
```

### 3.添加Docker的官方GPG密钥

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

### 4.验证密钥

```
sudo apt-key fingerprint 0EBFCD88
```

### 5.根据你不同的系统去设置一个稳定的仓库（根据系统选一个哦，不要每个都搞）

#### 1) x86_64/amd64

```
sudo add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
       $(lsb_release -cs) \
       stable"
```

#### 2) armhf

```
sudo add-apt-repository \
       "deb [arch=armhf] https://download.docker.com/linux/ubuntu \
       $(lsb_release -cs) \
       stable"
```

#### 3) arm64

```
sudo add-apt-repository \
       "deb [arch=arm64] https://download.docker.com/linux/ubuntu \
       $(lsb_release -cs) \
       stable"
```

### 6.再更新一次apt

```
sudo apt-get update
```

### 7.到目录中设置daemon.json文件（值得注意的是，如果是首次安装，理论上是不会有 cd /etc/docker 这个目录滴，所以如果你真的是第一次安装， 请新建目录或者跳过此步骤，等你下面步骤报错之后，嘿嘿这个目录就会有了，然后从头走一遍你就会发现，这步可以用了）

#### 1) 进入Docker文件夹

```
cd /etc/docker/
```

#### 2) 查看是否有daemon.json文件，没有就创建

```
ls
```

#### 3) 创建daemon.json文件

```
touch daemon.json
```

#### 4) 编辑文件

```
sudo vi /etc/docker/daemon.json
```

#### 5)在vim中输入以下内容。

```
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

### 8.安装最新的Docker

```
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### 9.运行hello-world测试

```
sudo docker run hello-world
```

## 部署EasyConnect

### 1.在服务器中的Docker运行EasyConnect的容器

```
touch ~/.easyconn
docker run --device /dev/net/tun --cap-add NET_ADMIN -ti -v $HOME/.easyconn:/root/.easyconn -p 127.0.0.1:1080:1080 -e EC_VER=7.6.7 hagb/docker-easyconnect:cli
```

其中 `EC_VER` 表示EasyConnect版本号,厦门大学使用的为 `7.6.7` 版本;  `hagb/docker-easyconnect:cli` 表示多版本（ `7.6.3` , `7.6.7` , `7.6.8` ）纯命令行版

详细信息见[@Hagb](https://github.com/Hagb)分享的内容

[![standard-readme compliant](https://img.shields.io/badge/readme%20style-standard-brightgreen.svg?style=flat-square)](https://github.com/Hagb/docker-easyconnect)

### 2.运行完上面的命令后，根据提示输入对应的
- `SSLVPN地址` : `https://sslvpn.xmu.edu.cn` 厦门大学SSLVPN地址

- `用户名` : 厦门大学学号

- `密码` : [厦门大学Wi-Fi/VPN/宿舍网密码](https://pass.xmu.edu.cn/)


### 3.不出意外就会有一个socks5的代理跑在服务器的 '1080' 端口了

```
lsof -i tcp:1080
```

### 4.如果要访问git/ssh服务请在 `~/.ssh/config` 中添加配置文件

#### 1) 查看是否存在 `~/.ssh/` 如果没有就创建。

```
cd 
ls -a
```

#### 2) 创建.ssh文件夹和config文件并编辑

```
mkdir .ssh
cd .ssh
vi config
```

#### 3) 在vim中输入以下内容

```
Host example.com #你要访问的SSH地址
ProxyCommand=nc -X5 -x localhost:1080 %h %p
```

#### 4) 修改完成后可以先在服务器尝试连接SSH验证

```
ssh username@example.com
```

## 用 ssh 转发远服务器端口到本地

由于这个sock5是不带鉴权的，所以不能直接把公网的端口打开，这样会有安全问题，我们用 ssh 把服务器的 `1080` 端口转发到本地的 `1080` 端口

```
ssh -L 1080:127.0.0.1:1080 你服务器的用户名@你的服务器 IP
```

## 配置clash代理规则

clash的配置文件默认路径（以windows为例）

```
C:\Users\你电脑的用户名\.config\clash\profiles
```

在 `*********.yml`（一般为一串数字）中增加以下内容

```
proxies:
  - {name: "XMUVPN", server: "127.0.0.1", port: 1080, type: socks5}
rules:
 - IP-CIDR,***.**.**.*/24,XMUVPN
```

- `***.**.**.*` 表示需要访问的内网地址，如果有其他规则在对应部分添加即可。

## 到这里所有配置都完成了，并且只有访问 `http://***.***.**.*` 的流量才会走 vpn 的流量。



## 参考资料

EasyConnect来自[@shmille](https://github.com/shmilee) 提供的[命令行版客户端 deb 包](https://github.com/shmilee/scripts/releases/download/v0.0.1/easyconn_7.6.8.2-ubuntu_amd64.deb)

特别感谢[@Hagb](https://github.com/Hagb)慷慨分享的[docker-easyconnect](https://github.com/Hagb/docker-easyconnect)

[Ubuntu 20.04 安装 docker 详解](https://blog.csdn.net/u010381752/article/details/114086343)

[ubuntu 完全干净的卸载docker](https://blog.csdn.net/xbeethoven/article/details/107883970)

[使用 docker 封印 EasyConnect](https://taoshu.in/easyconnect-in-docker.html)

[M1 Mac 用不了深信服 easyconnet？ 用 docker+clash封印它](https://zhuanlan.zhihu.com/p/385845245)
