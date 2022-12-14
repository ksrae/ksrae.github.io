---
title: "ng: 이 시스템에서 스크립트를 실행할 수 없으므로 ... (cannot be loaded because 
running scripts is disabled on this system)"
date: 2020-09-01 21:11:00 +0900
comments: true
categories: angular
tags: [error]
---


## 환경
- windows
- vscode
- powershell
- Angular 10.0.8


## 원인
Environment Variables를 설정하였음에도 스크립트를 실행할 수 없다는 에러가 발생합니다.

```
ng : 이 시스템에서 스크립트를 실행할 수 없으므로 C:\Users\< username >\AppData\Roaming\npm\ng.ps1 파일을 로드할 수 없습니다. 
자세한 내용은 about_Execution_Policies(https://go.microsoft.com/fwlink/?LinkID=135170)를 참조하십시오.
위치 줄:1 문자:1
...
...
...
```
## 실패 케이스
1. `C:\Users\< username >\AppData\Roaming\npm\ng.ps1` 읽기 모드 해제<br/>
-> 이미 해제 되어 있으며, 상위 폴더의 읽기 모드를 해제 하여도 동작하지 않습니다.<br/>
<br/>
2. `C:\Users\< username >\AppData\Roaming\npm\ng.ps1` 삭제<br/>
-> npm install -g @angular/cli 실행하면 다시 설치되며 같은 에러가 발생합니다.<br/>


## 해결

```
Set-ExecutionPolicy -ExecutionPolicy
// 또는
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

스크립트 정책이 풀리면서 실행 가능해집니다.




## 참고 사이트
- [why-powershell-does-not-run-angular-commands](https://stackoverflow.com/questions/58032631/why-powershell-does-not-run-angular-commands)