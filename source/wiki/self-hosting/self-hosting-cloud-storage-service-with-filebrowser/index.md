---
layout: wiki
wiki: self-hosting
title: 使用 File Browser 自建云盘服务
date: 2022-12-11
references:
  - '[Installation - File Browser](https://filebrowser.org/installation)'
  - '[Compose specification | Docker Documentation](https://docs.docker.com/compose/compose-file/)'
  - '[Getting Started | NGINX](https://www.nginx.com/resources/wiki/start/)'
  - '[User Guide — Certbot 2.2.0 documentation](https://eff-certbot.readthedocs.io/en/stable/using.html#certbot-commands)'
  - '[Command Line Interface - AWS CLI - AWS](https://aws.amazon.com/cli/)'
---

## 前言

一直以来，我都是通过同步本地文件到 AWS S3 的方式实现数据备份的。不过，由于 S3 并不支持在线编辑、图片预览等功能，因此我还需要一款合适的网盘应用来充当中间件。

经过一番探索，我最终锁定了 File Browser。这是一款基于 Go 语言开发的网盘应用，相较于 Minio、Nextcloud 等主流产品，它更加轻量简洁，足够贴合我的需求。

## 准备工作

1. 准备一台 Linux 服务器，我使用的是 Ubuntu Server 22.04 LTS。
2. 配置服务器对外开放 80 和 443 端口，前者是 Certbot 的注册端口，后者是 HTTPS 的默认端口。
3. 在域名提供商处添加一条 DNS A 记录指向服务器的公网 IP 地址，如 `files.waddledee.com`。

## 安装 Docker Engine

1. SSH 远程连接到服务器，更新 apt 包索引：
```bash
sudo apt-get update
```
2. 安装软件包以允许 apt 通过 HTTPS 使用存储库，按 Y 继续：
```bash
sudo apt-get install ca-certificates curl gnupg lsb-release
```
3. 添加 Docker 官方的 GPG 密钥：
```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
4. 配置 stable 版本的 Docker 存储库：
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
5. 再次更新 apt 包索引：
```bash
sudo apt-get update
```
6. 安装 Docker Engine，按 Y 继续：
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
7. 检查 Docker Engine 是否安装成功：
```bash
docker --version
```

## 安装 Docker Compose

1. 下载 stable 版本的 Docker Compose：
```bash
sudo curl -SL https://github.com/docker/compose/releases/download/v2.14.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```
2. 为 Docker Compose 添加执行权限：
```bash
sudo chmod +x /usr/local/bin/docker-compose
```
3. 检查 Docker Compose 是否安装成功：
```bash
docker-compose --version
```

## 配置目录与权限

1. 添加当前用户到 docker 组：
```bash
sudo usermod -aG docker $USER
```
2. 注销并重新登录以使权限生效：
```bash
exit
```
3. 创建用于存放容器数据的目录：
```bash
sudo mkdir /opt/services
```
4. 配置当前用户对该目录的执行权和所有权：
```bash
sudo chmod -R 700 /opt/services
sudo chown -R $USER:$USER /opt/services
```
5. 切换到该目录：
```bash
cd /opt/services
```

## 安装 File Browser

1. 创建用于存放 File Browser 数据的目录：
```bash
mkdir filebrowser
```
2. 创建 File Browser 数据库文件：
```bash
touch filebrowser/filebrowser.db
```
3. 创建 Docker Compose 文件：
```bash
nano docker-compose.yml
```
4. 复制如下内容，按 Ctrl+X 保存并退出，按 Y 确认，按 Enter 返回命令行：
```yml
version: '3'

services:
  filebrowser:
    container_name: services-filebrowser
    image: filebrowser/filebrowser:latest
    volumes:
      - ./filebrowser:/srv
      - ./filebrowser/filebrowser.db:/database.db
    restart: unless-stopped
```
5. 在后台创建并启动 filebrowser 容器：
```bash
docker-compose up -d
```
6. 检查 filebrowser 容器的状态：
```bash
docker ps
```
状态为 healthy 即表示容器运行正常：
```
CONTAINER ID   IMAGE                            COMMAND          CREATED              STATUS                        PORTS                                   NAMES
d7e887b393d6   filebrowser/filebrowser:latest   "/filebrowser"   About a minute ago   Up About a minute (healthy)   0.0.0.0:8080->80/tcp, :::8080->80/tcp   services-filebrowser
```

## 配置 File Browser

### 申请 SSL 证书

1. 编辑 Docker Compose 文件：
```bash
nano docker-compose.yml
```
2. 覆盖如下内容，按 Ctrl+X 保存并退出，按 Y 确认，按 Enter 返回命令行：
```yml
version: '3'

services:
  filebrowser:
    container_name: services-filebrowser
    image: filebrowser/filebrowser:latest
    volumes:
      - ./filebrowser:/srv
      - ./filebrowser/filebrowser.db:/database.db
    restart: unless-stopped

  nginx:
    container_name: services-nginx
    image: nginx:latest
    ports:
      - 80:80
    volumes:
      - ./nginx:/etc/nginx/conf.d
      - ./certbot/conf:/etc/nginx/ssl
      - ./certbot/data:/var/www/certbot
    restart: unless-stopped

  certbot:
    container_name: services-certbot
    image: certbot/certbot:latest
    command: certonly --webroot --webroot-path=/var/www/certbot --email parasol@waddledee.com --agree-tos --no-eff-email --keep-until-expiring -d files.waddledee.com
    volumes:
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/data:/var/www/certbot
      - ./certbot/logs:/var/log/letsencrypt
```
{% box color:blue %}
NGINX 在这里的作用是通过其侦听的 80 端口帮助 Certbot 完成 ACME Challenge。
{% endbox %}
3. 创建用于存放 NGINX 配置的目录：
```bash
mkdir nginx
```
4. 创建用于 ACME Challenge 的 NGINX 配置文件：
```bash
nano nginx/certbot.conf
```
5. 复制如下内容，按 Ctrl+X 保存并退出，按 Y 确认，按 Enter 返回命令行：
```nginx
server {
    listen 80;

    location ~ /.well-known/acme-challenge {
        allow all;
        root /var/www/certbot;
    }
}
```
6. 在后台创建并启动 NGINX 和 Certbot 容器：
```bash
docker-compose up -d
```
7. 检查 NGINX 和 Certbot 容器的状态：
```bash
docker ps -a
```
NGINX 的状态为 Up，Certbot 的状态为 Exited (0) 即表示容器运行正常：
```
CONTAINER ID   IMAGE                            COMMAND                   CREATED              STATUS                          PORTS                                                                      NAMES
ee83f9c68c64   nginx:latest                     "/docker-entrypoint.…"   About a minute ago   Up About a minute               0.0.0.0:80->80/tcp, :::80->80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp   services-nginx
a70f71af35e2   certbot/certbot:latest           "certbot certonly --…"   About a minute ago   Exited (0) About a minute ago                                                                              services-certbot
d7e887b393d6   filebrowser/filebrowser:latest   "/filebrowser"            About a minute ago   Up About a minute (healthy)     0.0.0.0:8080->80/tcp, :::8080->80/tcp                                      services-filebrowser
```
8. 查看 Certbot 日志以确认证书申请结果：
```bash
cat certbot/logs/letsencrypt.log
```
看到 Successfully received certificate 即表示证书申请成功：
```
...
2022-12-11 13:42:51,842:DEBUG:certbot._internal.display.obj:Notifying user:
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/files.waddledee.com/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/files.waddledee.com/privkey.pem
This certificate expires on 2023-03-11.
These files will be updated when the certificate renews.
...
```

### 绑定 SSL 证书

1. 编辑 Docker Compose 文件：
```bash
nano docker-compose.yml
```
2. 覆盖如下内容，按 Ctrl+X 保存并退出，按 Y 确认，按 Enter 返回命令行：
```yml
version: '3'

services:
  filebrowser:
    container_name: services-filebrowser
    image: filebrowser/filebrowser:latest
    volumes:
      - ./filebrowser:/srv
      - ./filebrowser/filebrowser.db:/database.db
    restart: unless-stopped

  nginx:
    container_name: services-nginx
    image: nginx:latest
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./nginx:/etc/nginx/conf.d
      - ./certbot/conf:/etc/nginx/ssl
      - ./certbot/data:/var/www/certbot
    restart: unless-stopped

  certbot:
    container_name: services-certbot
    image: certbot/certbot:latest
    command: certonly --webroot --webroot-path=/var/www/certbot --email parasol@waddledee.com --agree-tos --no-eff-email --keep-until-expiring -d files.waddledee.com
    volumes:
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/data:/var/www/certbot
      - ./certbot/logs:/var/log/letsencrypt
```
3. 添加当前用户对证书所在目录的读取权限：
```bash
sudo chmod -R 755 certbot/conf/live
```
4. 创建 File Browser 的 NGINX 配置文件：
```bash
nano nginx/filebrowser.conf
```
5. 复制如下内容，按 Ctrl+X 保存并退出，按 Y 确认，按 Enter 返回命令行：
```nginx
server {
    listen 80;
    server_name files.waddledee.com;

    location / {
        return 301 https://files.waddledee.com$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name files.waddledee.com;

    ssl_certificate /etc/nginx/ssl/live/files.waddledee.com/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/live/files.waddledee.com/privkey.pem;

    location / {
        proxy_pass http://services-filebrowser:80;
        client_max_body_size 0;
    }
}
```
6. 在后台创建并启动所有容器：
```bash
docker-compose up -d
```
7. 检查所有容器的状态：
```bash
docker ps -a
```
确认所有容器运行正常：
```
CONTAINER ID   IMAGE                            COMMAND                   CREATED              STATUS                          PORTS                                                                      NAMES
ee83f9c68c64   nginx:latest                     "/docker-entrypoint.…"   About a minute ago   Up About a minute               0.0.0.0:80->80/tcp, :::80->80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp   services-nginx
a70f71af35e2   certbot/certbot:latest           "certbot certonly --…"   About a minute ago   Exited (0) About a minute ago                                                                              services-certbot
d7e887b393d6   filebrowser/filebrowser:latest   "/filebrowser"            About a minute ago   Up About a minute (healthy)     0.0.0.0:8080->80/tcp, :::8080->80/tcp                                      services-filebrowser
```

### 应用更新与证书续订

1. 创建更新脚本：
```bash
nano upgrade.sh
```
2. 复制如下内容，按 Ctrl+X 保存并退出，按 Y 确认，按 Enter 返回命令行：
```
#!/bin/bash
cd /opt/services
docker-compose pull
docker-compose up -d --remove-orphans
docker image prune -af
```
3. 为脚本添加执行权限：
```bash
chmod 700 upgrade.sh
```
4. 打开 crontab 配置文件，首次打开时选择一款编辑器：
```bash
crontab -e
```
5. 配置 cron 任务，完成后按 Ctrl+X 保存并退出，按 Y 确认，按 Enter 返回命令行。以下示例配置为每周一 00:00 触发更新脚本，并将日志输出到 /opt/services/upgrade.log：
```
0 0 * * 1 /opt/services/upgrade.sh >/opt/services/upgrade.log 2>&1
```
6. 检查脚本是否执行成功：
```bash
cat /opt/services/upgrade.log
```
成功的输出结果如下所示：
```
nginx Pulling
certbot Pulling
filebrowser Pulling
nginx Pulled
certbot Pulled
filebrowser Pulled
Container nginx  Running
Container certbot  Created
Container filebrowser  Running
Container certbot  Starting
Container certbot  Started
Total reclaimed space: 0B
```
7. 查看 Certbot 日志以确认证书续订情况：
```bash
cat certbot/logs/letsencrypt.log
```
因为我在 Certbot 命令中指定了 \-\-keep-until-expiring 参数，所以证书会在到期时自动续订：
```
...
2022-12-11 13:51:18,686:DEBUG:certbot._internal.display.obj:Notifying user: Certificate not yet due for renewal
2022-12-11 13:51:18,686:INFO:certbot._internal.main:Keeping the existing certificate
2022-12-11 13:51:18,686:DEBUG:certbot._internal.display.obj:Notifying user: Certificate not yet due for renewal; no action taken.
...
```

### 更改管理员密码

1. 访问 `https://files.waddledee.com`，登录默认的管理员账号，用户名和密码都是 admin：
{% image /assets/wiki/self-hosting/self-hosting-cloud-storage-service-with-filebrowser/self-hosting-cloud-storage-service-with-filebrowser-01.jpg %}
2. 点击 Settings，输入新的管理员密码后点击 UPDATE：
{% image /assets/wiki/self-hosting/self-hosting-cloud-storage-service-with-filebrowser/self-hosting-cloud-storage-service-with-filebrowser-02.jpg %}

## 备份 File Browser

最后，我将 File Browser 的文件同步到 AWS S3 实现数据备份。

{% box color:warning %}
此方法仅适用于 File Browser 服务器是一台 EC2 实例，且与 S3 存储桶位于同一 AWS 账号下的情况，其他 VPS 请使用 [Access Keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html) 方法。
{% endbox %}

### 配置 AWS 服务

1. 登录 [AWS Console](https://console.aws.amazon.com/console/home) 并切换到支持 AWS CloudShell 的区域，如 `N.Virginia`，随后打开 CloudShell：
{% image /assets/wiki/self-hosting/self-hosting-cloud-storage-service-with-filebrowser/self-hosting-cloud-storage-service-with-filebrowser-03.jpg %} 
2. 创建一个 S3 存储桶：
```bash
aws s3api create-bucket --bucket filebrowser --create-bucket-configuration LocationConstraint=ap-east-1 --region ap-east-1
```
{% box color:blue %}
S3 存储桶的名称在全球范围内是唯一的，请注意替换上面 \-\-bucket 的值。
{% endbox %}
3. 阻止该 S3 存储桶的公共访问：
```bash
aws s3api put-public-access-block --bucket filebrowser --public-access-block-configuration '{"BlockPublicAcls": true, "IgnorePublicAcls": true, "BlockPublicPolicy": true, "RestrictPublicBuckets": true}' --region ap-east-1
```
4. 创建一个 IAM 角色：
```bash
aws iam create-role --role-name filebrowser --assume-role-policy-document '{"Version": "2012-10-17", "Statement": [{"Effect": "Allow", "Principal": {"Service": "ec2.amazonaws.com"}, "Action": "sts:AssumeRole"}]}'
```
5. 赋予该角色读写 S3 存储桶的权限：
```bash
aws iam put-role-policy --role-name filebrowser --policy-name filebrowser --policy-document '{"Version": "2012-10-17", "Statement": [{"Effect": "Allow", "Action": ["s3:ListBucket", "s3:GetObject", "s3:PutObject", "s3:DeleteObject"], "Resource": ["arn:aws:s3:::filebrowser", "arn:aws:s3:::filebrowser/*"]}]}'
```
6. 创建一个 IAM 实例配置文件，它的作用是将 IAM 角色传递给 EC2 实例：
```bash
aws iam create-instance-profile --instance-profile-name filebrowser
```
7. 添加 IAM 角色到 IAM 实例配置文件：
```bash
aws iam add-role-to-instance-profile --role-name filebrowser --instance-profile-name filebrowser
```
8. 关联 IAM 实例配置文件与 EC2 实例：
```bash
aws ec2 associate-iam-instance-profile --iam-instance-profile Name=filebrowser --instance-id i-1234567890abcdefg --region ap-east-1
```

### 同步到 AWS S3

1. 返回服务器 SSH 远程连接窗口，安装 unzip：
```bash
sudo apt install unzip
```
2. 下载、解压并安装 AWS CLI：
```bash
curl https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip -o awscliv2.zip
unzip awscliv2.zip
sudo ./aws/install
```
3. 将 File Browser 文件同步到 S3 存储桶：
```bash
aws s3 sync /opt/services/filebrowser s3://filebrowser --delete
```
4. 打开 crontab 配置文件：
```bash
crontab -e
```
5. 添加 cron 任务，完成后按 Ctrl+X 保存并退出，按 Y 确认，按 Enter 返回命令行。以下示例配置为每 15 分钟和 S3 存储桶同步一次：
```
*/15 * * * * aws s3 sync /opt/services/filebrowser s3://filebrowser --delete
```
