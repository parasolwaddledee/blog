---
wiki: ad-ds
comment_id: /wiki/ad-ds/
title: 安装辅域控制器
date: 2024-04-24
---

{% image /assets/wiki/ad-ds/install-the-backup-domain-controller/install-the-backup-domain-controller-01.png 辅域控制器 (Backup Domain Controller, BDC) 又称备份域控制器，通常是指除了主域控制器之外的其他域控制器。 %}

## 准备工作

1. 将服务器的计算机名更改为 SRV-SHDC02。
2. 将服务器的 IP 地址设置为 10.0.1.2，子网掩码 255.255.255.0，默认网关 10.0.1.254。
3. 将服务器的首选 DNS 服务器设置为主域控制器的 IP 地址 10.0.1.1。
4. 安装 Active Directory 域服务，步骤参考[这里](/wiki/ad-ds/install-the-primary-domain-controller/#安装-Active-Directory-域服务)。

## 配置 Active Directory 域服务

1. 输入域名 corp.waddledee.com，点击 Select。
{% image /assets/wiki/ad-ds/install-the-backup-domain-controller/install-the-backup-domain-controller-02.png %}
2. 输入域管理员账号 administrator@corp.waddledee.com 和密码，点击 OK。
{% image /assets/wiki/ad-ds/install-the-backup-domain-controller/install-the-backup-domain-controller-03.png %}
3. 选择域名 corp.waddledee.com，点击 OK。
{% image /assets/wiki/ad-ds/install-the-backup-domain-controller/install-the-backup-domain-controller-04.png %}
4. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-backup-domain-controller/install-the-backup-domain-controller-05.png %}
5. 设置目录服务还原模式的密码，点击 Next。
{% image /assets/wiki/ad-ds/install-the-backup-domain-controller/install-the-backup-domain-controller-06.png %}
6. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-backup-domain-controller/install-the-backup-domain-controller-07.png %}
7. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-backup-domain-controller/install-the-backup-domain-controller-08.png %}
8. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-backup-domain-controller/install-the-backup-domain-controller-09.png %}
9.  点击 Next。
{% image /assets/wiki/ad-ds/install-the-backup-domain-controller/install-the-backup-domain-controller-10.png %}
10. 点击 Install。
{% image /assets/wiki/ad-ds/install-the-backup-domain-controller/install-the-backup-domain-controller-11.png %}
11. 配置成功，服务器将在一分钟内自动重启。
{% image /assets/wiki/ad-ds/install-the-backup-domain-controller/install-the-backup-domain-controller-12.png %}
12. 使用相同的方法将深圳站点的两台服务器提升为辅域控制器。
{% image /assets/wiki/ad-ds/install-the-backup-domain-controller/install-the-backup-domain-controller-13.png %}
