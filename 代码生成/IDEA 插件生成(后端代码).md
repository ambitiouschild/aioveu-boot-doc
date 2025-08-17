# 代码生成

本节将详细介绍两种代码生成方式：

**1. IDEA 插件生成** - 专注快速生成后端 CRUD 代码

**2. 项目代码生成** - 通过前端页面一键生成前后端完整模块

## IDEA 插件生成(后端代码)

### 安装 MybatisX 插件

在 IDEA 中依次点击 **File → Settings**（快捷键 **Ctrl + Alt + S**），打开设置面板，切换到 **Plugins** 选项卡，搜索 **MybatisX** 并安装插件。

![](F:\Coding\Github\aioveu-boot-doc\代码生成\1.png)

### 自动代码生成

在 IDEA 右侧导航栏点击 **Database**，打开数据库配置面板，选择新增数据源。



![](F:\Coding\Github\aioveu-boot-doc\代码生成\2.png)

输入数据库的 **主机地址**、**用户名** 和 **密码**，测试连接成功后点击 `OK` 保存。

![](F:\Coding\Github\aioveu-boot-doc\代码生成\3.png)

配置完数据源后，展开数据库中的表，右击 **aioveu-department** 表，选择 **MybatisX-Generator** 打开代码生成面板。



![](F:\Coding\Github\aioveu-boot-doc\代码生成\4.png)

设置代码生成的目标路径，并选择 **Mybatis-Plus 3 + Lombok** 代码风格。

![](F:\Coding\Github\aioveu-boot-doc\代码生成\5.png)

![](F:\Coding\Github\aioveu-boot-doc\代码生成\6.png)



点击 `Finish` 生成，自动生成相关代码。



![](F:\Coding\Github\aioveu-boot-doc\代码生成\7.png)

MybatisX 生成的代码存在以下问题：

- `AioveuDepartmentMapper.java` 文件未标注 `@Mapper` 注解，导致无法被 Spring Boot 识别为 Mybatis 的 Mapper 接口。如果已配置 `@MapperScan`，可以省略此注解，但最简单的方法是直接在 `AioveuDepartmentMapper.java` 文件中添加 `@Mapper` 注解。注意避免导入错误的包。



![](F:\Coding\Github\aioveu-boot-doc\代码生成\8.png)
