---
title: Android - 내부 동작으로 살펴보는 LiveData 값 설정부터 전달까지 과정
author: yoonmin
date: 2024-08-18 00:00:00 +0900
categories: [Android, 라이브러리]
tags: [Android, LiveData, UI]
render_with_liquid: true
---

![jr-korpa-vYEzpILoNAg-unsplash](https://gist.github.com/user-attachments/assets/297fdf75-a69b-409f-9198-fb71faa96aa0)_사진: [Unsplash](https://unsplash.com/ko/사진/파란색-분홍색-노란색-주황색의-말의-추상-그림-vYEzpILoNAg?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)의[Jr Korpa](https://unsplash.com/ko/@jrkorpa?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)_

## Intro

안드로이드의 `LiveData` 라이브러리를 사용하다 보면 라이브 데이터를 구독하고 있는 컴포넌트가 어떻게 값 변경에 대한 알림을 받는지 궁금할 수 있습니다. 저도 해당 라이브러리를 사용하면서 이 부분이 궁금했습니다. 그래서 라이브 데이터의 내부 코드를 분석하여 이를 정리하고자 합니다.

​		

## Observe

<script src="https://gist.github.com/Yoon-Min/8423acf0c2f974d429b7d38766c82c00.js"></script>

라이브 데이터는 보통 `ViewModel` 에서 생성 및 업데이트를 관리합니다. 그리고 이러한 변경사항을 라이브 데이터의 구독자인 액티비티 혹은 프래그먼트가 전달받습니다. 이러한 알림 수신이 가능한 이유는 라이브 데이터가 가지고 있는 `observe` 함수 덕분입니다.

<script src="https://gist.github.com/Yoon-Min/c14db4777e02bf21ed32c67bb105c172.js"></script>

라이브 데이터를 구독할 수 있는 `observe` 메서드는 라이브 데이터 클래스 내 옵저버 맵에 구독자 정보를 저장합니다. 좀 더 자세히 설명하자면 구독자가 되는 액티비티와 액티비티에서 정의한 콜백 메서드 정보를 `mObservers` 에 저장하여 알림을 보낼 준비를 하는 것입니다.

​		

## ObserverWrapper

<script src="https://gist.github.com/Yoon-Min/75e77f1d654b3e0b56fdbcb4c2888d88.js"></script>



​		

## setValue

<script src="https://gist.github.com/Yoon-Min/29565ffa35911e2eb79961f017ae9d2e.js"></script>

