---
title: Android ViewModel 생성 과정
author: yoonmin
date: 2024-07-19 00:00:00 +0900
categories: [Android, 라이브러리]
tags: [Android, 뷰모델, ViewModel]
render_with_liquid: true

---

![1](https://gist.github.com/user-attachments/assets/26c57e61-2115-41d8-9c62-1188f7504493)

## Intro

`ViewModel` 은 안드로이드의 `UI` 층을 구현하는 데 많은 도움을 주는 Jetpack 라이브러리입니다. 화면 구성의 변경이나 여러 프래그먼트에서 `UI` 상태 데이터를 보존하고 공유할 수 있다는 장점 덕분에 많은 사람들이 사용하고 있고, 저 역시 뷰모델을 애용하고 있습니다.

그러나 지금까지 뷰모델의 사용 방법과 역할만 숙지한 탓에 최근 내부 동작이 어떻게 되는지에 대한 이해가 많이 부족한 것을 느꼈습니다. 현재 계획하고 있는 프로젝트가 두 개이고 해당 프로젝트를 앞으로 제대로 수행하려면 사용하는 라이브러리에 대한 이해가 필요할 것 같아 이번 기회에 정리하고자 합니다.