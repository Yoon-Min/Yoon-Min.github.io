---
title: Android Compose Navigation 01 - 컴포즈 내비게이션 소개 
author: yoonmin
date: 2024-09-01 00:00:00 +0900
categories: [Android, 라이브러리]
tags: [Android, Compose, Navigation]
render_with_liquid: true
---

![katie-drazdauskaite-xKFVBD4UAQA-unsplash](https://gist.github.com/user-attachments/assets/fe06427a-bba4-4d35-9dfd-f344ceef4022)_[Unsplash](https://unsplash.com/ko/사진/자동차-내부지도-차트의-매크로-사진-xKFVBD4UAQA?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)의[Katie Drazdauskaite](https://unsplash.com/ko/@kotrad?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)_

​		

**Compose Navigation 포스팅 현황**

+ [x] <span style="color: #2dc39e">**Android Compose Navigation 01 - 컴포즈 내비게이션 소개**</span>
+ [ ] <span style="color: #898989">Android Compose Navigation 02 - NavGraph 생성과정 (예정)</span>
+ [ ] <span style="color: #898989">Android Compose Navigation 03 - Navigator (예정)</span>

​		

## Intro

화면 이동은 모바일 애플리케이션에서 중요한 요소입니다. 당장 우리가 일상에서 사용하는 앱을 살펴봐도 단순히 하나의 화면만 있는 것이 아닌 여러 화면을 통해 사용자에게 다양한 서비스를 제공하는 것을 볼 수 있습니다. 모바일 운영체제 중 하나인 Android도 마찬가지입니다. 

그래서 Android는 이러한 화면 이동 구현을 지원하는 내비게이션 라이브러리를 지원하는데, 제가 이번 글에서 소개할 것은 **컴포즈 내비게이션**입니다. 몇 년 전부터 Android가 강하게 밀고 있는 UI 제작 키트인 컴포즈에서 내비게이션은 어떤 구조로 이루어져 있고 각각 어떤 역할을 하는지 간략하게 정리하고자 합니다.

내비게이션을 공부하면서 해당 라이브러리의 구조가 생각보다 깊고 복잡한 것을 느꼈기 때문에 컴포즈 내비게이션 포스팅은 단편이 아닌 여러 편으로 구성된 시리즈로 진행합니다. 그래서 이번 첫 번째 포스팅 이후에도 계속해서 글을 올릴 예정이니 필요하면 다음 편도 읽어보는 것도 좋을 것 같습니다.

​		

## 내비게이션은 여행과 같다.

![pin-adventure-map-0EZcIvzTjyo-unsplash](https://github.com/user-attachments/assets/83ade4b6-1e6a-4e9f-b856-677fda87d9ca)

장소(목적지) 형성

: 내비게이션은 여행과도 같습니다. 가고 싶은 장소들을 정하고 일정에 따라 원하는 목적지로 이동하여 관광을 즐깁니다. 예를 들어 여름철 강릉으로 여행을 출발한다면 갈 수 있는 장소로 숙소, 바닷가, 맛집, 카페 등을 정합니다. 이것들이 모이면 하나의 **여행지도**가 완성됩니다.

이동 수단

: 한 장소에서 다른 장소로 옮길 때 어떻게 이동할 것인지도 여행의 한 요소입니다. 한 장소에서 다른 장소로 이동하는 과정은 다양한데 당장 생각나는 것들을 나열하면 다음과 같습니다.

- 기름값을 나눠 내고 일행 중 자차가 있는 사람의 차를 이용
- 자동차 렌트
- 지하철, 버스, 택시와 같은 대중교통을 이용

가이드(플래너)

: 여행을 가게 되면 일행 중 누군가는 여행 계획을 세웁니다. 이 사람은 여행 중 가이드 역할을 진행하며 일행들의 상황을 파악하며 목적지 이동이나 목적지에서의 관광 과정에서 중간 소통자 역할을 합니다.

​		

## 여행 예시를 내비게이션 개념으로 연결짓기

![navigation-design-graph-top-level 2](https://github.com/user-attachments/assets/62feccf3-21d9-468d-92b6-47784def000c)

장소(목적지) 형성

: 여행을 가면 여러 곳을 방문하게 됩니다. 그래서 미리 지도 앱을 켜서 일정 중에 갈 곳을 확인하고 동선을 정합니다. 이것은 그래프 안에 필요한 목적지들을 등록하는 내비게이션의 그래프와 비슷합니다. 앱 내에서 `NavGraph` 는 지도의 역할을 수행하고 `NavDestination` 은 지도 내 목적지의 역할을 수행합니다.

이동 수단

: 여행에서 장소와 장소 사이를 옮길 때 이동 방법을 생각하는 것은 그래프에서 목적지와 목적지 사이를 이동할 때 메커니즘을 생각하는 것과 비슷합니다. 앱 내에서 내비게이팅에 대한 메커니즘을 정의하는 클래스를 `Navigator` 라고 합니다.

가이드(플래너)

: 원활한 여행을 위해 중간에서 주도하는 가이드는 내비게이션의 `NavController` 와 비슷합니다. 컨트롤러는 지정된 경로를 기반으로 다른 컴포저블 대상을 표시하는 컴포저블인 `NavHost` 내에서 앱 탐색을 관리합니다. 그래서 `NavGraph` 을 가지고 특정 목적지에 대한 내비게이팅 요청을 할 수 있습니다.

​				

## 컴포즈 내비게이션 제작 과정

### 1. 필요한 컴포저블 정의

![Group 76](https://github.com/user-attachments/assets/fc597814-0b9f-4a8e-beb1-679a97872c6e)

주문 시작화면 (1)

: 첫 번째 화면에는 주문할 컵케이크 수량에 해당하는 버튼 세 개가 표시됩니다.

맛 선택화면 (2)

: 수량을 선택하고 나면 앱에서 컵케이크 맛을 선택하라는 메시지를 사용자에게 표시합니다. 앱에서는 *라디오 버튼*을 사용하여 여러 옵션을 표시합니다. 사용자는 가능한 맛 옵션 중에서 하나를 선택할 수 있습니다.

수령일 선택화면 (3)

: 맛을 선택하면 앱에서는 수령일을 선택할 수 있는 여러 라디오 버튼을 사용자에게 표시합니다.

주문 요약화면 (4)

: 수령일을 선택하고 나면 사용자는 주문을 검토하고 완료합니다.

​		

### 2. NavHost 정의

<script src="https://gist.github.com/Yoon-Min/2fd995db0c495dafee855b2d416bd379.js"></script>

최상위 컴포저블 함수 내에 내비게이션 정보를 정의합니다. 내비게이션 정보는 `NavHost` 내에 정의합니다. 호스트는 내비게이션 정보를 담는 컨테이너입니다. 그래서 최종적으로 목적지 정보에 따른 다른 컴포저블을 화면에 표시합니다.

<script src="https://gist.github.com/Yoon-Min/ad976f50fcf53a113d975573b37919cc.js"></script>

내비게이션은 컨트롤러가 그래프를 읽고 제어하는 방식입니다. 그래서 호스트는 중간 다리 역할인 `NavController` 와 지도 역할인 `NavGraph` 을 연결합니다.

`NavController` 객체는 외부에서 직접 생성해서 인자로 전달해야 하지만 `NavGraph` 객체는 호스트 내부에서 생성을 해주기 때문에 개발자가 객체 생성을 할 필요는 없습니다. 대신에 그래프 구성요소인 목적지(1번~4번 화면) 내용을 정의해서 전달해야 합니다.

<script src="https://gist.github.com/Yoon-Min/50488e6e84f841958c92cd5dd7e79acc.js"></script>

목적지 정보 정의는 `composable` 함수를 이용해서 정의합니다. 인자로 화면을 지칭할 이름을 전달합니다. 예제에서는 `enum class` 로 각 화면의 이름을 정의해서 `route` 등록을 진행했습니다.

<script src="https://gist.github.com/Yoon-Min/56ad69e687c06a26fc78f9cdefd81735.js"></script> 

이후에는 `composable` 의 내용을 정의합니다. 내용 정의는 `content` 라는 이름을 가진 마지막 순서의 람다 함수에 진행합니다. 화면 컴포저블 함수를 등록하고 필요한 내비게이션 동작을 추가합니다. 여기서 다른 컴포저블 화면으로 이동하는 `navigate` 나 이전 화면으로 돌아가는 `popBackStack` 을 등록할 수 있습니다.

​		

## NavGraph 생성

<script src="https://gist.github.com/Yoon-Min/ad976f50fcf53a113d975573b37919cc.js"></script>

호스트 정의에서 설명했듯이 호스트는 내부에서 자체적으로 `navController.createGraph` 코드를 통해 그래프를 생성합니다. 해당 코드를 통해 우리가 호스트 영역에 정의했던 여러 `composable` 함수들이 담긴 `builder` 인자가 그래프의 구성요소로 들어갑니다. 필요한 화면들을 등록하면서 `startDestintation` 인자로 시작 화면을 설정합니다.

![Group 77](https://gist.github.com/user-attachments/assets/4c36c0c6-bdf4-4b4d-b086-eda8fb0a4bb7)

​		

### NavController 생성

<script src="https://gist.github.com/Yoon-Min/f6c36177d87a03e9049dcabcc118f165.js"></script>

호스트 정의 과정에서 보았듯이 최상위 컴포저블인 `CupcakeApp` 의 파라미터를 보면 컨트롤러를 가져옵니다. 컨트롤러의 디폴트 값으로 `rememberNavController()` 을 설정하는데 이를 통해 `NavHostController` 을 저장합니다. 그리고 이 컨트롤러를 호스트의 인자로 전달합니다.

​		

## 에뮬레이터로 동작 확인

![Screen_recording_20240901_161725](https://gist.github.com/user-attachments/assets/62b085b9-8f29-4b14-a622-672325ffdae9){: .w-50 .h-50}

​		

## 참조

[**Android Cupcake**](https://developer.android.com/codelabs/basic-android-kotlin-compose-navigation?hl=ko&continue=https%3A%2F%2Fdeveloper.android.com%2Fcourses%2Fpathways%2Fandroid-basics-compose-unit-4-pathway-2%3Fhl%3Dko%23codelab-https%3A%2F%2Fdeveloper.android.com%2Fcodelabs%2Fbasic-android-kotlin-compose-navigation#0)

[**Navigation with Compose**](https://developer.android.com/develop/ui/compose/navigation?authuser=1&hl=ko)
