---
title: "Jetpack Compose 이해 - 공식문서 읽어보기"
date: 2022-08-31 17:20:00 +0900
categories: android
---

# Android Developers Compose 공식 문서

Jetpack Compose는 Android를 위한 현대적인 선언형 UI 도구 키트이다. Compose는 프론트엔드 뷰를 명령형으로 변형하지 않고도 UI를 렌더링할 수 있게 하는 선언형 API를 제공하여 앱 UI를 더 쉽게 작성하고 유지관리할 수 있도록 지원한다. 이 용어에 관해 몇 가지 설명이 필요하며, 앱 디자인에 있어 중요한 함의를 갖는다.

## 선언형 프로그래밍 패러다임

지금까지 Android 뷰 계층 구조는 UI 위젯의 트리로 표시할 수 있었다. 사용자 상호작용 등의 이유로 인해 앱의 상태가 변경되면, 현재 데이터를 표시하기 위해 UI 계층 구조를 업데이트 해야함. UI를 업데이트하는 가장 일반적인 방법은 `findViewById()`와 같은 함수를 사용하여 트리를 탐색하고 `button.setText(String)`, `container.addChild(View)` 또는 `img.setImageBitmap(String)`과 같은 메서드를 호출하여 노드를 변경하는 것이다. 이러한 메서드는 위젯의 내부 상태를 변경한다.

뷰를 수동으로 조작하면 오류가 발생할 가능성이 커진다. 데이터를 여러 위치에서 렌더링 한다면 데이터를 표시하는 뷰 중 하나를 업데이트하는 것을 잊기 쉽다. 또한 두 업데이트가 예기치 않은 방식으로 충동할 경우 잘못된 상태를 야기하기도 쉽다. 예를 등러 업데이트가 UI에서 방금 삭제된 노드의 값을 설정하려고 할 수 있다. 일반적으로 업데이트가 필요한 뷰의 수가 많을수록 소프트웨어 유지관리 복잡성이 증가함.

지난 몇 년에 걸쳐 업계 전반에서 선언형 UI 모델로 전환하기 시작했으며, 이에 따라 사용자 인터페이스 빌드 및 업데이트와 관련된 엔지니어링이 크게 간소화되었다. 이 기법은 처음부터 화면 전체를 개념적으로 재생성한 후 필요한 변경사항만 적용하는 방식으로 작동한다. 이러한 접근 방식을 `Stateful` 뷰 계층 구조를 수동으로 업데이트할 때의 복잡성을 방지할 수 있다.

화면 전체를 재생성하는데 있어 한 가지 문제는 시간, 디바이스 성능 및 배터리 사용량 측면에서 잠재적으로 비용이 많이 든다는 것이다. 이 비용을 줄이기 위해 Compose는 특정 시점에 UI의 어떤 부분을 다시 그려야 하는지를 지능적으로 선택한다. 이는 재구성에 설명된 대로 UI 구성요소를 디자인하는 방식에 몇 가지 영향을 미친다.

## 간단한 Composable 함수

Compose를 사용하면 데이터를 받아서 UI 요소를 내보내는 Composable 함수 집합을 정의하여 사용자 인터페이스를 빌드할 수 있다.

```

// 데이터를 전달받고 이를 사용하여 화면에 텍스트 위젯을 렌더링하는 간단한 Composable 함수
@Composable
fun Greeting(name: String) {
    Text("Hello $name")
}
```

이 함수에 관해 몇 가지 주목할 만한 참고 사항
- `@Composable` 어노테이션은 함수가 데이터를 UI로 변환하기 위한 함수라는 것을 Compose 컴파일러에게 알린다.
- 매개변수를 통해 사용자 이름을 렌더링한다.
- 실제로 UI 요소를 생성하는 `Text()` Composable 함수를 호출한다. Composable 함수는 다른 Composable 함수를 고출하여 UI 계층 구조를 생성할 수 있다.
- 함수는 리턴타입이 없다. UI를 내보내는 Compose 함수는 UI 위젯을 구성하는 대신 원하는 화면 상태를 설명하므로 아무것도 반환할 필요가 없다.
- 이 함수는 빠르고 멱등원이며 부작용이 없다.
    - 함수는 동일한 인수로 여러 번 호출될 때 마다 동일한 방식으로 작동하며, 전역 변수 또는 `random()` 호출과 같은 다른 값을 사용하지 않는다.
    - 함수는 속성 또는 전역 변수 수정과 같은 부작용 없이 UI를 형성한다.
      일반적으로 모든 Composable 함수는 재구성에서 설명한 이류로 인해 이러한 속성을 사용하여 작성해야한다.

## 선언형 패러다임 전환

많은 명령형 객체 지향 UI 도구 키트를 사용하여 위젯의 트리를 인스턴스화함으로써 UI를 초기화한다. 흔히 XML 레이아웃 파일을 확장하여 이 작업을 한다. 각 위젯은 자체의 내부 상태를 유지하고 앱 로직이 위젯과 상호작용할 수 있도록 하는 getter 및 setter 메서드를 노출한다.

### 데이터가 곧 UI 이다.

Compose의 선언형 접근 방식에서 위젯은 비교적 Stateless 상태이며 setter 또는 getter 함수를 노출하지 않는다. 사실상 위젯은 객체로 노출되지 않는다. 동일한 Composable 함수를 다른 인수로 호출하여 UI를 업데이트합니다. 이렇게 하면 공식문서의 아키텍처 가이드에서 설명된 대로 ViewModel과 같은 아키텍처 패턴에 상태를 쉽게 제공할 수 있다. 그런 다음, 컴포저블은 식별 가능한 데이터가 업데이트될 때마다 현재 애플리케이션 상태를 UI로 변환한다.

![컴포저블 함수의 UI 렌더링 동작 메커니즘](https://developer.android.com/static/images/jetpack/compose/mmodel-flow-data.png)

> 앱 로직은 최상위의 Composable 함수에 데이터를 제공한다. 그러면 함수는 데이터를 사용하여 다른 컴포저블을 호출함으로써 UI를 형성하고 적절한 데이터를 해당 컴포저블 및 계층 구조 아래로 전달한다.(Screen -> LazyList -> Item 형태)

사용자가 UI와 상호작용할 때 UI는 `onClick`과 같은 이벤트가 발생했을때, 앱의 상태가 변경될것이며, Composable 함수(Composable)는 새 데이터와 함께 다시 호출된다. 이렇게 하면 UI 요소가 다시 그려진다. 이러한 프로세스를 재구성이라고 한다.

![컴포저블 함수에서 UI 이벤트가 발생했을때 새로운 데이터와 함께 재호출되는 동작 메커니즘](https://developer.android.com/static/images/jetpack/compose/mmodel-flow-events.png)

## 동적 콘텐츠

Composable 함수는 XML이 아닌 Kotlin으로 작성되기 때문에 다른 Kotlin 코드와 마찬가지로 동적일 수 있다.

## 재구성

명령형 UI 모델에서 위젯을 변경하려면 위젯에서 setter를 호출하여 내부 상태를 변경한다. Compose에서는 새 데이터를 사용하여 Composable 함수를 다시 호출한다. 이렇게 하면 함수가 재구성되며, 필요한 경우 함수에서 내보낸 위젯이 새 데이터로 다시 그려진다. Compose 프레임워크는 변경된 구성요소만 지능적으로 재구성 할 수 있다.

```
@Composable
fun ClickCounter(clicks: Int, onClick: () -> Unit) {
    Button(onClick = onClick) { Text("클릭 횟수 $clicks") }
}
```

버튼이 클릭될 때마다 호출자는 `clicks` 값을 업데이트 한다. Compose는 `Text` 함수를 사용해 람다를 다시 호출하여 새 값을 표시한다. 이 프로세스를 재구성이라고 한다. 값에 종속되지 않은 다른 함수는 재구성 되지 않는다.

앞서 논의했듯이 전체 UI 트리를 재구성하는 작업은 컴픁팅 성능 및 배터리 수명을 사용한다는 측면에서 컴퓨팅 비용이 많이 들 수 있다. Compose는 이 지능적 재구성을 통해 이 문제를 해결한다.
(즉 리액트 처럼 변경이 필요한 컴포넌트만 UI 렌더링이 다시 된다는 말)

재구성은 입력이 변경될 때 Composable 함수를 다시 호출하는 프로세스이다. 이는 함수의 입력이 변경될 때 발생한다.
Compose는 새 입력을 기반으로 재구성할 때 변경되었을 수 있는 함수 또는 람다만 호출하고 나머지는 건너뛴다. 매개변수가 변경되지 않은 함수 또는 람다를 모두 건너뜀으로써 Compose의 재구성이 효율적으로 이루어질 수 있다.

함수의 재구성을 건너뛸 수 있으므로 Composable 함수(Composable) 실행의 부작용에 의존해서는 안된다. 그렇게 하면 사용자가 앱에서 이상하고 예측할 수 없는 동작을 경험할 수 있다. 부작용은 앱의 나머지 부분에 표시되는 변경사항이다. 예를 들어 다음 작업은 모두 위험한 부작용이다.

- 공유 객체의 속성에 쓰기
- ViewModel에서 식별 가능한 요소 업데이트
- 공유 환경설정 업데이트

Composable 함수(Composable)는 애니메이션이 렌더링될 때와 같이 모든 프레임에서와 같은 빈도로 재실행될 수 있다. 애니메이션 도중 버벅거림을 방지하려면 Composable 함수가 빨라야한다. 공유 환결설정에서 읽기와 같은 비용이 많은 작업을 실행해야 하는 경우 백그라운드 코루틴에서 작업을 실행하고 값 결과를 Composable 함수에 매개변수로 전달한다.

예를 들어 다음 코드는 컴포저블을 생성하여 `SharedPreferences`의 값을 업데이트한다. 컴포저블은 공유 환경설정 자체에서 읽거나 쓰지 않아야 한다. 대신 이 코드는 백그라운드 코루틴의 `ViewModel`로 읽기 및 쓰기를 이동한다. 앱 로직은 콜백과 함께 현재 값을 전달하는 업데이트를 트리거한다.

```
@Composable
fun SharedPrefsToggle(
    text: String,
    value: Boolean,
    onValueChanged: (Boolean) -> Unit
) {
    Row {
        Text(text)
        Checkbox(checked = value, onCheckedChange = onValueChanged)
    }
}
```

이 문서에서는 Compose에서 프로그래밍할 때 알아야 할 여러 가지 사항을 설명한다.

- Composable 함수는 순서와 관계없이 실행할 수 있다.
- Composable 함수는 동시에 실행할 수 있다.
- 재구성은 최대한 많은 수의 Composable 함수 및 람다를 건너뛴다.
- 재구성은 낙관적이며 취소될 수 있다.
- Composable 함수는 애니메이션의 모든 프레임에서와 같은 빈도로 매우 자주 실행될 수 있다.(달라의 경우 선물 아이템 큐 시스템)

다음 섹션에서는 Composable 함수를 빌드하여 재구성을 지원하는 방법을 설명한다. 어떤 경우든, Composable 함수를 빠르고 멱등원이며 부작용 없는 상태로 유지하는 것이 좋다.

## Composable 함수는 순서와 관계없이 실행할 수 있음

Composable 함수의 코드를 살펴보면 코드가 표시된 순서대로 실행된다고 가정할 수 있다. 그러나 코드가 반드시 표시된 순서대로 실행되는 것은 아니다. Composable 함수에 다른 Composable 함수 호출이 포함되어 있다면 그 함수는 순서와 관계없이 실행될 수 있다. Compose에는 일부 UI 요소가 다른 UI 요소보다 우선순위가 높다는 것을 인식하고 그 요소를 먼저 그리는 옵션이 있다.

예를 들어 탭 레이아웃에 세 개의 화면을 그리는 다음과 같은 코드가 있다고 가정해보자.
```
@Composable
fun ButtonRow() {
    MyFancyNavigation {
        StartScreen()
        MiddleScreen()
        EndScreen()
    }
}
```

`StartScreen`, `MiddleScreen` 및 `EndScreen` 호출은 순서와 관계없이 발생할 수 있다. 즉, 예를 들어 `StartScreen`이 일부 전역 변수(부작용)를 설정하고 `MiddleScreen`이 이러한 변경사항을 활용하도록 할 수 없음을 의미한다. 대신 이러한 각 함수는 독립적이어야 한다.

## Composable 함수는 동시에 실행할 수 있음

Compose는 Composable 함수를 동시에 실행하여 재구성을 최적화할 수있다. 이를 통해 Compose는 다중 코어를 활용하고 화면에 없는 Composable 함수를 낮은 우선순위로 실행할 수 있다.

이 최적화는 Composable 함수가 백그라운드 스레드 풀 내에서 실행될 수 있음을 의미한다. Composable 함수가 `ViewModel`의 함수를 호출하면 Compose는 동시에 여러 스레드에서 이 함수를 호출할 수있다.

애플리케이션이 올바르게 작동하도록 하려면 모든 Composable 함수에 부작용이 없어야 한다. 대신 UI 스레드에서 항상 실행되는 `onClick`과 같은 콜백에서 부작용을 트리거한다.

Composable 함수가 호출될 때 호출자와 다른 스레드에서 호출이 발생할 수 있다. 즉, 구성 가능한 람다의 허용되지 않는 부작용 이기 때문이다.(뭔말이지)

다음은 목록과 개수를 표시하는 Composable 예시이다.
```
@Composable
fun ListComposable(myList: List<String>) {
    Row(horizontalArrangment = Arrangment.SpaceBetween) {
        Column {
            myList.forEach { Text("item: $it") }
        }
        Text("count: ${myList.size}")
    }
}
```

이 코드는 부작용이 없으며 입력 목록을 UI로 변환한다. 하지만 함수가 로컬변수에 쓰는 경우 코드는 스레드로부터 안전하거나 적절치않다.
```
@Composable
fun ListWithBug(myList: List<String>) {
    var items = 0

    Row(horizontalArrangement = Arrangment.SpaceBetween) {
        Column {
            myList.forEach { item ->
                Text("item: $item")
                items++
            }
        }
    Text("count: $items")
    }
}
```

이 예에서 `items`는 모든 재구성을 통해 수정된다.
수정은 애니메이션의 모든 프레임에서 또는 목록이 업데이트될 때 실행될 수 있다.
어느 쪽이는 UI에 잘못된 개수가 표시된다.
이 때문에 이와 같은 쓰기는 Compose에서 지원하지 않는다.
이러한 쓰기를 금지함으로써 프레임워크가 구성 가능한 람다를 실행하도록 변경할 수 있다.

## 재구성은 가능한 한 많이 건너뜀

UI의 일부가 잘못된 경우 Compose는 업데이트해야 하는 부분만 재구성하기 위해 최선을 다한다. 즉, UI 트리에서 위 또는 아래에 있는 컴포저블을 실행하지 않고 단일 버튼의 컴포저블을 다시 실행하는 것을 건너뛸 수 있다.

모든 구성 가능한 함수 및 람다는 자체적으로 재구성할 수 있다.
다음은 목록을 렌더링할 때 재구성이 일부 요소를 건너뛸 수 있는 방법을 보여주는 예시이다.

```
@Composable
fun NamePicker(
    header: String,
    names: List<String>,
    onNameClicked: (String) -> Unit
) {
    Column {
        Text(
            text = header, 
            style = MaterialTheme.typeography.h5
        )
        Divider()
        LazyColumn {
            items(names) { name ->
                NamePickerItem(name, onNameClicked)
            }
        }
    }
}

@Composable
private fun NamePickerItem(name: String, onClicked: (String) -> Unit) {
    Text(name, Modifier.clickable(onClick = { onClicked(name) }))
}
```

이러한 각 범위는 재구성 도중에 실행할 유일한 사항일 수 있다.
Compose는 `header`가 변경될 때 상위 요소중 어느 것도 실행하지 않고 `Column` 람다로 건너뛸 수 있다.
그리고 `Column`을 실행할 때 Compose는 `names`가 변경되지 않았다면 `LazyColumn`의 항목을 건너뛰도록 선택할 수 있다.

다시 말하지만, 모든 구성 가능한 함수 또는 람다를 실행하는 작업에는 부작용이 없어야 한다.
부작용을 실행해야하는 할 때는 콜백에서 부작용을 트리거해야 한다.

## 재구성은 낙관적임

Compose가 컴포저블의 매개변수가 변경되었을 수 있다고 생각할 때마다 재구성이 시작된다.
재구성은 낙관적이다. 즉, Compose는 매개변수가 다시 변경되기 전에 재구성을 완료할 것으로 예상한다.
재구성이 완료되기 전에 매개변수가 변경되면 Compose는 재구성을 취소하고 새 매개변수를 사용하여 재구성을 다시 시작할 수 있다.

재구성이 취소되면 Compose는 재구성에서 UI 트리를 삭제한다.
표시되는 UI에 종속되는 부작용이 있다면 구성이 취소된 경우에도 부작용이 적용된다.
이로 인해 일관되지 않은 앱 상태가 발생할 수 있다.

낙관적 재구성을 처리할 수 있도록 모든 구성 가능한 함수 및 람다가 멱등원이고 부작용이 없는지 확인해야 한다.

## Composable 함수는 매우 자주 실행될 수 있음

경우에 따라 구성 가능한 함수는 UI 애니메이션의 모든 프레임에서 실행될 수 있다. 함수가 기기 저장소에서 읽기와 같이 비용이 많이 드는 작업을 실행하면 이 함수로 인해 UI 버벅거림이 발생할 수 있다.

예를 들어 위젯이 기기 설정을 읽으려고 하면 잠재적으로 이 설정을 초당 수백 번 읽을 수 있으며 이는 앱 성능에 치명적인 영향을 줄 수 있다.

Composable 함수에 데이터가 필요하다면 데이터의 매개변수를 정의해야 한다. 그런 다음, 비용이 많이 드는 작업을 구성 외부의 다른 스레드로 이동하고 `mutableStateOf` 또는 `LiveData`를 사용하여 Compose에 데이터를 전달할 수 있다.