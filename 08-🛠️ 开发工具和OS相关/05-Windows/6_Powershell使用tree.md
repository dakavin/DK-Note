
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
