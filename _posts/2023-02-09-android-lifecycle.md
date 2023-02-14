---
title: Android 액티비티 생명주기
author: yoonmin
date: 2023-02-14 00:00:00 +0900
categories: [Android, 기본]
tags: [Android, 모바일, 안드로이드, 생명주기, 개발]
render_with_liquid: false
---

## 생명주기가 무엇인가?

```
1. 소프트웨어 개발 계획의 시작부터 끝까지 따라야 할 단계와 그것에 따른 계획의 형태 변화.
2. 소프트웨어가 만들어져서 폐기될 때까지의 기간. 
   사용자의 요구에 대한 분석 및 설계, 그리고 프로그램 작성을 거쳐 운용되는 일련의 과정을 포함한다.
```

네이버 국어사전에서 말하는 생명 주기는 다음과 같습니다. 즉, 액티비티가 만들어지고 없어질 때까지의 과정을 말합니다. 어떤 생명이 탄생해서 죽을 때까지의 과정이 존재하듯  <span style="color: #30aaa0">**안드로이드의 액티비티도 생성되어 소멸하기까지의 과정을 가지고 있습니다.**</span>

​		

## 생명주기는 왜 있는가?

그렇다면 이쯤에서 드는 생각이 있을 겁니다. 그러면 생명주기는 왜 있는 것인가? 이 질문에 대한 해답은 모바일 사용 환경을 생각해 보면 됩니다. 다음과 같은 경우를 생각해 봅시다.

---



- **사용자가 앱을 사용하는 도중에 전화가 왔을 때**
- **사용자가 앱에서 나갔다가 돌아왔을 때 이전의 데이터들이 저장되지 않는 문제**
- **세로 모드로 사용하다가 가로로 돌려서 가로 모드로 전환**
- **앱을 사용하는 도중에 홈 화면으로 이동 후, 다른 앱을 실행시킬 때**

---



네 가지 경우를 보면 모바일 사용자 환경에서는 많은 상태 변화가 발생한다는 것을 알 수 있습니다. 아마 사용자 입장에서는 위의 상황들이 발생해도 아무런 오류가 없이 매끄럽게 앱이 작동되길 바랄 겁니다. 

앱이 아무런 오류 없이 매끄럽게 동작하려면 어떻게 해야 할까요? 바로 <span style="color: #30aaa0">**다양한 상태 변화가 발생했을 때 상태에 따라 알맞은 작업을 하고 적절한 전환을 처리**</span>해야 할 것입니다. 그래서 현재 앱이 어떤 상태에 있는지를 계속 파악하는 것이 중요하고 이를 파악하기 위해서 생명주기가 있는 것입니다.

​		

## Android의 생명주기

생명주기의 시작은 탄생이고 끝은 죽음입니다. 그리고 그 사이에는 다양한 상태들이 존재합니다. 탄생 후에 본격적으로 활동을 위한 준비단계가 있을 것이며 많은 활동 끝에 죽음을 준비하기 위한 단계도 있을 것입니다. 

안드로이드에서 정의한 액티비티의 생명주기 단계들은 시작과 실행 사이에  `onCreate()`,  `onStart()`,  `onResume()`  순서로 있고 실행과 종료 사이에  `onPause()`, `onStop()`, `onDestroy()` 순으로 여러 상태들이 존재합니다. 따라서 사용자가 앱을 이용하면 그 과정에서 여러 가지의 상태 변화가 발생하는데 이는 생명주기에 따라 적절히 처리됩니다.

![생명주기 자체제작](https://user-images.githubusercontent.com/80873132/218137047-4609a769-b7da-4ad4-88f3-dbf3dee592f6.png)

​		

## 생명주기 처리를 위한 콜백 메서드

<span style="color: #30aaa0">**상황에 따라 적절한 처리를 해주려면 생명주기 내에서 해당 상황에 맞는 특정 상태를 알 수 있어야 합니다.**</span> 어떤 상태인지를 알아야 그에 맞는 처리를 해줄 수 있으니까요. 안드로이드는 이를 위해 생명주기 내의 다양한 단계들을 콜백 형태로 제공합니다. 

액티비티 클래스는 `onCreate()`,  `onStart()`, `onResume()` , `onPause()`, `onStop()`,`onDestroy()` 함수들을 상속받습니다. 상속을 받고 이 함수들을 필요에 따라 재정의 하여 특정 상황의 발생으로 인해서 호출이 됐을 때 재정의 한 내용을 토대로 처리를 하게 됩니다. 예를 들어서 가로 모드로 전환을 했을 때는 화면의 구성이 바뀌므로 수명주기의 상태는 `onCreate()` 에 해당이 되어 시스템에서 `onCreate()` 콜백을 호출합니다.

{: .prompt-tip }

> 이렇게 각 콜백은 상태 변화에 적합한 특정 작업을 실행할 수 있도록 하고 적시에 알맞은 작업과 적절한 전환을 통해 앱이 더욱 안정적으로 기능할 수 있도록 해줍니다.

​		

## 콜백 메서드 소개

지금까지 생명주기가 왜 존재하고 어떻게 처리하는지 알아봤습니다. 콜백 호출을 통해서 상태 변화에 대응한다는 것을 알았으므로 이제는 각 콜백 메서드의 역할과 특징에 대해 알아보겠습니다. 

### onCreate()

<span style="color: #30aaa0">**시스템이 액티비티를 생성할 때 실행되고 필수적으로 구현해야 하는 콜백 메서드입니다.**</span> `onCreate()` 메서드에서는 액티비티의 전체 수명 주기 동안 한 번만 발생해야 하는 기본 애플리케이션 시작 로직을 실행합니다. 예를 들어 데이터 바인딩, `ViewModel` 연결, 클래스 인스턴스화 등을 `onCreate()` 에서 처리한다고 보면 되겠습니다. 대표적으로 레이아웃 XML을 액티비티로 불러올 때 `setContentView()` 함수를 `onCreate()` 에서 호출합니다.

`onCreate()` 메서드는 `savedInstanceState` 를 매개변수로 받는데 이는 `Bundle` 객체입니다. `savedInstanceState` 는 액티비티의 이전 상태 정보들을 가지고 있습니다. 액티비티가 어떤 이유로 화면을 재구성하게 된다면 기존에 존재하던 값들이 초기화되는 것을 방지할 필요가 있을 수도 있습니다. 그래서 따로 저장을 해두면 나중에 액티비티가 다시 생성될 때 `savedInstanceState` 를 이용하여 저장했던 값들을 불러올 수 있습니다.

마지막으로 `onCreate()` 가 호출이 완료되면 시스템은 거기서 멈추지 않고 뒤이어 `onStart()` 와  `onResume()` 콜백 메서드를 호출합니다.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    binding = ActivityMainBinding.inflate(layoutInflater)
    val view = binding.root
    setContentView(view)
}
```

​		

### onStart()

<span style="color: #30aaa0">**액티비티가 사용자에게 표시되고 앱은 액티비티를 포그라운드에 보내서 상호작용을 준비합니다.**</span> 액티비티는 사용자에게 표시되지만 아직은 상호작용을 할 수 없는 단계입니다. 상호작용 준비를 위해서 이 단계에서 UI를 관리하는 코드를 초기화합니다.

 `onStart()` 는 매우 빠르게 완료되고 `onCreate()` 때와 마찬가지로 시스템은 여기서 머무르지 않고  `onResume()`  콜백 메서드를 호출하게 됩니다.

​		

### onResume()

<span style="color: #30aaa0">**액티비티가 포그라운드에 표시되고 사용자와 상호작용을 시작합니다.**</span> 중간에 전화가 와서 포커스가 전화 알람으로 이동하거나 다른 액티비티로 화면 전환을 하여 포커스가 이동한 액티비티로 전환되거나 기기 화면이 꺼져서 포커스를 잃는 등의 현재 액티비티에 대한 포커스 소실이 발생하면 앱은  `onResume()` 에서 벗어나고 그렇지 않다면 `onResume()`  상태에서 머물게 됩니다.

만약 현재 상태에서 방해 이벤트가 발생한다면 액티비티는 일시 중지 상태를 의미하는 `onPause()` 콜백 메서드를 호출해서 상태 변화를 합니다. 일시 중지 상태에서 다시 액티비티로 돌아온다면 시스템은 다시  `onResume()` 를 호출합니다.

​		

### onPause()

<span style="color: #30aaa0">**사용자가 액티비티를 떠날 때 첫 번째로 호출되는 콜백 메서드입니다.**</span>  액티비티가 포그라운드에서 벗어나는 경우 일시중지 상태로 전환됩니다. 이 일시 중지는 `onPasue()` 를 의미합니다. `onPause()` 가 호출되는 경우는 `onResume()` 에서 설명한 예시들에 해당됩니다.

액티비티가 포그라운드에 있지 않다면 일시 중지 상태가 되어 원래 사용됐던 기능들을 실행할 필요가 없어지게 됩니다. 그래서 이때는 필요에 따라 `onPause()` 메서드를 이용하여 시스템 리소스, GPS, 배터리 수명에 영향을 미칠 수 있는 리소스들을 해제할 수도 있습니다.

하지만 주의할 점이 있습니다.  `onPause()` 는 잠깐동안 실행되므로 시간이 길어질 수 있는 애플리케이션 또는 사용자 데이터 저장, 네트워크 호출, 데이터베이스 트랜잭션 등의 작업은 `onPause()` 가 끝난 후에도 완료가 안될 수도 있습니다. 그래서 공식 문서에서는 이러한 작업들은 `onPause()` 에서 하지 말고 `onStop()` 상태일 때 처리하라고 추천합니다.

액티비티가 다시 포그라운드에 복귀하면 시스템은 다시 `onResume()` 을 호출하게 됩니다. 일시 중지 상태에서 재개 상태로 돌아오면 그 과정에서 시스템은 액티비티의 인스턴스를 메모리에 보관했다가 `onResume()` 을 호출할 때 메모리에 머물고 있던 인스턴스를 호출합니다.

​		

### onStop()

<span style="color: #30aaa0">**액티비티가 사용자에게 더 이상 표시되지 않으면 `onStop()` 을 호출합니다.**</span> 기존 액티비티에서 다른 액티비티로 완전히 전환되거나 액티비티의 실행이 완료되어 종료되는 시점에 해당 콜백을 호출합니다. 예를 들어서 앱 사용 중에 홈 화면으로 이동하면 액티비티가 포그라운드에서 벗어나 화면에서 사라지므로  `onPause()` 가 실행된 후에 `onStop()` 이 호출됩니다.

`onStop()` 에서도 `onPause()` 와 마찬가지로 앱이 사용자에게 보이지 않는 동안 앱에서 필요하지 않은 리소스를 해제하거나 조정할 수 있습니다. `onPause()` 에서는 비교적 가벼운 작업을 처리했다면 여기서는 CPU를 비교적 많이 소모하는 종료 작업 실행을 추천합니다. 데이터베이스에 정보를 저장하는 작업 같은 것을 현재 상태일 때 처리할 수도 있습니다.

또한 액티비티가 중단 상태에서 재개 상태로 바뀔 때 `onPause()` 와 마찬가지로 액티비티 객체가 메모리에 머물다가 다시 시작되면 액티비티 객체 정보가 호출됩니다.

`onStop()` 상태에서는 두 가지 경우가 발생할 수 있습니다. 첫 번째는 액티비티를 다시 재개하는 것이고 두 번째는 액티비티를 종료하는 것입니다. 첫 번째 경우는 `onRestart()` 가 호출되어 다시 재개하는 과정을 거치고 두 번째는 `onDestroy()` 가 호출됩니다.

​		

### onDestroy()

<span style="color: #30aaa0">**액티비티가 소멸되기 전에 호출**</span>되는데 해당 콜백이 호출되는 경우는 다음과 같습니다.

1. 사용자가 액티비티를 완전히 닫거나 `finish()`  를 호출하여 액티비티가 종료되는 경우
2. 기기 회전이나 멀티 윈도우 모드로 인해서 일시적으로 액티비티를 소멸시키는 경우

`onDestroy()` 는 액티비티가 종료된다면 수명 주기에서 마지막으로 수신하는 콜백 메서드가 됩니다. 따라서 이전에 미처 해제하지 못했던 모든 리소스들을 여기서 처리하고 끝내야 합니다.

​		

## 여러 상황에서 생명주기

지금까지 생명주기의 각 콜백들의 역할과 특징들을 알아봤습니다. 이제는 직접 여러 상황들을 구현하여 생명주기가 어떻게 되는지 알아보겠습니다. 실습에 사용되는 액티비티는 총 두 개, 메인 액티비티와 서브 액티비티가 되겠습니다. 

앱 실행을 하면 시작 액티비티는 메인 액티비티가 되고 메인에는 서브 액티비티로 전환할 수 있는 버튼이 하나 있습니다. 서브 액티비티에는 메인 액티비티로 전환할 수 있는 버튼 하나와 숫자를 카운트할 수 있는 카운트 버튼이 존재합니다. 각 액티비티의 코드는 다음과 같습니다.

### MainActivity 코드

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        Log.d("MainActivity", "onCreate")

        binding = ActivityMainBinding.inflate(layoutInflater)
        val view = binding.root
        setContentView(view)

        binding.btnMoveAnotherActivity.setOnClickListener {
            startActivity(Intent(this, SubActivity::class.java))
            finish()
        }
    }

    override fun onStart() {
        super.onStart()
        Log.d("MainActivity", "onStart")
    }

    override fun onResume() {
        super.onResume()
        Log.d("MainActivity", "onResume")
    }

    override fun onPause() {
        super.onPause()
        Log.d("MainActivity", "onPause")
    }

    override fun onStop() {
        super.onStop()
        Log.d("MainActivity", "onStop")
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d("MainActivity", "onDestroy")
    }

    override fun onRestart() {
        super.onRestart()
        Log.d("MainActivity", "onRestart")
    }
}
```

​		

### SubActivity 코드

```kotlin
class SubActivity : AppCompatActivity() {

private lateinit var binding: ActivitySubBinding
private var counter: Int = 0

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        Log.d("SubActivity", "onCreate")

        binding = ActivitySubBinding.inflate(layoutInflater)
        val view = binding.root
        setContentView(view)

        binding.tvCounter.text = counter.toString()

        binding.btnMoveMainActivity.setOnClickListener {
            startActivity(Intent(this, MainActivity::class.java))
            finish()
        }

        binding.btnAddCount.setOnClickListener {
            counter += 1
            binding.tvCounter.text = counter.toString()
            Log.d("Counter 증가 요청", "${binding.tvCounter.text}")
        }

    }

    override fun onStart() {
        super.onStart()
        Log.d("SubActivity", "onStart")
    }

    override fun onResume() {
        super.onResume()
        Log.d("SubActivity", "onResume")
    }

    override fun onPause() {
        super.onPause()
        Log.d("SubActivity", "onPause")
    }

    override fun onStop() {
        super.onStop()
        Log.d("SubActivity", "onStop")
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d("SubActivity", "onDestroy")
    }

    override fun onRestart() {
        super.onRestart()
        Log.d("SubActivity", "onRestart")
    }
}
```

​		

### 앱 실행

홈 화면에서 앱을 실행합니다. 

초기 액티비티를 생성하므로 `onCreate()` 부터 호출이 됩니다.

```
MainActivity onCreate
MainActivity onStart
MainActivity onResume
```

​		

### 액티비티 전환

메인 액티비티에서 서브 액티비티로 전환합니다. 

메인 액티비티의 소멸까지 기다렸다가 서브 액티비티가 생성되는 것이 아닙니다. 메인 액티비티를 일시 중지 시켜놓고 서브 액티비티를 `onResume()` 까지 진행시킨 다음, 그 뒤에 메인 액티비티를 종료시키는 겁니다. 액티비티의 전환은 이렇게 처리됩니다.

1. MainActivity의 `onPause()` 메서드가 실행된다.
2. SubActivity의 `onCreate()`, `onStart()`, `onResume()` 메서드가 순차적으로 실행된다.
3. MainActivity는 `finish()`  호출로 `onDestroy()` 까지 호출되어 소멸된다.

```
MainActivity onPause
SubActivity onCreate
SubActivity onStart
SubActivity onResume
MainActivity onStop
MainActivity onDestroy
```

​		

### 세로 모드에서 가로 모드로 전환

메인 액티비티에서 가로 모드로 전환했습니다. 

위의 콜백 메서드 설명 부분에서도 언급했지만 화면 구성이 변경될 경우(가로 모드로 전환) 시스템은 액티비티를 소멸시키고 다시 생성하여 가로 모드에 대한 구성을 진행합니다.

```
MainActivity onPause
MainActivity onStop
MainActivity onDestroy
MainActivity onCreate
MainActivity onStart
MainActivity onResume
```

​		

### 가로 모드로 전환될 때 데이터 상태

서브 액티비티에서 카운터를 증가시킨 후 가로 모드로 전환했을 때 과연 카운터 데이터는 유지될까요? 유지되지 않습니다. 가로 모드로 전환하면 액티비티가 소멸되고 다시 생성되기 때문에 데이터도 초기화가 됩니다.

​		

### 홈 화면으로 이동

메인 액티비티에서 홈 화면으로 이동을 해봤습니다.

액티비티 화면이 포그라운드에서 벗어나 사용자 시야에서 완전히 사라지므로 `onStop()` 까지 진행됐습니다.

```
MainActivity onPause
MainActivity onStop
```

​		

### 홈 화면에서 앱으로 다시 이동

이제 다시 홈 화면에서 메인 액티비티로 돌아갑니다.

중단 상태에서 액티비티가 재개하므로 `onRestart()` 를 시작으로 `onResume()` 까지 호출됩니다.

```
MainActivity onRestart
MainActivity onStart
MainActivity onResume
```

​		

### 앱 종료

앱을 완전히 종료시킵니다.

앱이 종료됨에 따라서 액티비티도 소멸하므로 `onDestroy()` 까지 호출되고 액티비티는 소멸합니다.

```
MainActivity onPause
MainActivity onStop
MainActivity onDestroy
```

​		

## 마치면서...

지금까지 생명주기의 뜻부터 예시까지 알아봤습니다. 이 글은 안드로이드 공식 문서를 참고하여 작성했으며 제 나름대로 해석을 하면서 작성한 거라 잘못된 부분이 있을 수도 있습니다. 혹시라도 잘못된 부분이 있어서 알려주신다면 바로 수정하겠습니다. 긴 글 읽어주셔서 감사합니다!

​		

## 참조

[**Android 공식 문서**]( https://developer.android.com/guide/components/activities/activity-lifecycle?hl=ko)

[**제이슨의 개발이야기**](https://jason-api.tistory.com/85)

[**Dev.Cho - Dev World**](https://kotlinworld.com/45)







