---
title: "subscribe 제한 (subscribe filtering)"
date: 2019-07-05 15:00:00 +0900
comments: true
categories: angular
tags: [rxjs, subscribe]
---


1. take(n): n번 subscribe 후 해제
![?fname=http%3A%2F%2Fcfile30.uf.tistory.com%2Fimage%2F997D983359BA8B6804950D](http://img1.daumcdn.net/thumb/R1920x0/?fname=http%3A%2F%2Fcfile30.uf.tistory.com%2Fimage%2F997D983359BA8B6804950D)

```ts
this.mount.pipe(take(2)).subscribe();
```

2. takeUntil(Observable): Observable이 데이터를 받거나 완료되면 해제
![?fname=http%3A%2F%2Fcfile3.uf.tistory.com%2Fimage%2F99591B3359BB22561F61F7](http://img1.daumcdn.net/thumb/R1920x0/?fname=http%3A%2F%2Fcfile3.uf.tistory.com%2Fimage%2F99591B3359BB22561F61F7)

```ts
unsubscribe: Subject = new Subject(void);

public ngOnInit() {
  this.tagService.getTags({category: 'keyword'})
    .takeUntil(this.unsubscribe)
    .subscribe(tags => {
      this.tags = tags;
    });
}

public ngOnDestroy() {
  this.unsubscribe.next();
  this.unsubscribe.complete();
}
```

3. takeWhile(function): 함수가 true이면 해제
![?fname=http%3A%2F%2Fcfile9.uf.tistory.com%2Fimage%2F99F6553359BA8BA230F5B0](http://img1.daumcdn.net/thumb/R1920x0/?fname=http%3A%2F%2Fcfile9.uf.tistory.com%2Fimage%2F99F6553359BA8BA230F5B0)

```ts
alive = true;

ngOnInit() {
  this.filterChange$
    .takeWhile(() => this.alive)
    .subscribe(filter => {
      // 필터가 변경되었을때의 처리
    });

  this.keywordChange$
    .takeWhile(() => this.alive)
    .subscribe(keyword => {
      // 키워드가 변경되었을때의 처리
    });
}

ngOnDestroy() {
  this.alive = false;
}
```

4. takeLast(n): 마지막 n번째 값만 처리하고 해제. 마지막이 어디인지 명확히 알아야 함
![takeLast.png](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/takeLast.png)

5. first(): 첫번째만 처리하고 해제
![first.png](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/first.png)

```ts
this.mount.pipe(first()).subscribe();
```

6. take(function): 조건이 맞으면 해제
![take.png](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/take.png)

```ts
this.mount.pipe(take(2)).subscribe();
```

7. skip(n): 1부터 n개  스킵하고 나머지만 처리
![skip.png](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/skip.png)


8. Last() , Last(function): takeLast(1)과 동일하거나 조건에 맞는 최종 데이터만 처리
![last.png](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/last.png)


9. SkipLast(n):  마지막부터 n개 스킵한 나머지만 처리
![skipLast.png](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/skipLast.png)


10. SkipUntil(Observable): Observable이 Observable이 데이터를 받거나 완료되기 전까지를 스킵
![skipUntil.png](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/skipUntil.png)


11. SkipWhile(function): 함수가 true이기 전까지 스킵
![skipWhile.png](https://rxjs-dev.firebaseapp.com/assets/images/marble-diagrams/skipWhile.png)


* 위의 내용은 Rxjs 공식 홈페이지를 참조하였습니다.
* [RxJS](https://rxjs-dev.firebaseapp.com/api/operators)

