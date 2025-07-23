

Ubuntu安装SSH及问题解决，idea使用root账号登录
## 2、安装ssh服务器端

Ubuntu默认没有安装ssh的server，需要安装

```bash
sudo apt-get install openssh-server
```

## 3、允许远程使用root账号ssh连接本机

修改/etc/ssh/sshd_config文件

```text
sudo vim /etc/ssh/sshd_config
```

将 `# PermitRootLogin prohibit-password`修改为 `PermitRootLogin yes`

![](F:\Coding\Github\aioveu-boot-doc\idea使用root账号登录1.png)

## 4、需要重启系统或者sshd服务

```text
重新加载 systemd 配置（关键步骤）：
sudo systemctl daemon-reload


sudo /etc/init.d/ssh stop
sudo /etc/init.d/ssh start
sudo service ssh restart

现在安全地重启 SSH 服务：
sudo systemctl restart ssh

sudo service ssh restart
sudo systemctl status ssh


‌查看Ubuntu服务器的防火墙设置，确保远程连接端口没有被阻止‌：
Ubuntu通常使用ufw（Uncomplicated Firewall）作为防火墙。你可以使用以下命令查看防火墙状态：
sudo ufw status
如果防火墙阻止了SSH端口（22），你可以使用以下命令允许该端口：

sudo ufw allow 22/tcp
重新加载防火墙规则：

sudo ufw reload

```



## 5、安装ssh服务后，系统默认开启系统sshd，查看sshd状态如果不是默认启动，修改服务为enable

```text
sudo systemctl enable ssh
```

## 6、查看Ubuntu的ip

```text
ifconfig
```

## 7、打开刚下载的Idea,XShell,HexHub

然后连接输入ubuntu的用户名和密码

连接成功

![](F:\Coding\Github\aioveu-boot-doc\idea使用root账号登录2.png)

