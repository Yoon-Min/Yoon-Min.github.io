---
title: Android - ViewModel 생성과 반환
author: yoonmin
date: 2024-07-19 00:00:00 +0900
categories: [Android, 라이브러리]
tags: [Android, 뷰모델, ViewModel]
render_with_liquid: true

---

![1](https://gist.github.com/user-attachments/assets/26c57e61-2115-41d8-9c62-1188f7504493)

## Intro

`ViewModel` 은 안드로이드의 `UI` 층을 구현하는 데 많은 도움을 주는 Jetpack 라이브러리입니다. 화면 구성의 변경이나 여러 프래그먼트에서 `UI` 상태 데이터를 보존하고 공유할 수 있다는 장점 덕분에 많은 사람들이 사용하고 있고, 저 역시 뷰모델을 애용하고 있습니다.

그러나 지금까지 뷰모델의 사용 방법과 역할만 숙지한 탓에 최근 내부 동작이 어떻게 되는지에 대한 이해가 매우 부족한 것을 느꼈습니다. 현재 계획하고 있는 프로젝트가 두 개이고 해당 프로젝트를 앞으로 제대로 수행하려면 사용하는 라이브러리에 대한 이해가 필요할 것 같아 이번 기회에 정리하고자 합니다.

​		

## ViewModel 관련 관계도

![Group 51](https://gist.github.com/user-attachments/assets/9db58b67-2ce9-40be-984a-e69a7eb76bd6)

먼저 뷰모델 생성 및 관리에 관여하는 객체들의 관계도는 위 그림과 같습니다. 뷰모델을 생성할 때는 바로 뷰모델 클래스를 통해 객체를 `create` 하는 것이 아니라 프로바이더를 통해 뷰모델 객체를 공급받습니다. 프로바이더는 뷰모델의 생성과 관리를 위한 객체들 - `Store` , `Factory` 을 가지고 있습니다.

1. 프로바이더 호출
2. 프로바이더 내부 객체들을 이용해 뷰모델을 생성하거나 기존 객체 반환
   - `ViewModelStore` - 뷰모델 객체가 보관되는 장소
   - `ViewModelProvider.Factory` - 뷰모델 객체를 생성하는 역할

​		

### ViewModelProvider

<script src="https://gist.github.com/Yoon-Min/42eb070732c6d7cb0458abeb7f540f58.js"></script>

프로바이더는  `StoreOwner`  와 `Factory` 을 가지고 있고 내부에는 프로바이더 객체 생성, 뷰모델 로딩, 팩토리 인터페이스 등의 코드가 정의되어 있습니다. 즉, 프로바이더 객체를 생성( `create` )하고 `get` 을 통해 뷰모델을 가져오는 것이 주가 되겠습니다.

- **생성자 구성**

  - `ViewModelStoreOwner` - `ViewModelStore` 보유

  - `ViewModelStore` - 뷰모델 객체가 저장되는 장소

  - `Factory` - 뷰모델 객체를 생성하는 팩토리 (프로바이더 내부에 인터페이스 정의)

- **`get`** - 뷰모델을 가져옵니다.

- **`create`** - 프로바이더 객체를 생성합니다.

​		

### ViewModelStoreOwner

<script src="https://gist.github.com/Yoon-Min/70057bbed38074a99e710d40b99643a2.js"></script>

`ViewModelStoreOwner` 은 이름 그대로 `ViewModelStore` 을 가지고 있습니다. 해당 인터페이스의 `owner` 는 액티비티 혹은 프래그먼트이기 때문에 이들 클래스 내부에 `viewModelStore` 가 멤버변수로 자리합니다. 

 액티비티 클래스의 첫 번째 뼈대인 `ComponentActivity` 클래스를 보면 `ViewModelStoreOwner` 인터페이스를 받아서 구현한 코드를 확인할 수 있습니다.

<script src="https://gist.github.com/Yoon-Min/86f0df688136265010e9d0f532ada310.js"></script>

![Group 53](https://gist.github.com/user-attachments/assets/c1bcd4ac-bde4-4128-bfcf-fd8757a43302)

​		

### ViewModelStore

<script src="https://gist.github.com/Yoon-Min/c5ec04439631b9c7ebf437991a31271d.js"></script>

뷰모델 저장소인 스토어는 `Map` 으로 관리합니다. 키를 등록하기 위한 `put` , 뷰모델을 꺼내기 위한 `get` , 맵을 정리하기 위한 `clear` 가 멤버 메서드로 자리하고 있습니다. 새로 생성된 뷰모델 정보는 `put` 으로 맵에 등록되며 등록된 뷰모델을 꺼낼 때는 `get` 을 사용합니다.

​		

### ViewModelProvider.Factory

<script src="https://gist.github.com/Yoon-Min/d2cea670c9ec6bb8be4bd1adcd6a417d.js"></script>

뷰모델 프로바이더 내부에 인터페이스로 자리하고 있는 `Factory` 는 `ViewModelProvider` 내부의 `get` 메서드 호출을 통해 뷰모델을 가져오는 작업이 필요할 때 사용됩니다.

뷰모델을 가져오는 작업을 한다면 첫 번째로 `ViewModelStore` 에 저장되어 있는 뷰모델인지 먼저 확인해야 합니다. 스토어에 저장되어 있는 뷰모델이라면 굳이 생성할 필요 없이 꺼내 쓰면 됩니다. 반대로 저장되지 않은 뷰모델이라면 새로 생성하는 과정을 거쳐야 하는데 여기서 `Factory` 가 `create` 역할을 수행합니다.

팩토리에서 생성된 뷰모델은 반환되기 전에 스토어에 정보를 저장하는 작업을 거칩니다. 그래서 다음에 호출되면 새로 생성하는 것이 아닌 스토어에서 바로 반환합니다.

​		

## ViewModel 호출 흐름이 어떻게 되는지 살펴보기

{: .prompt-tip}

>
> 실행 환경 :  액티비티 내 ViewModel을 호출



### 1. Activity에서 ViewModel 변수 설정

<script src="https://gist.github.com/Yoon-Min/e3e4929549ca8914377b3ef1d0bba3e9.js"></script>

다음과 같이 뷰모델을 액티비티에서 사용한다고 가정하겠습니다. `viewModels()` 에 뷰모델 로딩을 위임하여 제가 임의로 정의한 `ViewModel` 을 불러옵니다.

​		

### 2. viewModels()

<script src="https://gist.github.com/Yoon-Min/91f06a8b35b01724d4165507d97e2fe3.js"></script>

`viewModels()` 내부는 `ViewModelLazy` 의 반환값을 반환하는 코드로 정의되어 있습니다. 반환 함수는 필요한 뷰모델의 클래스 정보, `ViewModelStore` , `ViewModelProvider.Factory` , `Extras` 을 파라미터로 가집니다.

팩토리의 경우는 액티비티에서 사용할 팩토리를 명시하지 않았으면 `defaultViewModelProviderFactory` 을 사용합니다. 스토어는 액티비티가 가지고 있는 스토어를 사용합니다. `Extras` 는 팩토리와 마찬가지로 확인해서 `null` 이면 기본 값을 사용합니다.

​		

### 3. ViewModelLazy

<script src="https://gist.github.com/Yoon-Min/8bf1ef9a5ad200edc78d519a4d6120ae.js"></script>

여기서는 캐싱 기법을 사용합니다. `cached` 변수에 이전에 사용했던 뷰모델 정보를 저장해서 재사용에 대비합니다. 만약 저장된 뷰모델이 없다면 이때부터는 `ViewModelProvider` 에게 뷰모델을 요청하는 과정을 거칩니다.

위 설명대로 프로바이더 객체를 `create` 로 생성하고 `get` 을 통해 원하는 뷰모델을 요청합니다. 뷰모델을 불러오는 데 성공했다면 `cached` 변수에 뷰모델 정보를 저장합니다.

​		

### 4. ViewModelProvider

<script src="https://gist.github.com/Yoon-Min/e93535fdba483955601fb9ed9e13e0f4.js"></script>

프로바이더 내 `create` 를 호출하면 생성자로 `ViewModelProviderImpl` 이 생성되고 `create` 다음에 호출하는 `get` 은 `ViewModelProviderImpl` 을 통해 뷰모델을 가져오는 작업을 수행합니다.

`get` 을 실행하면 `Impl` 내 `getViewModel` 함수가 호출되는데 여기서는 스토어에 뷰모델이 저장되어 있는지 확인하고 있다면 반환하고 없다면 새로 생성하고 반환합니다. 

​		

### 5. createViewModel

<script src="https://gist.github.com/Yoon-Min/15f6f2203f1c72678dc833ac923da837.js"></script>

스토어에 뷰모델 정보가 존재하지 않아 새로 만들어야 하는 상황이라면 팩토리에게 생성을 요청합니다. 팩토리에 의해 뷰모델이 생성되면 마지막에 스토어에 저장하고 반환합니다. 이를 통해 최종적으로 액티비티에 뷰모델이 전달됩니다. 

​		

## 참조

[**Android Overview**](https://developer.android.com/topic/libraries/architecture/viewmodel?hl=ko)

[**ViewModel이란 무엇인가? ViewModel 초보를 위한 가이드**](https://charlezz.medium.com/viewmodel%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80-viewmodel-%EC%B4%88%EB%B3%B4%EB%A5%BC-%EC%9C%84%ED%95%9C-%EA%B0%80%EC%9D%B4%EB%93%9C-e1be5dc1ac18)

[**ViewModel’s Internal working in Android. A full guide**](https://medium.com/@KaushalVasava/viewmodels-internal-working-in-android-a-full-guide-757afaf6510)

[**Deep dive inside of Android’s ViewModel (Architecture Components)**](https://medium.com/android-news/deep-dive-inside-of-androids-viewmodel-architecture-components-e6756dc0bb11)















