---
title: "entire search and range search query for 2-byte-letters"
date: 2019-07-26 16:53:00 +0900
comments: true
categories: mongodb
tags: [query]
---


MongoDB에서 전체 검색과 범위 검색 쿼리 만드는 방법입니다.


## 전체검색
전체 검색 쿼리는 그냥 쿼리문에 해당 키를 적지 않으면 되는데 코딩하다보면 if 조건에 의해 키를 넣어야 하는 상황이 생깁니다. <br><br>

이 때, 전체 검색하는 방법은 regex 를 활용하는 것인데 /./ 로 처리할 수 있습니다.

```ts
if(value) {
  DB.find({
    key: value ? value : /./
  }).toArray();
}
```

## 범위 검색

전화번호부 검색과 같이 첫글자 '가~나' 인 모든 값을 검색하는 방법입니다.<br>
구글에 검색해봐도 없어서 어려운건가 했는데 생각보다 쉽게 검색할 수 있습니다.<br><br>

바로 $gte와 $lt를 사용하는 방법입니다.<br>
$gte는 ()>=) $lt는 (<)를 의미합니다. <br><br>

일반적으로 비교 연산은 숫자에 사용하는데 문자에 그것도 한글과 같은 2바이트 문자에도 사용이 가능합니다.


```ts
  DB.find({
    key: {
      '$gte':'가', '$lt': '나'
    }
  };
```  
  
쉽지만 모르면 헤매게 되는 검색 방법에 대해 알아보았습니다.<br><br>

끝.