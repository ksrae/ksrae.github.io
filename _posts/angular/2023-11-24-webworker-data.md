---
title: "ngDoc - Install"
date: 2023-09-27 16:32:00 +0900
comments: true
categories: angular
tags: [document, ngdoc]
---

# Angular에서 Worker 활용하기

Worker는 JavaScript 코드를 백그라운드 스레드에서 실행할 수 있게 해주는 기술입니다. Angular 애플리케이션에서 Worker를 통해 백그라운드에서 작업을 처리하면 메인 스레드의 블로킹을 피하고 성능을 향상시킬 수 있습니다. 이번 글에서는 Angular에서 Worker를 만들고 사용하는 방법에 대해 알아보겠습니다.


## 1. Worker 생성
아래의 명령을 사용하면 worker를 자동으로 생성해주어 편리합니다. 특히 nx를 사용하는 경우  tsconfig.webworker.ts를 자동으로 생성해주므로 반드시 ng generate 또는 ng g를 사용해서 생성하는 것을 권장합니다.<br/>

```
ng generate web-worker [경로]

```
다만 자동으로 만드는 코드는 component에서 직접 worker에 접근하는 코드이며, 아래에서 설명할 instance 를 통한 방법은 아니기 때문에 아래 코드를 적용하기 위해서는 생성 후 일부 코드를 수정해야 합니다.



## 2. Worker File 작성

먼저, Angular 프로젝트 내에서 Worker 파일을 생성합니다. 일반적으로 `.worker.ts` 확장자를 사용합니다. 아래는 간단한 예제입니다.

```typescript
// app.worker.ts
self.addEventListener('message', ({ data }) => {
  // Do some work in the background
  const result = data * data;

  // Send the result back to the main thread
  postMessage(result);
});
```

## 3. Worker Service 작성
이제 Angular 서비스를 생성하여 Worker를 사용합니다. <br/>
일반적으로 Service에는 Injectable을 주입하지만, Worker는 Component가 별도의 Worker를 갖도록 하여 재활용 시 Worker가 공유되는 문제를 막기 위해서 Injectable을 주입하지 말고, 각 Component에서 Instance를 각각 생성하는 것이 좋습니다.<br/>

```typescript
// worker.service.ts
export class WorkerService {
  private worker: Worker;
  worker$ = new Subject();

  constructor() {
    this.worker = new Worker('./app.worker', { type: 'module' });
  }

  runWorker(input: number) {
    this.worker.onmessage = ({ data }) => {
      this.worker$.next(data);
    };

    this.worker.onerror = (error) => {
      console.error(error);
    };

      // Start the worker task
    this.worker.postMessage(input);
  }
}
```

## 4. Component 에서 Worker 연결
마지막으로, Angular 컴포넌트에서 위에서 만든 서비스를 사용하여 웹 워커를 호출합니다.

```typescript
// app.component.ts
import { Component } from '@angular/core';
import { WorkerService } from './worker.service';

@Component({
  selector: 'app-root',
  template: `
    <button (click)="runWorker()">Run Worker</button>
    <p *ngIf="result !== undefined">Result: {{ result }}</p>
  `,
})
export class AppComponent {
  result: number | undefined;
  worker = new WorkerController();
  constructor() {}

  ngOnInit(): void {
    this.worker.worker$.pipe(
      tap((response: any) => {
        result = response;
        // set change detection if template is not updated.
      }),
      takeUntil(this.worker$) // subscription
    ).subscribe();
  }

  runWorker() {
    this.worker.runWorker();
  }

  ngOnDestroy() {
    this.worker$.next(); // unsubscription
    this.worker$.complete();
  }
}
```

이제 Worker를 통해 백그라운드에서 작업을 처리하고 결과를 메인 스레드에 전달할 수 있습니다. 이를 통해 애플리케이션의 성능을 향상시킬 수 있습니다.

