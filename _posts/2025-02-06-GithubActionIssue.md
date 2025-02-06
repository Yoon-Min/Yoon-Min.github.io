---
title: Github Action 버전 마이그레이션 - Deprecation notice v3 of the artifact actions
author: yoonmin
date: 2025-02-06 00:00:00 +0900
categories: [블로그, Github 블로그 만들기]
tags: [GitHub, Blog, 깃허브 블로그]
render_with_liquid: true
---

## Github Action 오류 발생

깃허브 블로그 글을 포스팅하기 위해 깃허브 액션을 사용하던 도중 해당 오류를 마주했습니다. 확인해 보니 `action/upload-artifact` 의 새 버전을 사용하라는 내용이었습니다.

![Image](https://github.com/user-attachments/assets/ccf5f01d-8d64-4d07-b263-597415876400)

​		

## Deprecation notice: v3 of the artifact actions

> Starting **January 30th, 2025**, GitHub Actions customers will no longer be able to use v3 of [actions/upload-artifact](https://github.com/actions/upload-artifact) or [actions/download-artifact](https://github.com/actions/download-artifact). Customers should update workflows to begin using [v4 of the artifact actions](https://github.blog/2024-02-12-get-started-with-v4-of-github-actions-artifacts/) as soon as possible. While v4 of the artifact actions improves upload and download speeds by up to 98% and includes several new features, there are [key differences](https://github.com/actions/upload-artifact?tab=readme-ov-file#breaking-changes) from previous versions that may require updates to your workflows. Please see [the documentation](https://github.com/actions/upload-artifact/blob/main/docs/MIGRATION.md) in the project repositories for guidance on how to migrate your workflows.
>
> Github 공식 블로그 참고

**2025년 1월 30일부터** GitHub Actions 고객은 더 이상 동작/업로드 아티팩트 또는 동작/다운로드 아티팩트 v3를 사용할 수 없습니다. **사용자는 가능한 한 빨리 아티팩트 액션 v4를 사용할 수 있도록 워크플로를 업데이트해야 합니다.** 

아티팩트 작업 v4는 업로드 및 다운로드 속도를 최대 98% 높이고 몇 가지 새로운 기능을 포함하지만, 이전 버전과 주요 차이점이 있어 워크플로를 업데이트해야 할 수도 있습니다. 워크플로를 마이그레이션 하는 방법에 대한 안내는 프로젝트 저장소에 있는 문서를 참조하세요.

내용을 요약하면 `V3` 은 1월 30일부터 사용할 수 없으니, 사용자들은 업그레이드 버전을 사용하라는 말입니다. 그래서 1월 30일 이전에는 잘 동작하던 액션이 최근 동작 과정에서 버전 관련 오류로 동작하지 않아 당황하신 분들이 있을 거라 예상됩니다.

​		

## 해결 방법 - 버전명을 수정하자

해결 방법은 굉장히 간단합니다. 워크플로 파일 내 더 이상 지원하지 않는 버전을 최신 버전 명으로 수정하면 됩니다. 다음 표를 확인하고 `Old value`에 해당하는 값이 있다면 새 버전으로 교체하시면 되겠습니다. 저는 깃허브 블로그를 운영하는 입장이라 `actions/upload-pages-artifact` 버전을 `V3` 으로 수정했습니다.

| Old value                        | New value                        |
| :------------------------------- | :------------------------------- |
| actions/cache@v3                 | actions/cache@v4                 |
| actions/configure-pages@v3       | actions/configure-pages@v5       |
| actions/deploy-pages@v2          | actions/deploy-pages@v4          |
| actions/upload-pages-artifact@v2 | actions/upload-pages-artifact@v3 |
| peaceiris/actions-hugo@v2        | peaceiris/actions-hugo@v3        |

​		

## 참조

[**Build failed because it uses a deprecated version of `actions/upload-artifact: v3`**](https://discourse.gohugo.io/t/build-failed-because-it-uses-a-deprecated-version-of-actions-upload-artifact-v3/53335)

[**Deprecation notice: v3 of the artifact actions**](https://github.blog/changelog/2024-04-16-deprecation-notice-v3-of-the-artifact-actions/)

[**Migration**](https://github.com/actions/upload-artifact/blob/main/docs/MIGRATION.md)