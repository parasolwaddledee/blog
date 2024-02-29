---
layout: wiki
wiki: powershell
title: 按周获取日期
date: 2021-10-10
---

## 脚本

```powershell
function Get-DateByWeek {
    param (
        [Parameter()]
        [ValidateRange(1, 9999)]
        [int]$Year = (Get-Date).Year,

        [Parameter()]
        [ValidateRange(1, 12)]
        [int]$Month = (Get-Date).Month,

        [Parameter(Mandatory = $true)]
        [ValidateRange(1, 5)]
        [int]$Week,

        [Parameter(Mandatory = $true)]
        [ValidateSet('Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday')]
        [string]$DayOfWeek
    )

    $weekdays = 1..[datetime]::DaysInMonth($Year, $Month) | 
        ForEach-Object {Get-Date -Year $Year -Month $Month -Day $_ -Hour 0 -Minute 0 -Second 0 -Millisecond 0} | 
        Where-Object {$_.DayOfWeek -eq $DayOfWeek}

    $weekdays[$Week - 1]
}
```

{% box color:blue %}
- 此脚本适用于 PowerShell 5.1 及更高版本。
- 此脚本基于 Functions 编写，保存到本地后可通过 [Dot Sourcing](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_scripts#script-scope-and-dot-sourcing) 进行调用。
{% endbox %}

## 语法

```powershell
Get-DateByWeek [[-Year] <int>] [[-Month] <int>] [-Week] <int> [-DayOfWeek] {Sunday | Monday | Tuesday | Wednesday | Thursday | Friday | Saturday} [<CommonParameters>]
```

## 描述

Get-DateByWeek 命令可以输出任意年月第几周的星期几所对应的日期。如不指定年月参数，则默认为当前年月。

## 示例

- 获取当年 (2021) 当月 (10) 第二周星期天的日期：
```
PS C:\> Get-DateByWeek -Week 2 -DayOfWeek 'Sunday'

Sunday, October 10, 2021 00:00:00
```

- 获取 2021 年 1 月第一周星期一的日期：
```
PS C:\> Get-DateByWeek -Year 2021 -Month 1 -Week 1 -DayOfWeek 'Monday'

Monday, January 4, 2021 00:00:00
```
