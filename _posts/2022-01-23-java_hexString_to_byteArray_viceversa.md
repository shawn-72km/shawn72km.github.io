---
layout: post
title: Java에서 Byte Array를 HexString으로 또는 그 반대로 쉽게 전환하기
date: 2022-01-23 11:30:00 +0900
categories: development
tags: 팁 개발 JAVA
---

최근 별도 라이브러리 없이 순수 Java 언어만 이용하여 코드 샘플을 작성해야 하는 것 중에서 Hex String을 Byte Array로 또는 그 반대로 Byte Array를 Hex String으로 전환해야 하는 경우가 있었다. 이 경우 내가 선호하는 방식은 다음과 같다.

<br>

**Hex String → Byte Array**

```java
new BigInteger("0123456789abcdef", 16).toByteArray();
```

**Byte Array → Hex String**

```java
var byteArray = new byte[] { (byte) 0x01, (byte) 0x23, (byte) 0xab, (byte) 0xef };
String.format("%0" + (byteArray.length << 1) + "x", new BigInteger(1, byteArray));
```

<br>

Stack Overflow에서 [관련 질문의 답변 중 1개](https://stackoverflow.com/a/943963)인 위의 방식을 내가 가장 선호하는 이유는 다른 것보다 코드가 간결하면서도 꽤 직관적이기 때문이다. 그리고 이 방식을 쓰는 상황들은 앞서 말한 것과 같은 샘플이나 잠깐만 사용하는 작업인 상황이 대부분이기 때문에 위의 방식을 선호한다.

하지만 속도를 생각한다면 위의 방식은 효율적이지 않다. 느리다. 만약 실제 운영 중인 환경에서 사용한다면 Apache Commons Codec 라이브러리나 Google Guava 라이브러리를 사용하는 게 좋다. 만약 라이브러리 사용이 민감하거나 불가능한 운영 환경이라면 Byte Array를 Hex String으로 전환할 때에 대해 아래와 같은 코드를 사용하는 게 좋다.

<br>

Stack Overflow의 ['How to convert a byte array to a hex string in Java?'의 답변](https://stackoverflow.com/a/9855338)

```java
private static final byte[] HEX_ARRAY = "0123456789ABCDEF".getBytes(StandardCharsets.US_ASCII);

public static String bytesToHex(byte[] bytes) {
    byte[] hexChars = new byte[bytes.length * 2];
    for (int j = 0; j < bytes.length; j++) {
        int v = bytes[j] & 0xFF;
        hexChars[j * 2] = HEX_ARRAY[v >>> 4];
        hexChars[j * 2 + 1] = HEX_ARRAY[v & 0x0F];
    }
    return new String(hexChars, StandardCharsets.UTF_8);
}
```

<br>

기본 데이터 유형과 기본 연산자만을 이용하여 계산하기 때문에 그것만 보더라도 ‘아, 이것 정말 빠르겠구나!’라고 느낄 수가 있다. 코드도 복잡하지 않다. Hex의 2개 문자가 8bit의 각 4bit들을 표현한다는 것을 알고 있다면 쉽게 생각할 수 있는 방법이다. 조금 더 설명한다면 아래와 같은 것을 구현했다고 보면 되겠다.

- 16진수 `7b` 는 2진수로 `0111 1011` 이다.
    + `0111` 은 16진수로 `7` 이다.
    + `1011` 은 16진수로 `b` 이다.
- 16진수 `0f` 는 2진수로 `0000 1111` 이다.
- `0111 1011` 을 rshift 4 하면 `0000 0111` 이 되고 이는 `7` 이 된다.
- `0111 1011` 을 `0f` 와 `AND` 연산을 하면 `0000 1011` 이 되고 이는 `b` 가 된다.

작성해보니 괜한 설명을 추가한 것 같기도 하다. 아무튼... Java 17에서는 [HexFormat](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/HexFormat.html)이 포함된다고 하니 17부터는 그냥 이것을 사용하자.!?
