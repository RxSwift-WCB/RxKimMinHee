# Chapter 2: Observable

## 🦊 observable 이란?
- observable = observable sequence = sequence
- 비동기적!
- emitting : Observable이 일정 기간 동안 이벤트를 생성하는 것
- 각각의 이벤트들은 값을 가질 수 있으며, 또는 탭과 같은 제스처를 인식할 수도 있음


## 🐻 생명주기

<img width="333" alt="스크린샷 2020-10-17 오전 5 18 21" src="https://user-images.githubusercontent.com/51286963/96305064-3480c780-1038-11eb-91d5-1cb66267940f.png">

: 3개의 tap을 내보낸 뒤(next) 정상적으로 종료된다. -> completed 이벤트!

<img width="329" alt="스크린샷 2020-10-17 오전 5 20 00" src="https://user-images.githubusercontent.com/51286963/96305166-6abe4700-1038-11eb-93c8-c39db080dfef.png">

: 오류가 발생했다! 완전 종료가 된 것은 맞지만 위와 다르게 error 이벤트를 통해 종료됐다는 점이 차이점이다.

- 결론
1) observable은 next를 통해 이벤트를 방출한다.
2) completed 이벤트를 통해 완전 종료가 가능하다.
3) error 이벤트를 통해 완전 종료가 가능하다.
 
``` swift
 public enum Event<Element> {
 	/// Next이벤트는 어떤 인스턴스를 가지고 있음
 	case next(Element)
 	
 	/// .error 이벤트는 Swift.Error 인스턴스를 가짐
 	case error(Swift.Error)
 	
 	/// 하지만 completed 이벤트는 아무런 인스턴스를 가지지 않고 단순히 이벤트를 종료함
 	case completed
 }
```
## 🐼 observable 만들어보기 
- just : 오직 하나의 Element를 포함하는 Observable Sequence를 생성
``` swift
    let observable: Observable<Int> = Observable<Int>.just(1)
    let observable: Observable<String> = Observable<String>.just("1")
```
- of : 가변적인 element를 포함하는 Observable Sequence를 생성
``` swift
    let observable: Observable<Int> = Observable<Int>.of(1,2,3,4,5)
    let observable: Observable<[Int]> = Observable<[Int]>.of([1,2,3])
```
- from : array의 요소들로 Observable Sequence 생성, 타입은 Observable < Int >
``` swift
    let observable: Observable<Int> = Observable<Int>.from([1,2,3,4,5])
```
- empty : 요소를 가지지 않는 Observable, empty연산자를 통해 completed 이벤트만 방출
``` swift
    let observable = Observable<Void>.empty()
```
- never : empty와 반대되는 연산자로 never는 이벤트를 방출 조차 하지 않음
``` swift
    let observable = Observable<Any>.never()
```
- range : start 부터 count크기 만큼의 값을 갖는 Observable을 생성(시퀀스를 갖는다는 말이 더 확실할듯..?)
``` swift
    let observable = Observable<Int>.range(start: 3, count: 6)
```
- repeatElement : 지정된 element를 계속 Emit
``` swift
    let observable = Observable<Int>.repeatElement(3)
```
- interval : 지정된 시간에 한번씩 event를 emit
``` swift
    Observable<Int>.interval(3, scheduler: MainScheduler.instance)
```
- creat : Observer에 직접 Event를 방출
``` swift
    Observable<Int>.create({ (observer) -> Disposable in
        
    observer.onNext(3)
    observer.onNext(4)
    observer.onNext(5)
        
    observer.onCompleted()
        
    return Disposables.create()
    })
```

## 🐨 observable factory 만들기
``` swift
example(of: "deferred") {
     let disposeBag = DisposeBag()
     
    // 관찰 가능한 Bool 타입의 플래그를 생성한다
     var flip = false
     
     // deferred 연산자를 이용해서 Int factory Observable을 생성한다
     let factory: Observable<Int> = Observable.deferred{
         
         // factory가 구독할 flip의 Bool타입을 전환함 (false > true)
         flip = !flip
         
         // 그래서 flip의 true, false에 따라 값이 달라짐
         if flip {
             return Observable.of(1,2,3)
         } else {
             return Observable.of(4,5,6)
         }
     }
     
     for _ in 0...3 {
         factory.subscribe(onNext: {
             print($0, terminator: "")
         })
             .disposed(by: disposeBag)
         
         print()
     }
 }
 /* Prints:
 123
 456
 123
 456
 */
```
## 🐯 Traits 사용하기
: Observable보다 좁은 범위의 Observable. trait는 옵션으로 사용하면서 코드의 가독성을 높일 수 있다.
- single<br>
: .success(.next + .completed) 또는 .error 이벤트를 방출한다.
- Completable<br>
: .completed 또는 .error 이벤트만 방출함. 다른 값은 방출안함
- Maybe<br>
: 위에 나온 single과 Completable를 합쳐놓은 것(success(value), .completed, .error 이벤트 모두 방출 가능)