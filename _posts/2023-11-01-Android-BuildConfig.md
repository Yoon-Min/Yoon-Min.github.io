---
title: Android BuildConfig 적용 안되는 현상
author: yoonmin
date: 2023-11-01 12:00:00 +0900
categories: [Android, 이슈]
tags: [Android, BuildConfig]
render_with_liquid: false
---

## 빌드를 다시 해봐도 `BuildConfig` 가 없음

`BuildConfig` 는 보통 노출되면 안되는 값을 저장할 때 사용되곤 합니다. 그래서 프로젝트를 몇번 해봤다면 해당 기능을 사용해봤을 겁니다. 보통 서버 주소나 API 키와 같은 중요한 값들을 로컬 프로퍼티에 저장을 하고 재빌드를 해서 `BuildConfig` 를 참조하게 됩니다. 그런데 재빌드를 했음에도 불구하고 `BuildConfig` 가 참조되지 않는 경우가 있었을 겁니다.



## AGP 8.0 부터 `BuildConfig` 는 기본으로 적용되지 않음

빌드 퍼포먼스 향상을 위해서 그래들 플러그인 8.0 버전부터는 `BuildConfig` 생성이 되지 않도록 변경됐습니다.

#### Android 공식문서 내용 발췌

> Starting with AGP 8.0, the default values for these flags have changed to improve build performance. To get help adjusting your code to support some of these changes, use the AGP Upgrade Assistant (**Tools > AGP Upgrade Assistant**). The Upgrade Assistant guides you through updating your code to accommodate the new behavior or setting flags to preserve the previous behavior.

| Flag                                         | New default value | Previous default value | Notes                                                        |
| :------------------------------------------- | :---------------- | :--------------------- | :----------------------------------------------------------- |
| `android.defaults.buildfeatures.buildconfig` | `false`           | `true`                 | AGP 8.0 doesn't generate `BuildConfig` by default. You need to specify this option using the DSL in the projects where you need it. |

#### Medium : Android Developers 내용 발췌

> If you call the `BuildConfig` class from your module code, you need to enable `buildConfig` in the `android {} `block in your module’s `build.gradle.kts` file. Otherwise, the `BuildConfig` file isn’t automatically generated anymore.

​		

## `BuildConfig` 사용을 명시해야 사용 가능

따라서 그래들 플러그인 버전이 8.0 이상이라면 해당 기능을 사용하기 위해서 그래들 파일에 직접 명시를 해줘야 합니다. `ViewBinding` `DataBinding` 을 사용하기 위해서 그래들 파일에 명시해주는 거랑 똑같다고 보시면 됩니다.

```kotlin
// Module build.gradle.kts
android {
  buildFeatures {
    buildConfig = true
  }
}
```

​		

## 참조

[**Android 공식문서**](https://developer.android.com/build/releases/past-releases/agp-8-0-0-release-notes)

[**Medium : Android Developers**](https://medium.com/androiddevelopers/5-ways-to-prepare-your-app-build-for-android-studio-flamingo-release-da34616bb946)