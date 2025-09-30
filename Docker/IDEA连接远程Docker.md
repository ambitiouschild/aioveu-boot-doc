## 1.完成 SSH 密钥生成（按提示操作）



````
```bash
# 1. 完成 SSH 密钥生成（按提示操作）
ssh-keygen -t rsa -b 4096
```
````



![](F:\Coding\Github\aioveu-boot-doc\Docker\1.png)



![](F:\Coding\Github\aioveu-boot-doc\Docker\2.png)





![](F:\Coding\Github\aioveu-boot-doc\Docker\3.png)



## 2.复制公钥到远程服务器





````
```bash
# 2. 复制公钥到远程服务器
#   手动方法：
#   a. 查看公钥内容：cat ~/.ssh/id_rsa.pub
#   b. 登录远程服务器：ssh root@aioveu.com
#   c. 添加公钥：echo "公钥内容" >> ~/.ssh/authorized_keys
```
````



![](F:\Coding\Github\aioveu-boot-doc\Docker\4.png)



## 3.使用环境变量方法连接

````
```bash
# 3. 使用环境变量方法连接
#$env:DOCKER_HOST = "ssh://root@aioveu.com"
```
````





## 4.创建远程上下文



````
```bash
# 4.创建远程上下文
docker context create remote --docker "host=ssh://root@aioveu.com"

```
````



![](F:\Coding\Github\aioveu-boot-doc\Docker\5.png)





## 5.切换到远程上下文

````
```bash
# 5.切换到远程上下文
docker context use remote
```
````



![](F:\Coding\Github\aioveu-boot-doc\Docker\6.png)







## 6.测试 SSH 连接

````
```bash
# 6.测试 SSH 连接
ssh root@www.aioveu.com "docker version"
```
````



![](F:\Coding\Github\aioveu-boot-doc\Docker\7.png)



## 7.部署容器



````
```bash
# 7. 部署容器
docker compose -f ./docker-compose.yml -p aioveu-boot up -d

```
````



在Windows本地使用Docker Compose部署到远程Linux服务器时，挂载路径的写法需要注意以下几点：

1. **避免使用Windows绝对路径**：远程服务器是Linux系统，无法识别Windows路径（如`C:\...`或`F:\...`）。
2. **使用相对路径或Linux绝对路径**：在`docker-compose.yml`中，应该使用Linux风格的路径。
3. **路径必须存在于远程服务器上**：挂载的目录在远程服务器上必须存在，否则Docker会自动创建，但可能权限不正确。

### 方法1：使用相对路径（推荐）

在`docker-compose.yml`中，使用相对于Compose文件所在目录的路径。Docker Compose会将相对路径解析为相对于Compose文件所在的目录。

例如，假设你的`docker-compose.yml`文件位于远程服务器的`/opt/aioveu-boot/docker`目录下，你可以这样写：

```
services:
  minio:
    image: minio/minio
    volumes:
      - ./minio/data:/data   # 挂载当前目录下的minio/data目录到容器的/data
```

这样，在远程服务器上，Docker会挂载`/opt/aioveu-boot/docker/minio/data`到容器中。

### 方法2：使用Linux绝对路径

如果你需要挂载远程服务器上的特定目录，可以使用绝对路径：

```
services:
  minio:
    image: minio/minio
    volumes:
      - /opt/aioveu-boot/docker/data/minio:/data   # 远程服务器上的绝对路径
```



### 在Windows上准备Compose文件时的注意事项

1. **路径分隔符**：在`docker-compose.yml`文件中，使用Linux风格的正斜杠`/`，而不是反斜杠``。
2. **文件编码**：确保文件使用Unix换行符（LF），而不是Windows的CRLF。可以在文本编辑器（如VS Code）中设置。

### 部署步骤

1. 1.

   **将Compose文件和相关目录上传到远程服务器**

   由于挂载的路径是相对于远程服务器上的Compose文件位置，所以你需要将整个项目目录（包括`minio/data`等目录）上传到远程服务器。

2. 2.

   **在远程服务器上运行Compose命令**

   登录到远程服务器，进入项目目录，然后运行：

   ```
   docker compose -p aioveu-boot up -d
   ```



这个错误再次确认了路径问题：您正在尝试将 Windows 本地路径挂载到远程 Linux 容器中，这是不可能的。

1. **所有路径都是远程路径**：当部署到远程 Docker 时，所有卷挂载路径必须指向远程服务器上的位置
2. **数据与代码分离**：初始化脚本应该上传到远程服务器，而不是尝试挂载本地文件
3. **自动化部署**：使用脚本自动化上传和部署过程
4. **考虑镜像构建**：对于初始化脚本，构建自定义镜像通常是更可靠的方法













## MySQL 容器初始化机制





是的，您放在 `/docker-entrypoint-initdb.d`目录下的 SQL 脚本**应该会自动初始化**，但需要满足特定条件。让我详细解释初始化机制和常见问题：1.

1.**首次启动时初始化**：

- 当 `/var/lib/mysql`目录**为空**时，容器会执行初始化

- 它会按字母顺序执行 `/docker-entrypoint-initdb.d`目录中的所有 `.sh`、`.sql`和 `.sql.gz`文件

  

2.**后续启动不会初始化**：

- 如果数据目录已有数据（即使容器被删除重建），初始化会被跳过
- 这是设计行为，防止意外覆盖现有数据库
- 

为什么您的 SQL 表没有加载？

1. 数据目录已有数据（最常见原因）

2. 文件权限问题



### 强制初始化的解决方案

#### 方法 1：完全重置

```
# 1. 停止并删除容器
docker compose -p aioveu-boot down -v

# 2. 删除数据目录
ssh root@aioveu.com "sudo rm -rf /opt/aioveu-boot/data/mysql/*"

# 3. 重新部署
docker compose -p aioveu-boot up -d
```

#### 方法 2：手动触发初始化

```
# 1. 进入 MySQL 容器
docker exec -it aioveu-boot-mysql bash

# 2. 执行初始化脚本
find /docker-entrypoint-initdb.d -name '*.sql' -exec mysql -u root -p{} \;
```

3.**使用数据库迁移工具**：





### 1. 确保SQL脚本按正确顺序执行

Docker会按照文件名的字母顺序执行`/docker-entrypoint-initdb.d`目录中的SQL脚本。因此，你需要确保表创建的脚本在外键约束脚本之前执行。



**建议的脚本命名顺序：**

```
01-create-database.sql
02-create-tables.sql
03-add-constraints.sql
```

### 2. 修改SQL脚本

在每个SQL脚本中，确保先创建被引用的表，然后再创建外键约束。



### 3. 重新初始化数据库

由于之前初始化失败，需要删除数据目录并重新初始化：

```
# 停止并删除MySQL容器
docker compose -p aioveu-boot stop mysql
docker compose -p aioveu-boot rm -f mysql

# 删除数据目录（注意：这会删除所有数据！）
sudo rm -rf /opt/aioveu-boot/data/mysql/*
```

根据日志，MySQL容器已经成功启动，并且初始化脚本开始执行。但是，在执行初始化脚本时出现了错误：

```
ERROR 1824 (HY000) at line 17: Failed to open the referenced table 'aioveu_category'
```

这个错误表明在尝试创建外键约束时，引用的表 'aioveu_category' 不存在。这通常是因为SQL脚本的执行顺序问题：在创建外键约束之前，被引用的表必须已经创建。

## 执行初始化脚本时出现了错误解决方案

### 1. 确保SQL脚本按正确顺序执行

Docker会按照文件名的字母顺序执行`/docker-entrypoint-initdb.d`目录中的SQL脚本。因此，你需要确保表创建的脚本在外键约束脚本之前执行。

**建议的脚本命名顺序：**

```
01-create-database.sql
02-create-tables.sql
03-add-constraints.sql
```

### 2. 修改SQL脚本

在每个SQL脚本中，确保先创建被引用的表，然后再创建外键约束。

### 3. 重新初始化数据库

由于之前初始化失败，需要删除数据目录并重新初始化：

```
# 停止并删除MySQL容器
docker compose -p aioveu-boot stop mysql
docker compose -p aioveu-boot rm -f mysql

# 删除数据目录（注意：这会删除所有数据！）
sudo rm -rf /opt/aioveu-boot/data/mysql/*

# 重新启动
docker compose -p aioveu-boot up -d mysql
```

### 4. 检查SQL脚本中的依赖关系

确保所有被引用的表都在外键约束之前创建。如果多个脚本之间有依赖关系，需要调整脚本的执行顺序。

### 5. 使用单个SQL脚本

考虑将所有的表创建和外键约束放在一个SQL脚本中，这样可以确保正确的执行顺序。

### 6. 使用数据库迁移工具

对于复杂的数据库结构，建议使用数据库迁移工具（如Flyway或Liquibase）来管理数据库版本和变更。

### 7.验证

重新初始化后，检查日志以确保所有脚本成功执行：

```
docker compose -p aioveu-boot logs mysql
```

然后检查数据库中的表和外键约束：

```
docker exec -it aioveu-boot-mysql mysql -u root -e "USE your_database; SHOW TABLES;"
docker exec -it aioveu-boot-mysql mysql -u root -e "USE your_database; SHOW CREATE TABLE your_table;"
```

通过以上步骤，你应该能够解决外键约束引用的表不存在的问题。
