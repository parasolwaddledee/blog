---
wiki: ad-ds
title: 配置站点复制
---

## 配置站点

1. 打开 Active Directory Sites and Services，选择 Sites，点击 New Site。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-01.png %}
2. 输入上海站点的站点名称 SH，选择默认的站点链接 DEFAULTIPSITELINK，点击 OK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-02.png width:75% %}
3. 点击 OK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-03.png width:75% %}
4. 站点创建成功，使用相同的方法创建深圳站点 SZ 和东莞站点 DG。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-04.png %}

## 配置子网

1. 选择 Sites > Subnets，点击 New Subnet。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-05.png %}
2. 输入上海站点的服务器网段 10.0.1.0/24，选择上海站点 SH，点击 OK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-06.png width:75% %}
3. 子网创建成功，使用相同的方法创建所有站点服务器网段和客户端网段的子网。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-07.png %}

## 配置站点链接

1. 选择 Sites > Inter-Site Transports > IP，点击 New Site Link。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-08.png %}
2. 输入上海和深圳站点的站点链接名称 SH-SZ，添加上海站点 SH 和深圳站点 SZ，点击 OK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-09.png width:75% %}
3. 站点链接创建成功，使用相同的方法创建上海站点和东莞站点的站点链接 SH-DG。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-10.png %}
4. 选择 Sites > Inter-Site Transports > IP > DEFAULTIPSITELINK，点击 Delete。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-11.png %}
5. 点击 Yes 删除默认的站点链接 DEFAULTIPSITELINK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-12.png width:75% %}
6. 站点链接配置完成。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-13.png %}

## 移动服务器

1. 选择 Sites > Default-First-Site-Name > Servers > SRV-SHDC01，点击 Move。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-14.png %}
2. 选择上海站点 SH，点击 OK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-15.png width:75% %}
3. 服务器移动成功，使用相同的方法将所有服务器移动到对应的站点。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-16.png %}
4. 选择 Sites > Default-First-Site-Name，点击 Delete。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-17.png %}
5. 点击 Yes 删除默认的站点 Default-First-Site-Name。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-18.png width:75% %}
6. 选择 Use Delete Subtree server control，点击 Yes。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-19.png width:75% %}
7. 服务器移动完成。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-20.png %}

## 配置桥头服务器

1. 选择 Sites > SH > Servers > SRV-SHDC01，点击 Properties。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-21.png %}
2. 添加传输协议 IP，点击 OK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-22.png width:75% %}
3. 上海站点的桥头服务器配置成功。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-23.png %}
4. 使用相同的方法将 SRV-SZDC01 配置为深圳站点的桥头服务器。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-24.png %}

## 检查复制拓扑

1. 选择 Sites > SH > Servers > SRV-SHDC01 > NTDS Settings，点击 All Tasks > Check Replication Topology。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-25.png %}
2. 点击 OK。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-26.png width:75% %}
3. 选择 Sites > SH > Servers > SRV-SHDC01 > NTDS Settings > \<automatically generated\>，点击 Replicate Now。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-27.png %}
4. 对所有的服务器和连接重复执行以上操作，直至自动生成的连接与架构设计完全匹配。
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-28.png %}
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-29.png %}
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-30.png %}
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-31.png %}
{% image /assets/wiki/ad-ds/configure-site-replication/configure-site-replication-32.png %}
