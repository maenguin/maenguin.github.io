---
title: "알고리즘 리뷰"
categories:
  - algorithm
tags:
  - 알고리즘
---

코딩테스트를 준비하면서, 무작정 문제를 많이 풀고 넘어가는것에 그치지 않고 사용했던 기술이나 느낀점을 기록해놓기로 하였다.


## 형변환

hashMap to int[]
```java
hashMap.entrySet()
      .stream()
      .map(s -> s.getKey())
      .sorted()
      .mapToInt(value -> value)
      .toArray()
```
