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



