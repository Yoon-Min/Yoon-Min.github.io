---
title: Github 블로그 로컬 에러 - Address already in use - bind(2) for 127.0.0.1:4000 (Errno::EADDRINUSE)
author: yoonmin
date: 2025-02-06 00:00:00 +0900
categories: [블로그, Github 블로그 만들기]
tags: [GitHub, Blog, 깃허브 블로그, 로컬 서버 구동 오류]
render_with_liquid: true
---

## 로컬 주소로 구동중에 실수로 비정상 종료를 했다면?

깃허브 블로그 포스팅을 위해 로컬 주소를 구동하던 중 실수로 `Control + Z`와 같은 비정상 종료가 발생한다면 높은 확률로 이후 로컬 서버 구동 시도에서 다음과 같은 오류가 발생할 것입니다. 강제로 구동을 멈췄을 뿐이지 프로세스는 아직 활동하고 있다는 뜻입니다.

```
Address already in use - bind(2) for 127.0.0.1:4000
```

​		

## 해결 방법 - 프로세스를 종료하자

해결 방법은 해당 로컬 주소 구동을 도와주는 프로세스를 강제로 종료하는 것입니다.

1. **현재 로컬 주소 구동을 담당하는 프로세스 탐색 (TCP 포트에 따라서 3000 혹은 4000 구분)**

   ```bash
   lsof -wni tcp:4000
   ```

2. **해당하는 프로세스의 아이디 찾기**

   ```bash
   COMMAND    PID        USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
   Google    2592 yunseungmin   52u  IPv4 0xd803a33c04f60f39      0t0  TCP 127.0.0.1:50069->127.0.0.1:terabase (ESTABLISHED)
   ruby      7749 yunseungmin    9u  IPv4 0x49f28a5832038ac0      0t0  TCP 127.0.0.1:terabase (LISTEN)
   ruby      7749 yunseungmin   16u  IPv4 0x74a9e2ce917ca071      0t0  TCP 127.0.0.1:terabase->127.0.0.1:49806 (CLOSED)
   ruby      7749 yunseungmin   17u  IPv4   0xbe0b1fa1b6622a      0t0  TCP 127.0.0.1:terabase->127.0.0.1:49809 (CLOSED)
   ruby      7749 yunseungmin   18u  IPv4 0x89f35f619c860ce7      0t0  TCP 127.0.0.1:terabase->127.0.0.1:49811 (CLOSED)
   ```

3. **프로세스 킬**

   ```bash
   kill -9 7749
   ```

​		

## 참조

[**How to Fix "Address already in use – bind(2) for “127.0.0.1” port 3000 (Errno::EADDRINUSE)"**](https://testsuite.io/fix-address-already-in-use)