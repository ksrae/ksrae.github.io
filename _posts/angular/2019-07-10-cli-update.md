---
title: "Angular-Cli Update"
date: 2019-07-10 11:00:00 +0900
comments: true
categories: angular
tags: [update]
---



버전 업데이트 과정을 설명합니다. 여기에서는 7버전에서 8버전 업데이트를 설명합니다. 모든 버전 업데이트의 방식이 동일하므로 다른 버전도 아래와 같이 진행하면 됩니다.

### 1. 업데이트 확인
> ng update

업데이트가 필요한지 확인할 수 있습니다. 만일 업데이트가 필요하면 아래와 같이 업데이트 목록이 표시됩니다.

```
    We analyzed your package.json, there are some packages to update:

      Name                               Version                  Command to update
     --------------------------------------------------------------------------------
      @angular/cli                       7.2.4 -> 8.1.0           ng update @angular/cli
      @angular/core                      7.2.15 -> 8.1.0          ng update @angular/core
      rxjs                               6.3.3 -> 6.5.2           ng update rxjs


    There might be additional packages that are outdated.
    Run "ng update --all" to try to update all at the same time.
```   
      


### 2. update

위에 표시된 목록을 업데이트 시켜줍니다.

> ng update @angular/cli @angular/core rxjs


```
    Updating package.json with dependency @angular/language-service @ "8.1.0" (was "7.2.15")...
    Updating package.json with dependency @angular/animations @ "8.1.0" (was "7.2.15")...
    Updating package.json with dependency @angular/compiler-cli @ "8.1.0" (was "7.2.15")...
    Updating package.json with dependency @angular/compiler @ "8.1.0" (was "7.2.15")...
    Updating package.json with dependency @angular/common @ "8.1.0" (was "7.2.15")...
    Updating package.json with dependency @angular/core @ "8.1.0" (was "7.2.15")...
    Updating package.json with dependency @angular/forms @ "8.1.0" (was "7.2.15")...
    Updating package.json with dependency zone.js @ "0.9.1" (was "0.8.29")...
    Updating package.json with dependency @angular/platform-browser-dynamic @ "8.1.0" (was "7.2.15")...
    Updating package.json with dependency @angular/platform-browser @ "8.1.0" (was "7.2.15")...
    Updating package.json with dependency @angular/router @ "8.1.0" (was "7.2.15")...
    Updating package.json with dependency rxjs @ "6.5.2" (was "6.3.3")...
    Updating package.json with dependency @angular/cli @ "8.1.0" (was "7.2.4")...
    Updating package.json with dependency @angular-devkit/build-angular @ "0.801.0" (was "0.12.4")...
    Updating package.json with dependency typescript @ "3.4.5" (was "3.2.4")...
UPDATE package.json (1305 bytes)

> core-js@3.1.4 postinstall E:\projects\babaopt\node_modules\@angular-devkit\build-angular\node_modules\core-js
> node scripts/postinstall || echo "ignore"


> @angular/cli@8.1.0 postinstall E:\projects\babaopt\node_modules\@angular\cli
> node ./bin/postinstall/script.js

npm WARN codelyzer@4.5.0 requires a peer of @angular/compiler@>=2.3.1 <8.0.0 || >7.0.0-beta <8.0.0 but none is installed. You must install peer dependencies yourself.
npm WARN codelyzer@4.5.0 requires a peer of @angular/core@>=2.3.1 <8.0.0 || >7.0.0-beta <8.0.0 but none is installed. You must install peer dependencies yourself.
npm WARN ajv-keywords@3.4.1 requires a peer of ajv@^6.9.1 but none is installed. You must install peer dependencies yourself.
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.9 (node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.9: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

added 71 packages from 46 contributors, removed 151 packages, updated 115 packages, moved 11 packages and audited 19057 packages in 107.435s
found 1 low severity vulnerability
  run `npm audit fix` to fix them, or `npm audit` for details
    ** Executing migrations for package '@angular/cli' **
RENAME src/browserslist => /browserslist
UPDATE tslint.json (2819 bytes)
UPDATE package.json (1306 bytes)
UPDATE src/polyfills.ts (2838 bytes)
UPDATE tsconfig.json (470 bytes)
UPDATE src/tsconfig.app.json (166 bytes)
UPDATE src/tsconfig.spec.json (256 bytes)
npm WARN ajv-keywords@3.4.1 requires a peer of ajv@^6.9.1 but none is installed. You must install peer dependencies yourself.
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.9 (node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.9: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

added 4 packages from 7 contributors, updated 1 package and audited 19063 packages in 7.557s
found 1 low severity vulnerability
  run `npm audit fix` to fix them, or `npm audit` for details

```


업데이트가 완료되었습니다.


### 3. Warning / Error fix
업데이트 중 WARN 또는 Error가 발생한 경우 [Error-Fix 페이지에서 확인하실 수 있습니다.](https://ksrae.github.io/angular/error-fix)