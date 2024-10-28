---
layout: post
title: "Java Volatile"
date: 2024-10-29 09:25:00 -0400 
categories: Java
tags: Java, JVM, Volatile
comments: 1
---

Java의 volatile은 멀티스레드 환경에서 변수의 가시성을 보장하고, 명령어 재정렬을 방지하기 위해 사용되는 키워드다.

런타임에서 일반적으로 자바의 변수는 각 스레드의 PC Register의 캐시 메모리(또는 작업 메모리)에 복사되어 효율적으로 연산을 수행한다. 그러나 이 방식은 멀티스레드 환경에서 일관성을 보장하지 못해, RaceCondition 같은 문제가 발생할 수 있다.

### Volatile 주요 기능
1. 가시성 보장
  - volatile 키워드를 사용한 변수는 바로 메인 메모리에 쓰기/참조되는게 아니다.
  - 먼저, 해당 변수에 참조할 때 메인 메모리에 원본 변수를 참조하여, 캐시 메모리에 저장된 변수의 값을 업데이트 해, 동기화를 보장한다. 이를 가시성을 보장한다고 한다.
  - 하지만, 원자성을 100% 보장하지는 않는다. 이를 위해 엄격한 규칙이 필요하다.(이후 예시로 설명)

2. 명령어 재정렬 방지

먼저, 명령어 재정렬은 컴파일러와 CPU가 최적화를 위해 코드의 실행 순서를 효율적으로 변경하는 작업이다. 이 과정은 단일 스레드 환경에서는 문제가 없지만, 멀티스레드 환경에서는 예기치 않은 결과가 발생할 수 있다.

volatile 키워드는 이러한 명령어 재정렬이 방지되어, 변수의 쓰기 작업이 다른 연산들 앞뒤로 재배치되지 않도록 보장한다.

말로 설명하면 이해가 안될 수 있으니, 코드로 봐보자.

```
class ReorderingExample {
    private int x = 0;
    private boolean flag = false;

    public void writer() {
        x = 42;           // 명령어 1
        flag = true;      // 명령어 2
    }

    public void reader() {
        if (flag) {                 // 명령어 3
            System.out.println(x);   // 명령어 4
        }
    }
}
```

두 개의 스레드가 있다고 가정해보자.

- 스레드 A는 writer()를 호출해 x와 flag를 초기화
- 스레드 B는 reader()를 호출해 flag 값을 확인하고, true라면 x값 출력

**문제 발생 가능성**

컴파일러나 CPU가 최적화를 위해 명령어 1과 2의 순서를 바꾸면 다음과 같이 재정렬될 수 있다.

```
public void writer() {
    flag = true;     // 명령어 2
    x = 42;          // 명령어 1
}
```

이 경우 스레드 B가 reader()를 실행할 때 flag는 true이지만, x가 아직 0으로 나오는 상태를 볼 수 있다.(RaceCondition 발생)

**해결 방법**

flag 변수를 volatile로 선언하면, JVM은 명령어 재정렬을 방지하여 쓰기 작업의 순서를 보장한다.

```
// 이렇게 하면 writer()는 x = 42가 flag = true 앞에 있어야 한다는 규칙이 지켜진다.
private volatile boolean flag = false;

public void writer() {
    x = 42;           // 명령어 1
    flag = true;      // 명령어 2
}
```

### volatile 원자성 보장, 그리고 CopyOnWriteArrayList

앞서 설명 했듯이, volatile은 100% 원자성을 보장하지 못한다. volatile이 원자성을 보장하는 경우는 하나의 스레드에서 쓰기 작업을 하고, 다른 스레드에서 읽기 작업을 할 때이다.

아주 좋은 예시로, java.util.concurrent 패키지의 CopyOnWriteArrayList가 있다.

CopyOnWriteArrayList에 대해 짧게 설명해보자면,

- 쓰기 작업이 적고, 읽기 작업이 많은 경우에 사용해야한다.
- 쓰기 작업에만 동기화(synchronized)가 걸려있고, 읽기 작업에는 동기화가 되어있지 않으므로, 효율적으로 동기화 작업을 수행.

CopyOnWriteArrayList의 내부 코드를 참고해보자.

```
public class CopyOnWriteArrayList<E> implements List<E>, ... {
    ...
    final transient Object lock = new Object();
    private transient volatile Object[] array;

    static <E> E elementAt(Object[] var0, int var1) {
        return var0[var1];
    }

    public E get(int var1) {
        return elementAt(this.getArray(), var1);
    }
    
    public boolean add(E var1) {
        synchronized(this.lock) {
            ...
        }
    }
    ...
}
```

- CopyOnWriteArrayList의 원소를 담는 array에는 volatile 키워드가 선언되어 있다.
- get()과 같은 읽기 작업에는 동기화가 걸려있지 않고, add()와 같은 쓰기 작업에는 동기화가 걸려있다.(remove, set 등 포함)

즉 volatile 변수를 멀티스레드 환경에서 원자성을 보장하기 위해서 쓰기 작업은 단일 스레드에서 작업하거나 synchronized를 사용해야한다.

읽기 작업에는 가시성을 보장하기에 참조시 최신값을 가지고 연산을 진행한다.

### 참고 자료
- [JVM 밑바닥까지 파헤치기](https://github.com/WegraLee/JVM)
- [갓생사는 김초원의 개발 블로그](https://programmer-chocho.tistory.com/82)