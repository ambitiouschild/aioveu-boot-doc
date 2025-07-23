# Linux (Ubuntu) 安装 Docker 和 Docker Compose

在Ubuntu上安装Docker和Docker Compose是一个相对直接的过程。

### 1. 更新包索引

首先，确保你的包列表是最新的，这是通过运行以下命令完成的：

```
sudo apt-get update
```

### 2. 安装Docker的依赖包

安装一些必要的系统依赖包，这些包将帮助Docker正常运行：

```
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

### 3. 添加Docker的官方GPG密钥

为了确保从Docker的官方仓库下载的包是安全的，你需要添加Docker的官方GPG密钥：

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

### 4. 设置Docker仓库

添加Docker仓库到你的APT源列表中：

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```



aioveu@aioveu:~$   sudo systemctl daemon-reload     #重新加载配置

aioveu@aioveu:~$   sudo systemctl restart docker      #重启docker服务





### 5. 安装Docker Engine

现在，你可以安装Docker Engine了：

```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### 6. 启动Docker服务并设置为开机启动

```
sudo systemctl start docker
sudo systemctl enable docker
```



```
找不到命令 “docker”，但可以通过以下软件包安装它：
sudo apt install docker.io      # version 26.1.3-0ubuntu1~24.04.1, or
sudo apt install podman-docker  # version 4.9.3+ds1-1ubuntu0.2
```

### 7. 验证Docker安装是否成功

运行以下命令来验证Docker是否正确安装并运行：

```
docker --version
```

你应该会看到Docker的版本号。

### 8. 安装Docker Compose（可选，但推荐）

虽然从Docker 17.06版本开始，Docker已经内置了`docker compose`命令，但从GitHub上安装最新版本的`docker-compose`插件通常是一个好主意。首先，卸载内置的`docker compose`（如果已安装）：

```
sudo rm /usr/local/bin/docker-compose
```

然后，安装最新版本的`docker-compose`：

```
sudo curl -L "https://github.com/docker/compose/releases/download/v2.10.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

请检查GitHub上的最新版本号，并根据需要替换上述命令中的版本号。例如，如果你想要安装最新版本，可以去[Docker Compose GitHub Releases](https://github.com/docker/compose/releases)页面查找最新的版本号。

### 9. 验证Docker Compose安装是否成功（如果已安装）

运行以下命令来验证Docker Compose是否正确安装并运行：

```

docker-compose --version
```

你应该会看到Docker Compose的版本号。这样，你就成功地在Ubuntu上安装了Docker和Docker Compose。

```
aioveu@aioveu:~$ docker-compose --version
找不到命令 “docker-compose”，但可以通过以下软件包安装它：
sudo apt install docker-compose
```

### 解决方案二：修复 Python 环境（如果必须使用旧版）

如果必须使用旧版，可以安装 `python3-distutils` 模块，但这个模块在 Ubuntu 24.04 中已经不可用了（因为 Python 3.12 移除了 `distutils`）。因此，这个方案可能不适用于 Ubuntu 24.04。

对于 Ubuntu 22.04 或更早版本，你可以尝试：

```
sudo apt install python3-distutils
```

### 最终建议：

使用 Docker Compose V2 是更可靠的方法，因为它不依赖于 Python 环境。安装完成后，您可以使用 `docker-compose` 或 `docker compose` 命令（插件方式）。如果您使用的是插件方式，可以创建一个别名以便继续使用 `docker-compose` 命令





### 解决方案：完全迁移到 Docker Compose V2

Docker Compose V2 是独立于 Python 的解决方案，完美解决此问题：

#### 步骤 1：移除旧的 Docker Compose（V1）

```bash
sudo apt remove docker-compose
```

#### 步骤 2：安装 Docker Compose V2（插件式版本）

sudo apt install -y docker-compose-plugin  # 安装官方插件版

该版本相较于传统的[docker-compose](https://so.csdn.net/so/search?q=docker-compose&spm=1001.2101.3001.7020)有以下优势：
\- 直接集成到docker CLI
\- 版本与Docker Engine同步更新
\- 兼容compose v2语法

2.验证安装结果

```csharp
docker compose version  # 注意中间没有短横线
```

三、进阶配置建议

1. 非root用户权限配置（可选）
   为避免每次使用sudo，可将当前用户加入docker组：

1. sudo usermod -aG docker $USER   # $USER替换为具体用户名
2. newgrp docker                   # 刷新用户组

注意：修改后需要重新登录生效
