---
layout: wiki
wiki: powershell
title: 定制 Windows Terminal
date: 2022-06-26
references:
  - '[Introduction | Oh My Posh](https://ohmyposh.dev/docs/)'
---

## 更换字体

为了能够正确显示图标和特殊字符，推荐从 Nerd Fonts 挑选并安装一款字体：
{% link https://www.nerdfonts.com/ %}

随后进入 Windows Terminal 的设置页面更换默认字体，这里以 `JetBrainsMono Nerd Font` 为例：
{% image /assets/wiki/powershell/customizing-windows-terminal/customizing-windows-terminal-01.jpg %}

## 更换配色

除了自带的几种配色方案之外，Windows Terminal Themes 还有上百种配色方案可供选择：
{% link https://windowsterminalthemes.dev/ %}

挑选一种配色方案并点击 Get theme，下面以 `GitHub Dark` 为例：
{% image /assets/wiki/powershell/customizing-windows-terminal/customizing-windows-terminal-02.jpg %}

打开 Windows Terminal 配置文件，在 defaults 中指定默认配色并将配色方案粘贴到 schemes 数组：
```diff
{
  ...
  "profiles": {
    "defaults": {
+     "colorScheme": "GitHub Dark",
      ...
    },
    ...
  },
  "schemes": [
+   {
+     "background": "#101216",
+     "black": "#000000",
+     "blue": "#6CA4F8",
+     "brightBlack": "#4D4D4D",
+     "brightBlue": "#6CA4F8",
+     "brightCyan": "#2B7489",
+     "brightGreen": "#56D364",
+     "brightPurple": "#DB61A2",
+     "brightRed": "#F78166",
+     "brightWhite": "#FFFFFF",
+     "brightYellow": "#E3B341",
+     "cursorColor": "#C9D1D9",
+     "cyan": "#2B7489",
+     "foreground": "#8B949E",
+     "green": "#56D364",
+     "name": "GitHub Dark",
+     "purple": "#DB61A2",
+     "red": "#F78166",
+     "selectionBackground": "#3B5070",
+     "white": "#FFFFFF",
+     "yellow": "#E3B341"
+   },
    ...
  ]
}
```

## 定制提示符 (PowerShell)

{% folding 什么是提示符？ %}
提示符是一串用来显示命令行当前所处位置及状态的字符，PowerShell 的默认提示符是 `PS C:\>`。
{% endfolding %}

安装 Oh My Posh，这是一款支持通过 JSON 配置文件定制提示符的工具：
```powershell
Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://ohmyposh.dev/install.ps1'))
```

在 Oh My Posh Themes 挑选一个主题，下面以 `jandedobbeleer` 为例：
{% link https://ohmyposh.dev/docs/themes %}

使用记事本打开或创建 PowerShell 配置文件：
```powershell
if ($PROFILE) {
    New-Item -Path $PROFILE -Type File -Force
}

notepad.exe $PROFILE
```

复制如下内容，用于在 PowerShell 启动时初始化 Oh My Posh 并应用主题文件：
```powershell
oh-my-posh.exe init pwsh --config "$env:POSH_THEMES_PATH/jandedobbeleer.omp.json" | Invoke-Expression
```

保存 PowerShell 配置文件，然后打开一个新的 Windows Terminal 标签页查看主题是否生效：
{% image /assets/wiki/powershell/customizing-windows-terminal/customizing-windows-terminal-03.jpg %}

## Oh My Posh 主题分享

最后，附上一款我自己制作的 Oh My Posh 主题：
{% image /assets/wiki/powershell/customizing-windows-terminal/customizing-windows-terminal-04.jpg %}

配置文件如下所示：
```json
{
  "$schema": "https://raw.githubusercontent.com/JanDeDobbeleer/oh-my-posh/main/themes/schema.json",
  "blocks": [
    {
      "alignment": "left",
      "segments": [
        {
          "foreground": "#cccccc",
          "style": "plain",
          "template": "[{{ .CurrentDate | date .Format }}] ",
          "type": "time"
        },
        {
          "foreground": "#98c379",
          "properties": {
            "always_enabled": true
          },
          "style": "plain",
          "template": "{{ .FormattedMs }} ",
          "type": "executiontime"
        },
        {
          "foreground": "#55b9c4",
          "style": "plain",
          "template": "{{ .Location }} ",
          "type": "path"
        },
        {
          "foreground": "#c678dd",
          "foreground_templates": [
            "{{ if or (.Working.Changed) (.Staging.Changed) }}#ff9248{{ end }}"
          ],
          "properties": {
            "branch_ahead_icon": "\uf55c",
            "branch_behind_icon": "\uf544",
            "branch_icon": "\ufb2b ",
            "fetch_stash_count": true,
            "fetch_status": true
          },
          "style": "plain",
          "template": "<#cccccc>on</> {{ .HEAD }}{{ .BranchStatus }}{{ if .Working.Changed }} \uf044 {{ .Working.String }}{{ end }}{{ if and (.Working.Changed) (.Staging.Changed) }} |{{ end }}{{ if .Staging.Changed }} \uf046 {{ .Staging.String }}{{ end }} ",
          "type": "git"
        },
        {
          "foreground": "#c94a16",
          "style": "plain",
          "template": "x",
          "type": "exit"
        }
      ],
      "type": "prompt"
    },
    {
      "alignment": "left",
      "newline": true,
      "segments": [
        {
          "foreground": "#e5c07b",
          "style": "plain",
          "template": "\uf63d ",
          "type": "text"
        }
      ],
      "type": "prompt"
    }
  ],
  "version": 2
}
```
