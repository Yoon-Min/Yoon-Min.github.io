---
title: Android Compose Navigation 02 - NavGraph 생성과정 
author: yoonmin
date: 2025-01-30 00:00:00 +0900
categories: [Android, 라이브러리]
tags: [Android, Compose, Navigation]
render_with_liquid: true
---

![Image](https://github.com/user-attachments/assets/52478b00-7706-415a-a732-9f4a696d2d48)_사진: [Unsplash](https://unsplash.com/ko/사진/둥근-흰색-나침반-iDzKdNI7Qgc?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)의[Jordan Madrid](https://unsplash.com/ko/@jordanmadrid?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)_

​		

**Compose Navigation 포스팅 현황**

+ [x] <span style="color: #898989">Android Compose Navigation 01 - 컴포즈 내비게이션 소개</span>
+ [x] <span style="color: #2dc39e">**Android Compose Navigation 02 - NavGraph 생성과정**</span>
+ [ ] <span style="color: #898989">Android Compose Navigation 03 - Navigator (예정)</span>

​		

## Intro

지난 포스팅에서 Compose Navigation에 대해 간단하게 소개하는 시간을 가졌습니다. 이를 통해 `NavHost` 내부에 `NavController`와 `NavGraph`를 가지고 내비게이션을 생성하는 것을 알았습니다. 이번 포스팅은 좀 더 들어가서 `NavHost` 내부에 생성되는 `NavGraph`에 대해 알아보겠습니다.

​		

## NavHost 내 NavGraph 생성

### 그래프 생성 과정

<script src="https://gist.github.com/Yoon-Min/ad976f50fcf53a113d975573b37919cc.js"></script>

1편에서 봤듯이 `NavHost` 는 최상위 컴포저블에 정의합니다. 그 과정에서 `NavController` 와 `NavGraph` 을 내부에서 사용하는데 전자는 인자로 받지만, 후자는 인자가 아닌 내부에서 직접 생성합니다.

<script src="https://gist.github.com/Yoon-Min/ae6e72b8b12ba433183abf33a66b451d.js"></script>

컨트롤러가 `createGraph` 메서드를 호출하면서 그래프를 생성하는데, 메서드 내부를 보면 컨트롤러가 가지고 있는 내비게이터 객체를 통해 그래프 생성을 하는 것을 확인할 수 있습니다. 여기서 내비게이터 객체에 해당하는 `navigatorProvider` 은 `NavigatorProvider` 클래스를 인스턴스화한 것입니다.

<script src="https://gist.github.com/Yoon-Min/d684c2d694c8c4fb6c04703b2fab0818.js"></script>

내비게이터 프로바이더 객체를 가져온 후,  `navigation` 메서드를 호출하여 그래프를 생성하게 됩니다. 그래프 생성은 프로바이더가 직접 하는 것이 아닌 그래프 빌더 객체가 진행합니다. 그래서 내비게이터가 그래프 빌더에게 그래프 생성을 맡기면서 본인을 인자로 전달합니다.

​		

### 그래프 생성 흐름

![제목 없는 다이어그램 drawio](https://gist.github.com/user-attachments/assets/dcf65f53-46c4-497e-8112-f6ae10b61fa1)

​		

## NavGraphBuilder

### NavGraphBuilder

<script src="https://gist.github.com/Yoon-Min/3a7cb08b459e36aa700b833870139633.js"></script>

`NavGraphBuilder` 의 클래스 구조입니다. 우선 생성자로 출발지 정보와 `NavigatorProvider` 을 전달합니다. 이것들을 통해 그래프 빌더를 생성합니다. 그리고 나머지 멤버 메서드를 살펴보면 생각보다 복잡하지 않습니다. 각 목적지를 저장하는 가변 리스트 하나, 목적지를 추가하는 메서드, 빌더 패턴에 필수적인 `build` 메서드 이렇게 세 가지 요소가 있습니다.

​		

### apply

<script src="https://gist.github.com/Yoon-Min/d684c2d694c8c4fb6c04703b2fab0818.js"></script>

그래프 빌더 객체를 생성했다면 코틀린 스코프 함수인 `apply` 을 통해 `builder` 라는 인자를 처리합니다. 여기서 `builder` 는 처음 `NavHost` 을 컴포저블 함수 내에 생성할 때, 필요한 화면을 `composable` 내에 정의했던 코드 블록을 말합니다. 

<script src="https://gist.github.com/Yoon-Min/2fd995db0c495dafee855b2d416bd379.js"></script>

그런데 주목해야 할 부분이 `builder` 인자의 타입이 함수이며 해당 함수는 일반 함수가 아닌 `NavGraphBuilder` 객체의 확장 함수라는 것입니다. 따라서 위 코드처럼 호스트 내에 정의한 코드 블록은 그래프 빌더 객체를 참조할 수 있으며, `composable` 확장 함수를 바로 호출할 수 있는 것입니다.

<script src="https://gist.github.com/Yoon-Min/56ad69e687c06a26fc78f9cdefd81735.js"></script>

`composable` 확장 함수 내부를 보면 그래프 빌더 내 목적지들을 추가할 수 있는 `addDestination` 을 호출하는 것을 볼 수 있습니다. 이곳에서 개발자가 전달한 목적지 경로 정보를 `NavDestination` 형태로 감싸고 이를 그래프 빌더에 목적지 리스트에 추가합니다. 제가 예제로 사용한 CupCake 는 4개의 화면 구성을 위해 4개의 `composable`을 사용했으니 `apply` 함수 내에서 4번의 `addDestination` 작업이 발생할 것입니다.

​		

### build

<script src="https://gist.github.com/Yoon-Min/4748bda565b73ee547e0ac13f60ee28a.js"></script>

빌더 패턴의 대표 메서드인 빌드 함수입니다. 이곳에서 진짜 그래프 객체를 생성하여 반환합니다. 여기서 `super.build()`를 통해 `ComposeNavGraph` 을 생성 및 불러오고 빌더 객체에 임시로 저장했던 목적지 객체들을 전달합니다. 최종적으로 사용자가 지정한 모든 목적지 정보를 가진 내비게이션 그래프가 생성되는 것입니다.

<script src="https://gist.github.com/Yoon-Min/a36c19bf4b81756a6ded63356d5bdddb.js"></script>

빌더 객체의 빌드 함수를 보면 부모 클래스가 대신 생성하여 전달해 줍니다. 부모 클래스인 `NavDestinationBuilder` 을 보면 빌드 함수 내에 `createDestination()` 을 호출합니다. `navigator` 은 컴포즈 내비게이터를 가리키며 이를 통해 `ComposeNavGraph` 을 생성합니다. 이는 `ComposeNavGraphNavigator` 클래스 코드를 보면 이해하기 쉽습니다.

<script src="https://gist.github.com/Yoon-Min/00b10811a98d627321b1ce6124419ce8.js"></script>

​		

## 정리

```kotlin
private fun createNavController(context: Context) =
    NavHostController(context).apply {
        navigatorProvider.addNavigator(ComposeNavGraphNavigator(navigatorProvider))
        navigatorProvider.addNavigator(ComposeNavigator())
        navigatorProvider.addNavigator(DialogNavigator())
    }
```

최상위 컴포저블 함수에서 `NavHost` 생성을 위해 인자로 넘기는 컨트롤러는 `navController: NavHostController = rememberNavController()` 입니다. 이 컨트롤러는 위 함수를 거쳐 생성되고 그 결과로  `ComposeNavigator` 와`ComposeNavGraphNavigator` 을 관리합니다. 그리고 이 내비게이터들을 이용하여 `NavGraphBuilder` 로부터 그래프 객체인 `ComposeNavGraph` 을 받아 반환합니다. 

![제목 없는 다이어그램 drawio](https://gist.github.com/user-attachments/assets/fbb26c1d-ce8d-4461-8d88-162dbb4372ff)

​		

## 참조

[**Android Cupcake**](https://developer.android.com/codelabs/basic-android-kotlin-compose-navigation?hl=ko&continue=https%3A%2F%2Fdeveloper.android.com%2Fcourses%2Fpathways%2Fandroid-basics-compose-unit-4-pathway-2%3Fhl%3Dko%23codelab-https%3A%2F%2Fdeveloper.android.com%2Fcodelabs%2Fbasic-android-kotlin-compose-navigation#0)
