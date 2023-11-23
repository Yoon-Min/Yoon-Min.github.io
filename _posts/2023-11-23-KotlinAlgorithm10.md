---
title: (프로그래머스 | C++) - 방금그곡
author: yoonmin
date: 2023-11-23 12:00:00 +0900
categories: [CS, 알고리즘 문제]
tags: [Kotlin, Algorithm, 알고리즘, 프로그래머스, 카카오 코딩테스트]
render_with_liquid: false
---

## 해결 방법

문제에서 주어진 조건을 가지고 시킨 것만 잘 수행하면 된다. `musicinfos` 에서는 `"12:00,12:14,HELLO,CDEFGAB"` 형식의 문자열을 원소로 가지고 있다. 이 문자열을 잘 분해해서 처리해야 하는데 순서는 다음과 같다.

1. 문자열을 `,` 기준으로 `split` 한 배열을 생성한다.  `["12:00", "12:14", "HELLO", "CDEFGAB"]` 

   ```c++
   vector<string> split(string str, char splitChar) {
       vector<string> result;
       istringstream iss(str);
       string buffer;
       while(getline(iss, buffer, splitChar)) {
           result.push_back(buffer);
       }
       return result;
   }
   
   for(string info: musicinfos) {
       vector<string> infoList = split(info, ',');
   }
   ```

2. `#` 이 붙은 음을 하나로 합치기 위해서 알파벳 소문자로 변환해준다. 예를 들어서 `C#` 이 있다면 `c` 이렇게 하나의 문자로 바꿔준다. 이렇게 해야 `ABC` 를 찾아야 하는데 `ABC#` 에서 `C#` 을 `C` 로 착각해서 `ABC` 가 있는 것으로 인식하는 변수를 방지할 수 있다.

   ```c++
   string convertSharpToLowercase(string str) {
       int i = 0;
       while(i < str.length()) {
           if(str[i] == '#') {
               char lowercase = tolower(str[i-1]);
               str.replace(i-1, 2, string(1, lowercase));
               continue;
           }
           i++;
       }
       return str;
   }
   ```

   

3. 실행시간을 정수형으로 구한다.  `12:00, 12:14` 의 수행시간은 총 14분이다.

   ```kotlin
   int getPlaytimeWithString(string start, string end) {
       int startForMinute = stoi(start.substr(0,2)) * 60 + stoi(start.substr(3));
       int endForMinute = stoi(end.substr(0,2)) * 60 + stoi(end.substr(3));
       return endForMinute - startForMinute;
   }
   ```

4. 실행시간 크기만큼 악보를 재구성한다. 예를 들어서  `CDEFGAB` 이게 기본 악보인데 1분당 1개의 음을 구성하므로 14분으로 재구성하면 `CDEFGABCDEFGAB` 이다.

   ```c++
   string getMelodyDuringPlaytime(string melodyOrigin, int playtime) {
       string melodyDuringPlaytime = "";
       if(melodyOrigin.length() <= playtime) {
           melodyDuringPlaytime = melodyOrigin;
           for(int i = 0; i < playtime - melodyOrigin.length(); i++) {
               melodyDuringPlaytime += melodyOrigin[i%melodyOrigin.length()];
           }
       }
       else {
           for(int i = 0; i < playtime; i++) {
               melodyDuringPlaytime += melodyOrigin[i];
           }
       }
       return melodyDuringPlaytime;
   }
   ```

5. 재구성한 악보에서 `m` 에 해당되는 구간이 있는지 찾는다. 이런 방식으로 `musicinfos` 의 모든 음악들을 비교하면 된다.

   ```c++
   bool isValid(string userMelody, string playtimeMelody) {
       return playtimeMelody.find(userMelody) != string::npos;
   }
   ```

   

## 전체 코드

```c++
#include <string>
#include <vector>
#include <sstream>
#include <iostream>
#include <algorithm>
using namespace std;

vector<string> split(string str, char splitChar) {
    vector<string> result;
    istringstream iss(str);
    string buffer;
    while(getline(iss, buffer, splitChar)) {
        result.push_back(buffer);
    }
    return result;
}

int getPlaytimeWithString(string start, string end) {
    int startForMinute = stoi(start.substr(0,2)) * 60 + stoi(start.substr(3));
    int endForMinute = stoi(end.substr(0,2)) * 60 + stoi(end.substr(3));
    return endForMinute - startForMinute;
}

string convertSharpToLowercase(string str) {
    int i = 0;
    while(i < str.length()) {
        if(str[i] == '#') {
            char lowercase = tolower(str[i-1]);
            str.replace(i-1, 2, string(1, lowercase));
            continue;
        }
        i++;
    }
    return str;
}

string getMelodyDuringPlaytime(string melodyOrigin, int playtime) {
    string melodyDuringPlaytime = "";
    if(melodyOrigin.length() <= playtime) {
        melodyDuringPlaytime = melodyOrigin;
        for(int i = 0; i < playtime - melodyOrigin.length(); i++) {
            melodyDuringPlaytime += melodyOrigin[i%melodyOrigin.length()];
        }
    }
    else {
        for(int i = 0; i < playtime; i++) {
            melodyDuringPlaytime += melodyOrigin[i];
        }
    }
    return melodyDuringPlaytime;
}

bool isValid(string userMelody, string playtimeMelody) {
    return playtimeMelody.find(userMelody) != string::npos;
}

string solution(string m, vector<string> musicinfos) {
    string targetTitle = "(None)";
    int targetPlaytime = -1;
    for(string info: musicinfos) {
        vector<string> infoList = split(info, ',');
        int playtime = getPlaytimeWithString(infoList[0], infoList[1]);
        if(playtime <= targetPlaytime) continue;

        string melodyOrigin = convertSharpToLowercase(infoList[3]);
        string melodyDuringPlaytime = getMelodyDuringPlaytime(melodyOrigin, playtime);
        if(isValid(convertSharpToLowercase(m), melodyDuringPlaytime)) {
            targetPlaytime = playtime;
            targetTitle = infoList[2];
        }
    }
    return targetTitle;
}
```

