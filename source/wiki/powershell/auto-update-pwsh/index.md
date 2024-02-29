---
layout: wiki
wiki: powershell
title: 自动更新 PWSH
date: 2021-10-03
---

{% box color:warning %}
PWSH 现已支持通过 Microsoft Update 自动更新，本文不再具备实用价值，仅作归档使用，详情可参考 [Microsoft Update for PowerShell FAQ](https://learn.microsoft.com/en-us/powershell/scripting/install/microsoft-update-faq)。
{% endbox %}

---

复制以下代码并保存到本地，此脚本也可用作 PWSH 的全新安装：

```powershell C:\Users\Waddledee\Documents\WindowsPowerShell\Scripts\UpdatePWSH.ps1
#Requires -PSEdition Desktop -RunAsAdministrator

$ErrorActionPreference = 'Stop'

$latest = Invoke-RestMethod -Uri 'https://api.github.com/repos/PowerShell/PowerShell/releases/latest'
$local = Get-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*' | Where-Object {$_.DisplayName -match 'powershell' -and $_.DisplayName -notmatch 'preview'}

if (-not $local -or ($local -and [version]($local.DisplayVersion.Split('.')[0..2] -join '.') -lt [version]$latest.tag_name.Trim('v'))) {
    $installer = $latest.assets | Where-Object {$_.name -match 'win-x64.msi'}
    $installerPath = "$env:LOCALAPPDATA\Temp\$($installer.name)"

    Invoke-RestMethod -Uri $installer.browser_download_url -OutFile $installerPath
    Start-Process -FilePath 'msiexec.exe' -ArgumentList "/i $installerPath /quiet" -Wait
    Remove-Item -Path $installerPath
}
```

{% box color:blue %}
- 此脚本必须在 [Windows PowerShell](https://github.com/PowerShell/PowerShell#windows-powershell-vs-powershell-core) 中以管理员身份运行。
- 此脚本仅安装 [Windows (x64) Stable 版本](https://github.com/PowerShell/PowerShell#get-powershell) 的 PWSH。
{% endbox %}

创建一个 Windows 计划任务定时触发脚本，以下示例配置为每天 00:00 触发：

```powershell
$task = @{
    TaskName = 'Update PWSH'
    User     = '<user>'
    Password = '<password>'
    RunLevel = 'Highest'
    Trigger  = New-ScheduledTaskTrigger -At '00:00' -Daily
    Action   = New-ScheduledTaskAction -Execute 'powershell.exe' -Argument '-NoProfile -File C:\Users\Waddledee\Documents\WindowsPowerShell\Scripts\UpdatePWSH.ps1'
}
Register-ScheduledTask @task
```
