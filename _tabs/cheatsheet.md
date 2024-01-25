---
# the default layout is 'page'
icon: fas fa-info-circle
order: 2
---


This is my own cheatsheet splitted in different topics with chunks of useful code.

> **Note:** This is an initial version.
{: .prompt-info }

## Table of contents
1. PowerShell
2. Malware Debugging

-----

## 1. PowerShell

* Get file's hashes via PowerShell:  
`Get-FileHash -Path file.exe -Algorithm MD5 | Format-List`
  
  
* Watch a folder for file creation and copy files to Desktop:

```powershell
$watcher = New-Object System.IO.FileSystemWatcher
$watcher.IncludeSubdirectories = $true
$watcher.Path = 'C:\'
$watcher.EnableRaisingEvents = $true

$action = {
    $path = $event.SourceEventArgs.FullPath
    copy $path %userprofile%\Desktop
}

Register-ObjectEvent $watcher 'Created' -Action $action
```

## 2. Malware Debugging

General manual unpacking breakpoints:
* CreateProcessInternalW
* VirtualProtect
* ResumeThread
* VirtualAlloc (on its return)