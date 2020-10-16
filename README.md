# Chapter 1: Hello RxSwift!

## What is RxSwift ?

    RxSwift is a library for composing asynchronous and event-based code by using observable sequences and functional style operators, allowing for parameterized execution via schedulers. 

    : RxSwift는 Observable들의 순서와 함수적 스타일 연산자를 사용한 비동기 및 이벤트 기반 코드 작성을 위한 라이브러리로써 스케줄러를 통한 매개 변수화된 실행을 하는 라이브러리입니다.
<br>

    RxSwift, in its essence, simplifies developing asynchronous programs by allowing your code to react to new data and process it in a sequential, isolated manner.

    : RxSwift는 너의 코드가 새로운 데이터에 반응하고 순차적으로 격리된 방식으로 진행되므로써 본질적으로 비동기 프로그램 개발을 단순화 시킵니다. 


### 중요 KeyWord 
- *asynchronous* : 비동기적, 비동기식은 동시에 일어나지 않는다는 뜻이다. 요청과 결과가 동시에 일어나지 않는다는 말(요청한 그 자리에서 결과가 주어지지 않음)
- iOS는 다양한 스레드에서 다양한 작업을 수행하고 기기 CPU의 다양한 코어에서 작업을 수행 할 수 있다
- ex) 우리가 노래를 들을 때 텍스트를 친다고 해서 노래가 끊기진 않는다.

----------------------------

## 🐶 Cocoa and UIKit Asynchronous APIs
1) NotificationCenter : 유저가 기계의 방향을 바꿀 때, 혹은 소프트웨어 키보드가 화면에서 보여지거나 숨겨질 때와 같은 이벤트가 실행될 때 마다 코드를 실행한다. => 어떤 작업이 완료되거나 특정 이벤트가 실행될 경우 이를 다른 객체들에게 알려줄 수 있다

2) The delegate pattern : 임의의 시간에 다른 클래스나 api에서 실행될 메소드를 정의한다. 예를들어 너의 어플리케이션 딜리게이트 안에서 너는 너의 새로운 원격 알림이 도착했을 때 무슨일이 일어날지 정의할 수 있다. 하지만 이 코드 조각들이 언제 실행될지 혹은 얼마나 많이 실행될 지는 알 수 없다. 
 => MVC에서 View에서 데이터를 다루는 부분과는 독립적이어야 하기 때문에 이를 위임하는것임.


3) Grand Central Dispatch : 너의 실행 작업을 추상화 하는 것을 도와준다. 너는 순차적으로 연속적인 큐에서 코드가 실행되게 혹은 아주 많은 task를 동시에 우선순위가 다른 여러개의 큐에서 실행되게 예약할 수 있다.
    - 앱을 실행하면 시스템이 자동으로 메인스레드 위에서 동작하는 Main 큐(Serial Queue)를 만들어서 작업을 수행하고, 그 외에 추가적으로 여러 개의 Global 큐(Cuncurrent Queue)를 만들어서 큐를 관리합니다.
    - Main 큐에서는 비동기만 사용 가능
    - Serial Queue : 큐에 추가된 순서대로 한번에 하나의 task를 실행
    - Concurrent Queue : 동시에 하나 이상의 task를 실행하지만 큐에 추가됬을시에 추가된 순서대로 작업을 계속 진행

4) Closures : 일정 기능을 하는 코드를 하나의 블럭으로 모아놓은 것을 의미.
    - 동기적 코드
    : 동기적으로 실행되고, 컬렉션이 실행되는 동안 변경할 수 없음. 그렇기때문에 모든 요소가 있는지 확인하지 않아도됨.
    ``` swift
    var array = [1, 2, 3]
    for number in array {
        print(number)
        array = [4, 5, 6]
    }
    print(array)
    ```
    <img width="141" alt="스크린샷 2020-10-16 오후 9 58 59" src="https://user-images.githubusercontent.com/51286963/96261081-e4cfdb00-0ffa-11eb-91ce-666fd6a38414.png">

    - 비동기적 코드
    ```swift
        var array = [1, 2, 3]
        var currentIndex = 0
        //this method is connected in IB to a button

        @IBAction func printNext(_ sender: Any) {
            print(array[currentIndex])
            if currentIndex != array.count-1 {
                currentIndex += 1
            }
        }
    ```
    <img width="87" alt="스크린샷 2020-10-16 오후 9 59 04" src="https://user-images.githubusercontent.com/51286963/96261087-e8636200-0ffa-11eb-8e3c-2bc57ab5fefb.png">

## 🐱 비동기적 프로그래밍 용어 알아보기
1. State, and specifically, shared mutable state
    - state 란?<br>
    : 우리가 컴퓨터를 새로 사면 처음에는 너무 잘 작동하지만 나중되면 느려지거나 작동이 이상해짐. 왤까? 왜냐면 소프트웨어와 하드웨어는 그대로지만 state가 변하기때문! -> 사용하면 사용할 수록 데이터의 교환 등등이 이루어지고 또 남는 것들이 생기면서 상태가 변화한다는 뜻.

2. Imperative programming(명령형 프로그래밍)<br>
    : 명령문을 사용해 프로그램의 상태를 변화시키는 것. 명령형 코드를 사용하여 작업을 수행하는시기와 방법을 앱에 정확히 알려줍니다. 하지만 문제는 사람이 복잡한 비동기 앱을 위한 명령형 코드를 작성하는것은 너무 어렵다..
    ``` swift
    override func viewDidAppear(_ animated: Bool) { 
        super.viewDidAppear(animated)
        setupUI()
        connectUIControls()
        createDataSource()
        listenForChanges()
        }
    ```
    : 이걸 보고 각자의 함수들이 무슨 일을 하는지 알 수 있나요? 순서대로 실행된다는 보장이 있나요? 만약 누가 method의 순서를 바꿨을 때 문제가 생길 수도 있음!

3. 다른 부수적인 작용들<br>
    : 스크린 상 label의 text를 추가하거나 변경한다는 것 > 디스크에 저장된 데이터를 수정한다는 것 > 부수작용 발생한다는 것. 각각의 코드에 대해서, 해당 코드가 어떤 부수작용을 발생시킬 수 있는 코드인지, 단순 과정을 나열한 것인지, 명확한 결과값만을 발생시키는 것인지 정확히 인지하고 있는 것이 중요하다.

4. 선언형 코드<br>
    : 상태를 마음대로 변경할 수 있음. 명령형 프로그래밍 + 함수형 프로그래밍 = 자유로운 상태변화 + 추적/예측가능한 결과값 = 변경 불가능한 데이터로 작업하고, 순차적이고 결과론적인 방식으로 코드를 실행 가능

5. Reactive systems
    - Responsive(반응) : 항상 UI를 최신 상태로 유지하며, 가장 최근의 앱 상태를 표시함
    - Resilient(복원력) : 각각의 행동들은 독립적으로 정의되며, 에러 복구를 위해 유연하게 제공됨
    - Elastic(탄성력) : 코드는 다양한 작업 부하를 처리하며, 종종 lazy full 기반 데이터 수집, 이벤트 제한 및 리소스 공유와 같은 기능을 구현
    - Message driven(메세지 전달) : 구성요소는 메시지 기반 통신을 사용하여 재사용 및 고유한 기능을 개선하고, 라이프 사이클과 클래스 구현을 분리함

----------------------------

## 🐭 RxSwift 기초 - Rx code의 세가지 요소
1. Observables 
: Observable 클래스는 Rx 코드의 기반으로 T형태의 데이터를 전달 할 수 있는 이벤트들을 비동기적으로 생성하는 기능.
    1) A next event : 최신(다음) 데이터값을 전달 하는 이벤트
    2) A completed event : 성공적으로 이벤트를 종료하는 이벤트, 생명주기를 다 했으므로 다른 이벤트 생성 X
    3) An error event : Observable이 오류로 종료되었고 다른 추가적인 이벤트는 생성하지 X(에러와 함께 완전히 종료)
    <br>
    - Finite observable sequences <br>
    : 몇몇의 observable한 이벤트들은 0 혹은 1개 혹은 그 이상의 값을 내보낸 후 성공적으로 종료되거나 오류와 함께 종료된다. 
        - 일반적인 observable한 생명주기 <br>
        : 다운로드 시작 -> 들어오는 데이터 관찰 -> 파일이 계속 들어옴 -> if 연결이 끊어진다면? 다운로드 중지 + 오류 + 연결 시간 초과 -> else 성공적 완료
        ``` swift 
        API.download(file: "http://www...") 
        // 네트워크를 통해 들어오는 Data값을 방출하는 Observable<Data> 인스턴스를 리턴.
            .subscribe(onNext: { data in
            // onNext 클로저를 이용해 다음 이벤트를 받을 수 있음
                ... append data to temporary file 
            },
            onError: { error in
            // onError 클로저를 이용하면 오류 이벤트를 처리할 수 있음.
                ... display error to user 
            },
            onCompleted: {
            // onCompleted는 완료된 이벤트를 처리하기 위한 클로저
                ... use downloaded file 
            })
        ```

    - Infinite observable sequences <br>
    : 보통 UI이벤트는 무한하게 관찰 가능한 이벤트이다.
        - ex) 기기의 방향에 따라 반응해야하는 코드가 있다면
            1) NotificationCenter를 통해 UIDeviceOrientationDidChange의 반응을 알려주는 관찰자를 class에 선언한다
            2) 방향 전환을 관리하는 callback method 제공해야함, UIDevice의 현재 방향을 확인한 후에 이 값에 따라 화면에 어떻게 표시할 지 구현해야한다. 

            => 화면이 한번 돌려진다고 해서 이벤트가 끝난게 아님! 언제든 사용자든 화면을 돌릴 수 있음. 그렇기 때문에 이것은 무한한 시퀀스라고 할 수 있지요~
        ``` swift
        UIDevice.rx.orientation 
        // Observable<Orientation>을 통해 만든 가상의 코드로써 방향을 확인받을 수 있고 받은 값은 UI에 업데이트 가능

            .subscribe(onNext: { current in
                switch current { 
                    case .landscape:
                        ... re-arrange UI for landscape 
                    case .portrait:
                        ... re-arrange UI for portrait 
                    }
                })
        ```
        - 해당 이벤트에서는 onError나 onCompleted는 절대 발생하지 않는 이벤트!

2. Operators
: ObservableType과 Observable 클래스에 복잡한 논리를 구현하기 위해 있는 메서드들. 비동기적 입력을 받고 출력만 존재하기 때문에 혼합해 사용하기 편하다.

    ``` swift
    UIDevice.rx.orientation 
        .filter { value in
            return value != .landscape 
            //landsape이 아닌 값만 리턴(그니깐 portrait 값일때만 리턴한다는 것), landsape이 들어오면 코드 진행 X
        }
        .map { _ in
        // portrait 일 땐 return 값 존재~
            return "Portrait is the best!"
        }
        .subscribe(onNext: { string in
        // onNext를 이용해 다음 이벤트를 받음
        // 여기서는 특정 string을 보여주는 alert을 띄어주는 이벤트를 실행하네요~
            showAlert(text: string)
        })
    ```
3. Schedulers
: Schedulers는 Rx에서 dispatch queue와 동일하다. RxSwift에는 여러가지의 스케줄러가 이미 정의되어 있어서 갖다 쓰면 된다. -> 스케쥴러는 후반부에 진행~

--------------------

## 🐹 App architecture 

: RxSwift는 어떤식으로든 앱의 아키텍쳐를 변경하지 않음. 그래서 MVC에 RxSwift를 도입할 수 있다. 물론 MVVC에서도 멋지게 작동 가능!

<img width="471" alt="스크린샷 2020-10-17 오전 4 06 24" src="https://user-images.githubusercontent.com/51286963/96298939-3c3b6e80-102e-11eb-9dbc-effb623fa3de.png">
- MVC에서의 아키텍쳐 방식~

## 🐰 RxCocoa
: RxSwift의 동반 라이브러리, UIKit과 Cocoa 프레임워크 기반 개발을 지원하는 모든 클래스를 보유하고 있다. 
``` swift
toggleSwitch.rx.isOn
//UISwitch 에 isOn이라는 속성 추가 가능~
 	.subscribe(onNext: { enabled in
 		print( enabled ? "It's ON" : "it's OFF")
 	})
```