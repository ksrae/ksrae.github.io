---
title: "Enhancing Web Worker Performance with Transferable Objects in Angular"
date: 2023-11-24 16:22:00 +0900
comments: true
categories: angular
tags: [webworker]
---

# 소개

웹 워커(worker)는 복잡한 계산과 다양한 작업을 동시에 처리하는 성능을 향상 시키기 위한 강력한 도구 중 하나로, 별도의 스레드에서 스크립트를 실행하여 메인 스레드의 부하를 줄일 수 있습니다.
<br/>
그러나 메인 스레드와 워커 간 데이터 전송은 여전히 성능 문제를 야기할 수 있습니다. 이때 Transferable Objects가 강력한 도움을 줄 수 있습니다.

# Transferable Objects란?

Transferable Objects는 메인 스레드와 워커 사이에서 데이터를 효율적으로 전송하기 위한 방법 입니다. 이를 통해 데이터를 복사하는 대신 소유권을 전송함으로써 성능을 향상시킬 수 있습니다.

주로 사용되는 Transferable Objects는 다음과 같습니다.

- ArrayBuffer: 바이너리 데이터를 효율적으로 다룰 수 있는 객체로, 메모리를 공유하는 데에 적합합니다.

- MessagePort: 워커 간 메시지를 전송하는 데 사용되며, 데이터를 복사하지 않고도 전송할 수 있습니다.

- ImageBitmap: 이미지 데이터를 처리할 때 유용하게 활용됩니다.

# Transferable Objects의 활용 예시

## component
먼저, Angular 애플리케이션에서 Transferable Objects를 사용하는 예시 코드를 살펴봅시다. <br/>
AppComponent는 웹 워커를 통해 병렬 처리를 수행하고, Transferable Objects를 사용하여 데이터를 주고받습니다.

```tsx
export class AppComponent implements OnInit, OnDestroy {
  worker = new WorkerController();
  readonly worker$ = new Subject<void>(); // subscription

  ngOnInit(): void {
    this.worker.worker$.pipe(
      tap((response: any) => {
        console.log({response});
      }),
      takeUntil(this.worker$) // subscription
    ).subscribe();
  }

  calculate() {
    this.worker.calculate();
  }

  ngOnDestroy() {
    this.worker$.next(); // unsubscription
    this.worker$.complete();
  }
}

```

## worker instance
instance는 웹 워커를 생성하고 데이터를 주고받는 역할을 합니다. 데이터를 ArrayBuffer 형태로 encoding하여 전송하고, 수신한 데이터를 json으로 decoding하여 Component에 전달합니다.


```tsx
import { Subject } from 'rxjs';

export class WorkerController {
  worker: any;
  worker$ = new Subject();

  constructor() {
    if (typeof Worker !== 'undefined') {
      // creating web worker
      this.worker = new Worker(new URL('./app.worker', import.meta.url));

      // receive
      this.worker.onmessage = ({ data }: any) => {
        // transferable object 형태
        const resultBuffer = new Uint8Array(data);
        const resultString = new TextDecoder().decode(resultBuffer);
        this.worker$.next(resultString ? JSON.parse(resultString) : '');
      };




    } else {
      console.error('this environment does not support web worker');
    }
  }
  calculate(min: number = 1, max: number = 100) {
    // send
    this.postMessage({
      {
        min,
        max,
      }
    });
  }

  private postMessage(data: any) {
    // json -> string -> buffer 형태로 변환
    const jsonBuffer = this.convertToBuffer(data);
    this.worker.postMessage(jsonBuffer, [jsonBuffer]);
  }
  private convertToBuffer(jsonData: any) {
    return new TextEncoder().encode(JSON.stringify(jsonData)).buffer;
  }
  destroy() {
    this.worker.terminate();
  }
}

```


## worker
worker 에서는 수신한 ArrayBuffer 데이터를 다시 json으로 decoding하여 처리한 뒤 다시 encoding하여 메인 스레드에 전달합니다.


```tsx
/// <reference lib="webworker" />

addEventListener('message', ({data}) => {
  console.log('getmessage', data);

  // data를 받아 decode 하고 처리한 뒤 다시 encode
  // buffer -> string -> json으로 변환 후 postMessage
    const obj = decodeBuffer(data);

    let sum = 0;
    for(let i=obj.min; i<=obj.max;i++) {
      sum+=(Math.pow(i, 2));
    }

    postMessage(encodeString(sum));
  
});


function decodeBuffer(data: any) {
  const jsonBuffer = new Uint8Array(data);
  const jsonString = new TextDecoder().decode(jsonBuffer);
  return JSON.parse(jsonString);
}

function encodeString(data: any) {
  const resultString = JSON.stringify(data);
  return new TextEncoder().encode(resultString);
}

```

# 결론
Transferable Objects를 통해 메인 스레드와 워커 간의 데이터 전송 성능을 향상시킬 수 있습니다. 특히 대용량의 데이터를 다루거나 병렬 처리가 필요한 웹 애플리케이션에서는 이를 적극적으로 활용하여 성능 최적화를 달성할 수 있습니다. Transferable Objects의 활용은 데이터 복사 비용을 최소화하고 메모리를 효율적으로 활용하는 데에 큰 도움이 됩니다.
