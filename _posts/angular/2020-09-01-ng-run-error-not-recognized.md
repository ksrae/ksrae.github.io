---
title: "'ng' 용어가 cmdlet, 함수, 스크립트 파일 또는 실행할 수 있는 프로그램 이름으로 인식되지 않습니다. (the term 'ng' is not recognized as an internal or external command, operable program or batch file.)"
date: 2020-09-01 17:30:00 +0900
comments: true
categories: angular
tags: [error]
---


## 현상
windows에서 Angular cli를 설치합니다.

```
> npm i -g @angular/cli
```

ng 명령을 실행하면 다음과 같은 에러가 발생합니다.

```
'ng' 용어가 cmdlet, 함수, 스크립트 파일 또는 실행할 수 있는 프로그램 이름으로 인식되지 않습니다.
(the term 'ng' is not recognized as an internal or external command, operable program or batch file.)
```


## 해결

### 환경변수 수정
1. 환경변수 (Environment Variables)를 엽니다.<br/>
2. 시스템 환경에서 Path를 찾아 엽니다. <br/>
3. `%AppData%\npm` 를 추가합니다. <br/>
4. nodejs 보다 위에 위치하도록 위치를 옮깁니다. 또는 맨 앞으로 옮깁니다.<br/>


### angular/cli 재설치
1. 기존 angular/cli 제거
```
npm uninstall -g @angular/cli
```

2. 캐시 제거
```
npm cache clean --force
```

이 때 반드시 --force 옵션을 넣어야 합니다. 그렇지 않으면 실행이 취소되며 npm cache verify 를 권장합니다.

3. 재설치
```
npm install -g @angular/cli
```


## 참고 사이트
[the-term-ng-is-not-recognized-as-the-name-of-a-cmdlet](https://stackoverflow.com/questions/44958847/the-term-ng-is-not-recognized-as-the-name-of-a-cmdlet/44958882)

[the-term-ng-is-not-recognized-as-the-name-of-a-cmdlet-in-angular](https://stackoverflow.com/questions/59545882/the-term-ng-is-not-recognized-as-the-name-of-a-cmdlet-in-angular)