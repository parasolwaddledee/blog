---
wiki: ad-ds
comment_id: /wiki/ad-ds/
title: 配置站点复制
date: 2024-04-26
---

{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-01.png 根据架构设计配置域控制器在站点内外的复制关系，实现客户端就近登录。 %}

## 配置站点

1. 打开 Active Directory Sites and Services，选择 Sites，点击 New Site。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-02.png %}
2. 输入上海站点的站点名称 SH，选择默认的站点链接 DEFAULTIPSITELINK，点击 OK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-03.png width:65% %}
3. 点击 OK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-04.png width:65% %}
4. 站点创建成功，使用相同的方法创建深圳站点 SZ 和东莞站点 DG。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-05.png %}

## 配置子网

1. 选择 Sites > Subnets，点击 New Subnet。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-06.png %}
2. 输入上海站点的服务器网段 10.0.1.0/24，选择上海站点 SH，点击 OK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-07.png width:65% %}
3. 子网创建成功，使用相同的方法创建所有服务器网段和客户端网段的子网。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-08.png %}

## 配置站点链接

1. 选择 Sites > Inter-Site Transports > IP，点击 New Site Link。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-09.png %}
2. 输入上海和深圳站点的站点链接名称 SH-SZ，添加上海站点 SH 和深圳站点 SZ，点击 OK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-10.png width:65% %}
3. 选择站点链接 SH-SZ，点击 Properties。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-11.png %}
4. 将站点链接的复制间隔设置为最小值 15 分钟，点击 OK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-12.png width:65% %}
5. 站点链接创建成功，使用相同的方法创建上海和东莞站点的站点链接 SH-DG。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-13.png %}
6. 选择 Sites > Inter-Site Transports > IP > DEFAULTIPSITELINK，点击 Delete。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-14.png %}
7. 点击 Yes 删除默认的站点链接 DEFAULTIPSITELINK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-15.png width:65% %}
8. 站点链接配置完成。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-16.png %}

## 移动服务器

1. 选择 Sites > Default-First-Site-Name > Servers > SRV-SHDC01，点击 Move。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-17.png %}
2. 选择上海站点 SH，点击 OK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-18.png width:65% %}
3. 服务器移动成功，使用相同的方法将所有服务器移动到对应的站点。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-19.png %}
4. 选择 Sites > Default-First-Site-Name，点击 Delete。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-20.png %}
5. 点击 Yes 删除默认的站点 Default-First-Site-Name。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-21.png width:65% %}
6. 选择 Use Delete Subtree server control，点击 Yes。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-22.png width:65% %}
7. 服务器移动完成。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-23.png %}

## 配置桥头服务器

1. 选择 Sites > SH > Servers > SRV-SHDC01，点击 Properties。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-24.png %}
2. 添加传输协议 IP，点击 OK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-25.png width:65% %}
3. 上海站点的桥头服务器配置成功。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-26.png %}
4. 使用相同的方法将 SRV-SZDC01 配置为深圳站点的桥头服务器。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-27.png %}

## 检查复制拓扑

1. 选择 Sites > SH > Servers > SRV-SHDC01 > NTDS Settings，点击 All Tasks > Check Replication Topology。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-28.png %}
2. 点击 OK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-29.png width:65% %}
3. 选择 Sites > SH > Servers > SRV-SHDC01 > NTDS Settings > \<automatically generated\>，点击 Replicate Now。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-30.png %}
4. 对所有的服务器和连接重复执行以上操作，直至自动生成的连接与架构设计完全匹配。
{% swiper width:max %}
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-31.png %}
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-32.png %}
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-33.png %}
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-34.png %}
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-35.png %}
{% endswiper %}

## 验证就近登录

1. 登录上海站点的客户端 WKS-SHOFC01，显示已连接到上海站点的域控制器 SRV-SHDC01。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-36.png %}
2. 登录深圳站点的客户端 WKS-SZOFC01，显示已连接到深圳站点的域控制器 SRV-SZDC01。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-37.png %}

## 配置密码复制策略

1. 为了让用户在没有可写域控制器的环境下也能登录只读域控制器，需要预先在只读域控制器上存储用户及其计算机账户的密码。创建两个安全组，分别用于存放授权允许将密码复制到只读域控制器的用户和计算机账户，然后将东莞站点的用户和计算机账户添加到对应的安全组中。
{% grid %}
<!-- cell -->
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-38.png %}
<!-- cell -->
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-39.png %}
{% endgrid %}
2. 打开 Active Directory Users and Computers，选择 corp.waddledee.com > Domain Controllers，打开只读域控制器 SRV-DGRODC。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-40.png %}
3. 选择 Password Replication Policy 选项卡，打开授权允许将密码复制到只读域控制器的内置安全组 Allowed RODC Password Replication Group。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-41.png width:65% %}
4. 添加刚才创建的安全组 USG-RODC-PWDREPL-ALLOW 和 CSG-RODC-PWDREPL-ALLOW。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-42.png width:65% %}
5. 重启东莞站点的客户端 WKS-DGFAC01，登录后显示已连接到只读域控制器 SRV-DGRODC。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-43.png %}
6. 返回 Password Replication Policy 选项卡，点击 Advanced。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-44.png width:65% %}
7. 可以看到东莞站点的用户及其计算机账户的密码已经预先存储在只读域控制器上了。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-45.png %}
