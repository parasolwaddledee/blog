---
wiki: ad-ds
title: 安装首台域控制器
---

{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-01.png  %}

## 准备工作

1. 将服务器的计算机名更改为 SRV-SHDC01。
2. 将服务器的 IP 地址设置为 10.0.1.1，子网掩码 255.255.255.0，默认网关 10.0.1.254。
3. 将服务器的首选 DNS 服务器设置为自身的 IP 地址 10.0.1.1，备用 DNS 服务器留空。

## 安装 AD DS 角色

1. 打开 Server Manager，点击 Add roles and features。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-02.png %}
2. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-03.png %}
3. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-04.png %}
4. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-05.png %}
5. 选择 Active Directory Domain Services。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-06.png %}
6. 点击 Add Features。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-07.png %}
7. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-08.png %}
8. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-09.png %}
9. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-10.png %}
10. 点击 Install。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-11.png %}
11. 安装成功，点击 Promote this server to a domain controller。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-12.png %}

## 配置 AD DS 角色

1. 选择 Add a new forest，输入根域名 corp.waddledee.com，点击 Next。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-13.png %}
2. 设置目录服务还原模式的密码，点击 Next。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-14.png %}
3. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-15.png %}
4. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-16.png %}
5. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-17.png %}
6. 点击 Next。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-18.png %}
7. 点击 Install。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-19.png %}
8. 配置成功，服务器将在一分钟内自动重启。
{% image /assets/wiki/ad-ds/install-the-first-domain-controller/install-the-first-domain-controller-20.png %}