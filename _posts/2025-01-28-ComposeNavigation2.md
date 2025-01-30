---
title: Android Compose Navigation 02 - NavGraph 생성과정 
author: yoonmin
date: 2025-01-28 00:00:00 +0900
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

지난 포스팅에서 Compose Navigation에 대해 간단하게 소개하는 시간을 가졌습니다. 이를 통해 `NavHost` 내부에 `NavController`와 `NavGraph`를 가지고 네비게이션을 생성하는 것을 알았습니다. 이번 포스팅은 좀 더 들어가서 `NavHost` 내부에 생성되는 `NavGraph`에 대해 알아보겠습니다.

​		

## NavHost 내 NavGraph 생성

### 그래프 생성 과정

<script src="https://gist.github.com/Yoon-Min/ad976f50fcf53a113d975573b37919cc.js"></script>

1편에서 봤듯이 `NavHost` 는 최상위 컴포저블에 정의합니다. 그 과정에서 `NavController` 와 `NavGraph` 를 내부에서 사용하는데 전자는 인자로 받지만 후자는 인자가 아닌 내부에서 직접 생성합니다.

<script src="https://gist.github.com/Yoon-Min/ae6e72b8b12ba433183abf33a66b451d.js"></script>

컨트롤러가 `createGraph` 메서드를 호출하면서 그래프를 생성하는데, 해당 메서드 내부를 보면 컨트롤러가 가지고 있는 네비게이터 객체를 통해 그래프 생성을 하는 것을 확인할 수 있습니다. 여기서 네비게이터 객체에 해당하는 `navigatorProvider` 은 `NavigatorProvider` 클래스를 인스턴스화한 것입니다.

<script src="https://gist.github.com/Yoon-Min/d684c2d694c8c4fb6c04703b2fab0818.js"></script>

네비게이터 프로바이더 객체를 가져온 후,  `navigation` 메서드를 호출하여 그래프를 생성하게 됩니다. 그래프 생성은 프로바이더가 직접 하는 것이 아닌 그래프 빌더 객체가 진행합니다. 그래서 네비게이터가 그래프 빌더에게 그래프 생성을 맡기면서 본인을 인자로 전달합니다.

​		

### 그래프 생성 흐름

![제목 없는 다이어그램 drawio](https://gist.github.com/user-attachments/assets/dcf65f53-46c4-497e-8112-f6ae10b61fa1)

​		

## NavGraphBuilder

### NavGraphBuilder

<script src="https://gist.github.com/Yoon-Min/3a7cb08b459e36aa700b833870139633.js"></script>

`NavGraphBuilder` 의 클래스 구조입니다. 우선 생성자로 출발지 정보와 `NavigatorProvider` 을 전달합니다. 이것들을 통해 그래프 빌더를 생성합니다. 그리고 나머지 멤버 메서드를 살펴보면 생각보다 복잡하지 않습니다. 각 목적지들을 저장하는 가변리스트 하나, 목적지를 추가하는 메서드, 빌더 패턴에 필수적인 `build` 메서드 이렇게 세 가지 요소가 있습니다.

​		

### apply



### build



