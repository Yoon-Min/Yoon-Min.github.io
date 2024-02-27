---
title: Android Retrofit - 1. Retrofit 객체 생성과 API Interface의 동작 과정
author: yoonmin
date: 2024-02-26 00:00:00 +0900
categories: [Android, 라이브러리]
tags: [Kotlin, Retrofit2, REST API, Android]
render_with_liquid: true
---

![eberhard-grossgasteiger-pj9-Gb8dEHE-unsplash 1](https://github.com/Yoon-Min/Yoon-Min.github.io/assets/80873132/9ce1c135-6d5a-426e-9bee-04a3cd89bd00)

<span style="color: #898989">*Retrofit version - 2.9.0*</span>

## Retrofit Intro

> *A type-safe HTTP client for Android and Java* - **Square** -

레트로핏은 Android, Java에서 사용되는 타입 세이프한 `Http Client`이며 안드로이드 애플리케이션에서 네트워크 요청 및 응답 처리에 대한 과정을 단순화시켜주는 라이브러리입니다. 이 라이브러리를 알아보기 전에 레트로핏의 기반이 되는 `OkHttpClient` 가 무엇인지부터 알아야 할 필요가 있습니다.

Square에서 레트로핏 이전에 `OkHttpClient` 라는 `Http Client`를 먼저 제공했습니다. 이후에 `OkHttpClient`의 네트워크 처리 기능 기반에 어노테이션을 이용한 API 인터페이스로 응답을 처리하는 좀 더 높은 수준의 추상화를 추가하게 됐는데 이것이 `Retrofit`입니다.

실제로 레트로핏을 안드로이드 프로젝트에 사용하게 되면, 어노테이션을 이용하여 요청이 처리되는 방법에 대한 인터페이스를 정의하고 해당 인터페이스의 메서드를 호출하는 방식으로 API 응답을 얻습니다.

​		

- `OkHttpClient` 방식

  <span style="color: #898989">클라이언트 객체를 생성하고, 요청 처리 방법을</span> `Request` <span style="color: #898989">객체로 만들어서 해당 요청을 클라이언트가 처리합니다.</span>

  ```java
  OkHttpClient client = new OkHttpClient();
  
  String run(String url) throws IOException {
    Request request = new Request.Builder()
        .url(url)
        .build();
  
    try (Response response = client.newCall(request).execute()) {
      return response.body().string();
    }
  }
  ```

​	

- `Retrofit` 방식

  <span style="color: #898989">요청 처리 방법을 어노테이션을 이용한 인터페이스로 정의하고, 레트로핏 객체를 통해서 인터페이스 내 메서드를 호출할 수 있게 내부적으로</span> `create` <span style="color: #898989">합니다. 생성된 이후에 메서드 호출을 통해 응답을 받습니다.</span>

  ```java
  public interface GitHubService {
    @GET("users/{user}/repos")
    Call<List<Repo>> listRepos(@Path("user") String user);
  }
  
  Retrofit retrofit = new Retrofit.Builder()
      .baseUrl("https://api.github.com/")
      .build();
  
  GitHubService service = retrofit.create(GitHubService.class);
  Call<List<Repo>> repos = service.listRepos("octocat");
  ```

​		

| 그러면 여기서 생기는 궁금증                        |
| -------------------------------------------------- |
| *1. Retrofit 객체 생성 과정은?*                    |
| *2. 인터페이스를 사용가능한 객체로 만드는 과정은?* |

두 가지 모두 레트로핏 라이브러리 자체에서 처리되기 때문에 내부를 들여다보지 않는 한 단순 인터페이스 정의로 HTTP 통신을 할 수 있다는 것을 완벽히 이해하기는 힘듭니다.

따라서 이번 글은 레트로핏의 인스턴스 생성 과정과 전반적인 구조를 훑어보고 인터페이스 메서드 호출이 가능한 이유를 파악하여 `Retrofit` 라이브러리에 대한 시야를 넓히고자 합니다.

​		

## Retrofit 구성하는 클래스와 인터페이스

`Retrofit` 객체 생성과 인터페이스 처리 과정을 알아보기 전에 레트로핏을 구성하는 클래스와 인터페이스가 무엇이고 어떤 역할을 하는지 알아야 할 필요가 있습니다. 그리고 각 클래스와 인터페이스를 구성하는 주요한 메서드가 무엇인지도 같이 알아보겠습니다.

### Call

```kotlin
interface Call : Cloneable { /* ... */ }
```

>  *"An invocation of a Retrofit method that sends a request to a webserver and returns a response. Each call yields its own HTTP request and response pair."*
>
> ***웹 서버에 요청 및 응답을 반환하고 HTTP 요청과 응답에 대한 쌍을 생성하는 역할을 합니다.***

`Call` 에서 주요하게 볼 부분은 동기, 비동기 처리 방식이 모두 존재한다는 것입니다. 그래서 코틀린에서 지원하는 코루틴과 적절히 섞어 사용한다면 효율적인 부분을 기대할 수 있습니다. 

​				

```kotlin
fun execute(): Response
```

> *"Synchronously send the request and return its response."*
>
> ***요청과 응답을 받는 과정을 동기적으로 처리합니다.***

​		

```kotlin
fun enqueue(responseCallback: Callback)
```

> *"Asynchronously send the request and notify callback of its response or if an error occurred talking to the server, creating the request, or processing the response."*
>
> ***요청과 응답을 받는 과정을 비동기로 처리합니다.***

 `Callback` 이라는 파라미터가 있는데 서버 또는 오프라인 요청에 대한 결과를 가져옵니다. 우리는 해당 `Callback` 인터페이스를 구현하여 결과에 대한 원하는 동작을 정의할 수 있습니다.

​		

---

​		

### Callback

```kotlin
interface Callback { /* ... */ }
```

> *"Communicates responses from a server or offline requests."*
>
> ***서버 또는 오프라인 요청의 응답을 전달하는 역할입니다.***

콜백 메서드는 레트로핏이 가지고 있는 콜백 실행기에 의해 실행됩니다. 만약 콜백 실행기를 따로 설정하지 않는다면 내부에서 지정한 기본 실행기를 사용합니다. 

기본 실행기는 `Platform` 클래스에서 가져오는데 여기서 기본 실행기로 처리되는 콜백은 메인 스레드에서 처리되는 것을 알 수 있습니다. 즉, 기본적으로 `Android` 내에서  콜백 처리는 메인 스레드에서 처리하도록 설정되어 있습니다.

```kotlin
@Override
public Executor defaultCallbackExecutor() {
  return new MainThreadExecutor();
}

static final class Android extends Platform {
	
  /* ... */

  static final class MainThreadExecutor implements Executor {
    private final Handler handler = new Handler(Looper.getMainLooper());

    @Override
    public void execute(Runnable r) {
      handler.post(r);
    }
  }
}
```

​		

```kotlin
fun onFailure(call: Call, e: IOException)
```

> *"Invoked when a network exception occurred talking to the server or when an unexpected exception occurred creating the request or processing the response."*
>
> ***서버 통신 과정에서 네트워크 예외, 응답을 처리하거나 요청을 만드는 과정에서 예기치 못한 예외가 발생했을 때 실행됩니다.***

​		

```kotlin
@Throws(IOException::class)
fun onResponse(call: Call, response: Response)
```

> *"Called when the HTTP response was successfully returned by the remote server."*
>
> ***HTTP 응답이 원격 서버에 의해 성공적으로 반환됐을 때 호출됩니다.***

`response` 를 통해서 응답 본문을 읽을 수 있습니다. 

​		

## Retrofit 인스턴스 생성 과정

우선 레트로핏은 인스턴스를 생성하는 데 빌더 패턴을 사용합니다. 빌더 패턴의 특성상, 빌더 클래스 내에서 타겟 인스턴스를 생성하므로 빌더의 생성자부터 보겠습니다.

```java
Builder(Platform platform) {
  this.platform = platform;
}

public Builder() {
  this(Platform.get());
}
```

생성자로 `Platform.get()` 을 호출하게 되는데 플랫폼 클래스 내에 시스템의 가상 머신 이름으로 플랫폼을 구분하는 메서드가 존재합니다. 안드로이드는  `Dalvik VM` 이므로 `Android()` 가 반환됩니다.

```kotlin
/* Platform.java */
private static Platform findPlatform() {
  return "Dalvik".equals(System.getProperty("java.vm.name"))
      ? new Android() //
      : new Platform(true);
}
```

이제 빌더 객체의 생성이 완료되어 레트로핏 객체 생성에 필요한 준비물을 전달할 수 있습니다. 여기서 필수로 전달해야 할 정보와 전달하지 않아도 되는 정보가 있는데 이는 `build` 메서드 코드를 보면 알 수 있습니다.

```java
public Retrofit build() {
  if (baseUrl == null) {
    throw new IllegalStateException("Base URL required.");
  }

  okhttp3.Call.Factory callFactory = this.callFactory;
  if (callFactory == null) {
    callFactory = new OkHttpClient();
  }

  Executor callbackExecutor = this.callbackExecutor;
  if (callbackExecutor == null) {
    callbackExecutor = platform.defaultCallbackExecutor();
  }

  // Make a defensive copy of the adapters and add the default Call adapter.
  List<CallAdapter.Factory> callAdapterFactories = new ArrayList<>(this.callAdapterFactories);
  callAdapterFactories.addAll(platform.defaultCallAdapterFactories(callbackExecutor));

  // Make a defensive copy of the converters.
  List<Converter.Factory> converterFactories =
      new ArrayList<>(
          1 + this.converterFactories.size() + platform.defaultConverterFactoriesSize());

  // Add the built-in converter factory first. This prevents overriding its behavior but also
  // ensures correct behavior when using converters that consume all types.
  converterFactories.add(new BuiltInConverters());
  converterFactories.addAll(this.converterFactories);
  converterFactories.addAll(platform.defaultConverterFactories());

  return new Retrofit(
      callFactory,
      baseUrl,
      unmodifiableList(converterFactories),
      unmodifiableList(callAdapterFactories),
      callbackExecutor,
      validateEagerly);
}
```

*`BaseUrl`*

: 해당 정보를 전달하지 않는다면 빌드 과정에서 `IllegalStateException` 예외가 발생하므로 이 정보는 필수입니다.

*`Call.Factory`*

: 이 정보는 전달하지 않아도 기본으로 `OkHttpClient` 를 사용하기 때문에 선택사항입니다. 여기서 내부적으로 `OkHttpClient` 를 이용한다는 것을 알 수 있습니다.

*`CallbackExecutor`*

: 레트로핏을 사용하는  `Android` 에서 `Callback` 은 `MainThread` 에서 처리가 됩니다. 따라서 커스텀 콜백 실행자가 따로 없다면 기본으로 플랫폼 객체에서 `MainThreadExecutor` 를 가지고 와서 사용합니다.

*`CallAdapter.Factory`*

: `CallAdapter` 를 만드는 역할입니다.  `defaultCallAdapterFactories` 메서드를 호출해서 따로 커스텀을 추가하지 않아도 기본 팩토리를 리스트에 추가합니다.

*`Converter.Factory`*

: `Converter` 를 만드는 역할입니다. 어답터 팩토리와 마찬가지로 `BuiltInConverters` 객체와 `defaultConverterFactories` 메서드 호출을 통해 리스트를 채웁니다.

 *`Retrofit`*

: 최종적으로 위의 정보들이 `Retrofit` 인스턴스 생성을 위한 인자로 전달되는데 마지막 인자인 `validateEagerly` 는 빌드 과정에서 건들지 않기 때문에 `false` 로 전달됩니다. 이런 과정을 통해 최종적으로 생성된 레트로핏 인스턴스가 반환됩니다.

​		

## 인터페이스 내부 처리

지금까지 레트로핏 인스턴스 생성과정을 알아봤습니다. 이제는 인스턴스로 요청이 담긴 인터페이스의 메서드 호출을 어떻게 내부에서 처리하는지 알아보겠습니다.











## 참조

[**Retrofit And OkHttp**](https://medium.com/@erdi.koc/retrofit-and-okhttp-675d34eb7458)

[**Square Retrofit**](https://square.github.io/retrofit/)

[**Retrofit Javadoc**](https://square.github.io/retrofit/2.x/retrofit/)
