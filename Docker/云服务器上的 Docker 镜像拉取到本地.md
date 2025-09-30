# 云服务器上的 Docker 镜像拉取到本地

关键是确保标签正确、仓库权限匹配，并处理网络问题



## 1.**推荐方式**：通过远程仓库推送/拉取（适合需要共享或频繁更新的镜像）



核心思路是**先将云服务器上的镜像推送到一个远程镜像仓库（如 Docker Hub、阿里云 ACR 等），然后本地从该仓库拉取**。



### **一、前提条件**

1. 云服务器已安装 Docker，且目标镜像存在（可通过 `docker images`查看）。
2. 本地已安装 Docker（Docker 安装教程）。
3. 已注册一个远程镜像仓库账号（如 Docker Hub，或阿里云/腾讯云的私有仓库）。



### **二、步骤 1：将云服务器的镜像推送到远程仓库**

若镜像未推送到过远程仓库，需先完成此步骤。

#### **1.1 在云服务器上给镜像打「远程标签」**

Docker 镜像需要通过 **标签（Tag）** 关联到远程仓库。标签格式为：

`[远程仓库地址]/[用户名]/[镜像名]:[版本号]`（版本号可选，默认 `latest`）。

**示例**（假设远程仓库是 Docker Hub，你的用户名是 `alice`，镜像名为 `my-app`，版本为 `v1`）：

```
# 查看云服务器上的本地镜像（找到你要推送的镜像名和标签）
docker images

# 给本地镜像打远程标签（原镜像名:标签 → 远程仓库地址/用户名/镜像名:版本）
docker tag my-app:v1 alice/my-app:v1  # 若远程是 Docker Hub，地址可省略（默认是 Docker Hub）


# 格式：docker tag 原镜像名:原版本 新仓库地址/命名空间/镜像名:新版本
docker tag 756a87f57763  crpi-s90cufgtjv4fnw98.cn-shanghai.personal.cr.aliyuncs.com/ambitiouschild/aioveu-boot:v1.2.0
```



![](F:\Coding\Github\aioveu-boot-doc\Docker\8.png)







#### **1.2 登录远程仓库**

在云服务器上登录远程仓库（以 Docker Hub 为例）：



```
docker login  # 输入你的 Docker Hub 用户名和密码（或访问令牌）
```

若使用**私有仓库**（如阿里云 ACR）：

```
docker login --username=你的阿里云账号 仓库地址  

# 例如

docker login --username=可我不敌可爱 crpi-s90cufgtjv4fnw98.cn-shanghai.personal.cr.aliyuncs.com
```



![](F:\Coding\Github\aioveu-boot-doc\Docker\9.png)





#### **1.3 推送镜像到远程仓库**

在云服务器上执行推送命令：

```
  # 推送标签后的镜像

docker push crpi-s90cufgtjv4fnw98.cn-shanghai.personal.cr.aliyuncs.com/ambitiouschild/aioveu-boot:v1.2.0  # 推送标签后的镜像

```

推送成功后，可在远程仓库的控制台（如 Docker Hub 控制台）查看镜像。



![](F:\Coding\Github\aioveu-boot-doc\Docker\10.png)



![](F:\Coding\Github\aioveu-boot-doc\Docker\11.png)













### **三、步骤 2：本地拉取远程仓库的镜像**

本地通过 `docker pull`命令从远程仓库拉取镜像。

#### **2.1 本地登录远程仓库（若需要）**

若远程仓库是**私有仓库**（如阿里云 ACR），本地需先登录认证：



```
docker login --username=你的阿里云账号 仓库地址 

# 例如：

docker login --username=可我不敌可爱 crpi-s90cufgtjv4fnw98.cn-shanghai.personal.cr.aliyuncs.com
```

若远程仓库是**公共仓库**（如 Docker Hub）且镜像已公开，可直接拉取（无需登录）。



#### **2.2 执行拉取命令**

在本地终端输入拉取命令（标签需与推送时一致）：

```
docker pull alice/my-app:v1  # 拉取镜像


首先，用户可能对Docker镜像的命名结构不太清楚。需要说明镜像的完整格式是[仓库地址/][用户名/]镜像名:标签。对于公共镜像如nginx，通常属于Docker Hub的官方仓库，所以仓库地址可以省略，默认使用Docker Hub的公共仓库。而用户的阿里云镜像属于私有仓库，必须包含完整的仓库地址、用户名和镜像名。

Docker Hub的官方镜像（如nginx）的结构是library/nginx，其中library是官方命名空间，所以可以简化为nginx。而私有仓库的镜像需要完整的路径，包括地域、命名空间、镜像名等，所以看起来更长。

用户可能混淆了镜像的存储位置。Docker Hub的公共镜像可以被全球用户拉取，所以不需要额外的仓库地址。而私有镜像存储在特定的私有仓库（如阿里云ACR），必须通过完整的地址来定位，因此镜像名更长。

docker pull crpi-s90cufgtjv4fnw98.cn-shanghai.personal.cr.aliyuncs.com/ambitiouschild/aioveu-boot:v1.2.0
# 拉取后打短标签
docker pull crpi-s90cufgtjv4fnw98.cn-shanghai.personal.cr.aliyuncs.com/ambitiouschild/aioveu-boot:v1.2.0
docker tag crpi-s90cufgtjv4fnw98.cn-shanghai.personal.cr.aliyuncs.com/ambitiouschild/aioveu-boot:v1.2.0 aioveu-boot:v1.2.0

```



![](F:\Coding\Github\aioveu-boot-doc\Docker\12.png)





### **为什么 `docker pull nginx`很短？**



#### 1. **默认使用 Docker Hub 公共仓库**

Docker 客户端默认将未指定仓库地址的镜像视为来自 **Docker Hub 公共仓库**（`docker.io`）。因此：

- `docker pull nginx`等价于 `docker pull docker.io/library/nginx:latest`。

#### 2. **官方镜像的命名空间是 `library`**

Docker Hub 中，官方维护的镜像（如 `nginx`、`mysql`）属于 `library`命名空间（官方命名空间）。因此，完整路径为：

`docker.io/library/nginx:latest`。

由于 `library`是 Docker Hub 的默认命名空间，客户端允许省略 `docker.io/library/`前缀，直接使用 `nginx`作为镜像名。

#### 3. **标签默认是 `latest`**

若未指定标签（如 `v1.2.0`），Docker 会默认使用 `latest`标签。因此：

`docker pull nginx`等价于 `docker pull nginx:latest`。

### **为什么你的阿里云镜像名很长？**

你的镜像 `crpi-s90cufgtjv4fnw98.cn-shanghai.personal.cr.aliyuncs.com/ambitiouschild/aioveu-boot:v1.2.0`属于 **阿里云 ACR 私有仓库的镜像**，其完整引用格式必须包含以下部分：

#### 1. **仓库地址（必须）**

私有仓库需明确指定仓库地址（如阿里云 ACR 的地域地址 `cn-hangzhou.mirror.aliyuncs.com`或个人版实例地址 `crpi-s90cufgtjv4fnw98.cn-shanghai.personal.cr.aliyuncs.com`）。

#### 2. **用户名/命名空间（必须）**

私有仓库的镜像通常属于用户或组织的命名空间（如你的 `ambitiouschild`），用于隔离不同用户的镜像。

#### 3. **明确的标签（可选但推荐）**

你指定了标签 `v1.2.0`，因此完整镜像名需包含该版本信息。



//==============================

#### 1. 配置别名（临时简化）

```
alias aioveu-boot:v1.2.0='docker pull crpi-s90cufgtjv4fnw98.cn-shanghai.personal.cr.aliyuncs.com/ambitiouschild/aioveu-boot:v1.2.0'
```



之后只需执行 `aioveu-boot:v1.2.0`即可拉取镜像（仅当前终端有效）。

#### 2. 修改 `daemon.json`（全局简化，不推荐）

Docker 客户端不支持直接配置镜像别名，但可以通过 **镜像标签** 间接实现：

```
# 拉取后打短标签
docker pull crpi-s90cufgtjv4fnw98.cn-shanghai.personal.cr.aliyuncs.com/ambitiouschild/aioveu-boot:v1.2.0
docker tag crpi-s90cufgtjv4fnw98.cn-shanghai.personal.cr.aliyuncs.com/ambitiouschild/aioveu-boot:v1.2.0 aioveu-boot:v1.2.0
```







## 2.**备用方式**：通过 `docker save/load`传输文件（适合小镜像或无网络场景）

若远程仓库不可用（如内网环境），可将云服务器的镜像保存为 `tar`文件，通过文件传输工具（如 `scp`、U盘）下载到本地，再加载到本地 Docker。

#### **4.1 在云服务器上导出镜像**

bash

复制

```
# 查看镜像 ID（假设镜像 ID 为 1234567890ab）
docker images

# 导出镜像为 tar 文件（替换为你的镜像 ID 和文件名）
docker save -o my-app.tar 1234567890ab
```

#### **4.2 下载文件到本地**

通过 `scp`命令将 `my-app.tar`从云服务器下载到本地（需知道云服务器 IP 和登录密码/密钥）：

bash

复制

```
# 本地终端执行（替换为你的云服务器信息）
scp 用户名@云服务器IP:/路径/my-app.tar /本地路径/
```

#### **4.3 本地加载镜像**

在本地终端执行加载命令：

```
docker load -i /本地路径/my-app.tar
```
