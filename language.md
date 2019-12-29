---
title: 언어 구조
prev: "/intro/quickstart.html"
next: "/language/keywords.html"
---

## 루비 언어 구조[](#ruby-language-structure)

이 장은 구문 구조와 루비 프로그램의 일반 구조를
설명합니다.

간략히 살펴보면, 다음과 같습니다:

* 루비 프로그램을 구성하는 표현식을
  [리터럴](language/literals.md),
  [변수](language/variables-constants.md)
  [상수](language/variables-constants.md#constants)와 함께 사용합니다.
* 표현식은 다음과 같습니다:
  * [할당](language/assignment.md)
  * [제어문](language/control-expressions.md)
  * [메소드 호출](language/methods-call.md)
  * 모듈, 클래스, 메소드 정의하기

* 루비는 객체지향 언어이므로, 프로그램을 구성하려면
  [클래스와 모듈](language/modules-classes.md) 그리고 각각의
  [메소드](language/methods-def.md)를 정의합니다.
  * 루비는 오픈 클래스입니다. 따라서 언제라도 클래스를 수정할 수 있습니다. (심지어 코어
    클래스인 `String`이라도). 클래스를 수정하고
    확장을 구현하려면,
    [리파인](language/refinements.md)를 사용할 수 있습니다.

* 에러 출력하고 처리하려면
  [예외처리](language/exceptions.md)를 참고합니다.

주의사항. 앞으로 보게 될 루비 프로그램의 대다수에서 언어를 구성하는 것은
*메소드*입니다. 예를 들어 <a
href='https://ruby-doc.org/core-2.6/Kernel.html#method-i-raise'
class='ruby-doc remote' target='_blank'>`Kernel#raise`</a>는
예외를 발생시키고, <a
href='https://ruby-doc.org/core-2.6/Module.html#method-i-private'
class='ruby-doc remote' target='_blank'>`Module#private`</a>는
메소드 사용범위를 바꿉니다. 따라서, 이 장에서 설명하는 언어의 코어 부분은
매우 적으며, 그 밖의 다른 부분은
모듈, 메소드, 표현식에 대한 규칙을 다룹니다.



### 표현식 닫기[](#ending-an-expression)

루비는 줄바꿈 글자로 표현식을 닫습니다. 다만
오퍼레이터, 괄호 열기, 콤마 등이 줄 끝에 오면, 표현식을 닫지 않습니다.

`;`(세미콜론)으로 표현식을 닫을 수 있습니다. 세미콜론은 대체로
`ruby -e`와 함께 사용합니다.

### 들여쓰기[](#indentation)

루비는 들여쓰기를 하지 않아도 됩니다. 대체로 루비 프로그램은
두칸을 들여씁니다.

루비의 워닝을 켜고 들여쓰기를 맞지 않게 실행하면,
경고 메시지를 볼 수 있습니다.

### `defined?`[](#defined)

`defined?` 는 뒤에 오는 아규먼트가 무엇인지 문자열로 알려주는 키워드입니다:


```ruby
p defined?(UNDEFINED_CONSTANT) # nil 출력
p defined?(RUBY_VERSION)       # "constant" 출력
p defined?(1 + 1)              # "method" 출력
```

`defined?` 뒤에 괄호를 쓰지 않아 되지만, 괄호를 쓰도록 추천합니다.
`defined?`의 [우선순위](language/precedence.md)가
낮기 때문입니다.

만약 다음과 같이, 인스턴스 변수가 있는지 확인하고
그 인스턴스 변수값이 0인지 확인한다면:


```ruby
defined? @instance_variable && @instance_variable.zero?
```

실행 결과는 `"expression"`입니다. 인스턴스 변수가 정의되었는지 확인하려고 했던
의도와는 다릅니다.


```ruby
@instance_variable = 1
defined?(@instance_variable) && @instance_variable.zero?
```

인스턴스 변수가 정의되었는지 확인하려면
괄호를 사용하는 것이 좋습니다. 인스턴스 변수가 정의되지 않은 경우 실행결과는 `nil` 이고
인스턴스 변수가 0이 아니면 실행결과는 `false`입니다.

인스턴스 변수에 대해서는 instance\_variable\_defined?이거나
상수에 대해서는 const\_defined?와 같은 특정 상황에 맞춘 메소드를 사용하는 것이
`defined?`를 사용하는 것보다 에러 발생 가능성을 줄입니다.

,
