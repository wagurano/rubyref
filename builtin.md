---
title: 내장 클래스
prev: "/language/metaprogramming.html"
next: "/builtin/core.html"
---

## 내장 (코어) 클래스[](#built-in-core-classes)

이 장은 언어에서 항상 사용가능한
"코어" 클래스라고 불리는 것을 설명합니다.

루비에서 모든 것은 객체이다라는 명제는 매우 중요합니다.
모든 표현식은 [특정
장](language/control-expressions.md)을 제외하고는 메소드 호출입니다.

다음과 같이, 이 코드는 숫자 "5"를 표준 출력으로 보여줍니다.


```ruby
puts 2 + 3
```

이 코드는 실제로, `Kernel#puts` 메소드를 호출합니다. receives one
이 메소드로 전달하는 아규먼트는 `2`객체가 `3`으로 `Integer#+`메소드를 호출한 결과입니다.

> **Note**\: 문서 관습에 따라 인스턴스 메소드는
> `ClassName#method_name`와 같이 사용하고, 클래스 메소드는
> `ClassName.method_name`와 같이 사용하여 구분합니다. 루비 프로그램에서
> `#`를 사용하여 메소드를 호출하지 않습니다.

주의할 점: 객체를 직접 표시하지 않고
어디서나 사용할 수 있는 "기본" 메소드 대부분은 (예, `puts`, `exit`)
[`Kernel` 모듈](builtin/core.md#kernel)에서 설명합니다. 이 모듈은
모든 객첵에 포함되어 있습니다. 따라서 `puts`는 실제로 `self.puts`입니다.
`Kernel` 메소드의 목록은 [부록 A](appendix-a.md)를 참고합니다.

이 장은 표준 라이브러리의 부분을 다룹니다.
예를 들어, `Date` 클래스 (표준 라이브러리)는
`Time` 클래스 (코어 클래스) 아래에 설명합니다.
표준 라이블러리와 모듈에 대해 지정하려면
`require` 명령어를 사용합니다.

그 밖의 표준 라이브러리는 [표준
라이브러리](stdlib.md) 장에 있습니다.

