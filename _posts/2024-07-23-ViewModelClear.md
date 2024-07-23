---
title: Android - 화면 구성 변환에도 ViewModel 데이터가 보존될 수 있는 이유
author: yoonmin
date: 2024-07-23 00:00:00 +0900
categories: [Android, 라이브러리]
tags: [Android, 뷰모델, ViewModel]
render_with_liquid: true
---

![lance-anderson-QdAAasrZhdk-unsplash 1](https://gist.github.com/user-attachments/assets/0a434876-f30d-412e-95ce-e87d3866e09c)

## Intro

뷰모델은 `ViewModelStoreOwner` 가 사라질 때까지 메모리에 남아 있습니다. 덕분에 액티비티의 화면 구성 변경 후에도 뷰모델을 유지할 수 있습니다. 뷰모델이 파괴되는 조건은 다음과 같이 세 가지가 있습니다.

- 액티비티가 완료될 때 -> `onDestroy`
- 프래그먼트가 분리될 때
- 네비게이션 백 스택에서 삭제될 때

![diagram04_ViewModel](https://gist.github.com/user-attachments/assets/3aefaaa5-bb11-4d22-8ded-db26730d456d)

그런데 여기서 궁금한 점이 생겼습니다. 대표적인 구성 변경인 화면 전환이 발생하면 분명 액티비티는 `onDestroy` 후에 `onCreate` 으로 재생성을 하는 과정을 거치는데 뷰모델의 파괴 조건에 따르면 이때 뷰모델도 파괴되어야 합니다. 하지만 구성 변경이 발생하면 뷰모델은  `Clear` 되지 않습니다.

안드로이드는 구성 변경과 같이 액티비티가 잠깐 파괴되는 경우는 뷰모델을 클리어하지 않고 액티비티가 완전히 전환되거나 메모리 관리로 인한 앱 종료와 같은 완전 파괴는 클리어를 진행한다고 합니다. 그렇다면 내부에서 액티비티 종료 케이스를 어떻게 구분하는 걸까요?

​		

## Activity 관계도

이를 알기 위해서는 우선 액티비티 관련 클래스의 관계를 알아야 합니다. 내용이 빈 액티비티를 새로 생성하면 `AppCompatActivity` 을 상속받은 `MainActivity` 를 볼 수 있습니다.

<script src="https://gist.github.com/Yoon-Min/7942e99ea6ef0fed46eda9dd3fc22069.js"></script>

이 `AppCompatActivity` 클래스 내부에 들어가면 `FragmentActivity` 를 상속받은 것을 볼 수 있고 `FragmentActivity` 내부로 들어가면 `ComponentActivity` 을 상속받은 것을 볼 수 있습니다. 결국 이를 종합하면 액티비티 클래스는 아래와 같은 구조로 이루어져 있음을 알 수 있습니다.

![Group 53](https://gist.github.com/user-attachments/assets/08b6c4f9-df4b-4c1a-8d16-d7281a2c25a9)

​		

## ComponentActivity

액티비티의 가장 기본 뼈대가 되는 `ComponentActivity` 는 생명주기 관련 콜백 메서드가 정의되어 있습니다. 그래서 해당 클래스 내부에 액티비티가 `onDestroy` 됐을 때의 동작 코드를 볼 수 있습니다.

<script src="https://gist.github.com/Yoon-Min/801a5550f0428d652ef14db1e012b5c2.js"></script>

​		

## isChangingConfigurations

`init` 내의 코드를 보면 생명주기 옵저버 관련 코드를 볼 수 있습니다. 생명주기를 관찰하는 옵저버를 추가하여 특정 생명주기 단계가 발생했을 때 어떤 동작을 할 것인지 정의하는 것입니다. 그중에서 `onDestroy` 상황과 관련한 코드는 다음과 같습니다.

<script src="https://gist.github.com/Yoon-Min/993ed24c513707724e7126e1eea97c74.js"></script>

여기서 `isChangingConfigurations` 을 보면 딱 감이 옵니다. 해당 변수는 구성 변경 발생 유무에 따라 논리 값을 저장합니다. 화면 전환과 같은 구성 변경이 발생하면 `true` 로 저장되고 위의 조건문에서는 `!` 로 인해 `false` 로 바뀝니다. 따라서 뷰모델 저장소 내 뷰모델의 클리어를 막을 수 있습니다.

반대로 화면 전환과 같은 구성 변경에 의한 `onDestroy` 가 아니라면 `isChangingConfigurations` 은 `false` 가 되고 조건문에서는 `!` 로 인해  `true` 로 적용되어 `viewModelStore.clear()` 가 실행됩니다.

<script src="https://gist.github.com/Yoon-Min/c5ec04439631b9c7ebf437991a31271d.js"></script>

​		

## 정리

결국 액티비티의 `onDestroy` 가 발생하면 `isChangingConfigurations` 을 통해 액티비티의 파괴가 구성 변환에 의한 것인지 아닌지 판별을 하는 과정 덕분에 내부에서 액티비티 종료 케이스를 구분할 수 있습니다. 그리고 이러한 결과를 가지고 `ViewModel` 의 `clear` 작업을 결정합니다.



## 참조

[**Android Architecture Components: Lifecycle and LiveModel**](https://code.tutsplus.com/android-architecture-components-lifecycle-and-livemodel--cms-29275t)

[**Android Developers - ViewModel 개요**](https://developer.android.com/topic/libraries/architecture/viewmodel?hl=ko)



