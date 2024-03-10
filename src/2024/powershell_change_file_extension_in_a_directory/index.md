---
title: 'PowerShell, ディレクトリ内のファイルの拡張子をまとめて変更する'
date: '2024-03-10T19:20:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Windows
tags:
  - Windows
  - PowerShell
---
# PowerShell, ディレクトリ内のファイルの拡張子をまとめて変更する

- Windows 11

ディレクトリ内の拡張子`.jfif`のファイルを拡張子`.jpg`に変更する。
以下のコマンドでは、直下のファイルだけが対象となり、サブディレクトリ以下は対象とならない。

```powershell
Dir *.jfif | Rename-Item -NewName { [io.path]::ChangeExtension($_.name, "jpg") }
```

## 参考

- [bash - How do I change the extension of many files in a directory? - Stack Overflow](https://stackoverflow.com/questions/12120326/how-do-i-change-the-extension-of-many-files-in-a-directory)
- [powershell - How to add a suffix to all the files - Stack Overflow](https://stackoverflow.com/questions/57041671/how-to-add-a-suffix-to-all-the-files)
