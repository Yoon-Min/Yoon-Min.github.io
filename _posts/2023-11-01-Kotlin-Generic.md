---
title: Kotlin Generic 제네릭
author: yoonmin
date: 2023-11-01 12:00:00 +0900
categories: [CS, 프로그래밍 언어, Kotlin]
tags: [Kotlin, Java, Generic, 제네릭]
render_with_liquid: false
---

## Kotlin도 Java의 제네릭 기능을 제공한다.

> **"Classes in Kotlin can have type parameters, just like in Java:"** - Kotlin 공식 문서 발췌

코틀린의 클래스는 자바와 마찬가지로 타입 파라미터를 가질 수 있습니다. 제네릭을 사용함으로써 컴파일 타임에 타입 안전성을 관리할 수 있고 여러 타입에 대응하여 코드를 재사용하는 이점을 얻을 수 있습니다. 그렇다면 코틀린에서의 제네릭은 어떤 특징을 가지고 있는지 알아보겠습니다.

## 타입 안전성

