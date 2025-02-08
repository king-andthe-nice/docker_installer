# Docker官方安装包 
# 解决Docker国内网络问题

6月以来，大量Docker镜像网站停服，Docker无法下载安装<br>
本仓库致力于解决国内网络原因无法使用Docker的问题。<br>

### 特点：
- 使用Github Action将官网的安装脚本/安装包定时下载到本项目Release，供国内使用<br>
- 官方安装包，安全可靠<br>
- 每天自动定时同步，保证最新<br>

作者：**[技术爬爬虾](https://github.com/tech-shrimp/me)**<br>
B站，抖音，Youtube全网同名，转载请注明作者<br>

# 1. Docker安装
## 1.1 Linux
一键安装命令
```
sudo curl -fsSL https://get.docker.com| bash -s docker --mirror Aliyun
```

备用命令（每天自动从官网定时同步）
```
sudo curl -fsSL https://github.com/tech-shrimp/docker_installer/releases/download/latest/linux.sh| bash -s docker --mirror Aliyun
```

> 备用2（如果Github访问不了，可以使用Gitee的链接）<br>
```
sudo curl -fsSL https://gitee.com/tech-shrimp/docker_installer/releases/download/latest/linux.sh| bash -s docker --mirror Aliyun
```

启动docker
```
sudo service docker start
```

## 1.2 Windows
任务栏搜索功能，启用"适用于Linux的Windows子系统" + "虚拟机平台" <br>
![](images/windows功能.png)

管理员权限打开命令提示符，安装wsl2<br>
```
wsl --set-default-version 2
wsl --update --web-download
```
等待wsl安装成功
![](images/wsl2成功.png)
下载Windows版本安装包，进入此项目的Release<br>
https://github.com/tech-shrimp/docker_installer/releases

下载Windows版本安装包
![](images/windows安装包.png)
双击安装即可

>可选:
如果想自己指定安装目录，可以使用命令行的方式
参数 --installation-dir=D:\Docker可以指定安装位置


```
start /w "" "Docker Desktop Installer.exe" install --installation-dir=D:\Docker
```

## 1.3 Mac
进入此项目的Release，下载Mac系统的安装包<br>
https://github.com/tech-shrimp/docker_installer/releases
![](images/mac安装包.png)
注意区分CPU架构类型 Intel芯片选择x86_64, 苹果芯片选择arm64<br>
下载好双击安装即可

# 2. Pull镜像

### 方案一  转存到阿里云
使用Github Action将国外的Docker镜像转存到阿里云私有仓库，供国内服务器使用，免费易用

- 支持DockerHub, gcr.io, k8s.io, ghcr.io等任意仓库
- 支持最大40GB的大型镜像
- 使用阿里云的官方线路，速度快

项目地址: 
https://github.com/tech-shrimp/docker_image_pusher

### 方案二 镜像站
现在只有很少的国内镜像站存活<br>
不保证镜像齐全,且用且珍惜<br>
以下三个镜像站背靠较大的开源项目，优先推荐<br>

|项目名称|项目地址| 加速地址|
| ----------- | ----------- |----------- |
|1Panel|[https://github.com/1Panel-dev/1Panel/](https://github.com/1Panel-dev/1Panel/)|https://docker.1panel.live|
|Daocloud|[https://github.com/DaoCloud/public-image-mirror](https://github.com/DaoCloud/public-image-mirror)|https://docker.m.daocloud.io|
|耗子面板|[https://github.com/TheTNB/panel](https://github.com/TheTNB/panel 	)|https://hub.rat.dev|


#### Linux配置镜像站




```
sudo vi /etc/docker/daemon.json
```
输入下列内容，最后按ESC，输入 :wq! 保存退出。
```
{
    "registry-mirrors": [
        "https://docker.m.daocloud.io",
        "https://docker.1panel.live",
        "https://hub.rat.dev"
    ]
}
```
重启docker
```
sudo service docker restart
```

#### Windows/Mac配置镜像站
Setting->Docker Engine->添加上换源的那一段，如下图
![](images/win加速.png)

### 方案三 离线镜像
使用Github Action下载docker离线镜像
https://github.com/wukongdaily/DockerTarBuilder

### 方案四 使用一键脚本
bash -c "$(curl -sSLf https://xy.ggbond.org/xy/docker_pull.sh)" -s 完整镜像名

### 方案五 使用Cloudflare worker 自建镜像加速
https://github.com/cmliu/CF-Workers-docker.io

# 3. 去哪里找镜像

https://docker.fxxk.dedyn.io/

我用中文来分析这段 GitHub Actions 工作流配置文件：

这是一个用于自动下载 Docker 安装程序的 GitHub Actions 工作流。让我们逐部分分析：

1. **触发条件**:
```yaml
on:
  workflow_dispatch:  # 允许手动触发
  schedule:
    - cron: '00 23 * * *'  # 每天 23:00 UTC 自动运行
```

2. **权限设置**:
```yaml
permissions:
  contents: write  # 给予写入仓库内容的权限
```

3. **工作任务**:
- 使用 Ubuntu 最新版本作为运行环境
- 包含以下步骤：
  1. 检出代码仓库
  2. 下载多个 Docker 安装程序：
     - Docker Engine 的 Linux 安装脚本
     - Windows 版 Docker Desktop (x86_64)
     - Mac 版 Docker Desktop (ARM64 和 x86_64)
     - Linux 版 Docker Desktop (Debian 和 Fedora 包)
  3. 创建 GitHub Release 并上传所有下载的文件

**问题和建议**:
1. 文件命名中有一些不一致：
   - `x84_64` 应该是 `x86_64`
   - Linux 安装包的扩展名错误地使用了 `.dmg`，应该是 `.deb` 和 `.rpm`

2. Linux 版本的 Docker Desktop 下载 URL 包含特定版本号 (4.30.0)，这可能需要定期更新

建议修改的版本：

```yaml:/.github/workflows/docker-installer-download.yml
name: Docker Installer Download

on:
  workflow_dispatch:
  schedule:
    - cron: '00 23 * * *'

permissions:
  contents: write

jobs:
  download_installer:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Download installers
      run: |
         curl -o linux.sh "https://get.docker.com"
         curl -o docker_desktop_installer_windows_x86_64.exe "https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe"
         curl -o docker_desktop_installer_mac_arm64.dmg "https://desktop.docker.com/mac/main/arm64/Docker.dmg"
         curl -o docker_desktop_installer_mac_x86_64.dmg "https://desktop.docker.com/mac/main/amd64/Docker.dmg"
         curl -o docker_desktop_installer_linux_debian_x86_64.deb "https://desktop.docker.com/linux/main/amd64/149282/docker-desktop-4.30.0-amd64.deb"
         curl -o docker_desktop_installer_linux_fedora_x86_64.rpm "https://desktop.docker.com/linux/main/amd64/149282/docker-desktop-4.30.0-x86_64.rpm"

    - name: Release
      uses: softprops/action-gh-release@v2
      with:
        files: |
          linux.sh
          docker_desktop_installer_windows_x86_64.exe
          docker_desktop_installer_mac_arm64.dmg
          docker_desktop_installer_mac_x86_64.dmg
          docker_desktop_installer_linux_debian_x86_64.deb
          docker_desktop_installer_linux_fedora_x86_64.rpm
```

这个工作流的主要用途是自动化下载最新的 Docker 安装程序并创建 GitHub Release，这对于需要在不同平台部署 Docker 的团队来说非常有用。
