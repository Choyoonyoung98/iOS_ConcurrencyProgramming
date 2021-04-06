## weak self와 관련

```
//상단에서 weak self를 선언하면 하단에도 동일하게 지정된다.
DispatchQueue.global().async { [weak self] in
  DispatchQueue.main.async {
  }
}
```

### ARC/클로저의 캡처 리스트
