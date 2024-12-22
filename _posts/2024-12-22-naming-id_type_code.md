---
layout: post
title: "이름 짓기: ID, Type, Code"
date: 2024-12-22 18:10:00 +0900
categories: development
tags: naming
---

개발할 때 이름을 짓는 것이 가장 어렵다. 너무 길지 않으면서 의미 전달이 잘 되는 이름을 지어야 하는데 어휘가 풍부하지 않으니 어려움을 많이 느낀다. 이 이름 짓기에 대해 어느 정도 규칙을 만들어 놓으면 그나마 그 어려움을 조금씩 덜어준다. 규칙에 따라 생성되었기 때문에 다른 개발자들도 빠르게 이해할 수 있으며, 이름 지을 때 우선 기계적으로 조립해서 사용할 수 있어서 이름 짓는 고민을 덜어준다.

이번 글은 이와 관련된 이름 짓기 첫 번째 글이며 ID, Type, Code를 접미사로 사용할 때 구분에 도움이 되는 설명이다. 관련해서 동료들과 논의한 적이 있고 그때 어떤 예시를 활용한 사례가 있었는데 그 예시가 괜찮아 보여 이를 사용해 설명하고자 한다.

어떤 값을 분류하고자 할 때 ID, Type, Code 접미사를 이용하면 어떤 맥락을 가지고 있는지 쉽게 이해할 수 있다. 다만 간혹 Type과 Code는 헷갈리는 경우가 있다. ID는 다른 2개에 비해 명확하지만 Type 또는 Code는 이 경우에도 어울리고, 저 경우에도 어울리는 느낌을 받을 때가 있다. 이런 경우 ID, Type, Code를 어떻게 잘 분류해서 사용할 수 있을까?

구분하기 가장 쉬운 ID 먼저 설명하겠다. ID는 말 그대로 개체의 유일성을 나타내는 값이다. 가장 쉽게 생각할 수 있는 것은 주민등록번호다. 한국에서 주민등록번호는 한국 국적인 사람 각자에게 주어지는 유일한 값이다. 즉, 주민등록번호로 한국인을 각 1명씩 구분할 수 있다. 즉, 여러 개체가 있는데 특정 개체를 ID라는 값으로 명확하게 한 개로 구분할 수 있다면 ID가 될 수 있다는 것이다.

코드로 풀어내면 다음과 같겠다.

```kotlin
import java.util.UUID

class Person() {
    val id = UUID.randomUUID().toString()

    override fun toString(): String {
        return "id=$id"
    }
}

fun main() {
    val person1 = Person()
    val person2 = Person()

    println(person1)
    println(person2)
}
```

Type은 대상의 본질적인 성질 또는 특성에 따라 분류될 수 있을 때 사용한다. 동료들과 논의하면서 내가 사용한 예시는 혈액형이다. 혈액형은 미영어로 blood type이다. 대표적인 혈액형 A, B, AB, O형만 가지고 설명을 할 경우 A형인 사람 여러명, B형인 사람 여러명, AB형인 사람 여러명, O형인 사람 여러명이 있을 수 있다. 이와 같이 type은 동일한 type을 가지는 여러 개체를 만들 수 있고, 그런 개체들을 동일한 type의 그룹으로 묶을 수 있다.

우리가 사용하는 언어에서도 int, long 등을 나타내는 type이 있다. 이것들도 본질적인 성질로 구분이 되기 때문에 type으로 불리는 것이다.

코드로 풀어낸다면 다음과 같겠다.

```kotlin
import java.util.UUID

enum class BloodType {
    A, B, AB, O
}

class Person(val bloodType: BloodType) {
    val id = UUID.randomUUID().toString()
    
    override fun toString(): String {
        return "id=$id, bloodType=$bloodType"
    }
}

fun main() {
    val person1 = Person(BloodType.O)
    val person2 = Person(BloodType.A)
    val person3 = Person(BloodType.O)

    println(person1)
    println(person2)
    println(person3)
}
```

Code는 외부에 의해 특정 규칙이나 약속으로 분류할 때 사용한다. 역시 동료들과 논의할 때 사용한 예시는 국가다. 국가는 국가 코드인 country code로 분류한다. Country type이 아니다.

혈액형은 사람이 태어날때 이미 결정되어 있으며, 일반적으로 그 이후에 바뀌지 않는다. 이는 사람의 본질적인 속성 중 하나다. 하지만 국가는 사회적, 법적 합의에 의해 결정되는 외부적인 분류 방식이며 본질적인 특성이 없다. 국가 코드를 예로 들자면 “국가를 분류하기 위해 2자리로 사용하고 한국은 KR, 미국은 US로 씁시다”와 같이 합의에 의해 결정하는 것이다. “이런 특성을 가지면 KR로 쓰고 저런 특성을 가지면 US 씁시다”가 아니라는 것이다.

이런 code들은 type과 달리 변화 가능성이 높다. 이는 본질적인 특성이 없기 때문에 상황에 따라 사용할 수 있기 때문이다. 예로 태어날 때는 미국 국적으로 가졌더라도, 언제든지 법적 절차를 통해 한국 국적으로 변경할 수 있다.

코드로 풀어내면 다음과 같겠다.

```kotlin
import java.util.UUID

enum class BloodType {
    A, B, AB, O
}

enum class CountryCode {
    KR, US
}

class Person(val bloodType: BloodType, private var countryCode: CountryCode) {
    val id = UUID.randomUUID().toString()

    fun immigrate(newCountryCode: CountryCode) {
        countryCode = newCountryCode
    }

    override fun toString(): String {
        return "id=$id, bloodType=$bloodType, countryCode=$countryCode"
    }
}

fun main() {
    val person1 = Person(BloodType.O, CountryCode.KR)
    val person2 = Person(BloodType.A, CountryCode.US)
    val person3 = Person(BloodType.O, CountryCode.US)

    person2.immigrate(CountryCode.KR)

    println(person1)
    println(person2)
    println(person3)
}
```

ID, Type, Code의 개념을 위와 같이 분류하고 그에 맞게 이름 짓기를 하면 다음과 같은 효과가 있다.

- ID로 지정하면 개체의 유일성을 나타냄을 의미
- Type으로 지정된 것들은 내부의 특수 성질로 인해 스스로 분류되는 것이며 변경 가능성이 낮다는 의미
- Code는 외부 관점으로 특정 규칙에 따라 분류되는 것이며 변경 가능성이 높다는 의미

Type, Code가 여전히 헷갈린다면 합의를 통해 Code로만 통일해서 사용하는 것도 좋다고 생각한다. Code는 상황과 맥락에 따라 의미가 달라질 수 있어서 Type보다 더 추상적인 개념이기 때문이다. 어느 것이든 소통과 협업을 강화할 수 있는 이름을 선택해서 규칙으로 정하고 유지해야 한다.
