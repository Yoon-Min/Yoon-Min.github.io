---
title: Android - LiveData의 생명주기(Lifecycle) 인식 원리
author: yoonmin
date: 2024-07-31 00:00:00 +0900
categories: [Android, 라이브러리]
tags: [Android, LiveData, UI]
render_with_liquid: true
---

![nitish-meena-s7rU_y27PJU-unsplash 1](https://gist.github.com/user-attachments/assets/d67a3b3e-1bfe-4332-832b-4078d13a608b)_[Unsplash](https://unsplash.com/ko/사진/흰-꽃잎이-달린-꽃의-근접-촬영-사진-s7rU_y27PJU?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)의[Nitish Meena](https://unsplash.com/ko/@nitishm?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)_

## Intro

`LiveData` 는 관찰 가능한 UI 데이터 홀더 클래스입니다. 라이브 데이터는 안드로이드의 수명주기를 인식하여 수명주기에 맞춰 동작한다는 특징을 가지고 있어서 UI Layer에서 UI 데이터를 보유하는 역할을 수행합니다.

그런데 여기서 한 가지 궁금증이 생깁니다. 라이브 데이터가 안드로이드의 생명주기에 맞춰서 동작한다는 것은 알겠는데 어떻게 인식을 하는 걸까요? 이번 포스팅은 그 원리를 알아보는 시간을 가지고자 합니다.

​		

## Lifecycle

라이브 데이터를 분석하기 전에 먼저 생명주기에 대해서 알 필요가 있습니다. 보통 안드로이드에서 생명주기라 하면 액티비티 혹은 프래그먼트의 생명주기를 떠올릴 것입니다. 

그 생명주기의 근간이 되는 클래스가 `Lifecycle` 입니다. 액티비티 혹은 프래그먼트와 같은 구성요소의 생명주기 상태 관련 정보를 포함하며 **다른 객체가 이 상태를 관찰할 수 있게 하는 클래스**입니다.

> *"Defines an object that has an Android Lifecycle. Fragment and FragmentActivity classes implement LifecycleOwner interface which has the LifecycleOwner. getLifecycle method to access the Lifecycle. You can also implement LifecycleOwner in your own classes."*
>
> *"Android Lifecycle을 가진 개체를 정의합니다. Fragment 및 FragmentActivity 클래스는 Lifecycle Owner를 가진 Lifecycle Owner 인터페이스를 구현합니다. Lifecycle 메서드를 가져와 Lifecycle에 액세스할 수 있습니다. 자신의 클래스에서 Lifecycle Owner를 구현할 수도 있습니다."*
>
> Android Developers

​		

### LifecycleOwner

`LifecycleOwner` 는 `Lifecycle` 을 가지고 있는 인터페이스입니다. 뷰모델 저장소인 `ViewModelStore` 을 `ViewModelStoreOwner` 인터페이스가 가지고 있는 것과 동일한 방식입니다. 이 뷰모델 저장소의 오너는 액티비티 혹은 프래그먼트가 됩니다.

생명주기 오너도 마찬가지로 액티비티 혹은 프래그먼트가 `Lifecycle` 의 `Owner` 가 되어 내부의 `Lifecycle` 클래스를 통해 생명주기를 관리합니다.

<script src="https://gist.github.com/Yoon-Min/dc1eba4dba3d4601c8b9ee262ec10b2a.js"></script>

액티비티의 첫 번째 뼈대가 되는 컴포넌트 액티비티와 프래그먼트에서 구현 인터페이스 목록을 보면 `LifecycleOwner` 가 있는 것을 볼 수 있습니다. 액티비티와 프래그먼트가 생명주기의 주인이 되어 `LifecycleOwner` 내부의 `lifecycle` 을 구현합니다.

<script src="https://gist.github.com/Yoon-Min/5cac59b3039185a3b5ddf627d4f5570b.js"></script>

`lifecycleRegistry` 는 추상 클래스인 `Lifecycle` 의 하위(상속)클래스입니다. `Lifecycle` 은 생명주기의 기본적인 상태와 이벤트를 정의한 클래스라면 레지스트리는 생명주기를 관찰하여 처리하는 방법들을 정의한 클래스라 할 수 있습니다.

​		

### LifeCycle

`LifeCycle` 객체는 `Event` , `State` 의 두 가지 열거형 클래스를 관리합니다. 생명주기에 대한 공부를 했다면 매우 익숙한 키워드일 것입니다.

<script src="https://gist.github.com/Yoon-Min/ff60878fabc630caf0b00a38d706cf52.js"></script>

이벤트와 상태가 어떤 느낌인지 정확히 알려면 아래의 Android 수명주기를 인식하는 상태와 이벤트표를 참고하세요. 이 표를 보면 상태와 이벤트가 어떻게 연결되어 있는지 이해하는 데 도움이 될 것입니다.

아래 그림을 통해 `targtState` , `downFrom` , `downTo` , `upFrom` , `upTo` 내부 코드는 충분히 이해가 가능합니다. 그리고 `State` 의 `isAtLeast` 는 `State` 열거 클래스 내 요소들의 순서 번호를 비교하여 최소 `state` 인자 이상의 상태인지 확인합니다.

예를 들어서 `isAtLeast` 의 인자로 `STARTED` 가 전달되면 현재 상태가 최소한 `STARTED` 이상의 상태인지 확인한다고 이해하면 될 것 같습니다.

![Group 54](https://gist.github.com/user-attachments/assets/29bffb38-0078-4a2c-ac8e-4d3a61d05fc9)

​		

### LifecycleRegistry

마지막으로  `Lifecycle` 의 `Owner` 가 가지고 있는 `Lifecycle` 객체를 보겠습니다. 가지고 있는 라이프 사이클 구현체는 위에서 설명했던 대로 `LifecycleRegistry` 가 되겠습니다.

이 구현체는 여러 관찰자(Observers)를 처리(Handle)하는 역할을 가지고 있습니다. 그리고 생명주기에 변화가 생기면 해당 정보를 관찰자들에게 전달하여 관찰자들이 바로바로 상태를 감지할 수 있도록 합니다. 이러한 점 때문에 `LiveData` 도 해당 객체에 생명주기를 관찰하는 관찰자로 들어가서 생명주기를 인식할 수 있습니다.

![Group 55](https://gist.github.com/user-attachments/assets/e56515e5-504d-462a-986b-a54da4e27722)

​		

### LifecycleRegistry 주요 메서드1 - addObserver

`LifecycleOwner`  가 되는 액티비티 혹은 프래그먼트의 상태가 변경될 때 알림을 받을 옵저버를 추가합니다.

<script src="https://gist.github.com/Yoon-Min/07b4b8e0055039f9faea9a719ae2ce49.js"></script>

1. 메인스레드에서 동작되는지 확인 (해당 동작은 메인스레드에서 동작되어야 함)
2. 현재 상태에 따라 초기 상태를 설정하고 추가하려는 옵저버가 이미 존재하는지 확인
3. 이미 존재하는 옵저버면 동기화 작업이 필요하지 않으므로 종료
4. `observerMap` 에 옵저버를 추가하고 현재 생명주기 상태에 맞춰주는 동기화 작업을 진행
   - 한 단계씩 차례대로 상태 갱신

5. 현재 `addObserver` 가 재진입 상태면 `sync()` 작업을 진행하지 스택의 `Top` 일 때 `sync()` 작업을 진행한다. 
   - `sync()` - `LifecycleOwner` 의 상태와 모든 옵저버의 상태를 동기화하는 작업
   - `isReentrance` ? - 함수가 실행되는 도중에 같은 함수가 다시 호출(옵저버 추가 과정)될 수 있기 때문에 오류방지 차원에서 확인

​		

### LifecycleRegistry 주요 메서드2 - ObserveWithState

옵저버와 옵저버의 현재 상태를 함께 저장하기 위한 클래스입니다. 그리고 내부의 `dispatchEvent` 메서드를 통해 옵저버에게 이벤트를 전달하여 옵저버가 생명주기를 인식할 수 있게 합니다.

<script src="https://gist.github.com/Yoon-Min/96f2ebf80734ac09a5ec2ad2b9d4c8be.js"></script>

​		

### LifecycleRegistry 주요 메서드3 - calculateTargetState

옵저버의 목표 상태를 계산해서 반환합니다.

<script src="https://gist.github.com/Yoon-Min/ad1239b5428abf5d5b4cb64f169d10a1.js"></script>

​		

### LifecycleRegistry 주요 메서드4 - sync

`LifecycleOwner` 의 상태와 모든 옵저버의 상태를 동기화하는 작업을 진행합니다. 현재 등록된 옵저버들 중에서 가장 오래된 것과 가장 최신의 것을 각각 현재 상태와 비교하여 `backwarPass` 혹은 `forwarPass` 기법으로 동기화를 진행합니다.

<script src="https://gist.github.com/Yoon-Min/12a46f537f706082f8d91ae3ea32a626.js"></script>

`backwarPass` 와 `forwardPass` 의 차이는 동기화를 진행하는 방향에 있습니다. 전자는 `Event.downFrom` , 후자는 `Event.upFrom` 으로 동기화를 진행합니다.

![Group 56](https://gist.github.com/user-attachments/assets/52c8b88e-bf92-45cd-9855-f5d5e2cd917c)

​		

### LifecycleRegistry 주요 메서드5 - handleLifecycleEvent

생명주기의 변화가 생기면 내부적으로 `handleLifecycleEvent` 가 실행되고 상태값을 새로 설정하여 옵저버들에게 이를 알립니다. 현재 상태 값을 갱신하게 되면 `sync` 메서드를 통해 옵저버들에게 알림을 보냅니다.

<script src="https://gist.github.com/Yoon-Min/6e0b706c64ae1998f15804aebe74dbf8.js"></script>

