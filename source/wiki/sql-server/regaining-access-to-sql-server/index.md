---
layout: wiki
wiki: sql-server
title: 重新获取 SQL Server 的访问权限
date: 2022-07-03
---

{% box color:blue %}
使用此方法需要当前用户是 SQL Server 服务器的本地管理员。
{% endbox %}

1. 停止 SQL Server 实例：
```powershell
Stop-Service -Name 'MSSQLSERVER' -Force
```
2. 以单用户模式启动 SQL Server 实例：
```powershell
$service = Get-Service -Name 'MSSQLSERVER'
$service.Start(@('/f', '/mSQLCMD'))
```
3. 创建当前用户的 LOGIN 并赋予 sysadmin 权限：
```powershell
$query = 'CREATE LOGIN [{0}\{1}] FROM WINDOWS; ALTER SERVER ROLE sysadmin ADD MEMBER [{0}\{1}];'
sqlcmd.exe -E -Q ($query -f $env:USERDOMAIN, $env:USERNAME)
```
4. 重启 SQL Server 实例恢复多用户模式：
```powershell
Restart-Service -Name 'MSSQLSERVER'
```
