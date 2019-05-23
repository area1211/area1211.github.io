
---
title: "[nodejs] Node.js란 무엇인가!?"
date: 2019-05-21 15:05:28 -0400
categories: jekyll update
---
nodejs 의 핵심 철학은 비동기로 동작하며, I/O 작업일 경우 워커 스레드에게 던져두고 이벤트 루프는 다른 일을 한다는 것이다.
이 때 I/O 작업 처리는 libuv가 담당하며, 기본적으로 4개의 스레드가 있다.
하지만, 단순히 libuv 에서만 처리하는게 아니라 이미 OS 단에서 많은 I/O 작업을 지원해 주고 있다.(linux의 AIO) 
따라서 꼭 필요할 때만 libuv의 스레드풀을 사용한다.
이 스레드의 개수는 일반적으로 코어의 개수와 동일할 때가 최적이다.
[출처](https://sjh836.tistory.com/99)

nodejs는 기본적으로 libuv 위에서 동작하며, node 인스턴스가 뜰 때, libuv에는 워커 스레드풀(기본 4개의 스레드)이 생성된다.
    libuv란? 비동기 입출력, 이벤트 기반에 초점을 맞춘 라이브러리이다. I/O 처리에 대해서 요청마다 스레드를 사용하지만 비동기, 논블로킹 스타일을 사용한다. 이를 위해 각 커널의 비동기 I/O를 이용한다.


[출처](https://www.simplilearn.com/node-js-interview-questions-and-answers-article)
1. Node.js는 무엇인가? 어디에 사용할 수 있는가?
노드는 V8 자바스크립트 엔진을 기반으로 작동하는 서버 사이드 scripting 이다.
scalable 프로그램을 만드는데 사용된다. 특히 간단한 계산을 하지만 빈번하게 접근되는 웹애플리케이션에서 사용된다.
비디오 스트리밍 사이트와 같은 I/O intensive 웹 애플리케이션을 개발할 수 있다.
또한, 실시간 웹 애플리케이션, 네트워크 애플리케이션, 일반적인 목적의 애플리케이션, 분산된 시스템을 개발할 때도 사용할 수 있다.

2. 왜 Node.js를 사용하는가?
- 일반적으로 빠르다.
- 거의 차단(block) 하지 않는다.
- 통일된 프로그래밍 언어와 데이터 타입을 제공한다.
- 모든 것은 비동기이다.
- 대단한 동시성을 만들 수 있다.

3. Node.js의 특징은 무엇인가?
노드는 싱글 스레드이지만 확장성있는 시스템이다. 스크립트 언어로 자바스크립트를 사용한다.
분리된 프로세스나 스레드 대신에 비동기, event-driven I/O를 사용한다.
싱글스레드 이벤트 루프와 논블로킹 I/O 를 통해서 높은 output을 달성할 수 있다.

4. 다음 자바스크립트 코드를 Node.js를 사용해서 같은 결과를 내도록 하라.

``` javascript
console.log("first");
setTimeout(function() {
    console.log("second");
}, 0);
console.log("third");
```

``` javascript
console.log("first");
setImmediate(function(){
    console.log("second");
});
console.log("third");
```

5. NPM을 새 버전으로 어떻게 업데이트 하는가?
6. 왜 Node.js는 싱글스레드 인가?
7. Node.js에서의 콜백을 설명해보아라.
콜백 함수는 주어진 task가 완료되는 시점에 실행된다. 이는 그 동안 다른 코드가 실행되도록 하고 blocking을 막는다.

8. 콜백 지옥은 무엇인가?
콜백을 계속 포함하는 코드를 만드는 것이다. 이는 읽기도 어려울 뿐더러 유지보수하기도 힘들다.

9. 그렇다면, 콜백 지옥을 어떻게 막거나/고칠 수 있는가?
- 모든 단일 에러를 처리한다.
- Keep your code shallow
- Modularize : 더 작고 독립적인 함수로 콜백을 쪼갠다. 그리고 다른 파라미터와 함께 호출될 수 있게 하여 바라는 결과를 이루기 위해 결합한다.
- Promise, Generator, Async를 사용한다.

10. Node.js에서 REPL의 역할을 설명하시오.
11. Node.js의 API 함수들의 타입은?
- Blocking functions
- Non-blocking functions

12. Node.js 콜백 핸들러에 전달되는 첫번째 인자는 일반적으로 어떤 것인가?
일반적으로, optional error object 이다. 에러가 없으면 null이거나 undefined이다.

13. NPM의 역할은 무엇인가?
- Node.js 패키지의 온라인 저장소
- 패키지를 설치하고 버전 관리 그리고 의존성 관리를 도와주는 유틸리티(커맨드라인)

14. Node.js와 Ajax의 차이는 무엇인가?
- Ajax는 주로 페이지 content의 특정 섹션을 전체 페이지를 업데이트 하지 않고도 동적으로 업데이트 하기 위해 설계되었다.
- Node.js는 클라이언트-서버 애플리케이션을 개발하기 위해 사용된다.

15. Node.js의 체이닝에 대해 설명하라.


16. What are “streams” in Node.js? Explain the different types of streams present in Node.js.
스트림은 연속적인 프로세스로서 source로부터 데이터를 읽어서 destination에 데이터를 쓸 수 있게 해주는 객체이다.
스트림에는 4가지 타입이 있다.
    1. Readable
    2. Writable
    3. Duplex
    4. Transform

17. exit code는 무엇인가? exit code 들을 나열해 보아라.
프로세스를 끝내기 위해 사용되는 특정 코드들이다.
- Unused
- Uncaught Fatal Exception
- Fatal Error
- Non-function Internal Exception Handler
- Internal Exception handler Run-Time Failure
- Internal JavaScript Evaluation Failure

18. Node.js의 글로벌은 무엇인가?
3가지 키워드를 globals로서 구성한다.
    1. Global: 글로벌 네임스페이스 객체를 나타내고 모든 다른 `global` 객체들에 대한 컨테이너로서 작동한다.
    2. Process: 글로벌 객체들 중 하나지만, 동기적 함수에서 비동기적 콜백으로 바뀔 수 있다. 코드 어디에서나 접근할 수 있으며 주로 애플리케이션이나 환경에 대한 정보를 준다.
    3. Buffer: 바이너리 데이터를 처리하는 클래스이다.

19. AngularJS 와 Node.js의 차이는?
20. Why is consistent style important and what tools can be used to assure it?


Scripting language란 무엇인가?
소스코드를 컴파일하지 않고도 실행할 수 있는 언어를 말한다. 바로 인터프리터에 의해 실행이 된다.
컴파일 언어에 비해 빠른 실행 속도를 보인다는 것이 장점이다.

출처:[https://bytearcher.com/articles/io-vs-cpu-bound/](https://bytearcher.com/articles/io-vs-cpu-bound/)
'CPU bound' 와 'I/O bound'는 무엇을 의미하는가?

`I/O bound application`은 대부분의 시간을 네트워크, 파일시스템, 데이터베이스를 기다리는데 보낸다.
하드 디스크 속도나 네트워크 커넥션을 증가시키는 것이 전체 성능을 향상시킨다.

`CPU bound application`의 예로 SHA-1 체크섬을 계산하는 서비스를 생각해볼 수 있다.
대부분의 시간을 hash를 crunching 하는 데에 사용한다. 입력 문자열에 대해 많은 양의 bitwise xor와 shift를 실행하는 것이다.
이런 형태의 application은 Node.js에서 문제를 일으킨다. 대부분의 시간을 CPU 집약적인 일을 수행하다면 다른 모든 요청은 보류된다.
CPU 집약적인 일을 처리하기 위한 전략들이 있다. 
 - 계산하는 부분을 다른 곳으로 분리 시키는 것이다. libuv로 부터의 저수준 워커 스레드를 사용하거나 분리된 서비스를 만들어서, 자식 프로세스를 fork하거나 cluster 모듈을 사용한다. 
 - 최소한 setImeediate()를 설정하여 이벤트 루프에 실행을 자주 반환하는것을 할 수 있다.

출처:[https://bcho.tistory.com/881](https://bcho.tistory.com/881)
언제 Node.js 를 써야할까?
 - file upload/download와 같은 네트워크 스트리밍 서비스
 - real time web application; 채팅 서비스(socket.io)
 - SPA(Single Page Application)
 가볍고 생산성이 높은 웹 개발 프레임워크를 갖고 있고, 간단한 로직을 가지면서 대용량, 그리고 빠른 응답 시간을 요구 하는 애플리케이션에 적절한다.

언제 Node.js 를 쓰지 말아야 할까?
 - CRUD가 많고 페이지가 많은 웹 개발