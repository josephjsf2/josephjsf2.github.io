---

title: Rxjs - Operators part I
layout: post
categories: angular
tags:

- angular
- Observable
- Operator
- rxjs

---

Observable Operator使用方式。

<!--more-->

### 1. tap

> [Perform a side effect for every emission on the source Observable, but return an Observable that is identical to the source.](https://rxjs-dev.firebaseapp.com/api/operators/tap)

原本的 do 用法，通常 Observable 會搭配 tap 來執行有 side effect的工作，如Observable emit value後，要改變其他物件的狀態，或是更新UI 畫面等，每一次 Observable emit 一個內容都會進入 tap 中。

如果要透過 async pipe來訂閱 Observable， 搭配 tap 會是不錯的選擇( 純個人意見)。

下列程式碼寫法不同，但是做的事情是相同的

```typescript
    of(1,2,3,4,5).subscribe(
      val => this.count1 +=val
    );

    of(1,2,3,4,5).pipe(
      tap(
        val => this.count2 += val   
      )
    ).subscribe()

    const obs:Observer<number> = {
      next: val => this.count3+=val,
      error: noop,
      complete: noop
    }
    
    this.count$ = of(1,2,3,4,5).pipe(tap(obs))
```

[stackblitz](https://stackblitz.com/edit/angular-nhh8rd)

### 2. map

>[Applies a given `project` function to each value emitted by the source Observable, and emits the resulting values as an Observable.](https://rxjs-dev.firebaseapp.com/api/operators/map)

```typescript
    of(1,2,3,4,5).pipe(
      tap( val => console.log(`receive val ${val}`)),
      map( val => val *2),
      tap( val => console.log(`revceive val after map: ${val}`))
    ).subscribe()
```

[stackblitz](https://stackblitz.com/edit/angular-ao8zo2)

### 3. pluck

> [Maps each source value (an object) to its specified nested property.](https://rxjs-dev.firebaseapp.com/api/operators/pluck)

將 Observable資料流中之物件，透過 pluck可以轉為物件中特定屬性內容。

```typescript
    const persons = [{
      name: 'Tom',
      age: 18
    },{
      name: 'Jessie',
      age: 26
    }]
    of(...persons).pipe(
      pluck('age'),
      tap(val => console.log(val))
    ).subscribe()
```

[stackblitz](https://stackblitz.com/edit/angular-3umd1g)

### 4. mapTo

>[Emits the given constant value on the output Observable every time the source Observable emits a value.](https://rxjs-dev.firebaseapp.com/api/operators/mapTo)

將Observable emit 出來的 value  轉為一個常數(constant)

```typescript
    of(1,2,3,4,5).pipe(
      mapTo(() => console.log('Hello World')),
      tap(val => val())
    ).subscribe()
```

[stackblitz](https://stackblitz.com/edit/angular-vjkutd)

練習題：https://stackblitz.com/edit/angular-spv3wd

### 5. filter

>[Filter items emitted by the source Observable by only emitting those that satisfy a specified predicate.](https://rxjs-dev.firebaseapp.com/api/operators/filter)

功能與陣列中的 filter相似，傳入一個 predicate function後，會將符合條件的 observable value過濾出來。

```typescript
    // 過濾大於3的數值，所以只會得到 4與 5
    of(1,2,3,4,5).pipe(
      filter( val => val >3),
      tap(val => console.log(val))
    ).subscribe()
```

[stackblitz](https://stackblitz.com/edit/angular-ffdqw9)

### 6. skip

> [Returns an Observable that skips the first `count` items emitted by the source Observable.](https://rxjs-dev.firebaseapp.com/api/operators/skip)

忽略 Observable送出的 資料中前 N筆資料。

```typescript
    // emit 3, 4, 5
    of(1,2,3,4,5).pipe(
      skip(2),
      tap(val => console.log(val))
    ).subscribe()
```
[stackblitz](https://stackblitz.com/edit/angular-qfmhj3)

### 7. first

> [Emits only the first value (or the first value that meets some condition) emitted by the source Observable.](https://rxjs-dev.firebaseapp.com/api/operators/first)

在Observable資料流中，只會emit第一筆資料，或是第一筆符合條件之資料。

```typescript
    of(1,2,3,4,5).pipe(
      first(),
      tap(val => console.log(val))
    ).subscribe();

    of(1,2,3,4,5).pipe(
      first( val => val >4),
      tap(val => console.log(val))    
    ).subscribe()
```

[stackblitz](https://stackblitz.com/edit/angular-bwptfc)

### 8. last

> [Returns an Observable that emits only the last item emitted by the source Observable. It optionally takes a predicate function as a parameter, in which case, rather than emitting the last item from the source Observable, the resulting Observable will emit the last item from the source Observable that satisfies the predicate.](https://rxjs-dev.firebaseapp.com/api/operators/last)

相較於 first取第一個， last則是取最後一筆，或是滿足條件之最後一筆。
```typescript
   // emit 5 
   of(1,2,3,4,5).pipe(
      last(),
      tap(val => console.log(val))
    ).subscribe();
    // emit 5
    of(1,2,3,4,5).pipe(
      last( val => val >4),
      tap(val => console.log(val))    
    ).subscribe();
```

[stackblitz](https://stackblitz.com/edit/angular-exnx48)

### 9. take

> [Emits only the first `count` values emitted by the source Observable.](https://rxjs-dev.firebaseapp.com/api/operators/take)

指定要抓取 N筆 Observable中 emit出來之資料，一旦滿足條件後變停止。

```typescript
		// 1, 2
    of(1,2,3,4,5).pipe(
      take(2),
      tap(val => console.log(val))
    ).subscribe();
```

[stackblitz](https://stackblitz.com/edit/angular-evtdy6)

### 10. takeLast

>[Emits only the last `count` values emitted by the source Observable.](https://rxjs-dev.firebaseapp.com/api/operators/takeLast)

指定要抓取 Observable資料中最後 N筆資料。

```typescript
    // 4,5
    of(1,2,3,4,5).pipe(
      takeLast(2),
      tap(val => console.log(val))
    ).subscribe();
```

[stackblitz](https://stackblitz.com/edit/angular-59i6ty)

### 11. takeWhile

> [Emits values emitted by the source Observable so long as each value satisfies the given `predicate`, and then completes as soon as this `predicate` is not satisfied.](https://rxjs-dev.firebaseapp.com/api/operators/takeWhile)

與 filter方法類似，會傳入一個 predicate function 來過濾資料，與 filter不同的是，當遇到第一個不滿足 predicate function時，便會進入 complete。

```typescript
    // emit 0 to 14
    range(0, 30).pipe(
      takeWhile( val => val < 15),
      tap(val => console.log(val))
    ).subscribe();

```

[stackblitz](https://stackblitz.com/edit/angular-zybwyw?file=src/app/hello.component.ts)

### 12. takeUntil

> [Emits the values emitted by the source Observable until a `notifier` Observable emits a value.](https://rxjs-dev.firebaseapp.com/api/operators/takeUntil)

讓 Observable持續 emit資料，等到另一個 Observable emit value後便進入 complete。

```typescript
    // after clicking on DOM, interval observable complete
    interval(5000).pipe(
      takeUntil( fromEvent(document, 'click')),
      tap(val => console.log(val))
    ).subscribe();
```

[stackblitz](https://stackblitz.com/edit/angular-asag2z)

如果要透過 takeUntil來取消訂閱 Observable，也需要注意 takeUntil擺放的位置，若沒放在適當的位置，可能還是會因為沒有正確的 unscribe而導致問題。[Avoiding takeUntil Leaks](https://blog.angularindepth.com/rxjs-avoiding-takeuntil-leaks-fb5182d047ef)，文章中提到必須將 takeUntil放在最後一個 operator，如果 takeUntil operator後還有其他訂閱其他 observable之operator，可能會沒辦法套用到 takeUntil之效果，如 switchMap。

下面的例子中， switchMap中的 Observable將不會在 takeUntil觸發後停止：

```typescript
    interval(1000).pipe(
      takeUntil( fromEvent(document, 'click').pipe(
        tap(val=> console.log('Click triggered'))
      )),
      tap(val => console.log(`interval val: ${val}`)),
      switchMap(val => interval(100)),
      tap(val => console.log(`switchMap val: ${val}:`))
    ).subscribe();
```

調整方式是將 takeUntil移到 operators最後一個：

```typescript
    interval(1000).pipe(
      tap(val => console.log(`interval val: ${val}`)),
      switchMap(val => interval(100)),
      tap(val => console.log(`switchMap val: ${val}:`)),
      takeUntil( fromEvent(document, 'click').pipe(
        tap(val=> console.log('Click triggered'))
      )),
    ).subscribe();
```

[stackblitz](https://stackblitz.com/edit/angular-wmc31a)

### 13. scan

> [Applies an accumulator function over the source Observable, and returns each intermediate result, with an optional seed value.](https://rxjs-dev.firebaseapp.com/api/operators/scan)

與reduce功能相同，差異在於Observable每做一次就會 emit 一次 value

```typescript
    fromEvent(document, 'click').pipe(
      mapTo(1),
      scan((acc, val)=> acc+val, 0),
      tap(val => console.log(`accumulated val ${val}`))
    ).subscribe()
```

[stackblitz](https://stackblitz.com/edit/angular-p8yrjg)

### 14. reduce

>[Applies an accumulator function over the source Observable, and returns the accumulated result when the source completes, given an optional seed value.](https://rxjs-dev.firebaseapp.com/api/operators/reduce)

同scan之功能，但只有在 reduce 完成後才會 emit value，也因為資料計算完後才會 emit 結果，所以必須確保 Observable事件是會結束的，否則 reduce 在未能確定前面 Observable資料結束前不會將壘加完資料送出。

```typescript
    fromEvent(document, 'click').pipe(
      take(5),
      mapTo(1),
      reduce((acc, val)=> acc+val, 0),
      tap(val => console.log(`accumulated val ${val}`)),
    ).subscribe()
```

[stackblitz](https://stackblitz.com/edit/angular-apyb5q)

### 15. delay

> [Delays the emission of items from the source Observable by a given timeout or until a given Date.](https://rxjs-dev.firebaseapp.com/api/operators/delay)

Observable 執行過程中，如果遇到 delay，則會依據 delay 上設置延遲 N ms emit value到下一個 operator。

**delay 參數也可以傳入一個日期參數。**

```typescript
    interval(500).pipe(
      take(5),
      tap(val => console.log(`first val: ${val}`)),
      delay(1000),
      tap(val => console.log(`second val: ${val}`)),
      delay(3000),
      tap(val => console.log(`third val: ${val}`)),
    ).subscribe()
```

[stackblitz](https://stackblitz.com/edit/angular-11h5hs)

### 16. delayWhen

>[Delays the emission of items from the source Observable by a given time span determined by the emissions of another Observable.](https://rxjs-dev.firebaseapp.com/api/operators/delayWhen)

功能與 delay相似，差異在於 delay多久後觸發由另一個 observable觸發。

```typescript
    const delayParams = [1000,6000,3000,5000,500];
    interval(500).pipe(
      take(5),
      tap(val => console.log(`first val: ${val}`)),
      delayWhen(val => timer(delayParams[val])),
      tap(val => console.log(`second val: ${val} delay ${delayParams[val]}`)),
    ).subscribe()
```

[stackblitz](https://stackblitz.com/edit/angular-uagx2d)

### 17. catchError

> [Catches errors on the observable to be handled by returning a new observable or throwing an error.](https://rxjs-dev.firebaseapp.com/api/operators/catchError)

在Observable執行出錯時，如果有使用catchError operator，則會被 catchError抓住，這時可以選擇讓整個 observabl3 retry，或是改傳回一個Observable。若沒有catchError，則會進入 observer的 error function中。

**catchError中 return的Observable會自動被訂閱接著執行**

catchError後改傳一個新的 observable：

```typescript
    interval(1000).pipe(
      map(val => {
        if(val === 4){
          throw '4 is not good';
        }
        return val * 2;
      }),
      catchError(err => of('A','B','C')),
      tap(val => console.log(`value: ${val}`)),
    ).subscribe()

```

catchError後 retry，主要是根據 catchError中傳入的第二個參數，第二個參數為原本執行的整個 observable：

第二個參數為原本執行的整個 observable

```typescript
    // 因為會一直 retry，故要加上take(20)
    interval(1000).pipe(
      map(val => {
        if(val === 4){
          throw '4 is not good';
        }
        return val * 2;
      }),
      catchError((err, caught) => caught),
      tap(val => console.log(`value: ${val}`)),
      take(20)
    ).subscribe()
```

[stackblitz](https://stackblitz.com/edit/angular-qdn7sj)

### 18. retry

> [Returns an Observable that mirrors the source Observable with the exception of an `error`. If the source Observable calls `error`, this method will resubscribe to the source Observable for a maximum of `count` resubscriptions (given as a number parameter) rather than propagating the `error` call.](https://rxjs-dev.firebaseapp.com/api/operators/retry)

在 Observable執行失敗時，會重新將整個 Observable 重新執行 N 次。

```typescript
    interval(1000).pipe(
      map(val => {
        if(val === 4){
          throw '4 is not good';
        }
        return val * 2;
      }),
      retry(2),
      tap(
        val => console.log(`value: ${val}`),
        err => console.log(`error: ${err}`)
      ),
    ).subscribe()
```

[stackblitz](https://stackblitz.com/edit/angular-cqdfdt)

### 19. retryWhen

> [Returns an Observable that mirrors the source Observable with the exception of an `error`. If the source Observable calls `error`, this method will emit the Throwable that caused the error to the Observable returned from `notifier`. If that Observable calls `complete` or `error` then this method will call `complete` or `error` on the child subscription. Otherwise this method will resubscribe to the source Observable.](https://rxjs-dev.firebaseapp.com/api/operators/retryWhen)

retry是指定次數， retryWhen則是指定滿足特定條件才retry，如另一個 Observable觸發時重新嘗試。

```typescript
    interval(500).pipe(
      map(val => {
        if(val === 4){
          throw '4 is not good';
        }
        return val * 2;
      }),
      retryWhen( (errors: Observable<any>) => {
        return errors.pipe(
          tap(val => console.log(`waiting for retry: ${val}`)),
          delayWhen(() => timer(2000)),
          take(5) // 實際上最多 retry 4次
        );
      }),
      tap(
        val => console.log(`value: ${val}`),
        err => console.log(`error: ${err}`),
        ()=> console.log('completed')
      ),
    ).subscribe()

```

[stackblitz](https://stackblitz.com/edit/angular-fxeetk)

### 20. startWith

> [Returns an Observable that emits the items you specify as arguments before it begins to emit items emitted by the source Observable.](https://rxjs-dev.firebaseapp.com/api/operators/startWith)

不管 Observable資料是什麼，都會強迫在原本的 observable emit 出來前，先 emit指定的 value出來。

```typescript
    // emit 11,12,13,0,1
    interval(500).pipe(
      startWith(11,12,13),
      take(5),
      tap(val => console.log(`val: ${val}`))
    ).subscribe()
```

[stackblitz](https://stackblitz.com/edit/angular-c6fh96)