## GCD 사용 시 주의해야할 사항

### 1. 반드시 메인큐에서 처리해야 하는 작업
1) UI 관련일들은 **메인큐** 에서 처리해야 한다
- 한 개의 스레드에서만 처리해야 (화면간의 간섭, 화면의 깜빡거림) 등을 방지할 수 있다.  
```
DispatchQueue.global().async {
  DispatchQueue.main.async {
    //다운로드한 이미지 표시코드: UI관련일
    self.imageView.image = image
  }
}
```  
-> 전체코드는 오래 걸리는 작업이니 비동기로 분산해서 작업을 처리하고 싶다!  
-> 하지만 UI관련일은 메인큐(DispatchQueue.main)로 보내서 작업한다.

```
URLSession.shared.dataTask(with: url) { (data, response, error) in
  if let error = error { 
    print("Failed to load image with error", error.localizedDescription)
  }
  
  if self.lastImgUrlUsedToLoadImage != url.absoluteString { return }
  
  guard let imageData = data else { return }
  let photoImage = UIImage(data: imageData)
  imageCache[url.absoluteString] = photoImage
  
  DispatchQueue.main.async {
    self.image = photoimage
  }
}.resume()

```

### 2. sync 메서드에 대한 주의사항
-> sync메서드와 관련해 절대 해서는 안되는 코드 2가지

#### 1) 메인큐에서는 다른큐로 보낼 떄 sync메서드를 부르면 절대 안된다: "메인큐에서는 항상 비동기적으로 보내야한다"  
즉, UI와 관련되지 않은 오래걸리는 작업(네트워크)들은 다른 쓰레드에서 일을 할 수 있도록 **비동기적** 으로 실행하여야 하며, 동기적으로 시키면 UI가 멈춰서 유저한테 반응을 늦게하고 버벅거린다.

#### 2) 현재의 큐에서 현재의 큐로 "동기적으로" 보내서는 안된다
현재의 큐를 블락하는 동시에 다시 현재의 큐에 접근하기 때문에 **교착상황(데드락)** 이 발생한다.  
```
//교착상황이 발생하는 코드
DispatchQueue.global().async {
  DispatchQueue.global().sync {
  }
}
//그런데 원래 사용하던 스레드가 아닌 다른 스레드로 배치되면 교착상황이 발생되지 않는다
//하지만 교착상태가 발생할 수 있는 가능성을 가지고 있기 때문에 사용해서는 안된다!
```

### 3. weak, strong 캡처 주의(⭐️)
```
//작업을 보내는 행위는 클로저를 보내는 행위! 그렇기 때문에 당연히 객체에 대한 캡처가 발생한다
DispatchQueue.global(qos: .utility).async { [weak self] in //weark: +0, string: +1
  guard let self = self else { return }
  
  DispatchQeue.main.async {
    self.textLabel.text = "New posts updated!"
  }
}  
```  
```
let viewcon = Viewcontroller() //ARC -> +1
```  
- weak을 사용할 경우: ViewController가 dismiss되더라도, 큐로 보낸 클로저(작업)도 중단된다
- strong을 사용할 경우: ViewController가 dismiss되더라도, 큐로 보낸 클로저(작업) 여전히 동작한다.


### 4. (비동기 작업에서)completionHandler 존재 이유
-> 비동기적으로 보낸 작업이 끝나는 시점을 알 필요성이 있다. 예를 들어 다운로드 작업을 진행하고, 다운로드한 결과를 사용할 수도 있다.  
-> 결국 completionHandler는 비동기작업이 명확하게 끝나는 시점을 활용할 수 있게 해준다.

- 비동기로 작업을 시키고 나서, 작업에 해당하는 값을 바로 사용하면 안된다.
- 작업이 아직 종료하지 않았는데, 해당 값에 접근하면 잘못된 값을 사용할 확률이 높다.
- 그래서, 해당 **비동기 작업이 끝났다는 것을 정확히 알려주는 시점** 이 컴플리션핸들러(completionHandler)이다.

### 5. 동기적함수를 비동기함수처럼 만드는 방법
왜하냐? 여러번 재활용 위해

1) 직접적으로 작업을 실행할 큐: runQueue
2) 작업을 마치고나서의 큐: completionQueue
3) completionHandelr 필요
4) 에러 처리에 대한 내용

```
func asyncTiltShift(_ inputImage: UIImage?, runQueue, DispatchQueue, completionQueue: DispatchQueue, completion: @escaping (UIImage?, Error?) -> () ) {
  runQueue.async{
    var error: Error?
    error = .none
    
    let outputImage = tiltShift(image: inputImage)
    completionQueue.async {
      completion(outputImage, error)
    }
  }
}

asyncTilShift(image, runQueue: workingQueue, completionQueue: resultQueue) { image, error in image
  print("비동기 작업의 실제종료시점")
}
```

+) URLSession
