## 自定义端口访问问题

### 步骤1：验证端口映射配置

```
# 查看容器端口映射状态
docker port aioveu-boot-nginx
# 正确输出应显示：9999/tcp -> 0.0.0.0:9999
```

### 步骤2：检查服务器防火墙设置

```
# 检查防火墙状态
sudo ufw status

# 如果激活了防火墙，添加规则
sudo ufw allow 9999
sudo ufw reload

# 检查端口是否开放
sudo netstat -tuln | grep 9999
```

### 步骤3：检查云服务器安全组设置

•在安全组设置中：1.添加入站规则：端口范围 99992.协议类型：TCP3.来源：0.0.0.0/0（或您的IP范围）4.保存规则

### 步骤4：验证端口连通性

```
# 在服务器本地测试
curl -v http://localhost:9999

# 在外部机器测试（替换真实IP）
telnet your-server-ip 9999
```

### 步骤5：解决被占用的80端口问题

```
# 找出占用80端口的进程
sudo lsof -i :80

# 输出示例：
# COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
# nginx    123 root    6u  IPv4  12345      0t0  TCP *:http (LISTEN)

# 停止占用程序
sudo systemctl stop nginx  # 如果是系统nginx
sudo systemctl disable nginx

# 或者直接终止进程
sudo kill -9 PID
```

### 步骤6：重新配置Nginx容器

```
services:
  nginx:
    image: nginx:latest
    ports:
      - "9999:80"   # 外部9999 -> 容器80
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./html:/usr/share/nginx/html
    restart: unless-stopped
```

确保nginx.conf包含：

```
server {
    listen 80;  # 容器内部监听80端口
    server_name feifei.aioveu.com;
    
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
}
```

### 步骤7：解决DNS解析问题

```
# 验证域名解析
nslookup feifei.aioveu.com

# 如果没有正确解析，临时修改hosts文件
echo "服务器公网IP feifei.aioveu.com" | sudo tee -a /etc/hosts
```

### 步骤9：HTTPS支持（如果需要）

```
# 创建测试证书
mkdir -p docker/nginx/ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout docker/nginx/ssl/selfsigned.key \
  -out docker/nginx/ssl/selfsigned.crt \
  -subj "/CN=feifei.aioveu.com"
```

修改nginx.conf:

```
server {
    listen 80;
    server_name feifei.aioveu.com;
    return 301 https://$host:9999$request_uri;
}

server {
    listen 443 ssl;
    server_name feifei.aioveu.com;
    
    ssl_certificate /etc/nginx/ssl/selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/selfsigned.key;
    
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}
```

更新docker-compose.yml:

```
ports:
  - "80:80"   # HTTP重定向
  - "443:443" # HTTPS
```

然后访问: `https://feifei.aioveu.com:9999`
