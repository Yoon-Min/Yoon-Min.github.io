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

라이브 데이터 내에 저장되는 옵저버는 옵저버 클래스 자체로 저장되지 않습니다. 옵저버 객체에 `ObserverWrapper` 추상 클래스 기반의 `LifecycleBoundObserver` 클래스로 감싸서 저장합니다.

먼저 `ObservrWrapper` 는 말그대로 옵저버와 옵저버의 상태 정보를 저장하는 래핑 클래스입니다. 이러한 정보에 생명주기 관련 정보를 추가(LifecycleBoundObserver)하는 방식으로 생명주기 관찰에 기반한 알림 전달을 진행합니다.

![제목 없는 다이어그램 drawio](https://gist.github.com/user-attachments/assets/0b0c1876-b215-4fc7-ad8a-eb3517083d5e)

​		

## setValue

<script src="https://gist.github.com/Yoon-Min/29565ffa35911e2eb79961f017ae9d2e.js"></script>

이제 라이브 데이터 값을 설정하는 메서드를 살펴 보겠습니다. 우선 `setValue` 입니다. 해당 메서드는 메인 스레드에서 라이브 데이터의 값을 설정한다는 특징을 가지고 있습니다.

그래서 메서드 첫 번째 라인에 `assertMainThrad` 를 통해 메인 스레드에서의 실행을 강제합니다. 이후에는 `mData` 에 설정하려는 값을 저장하여 `dispatchingValue` 를 통해 `mData` 를 옵저버에게 알립니다. 라이브 데이터에 값을 설정하면 내부에서는 `mData` 라는 변수에 값을 저장하여 관리합니다.

​		

## postValue

<script src="https://gist.github.com/Yoon-Min/1d31bdf52c3bc7e101f26bada17876b2.js"></script>

이 메서드는 `setValue` 와 마찬가지로 라이브 데이터의 값을 설정하는 특징을 가집니다. 그러나 차이점이라 하면 백그라운드 스레드에서 값을 변경하여 메인 스레드에서 실제 적용하는 과정을 거칩니다.

기본적으로 라이브 데이터는 메인 스레드에서 값을 변경할 수 있습니다. 그런데 상황에 따라 백그라운드 환경에서 값을 설정해야 하는 경우도 있습니다. 그래서 `postValue` 메서드를 지원하여 백그라운드 환경에서 값을 변경하고 이를 메인 스레드에게 전달해 최종 적용하는 것입니다.

백그라운드에서 값을 변경하기 위해서는 동기화 문제를 신경써야 합니다. 그래서 `synchronized` 를 통해 `mData` 가 아닌 `mPendingData` 에 임시로 값을 저장합니다. 임시로 값을 저장했으면 그 다음은 메인 스레드에게 `post` 합니다.

<script src="https://gist.github.com/Yoon-Min/d82b8d40ac48fa3fb8cac647aef5bd2c.js"></script>

메인 스레드에 전달하는 `Runnable` 객체는 임시로 저장한 값을 `setValue` 메서드를 호출하여 설정한 값을 실제 적용하는 작업을 진행합니다. 이를 통해 결국 `postValue` 는 최종적으로 `setValue` 를 호출한다는 것을 알 수 있습니다.

​		

## dispatchingValue

<script src="https://gist.github.com/Yoon-Min/c49c4a9e03c541a0de155955687285be.js"></script>

`setValue` 에서 마지막에 호출하는 이 메서드는 라이브 데이터 내 옵저버 맵에 저장되어 있는 옵저버에 알림(값이 변경됨)을 보냅니다. 이 알림은 `considerNotify` 에 위임합니다.

여기서 단일(특정) 옵저버에 알림을 보낼 것인지, 모든 옵저버에 알림을 보낼 것인지 정해야 합니다. 만약 특정 옵저버에게만 알림을 보내고 싶다면 파라미터인 `initiator` 에 옵저버를 기입하고, 반대로 모든 옵저버에 알림을 보내고 싶다면 `null` 로 기입합니다.

`setValue` 는 `null` 로 설정하기 때문에 해당 라이브 데이터를 관찰하고 있는 모든 옵저버에게 알림을 전송하게 되는 것입니다. 제가 설명하는 예제에서는 액티비티 하나만 라이브 데이터를 관찰하고 있기 때문에 이 액티비티에만 알림이 전송됩니다. 그러나 액티비티 이외에 다른 컴포넌트에서도 라이브 데이터를 관찰하고 있다면 해당 컴포넌트에도 알림이 전송됩니다.

​		

## considerNotify

<script src="https://gist.github.com/Yoon-Min/8c8ee54aa432d7140e6060eafe837a45.js"></script>

이 메서드에서 최종적으로 옵저버에 알림을 보냅니다. 먼저 옵저버가 활성화 상태인지 확인하는 작업을 거친 후에 마지막에 옵저버(액티비티)가 정의했던 `onChanged` 를 호출합니다.

<script src="https://gist.github.com/Yoon-Min/8423acf0c2f974d429b7d38766c82c00.js"></script>

라이브 데이터 내 `onChanged` 호출로 인해서 제가 액티비티에 정의했던 `Timber.d("result : $it")` 코드 블럭이 실행됩니다. 





