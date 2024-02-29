---
layout: wiki
wiki: powershell
title: 生成随机密码
date: 2021-10-24
---

## 脚本

```powershell
function New-Password {
    param (
        [Parameter()][ValidateRange(8, 128)][int]$Length = 16,
        [Parameter()][switch]$NoUppercase,
        [Parameter()][switch]$NoLowercase,
        [Parameter()][switch]$NoNumbers,
        [Parameter()][switch]$NoSymbols
    )

    $chars = @(
        [PSCustomObject]@{Name = 'NoUppercase'; Number = 65..90}
        [PSCustomObject]@{Name = 'NoLowercase'; Number = 97..122}
        [PSCustomObject]@{Name = 'NoNumbers'; Number = 48..57}
        [PSCustomObject]@{Name = 'NoSymbols'; Number = 33..47 + 58..64 + 94..96 + 123..126}
    )

    $switches = $MyInvocation.MyCommand.Parameters.Keys | Where-Object {$_ -match 'No'}
    $selected = $PSBoundParameters.Keys | Where-Object {$_ -match 'No'}
    $luckySwitch = $switches | Where-Object {$_ -notin $selected} | Get-Random

    if ($selected.Count -eq 4) {throw 'Please select at least one character type'}

    $divisor = 4 - $selected.Count
    $quotient = [Math]::Floor($Length / $divisor)
    $remainder = $Length % $divisor

    $randomChars = foreach ($char in $chars) {
        if ($char.Name -notin $selected) {
            $count = if ($char.Name -eq $luckySwitch) {$quotient + $remainder} else {$quotient}
            1..$count | ForEach-Object {Get-Random -InputObject $char.Number}
        }
    }

    -join ($randomChars | ForEach-Object {[char]$_} | Sort-Object {Get-Random})
}
```

{% box color:blue %}
- 此脚本适用于 PowerShell 5.1 及更高版本。
- 此脚本基于 Functions 编写，保存到本地后可通过 [Dot Sourcing](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_scripts#script-scope-and-dot-sourcing) 进行调用。
{% endbox %}

## 语法

```powershell
New-Password [[-Length] <int>] [-NoUppercase] [-NoLowercase] [-NoNumbers] [-NoSymbols] [<CommonParameters>]
```

## 描述

New-Password 命令可生成长度 8 到 128，四种字符类型（大写、小写、数字、符号）随意组合的随机密码。默认情况下，New-Password 命令生成长度 16，包含所有字符类型的随机密码。

## 示例

- 生成默认长度 (16) 的随机密码：
```
PS C:\> New-Password
z^ZY_1'6XmSu94&u
```

- 生成特定长度 (32) 的随机密码：
```
PS C:\> New-Password -Length 32
Kgk&I3KQp77p4>d7~(?oKRs0J/U2a+{5
```

- 生成不包含符号的随机密码：
```
PS C:\> New-Password -NoSymbols
4IoXX26yoQdU5U2b
```

- 生成特定长度 (32) 且不包含小写字母和符号的随机密码：
```
PS C:\> New-Password -Length 32 -NoLowercase -NoSymbols
446D7M9E207SE9Y9J15W0ER4P4PM0TDE
```

- 生成随机密码并将其转换为加密字符串：
```
PS C:\> New-Password | ConvertTo-SecureString -AsPlainText -Force
System.Security.SecureString
```
