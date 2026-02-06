---
title: "cannot be loaded because running scripts is disabled on this system"
date: 2020-09-01 21:11:00 +0900
comments: true
categories: angular
tags: [error]
---

## Environment Setup

- Operating System: Windows
- IDE: VS Code
- Shell: PowerShell
- Framework: Angular 10.0.8

## Problem Description

An error occurs preventing script execution, despite having configured environment variables correctly. The error message is similar to:

```
ng : Cannot load the file C:\Users\< username >\AppData\Roaming\npm\ng.ps1 because running scripts is disabled on this system. 
For more information, see about_Execution_Policies at https://go.microsoft.com/fwlink/?LinkID=135170.
At line:1 char:1
...
...
...
```

This indicates an issue with PowerShell's execution policy preventing the `ng` command from running.

## Unsuccessful Attempts

1. Removing Read-Only Attribute from `C:\Users\< username >\AppData\Roaming\npm\ng.ps1`

Attempting to remove the read-only attribute from the `ng.ps1` file, or even the parent directory, does not resolve the issue. The file is often already not read-only, and changing this setting has no effect.

1. Deleting and Reinstalling `ng.ps1`

Deleting the `ng.ps1` file and reinstalling the Angular CLI via `npm install -g @angular/cli` results in the file being recreated, but the same error persists upon execution. This confirms the issue is not with a corrupted file, but rather with the execution policy.

## Solution

The problem lies in the PowerShell execution policy, which restricts the execution of scripts. The following command resolves this:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

This command sets the execution policy to `RemoteSigned` for the current user.  `RemoteSigned` allows execution of scripts that are downloaded from the internet and signed by a trusted publisher or scripts that are written locally.  This policy scope limits the change to the current user, minimizing potential security risks.

After running this command, PowerShell will be able to execute the `ng` command and other Angular CLI commands. You might need to close and reopen your PowerShell session for the changes to take effect.

## References

- [why-powershell-does-not-run-angular-commands](https://stackoverflow.com/questions/58032631/why-powershell-does-not-run-angular-commands)

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/ng-%EC%9D%B4-%EC%8B%9C%EC%8A%A4%ED%85%9C%EC%97%90%EC%84%9C-%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EB%A5%BC-%EC%8B%A4%ED%96%89%ED%95%A0-%EC%88%98-%EC%97%86%EC%9C%BC%EB%AF%80%EB%A1%9C-)
<br/>