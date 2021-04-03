# iOS_ConcurrencyProgramming
iOS 동시성 프로그래밍에 대한 이해
- 참고 자료: [인프런 강의] [iOS Concurrency(동시성) 프로그래밍, 동기 비동기 처리 그리고 GCD/Operation - 디스패치큐와 오퍼레이션큐의 이해](https://www.inflearn.com/course/iOS-Concurrency-GCD-Operation)
- 정리 목적: 공식 문서와 해당 강의를 바탕으로 GCD 사용성 정의를 내려보자!

## 목차 1: GCD
1) [GCD/Operation에 앞서](./GCD1.md)
3) 디스패치큐(GCD)의 종류와 특성
4) 디스패치큐(GCD) 사용시 주의해야할 사항
5) 디스패치 그룹
6) GCD 프로젝트 살펴보기
7) (심화) 동시성과 관련된 문제
8) (심화) Tread-safe한 코드의 구현과 방법
9) (심화) Lazy var과 관련된 이슈들

## 목차 2: Operation
1) Operation에 앞서
2) Operation / OperationQueue
3) AsyncOperation
4) 오퍼레이션큐의 주요 기능
5) 오퍼레이션 프로젝트 살펴보기
6) 총정리: GCD VS Operation
