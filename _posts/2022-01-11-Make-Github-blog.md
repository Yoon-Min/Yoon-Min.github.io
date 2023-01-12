---
title: Chirpy 테마를 이용한 Github 블로그 만들기
author: Yoon-Min
date: 2022-01-14 14:00:00 +0800
categories: [블로그, Github 블로그 만들기]
tags: [Github 블로그]
render_with_liquid: false

---

## 시작하기에 앞서 참고해주세요!

`jekyll`의 `Chirpy` 테마를 기준으로 진행합니다. 현재 이 블로그의 테마가 `Chirpy`테마이며 이 블로그의 테마 디자인이 마음에 들어서 해당 테마가 적용된 깃허브 블로그를 만들고 싶다면 이 글을 참고하시면 좋을 것 같습니다.

저는 `Chirpy`테마를 적용하기 위해서 해당 테마에 대한 여러 블로그 글과 제작자의 가이드를 참고해서 하라는 대로 따라했는데 중간 과정들의 결과가 블로그의 글과 달라서 당황했던 기억이 있습니다. 아무래도 제작자 분이 해당 테마의 적용 방식에 대한 업데이트를 여러 번 진행해서 적용 방법이 이전과 약간 달라진 것 같습니다.

그래서 앞으로 `Chirpy`테마를 적용하려는 분들에게 적용 방법과  제가 겪었던 문제점들을 공유하고자 합니다. 아마 이 글을 쓰는 와중에도 제가 적용한 방식이 최신이 아닌 이전 방식이 될수도 있습니다. 이 점을 참고하고 읽어주시면 감사하겠습니다.

​        

## 사전 준비

다음 세 가지를 준비해주세요. 테마 적용 과정과 에러 해결에 중점을 두기 때문에 다음 세 가지에 대한 자세한 설명은 생략하겠습니다. 

{: .prompt-tip }

> 구글에 자세하게 정리된 글이 많으므로 검색해서 참고하시길 바랍니다.



- `Git`, `Github`

- `Ruby` 를 설치해주세요. 제가 사용하고 있는 루비 버전은 `3.1.3p` 입니다. 설치가 완료되면 다음 명령과 같이 루비 버전이 출력되는지 확인해주세요.

```bash
Ruby -v
ruby 3.1.3p185 (2022-11-24 revision 1a6b16756e) [x86_64-darwin22]
```

- `jekyll`을 설치해주세요.

```
gem install jekyll bundler
```

​    

## Github에서 저장소 생성하기

모든 준비가 끝났다면 블로그 운영에 필요한 파일들을 저장할 저장소부터 생성합니다. 다음과 같이 본인의 깃허브 네임과 그 뒤에  `github.io` 을 붙여서 저장소 이름을 만들어주세요. 공개 범위는 `public`으로 설정합니다.

저는 이미 블로그를 위한 저장소가 존재하므로 다음과 같이 이미 해당 이름의 저장소가 존재한다고 메세지가 나옵니다.

![createRepository](https://user-images.githubusercontent.com/80873132/211809035-4518cb04-88c0-4a53-8a8b-2759cb7c9400.png){: width="972" height="589"}

​    

## 테마 파일들 가져오기

포크 방식과 압축파일을 다운하는 방법이 있는데 저는 두 번째 방법인 압축파일 다운 방식을 기준으로 설명하겠습니다.

### 두 가지 방법

1. [**테마 저장소**](https://github.com/cotes2020/jekyll-theme-chirpy) **를  `fork` 해서 `fork`한 저장소 이름을 `{본인의 깃허브 네임}.github.io`로 변경해서 사용**
   - 포크한 저장소를 `git clone`을 통해서 연결한다.
2. [**테마 저장소**](https://github.com/cotes2020/jekyll-theme-chirpy) **`Download zip`을 통해서 가져오기**
   - 저장소를 직접 만들고 `git clone`으로 연결한 후에 다운받은 폴더를 저장소와 연결된 로컬 폴더에 넣는다.

### zip 다운로드

![image](https://user-images.githubusercontent.com/80873132/211842094-6d9046ed-0bad-4588-905a-3e50e6bdaddf.png)



### 테마 파일 초기화

현재 다운받은 테마 파일들은 완전히 초기의 상태가 아니라 제작자에 의해서 계속해서 업데이트가 된 상태입니다. 따라서 본인만의 블로그를 세팅하려면 초기 상태로 만들어줘야 합니다. 초기화 코드는 다음과 같습니다. 해당 코드는 제작자의 시작 가이드에서 가져왔습니다.

```bash
bash tools/init
```

### 그런데 문제가....

테마 파일을 다운받으면 그 중에 `tools` 라는 이름의 폴더가 있는데 그 안에 `init` 이 존재합니다. 이 `init` 을 이용해서 초기화를 진행시키는 것입니다. 그런데 저는 아무리 시도해봐도 `tools` 디렉토리가 인식이 안되는 건지 초기화가 안됐습니다. 폴더가 존재하는데 디렉토리가 없다고 계속 오류가 발생해서 결국 터미널을 통한 초기화를 할 수 없었습니다.

{: .prompt-tip }

> bash 사용이 제한이 되거나 저처럼 초기화가 안되는 경우에는 **직접 테마 폴더를 건드려서 초기화**를 하면 됩니다.

### 직접 초기화?

직접 초기화는 다운받은 테마 폴더 내의 파일 혹은 폴더들을 수정하거나 삭제하는 방식입니다. 다른 블로그들의 글에서 삭제하라는 파일 혹은 폴더가 현재 다운받은 폴더 내에 없을 수 있습니다. 아마 이 부분도 제작자가 업데이트 하면서 일부는 없앤 것 같습니다. 그래서 있는 것들만 수정하거나 삭제하시면 됩니다.

### 수정 및 삭제 목록

{: .prompt-warning }

> 숨긴 파일들 표시를 꼭 해주세요.

​    

- `.github` 폴더 내에 `workflows` 폴더를 제외하고 모두 삭제
- `github/workflows` 내에 `commitlint.yml` 과 `page-deploy.yml.hook` 제외하고 모두 삭제
- `page-deploy.yml.hook` 파일의 `.hook` 부분을 없애서 `page-deploy.yml` 로 수정
- `_config.yml` 에서 `url: ''` 부분에 본인의 깃허브 블로그 주소 넣기

​    

---

​    

- `.github/workflows/page-deploy.yml` 에서 루비 버전을 로컬 버전이랑 맞춰주세요. `jobs:`  내의  `build:` 부분을 찾아보면 밑의 코드처럼 루비 설정에 관한 부분을 찾을 수 있습니다. 제가 사용하는 루비 버전은 `3.1.3` 이므로 `ruby-version: "" ` 에 `3.1.3`  를 적었습니다. 

```yaml
jobs:
  build:
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: "3.1.3" # reads from a '.ruby-version' or '.tools-version' file if 'ruby-version' is omitted
          bundler-cache: true
```



- `.github/workflows/page-deploy.yml` 에서 설정된 브랜치를 확인해주세요. 2번 라인에 밑의 코드를 확인할 수 있습니다. 연결된 저장소에 저 둘 중에 해당되는 브랜치가 있으면 됩니다. 제 저장소는 `main` 브랜치를 사용했습니다.

```yaml
on:
  push:
    branches:
      - main
      - master
    paths-ignore:
      - .gitignore
      - README.md
      - LICENSE
```

{: .prompt-tip }

> `page-deploy.yml` 에 등록된 브랜치와 루비 정보를 토대로 build가 진행됩니다.



## 깃허브 저장소에 PUSH





