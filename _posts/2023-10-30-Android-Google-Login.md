---
title: Android 구글 로그인 구현하기
author: yoonmin
date: 2023-10-31 22:00:00 +0900
categories: [Android, 라이브러리]
tags: [Android, 소셜 로그인, 구글, Google Login]
render_with_liquid: false
---

## 사전 준비

안드로이드에서 구글 로그인을 구현하기 위해서는 준비해야 될 게 몇 가지가 있습니다. 구글 클라우드에 들어가서 OAuth 클라이언트 아이디를 만들고 그 과정에서 본인의 앱을 등록해야 합니다. 그러므로 구체적으로 무엇을 먼저 준비해야 하는지 알아보겠습니다.

{: .prompt-warning }

> 파이어베이스에 앱을 등록해서 관리하는 것까지 할 예정이니 파이어베이스 과정이 필요없다면 해당 부분은 스킵하세요!

​		

### 1. 구글 클라우드에서 클라이언트 아이디 생성

1. 구글 클라우드에 들어가서 오른쪽 상단의 `콘솔` 을 클릭합니다.

2.  `API 및 서비스`로 들어갑니다. (만약 프로젝트가 없다면 새 프로젝트부터 만들고 시작)

3. 왼쪽에 메뉴가 다섯 가지가 존재하는데 거기서 `사용자 인증 정보`를 클릭합니다. 

4. `+ 사용자 인증 정보 만들기` 를 눌러서 `OAuth 클라이언트 ID` 를 클릭해서 아이디를 생성합니다. 여기서 애플리케이션 유형을 선택할 수 있는데 <span style="color: #30aaa0">**웹 애플리케이션**, **Android**</span> 이 두 개를 각각 만들어야 합니다.

   ![스크린샷 2023-10-30 오후 8 28 01](https://github.com/Yoon-Min/Yoon-Min.github.io/assets/80873132/7638e2ba-8d59-48a7-8516-7908da04e136)

​		

### 2. SHA-1 값 찾기

**웹 애플리케이션** 유형은 그냥 생성하면 되므로 <span style="color: #30aaa0">**Android**</span> 유형 만드는 방법을 중점으로 알아보겠습니다. 여기서는 중요한 게 패키지 이름과 `SHA-1 인증서 디지털 지문` 입니다. 패키지 이름은 본인 프로젝트의 패키지 이름을 그대로 적으면 됩니다. 

인증서 디지털 지문은 그래들 태스크를 이용해서 찾을 수 있습니다. 그래들 태스크는 다음과 같이 두 가지 방법으로 실행할 수 있습니다. 태스크를 실행시켰다면 `gradle signingreport` 를 입력해서 실행시키면 됩니다. 그러면 터미널 창에서 `SHA1` 을 확인할 수 있습니다.

{: .prompt-tip }

> 1. 안드로이드 스튜디오 오른쪽 메뉴를 클릭한 다음, 터미널 아이콘을 클릭해서 그래들 태스크를 실행합니다.
> 2. 안드로이드 스튜디오 오른쪽 상단의 검색 아이콘을 눌러 `Gradle Task` 를 검색해서 그래들 태스크를 실행합니다.

![스크린샷 2023-10-30 오후 9 55 1](https://github.com/Yoon-Min/Yoon-Min.github.io/assets/80873132/61dc34a2-1bb9-459e-9538-2ca5b577d491){: .normal}	![Group 1](https://github.com/Yoon-Min/Yoon-Min.github.io/assets/80873132/f3fc98d8-7b76-4bd4-b25d-d8e095476e54)

​				

## 파이어베이스에 앱 등록하기

파이어베이스에 가서 프로젝트를 새로 만드시고 프로젝트에 들어가서 앱 추가를 해주세요. 당연히 안드로이드로 해주시면 됩니다. 그러면 구글 클라이언트 아이디 생성할 때와 똑같이 패키지 이름과 `SHA-1` 값을 요구할 겁니다. 동일하게 작성하고 완료하면 됩니다. 다 만들면 `google-services.json` 파일을 다운받을 수 있을텐데 이거를 프로젝트 폴더 내 `app` 에 붙여넣기 해주세요.

![스크린샷 2023-10-31 오후 10 31 1](https://github.com/Yoon-Min/Yoon-Min.github.io/assets/80873132/f03d2af2-874b-43b4-905f-73eee6b73eec)

![스크린샷 2023-10-31 오후 10 33 1](https://github.com/Yoon-Min/Yoon-Min.github.io/assets/80873132/d5a0abe9-2847-445a-a5e9-c08d4340042d)

![Group 2](https://github.com/Yoon-Min/Yoon-Min.github.io/assets/80873132/58165e22-1cb9-42f1-b7b7-d69d168282c6)

​		

## 의존성 추가

그래들 앱 수준 -  `Module:app` 에서 추가해야 하는 의존성 및 플러그인입니다.

```kotlin
plugins {
    id("com.google.gms.google-services") // <- 추가
}
```

```kotlin
dependencies {
    implementation(platform("com.google.firebase:firebase-bom:32.3.1"))
    implementation("com.google.firebase:firebase-auth-ktx")
    implementation("com.google.android.gms:play-services-auth:20.7.0")
}
```

그래들 프로젝트 수준 - `Project:{본인 프로젝트명}` 에서 추가해야 하는 플러그인입니다.

```kotlin
plugins {
    id("com.google.gms.google-services") version "4.4.0" apply false // <- 추가
}
```

​		

## 구글 아이디 토큰 발급하기

이제 코드를 작성할 단계입니다. 파이어베이스에 로그인 유저 관리를 하기 위해서는 먼저 구글 아이디 토큰이란 것을 받아와야 합니다. 아이디 토큰을 이용해서 파이어베이스에 연결하기 때문에 이 작업을 먼저 처리해야 합니다.

1. `GoogleSignInOptions` 을 생성합니다. 저는 아이디 토큰 요청, 이메일 요청만 추가했지만 다른 옵션이 더 필요하다면 추가하면 됩니다. 여기서 `requestIdToken` 에 넘기는 인자가 <span style="color: #30aaa0">**웹 클라이언트 유형으로 만들었던 클라이언트 아이디 값**</span>입니다. 

   ```kotlin
   private val gso = GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
       .requestIdToken(BuildConfig.GOOGLE_CLIENT_ID)
       .requestEmail()
       .build()
   ```

2. `GoogleSignInClient` 을 생성합니다.

   ```kotlin
   val googleSignInClient = GoogleSignIn.getClient(this, gso)
   ```

3. 사용자가 구글 로그인을 요청했을 때 결과를 처리하는 코드를 작성

   ```kotlin
   val googleLogInRequest =
       registerForActivityResult(ActivityResultContracts.StartActivityForResult()) {
           if (it.resultCode == RESULT_OK) {
               val task = GoogleSignIn.getSignedInAccountFromIntent(it.data)
               try {
                   val account = task.getResult(ApiException::class.java)
                   account?.let { user ->
                       user.idToken?.let { idToken ->
                           Timber.d("Get Account idToken! : ${account.idToken}")
                       }
                   }
               } catch (e: ApiException) {
                   Timber.d(e)
               }
           }
       }
   ```

4. 사용자가 구글 로그인 버튼을 클릭했을 때 이벤트를 처리하는 코드 작성

   ```kotlin
   _binding.btnRequestGoogleLogin.setOnClickListener {
       googleLogInRequest.launch(googleSignInClient.signInIntent)
   }
   ```

​		

## 파이어베이스에 연결하기

아이디 토큰을 받았다면 이제 해당 토큰을 가지고 파이어베이스에 연결을 해야 합니다. 그러기 위해서 먼저 파이어베이스 `auth` 를 생성합니다.

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var auth: FirebaseAuth

    override fun onCreate(savedInstanceState: Bundle?) {
				// ...
        auth = Firebase.auth
    }
}
```

그리고 아이디 토큰을 가지고 파이어베이스에 연결을 시도합니다.

```kotlin
private fun handleGoogleIdToken(idToken: String) {
    val firebaseCredential = GoogleAuthProvider.getCredential(idToken, null)
    auth.signInWithCredential(firebaseCredential)
        .addOnCompleteListener(this) {
            if(it.isSuccessful) {
                // 정상 완료
            }
        }
}
```

​		

## 정리

지금까지 구글 로그인 연결을 위해 필요한 과정들을 알아봤습니다. 최종적으로 구한 아이디 토큰이나 파이어베이스 `auth` 가 가지고 있는 값들을 프로젝트에 연결된 서버에 전송하여 유저 등록을 하게 됩니다. 지금까지의 코드를 간단하게 정리한 예시를 끝으로 마치겠습니다. 감사합니다.

```kotlin
class MainActivity : AppCompatActivity() {

    private val viewModel: GoogleLoginViewModel by viewModel()

    private lateinit var _binding: ActivityMainBinding
    private lateinit var auth: FirebaseAuth
    private val gso = GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
        .requestIdToken(BuildConfig.GOOGLE_CLIENT_ID)
        .requestEmail()
        .build()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        _binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        setContentView(_binding.root)

        _binding.vm = viewModel
        auth = Firebase.auth

        val googleSignInClient = GoogleSignIn.getClient(this, gso)
        val googleLogInRequest =
            registerForActivityResult(ActivityResultContracts.StartActivityForResult()) {
                if (it.resultCode == RESULT_OK) {
                    val task = GoogleSignIn.getSignedInAccountFromIntent(it.data)
                    try {
                        val account = task.getResult(ApiException::class.java)
                        account?.let { user ->
                            user.idToken?.let { idToken ->
                                Timber.d("Get Account idToken! : ${account.idToken}")
                                handleLoginResponse()
                                viewModel.googleLoginByToken(idToken)
                            }
                        }
                    } catch (e: ApiException) {
                        Timber.d(e)
                    }
                }
            }

        _binding.btnRequestGoogleLogin.setOnClickListener {
            googleLogInRequest.launch(googleSignInClient.signInIntent)
        }

        _binding.btnRequestGoogleSignOut.setOnClickListener {
            googleSignInClient.signOut().addOnCompleteListener {
                if(it.isComplete) {
                    viewModel.googleSignOut()
                    toast("Complete sign out!")
                }
                else { toast("Failure sign out!") }
            }
        }
    }

    private fun handleLoginResponse() {
        viewModel.account.observe(this) { _binding.accountInfo = it }
    }

    private fun handleGoogleIdToken(idToken: String) {
        val firebaseCredential = GoogleAuthProvider.getCredential(idToken, null)
        auth.signInWithCredential(firebaseCredential)
            .addOnCompleteListener(this) {
                if(it.isSuccessful) {
                    // 정상 완료
                }
            }
    }
}
```

​		

## 참조

[**Android용 로그인**](https://developers.google.com/identity/sign-in/android/start?hl=ko)

[**Android에서 Google로 인증**](https://firebase.google.com/docs/auth/android/google-signin?hl=ko)
