DockerDesktop安装如何指定安装目录



docker安装过程，默认不能修改安装目录，如果想自己指定安装目录，可以使用命令行的方式参数干-installation-dir=D:\Docker可以指定安装位置
start /w "" "Docker Desktop Installer.exe" install --installation-dir=D:\Docker

把下载的安装程序Docker Desktop Installer.exe 放到D:

CMD进入D：，执行命令：



```
start /w "" "Docker Desktop Installer.exe" install --installation-dir=D:\Docker
```

其中：在 Docker Desktop 的安装配置里，“Allow Windows Containers to be used with this installation” 其实是控制它是否同时具备 运行 Windows 容器的能力 的一个选项。默认情况下，Docker Desktop 主要运行 Linux 容器（通过 WSL 2 或 Hyper-V 虚拟机），而启用这个选项会让它的安装包额外为 Windows 容器模式准备运行环境。


这个弹出的 Docker Subscription Service Agreement（Docker 订阅服务协议）不是一个“技术配置”，而是 Docker 官方的 使用许可与订阅条款，作用主要是告诉你——在安装或继续使用 Docker Desktop 前，你需要同意它的最新授权规则和使用条件。

