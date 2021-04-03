## 2. 디스패치큐(GCD)의 종류와 특성
Queue의 종류(대기열의 종류)
1)dispatchQueue
1-1) Global(Main) DispatchQueue - 유일한 한 개의 큐, Serial, Main Thread
1-2) Global DispatchQueue
1-3) Private(Custom) DispatchQueue
2) OperationQueue

### 2-1. MainQueue
```DispatchQueue.main.async{ }```  
- DispatchQueue의 한 종류
- 유일한 하나의 큐
- Serial Queue
- Main Thread이자 MainQueue

ex)
```DispatchQueue.main.asyncAfter(.now() + 2) { } ```
-> 지금으로부터 2초 뒤에 메인 스레드로 작업을 비동기적으로 보낼거야!

#### mainQueue.async 
```
//1
task1()

//2
mainQueue.sync {
  task1()
}
```
위의 두 동작은 의미상 같지만, 아래 코드는 에러가 발생된다!

### 2-2. GlobalQueue
- 기본설정이 Concurrent로 되어 있다(분산하여 일을 처리하는)

#### (중요한 순서로 qos 종류)  
1) **userInteractive** : 거의 즉시
> 유저와 직접적 인터렉티브: UI업데이트, 애니메이션, UI반응 관련 어떤 것이든(사용자와의 상호작용)
2) **userIniated** : 몇초
> 유저가 즉시 필요하긴 하지만, 비동기적으로 처리된 작업(ex 앱내에서 pdf파일을 여는 것과 같은)
3) **default**  
> 일반적인 작업
4) **utility** : 몇초에서 몇분
> 보통 프로그레스바 indicator와 함께 길게 실행되는 작업, 계산, IO
> (ex IO, Networking, 지속적인 데이터 feeds)
5) **background** : 속도보다는 에너지 효율성을 중시, 몇분 이상
> 유저가 직접적으로 인지하지 않고(시간이 안 중요한) 작업
> ex. 데이터 미리가져오기, 데이터베이스 유지보수, 원격 서버 동기화 및 백업 수행 
6) **unspecified** : -
> legacy API 지원, 스레드를 서비스 품질에서 제외시키는(거의 사용하지 않음) 
- 근데 OS가 알아서 qos를 추론한다고 한다
- iOS가 알아서 우선적으로 중요한 일임을 인지하고 쓰레드에 우선순위를 매겨 **우선순위가 더 높을수록!** 더 여러개의 쓰레드를 배치하고 배터리를 더 집중해서 사용하도록 함
  
#### (만약에?) qos가 중첩되는 경우 작업을 보낼 때 더 높은 수준으로 보낼 경우?
```
let queue = DispatchQueue.global(qos: .background) //백그라운드로 정의
queue.async(qos: .utility) { } //보낼 때는 더 높은 수준으로
```
-> queue가 작업에 영향을 받아서 background가 utility로 품질이 올라간다!

### 2-3. Private(Custom) Queue
- 커스텀으로 만든다
- 디폴트설정은 Serial Queue이나, 원한다면 Concurrent Queue로 설정할 수 있다.
- 레이블을 붙일 수 있다!
```
let queue = DispatchQueue(label: "com.infleran.serial")
let quyeue2 = DispatchQueue(label: "com.inflearn.concurrent", attributes: .concurrent)
```

```
let privateQueue = DispatchQueue(label: "com.inflearn.serial")

privateQueue.async{ task1() }
privateQueue.async{ task2() }
privateQueue.async{ task3() }
```
-> 비동기적으로 실행하지만, serialQueue이기 때문에 하나의 Thread 안에서 실행되므로 task1()의 실행이 완료되어야 다음 task를 수행할 수 있다.

### (정리) 큐의 종류
1) DispatchQueue(GCD)  

| **큐의 종류** | **생성 코드** | **특징** | **직렬/동시** |
|------|---|---|---|
|.main|`DispatchQueue.main`|메인큐 = 메인쓰레드(1번 쓰레드): UI업데이트 내용을 처리하는 큐| 직렬(Serial) |
|.global()|`DispatchQueue.global(qos:)`|6가지 qos| 동시(Concurrent) |
|custom(프라이빗)|`DispatchQueue(label: "..."`|qos 추론 / qos 설정 가능| 디폴트: Serial( attributes로 설정 가능) |

* 메인이 직렬큐인 이유? 쓰레드가 하나밖에 없기 때문에 직렬로 동작할 수 밖에 없다!

2) OperationQueue(내부적으로 GCD 기반으로 구현)

|**생성 코드** | **특징** | **직렬/동시** |
|------|---|---|
|`let opQ = OperationQueue()`|디폴트: background, 기반(underlying) 디스패치큐에 영향 받음 | 디폴트: Concurrent(maxConccurnetOperationCount로 사용할 쓰레드 갯수 설정 가능) |




