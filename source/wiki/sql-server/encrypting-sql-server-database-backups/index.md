---
layout: wiki
wiki: sql-server
title: 加密 SQL Server 数据库备份
date: 2022-07-10
---

## 创建主密钥和证书

1. 切换到 master 数据库：
```sql
USE master
```
2. 使用密码创建主密钥：
```sql
CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<master_key_password>'
```
3. 通过主密钥创建证书：
```sql
CREATE CERTIFICATE DBC2022
WITH SUBJECT = 'Database Backup Certificate 2022'
```
{% box color:blue %}
- 默认情况下的证书有效期是一年，可以通过 EXPIRY_DATE 参数自定义证书有效期。
- 过期的证书无法进行加密备份，但仍然可以还原之前使用该证书加密的备份文件。
{% endbox %}
4. 导出证书和私钥到文件：
```sql
BACKUP CERTIFICATE DBC2022
TO FILE = 'D:\MSSQLSERVER\DBC2022.cer'
WITH PRIVATE KEY (FILE = 'D:\MSSQLSERVER\DBC2022.key', ENCRYPTION BY PASSWORD = '<private_key_password>')
```

## 备份数据库并加密

1. 使用证书加密备份数据库：
```sql
BACKUP DATABASE AdventureWorks
TO DISK = 'D:\MSSQLSERVER\AdventureWorks.bak'
WITH ENCRYPTION (ALGORITHM = AES_256, SERVER CERTIFICATE = DBC2022)
```
2. 检查数据库备份文件信息：
```sql
SELECT b.backup_start_date, b.database_name, b.type, b.key_algorithm, b.encryptor_type, b.encryptor_thumbprint, bf.physical_device_name
FROM msdb.dbo.backupset b
INNER JOIN msdb.dbo.backupmediafamily bf on bf.media_set_id = b.media_set_id
WHERE b.database_name = 'AdventureWorks'
ORDER BY b.backup_start_date DESC
```

## 还原主密钥、证书和数据库

1. 切换到 master 数据库：
```sql
USE master
```
2. 使用密码还原主密钥：
```sql
CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<master_key_password>'
```
3. 通过导出的证书和私钥文件还原证书：
```sql
CREATE CERTIFICATE DBC2022
FROM FILE = 'D:\MSSQLSERVER\DBC2022.cer'
WITH PRIVATE KEY (FILE = 'D:\MSSQLSERVER\DBC2022.key', DECRYPTION BY PASSWORD = '<private_key_password>')
```
4. 还原数据库：
```sql
RESTORE DATABASE AdventureWorks
FROM DISK = 'D:\MSSQLSERVER\AdventureWorks.bak'
```
