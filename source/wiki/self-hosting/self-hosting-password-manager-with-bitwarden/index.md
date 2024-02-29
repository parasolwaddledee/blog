---
layout: wiki
wiki: self-hosting
title: 使用 Bitwarden 自建密码管理器
date: 2022-12-18
references:
  - '[Install and Deploy - Linux | Bitwarden Help Center](https://bitwarden.com/help/install-on-premise-linux/)'
---

## 前言

大家平时都是如何管理自己的账号密码的？是使用浏览器自带的密码管理器？还是购买诸如 LastPass、1Password 之类的专业密码管理器？你真的放心将账号密码这么重要的数据存放在别人那里吗？

带着各种疑问，我找到了 Bitwarden。这是一款开源的跨平台密码管理器，支持所有主流浏览器与操作系统，最重要的是它支持自托管。从某种程度上来说，自托管能够有效规避大规模的密码泄露事件。

## 准备工作

1. 准备一台内存 ≥ 2GB 的 Linux 服务器，我使用的是 Ubuntu Server 22.04 LTS。
2. 配置服务器对外开放 80 和 443 端口，前者是 Certbot 的注册端口，后者是 HTTPS 的默认端口。
3. 在域名提供商处添加一条 DNS A 记录指向服务器的公网 IP 地址，如 `vault.waddledee.com`。
4. 访问 [https://bitwarden.com/host](https://bitwarden.com/host)，申请自托管 Bitwarden 所需的安装 ID 和 Key：
{% image /assets/wiki/self-hosting/self-hosting-password-manager-with-bitwarden/self-hosting-password-manager-with-bitwarden-01.jpg %}

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
sudo curl -SL https://github.com/docker/compose/releases/download/v2.14.1/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```
2. 为 Docker Compose 添加执行权限：
```bash
sudo chmod +x /usr/local/bin/docker-compose
```
3. 检查 Docker Compose 是否安装成功：
```bash
docker-compose --version
```

## 创建服务账户

创建一个专门用于运行 Bitwarden 的服务账户，目的是为了隔离 Bitwarden 实例。

1. 创建用户 bitwarden，输入密码和基本信息后按 Y 确认：
```bash
sudo adduser bitwarden
```
2. 添加该用户到 docker 组：
```bash
sudo usermod -aG docker bitwarden
```
3. 创建目录 /opt/bitwarden：
```bash
sudo mkdir /opt/bitwarden
```
4. 配置该目录的执行权和所有权：
```bash
sudo chmod -R 700 /opt/bitwarden
sudo chown -R bitwarden:bitwarden /opt/bitwarden
```
5. 切换到 bitwarden 用户：
```bash
su bitwarden
```
6. 切换到 /opt/bitwarden 目录：
```bash
cd /opt/bitwarden
```

## 安装 Bitwarden

1. 下载 Bitwarden 安装脚本并为其添加执行权限：
```bash
curl -Lso bitwarden.sh https://go.btwrdn.co/bw-sh && chmod 700 bitwarden.sh
```
2. 安装 Bitwarden：
```bash
./bitwarden.sh install
```
安装过程需要输入参数，如下所示：
{% box %}
- Enter the domain name for your Bitwarden instance (ex. bitwarden.example.com):
输入在准备工作中事先配置的域名 `vault.waddledee.com`。
- Do you want to use Let's Encrypt to generate a free SSL certificate? (y/n):
选择是否使用 Let's Encrypt 的 SSL 证书，输入 y。
- Enter your email address (Let's Encrypt will send you certificate expiration reminders):
输入用于接收 Let's Encrypt 证书过期提醒的邮箱，如 `parasol@waddledee.com`。
- Enter the database name for your Bitwarden instance (ex. vault):
输入新 Bitwarden 数据库的名称，如 `vault`。
- Enter your installation id (get at https[]()://bitwarden.com/host):
输入在准备工作中事先申请的安装 ID `76a32426-a6fb-4fde-b4c6-af780093ddc4`。
- Enter your installation key:
输入在准备工作中事先申请的安装 Key `AcTf08x2O22T8HmaAUcY`。
{% endbox %}
3. 启动 Bitwarden：
```bash
./bitwarden.sh start
```
4. 检查 Bitwarden 容器的状态：
```bash
docker ps
```
首次启动需要花费一些时间，耐心等待所有容器的状态变为 healthy：
```
CONTAINER ID   IMAGE                               COMMAND            CREATED              STATUS                        PORTS                                                                                    NAMES
a93f8cf12ba8   bitwarden/nginx:2022.12.0           "/entrypoint.sh"   About a minute ago   Up About a minute (healthy)   80/tcp, 0.0.0.0:80->8080/tcp, :::80->8080/tcp, 0.0.0.0:443->8443/tcp, :::443->8443/tcp   bitwarden-nginx
b1a6ba29fd34   bitwarden/admin:2022.12.0           "/entrypoint.sh"   About a minute ago   Up About a minute (healthy)   5000/tcp                                                                                 bitwarden-admin
2522752d32a0   bitwarden/api:2022.12.0             "/entrypoint.sh"   About a minute ago   Up About a minute (healthy)   5000/tcp                                                                                 bitwarden-api
07ebd607fd75   bitwarden/identity:2022.12.0        "/entrypoint.sh"   About a minute ago   Up About a minute (healthy)   5000/tcp                                                                                 bitwarden-identity
aa4f48cef3af   bitwarden/web:2022.12.0             "/entrypoint.sh"   About a minute ago   Up About a minute (healthy)                                                                                            bitwarden-web
0b4ee0c29c52   bitwarden/notifications:2022.12.0   "/entrypoint.sh"   About a minute ago   Up About a minute (healthy)   5000/tcp                                                                                 bitwarden-notifications
659f1f3e5556   bitwarden/mssql:2022.12.0           "/entrypoint.sh"   About a minute ago   Up About a minute (healthy)                                                                                            bitwarden-mssql
319543e05b88   bitwarden/sso:2022.12.0             "/entrypoint.sh"   About a minute ago   Up About a minute (healthy)   5000/tcp                                                                                 bitwarden-sso
0125ba5e879a   bitwarden/icons:2022.12.0           "/entrypoint.sh"   About a minute ago   Up About a minute (healthy)   5000/tcp                                                                                 bitwarden-icons
0ed826f85d4a   bitwarden/events:2022.12.0          "/entrypoint.sh"   About a minute ago   Up About a minute (healthy)   5000/tcp                                                                                 bitwarden-events
0f2879b8c9e8   bitwarden/attachments:2022.12.0     "/entrypoint.sh"   About a minute ago   Up About a minute (healthy)                                                                                            bitwarden-attachments
```

## 配置 Bitwarden

1. 打开 Bitwarden 环境文件：
```bash
nano ./bwdata/env/global.override.env
```
2. 配置下列用于发送邮件的参数，完成后按 Ctrl+X 保存并退出，按 Y 确认，按 Enter 返回命令行：
```
...
globalSettings__mail__replyToEmail=no-reply@vault.waddledee.com
globalSettings__mail__smtp__host=REPLACE
globalSettings__mail__smtp__port=587
globalSettings__mail__smtp__ssl=false
globalSettings__mail__smtp__username=REPLACE
globalSettings__mail__smtp__password=REPLACE
...
```
3. 重启 Bitwarden 以应用最新配置：
```bash
./bitwarden.sh restart
```
4. 访问 Bitwarden 网页版 `https://vault.waddledee.com/`，注册一个新账户：
{% image /assets/wiki/self-hosting/self-hosting-password-manager-with-bitwarden/self-hosting-password-manager-with-bitwarden-02.jpg %}
5. 注册成功后登录，点击右上角的 Send Email 验证邮箱：
{% image /assets/wiki/self-hosting/self-hosting-password-manager-with-bitwarden/self-hosting-password-manager-with-bitwarden-03.jpg %}
6. 收到邮件后点击 Verify Email Address Now 激活账户：
{% image /assets/wiki/self-hosting/self-hosting-password-manager-with-bitwarden/self-hosting-password-manager-with-bitwarden-04.jpg %}

至此，Bitwarden 的基本配置已全部完成，接下来可以开始正常使用了。

## 配置客户端

{% tabs %}
<!-- tab Edge 浏览器 -->
1. 安装浏览器插件 Bitwarden - Free Password Manager：
{% link https://microsoftedge.microsoft.com/addons/detail/bitwarden-free-password/jbkfoedolllekgbhcbcoahefnbanhhlh %}
2. 打开插件，点击左上角的设置图标，输入 Server URL 保存后即可登录：
{% grid %}
<!-- cell -->
{% image /assets/wiki/self-hosting/self-hosting-password-manager-with-bitwarden/self-hosting-password-manager-with-bitwarden-05.jpg %}
<!-- cell -->
{% image /assets/wiki/self-hosting/self-hosting-password-manager-with-bitwarden/self-hosting-password-manager-with-bitwarden-06.jpg %}
{% endgrid %}
<!-- tab iOS 客户端 -->
1. 安装应用程序 Bitwarden Password Manager：
{% link https://apps.apple.com/us/app/bitwarden-password-manager/id1137397744 %}
2. 打开 APP，点击右上角的设置图标，输入 Server URL 保存后即可登录：
{% grid %}
<!-- cell -->
{% image /assets/wiki/self-hosting/self-hosting-password-manager-with-bitwarden/self-hosting-password-manager-with-bitwarden-07.jpg %}
<!-- cell -->
{% image /assets/wiki/self-hosting/self-hosting-password-manager-with-bitwarden/self-hosting-password-manager-with-bitwarden-08.jpg %}
{% endgrid %}
{% endtabs %}

## 更新 Bitwarden

保持 Bitwarden 服务器的最新状态对密码库的安全至关重要，我使用 cron 任务定期更新 Bitwarden。

1. 创建 Bitwarden 更新脚本：
```bash
nano ./bwdata/scripts/updatebw.sh
```
2. 复制如下内容，按 Ctrl+X 保存并退出，按 Y 确认，按 Enter 返回命令行：
```
#!/bin/bash
/opt/bitwarden/bitwarden.sh updateself
/opt/bitwarden/bitwarden.sh update
```
3. 为脚本添加执行权限：
```bash
chmod 700 ./bwdata/scripts/updatebw.sh
```
4. 打开 crontab 配置文件，首次打开时选择一款编辑器：
```bash
crontab -e
```
5. 配置 cron 任务，完成后按 Ctrl+X 保存并退出，按 Y 确认，按 Enter 返回命令行。以下示例配置为每周一 00:00 触发 Bitwarden 更新脚本，并将日志输出到 /home/bitwarden/updatebw.log：
```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
0 0 * * 1 /opt/bitwarden/bwdata/scripts/updatebw.sh >/home/bitwarden/updatebw.log 2>&1
```
6. 检查脚本是否执行成功：
```bash
cat /home/bitwarden/updatebw.log
```
成功的输出结果如下所示：
```
 _     _ _                         _
| |__ (_) |___      ____ _ _ __ __| | ___ _ __
| '_ \| | __\ \ /\ / / _` | '__/ _` |/ _ \ '_ \
| |_) | | |_ \ V  V / (_| | | | (_| |  __/ | | |
|_.__/|_|\__| \_/\_/ \__,_|_|  \__,_|\___|_| |_|

Open source password management solutions
Copyright 2015-2022, 8bit Solutions LLC
https://bitwarden.com, https://github.com/bitwarden

===================================================

bitwarden.sh version 2022.12.0
Docker version 20.10.22, build 3a2c30b
Docker Compose version v2.14.1

Updated self.
 _     _ _                         _
| |__ (_) |___      ____ _ _ __ __| | ___ _ __
| '_ \| | __\ \ /\ / / _` | '__/ _` |/ _ \ '_ \
| |_) | | |_ \ V  V / (_| | | | (_| |  __/ | | |
|_.__/|_|\__| \_/\_/ \__,_|_|  \__,_|\___|_| |_|

Open source password management solutions
Copyright 2015-2022, 8bit Solutions LLC
https://bitwarden.com, https://github.com/bitwarden

===================================================

bitwarden.sh version 2022.12.0
Docker version 20.10.22, build 3a2c30b
Docker Compose version v2.14.1

Update not needed
```

## 备份 Bitwarden

如果不慎丢失了 Bitwarden 密码库，那将是一件非常头疼的事情，因此我们需要定期备份 Bitwarden。下面提供了三种不同的备份方案，选择最适合自己的一种即可：

1. 定期备份 Bitwarden 服务器，在必要时还原服务器快照。
2. 定期备份 Bitwarden 数据库，在必要时还原数据库，方法可参考：
{% link https://bitwarden.com/help/backup-on-premise/ %}
3. 定期导出 Bitwarden 密码库，推荐通过 CLI 编写脚本的方式实现，方法可参考：
{% link https://bitwarden.com/help/export-your-data/ %}

## 更多安全性建议

1. 如果是仅供个人使用的 Bitwarden 服务器，则建议关闭新用户注册功能。具体方法是将 Bitwarden 环境文件的 globalSettings__disableUserRegistration 设置为 true 并重启 Bitwarden。
2. 为 Bitwarden 账户启用 2FA 可以有效防止由主密码泄露引起的密码库泄露，方法可参考：
{% link https://bitwarden.com/help/setup-two-step-login/ %}
