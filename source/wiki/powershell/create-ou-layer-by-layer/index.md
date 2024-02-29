---
layout: wiki
wiki: powershell
title: 逐层创建 OU
date: 2021-10-17
---

## 前言

当我们使用 ActiveDirectory 模块的 New-ADOrganizationalUnit 命令创建多层组织单位 (Organizational Unit, OU) 时，往往需要依次执行多条命令。以下是一个三层 OU 的创建示例：

```powershell
New-ADOrganizationalUnit -Name 'L1' -Path 'DC=contoso,DC=com'
New-ADOrganizationalUnit -Name 'L2' -Path 'OU=L1,DC=contoso,DC=com'
New-ADOrganizationalUnit -Name 'L3' -Path 'OU=L2,OU=L1,DC=contoso,DC=com'
```

为了能够基于可分辨名称 (Distinguished Name, DN) 一行搞定多层 OU 的创建，我制作了这个脚本。

## 脚本

```powershell
function New-ADOrganizationalUnitByDN {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory = $true)]
        [ValidateNotNullOrEmpty()]
        [string]$DistinguishedName,

        [Parameter()]
        [ValidateScript({
            if ($_.ContainsKey('Name') -or $_.ContainsKey('Path')) {
                throw "Please enter arguments other than 'Name' and 'Path'."
            }
            else {
                $true
            }
        })]
        [hashtable]$ArgumentList
    )

    $dnArray = @($DistinguishedName.Split(','))
    $ouArray = @($dnArray | Where-Object {$_ -match 'OU='})
    $dcArray = @($dnArray | Where-Object {$_ -match 'DC='})

    for ($index = $ouArray.Count - 1; $index -ge 0; $index--) {
        $identity = ($ouArray[$index..($ouArray.Count - 1)] + $dcArray) -join ','

        if (-not (Get-ADOrganizationalUnit -Filter "DistinguishedName -eq '$identity'")) {
            $path = @(if ($index -ne $ouArray.Count - 1) {
                $ouArray[($index + 1)..($ouArray.Count - 1)]
            })

            $newOU = @{
                Name = $ouArray[$index].TrimStart('OU=')
                Path = ($path + $dcArray) -join ','
            }

            if ($PSBoundParameters.ContainsKey('ArgumentList')) {
                $newOU += $ArgumentList
            }

            New-ADOrganizationalUnit @newOU
        }
    }
}
```

{% box color:blue %}
- 此脚本依赖 [ActiveDirectory](https://learn.microsoft.com/en-us/powershell/module/activedirectory) 模块，适用于 PowerShell 5.1 及更高版本。
- 此脚本基于 Functions 编写，保存到本地后可通过 [Dot Sourcing](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_scripts#script-scope-and-dot-sourcing) 进行调用。
{% endbox %}

## 语法

```powershell
New-ADOrganizationalUnitByDN [-DistinguishedName] <string> [[-ArgumentList] <hashtable>] [<CommonParameters>]
```

## 描述

New-ADOrganizationalUnitByDN 命令通过解析 DN 逐层创建 OU。执行过程中，如果当前层级的 OU 已存在，则解析下一层级继续尝试创建，直至最后一层。可选参数 ArgumentList 支持以 [hashtable] 的形式附加 New-ADOrganizationalUnit 命令的参数（Name 和 Path 除外）。

## 示例

- 创建一个三层 OU：
```
PS C:\> New-ADOrganizationalUnitByDN -DistinguishedName 'OU=L3,OU=L2,OU=L1,DC=contoso,DC=com'
```

- 创建一个三层 OU，关闭「防止容器被意外删除」属性，同时开启详细日志：
```
PS C:\> New-ADOrganizationalUnitByDN -DistinguishedName 'OU=L3,OU=L2,OU=L1,DC=contoso,DC=com' -ArgumentList @{ProtectedFromAccidentalDeletion = $false; Verbose = $true}
VERBOSE: Performing the operation "New" on target "OU=L1,DC=contoso,DC=com".
VERBOSE: Performing the operation "New" on target "OU=L2,OU=L1,DC=contoso,DC=com".
VERBOSE: Performing the operation "New" on target "OU=L3,OU=L2,OU=L1,DC=contoso,DC=com".
```
