---
title: "Error Fix for Keycloak with Angular Module Federation (Angular Module Federation에서 keycloak 에러 해결하기)"
date: 2024-09-26 13:59:00 +0900
comments: true
categories: angular
tags: [standalone, keycloak, module-federation]
---

오늘은 keycloak과 module federation이 무엇인지 정의하고 발생한 에러를 해결할 방법을 알아보겠습니다.

# Keycloak
Keycloak은 통합 인증 시스템으로, 아이덴티티 및 접근 관리(Identity and Access Management)를 제공하는 오픈 소스 소프트웨어입니다.<br/>
주요 기능으로는 싱글 사인온(SSO)을 지원하여 사용자가 여러 애플리케이션에 대해 반복적인 로그인 과정을 겪지 않도록 도와줍니다.

- 사용자 관리: Keycloak은 다양한 인증 방법을 지원하고, 중앙 집중식 사용자 관리 기능을 제공합니다. 이를 통해 사용자는 하나의 계정으로 여러 서비스에 접근할 수 있습니다.
- 보안 강화: OAuth2, OpenID Connect와 같은 현대적 보안 프로토콜을 지원하여, 안전하게 애플리케이션과 서비스 간의 인증을 처리합니다.
- 커스터마이징 용이성: Keycloak은 기본 제공되는 기능을 바탕으로 필요한 대로 커스터마이즈할 수 있어, 개발자에게 유연한 솔루션을 제공합니다.
- 오픈 소스: Keycloak은 오픈 소스 프로젝트로, 커뮤니티의 지원을 받을 수 있으며, 필요할 경우 직접 소스를 수정할 수 있습니다.

# Module Federation
Module Federation은 웹팩 5의 기능으로, 마이크로 프론트엔드 아키텍처를 손쉽게 구현할 수 있게 도와줍니다. 이를 통해 여러 개의 독립적인 애플리케이션 모듈 간에 구성을 공유하고 동적으로 로드할 수 있습니다. <br/>

- 독립적인 빌드와 배포: 각 모듈은 독립적으로 개발되고 배포될 수 있어, 특정 기능의 변경이 전체 애플리케이션에 영향을 주지 않습니다. 이는 CI/CD 파이프라인을 보다 효율적으로 구성할 수 있게 합니다.
- 공통 코드의 재사용: Module Federation을 통해 애플리케이션 간에 공통 라이브러리나 컴포넌트를 공유할 수 있습니다. 이를 통해 중복된 코드를 줄이고, 일관된 사용자 경험을 제공할 수 있습니다. 예를 들어, 여러 개의 애플리케이션에서 공유하는 디자인 시스템을 쉽게 구축할 수 있습니다.
- 실시간 업데이트: 공통 컴포넌트가 업데이트되었을 경우, 애플리케이션은 재배포 없이 런타임에서 최신 버전을 자동으로 사용할 수 있습니다. 이는 버전 관리에서 상당한 편리를 제공합니다.
- 규모 확장 용이: 각 모듈이 독립적으로 작동하므로, 필요에 따라 시스템을 손쉽게 확장할 수 있는 구조를 갖출 수 있습니다. 이로 인해, 특정 기능이나 서비스가 증가하여도 전체 애플리케이션의 성능에 미치는 영향을 줄일 수 있습니다.

# Angular 17에서 에러 해결하기

## Environment
- Angular with standalone (17.0.1)
- Angular module federation (17.0.1)
- nx (@nx/js 17.1.3)
- keycloak-angular (15.0.0)
- keycloak-js (22.0.5)

Angular 17 환경에서 Keycloak을 통합하는 과정에서 발생하는 "WEBPACK_IMPORTED_MODULE_5 is not a constructor" 에러는 다양한 요인에 의해 발생할 수 있으며, 이를 해결하기 위해 체계적인 점검과 수정이 필요합니다. 다음은 이 문제의 원인과 함께 효과적인 해결책을 제시합니다.

## 버전 호환성 확인
Angular 17과 함께 사용하는 keycloak-angular 및 keycloak-js와 같은 라이브러리의 버전이 호환되는지 먼저 확인해야 합니다.<br/>
잘못된 버전의 라이브러리를 사용하면 의도한 대로 동작하지 않거나 오류가 발생할 수 있습니다. 다음 단계에 따라 확인하시기 바랍니다:

### 현재 설치된 패키지 버전 확인
npm list keycloak-angular keycloak-js 명령어를 사용하여 설치된 버전을 확인합니다. 이때 설치된 버전이 Angular 17과 호환되는지 확인해야 합니다. Angular의 공식 문서나 패키지의 GitHub 페이지를 참고하여 권장하는 버전을 찾아보세요.


### 의존성 업데이트
만약 버전이 호환되지 않는다면, npm install keycloak-angular@<희망하는버전> 또는 npm install keycloak-js@<희망하는버전>과 같은 명령어로 적절한 버전으로 업데이트합니다.

## angular.json 파일 설정

angular.json 파일에서 빌드 설정이 올바르게 되어 있는지도 확인해야 합니다. 특히 allowedCommonJsDependencies 속성은 CommonJS 모듈 사용을 허용하는 설정으로, 다음과 같이 구성되어야 합니다:

```
"allowedCommonJsDependencies": [
  "base64-js",
  "js-sha256",
  "keycloak-js"
]
```

이 설정은 Angular 애플리케이션이 사용하는 CommonJS 모듈을 명시적으로 허용하여 모듈 페더레이션에서 발생할 수 있는 충돌을 최소화하는 데 기여합니다.

## Webpack 설정 점검

Keycloak과 관련된 차별화된 설정이 필요합니다. Module Federation의 목적상 Keycloak을 공유와 스킵 리스트에 추가하는 것이 중요합니다. 다음과 같이 Webpack 설정 파일을 조정하십시오:

```
const sharedConfig = {
  singleton: false,
  strictVersion: false,
  requiredVersion: 'auto'
};

const skipList = [
  '@angular-architects/module-federation',
  'tslib',
  'zone.js',
  'keycloak-js'
];

shareAll(sharedConfig, skipList);
```

이렇게 설정하면 Keycloak의 의존성이 Module Federation에 의해 처리되지 않도록 하며, 이는 모듈 간의 충돌을 방지하고 런타임에서 발생할 수 있는 오류를 사전에 예방합니다.

## 디버깅과 추가 로그

문제가 지속되면, 에러 메시지가 발생하는 부분에서 추가적인 로깅을 통해 문제의 근본 원인을 파악하는 것도 중요한 방법입니다. 예를 들어, Keycloak을 초기화할 때 각 단계를 헬퍼 함수로 분리하여 어떤 단계에서 오류가 발생하는지 확인할 수 있습니다:

```
const initKeycloak = async () => {
  try {
    const keycloak = new Keycloak();
    await keycloak.init({
      onLoad: 'login-required',
      checkLoginIframe: false,
    });
    console.log('Keycloak initialized successfully');
  } catch (error) {
    console.error('Error initializing Keycloak: ', error);
  }
};
```

이와 같은 형식으로 로깅을 추가하면, 정확한 오류의 위치와 이유를 진단하는 데 큰 도움이 됩니다.


## 결론

이렇게 여러 단계를 통해 "WEBPACK_IMPORTED_MODULE_5 is not a constructor" 문제를 체계적으로 해결할 수 있습니다. 각 점검 항목과 조정 사항들을 바탕으로 Keycloak과 Angular 환경을 효과적으로 통합하여 안정적인 애플리케이션을 구축하시기 바랍니다.


# References
[MFE WEBPACK_IMPORTED_MODULE_5 is not a constructor #465](https://github.com/mauriciovigolo/keycloak-angular/issues/465)