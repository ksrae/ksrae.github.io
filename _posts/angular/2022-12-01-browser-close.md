---
title: "How to handle browser close (unload event)"
date: 2022-12-01 10:47:00 +0900
comments: true
categories: angular
tags: [unload, beforeunload]
---
이번 시간은 Browser를 닫을 때 이벤트를 처리하는 방법에 대해 정리해보고 예제를 통해 알아보겠습니다.

.

# Beforeunload & Unload Event

browser를 닫을 경우 beforeunload와 unload 두 가지 이벤트가 발생합니다.

우선 각각의 이벤트에 대해 알아봅시다.

.

## beforeunload

document unload를 진행하기 전에 이벤트가 발생합니다. document는 여전히 visible 상태이며 이 이벤트는 취소 가능합니다.

이 이벤트가 발생하면 confirm 이벤트가 발생하며, 이 때 unload를 진행하거나 또는 취소 할 수 있습니다.

단, confirm 이벤트를 발생시키려면 preventDefault() 또는 return false를 구현해야 합니다.

.

## unload

beforeunload 이후에 발생하는 이벤트 입니다. 일반적으로 document가 unload 될 때 발생하는 이벤트로,

- 모든 리소스가 아직 존재합니다.
- 존재하지만 user는 더이상 리소스를 볼 수 없습니다.
- UI 이벤트는 동작하지 않습니다. (open, alert, confirm 등)
- unload 이벤트는 어떠한 상황에도 취소할 수 없습니다.

.

# Angular Example

angular에서는 @Hostlistener를 통해 이벤트를 받아 처리합니다.

.

## beforeunload

### 별도의 함수에서 처리하는 방법

```
  @HostListener(`window:beforeunload`, [ `$event` ])
  beforeunload(e: any) {
    return false;
  }
```

### OnDestroy에서 처리하는 방법

```
  @HostListener('window:beforeunload')
  ngOnDestroy() {
    console.log('destroy');
    return false;
  }
```

.

# Avoid unload Event

unload 이벤트는 더이상 사용하지 않을 것을 권장합니다. 이유는 다음과 같습니다.

- 모바일 유저가 접속한 뒤 다른 앱으로 전환한 뒤 app manager를 통해 browser를 닫을 경우 unload 이벤트가 동작하지 않습니다.
- unload event 이후에는 bfcache는 발생하지 않습니다.
- 일부 페이지에서는 unload 이벤트가 있는 페이지의 경우 bfcache를 적용하지 않으며 이는 매우 좋지 않은 performance를 유발합니다.
- 일부 브라우저는 bfcache 이슈로 unload 이벤트를 금지합니다.

.

# Why Alert, Confirm not working?

HTML 규약에 따르면 window.alert, window.confirm, window.prompt 이벤트는 무시됩니다.

또한 최신 브라우저의 대부분은 더 강화된 보안에 따라 browser의 confirm message를 더이상 customize할 수 없습니다.

.

# Why Not Fire Such Events?

beforeunload 또는 unload event는 다음의 현상에 동작합니다.

- browser를 닫음
- 해당 tab을 닫음
- 해당 tab에서 refresh를 시도함 (f5 또는 브라우저의 refresh button 또는 스크립트에서 강제 실행 등)
- back 또는 forward (브라우저의 버튼 또는 스크립트에서 강제 실행)

그런데 막상 구현하여 테스트를 해보면 이벤트가 전혀 동작하지 않는 것을 확인할 수 있습니다.

최신 브라우저들은 페이지 내에서 유저의 동작이 없는 경우 탭 또는 브라우저를 닫을 때 beforeunload 또는 unload 이벤트를 발생시키지 않고 즉시 닫아버리기 때문입니다.

따라서 해당 이벤트를 적용하려면 닫기 전에 반드시 페이지 내에서 활동을 (최소한 화면 drag라도) 해야 적용됩니다.

.

# 대안 Event

## visibilitychange

browser의 모든 변화를 감지하고 이를 알려줍니다.

다만, 최소화 여부 일 때만 감지가 가능하며, (최소화 = hidden, 최소화가 아닐 때 = visible) close는 감지할 수 없으므로, 이 Event는 unload에는 적합하지 않습니다.

```
  @HostListener('document:visibilitychange', ['$event'])
  visibilitychanges() {
    console.log(document.visibilityState);
    return false;
  }
```

.

## pagehide

unload 이벤트의 대안으로 추천하는 방법입니다.

그 이유는 바로 bfcache의 적용 때문인데 pagehide를 활용할 경우 bfcache의 영향을 받지 않아 페이지의 performance에 영향을 주지 않기 때문입니다.

따라서 많은 블로그들이 이 Event를 대안으로 내세우고 있습니다. 이와 반대로 load 이벤트의 대안으로 pageshow Event가 있습니다.

참고로 이 이벤트는 unload의 대안이므로 취소가 불가능합니다.

```
  @HostListener(`window:pagehide`, [ `$event` ])
  pageHide(e: any) {
    e.preventDefault();
  }
```

.

# beforeunload 와 pagehide 함께 사용하기

일반적으로 beforeunload 이벤트가 동작하면 pagehide는 동작하지 않습니다.

그러나 refresh 일 때는 beforeunload 이후 pagehide가 동작합니다. (이는 unload 이벤트가 동작하는 방식과 같습니다.)

따라서 만일 refresh 시에만 처리해야할 로직이 있는 경우 함께 사용할 수 있습니다.

```
  @HostListener(`window:pagehide`, [ `$event` ])
  pageHide(e: any) {
    console.log('pagehide');
  }

  @HostListener(`window:beforeunload`, [ `$event` ])
  beforeunload(e: any) {
    console.log('beforeunload')
    return false;
  }
```

.

# Reference

[unload event](https://developer.mozilla.org/en-US/docs/Web/API/Window/unload_event)

[becache](https://web.dev/bfcache/#only-add-beforeunload-listeners-conditionally)

[custom confirmation message is not working](https://stackoverflow.com/questions/38879742/is-it-possible-to-display-a-custom-message-in-the-beforeunload-popup)
