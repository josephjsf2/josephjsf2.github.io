---


title: Rxjs - 建立Observable
layout: post
categories: angular
tags:

- angular
- Observable
- rxjs

---

查了一下 rxjs文件，發現rxjs 上除了自己平常使用的建立Obsservable方法外，還有提供其他沒用過的建立方式，這邊看的同時也做一個記錄。



<!--more-->

### 1. of

>[Converts the arguments to an observable sequence.](https://rxjs-dev.firebaseapp.com/api/index/function/of)

依照傳入的參數，逐一 emit value
```typescript
    const source$ = of(1, 2, 3, 4, 5);
    console.log('Trying to subscribe.');
    source$.subscribe(val => console.log(`Receive value ${val}`));
    console.log('Subscribed.');
```

[stackblitz](https://stackblitz.com/edit/angular-vtymgm)

### 2. from

>[Creates an Observable from an Array, an array-like object, a Promise, an iterable object, or an Observable-like object.](https://rxjs-dev.firebaseapp.com/api/index/function/from)

依照傳入的陣列，逐一 emit 陣列內容，從 rxjs6之後， from 可以傳入 promise，並移除了 fromPromise方法。

```typescript
    const source$ = from([1, 2, 3, 4, 5]);

    this.textLogSrv.addLogs('Trying to subscribe.');
    source$.subscribe(val => this.textLogSrv.addLogs(`Receive value ${val}`));
    this.textLogSrv.addLogs('Subscribed.');
```

[stackblitz](https://stackblitz.com/edit/angular-2uaeum?file=src%2Fapp%2Fdemo.component.ts)

除了上面用法，透過 from('string')方式使用，結果會逐一傳回 string中的每一個字元。

### 3. fromEvent

> Creates an Observable that emits events of a specific type coming from the given event target.

Javascript event以Observable方式處理，在Angular中除了以 ViewChild方式實做外，也可以傳入javascript DOM element。

Code:

```typescript
export class DemoComponent implements AfterViewInit {

  @ViewChild('myInput', {static: true}) myInput: ElementRef;

  constructor(private textLogSrv: TextLogService) { }

  ngAfterViewInit(): void {
    /**
     * From ViewChild
     */
    const click$ = fromEvent(this.myInput.nativeElement, 'click');
    const focus$ = fromEvent(this.myInput.nativeElement, 'focus');
    const blur$ = fromEvent(this.myInput.nativeElement, 'blur');
    
    click$.subscribe((val:MouseEvent) => this.textLogSrv.addLogs(`Click event ${val}`));
    focus$.subscribe((val:FocusEvent) => this.textLogSrv.addLogs(`Focus event ${val}`));
    blur$.subscribe((val: FocusEvent) => this.textLogSrv.addLogs(`Blur event ${val}`));
    
    /**
     * From native dom element
     */
    const clickFromDom$ = fromEvent(document.querySelector('input[type=text]'), 'click');
    clickFromDom$.subscribe((val:MouseEvent) => this.textLogSrv.addLogs(`From DOM click event ${val}`));
  }
}
```

Template:
```html
    <p>
      <input #myInput type="text"/>
      <text-log></text-log>
    </p>
```

[stackblitz](https://stackblitz.com/edit/angular-v5p3kt)

### 4. NEVER

> [An Observable that emits no items to the Observer and never completes.](https://rxjs-dev.firebaseapp.com/api/index/const/NEVER)

訂閱後就不會 complete的Observable，因為這特性，所以一定要手動將 observable做取消訂閱的動作，通常不會獨自使用，會搭配其他operators或是用來測試程式。

```typescript
NEVER
    .subscribe( 
      val => this.textLogSrv.addLogs('Value emit'),
      noop,
      ()=> this.textLogSrv.addLogs('Completed')
      );
```

[stackblitz](https://stackblitz.com/edit/angular-xxcndq)

### 5. EMPTY

> [The same Observable instance returned by any call to empty without a scheduler. It is preferrable to use this over empty().](https://rxjs-dev.firebaseapp.com/api/index/const/EMPTY)

訂閱後馬上結束的Observable。

```typescript
    EMPTY
    .subscribe( 
      val => this.textLogSrv.addLogs('Value emit'),
      noop,
      ()=> this.textLogSrv.addLogs('Completed')
      );
```

### 6. throwError

> [Creates an Observable that emits no items to the Observer and immediately emits an error notification.](https://rxjs-dev.firebaseapp.com/api/index/function/throwError)

建立一個新的Observable，並進入 error callback。

```typescript
of(1,2,3,4,5)
    .pipe(
      switchMap( val => {
        if(val === 3){
          // throwError creates new observable
          return throwError('I dont like number 2.');
        }
        return of(val);
      })
    )
    .subscribe( 
      val => this.textLogSrv.addLogs(`Value emit ${val}`),
      err => this.textLogSrv.addLogs(`Got an error: ${err}`),
      ()=> this.textLogSrv.addLogs('Completed')
      );
  }
```
[stackblitz](https://stackblitz.com/edit/angular-d93xet)

### 7. interval

> [Creates an Observable that emits sequential numbers every specified interval of time, on a specified [`SchedulerLike`](https://rxjs-dev.firebaseapp.com/api/index/function/interval).]

固定時間emit value之Observable，依序回傳 0, 1, 2, 3 …，因為不會自動停止，通常會搭配其他operator使用，或是手動取消訂閱。

```typescript
    interval(2000)
    .subscribe( 
      val => this.textLogSrv.addLogs(`Value emit ${val}`),
      err => this.textLogSrv.addLogs(`Got an error: ${err}`),
      ()=> this.textLogSrv.addLogs('Completed')
      );
```

[stackblitz](https://stackblitz.com/edit/angular-gx2r9y)

### 8. timer

> [Creates an Observable that starts emitting after an `dueTime` and emits ever increasing numbers after each `period` of time thereafter.](https://rxjs-dev.firebaseapp.com/api/index/function/timer)

與interval相似，但可以指定第一次在過多久之後才執行

```typescript
    // 過三秒後執行，之後每一秒送出一個value
    timer(3000, 1000)
    .subscribe( 
      val => this.textLogSrv.addLogs(`Timer1 value emit ${val}`),
      err => this.textLogSrv.addLogs(`Got an error: ${err}`),
      ()=> this.textLogSrv.addLogs('Timer1 completed')
      );
    // 一秒後送出 0後便 complete
    timer(1000)
    .subscribe( 
      val => this.textLogSrv.addLogs(`Timer2 value emit ${val}`),
      err => this.textLogSrv.addLogs(`Got an error: ${err}`),
      ()=> this.textLogSrv.addLogs('Timer2 completed')
      );

		// 可以指定 Date作為起始時間
    timer(new Date(new Date().getTime()+ 5000))
    .subscribe( 
      val => this.textLogSrv.addLogs(`Timer3 value emit ${val}`),
      err => this.textLogSrv.addLogs(`Got an error: ${err}`),
      ()=> this.textLogSrv.addLogs('Timer3 completed')
      );

		// 用法如同timer1，但改以Date作為起始時間
    timer(new Date(new Date().getTime()+ 10000),500)
    .subscribe( 
      val => this.textLogSrv.addLogs(`Timer4 value emit ${val}`),
      err => this.textLogSrv.addLogs(`Got an error: ${err}`),
      ()=> this.textLogSrv.addLogs('Timer4 completed')
      );
```

[stackblitz](https://stackblitz.com/edit/angular-o2qdws)

### 9.  range

> [Creates an Observable that emits a sequence of numbers within a specified range.](https://rxjs-dev.firebaseapp.com/api/index/function/range)

可以指定從某個數值開始，產生 n 個數值

```typescript
    // starts from 3 and generate 20 values
    range(3, 20)
    .subscribe( 
      val => this.textLogSrv.addLogs(`value emit ${val}`),
      err => this.textLogSrv.addLogs(`Got an error: ${err}`),
      ()=> this.textLogSrv.addLogs('completed')
      );
  	this.textLogSrv.addLogs('Subscribe completed');
```

[stackblitz](https://stackblitz.com/edit/angular-yfpngu)

### 10. generate

> 官方文件沒有文字說明...

類似 for loop

```typescript
    generate(0, x=> x< 10, x=> x+2)
    .subscribe( 
      val => this.textLogSrv.addLogs(`Timer1 value emit ${val}`),
      err => this.textLogSrv.addLogs(`Got an error: ${err}`),
      ()=> this.textLogSrv.addLogs('Timer1 completed')
      );
  	this.textLogSrv.addLogs('Subscribe completed');
```

 [stackblitz](https://stackblitz.com/edit/angular-qm3mat)

### 11. ajax

>[There is an ajax operator on the Rx object.](https://rxjs-dev.firebaseapp.com/api/ajax/ajax)

Rxjs也有提供 ajax請求，用的感覺有 $.ajax()的味道。

下面是RXJS官網[範例](https://rxjs-dev.firebaseapp.com/api/ajax/ajax)：

```typescript
    ajax({
      url: 'https://httpbin.org/delay/2',
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'rxjs-custom-header': 'Rxjs'
      },
      body: {
        rxjs: 'Hello World!'
      }
    }).pipe(
      map(response => this.textLogSrv.addLogs(JSON.stringify(response))),
      catchError(error => {
        this.textLogSrv.addLogs('error: ', error);
        return of(error);
      })
    ).subscribe(); 
```

[stackblitz](https://stackblitz.com/edit/angular-cnh4p6)

### 12. Observable.create

> [Creates a new cold Observable by calling the Observable constructor](https://rxjs-dev.firebaseapp.com/api/index/class/Observable)

自己手寫一個 Observable。

Observable.create中傳入之參數為一個 Observer，可以看Observer原始碼：

```typescript
export interface Observer<T> {
    closed?: boolean;
    next: (value: T) => void;
    error: (err: any) => void;
    complete: () => void;
}
```

Observer 就是當程式在訂閱 Observable時傳入的三個 callback function。

這邊可以看Observable.subscribe方法定義：

```typescript
    subscribe(observer?: PartialObserver<T>): Subscription;
    /** @deprecated Use an observer instead of a complete callback */
    subscribe(next: null | undefined, error: null | undefined, complete: () => void): Subscription;
    /** @deprecated Use an observer instead of an error callback */
    subscribe(next: null | undefined, error: (error: any) => void, complete?: () => void): Subscription;
    /** @deprecated Use an observer instead of a complete callback */
    subscribe(next: (value: T) => void, error: null | undefined, complete: () => void): Subscription;
    subscribe(next?: (value: T) => void, error?: (error: any) => void, complete?: () => void): Subscription;
```

所以當程式在做訂閱時，傳入的內容就是Observer需要的屬性：

```typescript
	// 參數依序會是 next、error與complete 
	of(1,2,3).subscribe(
      val => {},
      err => {},
      () => {}
    );
```

也可以用標準Observer方式傳入

```typescript
    const myObserver: Observer<any> = {
      next: val => console.log(val),
      error: err => console.log(err),
      complete: () => console.log('complete')
    }

    of(1,2,3).subscribe(myObserver);
```

[stackblitz](https://stackblitz.com/edit/angular-uely4j)

在知道這點之後，就可以開始說明 observable.create方法，直接用程式碼搭配說明：

```typescript
    // 建立Observable
    const source$ = Observable.create( (observer: Observer<number>) => {
      // 發送第一個next
      observer.next(100);
      
      // 之後每一秒發送一個value，執行10次後結束
      interval(1000).pipe(
        take(10)
      ).subscribe(
        val => observer.next(val *2),
        noop,
        () => observer.complete()
      )
    });

    source$.subscribe( 
      val => this.textLogSrv.addLogs(`value emit ${val}`),
      err => this.textLogSrv.addLogs(`Got an error: ${err}`),
      ()=> this.textLogSrv.addLogs('completed')
      );
  	this.textLogSrv.addLogs('Subscribe completed');

```

* 首先透過Observable.create建立一個Observable
* 透過Observer.next 送出第一個內容，訂閱者會在這時候收到第一個value
* 接著Observable內開始處理其他非同步事件，可以是抓http請求，或是執行任何事情，這邊以interval作為範例
* interval依序送出 0 1 2…9，在收到interval內容時，對收到內容作處理，讓訂閱者收到不同內容
* 在interval complete後，使用Observable.complete告訴訂閱者事件結束，訂閱者變結束訂閱。

而上面source$.subscribe中，傳入之function：
```typescript
 val =>this.textLogSrv.addLogs(`value emit ${val}`)
```

即為Observable.create( observer) 裡面 observer.next function，所以當程式執行到 observer.next(100)時，實際上就是把 100 傳入上面 function中執行， error與 complete也是同樣的原理。

如果Observable執行完，建議要調用 complete方法告訴訂閱者結束，否則會跟NEVER一樣永遠不會停止，如果忘記取消訂閱，可能會造成其他問題。

[stackblitz](https://stackblitz.com/edit/angular-ic1knd)

### 13. defer

>[Creates an Observable that, on subscribe, calls an Observable factory to make an Observable for each new Observer.](https://rxjs-dev.firebaseapp.com/api/index/function/defer)

不管以什麼方式產生Observable stream，都只會在開始訂閱的當下才開始執行。defer 比較常見是用在 promise，用過promise都知道，promise在宣告的當下就會開始執行，不像Observable需要等訂閱時才執行，所以下面的程式碼使用 from 將 promise轉為 observable，等 10秒後才訂閱，這時會發現在observable被訂閱前，promise已經將請求發出去。

```typescript
   const req$ = from(fetch('https://httpbin.org/delay/2',{
      cache: 'no-cache',
      headers: {
        'user-agent': 'Mozilla/4.0 MDN Example',
        'content-type': 'application/json'
      },
      method: 'POST', 
      mode: 'cors',
    }));

    setTimeout(()=>{
      this.textLogSrv.addLogs('req$ start subscribing')
      req$
      .subscribe(
        val=> this.textLogSrv.addLogs(`req$ value: ${val.url}`),
        err=> this.textLogSrv.addLogs(`req$ return error ${err}`),
        ()=> this.textLogSrv.addLogs(`req$ complete`));
      this.textLogSrv.addLogs('req$ subscribed')
    }, 10000);
```

要避免這情況，就會需要以defer方式建立 Observable，這方式可以讓原本立刻執行的 promise，等到Observable被訂閱的當下才執行

```typescript
    // send request when subscribeing to observable
    const deferReq$ = defer(()=>fetch('https://httpbin.org/delay/2',{
      cache: 'no-cache',
      headers: {
        'user-agent': 'Mozilla/4.0 MDN Example',
        'content-type': 'application/json'
      },
      method: 'POST', 
      mode: 'cors',
    }));

    setTimeout(()=>{
      this.textLogSrv.addLogs('deferReq$ start subscribing')
      deferReq$
      .subscribe(
        val=> this.textLogSrv.addLogs(`deferReq$ value: ${val.url}`),
        err=> this.textLogSrv.addLogs(`deferReq$ return error ${err}`),
        ()=> this.textLogSrv.addLogs(`deferReq$ complete`));
      this.textLogSrv.addLogs('deferReq$ subscribed')
    }, 15000);
```

兩段程式碼請求方式相同，只差在於第二段程式使用了 defer方法來延遲執行。

[stackblitz](https://stackblitz.com/edit/angular-5buwmp)

