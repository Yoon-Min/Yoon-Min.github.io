---
title: Github 블로그 로컬 이미지 파일 표시 오류
author: yoonmin
date: 2025-03-29 00:00:00 +0900
categories: [블로그, Github 블로그 만들기]
tags: [GitHub, Blog, 깃허브 블로그, 로컬 서버 구동 오류]
render_with_liquid: true
---

# 로컬 이미지의 경로를 이용한 표시 오류 발생

블로그 빌드 버전을 업그레이드하고 난 뒤, 새로운 글 작성을 하던중 이미지 표시가 안 되는 오류를 마주했습니다. 원래 빌드 버전 업그레이드 전까지는 외부 도메인을 이용한 이미지 첨부 방식을 사용했습니다. 그런데 버전을 높이자 외부 도메인 첨부는 보안상 허용하지 않는다는 오류가 발생했습니다.

그래서 외부 도메인에서 이미지를 블로그 폴더 내 저장해서 경로를 첨부하는 로컬 이미지 업로딩 방식으로 바꾸고 테스트를 해봤는데 이번에는 이미지가 표시되지 않는 오류가 발생했습니다.

## Chirpy 테마에서 제시하는 이미지 첨부 방법

제가 사용하고 있는 블로그 테마는 Chirpy입니다. 해당 테마는 이미지 첨부를 로컬 폴더에 저장해서 경로를 명시하는 방식으로 할 것을 추천합니다. 블로그 폴더 루트에 `assets` 라는 폴더가 있는데 이 폴더에 첨부하고 싶은 이미지를 첨부하고 경로를 명시하면 첨부가 된다는 것입니다.

```markdown
![image](/assets/img/thumbnail.jpg)
```

예를 들어서 `assets` 폴더 내 `img` 라는 이름의 폴더를 만들고 그 안에다 썸네일 사진을 저장했다면 위와 같이 이미지를 첨부할 수 있습니다. 하지만 로컬 서버로 구동해 보니 이미지가 표시되지 않았습니다. 

## 원인 발견 - 잘못된 베이스 주소

크롬 개발자 도구를 통해 이미지 요청에 대한 네트워킹 정보를 훑어 봤는데 이미지 주소에서 문제점을 발견했습니다. 이미지 경로의 베이스 주소가 제 깃허브 주소가 아닌 다른 주소가 첨부된 것이었습니다.

​		

# 해결 방법

> These URL variables are not magic and need to be applied to links in your layouts, includes, or themes. This can be done by prefixing all links with `{{ site.url }}{{ site.baseurl }}` or by apply Jekyll’s [`absolute_url`](https://mademistakes.com/mastering-jekyll/site-url-baseurl/#absolute_url-filter) or [`relative_url`](https://mademistakes.com/mastering-jekyll/site-url-baseurl/#relative_url-filter) filters to them.
>
> mademistakes - Jekyll’s site.url and baseurl

해결 방법은 간단합니다. 정확한 베이스 주소(나의 깃허브 블로그 주소)를 이미지 로컬 경로 맨 앞에 추가해 주기만 하면 됩니다. 베이스 주소 첨부 방법은 다음과 같습니다.

![image]({{ site.url }}/assets/img/post/branch7/1.png)

위 코드를 글을 작성할 마크다운 파일에 추가하고, 로컬 서버를 구동해서 페이지에 접속해 보면 아래처럼 주소가 입혀져 이미지가 정상 표시됩니다.

```markdown
![image]({{ site.url }}/assets/img/thumbnail.jpg)
```

​		

# 참조

[mademistakes - Jekyll’s site.url and baseurl](https://mademistakes.com/mastering-jekyll/site-url-baseurl/)