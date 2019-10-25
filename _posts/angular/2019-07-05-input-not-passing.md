---
title: "@Input Not Passing Data"
date: 2019-07-05 12:08:00 +0900
comments: true
categories: angular
tags: [input, ngonchange, subject, observable]
---

@Input이 데이터를 전달하지 못하는 원인을 분석합니다.<br>



## 상황

페이지를 유지한 상태(route 하지 않은 상태)에서 데이터를 업데이트할 때 <br>@Input으로 전달한 child나 directive가 동작하지 않는 상태.<br>

## 증상

- 동일한 데이터는 강제로 보내더라도 기존 상태를 유지하기 때문에 child나 directive에 전달하지 않음.
- Array에 아무리 push를 해도 이를 child나 directive를 전달하지 않음.

## 해결
1. 기존과 같은 값을 전달하였는지 확인합니다. <br>Input은 동일한 값은 전달하지 않습니다. <br>즉, 동일한 값이 여러번 들어오면 이를 무시하여 이벤트를 일으키지 않습니다.<br><br>
2. ngOnChange(value: SimpleChanges) 에 이벤트가 들어오는지 확인합니다. <br>만일 들어온다면 LifeCycle의 문제로 받지 못하였을 수 있습니다. <br>ngOnChange에서 직접 처리하면 되는데 ngOnChange는 ngOnInit보다 선행처리 되므로 유의하여야 합니다.<br><br>
3. 그래도 안되면 Subject나 Observable로 전달하는게 확실합니다. <br>위의 모든 증상에도 확실히 전달하므로 가장 확실한 해결 방법입니다.<br>
