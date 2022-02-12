---
layout: post
title: Java에서 Exception - 1
date: 2022-02-12 17:00:00 +0900
categories: development
tags: 개발 java go
---

Java는 오류 발생시 Exception을 사용하도록 만든 언어이다. 현재 시점으로 최신 [Java SE Specification의 Exceptions](https://docs.oracle.com/javase/specs/jls/se17/html/jls-11.html)를 확인해 보면 그 의도가 잘 나와 있다. 그것을 요약하면 다음과 같다.

- Java의 설계 목표는 이식성과 **견고성**을 제공하는 것.
- -1과 같은 funny value를 이용하는 것이 아닌 **명시적인 문법을 이용하게 하는** 것.
- 예외가 처리되지 않는 **상태가 되도록 하는** 것.

--

서두에 위를 먼저 언급한 이유는 요새 개발자들이 다양한 언어를 접하고 사용하게 됨으로써 ‘Exception을 사용할 필요가 있는가?’ 라는 의문점을 제시하는 것을 종종 보았기 때문이다. 그럴 때마다 나는 아래와 같이 말한다.

“언어는 그 언어 나름대로 철학이 있다. 그런 철학이 설계(디자인)로 나타난다. 그러므로 큰 이유가 없다면 100% 맞출 필요는 없지만 되도록 그 언어 철학에 맞추어 사용하는 것이 좋다. Java는 Exception을 사용하도록 한 언어이다.”

내가 좋아하는 언어인 Go를 봐보자. Go는 오류를 Java의 Exception으로 간주하지 않고 일반 값으로 본다. 여기서 일반 값은 위 Java의 Exception 의도에서 나온 funny value와는 다르다. Go에서는 이러한 오류를 일반 값으로 사용하게 함으로써 개발자가 오류 처리시 간결한 흐름으로 처리하게 만들고 또한 오류 처리시 더 많은 신경을 쓰도록 하고자 한 목표가 있다. 이게 오류를 처리하기 위한 Go 언어의 철학이다.

**Go의 예시**:

```go
resp, err := http.Get(url)
if err != nil {
    return nil, err
}
```

이렇듯 언어마다 의도한 그 나름대로 철학이 있으므로 나는 될 수 있으면 그 언어의 목적에 맞게 사용하는 것이 좋다고 생각한다.

물론 예외적으로 생각해볼 만한 것도 있다. 비록 Java 언어를 사용하고 있더라도 애플리케이션을 설계하는 아키텍트가 Exception을 사용하지 않는다는 철학과 오류를 처리할 수 있는 디자인을 확립하여 일관성 있게 진행할 수 있다면 그 언어의 본래 목적과는 달라도 괜찮다고 나는 생각한다. 예로 다음과 같이 함수형 언어 철학을 일부 사용하여 Go 언어와 같이 오류는 값으로 전달한다는 철학이다. 실제로 Java의 많은 라이브러리, 프레임워크에서 이와 같은 모습을 확인할 수 있다. 참고로 실제로는 결괏값을 처리하는 것을 다음의 예시처럼 단순하게 처리하지 않을 것이므로 본 예시는 단순히 참고만 하면 되겠다.

**첫 번째 예시**:

```java
// ResultOrError<MyResult, AppError> handle(MyData data)
var roe = handle(data);
if (roe.isError()) {
    // do something
}
```

**두 번째 예시**:

```java
// class Success<V> extends Result<V>
// class Failure<V> extends Result<V>
//
// Result<String> handle(MyData data)
var result = handle(data)
if (result instanceof Failure) {
    // do something
}
```

결국 내 생각을 정리하자면 위와 같이 일관성 있게 아키텍처를 이끌어 가기 어렵다면 그 언어 특성에 맞게 규칙을 만들어 사용하면 된다는 것이 나의 생각이다.

<br>

그렇다면 Java에서 Exception을 어떻게 사용하는 것이 좋을까? 여러 가지 의견이 있지만 다음과 같은 기조를 이용하면 좋을 듯하다.

1. Checked Exception을 잘 활용하기 어렵다면 Unchecked Exception만 활용.
2. Throw early, catch late.
3. 호출자를 위해 많은 콘텍스트를 포함한 Exception.

그런데 위 기조가 어떤 의미를 말하는 것일까?

첫 번째의 기조는 몇십 년 동안 말이 많은 부분이다. Java를 사용하는 개발자라면 Checked Exception과 Unchecked Exception이 무엇인지는 잘 이해하고 있을 것으로 생각한다. 이 중에서 Checked Exception 같은 경우 무용론 주장이 20년 넘게 지속되고 있다. 현재 Java가 함수형 언어 철학을 조금씩 도입하고 있는데 여기서 함수형 인터페이스를 사용하면서 Checked Exception을 제어해야 하는 불만도 많다. 무용론자들이 말하는 “Checked Exception은 Java 언어의 큰 실수다.”라는 것은 정말일까? 그것에 따라 Unchecked Exception만 사용해야 할까?

두 번째, “Throw early, catch late”는 무슨 말일까. 세 번째, 호출자를 위해 많은 콘텍스트를 포함한 Exception은 어떤 형태의 Exception일까?

한 개의 포스트에서 쓰기에는 글이 길어질듯 하여 위의 기조들에 대해서는 다음 글에서 설명하겠다.
