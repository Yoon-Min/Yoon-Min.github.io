---
title: Android Navigation에 대해서 알아보자
author: yoonmin
date: 2023-05-05 00:00:00 +0900
categories: [Android, 라이브러리]
tags: [Android, UI, 레이아웃, 네비게이션, JetPack]
render_with_liquid: false
---

## Jetpack에서 제공하는 Navigation

내비게이션(Navigation)은 안드로이드 Jetpack에서 제공하는 라이브러리 중 하나입니다. 내비게이션은 말 그대로 탐색을 지원하는 라이브러리며 기존의 방식보다 <span style="color: #30aaa0">**화면 전환을 쉽고 빠르고 유연하게 도와줍니다.**</span> 아마 안드로이드 앱 개발을 시작한 지 얼마 안 됐다면 꼭 알아야 하는 라이브러리 중 하나이지 않을까 싶습니다. 그래서 이번 포스팅은 내비게이션 라이브러리가 무엇이고 왜 사용하며 어떻게 사용하는지 알아보겠습니다.

​		

## 기존의 화면 전환 방식

안드로이드에서 UI를 제공할 수 있는 창은 `Activity` 와 이에 종속되어서 부분 UI를 나타내는 `Fragment` 입니다. 그래서 안드로이드 앱 내에서 화면 전환이라 하면 보통 다음과 같이 세 가지가 있습니다.

- `Activity` to `Activity`
- `Fragment` to `Fragment`
- `Activity` to `Fragment`

액티비티와 액티비티 사이의 전환은 `Intent` 객체를 이용해서 구현합니다. 그리고 프래그먼트로의 이동은 `fragmentManager` 를 이용해서 `beginTransaction` 을 통해 구현하게 됩니다. 앱 프로젝트 규모가 작다면 상관없지만 규모가 큰 프로젝트라면 사용해야 될 화면도 많아질 것이고 복잡한 화면 전환 구성을 가지게 될 가능성이 높습니다.

화면만 전환하면 끝일까요? 화면 전환에 데이터를 담아서 전송할 수도 있고 화면 전환 애니메이션도 적용할 수도 있습니다. 따라서 <span style="color: #30aaa0">**화면 전환은 많은 작업을 필요로 합니다.**</span>

​		

## 그래서 하나로 합쳐드렸습니다~

그래서 구글은 화면 전환에 필요한 라이브러리, 툴, 플러그인 등을 하나로 합쳐서 Navigation이라는 라이브러리를 만들었습니다. 기존에 인텐트 혹은 프래그먼트 매니저를 통한 화면 전환에서 내비게이션 XML 공간에서 쉽게 탐색구조를 구축하는 방식으로 전환된 것입니다. 

아래 사진과 같이 내비게이션 XML에서 기존에 가지고 있는 프래그먼트나 액티비티를 추가해서 원하는 탐색구조를 마우스로 연결해 주기만 하면 화살표가 생성되어 <span style="color: #30aaa0">**화면 전환 구조에 대한 직관적인 파악이 가능해집니다.**</span>

<img src="https://user-images.githubusercontent.com/80873132/236272903-1a0112c7-d409-4fbe-aff3-2c428a9cad97.png" alt="navigation-graph_2x-callouts 1" style="zoom:80%;" />

​		

## 쉽고 깔끔한 프래그먼트로의 전환

내비게이션 라이브러리를 사용했을 때 얻을 수 있는 장점은 <span style="color: #30aaa0">**프래그먼트 전환이 굉장히 쉬워집니다.**</span> 설명드린 대로 현재 생성한 프래그먼트들을 다 추가해서 연결하기만 하면 생성된 `id` 를 통해서 전환을 할 수 있는데 이 전환을 요청하는 코드도 굉장히 짧고 간단합니다.  그래서 이 라이브러리를 사용하면 프래그먼트를 실컷 생성해서 복잡한 탐색구조를 구축할 수 있습니다.

​		

## Single Activity

이러한 장점 때문에 구글에서는 프래그먼트 전환 관리가 쉬운 <span style="color: #30aaa0">**네비게이션을 통해 단일 액티비티 구조를 권장합니다.**</span> 액티비티를 하나만 생성하고 나머지는 전부 프래그먼트로 채우라는 소리입니다. 단일 액티비티 구조를 주장하는 이유는 다음과 같습니다.

- `Activity` 는 `Fragment` 에 비해 덩치가 큰 객체라 메모리, 속도 면에서 불리하다.
- `Fragment` 는 `Activity` 에 비해 비교적 쉬운 데이터 공유, 유연한 UI 구축의 장점을 가지고 있다.

​		

## Navigation 대표 구성요소 세 가지

### <span style="color: #30aaa0">**Navigation Graph**</span>

탐색구조를 구축할 수 있는 XML 리소스 파일입니다. 여기서 화면 전환, 애니메이션 등을 세팅할 수 있습니다.

### <span style="color: #30aaa0">**NavHost**</span>

여러 프래그먼트를 담을 수 있는 하나의  컨테이너입니다. 이 컨테이너에서 여러 프래그먼트의 전환이 일어나게 됩니다.  탐색구조를 여러 개의 그룹으로 쪼개고 싶다면 `NavHost` 를 여러 개 만들어서 구축할 수도 있습니다.

### <span style="color: #30aaa0">**NavController**</span>

각각의 `NavHost` 는 하나의  `NavController` 를 가지고 있습니다. 이 친구는 앱 내의 탐색을 관리하여 A에서 B로 전환해야 되는 상황이 발생하면 이를 인식하고 화면 전환을 수행합니다.

```kotlin
findNavController().navigate(R.id.action)
```



### 세 가지를 종합해보면?

Navigation Graph를 통해서 탐색구조를 정의를 하여 전환이 필요할 때 `NavController` 에게 요청을 합니다. 그럼 컨트롤러는 이를 수행하고 `NavHost` 는 이를 인식하여 기존 프래그먼트에서 알맞은 프래그먼트로 교체하여 화면에 표시합니다.

​		

## Navigation 사용 방법

### 0. Dependency 설정

```kotlin
dependencies {
  // 글 작성 기준으로 2.5.3 버전입니다.
  def nav_version = "2.5.3"

  implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
  implementation "androidx.navigation:navigation-ui-ktx:$nav_version"
}
```

​		

### 1. 프래그먼트를 보여줄 NavHost 지정하기

지금부터 설명드리는 예시는 액티비티에 여러 프래그먼트를 띄우는 방식입니다. 따라서 저는 액티비티 레이아웃 XML에 `FragmentContainer`를 만들어서 이 친구를 `NavHost` 로 지정하겠습니다. `android:name`  을 호스트 프래그먼트로 지정해 주고 `app:navGraph` 는 뒤에 그래프를 만들면 그때 추가해 주세요.

```xml
<androidx.fragment.app.FragmentContainerView
    android:id="@+id/fcv_main"
    android:name="androidx.navigation.fragment.NavHostFragment"
    android:layout_width="0dp"
    android:layout_height="0dp"
    app:defaultNavHost="true"
    app:layout_constraintBottom_toTopOf="@id/bnv_main"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent"
    app:navGraph="@navigation/nav_main" />
```

​		

### 2. Navigation Graph 정의

안드로이드 `res` 폴더 내에 `New `를 통해  `Android Resource Directory` 를 누르시면 `navigation` 디렉토리를 따로 만들 수 있습니다. 생성하고 그 안에 리소스 파일을 새로 생성해 주세요. 생성했다면 밑의 예시처럼 필요한 액티비티나 프래그먼트를 불러서 마우스로 연결해 주세요.

![Group 12](https://user-images.githubusercontent.com/80873132/236292035-f2df9b47-9294-49da-99ce-5210b38633e2.png)

![Group 8](https://user-images.githubusercontent.com/80873132/236288992-e6f524f9-aed3-4e5a-bddd-628b8f784dfb.png)

![Group 11](https://user-images.githubusercontent.com/80873132/236291859-e6011787-186e-4bbb-97d9-8b1cfe1b9740.png)

​		

### 3. navController를 통해 화면 전환 요청 코드 작성하기

그래프를 다 만들었다면 Split을 통해서 XML 코드도 확인해 보세요. 그러면 `<fragment>`  태그로 감싸진 코드가 굉장히 많아진 것을 볼 수 있습니다. 그중에서 하나만 보겠습니다. 해당 프래그먼트는 연결 지점이 한 곳입니다. 이는 `<action>`에 정의되어 있습니다. 만약에 연결한 프래그먼트가 세 곳이면 `<action>` 은 세 개가 됩니다. 

```xml
    <fragment
        android:id="@+id/nav_my_page"
        android:name="com.example.travelfeeldog.presentation.mypage.MyPageFragment"
        android:label="fragment_my_page"
        tools:layout="@layout/fragment_my_page" >
        <action
            android:id="@+id/action_nav_my_page_to_myReviewFragment"
            app:destination="@id/myReviewFragment"
            app:enterAnim="@anim/nav_default_enter_anim"
            app:exitAnim="@anim/nav_default_exit_anim"
            app:popEnterAnim="@anim/nav_default_pop_enter_anim"
            app:popExitAnim="@anim/nav_default_pop_exit_anim" />
    </fragment>
```

연결한 프래그먼트로 이동을 위해서 액션 아이디인 `android:id` 를 통해서 요청하면 됩니다.

```kotlin
binding.clReviewArea.setOnClickListener{
    findNavController().navigate(R.id.action_nav_my_page_to_myReviewFragment)
}
```

​		

## BottomNavigationView 연동

내비게이션은 바텀 내비게이션 뷰와 연결이 가능합니다. 방법은 다음과 같습니다.

### 1. 필요한 메뉴 구성

`res`  에서 `menu`  디렉토리(없으면 생성)에 바텀 내비게이션 전용 메뉴 XML 을 생성합니다. 생성했으면 해당 파일에 필요한 메뉴를 정의합니다. 저는 홈, 검색, 마이 페이지가 필요하므로 세 개를 만들었습니다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/nav_home"
        android:title="@string/navigation_home"
        android:icon="@drawable/ic_home"/>

    <item android:id="@+id/nav_search"
        android:title="@string/navigation_search"
        android:icon="@drawable/ic_search"/>

    <item android:id="@+id/nav_my_page"
        android:title="@string/navigation_my_page"
        android:icon="@drawable/ic_my_page"/>
</menu>
```

{: .prompt-danger }

> 메뉴 아이템 태그의 아이디, 네비게이션 그래프에 정의한 프래그먼트 태그의 아이디는 **동일해야 한다.**

​		

### 2. BottomNavigationView 생성

액티비티 레이아웃 XML에 바텀 내비게이션 뷰를 생성합니다. 생성했으면 속성으로 `app:menu` 를 추가하여 만든 메뉴 파일을 연결합니다.

```xml
<com.google.android.material.bottomnavigation.BottomNavigationView
    android:id="@+id/bnv_main"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:background="@color/white"
    android:paddingTop="6dp"
    app:itemIconTint="@color/selector_bottom_nav_color"
    app:itemRippleColor="@null"
    app:itemTextAppearanceActive="@style/Widget.BottomNavigationView.Active"
    app:itemTextAppearanceInactive="@style/Widget.BottomNavigationView.InActive"
    app:itemTextColor="@color/selector_bottom_nav_color"
    app:labelVisibilityMode="labeled"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toBottomOf="@id/fcv_main"
    app:menu="@menu/menu_bottom" />
```

​		

### 3. BottomNavigationView 호스트에 NavController 연결

마무리로 바텀 내비게이션 뷰를 가지고 있는 액티비티에 컨트롤러를 연결해야 합니다. 해당 코드는 호스트가 되는 액티비티에서 일부를 가져왔습니다.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(binding.root)

    binding.bnvMain.setupWithNavController(findNavController())
}

private fun findNavController(): NavController {
    val navHostFragment =
        supportFragmentManager.findFragmentById(R.id.fcv_main) as NavHostFragment

    return navHostFragment.navController
}
```

​		

## 마무리

안 중요한 라이브러리는 없지만 내비게이션 라이브러리는그중에서도 유용하고 정말 중요한 것 같습니다. 제가 첫 안드로이드 프로젝트를 했을 때 내비게이션의 존재를 모르고 수작업으로 전환 코드를 작성해서 엄청 고생했던 기억이 있습니다. 처음 시작하는 분들은 저처럼 삽질하지 말고 내비게이션으로 편한 프로젝트 되시길 바랍니다 :)



## 참조

[<span style="color: #30aaa0">**Android 공식 문서 - Navigation**</span>](https://developer.android.com/guide/navigation)

[<span style="color: #30aaa0">**Android 공식 문서 - Navigation 시작하기**</span>](https://developer.android.com/guide/navigation/navigation-getting-started)