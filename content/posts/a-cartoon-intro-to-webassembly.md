+++
title = "[번역] 만화로 소개하는 웹어셈블리"
date = "2019-01-14"
description = "mozilla 시리즈 번역 - A cartoon intro to WebAssembly"
tags = [
    "웹어셈블리",
]
+++

원문: [cartoon intro to WebAssembly](https://hacks.mozilla.org/2017/02/a-cartoon-intro-to-webassembly/)

- 번역기와 함께 주관적인 이해를 바탕으로 작성되어, 오역 가능성이 높습니다. 모든 저작권은 원작자에게 있습니다.

자바스크립트는 1995년에 만들어졌습니다. 처음 10년 동안은 빠르게 동작하도록 설계되지 않았고, 빠르지도 않았습니다.

그 이후에 브라우저의 경쟁이 치열해지기 시작했습니다.

2008년에 브라우저의 성능 전쟁이 시작되면서, 다수의 브라우저에서 JITs라고도 불리는 JIT(실시간 just-in-time) 컴파일러를 도입했습니다. JIT는 자바스크립트가 실행되는 패턴을 관찰하고 그것을 기반으로 코드를 더 빠르게 실행할 수 있었습니다.

이러한 JIT의 도입은 자바스크립트 성능의 변곡점을 가져왔습니다. 자바스크립트의 실행이 10배 빨라졌습니다.

![](https://hacks.mozilla.org/files/2017/02/01-01-perf_graph05-768x628.png)

이렇게 성능이 향상되면서, 자바스크립트는 Node.js로 서버 측 프로그래밍을 작성하는 것처럼 아무도 예상하지 못했던 곳에서 사용되기 시작했습니다. 성능 향상은 자바스크립트를 완전히 새로운 종류의 문제에서 사용하는 것을 가능하게 했습니다.

우리는 지금 웹어셈블리와 함께 그 변곡점 중 어느 다른 한 곳에 있는지도 모릅니다.

![](https://hacks.mozilla.org/files/2017/02/01-02-perf_graph10-768x633.png)

그럼, 이제 무엇이 웹어셈블리를 빠르게 만드는지 알아보려고 합니다.

**Background:**

- [A crash course in just-in-time (JIT) compilers](https://hacks.mozilla.org/2017/02/a-crash-course-in-just-in-time-jit-compilers/)
- [A crash course in assembly](https://hacks.mozilla.org/2017/02/a-crash-course-in-assembly/)

**WebAssembly, the present:**

- [Creating and working with WebAssembly modules](https://hacks.mozilla.org/2017/02/creating-and-working-with-webassembly-modules/)
- [What makes WebAssembly fast?](https://hacks.mozilla.org/2017/02/what-makes-webassembly-fast/)

**WebAssembly, the future:**

- [Where is WebAssembly now and what’s next](https://hacks.mozilla.org/?p=30522)

### About [Lin Clark](https://twitter.com/linclark)

Lin works in Advanced Development at Mozilla, with a focus on Rust and WebAssembly.

- [https://twitter.com/linclark](https://twitter.com/linclark)
- [@linclark](http://twitter.com/linclark)

[More articles by Lin Clark…](https://hacks.mozilla.org/author/lclarkmozilla-com/)
