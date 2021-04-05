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

### 3. weak, strong 캡처 주의

### 4. completionHandler 존재 이유

### 5. 동기적함수를 비동기함수처럼 만드는 방법

