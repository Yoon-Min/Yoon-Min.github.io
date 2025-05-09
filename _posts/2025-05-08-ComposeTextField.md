---
title: Android Compose - TextField을 이용한 입력 텍스트 필드 구현
author: yoonmin
date: 2025-05-08 00:00:00 +0900
categories: [Android, Compose]
tags: [Android, Compose, UI, TextField]
render_with_liquid: true
---

![1](/assets/img/post/branch9/thumbnail.jpg)_사진: [Unsplash](https://unsplash.com/ko/사진/책상-위에-놓인-노트북-컴퓨터-FLuY1URxpus?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)의[Nik](https://unsplash.com/ko/@helloimnik?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)_

# Intro

최근 UI 제작 키트로 컴포즈를 이용한 프로젝트를 진행하고 있습니다. 검색할 수 있는 검색 바와 특정 텍스트를 입력하는 인풋 텍스트 필드가 주요 기능 요구사항중 하나였는데요, 컴포즈 기반의 프로젝트가 처음이다 보니 구현과 활용 방법을 정리할 필요성을 느껴 기록합니다.

이 글은 Material Design에서 제공하는 `TextField` 컴포저블을 활용하여 서비스 요구사항에 맞는 컴포저블로 제작하는 과정을 설명합니다. 좀 더 나은 구현 방법을 찾아내거나 텍스트 필드 컴포저블 내부 구조에 변경사항이 발생한다면, 시리즈로 포스팅을 추가하거나 현재 포스팅에 내용을 덧붙이는 방식으로 보완해 나갈 예정입니다.

​		

# 프로젝트 기능 요구사항

![2](/assets/img/post/branch9/1.png){: width="300" .normal}![3](/assets/img/post/branch9/2.png){: width="300" .normal}

첫 번째 사진은 커뮤니티 공간의 제목과 설명을 추가하는 요구사항을 가집니다. 두 번째 사진은 커뮤니티 공간의 소유자(유저)를 검색할 수 있어야 합니다. 정리하면 다음과 같이 몇 가지의 동작과 그에 따른 UI 요소가 필요합니다.

텍스트를 입력하면 필드에 반영

: 사용자가 입력한 텍스트를 저장하고 표시할 `Text` 컴포저블이 필요합니다.

현재  필드의 텍스트 존재여부에 따라 클리어(적었던 텍스트 전체 삭제) 활성화

: 필드 앞 혹은 뒷 부분에 `Button` 을 추가할 수 있는 구조가 필요합니다.

​		

# TextField 제작

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun TextField(
    value: String,
    onValueChange: (String) -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true,
    readOnly: Boolean = false,
    textStyle: TextStyle = LocalTextStyle.current,
    label: @Composable (() -> Unit)? = null,
    placeholder: @Composable (() -> Unit)? = null,
    leadingIcon: @Composable (() -> Unit)? = null,
    trailingIcon: @Composable (() -> Unit)? = null,
    prefix: @Composable (() -> Unit)? = null,
    suffix: @Composable (() -> Unit)? = null,
    supportingText: @Composable (() -> Unit)? = null,
    isError: Boolean = false,
    visualTransformation: VisualTransformation = VisualTransformation.None,
    keyboardOptions: KeyboardOptions = KeyboardOptions.Default,
    keyboardActions: KeyboardActions = KeyboardActions.Default,
    singleLine: Boolean = false,
    maxLines: Int = if (singleLine) 1 else Int.MAX_VALUE,
    minLines: Int = 1,
    interactionSource: MutableInteractionSource? = null,
    shape: Shape = TextFieldDefaults.shape,
    colors: TextFieldColors = TextFieldDefaults.colors()
) {
  ...
  CompositionLocalProvider(LocalTextSelectionColors provides colors.textSelectionColors) {
    BasicTextField(
      ...
      decorationBox =
        @Composable { innerTextField ->
            // places leading icon, text field with label and placeholder, trailing icon
            TextFieldDefaults.DecorationBox(
                value = value,
                visualTransformation = visualTransformation,
                innerTextField = innerTextField,
                placeholder = placeholder,
                label = label,
                leadingIcon = leadingIcon,
                trailingIcon = trailingIcon,
                prefix = prefix,
                suffix = suffix,
                supportingText = supportingText,
                shape = shape,
                singleLine = singleLine,
                enabled = enabled,
                isError = isError,
                interactionSource = interactionSource,
                colors = colors
            )
        }
    )
  }
}
```

뭔가 굉장히 복잡해 보이지만 파라미터가 많은 것을 제외하면 내부 코드는 단순합니다. `BasicTextField` 와 `DecorationBox` 컴포저블을 사용해서 텍스트 필드를 제작하는 방식입니다. 데코레이션 박스는 인자로 넘긴 클리어 버튼과 플레이스 홀더를 배치합니다.

​		

## ExperimentalMaterial3Api

```kotlin
@RequiresOptIn(
  "This material API is experimental and is likely to change or to be removed in" + " the future."
)
@Retention(AnnotationRetention.BINARY)
annotation class ExperimentalMaterial3Api
```

맨 위, 먼저 눈에 띄는 것은 `실험적인 API` 라는 문구의 어노테이션입니다. 해당 어노테이션 내부 메시지를 보면 현재 텍스트 필드 컴포저블은 안정적인 버전이 아닙니다. 그래서 언제든지 삭제되거나 변경될 수 있습니다. 

​		

## 주요 파라미터

### onValueChange

```kotlin
TextField(
  onValueChange = { textField ->
      // textField.text로 상태 값 변경 요청
  }
)
```

사용자가 텍스트를 입력할 때마다 변경된 값을 받아 볼 수 있습니다. 상태 홀더에 관리되는 사용자 입력 값은 이 파라미터 함수에 의해 최신 값으로 업데이트 될 것입니다.

```kotlin
TextField(
  onValueChange = { textField ->
    if (textField.text.length <= maxLength) {
      if (inputText.length != maxLength || textField.text.length != maxLength) {
          // textField.text로 상태 값 변경 요청
      }
    }
  }
)
```

그런데 첫 번째 기능 요구사항의 텍스트 필드는 글자 수 제한이 있습니다. 따라서 최대 길이가 넘어가면 입력 텍스트 상태를 업데이트하지 않도록 코드를 추가해야 합니다. 위 예시처럼 로직과 로직을 보관하는 장소는 개인이 판단하여 설계하면 될 것 같습니다.

​		

### placeholder

```kotlin
placeholder = {
  Text(text = hint)
}
```

XML 에서 `hint` 역할을 합니다. 텍스트 필드에 어떤 것을 입력하도록 유도하는 보조 문구입니다. 닉네임을 설정하는 필드라면 "닉네임을 입력해 주세요"가 플레이스홀더로 적합할 것입니다. 컴포저블 블럭이기 때문에 텍스트 컴포저블을 활용해서 보조 문구를 표시합니다.

​		

### trailingIcon, leadingIcon

```kotlin
leadingIcon = {
    Icon(
        painter = painterResource(id = R.drawable.ic_round_search),
        contentDescription = null,
        modifier = Modifier.size(dimensionResource(R.dimen.text_field_leading_icon_size)),
        tint = AppServiceColors.textHintMain
    )
}
trailingIcon = {
    ServiceInputFieldClearIcon(
        visibility = clearIconVisible,
        onClick = onClickTrailingIcon
    )
},
```

꼬리(오른쪽 끝) 부분에 아이콘을 배치할 수 있는 슬롯입니다. 이곳에 입력한 텍스트를 초기화 할 수 있는 버튼을 추가합니다. 입력 초기화 버튼 아이콘은 입력 값 상태에 따라 가시성을 설정해야 하므로 아이콘 버튼 컴포저블을 래핑한 형태로 정의합니다.

​		

### colors

```kotlin
colors = TextFieldDefaults.colors(
  unfocusedContainerColor = containerColor,
  focusedContainerColor = containerColor,
  unfocusedIndicatorColor = Color.Transparent,
  focusedIndicatorColor = Color.Transparent,
),
```

텍스트 필드의 여러 상태와 구성요소에 대한 색 지정이 가능합니다. 색상으로 지정할 수 있는 요소가 굉장히 많은데, 기능 요구사항에 따라 포커싱 여부에 따라 컨테이너와 인디케이터 색상만 지정해도 충분합니다.

해당 파라미터의 타입은 `TextFieldColors` 이고 기본 값으로 `TextFieldDefaults.colors()` 이 사용됩니다. 내부에 `copy` 메서드가 존재하기 때문에 기본 색상 + 일부 색상 직접 지정 방식도 가능합니다.

​		

### keyboardOptions, keyboardActions

```kotlin
keyboardOptions = KeyboardOptions.Default.copy(imeAction = ImeAction.Done),
keyboardActions = KeyboardActions(onDone = keyboardActions),
```

모바일 디바이스의 스크린 키보드의 버튼 종류 설정과 해당 버튼을 눌렀을 때 동작을 정의할 수 있습니다. 저는 액션 타입으로 `Done` 와 `Search` 을 사용할 것입니다.

​		

### value

```kotlin
TextFieldValue(text = inputText, selection = TextRange(inputText.length))
```

사용자가 입력한 텍스트 값을 의미합니다. `String` 이 아닌 `TextFieldValue` 타입이며 내부적으로 커서 위치 설정이 가능합니다. 예시 코드처럼 시작 커서 위치를 맨 뒤로 설정할 수 있습니다.

​		

### singleLine

```kotlin
singleLine = false,
```

아이디나 닉네임처럼 단일 라인으로 충분한 경우는 `false` 로 설정하지만, 첫 번째 요구사항처럼 여러 줄로 입력될 수 있는 경우는 `true` 로 설정합니다.

​		

### shape

```kotlin
shape = RoundedCornerShape(dimensionResource(R.dimen.radius_medium)),
```

텍스트 필드 컨테이너 모서리를 라운딩 처리합니다.

​		





