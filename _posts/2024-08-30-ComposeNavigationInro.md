---
title: Android Compose Navigation 01 - 컴포즈 네비게이션 소개 
author: yoonmin
date: 2024-08-30 00:00:00 +0900
categories: [Android, 라이브러리]
tags: [Android, Compose, Navigation]
render_with_liquid: true
---

![katie-drazdauskaite-xKFVBD4UAQA-unsplash](https://gist.github.com/user-attachments/assets/fe06427a-bba4-4d35-9dfd-f344ceef4022)_[Unsplash](https://unsplash.com/ko/사진/자동차-내부지도-차트의-매크로-사진-xKFVBD4UAQA?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)의[Katie Drazdauskaite](https://unsplash.com/ko/@kotrad?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)_

​		

**Compose Navigation 포스팅 현황**

+ [x] <span style="color: #2dc39e">**Android Compose Navigation 01 - 컴포즈 네비게이션 소개**</span>
+ [ ] <span style="color: #898989">Android Compose Navigation 02 - NavGraph 생성과정 (예정)</span>
+ [ ] <span style="color: #898989">Android Compose Navigation 03 - Navigator (예정)</span>

​		

## Intro

화면 이동은 모바일 애플리케이션에서 중요한 요소입니다. 당장 우리가 일상에서 사용하는 앱을 살펴봐도 단순히 하나의 화면만 있는 것이 아닌 여러 화면을 통해 사용자에게 다양한 서비스를 제공하는 것을 볼 수 있습니다. 모바일 운영체제중 하나인 Android도 마찬가지입니다. 

그래서 Android는 이러한 화면 이동 구현을 지원하는 네비게이션 라이브러리를 지원하는데, 제가 이번 글에서 소개할 것은 **컴포즈 네비게이션**입니다. 몇 년 전부터 Android가 강하게 밀고 있는 UI 제작 킷인 컴포즈에서 네비게이션은 어떤 구조로 이루어져 있고 각각 어떤 역할을 하는지 간략하게 정리하고자 합니다.

네비게이션을 공부하면서 해당 라이브러리의 구조가 생각보다 깊고 복잡한 것을 느꼈기 때문에 컴포즈 네비게이션 포스팅은 단편이 아닌 여러 편으로 구성된 시리즈로 진행합니다. 그래서 이번 첫 번째 포스팅 이후에도 계속해서 글을 올릴 예정이니 필요하면 다음 편도 읽어보는 것도 좋을 것 같습니다.

​		

## 여행으로 보는 네비게이션

![drawkit-transport-scene-3](https://gist.github.com/user-attachments/assets/d0333a4c-d993-460c-93c5-b0ddbef4db54)

장소(목적지) 형성

: 네비게이션은 여행과도 같습니다. 가고 싶은 장소들을 정하고 일정에 따라 원하는 목적지로 이동하여 관광을 즐깁니다. 하루에 3곳의 장소를 방문한다고 가정하면 동선을 생각하여 가장 가까운 곳부터 가장 먼곳까지 차례대로 방문할 수 있습니다. 그리고 돌아올 땐 처음에 방문했던 장소들을 마주하며 지나칩니다.

이동 수단

: 한 장소에서 다른 장소로 옮길 때 어떻게 이동할 것인지도 여행의 한 요소입니다. 한 장소에서 다른 장소로 이동하는 과정은 다양한데 당장 생각나는 것들을 나열하면 다음과 같습니다.

- 기름값을 나눠 내고 일행중 자차가 있는 사람의 차를 이용
- 자동차 렌트
- 지하철, 버스, 택시와 같은 대중교통을 이용

가이드(플래너)

: 여행을 가게 되면 일행중 누군가는 여행 계획을 세웁니다. 이 사람은 여행 일정동안 가이드 역할을 진행하며 일행들의 상황을 파악하며 목적지 이동이나 목적지에서의 관광 과정에서 중간 소통자 역할을 합니다.







