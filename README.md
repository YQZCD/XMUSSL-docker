# XMUSSL-docker
由于厦门大学SSLVPN（EasyConnect）较差的兼容性和规范性，故使用“docker容器+服务器转发”的方法实现内网访问。

本方法参考资料将在尾部统一列出。

EasyConnect来自[@shmille](https://github.com/shmilee) 提供的[命令行版客户端 deb 包](https://github.com/shmilee/scripts/releases/download/v0.0.1/easyconn_7.6.8.2-ubuntu_amd64.deb)。
## 准备工作

 · 拥有公网IP的服务器（Debian系）

 · Docker
 
 · [Clash](https://github.com/Fndroid/clash_for_windows_pkg/releases)
 
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

### 7.到目录中设置daemon.json文件（值得注意的是，如果是首次安装，理论上是不会有 cd /etc/docker 这个目录滴，所以如果你真的是第一次安装， 请跳过此步骤，等你下面步骤报错之后，嘿嘿这个目录就会有了，然后从头走一遍你就会发现，这部可以用了）

#### 1) 进入Docker文件夹

```
cd /etc/docker/
```

#### 2) 查看是否有daemon.json文件，没有就创建。

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

#### 5)在vim中按i编辑，输入以下内容后按ESC退出编辑模式，输入ZZ保存。

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




