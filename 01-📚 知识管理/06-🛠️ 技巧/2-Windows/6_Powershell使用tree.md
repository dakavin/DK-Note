---
文章标题: "[[6_Powershell使用tree]]" 
文章作者: Dakkk
文章概要: |
  文章介绍了在Windows PowerShell中显示目录树结构的多种方法。它涵盖了使用外部`tree`命令进行基本过滤，以及利用PowerShell原生`Get-ChildItem`命令配合条件过滤、路径处理和输出格式化，实现更灵活和高级的目录视图定制。
tags:
- "PowerShell"
- "目录结构"
- "文件系统"
- "Get-ChildItem"
- "tree命令"
- "命令行"
- "脚本"
相关文章:
- "[[9_Linux文件目录]]"
- "[[0_快捷键大全]]"
- "[[05_U-boot的介绍]]"
- "[[07_Linux的介绍]]"
- "[[1_常用文件夹]]"
文章分类: "🛠️ 开发工具和OS相关"
文章路径: "08-🛠️ 开发工具和OS相关/05-Windows/6_Powershell使用tree.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2025-05-25 17:19:45
修改时间: 2025-05-28 01:03:09
---

在Windows 11的PowerShell中，你可以使用以下命令来显示目录树结构并排除特定目录：

## 1 方法1: 使用 `tree` 命令（推荐）

```powershell
tree /F /A | Select-String -NotMatch "\.obsidian|\.git|node_modules"
```
参数说明：
- `/F` - 显示文件
- `/A` - 使用ASCII字符
- `Select-String -NotMatch` - 排除包含指定模式的行

## 2 方法2: 使用 PowerShell 原生命令

**包含根路径**
```shell
Get-ChildItem -Recurse | Where-Object { $_.FullName -match "(0[1-9]|1[0-4])-|README" -and $_.FullName -notmatch "\.obsidian|\.git|\.smtcmp_json_db|\.trash|\.smtcmp_vector_db\.tar\.gz|LICENSE|\.gitignore" } | Select-Object FullName
```

**不包含根路径**
```shell
Get-ChildItem -Recurse | Where-Object { $_.FullName -match "(0[1-9]|1[0-4])-|README" -and $_.FullName -notmatch "\.obsidian|\.git|\.smtcmp_json_db|\.trash|\.smtcmp_vector_db\.tar\.gz|LICENSE|\.gitignore" } | Select-Object @{Name="RelativePath"; Expression={$_.FullName -replace [regex]::Escape($PWD.Path + "\"), ""}}
```

```shell
Get-ChildItem -Recurse | Where-Object { $_.FullName -match "(0[1-9]|1[0-4])-|README" -and $_.FullName -notmatch "\.obsidian|\.git|\.smtcmp_json_db|\.trash|\.smtcmp_vector_db\.tar\.gz|LICENSE|\.gitignore" } | ForEach-Object { $_.FullName.Substring($PWD.Path.Length + 1) }
```

**不包含根路径，以及最小目录下的文件**
```shell
Get-ChildItem -Recurse -Directory | Where-Object { $_.FullName -match "(0[1-9]|1[0-4])-" -and $_.FullName -notmatch "\.obsidian|\.git|\.smtcmp_json_db|\.trash" } | ForEach-Object { $_.FullName.Substring($PWD.Path.Length + 1) }
```
- 添加了 `-Directory` 参数，只检索目录
- 移除了 `README` 匹配条件，因为README通常是文件
- 简化了排除条件，移除了不必要的文件类型匹配

**包含README文件但排除其他文件**
```shell
Get-ChildItem -Recurse | Where-Object { ($_.PSIsContainer -and $_.FullName -match "(0[1-9]|1[0-4])-") -or ($_.Name -match "README") } | Where-Object { $_.FullName -notmatch "\.obsidian|\.git|\.smtcmp_json_db|\.trash" } | ForEach-Object { $_.FullName.Substring($PWD.Path.Length + 1) }
```
