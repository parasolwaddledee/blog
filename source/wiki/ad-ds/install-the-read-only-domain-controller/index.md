---
wiki: ad-ds
comment_id: /wiki/ad-ds/
title: 安装只读域控制器
date: 2024-04-25
---

{% image /assets/wiki/ad-ds/install-the-read-only-domain-controller/install-the-read-only-domain-controller-01.png 只读域控制器 (Read-Only Domain Controller, RODC) 是一种承载 AD 数据库只读分区的域控制器，适用于分支机构。 %}

## 准备工作

1. 将服务器的计算机名更改为 SRV-DGRODC。
2. 将服务器的 IP 地址设置为 10.0.3.1，子网掩码 255.255.255.0，默认网关 10.0.3.254。
3. 将服务器的首选 DNS 服务器设置为主域控制器的 IP 地址 10.0.1.1。
4. 安装 Active Directory 域服务，步骤参考[这里](/wiki/ad-ds/install-the-primary-domain-controller/#安装-Active-Directory-域服务)。

## 配置 Active Directory 域服务

1. 输入域名 corp.waddledee.com，点击 Select。
{% image /assets/wiki/ad-ds/install-the-read-only-domain-controller/install-the-read-only-domain-controller-02.png %}
2. 输入域管理员账号 administrator@corp.waddledee.com 和密码，点击 OK。
{% image /assets/wiki/ad-ds/install-the-read-only-domain-controller/install-the-read-only-domain-controller-03.png %}
3. 选择域名 corp.waddledee.com，点击 OK。
{% image /assets/wiki/ad-ds/install-the-read-only-domain-controller/install-the-read-only-domain-controller-04.png %}
4. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-read-only-domain-controller/install-the-read-only-domain-controller-05.png %}
5. 选择 Read only domain controller (RODC)，设置目录服务还原模式的密码，点击 Next。
{% image /assets/wiki/ad-ds/install-the-read-only-domain-controller/install-the-read-only-domain-controller-06.png %}
6. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-read-only-domain-controller/install-the-read-only-domain-controller-07.png %}
7. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-read-only-domain-controller/install-the-read-only-domain-controller-08.png %}
8. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-read-only-domain-controller/install-the-read-only-domain-controller-09.png %}
9. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-read-only-domain-controller/install-the-read-only-domain-controller-10.png %}
10. 点击 Install。
{% image /assets/wiki/ad-ds/install-the-read-only-domain-controller/install-the-read-only-domain-controller-11.png %}
11. 配置成功，服务器将在一分钟内自动重启。
{% image /assets/wiki/ad-ds/install-the-read-only-domain-controller/install-the-read-only-domain-controller-12.png %}
