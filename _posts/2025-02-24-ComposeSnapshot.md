---
title: Android Compose - 내부 코드로 알아보는 Compose UI 상태 자동 업데이트 과정
author: yoonmin
date: 2025-02-24 00:00:00 +0900
categories: [Android, Compose]
tags: [Android, Compose, UI, State, Snapshot]
render_with_liquid: true
---

![Image](https://github.com/user-attachments/assets/a032896c-9a78-480c-876f-bc171ff2a231)_사진: [Unsplash](https://unsplash.com/ko/사진/흰색-직물-위에-여러-초상화-qqd8APhaOg4?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)의[sarandy westfall](https://unsplash.com/ko/@sarandywestfall_photo?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)_

# Intro

안드로이드의 최신 UI 제작 킷인 컴포즈는 상태를 등록하면 자동으로 관찰 및 수정을 처리하는 특징을 가지고 있습니다. 덕분에 컴포즈를 사용하는 개발자는 단순히 상태를 전달하기만 하면 되고, 상태가 변경됐을 때 이를 적용하는 수고를 컴포즈 런타임이 대신합니다. 이를 가능하게 만드는 대표 객체중 하나가 `MutableState` 입니다. 해당 상태 객체에 값을 저장하고 UI 데이터로 초기 설정하면 나중에 수정됐을 때 자동으로 적용됩니다.

이것은 이전 뷰 시스템에 비해 혁신적이라 할 수 있습니다. 굳이 **상태 변경을 적용한다는 코드를 명시할 필요 없이, 컴포즈가 자체적으로 다 처리**하기 때문입니다. 그렇다면 여기서 궁금한 점이 생깁니다. 컴포즈 런타임은 어떻게 상태를 관찰하고 변경사항을 알아서 잘 적용할 수 있는 걸까요?

이번 포스트는 컴포즈의 상태 추적 및 적용 시스템을 `MutableState` 객체를 통해 추적해보고 내부 코드와 동작을 분석해 보겠습니다. 상태 관리가 매우 중요한 컴포즈에서 해당 동작이 어떻게 처리되는지 이해하면 컴포즈 기반의 UI 제작에 많은 도움이 될 것입니다.

​		

# State 처리 과정 되짚기

## 상태 값을 UI 데이터로 적용하는 과정

```kotlin
@Composable
fun StatefulCounter(modifier: Modifier = Modifier) {
    var count by rememberSaveable { mutableStateOf(0) }
    StatelessCounter(
        count = count,
        onIncrement = { count++ },
        modifier = modifier
    )
}
```

1. 특정 뷰(위젯)에 표시해야 하는 데이터를 위해 상태 홀더 클래스인 `MutableState` 을 생성합니다. 

2. 해당 객체는 `count` 변수에 저장됩니다. 

3. 하위 컴포저블의 인자를 정의하는 과정에서 상태 값을 전달합니다.

   ```kotlin
   @Composable
   fun StatelessCounter(count: Int, onIncrement: () -> Unit, modifier: Modifier = Modifier) {
       Column(modifier = modifier.padding(16.dp)) {
           if (count > 0) {
               Text("You've had $count glasses.")
           }
           Button(
               onClick = onIncrement,
               enabled = count < 10,
               modifier = Modifier.padding(top = 8.dp)
           ) {
               Text("Add one")
           }
       }
   }
   ```

   하위 컴포저블에서 버튼을 클릭하면 상태 값이 증가한다는 것을 `onClick = onIncrement` 코드로 확인할 수 있습니다. 그리고 텍스트 뷰에 표시할 데이터로 `count` 를 지정했습니다.

​		

## 상태 값이 변경되는 과정

```kotlin
@Composable
fun StatelessCounter(count: Int, onIncrement: () -> Unit, modifier: Modifier = Modifier) {
    Column(modifier = modifier.padding(16.dp)) {
        if (count > 0) {
            Text("You've had $count glasses.")
        }
        Button(
            onClick = onIncrement,
            enabled = count < 10,
            modifier = Modifier.padding(top = 8.dp)
        ) {
            Text("Add one")
        }
    }
}
```

1. 모바일 애플리케이션 사용자가 버튼을 누릅니다.
2. 클릭 이벤트가 발생합니다, - `onClick`
3. 컴포저블 함수 내용 토대로 `onIncrement` - `onIncrement = { count++ }` 가 실행됩니다.
4. `count` 값이 증가합니다.
5. 컴포즈 시스템이 해당 컴포저블 내 `count` 상태 객체가 수정된 것을 인식합니다.
6. 리컴포지션을 진행하여 최신 값을 UI 데이터로 적용합니다.

​		

## 여기서 생기는 궁금증

```
1. MutableState 내부 구조는?
2. Compose 시스템의 상태 객체 감지 방법은?
```

컴포즈 런타임이 실시간으로 상태 객체를 관찰할 수 있는 것은 `State` 객체와 연관이 있습니다. 이 두 가지의 연관성을 이해하려면 먼저 `스냅샷` 시스템을 알 필요가 있습니다. 

​		

# Snapshot

## 특정 값의 순간을 캡처



