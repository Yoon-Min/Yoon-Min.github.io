---
title: (프로그래머스 | Kotlin) - 매칭 점수
author: yoonmin
date: 2023-11-19 21:00:00 +0900
categories: [CS, 알고리즘 문제]
tags: [Kotlin, Algorithm, 알고리즘, 프로그래머스]
render_with_liquid: false
---

## 해결 방법

정말 보자마자 머리가 지끈거리는 문제였다. 카카오 코테 문제를 풀어보면서 문자열 처리 로직을 많이 작성했었는데 설마 `html` 형식을 통째로 주는 건 상상도 못했다. 필요한 건 크게 `url`, 키워드, 외부 링크, 이렇게 세 가지다. 

여기서 문제는 해당 세 가지를 구하기 위해서 정규식을 사용해야 하는데 나는 정규식을 사용할 때 항상 검색해서 복붙한 게 전부여서 식을 어떻게 작성해야 하는지 몰랐다. 그래서 검색을 해서 몇몇 정규식을 참고했다.

### 1. 현재 페이지의 URL

```
"(<meta property=\"og:url\" content=\"(\\S*)//(\\S*)\"/>)"
```

페이지의 `url` 을 구할 때 중요한 점은 문제에서 준 양식과 정확히 일치해야 한다는 것이다. 보니까 `url` 부분을 포함해서 조금이라도 양식에 벗어나는 예제가 있는 것 같다.

### 2. `html` 페이지 내 키워드 수

```
"[^a-zA-Z]"
```

키워드 갯수는 알파벳이 아닌 문자로 둘러싸인 키워드를 찾으면 되므로 알파벳이 아닌 문자를 기준으로 `split` 해서 키워드 갯수를 구하는 방식으로 구현했다.

### 3. 외부링크 갯수

```
"<a href=\"https://(.+?)\">"
```

현재 페이지의 `url` 구하는 것처럼 외부 링크 태그 양식을 정확히 지켜야 한다. 외부링크는 여러 개 존재할 수 있기 때문에 하나만 `find` 하면 안되고 `findAll` 해야 한다.

​		

## 전체 코드

위의 세 가지를 구현하기 위해 필요한 정규식만 만들줄 알면 나머지는 구현하는 데 크게 어렵지 않다. 

```kotlin
class Solution {
    fun solution(word: String, pages: Array<String>): Int {
        var answer = 0
        var maxScore = -1.0
        val pageInfo = ArrayList<WebInfo>()
        val pageIndex = mutableMapOf<String, Int>()
        pages.forEach { html ->
            getSelfUrl(html)?.let { selfUrl ->
                val keyword = getKeywordCount(html.lowercase(), word.lowercase())
                val extLink = getLinkCount(html)
                pageIndex[selfUrl] = pageInfo.size
                pageInfo.add(WebInfo(selfUrl, keyword, extLink, mutableSetOf(), pageInfo.size))
            }
        }

        pageInfo.forEach { webInfo ->
            webInfo.extLink.forEach { extLink ->
                if(pageIndex.containsKey(extLink)) {
                    pageInfo[pageIndex[extLink]!!].linker.add(webInfo.url)
                }
            }
        }

        pageInfo.forEachIndexed { i, webInfo ->
            var webScore = webInfo.basic.toDouble()
            webInfo.linker.forEach { linker ->
                webScore += pageInfo[pageIndex[linker]!!].getSingleLinkScore()
            }
            if(maxScore < webScore) {
                maxScore = webScore
                answer = i
            }
        }
        return answer
    }

    private fun getSelfUrl(html: String): String? {
        var urlTag: String? = null
        Regex("(<meta property=\"og:url\" content=\"(\\S*)//(\\S*)\"/>)")
            .find(html)
            ?.value
            ?.let {
                urlTag = it.split(" ")[2].split("\"")[1]
            }
        return urlTag
    }

    private fun getKeywordCount(html: String, keyword: String): Int {
        return html
            .split(Regex("[^a-zA-Z]"))
            .filter { it == keyword }
            .size
    }

    private fun getLinkCount(html: String): MutableSet<String> {
        val linker = mutableSetOf<String>()
        Regex("<a href=\"https://(.+?)\">")
            .findAll(html)
            .forEach { linkTag ->
                linker.add(linkTag.value.split(" ")[1].split("\"")[1])
            }
        return linker
    }

    data class WebInfo(
        val url: String,
        val basic: Int,
        val extLink: MutableSet<String>,
        val linker: MutableSet<String>,
        val index: Int
    ) {
        fun getSingleLinkScore(): Double {
            return if (extLink.size == 0) 0.0 else basic.toDouble() / extLink.size
        }
    }
}
```

​		

## 참조

[**외부링크 정규식 참고**](https://stritegdc.tistory.com/286)

[**키워드 갯수와 페이지 URL 정규식 참고**](https://j-sik.tistory.com/120)