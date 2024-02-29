---
layout: wiki
wiki: powershell
title: 代码可读性
date: 2021-11-07
---

## 引号和转义字符

PowerShell 中的单引号和双引号分别具有不同的作用。在引入纯文本字符串时，应使用单引号：
```powershell
# 避免这样做
"Good morning, Waddledee!"

# 推荐的做法
'Good morning, Waddledee!'
```

只有在引入带变量的文本字符串时，才使用双引号：
```powershell
$name = 'Waddledee'
"Good morning, $name!"
```

双引号的另外一个作用是转义，例如 `{% raw %}`t{% endraw %}` 转义为 Tab，`{% raw %}`n{% endraw %}` 转义为换行等。不过我个人的建议是尽量不要使用转义字符，而是通过 [Here-strings](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_quoting_rules#here-strings) 提升代码可读性：
```powershell
# 避免这样做
"Good morning, Waddledee!`nHave a nice day."

# 推荐的做法
@'
Good morning, Waddledee!
Have a nice day.
'@
```

## 格式运算符

在双引号中引入变量的属性或方法时，通常需要额外嵌套一层 $()，例如 `"$($file.BaseName)"`。这种情况可以使用 [Format operator](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_operators/#format-operator--f) 提升代码可读性：
```powershell
# 避免这样做
$file = Get-Item -Path 'D:\Backups\Database.bak'
"s3://backups/$($file.BaseName)_$($file.CreationTime.ToString('yyyyMMdd_HHmmss'))$($file.Extension)"

# 推荐的做法
$file = Get-Item -Path 'D:\Backups\Database.bak'
's3://backups/{0}_{1}{2}' -f $file.BaseName, $file.CreationTime.ToString('yyyyMMdd_HHmmss'), $file.Extension
```

## 别名和位置参数

在终端敲命令的时候，别名和位置参数使用起来非常顺手。但在编写脚本的过程中，应当将其补全：
```powershell
# 避免这样做
cd C:\Users\Waddledee\Desktop

# 推荐的做法
Set-Location -Path 'C:\Users\Waddledee\Desktop'
```

## 命令换行

当一条命令指定了多个参数时，会因为过长而变得难以阅读：
```powershell
# 避免这样做
New-ADUser -Name 'Waddledee' -UserPrincipalName 'waddledee@contoso.com' -Company 'CONTOSO' -Department 'IT Operations' -Title 'Windows Administrator' -Path 'OU=IT Operations,OU=CONTOSO,DC=contoso,DC=com'
```

一种方法是使用反引号实现命令换行。不过我非常不推荐这种做法，因为反引号容易和单引号混淆：
```powershell
# 避免这样做
New-ADUser -Name 'Waddledee' `
           -UserPrincipalName 'waddledee@contoso.com' `
           -Company 'CONTOSO' `
           -Department 'IT Operations' `
           -Title 'Windows Administrator' `
           -Path 'OU=IT Operations,OU=CONTOSO,DC=contoso,DC=com'
```

推荐的做法是通过 [Splatting](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_splatting) 将参数事先储存在哈希表中：
```powershell
# 推荐的做法
$user = @{
    Name              = 'Waddledee'
    UserPrincipalName = 'waddledee@contoso.com'
    Company           = 'CONTOSO'
    Department        = 'IT Operations'
    Title             = 'Windows Administrator'
    Path              = 'OU=IT Operations,OU=CONTOSO,DC=contoso,DC=com'
}
New-ADUser @user
```

## 管道符换行

当多条命令经过管道符串联时，同样会因为过长而变得难以阅读：
```powershell
# 避免这样做
Get-ChildItem -Path 'D:\Backups' -File -Recurse | Group-Object -Property 'Directory' | Where-Object {$_.Count -gt 1} | ForEach-Object {$_.Group | Sort-Object -Property 'LastWriteTime' -Descending | Select-Object -Skip 1 | Remove-Item}
```

不过好在 PowerShell 支持管道符换行，代码解析器能够识别行末的管道符以持续构建管道：
```powershell
# 推荐的做法
Get-ChildItem -Path 'D:\Backups' -File -Recurse | 
    Group-Object -Property 'Directory' | 
    Where-Object {$_.Count -gt 1} | 
    ForEach-Object {
        $_.Group | 
            Sort-Object -Property 'LastWriteTime' -Descending | 
            Select-Object -Skip 1 | 
            Remove-Item
    }
```

## 逻辑运算符换行

还有一种容易发生代码过长问题的情况，是由逻辑运算符串联起来的多个比较运算符语句：
```powershell
# 避免这样做
if ($conditionA -eq 'A' -and $conditionB -ne 'B' -and $conditionC -match 'C' -or $conditionD -contains 'D') {
    # ...
}
```

对此，可以将判断条件事先储存在变量中并利用运算符换行不影响代码执行的特性分隔比较运算符：
```powershell
# 推荐的做法
$condition = $conditionA -eq 'A' -and 
             $conditionB -ne 'B' -and 
             $conditionC -match 'C' -or 
             $conditionD -contains 'D'

if ($condition) {
    # ...
}
```

## 外部程序调用

在调用外部程序时，应当补全文件后缀名以免发生意外。例如，`sc` 是 Set-Content 命令的别名，同时它也是一个命令行应用程序，因此在调用时须明确指定完整的文件名称 `sc.exe`：
```
PS C:\> Get-Alias -Name 'sc'

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           sc -> Set-Content

PS C:\> sc.exe
DESCRIPTION:
        SC is a command line program used for communicating with the
        Service Control Manager and services.
USAGE:
        sc <server> [command] [service name] <option1> <option2>...
        ...
```

## Write-Host

我注意到，有很多 PowerShell 用户习惯使用 Write-Host 命令监控脚本的运行状态和结果，例如：
```powershell
# 避免这样做
Copy-Item -Path 'D:\Backups\Database.bak' -Destination 'Z:\Archive\Database.bak' -ErrorAction 'Stop'
Write-Host -Object 'Successfully archived database backup!' -ForegroundColor 'Green'
```

这在调试的时候或许有用，但脚本通常是在后台运行的，Write-Host 命令无法经过管道传递对象的特性使得它的输出难以被系统捕获，毕竟它本身的功能就是打印输出到控制台：
```
PS C:\> Write-Host -Object 'Hello World' | Get-Member
Hello World
Get-Member : You must specify an object for the Get-Member cmdlet.
At line:1 char:36
+ Write-Host -Object 'Hello World' | Get-Member
+                                    ~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Get-Member], InvalidOperationException
    + FullyQualifiedErrorId : NoObjectInGetMember,Microsoft.PowerShell.Commands.GetMemberCommand
```

因此，在编写脚本时应尽量避免使用 Write-Host 命令，转而通过 [Error Handling](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_try_catch_finally) 将日志输出到文件：
```powershell
# 推荐的做法
$timestamp = Get-Date -Format 'yyyy-MM-dd HH:mm:ss '

$log = try {
    Copy-Item -Path 'D:\Backups\Database.bak' -Destination 'Z:\Archive\Database.bak' -ErrorAction 'Stop'
    $timestamp + 'Successfully archived database backup!'
}
catch {
    $timestamp + $_.Exception.Message
}

$log | Out-File -FilePath 'D:\Scripts\Logs\ArchiveDatabaseBackup.log' -Append
```
