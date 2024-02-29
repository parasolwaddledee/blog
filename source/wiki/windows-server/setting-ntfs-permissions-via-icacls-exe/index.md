---
layout: wiki
wiki: windows-server
title: 通过 icacls.exe 设置 NTFS 权限
date: 2021-09-19
---

- 授予用户对目标文件夹的只读权限：
```powershell
icacls.exe 'D:\Folder' /grant 'UserA:(OI)(CI)R'
```
- 授予用户对目标文件夹的执行权限：
```powershell
icacls.exe 'D:\Folder' /grant 'UserA:(OI)(CI)RX'
```
- 授予用户对目标文件夹的修改权限：
```powershell
icacls.exe 'D:\Folder' /grant 'UserA:(OI)(CI)M'
```
- 授予用户对目标文件夹的完全控制权限：
```powershell
icacls.exe 'D:\Folder' /grant 'UserA:(OI)(CI)F'
```
- 移除用户对目标文件夹的全部权限：
```powershell
icacls.exe 'D:\Folder' /remove 'UserA'
```
- 同时对目标文件夹设置多条权限：
```powershell
icacls.exe 'D:\Folder' /grant 'UserA:(OI)(CI)R' /grant 'UserB:(OI)(CI)M' /remove 'UserC'
```
- 设置目标文件夹的所有者：
```powershell
icacls.exe 'D:\Folder' /setowner 'UserA'
```
- 将目标文件夹设置为禁用继承并复制已继承权限：
```powershell
icacls.exe 'D:\Folder' /inheritance:d
```
- 将目标文件夹设置为禁用继承并删除已继承权限：
```powershell
icacls.exe 'D:\Folder' /inheritance:r
```
- 将目标文件夹设置为启用继承：
```powershell
icacls.exe 'D:\Folder' /inheritance:e
```
