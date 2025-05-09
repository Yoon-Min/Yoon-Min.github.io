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

# TextField

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
    @Suppress("NAME_SHADOWING")
    val interactionSource = interactionSource ?: remember { MutableInteractionSource() }
    // If color is not provided via the text style, use content color as a default
    val textColor =
        textStyle.color.takeOrElse {
            val focused = interactionSource.collectIsFocusedAsState().value
            colors.textColor(enabled, isError, focused)
        }
    val mergedTextStyle = textStyle.merge(TextStyle(color = textColor))

    CompositionLocalProvider(LocalTextSelectionColors provides colors.textSelectionColors) {
        BasicTextField(
            value = value,
            modifier =
                modifier
                    .defaultErrorSemantics(isError, getString(Strings.DefaultErrorMessage))
                    .defaultMinSize(
                        minWidth = TextFieldDefaults.MinWidth,
                        minHeight = TextFieldDefaults.MinHeight
                    ),
            onValueChange = onValueChange,
            enabled = enabled,
            readOnly = readOnly,
            textStyle = mergedTextStyle,
            cursorBrush = SolidColor(colors.cursorColor(isError)),
            visualTransformation = visualTransformation,
            keyboardOptions = keyboardOptions,
            keyboardActions = keyboardActions,
            interactionSource = interactionSource,
            singleLine = singleLine,
            maxLines = maxLines,
            minLines = minLines,
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

## ExperimentalMaterial3Api

가장 먼저 눈에 띄는 것은 `실험적인 API` 라는 문구의 어노테이션입니다. 해당 어노테이션 내부 메시지를 보면 현재 텍스트 필드 컴포저블은 안정적인 버전이 아닙니다. 그래서 언제든지 삭제되거나 변경될 수 있습니다.

```kotlin
@RequiresOptIn(
    "This material API is experimental and is likely to change or to be removed in" + " the future."
)
@Retention(AnnotationRetention.BINARY)
annotation class ExperimentalMaterial3Api
```



