---
# the default layout is 'page'
icon: fas fa-file-text
order: 2
---


This is my own cheatsheet splitted in different topics with chunks of useful code.

> **Note:** This is an initial version.
{: .prompt-info }

## Table of contents
1. PowerShell
2. Malware Debugging
3. Redirect Malware Connection

-----

## 1. PowerShell

* Getting file's hashes via PowerShell:  
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

## 3. Redirect Malware Connection

This is useful for when I need to control a malware using its C2, making it to easily connect to wherever I need, without changing the malware configuration (i.e: patching the binary).
1. Open cmd.exe as Administrator
2. Execute the command below (change listen/connect address/port values):
```bash
netsh interface portproxy add v4tov4 listenaddress=127.0.0.1 listenport=6651 connectaddress=192.168.232.140 connectport=6651
```