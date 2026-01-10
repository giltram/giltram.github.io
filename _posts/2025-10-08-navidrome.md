---
title: navidrome
layout: post
permalink: /navidrome.html
---

# navidrome

## 0. Environment
Ubuntu20.04


## 1. Navidrome Configuration
```shell
# 1. 创建目录
sudo mkdir /home/navidrome/music
sudo mkdir /home/navidrome/data

# 2. 运行 Navidrome 容器
sudo docker run -d \
  --name navidrome \
  --restart=unless-stopped \
  -p 4533:4533 \
  -v /home/navidrome/music:/music \
  -v /home/navidrome/data:/data \
  -e ND_LOGLEVEL=info \
  deluan/navidrome:latest

# 3. 检查运行状态
sudo docker ps
# 应该能看到 navidrome 容器正在运行

# 4. 查看日志（确保没有错误）
sudo docker logs navidrome

```

正常来说，到这里navidrome就算是安装完了。docker真是一个伟大的发明。


## 2. Firewall

```bash
# Ubuntu 20.04 通常使用 ufw

# 1. 检查防火墙状态
sudo ufw status

# 2. 如果防火墙是激活的，开放端口
sudo ufw allow 4533/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 3. 如果使用云服务器，还需要在云服务商控制台开放这些端口

```

现在可以通过 `http://你的服务器IP:4533` 访问了！



## 3. HTTPS

使用域名 + Nginx + Let's Encrypt
```bash
# 1. 安装 Nginx
sudo apt update
sudo apt install nginx -y

# 2. 安装 Certbot（用于获取免费 SSL 证书）
sudo apt install certbot python3-certbot-nginx -y

# 3. 创建 Nginx 配置文件
sudo vim /etc/nginx/sites-available/navidrome
```

**将以下内容粘贴进去**（替换 `music.yourdomain.com` 为实际域名）：

```nginx
server {
    listen 80;
    server_name music.yourdomain.com;  # 改成音乐服务域名

    location / {
        proxy_pass http://127.0.0.1:4533;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket 支持
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # 超时设置
        proxy_read_timeout 300s;
        proxy_send_timeout 300s;
    }
}
```



附：Nginx常用命令
```bash
sudo systemctl start nginx      # 启动服务
sudo systemctl stop nginx       # 停止服务
sudo systemctl restart nginx    # 重启服务
sudo systemctl reload nginx     # 平滑重载配置
sudo systemctl status nginx     # 查看运行状态
```




```bash
# 2. 启用配置（创建软链接）
sudo ln -s /etc/nginx/sites-available/navidrome /etc/nginx/sites-enabled/

# 3. 测试 Nginx 配置（非常重要！）
sudo nginx -t
# 应该显示 "syntax is ok" 和 "test is successful"

# 4. 重新加载 Nginx（不会中断现有服务）
sudo systemctl reload nginx
```


去域名服务提供商修改DNS：

```
类型: A
主机记录: music
记录值: 你的服务器公网 IP
TTL: 默认
```

这样 `music.yourdomain.com` 就指向你的服务器了。




配置 HTTPS（使用 Certbot）:

```bash
# 1. 检查 Certbot 是否已安装
certbot --version

# 如果未安装，执行：
sudo apt update
sudo apt install certbot python3-certbot-nginx -y

# 2. 为 Navidrome 域名获取 SSL 证书
sudo certbot --nginx -d music.yourdomain.com

# 按提示操作：
# - 输入邮箱（如果之前配置过可能会跳过）
# - 同意服务条款（如果之前配置过可能会跳过）
# - 选择 "2" (重定向所有 HTTP 请求到 HTTPS)
```

Certbot 会自动修改你的 Navidrome 配置文件，添加 SSL 相关配置，不会影响其他站点。

完成！现在可以通过 `https://music.yourdomain.com` 访问了。

