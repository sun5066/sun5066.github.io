---
layout: post
title: "JVM의 String 객체 생성과 관리 방식"
date: 2024-10-18 09:25:00 -0400 
categories: String
tags: JVM, Kotlin, Java, String constructor
comments: 1
---

Java, Kotlin과 같은 JVM 기반 언어를 사용해 앱 개발을 하다 보면, 여러 방법으로 String을 사용할 수 있다.

```
val strLiteral = "Hello World"
val charSequence = StringBuilder().append("hi")

// Byte 배열, Char 배열, CharSequence 등등..
val strConstructor1 = String(strLiteral.toByteArray())
val strConstructor2 = String(charSequence)
```

보통 문자열을 만들 때는 보통 리터럴 방식을 많이 사용하며,
이렇게 만들어진 문자열 객체는 JVM Heap에 존재하는 String Pool에 캐싱되어 같은 문자열에 대해서는 효율적으로 재사용한다.

그럼, `String(charSequence)`와 같은 방식은 캐싱이 안되는 걸까?

물론 가능하다. String 클래스에서 제공하는 `intern()`이라는 메서드를 사용한다면 말이다.

> intern() 메서드는 문자열을 String Pool에 저장하거나, 이미 존재하는 문자열이라면 해당 객체를 리턴한다.
>
> 주의해야 할 점으로 확실한 상황에서만 사용해야 하며, 그렇지 않다면 예기치 못한 문제가 발생할 수 있다.
>
> 해당 메서드에 대해 잘 모르겠다면, 찾아보길 권장한다.

기본적으로는 String Pool에 캐싱 되지 않으며, 일반 객체와 같이 Heap에 저장된다.

그럼, 이 코드는 어떻게 될까?

```
/**
 * 코틀린의 String 클래스는 String 매개 변수를 통한 생성자를 제공하지 않는다.
 * 따러서 java.lang.String 클래스를 참조해 생성해보자.
 * */
val str = java.lang.String("Hello Kotlin World")
```

위 코드에서는 String 생성자와 리터럴 방식 둘 다 사용하고 있다.

그리고 앞서 말했듯이, 리터럴 방식은 String Pool에 저장되고, 생성자 방식은 Heap에 저장된다.

정답은 둘 다 저장된다.

물론, 생성자와 리터럴을 함께 사용한 방식은 실무에서 거의 안쓴다.(적어도 나는 한 번도 본적이 없음)

그럼, 실무에서도 안쓰고, Kotlin은 이런 방식 자체를 제한했는데 굳이 왜? 라고 생각할 수 있다.

사실은 멘토님께서 질문을 주셨던 문제였고, 질문을 들었던 당시 나는 이것에 대해 답변을 하지 못했다.

나는 이것에 대해 학습을 했고, 적어도 앞으로 내가 작성한 코드가 내부적으로 어떻게 동작되는지 정확하게 아는 개발자가 될 수 있는 한 걸음이었다고 생각된다.