---
layout: post
title: Java에서 Exception - 2
date: 2022-02-20 17:00:00 +0900
categories: development
tags: 개발 java exception
---

[저번 글](/development/2022/02/12/java_exception_1.html)에 이어서 Java의 Exception을 잘 사용하기 위한 나의 생각을 계속 말하겠다. 당시 말한 기조는 다음과 같았다.

1. Checked Exception을 잘 활용하기 어렵다면 Unchecked Exception만 활용.
2. Throw early, catch late.
3. 호출자를 위해 많은 콘텍스트를 포함한 Exception.

<br>

첫 번째, ‘Checked Exception을 잘 활용하기 어렵다면 Unchecked Exception만 활용’ 부분이다. 여기서 말한 Checked Exception은 여전히 논란이 많으며 인기가 없는 Exception이다. 사실 본질적으로는 Java의 Exception이 더 인기가 없을지도 모르겠다.  
이 Checked Exception은 자바의 견고성 때문에 고안된 부분이다. [James Gosling이 말하길](https://www.artima.com/articles/failure-and-exceptions) 프로그래밍 언어 자체가 모든 것을 해결할 수는 없지만 해당 언어로 작성된 애플리케이션(=프로그램)이 어떤 환경에서도 최대한 살아 남을 수 있도록 할 수 있는 모든 것들을 해주는 것이 좋은 일이며 그 예로서 이 Checked Exception이 될 수가 있다고 했다. 이 Checked Exception을 사용함으로서 언어는 개발자에게 명시적으로 “너는 이 예외가 발생할 수 있음을 지금 확인 했어.”라고 알려주게 된다는 것이다. 비록 그 개발자가 그 예외를 단순히 무시하는 행위를 하더라도 Unchecked Exception 처럼 못 알아차린 채로 지나치는 것보다 어느 정도 알게 만들었다는 환경이 중요하다는 것이다.

문제는 실제 개발되면서 운영되는 애플리케이션에 Checked Exception이 생각보다 너무 많고 해당 Checked Exception을 처리할 유의미한 과정이 없다는 것이다. 이런 Checked Exception을 제어하기 위하여 무의미한 `try - catch` 제어 로직 추가로 코드의 가독성을 떨어트리고 복잡성을 높이고 있다. 그리고 이와 관련된 [2016년 연구](https://www.overops.com/blog/ignore-checked-exceptions-all-the-cool-devs-are-doing-it-based-on-600000-java-projects/)에 따르면 Github에 존재하는 자바 프로젝트들 중에서 Checked Exception을 제어하는 79%가 단순히 로깅만 하거나 비어둔 채로 사용한다는 결과도 있다.

결과만 보면 James Gosling이 원하는 대로 의도적으로 예외를 제어하고 있다는 것이 보이지만 이게 과연 효율적인가는 다른 문제이다. 결국 이러한 문제 때문에 Checked Exception이 바라는 목적과는 별개로 부정적인 느낌과 결과만 만들어 냈고 “Checked Exception을 사용하지 말자”라는 움직임은 여전히 진행 중이다.

여기서의 나의 의견은 “Checked Exception을 무조건 배제하지 않는다”라는 것이 내 의견이다. 다만 이 Exception은 명확하게 써야 할 곳만 써야 하며 그 장소는 최소화 해야 한다. 지금의 문제는 File I/O와 같이 너무나 일반적이면서도 저수준의 영역에서 Checked Exception을 사용할 경우 이를 제대로 처리할 수 없어서 무의미한 제어 로직을 사용한다는 것이 문제이다. 실제 우리가 작성하는 애플케이션의 규모는 매우 크다. 실 세계에서 운용되는 애플리케이션은 우리가 프로그래밍 언어 배울 때 단순히 파일을 읽고 내용을 저장하는 일만 하는 것이 아니다. 이렇게 규모가 큰 애플리케이션을 만드는데 특정 부분에서 File I/O와 관련된 Exception 처리를 해야 한다고 생각해보자. 이 경우 어떻게 처리할 것인가? 해당 레이어에서 처리해야 할까? 상위 레이어에 전달하여 처리하도록 해야 할까? 상위 레이어에 전달 시 Checked Exception으로 그대로 보내줘야 할까? 이런 부분은 애플리케이션의 확장성에서도 똑같은 문제가 나타난다. 이는 앞서 말했듯이 너무나 일반적인 부분에서 Checked Exception을 사용했기 때문이다. 이런 부분은 Unchecked Exception을 사용하는 게 좋다. Unchecked Exception을 사용하여 관심이 있는 영역 또는 레이어에서 이 Exception을 처리하는 게 좋다. 그래야 Exception이 처리되어야 하는 지점에서 명확히 처리가 되고, 불필요한 코드가 작성되지 않을 것이기 때문이다.

그렇다면 언제 Checked Exception을 사용하는 게 좋을까? 일반적인 영역이 아니라 특수한 상황이며 이를 제어 가능한 영역에서 사용해야 한다. 특수한 상황의 한 예로 비즈니스 도메인이 될 수 있겠다. 더 직접적인 예로 금융과 관련된 애플리케이션을 개발한다고 가정해 보자. 사용자 계좌에서 돈을 차감 후 특정 계좌로 돈을 이동 시킬 때 이 돈을 이동 시키는 모듈에서 문제가 발생할 수 있다. 문제가 발생했음을 알려주는 방법으로 Checked Exception, Unchecked Exception, 또는 결괏값을 이용할 수 있다. 만약 아키텍트가 우리의 애플리케이션은 어떤 일이 발생하더라도 금액과 관련하여서는 무결성을 보장해야 한다는 주목적을 가지고 있을 경우 이런 돈을 이동 시키는 모듈에서 문제 발생 시 Checked Exception을 통해 알려주는 방법을 취할 수 있다. 그렇게 함으로서 해당 애플리케이션에 참여하는 모든 개발자들이 이런 예외 상황을 강제로 인지하게 만들고 이러한 예외 발생 시 무결성을 달성할 수 있도록 추가 로직을 개발하게 만들 것이다. 그리고 File I/O와 같이 너무나 일반적인 부분이 아니기 때문에 예외가 무분별하게 재전달되는 상황도 막을 수 있다. 만약 이를 가이드해줄 수 있는 아키텍트가 없거나 관련 아키텍처 문서가 잘 유지되고 있지 않다면 Unchecked Exception을 무조건 사용하는 게 좋다. 그래야 애플리케이션이 계속 성장하면서 발생할 수 있는 복잡성을 Checked Exception을 사용할 떄보다 더 줄일 수 있다.

그럼 이제 Checked Exception이 아닌 Unchecked Exception 이야기를 해보자. Unchecked Exception의 단점은 잘 정의되어 있는 문서가 없다면 코드를 보지 않는 이상 어떤 예외가 발생할 수 있는지 알 수 없다는 것이다. 애플리케이션을 운영하는 중 특정 상황에 해당되어 Exception 발생하는 환경이 구축되어야 확인이 가능하다. 사실 이건 어떻게 보면 당연하다. Unchecked Exception의 목적 중 하나가 어떤 개발의 실수를 알리기 위한 용도이기도 하기 때문이다. 하지만 아무리 그렇다고 하더라도 운영할 때마다 Exception을 발견하여 처리하도록 하는 것은 문제다. 이를 해소하기 위해서 다음과 같은 방법을 사용하는 게 좋다고 생각한다.

- 발생 가능한 Unchecked Exception을 문서(JavaDoc)로 알린다.
- 의미 있는 Unchecked Exception을 제공한다.

발생 가능한 Unchecked Exception은 JavaDoc 같은 것을 이용하여 최대한 힌트를 주는 것이 좋다. 예로 Spring Framework의 [TagUtils#assertHasAncestorOfType](https://github.com/spring-projects/spring-framework/blob/main/spring-web/src/main/java/org/springframework/web/util/TagUtils.java) 처럼 주면 좋다고 생각한다.

```java
/**
 * Determine whether the supplied {@link Tag} has any ancestor tag
 * of the supplied type, throwing an {@link IllegalStateException}
 * if not.
 * @param tag the tag whose ancestors are to be checked
 * @param ancestorTagClass the ancestor {@link Class} being searched for
 * @param tagName the name of the {@code tag}; for example '{@code option}'
 * @param ancestorTagName the name of the ancestor {@code tag}; for example '{@code select}'
 * @throws IllegalStateException if the supplied {@code tag} does not
 * have a tag of the supplied {@code parentTagClass} as an ancestor
 * @throws IllegalArgumentException if any of the supplied arguments is {@code null},
 * or in the case of the {@link String}-typed arguments, is composed wholly
 * of whitespace; or if the supplied {@code ancestorTagClass} is not
 * type-assignable to the {@link Tag} class
 * @see #hasAncestorOfType(jakarta.servlet.jsp.tagext.Tag, Class)
 */
public static void assertHasAncestorOfType(Tag tag, Class<?> ancestorTagClass, String tagName,
        String ancestorTagName) {

    Assert.hasText(tagName, "'tagName' must not be empty");
    Assert.hasText(ancestorTagName, "'ancestorTagName' must not be empty");
    if (!TagUtils.hasAncestorOfType(tag, ancestorTagClass)) {
        throw new IllegalStateException("The '" + tagName +
                "' tag can only be used inside a valid '" + ancestorTagName + "' tag.");
    }
}
```

의미 있는 Unchecked Exception이란 Exception 발생 시 해당 Exception이 어떤 이유로 발생했는지를 쉽게 파악할 수 있도록 해주라는 것이다. Guava의 [Conditional Failures](https://github.com/google/guava/wiki/ConditionalFailuresExplained#user-content-summary) 부분이 딱 좋은 예시이다.

- Precondition: You messed up (caller).
- Assertion: I messed up.
- Verification: Someone I depend on messed up.
- Test assertion: The code I'm testing messed up.
- Impossible condition: What the? the world is messed up!
- Exceptional result: No one messed up, exactly (at least in this VM).

<br>

두 번째, ‘[Throw early, catch late](https://learning-notes.mistermicheels.com/architecture-design/exception-handling/#throw-early-catch-late)’ 부분이다. 간단히 설명하자면 Exception 발생 시 그 Exception을 처리하는 영역 또는 레이어가 최적의 위치가 아니라면 처리하지 않고 바로 전파하라는 것이다. 더 실질적인 예를 든다면 다음과 같이 처리하지 말라는 것이다.

```java
try {
    foo.bar();
} catch (Exception ignore) {
}
```

위와 같이 작성하기 보다 이를 처리할 수 있는 적합한 곳으로 전파시키는 것이 옳다. 만약 Checked Exception을 처리해야 하고 이 Checked Exception을 전파시키지 않아야 하는데 해당 영역에서 특별히 할 수 있는 게 없다면 애플리케이션에서 공유되는 커스텀 Exception을 이용해 재전파 하는 게 좋다. Java에서 사용하는 일반적인 RuntimeException이 아니라 애플리케이션에서 공유되는 커스텀 Exception을 이용하는 게 좋은 이유는 해당 Exception을 사용했다는 것만으로도 특정 지점에서 일종의 제어가 있었다는 힌트를 주기 때문이다.

```java
try {
    foo.bar();
} catch (FileNotFoundException e) {
    throw new MyCustomException("Failed to handle foobar files.", e);
}
```

<br>

세 번째, ‘호출자를 위해 많은 콘텍스트를 포함한 Exception’ 부분이다. Exception이 발생했다면 그 Exception 만으로 어떤 이유 때문에 문제가 발생했는지 알 수 있어야 한다. 스택 트레이스에 출력된 코드 위치로 가서 확인한다는 것 자체가 많은 수고스러움을 준다. 예로 다음 `foo` 메서드는 1 이상의 숫자만 받도록 되어 있는 비즈니스 메서드라고 가정하자. 이때 아래와 같이 콘텍스트 없이 Exception을 발생한다면 결국 구현된 코드를 살펴봐야 하는 수고스러움이 있다.

```java
public static void foo(int num) {
    if (num < 0) {
        throw new MyCustomIllegalArgumentException("I warn you!");
    }
    // ... do something ...
}
```

다음과 같이 작성해야 Exception 발생 시 원인 분석이나 제어 시 수월하다.

```java
public static void foo(int num) {
    if (num < 0) {
        var msg = String.format("Num is less than 1. Num is %d.", num);
        throw new MyCustomIllegalArgumentException(msg, num);
    }
    // ... do something ...
}
```

<br>

이것 외에도 Exception을 잘 사용하기 위한 좋은 방법들이 많이 있다. 나는 그중에서 가장 중요하다고 생각하는 것을 적은 것 뿐이다. Java 언어를 사용하고 있고 앞서 설명한 별도의 예외 메커니즘(`ResultOrError`나 `Either` 같은 Tuple)을 수립하여 일관성 있게 진행하지 않을 것이라면 적어도 이 기조를 사용하는 게 좋다고 생각한다.

개인적으로 Java에서의 Exception 보다는 Go 언어에서의 오류 처리 방식을 선호한다. 아마도 나와 같이 생각하는 개발자가 많이 있을 것이다(...없을 수도 있겠다). 이때 “Java의 Exception은 좋지 않다.”라는 자세를 취하여 무작정 다른 언어의 기본 오류 메커니즘을 도입하기 보다는 Java에서의 Exception이 어떤 의미인지를 먼저 이해하고 이런 언어의 목적 안에서 내가 원하는 바를 달성하기 위해 어떤 것을 해보면 좋을지를 한 번 더 생각해보고 그 생각의 결과에 따라 진행하면 좋겠다.
