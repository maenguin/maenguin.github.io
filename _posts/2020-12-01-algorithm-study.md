---
title: "알고리즘 리뷰"
categories:
  - algorithm
tags:
  - 알고리즘
---

코딩테스트를 준비하면서, 무작정 문제를 많이 풀고 넘어가는것에 그치지 않고 사용했던 기술이나 느낀점을 기록해놓기로 하였다.


## 형변환

### int array로 반환해야 할때

1. 스트림이용(속도가 느림)
```java
collection.stream()
         .mapToInt(Integer::intValue)
         .toArray()
```


2. iterator 이용
```java
int[] answer = new int[size()];
int index = 0;
Iterator itor = collection.iterator();
while(itor.hasNext()){
    answer[index++] = (int)itor.next();
}
```

## 깔끔한 코드

* switch말고 배열과 %
```java
String[] dayofWeek = {"THU","FRI","SAT","SUN","MON","TUE","WED"};
String answer = dayofWeek[sumOfDays%7];
```





