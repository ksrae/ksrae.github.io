---
title: "compodoc - 새 라우팅 방식 인식 실패 (Fail to Understand New Routing Structure)"
date: 2019-07-11 13:44:00 +0900
comments: true
categories: angular
tags: [compodoc, error]
---



compodoc에서 문서화를 시도할 때 Angular 8 에서 변경된 routing 방식 ( import().then(m->m.Module) ) 을 인식하지 못하는 문제가 발생합니다.<br><br>


      Routes parsing error, maybe a trailing comma or an external variable, trying to fix that later after sources scanning.


딱히 해결 방법은 없고 이 문제가 해결될 때까지 문서를 구성할 때 routing 구조를 그려주는 것을 무시하도록 하거나 <br>코드를 기존 routing 구조로 변경하여 문서를 만든 뒤 다시 새 routing 방식으로 변경하는 방법 밖에는 없습니다.<br><br>


공식 사이트의 Usage 메뉴를 보면 옵션 목록이 나오는데 --disableRoutesGraph 옵션을 사용하면 routing 그리는 것을 무시할 수 있습니다.<br>

      npx compodoc -p src/tsconfig.app.json --disableRoutesGraph 


[업데이트 2021.07.12]
- compodoc이 업데이트 되어 위의 문제는 해결 되었습니다.

참고: [Compodoc Guides](https://compodoc.app/guides/getting-started.html)